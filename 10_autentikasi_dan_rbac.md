# Modul 10: Autentikasi dan Role-Based Access Control (RBAC)

## Deskripsi Modul

Setelah aplikasi greenhouse monitoring memiliki database, model, relasi, form, dan upload file, langkah berikutnya adalah memastikan bahwa sistem hanya dapat diakses oleh pengguna yang sah dan setiap pengguna hanya bisa melakukan tindakan sesuai hak aksesnya.

Modul ini membahas dua konsep yang saling terkait:

- **autentikasi**: memastikan siapa pengguna yang masuk ke sistem
- **otorisasi berbasis role**: memastikan apa yang boleh dilakukan pengguna tersebut

Pada aplikasi greenhouse monitoring, pengaturan akses sangat penting karena tidak semua pengguna boleh mengelola seluruh data. Admin harus memiliki akses penuh, operator hanya mengelola operasional greenhouse, dan viewer hanya melihat informasi.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan perbedaan autentikasi dan otorisasi
- memasang autentikasi Laravel menggunakan starter kit
- menjalankan login, register, dan logout
- menambahkan dan menggunakan kolom `role`
- membuat middleware untuk pemeriksaan role
- melindungi route berdasarkan role
- mengelola role user melalui halaman manajemen user
- menguji hak akses `admin`, `operator`, dan `viewer`

## Keterkaitan dengan Studi Kasus

Pada sistem greenhouse monitoring:

- `admin` mengelola user, alat, tanaman, laporan, dan konfigurasi API
- `operator` mengelola alat, tanaman, serta laporan operasional
- `viewer` hanya melihat dashboard dan laporan

Dengan demikian, sistem harus mampu menjawab pertanyaan berikut:

- apakah user sudah login
- siapa role user yang login
- apakah route tertentu boleh diakses user tersebut

## Autentikasi vs Otorisasi

Mahasiswa sering mencampur dua istilah ini. Padahal fungsinya berbeda.

### Autentikasi

Autentikasi menjawab:

```text
Siapa Anda?
```

Contoh:

- login dengan email dan password
- logout
- session login

### Otorisasi

Otorisasi menjawab:

```text
Apa yang boleh Anda lakukan?
```

Contoh:

- hanya admin boleh mengelola user
- operator boleh mengelola tanaman
- viewer tidak boleh menghapus data

## Strategi Implementasi pada Modul Ini

Modul ini memakai pendekatan sederhana dan cocok untuk praktikum:

1. pasang autentikasi dengan Laravel Breeze
2. gunakan kolom `role` pada tabel `users`
3. buat middleware `EnsureUserHasRole`
4. kelompokkan route berdasarkan role

Pendekatan ini cukup kuat untuk aplikasi pembelajaran dan sangat cocok untuk studi kasus greenhouse monitoring.

## Menyiapkan Kolom `role`

Pastikan tabel `users` sudah memiliki kolom `role` dari Modul 5. Contoh nilainya:

- `admin`
- `operator`
- `viewer`

Pastikan juga model `User` mengizinkan field tersebut:

```php
protected $fillable = [
    'name',
    'email',
    'password',
    'role',
];
```

## Menyiapkan Akun Awal

Sebelum mulai menguji autentikasi dan RBAC, sebaiknya database sudah memiliki akun contoh dari seeder:

- `admin@greenhouse.test`
- `operator@greenhouse.test`
- `viewer@greenhouse.test`

Dengan akun ini, mahasiswa dapat langsung menguji perbedaan akses.

## Instalasi Laravel Breeze

Laravel Breeze adalah starter kit ringan yang menyediakan:

- halaman login
- halaman register
- logout
- proteksi route dasar
- tampilan autentikasi berbasis Blade

Jalankan:

```bash
composer require laravel/breeze --dev
php artisan breeze:install
npm install
npm run dev
php artisan migrate
```

Jika installer meminta pilihan stack, gunakan stack Blade agar tetap konsisten dengan modul-modul sebelumnya.

### Penjelasan perintah

- `composer require laravel/breeze --dev`: memasang package Breeze
- `php artisan breeze:install`: membuat file autentikasi
- `npm install`: memasang dependency frontend
- `npm run dev`: menjalankan asset development
- `php artisan migrate`: memastikan tabel yang dibutuhkan sudah ada

## Hasil Setelah Breeze Dipasang

Setelah instalasi berhasil, aplikasi akan memiliki:

- halaman login
- halaman register
- halaman dashboard bawaan yang diproteksi
- route autentikasi

Ini menjadi fondasi untuk implementasi role-based access control.

## Membuat Middleware Role

Agar Laravel dapat memeriksa role user, buat middleware:

```bash
php artisan make:middleware EnsureUserHasRole
```

File:

```text
app/Http/Middleware/EnsureUserHasRole.php
```

Isi:

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class EnsureUserHasRole
{
    public function handle(Request $request, Closure $next, ...$roles)
    {
        $user = $request->user();

        if (! $user || ! in_array($user->role, $roles, true)) {
            abort(403, 'Anda tidak memiliki akses ke halaman ini.');
        }

        return $next($request);
    }
}
```

## Penjelasan Middleware

- `$request->user()` mengambil user yang sedang login
- `...$roles` berarti middleware bisa menerima banyak role sekaligus
- `in_array()` memeriksa apakah role user ada dalam daftar role yang diizinkan
- jika tidak cocok, aplikasi akan menampilkan error `403 Forbidden`

Contoh pemakaian nantinya:

```php
->middleware('role:admin,operator')
```

## Registrasi Middleware pada Laravel 11

Tambahkan alias middleware di `bootstrap/app.php`:

```php
use App\Http\Middleware\EnsureUserHasRole;
use Illuminate\Foundation\Configuration\Middleware;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'role' => EnsureUserHasRole::class,
    ]);
})
```

### Mengapa perlu alias

Agar middleware bisa dipanggil dengan nama singkat seperti:

```php
middleware('role:admin')
```

## Melindungi Route dengan `auth`

Sebelum memeriksa role, pastikan user sudah login. Itulah fungsi middleware `auth`.

Contoh dasar:

```php
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
});
```

Artinya, semua route di dalam group hanya bisa diakses user yang sudah login.

## Proteksi Route Berdasarkan Role

Contoh `routes/web.php`:

```php
use App\Http\Controllers\DashboardController;
use App\Http\Controllers\DeviceController;
use App\Http\Controllers\PlantController;
use App\Http\Controllers\ReportController;
use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');

    Route::middleware('role:admin,operator')->group(function () {
        Route::resource('devices', DeviceController::class);
        Route::resource('plants', PlantController::class);
        Route::resource('reports', ReportController::class)->except(['index', 'show']);
    });

    Route::middleware('role:admin')->group(function () {
        Route::resource('users', UserController::class);
    });

    Route::get('/reports', [ReportController::class, 'index'])->name('reports.index');
    Route::get('/reports/{report}', [ReportController::class, 'show'])->name('reports.show');
});
```

## Penjelasan Pengelompokan Route

### Semua user login

Boleh mengakses:

- dashboard
- daftar laporan
- detail laporan

### Admin dan operator

Boleh mengakses:

- CRUD alat
- CRUD tanaman
- tambah dan ubah laporan

### Admin saja

Boleh mengakses:

- manajemen user

## Mengatur Role Saat Registrasi

Pada aplikasi produksi, user sebaiknya tidak memilih role secara bebas saat register. Strategi yang disarankan:

- user baru otomatis menjadi `viewer`
- `admin` dapat mengubah role dari halaman manajemen user

Dengan pendekatan ini, keamanan lebih terjaga.

## Menetapkan Default Role pada Register

Jika memakai Breeze, proses registrasi biasanya berada pada controller autentikasi yang dibuat Breeze. Saat user dibuat, tetapkan role default:

```php
'role' => 'viewer',
```

Contoh pengisian data user:

```php
User::create([
    'name' => $request->name,
    'email' => $request->email,
    'password' => Hash::make($request->password),
    'role' => 'viewer',
]);
```

### Mengapa tidak membiarkan user memilih role

Karena jika role diambil dari form publik tanpa kontrol, siapa pun bisa mendaftar sebagai admin.

## Menyiapkan `UserController`

Agar admin bisa mengelola role, siapkan controller:

```bash
php artisan make:controller UserController --resource
```

Fitur minimum untuk admin:

- melihat daftar user
- melihat detail user
- mengubah role user

## Contoh Method `index()` pada `UserController`

```php
use App\Models\User;

public function index()
{
    $users = User::latest()->get();

    return view('users.index', compact('users'));
}
```

## Contoh Method `update()` untuk Mengubah Role

```php
use Illuminate\Http\Request;

public function update(Request $request, User $user)
{
    $validated = $request->validate([
        'name' => ['required', 'string', 'max:100'],
        'email' => ['required', 'email'],
        'role' => ['required', 'in:admin,operator,viewer'],
    ]);

    $user->update($validated);

    return redirect()
        ->route('users.index')
        ->with('success', 'Data user berhasil diperbarui.');
}
```

## Contoh Halaman Daftar User

File:

```text
resources/views/users/index.blade.php
```

Isi sederhana:

```blade
@extends('layouts.app')

@section('title', 'Manajemen User')

@section('content')
    <x-page-header
        title="Manajemen User"
        subtitle="Kelola akun dan role pengguna greenhouse."
    />

    <div class="overflow-hidden rounded-xl bg-white shadow-sm ring-1 ring-slate-200">
        <table class="w-full text-left">
            <thead class="bg-slate-100 text-sm text-slate-600">
                <tr>
                    <th class="px-4 py-3">Nama</th>
                    <th class="px-4 py-3">Email</th>
                    <th class="px-4 py-3">Role</th>
                    <th class="px-4 py-3">Tanggal Dibuat</th>
                </tr>
            </thead>
            <tbody>
                @forelse ($users as $user)
                    <tr class="border-t border-slate-200">
                        <td class="px-4 py-3">{{ $user->name }}</td>
                        <td class="px-4 py-3">{{ $user->email }}</td>
                        <td class="px-4 py-3">{{ $user->role }}</td>
                        <td class="px-4 py-3">{{ $user->created_at->format('d-m-Y') }}</td>
                    </tr>
                @empty
                    <tr>
                        <td colspan="4" class="px-4 py-6 text-center text-slate-500">
                            Belum ada data user.
                        </td>
                    </tr>
                @endforelse
            </tbody>
        </table>
    </div>
@endsection
```

## Menampilkan Role User pada Layout

Untuk membantu pengguna memahami aksesnya, tampilkan role user di layout atau navbar.

Contoh:

```blade
@auth
    <p class="text-sm text-slate-500">
        Login sebagai: {{ auth()->user()->name }} ({{ auth()->user()->role }})
    </p>
@endauth
```

Ini bermanfaat saat dosen atau mahasiswa sedang menguji akun berbeda.

## Mengarahkan User Setelah Login

Strategi sederhana yang bisa diterapkan:

- semua role diarahkan ke dashboard

Strategi yang lebih lanjut:

- admin ke dashboard admin
- operator ke dashboard operasional
- viewer ke halaman ringkasan

Untuk praktikum ini, satu dashboard bersama sudah cukup.

## Menguji Hak Akses

Setelah middleware dan route diterapkan, lakukan pengujian:

### Uji akun `admin`

Harus bisa:

- membuka dashboard
- membuka manajemen user
- membuka alat
- membuka tanaman
- membuka laporan

### Uji akun `operator`

Harus bisa:

- membuka dashboard
- membuka alat
- membuka tanaman
- membuka laporan

Tidak boleh:

- membuka manajemen user

### Uji akun `viewer`

Harus bisa:

- membuka dashboard
- melihat laporan

Tidak boleh:

- membuka manajemen user
- menambah atau mengubah alat
- menambah atau mengubah tanaman

## Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi:

- lupa menambahkan kolom `role` di tabel `users`
- lupa menambahkan `role` ke `$fillable`
- middleware sudah dibuat tetapi belum didaftarkan
- route hanya memakai `auth` tanpa pemeriksaan role
- user bisa memilih role saat register
- pengujian role tidak dilakukan dengan akun berbeda

## Tips Debug

Jika role-based access tidak berjalan:

1. cek apakah user benar-benar login
2. cek nilai kolom `role` di tabel `users`
3. cek apakah middleware alias `role` sudah didaftarkan
4. cek apakah route group sudah benar
5. uji akses dengan akun berbeda
6. cek pesan error `403`

Untuk mengecek route yang aktif:

```bash
php artisan route:list
```

## Praktik Keamanan yang Disarankan

- selalu gabungkan `auth` dan pemeriksaan role untuk route backend
- jangan mengambil role dari input register publik
- buat akun admin awal melalui seeder
- tampilkan error `403` untuk akses yang ditolak
- batasi pengelolaan user hanya untuk admin

## Pengembangan Lanjutan dari Modul Ini

Setelah autentikasi dan RBAC selesai:

- form dan upload file bisa dibatasi sesuai role
- laporan dapat dibedakan hak akses lihat dan edit
- API backend dapat diberi lapisan otorisasi tambahan
- dashboard dapat menampilkan menu berbeda tergantung role

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review konsep login dan role
2. instal Breeze
3. uji login dan register
4. tambahkan kolom `role`
5. buat middleware role
6. kelompokkan route berdasarkan role
7. buat halaman manajemen user
8. uji akses dengan tiga akun berbeda

## Hasil Akhir Modul

Mahasiswa memiliki sistem login dan hak akses berbasis role yang sesuai dengan kebutuhan greenhouse monitoring. Admin dapat mengelola seluruh sistem, operator mengelola operasional, dan viewer hanya melihat data yang diizinkan.

## Tugas Praktik

1. Buat halaman daftar user yang hanya bisa diakses admin.
2. Tambahkan badge role pada layout setelah user login.
3. Uji akses menggunakan tiga akun: `admin`, `operator`, dan `viewer`.
4. Pastikan user baru hasil register selalu mendapat role `viewer`.
5. Tambahkan proteksi agar `viewer` tidak dapat membuka halaman create dan edit untuk alat serta tanaman.
