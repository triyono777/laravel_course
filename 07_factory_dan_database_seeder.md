# Modul 7: Factory dan Database Seeder

## Deskripsi Modul

Setelah database dan model berhasil dibuat, aplikasi greenhouse monitoring sebenarnya sudah bisa berjalan. Namun pada praktik pengembangan, aplikasi sering kali masih sulit diuji jika tabel kosong. Dashboard tidak menampilkan apa-apa, tabel alat tidak berisi data, dan fitur laporan tidak bisa didemokan dengan baik.

Di sinilah **factory** dan **database seeder** menjadi sangat penting. Factory digunakan untuk membuat data dummy secara otomatis, sedangkan seeder digunakan untuk mengatur proses pengisian data awal secara terstruktur.

Dengan modul ini, mahasiswa akan belajar mengisi database secara cepat sehingga aplikasi greenhouse monitoring bisa langsung diuji tanpa harus memasukkan data satu per satu lewat form.

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan fungsi factory dan database seeder
- membuat data dummy dengan factory
- membuat data awal dengan seeder
- mengisi database secara otomatis
- mengatur urutan seeding agar sesuai relasi tabel
- membuat akun awal untuk role `admin`, `operator`, dan `viewer`
- menyiapkan data uji untuk dashboard greenhouse

## Keterkaitan dengan Studi Kasus

Dashboard greenhouse akan sulit diuji jika tabel masih kosong. Karena itu kita memerlukan data dummy untuk:

- akun pengguna
- alat atau sensor
- tanaman
- histori sensor
- laporan

Data-data ini akan membantu mahasiswa:

- melihat dashboard lebih realistis
- menguji tampilan tabel
- menguji pencarian dan pagination
- menguji autentikasi dengan beberapa role
- menguji query relasi dan laporan

## Apa Itu Factory

Factory adalah class yang bertugas menghasilkan data dummy secara otomatis. Factory sangat berguna saat:

- ingin membuat banyak data uji
- ingin menghindari input manual satu per satu
- ingin mengisi database dengan data yang bervariasi

Laravel memanfaatkan Faker untuk menghasilkan data acak seperti:

- nama
- kalimat
- tanggal
- angka
- email

## Apa Itu Seeder

Seeder adalah class yang digunakan untuk mengatur proses pengisian data ke database.

Seeder cocok dipakai untuk:

- akun default
- data awal sistem
- data uji untuk demo atau praktikum
- kombinasi beberapa factory

## Perbedaan Factory dan Seeder

### Factory

Fokus pada:

- bagaimana satu record dummy dibentuk

Contoh:

- satu alat
- satu tanaman
- satu pembacaan sensor

### Seeder

Fokus pada:

- kapan dan berapa banyak data dimasukkan
- urutan pengisian data

Contoh:

- buat 10 alat
- buat 20 tanaman
- buat 200 histori sensor
- buat 3 akun pengguna awal

## Kapan Factory dan Seeder Dibutuhkan

Pada proyek greenhouse monitoring, factory dan seeder sangat membantu ketika:

- dashboard butuh ringkasan data
- dosen ingin demo cepat tanpa input manual
- mahasiswa ingin menguji query relasi
- mahasiswa ingin menguji fitur login role

## Persiapan Model

Agar factory dapat dipakai dengan baik, model sebaiknya menggunakan trait `HasFactory`.

Contoh pada model `Device`:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Device extends Model
{
    use HasFactory;

    protected $fillable = [
        'code',
        'name',
        'type',
        'location',
        'mqtt_topic',
        'status',
        'last_seen_at',
    ];
}
```

Trait yang sama juga sebaiknya dipakai pada:

- `Plant`
- `SensorReading`
- `Report`
- `User`

Catatan:

- model `User` pada Laravel biasanya sudah memakai `HasFactory`
- jika model lain belum memakai trait ini, factory tidak akan berjalan sebagaimana mestinya

## Langkah Praktik

## 1. Membuat Factory

Jalankan perintah berikut:

```bash
php artisan make:factory DeviceFactory --model=Device
php artisan make:factory PlantFactory --model=Plant
php artisan make:factory SensorReadingFactory --model=SensorReading
php artisan make:factory ReportFactory --model=Report
```

### Penjelasan

- `DeviceFactory` untuk data alat
- `PlantFactory` untuk data tanaman
- `SensorReadingFactory` untuk histori pembacaan sensor
- `ReportFactory` untuk laporan monitoring

## 2. Membuat `DeviceFactory`

File:

```text
database/factories/DeviceFactory.php
```

Isi:

```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class DeviceFactory extends Factory
{
    public function definition(): array
    {
        return [
            'code' => 'GH-' . fake()->unique()->numberBetween(100, 999),
            'name' => fake()->randomElement([
                'Sensor Suhu Barat',
                'Sensor Kelembapan Timur',
                'Controller Irigasi',
                'Gateway MQTT',
            ]),
            'type' => fake()->randomElement(['sensor', 'controller', 'gateway']),
            'location' => 'Blok ' . fake()->randomElement(['A', 'B', 'C']),
            'mqtt_topic' => 'greenhouse/' . fake()->numberBetween(1, 3) . '/sensor',
            'status' => fake()->randomElement(['active', 'inactive', 'maintenance']),
            'last_seen_at' => now()->subMinutes(fake()->numberBetween(1, 60)),
        ];
    }
}
```

### Penjelasan

- `code` dibuat unik agar tidak bentrok
- `name`, `type`, dan `location` dibuat bervariasi
- `mqtt_topic` menyesuaikan skenario perangkat IoT
- `status` dibuat acak agar dashboard punya variasi data

## 3. Membuat `PlantFactory`

File:

```text
database/factories/PlantFactory.php
```

Isi:

```php
namespace Database\Factories;

use App\Models\Device;
use Illuminate\Database\Eloquent\Factories\Factory;

class PlantFactory extends Factory
{
    public function definition(): array
    {
        return [
            'device_id' => Device::inRandomOrder()->value('id'),
            'name' => fake()->randomElement(['Tomat', 'Selada', 'Cabai', 'Mentimun']),
            'variety' => fake()->word(),
            'greenhouse_block' => 'Blok ' . fake()->randomElement(['A', 'B', 'C']),
            'planted_at' => fake()->dateTimeBetween('-60 days', 'now'),
            'notes' => fake()->sentence(),
        ];
    }
}
```

### Penjelasan

- `device_id` mengambil salah satu alat secara acak
- tanaman dibagi ke beberapa blok greenhouse
- tanggal tanam dibuat realistis agar laporan lebih masuk akal

### Catatan penting

Karena `PlantFactory` mengambil `device_id` dari tabel `devices`, maka data `Device` harus dibuat lebih dulu sebelum `Plant`.

## 4. Membuat `SensorReadingFactory`

File:

```text
database/factories/SensorReadingFactory.php
```

Isi:

```php
namespace Database\Factories;

use App\Models\Device;
use Illuminate\Database\Eloquent\Factories\Factory;

class SensorReadingFactory extends Factory
{
    public function definition(): array
    {
        $temperature = fake()->randomFloat(2, 23, 34);
        $humidity = fake()->randomFloat(2, 55, 95);
        $soil = fake()->randomFloat(2, 25, 85);

        return [
            'device_id' => Device::inRandomOrder()->value('id'),
            'temperature' => $temperature,
            'humidity' => $humidity,
            'soil_moisture' => $soil,
            'light_intensity' => fake()->numberBetween(100, 950),
            'payload' => [
                'temperature' => $temperature,
                'humidity' => $humidity,
                'soil_moisture' => $soil,
            ],
            'recorded_at' => now()->subMinutes(fake()->numberBetween(1, 1000)),
        ];
    }
}
```

### Penjelasan

- suhu, kelembapan, dan kelembapan tanah dibuat dengan angka realistis
- `payload` dipakai untuk menyimpan versi mentah data sensor
- `recorded_at` dibuat menyebar ke banyak waktu agar histori lebih bervariasi

## 5. Membuat `ReportFactory`

File:

```text
database/factories/ReportFactory.php
```

Isi:

```php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class ReportFactory extends Factory
{
    public function definition(): array
    {
        $start = fake()->dateTimeBetween('-30 days', '-5 days');
        $end = fake()->dateTimeBetween($start, 'now');

        return [
            'user_id' => User::whereIn('role', ['admin', 'operator'])->inRandomOrder()->value('id'),
            'title' => fake()->randomElement([
                'Laporan Monitoring Mingguan',
                'Laporan Kondisi Tanaman',
                'Laporan Evaluasi Sensor',
                'Laporan Operasional Greenhouse',
            ]),
            'period_start' => $start,
            'period_end' => $end,
            'summary' => fake()->paragraph(),
            'attachment' => null,
        ];
    }
}
```

### Penjelasan

- laporan dibuat oleh `admin` atau `operator`
- judul laporan dibuat bervariasi
- rentang tanggal disiapkan agar sesuai logika periode laporan

## State pada Factory

Factory bisa dibuat lebih fleksibel dengan state. Contoh pada `DeviceFactory`:

```php
public function active(): static
{
    return $this->state(fn () => [
        'status' => 'active',
    ]);
}

public function maintenance(): static
{
    return $this->state(fn () => [
        'status' => 'maintenance',
    ]);
}
```

Contoh penggunaan:

```php
Device::factory()->active()->count(5)->create();
Device::factory()->maintenance()->count(2)->create();
```

State seperti ini membantu ketika dosen ingin mengatur data dummy dengan komposisi tertentu.

## Membuat Seeder

## 1. Membuat `UserSeeder`

Jalankan:

```bash
php artisan make:seeder UserSeeder
```

Isi `database/seeders/UserSeeder.php`:

```php
namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::updateOrCreate(
            ['email' => 'admin@greenhouse.test'],
            [
                'name' => 'Admin Greenhouse',
                'password' => Hash::make('password'),
                'role' => 'admin',
            ]
        );

        User::updateOrCreate(
            ['email' => 'operator@greenhouse.test'],
            [
                'name' => 'Operator Greenhouse',
                'password' => Hash::make('password'),
                'role' => 'operator',
            ]
        );

        User::updateOrCreate(
            ['email' => 'viewer@greenhouse.test'],
            [
                'name' => 'Viewer Greenhouse',
                'password' => Hash::make('password'),
                'role' => 'viewer',
            ]
        );
    }
}
```

### Mengapa memakai `updateOrCreate()`

Karena perintah seeding bisa dijalankan berulang kali. Dengan `updateOrCreate()`:

- akun yang sama tidak digandakan
- data bisa diperbarui bila perlu

## 2. Membuat Seeder Tambahan Jika Diperlukan

Bila ingin memisahkan logika seeding per entitas, mahasiswa juga dapat membuat:

```bash
php artisan make:seeder DeviceSeeder
php artisan make:seeder PlantSeeder
php artisan make:seeder ReportSeeder
```

Namun untuk praktikum dasar, penggunaan factory langsung dari `DatabaseSeeder` sudah cukup.

## 3. Mengatur `DatabaseSeeder`

Perbarui file:

```text
database/seeders/DatabaseSeeder.php
```

Menjadi:

```php
namespace Database\Seeders;

use App\Models\Device;
use App\Models\Plant;
use App\Models\Report;
use App\Models\SensorReading;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call(UserSeeder::class);

        Device::factory()->active()->count(6)->create();
        Device::factory()->maintenance()->count(2)->create();
        Device::factory()->count(2)->create();

        Plant::factory(20)->create();
        SensorReading::factory(200)->create();
        Report::factory(8)->create();
    }
}
```

### Penjelasan urutan

Urutan seeding penting:

1. `UserSeeder` lebih dulu agar laporan punya user pembuat
2. `Device` lebih dulu agar `Plant` dan `SensorReading` bisa mengambil `device_id`
3. `Plant` dan `SensorReading` setelah device tersedia
4. `Report` setelah user tersedia

## Menjalankan Seeder

### Menjalankan semua migration dan seeder dari awal

```bash
php artisan migrate:fresh --seed
```

Perintah ini akan:

- menghapus seluruh tabel
- membuat ulang tabel
- langsung menjalankan `DatabaseSeeder`

### Menjalankan seeder tertentu saja

```bash
php artisan db:seed --class=UserSeeder
```

Ini berguna jika hanya ingin memperbarui akun awal tanpa menghapus seluruh database.

## Contoh Hasil yang Diharapkan

Setelah seeding berjalan, database idealnya berisi:

- 3 akun pengguna utama
- 10 data alat
- 20 data tanaman
- 200 data pembacaan sensor
- 8 laporan

Dengan data ini, halaman berikut sudah bisa diuji lebih nyata:

- dashboard
- daftar alat
- daftar tanaman
- laporan
- autentikasi role

## Cara Mengecek Hasil Seeding

Setelah menjalankan seeder, cek melalui:

- phpMyAdmin
- DBeaver
- SQLite viewer
- `php artisan tinker`

Contoh di Tinker:

```bash
php artisan tinker
```

Lalu:

```php
App\Models\Device::count();
App\Models\Plant::count();
App\Models\SensorReading::count();
App\Models\Report::count();
App\Models\User::select('name', 'email', 'role')->get();
```

## Strategi Data Dummy yang Baik

Data dummy yang baik sebaiknya:

- tidak terlalu seragam
- memiliki variasi status
- memiliki variasi waktu
- cukup realistis untuk studi kasus greenhouse

Jika semua alat aktif dan semua pembacaan sama, dashboard akan kurang berguna untuk demonstrasi.

## Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi:

- lupa menambahkan `HasFactory` pada model
- seeding dijalankan sebelum migration
- `device_id` gagal karena tabel `devices` masih kosong
- laporan gagal dibuat karena user admin atau operator belum ada
- memakai `create()` tanpa memasukkan field ke `$fillable`

## Tips Debug

Jika factory atau seeder gagal:

1. baca error di terminal
2. cek apakah semua migration sudah berjalan
3. cek relasi foreign key
4. cek apakah model punya `HasFactory`
5. cek apakah field yang diisi sesuai dengan tabel
6. cek urutan pembuatan data

## Pengembangan Lanjutan dari Modul Ini

Setelah database berisi data dummy:

- dashboard dapat menampilkan ringkasan data lebih realistis
- relasi antar model dapat diuji
- pencarian dan pagination dapat didemokan
- role-based access dapat diuji dengan akun nyata
- endpoint API nantinya dapat diuji dengan data yang sudah tersedia

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review model dan migration dari modul sebelumnya
2. jelaskan perbedaan factory dan seeder
3. buat factory untuk tiap entitas utama
4. buat `UserSeeder`
5. atur `DatabaseSeeder`
6. jalankan `migrate:fresh --seed`
7. cek hasil di database viewer
8. uji halaman dashboard atau daftar alat

## Hasil Akhir Modul

Database berisi data dummy dan data awal yang cukup untuk pengujian dashboard, tabel CRUD, autentikasi, relasi, pencarian, dan laporan pada aplikasi greenhouse monitoring.

## Tugas Praktik

1. Tambahkan state factory `active()` dan `maintenance()` pada `DeviceFactory`.
2. Buat variasi `PlantFactory` untuk dua jenis tanaman dominan, misalnya tomat dan selada.
3. Tambahkan `ReportFactory` yang menghasilkan judul dan ringkasan laporan monitoring yang lebih beragam.
4. Jalankan `php artisan migrate:fresh --seed`.
5. Uji login menggunakan akun `admin`, `operator`, dan `viewer` setelah seeding selesai.
