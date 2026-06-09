---
name: ekinerja
description: Automate absensi (check-in/check-out) and manage request tasks on E-Kinerja PaperlessHospital (https://ekinerja.paperlesshospital.id). Use when the user needs absensi, buat request task, update pengerjaan task, pengajuan cuti/izin/WFH, or navigate the e-kinerja dashboard. All interaction via Chrome DevTools MCP (navigate + evaluate + screenshot).
---

# E-Kinerja PaperlessHospital — Agent Guide

> **URL:** https://ekinerja.paperlesshospital.id/ | **Updated:** 2026-06-08
> **MCP:** Chrome DevTools MCP (`navigate`, `evaluate`, `screenshot`)

## 🔐 Credentials (SECURE — NEVER HARDCODE)

Credentials disimpan terpisah di file `~/.agents/skills/ekinerja/.env`. Agent HARUS membaca file ini saat pertama kali butuh login.

```
C:\Users\ThinkPad\.agents\skills\ekinerja\.env
```

Format:
```
EKNERJA_EMAIL=xxx@gmail.com
EKNERJA_PASSWORD=xxx

# Hospital-specific (opsional, sesuaikan RS masing-masing)
EKNERJA_RS_SEARCH=Batin
EKNERJA_RS_NAME=RS Umum Daerah Batin Mangunang
EKNERJA_PERMINTAAN_SEARCH=SIMRS
EKNERJA_PERMINTAAN_NAME=Instalasi SIMRS (IT)
```

File ini **gitignored** (tidak akan pernah masuk repo). Agent wajib membaca file ini untuk mendapatkan credentials.

Jika file `.env` tidak ditemukan, **tanya user** untuk email & password, lalu simpan ke file tersebut.

### Cara menemukan nilai RS & Permintaan untuk RS lain

Kalau teman dari RS lain mau pakai skill ini:
1. Buka `https://ekinerja.paperlesshospital.id/request-task/create`
2. Ketik partial name RS di field autocomplete, lihat hasil dropdown
3. Catat **search term** (yang diketik) dan **full name** (yang muncul)
4. Ulangi untuk field Permintaan (User/Instalasi)
5. Simpan di `.env` masing-masing

### ⚠️ Permintaan (User/Instalasi) — KONTEKS DINAMIS

Isi dropdown Permintaan **berbeda tiap RS** — tergantung instalasi/unit yang terdaftar. Agent HARUS eksplorasi dulu:

```js
// Step: Cek isi dropdown Permintaan
evaluate → new Promise(resolve => {
  function setNativeValue(el, value) {
    const setter = Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, 'value')?.set;
    setter?.call(el, value);
    el.dispatchEvent(new Event('input', {bubbles: true}));
  }
  const input = document.getElementById('id_permintaan');
  input.focus();
  setNativeValue(input, '');  // kosongkan — trigger semua opsi
  setTimeout(() => {
    const items = Array.from(document.querySelectorAll('ul.absolute.z-50 li'))
      .map(li => li.innerText.trim());
    resolve(JSON.stringify(items));
  }, 1500);
})
```

Dari hasil eksplorasi, agent bisa mencatat search term yang valid. Simpan di `.env` kalau sudah fix.

Kalau `.env` cuma ada EMAIL & PASSWORD (tanpa config RS/Permintaan), agent WAJIB eksplorasi dropdown dulu sebelum create task — jangan asal isi.

---

## Tech Stack

| Layer | Detail |
|-------|--------|
| Backend | Laravel 13.4.0 + PHP 8.4.17 (Nginx) |
| Frontend | SPA (Livewire/Alpine-like) |
| CDN | Cloudflare |
| Auth | Session-based (`e-kinerja-session` + `XSRF-TOKEN` cookie) |
| Debug mode | ON |

## Current User

- Nama: **AL Akbar Hamonangan Lubis**
- Email: **alubis87@gmail.com**
- Role: Developer (task: SIMRS development, bug fixing)

---

## 🚀 Login Flow (Chrome DevTools MCP)

Karena Chrome DevTools MCP membuka browser fresh tanpa session, login HARUS dilakukan setiap sesi baru.

### Step 1: Navigasi
```
navigate → https://ekinerja.paperlesshospital.id
```
Akan redirect ke `/login`.

### Step 2: Cek apakah perlu login
```js
evaluate → JSON.stringify({url: window.location.href, title: document.title})
```
Jika URL mengandung `/login` → lanjut Step 3. Jika `/dashboard` → sudah login (skip).

### Step 3: Baca credentials dari .env
Gunakan `read_file` untuk membaca `C:\Users\ThinkPad\.agents\skills\ekinerja\.env`.

### Step 4: Isi form dan submit via evaluate
```js
(() => {
  const form = document.querySelector('form');
  form.querySelector('input[name="email"]').value = 'EMAIL_DARI_ENV';
  form.querySelector('input[name="password"]').value = 'PASSWORD_DARI_ENV';
  form.submit();
  return 'Login submitted';
})()
```

### Step 5: Tunggu redirect & verifikasi
```js
new Promise(resolve => {
  setTimeout(() => {
    resolve(JSON.stringify({url: location.href, title: document.title}));
  }, 3000);
})
```
Harus di `/dashboard`. Jika masih `/login` → credentials salah, tanya user.

---

## Navigation Structure

```
Dashboard           → /dashboard
Schedule            → /schedule
Absensi             → /absensi
Managemen Task (dropdown)
  └─ Request        → /request-task
     └─ Tambah      → /request-task/create
     └─ Pengerjaan  → /request-task/{ID}/pengerjaan
Pengajuan (dropdown)
  ├─ Perbaikan Absensi → /pengajuan-perbaikan
  ├─ Cuti              → /pengajuan-cuti
  ├─ Izin              → /pengajuan-izin
  └─ WFH               → /pengajuan-wfh
```

---

## Absensi (`/absensi`) — Check-in/Check-out

**URL:** `https://ekinerja.paperlesshospital.id/absensi`

### State tombol:
- **Check-in** — ENABLED jika belum check-in; DISABLED + tampil jam setelah check-in
- **Check-out** — DISABLED jika belum check-in; ENABLED setelah check-in; DISABLED + tampil jam setelah check-out

### ⚠️ Validasi 8 Jam Kerja

Saat check-out sebelum 8 jam kerja:
- Muncul warning: "Belum Memenuhi Jam Kerja — Anda baru bekerja X jam. Sisa sekitar Y jam lagi."
- Ada tombol "Mengerti" untuk acknowledge
- Check-out mungkin tetap bisa dilanjutkan setelah klik Mengerti

### Check-in — via evaluate
```js
navigate → https://ekinerja.paperlesshospital.id/absensi

evaluate → (() => {
  const btn = Array.from(document.querySelectorAll('button'))
    .find(b => b.innerText.trim() === 'Check-in' || b.innerText.includes('Check-in'));
  if (btn && !btn.disabled) { btn.click(); return 'Check-in clicked'; }
  return JSON.stringify({error: 'Button not available', disabled: btn?.disabled});
})()

// Optional: isi keterangan
evaluate → (() => {
  function setNativeValue(el, value) {
    const setter = Object.getOwnPropertyDescriptor(HTMLTextAreaElement.prototype, 'value')?.set;
    setter?.call(el, value);
    el.dispatchEvent(new Event('input', {bubbles: true}));
  }
  const ta = document.querySelector('textarea');
  if (ta) setNativeValue(ta, 'WFH - Development');
  return ta?.value;
})()
```

### Check-out — via evaluate
```js
navigate → https://ekinerja.paperlesshospital.id/absensi

// Cek state dulu
evaluate → (() => {
  const checkinBtn = Array.from(document.querySelectorAll('button'))
    .find(b => b.innerText.includes('Check-in'));
  const checkoutBtn = Array.from(document.querySelectorAll('button'))
    .find(b => b.innerText.includes('Check-out'));
  return JSON.stringify({
    checkinDisabled: checkinBtn?.disabled,
    checkoutDisabled: checkoutBtn?.disabled,
    checkinTime: checkinBtn?.innerText,
    checkoutTime: checkoutBtn?.innerText
  });
})()

// Isi keterangan + klik
evaluate → (() => {
  function setNativeValue(el, value) {
    const setter = Object.getOwnPropertyDescriptor(HTMLTextAreaElement.prototype, 'value')?.set;
    setter?.call(el, value);
    el.dispatchEvent(new Event('input', {bubbles: true}));
  }
  const ta = document.querySelector('textarea');
  if (ta) setNativeValue(ta, 'Selesai bekerja');
  
  const btn = Array.from(document.querySelectorAll('button'))
    .find(b => b.innerText.includes('Check-out'));
  if (btn && !btn.disabled) { btn.click(); return 'Check-out clicked'; }
  return 'Not available';
})()

// Kalau muncul warning 8 jam, klik "Mengerti":
evaluate → (() => {
  const btn = Array.from(document.querySelectorAll('button'))
    .find(b => b.innerText.includes('Mengerti'));
  if (btn) { btn.click(); return 'Mengerti clicked'; }
  return 'No warning';
})()
```

---

## Request Task List (`/request-task`)

List semua task. Kolom: #, ID Request, Nama Task, Jenis Task, Status Permintaan, Status, Progress, Tgl Target, Catatan Approval, Aksi.

### Lihat list task
```
navigate → https://ekinerja.paperlesshospital.id/request-task
```

### Cek halaman lain (pagination)
Ganti URL: `/request-task?page=2`

### Ambil data task via evaluate
```js
evaluate → (() => {
  const rows = Array.from(document.querySelectorAll('table tbody tr'));
  return JSON.stringify(rows.map(row => {
    const cells = Array.from(row.querySelectorAll('td'));
    return {
      id: cells[1]?.innerText?.trim(),
      task: cells[2]?.innerText?.trim(),
      jenis: cells[3]?.innerText?.trim(),
      status_permintaan: cells[4]?.innerText?.trim(),
      status: cells[5]?.innerText?.trim(),
      progress: cells[6]?.innerText?.trim()
    };
  }));
})()
```

---

## ⚠️ Autocomplete (shadcn/ui Combobox) — CRITICAL PATTERN

Field `id_rs_klinik` dan `id_permintaan` adalah **autocomplete shadcn/ui**, BUKAN input biasa. Value disimpan di React state, tidak di hidden input. Ini adalah bagian PALING SULIT.

### Pattern yang BERHASIL:

```js
// 🔑 KUNCI: native setter + input event + wait + MouseEvent
function setNativeValue(el, value) {
  const setter = Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, 'value')?.set;
  setter?.call(el, value);
  el.dispatchEvent(new Event('input', {bubbles: true}));
}

// Step 1: Kosongkan + isi search term (PAKAI TERM PENDEK!)
const input = document.getElementById('id_rs_klinik');
input.focus();
setNativeValue(input, '');        // clear dulu
setNativeValue(input, 'Batin');   // search term PENDEK — "RSUD Batin Mangunang" GAGAL, "Batin" BERHASIL

// Step 2: TUNGGU dropdown muncul (1.5 detik)
// Step 3: Pilih item dengan MouseEvent (BUKAN .click() biasa!)
const dropdown = document.querySelector('ul.absolute.z-50');
const item = dropdown.querySelector('li');
item.dispatchEvent(new MouseEvent('mousedown', {bubbles: true}));
item.dispatchEvent(new MouseEvent('mouseup', {bubbles: true}));
item.dispatchEvent(new MouseEvent('click', {bubbles: true}));
```

### ❌ Yang GAGAL:
- `.click()` biasa — tidak trigger React state
- Search term panjang — "RSUD Batin Mangunang" tidak match, pakai "Batin"
- Tanpa `mousedown` + `mouseup` sebelum `click`
- Tanpa `new Promise` + `setTimeout` untuk tunggu dropdown

### 🏥 RS yang benar (contoh — RSUD Batin Mangunang):
Search: `Batin` → pilih: **"RS Umum Daerah Batin Mangunang"**

### 👤 Permintaan yang benar (contoh — RSUD Batin Mangunang):
Search: `SIMRS` → pilih: **"Instalasi SIMRS (IT)"**

<details>
<summary>📋 Daftar lengkap 25 opsi Permintaan (klik expand)</summary>

| Search Term | Full Name | Kategori |
|-------------|-----------|----------|
| SIMRS | Instalasi SIMRS (IT) | IT |
| Casemix | Casemix | Medis |
| IGD | IGD | Medis |
| Rawat | Rawat Jalan / Rawat Inap | Medis |
| Farmasi | Farmasi | Penunjang |
| Kasir | Kasir | Keuangan |
| Pendaftaran | Pendaftaran | Admin |
| Radiologi | Radiologi | Penunjang |
| Rekam | Rekam Medis | Penunjang |
| Management | Management | Manajemen |
| Dokter | Dokter | Medis |
| Fisioterapi | Fisioterapi | Penunjang |
| Laboratorium | Laboratorium | Penunjang |
| Gizi | Gizi | Penunjang |
| Perawat | Petugas (Perawat) | Medis |
| Keuangan | Keuangan | Keuangan |
| Inventori | Inventori Obat | Logistik |
| Gudang | Gudang Barang | Logistik |
| CEO | CEO (Tri Wahyudi) | C-Level |
| CTO | CTO (Jiwana Syah Putra) | C-Level |
| COO | COO (Ade Rahman) | C-Level |
| Leader | Leader (T. Nopriansyah) | Lead |
| CMO | CMO (Irhash) | C-Level |
| CFO | CFO (Iqbal Yusuf) | C-Level |

</details>

> ⚠️ Nilai ini **berbeda tiap RS**. Agent harus eksplorasi dropdown atau baca `.env` untuk RS lain.

---

## Create Request Task (`/request-task/create`)

### Full Flow — TERBUKTI BERHASIL (2026-06-08)

```js
// === Step 1: Navigasi ===
navigate → https://ekinerja.paperlesshospital.id/request-task/create

// === Step 2: Isi autocomplete RS/Klinik (WAJIB) ===
evaluate →
new Promise(resolve => {
  function setNativeValue(el, value) {
    const setter = Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, 'value')?.set;
    setter?.call(el, value);
    el.dispatchEvent(new Event('input', {bubbles: true}));
  }
  const input = document.getElementById('id_rs_klinik');
  input.focus();
  setNativeValue(input, '');
  setNativeValue(input, 'Batin');
  setTimeout(() => {
    const dropdown = document.querySelector('ul.absolute.z-50');
    const item = dropdown?.querySelector('li');
    if (item) {
      item.dispatchEvent(new MouseEvent('mousedown', {bubbles: true}));
      item.dispatchEvent(new MouseEvent('mouseup', {bubbles: true}));
      item.dispatchEvent(new MouseEvent('click', {bubbles: true}));
    }
    resolve(JSON.stringify({selected: item?.innerText, value: input.value}));
  }, 1500);
})

// === Step 3: Isi autocomplete Permintaan (WAJIB) ===
evaluate →
new Promise(resolve => {
  function setNativeValue(el, value) {
    const setter = Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, 'value')?.set;
    setter?.call(el, value);
    el.dispatchEvent(new Event('input', {bubbles: true}));
  }
  const input = document.getElementById('id_permintaan');
  input.focus();
  setNativeValue(input, '');
  setNativeValue(input, 'SIMRS');
  setTimeout(() => {
    const dropdown = document.querySelector('ul.absolute.z-50');
    const item = dropdown?.querySelector('li');
    if (item) {
      item.dispatchEvent(new MouseEvent('mousedown', {bubbles: true}));
      item.dispatchEvent(new MouseEvent('mouseup', {bubbles: true}));
      item.dispatchEvent(new MouseEvent('click', {bubbles: true}));
    }
    resolve(JSON.stringify({selected: item?.innerText, value: input.value}));
  }, 1500);
})

// === Step 4: Isi sisanya (select + text + date) ===
evaluate → (() => {
  function setNativeValue(el, value) {
    const setter = Object.getOwnPropertyDescriptor(
      el instanceof HTMLInputElement ? HTMLInputElement.prototype : HTMLTextAreaElement.prototype, 'value'
    )?.set;
    setter?.call(el, value);
    el.dispatchEvent(new Event('input', {bubbles: true}));
    el.dispatchEvent(new Event('change', {bubbles: true}));
  }

  // Status Permintaan (select native)
  document.querySelectorAll('select')[0].value = '3';
  document.querySelectorAll('select')[0].dispatchEvent(new Event('change', {bubbles: true}));

  // Bobot (select native)
  document.querySelectorAll('select')[1].value = '2';
  document.querySelectorAll('select')[1].dispatchEvent(new Event('change', {bubbles: true}));

  // Nama Task (text input)
  setNativeValue(document.getElementById('task_request'), 'Judul Task Kamu');

  // Detail (textarea)
  setNativeValue(document.getElementById('detail_task_request'), 'Deskripsi detail...');

  // Tanggal Mulai (date input)
  setNativeValue(document.getElementById('tgl_target_mulai'), '2026-06-08');

  // Durasi (number input)
  setNativeValue(document.getElementById('durasi_target'), '3');

  return 'OK';
})()

// === Step 5: Verifikasi & Submit ===
evaluate → (() => {
  const btn = Array.from(document.querySelectorAll('button'))
    .find(b => b.innerText.includes('Buat Request Task'));
  if (btn) { btn.click(); return 'Submitted'; }
  return 'Button not found';
})()

// === Step 6: Verifikasi redirect ===
evaluate →
new Promise(resolve => setTimeout(() => resolve(JSON.stringify({
  url: location.href,
  sukses: location.href.includes('/request-task') && !location.href.includes('/create')
})), 3000))
```

### Status Permintaan Options (19)

**Developer:**
| Value | Nama |
|-------|------|
| 1 | Develop Baru Fitur, Modul Atau Form (New Feature Development) |
| 2 | Analisis Kebutuhan Sistem (Discovery) |
| 3 | Revisi / Perbaiki Bug (Fixing) |
| 4 | Update Fitur, Modul Atau Form (Feature Updates) |
| 5 | Migrasi & Sinkronisasi Data |
| 12 | Data Audit (Pengecekan Konsistensi) |
| 14 | Penyediaan & Inisialisasi Server Baru (Server Provisioning) |
| 15 | Optimasi & Perbaikan Konfigurasi Server (Server Tuning & Fixing) |
| 16 | Analisis Kebutuhan Arsitektur Server (Infrastructure Discovery) |
| 17 | Konfigurasi Backup & Migrasi Server (Server Backup & Migration) |
| 19 | Kloning & Adaptasi Source Code (Repo/SIMRS Lain) |

**Non Developer:**
| Value | Nama |
|-------|------|
| 6 | Quality Control (QC) |
| 7 | Rapat / Diskusi (Requirement Fix) |
| 9 | Sosialisasi / Implementasi |
| 10 | Testing dengan End User (UAT) |
| 11 | Training (Pelatihan User / Tim) |
| 13 | User Documentation & Manual |
| 18 | Uji Beban & Keamanan Server (Server Load & Security Testing) |

### Bobot Options
| Value | Nama |
|-------|------|
| 1 | mudah |
| 2 | sedang |
| 3 | sulit |

---

## Pengerjaan Task (`/request-task/{ID}/pengerjaan`)

### Full Flow — TERBUKTI BERHASIL (2026-06-08)

```js
// === Step 1: Navigasi ===
// Pattern URL: /request-task/{ID}/pengerjaan
// Contoh: /request-task/RQT-260608032/pengerjaan
navigate → https://ekinerja.paperlesshospital.id/request-task/RQT-{ID}/pengerjaan

// === Step 2: Isi semua field & submit ===
evaluate → (() => {
  function setNativeValue(el, value) {
    const setter = Object.getOwnPropertyDescriptor(
      el instanceof HTMLInputElement ? HTMLInputElement.prototype : HTMLTextAreaElement.prototype, 'value'
    )?.set;
    setter?.call(el, value);
    el.dispatchEvent(new Event('input', {bubbles: true}));
    el.dispatchEvent(new Event('change', {bubbles: true}));
  }

  // Status Pengerjaan (select native)
  document.querySelector('select').value = '6'; // 6 = Selesai
  document.querySelector('select').dispatchEvent(new Event('change', {bubbles: true}));

  // Tanggal Mulai
  setNativeValue(document.getElementById('tgl_mulai'), '2026-06-08');

  // Durasi (number input)
  setNativeValue(document.getElementById('durasi_selesai'), '1');

  // Informasi Tambahan
  setNativeValue(document.getElementById('informasi_tambahan'), 'Catatan pengerjaan...');

  // Keterangan Selesai (WAJIB saat status Selesai)
  setNativeValue(document.getElementById('keterangan_task_selesai'), 'Link commit atau keterangan...');

  // Submit
  const btn = Array.from(document.querySelectorAll('button'))
    .find(b => b.innerText.includes('Perbarui Pengerjaan'));
  if (btn) btn.click();

  return 'Submitted';
})()

// === Step 3: Verifikasi dari list task ===
navigate → https://ekinerja.paperlesshospital.id/request-task
// Cek progress — harus menunjukkan "Selesai 100%"
```

### Status Pengerjaan (select value)
| Value | Status | Progress |
|-------|--------|----------|
| 1 | Pending | 0% |
| 2 | On Proses | 30% |
| 3 | Revisi | 50% |
| 4 | Review | 70% |
| 5 | Testing | 90% |
| 6 | Selesai | 100% |

---

## Delete Task

### 🔴 ATURAN WAJIB — KONFIRMASI DULU SEBELUM HAPUS

> ⚠️ **HARAM hukumnya menghapus task tanpa konfirmasi user!**
> Wajib tanya: "Yakin hapus task [ID] - [Nama Task]?"
> Jangan pernah langsung klik Hapus meskipun dialog konfirmasi sudah muncul.
> Tunggu user bilang "ya" / "lanjut" / "hapus" dulu.

### Full Flow — TERBUKTI BERHASIL

```js
// === Step 1: Cari task di list ===
navigate → https://ekinerja.paperlesshospital.id/request-task

// === Step 2: Klik tombol delete (merah) ===
evaluate → (() => {
  const rows = Array.from(document.querySelectorAll('table tbody tr'));
  const targetRow = rows.find(row => row.innerText.includes('RQT-XXX'));
  const deleteBtn = Array.from(targetRow.querySelectorAll('button'))
    .find(b => b.className.includes('red'));
  if (deleteBtn) { deleteBtn.click(); return 'Delete clicked'; }
  return 'Not found';
})()

// === Step 3: Konfirmasi dialog ===
evaluate → (() => {
  const dialog = document.querySelector('[role="alertdialog"]');
  const hapusBtn = Array.from(dialog.querySelectorAll('button'))
    .find(b => b.innerText.trim() === 'Hapus');
  if (hapusBtn) { hapusBtn.click(); return 'Confirmed delete'; }
  return 'Dialog not found';
})()

// === Step 4: Verifikasi ===
evaluate →
new Promise(resolve => setTimeout(() => resolve(JSON.stringify({
  deleted: !document.body.innerText.includes('RQT-XXX')
})), 2000))
```

---

## Pengajuan

### Perbaikan Absensi (`/pengajuan-perbaikan`)
```
navigate → https://ekinerja.paperlesshospital.id/pengajuan-perbaikan
```

### Cuti, Izin, WFH
```
navigate → https://ekinerja.paperlesshospital.id/pengajuan-cuti
navigate → https://ekinerja.paperlesshospital.id/pengajuan-izin
navigate → https://ekinerja.paperlesshospital.id/pengajuan-wfh
```

Klik "Buat Pengajuan" button via evaluate:
```js
evaluate → (() => {
  const btn = Array.from(document.querySelectorAll('button'))
    .find(b => b.innerText.includes('Buat Pengajuan'));
  if (btn) btn.click();
  return 'Clicked';
})()
```

---

## Tips Chrome DevTools MCP

1. **Gunakan `evaluate` untuk SEMUA interaksi** — click, type, isi form, submit, cek state. Jauh lebih cepat dan reliable daripada browser automation tradisional.
2. **`screenshot` untuk verifikasi visual** — opsional, gunakan saat perlu lihat tampilan.
3. **Combobox > pilih via `evaluate`** jika `<select>` native. Jika custom dropdown (listbox), gunakan keyboard nav (`browser_click` + `browser_press_key`).
4. **Tidak perlu snapshot** (`browser_snapshot`) lagi — ganti dengan `evaluate` yang return data terstruktur.
5. **Sequential tetap berlaku** — jangan parallel navigate + evaluate ke halaman berbeda.
6. **Login di awal sesi** — selalu cek apakah sudah di dashboard atau perlu login ulang.

### ⚠️ BEFORE CREATE TASK — WAJIB KONFIRMASI

> Sebelum klik "Buat Request Task", **WAJIB** tampilkan ringkasan ke user:
> ```
> Task  : [nama task]
> Status: [Fixing/Update/New Dev/...]
> Bobot : [mudah/sedang/sulit]
> Tgl   : [YYYY-MM-DD]
> Durasi: [N] hari
> ```
> Tunggu user konfirmasi sebelum submit!
> Jangan asal submit — cek tanggal hari ini dulu, jangan asumsi.

### ⚠️ AFTER EDIT — WAJIB VERIFIKASI

> Setelah edit task, selalu verifikasi perubahan tersimpan:
> 1. `evaluate` cek nilai field yang diubah
> 2. Atau navigasi ke list dan cek ulang

---

## GitLab Integration — Cross-Reference Pengerjaan

### Branches yang harus dicek

> ⚠️ JANGAN cuma cek `develop`! Cek **kedua** branch ini:
> 1. `develop` — branch utama
> 2. `develop_gabungcode` — branch penggabungan (sering ada commit tambahan)

### Step 1: Ambil data E-Kinerja
```
navigate → https://ekinerja.paperlesshospital.id/request-task
// Ambil task + filter: status=approved + progress=pending
// Cek page 2 juga kalau perlu
```

### Step 2: Ambil data GitLab (via MCP GitLab)
```js
// User = alubis87 (ID 36), project = rsud-batin-mangunang-lampung (ID 43)
// Cek KEDUA branch:
mcp_server_gitlab_list_commits → project_id=43, ref_name=develop, per_page=20
mcp_server_gitlab_list_commits → project_id=43, ref_name=develop_gabungcode, per_page=20

// Filter: author_name = "Akbar Hamonangan Lubis" (hanya commit user sendiri)
// Abaikan commit oleh author lain (Bim, dll)
```

### Step 3: Matching — cocokkan keyword

Cocokkan **kata kunci** dari nama task dengan commit message:
- "tindakan" ↔ "Tindakan"
- "obat" / "resep" ↔ "medication" / "obat" / "resep"
- "antrian" / "MJKN" ↔ "booking" / "estimation"
- "amprahan" ↔ "amprahan" / "retur"
- "dokter" ↔ "dokter"

### Step 4: Update hanya task APPROVED + Pending

> ⚠️ **HANYA update task yang status=approved + progress=pending**
> Jangan sentuh task yang belum approved!

### 🔗 WAJIB: Link Commit Full URL

> ⚠️ **Developer task HARUS pakai full GitLab URL di keterangan!**
> JANGAN cuma commit SHA pendek (a47bb408). HARUS full URL:
> `https://git.paperlesshospital.id/paperless/simrs/rsud-batin-mangunang-lampung/-/commit/a47bb4088ea73dbabadd8e14ef590ce5fa1a6383`
>
> Kalau multi-commit: pisahkan dengan newline, satu URL per baris.

### Identitas GitLab
| Field | Value |
|-------|-------|
| GitLab URL | https://git.paperlesshospital.id |
| Username | `alubis87` |
| User ID | 36 |
| Project | `paperless/simrs/rsud-batin-mangunang-lampung` (ID 43) |
| Branches | `develop`, `develop_gabungcode` |

