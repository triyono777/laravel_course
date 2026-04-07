# Modul 4: Blade Templating Engine dan UI Dasar

## Deskripsi Modul

Pada Modul 3, mahasiswa sudah berhasil membuat halaman dashboard dan halaman daftar alat menggunakan route, controller, dan view. Namun tampilan yang dibuat masih sangat dasar dan cenderung berulang. Setiap file view masih menulis struktur HTML dari awal, sehingga jika aplikasi membesar maka penulisan seperti itu akan cepat menjadi tidak rapi.

Modul ini membahas cara Laravel menyederhanakan tampilan menggunakan **Blade Templating Engine**. Dengan Blade, mahasiswa dapat:

- menampilkan data dengan sintaks yang ringkas
- membuat layout yang dipakai ulang
- memecah tampilan menjadi komponen kecil
- membangun antarmuka yang lebih konsisten

Pada akhir modul, proyek greenhouse monitoring akan memiliki layout awal, komponen statistik, komponen header halaman, dan struktur UI yang lebih siap untuk dikembangkan ke halaman user, alat, tanaman, dan laporan.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan fungsi Blade pada Laravel
- menggunakan sintaks dasar Blade untuk menampilkan data
- menggunakan directive Blade seperti `@if`, `@foreach`, dan `@extends`
- membuat layout utama aplikasi
- membuat komponen Blade dengan Artisan
- menggunakan slot pada komponen Blade
- mengintegrasikan UI dasar dengan Tailwind CSS
- menyiapkan antarmuka awal dashboard greenhouse

## Keterkaitan dengan Studi Kasus

Sistem greenhouse monitoring memiliki banyak halaman:

- dashboard
- user
- alat
- tanaman
- laporan

Jika semua halaman dibuat dengan HTML penuh secara berulang, maka:

- navigasi akan sulit dijaga konsistensinya
- perubahan desain akan memakan waktu lama
- struktur kode view akan cepat berantakan

Karena itu kita membutuhkan:

- satu layout utama
- komponen yang reusable
- pola tampilan yang seragam

## Apa Itu Blade

Blade adalah templating engine bawaan Laravel yang digunakan untuk menulis tampilan dengan lebih ringkas dan terstruktur.

Blade tetap menghasilkan HTML biasa di browser, tetapi penulisannya dibantu dengan sintaks khusus seperti:

- `{{ }}`
- `@if`
- `@foreach`
- `@extends`
- `@section`
- `@yield`
- komponen seperti `<x-stat-card />`

Blade memudahkan pengembang memisahkan:

- data dari controller
- struktur tampilan
- elemen antarmuka yang dipakai berulang

## Mengapa Blade Penting

Tanpa Blade, mahasiswa harus sering menulis HTML yang berulang pada setiap halaman, misalnya:

- `<html>`
- `<head>`
- `<body>`
- navbar
- sidebar
- footer

Jika ada 10 halaman, maka perubahan kecil pada navigasi bisa berarti harus mengedit 10 file. Blade menyelesaikan masalah ini dengan layout dan komponen.

## Konsep Dasar Blade

## 1. Menampilkan Variabel

Gunakan:

```blade
{{ $nama }}
```

Contoh:

```blade
<h1>{{ $title }}</h1>
```

Blade akan menampilkan isi variabel dengan aman karena karakter HTML akan di-escape secara default.

## 2. Menampilkan HTML Mentah

Jika benar-benar dibutuhkan:

```blade
{!! $html !!}
```

Namun ini harus dipakai hati-hati karena dapat menimbulkan celah keamanan jika sumber datanya tidak terpercaya.

Untuk praktikum dasar, lebih aman menggunakan `{{ }}`.

## 3. Kondisi dengan `@if`

```blade
@if ($summary['active_devices'] < $summary['total_devices'])
    <p>Ada alat yang memerlukan pengecekan.</p>
@endif
```

Directive ini berguna saat dashboard perlu menampilkan peringatan.

## 4. Perulangan dengan `@foreach`

```blade
@foreach ($devices as $device)
    <li>{{ $device['name'] }}</li>
@endforeach
```

Directive ini sering dipakai untuk:

- tabel data
- daftar notifikasi
- daftar alat
- daftar tanaman

## 5. Perulangan dengan `@forelse`

Untuk tampilan yang lebih ramah, Blade menyediakan `@forelse`:

```blade
@forelse ($devices as $device)
    <li>{{ $device['name'] }}</li>
@empty
    <li>Belum ada data alat.</li>
@endforelse
```

Ini berguna agar halaman tidak kosong tanpa pesan saat data belum tersedia.

## 6. Komentar Blade

```blade
{{-- Ini komentar Blade dan tidak tampil di browser --}}
```

## Struktur UI yang Akan Dibangun

Pada modul ini kita akan menyiapkan:

1. layout utama aplikasi
2. sidebar sederhana
3. komponen kartu statistik
4. komponen header halaman dengan slot aksi
5. dashboard dengan tampilan yang lebih rapi

## Strategi yang Digunakan

Pada modul ini kita akan memadukan dua teknik Blade:

- `@extends`, `@section`, dan `@yield` untuk layout utama
- Blade component untuk elemen UI yang reusable

Alasan memilih pola ini:

- lebih mudah dipahami oleh mahasiswa pemula
- tetap sesuai praktik Laravel
- cukup fleksibel untuk dikembangkan pada modul selanjutnya

## Langkah Praktik

## 1. Membuat Layout Utama

Pertama, buat folder:

```text
resources/views/layouts
resources/views/partials
resources/views/components
```

Buat file layout utama:

```text
resources/views/layouts/app.blade.php
```

Isi file tersebut dengan:

```blade
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Greenhouse Monitoring')</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="bg-slate-100 text-slate-800">
    <div class="min-h-screen md:flex">
        @include('partials.sidebar')

        <div class="flex-1">
            <main class="p-6">
                @yield('content')
            </main>
        </div>
    </div>
</body>
</html>
```

### Penjelasan

- `@yield('title')` menjadi tempat judul halaman
- `@yield('content')` menjadi tempat isi halaman
- `@include('partials.sidebar')` memanggil sidebar dari file terpisah
- `@vite(...)` memuat asset CSS dan JavaScript

## 2. Membuat Partial Sidebar

Buat file:

```text
resources/views/partials/sidebar.blade.php
```

Isi dengan:

```blade
<aside class="bg-emerald-700 p-5 text-white md:min-h-screen md:w-64">
    <h1 class="text-xl font-bold">Greenhouse Monitoring</h1>
    <p class="mt-1 text-sm text-emerald-100">Web monitoring dan manajemen greenhouse</p>

    <nav class="mt-6 space-y-2">
        <a href="{{ route('dashboard') }}" class="block rounded px-3 py-2 hover:bg-emerald-600">
            Dashboard
        </a>
        <a href="{{ route('devices.index') }}" class="block rounded px-3 py-2 hover:bg-emerald-600">
            Alat
        </a>
        <a href="#" class="block rounded px-3 py-2 hover:bg-emerald-600">
            Tanaman
        </a>
        <a href="#" class="block rounded px-3 py-2 hover:bg-emerald-600">
            Laporan
        </a>
    </nav>
</aside>
```

### Mengapa memakai partial

Sidebar adalah elemen yang akan dipakai pada banyak halaman. Memisahkannya ke file tersendiri membuat:

- kode lebih pendek
- perubahan menu lebih mudah
- layout lebih bersih

## 3. Memakai Layout pada Dashboard

Perbarui file:

```text
resources/views/dashboard/index.blade.php
```

Menjadi:

```blade
@extends('layouts.app')

@section('title', 'Dashboard Greenhouse')

@section('content')
    <x-page-header
        title="Dashboard Greenhouse"
        subtitle="Ringkasan kondisi greenhouse berdasarkan data monitoring terbaru."
    >
        <x-slot:actions>
            <a href="{{ route('devices.index') }}" class="rounded bg-emerald-600 px-4 py-2 text-sm font-medium text-white hover:bg-emerald-700">
                Lihat Alat
            </a>
        </x-slot:actions>
    </x-page-header>

    <div class="grid gap-4 md:grid-cols-2 xl:grid-cols-4">
        <x-stat-card label="Total Alat" :value="$summary['total_devices']" />
        <x-stat-card label="Alat Aktif" :value="$summary['active_devices']" />
        <x-stat-card label="Total Tanaman" :value="$summary['total_plants']" />
        <x-stat-card label="Rata-rata Suhu" :value="$summary['avg_temperature'].' C'" />
    </div>

    @if (! empty($alerts))
        <div class="mt-6 rounded-xl border border-yellow-200 bg-yellow-50 p-4">
            <h3 class="font-semibold text-yellow-800">Peringatan Sistem</h3>

            <ul class="mt-2 list-disc pl-5 text-sm text-yellow-700">
                @foreach ($alerts as $alert)
                    <li>{{ $alert }}</li>
                @endforeach
            </ul>
        </div>
    @endif
@endsection
```

### Konsep yang digunakan

- `@extends('layouts.app')` memakai layout utama
- `@section('title')` mengisi judul halaman
- `@section('content')` mengisi area konten utama
- `<x-page-header>` dan `<x-stat-card>` memakai komponen Blade
- `<x-slot:actions>` menunjukkan penggunaan slot bernama

## 4. Membuat Komponen Blade dengan Artisan

Gunakan Artisan agar struktur komponen lebih rapi:

```bash
php artisan make:component StatCard
php artisan make:component PageHeader
```

Perintah di atas akan membuat:

- `app/View/Components/StatCard.php`
- `resources/views/components/stat-card.blade.php`
- `app/View/Components/PageHeader.php`
- `resources/views/components/page-header.blade.php`

## 5. Mengisi Class Komponen `StatCard`

Perbarui file:

```text
app/View/Components/StatCard.php
```

Menjadi:

```php
namespace App\View\Components;

use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class StatCard extends Component
{
    public function __construct(
        public string $label,
        public string $value
    ) {
    }

    public function render(): View|Closure|string
    {
        return view('components.stat-card');
    }
}
```

### Penjelasan

- `label` dan `value` akan menjadi properti yang bisa dipakai di Blade
- komponen ini cocok untuk kartu ringkasan dashboard

## 6. Mengisi View Komponen `StatCard`

Isi file:

```text
resources/views/components/stat-card.blade.php
```

Dengan:

```blade
<div class="rounded-2xl bg-white p-5 shadow-sm ring-1 ring-slate-200">
    <p class="text-sm text-slate-500">{{ $label }}</p>
    <h3 class="mt-2 text-3xl font-bold text-emerald-700">{{ $value }}</h3>
</div>
```

## 7. Mengisi Class Komponen `PageHeader`

Perbarui file:

```text
app/View/Components/PageHeader.php
```

Menjadi:

```php
namespace App\View\Components;

use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class PageHeader extends Component
{
    public function __construct(
        public string $title,
        public string $subtitle = ''
    ) {
    }

    public function render(): View|Closure|string
    {
        return view('components.page-header');
    }
}
```

## 8. Mengisi View Komponen `PageHeader`

Isi file:

```text
resources/views/components/page-header.blade.php
```

Dengan:

```blade
<div class="mb-6 flex flex-col gap-4 rounded-2xl bg-white p-5 shadow-sm ring-1 ring-slate-200 md:flex-row md:items-center md:justify-between">
    <div>
        <h2 class="text-2xl font-bold text-slate-800">{{ $title }}</h2>

        @if ($subtitle)
            <p class="mt-1 text-sm text-slate-500">{{ $subtitle }}</p>
        @endif
    </div>

    @if (isset($actions))
        <div>
            {{ $actions }}
        </div>
    @endif
</div>
```

### Di mana konsep slot dipakai

Bagian berikut:

```blade
{{ $actions }}
```

akan menerima isi dari:

```blade
<x-slot:actions>
    ...
</x-slot:actions>
```

Ini disebut **named slot** dan sangat berguna untuk membuat komponen yang fleksibel.

## 9. Memahami Perbedaan Layout, Partial, dan Component

Mahasiswa sering bingung membedakan ketiganya. Ringkasnya:

### Layout

Dipakai untuk kerangka besar halaman, misalnya:

- `<html>`
- `<head>`
- sidebar
- area utama

Contoh:

- `layouts/app.blade.php`

### Partial

Dipakai untuk potongan tampilan yang sederhana dan sering disisipkan.

Contoh:

- sidebar
- footer
- breadcrumb sederhana

Contoh:

- `partials/sidebar.blade.php`

### Component

Dipakai untuk elemen UI reusable yang memiliki perilaku atau parameter tertentu.

Contoh:

- stat card
- page header
- alert box
- badge status

## 10. Integrasi Tailwind CSS

Laravel modern sudah nyaman digunakan bersama Tailwind CSS. Jalankan:

```bash
npm install
npm run dev
```

Pastikan file berikut memuat Tailwind:

```text
resources/css/app.css
```

Isi sederhana:

```css
@import "tailwindcss";

body {
    font-family: "Segoe UI", sans-serif;
}
```

Jika `npm run dev` tidak dijalankan, perubahan desain bisa jadi tidak terlihat saat pengembangan.

## 11. Optional: Integrasi Library UI

Jika dosen ingin pengayaan, dapat diperkenalkan library komponen tambahan seperti:

- Flowbite

Namun untuk praktikum inti, Tailwind dasar sudah cukup agar mahasiswa fokus ke struktur Blade terlebih dahulu.

## 12. Contoh Directive Blade yang Sering Dipakai

### `@if`

```blade
@if ($summary['avg_temperature'] > 30)
    <p class="text-red-600">Suhu greenhouse terlalu tinggi.</p>
@endif
```

### `@foreach`

```blade
@foreach ($alerts as $alert)
    <li>{{ $alert }}</li>
@endforeach
```

### `@forelse`

```blade
@forelse ($alerts as $alert)
    <li>{{ $alert }}</li>
@empty
    <li>Tidak ada peringatan.</li>
@endforelse
```

### `@include`

```blade
@include('partials.sidebar')
```

### `@extends` dan `@section`

```blade
@extends('layouts.app')

@section('content')
    <h1>Isi Halaman</h1>
@endsection
```

## 13. Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi:

- lupa menjalankan `npm run dev`
- nama komponen tidak sesuai dengan file Blade
- salah menulis `@section` atau lupa `@endsection`
- salah path saat `@include`
- terlalu banyak menaruh logika di Blade
- belum membedakan layout, partial, dan component

## 14. Cara Uji Hasil Modul

Setelah semua file selesai:

1. jalankan `php artisan serve`
2. jalankan `npm run dev`
3. buka halaman `/dashboard`
4. pastikan layout tampil
5. pastikan sidebar muncul
6. pastikan kartu statistik muncul
7. pastikan tombol aksi pada header tampil

## 15. Pengembangan Lanjutan dari Modul Ini

Struktur UI yang dibuat di modul ini akan dipakai lagi pada:

- halaman user
- halaman alat
- halaman tanaman
- halaman laporan
- halaman detail data

Pada modul berikutnya, isi halaman tetap bisa berubah, tetapi layout dan komponen yang sudah dibuat akan tetap dipakai sehingga aplikasi terasa konsisten.

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review hasil halaman dari Modul 3
2. tunjukkan masalah HTML yang berulang
3. perkenalkan Blade dan directive dasarnya
4. buat layout utama
5. buat partial sidebar
6. buat komponen stat card
7. buat komponen page header dengan slot
8. uji dashboard hasil akhir

## Hasil Akhir Modul

Mahasiswa memiliki layout utama, partial sidebar, komponen statistik, dan komponen header halaman yang reusable. Dashboard greenhouse menjadi lebih rapi, konsisten, dan siap dikembangkan untuk fitur-fitur berikutnya.

## Tugas Praktik

1. Tambahkan menu `Tanaman` dan `Laporan` pada sidebar dengan tampilan yang konsisten.
2. Buat komponen `StatusBadge` menggunakan `php artisan make:component StatusBadge`.
3. Tampilkan badge status `active` atau `maintenance` pada halaman alat.
4. Ubah daftar alert pada dashboard menggunakan `@forelse`.
5. Tambahkan satu blok informasi baru pada dashboard, misalnya `Rata-rata Kelembapan`.
