# Como crear una API REST

## Paso 0: Tener Modelo, Controlador, Vistas, Migraciones creadas

Necesitamos tener nuestra entidad con su *CRUD* ya funcionando correctamente. Para este ejemplo, será la entidad `Student`.


## Paso 1: Crear un controlador de API

Para esto debemos crear un controlador común y corriente, con la diferencia, de que en vez de retornar una vista, retornaremos un JSON.

```bash
php artisan make:controller Api/StudentController --resource
```

Esto nos creará un `StudentController.php` dentro de `Http/Controllers/Api`.

## Paso 2: Configurar las rutas en `routes/api.php`

En `routes/api.php` debemos llamar a nuestro controlador, además de indicar sus recursos. Debemos hacerlo bajo un alias llamado "api", esto nos pondrá como prefijo en la URL el alias, de forma de diferenciarlo de las rutas normales (Controladores comunes que retornan vistas y que tienen el mismo nombre de archivo).


```php
use App\Http\Controllers\Api\StudentController;

Route::group(['as' => 'api.'], function () {
    Route::apiResource('courses', StudentController::class);
});
```

Hasta aquí tenemos creada una API REST pública, lista para su uso. Podemos realizar un método `GET` a `http://127.0.0.1:8000/api/students`, y esto debería devolvernos el listado de estudiantes.

Para esto podemos utilizar **Postman**.

## Paso 3: Crear una API privada

Se utilizará **Laravel Sanctum** (que viene ya incluido en el proyecto). Sanctum funciona solicitando un **Token** que validará al usuario que quiere acceder a los recursos.

Para esto, debemos tener lo anterior, pero se modifican los recursos en el controlador y las rutas.

En `routes/api.php` debemos tener:

```php
use App\Http\Controllers\Api\StudentController;

Route::group(['as' => 'api.'], function () {
    Route::apiResource('students', StudentController::class)->middleware(['auth:sanctum', 'role:admin']);
});
```

Esto hará que para realizar una petición a la API, se cumplan 2 condiciones:

- El usuario debe estar autenticado (Usuario registrado en el sistema).
- El usuario debe tener los permisos para acceder al recurso, en este caso tener el Rol de `admin`.

Sin embargo, Sanctum tiene una dificultad, la cual es rescatar el Token del usuario al momento de crearlo. Este Token solo es visible en este momento, luego en la base de datos se registra un *Hash* de este Token, y no se puede descifrar.

## Paso 4: Recuperar el Token

Para esto debemos ir a dos lugares donde registramos usuarios en el sistema, el primero es en los `Seeders`, y el segundo es el `RegisterController.php`.

### 4.1: Seeders

Luego de registrar los datos de nuestro usuario en un `UserSeeder.php`, debemos asignarle un Rol teniendo tambien un `RoleSeeder.php`.

Entonces vamos a suponer que ingresaremos el siguiente usuario mediante `UserSeeder.php`:
```php
public function run()
    {
        User::create([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
            'password' => Hash::make('123'),
        ]);
    }
```

Y los siguientes roles en `RoleSeeder.php`:
```php
public function run()
    {
        Role::create(['name' => 'admin']);
        Role::create(['name' => 'user']);
    }
```

En nuestro `AssignRoleSeeder.php` debemos enlazar nuestro usuario con el rol `admin` (o el que deseen). Y luego vamos a volver a crear un Token para el usuario, el cual contendrá el rol asignado.

Rescataremos el Token de forma de guardarlo en un archivo de texto que guardaremos en `storage/tokens/token_admin.txt`. Guardar archivos de este nivel de sensibilidad **no es una buena práctica**, sin embargo para ejemplo práctico se guardará este Token de forma local en el dispositivo. Es importante considerar que, en caso de querer guardar estos Tokens, se haga en un lugar y dispositivos seguros, y que la ruta de `storage/tokens` se encuentre registrada en el archivo `.gitignore`.


```php
public function run()
    {
        // Find the user by email
        $user = User::where('email', 'admin@example.com')->first();

        // Assign role
        $user->assignRole('admin');

        // Create a personal access token and save it to storage for testing (only for local/dev)
        try {
            $token = $user->createToken('seeder-token')->plainTextToken;
            $path = storage_path('token_admin.txt');
            file_put_contents($path, $token);
            if ($this->command) {
                $this->command->info('Admin token created and saved to: ' . $path);
            }
        } catch (\Exception $e) {
            if ($this->command) {
                $this->command->error('Failed to create token: ' . $e->getMessage());
            }
        }
    }
```

### 4.2: RegisterController

Acá haremos algo similar, guardaremos dentro de la misma ruta, los Tokens de los usuarios creados mediante el formulario de registro.

En `Auth/RegisterController.php`:
```php
protected function create(array $data)
    {
        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
        ]);

        // Asignar el rol "user" al nuevo usuario
        $user->assignRole('user');

        // Generar token y guardar en archivo por usuario (solo en entornos no productivos)
        try {
            if (! app()->environment('production')) {
                $plainToken = $user->createToken('register-token')->plainTextToken;

                $dir = storage_path('tokens');
                if (! is_dir($dir)) {
                    mkdir($dir, 0700, true);
                }

                $file = $dir . DIRECTORY_SEPARATOR . "user_{$user->id}.txt";
                // Guardar token y metadatos
                $content = "token: {$plainToken}\n" .
                           "user_id: {$user->id}\n" .
                           "email: {$user->email}\n" .
                           "role: user\n" .
                           "created_at: " . now()->toDateTimeString() . "\n";

                file_put_contents($file, $content);
                @chmod($file, 0600);
            }
        } catch (\Throwable $e) {
            // No interrumpir el registro por fallo al guardar token; lo registramos
            \Log::error('No se pudo generar/guardar token para user '.$user->id.': '.$e->getMessage());
        }

        return $user;
    }
```

## Paso 5: Acceder a una solicitud privada mediante Postman

En Postman debemos hacer lo siguiente:

- Configurar el método correspondiente y la URL.
- Debemos configurar en **Authorizatión**, el *tipo de autorización* `Bearer Token`, y en *Token* el valor del token rescatado anteriormente en los archivos de texto.
- En **Headers**, crear una *Key* llamada `Accept` y su value `application/json`.
- En **Body**, colocar el contenido según el método que corresponda.
- Y enviar.

Podemos obtener tres posibles respuestas:
- Code `200`: Nos responderá con los datos solicitados correctamente.  
- Code `401`: Nos responderá que ingresamos sin un token, o uno invalido.
- Code `403`: Nos responderá que ingresamos un token válido, pero no con los permisos necesarios.





