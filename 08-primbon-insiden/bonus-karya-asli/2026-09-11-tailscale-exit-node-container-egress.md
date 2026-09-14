---
aliases:
  - "Incident 2026-09-11 - Tailscale Exit Node vs Container Egress"
  - "Production login 500 caused by exit node"
tags:
  - incident
  - tailscale
  - docker-swarm
  - networking
---

# 2026-09-11 — Tailscale Exit Node vs Container Egress

Related: [Incident Hub](README.md), [Volume and Network Conventions](../03-ARCHITECTURE/docker/volume-and-network-conventions.md), [Swarm Stack Patterns](../03-ARCHITECTURE/docker/swarm/swarm-stack-patterns.md)

---

## 1. Summary

| | |
|---|---|
| **Date** | 11 September 2026 |
| **Window** | 14:23 – 16:33 WIB (≈ 2h 10m) |
| **Impact** | Production login for kasir BE fully down. Every auth endpoint returned HTTP 500 |
| **Host** | `<worker-host>` (Docker Swarm worker, public `<public-ip-worker>`) |
| **Detected by** | User complaint — **not** by monitoring |
| **Resolved by** | Toggling the Tailscale exit node off and on |
| **Root cause** | A provider-side network flap on the VPS link (see §6, *Trigger — found*) |

**Exact symptom pattern — and this is what made it misleading:**

- `POST /api/v1/auth/login` → 500 (22×)
- `POST /api/v1/field-ops/auth/login` → 500 (3×)
- `GET /api/v1/auth/me` → 500
- `GET /api/health` → **200** ✅
- `GET /up` → **200** every 30 seconds ✅
- CORS preflight (OPTIONS) → **204** ✅
- Service `<backend-service>` → **1/1 healthy**, zero restarts, 24h uptime

Every signal normally used to judge health said "healthy". The only thing broken was anything that needed to leave the container.

---

## 2. Mechanism — what actually happened

The worker uses a **Tailscale exit node** (`<exit-node-host>`, public IP `<public-ip-exit-node>`) so that outbound traffic carries the IP registered on the payment gateway's whitelist.

Tailscale implements this through policy routing rules:

```
5210: from all fwmark 0x80000/0xff0000 lookup main
5230: from all fwmark 0x80000/0xff0000 lookup default
5250: from all fwmark 0x80000/0xff0000 unreachable
5270: from all lookup 52            ← everything else
32766: from all lookup main
```

And table 52, while the exit node is active:

```
default dev tailscale0
```

**Here is the crux:** traffic **originating on the host** is marked with `fwmark 0x80000`, so it escapes via rules 5210/5230 and uses the `main` table. Container traffic is **forwarded**, never receives that mark, and therefore falls through to rule `5270` → table 52 → `default dev tailscale0`.

As long as table 52 is built correctly, this is fine. What happened on 11 September: **the build went out of sync**, so reply packets addressed to container IPs were also pushed out through `tailscale0`.

### Raw evidence — tcpdump

```
1. veth0048d7d     P   172.18.0.8.34924 > 1.1.1.1.53      [S]   ← container sends
2. docker_gwbridge In  172.18.0.8.34924 > 1.1.1.1.53      [S]   ← enters bridge
3. tailscale0      Out <tailnet-ip-worker>.34924 > 1.1.1.1.53   [S]   ← leaves via exit node (SNAT)
4. tailscale0      In  1.1.1.1.53 > <tailnet-ip-worker>.34924   [S.]  ← reply arrives
5. tailscale0      Out 1.1.1.1.53 > 172.18.0.8.34924      [S.]  ← ❌ REPLY SENT BACK OUT
```

Line 5 is the failure. The reply had already been de-NATed to `172.18.0.8` — a container address on `docker_gwbridge` — yet it was sent out through `tailscale0`. The packet was lost; the container waited until timeout.

**Knock-on effect:** `<db-host>` (which hosts **both** Redis and Postgres) became unreachable from the container → login failed → HTTP 500. Endpoints touching no dependency stayed at 200.

---

## 3. Diagnostic traps

The most important part of this record. All five guesses below were **wrong**, and all five looked convincing at the time.

### Trap 01 — Auth-shaped symptoms read as an auth problem

**Assumption:** JWT, credentials, or the Redis blacklist must be broken. Every failing endpoint was an auth endpoint, so the fault must live in auth.

**Reality:** it had nothing to do with auth. Auth endpoints simply happened to be **the only ones needing an outbound connection** (Redis for session/JWT blacklist, Postgres for user records). `/api/health` and `/up` touch nothing, so they stayed green.

**Why it misleads:** the failure pattern was clean and consistent — all auth down, everything else up. A pattern that tidy invites the conclusion "something is wrong in the auth module", when what was actually being mapped is **which endpoints need the network**.

**Countermeasure:** do not group failing endpoints by feature. Group them by **dependency**.

---

### Trap 02 — Testing from the manager, assuming it represents the worker

**Assumption:** Redis was tested from `prod-swarm-manager` — `AUTH +OK`, `PING +PONG`, 1.89 MB memory, no limit. Postgres answered its handshake too. Conclusion: "Redis and the database are healthy, strike them off the suspect list."

**Reality:** the manager was indeed healthy. The failing container runs on the **worker** — a different machine with a different routing state.

**Why it misleads:** the result was correct; it just answered the wrong question. And because the result was positive, the actual culprit (the path to Redis) was eliminated early.

**Countermeasure:** test **from the place experiencing the problem**, not from the place easiest to reach. If that place is inaccessible, record it as *not yet tested* — do not substitute a result from somewhere else.

---

### Trap 03 — Reading `iptables` counters at face value

**Assumption:** the `FORWARD` chain showed 4,779K packets traversing `DOCKER-USER`, yet every per-rule counter inside `DOCKER-USER` was near zero. Conclusion: "this chain must have just been rewritten by a firewall script."

**Reality:** two separate problems —
- The columns in `iptables -L -n -v` are **rounded to thousands** (`4779K`). A change of a few packets is invisible. `-x` is required for exact numbers.
- The per-rule counters really were small, but not because of a flush — because **almost no forwarded traffic was passing at all**, which was the very failure being investigated.

**Why it misleads:** numbers feel objective. But rounding silently erases signals as small as a manual test connection, and "small counter" has at least two very different explanations.

**Countermeasure:** use `iptables -L -n -v -x` when watching for small deltas. And never infer a cause from a single number that has multiple explanations.

---

### Trap 04 — ARP, bridge, conntrack, `ip_forward`

**Assumption:** in sequence — ARP to the gateway failing, `docker_gwbridge` down, veth detached from the bridge, conntrack table full, `ip_forward` disabled, `DOCKER-USER` rules blocking.

**Reality:** all healthy.

| Checked | Result |
|---|---|
| ARP gateway `172.18.0.1` | `0x2` REACHABLE ✅ |
| `docker_gwbridge` | UP, 7 members ✅ |
| conntrack | 89 / 262144 ✅ |
| `ip_forward` | `1` ✅ |
| chain `ts-forward` | present, `-o tailscale0 -j ACCEPT` ✅ |
| `DOCKER-USER` | no DROP rule matching container traffic ✅ |

**Why it misleads:** each is a plausible candidate for "silent timeout with no trace". Checking them one by one feels like progress — but the only thing growing is the list of things that are *not* the cause.

**Countermeasure:** when three guesses miss in a row, **stop guessing**. Go get direct evidence.

---

### Trap 05 — Treating `tailscale debug prefs` as a statement of current state

**Assumption:** check `tailscale debug prefs` to confirm the exit node configuration is correct.

**Reality:** the output was **identical** when broken and when healthy:

```
"ExitNodeID": "<node-id>"     ← byte-for-byte identical, broken or healthy
"RouteAll": false
"ExitNodeAllowLANAccess": true
```

**Why it misleads:** `prefs` stores **intent**, not **state**. What actually moves packets is the translation of that intent into `ip rule` + table 52 — and **the translation** was what broke. The source of truth lives elsewhere.

**Countermeasure:** to judge state, inspect `ip rule` and `ip route show table 52`. Better still: test actual function (can the container reach Redis), rather than reading configuration.

---

## 4. What finally broke it open: tcpdump

After five misses, a single command resolved it in seconds:

```bash
CID=$(docker ps -qf name=<backend-service>)
timeout 8 tcpdump -nn -i any -c 6 'host 1.1.1.1' 2>/dev/null &
sleep 1
docker exec $CID php -r '@fsockopen("1.1.1.1",53,$e,$s,3);'
wait
```

`-i any` is the key — it shows the same packet crossing veth, bridge, and `tailscale0`, with direction on every line. Once line 5 appeared, there was nothing left to guess.

**Lesson on ordering:** for "silent timeout with no trace" symptoms, **tcpdump first, firewall second**. The firewall answers "was it blocked"; tcpdump answers "where did the packet go" — and the second question is far more often the one that matters.

---

## 5. Quick check & remedy

**If it recurs** — three lines, immediate answer:

```bash
CID=$(docker ps -qf name=<backend-service>)
ip route show table 52 | head -3
docker exec $CID php -r '$f=@fsockopen("<tailnet-ip-db>",6379,$e,$s,3); echo "REDIS: ".($f?"OK":"FAIL")."\n";'
docker exec $CID php -r 'echo "EGRESS-IP: ".trim(@file_get_contents("http://ifconfig.me/ip") ?: "FAIL")."\n";'
```

**Healthy** = `REDIS: OK` plus the expected egress IP (`.225` with the exit node on, `.228` without).

**Remedy** — toggle the exit node:

```bash
tailscale set --exit-node=
tailscale set --exit-node=<exit-node-hostname-or-ip>
```

The configuration does not change at all; what happens is that tailscaled **rebuilds** table 52 and the `ip rule` set from scratch. No Docker restart, no container restart, no additional downtime.

---

## 6. What is still open

**The trigger has since been found** (added 11 Sep, evening). This section is kept in its original shape because the reasoning it records — the ruled-out list, and the wrong class of cause it pointed to — is the useful part. The answer is at the end, under *Trigger — found*.

At the time of writing: only the failure mechanism was proven. Why routing went stale was unanswered, and until answered, **the incident could recur**.

### Correction to the timeline

The window in section 1 (14:23) is **when the first login was attempted**, not when the breakage began. Full login history from the retained service log:

```
09-10 18:13 → 22:27   5× HTTP 200          ← healthy
09-10 22:27:49        last success
        ⋯ 16 hours with zero login attempts ⋯
09-11 14:23:53        first attempt after the gap → 500
09-11 16:28:31        first 200 after the fix
```

So the breakage happened somewhere inside that **16-hour gap** (22:27 on 10 Sep → 14:23 on 11 Sep), overnight. Note that logins were still healthy on the evening of 10 Sep, *after* the the payment gateway/QRIS work of that afternoon.

### Ruled out by evidence

Whatever changed the routing did so **without restarting anything**:

| Hypothesis | Evidence |
|---|---|
| Container/task rescheduled on the worker overnight | No. Backend 26h, frontend 27h, everything else days–weeks |
| tailscaled restarted on the worker | No. Active since 3 Sep |
| tailscaled restarted on the exit node (manager) | No. Active since 3 Sep |
| Manager rebooted | No. 7 weeks uptime |
| Docker daemon restarted | No. Active since 20 Jul |
| Tailscale events on the manager overnight | None in journal |

That leaves one class of cause capable of reconfiguring routes with nothing restarting: a **control-plane netmap update** — an exit-node selection change, an ACL edit, or a device joining/leaving the tailnet. Plausible, but **not proven**.

### The one unread source

The worker's own tailscaled journal for that window. Not accessible from the manager (no SSH path to the worker; the only open route is from `kasir-app-s01`, where the key is not authorised).

```bash
journalctl -u tailscaled --since "2026-09-10 22:00" --until "2026-09-11 15:00" --no-pager \
  | grep -iE "exit|route|netmap|reconfig|peer|linkchange" | tail -25
```

If that window shows the routes being rebuilt, the trigger is identified. If it is empty, the honest conclusion is that the trigger left no readable trace.

**The current healthy state is the result of a toggle, not a root-cause fix.**

### Trigger — found

The journal above was read from a console on the worker itself. It was not empty.

```
Sep 11 04:35:52  control: netmap: got new dial plan from control
Sep 11 04:35:52  netmap: suggested exit node:  ()
Sep 11 04:36:54  netmap: suggested exit node: <exit-node-host> (<node-id>)
Sep 11 05:21:30  control: netmap: got new dial plan from control
Sep 11 05:21:30  netmap: suggested exit node:  ()
Sep 11 05:22:28  netmap: suggested exit node: <exit-node-host> (<node-id>)
Sep 11 06:54:21  monitor: RTM_DELROUTE: dst=<public-subnet>, table=52
Sep 11 06:54:21  monitor: RTM_DELROUTE: dst=127.0.0.0/8,      table=52
Sep 11 06:54:21  monitor: RTM_DELROUTE: dst=172.17.0.0/16,    table=52
Sep 11 06:54:21  monitor: RTM_DELROUTE: dst=fe80::/64,        table=52
Sep 11 06:54:21  monitor: RTM_DELROUTE: dst=172.18.0.0/16,    table=52
Sep 11 06:54:21  router: somebody (likely systemd-networkd) deleted ip rules; restoring Tailscale's
```

The causal chain, from the bottom up:

1. **The VPS provider's network glitched.** This is the root cause. The tell is the pair of `got new dial plan from control` at 04:35 and 05:21 — tailscaled re-establishing its control connection twice within the hour, each time with the netmap briefly empty (`suggested exit node: ()`) before repopulating. That is a connectivity flap fingerprint, not a configuration change. Corroborating: **no ACL or configuration change was made in Tailscale**, and the same class of flap is a familiar recurring event on the operator's other Tailscale hosts on ordinary consumer links. Tailscale is not the defective component; the link underneath it is.
2. **systemd-networkd reacted to the link event** and deleted the routing state it does not own — the `throw` routes tailscaled had installed in table 52 for `127.0.0.0/8`, `<public-subnet>`, `172.17.0.0/16`, `172.18.0.0/16`, `fe80::/64`.
3. **tailscaled restored the `ip rules` but not the `throw` routes.** Its own log says so explicitly: *"deleted ip rules; restoring Tailscale's"* — rules only. This asymmetry is what made the outage persistent rather than self-healing, and it is why `ip rule` looked perfectly healthy during diagnosis while table 52 was gutted.
4. **Table 52 was left holding only `default dev tailscale0`.** Forwarded container traffic carries no `fwmark 0x80000`, so rule 5270 sent it into table 52, and with the `throw` escape hatches gone it went into the tunnel. Replies to `172.18.0.8` never returned. This is the mechanism in section 2.

Everything in steps 2–4 is a **link in the chain**, not the disease. Fixing any one of them — pinning `throw` routes, hardening networkd, alarming on table 52 — reduces blast radius but leaves the actual cause untouched: an unreliable provider link that will flap again.

**Diagnostic trap 06 — mistaking a link in the chain for the root cause.** Over roughly an hour of questioning, six successive answers were offered: stale routing, exit node being toggled, exit node enabled that morning, systemd-networkd deleting the routes, the exit-node dependency itself, and Redis not sharing an overlay network. Every one of them was factually true and verifiable in the evidence. Not one was the cause. A chain of true statements reads exactly like an explanation, and the deepest verifiable link feels like bedrock — especially when it is the one with a log line pointing at it. The discipline that was missing: after naming a cause, ask *what made that happen*, and keep asking until the answer leaves the machine. Here the answer left the machine at step 1, which is precisely why no amount of reading logs on the host would ever have produced it.

**Note on the ruled-out table above.** It is still correct, and it is what made the trigger findable: by eliminating every restart-shaped explanation, it forced the search toward something that reconfigures routing with nothing restarting. It named the wrong candidate for that class — a control-plane netmap update — but it was pointing in the right direction. A netmap update did occur; it was a *symptom* of the flap, not the cause of the route deletion.

**There is no alarm.** This incident surfaced through a user complaint after two hours. Not a single monitor fired — because every existing health check inspects something that stayed healthy throughout.

---

## 7. Architectural debt

### Why the exit node exists

the payment gateway (QRIS provider) enforces an **IP whitelist**. The registered address is `<public-ip-exit-node>` — the manager's public IP. The worker (`<public-ip-worker>`) is not registered. To make container traffic leave with the registered IP, the worker was routed through the exit node.

An easily missed detail: the env var `PG_OUTBOUND_IP_ADDRESS=<public-ip-exit-node>` is sent as the `X-IP-ADDRESS` header on every request. With the exit node off, the header claims `.225` while packets arrive from `.228` — **claiming something different from reality**, which can be rejected independently of the whitelist.

### Options for removing the dependency

| Option | Upside | Downside |
|---|---|---|
| **Ask the payment gateway to whitelist the worker IP / subnet** | Zero engineering. Exit node can be removed entirely | Depends on another party's schedule |
| **Egress proxy on the registered-IP node** | Only the payment gateway traffic is special-cased. Survives reboots | One more component to maintain |
| **Swarm placement constraint to the registered node** | No proxy, no routing tricks | Service placement becomes hostage to a networking concern |
| **Destination-specific routing for the payment gateway** | Closest to the current setup | **Fragile** — the payment gateway changes IP, it breaks again |

---

## 8. General lesson

**Do not patch an application-level requirement at the host routing layer.**

Exactly **one destination** needed special treatment (the payment gateway). The patch applied: force **all machine traffic** to detour through another node. The side effect landed on something entirely unrelated — login was down for two hours.

The further apart **where the problem lives** and **where the patch is applied**, the larger and less predictable the side effects. A good patch sits as close to the problem as possible: a proxy and a placement constraint sit next to the application; an exit node sits in the kernel.

**Corollary:** when a solution feels like a sledgehammer — "all X now goes through Y" — first verify that the thing needing special treatment really is all of X, and not one in a thousand.

### Second lesson — stop calling the deepest link in the chain the root cause

Every explanation offered during this incident was true. Routing was stale. systemd-networkd did delete the routes. The exit-node dependency is real architectural debt. Redis genuinely is not on a shared overlay. Each one survived scrutiny, each one had evidence behind it, and each one was wrong as a root cause.

A causal chain of true statements is indistinguishable from an explanation while you are standing inside it. The deepest link you can verify **feels** like bedrock — especially the link that a log line is pointing at, because a log line reads as a verdict.

The test that separates them: **can this thing have been caused by something else?** If yes, keep going. Here, "networkd deleted the routes" had an obvious parent — *why did networkd reconfigure at 06:54 on an idle machine?* — and that question was never asked. The answer was not on the machine at all. It was the link underneath it.

**Practical form:** when every answer comes from the same host's logs, suspect that the cause is not on that host. Machines faithfully record what happened *to* them; they rarely record what happened *around* them.

### The rule this incident exists to teach

> **When software shows no anomaly, the anomaly is below software.**

Read the diagnostic record of this incident and notice what every single check had in common: `ip rule`, `iptables` counters, the ARP table, conntrack, `docker events`, service journals, container health. All of them software. All of them clean. Hours were spent adding *depth* within that layer instead of *changing* layer.

Clean software telemetry is not the absence of a problem — it is a **finding**, and what it points at is the layer underneath: the link, the NIC, the provider's network, the disk, the power. Those layers mostly do not write logs. Their presence is inferred from what they leave behind in the layer above — a control-plane reconnect, a DHCP renew, a carrier event, a route rewrite — which is exactly why those artefacts get misread as causes.

**Operational form:** after two or three software-layer hypotheses come back clean, stop generating a fourth. Ask instead whether anything physical or external changed in that window, and treat "nothing was configured, nothing was deployed, nothing restarted" as strong evidence *for* that possibility rather than as a dead end.
