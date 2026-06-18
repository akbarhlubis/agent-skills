# Laporan Bulanan E-Kinerja → Google Docs

## Overview

Generate laporan bulanan kinerja dari data E-Kinerja ke Google Docs. Terdapat 2 template:
- **Akbar Lubis** — Engineer Programming (fokus: development, bug fixing, laporan)
- **Tri Wahyudi** — UI/UX Engineer (fokus: desain antarmuka, UX)

Keduanya memiliki struktur dan format yang sama, hanya berbeda pada judul dan sudut pandang narasi.

---

## Tools Tersedia

### Google Docs MCP (Baru — WAJIB PAKAI INI)

| Tool | Fungsi | Keterangan |
|------|--------|------------|
| `docs_create` | Buat dokumen baru | parameter: `title` |
| `docs_read` | Baca dokumen | parameter: `documentId` — **hanya baca tab pertama** |
| `docs_insert_text` | Insert di index | parameter: `documentId`, `text`, `index` |
| `docs_append_text` | Tambah di akhir | parameter: `documentId`, `text` |
| `docs_replace_text` | **Find & Replace** | parameter: `documentId`, `searchText`, `replaceText` |

### Chrome DevTools (untuk E-Kinerja login)
`navigate`, `evaluate`, `screenshot`

### Google Sheets (jika masih ada data lama)
`sheets_read_values`

---

## 🔴 KRITIKAL: Auto-Numbering Google Docs

> **JANGAN kirim nomor eksplisit** seperti `57. Menambahkan Sidebar...`
> Google Docs mendeteksi pola "nomor." dan **menerapkan auto-numbering sendiri**.
> Hasilnya: angka yang dikirim (57.) ditimpa jadi format Romawi (II, III, IV) atau format lain.

### ✅ Yang benar:
```
    Menambahkan Sidebar Akses Cepat
    Perbaikan dan Improvisasi alur verifikasi retur obat
```

Biarkan **auto-numbering Google Docs** yang handle penomoran.

### ❌ Yang salah:
```
    57. Menambahkan Sidebar Akses Cepat
    58. Perbaikan dan Improvisasi alur verifikasi retur obat
```

---

## 🔴 KRITIKAL: Strategi Write

### ✅ BENAR — Generate sekali, replace sekali
```
Step 1: Ambil task dari E-Kinerja
Step 2: Generate 1 teks UTUH (list + semua deskripsi)
Step 3: docs_replace_text 1× → ganti placeholder dengan teks utuh
```

### ❌ SALAH — Replace per-header (YANG KEMARIN)
```
Header 1 → replace → simpan    ← TERLALU BANYAK LANGKAH
Header 2 → replace → simpan
... 18× lagi
Hapus narasi sisa → 18× lagi   ← NARASI LAMA NYISA
```

### Teknis Replace:
```js
docs_replace_text(
  documentId: "ID_DOC",
  searchText: "Task",            // Cari placeholder unik
  replaceText: "list task\n...\ndeskripsi 1\n...\ndeskripsi 2\n..."
)
```

---

## Template Struktur Dokumen

### Akbar Lubis (Engineer Programming)
```
LAPORAN TIM IT
RSUD BATIN MANGUNANG
Akbar Lubis Engineer Programming
Bulan [BULAN], [TAHUN]

Pendahuluan
[Paragraf pembuka — ambil dari bulan sebelumnya, ganti nama bulan]

[List Task — TANPA NOMOR, biar auto-numbering Docs]
Menambahkan Sidebar Akses Cepat
Perbaikan dan Improvisasi alur verifikasi retur obat
...

Dari informasi di atas di bagi beberapa tahapan yang telah saya lakukan
sebagai Engineer Programming di RSUD Batin Mangunang yang dapat dilihat pada
informasi kegiatan yang telah dilakukan dibawah.

[Deskripsi per task — header TANPA NOMOR]
Menambahkan Sidebar Akses Cepat (5 Juni 2026)
Penambahan sidebar akses cepat dilakukan untuk mempermudah...
[1-3 paragraf narasi: latar belakang → implementasi → hasil/dampak]

[...ulang untuk setiap task...]

Lampiran
Berikut merupakan dokumentasi hasil dari pekerjaan...

Kesimpulan
Demikian kesimpulan dan laporan kinerja pada bulan [BULAN] [TAHUN]...
                    Kota Agung 30 [BULAN] [TAHUN]

Akbar Hamonangan Lubis, S.Kom
```

### Tri Wahyudi (UI/UX Engineer)
Sama persis, hanya:
- Judul: **Tri Wahyudi UI/UX Engineer**
- Nama: **Tri Wahyudi, S.T**
- Fokus narasi: **desain, tata letak, antarmuka, user experience**
- Prefix task: **UX_UI** (contoh: `UX_UI Penyesuaian kata kata...`)

---

## E-Kinerja → Ambil Task

### Login
```js
navigate → https://ekinerja.paperlesshospital.id
// Cek login, submit form via evaluate
```

### Ambil data task
```js
navigate → https://ekinerja.paperlesshospital.id/request-task?page=1
evaluate → ambil semua row table → filter status=approved + progress=Selesai 100%

// Cek page 2, 3 juga
```

Filter:
- **Untuk Akbar**: hanya task punya Akbar (`kry4`) + status `approved` + progress `Selesai`
- **Untuk bg_tw**: task dari Akbar & Alamsyah yang relevan UI/UX (ada perubahan tampilan/antarmuka)

### Data yang diambil per task
- ID (RQT-XXXX)
- Nama task
- Status permintaan (Fixing / Update / New Dev / Diskusi)
- Progress (Selesai 100% / Pending)
- Tanggal target

---

## Generate Narasi — Gaya Penulisan

Setiap deskripsi task mengikuti pola:
```
[Latar belakang — kenapa task ini perlu dilakukan]
[Implementasi — apa yang dikerjakan/diubah]
[Hasil/Dampak — manfaat setelah implementasi]
```

### Contoh Engineer (Akbar):
```
Perbaikan dan Analisis bug di Verifikasi Permintaan Amprahan Obat (5 Juni 2026)
Perbaikan dan analisis dilakukan pada bug yang ditemukan di alur verifikasi
permintaan amprahan obat. Setelah dilakukan penelusuran, ditemukan beberapa
kondisi yang menyebabkan proses verifikasi tidak berjalan sesuai alur yang
diharapkan. Dengan perbaikan ini, sistem verifikasi amprahan menjadi lebih
stabil dan mengurangi potensi kesalahan dalam proses distribusi obat ke ruangan.
```

### Contoh UI/UX (Tri Wahyudi):
```
UX_UI Perbaikan dan Analisis bug di Verifikasi Permintaan Amprahan Obat (5 Juni 2026)
Perancangan ulang tampilan verifikasi amprahan difokuskan pada kemudahan
pengguna dalam melihat status dan detail permintaan. Indikator visual
ditambahkan untuk membedakan setiap tahap verifikasi, sehingga petugas
dapat mengetahui posisi permintaan secara langsung. Dengan tampilan yang
lebih informatif, proses verifikasi menjadi lebih efisien dan mengurangi
kebingungan pengguna.
```

### Kata kunci Engineer: implementasi, logika, data, sistem, perbaikan
### Kata kunci UI/UX: tampilan, antarmuka, desain, pengguna, visual, navigasi

---

## READ dari Google Docs

| Metode | Keterangan |
|--------|------------|
| `docs_read(documentId)` | ✅ Baca tab **pertama** saja |
| `fetch(export?format=txt)` | ✅ Baca **SEMUA tab** — dipisah garis `____` |

Untuk akses tab lain, user perlu memindahkan tab yang diinginkan ke posisi
pertama, lalu `docs_read` akan membacanya.

---

## Write ke Google Docs via MCP Baru

```js
// Strategy: Generate 1 teks utuh → replace 1×
const fullContent = `
[List task]
Menambahkan Sidebar Akses Cepat
Perbaikan dan Improvisasi alur verifikasi retur obat
...

Dari informasi di atas di bagi beberapa tahapan...

[Deskripsi]
Menambahkan Sidebar Akses Cepat (5 Juni 2026)
Penambahan sidebar...
...
`;

docs_replace_text({
  documentId: "ID_DOC",
  searchText: "Task",         // Placeholder di dokumen
  replaceText: fullContent
});

// Verifikasi
docs_read({ documentId: "ID_DOC" });
// Screenshot untuk cek format visual
```

---

## Dokumen yang Sering Dipakai

| Dokumen | ID | Untuk |
|---------|----|-------|
| Laporan Akbar 2026 | `1pCYuMIiwKSFQ8EWnMuM96TLsGmPUL4-gqivanswkLG8` | Engineer Programming |
| Laporan bg tw 2026 | `1-1QQa-JT2c4FJECHxB6gRIjXxX2zbNme7wr6QVFlty8` | UI/UX Engineer |

---

## Verifikasi

Setelah write, WAJIB:
1. `docs_read` → cek teks sudah masuk
2. `screenshot` → cek format visual (auto-numbering, bold, dll)
3. Jangan percaya plain text API saja — Google Docs bisa ubah format rendering