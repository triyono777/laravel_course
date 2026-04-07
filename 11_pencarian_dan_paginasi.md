# Modul 11: Fitur Pencarian dan Paginasi

## Deskripsi Modul

Setelah aplikasi greenhouse monitoring memiliki autentikasi, CRUD, relasi, dan data yang cukup banyak dari seeder, tantangan berikutnya adalah kenyamanan penggunaan. Saat data alat, tanaman, user, histori sensor, dan laporan terus bertambah, halaman daftar akan cepat menjadi terlalu panjang dan sulit digunakan jika semua data langsung ditampilkan sekaligus.

Modul ini membahas dua solusi penting:

- **pencarian** untuk membantu user menemukan data tertentu
- **paginasi** untuk membagi data menjadi beberapa halaman agar tampilan tetap ringan dan rapi

Pada tahap ini, aplikasi greenhouse monitoring akan mulai terasa seperti aplikasi yang siap dipakai, bukan hanya sekadar demo CRUD.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan fungsi pencarian dan pagination pada aplikasi web
- membuat fitur pencarian berbasis input user
- menggunakan local query scope pada Eloquent
- membuat filter sederhana pada controller
- menampilkan data dengan pagination
- mempertahankan query pencarian pada URL
- menggabungkan search, filter, dan pagination secara rapi

## Keterkaitan dengan Studi Kasus

Pada aplikasi greenhouse monitoring, data berikut akan terus bertambah:

- alat
- tanaman
- user
- laporan
- histori sensor

Jika semua data ditampilkan sekaligus:

- halaman akan terasa berat
- user kesulitan menemukan data tertentu
- pengalaman penggunaan menjadi buruk

Karena itu fitur pencarian dan paginasi sangat dibutuhkan, misalnya untuk:

- mencari alat berdasarkan kode atau lokasi
- mencari tanaman berdasarkan nama atau blok greenhouse
- mencari laporan berdasarkan judul dan periode
- menelusuri histori sensor tanpa harus membaca ratusan baris dalam satu halaman

## Mengapa Pencarian Penting

Bayangkan ada:

- 100 alat
- 300 tanaman
- 1000 histori sensor
- 50 laporan

Tanpa pencarian, user harus scroll panjang hanya untuk menemukan satu data. Ini tidak efisien dan tidak realistis untuk aplikasi operasional.

Pencarian membantu menjawab kebutuhan seperti:

- "Cari alat dengan kode GH-102"
- "Tampilkan tanaman di Blok B"
- "Cari laporan minggu lalu"

## Mengapa Pagination Penting

Pagination membagi data menjadi halaman kecil, misalnya:

- 10 data per halaman
- 15 data per halaman
- 20 data per halaman

Keuntungan pagination:

- halaman lebih ringan
- data lebih mudah dibaca
- user tidak kewalahan
- query database lebih efisien

## Persiapan File yang Digunakan

Pada modul ini, pencarian dan pagination diterapkan ke file yang umumnya sudah ada dari modul-modul sebelumnya, terutama model dan controller.

Jika beberapa controller belum dibuat, Anda dapat menyiapkannya dengan Artisan:

```bash
php artisan make:controller UserController --resource
php artisan make:controller ReportController --resource
```

Jika model tertentu belum tersedia, buat juga dengan Artisan:

```bash
php artisan make:model Report
```

Catatan:

- `DeviceController` dan `PlantController` umumnya sudah dibuat pada Modul 6
- Modul 11 lebih berfokus pada pengembangan query di model dan method `index()` di controller
- tidak semua perintah di atas harus dijalankan jika file yang dimaksud sudah tersedia

## Strategi Implementasi Modul Ini

Untuk menjaga kode tetap rapi, strategi yang digunakan adalah:

1. simpan logika pencarian sederhana di model melalui local query scope
2. gunakan controller untuk menggabungkan search, filter, dan pagination
3. gunakan form `GET` agar parameter pencarian muncul di URL
4. gunakan `withQueryString()` agar saat pindah halaman filter tetap terbawa

Pendekatan ini sederhana tetapi sangat efektif untuk aplikasi Laravel skala praktikum.

## Apa Itu Local Query Scope

Local query scope adalah method khusus pada model yang digunakan untuk membungkus potongan query agar bisa dipakai berulang.

Keuntungannya:

- query lebih rapi
- controller lebih pendek
- logika pencarian dapat dipakai ulang di banyak tempat

Contoh penggunaan:

```php
Device::query()->search($request->keyword)->get();
```

Method `search()` di atas berasal dari model, bukan method bawaan Laravel.

## Membuat Scope pada Model `Device`

File:

```text
app/Models/Device.php
```

Tambahkan:

```php
use Illuminate\Database\Eloquent\Builder;

public function scopeSearch(Builder $query, ?string $keyword): Builder
{
    if (! $keyword) {
        return $query;
    }

    return $query->where(function (Builder $query) use ($keyword) {
        $query->where('code', 'like', "%{$keyword}%")
            ->orWhere('name', 'like', "%{$keyword}%")
            ->orWhere('location', 'like', "%{$keyword}%");
    });
}
```

### Penjelasan

- `?string $keyword` berarti keyword boleh kosong
- jika keyword kosong, query dikembalikan tanpa perubahan
- pencarian dilakukan pada `code`, `name`, dan `location`

Pendekatan ini cocok untuk halaman daftar alat.

## Membuat Scope pada Model `Plant`

File:

```text
app/Models/Plant.php
```

Tambahkan:

```php
use Illuminate\Database\Eloquent\Builder;

public function scopeSearch(Builder $query, ?string $keyword): Builder
{
    if (! $keyword) {
        return $query;
    }

    return $query->where(function (Builder $query) use ($keyword) {
        $query->where('name', 'like', "%{$keyword}%")
            ->orWhere('variety', 'like', "%{$keyword}%")
            ->orWhere('greenhouse_block', 'like', "%{$keyword}%");
    });
}
```

### Penjelasan

Pencarian tanaman biasanya dilakukan berdasarkan:

- nama tanaman
- varietas
- blok greenhouse

## Membuat Scope pada Model `User`

Jika halaman user ingin bisa dicari:

File:

```text
app/Models/User.php
```

Tambahkan:

```php
use Illuminate\Database\Eloquent\Builder;

public function scopeSearch(Builder $query, ?string $keyword): Builder
{
    if (! $keyword) {
        return $query;
    }

    return $query->where(function (Builder $query) use ($keyword) {
        $query->where('name', 'like', "%{$keyword}%")
            ->orWhere('email', 'like', "%{$keyword}%")
            ->orWhere('role', 'like', "%{$keyword}%");
    });
}
```

## Menggabungkan Search dan Filter di Controller

## Contoh `DeviceController@index`

```php
use App\Models\Device;
use Illuminate\Http\Request;

public function index(Request $request)
{
    $devices = Device::query()
        ->search($request->keyword)
        ->when($request->status, function ($query, $status) {
            $query->where('status', $status);
        })
        ->latest()
        ->paginate(10)
        ->withQueryString();

    return view('devices.index', compact('devices'));
}
```

### Penjelasan

- `search($request->keyword)` memanggil local query scope
- `when()` hanya menambahkan filter jika nilai tersedia
- `paginate(10)` membatasi 10 data per halaman
- `withQueryString()` menjaga keyword dan filter tetap terbawa saat pindah halaman

## Contoh `PlantController@index`

```php
use App\Models\Plant;
use Illuminate\Http\Request;

public function index(Request $request)
{
    $plants = Plant::query()
        ->with('device')
        ->search($request->keyword)
        ->when($request->greenhouse_block, function ($query, $block) {
            $query->where('greenhouse_block', $block);
        })
        ->latest()
        ->paginate(10)
        ->withQueryString();

    return view('plants.index', compact('plants'));
}
```

### Catatan

Pada halaman tanaman, kita memadukan:

- relasi `device`
- pencarian
- filter blok greenhouse
- pagination

## Contoh `UserController@index`

```php
use App\Models\User;
use Illuminate\Http\Request;

public function index(Request $request)
{
    $users = User::query()
        ->search($request->keyword)
        ->latest()
        ->paginate(10)
        ->withQueryString();

    return view('users.index', compact('users'));
}
```

## Contoh `ReportController@index`

Untuk laporan, pencarian biasanya lebih kompleks karena sering melibatkan judul dan rentang tanggal.

```php
use App\Models\Report;
use Illuminate\Http\Request;

public function index(Request $request)
{
    $reports = Report::query()
        ->with('user')
        ->when($request->keyword, function ($query, $keyword) {
            $query->where('title', 'like', "%{$keyword}%");
        })
        ->when($request->start_date, function ($query, $date) {
            $query->whereDate('period_start', '>=', $date);
        })
        ->when($request->end_date, function ($query, $date) {
            $query->whereDate('period_end', '<=', $date);
        })
        ->latest()
        ->paginate(10)
        ->withQueryString();

    return view('reports.index', compact('reports'));
}
```

## Mengapa Menggunakan Form `GET`

Untuk pencarian dan filter, form sebaiknya memakai method `GET`.

Contoh:

```blade
<form method="GET" action="{{ route('devices.index') }}">
```

Keuntungan:

- parameter tampil di URL
- hasil pencarian bisa di-refresh tanpa kehilangan filter
- link hasil pencarian bisa dibagikan
- lebih cocok untuk operasi baca data

## Form Pencarian pada Blade

Contoh untuk halaman alat:

```blade
<form method="GET" action="{{ route('devices.index') }}" class="mb-4 flex flex-wrap gap-2">
    <input
        type="text"
        name="keyword"
        value="{{ request('keyword') }}"
        placeholder="Cari kode, nama, lokasi"
        class="rounded border px-3 py-2"
    >

    <select name="status" class="rounded border px-3 py-2">
        <option value="">Semua status</option>
        <option value="active" @selected(request('status') === 'active')>Active</option>
        <option value="inactive" @selected(request('status') === 'inactive')>Inactive</option>
        <option value="maintenance" @selected(request('status') === 'maintenance')>Maintenance</option>
    </select>

    <button type="submit" class="rounded bg-emerald-600 px-4 py-2 text-white">
        Cari
    </button>

    <a href="{{ route('devices.index') }}" class="rounded border px-4 py-2">
        Reset
    </a>
</form>
```

### Mengapa `request('keyword')` dipakai

Supaya nilai pencarian tetap tampil di input setelah form dikirim.

## Menampilkan Hasil Filter Aktif

Supaya user tahu halaman sedang difilter, tampilkan informasi filter aktif:

```blade
@if (request('keyword') || request('status'))
    <p class="mb-4 text-sm text-slate-500">
        Filter aktif:
        kata kunci = "{{ request('keyword') ?: '-' }}",
        status = "{{ request('status') ?: 'semua' }}"
    </p>
@endif
```

Ini membantu UX karena user tahu mengapa data yang muncul tidak lengkap.

## Menampilkan Pagination

Laravel menyediakan link pagination otomatis:

```blade
{{ $devices->links() }}
```

Contoh ini akan menghasilkan tombol:

- Previous
- nomor halaman
- Next

## Mengapa `withQueryString()` Penting

Tanpa `withQueryString()`, saat user pindah ke halaman 2:

- keyword pencarian bisa hilang
- filter status bisa hilang

Akibatnya hasil pencarian menjadi membingungkan.

Dengan:

```php
->withQueryString()
```

semua parameter pada URL akan tetap ikut terbawa.

## Simple Pagination vs Pagination Biasa

Laravel menyediakan dua pendekatan umum:

### `paginate()`

```php
->paginate(10)
```

Kelebihan:

- ada nomor halaman
- cocok untuk tabel CRUD umum

### `simplePaginate()`

```php
->simplePaginate(10)
```

Kelebihan:

- lebih ringan
- cocok untuk data sangat besar

Untuk praktikum ini, `paginate()` lebih mudah dipahami dan lebih cocok untuk sebagian besar halaman.

## Contoh Halaman Daftar Alat dengan Search dan Pagination

```blade
@extends('layouts.app')

@section('title', 'Data Alat')

@section('content')
    <x-page-header
        title="Data Alat"
        subtitle="Cari dan telusuri perangkat greenhouse."
    />

    <form method="GET" action="{{ route('devices.index') }}" class="mb-4 flex flex-wrap gap-2">
        <input type="text" name="keyword" value="{{ request('keyword') }}" placeholder="Cari kode, nama, lokasi" class="rounded border px-3 py-2">

        <select name="status" class="rounded border px-3 py-2">
            <option value="">Semua status</option>
            <option value="active" @selected(request('status') === 'active')>Active</option>
            <option value="inactive" @selected(request('status') === 'inactive')>Inactive</option>
            <option value="maintenance" @selected(request('status') === 'maintenance')>Maintenance</option>
        </select>

        <button type="submit" class="rounded bg-emerald-600 px-4 py-2 text-white">Cari</button>
    </form>

    <div class="overflow-hidden rounded-xl bg-white shadow-sm ring-1 ring-slate-200">
        <table class="w-full text-left">
            <thead class="bg-slate-100 text-sm text-slate-600">
                <tr>
                    <th class="px-4 py-3">Kode</th>
                    <th class="px-4 py-3">Nama</th>
                    <th class="px-4 py-3">Lokasi</th>
                    <th class="px-4 py-3">Status</th>
                </tr>
            </thead>
            <tbody>
                @forelse ($devices as $device)
                    <tr class="border-t border-slate-200">
                        <td class="px-4 py-3">{{ $device->code }}</td>
                        <td class="px-4 py-3">{{ $device->name }}</td>
                        <td class="px-4 py-3">{{ $device->location }}</td>
                        <td class="px-4 py-3">{{ $device->status }}</td>
                    </tr>
                @empty
                    <tr>
                        <td colspan="4" class="px-4 py-6 text-center text-slate-500">
                            Data tidak ditemukan.
                        </td>
                    </tr>
                @endforelse
            </tbody>
        </table>
    </div>

    <div class="mt-4">
        {{ $devices->links() }}
    </div>
@endsection
```

## Pencarian pada Histori Sensor

Histori sensor sering memiliki data sangat banyak. Untuk itu, strategi yang lebih tepat bisa berupa:

- filter berdasarkan perangkat
- filter tanggal
- pagination terpisah

Contoh:

```php
$readings = SensorReading::query()
    ->with('device')
    ->when($request->device_id, fn ($query, $deviceId) => $query->where('device_id', $deviceId))
    ->when($request->date, fn ($query, $date) => $query->whereDate('recorded_at', $date))
    ->latest('recorded_at')
    ->paginate(20)
    ->withQueryString();
```

Pendekatan ini lebih berguna daripada hanya mencari dengan keyword biasa.

## Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi:

- query pencarian ditulis berulang di banyak controller
- lupa memakai `withQueryString()`
- lupa memakai method `GET` pada form search
- parameter form tidak sama dengan yang dibaca di controller
- pencarian memakai `orWhere` tanpa grouping yang benar
- pagination sudah ada tetapi tabel tetap terlalu berat karena query tidak efisien

## Tips Debug

Jika pencarian atau paginasi tidak berjalan sesuai harapan:

1. cek nama input form
2. cek parameter request di controller
3. cek apakah scope dipanggil dengan benar
4. cek apakah `paginate()` dipakai, bukan `get()`
5. cek apakah `links()` ditampilkan di Blade
6. cek apakah filter hilang saat pindah halaman

## Praktik yang Disarankan

- gunakan pagination pada semua tabel master
- batasi jumlah data per halaman, misalnya 10 atau 15
- tampilkan filter aktif agar user tahu hasil sedang difilter
- gunakan scope pada model untuk pencarian yang berulang
- gunakan `with()` bila halaman menampilkan relasi

## Pengembangan Lanjutan dari Modul Ini

Setelah search dan pagination diterapkan:

- halaman master data menjadi lebih nyaman digunakan
- laporan lebih mudah dicari
- histori sensor lebih mudah ditelusuri
- dashboard bisa menyediakan link menuju data terfilter

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review data hasil seeding
2. tunjukkan masalah jika data ditampilkan semua sekaligus
3. buat local query scope
4. tambahkan search pada controller
5. tambahkan filter pada form Blade
6. tambahkan pagination
7. uji perpindahan halaman dengan filter aktif
8. terapkan pola yang sama ke tabel lain

## Hasil Akhir Modul

Mahasiswa dapat membuat tabel data yang lebih nyaman dipakai, cepat dicari, tetap ringan saat datanya besar, dan siap dipakai pada aplikasi greenhouse monitoring yang terus bertambah datanya.

## Tugas Praktik

1. Tambahkan pencarian pada halaman user.
2. Tambahkan filter blok greenhouse pada halaman tanaman.
3. Buat pagination terpisah untuk histori data sensor.
4. Tambahkan tombol reset filter pada halaman alat.
5. Bandingkan penggunaan `paginate()` dan `simplePaginate()` pada salah satu tabel.
