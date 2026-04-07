# Modul 9: Form, Validasi, dan Upload File

## Deskripsi Modul

Setelah mahasiswa memahami database, Eloquent, relasi, dan tampilan dasar, langkah berikutnya adalah membuat aplikasi benar-benar bisa menerima input dari pengguna. Pada tahap ini, sistem greenhouse monitoring harus mampu menerima data dari form, memeriksa apakah input valid, dan menyimpan file seperti gambar tanaman atau lampiran laporan.

Modul ini membahas tiga bagian yang saling berkaitan:

- form input pada Blade
- validasi data request
- upload file dan pengelolaan storage

Ketiga hal ini sangat penting karena hampir semua fitur aplikasi bisnis modern bergantung pada proses input data yang aman dan terstruktur.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- membuat form input data dengan Blade
- memahami alur pengiriman data dari form ke controller
- memvalidasi request secara langsung atau melalui Form Request
- mengunggah file dan gambar
- menyimpan path file ke database
- menampilkan file upload pada aplikasi
- menangani update file lama saat data diubah
- menerapkan praktik dasar keamanan pada upload file

## Keterkaitan dengan Studi Kasus

Pada aplikasi greenhouse monitoring, operator atau admin harus dapat:

- menambah alat
- menambah tanaman
- mengunggah foto tanaman
- membuat laporan monitoring
- mengunggah lampiran laporan

Karena itu modul ini akan berfokus pada dua skenario nyata:

- form input tanaman dengan upload gambar
- form input laporan dengan upload lampiran

## Mengapa Form dan Validasi Penting

Jika aplikasi menerima input tanpa validasi:

- data bisa kosong padahal wajib
- format tanggal bisa salah
- file yang diunggah bisa tidak sesuai
- ukuran file bisa terlalu besar
- data yang masuk ke database menjadi tidak konsisten

Validasi membantu aplikasi menjaga kualitas data sejak awal.

## Alur Form pada Laravel

Secara umum, alur kerja form pada Laravel adalah:

1. user membuka halaman form
2. user mengisi data
3. browser mengirim request ke route
4. controller atau Form Request memvalidasi input
5. jika valid, data disimpan ke database
6. jika tidak valid, user dikembalikan ke form beserta pesan error

## Struktur Fitur yang Akan Dibangun

Pada modul ini, kita fokus pada:

- `PlantController`
- `ReportController`
- `StorePlantRequest`
- `StoreReportRequest`
- file Blade form tambah tanaman
- file Blade form tambah laporan

## Form Request

Laravel menyediakan **Form Request** agar validasi lebih rapi dan terpisah dari controller.

Keuntungan Form Request:

- aturan validasi lebih terstruktur
- controller menjadi lebih bersih
- pesan error lebih mudah diatur
- logika validasi lebih mudah dipelihara

## Menyiapkan Request Class

Jalankan:

```bash
php artisan make:request StorePlantRequest
php artisan make:request UpdatePlantRequest
php artisan make:request StoreReportRequest
php artisan make:request UpdateReportRequest
```

### Mengapa membuat request terpisah

Sering kali validasi untuk `store` dan `update` tidak persis sama. Misalnya:

- saat tambah data, gambar mungkin opsional
- saat edit data, file lama mungkin boleh tetap dipakai

## Validasi Data Tanaman

File:

```text
app/Http/Requests/StorePlantRequest.php
```

Isi:

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePlantRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'device_id' => ['nullable', 'exists:devices,id'],
            'name' => ['required', 'string', 'max:100'],
            'variety' => ['nullable', 'string', 'max:100'],
            'greenhouse_block' => ['required', 'string', 'max:50'],
            'planted_at' => ['nullable', 'date', 'before_or_equal:today'],
            'image' => ['nullable', 'image', 'mimes:jpg,jpeg,png', 'max:2048'],
            'notes' => ['nullable', 'string'],
        ];
    }
}
```

### Penjelasan aturan validasi

- `exists:devices,id`: memastikan `device_id` benar-benar ada di tabel `devices`
- `required`: field wajib diisi
- `max:100`: membatasi panjang teks
- `before_or_equal:today`: tanggal tanam tidak boleh melebihi hari ini
- `image`: memastikan file yang diunggah benar-benar file gambar
- `mimes`: membatasi format file
- `max:2048`: ukuran maksimum 2 MB

## Validasi Data Laporan

File:

```text
app/Http/Requests/StoreReportRequest.php
```

Isi:

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreReportRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:150'],
            'period_start' => ['required', 'date'],
            'period_end' => ['required', 'date', 'after_or_equal:period_start'],
            'summary' => ['required', 'string'],
            'attachment' => ['nullable', 'file', 'mimes:pdf,xlsx,csv', 'max:4096'],
        ];
    }
}
```

### Penjelasan aturan validasi

- `period_end` tidak boleh lebih awal dari `period_start`
- lampiran hanya boleh file tertentu seperti `pdf`, `xlsx`, atau `csv`
- ukuran file dibatasi 4 MB

## Menyimpan Data Tanaman dengan Upload Gambar

Contoh `PlantController@store`:

```php
use App\Http\Requests\StorePlantRequest;
use App\Models\Plant;

public function store(StorePlantRequest $request)
{
    $data = $request->validated();

    if ($request->hasFile('image')) {
        $data['image'] = $request->file('image')->store('plants', 'public');
    }

    Plant::create($data);

    return redirect()
        ->route('plants.index')
        ->with('success', 'Tanaman berhasil ditambahkan.');
}
```

### Penjelasan

- `$request->validated()` hanya mengambil data yang lolos validasi
- `store('plants', 'public')` menyimpan file ke folder `storage/app/public/plants`
- path file disimpan ke kolom `image`

## Menyimpan Data Laporan dengan Lampiran

Contoh `ReportController@store`:

```php
use App\Http\Requests\StoreReportRequest;
use App\Models\Report;

public function store(StoreReportRequest $request)
{
    $data = $request->validated();
    $data['user_id'] = auth()->id();

    if ($request->hasFile('attachment')) {
        $data['attachment'] = $request->file('attachment')->store('reports', 'public');
    }

    Report::create($data);

    return redirect()
        ->route('reports.index')
        ->with('success', 'Laporan berhasil ditambahkan.');
}
```

### Catatan

Pada laporan, `user_id` biasanya diisi otomatis dari user yang sedang login.

## Membuat Form Blade untuk Tanaman

File:

```text
resources/views/plants/create.blade.php
```

Isi:

```blade
@extends('layouts.app')

@section('title', 'Tambah Tanaman')

@section('content')
    <x-page-header
        title="Tambah Tanaman"
        subtitle="Masukkan data tanaman greenhouse dan unggah gambar bila tersedia."
    />

    <form action="{{ route('plants.store') }}" method="POST" enctype="multipart/form-data" class="space-y-4 rounded-xl bg-white p-6 shadow-sm ring-1 ring-slate-200">
        @csrf

        <div>
            <label for="name">Nama Tanaman</label>
            <input type="text" name="name" id="name" value="{{ old('name') }}" class="mt-1 w-full rounded border px-3 py-2">
            @error('name')
                <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
            @enderror
        </div>

        <div>
            <label for="variety">Varietas</label>
            <input type="text" name="variety" id="variety" value="{{ old('variety') }}" class="mt-1 w-full rounded border px-3 py-2">
        </div>

        <div>
            <label for="greenhouse_block">Blok Greenhouse</label>
            <input type="text" name="greenhouse_block" id="greenhouse_block" value="{{ old('greenhouse_block') }}" class="mt-1 w-full rounded border px-3 py-2">
            @error('greenhouse_block')
                <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
            @enderror
        </div>

        <div>
            <label for="planted_at">Tanggal Tanam</label>
            <input type="date" name="planted_at" id="planted_at" value="{{ old('planted_at') }}" class="mt-1 w-full rounded border px-3 py-2">
        </div>

        <div>
            <label for="image">Foto Tanaman</label>
            <input type="file" name="image" id="image" class="mt-1 w-full rounded border px-3 py-2">
            @error('image')
                <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
            @enderror
        </div>

        <div>
            <label for="notes">Catatan</label>
            <textarea name="notes" id="notes" rows="4" class="mt-1 w-full rounded border px-3 py-2">{{ old('notes') }}</textarea>
        </div>

        <button type="submit" class="rounded bg-emerald-600 px-4 py-2 text-white">
            Simpan
        </button>
    </form>
@endsection
```

## Mengapa `enctype="multipart/form-data"` Wajib

Jika form mengandung file upload, atribut ini wajib ada:

```html
enctype="multipart/form-data"
```

Tanpa atribut tersebut, file tidak akan terkirim dengan benar ke server.

## Membuat Form Blade untuk Laporan

File:

```text
resources/views/reports/create.blade.php
```

Isi sederhana:

```blade
@extends('layouts.app')

@section('title', 'Tambah Laporan')

@section('content')
    <x-page-header
        title="Tambah Laporan"
        subtitle="Masukkan laporan monitoring dan unggah lampiran jika diperlukan."
    />

    <form action="{{ route('reports.store') }}" method="POST" enctype="multipart/form-data" class="space-y-4 rounded-xl bg-white p-6 shadow-sm ring-1 ring-slate-200">
        @csrf

        <div>
            <label for="title">Judul Laporan</label>
            <input type="text" name="title" id="title" value="{{ old('title') }}" class="mt-1 w-full rounded border px-3 py-2">
            @error('title')
                <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
            @enderror
        </div>

        <div>
            <label for="period_start">Periode Awal</label>
            <input type="date" name="period_start" id="period_start" value="{{ old('period_start') }}" class="mt-1 w-full rounded border px-3 py-2">
        </div>

        <div>
            <label for="period_end">Periode Akhir</label>
            <input type="date" name="period_end" id="period_end" value="{{ old('period_end') }}" class="mt-1 w-full rounded border px-3 py-2">
        </div>

        <div>
            <label for="summary">Ringkasan</label>
            <textarea name="summary" id="summary" rows="5" class="mt-1 w-full rounded border px-3 py-2">{{ old('summary') }}</textarea>
        </div>

        <div>
            <label for="attachment">Lampiran</label>
            <input type="file" name="attachment" id="attachment" class="mt-1 w-full rounded border px-3 py-2">
            @error('attachment')
                <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
            @enderror
        </div>

        <button type="submit" class="rounded bg-emerald-600 px-4 py-2 text-white">
            Simpan
        </button>
    </form>
@endsection
```

## Menampilkan Error Validasi

Laravel otomatis mengembalikan user ke form jika validasi gagal. Data lama juga akan dibawa kembali melalui helper `old()`.

Contoh:

```blade
@error('name')
    <p class="text-sm text-red-600">{{ $message }}</p>
@enderror
```

Ini membuat pengalaman pengguna lebih baik karena:

- user tahu field mana yang salah
- user tidak perlu mengisi ulang semua data dari awal

## Menjalankan `storage:link`

Supaya file pada disk `public` dapat diakses melalui browser, jalankan:

```bash
php artisan storage:link
```

Perintah ini membuat symbolic link dari:

```text
storage/app/public
```

ke:

```text
public/storage
```

Tanpa langkah ini, gambar atau lampiran yang sudah diunggah tidak akan mudah diakses dari browser.

## Menampilkan Gambar Tanaman

Contoh pada halaman detail tanaman:

```blade
@if ($plant->image)
    <img
        src="{{ asset('storage/'.$plant->image) }}"
        alt="{{ $plant->name }}"
        class="h-32 w-32 rounded-lg object-cover"
    >
@endif
```

## Menampilkan Link Lampiran Laporan

```blade
@if ($report->attachment)
    <a href="{{ asset('storage/'.$report->attachment) }}" target="_blank" class="text-emerald-700 underline">
        Lihat Lampiran
    </a>
@endif
```

## Mengganti File Lama Saat Update

Saat data diubah, file lama sebaiknya dihapus jika diganti agar storage tidak penuh.

Contoh pada `PlantController@update`:

```php
use Illuminate\Support\Facades\Storage;

public function update(UpdatePlantRequest $request, Plant $plant)
{
    $data = $request->validated();

    if ($request->hasFile('image')) {
        if ($plant->image) {
            Storage::disk('public')->delete($plant->image);
        }

        $data['image'] = $request->file('image')->store('plants', 'public');
    }

    $plant->update($data);

    return redirect()->route('plants.index')->with('success', 'Data tanaman berhasil diperbarui.');
}
```

## Menghapus File Saat Record Dihapus

Contoh:

```php
public function destroy(Plant $plant)
{
    if ($plant->image) {
        Storage::disk('public')->delete($plant->image);
    }

    $plant->delete();

    return redirect()->route('plants.index')->with('success', 'Data tanaman berhasil dihapus.');
}
```

Pendekatan serupa dapat dipakai untuk lampiran laporan.

## Validasi Langsung di Controller vs Form Request

### Validasi langsung di controller

Contoh:

```php
$validated = $request->validate([
    'name' => ['required'],
]);
```

### Validasi dengan Form Request

Contoh:

```php
public function store(StorePlantRequest $request)
```

Untuk praktikum yang mulai kompleks, Form Request lebih disarankan karena:

- lebih rapi
- lebih mudah dibaca
- lebih mudah dipakai ulang

## Praktik Keamanan Dasar pada Upload File

Beberapa hal yang perlu dijaga:

- validasi tipe file
- validasi ukuran file
- jangan percaya nama file dari user
- simpan file pada disk yang benar
- jangan tampilkan file sensitif tanpa kontrol akses bila dibutuhkan

## Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi:

- lupa menambahkan `enctype="multipart/form-data"`
- lupa menjalankan `php artisan storage:link`
- nama field form tidak sama dengan aturan validasi
- lupa menambahkan field upload ke `$fillable`
- file lama tidak dihapus saat update
- memakai validasi terlalu longgar

## Tips Debug

Jika upload atau validasi tidak berjalan:

1. cek error pada browser
2. cek aturan validasi
3. cek nama input form
4. cek folder `storage/app/public`
5. cek apakah symbolic link sudah dibuat
6. cek apakah kolom database untuk path file tersedia

## Pengembangan Lanjutan dari Modul Ini

Setelah modul ini selesai:

- form CRUD akan menjadi lebih matang
- laporan bisa memiliki dokumen pendukung
- halaman tanaman bisa menampilkan foto
- input data siap dipadukan dengan autentikasi role
- fitur pencarian dan paginasi bisa diterapkan pada data yang sudah lengkap

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review CRUD alat dan tanaman dari modul sebelumnya
2. jelaskan alur request form
3. buat Form Request untuk tanaman dan laporan
4. buat form Blade
5. uji validasi gagal
6. uji upload gambar tanaman
7. jalankan `storage:link`
8. tampilkan file hasil upload di browser

## Hasil Akhir Modul

Mahasiswa dapat membuat form input yang aman, menerapkan validasi yang terstruktur, mendukung upload gambar tanaman dan lampiran laporan, serta menampilkan file yang sudah diunggah pada aplikasi greenhouse monitoring.

## Tugas Praktik

1. Tambahkan preview gambar tanaman pada halaman detail.
2. Buat validasi agar `planted_at` tidak boleh lebih besar dari tanggal hari ini.
3. Tambahkan fitur edit laporan dengan penggantian file lampiran.
4. Tambahkan validasi ukuran maksimal gambar tanaman 2 MB dan lampiran 4 MB.
5. Pastikan file lama terhapus saat gambar tanaman diganti.
