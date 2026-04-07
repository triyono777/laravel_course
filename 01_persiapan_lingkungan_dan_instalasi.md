# Modul 1: Persiapan Lingkungan dan Instalasi

## Deskripsi Modul

Modul pertama ini berfungsi sebagai fondasi seluruh praktikum. Fokus utamanya bukan hanya menginstal Laravel, tetapi juga memastikan setiap mahasiswa memiliki lingkungan kerja yang seragam, memahami fungsi tool yang dipakai, dan mampu menjalankan proyek awal sebelum masuk ke pembuatan fitur greenhouse monitoring.

Pada akhir modul, mahasiswa diharapkan sudah memiliki proyek Laravel bernama `greenhouse-monitoring` yang dapat dijalankan di komputer masing-masing.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan kebutuhan software untuk pengembangan Laravel
- menginstal PHP, Composer, Node.js, database server, dan editor kode
- membuat proyek Laravel baru
- menjalankan development server Laravel
- menjalankan Vite untuk asset frontend
- mengenali struktur awal proyek Laravel
- memahami target akhir aplikasi greenhouse monitoring

## Prasyarat

Sebelum mulai, mahasiswa disarankan sudah memahami:

- dasar penggunaan terminal atau command prompt
- konsep folder dan file pada sistem operasi
- dasar bahasa PHP
- konsep web server lokal dan database secara umum

Jika ada mahasiswa yang belum terbiasa dengan terminal, dosen dapat mengawali pertemuan dengan demonstrasi singkat perintah dasar seperti:

```bash
cd
dir
ls
mkdir
```

## Gambaran Studi Kasus

Seluruh modul akan menggunakan satu studi kasus yang sama, yaitu **web monitoring greenhouse**. Sistem ini dirancang untuk membantu pengelolaan rumah kaca pertanian dengan fitur:

- dashboard kondisi greenhouse
- manajemen user berbasis role
- manajemen alat atau sensor
- manajemen tanaman
- manajemen laporan monitoring
- API untuk menerima data sensor dari perangkat IoT
- integrasi ke MQTT melalui bridge atau service perantara

## Peran Pengguna dalam Sistem

Supaya mahasiswa sejak awal memahami konteks proyek, berikut role utama yang nanti dibangun:

- `admin`: mengelola semua data termasuk user
- `operator`: mengelola alat, tanaman, dan laporan
- `viewer`: hanya melihat dashboard dan laporan

## Perangkat Lunak yang Dibutuhkan

### 1. PHP

Laravel 11 membutuhkan PHP modern. Gunakan minimal:

- `PHP 8.2`

Fungsi PHP pada proyek ini adalah menjalankan logika backend, route, controller, model, validasi, dan API.

### 2. Composer

Composer adalah dependency manager untuk PHP. Fungsinya:

- mengunduh framework Laravel
- memasang package tambahan
- mengelola autoload class

Tanpa Composer, instalasi Laravel tidak dapat dilakukan dengan benar.

### 3. Node.js dan npm

Laravel modern menggunakan Vite untuk memproses asset frontend seperti:

- CSS
- JavaScript
- komponen interaktif

Karena itu Node.js dan npm tetap diperlukan walaupun fokus utama proyek ini adalah backend.

### 4. Database

Mahasiswa dapat menggunakan salah satu dari dua opsi berikut:

- `MySQL`: cocok untuk simulasi aplikasi nyata dan umum dipakai pada hosting
- `SQLite`: lebih ringan dan cepat untuk latihan awal

Untuk mata kuliah ini, MySQL lebih disarankan karena fitur deployment nanti akan lebih relevan.

### 5. Web Server Lokal

Pilih salah satu:

- `Laragon`
- `XAMPP`
- `Laravel Herd`

Fungsinya adalah menyediakan lingkungan lokal yang memudahkan pengembangan, terutama untuk PHP dan database.

### 6. Code Editor

Gunakan:

- Visual Studio Code

Ekstensi yang direkomendasikan:

- PHP Intelephense
- Laravel Blade Snippets
- DotENV
- Error Lens
- Prettier

### 7. Git

Git digunakan untuk:

- menyimpan riwayat perubahan
- backup perkembangan proyek
- mendukung kolaborasi

## Rekomendasi Setup Berdasarkan Sistem Operasi

### Windows

Pilihan termudah biasanya:

- Laragon
- XAMPP

Laragon sering lebih nyaman untuk Laravel karena struktur folder dan pengaturan PATH cenderung lebih sederhana.

### macOS

Pilihan yang umum:

- Laravel Herd
- Homebrew + PHP + Composer + MySQL

### Linux

Biasanya instalasi dilakukan melalui package manager distribusi masing-masing, misalnya:

- `apt`
- `dnf`
- `pacman`

## Langkah Praktik

### 1. Memeriksa apakah tool sudah terpasang

Jalankan perintah berikut pada terminal:

```bash
php -v
composer -V
node -v
npm -v
git --version
```

Jika semua berhasil, terminal akan menampilkan versi masing-masing tool. Contoh:

```text
PHP 8.2.x
Composer version 2.x
v20.x.x
10.x.x
git version 2.x.x
```

Jika ada perintah yang tidak dikenali, berarti tool tersebut belum terpasang atau PATH belum benar.

### 2. Menentukan lokasi kerja proyek

Sebelum membuat proyek, tentukan folder penyimpanan. Contoh:

```bash
cd ~/Documents/project-web
```

Atau pada Windows:

```text
C:\laragon\www
```

Sebaiknya satu kelas menggunakan struktur yang seragam agar dosen lebih mudah membimbing.

### 3. Membuat proyek Laravel baru

Jalankan:
untuk laravel terbaru
```bash
composer create-project laravel/laravel greenhouse-monitoring
```
untuk php versi 8.2  dapat menggunakan laravel 12
```bash
composer create-project laravel/laravel:^12.0 nama-projek-anda
```
Penjelasan perintah:

- `composer create-project`: membuat proyek baru dari package
- `laravel/laravel`: package starter resmi Laravel
- `greenhouse-monitoring`: nama folder proyek

Jika proses berhasil, masuk ke folder proyek:

```bash
cd greenhouse-monitoring
```

### 4. Mengenali isi folder setelah instalasi

Setelah instalasi selesai, Laravel akan membuat banyak file dan folder. Beberapa yang paling penting adalah:

- `app/`: berisi kode inti aplikasi seperti model dan controller
- `bootstrap/`: proses bootstrap aplikasi
- `config/`: file konfigurasi
- `database/`: migration, seeder, dan factory
- `public/`: file yang diakses browser
- `resources/`: view Blade, CSS, dan JS
- `routes/`: routing web dan API
- `storage/`: file log, cache, dan upload
- `vendor/`: dependency bawaan dari Composer
- `.env`: konfigurasi environment aplikasi

## Menyiapkan File Environment

### 1. Membuat file `.env`

Biasanya Laravel sudah menyertakan `.env`, tetapi jika belum ada maka jalankan:

```bash
cp .env.example .env
```

Jika menggunakan Windows dan perintah `cp` tidak tersedia, salin file secara manual.

### 2. Membuat application key

Jalankan:

```bash
php artisan key:generate
```

Perintah ini membuat kunci enkripsi aplikasi yang digunakan Laravel untuk:

- session
- cookie
- enkripsi data tertentu

Jika key tidak dibuat, beberapa fitur Laravel tidak akan berjalan normal.

## Konfigurasi Database Awal

Pada modul pertama, mahasiswa boleh memilih `SQLite` agar lebih cepat, atau `MySQL` jika ingin langsung menyesuaikan lingkungan produksi.

### Opsi A: SQLite

Buat file database kosong:

```bash
touch database/database.sqlite
```

Lalu ubah `.env`:

```env
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/ke/database/database.sqlite
```

Jika memakai path absolut dirasa terlalu rumit untuk awal pertemuan, dosen dapat menunda detail ini sampai modul database.

### Opsi B: MySQL

Buat database baru, misalnya:

```text
greenhouse_monitoring
```

Lalu ubah `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=greenhouse_monitoring
DB_USERNAME=root
DB_PASSWORD=
```

Catatan:

- `DB_USERNAME` dan `DB_PASSWORD` menyesuaikan konfigurasi lokal
- pada Laragon dan XAMPP, user default sering kali `root`

## Menjalankan Aplikasi Laravel

### 1. Menjalankan development server

Gunakan:

```bash
php artisan serve
```

Jika berhasil, akan muncul informasi seperti:

```text
INFO  Server running on [http://127.0.0.1:8000].
```

Lalu buka browser:

```text
http://127.0.0.1:8000
```

Jika halaman welcome Laravel muncul, berarti aplikasi sudah berjalan.

### 2. Menjalankan asset frontend

Laravel modern memerlukan Vite saat pengembangan frontend. Jalankan pada terminal lain:

```bash
npm install
npm run dev
```

Penjelasan:

- `npm install`: mengunduh dependency frontend
- `npm run dev`: menjalankan Vite development server

Jika Vite berhasil berjalan, akan muncul informasi localhost untuk asset development.

## Verifikasi Instalasi

Mahasiswa dinyatakan berhasil menyelesaikan instalasi jika:

- folder proyek `greenhouse-monitoring` berhasil dibuat
- file `.env` tersedia
- `php artisan key:generate` berhasil dijalankan
- `php artisan serve` berjalan tanpa error
- browser menampilkan halaman awal Laravel
- `npm run dev` berjalan tanpa error

## Perintah Dasar Laravel yang Perlu Dikenal Sejak Awal

Walaupun belum semua dipakai hari ini, mahasiswa sebaiknya mulai mengenal beberapa perintah artisan berikut:

```bash
php artisan serve
php artisan make:controller NamaController
php artisan make:model NamaModel -m
php artisan migrate
php artisan route:list
```

Penjelasan singkat:

- `serve`: menjalankan server lokal
- `make:controller`: membuat controller
- `make:model -m`: membuat model sekaligus migration
- `migrate`: menjalankan migration database
- `route:list`: melihat daftar route aplikasi

## Struktur Awal yang Perlu Diperhatikan

Pada pertemuan pertama, dosen dapat menunjukkan file berikut:

- `routes/web.php`: route halaman web
- `routes/api.php`: route API
- `app/Http/Controllers`: logika request
- `app/Models`: model Eloquent
- `resources/views`: file Blade
- `resources/css/app.css`: stylesheet utama
- `database/migrations`: struktur tabel
- `.env`: konfigurasi aplikasi

## Target Fitur Akhir Proyek

Agar mahasiswa memahami arah pembelajaran sejak awal, proyek akhir yang akan dibangun minimal memiliki:

- dashboard suhu, kelembapan, kelembapan tanah, dan intensitas cahaya
- CRUD user
- CRUD alat atau sensor
- CRUD tanaman
- CRUD laporan monitoring
- login dan RBAC
- endpoint API untuk menerima data sensor
- integrasi IoT melalui MQTT bridge

## Arsitektur Besar yang Akan Dicapai

Secara umum, sistem akhir akan mengikuti alur:

```text
User/Admin -> Laravel Web App -> Database
Perangkat IoT -> MQTT Broker -> Bridge -> Laravel API -> Database -> Dashboard
```

Modul pertama belum masuk ke implementasi fitur tersebut, tetapi mahasiswa perlu memahami bahwa instalasi hari ini adalah landasan untuk semua modul berikutnya.

## Troubleshooting Umum

### 1. `php` tidak dikenali

Penyebab:

- PHP belum terinstal
- PATH belum diarahkan ke folder PHP

Solusi:

- pastikan instalasi PHP berhasil
- tambahkan folder PHP ke environment PATH
- restart terminal setelah mengubah PATH

### 2. `composer` tidak dikenali

Penyebab:

- Composer belum terpasang
- PATH Composer belum aktif

Solusi:

- instal Composer dari situs resmi
- restart terminal

### 3. `npm install` gagal

Penyebab umum:

- Node.js belum terpasang
- koneksi internet bermasalah
- cache npm bermasalah

Solusi:

- cek `node -v` dan `npm -v`
- ulangi `npm install`

### 4. Port `8000` sudah dipakai

Solusi:

```bash
php artisan serve --port=8080
```

Lalu akses:

```text
http://127.0.0.1:8080
```

### 5. Halaman kosong atau asset tidak muncul

Penyebab:

- `npm run dev` belum dijalankan
- cache browser

Solusi:

- jalankan Vite
- refresh browser

### 6. Gagal konek database

Penyebab:

- nama database salah
- username atau password salah
- service MySQL belum aktif

Solusi:

- cek `.env`
- pastikan MySQL sedang berjalan
- uji koneksi lewat phpMyAdmin, HeidiSQL, DBeaver, atau terminal

## Aktivitas Praktik yang Disarankan untuk Dosen

Urutan pembelajaran dalam 1 pertemuan dapat dibuat seperti ini:

1. penjelasan target proyek akhir
2. penjelasan software yang dibutuhkan
3. demonstrasi cek versi tool
4. demonstrasi instalasi proyek Laravel
5. praktik mandiri mahasiswa
6. verifikasi hasil instalasi
7. penjelasan struktur folder awal

## Hasil Akhir Modul

Mahasiswa memiliki proyek Laravel baru yang berjalan di komputer masing-masing, memahami fungsi tool yang dipakai, serta siap melanjutkan ke pembahasan struktur direktori dan konsep MVC pada modul berikutnya.

## Tugas Praktik

1. Install seluruh dependensi yang dibutuhkan.
2. Buat proyek `greenhouse-monitoring`.
3. Jalankan `php artisan serve`.
4. Jalankan `npm install` dan `npm run dev`.
5. Ambil tangkapan layar halaman welcome Laravel.
6. Catat versi PHP, Composer, Node.js, npm, dan Git yang digunakan.
7. Tuliskan ringkasan fungsi folder `app`, `routes`, `resources`, `database`, dan `public`.
