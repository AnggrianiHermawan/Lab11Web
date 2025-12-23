# Praktikum 11 – Autentikasi dan Session di project lab11_php_oop

- **Nama**  : _Anggriani Hermawan_  
- **NIM**   : _312410175_  
- **Kelas** : _TI.24.A2_

---

## 1. Persiapan Project

### 1.1. Membuat folder project

- Membuat folder `lab11_php_oop` di dalam `htdocs` XAMPP.  
- Struktur awal hanya berisi folder kosong `class`, `module`, `template`, `modul` dan file `index.php`, `config.php`.  
- Menambahkan folder baru yaitu `
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
        $this->conn = new mysqli(
            $this->host,
            $this->user,
            $this->password,
            $this->db_name
        );
        if ($this->conn->connect_error) {
            die("Connection failed: " . $this->conn->connect_error);
        }
    }

    private function getConfig()
    {
        include "config.php";
        $this->host     = $config['host'];
        $this->user     = $config['username'];
        $this->password = $config['password'];
        $this->db_name  = $config['db_name'];
    }

    public function query($sql)
    {
        return $this->conn->query($sql);
    }

    // dipakai di tambah artikel
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

        return $sql ? $sql : false;
    }

    // dipakai di ubah artikel (ambil 1 row)
    public function get($table, $where = null)
    {
        if ($where) {
            $where = " WHERE " . $where;
        }
        $sql = "SELECT * FROM " . $table . $where;
        $sql = $this->conn->query($sql);
        $sql = $sql->fetch_assoc();
        return $sql;
    }
}
?>
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
    RewriteBase /LAB11_PHP_OOP/

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
session_start();

include "config.php";
include "class/Database.php";
include "class/Form.php";

$path = isset($_SERVER['PATH_INFO']) ? $_SERVER['PATH_INFO'] : '/home/index';
$segments = explode('/', trim($path, '/'));

$mod  = isset($segments[0]) ? $segments[0] : 'home';
$page = isset($segments[1]) ? $segments[1] : 'index';

$public_pages = ['home', 'user'];

if (!in_array($mod, $public_pages)) {
    if (!isset($_SESSION['is_login'])) {
        header('Location: user/login');
        exit();
    }
}

$file = "module/{$mod}/{$page}.php";

if (file_exists($file)) {
    if ($mod == 'user' && $page == 'login') {
        include $file;              // login tanpa header/footer
    } else {
        include "template/header.php";
        include $file;
        include "template/footer.php";
    }
} else {
    echo "Halaman tidak ditemukan.";
}
?>
```

### 3.3. Template header & footer

- `template/header.php`:
  - Berisi tag `<html>`, `<head>`, dan judul halaman.  
  - Menampilkan judul "Framework Sederhana – Lab11".  
  - Menambahkan menu sederhana: link ke Data Artikel dan Tambah Artikel.
  - Menambahkan css supaya tampilan menarik
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Framework Sederhana - Lab11</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

    <style>
        body {
            background: #ffeef5;
            font-family: "Segoe UI", Arial, sans-serif;
        }

        .navbar {
            background: #ffb6d9;
        }

        .navbar .nav-link,
        .navbar-brand {
            color: #5a2342 !important;
            font-weight: 600;
        }

        .navbar .nav-link:hover {
            color: #2b0f22 !important;
        }

        .container-main {
            background: #ffffff;
            border-radius: 12px;
            padding: 20px;
            margin-top: 30px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.06);
        }

        h1, h2, h3 {
            color: #c2185b;
        }

        .btn-primary {
            background-color: #ff7fbf;
            border-color: #ff7fbf;
        }

        .btn-primary:hover {
            background-color: #ff5fae;
            border-color: #ff5fae;
        }

        table thead {
            background: #ffc1e3;
        }

        table thead th {
            color: #5a2342;
        }

        .alert {
            border-radius: 10px;
        }

        /* Rapikan form (Judul, Isi) rata kiri */
        table {
            width: auto;
        }
        table td {
            padding: 6px 10px;
        }
        table td:first-child {
            text-align: left;
            width: 80px;
        }
    </style>
</head>
<body>

<nav class="navbar navbar-expand-lg">
    <div class="container">
        <a class="navbar-brand" href="../home/index">Lab11 PHP OOP</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                <li class="nav-item"><a class="nav-link" href="../home/index">Home</a></li>
                <?php if (isset($_SESSION['is_login'])): ?>
                    <li class="nav-item"><a class="nav-link" href="../artikel/index">Data Artikel</a></li>
                <?php endif; ?>
            </ul>
            <ul class="navbar-nav ms-auto">
                <?php if (isset($_SESSION['is_login'])): ?>
                    <li class="nav-item">
                        <a class="nav-link" href="../user/logout">Logout (<?= $_SESSION['nama'] ?>)</a>
                    </li>
                <?php else: ?>
                    <li class="nav-item">
                        <a class="nav-link" href="../user/login">Login</a>
                    </li>
                <?php endif; ?>
            </ul>
        </div>
    </div>
</nav>

<div class="container container-main">
```
- `template/footer.php`:
  - Menampilkan garis pemisah dan teks copyright.
```
</div> <!-- /.container-main -->

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
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
$db  = new Database();
$sql = $db->query("SELECT * FROM artikel ORDER BY id ASC");
?>
<h3>Data Artikel</h3>
<p><a class="btn btn-link" href="../artikel/tambah">+ Tambah Artikel</a></p>

<table class="table table-bordered">
    <thead>
        <tr>
            <th>ID</th>
            <th>Judul</th>
            <th>Isi</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody>
    <?php while ($row = $sql->fetch_assoc()) : ?>
        <tr>
            <td><?= $row['id']; ?></td>
            <td><?= $row['judul']; ?></td>
            <td><?= $row['isi']; ?></td>
            <td>
                <a href="../artikel/ubah?id=<?= $row['id']; ?>">Ubah</a>
            </td>
        </tr>
    <?php endwhile; ?>
    </tbody>
</table>
```
<img width="960" height="523" alt="Cuplikan layar 2025-12-23 112633" src="https://github.com/user-attachments/assets/1f8b2a07-01c3-497a-8cff-6391d4e0c0ab" />

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
$db   = new Database();
$form = new Form("../artikel/tambah", "Simpan");

if ($_POST) {
    $data = [
        'judul' => $_POST['judul'],
        'isi'   => $_POST['isi'],
    ];
    $simpan = $db->insert('artikel', $data);
    if ($simpan) {
        echo "<div class='alert alert-success'>Data berhasil disimpan!</div>";
    } else {
        echo "<div class='alert alert-danger'>Gagal menyimpan data.</div>";
    }
}

echo "<h3>Tambah Artikel</h3>";
$form->addField("judul", "Judul");
$form->addField("isi", "Isi", "textarea");
$form->displayForm();
?>
```
<img width="960" height="522" alt="Cuplikan layar 2025-12-23 112122" src="https://github.com/user-attachments/assets/5a37ab92-745c-49f2-b5f8-c2b58068f0ad" />
<img width="867" height="336" alt="Cuplikan layar 2025-12-23 112156" src="https://github.com/user-attachments/assets/28bcadaa-0fc7-4361-a329-c36e7677bcdb" />

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
$id = isset($_GET['id']) ? (int) $_GET['id'] : 0;

// ambil data artikel berdasarkan id
$data = $db->get('artikel', "id={$id}");

if (!$data) {
    echo "<div class='alert alert-danger'>Data artikel tidak ditemukan.</div>";
    return;
}

// proses update saat form disubmit
if ($_POST) {
    $judul = $_POST['judul'];
    $isi   = $_POST['isi'];

    $sql = "UPDATE artikel SET judul='{$judul}', isi='{$isi}' WHERE id={$id}";
    $ok  = $db->query($sql);

    if ($ok) {
        echo "<div class='alert alert-success'>Data berhasil diubah.</div>";
    } else {
        echo "<div class='alert alert-danger'>Gagal mengubah data.</div>";
    }
}
?>

<h3>Ubah Artikel</h3>
<form method="post">
    <div class="mb-3">
        <label>Judul</label>
        <input type="text"
               name="judul"
               class="form-control"
               value="<?= htmlspecialchars($data['judul']); ?>">
    </div>
    <div class="mb-3">
        <label>Isi</label>
        <textarea name="isi"
                  class="form-control"
                  rows="4"><?= htmlspecialchars($data['isi']); ?></textarea>
    </div>
    <button type="submit" class="btn btn-primary">Simpan Perubahan</button>
</form>
```
<img width="960" height="522" alt="Cuplikan layar 2025-12-23 112936" src="https://github.com/user-attachments/assets/6544782d-4495-4a8e-9834-83edf43dc3ab" />

<img width="904" height="170" alt="Cuplikan layar 2025-12-23 113122" src="https://github.com/user-attachments/assets/7e39e14b-6ca0-4e4e-98e8-a419330ed36d" />

setelah dirubah artikel nya
<img width="957" height="523" alt="Cuplikan layar 2025-12-23 113144" src="https://github.com/user-attachments/assets/afdc52fa-ce4d-4f8d-8019-c1c576edcc54" />

---

## 5. Modul User (Login dan logout)

### 5.1. Struktur folder modul

- Di folder `module` dibuat folder `user`.  
- Di dalamnya terdapat:
  - `login.php` – form login user dan proses autentikasi.  
  - `logout.php` – proses logout user dan penghapusan session.

 ---
 
### 5.2. Halaman User (module/user/login.php)

- Menggunakan objek `Database` untuk menjalankan query: `SELECT * FROM users WHERE username = ... LIMIT 1.`.  
- Melakukan verifikasi password dengan fungsi `password_verify()` dan menyimpan data login ke session `(is_login, username, nama)`.  
- Jika login berhasil, pengguna diarahkan ke halaman /artikel/index.
- Jika login gagal, menampilkan pesan kesalahan "Username atau password salah!" dan menampilkan kembali form login.
```
<?php
if (isset($_SESSION['is_login'])) {
    header('Location: ../home/index');
    exit;
}

$message = "";

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $db = new Database();

    $username = $_POST['username'];
    $password = $_POST['password'];

    $sql    = "SELECT * FROM users WHERE username = '{$username}' LIMIT 1";
    $result = $db->query($sql);
    $data   = $result ? $result->fetch_assoc() : null;

    if ($data && password_verify($password, $data['password'])) {
        $_SESSION['is_login'] = true;
        $_SESSION['username'] = $data['username'];
        $_SESSION['nama']     = $data['nama'];

        header('Location: ../artikel/index');
        exit;
    } else {
        $message = "Username atau password salah!";
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Login System</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background: #ffeef5;
            font-family: "Segoe UI", Arial, sans-serif;
        }
        .login-container {
            max-width: 420px;
            margin: 80px auto;
            padding: 24px 28px;
            background: #ffffff;
            border-radius: 14px;
            box-shadow: 0 6px 18px rgba(0,0,0,0.08);
            border-top: 4px solid #ff9acc;
        }
        h3 { color: #c2185b; }
        .btn-primary {
            background-color: #ff7fbf;
            border-color: #ff7fbf;
        }
        .btn-primary:hover {
            background-color: #ff5fae;
            border-color: #ff5fae;
        }
    </style>
</head>
<body>
<div class="login-container">
    <h3 class="text-center mb-4">Login User</h3>

    <?php if ($message): ?>
        <div class="alert alert-danger"><?= $message ?></div>
    <?php endif; ?>

    <form method="POST" action="">
        <div class="mb-3">
            <label>Username</label>
            <input type="text" name="username" class="form-control" required>
        </div>
        <div class="mb-3">
            <label>Password</label>
            <input type="password" name="password" class="form-control" required>
        </div>
        <div class="d-grid">
            <button type="submit" class="btn btn-primary">Login</button>
        </div>
    </form>

    <div class="mt-3 text-center">
        <a href="../home/index">Kembali ke Home</a>
    </div>
</div>
</body>
</html>
```
<img width="960" height="525" alt="Cuplikan layar 2025-12-23 110752" src="https://github.com/user-attachments/assets/7a111f5d-1302-494b-8e94-5cd6f4f3c953" />
menampilkan langsung ke halaman index
<img width="957" height="522" alt="Cuplikan layar 2025-12-23 110816" src="https://github.com/user-attachments/assets/9cda656c-afa6-4dc9-a743-2354af42ba35" />

---

### 5.3. Halaman User (module/user/logout.php)

- Menghapus seluruh data session dengan `session_destroy()`.
- Setelah logout, pengguna diarahkan kembali ke halaman login `(/user/login)`.
```
<?php
session_destroy();
header('Location: ../user/login');
exit;
?>
```
<img width="960" height="198" alt="Cuplikan layar 2025-12-23 113223" src="https://github.com/user-attachments/assets/bcd0f77f-0e27-4db4-8589-a28899221f75" />

---

## 6. Halaman Home (module/home/index.php)
- Menampilkan judul `"Framework Sederhana - Lab11"` sebagai halaman utama setelah masuk ke aplikasi.
- Menyediakan link navigasi cepat ke modul artikel: Data Artikel `(menuju /artikel/index)` dan Tambah Artikel `(menuju /artikel/tambah)` sehingga pengguna mudah berpindah ke halaman daftar maupun form tambah artikel.
```
<h3>Framework Sederhana - Lab11</h3>
<p>Silakan pilih menu <a href="../artikel/index">Data Artikel</a> atau <a href="../artikel/tambah">Tambah Artikel</a>.</p>
```
<img width="955" height="523" alt="Cuplikan layar 2025-12-23 113239" src="https://github.com/user-attachments/assets/f2c59d90-5e86-49d1-8800-44fe23817322" />

---

## 7. Cek Data di Database (phpMyAdmin)

- Membuka `http://localhost/phpmyadmin`.  
- Memilih database yang digunakan (`latihan1` ).  
- Membuka tabel `artikel` dan klik tab **Jelajahi**.  
- Setelah menyimpan data dari aplikasi, baris baru dengan kolom `id`, `judul`, dan `isi` akan muncul.  

<img width="388" height="152" alt="image" src="https://github.com/user-attachments/assets/ef18c4c8-daa3-4d9a-a57b-26e9eb3e4c41" />

---

## 8. Cek Data di Database (phpMyAdmin)

- Membuka `http://localhost/phpmyadmin`.  
- Memilih database yang digunakan (`latihan1` ).  
- Membuka tabel `user` dan klik tab **Jelajahi**.  
- Setelah menyimpan data dari aplikasi, baris baru dengan kolom `id`, `username`, `password` dan `nama` akan muncul.  
<img width="479" height="48" alt="image" src="https://github.com/user-attachments/assets/1edbd98f-0dc7-462e-b64b-e7a69287e459" />

