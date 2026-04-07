# Modul 2: Struktur Direktori dan Konsep Arsitektur MVC

## Deskripsi Modul

Setelah mahasiswa berhasil menginstal dan menjalankan proyek Laravel pada Modul 1, langkah berikutnya adalah memahami bagaimana Laravel disusun dari dalam. Banyak mahasiswa bisa menjalankan proyek, tetapi belum memahami file mana yang harus dibuka saat ingin menambah halaman, membuat logika bisnis, atau menampilkan data dari database.

Modul ini membahas dua hal mendasar:

- struktur direktori Laravel
- konsep arsitektur `MVC` atau `Model-View-Controller`

Dengan memahami keduanya sejak awal, mahasiswa akan lebih mudah mengikuti modul-modul berikutnya, terutama saat mulai membuat dashboard, CRUD alat, CRUD tanaman, laporan, autentikasi, dan API IoT.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan konsep arsitektur MVC pada Laravel
- memahami alur request dari browser sampai response
- membedakan fungsi model, view, dan controller
- mengenali direktori penting dalam proyek Laravel
- memetakan kebutuhan studi kasus greenhouse ke struktur proyek Laravel
- menentukan file mana yang perlu diubah saat membuat fitur baru

## Keterkaitan dengan Studi Kasus

Pada proyek **greenhouse monitoring**, mahasiswa nantinya akan membangun fitur:

- dashboard monitoring
- manajemen user
- manajemen alat atau sensor
- manajemen tanaman
- manajemen laporan
- API penerimaan data sensor dari bridge MQTT

Setiap fitur tersebut tidak ditulis dalam satu file saja. Laravel memecah tanggung jawab ke beberapa bagian agar kode:

- lebih rapi
- lebih mudah dirawat
- lebih mudah dikembangkan
- lebih mudah diuji

Karena itu pemahaman terhadap MVC sangat penting.

## Konsep Dasar MVC

MVC adalah pola arsitektur yang membagi aplikasi menjadi tiga bagian utama:

- `Model`
- `View`
- `Controller`

Tujuan utamanya adalah memisahkan logika data, logika proses, dan tampilan antarmuka.

## 1. Model

Model adalah representasi data dan aturan bisnis yang terkait dengan database. Dalam Laravel, model biasanya berada di folder:

```text
app/Models
```

Fungsi model antara lain:

- berinteraksi dengan tabel database
- melakukan operasi CRUD
- mendefinisikan relasi antar tabel
- melakukan casting data
- menyediakan query yang reusable

### Contoh model pada studi kasus greenhouse

- `User`
- `Device`
- `Plant`
- `SensorReading`
- `Report`

### Contoh peran model dalam aplikasi

- `Device` menyimpan data alat seperti kode alat, lokasi, topic MQTT, dan status
- `Plant` menyimpan data tanaman seperti nama tanaman, varietas, dan blok greenhouse
- `SensorReading` menyimpan histori suhu, kelembapan, dan kelembapan tanah
- `Report` menyimpan laporan monitoring berkala

## 2. View

View adalah bagian yang bertugas menampilkan data ke pengguna. Dalam Laravel, view umumnya ditulis dengan Blade dan disimpan pada folder:

```text
resources/views
```

Fungsi view:

- menampilkan data ke browser
- membentuk halaman HTML
- menampilkan tabel, form, dashboard, dan navigasi
- mengatur antarmuka agar nyaman dipakai pengguna

### Contoh view pada studi kasus greenhouse

- halaman dashboard
- halaman daftar alat
- halaman daftar tanaman
- form tambah tanaman
- halaman laporan monitoring
- halaman login

View tidak seharusnya menyimpan logika bisnis yang kompleks. Tugas view adalah menerima data yang sudah disiapkan controller lalu menampilkannya.

## 3. Controller

Controller adalah penghubung antara request pengguna, model, dan view. Controller biasanya berada di:

```text
app/Http/Controllers
```

Fungsi controller:

- menerima request dari route
- memvalidasi input
- memanggil model
- mengolah data
- mengirim data ke view atau JSON response

### Contoh controller pada studi kasus greenhouse

- `DashboardController`
- `UserController`
- `DeviceController`
- `PlantController`
- `ReportController`
- `Api/IotReadingController`

## Perintah Artisan yang Berkaitan dengan MVC

Walaupun modul ini masih bersifat konseptual, mahasiswa perlu mulai mengenal perintah Artisan yang akan sering dipakai untuk membangun struktur MVC.

Contoh:

```bash
php artisan make:controller DashboardController
php artisan make:controller DeviceController
php artisan make:model Device
php artisan make:model Plant
```

Penjelasan:

- `make:controller` digunakan untuk membuat controller baru pada folder `app/Http/Controllers`
- `make:model` digunakan untuk membuat model baru pada folder `app/Models`

Perintah-perintah ini akan dipakai langsung pada modul berikutnya saat mahasiswa mulai membangun fitur dashboard, alat, dan tanaman.

## Mengapa MVC Penting

Tanpa pemisahan MVC, seluruh kode bisa bercampur dalam satu file. Akibatnya:

- sulit dibaca
- sulit diperbaiki
- sulit dikembangkan
- rawan error saat aplikasi membesar

Dengan MVC:

- data dikelola di model
- tampilan dikelola di view
- logika request dikelola di controller

## Analogi Sederhana MVC

Agar lebih mudah dipahami:

- `Model` adalah gudang data
- `View` adalah etalase yang dilihat pengguna
- `Controller` adalah petugas yang mengambil barang dari gudang lalu menaruhnya ke etalase sesuai permintaan

Pada sistem greenhouse:

- database menyimpan data alat dan sensor
- controller membaca data tersebut
- view menampilkan dashboard ke admin, operator, atau viewer

## Alur Request pada Laravel

Saat pengguna membuka halaman dashboard, prosesnya secara umum adalah:

```text
Browser -> Route -> Controller -> Model -> Controller -> View -> Browser
```

Penjelasan langkah demi langkah:

1. pengguna membuka URL di browser
2. Laravel mencocokkan URL tersebut di file route
3. route mengarahkan request ke controller tertentu
4. controller memanggil model untuk mengambil atau menyimpan data
5. controller mengirim data ke view
6. view menghasilkan HTML
7. browser menampilkan hasil ke pengguna

## Contoh Alur Request Dashboard

Misalnya pengguna membuka:

```text
/dashboard
```

Maka alurnya bisa seperti ini:

1. route `/dashboard` didefinisikan di `routes/web.php`
2. route tersebut memanggil `DashboardController@index`
3. controller mengambil jumlah alat, jumlah tanaman, dan rata-rata suhu dari model
4. controller mengirim data ke `resources/views/dashboard/index.blade.php`
5. Blade merender HTML
6. halaman dashboard tampil di browser

## Contoh Alur Request API IoT

Untuk integrasi IoT, alurnya sedikit berbeda karena output bukan HTML melainkan JSON.

```text
Sensor -> MQTT Broker -> Bridge -> Laravel API -> Database -> JSON Response
```

Detail alurnya:

1. perangkat sensor mengirim data ke broker MQTT
2. service bridge membaca pesan dari topik tertentu
3. bridge mengirim HTTP request ke endpoint Laravel
4. route di `routes/api.php` menerima request
5. controller API memvalidasi payload
6. model `SensorReading` menyimpan data ke database
7. Laravel mengembalikan JSON response sebagai tanda sukses

## Perbedaan Web Route dan API Route

Laravel memisahkan route web dan API agar lebih jelas.

### `routes/web.php`

Digunakan untuk:

- halaman dashboard
- halaman CRUD user
- halaman CRUD alat
- halaman CRUD tanaman
- halaman laporan
- halaman login setelah integrasi autentikasi

Ciri khas:

- biasanya menghasilkan HTML
- memakai session dan cookie
- cocok untuk aplikasi berbasis browser

### `routes/api.php`

Digunakan untuk:

- endpoint penerimaan data sensor
- endpoint data ringkasan dashboard
- endpoint histori pembacaan sensor

Ciri khas:

- biasanya menghasilkan JSON
- tidak berfokus pada tampilan Blade
- cocok untuk integrasi perangkat atau aplikasi lain

## Struktur Direktori Penting Laravel

Laravel memiliki banyak folder, tetapi tidak semuanya perlu dipelajari sekaligus. Pada tahap awal, mahasiswa cukup fokus ke direktori inti berikut.

## 1. Folder `app`

Folder ini menampung kode inti aplikasi.

Subfolder penting:

- `app/Models`: model Eloquent
- `app/Http/Controllers`: controller
- `app/Http/Middleware`: middleware
- `app/Providers`: service provider

### Relevansi pada studi kasus

- model `Device`, `Plant`, dan `SensorReading` akan berada di sini
- controller dashboard dan CRUD juga berada di sini
- middleware role untuk admin, operator, dan viewer juga akan dibuat di sini

## 2. Folder `routes`

Folder ini menyimpan definisi route aplikasi.

File yang paling sering dipakai:

- `routes/web.php`
- `routes/api.php`

### Relevansi pada studi kasus

- route dashboard dan CRUD ditulis di `web.php`
- route ingest data IoT ditulis di `api.php`

## 3. Folder `resources`

Folder ini menyimpan resource yang akan dipakai di sisi tampilan.

Subfolder penting:

- `resources/views`
- `resources/css`
- `resources/js`

### Relevansi pada studi kasus

- Blade dashboard ada di `resources/views/dashboard`
- Blade alat ada di `resources/views/devices`
- Blade tanaman ada di `resources/views/plants`
- CSS dashboard dan UI dasar dapat dikembangkan dari folder ini

## 4. Folder `database`

Folder ini menangani hal-hal yang berkaitan dengan basis data.

Subfolder penting:

- `database/migrations`
- `database/seeders`
- `database/factories`

### Relevansi pada studi kasus

- migration akan membuat tabel `devices`, `plants`, `sensor_readings`, dan `reports`
- seeder akan mengisi akun admin, operator, viewer, dan data dummy
- factory dipakai untuk membuat data sensor palsu dalam jumlah banyak

## 5. Folder `config`

Folder ini menyimpan konfigurasi aplikasi seperti:

- database
- mail
- queue
- cache
- filesystem

Walaupun belum sering diubah pada tahap awal, mahasiswa perlu tahu bahwa pengaturan inti Laravel banyak berada di sini.

## 6. Folder `public`

Folder ini adalah pintu masuk aplikasi web. File pentingnya adalah:

```text
public/index.php
```

Folder ini juga menjadi tempat asset publik jika aplikasi diakses lewat browser.

Pada saat deployment, folder ini sangat penting karena menjadi web root aplikasi.

## 7. Folder `storage`

Folder ini digunakan untuk:

- log aplikasi
- cache
- session
- file upload

Pada studi kasus greenhouse, folder ini nanti dipakai untuk:

- gambar tanaman
- lampiran laporan monitoring

## 8. Folder `bootstrap`

Folder ini menangani proses bootstrap aplikasi. Pada Laravel modern, salah satu file penting yang perlu dikenali adalah:

```text
bootstrap/app.php
```

File ini penting terutama saat mendaftarkan middleware dan konfigurasi inti aplikasi.

## 9. Folder `vendor`

Folder ini berisi seluruh dependency dari Composer, termasuk framework Laravel itu sendiri.

Catatan penting:

- folder ini jangan diedit manual
- folder ini biasanya tidak dipelajari isinya pada tingkat dasar
- jika hilang, bisa dibuat ulang dengan `composer install`

## 10. File `.env`

File `.env` menyimpan konfigurasi environment, misalnya:

- nama aplikasi
- mode debug
- koneksi database
- mail configuration
- secret key

Pada proyek greenhouse, `.env` juga nanti dapat dipakai untuk menyimpan:

- kredensial database
- secret key API IoT
- konfigurasi broker atau bridge bila diperlukan

## Ringkasan Direktori Inti

| Direktori atau File | Fungsi Utama | Contoh pada Proyek Greenhouse |
| --- | --- | --- |
| `app/Models` | model data | `Device`, `Plant`, `SensorReading` |
| `app/Http/Controllers` | logika request | `DashboardController`, `DeviceController` |
| `app/Http/Middleware` | filter request | middleware `role` |
| `routes/web.php` | route halaman web | dashboard, user, alat, tanaman |
| `routes/api.php` | route API | endpoint pembacaan sensor |
| `resources/views` | Blade view | dashboard, form, tabel |
| `database/migrations` | struktur tabel | tabel alat, tanaman, sensor |
| `database/seeders` | data awal | akun admin dan data dummy |
| `storage` | file upload dan log | foto tanaman, lampiran laporan |
| `.env` | konfigurasi | database dan secret API |

## Pemetaan Studi Kasus ke Struktur Laravel

Supaya mahasiswa melihat keterhubungan antar folder, berikut contoh pemetaan fitur ke file Laravel.

### 1. Fitur Dashboard

- route: `routes/web.php`
- controller: `DashboardController`
- model: `Device`, `SensorReading`, `Plant`
- view: `resources/views/dashboard/index.blade.php`

### 2. Fitur Manajemen User

- route: `routes/web.php`
- controller: `UserController`
- model: `User`
- view: `resources/views/users/index.blade.php`
- middleware: `auth`, `role:admin`

### 3. Fitur Manajemen Alat

- route: `routes/web.php`
- controller: `DeviceController`
- model: `Device`
- view: `resources/views/devices/index.blade.php`

### 4. Fitur Manajemen Tanaman

- route: `routes/web.php`
- controller: `PlantController`
- model: `Plant`
- view: `resources/views/plants/index.blade.php`

### 5. Fitur Laporan

- route: `routes/web.php`
- controller: `ReportController`
- model: `Report`
- view: `resources/views/reports/index.blade.php`

### 6. Fitur API IoT

- route: `routes/api.php`
- controller: `Api/IotReadingController`
- model: `Device`, `SensorReading`
- output: JSON, bukan Blade

## Contoh Kode Sederhana untuk Memahami Keterhubungan MVC

### Route

```php
use App\Http\Controllers\DashboardController;
use Illuminate\Support\Facades\Route;

Route::get('/dashboard', [DashboardController::class, 'index']);
```

### Controller

```php
namespace App\Http\Controllers;

use App\Models\Device;

class DashboardController extends Controller
{
    public function index()
    {
        $totalDevices = Device::count();

        return view('dashboard.index', compact('totalDevices'));
    }
}
```

### View

```blade
<h1>Dashboard Greenhouse</h1>
<p>Total alat: {{ $totalDevices }}</p>
```

### Penjelasan

- route menerima URL `/dashboard`
- controller mengambil data dari model
- view menampilkan data ke browser

Contoh ini sangat sederhana, tetapi sudah menunjukkan inti kerja MVC.

## Kesalahan Umum Mahasiswa Pemula

Beberapa kesalahan yang sering terjadi pada tahap awal:

- menulis query database langsung di file Blade
- menaruh HTML panjang di route closure
- tidak bisa membedakan fungsi route dan controller
- bingung antara `resources/views` dan `public`
- mengedit file di `vendor`

Penekanan penting:

- query sebaiknya dilakukan di model atau controller
- tampilan sebaiknya ditulis di Blade
- asset publik bukan tempat menyimpan logika aplikasi

## Praktik Observasi Proyek

Pada sesi ini, dosen dapat meminta mahasiswa membuka proyek Laravel dan melakukan pengamatan terhadap folder berikut:

1. buka folder `app`
2. buka `app/Models/User.php`
3. buka folder `routes`
4. buka file `routes/web.php`
5. buka folder `resources/views`
6. buka folder `database/migrations`
7. buka file `.env`

Mahasiswa tidak perlu mengedit semuanya pada tahap ini. Fokusnya adalah mengenali posisi dan fungsi masing-masing.

## Aktivitas Praktik yang Disarankan

Urutan aktivitas pertemuan dapat dibuat seperti ini:

1. review hasil instalasi Modul 1
2. penjelasan konsep MVC
3. demonstrasi alur request dari URL ke Blade
4. eksplorasi folder inti Laravel
5. diskusi pemetaan studi kasus greenhouse ke MVC
6. latihan identifikasi file yang akan diubah saat membuat fitur baru

## Mini Latihan Analisis

Jawab pertanyaan berikut:

1. jika ingin membuat halaman daftar alat, file apa saja yang kemungkinan harus dibuat atau diubah
2. jika ingin membuat endpoint API data sensor, file apa saja yang terlibat
3. jika ingin mengubah tampilan dashboard, folder mana yang paling sering disentuh
4. jika ingin menambah kolom baru pada tabel tanaman, folder mana yang harus dibuka

## Hasil Akhir Modul

Mahasiswa memahami letak file penting dalam proyek Laravel, memahami cara kerja MVC, dan mampu memetakan fitur greenhouse monitoring ke struktur direktori Laravel secara benar.

## Tugas Praktik

1. Jelaskan dengan bahasa sendiri apa itu `Model`, `View`, dan `Controller`.
2. Sebutkan perbedaan `routes/web.php` dan `routes/api.php`.
3. Buat tabel sederhana yang memetakan fitur `dashboard`, `alat`, dan `tanaman` ke `route`, `controller`, `model`, dan `view`.
4. Tulis ringkasan alur request halaman dashboard dalam 5 sampai 8 langkah.
5. Identifikasi 5 folder atau file Laravel yang menurut Anda paling penting untuk pengembangan proyek greenhouse.
