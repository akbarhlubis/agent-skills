---
name: hypothesis-driven-testing
description: Scientific method for debugging. Use when behavior doesn't match expectations, forms silently fail to save, UI state is inconsistent, or you need to systematically test a hypothesis against a live system. Don't guess — design a test, change one variable, verify, and draw conclusions.
---

# Hypothesis-Driven Testing

## Overview

Debugging bukan cuma soal code. Ketika sistem live berperilaku tidak sesuai ekspektasi — form gak ke-save, state UI gak konsisten, data gak muncul — kamu butuh **metode ilmiah** untuk menemukan root cause.

Jangan nebak. Rancang eksperimen.

## The Core Loop

```
🔍 OBSERVE
    ↓
🧠 HYPOTHESIZE
    ↓
🧪 DESIGN TEST
    ↓
⚡ EXECUTE
    ↓
✅ VERIFY
    ↓
📊 ANALYZE → hypothesis confirmed? → FIX
                     ↓
                hypothesis rejected? → new hypothesis
```

## Rule #1: Never Blindly Trust "It Worked"

> Kalau kamu klik submit dan halaman redirect — itu bukan bukti.
> Satu-satunya bukti: **data berubah di tempat seharusnya**.

| Jangan percaya | Percaya |
|----------------|---------|
| Tombol di-klik | Data di list berubah |
| Form ke-submit | Nilai di database berubah |
| Redirect terjadi | State UI konsisten |

## Rule #2: Test With Different Values

> Jangan cuma "cek ulang" — ganti ke nilai yang **jelas berbeda**.
> 
> Kalau nilainya `12 Juni` dan kamu cek ulang `12 Juni` — kamu gak tau apakah itu data lama atau hasil save.
> Ganti ke `13 Juni` — kalau setelah save tetap `12 Juni`, jelas gagal.

| Test lemah | Test kuat |
|-----------|----------|
| Cek ulang nilai yang sama | Ganti ke nilai berbeda |
| Verifikasi 1 field | Verifikasi dari sumber berbeda |
| Asumsi "pasti berhasil" | Selalu cek hasil |

## Rule #3: Isolate One Variable at a Time

> Kalau ada 3 hal yang mungkin salah, jangan ganti semua sekaligus.
> Ubah 1 → test → verifikasi → baru lanjut yang berikutnya.

```
Contoh: Form edit gagal save.

DONT: Ganti button click + ganti field + ganti URL → bingung mana fix
DO:  
  1. Ganti button target saja → test → gagal
  2. Ganti form submit method saja → test → gagal
  3. Ganti button text pattern saja → test → BERHASIL
  → Root cause ditemukan
```

## Pattern: UI Form Silent Failure

Pola paling umum untuk SPA/form modern:

```
1. OBSERVE: Edit form → save → data tidak berubah di list
2. HYPOTHESIZE: Mungkin button text/selector salah?
3. DESIGN TEST:
   - Eksplorasi semua button: evaluate → daftar button + text
   - Temukan button benar ("Simpan Perubahan", bukan "Update")
   - Ganti tanggal ke nilai test (12→13) 
4. EXECUTE: Klik button yang benar
5. VERIFY: Navigasi ke list → cek kolom tanggal
   - 13 → BERHASIL, root cause = button text pattern
   - 12 → GAGAL, butuh hypothesis baru
6. FIX: Perbaiki selector
7. RE-VERIFY: Balikkan nilai (13→12) → cek list → 12? Done.
```

## Pattern: SPA State Inconsistency

```
1. OBSERVE: Data tidak update setelah aksi
2. HYPOTHESIZE: Cache? Perlu refresh?
3. DESIGN TEST: Baca data sebelum & sesudah aksi, dari sumber berbeda
4. COMPARE: Data berubah atau tidak?
```

## Pattern: Autocomplete / Combobox

```
1. OBSERVE: Selection tidak diterima
2. HYPOTHESIZE: Event tidak tepat?
3. TESTS (isolated!):
   Test A: .click() → gagal
   Test B: MouseEvent chain → BERHASIL
   Test C: Search panjang → GAGAL vs pendek → BERHASIL
4. CONCLUSION: React combobox = MouseEvent + partial search
```

## Before Testing: Explore the System

```js
// Cek struktur sebelum buat hypothesis
evaluate → JSON.stringify({
  buttons: Array.from(document.querySelectorAll('button'))
    .map(b => ({ text: b.innerText?.trim(), type: b.type, disabled: b.disabled })),
  inputs: Array.from(document.querySelectorAll('input'))
    .map(i => ({ id: i.id, type: i.type, value: i.value })),
  formAction: document.querySelector('form')?.action
})
```

## Real-World Examples from Agent-Human Collaboration

Ini bukan teori — ini pola yang terjadi natural di percakapan sehari-hari antara user dan agent.

### 1. "Tool A vs Tool B — mana yang lebih bagus?"

```
OBSERVE: Browser MCP sering timeout
HYPOTHESIZE: Chrome DevTools MCP lebih stabil?
TEST: Switch ke navigate + evaluate
VERIFY: 10x operasi tanpa timeout
CONCLUSION: Migrasi permanen ke Chrome DevTools MCP
```

### 2. "Ini gak respon — kenapa ya?"

```
OBSERVE: Autocomplete dropdown muncul tapi klik item gagal
HYPOTHESIZE 1: React state butuh event khusus?
TEST: .click() → GAGAL

HYPOTHESIZE 2: MouseEvent chain?
TEST: mousedown + mouseup + click → BERHASIL
CONCLUSION: React combobox butuh MouseEvent, bukan click biasa
```

### 3. "Data gak ketemu — di mana ya?"

```
OBSERVE: Cross-reference GitLab ↔ E-Kinerja kosong
HYPOTHESIZE: Ada di branch lain?
TEST: Cek develop_gabungcode (bukan cuma develop)
VERIFY: 3 commit match ketemu
CONCLUSION: Skill perlu cek 2 branch, bukan 1
```

### 4. "Fitur gagal — ada constraint?"

```
OBSERVE: Check-out absensi gagal
HYPOTHESIZE 1: GPS tidak tersedia di browser otomatis?
TEST: Cek error → "Akses lokasi ditolak" → CONFIRMED

HYPOTHESIZE 2: Bisa override geolocation API?
TEST: navigator.geolocation.getCurrentPosition = mock
VERIFY: Check-out tanpa error lokasi
CONCLUSION: GPS mock works, tapi 8 jam rule masih berlaku
```

### 5. "Search term gak match — apa yang salah?"

```
OBSERVE: "RSUD Batin Mangunang" tidak muncul di autocomplete
HYPOTHESIZE: Search terlalu panjang?
TEST 1: "RSUD Batin Mangunang" → GAGAL
TEST 2: "Batin" → "RS Umum Daerah Batin Mangunang" muncul → BERHASIL
CONCLUSION: Autocomplete partial match, selalu pakai term pendek
```

### 6. "Button gak nemu — apa nama sebenarnya?"

```
OBSERVE: Edit form klik "Perbarui"/"Update" tidak menyimpan
HYPOTHESIZE: Nama button sebenarnya bukan itu?
TEST: Eksplorasi semua button di DOM → "Simpan Perubahan"
FIX: Ganti selector ke "Simpan Perubahan"
VERIFY: Ubah tanggal test (12→13) → cek list → 13 muncul
```

### 🎯 The Meta Pattern

Semua use case diatas punya struktur yang sama:

```
Kamu:    "Coba [something new]"          ← HYPOTHESIZE
Agent:   execute                          ← TEST
Kamu:    "Coba cek"                       ← VERIFY  
Kamu:    "Kenapa bisa gitu ya?"           ← ANALYZE
Kamu:    "Update skill"                   ← DOCUMENT
```

> **Skill ini menamai apa yang sudah kita lakukan.** Bedanya: sekarang agent bisa switch ke hypothesis mode secara proactive, tanpa user yang selalu trigger.

## When Hypothesis Fails

```
HYPOTHESIS FAILED → ini DATA, bukan kegagalan!

Analyze:
  - What did you expect?
  - What actually happened?
  - What does the difference tell you?

Each failed hypothesis = closer to root cause.
```

## Red Flags

- "It should work" — without verification
- Changing multiple things at once
- Accepting a fix without understanding WHY

## When NOT to use this skill

- 100% sure of root cause (rarely true)
- Simple typos / syntax errors
- Non-reproducible issues (use logging)
