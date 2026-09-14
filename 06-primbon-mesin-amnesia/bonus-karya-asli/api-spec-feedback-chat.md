> **Bonus karya — salinan asli dari lumbung.** Spesifikasi API beneran yang ditulis kuli QA pemilik buat rekan FE-nya, disalin atas izin pemilik sebagai contoh hidup entri 06-04: dokumen yang ngomelin pembacanya duluan. Penyamaran: nama pemilik → "pemilik", nomor tiket internal → placeholder, nama kanal tim disamarkan. Endpoint, contoh data, dan seluruh jebakannya utuh.

# API Spec — Feedback Chat (Jempol Atas/Bawah per Jawaban Bot)

> **Buat FE yang megang dokumen ini:** lucunya, dokumen tentang tombol jempol ini sendiri gak punya tombol jempol. Jadi kalau lu males baca terus lempar mentah ke AI dan hasilnya ngaco, gak ada yang bisa lu kasih jempol bawah — yang nongol tiket bug, atas nama lu. Baca sampe ngerti. Ngerti itu kerjaan lu, bukan kerjaan model yang lu suruh. — Kuli QA, atas nama pemilik

> ⚠️ **BELUM ADA DI SERVER.** Dokumen ini ditulis **sebelum** kodenya dibikin, beda dari spec-spec sebelah yang statusnya LIVE. Jangan integrate sekarang — nembak `PUT /chat/message/feed` hari ini balasannya `404`, dan itu bukan bug, itu emang belum ada. Tunggu tiket `<TIKET-FTR>` PASS. Dokumen ini bakal diperbarui statusnya kalau udah, lengkap sama nomor commit.

**Tiket:** `<TIKET-FTR>` [MED] — `docs/backlog/<TIKET-FTR>.md`
**Permukaan:** API doang. Worker gak disentuh, gak ada migrasi DB, gak ada tabel baru.

Ada **dua** perubahan: satu endpoint lama nambah field, satu endpoint baru.

---

## 1. YANG BERUBAH — `GET /chat/history/{session_id}`

Endpoint, auth, pagination, urutan, semua **tetap sama** kayak `api-spec-<TIKET-ENC>.md`. Yang berubah cuma isi tiap item di `messages[]`: **nambah 3 field**.

### Response — 200 OK

```json
{
  "session_id": "9f1c2e40-5b7a-4c31-9d88-3a02b1e4f7c5",
  "messages": [
    {
      "id": "0b4d9c7e-1f52-44a8-9e77-6c1b8d3a2f90",
      "role": "user",
      "content": "produk apa yg paling laris minggu ini",
      "created_at": "2026-09-14T01:12:03.441820Z",
      "feedback": null,
      "feedback_reason": null
    },
    {
      "id": "7e2a55b1-8c94-4d6f-a013-92fb7c40de18",
      "role": "assistant",
      "content": "Minggu ini yang paling laris Indomie Goreng, 142 pcs.",
      "created_at": "2026-09-14T01:12:11.208377Z",
      "feedback": "bad",
      "feedback_reason": "angkanya salah, harusnya 142 itu bulan bukan minggu"
    }
  ],
  "chat_count_today": 4,
  "chat_limit": 25,
  "has_more": false
}
```

### Field baru di `messages[]`

| Field | Tipe | Catatan |
|---|---|---|
| `id` | string (UUID) | **BARU.** Pengenal pesan. Ini yang dikirim balik pas mencet jempol. Sebelum ini gak ada — FE cuma pegang teks + jam, jadi gak punya cara nyebut "pesan yang mana" |
| `feedback` | `"good"` \| `"bad"` \| `null` | **BARU.** `null` = user belum pernah mencet, ATAU udah dicabut. Dua-duanya `null`, gak dibedain |
| `feedback_reason` | string \| null | **BARU.** Alasan yang ditulis user. `null` kalau gak diisi |

Field lama (`role`, `content`, `created_at`) **gak berubah** sama sekali.

### Tiga hal yang bikin FE salah kalau gak dibaca

**1. Pesan `role: "user"` juga dapet ketiga field itu, dan nilainya SELALU `null`.**
Ini disengaja — satu schema dipakai buat dua peran, biar gak ada dua bentuk objek yang beda. **FE yang nyembunyiin tombol jempol di pesan user**, bukan server yang ngilangin field-nya. Jangan nampilin tombol cuma karena field-nya ada.

**2. `feedback: null` bukan berarti "belum pernah disentuh".**
Bisa juga artinya user udah mencet terus dicabut lagi. Server gak nyimpen bedanya. Kalau FE mau nampilin "udah pernah kasih feedback", itu gak bisa dijawab dari sini.

**3. `id` itu BUKAN `request_id`.**
`request_id` yang FE terima pas `POST /chat` itu benda lain, UUID yang beda. Jangan dipakai buat ngerating — bakal `404`. Ambil `id` dari response history ini.

---

## 2. YANG BARU — `PUT /chat/message/feed`

**Endpoint:** `PUT /chat/message/feed`
**Auth:** `Authorization: Bearer <JWT>` (required)

Satu pintu buat **tiga** aksi: kasih feedback, ubah feedback, cabut feedback. Gak ada endpoint `DELETE` terpisah — nyabut = kirim `feedback: null`.

### Kapan dipanggil

Setiap kali user mencet jempol atas / jempol bawah di sebuah jawaban bot, atau mencet lagi buat ngebatalin.

### Request Body

```json
{
  "message_id": "7e2a55b1-8c94-4d6f-a013-92fb7c40de18",
  "feedback": "bad",
  "reason": "angkanya salah, harusnya 142 itu bulan bukan minggu"
}
```

| Field | Tipe | Wajib | Catatan |
|---|---|---|---|
| `message_id` | string (UUID) | ya | Dari `messages[].id` di response history. Harus pesan `role: "assistant"` |
| `feedback` | `"good"` \| `"bad"` \| `null` | ya | `null` = cabut. Field-nya tetap wajib ada, nilainya yang `null` — jangan dihilangkan dari body |
| `reason` | string \| null | tidak | Maks **500 karakter**. Boleh diisi di `good` maupun `bad` |

**`reason` boleh di dua-duanya.** Bukan cuma buat `bad`. Kalau user suka jawabannya dan mau nulis kenapa, diterima.

### Response — 200 OK

```json
{
  "message_id": "7e2a55b1-8c94-4d6f-a013-92fb7c40de18",
  "feedback": "bad",
  "feedback_reason": "angkanya salah, harusnya 142 itu bulan bukan minggu",
  "feedback_at": "2026-09-14T01:14:52.003914Z"
}
```

| Field | Tipe | Catatan |
|---|---|---|
| `message_id` | string (UUID) | sama dengan yang dikirim |
| `feedback` | `"good"` \| `"bad"` \| `null` | nilai yang **tersimpan** setelah operasi ini |
| `feedback_reason` | string \| null | |
| `feedback_at` | ISO8601 \| null | kapan feedback terakhir disentuh. `null` kalau barusan dicabut |

### Error

| Status | Kapan |
|---|---|
| `401` | Header `Authorization` gak ada / invalid |
| `403` | `message_id` ada, tapi sesinya bukan milik user yang lagi login |
| `404` | `message_id` gak ketemu |
| `422` | `feedback` di luar tiga nilai yang sah · `reason` lebih dari 500 karakter · `message_id` bukan UUID · `message_id` nunjuk pesan `role: "user"` |

**Rate limit:** belum ditentukan. Tetangganya `POST /chat` 10/menit dan `POST /chat/sessions` 60/menit. Angka buat endpoint ini nunggu ketokan pemilik — kalau nanti ada, munculnya `429`.

### Yang TIDAK diterima endpoint ini

Body cuma tiga field di atas. **Kirim field lain bakal diabaikan, bukan disimpan.** Jangan coba nyelipin `store_id`, `tenant_id`, atau apa pun yang keliatan nyambung — itu ditutup di sisi server dan ada testnya. Kalau ada kebutuhan nyimpen konteks tambahan, ngomong dulu, jangan diselundupin lewat body.

---

## Pesan manual juga bisa dirating

Jawaban yang masuk lewat `POST /chat/messages/append` (echo aksi FE — buka toko, tambah stok, dst) **ikut bisa dikasih jempol** kalau `role`-nya `assistant`. Ini keputusan sadar pemilik, bukan kelupaan: kalau user suka sama pesan manual itu, datanya tetap kepake.

Buat FE artinya satu hal: **jangan bikin logika khusus yang nyembunyiin tombol di pesan hasil append.** Perlakuin sama kayak jawaban bot biasa.

---

## Yang berubah dari ENC-12

| | ENC-12 (sekarang, live) | FTR-17 (setelah ini jalan) |
|---|---|---|
| Pengenal per pesan | gak ada | `messages[].id` |
| Field feedback | gak ada | `feedback`, `feedback_reason` |
| Endpoint feedback | gak ada | `PUT /chat/message/feed` |
| Pagination, urutan, auth | — | **tidak berubah** |

`api-spec-<TIKET-ENC>.md` **belum jadi stale** — semua yang ditulis di sana masih benar, cuma daftar field-nya belum lengkap. Baca dua-duanya: ENC-12 buat pagination dan urutan, dokumen ini buat field baru.

---
Ref: `<TIKET-FTR>` (`docs/backlog/<TIKET-FTR>.md`), naik dari `<TIKET-TBD>`. Spek diketok pemilik 2026-09-14 di kanal tim. Nambahin field ke endpoint yang dijelasin `docs/reports/api-spec-<TIKET-ENC>.md`.
