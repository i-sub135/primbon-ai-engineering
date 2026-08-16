# Layer 2: Prompt Engineering

> Hasil eksplorasi 2026-07-15. Konteks: optimasi model lokal (ollama) untuk use case kasir POS (baby-kasir).
> No GPU, no weights. Inject contoh/instruksi di runtime — lebih fleksibel dari Modelfile.

## Kenapa Layer 2 Sebelum Fine-tune

80% kasus intent extraction kasir bisa diselesaikan di sini. Fine-tune baru worth it kalau prompt engineering udah mentok:
- Volume query tinggi (few-shot per-request = mahal di latency/token)
- Behavior terlalu spesifik buat dijelasin lewat contoh
- Model terlalu sering "keluar jalur" meski sudah dikasih contoh

## Zero-shot vs Few-shot

### Zero-shot (tanpa contoh)

```
User: "tadi laku aqua 2 dus"
Model: "Oke, saya catat penjualan aqua 2 dus ya."  ← format ga konsisten
```

Model tebak-tebak sendiri apa yang diinginkan. Hasilnya tidak predictable.

### Few-shot (kasih contoh dulu)

```
User: Ekstrak transaksi ke JSON.

Contoh:
Input: "laku rokok 3 bungkus"
Output: {"intent":"CREATE_SALE","items":[{"name":"rokok","qty":3,"unit":"bungkus"}]}

Input: "tadi aqua 2 dus sama teh botol 5"
Output: {"intent":"CREATE_SALE","items":[{"name":"aqua","qty":2,"unit":"dus"},{"name":"teh botol","qty":5}]}

Sekarang ekstrak ini:
Input: "tadi laku aqua 2 dus"
Output:
```

Model ikut pola → output JSON konsisten.

### Yang Ngefek di Few-shot

- Minimal 2–3 contoh; makin bervariasi makin bagus
- Cover edge case yang sering muncul: typo, satuan beda, qty di tengah kalimat
- **Urutan contoh penting** — contoh terakhir paling diingat model
- Contoh buruk = model belajar pola yang salah

## SYSTEM vs Few-shot Runtime

| | SYSTEM (Modelfile) | Few-shot runtime |
|---|---|---|
| Kapan di-set | Saat `ollama create` | Per-request |
| Fleksibel | Tidak | Ya |
| Cocok untuk | Persona, constraint umum | Tugas spesifik |
| Ganti tanpa rebuild | Tidak | Ya |

**Kombinasi ideal:** SYSTEM untuk persona + constraint umum → few-shot di runtime untuk tugas spesifik.

## Cara Inject Few-shot

### 1. Via CLI (manual test)

```bash
ollama run baby-kasir "Ekstrak transaksi ke JSON.
Contoh:
Input: laku rokok 3 bungkus
Output: {\"intent\":\"CREATE_SALE\",\"items\":[{\"name\":\"rokok\",\"qty\":3}]}

Input: tadi laku aqua 2 dus
Output:"
```

Bagus untuk quick test, bukan untuk production.

### 2. Via `/api/generate` (string concatenation)

```python
import requests

few_shot = """Contoh:
Input: laku rokok 3 bungkus
Output: {"intent":"CREATE_SALE","items":[{"name":"rokok","qty":3}]}

Input: tadi aqua 2 dus sama teh botol 5
Output: {"intent":"CREATE_SALE","items":[{"name":"aqua","qty":2},{"name":"teh botol","qty":5}]}
"""

user_input = "tadi laku aqua 2 dus"
response = requests.post("http://localhost:11434/api/generate", json={
    "model": "baby-kasir",
    "prompt": f"{few_shot}\nInput: {user_input}\nOutput:",
    "stream": False
})
```

### 3. Via `/api/chat` — paling idiomatis

```python
messages = [
    # fake conversation history = few-shot examples
    {"role": "user", "content": "laku rokok 3 bungkus"},
    {"role": "assistant", "content": '{"intent":"CREATE_SALE","items":[{"name":"rokok","qty":3}]}'},
    {"role": "user", "content": "tadi aqua 2 dus sama teh botol 5"},
    {"role": "assistant", "content": '{"intent":"CREATE_SALE","items":[{"name":"aqua","qty":2},{"name":"teh botol","qty":5}]}'},
    # actual query
    {"role": "user", "content": "tadi laku aqua 2 dus"},
]

response = requests.post("http://localhost:11434/api/chat", json={
    "model": "baby-kasir",
    "messages": messages,
    "stream": False
})
```

Model baca ini sebagai "oh ini pola percakapan kita" — bukan instruksi eksplisit. Lebih natural, lebih konsisten.

**Di LangChain:** `FewShotChatMessagePromptTemplate` — wrapper untuk pattern ini.

## Kapan Layer 2 Cukup

- Contoh-contoh bisa mewakili semua variasi input yang mungkin
- Latency per-request masih acceptable
- Model base sudah cukup "mengerti" domain

Kalau few-shot sudah tidak cukup atau terlalu mahal per-request → lanjut ke Layer 3.
