# Lab7web-3


## PRAKTIKUM 13 - FRAMEWORK LANJUTAN (MODUL LOGIN)

Dipertemuan kali ini kita masih melanjutkan tugas sebelumnya namun kita akan membuat sekaligus mempelajari bagaimana membuat **System Login** dengan module login dalam **Framework CodeIgniter 4** 

## LANGKAH - LANGKAH PRAKTIKUM

## PERSIAPAN
Untuk memulai membuat modul login, yang perlu disiapkan adalah database server menggunakan MySQL. Pastikan MySQL server sudah dapat dijalankan melalui XAMPP.

## MEMBUAT TABEL: USER LOGIN

## 1). MEMBUAT TABEL USER
Buat Tabel User pada Database **lab_ci4**

![Table-user](https://github.com/Herli27052000/Lab11Web/blob/master/img/tabel-user.png)

**PENJELASAN**

Table berhasil dibuat

**Table user**
```MySQL
CREATE TABLE user (
  id INT(11) auto_increment,
  username VARCHAR(200) NOT NULL,
  useremail VARCHAR(200),
  userpassword VARCHAR(200),
  PRIMARY KEY(id)
);
```

## 2). MEMBUAT MODEL USER
Selanjutnya adalah membuat Model untuk memproses data Login. Buat file baru pada direktori **app/Models** dengan nama **UserModel.php**

![user-model](https://github.com/Herli27052000/Lab11Web/blob/master/img/user-models.png)

**code UserModel.php**
```php
<?php

namespace App\Models;

use CodeIgniter\Model;

class UserModel extends Model
{
    protected $table = 'user';
    protected $primaryKey = 'id';
    protected $useAutoIncrement = true;
    protected $allowedFields = ['username', 'useremail', 'userpassword'];
}
```

## 3). MEMBUAT CONTROLLERS USER
Buat Controllers baru dengan nama **User.php** pada direktori **app/Controllers.** Kemudian tambahkan method **index()** untuk menampilkan daftar user, dan method **login()** untuk proses login.

![function-index](https://github.com/Herli27052000/Lab11Web/blob/master/img/function-index.png)

![function-login](https://github.com/Herli27052000/Lab11Web/blob/master/img/function-login.png)

**code User.php**
```php
<?php

namespace App\Controllers;

use App\Models\UserModel;

class User extends BaseController
{
    public function index()
    {
        $title = 'Daftar User';
        $model = new UserModel();
        $users = $model->findAll();
        return view('user/index', compact('users', 'title'));
    }

    public function login()
    {
        helper(['form']);
        $email = $this->request->getPost('email');
        $password = $this->request->getPost('password');
        if (!$email)
        {
            return view('user/login');
        }

        $session = session();
        $model = new UserModel();
        $login = $model->where('useremail', $email)->first();
        if ($login)
        {
            $pass = $login['userpassword'];
            if (password_verify($password, $pass))
            {
            $login_data = [
                'user_id' => $login['id'],
                'user_name' => $login['username'],
                'user_email' => $login['useremail'],
                'logged_in' => TRUE,
            ];
            $session->set($login_data);
            return redirect('admin/artikel');
        }
        else
        {
            $session->setFlashdata("flash_msg", "Password salah.");
            return redirect()->to('/user/login');
            }
        }
        else
        {
            $session->setFlashdata("flash_msg", "email tidak terdaftar.");
            return redirect()->to('/user/login');
        }
    }
}
```

## 4). MEMBUAT VIEW LOGIN
Buat direktori baru dengan nama **user** pada direktori **app/views,** kemudian buat file baru dengan nama **login.php**

![login-user](https://github.com/Herli27052000/Lab11Web/blob/master/img/login.png)

**code login.php**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
    <link rel="stylesheet" href="<?= base_url('/style.css');?>">
</head>
<body>
    <div id="login-wrapper">
        <h1>Sign In</h1>
        <?php if(session()->getFlashdata('flash_msg')):?>
            <div class="alert alert-danger"><?=session()->getFlashdata('flash_msg') ?></div>
        <?php endif;?>
        <form action="" method="post">
            <div class="mb-3">
                <label for="InputForEmail" class="form-label">Email address</label>
                <input type="email" name="email" class="form-control" id="InputForEmail" value="<?= set_value('email') ?>">
            </div>
            <div class="mb-3">
                <label for="InputForPassword" class="form-label">Password</label>
                <input type="password" name="password" class="form-control" id="InputForPassword">
            </div>
            <button type="submit" class="btn btn-primary">Login</button>
        </form>
    </div>
</body>
</html>
```

## 5). MEMBUAT DATABASE SEEDER
Database seeder digunakan untuk membuat data dummy. Untuk keperluan ujicoba modul login, kita perlu memasukan data user dan password kedalam database. Untuk itu buat database seeder untuk tabel user. Buka CLI, kemudian tulis perintah berikut. 

```CLI
php spark make:seeder UserSeeder
```

Selanjutnya,buka file **UserSeeder.php** yang berada dilokasi direktori **/app/Database/Seeds/UserSeeder.php** kemudian isi dengan kode berikut:

![User-seeder](https://github.com/Herli27052000/Lab11Web/blob/master/img/users-seeder.png)

**code UserSeeder.php**
```php
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run()
    {
        $model = model('UserModel');
		$model->insert([
            'username' => 'Herli',
			'useremail' => 'herli27052000@gmail.com',
			'userpassword' => password_hash('herli1234', PASSWORD_DEFAULT),
        ]);
    }
}
```

* Selanjutnya buka kembali CLI dan ketik perintah berikut:

```CLI
php spark db:seed UserSeeder
```

dan jalankan dibrowser,sebelum itu jangan lupa nyalahkan server nya dengan ketik pada CLI yaitu:
```CLI
php spark serve
```

* Tambahkan CSS untuk mempercantik tampilan login. Buka file **style.css** pada direktori **ci4\public\style.css**

![style-login](https://github.com/Herli27052000/Lab11Web/blob/master/img/login-user.png)

## UJI COBA LOGIN
Selanjutnya buka url: http://localhost:8080/user/login


## 6). MENAMBAHKAN AUTH FILTER
Selanjutnya membuat filter untuk halaman admin. Buat file baru dengan nama **Auth.php** pada direktori **app/Filters.**

![auth-filters](https://github.com/Herli27052000/Lab11Web/blob/master/img/auth-filters.png)

**code Auth.php**
```php
<?php namespace App\Filters;

use CodeIgniter\HTTP\RequestInterface;
use CodeIgniter\HTTP\ResponseInterface;
use CodeIgniter\Filters\FilterInterface;

class Auth implements FilterInterface
{
    public function before(RequestInterface $request, $arguments = null)
    {
        // jika user belum login
        if(! session()->get('logged_in')){
            // maka redirct ke halaman login
            return redirect()->to('/user/login');
        }
    }

    public function after(RequestInterface $request, ResponseInterface $response, $arguments = null)
    {
        // Do something here
    }
}
```

* Selanjutnya buka file **app/Config/Filters.php** tambahkan kode berikut:

```php
'auth' => App\Filters\Auth::class,
```
![auth-filtrescode](https://github.com/Herli27052000/Lab11Web/blob/master/img/auth-class.png)

* Selanjutnya buka file **app/Config/Routes.php** dan sesuaikan kodenya.

![filters-auth](https://github.com/Herli27052000/Lab11Web/blob/master/img/filters.png)

## 7). FUNGSI LOGOUT
Tambahkan method logout pada Controllers User seperti berikut

![function-logout](https://github.com/Herli27052000/Lab11Web/blob/master/img/logout.png)

```php
public function logout()
    {
        session()->destroy();
        return redirect()->to('/user/login');
    }
```

* Tambahkan menu logout diheader admin. Ke direktori **app/views/template** lalu buka file **admin_header.php** tambahkan kode berikut.

![admin-logout](https://github.com/Herli27052000/Lab11Web/blob/master/img/admin-logout.png)

```html
<a href="<?= base_url('/admin/logout');?>">Logout</a> 
```

* Dan Tambahkan route logout dengan cara ke direktori **app/Config/Routes.php** lalu tambahkan kode berikut.

![routes-logout](https://github.com/Herli27052000/Lab11Web/blob/master/img/filters.png)

```php
$routes->add('logout', 'User::logout');
```

## 8) PERCOBAAN AKSES MENU ADMIN
Buka url http://localhost:8080/admin/artikel ketika alamat tersebut diakses maka, akan dimunculkan halaman login.


Setelah itu akan dibawa ke halaman seperti dibawah.

![admin-artikel](https://github.com/Herli27052000/Lab11Web/blob/master/img/admin-artikel.png)



-----------------------------------------------------------------------------------------------------------------------------------
