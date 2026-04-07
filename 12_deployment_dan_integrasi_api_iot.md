# Modul 12: Panduan Deployment ke Server Production dan Integrasi API IoT

## Deskripsi Modul

Setelah seluruh fitur utama aplikasi greenhouse monitoring selesai dibangun, tahap terakhir adalah menyiapkan aplikasi agar dapat digunakan di lingkungan nyata. Pada tahap ini, aplikasi tidak lagi hanya berjalan di komputer lokal, tetapi harus bisa:

- diakses dari server production
- menggunakan database production
- menyajikan asset frontend yang sudah dibuild
- menerima data sensor dari perangkat IoT

Modul ini menggabungkan dua topik besar yang saling berhubungan:

- deployment Laravel ke server production
- integrasi data IoT melalui API yang dihubungkan dari MQTT bridge

Dengan kata lain, modul ini menjadikan aplikasi greenhouse monitoring tidak hanya siap dipakai oleh admin dan operator, tetapi juga siap menerima data dari perangkat fisik.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan perbedaan environment development dan production
- menyiapkan konfigurasi Laravel untuk production
- melakukan deployment aplikasi ke hosting atau server
- menghubungkan aplikasi ke database production
- mengoptimalkan Laravel untuk server production
- mendesain endpoint API untuk integrasi IoT
- menyiapkan middleware keamanan untuk endpoint ingest data
- mensimulasikan alur data dari MQTT bridge ke Laravel API

## Keterkaitan dengan Studi Kasus

Setelah aplikasi greenhouse monitoring selesai dikembangkan, sistem harus bisa:

- diakses oleh admin, operator, dan viewer di server production
- menerima data sensor dari perangkat IoT
- menyimpan histori pembacaan sensor ke database
- menampilkan data terbaru di dashboard

Artinya, hasil akhir sistem bukan hanya web CRUD, tetapi sebuah aplikasi monitoring yang terhubung ke perangkat sensor.

## Gambaran Besar Arsitektur Akhir

Secara umum, arsitektur aplikasi akhir akan seperti ini:

```text
Admin/Operator/Viewer -> Laravel Web App -> MySQL Production
Sensor IoT -> MQTT Broker -> MQTT Bridge -> Laravel API -> MySQL Production -> Dashboard
```

Komponen pentingnya adalah:

- aplikasi web Laravel
- database MySQL production
- broker MQTT
- bridge yang mengubah data MQTT menjadi HTTP request ke Laravel API

## Mengapa Deployment Penting

Selama pengembangan lokal:

- aplikasi hanya bisa diakses dari komputer sendiri
- alamat biasanya `127.0.0.1:8000`
- data tersimpan di database lokal

Namun pada lingkungan nyata:

- aplikasi harus tersedia untuk pengguna lain
- URL harus stabil
- server harus lebih aman
- konfigurasi tidak boleh sama dengan mode development

Karena itu deployment bukan sekadar upload file, tetapi juga penyesuaian environment dan optimasi aplikasi.

## Perbedaan Development dan Production

### Development

Ciri umumnya:

- `APP_ENV=local`
- `APP_DEBUG=true`
- route dan config belum di-cache
- asset sering dijalankan dengan `npm run dev`

### Production

Ciri umumnya:

- `APP_ENV=production`
- `APP_DEBUG=false`
- asset hasil build production
- route dan config di-cache
- error tidak ditampilkan secara detail ke pengguna

## Persiapan Sebelum Deployment

Sebelum melakukan deployment, pastikan hal berikut sudah siap:

- aplikasi dapat berjalan stabil di lokal
- migration sudah benar
- role user sudah diuji
- upload file sudah diuji
- endpoint API dasar sudah dipahami
- tidak ada data dummy sensitif yang ikut dibawa ke production

## Konfigurasi Environment Production

Atur file `.env` production:

```env
APP_NAME="Greenhouse Monitoring"
APP_ENV=production
APP_KEY=base64:generated-key
APP_DEBUG=false
APP_URL=https://greenhouse-kampus.ac.id

LOG_CHANNEL=stack
LOG_LEVEL=error

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=greenhouse_prod
DB_USERNAME=greenhouse_user
DB_PASSWORD=password-kuat

FILESYSTEM_DISK=public
SESSION_DRIVER=file
CACHE_STORE=file
QUEUE_CONNECTION=database

IOT_INGEST_KEY=greenhouse-secret-key
```

## Penjelasan Konfigurasi Penting

- `APP_DEBUG=false`: error detail tidak ditampilkan ke pengguna
- `APP_URL`: URL utama aplikasi production
- `DB_*`: koneksi ke database production
- `QUEUE_CONNECTION`: berguna jika nantinya pemrosesan data sensor dibuat asynchronous
- `IOT_INGEST_KEY`: secret key untuk melindungi endpoint ingest data IoT

## Build Asset Frontend

Sebelum deployment, asset frontend harus dibuild:

```bash
npm install
npm run build
```

Perintah ini menghasilkan asset production yang lebih siap dipakai server.

Berbeda dengan `npm run dev`, hasil build production:

- lebih ringan
- sudah dioptimasi
- tidak bergantung pada Vite dev server

## Optimasi Laravel untuk Production

Jalankan:

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache
```

Atau gunakan:

```bash
php artisan optimize
```

### Fungsi optimasi ini

- mempercepat loading konfigurasi
- mempercepat pemanggilan route
- mempercepat rendering view
- mengurangi beban runtime

Jika ada perubahan route atau config di production, cache ini perlu dibersihkan dan dibangun ulang.

## Menjalankan Migration di Production

Gunakan:

```bash
php artisan migrate --force
```

Flag `--force` dibutuhkan karena Laravel akan meminta konfirmasi saat environment production aktif.

Jika aplikasi juga memerlukan data awal:

```bash
php artisan db:seed --force
```

Namun untuk production, seeding biasanya dibatasi pada data penting saja seperti akun admin awal atau data master tertentu.

## Deployment ke cPanel atau Shared Hosting

Shared hosting masih sering digunakan dalam konteks pembelajaran. Langkah umumnya:

1. upload source code ke server
2. letakkan isi folder `public` ke `public_html`
3. letakkan sisa proyek di luar `public_html`
4. sesuaikan `index.php` agar path `vendor` dan `bootstrap` mengarah ke lokasi yang benar
5. impor database production
6. set `.env` production
7. jalankan migrasi bila akses terminal tersedia
8. buat symbolic link storage bila diizinkan hosting

## Contoh Penyesuaian `public/index.php`

Misalnya struktur hosting seperti ini:

```text
/home/username/greenhouse-app
/home/username/public_html
```

Maka file `public_html/index.php` dapat disesuaikan:

```php
require __DIR__.'/../greenhouse-app/vendor/autoload.php';

$app = require_once __DIR__.'/../greenhouse-app/bootstrap/app.php';
```

Sesuaikan nama folder dengan lokasi proyek Anda.

## Deployment ke VPS atau Cloud Server

Jika memakai VPS, alur umumnya lebih fleksibel dan lebih ideal untuk aplikasi jangka panjang.

Contoh langkah:

```bash
git pull origin main
composer install --no-dev --optimize-autoloader
npm install
npm run build
php artisan migrate --force
php artisan optimize
```

Jika ada upload file dan cache:

```bash
php artisan storage:link
php artisan queue:restart
```

## Komponen Tambahan yang Umum pada VPS

Pada VPS, biasanya aplikasi Laravel didukung oleh:

- Nginx atau Apache
- PHP-FPM
- MySQL atau MariaDB
- Supervisor untuk queue worker
- cron untuk scheduler
- SSL certificate

Untuk praktikum, penjelasan detail konfigurasi web server bisa diberikan sebagai pengayaan.

## Checklist Setelah Deployment

Pastikan hal berikut berjalan:

- halaman login dapat dibuka
- proses login berhasil
- dashboard tampil
- gambar tanaman dapat diakses
- tabel CRUD berfungsi
- role-based access masih berjalan benar
- API menerima data sensor
- database production terisi sesuai harapan

## Pengujian Fungsional Setelah Deployment

Urutan pengujian yang disarankan:

1. buka halaman utama
2. login sebagai admin
3. cek menu user, alat, tanaman, dan laporan
4. unggah satu gambar tanaman
5. buka file hasil upload
6. login sebagai operator
7. pastikan operator tidak bisa mengakses manajemen user
8. uji endpoint API sensor

## Bagian Integrasi IoT

Laravel sebaiknya menerima data IoT melalui HTTP API. Perangkat atau broker MQTT dapat meneruskan data melalui bridge seperti:

- Node-RED
- Python subscriber
- service kecil berbasis Express atau FastAPI

Mengapa memakai bridge:

- banyak perangkat publish ke MQTT, bukan langsung HTTP
- Laravel lebih nyaman menerima data terstruktur melalui API
- bridge membantu memisahkan komunikasi perangkat dan aplikasi web

## Menyiapkan File Backend untuk API IoT

Sebelum menulis endpoint API, siapkan file yang diperlukan dengan Artisan:

```bash
php artisan make:controller Api/IotReadingController
php artisan make:controller Api/DeviceStatusController
php artisan make:middleware VerifyIotKey
php artisan make:request StoreIotReadingRequest
```

Catatan:

- `IotReadingController` dipakai untuk menerima data dari bridge MQTT
- `DeviceStatusController` dipakai untuk menyediakan data pembacaan terakhir alat
- `VerifyIotKey` dipakai untuk mengamankan endpoint ingest data
- `StoreIotReadingRequest` dipakai agar validasi request lebih rapi

## Mendesain Endpoint API

Contoh route pada `routes/api.php`:

```php
use App\Http\Controllers\Api\DeviceStatusController;
use App\Http\Controllers\Api\IotReadingController;
use Illuminate\Support\Facades\Route;

Route::middleware('iot.key')->group(function () {
    Route::post('/iot/readings', [IotReadingController::class, 'store']);
});

Route::get('/devices/{device:code}/latest', [DeviceStatusController::class, 'show']);
Route::get('/devices/{device:code}/readings', [DeviceStatusController::class, 'history']);
```

### Penjelasan endpoint

- `POST /api/iot/readings`: menerima data sensor dari bridge
- `GET /api/devices/{code}/latest`: mengambil pembacaan terakhir alat
- `GET /api/devices/{code}/readings`: mengambil histori pembacaan alat

## Contoh Payload dari Bridge MQTT

```json
{
  "device_code": "GH-101",
  "temperature": 28.5,
  "humidity": 77.2,
  "soil_moisture": 51.8,
  "light_intensity": 620,
  "recorded_at": "2026-04-07 09:30:00"
}
```

Payload seperti ini cukup umum untuk bridge yang menerima data sensor lalu meneruskannya ke API Laravel.

## Validasi Request dengan Form Request

File:

```text
app/Http/Requests/StoreIotReadingRequest.php
```

Isi:

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreIotReadingRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'device_code' => ['required', 'exists:devices,code'],
            'temperature' => ['nullable', 'numeric'],
            'humidity' => ['nullable', 'numeric'],
            'soil_moisture' => ['nullable', 'numeric'],
            'light_intensity' => ['nullable', 'integer'],
            'recorded_at' => ['required', 'date'],
        ];
    }
}
```

## Contoh Controller API

File:

```text
app/Http/Controllers/Api/IotReadingController.php
```

Isi:

```php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreIotReadingRequest;
use App\Models\Device;
use App\Models\SensorReading;

class IotReadingController extends Controller
{
    public function store(StoreIotReadingRequest $request)
    {
        $validated = $request->validated();

        $device = Device::where('code', $validated['device_code'])->firstOrFail();

        $reading = SensorReading::create([
            'device_id' => $device->id,
            'temperature' => $validated['temperature'] ?? null,
            'humidity' => $validated['humidity'] ?? null,
            'soil_moisture' => $validated['soil_moisture'] ?? null,
            'light_intensity' => $validated['light_intensity'] ?? null,
            'payload' => $validated,
            'recorded_at' => $validated['recorded_at'],
        ]);

        $device->update([
            'status' => 'active',
            'last_seen_at' => $validated['recorded_at'],
        ]);

        return response()->json([
            'message' => 'Pembacaan sensor berhasil disimpan.',
            'data' => $reading,
        ], 201);
    }
}
```

## Endpoint untuk Data Terbaru dan Histori

File:

```text
app/Http/Controllers/Api/DeviceStatusController.php
```

Isi sederhana:

```php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Device;

class DeviceStatusController extends Controller
{
    public function show(Device $device)
    {
        $device->load('latestReading');

        return response()->json([
            'device' => $device,
            'latest_reading' => $device->latestReading,
        ]);
    }

    public function history(Device $device)
    {
        $readings = $device->sensorReadings()
            ->latest('recorded_at')
            ->take(20)
            ->get();

        return response()->json([
            'device' => $device->only(['code', 'name', 'status']),
            'readings' => $readings,
        ]);
    }
}
```

## Mengamankan API IoT

Untuk integrasi perangkat, pendekatan sederhana adalah memakai secret key pada header:

```text
X-IOT-KEY: greenhouse-secret-key
```

Tambahkan di `.env`:

```env
IOT_INGEST_KEY=greenhouse-secret-key
```

## Contoh Middleware `VerifyIotKey`

File:

```text
app/Http/Middleware/VerifyIotKey.php
```

Isi:

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class VerifyIotKey
{
    public function handle(Request $request, Closure $next)
    {
        $key = $request->header('X-IOT-KEY');

        if (! $key || $key !== config('app.iot_ingest_key', env('IOT_INGEST_KEY'))) {
            return response()->json([
                'message' => 'Unauthorized IoT request.',
            ], 401);
        }

        return $next($request);
    }
}
```

Lalu alias-kan middleware ini, misalnya dengan nama `iot.key`.

## Mengapa Endpoint IoT Harus Diamankan

Tanpa proteksi:

- siapa pun bisa mengirim data palsu
- histori sensor bisa rusak
- dashboard menjadi tidak valid

Untuk praktikum, secret key header sudah cukup sebagai dasar keamanan.

## Contoh Pengiriman Data dengan `curl`

```bash
curl -X POST https://greenhouse-kampus.ac.id/api/iot/readings \
  -H "Content-Type: application/json" \
  -H "X-IOT-KEY: greenhouse-secret-key" \
  -d '{
    "device_code":"GH-101",
    "temperature":28.5,
    "humidity":77.2,
    "soil_moisture":51.8,
    "light_intensity":620,
    "recorded_at":"2026-04-07 09:30:00"
  }'
```

Jika berhasil, Laravel akan mengembalikan respons JSON `201 Created`.

## Contoh Alur MQTT Bridge

Urutan kerjanya:

1. perangkat publish ke topik `greenhouse/1/sensor`
2. bridge subscribe ke broker MQTT
3. bridge membaca payload sensor
4. bridge mengirim payload ke API Laravel
5. Laravel memvalidasi dan menyimpan ke `sensor_readings`
6. dashboard menampilkan pembacaan terbaru

## Contoh Peran Bridge

Bridge bisa ditulis dengan:

- Node-RED flow
- Python script dengan subscriber MQTT
- service kecil berbasis Express atau FastAPI

Tugas utamanya adalah:

- subscribe ke MQTT
- mengubah pesan menjadi format JSON API
- menambahkan header keamanan
- mengirim request ke Laravel

## Pengujian Integrasi IoT

Pengujian minimal yang disarankan:

1. pastikan `device_code` memang ada di tabel `devices`
2. kirim request pakai `curl` atau Postman
3. cek apakah data masuk ke `sensor_readings`
4. cek apakah `last_seen_at` pada device ter-update
5. buka dashboard dan endpoint latest reading

## Troubleshooting Umum Deployment

Beberapa masalah yang sering terjadi:

- `.env` production salah
- `APP_DEBUG` masih `true`
- permission folder `storage` dan `bootstrap/cache` bermasalah
- asset belum dibuild
- symbolic link storage belum dibuat
- database production belum sesuai

## Troubleshooting Umum API IoT

Beberapa masalah yang sering terjadi:

- `device_code` tidak ada di database
- header `X-IOT-KEY` tidak dikirim
- format tanggal salah
- payload tidak sesuai aturan validasi
- route API belum benar

## Rekomendasi Lanjutan

- tambahkan queue jika trafik sensor tinggi
- simpan log request IoT untuk audit
- buat alert jika suhu atau kelembapan melewati ambang batas
- pertimbangkan token per device jika jumlah perangkat banyak
- pertimbangkan job asynchronous bila bridge mengirim data sangat sering

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review fitur lengkap aplikasi greenhouse
2. jelaskan perbedaan local dan production
3. siapkan `.env` production
4. build asset dan optimasi Laravel
5. bahas deployment ke hosting atau VPS
6. buat endpoint API IoT
7. tambahkan middleware keamanan
8. uji kirim data dari Postman atau `curl`
9. cek hasil data di dashboard

## Hasil Akhir Modul

Mahasiswa memahami proses deployment aplikasi Laravel ke production dan mampu menyiapkan API agar sistem greenhouse dapat terhubung dengan ekosistem IoT berbasis MQTT bridge secara lebih realistis dan aman.

## Tugas Praktik

1. Buat middleware `VerifyIotKey`.
2. Tambahkan endpoint untuk mengambil 20 data sensor terbaru per alat.
3. Simulasikan pengiriman data dari Postman atau `curl`.
4. Buat checklist deployment production untuk aplikasi greenhouse.
5. Jelaskan alur data dari sensor sampai tampil di dashboard dalam 5 sampai 8 langkah.
