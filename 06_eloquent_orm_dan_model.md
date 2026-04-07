# Modul 6: Eloquent ORM dan Model

## Deskripsi Modul

Pada modul sebelumnya, mahasiswa sudah membuat struktur database menggunakan migration. Namun tabel yang sudah dibuat belum otomatis berguna jika aplikasi belum memiliki cara yang rapi untuk membaca, menyimpan, memperbarui, dan menghapus data. Pada titik inilah **Eloquent ORM** berperan.

Eloquent adalah ORM bawaan Laravel yang memungkinkan tabel database diperlakukan sebagai object PHP. Dengan Eloquent, mahasiswa tidak perlu selalu menulis query SQL mentah untuk operasi dasar seperti:

- mengambil data
- menyimpan data
- memperbarui data
- menghapus data

Modul ini akan mengubah halaman greenhouse monitoring dari yang sebelumnya memakai data dummy menjadi berbasis data sungguhan dari database, terutama untuk fitur:

- manajemen alat
- manajemen tanaman

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan konsep ORM dan fungsi Eloquent
- membuat dan mengonfigurasi model Eloquent
- menggunakan model untuk operasi CRUD dasar
- memahami mass assignment dan proteksinya
- membuat resource controller
- menghubungkan route resource dengan controller
- menampilkan data database pada view Blade
- menggunakan route model binding

## Keterkaitan dengan Studi Kasus

Pada proyek greenhouse monitoring, data yang akan mulai dikelola dari database pada modul ini adalah:

- alat atau sensor pada tabel `devices`
- tanaman pada tabel `plants`

Dengan demikian, aplikasi tidak lagi hanya menampilkan array dummy yang ditulis di controller, melainkan membaca data dari tabel hasil migration.

## Apa Itu ORM

ORM adalah singkatan dari **Object Relational Mapping**. Konsep ini menghubungkan tabel database dengan object pada bahasa pemrograman.

Dalam Laravel:

- satu model biasanya mewakili satu tabel
- satu baris data menjadi satu object model
- kolom tabel menjadi atribut object

Contoh:

- model `Device` mewakili tabel `devices`
- model `Plant` mewakili tabel `plants`

## Mengapa Eloquent Penting

Tanpa Eloquent, mahasiswa perlu menulis query SQL untuk hampir setiap operasi. Dengan Eloquent, kode menjadi lebih mudah dibaca dan lebih dekat dengan logika aplikasi.

Contoh sederhana:

```php
$devices = Device::all();
```

Kode di atas jauh lebih ringkas dibanding menulis query SQL mentah untuk mengambil seluruh data alat.

## Model yang Digunakan pada Modul Ini

- `Device`
- `Plant`
- `SensorReading`
- `Report`
- `User`

Pada praktik inti modul ini, fokus utama ada pada:

- `Device`
- `Plant`

## Menyiapkan Model

Jika model belum tersedia, buat dengan Artisan:

```bash
php artisan make:model Device
php artisan make:model Plant
php artisan make:model SensorReading
php artisan make:model Report
```

Namun jika model sudah dibuat pada modul migration dengan perintah `make:model -m`, maka file tersebut tinggal dilengkapi.

## Anatomi Dasar Model Eloquent

Model Laravel biasanya disimpan pada:

```text
app/Models
```

Contoh struktur dasar model:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Device extends Model
{
    //
}
```

## Konfigurasi Model `Device`

File:

```text
app/Models/Device.php
```

Isi:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Device extends Model
{
    protected $fillable = [
        'code',
        'name',
        'type',
        'location',
        'mqtt_topic',
        'status',
        'last_seen_at',
    ];

    protected $casts = [
        'last_seen_at' => 'datetime',
    ];
}
```

### Penjelasan

- `$fillable` menentukan field mana yang boleh diisi secara massal
- `$casts` mengubah format data tertentu agar otomatis diperlakukan sesuai tipe yang diinginkan

Pada contoh ini, `last_seen_at` diperlakukan sebagai tanggal dan waktu.

## Konfigurasi Model `Plant`

File:

```text
app/Models/Plant.php
```

Isi:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Plant extends Model
{
    protected $fillable = [
        'device_id',
        'name',
        'variety',
        'greenhouse_block',
        'planted_at',
        'image',
        'notes',
    ];

    protected $casts = [
        'planted_at' => 'date',
    ];
}
```

### Penjelasan

- `device_id` nanti menghubungkan tanaman dengan alat
- `planted_at` di-cast menjadi `date` agar lebih mudah diolah

## Konsep Mass Assignment

Mass assignment adalah proses mengisi banyak field sekaligus melalui array, misalnya:

```php
Device::create($validated);
```

Jika `$fillable` tidak diatur, Laravel akan melindungi model dari pengisian field yang tidak diizinkan.

Ini penting untuk mencegah field sensitif diisi sembarangan.

## `$fillable` vs `$guarded`

Ada dua pendekatan umum:

### Menggunakan `$fillable`

```php
protected $fillable = ['code', 'name', 'type'];
```

Artinya hanya field itu yang boleh diisi massal.

### Menggunakan `$guarded`

```php
protected $guarded = [];
```

Artinya semua field boleh diisi massal.

Untuk praktikum ini, penggunaan `$fillable` lebih disarankan karena lebih aman dan lebih jelas bagi mahasiswa pemula.

## Membuat Resource Controller

Karena kita akan membuat CRUD alat dan tanaman, gunakan resource controller:

```bash
php artisan make:controller DeviceController --resource
php artisan make:controller PlantController --resource
```

Perintah ini membuat controller dengan method bawaan seperti:

- `index`
- `create`
- `store`
- `show`
- `edit`
- `update`
- `destroy`

## Menambahkan Route Resource

Pada `routes/web.php`:

```php
use App\Http\Controllers\DeviceController;
use App\Http\Controllers\PlantController;

Route::resource('devices', DeviceController::class);
Route::resource('plants', PlantController::class);
```

## Apa Keuntungan Route Resource

Dengan satu baris route resource, Laravel otomatis membuat sekumpulan route CRUD standar.

Contoh route yang dihasilkan:

- `GET /devices`
- `GET /devices/create`
- `POST /devices`
- `GET /devices/{device}`
- `GET /devices/{device}/edit`
- `PUT/PATCH /devices/{device}`
- `DELETE /devices/{device}`

Mahasiswa dapat melihatnya lewat:

```bash
php artisan route:list
```

## Operasi CRUD Dasar dengan Eloquent

## 1. Create

Contoh menyimpan data alat pada `DeviceController@store`:

```php
use App\Models\Device;
use Illuminate\Http\Request;

public function store(Request $request)
{
    $validated = $request->validate([
        'code' => ['required', 'string', 'max:50', 'unique:devices,code'],
        'name' => ['required', 'string', 'max:100'],
        'type' => ['required', 'string', 'max:50'],
        'location' => ['required', 'string', 'max:100'],
        'mqtt_topic' => ['nullable', 'string', 'max:150'],
        'status' => ['required', 'in:active,inactive,maintenance'],
    ]);

    Device::create($validated);

    return redirect()
        ->route('devices.index')
        ->with('success', 'Data alat berhasil ditambahkan.');
}
```

### Penjelasan

- request divalidasi terlebih dahulu
- hasil validasi disimpan ke `$validated`
- `Device::create()` menyimpan data ke database
- setelah sukses, user diarahkan ke halaman daftar alat

## 2. Read

Contoh menampilkan semua data alat pada `DeviceController@index`:

```php
public function index()
{
    $devices = Device::latest()->get();

    return view('devices.index', compact('devices'));
}
```

### Penjelasan

- `latest()` mengurutkan data terbaru di atas
- `get()` mengambil seluruh hasil query
- data dikirim ke view untuk ditampilkan

## 3. Update

Contoh memperbarui data tanaman pada `PlantController@update`:

```php
use App\Models\Plant;
use Illuminate\Http\Request;

public function update(Request $request, Plant $plant)
{
    $validated = $request->validate([
        'name' => ['required', 'string', 'max:100'],
        'variety' => ['nullable', 'string', 'max:100'],
        'greenhouse_block' => ['required', 'string', 'max:50'],
        'planted_at' => ['nullable', 'date'],
        'notes' => ['nullable', 'string'],
    ]);

    $plant->update($validated);

    return redirect()
        ->route('plants.index')
        ->with('success', 'Data tanaman berhasil diperbarui.');
}
```

### Penjelasan

- objek `Plant $plant` diperoleh melalui route model binding
- method `update()` mengubah data berdasarkan hasil validasi

## 4. Delete

Contoh menghapus data alat:

```php
public function destroy(Device $device)
{
    $device->delete();

    return redirect()
        ->route('devices.index')
        ->with('success', 'Data alat berhasil dihapus.');
}
```

## Mengisi Method pada Resource Controller

## Contoh `DeviceController`

Potongan struktur minimal:

```php
namespace App\Http\Controllers;

use App\Models\Device;
use Illuminate\Http\Request;

class DeviceController extends Controller
{
    public function index()
    {
        $devices = Device::latest()->get();
        return view('devices.index', compact('devices'));
    }

    public function create()
    {
        return view('devices.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'code' => ['required', 'string', 'max:50', 'unique:devices,code'],
            'name' => ['required', 'string', 'max:100'],
            'type' => ['required', 'string', 'max:50'],
            'location' => ['required', 'string', 'max:100'],
            'mqtt_topic' => ['nullable', 'string', 'max:150'],
            'status' => ['required', 'in:active,inactive,maintenance'],
        ]);

        Device::create($validated);

        return redirect()->route('devices.index');
    }

    public function show(Device $device)
    {
        return view('devices.show', compact('device'));
    }

    public function edit(Device $device)
    {
        return view('devices.edit', compact('device'));
    }

    public function update(Request $request, Device $device)
    {
        $validated = $request->validate([
            'code' => ['required', 'string', 'max:50', 'unique:devices,code,'.$device->id],
            'name' => ['required', 'string', 'max:100'],
            'type' => ['required', 'string', 'max:50'],
            'location' => ['required', 'string', 'max:100'],
            'mqtt_topic' => ['nullable', 'string', 'max:150'],
            'status' => ['required', 'in:active,inactive,maintenance'],
        ]);

        $device->update($validated);

        return redirect()->route('devices.index');
    }

    public function destroy(Device $device)
    {
        $device->delete();

        return redirect()->route('devices.index');
    }
}
```

## Route Model Binding

Route model binding adalah fitur Laravel yang otomatis mengambil data model berdasarkan parameter route.

Contoh route resource:

```text
/plants/5/edit
```

Jika method controller ditulis seperti ini:

```php
public function edit(Plant $plant)
```

Laravel akan otomatis mencari data `Plant` dengan ID `5`.

### Keuntungan route model binding

- kode lebih singkat
- tidak perlu query manual seperti `Plant::findOrFail($id)`
- lebih konsisten

## Contoh Query Manual vs Route Model Binding

### Tanpa binding

```php
public function edit($id)
{
    $plant = Plant::findOrFail($id);

    return view('plants.edit', compact('plant'));
}
```

### Dengan binding

```php
public function edit(Plant $plant)
{
    return view('plants.edit', compact('plant'));
}
```

Pendekatan kedua lebih ringkas dan lebih sesuai dengan Laravel modern.

## Contoh View Index Sederhana

File:

```text
resources/views/devices/index.blade.php
```

Isi:

```blade
@extends('layouts.app')

@section('title', 'Data Alat')

@section('content')
    <x-page-header
        title="Data Alat"
        subtitle="Daftar perangkat greenhouse yang tersimpan di database."
    >
        <x-slot:actions>
            <a href="{{ route('devices.create') }}" class="rounded bg-emerald-600 px-4 py-2 text-sm font-medium text-white">
                Tambah Alat
            </a>
        </x-slot:actions>
    </x-page-header>

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
                            Belum ada data alat.
                        </td>
                    </tr>
                @endforelse
            </tbody>
        </table>
    </div>
@endsection
```

## Contoh Form Tambah Alat

File:

```text
resources/views/devices/create.blade.php
```

Isi sederhana:

```blade
@extends('layouts.app')

@section('title', 'Tambah Alat')

@section('content')
    <x-page-header
        title="Tambah Alat"
        subtitle="Masukkan data perangkat greenhouse baru."
    />

    <form action="{{ route('devices.store') }}" method="POST" class="space-y-4 rounded-xl bg-white p-6 shadow-sm ring-1 ring-slate-200">
        @csrf

        <div>
            <label for="code">Kode</label>
            <input type="text" name="code" id="code" value="{{ old('code') }}" class="mt-1 w-full rounded border px-3 py-2">
            @error('code')
                <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
            @enderror
        </div>

        <div>
            <label for="name">Nama Alat</label>
            <input type="text" name="name" id="name" value="{{ old('name') }}" class="mt-1 w-full rounded border px-3 py-2">
            @error('name')
                <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
            @enderror
        </div>

        <div>
            <label for="type">Tipe</label>
            <input type="text" name="type" id="type" value="{{ old('type') }}" class="mt-1 w-full rounded border px-3 py-2">
        </div>

        <div>
            <label for="location">Lokasi</label>
            <input type="text" name="location" id="location" value="{{ old('location') }}" class="mt-1 w-full rounded border px-3 py-2">
        </div>

        <div>
            <label for="mqtt_topic">MQTT Topic</label>
            <input type="text" name="mqtt_topic" id="mqtt_topic" value="{{ old('mqtt_topic') }}" class="mt-1 w-full rounded border px-3 py-2">
        </div>

        <div>
            <label for="status">Status</label>
            <select name="status" id="status" class="mt-1 w-full rounded border px-3 py-2">
                <option value="inactive">Inactive</option>
                <option value="active">Active</option>
                <option value="maintenance">Maintenance</option>
            </select>
        </div>

        <button type="submit" class="rounded bg-emerald-600 px-4 py-2 text-white">
            Simpan
        </button>
    </form>
@endsection
```

## Alur CRUD dengan Eloquent

Secara umum, alur CRUD pada Laravel adalah:

1. user membuka halaman form
2. route memanggil method controller
3. controller menampilkan view form
4. user mengirim data
5. controller memvalidasi request
6. model menyimpan data ke database
7. user diarahkan kembali ke halaman index atau detail

## Penggunaan Data Dummy vs Database

Pada modul sebelumnya, data alat mungkin masih ditulis seperti ini:

```php
$devices = [
    ['code' => 'GH-01', 'name' => 'Sensor Utama'],
];
```

Pada modul ini, pendekatan tersebut diganti menjadi:

```php
$devices = Device::latest()->get();
```

Ini adalah perubahan penting dari aplikasi demo menjadi aplikasi yang benar-benar dinamis.

## Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi:

- lupa menambahkan `$fillable`
- lupa menambahkan `use App\Models\Device;`
- view mengakses data object dengan sintaks array
- validasi `unique` salah saat update
- method resource controller belum lengkap
- route resource sudah dibuat tetapi view yang dipanggil belum ada

## Tips Debug

Jika data tidak tersimpan atau tidak tampil:

1. cek apakah migration sudah dijalankan
2. cek apakah tabel benar-benar ada
3. cek nama field form dan nama kolom tabel
4. cek aturan validasi
5. cek apakah `$fillable` sudah benar
6. cek hasil `php artisan route:list`

## Pengembangan Lanjutan dari Modul Ini

Setelah mahasiswa memahami Eloquent dasar:

- relasi antar model akan diperdalam pada modul berikutnya
- data dummy akan diganti penuh oleh data database
- dashboard akan mulai menampilkan data dari query Eloquent
- factory dan seeder akan digunakan untuk mengisi data uji

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review struktur tabel dari modul migration
2. jelaskan apa itu ORM dan model
3. lengkapi model `Device` dan `Plant`
4. buat resource controller
5. hubungkan route resource
6. buat form tambah data alat
7. uji penyimpanan data ke database
8. tampilkan data pada halaman index

## Hasil Akhir Modul

Mahasiswa mampu mengelola data alat dan tanaman menggunakan model Eloquent, validasi dasar, resource controller, serta route model binding. Aplikasi greenhouse mulai beralih dari data dummy menjadi berbasis database sungguhan.

## Tugas Praktik

1. Lengkapi method `create`, `edit`, dan `destroy` pada `DeviceController`.
2. Buat halaman CRUD dasar untuk `Plant`.
3. Tambahkan field `last_seen_at` agar dapat diubah dari form edit alat.
4. Uji `show`, `edit`, dan `destroy` menggunakan route resource.
5. Buat ringkasan singkat perbedaan data dummy dan data database pada proyek greenhouse.
