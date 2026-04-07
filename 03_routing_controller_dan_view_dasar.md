# Modul 3: Routing, Controller, dan View Dasar

## Deskripsi Modul

Setelah memahami struktur direktori Laravel dan konsep MVC pada Modul 2, mahasiswa perlu mulai menulis fitur pertama yang benar-benar bisa diakses lewat browser. Pada tahap ini, fokusnya belum pada database, melainkan pada bagaimana URL diarahkan ke controller dan bagaimana controller mengirim data ke view.

Modul ini sangat penting karena hampir seluruh fitur Laravel berawal dari pola kerja yang sama:

- route menerima URL
- controller memproses request
- view menampilkan hasil

Dengan memahami modul ini, mahasiswa akan siap membangun halaman dashboard, daftar alat, daftar tanaman, dan halaman lain pada studi kasus greenhouse monitoring.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- membuat route dasar pada Laravel
- memahami jenis penulisan route
- membuat controller dengan Artisan
- menghubungkan route ke controller
- mengirim data dari controller ke view
- menampilkan data dinamis pada Blade
- menggunakan route parameter dan named route
- menyiapkan halaman awal proyek greenhouse monitoring tanpa database

## Keterkaitan dengan Studi Kasus

Pada tahap awal proyek greenhouse monitoring, kita akan membuat tiga halaman sederhana:

- halaman beranda proyek
- halaman dashboard monitoring sederhana
- halaman daftar alat dengan data dummy

Data masih ditulis manual di controller agar mahasiswa fokus memahami alur request terlebih dahulu. Pada modul berikutnya, data dummy ini akan diganti dengan data database.

## Konsep Dasar Routing

Routing adalah mekanisme Laravel untuk menentukan respon apa yang diberikan ketika pengguna mengakses URL tertentu.

Contoh sederhana:

```text
/dashboard
```

Ketika URL itu diakses, Laravel akan mencari definisi route yang sesuai. Route tersebut bisa:

- langsung mengembalikan teks
- langsung mengembalikan view
- memanggil controller
- menerima parameter
- memberi nama route

## Mengapa Routing Penting

Tanpa route, Laravel tidak tahu apa yang harus dilakukan saat browser meminta halaman tertentu.

Routing berfungsi sebagai:

- pintu masuk fitur
- penghubung URL ke controller
- dasar pengelompokan halaman aplikasi

Pada proyek greenhouse, hampir semua halaman web akan didefinisikan di `routes/web.php`.

## File Route yang Digunakan

Untuk modul ini, kita fokus pada file:

```text
routes/web.php
```

File ini digunakan untuk route berbasis browser yang menampilkan HTML.

## Bentuk Route Paling Sederhana

Contoh route yang langsung mengembalikan teks:

```php
use Illuminate\Support\Facades\Route;

Route::get('/halo', function () {
    return 'Halo, selamat datang di praktikum Laravel.';
});
```

Penjelasan:

- `Route::get()` berarti route ini merespon HTTP method `GET`
- `'/halo'` adalah URL yang diakses
- function di dalamnya adalah respon yang dikembalikan

Jika browser membuka:

```text
http://127.0.0.1:8000/halo
```

maka teks tersebut akan tampil.

## HTTP Method yang Perlu Dikenal

Walaupun modul ini fokus pada `GET`, mahasiswa perlu mengenal method dasar berikut:

- `GET`: mengambil atau menampilkan data
- `POST`: mengirim data baru
- `PUT` atau `PATCH`: memperbarui data
- `DELETE`: menghapus data

Untuk sekarang, halaman dashboard dan daftar alat cukup menggunakan `GET`.

## Bentuk Route yang Mengembalikan View

Jika ingin langsung menampilkan view tanpa controller:

```php
Route::get('/profil-aplikasi', function () {
    return view('profile');
});
```

Jika file yang dipanggil adalah:

```text
resources/views/profile.blade.php
```

maka Laravel akan menampilkannya saat URL `/profil-aplikasi` dibuka.

Cara ini boleh dipakai untuk halaman sangat sederhana, tetapi untuk proyek yang lebih terstruktur sebaiknya gunakan controller.

## Mengapa Menggunakan Controller

Jika logika aplikasi semakin banyak, menuliskannya langsung di route akan membuat kode sulit dikelola. Controller membantu memindahkan logika dari route ke file yang lebih terorganisir.

Keuntungan memakai controller:

- route menjadi lebih bersih
- logika bisa dikelompokkan per fitur
- lebih mudah dikembangkan
- lebih konsisten dengan pola MVC

## Target Halaman Modul Ini

Pada modul ini kita akan membuat:

1. halaman dashboard sederhana
2. halaman daftar alat
3. halaman detail alat menggunakan parameter route

## Langkah Praktik

## 1. Menyiapkan Route Awal

Buka `routes/web.php` lalu isi seperti berikut:

```php
use App\Http\Controllers\DashboardController;
use App\Http\Controllers\DeviceController;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return redirect()->route('dashboard');
});

Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
Route::get('/devices', [DeviceController::class, 'index'])->name('devices.index');
Route::get('/devices/{code}', [DeviceController::class, 'show'])->name('devices.show');
```

### Penjelasan route di atas

- `/` diarahkan ke dashboard
- `/dashboard` memanggil `DashboardController@index`
- `/devices` memanggil `DeviceController@index`
- `/devices/{code}` memanggil `DeviceController@show`

`{code}` adalah route parameter yang nilainya berubah sesuai URL.

## 2. Memahami Named Route

Perhatikan bagian berikut:

```php
->name('dashboard')
```

Ini berarti route diberi nama `dashboard`.

Keuntungan named route:

- lebih mudah dipanggil dari Blade
- lebih aman jika URL berubah
- memudahkan redirect

Contoh penggunaan di Blade:

```blade
<a href="{{ route('dashboard') }}">Dashboard</a>
```

Contoh redirect dari controller:

```php
return redirect()->route('devices.index');
```

## 3. Membuat Controller dengan Artisan

Jalankan perintah berikut:

```bash
php artisan make:controller DashboardController
php artisan make:controller DeviceController
```

Perintah ini akan membuat file:

- `app/Http/Controllers/DashboardController.php`
- `app/Http/Controllers/DeviceController.php`

## 4. Membuat DashboardController

Isi `app/Http/Controllers/DashboardController.php`:

```php
namespace App\Http\Controllers;

class DashboardController extends Controller
{
    public function index()
    {
        $summary = [
            'total_devices' => 8,
            'active_devices' => 6,
            'total_plants' => 124,
            'avg_temperature' => 28.4,
            'avg_humidity' => 76.2,
        ];

        $alerts = [
            'Suhu blok A berada di atas ambang normal.',
            'Satu perangkat sedang dalam status maintenance.',
        ];

        return view('dashboard.index', compact('summary', 'alerts'));
    }
}
```

### Penjelasan controller di atas

- method `index()` akan dipanggil saat URL `/dashboard` diakses
- data `summary` disiapkan di controller
- data `alerts` juga disiapkan di controller
- seluruh data dikirim ke view `dashboard.index`

## 5. Membuat DeviceController

Isi `app/Http/Controllers/DeviceController.php`:

```php
namespace App\Http\Controllers;

class DeviceController extends Controller
{
    private array $devices = [
        [
            'code' => 'GH-01',
            'name' => 'Sensor Utama',
            'type' => 'sensor',
            'location' => 'Blok A',
            'status' => 'active',
        ],
        [
            'code' => 'GH-02',
            'name' => 'Sensor Timur',
            'type' => 'sensor',
            'location' => 'Blok B',
            'status' => 'maintenance',
        ],
        [
            'code' => 'GH-03',
            'name' => 'Controller Irigasi',
            'type' => 'controller',
            'location' => 'Blok C',
            'status' => 'active',
        ],
    ];

    public function index()
    {
        $devices = $this->devices;

        return view('devices.index', compact('devices'));
    }

    public function show(string $code)
    {
        $device = collect($this->devices)->firstWhere('code', $code);

        abort_if(! $device, 404);

        return view('devices.show', compact('device'));
    }
}
```

### Penjelasan controller di atas

- data dummy disimpan di property `$devices`
- method `index()` menampilkan seluruh data alat
- method `show($code)` menampilkan detail alat berdasarkan kode pada URL
- `abort_if(! $device, 404)` akan menampilkan halaman not found jika alat tidak ditemukan

## 6. Membuat Struktur Folder View

Buat folder berikut:

```text
resources/views/dashboard
resources/views/devices
```

Lalu buat file:

- `resources/views/dashboard/index.blade.php`
- `resources/views/devices/index.blade.php`
- `resources/views/devices/show.blade.php`

## 7. Membuat View Dashboard

Isi `resources/views/dashboard/index.blade.php`:

```blade
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard Greenhouse</title>
</head>
<body>
    <h1>Dashboard Greenhouse</h1>

    <p>Total alat: {{ $summary['total_devices'] }}</p>
    <p>Alat aktif: {{ $summary['active_devices'] }}</p>
    <p>Total tanaman: {{ $summary['total_plants'] }}</p>
    <p>Rata-rata suhu: {{ $summary['avg_temperature'] }} C</p>
    <p>Rata-rata kelembapan: {{ $summary['avg_humidity'] }} %</p>

    <h2>Peringatan</h2>
    <ul>
        @foreach ($alerts as $alert)
            <li>{{ $alert }}</li>
        @endforeach
    </ul>

    <p><a href="{{ route('devices.index') }}">Lihat daftar alat</a></p>
</body>
</html>
```

### Konsep yang dipakai

- `{{ }}` untuk menampilkan variabel
- `@foreach` untuk melakukan perulangan
- `route()` untuk membuat link berbasis named route

## 8. Membuat View Daftar Alat

Isi `resources/views/devices/index.blade.php`:

```blade
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daftar Alat</title>
</head>
<body>
    <h1>Daftar Alat Greenhouse</h1>

    <p><a href="{{ route('dashboard') }}">Kembali ke dashboard</a></p>

    <table border="1" cellpadding="8" cellspacing="0">
        <thead>
            <tr>
                <th>Kode</th>
                <th>Nama</th>
                <th>Tipe</th>
                <th>Lokasi</th>
                <th>Status</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($devices as $device)
                <tr>
                    <td>{{ $device['code'] }}</td>
                    <td>{{ $device['name'] }}</td>
                    <td>{{ $device['type'] }}</td>
                    <td>{{ $device['location'] }}</td>
                    <td>{{ $device['status'] }}</td>
                    <td>
                        <a href="{{ route('devices.show', $device['code']) }}">Detail</a>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>
</body>
</html>
```

### Konsep yang dipakai

- tabel HTML untuk menampilkan data
- `@foreach` untuk looping data dummy
- route parameter melalui `route('devices.show', $device['code'])`

## 9. Membuat View Detail Alat

Isi `resources/views/devices/show.blade.php`:

```blade
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Detail Alat</title>
</head>
<body>
    <h1>Detail Alat {{ $device['code'] }}</h1>

    <ul>
        <li>Nama: {{ $device['name'] }}</li>
        <li>Tipe: {{ $device['type'] }}</li>
        <li>Lokasi: {{ $device['location'] }}</li>
        <li>Status: {{ $device['status'] }}</li>
    </ul>

    <p><a href="{{ route('devices.index') }}">Kembali ke daftar alat</a></p>
</body>
</html>
```

## 10. Menguji Hasil

Jalankan server Laravel:

```bash
php artisan serve
```

Lalu uji URL berikut:

```text
http://127.0.0.1:8000/
http://127.0.0.1:8000/dashboard
http://127.0.0.1:8000/devices
http://127.0.0.1:8000/devices/GH-01
```

Pastikan:

- dashboard dapat dibuka
- daftar alat muncul
- link detail mengarah ke halaman alat yang benar

## Cara Kerja Passing Data dari Controller ke View

Ada beberapa cara mengirim data dari controller ke view.

### Menggunakan `compact()`

```php
return view('dashboard.index', compact('summary', 'alerts'));
```

### Menggunakan array asosiatif

```php
return view('dashboard.index', [
    'summary' => $summary,
    'alerts' => $alerts,
]);
```

Keduanya benar. Pada praktikum ini, `compact()` cukup nyaman untuk variabel yang namanya sudah jelas.

## Route Parameter

Perhatikan route berikut:

```php
Route::get('/devices/{code}', [DeviceController::class, 'show'])->name('devices.show');
```

`{code}` akan menangkap bagian URL yang berubah-ubah.

Contoh:

- `/devices/GH-01`
- `/devices/GH-02`
- `/devices/GH-03`

Nilai tersebut akan dikirim ke parameter method controller:

```php
public function show(string $code)
```

## Kapan Memakai Closure, Kapan Memakai Controller

### Gunakan closure jika

- hanya untuk percobaan sangat sederhana
- hanya menampilkan teks singkat
- tidak ada logika bisnis yang berarti

### Gunakan controller jika

- halaman sudah menjadi bagian fitur aplikasi
- ada data yang perlu diproses
- route akan terus dikembangkan
- ingin menjaga struktur proyek tetap rapi

Untuk proyek greenhouse, mayoritas route sebaiknya memakai controller.

## Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi pada modul ini:

- lupa menambahkan `use` controller di `routes/web.php`
- salah nama view pada `return view()`
- folder view belum dibuat
- typo pada named route
- parameter route tidak sama dengan link yang dipanggil
- menaruh terlalu banyak logika langsung di file route

## Cara Debug Sederhana

Jika route terasa tidak berjalan sesuai harapan, lakukan pengecekan berikut:

1. jalankan `php artisan route:list`
2. pastikan route yang dibuat muncul
3. cek nama controller dan method
4. cek nama file Blade
5. cek pesan error di browser atau terminal

Contoh:

```bash
php artisan route:list
```

Perintah ini menampilkan seluruh daftar route yang terdaftar di aplikasi.

## Pengembangan Lanjutan dari Modul Ini

Setelah modul ini selesai, struktur yang sudah dibuat akan berkembang menjadi:

- route CRUD user
- route CRUD alat
- route CRUD tanaman
- route CRUD laporan
- route autentikasi
- route API untuk data sensor

Data dummy yang saat ini ditulis manual di controller akan diganti dengan data database pada modul migration dan Eloquent ORM.

## Aktivitas Praktik yang Disarankan

Urutan aktivitas pertemuan dapat dibuat seperti ini:

1. review konsep MVC singkat
2. penjelasan konsep route dan HTTP method
3. praktik membuat route sederhana
4. praktik membuat controller
5. praktik mengirim data ke view
6. praktik route parameter
7. pengujian URL di browser

## Hasil Akhir Modul

Mahasiswa berhasil membuat halaman dashboard, daftar alat, dan detail alat sederhana dengan memanfaatkan route, controller, view, named route, serta route parameter pada proyek greenhouse monitoring.

## Tugas Praktik

1. Tambahkan route `/plants` dan `/plants/{code}`.
2. Buat `PlantController` dengan data dummy minimal 3 tanaman.
3. Buat view `plants/index.blade.php` dan `plants/show.blade.php`.
4. Tambahkan link menu sederhana dari dashboard ke halaman tanaman.
5. Jalankan `php artisan route:list` lalu catat route mana saja yang sudah Anda buat.
