# Praktikum 11 – Autentikasi dan Session di project lab11_php_oop

- **Nama**  : _Anggriani Hermawan_  
- **NIM**   : _312410175_  
- **Kelas** : _TI.24.A2_

---

## 1. Persiapan Project

### 1.1. Membuat folder project

- Membuat folder `lab11_php_oop` di dalam `htdocs` XAMPP.  
- Struktur awal hanya berisi folder kosong `class`, `module`, `template`, `modul` dan file `index.php`, `config.php`.  
- Menambahkan folder baru yaitu `modul`
<img width="156" height="291" alt="image" src="https://github.com/user-attachments/assets/2a206521-3962-43d9-afc1-dfd54b009725" />

### 1.2. Membuat konfigurasi database (config.php)

- Membuat file `config.php` dengan isi pengaturan koneksi ke MySQL: host, username, password, dan nama database (`latihan1`).  
- File ini akan dipakai oleh class `Database` untuk mengambil konfigurasi.  
```
<?php
$config = [
    'host'     => 'localhost',
    'username' => 'root',
    'password' => '',   // sesuaikan
    'db_name'  => 'latihan1' // sesuaikan
];
```

---

## 2. Membuat Class OOP (Database & Form)

### 2.1. Class Database (class/Database.php)

- Menambahkan file `class/Database.php` yang berisi class `Database`.  
- Fitur utama:
  - Koneksi ke MySQL di constructor.  
  - Method `query()` untuk menjalankan SQL.  
  - Method `get()` untuk mengambil satu record.  
  - Method `insert()` dan `update()` untuk operasi CRUD.  
```
<?php
class Database
{
    protected $host;
    protected $user;
    protected $password;
    protected $db_name;
    protected $conn;

    public function __construct()
    {
        $this->getConfig();
        $this->conn = new mysqli($this->host, $this->user, $this->password, $this->db_name);
        if ($this->conn->connect_error) {
            die("Connection failed: " . $this->conn->connect_error);
        }
    }

    private function getConfig()
    {
        include("config.php");
        $this->host     = $config['host'];
        $this->user     = $config['username'];
        $this->password = $config['password'];
        $this->db_name  = $config['db_name'];
    }

    public function query($sql)
    {
        return $this->conn->query($sql);
    }

    public function get($table, $where = null)
    {
        if ($where) {
            $where = " WHERE " . $where;
        }
        $sql = "SELECT * FROM " . $table . $where;
        $sql = $this->conn->query($sql);
        return $sql->fetch_assoc();
    }

    public function insert($table, $data)
    {
        if (is_array($data)) {
            foreach ($data as $key => $val) {
                $column[] = $key;
                $value[]  = "'{$val}'";
            }
            $columns = implode(",", $column);
            $values  = implode(",", $value);
        }

        $sql = "INSERT INTO " . $table . " (" . $columns . ") VALUES (" . $values . ")";
        $sql = $this->conn->query($sql);

        return $sql ? true : false;
    }

    public function update($table, $data, $where)
    {
        $update_value = [];
        if (is_array($data)) {
            foreach ($data as $key => $val) {
                $update_value[] = "$key='{$val}'";
            }
            $update_value = implode(",", $update_value);
        }

        $sql = "UPDATE " . $table . " SET " . $update_value . " WHERE " . $where;
        $sql = $this->conn->query($sql);

        return $sql ? true : false;
    }
}
```

### 2.2. Class Form (class/Form.php)

- Menambahkan file `class/Form.php` yang berisi class `Form`.  
- Fitur utama:
  - Menyimpan definisi field dalam array.  
  - Method `addField()` untuk menambah field (text, textarea, select, radio, checkbox, password).  
  - Method `displayForm()` untuk men-generate HTML form secara dinamis.  
```
<?php
class Form
{
    private $fields = array();
    private $action;
    private $submit = "Submit Form";
    private $jumField = 0;

    public function __construct($action, $submit)
    {
        $this->action = $action;
        $this->submit = $submit;
    }

    public function addField($name, $label, $type = "text", $options = array())
    {
        $this->fields[$this->jumField]['name']    = $name;
        $this->fields[$this->jumField]['label']   = $label;
        $this->fields[$this->jumField]['type']    = $type;
        $this->fields[$this->jumField]['options'] = $options;
        $this->jumField++;
    }

    public function displayForm()
    {
        echo "<form action='" . $this->action . "' method='POST'>";
        echo '<table width="100%" border="0">';

        foreach ($this->fields as $field) {
            echo "<tr><td align='right' valign='top'>" . $field['label'] . "</td>";
            echo "<td>";

            switch ($field['type']) {
                case 'textarea':
                    echo "<textarea name='" . $field['name'] . "' cols='30' rows='4'></textarea>";
                    break;

                case 'select':
                    echo "<select name='" . $field['name'] . "'>";
                    foreach ($field['options'] as $value => $label) {
                        echo "<option value='" . $value . "'>" . $label . "</option>";
                    }
                    echo "</select>";
                    break;

                case 'radio':
                    foreach ($field['options'] as $value => $label) {
                        echo "<label><input type='radio' name='" . $field['name'] . "' value='" . $value . "'> " . $label . "</label> ";
                    }
                    break;

                case 'checkbox':
                    foreach ($field['options'] as $value => $label) {
                        echo "<label><input type='checkbox' name='" . $field['name'] . "[]' value='" . $value . "'> " . $label . "</label> ";
                    }
                    break;

                case 'password':
                    echo "<input type='password' name='" . $field['name'] . "'>";
                    break;

                default:
                    echo "<input type='text' name='" . $field['name'] . "'>";
                    break;
            }

            echo "</td></tr>";
        }

        echo "<tr><td colspan='2'>";
        echo "<input type='submit' value='" . $this->submit . "'></td></tr>";
        echo "</table>";
        echo "</form>";
    }
}
```

---

## 3. Routing dan Template

### 3.1. File .htaccess (URL Rewrite)

- Menambahkan file `.htaccess` di root `lab11_php_oop`.  
- Mengaktifkan `RewriteEngine` dan mengarahkan semua request ke `index.php`.  
- Contoh: URL `localhost/lab11_php_oop/artikel/tambah` akan diproses oleh `index.php` dengan `PATH_INFO=/artikel/tambah`.  
```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /lab11_php_oop/

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f

    RewriteRule ^(.*)$ index.php/$1 [L]
</IfModule>
```

### 3.2. File index.php (Gerbang routing)

- Di `index.php`:
  - Meng-include `config.php`, `Database.php`, dan `Form.php`.  
  - Mengambil `PATH_INFO` lalu memecah menjadi `$mod` (module) dan `$page`.  
  - Menentukan file modul: `module/{$mod}/{$page}.php`.  
  - Meng-include `template/header.php` dan `template/footer.php`.  
```
<?php
include "config.php";
include "class/Database.php";
include "class/Form.php";

session_start();

// Contoh URL: /lab11_php_oop/artikel/tambah
$path = isset($_SERVER['PATH_INFO']) ? $_SERVER['PATH_INFO'] : '/artikel/index';
$segments = explode('/', trim($path, '/'));

$mod  = isset($segments[0]) ? $segments[0] : 'artikel';
$page = isset($segments[1]) ? $segments[1] : 'index';

$file = "module/{$mod}/{$page}.php";

include "template/header.php";

if (file_exists($file)) {
    include $file;
} else {
    echo "<h3>Halaman tidak ditemukan: {$mod}/{$page}</h3>";
}

include "template/footer.php";
```

### 3.3. Template header & footer

- `template/header.php`:
  - Berisi tag `<html>`, `<head>`, dan judul halaman.  
  - Menampilkan judul "Framework Sederhana – Lab11".  
  - Menambahkan menu sederhana: link ke Data Artikel dan Tambah Artikel.
```
<!DOCTYPE html>
<html>
<head>
    <title>Framework Sederhana - Lab11</title>
</head>
<body>
    <h1>Framework Sederhana - Lab11</h1>
    <hr>
    <p>
        <a href="/lab11_php_oop/artikel/index">Data Artikel</a> |
        <a href="/lab11_php_oop/artikel/tambah">Tambah Artikel</a>
    </p>
    <hr>
```
- `template/footer.php`:
  - Menampilkan garis pemisah dan teks copyright.
```
<!-- template/footer.php -->
    <hr>
    <p>&copy; 2025 - Praktikum 11</p>
</body>
</html>
```
---

## 4. Modul Artikel (index, tambah, ubah)

### 4.1. Struktur folder modul

- Di folder `module` dibuat folder `artikel`.  
- Di dalamnya terdapat:
  - `index.php` – menampilkan daftar artikel.  
  - `tambah.php` – form tambah artikel dan proses simpan.  
  - `ubah.php` – form ubah artikel dan proses update.  

### 4.2. Halaman Data Artikel (module/artikel/index.php)

- Menggunakan objek `Database` untuk menjalankan query: `SELECT * FROM artikel ORDER BY id DESC`.  
- Menampilkan hasil dalam bentuk tabel HTML:
  - Kolom: ID, Judul, Isi, Aksi.  
  - Di kolom Aksi ada link "Ubah" yang mengarah ke `/artikel/ubah?id=...`.  
- Menyediakan link `+ Tambah Artikel` ke halaman tambah.  
```
<?php
$db   = new Database();
$data = $db->query("SELECT * FROM artikel ORDER BY id DESC");
?>

<h2>Data Artikel</h2>
<p>
    <a href="/lab11_php_oop/artikel/tambah">+ Tambah Artikel</a>
</p>

<table border="1" cellpadding="5" cellspacing="0" width="100%">
    <tr>
        <th>ID</th>
        <th>Judul</th>
        <th>Isi</th>
        <th>Aksi</th>
    </tr>
    <?php if ($data && $data->num_rows > 0): ?>
        <?php while ($row = $data->fetch_assoc()): ?>
            <tr>
                <td><?= $row['id']; ?></td>
                <td><?= htmlspecialchars($row['judul']); ?></td>
                <td><?= nl2br(htmlspecialchars($row['isi'])); ?></td>
                <td>
                    <a href="/lab11_php_oop/artikel/ubah?id=<?= $row['id']; ?>">Ubah</a>
                </td>
            </tr>
        <?php endwhile; ?>
    <?php else: ?>
        <tr>
            <td colspan="4">Belum ada data.</td>
        </tr>
    <?php endif; ?>
</table>
```
<img width="959" height="334" alt="image" src="https://github.com/user-attachments/assets/04e423f3-f573-4475-8e87-deb6d752b2b7" />

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
```
<?php
$db = new Database();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $judul = $_POST['judul'] ?? '';
    $isi   = $_POST['isi'] ?? '';

    $data = [
        'judul' => $judul,
        'isi'   => $isi
    ];

    $simpan = $db->insert('artikel', $data);

    if ($simpan) {
        echo "<p style='color:green'>INSERT OK (ke tabel artikel)</p>";
    } else {
        echo "<p style='color:red'>INSERT GAGAL</p>";
    }

    echo "<pre>";
    echo "DB name  : " . $db->db_name ?? '' . "\n";
    echo "Judul    : " . $judul . "\n";
    echo "Isi      : " . $isi . "\n";
    echo "</pre>";
}
?>

<h2>Tambah Artikel</h2>

<form action="" method="POST">
    <p>
        <label>Judul</label><br>
        <input type="text" name="judul">
    </p>
    <p>
        <label>Isi</label><br>
        <textarea name="isi" cols="40" rows="5"></textarea>
    </p>
    <p>
        <button type="submit">Simpan</button>
    </p>
</form>
```
<img width="960" height="338" alt="Cuplikan layar 2025-12-09 150141" src="https://github.com/user-attachments/assets/916dc915-7bc6-4a8b-9abc-1402fb079f44" />
<img width="108" height="35" alt="Cuplikan layar 2025-12-09 154644" src="https://github.com/user-attachments/assets/90af0c0d-5588-4031-a8a1-d3d306bfb233" />

### 4.4. Halaman Ubah Artikel (module/artikel/ubah.php)

- Mengambil parameter `id` dari `$_GET`.  
- Menggunakan `$db->get('artikel', "id={$id}")` untuk mengambil data lama.  
- Menampilkan form yang sudah terisi nilai lama (judul & isi).  
- Saat form disubmit:
  - Mengambil nilai baru `judul` dan `isi`.  
  - Memanggil `$db->update('artikel', $data, "id={$id}")`.  
  - Menampilkan pesan "Data berhasil diupdate" dan redirect ke daftar artikel.  
```
<?php
$db = new Database();

$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;

// ambil data lama
$artikel = $db->get('artikel', "id={$id}");

if (!$artikel) {
    echo "<h3>Data tidak ditemukan.</h3>";
    return;
}

// jika form disubmit, proses update
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $judul = $_POST['judul'] ?? '';
    $isi   = $_POST['isi'] ?? '';

    $data = [
        'judul' => $judul,
        'isi'   => $isi
    ];

    $update = $db->update('artikel', $data, "id={$id}");

    if ($update) {
        echo "<p style='color:green'>Data berhasil diupdate.</p>";
        echo "<meta http-equiv='refresh' content='1; url=/lab11_php_oop/artikel/index'>";
    } else {
        echo "<p style='color:red'>Gagal mengupdate data.</p>";
    }
}
?>

<h2>Ubah Artikel</h2>

<form action="/lab11_php_oop/artikel/ubah?id=<?= $id; ?>" method="POST">
    <p>
        <label>Judul</label><br>
        <input type="text" name="judul" value="<?= htmlspecialchars($artikel['judul']); ?>">
    </p>
    <p>
        <label>Isi</label><br>
        <textarea name="isi" cols="40" rows="5"><?= htmlspecialchars($artikel['isi']); ?></textarea>
    </p>
    <p>
        <button type="submit">Update</button>
    </p>
</form>
```
<img width="959" height="320" alt="Cuplikan layar 2025-12-09 154131" src="https://github.com/user-attachments/assets/717d7678-ba42-4038-bed1-be83a7b23f16" />
<img width="156" height="33" alt="Cuplikan layar 2025-12-09 154208" src="https://github.com/user-attachments/assets/06547f58-989d-4275-91e5-0595d7c65051" />

---

## 5. Cek Data di Database (phpMyAdmin)

- Membuka `http://localhost/phpmyadmin`.  
- Memilih database yang digunakan (`latihan1` ).  
- Membuka tabel `artikel` dan klik tab **Jelajahi**.  
- Setelah menyimpan data dari aplikasi, baris baru dengan kolom `id`, `judul`, dan `isi` akan muncul.  

<img width="388" height="152" alt="image" src="https://github.com/user-attachments/assets/ef18c4c8-daa3-4d9a-a57b-26e9eb3e4c41" />


