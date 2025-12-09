# Praktikum 11 – PHP OOP Lanjutan (Framework Modular & Routing)

## Identitas

- **Nama**  : _Isi nama kamu_  
- **NIM**   : _Isi NIM kamu_  
- **Kelas** : _Isi kelas kamu_

---

## 1. Persiapan Project

### 1.1. Membuat folder project

- Membuat folder `lab11_php_oop` di dalam `htdocs` XAMPP.  
- Struktur awal hanya berisi folder kosong `class`, `module`, `template`, dan file `index.php`, `config.php`.  

**Screenshot:**  
`ss01-struktur-awal.png` – Menampilkan struktur folder `lab11_php_oop` di VS Code / Explorer.

### 1.2. Membuat konfigurasi database (config.php)

- Membuat file `config.php` dengan isi pengaturan koneksi ke MySQL: host, username, password, dan nama database (`latihan_oop` atau `latihan1`).  
- File ini akan dipakai oleh class `Database` untuk mengambil konfigurasi.  

**Screenshot:**  
`ss02-config-php.png` – Menampilkan isi file `config.php`.

---

## 2. Membuat Class OOP (Database & Form)

### 2.1. Class Database (class/Database.php)

- Menambahkan file `class/Database.php` yang berisi class `Database`.  
- Fitur utama:
  - Koneksi ke MySQL di constructor.  
  - Method `query()` untuk menjalankan SQL.  
  - Method `get()` untuk mengambil satu record.  
  - Method `insert()` dan `update()` untuk operasi CRUD.  

**Screenshot:**  
`ss03-database-php.png` – Menampilkan isi class `Database`.

### 2.2. Class Form (class/Form.php)

- Menambahkan file `class/Form.php` yang berisi class `Form`.  
- Fitur utama:
  - Menyimpan definisi field dalam array.  
  - Method `addField()` untuk menambah field (text, textarea, select, radio, checkbox, password).  
  - Method `displayForm()` untuk men-generate HTML form secara dinamis.  

**Screenshot:**  
`ss04-form-php.png` – Menampilkan isi class `Form`.

---

## 3. Routing dan Template

### 3.1. File .htaccess (URL Rewrite)

- Menambahkan file `.htaccess` di root `lab11_php_oop`.  
- Mengaktifkan `RewriteEngine` dan mengarahkan semua request ke `index.php`.  
- Contoh: URL `localhost/lab11_php_oop/artikel/tambah` akan diproses oleh `index.php` dengan `PATH_INFO=/artikel/tambah`.  

**Screenshot:**  
`ss05-htaccess.png` – Menampilkan isi `.htaccess`.

### 3.2. File index.php (Gerbang routing)

- Di `index.php`:
  - Meng-include `config.php`, `Database.php`, dan `Form.php`.  
  - Mengambil `PATH_INFO` lalu memecah menjadi `$mod` (module) dan `$page`.  
  - Menentukan file modul: `module/{$mod}/{$page}.php`.  
  - Meng-include `template/header.php` dan `template/footer.php`.  

**Screenshot:**  
`ss06-index-routing.png` – Menampilkan logika routing di `index.php`.

### 3.3. Template header & footer

- `template/header.php`:
  - Berisi tag `<html>`, `<head>`, dan judul halaman.  
  - Menampilkan judul "Framework Sederhana – Lab11".  
  - Menambahkan menu sederhana: link ke Data Artikel dan Tambah Artikel.  
- `template/footer.php`:
  - Menampilkan garis pemisah dan teks copyright.  

**Screenshot:**  
`ss07-template-header-footer.png` – Menampilkan isi header.php dan footer.php.

---

## 4. Modul Artikel (index, tambah, ubah)

### 4.1. Struktur folder modul

- Di folder `module` dibuat folder `artikel`.  
- Di dalamnya terdapat:
  - `index.php` – menampilkan daftar artikel.  
  - `tambah.php` – form tambah artikel dan proses simpan.  
  - `ubah.php` – form ubah artikel dan proses update.  

**Screenshot:**  
`ss08-struktur-module-artikel.png` – Menampilkan struktur `module/artikel` di VS Code.

### 4.2. Halaman Data Artikel (module/artikel/index.php)

- Menggunakan objek `Database` untuk menjalankan query: `SELECT * FROM artikel ORDER BY id DESC`.  
- Menampilkan hasil dalam bentuk tabel HTML:
  - Kolom: ID, Judul, Isi, Aksi.  
  - Di kolom Aksi ada link "Ubah" yang mengarah ke `/artikel/ubah?id=...`.  
- Menyediakan link `+ Tambah Artikel` ke halaman tambah.  

**Screenshot:**  
`ss09-artikel-index-code.png` – Menampilkan kode index.php.  
`ss10-artikel-index-view.png` – Menampilkan tampilan daftar artikel di browser.

### 4.3. Halaman Tambah Artikel (module/artikel/tambah.php)

- Menggunakan objek `Database` untuk insert ke tabel `artikel`.  
- Pada method POST:
  - Mengambil input `judul` dan `isi` dari form.  
  - Membuat array `$data = ['judul' => ..., 'isi' => ...]`.  
  - Memanggil `$db->insert('artikel', $data)`.  
  - Menampilkan pesan keberhasilan dan redirect kembali ke daftar artikel.  
- Form HTML berisi input:
  - Text `judul`.  
  - Textarea `isi`.  

**Screenshot:**  
`ss11-artikel-tambah-code.png` – Menampilkan kode tambah.php.  
`ss12-artikel-tambah-view.png` – Menampilkan tampilan form tambah di browser.  
`ss13-artikel-tambah-success.png` – Menampilkan pesan "Data berhasil disimpan".

### 4.4. Halaman Ubah Artikel (module/artikel/ubah.php)

- Mengambil parameter `id` dari `$_GET`.  
- Menggunakan `$db->get('artikel', "id={$id}")` untuk mengambil data lama.  
- Menampilkan form yang sudah terisi nilai lama (judul & isi).  
- Saat form disubmit:
  - Mengambil nilai baru `judul` dan `isi`.  
  - Memanggil `$db->update('artikel', $data, "id={$id}")`.  
  - Menampilkan pesan "Data berhasil diupdate" dan redirect ke daftar artikel.  

**Screenshot:**  
`ss14-artikel-ubah-code.png` – Menampilkan kode ubah.php.  
`ss15-artikel-ubah-view.png` – Menampilkan tampilan form ubah di browser.

---

## 5. Cek Data di Database (phpMyAdmin)

- Membuka `http://localhost/phpmyadmin`.  
- Memilih database yang digunakan (misalnya `latihan1` atau `latihan_oop`).  
- Membuka tabel `artikel` dan klik tab **Jelajahi**.  
- Setelah menyimpan data dari aplikasi, baris baru dengan kolom `id`, `judul`, dan `isi` akan muncul.  

**Screenshot:**  
`ss16-phpmyadmin-artikel-empty.png` – Tampilan sebelum insert (tabel kosong).  
`ss17-phpmyadmin-artikel-data.png` – Tampilan setelah insert (data artikel sudah masuk).

---

## 6. Kesimpulan

- Praktikum 11 berhasil menerapkan:
  - Konsep modularisasi dengan folder `module/artikel`.  
  - Konsep routing sederhana melalui `index.php` dan `.htaccess`.  
  - Penggunaan OOP di PHP melalui class `Database` dan `Form`.  
  - Operasi CRUD sederhana (Create dan Update) pada tabel `artikel`.  

Dengan struktur ini, penambahan modul baru (misalnya `user`, `produk`) cukup dibuat folder dan file baru di `module/`, kemudian diakses melalui URL sesuai pattern routing.
