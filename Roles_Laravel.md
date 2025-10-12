# Roles en Laravel

## Paso 1: Instalar paquete Spatie Laravel Permissions
Será necesario ejecutar mediante terminal el siguiente comando:
```bash
composer require spatie/laravel-permission
```

Luego se deben publicar y migrar las tablas necesarias con:
```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

Con `php artisan migrate` se publicarán las siguientes tablas en la base de datos:
- `roles`
- `permissions`
- `model_has_roles`
- `model_has_permissions`
- `role_has_permissions`

## Paso 2: Configurar `Kernel.php`
Abrir `app/Http/Kernel.php` y agregar las siguientes líneas donde corresponda:
```php
protected $routeMiddleware = [
    // ... otros middlewares ya existentes

    'role' => \Spatie\Permission\Middleware\RoleMiddleware::class,
    'permission' => \Spatie\Permission\Middleware\PermissionMiddleware::class,
    'role_or_permission' => \Spatie\Permission\Middleware\RoleOrPermissionMiddleware::class,
];
```

## Paso 3: Configurar el modelo `User`
Abrir `app/Models/User.php` y agrega el trait de Spatie:
```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];
}

```

## Paso 4: Crear semillas (Seeders)

### 4.1 Seeders para Roles
Creamos un seeder para los roles:
```bash
php artisan make:seeder RoleSeeder
```

Editamos `database/seeders/RoleSeeder.php` creado recientemente:
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Role;

class RoleSeeder extends Seeder
{
    public function run()
    {
        Role::create(['name' => 'admin']);
        Role::create(['name' => 'user']);
    }
}
```

### 4.2 Seeder para crear un usuario
Por terminal ingresamos:
```bash
php artisan make:seeder UserSeeder
```

Editamos `database/seeders/UserSeeder.php` creado recientemente:
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run()
    {
        User::create([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

### 4.3 Seeder para asignar rol al usuario
Por terminal ingresamos:
```bash
php artisan make:seeder AssignRoleSeeder
```

Editamos `database/seeders/AssignRoleSeeder.php` creado recientemente:
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;

class AssignRoleSeeder extends Seeder
{
    public function run()
    {
        $user = User::where('email', 'admin@example.com')->first();
        $user->assignRole('admin');
    }
}
```

### 4.4 Llamada de Seeders
Debemos llamar a los seeders desde su archivo raiz ubicado en `database/seeders/DatabaseSeeder.php`.

Editamos el archivo:
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $this->call([
            RoleSeeder::class,
            UserSeeder::class,
            AssignRoleSeeder::class,
        ]);
    }
}
```

Ahora ya podemos migrar todo con `php artisan migrate:fresh --seed`, nuestro sistema cargará las tablas necesarias y además ingresará las semillas creadas.

## Paso 5: Usuarios nuevos con rol de usuario común
Debemos editar el archivo `app/Http/Controllers/Auth/RegisterController.php` para que al crear un usuario a través del formulario de registro, este por defecto tenga el rol `user` (usuario común):

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Providers\RouteServiceProvider;
use App\Models\User;
use Illuminate\Foundation\Auth\RegistersUsers;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;

use Spatie\Permission\Models\Role;

class RegisterController extends Controller
{
    protected function create(array $data)
    {
        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
        ]);

        // Asignar el rol "user" al nuevo usuario
        $user->assignRole('user');

        return $user;
    }
}
```

**Requisitos previos:** el modelo `User` debe usar el *trait* `HasRoles` (en `app/Models/User.php`):

```php
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
    // ...
}

```

## Paso 6: Proteger vistas
En nuestras vistas Blade, podremos mostrar o no un contenido según nuestro rol en el sistema.

```blade
@role('admin')
    <h1>Welcome, Admin!</h1>
@endrole

@hasrole('user')
    <p>This content is for regular users</p>
@endhasrole

@unlessrole('admin')
    <p>You are not an admin.</p>
@endunlessrole
```

## Paso 7: Proteger rutas
En `routes/web.php`, puedes agrupar rutas con middleware de roles:
```php
<?php

use Illuminate\Support\Facades\Route;

use App\Http\Controllers\CarController;
use App\Http\Controllers\RegionController;
use App\Http\Controllers\CommuneController;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/

Route::get('/', function () {
    return view('welcome');
});

Auth::routes();

Route::get('/home', [App\Http\Controllers\HomeController::class, 'index'])->name('home');

// Solo usuarios autenticados pueden acceder a regions y communes
Route::middleware(['auth'])->group(function () {
    Route::resource('regions', RegionController::class);
    Route::resource('communes', CommuneController::class);

    // Solo admin puede acceder a cars
    Route::middleware(['role:admin'])->group(function () {
        Route::resource('cars', CarController::class);
    });
});
```

### Tambien es posible usar múltiples roles (ejemplo):
```php
Route::middleware(['role:admin|user'])->group(function () {
    Route::get('/dashboard', function () {
        return view('dashboard');
    });
});
```

### Y tambien una vista en específico (ejemplo):
```php
Route::get('/admin/config', [AdminController::class, 'config'])
    ->middleware('role:admin')
    ->name('admin.config');
```


