# Sistem Whistleblowing Anti-Perundungan (PKM-PM)

Aplikasi web pelaporan anonim tindak perundungan (bullying) untuk siswa SMP/SMA,
lengkap dengan alur SOP penanganan, notifikasi orang tua, Restorative Justice,
rekam jejak pelaku (matriks sanksi bertingkat), dan Papan Kehormatan kelas.

Dibangun dengan **Python Flask + SQLite** — ringan, gratis, tanpa API berbayar,
dan tetap bisa jalan lancar walau tanpa koneksi internet stabil (cocok untuk demo di sekolah).

---

## 1. Cara Menjalankan (Persiapan Sebelum Demo)

### a. Pastikan Python sudah terinstal
Cek dengan:
```
python --version
```
Minimal Python 3.9. Jika belum ada, download di https://www.python.org/downloads/

### b. Install dependensi
Buka terminal/command prompt di folder `whistleblowing-app`, lalu jalankan:
```
pip install -r requirements.txt
```

### c. Buat database awal (data contoh: akun login, daftar kelas, artikel edukasi)
```
python init_db.py
```
Akan muncul info akun login demo di layar.

### d. Jalankan aplikasi
```
python app.py
```
Buka browser ke: **http://localhost:5000**

Untuk mematikan server: tekan `CTRL + C` di terminal.

---

## 2. Akun Login Demo

| Peran | Username | Password |
|---|---|---|
| Guru BK / TPPK | `bk` | `bk12345` |
| Kepala Sekolah | `kepsek` | `kepsek12345` |
| Staf Tata Usaha | `tu` | `tu12345` |

⚠️ **Wajib ganti password ini** (lewat kode di `init_db.py`) sebelum dipakai di lingkungan sekolah sungguhan.

---

## 3. Alur Sistem Singkat

1. **Siswa** mengisi form di menu **Lapor Sekarang** secara anonim (tanpa perlu login) → dapat **kode tiket**.
2. **Guru BK** login → **Dashboard BK** → memverifikasi laporan (Valid/Ditolak).
3. Jika valid, Guru BK mengirim **notifikasi ke orang tua** (simulasi/email) dan menjadwalkan sesi **Restorative Justice**.
4. Setelah mediasi, Guru BK mencatat sanksi & menyelesaikan kasus → otomatis masuk **rekam jejak** pelaku.
5. **Kepala Sekolah** login → melihat **rekap agregat** (jumlah kasus, tren bulanan) tanpa melihat identitas pelapor/detail sensitif.
6. Publik bisa memantau **Papan Kehormatan** (kelas bebas laporan bulan ini) dan membaca **artikel edukasi**.

---

## 4. Fitur Anti-Spam / Anti-Laporan Palsu (Gratis, Tanpa API Berbayar)

- Verifikasi matematika sederhana (captcha) di form laporan.
- Rate limiting: maksimal 3 laporan per jam dari jaringan yang sama (bisa diubah di `config.py`).
- Kronologi wajib minimal 50 kata.
- Semua laporan tetap harus diverifikasi manual oleh Guru BK sebelum diproses lebih lanjut.

## 5. Autosave Draf Laporan (Perlindungan Psikologis Pelapor)

Form pelaporan otomatis menyimpan draf tulisan siswa ke **localStorage browser** (bukan ke server) setiap kali mereka mengetik, dengan jeda singkat agar tidak membebani perangkat. Jika halaman tidak sengaja tertutup, koneksi terputus, atau siswa butuh waktu untuk menenangkan diri lalu kembali lagi, tulisan mereka tidak hilang -- akan muncul banner "Draf tersimpan ditemukan" yang bisa dilanjutkan atau dihapus.

Pertimbangan penting:
- Draf **tidak pernah dikirim ke server** sampai tombol "Kirim Laporan" ditekan -- sepenuhnya tersimpan lokal di browser perangkat tersebut.
- Draf otomatis terhapus begitu laporan berhasil terkirim.
- Untuk komputer bersama (lab sekolah/perpustakaan), ada pengingat agar siswa menekan "Mulai Baru" setelah selesai, supaya draf tidak tertinggal bagi pengguna berikutnya.

## 6. Mode Gelap & Tombol Keluar Cepat (Stealth & Privacy)

Dua fitur ini bukan sekadar estetika, melainkan perlindungan psikologis tambahan -- karena korban perundungan sering melapor diam-diam (di kamar malam hari, sembunyi di toilet, dsb.), dan siswa harus bisa keluar dari halaman ini secepat mungkin jika ada orang mendekat.

**Mode Gelap (Dark Mode)**
Seluruh halaman otomatis mengikuti tema perangkat siswa lewat `@media (prefers-color-scheme: dark)` di CSS -- jika HP siswa disetel ke tema gelap, web ini ikut gelap tanpa perlu toggle manual atau setting tambahan. Ini mengurangi cahaya layar yang bisa menarik perhatian orang di sekitar saat malam hari.

**Tombol Keluar Cepat (Quick Exit)**
Tersedia di halaman **Lapor Sekarang** -- tombol merah melayang di pojok layar (kanan atas di desktop, kanan bawah di mobile) bertuliskan "🚪 Keluar Cepat", plus bisa dipicu dengan menekan tombol **Esc** di keyboard. Saat ditekan:
1. Draf tulisan langsung disimpan ke localStorage (memakai mekanisme autosave yang sama).
2. Halaman langsung berpindah ke Google memakai `location.replace()` -- bukan `location.href` biasa -- supaya halaman laporan **tidak tersimpan di riwayat browser**. Artinya, menekan tombol "kembali" setelah itu tidak akan membawa siapa pun kembali ke halaman ini.

Tujuan URL default adalah `https://www.google.com`, bisa diubah lewat variabel `EXIT_URL` di bagian akhir `templates/lapor.html` jika sekolah ingin mengarahkan ke halaman lain (mis. Wikipedia atau portal sekolah).

## 7. Ekspor Laporan (PDF & Excel)

Untuk kebutuhan dokumentasi resmi (rapat dewan guru, akreditasi sekolah, arsip), sistem menyediakan beberapa jenis ekspor:

| Ekspor | Diakses dari | Isi |
|---|---|---|
| Rekap seluruh laporan (Excel) | Dashboard Guru BK -> "Ekspor ke Excel" | Seluruh data laporan termasuk detail sensitif -- untuk arsip internal BK, **jangan disebarluaskan**. |
| Dokumentasi 1 kasus (PDF) | Halaman detail laporan -> "Unduh Dokumentasi (PDF)" | Kronologi, proses verifikasi, mediasi, sanksi, dan riwayat notifikasi untuk satu kasus -- siap dilampirkan sebagai bukti penanganan. |
| Rekap bulanan (PDF) | Dashboard Kepala Sekolah -> "Unduh Rekap Bulanan (PDF)" | Ringkasan agregat, tren 6 bulan, kelas rawan -- **tanpa** identitas pelapor/detail sensitif, siap dilampirkan ke laporan akreditasi. |
| Rekap bulanan (Excel) | Dashboard Kepala Sekolah -> "Unduh Rekap Bulanan (Excel)" | Versi spreadsheet dari rekap bulanan di atas, sekarang 5 sheet (Ringkasan, Tren 6 Bulan, Top 5 Kelas, Titik Rawan/Hotspot, Kombinasi Lokasi-Waktu). |

Semua ekspor dibuat langsung oleh server (memakai `openpyxl` untuk Excel dan `reportlab` untuk PDF), tanpa API pihak ketiga dan tanpa biaya tambahan.

## 8. Analitik Titik Rawan (Hotspot Analytics)

Dashboard Kepala Sekolah tidak cuma menampilkan *berapa banyak* kasus, tapi juga *di mana* dan *kapan* kasus paling sering terjadi -- supaya sekolah bisa mengambil kebijakan konkret (menambah CCTV, jadwal piket guru di jam/lokasi tertentu).

**Cara kerja:**
- Field `lokasi_kejadian` di form lapor diubah dari teks bebas menjadi **dropdown terkontrol** (Kantin, Toilet, Lorong/Koridor Kelas, Ruang Kelas, Lapangan/Halaman Sekolah, Belakang Sekolah, Media Sosial/Online, atau "Lainnya" + kolom teks) -- supaya datanya bisa diagregasi dengan bersih.
- Ditambah field baru `kategori_waktu` (Istirahat Pertama/Kedua, Pulang Sekolah, dst.) agar analitik bisa menyilangkan **lokasi × waktu**, jauh lebih actionable daripada lokasi saja (mis. "kantin saat istirahat kedua" bisa langsung diterjemahkan jadi jadwal piket spesifik).
- Ditampilkan sebagai bar chart horizontal berbasis CSS murni (tanpa CDN/library chart eksternal), lengkap dengan badge **"⚠ Naik 2 bln"** kalau suatu lokasi menunjukkan tren kenaikan 2 bulan berturut-turut (heuristik tren sederhana, bukan model machine learning).

**Kehati-hatian statistik & privasi yang sengaja diterapkan** (penting untuk dijelaskan ke juri jika ditanya):
- **Bukan analitik prediktif** dalam arti model ML -- ini analitik deskriptif/diagnostik dengan tambahan flag tren sederhana. Istilah "prediktif" sengaja dihindari di penamaan fitur supaya tidak overclaim.
- **Peringatan sampel kecil**: jika total data hotspot di bawah ambang tertentu (default 10, bisa diubah lewat `HOTSPOT_MIN_SAMPEL` di `config.py`), dashboard menampilkan peringatan bahwa polanya "masih indikasi awal" -- mencegah sekolah mengambil keputusan besar (mis. pasang CCTV) berdasarkan sampel yang secara statistik masih rapuh.
- **Cell suppression** pada tabel kombinasi lokasi x waktu: kombinasi dengan hanya 1 kasus tidak ditampilkan sama sekali, dan yang di bawah ambang `HOTSPOT_SEL_MIN_TAMPIL` (default 3) angkanya disamarkan jadi "beberapa kasus" -- supaya data yang terlalu spesifik tidak berpotensi mengarah ke identifikasi kasus/siswa tertentu, meski identitas pelapor sendiri tidak pernah disimpan.
- Data lokasi lama yang tidak sesuai daftar baku (misalnya dari isian bebas versi sebelumnya) otomatis dikelompokkan ke kategori **"Lainnya"** saat dihitung, bukan diabaikan.

## 9. Gerbang Persetujuan SKBB (Career & Academic Impact)

Fitur ini menghubungkan sistem pelaporan dengan penerbitan **Surat Keterangan Berkelakuan Baik (SKBB)** -- dokumen yang dibutuhkan siswa SMA untuk jalur SNBP dan siswa SMK untuk magang/melamar kerja. Tujuannya memberi efek jera nyata: tindakan perundungan yang terbukti berat bisa berdampak pada dokumen penting masa depan siswa.

### Kenapa BUKAN pemblokiran otomatis?

Desain awal yang dipertimbangkan adalah sistem otomatis memblokir SKBB begitu ada kasus "Berat" yang belum selesai. Setelah dikaji, pendekatan itu punya beberapa risiko serius:

- **Bertentangan dengan semangat restorative justice** di Permendikbudristek No. 46 Tahun 2023 (regulasi yang jadi payung hukum sistem ini) -- yang menekankan pemulihan, bukan diskualifikasi permanen.
- **Berisiko menekan pelapor**: makin tinggi taruhan bagi terduga pelaku (masa depan akademis/karier), makin besar dorongan untuk menekan pelapor agar mencabut laporan -- bertentangan dengan seluruh investasi privasi & keamanan (anonimitas, Quick Exit, dsb.) yang sudah dibangun di sistem ini.
- **Siswa adalah anak di bawah umur** -- keputusan sepenting ini semestinya tidak diambil algoritma tanpa nuansa/konteks.
- **Rapuh terhadap tenggat waktu**: SNBP punya deadline ketat: jika kasus tertahan lebih lama dari SOP (bukan salah siswa), kerugian yang timbul bisa permanen padahal siswa belum tentu terbukti bersalah.

Karena itu, sistem ini memakai pendekatan **Human-in-the-Loop**: sistem hanya mengangkat flag dan menyajikan informasi -- keputusan akhir SELALU di tangan manusia (Kepala Sekolah), dengan jejak audit penuh.

### Alur Kerja

1. Saat Guru BK menutup kasus (`aksi=selesaikan`), mereka **wajib** memilih Tingkat Pelanggaran final (Ringan/Sedang/Berat) -- sebelumnya kolom ini ada di skema database tapi tidak pernah benar-benar diisi di kode manapun; sekarang WAJIB sebelum status bisa jadi "Selesai".
2. Jika dipilih **Berat**, sistem otomatis membuat baris baru di tabel `status_skbb` berstatus **"Menunggu Persetujuan"** -- bukan blokir permanen, hanya flag.
3. Kepala Sekolah melihat kasus ini di panel **"Antrean Persetujuan SKBB"** pada dashboardnya -- lengkap dengan sanksi yang dijatuhkan (bukan kronologi mentah, tetap menjaga privasi korban) -- dan memutuskan **"Setujui Sementara"** (mis. untuk kebutuhan mendesak seperti deadline SNBP) atau **"Tolak"**, keduanya wajib disertai alasan tertulis.
4. Staf **Tata Usaha (TU)** yang ingin mencetak SKBB memakai halaman khusus (`/tu/cek-skbb`) untuk mengecek nama+kelas siswa -- **hanya melihat status hijau/kuning**, TIDAK PERNAH melihat detail kasus, kronologi, atau riwayat pelanggaran.
5. Guru BK bisa mengajukan **Rehabilitasi** kapan saja (dengan alasan/bukti wajib tertulis, mis. "sudah 5 sesi konseling, 1 semester tanpa pengulangan") -- begitu direhabilitasi, status siswa di mata TU kembali **identik** dengan siswa yang tidak pernah punya catatan sama sekali (lihat poin privasi di bawah).

### Segregation of Duties (Pemisahan Wewenang)

| Peran | Bisa Lihat | Tidak Bisa |
|---|---|---|
| **Guru BK** | Detail kasus penuh, bisa tetapkan tingkat pelanggaran, bisa ajukan rehabilitasi | Tidak bisa menyetujui/menolak SKBB (itu wewenang Kepsek) |
| **Kepala Sekolah** | Antrean persetujuan: nama siswa, kode kasus, sanksi yang dijatuhkan | Kronologi mentah / detail investigasi (tetap privasi Guru BK/TPPK) |
| **Tata Usaha** | Status boleh/tidak boleh cetak SKBB (hijau/kuning) | Detail kasus apa pun, termasuk alasan di balik status tersebut |

### Kehati-hatian yang Diterapkan

- **Bukan pemblokiran permanen** -- semua keputusan (approve/tolak/rehabilitasi) tercatat dengan alasan wajib, siapa yang memutuskan, dan kapan (kolom `diputuskan_oleh` & `waktu_keputusan` di tabel `status_skbb`, plus dicatat juga ke `log_akses`).
- **Privasi pasca-rehabilitasi**: begitu status berubah jadi "Direhabilitasi" atau "Disetujui Sementara", tampilan yang dilihat TU **sama persis** dengan siswa yang tidak pernah punya catatan sama sekali -- fungsi `cek_status_skbb()` di `app.py` sengaja dirancang tidak membedakan keduanya, supaya rehabilitasi benar-benar bersih, bukan sekadar label yang diam-diam tetap "menandai" siswa tsb.
- **Risiko pencocokan identitas (NIS)**: sistem ini belum terhubung ke basis data induk siswa sekolah (Dapodik dsb.), sehingga pencocokan default memakai **nama + kelas** (rawan tabrakan nama sama atau typo). Untuk mengurangi risiko ini, Guru BK bisa mengisi **NIS** siswa terduga pelaku saat tahap verifikasi (opsional, tapi sangat disarankan untuk kasus yang berpotensi Berat). Halaman cek TU juga menampilkan pengingat untuk verifikasi manual ke Guru BK jika ada keraguan identitas.
- Hanya tingkat pelanggaran **Berat** yang memicu gerbang ini -- kasus Ringan/Sedang tidak memengaruhi status SKBB sama sekali.

> **Untuk pengembangan lanjutan pasca-PKM:** idealnya fitur ini terintegrasi dengan basis data siswa resmi sekolah (by NIS, bukan nama+kelas) untuk menghilangkan risiko salah-cocok identitas sepenuhnya.

## 10. Digital Evidence Vault (Bukti Cyberbullying)

Cyberbullying punya tantangan khas: pelaku bisa menghapus pesan/postingan dalam hitungan detik setelah tahu ketahuan. Fitur ini memperkuat bukti yang diunggah siswa (screenshot chat, dsb.) supaya lebih sulit dibantah keberadaannya -- **bukan** dengan klaim berlebihan soal "bukti sah secara hukum", melainkan dengan praktik forensik digital yang jujur dan dapat dipertanggungjawabkan: **hash integritas**.

### Kenapa bukan sekadar "watermark = bukti sah"?

Menempelkan cap waktu di atas gambar itu gampang dipalsukan balik (tinggal edit lagi gambarnya). Watermark visual di sistem ini **hanya untuk kenyamanan tampilan/cetak**, bukan klaim keabsahan hukum. Yang benar-benar menjaga integritas bukti adalah **hash SHA-256** -- sidik jari digital dari file:

1. Begitu bukti diunggah, sistem menghitung hash SHA-256 dari file **persis seperti yang diterima**, sebelum diubah apa pun.
2. File asli **tidak pernah dimodifikasi** setelah itu -- supaya hash-nya tetap valid selamanya. Watermark dibuat di **salinan terpisah** (`..._watermark.jpg`), khusus untuk tampilan dan cetak PDF.
3. Jika suatu saat file "asli" di server berbeda hash-nya dari yang tercatat di database, itu tanda file sudah diubah/rusak sejak diunggah.

**Batasan yang jujur perlu disampaikan ke siswa/juri:** hash ini membuktikan *file tidak berubah setelah diunggah ke sistem* -- BUKAN membuktikan bahwa screenshot-nya asli/tidak diedit *sebelum* diunggah. Itu di luar jangkauan teknis sistem manapun tanpa integrasi API resmi platform media sosial (di luar cakupan proyek PKM ini). Kejujuran soal batasan ini justru memperkuat kredibilitas sistem di depan juri yang paham forensik digital, dibanding mengklaim "bukti sah secara hukum" yang mudah dipatahkan.

### Cara Kerja Teknis

- Watermark dibuat otomatis dengan `Pillow` (PIL) -- bar semi-transparan di bagian bawah gambar berisi waktu unggah + kode tiket + nama sekolah, dengan ukuran font otomatis menyesuaikan lebar gambar (screenshot HP yang sempit tetap terbaca).
- Hanya berlaku untuk bukti berformat **gambar** (JPG/PNG). Bukti berformat **PDF** tetap dihitung hash-nya, tapi tidak diberi watermark visual (dijelaskan apa adanya di tampilan, tidak dipaksakan).
- Guru BK melihat kedua versi di halaman detail laporan: salinan berwatermark (tampilan utama) dan tautan ke file asli tanpa watermark.
- Saat dokumentasi kasus diunduh sebagai PDF, gambar bukti (versi watermark) ikut tercetak beserta hash SHA-256 lengkap dan penjelasan batasannya.
- Font yang dipakai (`DejaVuSans-Bold.ttf`) di-bundle langsung di `static/fonts/` -- supaya watermark tetap konsisten di laptop mana pun tanpa bergantung font sistem yang belum tentu terpasang.

### Manajemen Kapasitas Penyimpanan (Mencegah Server Kehabisan Disk)

Foto dari kamera HP modern bisa berukuran 5-15 MB dan beresolusi sangat tinggi -- kalau disimpan apa adanya, server dengan disk terbatas (mis. paket hosting 2 GB) bisa penuh hanya dalam hitungan minggu. Solusinya **bukan** menolak file besar (itu cuma memindahkan beban ke siswa yang harus repot kompres manual saat sedang tidak dalam kondisi terbaik), melainkan **mengompres otomatis di server**:

- Foto diperkecil ke maksimal 1920px di sisi terpanjang, dan dikompres ke JPEG kualitas 85 -- biasanya menghasilkan pengurangan ukuran **70-85%** tanpa mengorbankan keterbacaan teks chat, sebelum disimpan ke disk sama sekali.
- Orientasi foto otomatis diperbaiki (`ImageOps.exif_transpose`) -- banyak foto kamera HP tersimpan "miring" dan hanya diputar lewat metadata EXIF, kalau tidak ditangani foto bisa tampil terbalik.
- **Urutan prosesnya penting**: kompresi dilakukan DULU, baru hash SHA-256 dihitung dari hasil kompresi -- bukan dari file mentah upload. Ini keputusan arsitektur yang disengaja: hasil kompresi itulah yang dianggap "bukti resmi" sejak masuk sistem, dan sejak titik itu tidak pernah diubah lagi -- sehingga jaminan integritas hash tetap konsisten walau ukurannya sudah dikecilkan.
- Batas unggah dinaikkan dari 5 MB menjadi **8 MB** (karena toh akan dikompres otomatis) -- mengurangi kemungkinan foto asli ditolak mentah-mentah.
- Dashboard Guru BK menampilkan **indikator total penyimpanan bukti terpakai**, dengan peringatan otomatis kalau mendekati ambang (`STORAGE_WARNING_MB` di `config.py`, default 1500 MB -- sesuaikan dengan kapasitas hosting yang sebenarnya dibeli sekolah).

### Kegagalan Penyimpanan Bukti TIDAK PERNAH Menggagalkan Laporan (Graceful Degradation)

Prinsip pentingnya: **teks kronologi jauh lebih penting daripada lampiran foto**, dan kegagalan infrastruktur (mis. disk penuh) tidak boleh ditanggung siswa yang baru saja memberanikan diri melapor.

- Kalau proses simpan bukti gagal (disk penuh, file korup, dsb.), laporan **tetap tersimpan lengkap** dengan kronologi dan seluruh data lain -- hanya bagian buktinya yang kosong.
- Siswa tetap mendapat kode tiket dan pesan yang menenangkan: *"Laporanmu berhasil terkirim dan akan tetap diproses"* -- tanpa detail teknis yang membingungkan (mis. tidak disebutkan "disk penuh").
- Sistem membedakan tegas antara **"memang tidak ada bukti"** (siswa tidak unggah apa-apa) dengan **"bukti gagal tersimpan"** (kolom `bukti_upload_gagal` di database) -- supaya Guru BK tahu ada upaya unggah yang gagal, bukan sekadar tidak ada bukti sama sekali, dan bisa memeriksa kapasitas server.
- File yang sempat tertulis sebagian (kalau proses gagal di tengah jalan) otomatis dibersihkan -- tidak ada file "setengah jadi" yang tertinggal di disk.

## 11. Tentang Notifikasi Orang Tua

Secara default sistem berjalan dalam **mode simulasi** (tidak butuh kredensial apapun, aman untuk demo).
Setiap kali Guru BK menekan tombol "Kirim Notifikasi", pesan akan tercatat di sistem sebagai bukti bahwa
prosedur pemanggilan telah dijalankan.

Ada 3 pilihan mode notifikasi, diatur lewat `MODE_NOTIFIKASI` di `config.py`:

### a. Simulasi (default)
Tidak perlu setup apapun. Cocok untuk demo/presentasi PKM.

### b. Email sungguhan (SMTP)
```python
MODE_NOTIFIKASI = "email_smtp"
SMTP_HOST = "smtp.gmail.com"
SMTP_PORT = 587
SMTP_USER = "email_sekolah@gmail.com"
SMTP_PASSWORD = "app-password-gmail"   # gunakan App Password, bukan password akun Gmail biasa
```

### c. WhatsApp sungguhan (Fonnte) — untuk pengembangan lanjutan bersama sekolah
[Fonnte](https://fonnte.com) menyediakan API WhatsApp dengan paket gratis untuk skala kecil, cocok
sebagai langkah lanjutan program ini setelah PKM selesai (tidak wajib diaktifkan untuk demo).

**Langkah aktivasi:**
1. Daftar akun di https://fonnte.com
2. Hubungkan nomor WhatsApp sekolah dengan cara scan QR code di dashboard Fonnte (mirip WhatsApp Web)
3. Salin **Token** dari dashboard Fonnte
4. Buka `config.py`, ubah menjadi:
   ```python
   MODE_NOTIFIKASI = "whatsapp_fonnte"
   FONNTE_TOKEN = "isi-token-dari-dashboard-fonnte"
   ```
5. Saat Guru BK menekan "Kirim Notifikasi" di dashboard, isi kolom **Nomor WhatsApp Orang Tua**
   (format: `6281234567890`, tanpa tanda `+` atau spasi)

**Kode integrasinya sudah tersedia** di `app.py` pada fungsi `kirim_wa_fonnte()` — tinggal aktifkan,
tidak perlu menulis ulang. Fungsi ini memanggil endpoint `https://api.fonnte.com/send` dengan token
sebagai header Authorization dan mengirim `target` (nomor tujuan) + `message` (isi pesan).

> Catatan: karena sistem demo ini belum terhubung ke basis data siswa resmi sekolah (yang biasanya
> menyimpan kontak orang tua), nomor WhatsApp untuk sementara diisi manual oleh Guru BK saat proses
> notifikasi. Untuk versi produksi, sebaiknya nomor ini ditarik otomatis dari data induk siswa sekolah.

## 12. PWA (Progressive Web App) -- Bisa "Diinstal" di HP Tanpa App Store

Web ini sudah dilengkapi manifest & service worker, sehingga saat siswa membuka web ini di HP, browser akan menawarkan opsi **"Tambahkan ke Layar Utama" / "Add to Home Screen"** -- setelah itu muncul ikon seperti aplikasi asli, terbuka tanpa address bar (mode `standalone`), dan tidak perlu mengetik URL lagi setiap kali.

**Kenapa nama & ikonnya sengaja generik ("Portal BK Sekolah", bukan "Lapor Bullying")?**
Ikon dan nama aplikasi akan muncul di home screen HP siswa -- terlihat oleh siapa pun yang kebetulan melihat layar tersebut, termasuk berpotensi pelaku atau orang tua pelaku. Supaya tidak membocorkan bahwa siswa memakai aplikasi pelaporan perundungan, nama dan ikon dibuat netral (ikon centang sederhana, nama "Portal BK Sekolah"). Ini sejalan dengan prinsip *stealth & privacy* yang sama seperti fitur Mode Gelap dan Keluar Cepat. Bisa disesuaikan lewat `PWA_APP_NAME`, `PWA_SHORT_NAME`, `PWA_DESCRIPTION` di `config.py`.

**Prinsip keamanan pada service worker (`static/service-worker.js`):**
Karena aplikasi ini menangani data sensitif, service worker **sengaja tidak pernah menyimpan** halaman-halaman berikut ke cache perangkat -- selalu diambil langsung dari server:
- Form pelaporan (`/lapor`), cek status (`/cek-status`)
- Semua halaman login/dashboard Guru BK & Kepala Sekolah (`/login`, `/bk/*`, `/kepsek/*`)

Yang di-cache **hanya** aset statis tak-sensitif (CSS, ikon) supaya tampilan tetap cepat dimuat, plus satu halaman offline generik sebagai fallback saat benar-benar tanpa koneksi. Jika suatu saat perlu menambah halaman ke daftar larangan cache, edit `NEVER_CACHE_PREFIXES` di `static/service-worker.js`.

**Cara mengujinya:**
1. Jalankan aplikasi lalu buka lewat HP di jaringan yang sama (atau `localhost` di Chrome desktop).
2. Chrome/Edge akan menampilkan ikon "instal" di address bar, atau notifikasi banner "Tambahkan ke Layar Utama" di Android.
3. Di Safari iOS: buka menu Share -> "Add to Home Screen" (Apple belum mendukung prompt otomatis seperti Android).

> Catatan: prompt instal PWA umumnya hanya muncul di koneksi **HTTPS** atau `localhost`. Jika nanti di-deploy ke server sekolah, pastikan diakses lewat HTTPS agar fitur ini aktif.

## 13. Rencana Pengembangan Lanjutan (jika sekolah ingin melanjutkan pasca-PKM)

- Integrasi nomor WhatsApp orang tua & NIS siswa otomatis dari basis data induk sekolah/Dapodik (saat ini diisi manual per kasus, lihat bagian 9 soal risiko pencocokan nama+kelas).
- Login siswa/orang tua untuk melihat progres tanpa kode tiket.
- Autentikasi dua faktor (2FA) untuk akun Guru BK, Kepala Sekolah & TU.
- Eskalasi kasus otomatis ke Dinas Pendidikan/Dinas PPPA untuk kasus berat sesuai
  Permendikbudristek No. 46 Tahun 2023.

---

## 14. Struktur Folder

```
whistleblowing-app/
├── app.py              # Routing & logika utama
├── config.py            # Pengaturan (SLA, rate limit, PWA, hotspot, SKBB, dsb.) + generator SECRET_KEY
├── db.py                 # Skema database
├── init_db.py            # Script pembuat data awal
├── export.py             # Pembuat file Excel & PDF (rekap dan dokumentasi kasus)
├── requirements.txt
├── templates/            # Halaman HTML (Jinja2), termasuk tu_dashboard.html
├── static/
│   ├── css/style.css
│   ├── icons/            # Ikon PWA (icon-192, icon-512, maskable, apple-touch-icon)
│   ├── fonts/            # Font DejaVu untuk watermark bukti (di-bundle, tak bergantung OS)
│   ├── service-worker.js # Service worker PWA (lihat bagian 12 di atas)
│   └── offline.html      # Halaman fallback saat tidak ada koneksi
├── secure_uploads/       # Bukti laporan -- DI LUAR static/, hanya via /bk/bukti/ (lihat bagian 15)
└── instance/
    ├── whistleblowing.db  # Database (dibuat otomatis, termasuk tabel status_skbb)
    └── secret_key.txt     # Kunci sesi login, dibuat otomatis, JANGAN dibagikan (lihat bagian 15)
```

---

## 15. Keamanan Aplikasi (Wajib Dibaca Sebelum Deploy)

### Kunci Sesi Login (SECRET_KEY) Otomatis & Unik

Versi awal sistem ini sempat memakai `SECRET_KEY` statis yang tertulis langsung di `config.py` -- ini berbahaya karena kode ini akan disebarluaskan (lampiran PKM, GitHub, dsb). Kalau kuncinya tertulis di kode sumber publik dan sekolah lupa menggantinya, siapa pun yang paham cara kerja Flask bisa memalsukan cookie sesi dan **login sebagai Guru BK/Kepala Sekolah/TU tanpa password sama sekali**.

Sekarang sistem otomatis membuat kunci acak (256-bit, lewat `secrets.token_hex`) saat pertama kali dijalankan, dan menyimpannya di `instance/secret_key.txt` -- **bukan di kode sumber**. Tiap instalasi punya kunci sendiri secara otomatis, sekolah tidak perlu mengedit apa pun.

⚠️ **Jangan pernah membagikan isi `instance/secret_key.txt`** (mis. dengan meng-commit ke Git publik, mengunggah screenshot, dsb.) -- kalau bocor, keamanan sesi login jadi percuma. Folder `instance/` sudah dikeluarkan dari zip yang dibagikan (dibuat otomatis saat pertama kali `python app.py` dijalankan).

### Bukti Cyberbullying: Tidak Bisa Diakses Tanpa Login

Awalnya bukti unggahan (screenshot chat, dsb.) tersimpan di `static/uploads/` dan disajikan lewat file statis Flask biasa -- artinya **siapa pun yang tahu atau menebak URL file-nya bisa membuka bukti sensitif tanpa login sama sekali**, bertentangan dengan seluruh prinsip Digital Evidence Vault yang dibangun.

Sekarang:
- File bukti disimpan di `secure_uploads/` -- folder **di luar** `static/`, sehingga tidak bisa dijangkau lewat URL statis biasa sama sekali.
- Satu-satunya jalur resmi untuk membukanya adalah route `/bk/bukti/<filename>`, yang **wajib login sebagai Guru BK** (`@login_required(role="guru_bk")`) -- Kepala Sekolah dan TU eksplisit ditolak (403) kalau mencoba membuka route ini, sejalan dengan prinsip pemisahan wewenang yang sudah dibangun di fitur SKBB.
- Percobaan *path traversal* (mis. `/bk/bukti/../../app.py`) otomatis ditolak oleh `send_from_directory` Flask.

### Catatan Keamanan Data Lainnya

- Identitas pelapor **tidak pernah diminta maupun disimpan** — sistem hanya menyimpan hash IP untuk keperluan rate-limiting, bukan IP mentah.
- Setiap akses staf ke detail laporan (termasuk membuka file bukti) tercatat di **Log Akses** (audit trail).
- Dashboard Kepala Sekolah hanya menampilkan data agregat, bukan detail kasus per individu.
- Password staf memakai hash `werkzeug.security` (bukan disimpan sebagai teks biasa).
- Selaras dengan semangat UU No. 27 Tahun 2022 tentang Pelindungan Data Pribadi (UU PDP).
- **Soal screenshot bukti cyberbullying**: gambar chat/media sosial berpotensi memuat identitas pihak ketiga (siswa lain dalam grup, dsb.) yang tidak terkait langsung dengan kasus. Sistem tidak melakukan sensor/blur otomatis terhadap konten ini -- Guru BK disarankan berhati-hati saat menangani dan menyebarluaskan file bukti, dan mempertimbangkan meminta pelapor meng-crop/blur bagian yang tidak relevan sebelum unggah jika memungkinkan.

### Yang Masih Jadi Pekerjaan Rumah (Belum Dikerjakan)

Demi transparansi -- ada beberapa celah keamanan lain yang sudah teridentifikasi tapi **belum** diperbaiki di versi ini, sengaja ditunda karena PKM ini masih tahap prototipe:

- **Proteksi CSRF**: form-form di sistem ini belum punya token CSRF, secara teori rentan terhadap serangan cross-site request forgery pada aksi sensitif (mis. keputusan SKBB) jika staf yang login membuka situs berbahaya di tab lain.
- **Proteksi brute-force di halaman login**: belum ada pembatasan jumlah percobaan login yang gagal.
- **Idle session timeout**: sesi login belum otomatis berakhir setelah periode tidak aktif tertentu -- penting untuk komputer sekolah yang dipakai bergantian.

Kalau sistem ini akan benar-benar dipakai sekolah (bukan cuma demo PKM), disarankan menambal ketiga hal ini terlebih dahulu -- **apalagi kalau dipilih opsi hosting publik di bagian selanjutnya**, karena begitu publik, siapa pun di internet (bukan cuma yang terhubung ke wifi sekolah) bisa mencoba mengeksploitasi ketiga celah ini.

---

## 16. Strategi Deployment: Publik vs Intranet Sekolah

Ini keputusan penting yang sebaiknya dijelaskan dengan alasannya di proposal PKM, karena berdampak langsung ke aspek psikologis pelapor -- bukan cuma soal teknis.

### Kenapa Direkomendasikan Hosting Publik (Bukan Intranet)

| Pertimbangan | Intranet Sekolah (Wi-Fi lokal saja) | Hosting Publik (Internet) |
|---|---|---|
| Kapan siswa bisa melapor | Hanya saat berada di lingkungan sekolah | Kapan saja, dari mana saja -- termasuk dari rumah, malam hari, saat merasa aman |
| Fitur Quick Exit, Dark Mode, PWA | Kurang relevan -- risiko fisik (pelaku di sekitar) tetap tinggi karena harus di sekolah | **Baru benar-benar berfungsi sesuai tujuan** -- fitur ini dirancang untuk situasi "melapor diam-diam dari mana saja" |
| Biaya kuota siswa | Gratis (pakai wifi sekolah) | Butuh kuota/wifi sendiri -- pertimbangan nyata untuk siswa kurang mampu |
| Kompleksitas & biaya infrastruktur | Lebih sederhana, cukup satu laptop/PC di sekolah | Butuh domain, VPS, sertifikat HTTPS -- ada biaya berkala kecil |

**Kesimpulan yang disarankan:** hosting publik lebih selaras dengan tujuan perlindungan psikologis pelapor, dengan catatan kuota data untuk siswa kurang mampu perlu dipikirkan sebagai isu keadilan akses terpisah (bisa jadi poin diskusi di proposal, bukan halangan).

### Konsekuensi Teknis yang WAJIB Dipenuhi Kalau Memilih Hosting Publik

1. **HTTPS wajib, bukan opsional.** Tanpa HTTPS, data laporan dan bukti cyberbullying dikirim dalam bentuk tidak terenkripsi lewat internet publik -- bisa disadap di jaringan wifi publik. HTTPS juga prasyarat teknis agar fitur PWA "Tambahkan ke Layar Utama" berfungsi sama sekali (lihat bagian 12) -- di localhost ia bekerja tanpa HTTPS, tapi di domain publik sungguhan tidak akan muncul tanpa HTTPS.
2. **Server produksi yang layak**, bukan `python app.py` biasa (server bawaan Flask secara eksplisit menyatakan dirinya tidak cocok untuk produksi). Disarankan: **Gunicorn** atau **Waitress** menjalankan aplikasi Flask di belakang **Nginx** (mengurus HTTPS & jadi gerbang depan).
3. **Disk penyimpanan permanen.** Beberapa layanan hosting gratis/murah punya disk yang "sementara" (terhapus tiap restart) -- kalau dipakai, database dan bukti unggahan bisa hilang total. Pilih VPS dengan disk permanen (mis. DigitalOcean, Lightsail AWS, atau VPS lokal Indonesia), bukan platform serverless/ephemeral.
4. **Tiga PR keamanan di bagian 15** (CSRF, brute-force login, idle timeout) sebaiknya diselesaikan dulu sebelum benar-benar go-live publik.
5. **Cadangan (backup) berkala** untuk file `instance/whistleblowing.db` -- karena sekarang berisi data sungguhan sekolah, bukan cuma demo.

### Rekomendasi Domain

Gunakan domain resmi **`.sch.id`** -- domain khusus lembaga pendidikan Indonesia yang dikelola PANDI, biayanya sekitar Rp75.000-200.000/tahun (jauh lebih murah dari domain komersial `.com`), dan syaratnya cukup surat permohonan Kepala Sekolah + NPSN yang sudah terdaftar di Dapodik (otomatis dimiliki semua sekolah negeri/swasta terdaftar).

Sesuai prinsip *stealth* yang sudah diterapkan di penamaan PWA (bagian 12), sebaiknya nama subdomain-nya **tidak eksplisit** menyebut "bullying" atau "lapor" -- misalnya `portalbk.smpnX.sch.id` -- supaya URL yang mungkin tertinggal di riwayat browser bersama tidak langsung membocorkan fungsinya.

### Yang Sudah Disiapkan di Kode untuk Mendukung Deployment

- `.gitignore` -- memastikan file sensitif (`instance/`, `secure_uploads/`, kredensial) tidak pernah ikut ter-commit kalau kode ini diunggah ke repo publik (GitHub) untuk lampiran PKM.
- **SQLite WAL mode** diaktifkan (`PRAGMA journal_mode=WAL`) -- meningkatkan keandalan saat banyak pengguna mengakses bersamaan (siswa submit laporan sementara BK/Kepsek membuka dashboard), yang jadi lebih mungkin terjadi begitu sistem diakses publik dibanding cuma dari satu laptop sekolah.
- Kredensial sensitif (SMTP, token Fonnte) sekarang bisa diisi lewat **environment variable** (`SMTP_HOST`, `SMTP_USER`, `SMTP_PASSWORD`, `FONNTE_TOKEN`, `NAMA_SEKOLAH`) alih-alih ditulis langsung di `config.py` -- praktik konfigurasi produksi yang lebih aman, karena kredensial asli sekolah tidak tertulis di kode yang mungkin diunggah ke repo publik.

> Panduan langkah-demi-langkah deploy ke VPS (instalasi Gunicorn/Nginx/Certbot, penyiapan systemd service, dsb.) belum disertakan di dokumen ini -- di luar cakupan kode aplikasi, dan sebaiknya disesuaikan dengan platform hosting final yang dipilih.

---

---

Selamat presentasi PKM! 🎓
