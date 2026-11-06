# Panduan Setup Database Google Sheets & WhatsApp Sales Taruna Fest 2026

Landing page ini telah dilengkapi **Formulir Pendaftaran Otomatis** yang:
1. **Menyimpan data pendaftar langsung ke Spreadsheet Google Sheets Anda**.
2. **Menyimpan cadangan lokal (LocalStorage)** di browser agar data tidak pernah hilang jika internet lambat.
3. **Membuka otomatis WhatsApp ke Tim Sales di `0813-2760-627`** dengan pesan konfirmasi yang sudah terformat rapi sesuai input pendaftar.

---

## 🚀 Langkah Menghubungkan ke Google Sheets (Hanya 2 Menit)

### Langkah 1: Buat Spreadsheet Baru di Google Drive
1. Buka [Google Sheets](https://sheets.new) di browser Anda.
2. Beri nama spreadsheet, misalnya: **Database Pendaftar Taruna Fest 2026**.
3. Pada baris pertama (Row 1), buat judul kolom berikut (opsional, script juga bisa membuatnya otomatis):
   - Kolom A: `Timestamp`
   - Kolom B: `Nama Lengkap`
   - Kolom C: `No WhatsApp`
   - Kolom D: `Asal Sekolah & Kelas`
   - Kolom E: `Target Impian`
   - Kolom F: `Kehadiran`
   - Kolom G: `Catatan`

---

### Langkah 2: Pasang Google Apps Script
1. Di Google Sheet tersebut, klik menu **Extensions (Ekstensi)** > **Apps Script**.
2. Hapus semua kode default di editor, lalu **copy & paste** kode berikut:

```javascript
/**
 * GOOGLE APPS SCRIPT - DATABASE PENDAFTARAN TARUNA FEST 2026
 * Menerima POST request JSON dari landing page dan menambahkannya sebagai baris baru.
 */

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000); // Mencegah race condition jika banyak yang mendaftar bersamaan

  try {
    var doc = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = doc.getSheetByName('Pendaftar') || doc.getActiveSheet();

    // Buat header jika sheet masih kosong
    if (sheet.getLastRow() === 0) {
      sheet.appendRow([
        'Timestamp',
        'Nama Lengkap',
        'No WhatsApp',
        'Asal Sekolah & Kelas',
        'Target Impian',
        'Kehadiran',
        'Catatan'
      ]);
      // Format header agar rapi
      sheet.getRange(1, 1, 1, 7).setFontWeight('bold').setBackground('#041a38').setFontColor('#ffffff');
    }

    // Parse data JSON yang dikirimkan dari website
    var rawData = e.postData.contents;
    var data = JSON.parse(rawData);

    // Tambah baris data baru
    sheet.appendRow([
      data.timestamp || new Date().toLocaleString('id-ID', { timeZone: 'Asia/Jakarta' }),
      data.nama || '',
      data.whatsapp || '',
      data.sekolah || '',
      data.target || '',
      data.kehadiran || '',
      data.catatan || '-'
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ result: 'success', row: sheet.getLastRow() }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

---

### Langkah 3: Deploy sebagai Web App
1. Di pojok kanan atas editor Apps Script, klik tombol biru **Deploy** > **New deployment**.
2. Klik ikon gear ⚙️ di sebelah *Select type*, pilih **Web app**.
3. Konfigurasikan seperti berikut:
   - **Description**: `Taruna Fest 2026 Form Handler`
   - **Execute as**: `Me (email Anda)`
   - **Who has access**: **`Anyone`** *(Wajib pilih "Anyone" agar form di landing page dapat mengirim data tanpa perlu login Google)*.
4. Klik tombol **Deploy**.
5. Jika diminta verifikasi izin (*Authorize access*), pilih akun Google Anda > klik *Advanced* > klik *Go to (project name) (unsafe)* > klik *Allow*.
6. Salin **Web app URL** yang muncul (formatnya: `https://script.google.com/macros/s/AKfycb.../exec`).

---

### Langkah 4: Masukkan URL ke `index.html`
Buka file `index.html`, cari baris berikut (di bagian bawah sebelum `</body>`):

```javascript
const GOOGLE_SHEETS_URL = window.TARUNA_SHEETS_URL || "https://script.google.com/macros/s/AKfycbyfE9HwJgU2yTj000000_SAMPLE/exec";
```

Ganti URL sample tersebut dengan URL Web App yang Anda dapatkan di Langkah 3.

---

## 📱 Alur Konfirmasi WhatsApp Sales (0813-2760-627)

Setelah calon peserta menekan tombol **"Kirim Pendaftaran & Konfirmasi WhatsApp"**:
1. Data langsung tersimpan di **Google Sheets**.
2. Browser secara otomatis membuka **WhatsApp ke `0813-2760-627`** dengan format teks rapi:

```text
*FORMULIR PENDAFTARAN TARUNA FEST 2026*
━━━━━━━━━━━━━━━━━━━━━━━━
Halo Tim Sales Akademi Taruna, saya ingin mengonfirmasi pendaftaran Taruna Fest 2026:

📋 *DATA PENDAFTAR:*
• *Nama Lengkap:* Muhammad Rayhan Pratama
• *No. WhatsApp:* 081234567890
• *Asal Sekolah & Kelas:* SMA Negeri 3 Yogyakarta / Kelas 12
• *Target Impian:* Akpol (Akademi Kepolisian - POLRI)
• *Kehadiran:* Siswa & Orang Tua (Sangat Disarankan)
• *Catatan:* Ingin konsultasi tes fisik

Data saya sudah terkirim ke formulir pendaftaran website. Mohon bantuan konfirmasi ketersediaan kuota (150 Peserta) dan informasi tiket/acara. Terima kasih!
```
3. Jika popup browser memblokir pembukaan tab baru secara otomatis, landing page menampilkan tombol hijau besar: **"📱 Klik di Sini jika WhatsApp tidak terbuka otomatis ↗"**.

