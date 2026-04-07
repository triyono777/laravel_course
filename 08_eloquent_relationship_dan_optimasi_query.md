# Modul 8: Eloquent Relationship dan Optimasi Query

## Deskripsi Modul

Pada modul sebelumnya, mahasiswa sudah berhasil menggunakan Eloquent untuk operasi CRUD dasar. Namun pada aplikasi nyata, data jarang berdiri sendiri. Data alat berhubungan dengan histori sensor, tanaman bisa dikaitkan dengan alat tertentu, dan laporan dibuat oleh user tertentu. Jika hubungan antardata ini tidak dikelola dengan benar, kode akan menjadi rumit dan query database bisa berulang-ulang tanpa disadari.

Modul ini membahas dua hal penting:

- cara mendefinisikan relasi antar model Eloquent
- cara mengoptimalkan query agar aplikasi tidak lambat

Pada studi kasus greenhouse monitoring, modul ini sangat penting karena dashboard akan banyak menampilkan data gabungan dari beberapa tabel sekaligus.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan konsep relasi antar model pada Eloquent
- mendefinisikan relasi `hasMany`, `belongsTo`, dan `hasOne`
- menggunakan relasi dalam controller dan view
- memahami masalah N+1 problem
- menggunakan eager loading untuk mengurangi query berulang
- menggunakan `withCount()` dan `select()` untuk optimasi query
- menampilkan data relasional pada dashboard greenhouse

## Keterkaitan dengan Studi Kasus

Dashboard greenhouse perlu menampilkan data seperti:

- alat beserta pembacaan terakhir
- tanaman beserta alat pemantau
- laporan beserta nama pembuatnya
- jumlah tanaman per alat
- jumlah histori sensor per alat

Jika semua data ini diambil secara terpisah tanpa relasi yang baik, aplikasi akan:

- lebih lambat
- lebih sulit dibaca
- lebih sulit dipelihara

Karena itu relasi Eloquent menjadi fondasi penting sebelum masuk ke fitur pencarian, laporan, dan API.

## Mengapa Relasi Dibutuhkan

Secara logika bisnis:

- satu alat dapat menghasilkan banyak pembacaan sensor
- satu alat dapat dikaitkan dengan beberapa tanaman
- satu user dapat membuat banyak laporan

Jika relasi ini ditulis dengan baik, kode menjadi lebih natural. Contoh:

```php
$device->sensorReadings
```

lebih mudah dipahami daripada harus terus menulis query manual berdasarkan `device_id`.

## Jenis Relasi yang Dipakai pada Modul Ini

| Model | Relasi |
| --- | --- |
| `Device` | hasMany `SensorReading`, hasMany `Plant`, hasOne latest reading |
| `Plant` | belongsTo `Device` |
| `SensorReading` | belongsTo `Device` |
| `Report` | belongsTo `User` |
| `User` | hasMany `Report` |

## Persiapan File Model

Pada modul ini, fokus utama adalah melengkapi model yang sudah dibuat pada modul sebelumnya. Jika file model belum tersedia, buat dahulu dengan Artisan:

```bash
php artisan make:model Device
php artisan make:model Plant
php artisan make:model SensorReading
php artisan make:model Report
```

Catatan:

- model `User` sudah tersedia secara bawaan pada Laravel
- jika model sudah pernah dibuat pada modul 5 atau 6, perintah di atas tidak perlu dijalankan lagi
- modul ini lebih banyak berisi pengembangan isi model daripada pembuatan file baru

## Memahami Arah Relasi

Sebelum menulis kode relasi, pahami arah hubungan berikut:

### 1. Device ke SensorReading

- satu alat punya banyak histori sensor
- berarti `Device` memiliki relasi `hasMany`
- `SensorReading` memiliki relasi `belongsTo`

### 2. Device ke Plant

- satu alat bisa dikaitkan dengan banyak tanaman
- berarti `Device` memiliki relasi `hasMany`
- `Plant` memiliki relasi `belongsTo`

### 3. User ke Report

- satu user bisa membuat banyak laporan
- berarti `User` memiliki relasi `hasMany`
- `Report` memiliki relasi `belongsTo`

## Implementasi Relasi pada Model

## 1. Relasi pada Model `Device`

File:

```text
app/Models/Device.php
```

Tambahkan:

```php
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Relations\HasOne;

public function sensorReadings(): HasMany
{
    return $this->hasMany(SensorReading::class);
}

public function plants(): HasMany
{
    return $this->hasMany(Plant::class);
}

public function latestReading(): HasOne
{
    return $this->hasOne(SensorReading::class)->latestOfMany('recorded_at');
}
```

### Penjelasan

- `sensorReadings()` mengambil semua histori sensor milik satu alat
- `plants()` mengambil semua tanaman yang terhubung ke alat tersebut
- `latestReading()` mengambil satu pembacaan sensor terbaru

Relasi `latestReading()` sangat cocok untuk dashboard karena kita sering hanya ingin melihat pembacaan terakhir.

## 2. Relasi pada Model `Plant`

File:

```text
app/Models/Plant.php
```

Tambahkan:

```php
use Illuminate\Database\Eloquent\Relations\BelongsTo;

public function device(): BelongsTo
{
    return $this->belongsTo(Device::class);
}
```

### Penjelasan

Setiap `Plant` mengetahui alat mana yang terkait dengannya melalui `device_id`.

Contoh pemakaian:

```php
$plant->device?->name
```

## 3. Relasi pada Model `SensorReading`

File:

```text
app/Models/SensorReading.php
```

Tambahkan:

```php
use Illuminate\Database\Eloquent\Relations\BelongsTo;

public function device(): BelongsTo
{
    return $this->belongsTo(Device::class);
}
```

### Penjelasan

Setiap histori pembacaan sensor selalu berasal dari satu alat tertentu.

Contoh pemakaian:

```php
$reading->device->code
```

## 4. Relasi pada Model `Report`

File:

```text
app/Models/Report.php
```

Tambahkan:

```php
use Illuminate\Database\Eloquent\Relations\BelongsTo;

public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}
```

## 5. Relasi pada Model `User`

File:

```text
app/Models/User.php
```

Tambahkan:

```php
use Illuminate\Database\Eloquent\Relations\HasMany;

public function reports(): HasMany
{
    return $this->hasMany(Report::class);
}
```

## Contoh Pemakaian Relasi di Controller

Relasi akan terasa manfaatnya ketika dipakai dalam controller.

### Contoh 1: Ambil alat dan pembacaan terbaru

```php
$devices = Device::with('latestReading')->get();
```

### Contoh 2: Ambil tanaman beserta alatnya

```php
$plants = Plant::with('device')->get();
```

### Contoh 3: Ambil laporan beserta pembuatnya

```php
$reports = Report::with('user')->latest()->get();
```

## Menampilkan Relasi di Blade

### Contoh menampilkan alat pada halaman tanaman

```blade
@foreach ($plants as $plant)
    <tr>
        <td>{{ $plant->name }}</td>
        <td>{{ $plant->greenhouse_block }}</td>
        <td>{{ $plant->device?->name ?? 'Belum terhubung' }}</td>
    </tr>
@endforeach
```

### Contoh menampilkan pembuat laporan

```blade
@foreach ($reports as $report)
    <tr>
        <td>{{ $report->title }}</td>
        <td>{{ $report->user->name }}</td>
    </tr>
@endforeach
```

## Apa Itu N+1 Problem

N+1 problem terjadi ketika aplikasi menjalankan satu query utama lalu diikuti banyak query tambahan di dalam loop.

Contoh buruk:

```php
$devices = Device::all();

foreach ($devices as $device) {
    echo $device->latestReading?->temperature;
}
```

Jika ada 50 alat:

- 1 query untuk mengambil semua alat
- 50 query tambahan untuk mengambil latest reading masing-masing alat

Total bisa menjadi 51 query hanya untuk satu halaman.

Inilah yang disebut N+1 problem.

## Mengapa N+1 Problem Berbahaya

Akibatnya:

- halaman menjadi lambat
- server bekerja lebih berat
- performa turun drastis saat data bertambah

Masalah ini sering tidak terasa pada data sedikit, tetapi sangat terasa saat data sudah ratusan atau ribuan.

## Solusi: Eager Loading

Laravel menyediakan eager loading melalui `with()`.

Contoh:

```php
$devices = Device::with(['latestReading', 'plants'])->latest()->get();
```

Dengan eager loading:

- relasi diambil lebih awal
- jumlah query jauh lebih sedikit
- controller lebih efisien

## Query Dashboard yang Lebih Efisien

Contoh `DashboardController@index`:

```php
use App\Models\Device;
use App\Models\Plant;
use App\Models\SensorReading;

public function index()
{
    $summary = [
        'total_devices' => Device::count(),
        'active_devices' => Device::where('status', 'active')->count(),
        'total_plants' => Plant::count(),
        'avg_temperature' => round(SensorReading::avg('temperature'), 2),
    ];

    $devices = Device::with(['latestReading', 'plants'])
        ->latest()
        ->take(5)
        ->get();

    return view('dashboard.index', compact('summary', 'devices'));
}
```

### Mengapa query ini baik

- ringkasan diambil sesuai kebutuhan dashboard
- hanya 5 alat terbaru yang ditampilkan
- relasi `latestReading` dan `plants` diambil sekaligus

## Contoh Tampilan Dashboard

```blade
@foreach ($devices as $device)
    <div class="rounded-lg bg-white p-4 shadow">
        <h3 class="font-semibold">{{ $device->name }}</h3>
        <p>Kode: {{ $device->code }}</p>
        <p>Tanaman terkait: {{ $device->plants->count() }}</p>
        <p>Suhu terakhir: {{ $device->latestReading?->temperature ?? '-' }} C</p>
        <p>Status: {{ $device->status }}</p>
    </div>
@endforeach
```

## Optimasi Tambahan

Selain `with()`, Laravel menyediakan beberapa teknik optimasi lain.

## 1. `withCount()`

Jika hanya ingin jumlah relasi tanpa mengambil seluruh datanya:

```php
$devices = Device::withCount('plants')->get();
```

Contoh pemakaian di Blade:

```blade
<p>Total tanaman: {{ $device->plants_count }}</p>
```

Ini lebih efisien daripada memanggil seluruh data `plants` jika yang dibutuhkan hanya jumlahnya.

## 2. `select()`

Jika hanya membutuhkan beberapa kolom:

```php
$devices = Device::select('id', 'code', 'name', 'status', 'last_seen_at')->get();
```

Ini membantu mengurangi data yang diambil dari database.

## 3. Menggabungkan `with()` dan `withCount()`

```php
$devices = Device::with('latestReading')
    ->withCount('plants')
    ->select('id', 'code', 'name', 'status', 'last_seen_at')
    ->get();
```

Pendekatan ini sangat cocok untuk halaman dashboard ringkas.

## 4. Menambahkan Index pada Kolom Penting

Pada tingkat database, performa bisa ditingkatkan dengan index, terutama untuk kolom yang sering:

- dicari
- diurutkan
- dipakai pada relasi

Contoh kolom penting:

- `devices.code`
- `devices.status`
- `sensor_readings.recorded_at`
- `sensor_readings.device_id`

Index biasanya ditambahkan di migration jika dibutuhkan.

## Contoh Query yang Kurang Efisien vs Lebih Efisien

### Kurang efisien

```php
$devices = Device::all();

foreach ($devices as $device) {
    echo $device->plants->count();
}
```

### Lebih efisien

```php
$devices = Device::withCount('plants')->get();

foreach ($devices as $device) {
    echo $device->plants_count;
}
```

## Kapan Memakai Relasi Khusus Seperti `latestOfMany()`

Pada dashboard greenhouse, kita sering tidak membutuhkan seluruh histori sensor. Yang sering dibutuhkan justru:

- data pembacaan terakhir
- data paling relevan saat ini

Karena itu `latestOfMany()` sangat berguna untuk relasi seperti:

```php
public function latestReading(): HasOne
{
    return $this->hasOne(SensorReading::class)->latestOfMany('recorded_at');
}
```

Ini jauh lebih nyaman daripada harus menulis query manual berulang-ulang di controller.

## Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi:

- lupa import class relasi seperti `HasMany` atau `BelongsTo`
- nama method relasi tidak konsisten
- salah menentukan arah relasi
- memanggil relasi di loop tanpa eager loading
- mengambil semua kolom walau hanya butuh sedikit
- memakai `with()` tapi tetap mengakses relasi yang tidak dimuat

## Tips Debug

Jika relasi tidak berjalan:

1. cek foreign key pada migration
2. cek nama model dan namespace
3. cek method relasi pada model
4. cek apakah data foreign key benar-benar ada
5. cek hasil query di Tinker

Contoh uji di Tinker:

```bash
php artisan tinker
```

Lalu:

```php
$device = App\Models\Device::with(['plants', 'latestReading'])->first();
$device->plants;
$device->latestReading;
```

## Pengembangan Lanjutan dari Modul Ini

Setelah relasi berhasil diterapkan:

- dashboard bisa menampilkan ringkasan yang lebih kaya
- laporan bisa menampilkan nama pembuat
- halaman tanaman bisa menampilkan alat terkait
- pencarian dan pagination bisa dipadukan dengan relasi
- API dapat mengambil data alat beserta pembacaan terakhir dengan lebih rapi

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review struktur tabel dan foreign key
2. jelaskan arah relasi antar tabel
3. lengkapi relasi pada model
4. tampilkan relasi di controller
5. tampilkan relasi di Blade
6. demonstrasikan N+1 problem
7. perbaiki dengan eager loading
8. optimalkan dengan `withCount()`

## Hasil Akhir Modul

Mahasiswa dapat menampilkan data dashboard yang efisien, relasional, dan siap dikembangkan menjadi halaman monitoring greenhouse yang lebih kompleks dan mendekati kebutuhan real-time.

## Tugas Praktik

1. Tampilkan 10 data sensor terbaru beserta nama alatnya.
2. Tambahkan `withCount('sensorReadings')` pada daftar alat.
3. Buat halaman tanaman yang menampilkan nama alat pemantau.
4. Bandingkan hasil query dengan dan tanpa eager loading.
5. Uji relasi `Report` dan `User` pada halaman laporan.
