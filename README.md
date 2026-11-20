🏢 Sistem Perusahaan Property (Real Estate Management System)
   
Sistem Perusahaan Property adalah proyek perancangan dan implementasi sistem basis data relasional berbasis PostgreSQL untuk mengelola operasional inti pada perusahaan/agen properti (Real Estate Management System). Sistem ini dirancang untuk mendigitalisasi pengelolaan aset properti, sumber daya manusia, serta otomasi transaksi penjualan dan penyewaan secara real-time menggunakan fitur-fitur SQL tingkat lanjut.

📁 Struktur Folder Repository
Agar repositori GitHub rapi, terorganisir, dan mudah dijalankan oleh pengguna lain, proyek ini disusun menggunakan struktur direktori berikut:
sistem-perusahaan-property/
├── backup/
│   └── sistem_propertiDB.sql            # File backup/dump lengkap database PostgreSQL
├── docs/
│   └── Laporan_Akhir_Project_WBD.docx   # Laporan akhir lengkap perancangan & pengujian sistem
├── erd/
│   ├── ERD_Dependency.pdf               # Diagram konseptual (Dependency & Partisipasi Entitas)
│   └── ERD_Full.pdf                     # Diagram ERD lengkap (Atribut, PK, FK, dan Tipe Data)
├── sql/
│   ├── 01_ddl_tables_and_enums.sql      # Definisi tipe data ENUM dan pembuatan 10 tabel
│   ├── 02_functions_and_triggers.sql    # Definisi PL/pgSQL Function dan Trigger otomatis
│   ├── 03_dml_seed_records.sql          # Data awal (seeding) untuk tabel master dan transaksi
│   ├── 04_views.sql                     # Definisi 3 View untuk pelaporan manajerial
│   └── full_script.sql                  # Skrip SQL gabungan keseluruhan dari awal hingga akhir
└── README.md                            # Dokumentasi utama repositori proyek

🎯 Pilar Utama Sistem
Sistem ini memfasilitasi tiga pilar utama operasional perusahaan properti:
Manajemen Aset Properti: Mengelola katalog unit properti (Apartemen, Rumah, Ruko, Villa, Tanah, Gudang, Pabrik, Kost, dll.), klasifikasi tipe properti, detail lokasi administratif (provinsi hingga kode pos), inventaris fasilitas per unit beserta kondisinya, serta pemantauan status ketersediaan (Tersedia, Terjual, Disewa).
Manajemen SDM & Klien: Menyimpan profil klien (pembeli/penyewa) dan data pegawai (agen pemasaran, admin, keuangan, operasional, dll.) yang terintegrasi dengan pencatatan log absensi harian pegawai.
Manajemen Transaksi Terpadu: Menangani transaksi penjualan (transaksi_jual) dan penyewaan (transaksi_sewa) dengan dukungan metode pembayaran Cash maupun Kredit, lengkap dengan kalkulasi otomatis sisa tagihan dan status pelunasan.

🗄️ Arsitektur & Skema Basis Data
1. Tipe Data Kustom (ENUM)
Sistem mendefinisikan 6 tipe data ENUM untuk menjaga konsistensi nilai atribut:
STATUS_PROPERTY: 'Tersedia', 'Terjual', 'Disewa'
STATUS_PEGAWAI: 'Aktif', 'Tidak aktif'
STATUS_HADIR: 'Hadir', 'Izin', 'Sakit', 'Alpa'
STATUS_SEWA: 'Aktif', 'Selesai'
GENDER: 'Perempuan', 'Laki laki'
VIA: 'Cash', 'Kredit'
2. Daftar Tabel (10 Entitas)
Kategori
Nama Tabel
Keterangan & Relasi Utama
Master
tipe_property
Kategori jenis properti (1:N dengan property).
Master
lokasi
Detail wilayah geografis properti (1:N dengan property).
Master
fasilitas
Daftar master fasilitas bangunan/lingkungan.
Master
property
Data unit fisik properti beserta luas, harga, diskon, dan status.
Master
pegawai
Data karyawan/agen perusahaan beserta jabatan dan gaji pokok.
Master
klien
Data pelanggan yang melakukan pembelian atau penyewaan.
Perantara
detail_fasilitas
Tabel penghubung relasi Many-to-Many (M:N) antara property dan fasilitas (mencatat kuantitas & kondisi).
Transaksi
absensi
Log kehadiran harian pegawai (1:N dengan pegawai, Composite Unique pada id_pegawai & tanggal).
Transaksi
transaksi_jual
Nota penjualan properti (relasi One-to-One (1:1) dengan property agar 1 unit hanya terjual 1 kali).
Transaksi
transaksi_sewa
Nota kontrak sewa properti (relasi One-to-Many (1:N) dengan property untuk penyewaan berulang).

3. Validasi Data & Aturan Bisnis (Constraint)
Integritas Referensial: Penerapan PRIMARY KEY dan FOREIGN KEY di seluruh tabel relasional, termasuk ON DELETE CASCADE pada tabel absensi.
Pencegahan Duplikasi (UNIQUE): Diterapkan pada no_hp (pegawai & klien), kombinasi (id_pegawai, tanggal) pada absensi, dan id_property pada transaksi_jual.
Validasi Logika (CHECK): Memastikan harga, luas tanah/bangunan, gaji, dan kuantitas fasilitas bernilai positif (> 0 atau >= 0), serta memvalidasi tanggal_selesai > tanggal_mulai pada kontrak sewa.
Nilai Bawaan (DEFAULT): Otomatisasi pengisian diskon = 0, gaji = 0, tanggal = CURRENT_DATE, tanggal_transaksi = CURRENT_TIMESTAMP, dan lunas = FALSE.

⚙️ Fitur Otomasi & Pelaporan (PL/pgSQL)
1. Function & Trigger Otomatis
Nama Trigger
Nama Function
Event
Fungsi Otomatisasi
trg_hitung_umur_pegawai
hitung_umur_pegawai()
BEFORE INSERT / UPDATE pada pegawai
Menghitung umur pegawai secara otomatis berdasarkan selisih CURRENT_DATE dan tanggal_lahir.
trg_pembayaran_jual
hitung_pembayaran_jual()
BEFORE INSERT / UPDATE pada transaksi_jual
Menghitung selisih kurang_bayar dan mengubah status lunas menjadi TRUE jika tagihan terpenuhi.
trg_pembayaran_sewa
hitung_pembayaran_sewa()
BEFORE INSERT / UPDATE pada transaksi_sewa
Menghitung sisa pembayaran sewa (kurang_bayar) dan memperbarui status lunas otomatis.
trg_update_status_terjual
update_status_terjual()
AFTER INSERT pada transaksi_jual
Mengubah status_ketersediaan unit di tabel property menjadi 'Terjual'.
trg_update_status_disewa
update_status_disewa()
AFTER INSERT pada transaksi_sewa
Mengubah status_ketersediaan unit di tabel property menjadi 'Disewa'.

2. Analytical Views
laporan_performa_agen: Rekapitulasi kinerja penjualan setiap agen (total transaksi, total omzet, penjualan tertinggi/terendah, dan rata-rata penjualan) menggunakan fungsi agregasi, GROUP BY, dan HAVING dengan subquery.
klien_sewa_beli: Menampilkan daftar klien loyal yang pernah melakukan transaksi pembelian sekaligus penyewaan properti menggunakan operasi himpunan INTERSECT.
properti_menganggur: Menampilkan daftar properti yang belum pernah terjual maupun tersewa sama sekali menggunakan kombinasi operasi himpunan EXCEPT dan UNION.

🚀 Cara Instalasi & Menjalankan Database
Prasyarat
PostgreSQL (disarankan versi 15 ke atas)
Client SQL seperti pgAdmin 4, DBeaver, atau terminal psql
Opsi 1: Restore Menggunakan File Backup (backup/sistem_propertiDB.sql)
Buat database baru bernama sistem_property_db melalui terminal atau pgAdmin:
createdb -U postgres sistem_property_db
Lakukan restore menggunakan pg_restore (karena file dump menggunakan format arsip PostgreSQL):
pg_restore -U postgres -d sistem_property_db backup/sistem_propertiDB.sql
Opsi 2: Eksekusi Skrip SQL Secara Berurutan (sql/)
Buat database sistem_property_db terlebih dahulu:
CREATE DATABASE sistem_property_db;
Hubungkan ke database sistem_property_db, lalu jalankan skrip pada folder sql/ secara berurutan:
sql/01_ddl_tables_and_enums.sql
sql/02_functions_and_triggers.sql
sql/03_dml_seed_records.sql
sql/04_views.sql (Atau jalankan langsung sql/full_script.sql untuk mengeksekusi seluruh skema dan data sekaligus).

📌 Ruang Lingkup & Pengembangan Selanjutnya
Batasan Sistem Saat Ini
Tidak mencakup penjadwalan jatuh tempo cicilan bulanan atau perhitungan bunga pinjaman untuk pembayaran metode Kredit.
Tidak mencakup perhitungan payroll bulanan, bonus komisi agen, maupun pemotongan gaji berdasarkan absensi.
Belum mencakup manajemen pemeliharaan fisik bangunan (maintenance) dan pencatatan keluhan penyewa.
Rencana Pengembangan (Future Works)
Penambahan modul jadwal angsuran kredit dan kalkulasi bunga otomatis.
Integrasi sistem penggajian (payroll) dan perhitungan komisi agen berbasis performa.
Penambahan entitas manajemen perawatan bangunan (maintenance) dan tiket komplain klien.
Integrasi payment gateway untuk verifikasi pembayaran real-time.

👥 Tim Penyusun (Kelompok 1)
Proyek ini disusun untuk memenuhi tugas akhir mata kuliah Workshop Basis Data — Politeknik Elektronika Negeri Surabaya (PENS):
Nama Mahasiswa
NRP
Kelas
Ananda Briant Pangestu
3225600091
1 D4 Teknik Komputer D
Muhammad Fatkhur R.
3225600110
1 D4 Teknik Komputer D
Lukman Aprianto Santoso
3225600114
1 D4 Teknik Komputer D

Dosen Pengampu: Dwi Kurnia Basuki, S.Si., M.Kom.
