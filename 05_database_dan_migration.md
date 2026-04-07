# Modul 5: Database dan Migration

## Deskripsi Modul

Setelah mahasiswa memahami alur route, controller, view, dan tampilan dengan Blade, langkah berikutnya adalah membuat fondasi data aplikasi. Pada tahap ini, aplikasi greenhouse monitoring belum cukup jika hanya memakai data dummy di controller. Kita membutuhkan database agar data dapat disimpan, diubah, dihapus, dan ditampilkan kembali secara konsisten.

Modul ini membahas bagaimana Laravel mengelola struktur database menggunakan **migration**. Migration dapat dipahami sebagai cara Laravel menyimpan riwayat perubahan struktur tabel dalam bentuk kode PHP, sehingga pembuatan database tidak perlu dilakukan secara manual lewat SQL editor setiap saat.

Dengan pendekatan ini, struktur database:

- lebih terkontrol
- mudah diulang
- mudah diperbaiki
- mudah dibagikan ke komputer lain

## Capaian Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa mampu:

- menjelaskan fungsi database pada aplikasi Laravel
- menjelaskan konsep migration
- menghubungkan Laravel ke database
- merancang struktur tabel sesuai studi kasus greenhouse
- membuat migration dengan Artisan
- menuliskan struktur kolom pada migration
- memahami foreign key dan relasi dasar antar tabel
- menjalankan dan mengulang migration secara benar

## Keterkaitan dengan Studi Kasus

Aplikasi greenhouse monitoring membutuhkan penyimpanan data untuk:

- akun pengguna
- alat atau sensor
- tanaman
- histori pembacaan sensor
- laporan monitoring

Tanpa database, fitur-fitur berikut tidak dapat berjalan dengan baik:

- login berbasis user
- pencatatan alat
- pengelolaan tanaman
- histori data sensor dari IoT
- dashboard berbasis data nyata
- laporan berkala

## Mengapa Database Penting

Bayangkan jika data alat hanya ditulis di array controller. Saat aplikasi ditutup:

- data tidak bisa diperbarui secara permanen
- data tidak bisa dibagi antar pengguna
- histori sensor tidak bisa disimpan
- dashboard tidak bisa menampilkan ringkasan nyata

Karena itu database menjadi pusat penyimpanan aplikasi.

## Apa Itu Migration

Migration adalah file PHP yang mendeskripsikan perubahan struktur database. Dengan migration, pengembang dapat:

- membuat tabel
- menambah kolom
- menghapus kolom
- membuat foreign key
- mengubah struktur database secara bertahap

Migration sangat penting karena:

- struktur database bisa dilacak
- seluruh anggota kelas bisa memiliki struktur tabel yang sama
- perubahan bisa dijalankan ulang dengan perintah artisan

## Analogi Sederhana

Jika Git adalah version control untuk source code, maka migration dapat dianggap sebagai version control untuk struktur database.

## Database yang Digunakan

Mahasiswa dapat menggunakan:

- `MySQL`
- `SQLite`

Untuk praktikum ini, `MySQL` lebih disarankan karena:

- lebih umum dipakai pada hosting dan server production
- cocok untuk simulasi aplikasi nyata
- lebih familiar untuk deployment nantinya

Namun `SQLite` tetap bisa digunakan pada tahap awal bila ingin setup yang lebih ringan.

## Konfigurasi Database

## Opsi 1: MySQL

Atur file `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=greenhouse_monitoring
DB_USERNAME=root
DB_PASSWORD=
```

### Penjelasan

- `DB_CONNECTION=mysql`: menggunakan MySQL
- `DB_HOST=127.0.0.1`: database berjalan di komputer lokal
- `DB_PORT=3306`: port default MySQL
- `DB_DATABASE=greenhouse_monitoring`: nama database yang dibuat
- `DB_USERNAME` dan `DB_PASSWORD`: menyesuaikan setup lokal

## Opsi 2: SQLite

Jika memakai SQLite, buat file database:

```bash
touch database/database.sqlite
```

Lalu atur `.env`:

```env
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database/database.sqlite
```

Catatan:

- SQLite cocok untuk praktik cepat
- jika ingin simulasi lebih dekat ke production, gunakan MySQL

## Merancang Struktur Data Sebelum Menulis Migration

Sebelum menjalankan Artisan, mahasiswa sebaiknya memahami dulu tabel apa saja yang dibutuhkan.

## Daftar Tabel Utama

| Tabel | Fungsi |
| --- | --- |
| `users` | Data akun aplikasi dan role |
| `devices` | Master alat, sensor, controller, atau gateway |
| `plants` | Data tanaman yang dipantau |
| `sensor_readings` | Histori pembacaan sensor dari perangkat |
| `reports` | Laporan monitoring yang dibuat admin atau operator |

## Alasan Membuat Tabel-Tabel Tersebut

### `users`

Tabel ini dibutuhkan untuk:

- login
- pengaturan role
- mencatat siapa pembuat laporan

### `devices`

Tabel ini menyimpan data alat, misalnya:

- kode alat
- nama alat
- lokasi alat
- topic MQTT
- status alat

### `plants`

Tabel ini menyimpan data tanaman yang dipantau pada greenhouse, termasuk kemungkinan keterhubungannya dengan alat tertentu.

### `sensor_readings`

Tabel ini sangat penting untuk dashboard monitoring karena semua data suhu, kelembapan, dan kelembapan tanah akan masuk ke sini.

### `reports`

Tabel ini menyimpan laporan monitoring berkala yang dibuat admin atau operator.

## Relasi Dasar Antar Tabel

Sebelum membuat migration, pahami relasi sederhananya:

- satu `device` dapat memiliki banyak `sensor_readings`
- satu `device` dapat dikaitkan dengan banyak `plants`
- satu `user` dapat membuat banyak `reports`

Sehingga foreign key yang dibutuhkan adalah:

- `plants.device_id`
- `sensor_readings.device_id`
- `reports.user_id`

## Menyiapkan File Migration dengan Artisan

Jalankan perintah berikut:

```bash
php artisan make:migration add_role_to_users_table --table=users
php artisan make:model Device -m
php artisan make:model Plant -m
php artisan make:model SensorReading -m
php artisan make:model Report -m
```

### Penjelasan perintah

- `make:migration add_role_to_users_table --table=users`: membuat migration untuk menambah kolom `role` ke tabel `users`
- `make:model Device -m`: membuat model `Device` sekaligus migration
- `make:model Plant -m`: membuat model `Plant` sekaligus migration
- `make:model SensorReading -m`: membuat model `SensorReading` sekaligus migration
- `make:model Report -m`: membuat model `Report` sekaligus migration

Setelah perintah di atas dijalankan, folder `database/migrations` akan berisi file-file migration baru.

## Struktur Dasar File Migration

Setiap migration umumnya memiliki dua method:

```php
public function up(): void
{
    //
}

public function down(): void
{
    //
}
```

### Fungsi method `up()`

Digunakan untuk menjalankan perubahan database, misalnya:

- membuat tabel
- menambah kolom
- menambah foreign key

### Fungsi method `down()`

Digunakan untuk membatalkan perubahan tersebut, misalnya:

- menghapus tabel
- menghapus kolom

## Implementasi Migration

## 1. Menambah Kolom `role` pada Tabel `users`

File:

```text
database/migrations/xxxx_xx_xx_xxxxxx_add_role_to_users_table.php
```

Isi:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('role')->default('viewer')->after('password');
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('role');
        });
    }
};
```

### Mengapa `role` ditambahkan

Karena aplikasi greenhouse akan memiliki tiga hak akses:

- `admin`
- `operator`
- `viewer`

Kolom ini nantinya akan sangat penting pada modul autentikasi dan RBAC.

## 2. Migration Tabel `devices`

File:

```text
database/migrations/xxxx_xx_xx_xxxxxx_create_devices_table.php
```

Isi method `up()`:

```php
Schema::create('devices', function (Blueprint $table) {
    $table->id();
    $table->string('code')->unique();
    $table->string('name');
    $table->string('type');
    $table->string('location');
    $table->string('mqtt_topic')->nullable();
    $table->enum('status', ['active', 'inactive', 'maintenance'])->default('inactive');
    $table->timestamp('last_seen_at')->nullable();
    $table->timestamps();
});
```

### Penjelasan kolom

- `id()`: primary key otomatis
- `code`: kode unik alat, misalnya `GH-01`
- `name`: nama alat
- `type`: jenis alat, misalnya `sensor`, `controller`, `gateway`
- `location`: posisi alat di greenhouse
- `mqtt_topic`: topik MQTT yang terkait dengan alat
- `status`: status alat
- `last_seen_at`: waktu terakhir alat mengirim data
- `timestamps()`: membuat `created_at` dan `updated_at`

## 3. Migration Tabel `plants`

File:

```text
database/migrations/xxxx_xx_xx_xxxxxx_create_plants_table.php
```

Isi method `up()`:

```php
Schema::create('plants', function (Blueprint $table) {
    $table->id();
    $table->foreignId('device_id')->nullable()->constrained()->nullOnDelete();
    $table->string('name');
    $table->string('variety')->nullable();
    $table->string('greenhouse_block');
    $table->date('planted_at')->nullable();
    $table->string('image')->nullable();
    $table->text('notes')->nullable();
    $table->timestamps();
});
```

### Penjelasan kolom

- `device_id`: relasi ke alat pemantau utama
- `name`: nama tanaman
- `variety`: varietas tanaman
- `greenhouse_block`: blok atau area greenhouse
- `planted_at`: tanggal tanam
- `image`: nama file gambar tanaman
- `notes`: catatan tambahan

### Mengapa `device_id` dibuat nullable

Karena tidak semua tanaman harus langsung terhubung ke alat saat pertama kali didata.

## 4. Migration Tabel `sensor_readings`

File:

```text
database/migrations/xxxx_xx_xx_xxxxxx_create_sensor_readings_table.php
```

Isi method `up()`:

```php
Schema::create('sensor_readings', function (Blueprint $table) {
    $table->id();
    $table->foreignId('device_id')->constrained()->cascadeOnDelete();
    $table->decimal('temperature', 5, 2)->nullable();
    $table->decimal('humidity', 5, 2)->nullable();
    $table->decimal('soil_moisture', 5, 2)->nullable();
    $table->unsignedInteger('light_intensity')->nullable();
    $table->json('payload')->nullable();
    $table->timestamp('recorded_at');
    $table->timestamps();
});
```

### Penjelasan kolom

- `device_id`: menghubungkan pembacaan ke alat tertentu
- `temperature`: suhu
- `humidity`: kelembapan udara
- `soil_moisture`: kelembapan tanah
- `light_intensity`: intensitas cahaya
- `payload`: menyimpan payload mentah bila perlu
- `recorded_at`: waktu sensor direkam

### Mengapa memakai `decimal(5, 2)`

Karena data seperti suhu dan kelembapan biasanya memiliki angka desimal dan tidak membutuhkan presisi yang terlalu besar.

## 5. Migration Tabel `reports`

File:

```text
database/migrations/xxxx_xx_xx_xxxxxx_create_reports_table.php
```

Isi method `up()`:

```php
Schema::create('reports', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->string('title');
    $table->date('period_start');
    $table->date('period_end');
    $table->text('summary');
    $table->string('attachment')->nullable();
    $table->timestamps();
});
```

### Penjelasan kolom

- `user_id`: siapa pembuat laporan
- `title`: judul laporan
- `period_start`: awal periode laporan
- `period_end`: akhir periode laporan
- `summary`: ringkasan laporan
- `attachment`: lampiran file bila ada

## Urutan Pembuatan Tabel

Laravel menjalankan migration berdasarkan urutan timestamp nama file. Karena itu, urutan file penting terutama jika ada foreign key.

Agar aman:

- tabel `devices` dibuat lebih dulu sebelum `plants` dan `sensor_readings`
- tabel `users` sudah ada dari default Laravel sebelum `reports`

Jika urutan salah, migration bisa gagal saat membuat foreign key.

## Menjalankan Migration

Setelah semua file migration selesai diisi, jalankan:

```bash
php artisan migrate
```

Jika berhasil, Laravel akan membuat tabel sesuai migration.

## Mengecek Hasil Migration

Setelah migration dijalankan, cek hasilnya dengan:

- phpMyAdmin
- DBeaver
- HeidiSQL
- TablePlus
- SQLite viewer

Pastikan tabel berikut terbentuk:

- `users`
- `devices`
- `plants`
- `sensor_readings`
- `reports`
- `migrations`

Tabel `migrations` digunakan Laravel untuk mencatat migration mana saja yang sudah dijalankan.

## Mengulang Migration dari Awal

Saat masih tahap belajar, kadang struktur tabel perlu diulang. Gunakan:

```bash
php artisan migrate:fresh
```

Perintah ini akan:

- menghapus seluruh tabel
- membuat ulang tabel dari semua migration

Gunakan dengan hati-hati karena seluruh data akan hilang.

Jika ingin rollback satu langkah:

```bash
php artisan migrate:rollback
```

Jika ingin rollback semua batch terakhir:

```bash
php artisan migrate:reset
```

## Kesalahan Umum Mahasiswa

Beberapa kesalahan yang sering terjadi:

- lupa membuat database terlebih dahulu pada MySQL
- salah menulis nama database di `.env`
- lupa menjalankan `php artisan migrate`
- nama tabel tidak sesuai dengan foreign key
- urutan migration tidak cocok untuk relasi
- salah menggunakan tipe data

## Strategi Debug Jika Migration Gagal

Jika migration gagal:

1. baca pesan error di terminal
2. cek file migration yang disebutkan
3. pastikan nama tabel referensi benar
4. pastikan database sudah dibuat
5. cek file `.env`
6. jalankan ulang setelah perbaikan

Jika cache konfigurasi diduga masih lama, jalankan:

```bash
php artisan config:clear
```

## Catatan Desain Database

Beberapa alasan desain yang dipilih:

- `devices` menyimpan metadata alat dan topik MQTT
- `sensor_readings` menyimpan histori detail monitoring
- `plants.device_id` menghubungkan tanaman dengan alat tertentu
- `reports.user_id` mencatat siapa pembuat laporan
- `payload` disediakan agar data mentah IoT tetap bisa disimpan bila format sensor berkembang

## Pengembangan Lanjutan dari Modul Ini

Setelah migration selesai:

- model akan memakai tabel-tabel ini
- relasi antar tabel akan dibuat pada modul Eloquent
- data dummy akan dimasukkan melalui factory dan seeder
- dashboard akan mulai menggunakan data sungguhan dari database

## Aktivitas Praktik yang Disarankan

Urutan kegiatan pertemuan dapat dibuat seperti ini:

1. review konsep data aplikasi greenhouse
2. jelaskan apa itu migration
3. identifikasi tabel dan relasi yang dibutuhkan
4. jalankan `php artisan make:model -m`
5. isi file migration satu per satu
6. jalankan `php artisan migrate`
7. cek hasil tabel di database viewer

## Hasil Akhir Modul

Database aplikasi greenhouse telah memiliki struktur dasar yang siap dipakai untuk proses CRUD, relasi data, dashboard monitoring, dan integrasi data sensor dari perangkat IoT.

## Tugas Praktik

1. Tambahkan kolom `unit` pada tabel `devices` jika alat memiliki satuan pembacaan khusus.
2. Tambahkan kolom `status` pada tabel `plants` dengan nilai seperti `sehat`, `perlu-perhatian`, dan `panen`.
3. Tambahkan kolom `ph_level` pada tabel `sensor_readings` untuk simulasi sensor tambahan.
4. Jalankan `php artisan migrate:fresh` lalu pastikan semua tabel terbentuk kembali.
5. Buat ringkasan relasi antar tabel dalam bentuk poin singkat.
