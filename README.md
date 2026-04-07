# Modul Praktikum Laravel per Bab

Dokumen pada folder ini memecah `silabus.md` menjadi modul praktikum terpisah berbasis satu studi kasus yang konsisten: **web monitoring greenhouse** berbasis Laravel.

## Ringkasan Studi Kasus

Mahasiswa akan membangun aplikasi `greenhouse-monitoring` dengan fitur:

- dashboard kondisi greenhouse
- manajemen user berbasis role
- manajemen alat/sensor
- manajemen tanaman
- manajemen laporan
- API ingest data sensor untuk integrasi IoT melalui MQTT bridge

## Role Aplikasi

- `admin`: mengelola user, alat, tanaman, laporan, dan konfigurasi API
- `operator`: mengelola alat, tanaman, input laporan, dan memantau dashboard
- `viewer`: melihat dashboard dan laporan tanpa hak ubah data

## Entitas Utama

- `users`
- `devices`
- `plants`
- `sensor_readings`
- `reports`

## Urutan Modul

1. [01_persiapan_lingkungan_dan_instalasi.md](./01_persiapan_lingkungan_dan_instalasi.md)
2. [02_struktur_direktori_dan_konsep_mvc.md](./02_struktur_direktori_dan_konsep_mvc.md)
3. [03_routing_controller_dan_view_dasar.md](./03_routing_controller_dan_view_dasar.md)
4. [04_blade_templating_engine_dan_ui_dasar.md](./04_blade_templating_engine_dan_ui_dasar.md)
5. [05_database_dan_migration.md](./05_database_dan_migration.md)
6. [06_eloquent_orm_dan_model.md](./06_eloquent_orm_dan_model.md)
7. [07_factory_dan_database_seeder.md](./07_factory_dan_database_seeder.md)
8. [08_eloquent_relationship_dan_optimasi_query.md](./08_eloquent_relationship_dan_optimasi_query.md)
9. [09_form_validasi_dan_upload_file.md](./09_form_validasi_dan_upload_file.md)
10. [10_autentikasi_dan_rbac.md](./10_autentikasi_dan_rbac.md)
11. [11_pencarian_dan_paginasi.md](./11_pencarian_dan_paginasi.md)
12. [12_deployment_dan_integrasi_api_iot.md](./12_deployment_dan_integrasi_api_iot.md)

## Alur Hasil Akhir

Pada akhir seluruh modul, mahasiswa diharapkan menghasilkan aplikasi dengan alur berikut:

1. pengguna login sesuai role
2. admin mengelola user, alat, dan tanaman
3. operator memantau dashboard dan mengunggah laporan
4. sensor IoT mengirim data ke broker MQTT
5. bridge MQTT meneruskan data ke endpoint API Laravel
6. dashboard menampilkan data terbaru, ringkasan, dan histori

## Saran Penggunaan

- Gunakan file ini sebagai peta pembelajaran.
- Setiap modul dapat dijadikan 1 pertemuan.
- Dosen dapat menambahkan penilaian mingguan dari bagian tugas pada masing-masing file.
