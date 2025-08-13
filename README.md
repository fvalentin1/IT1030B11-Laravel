# IT1030B11-Laravel
Tutorial para el curso de Programación Backend - IT UCSC

## Carpeta inicial
Debemos abrir desde Visual Studio Code la carpeta de XAMPP donde almacenamos todos nuestros proyectos. Esto es en `C://xampp/htdocs/xampp` en Windows.
Luego de abrir la carpeta de nuestros proyectos, desde la terminal de VS Code ya podemos crear nuestro proyecto de Laravel mediante un comando.

## ¿Como crear un proyecto en blanco en Laravel 8
```bash
composer create-project --prefer-dist laravel/laravel nombredesuproyecto "8.*"
```

Forzar instalación:
```bash
composer create-project --prefer-dist laravel/laravel nombredesuproyecto "8.*" --ignore-platform-reqs
```

Este comando creará una carpeta llamada `nombredesuproyecto` dentro de `C://xampp/htdocs/xampp`. Esta carpeta contiene un proyecto en blanco de Laravel 8.

Debemos ingresar a esta carpeta dentro de la terminal. Para esto debemos hacer:
```bash
cd nombredesuproyecto
```

## ¿Como levantar el proyecto creado en localhost en Laravel 8?
```bash
php artisan serve
```
Para detener el servicio, es necesario oprimir la combinación `Ctrl` + `C`.

## ¿Como configurar la base de datos?
Es necesario configurar la base de datos del proyecto, para esto debemos ubicar dentro de la carpeta de nuestro proyecto un archivo llamado `.env`, el cual contiene el valor de todas las variables de entorno del proyecto, entre ellas la configuración de la base de datos.

En caso de no encontrar el `.env`, se puede copiar el `.env.example` y se renombra como `.env` y se configura manualmente.

```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nombredesubasededatos #Por defecto tiene el nombre "laravel"
DB_USERNAME=root
DB_PASSWORD=
```

Despues de configurar el `.env`, nos debemos dirigir al **PhpMyAdmin** o a nuestro gestor de base de datos que estemos utilizando. En el caso de **PhpMyAdmin** debemos entrar desde el navegador a `localhost/phpmyadmin/` o tambien a `127.0.0.1/phpmyadmin/`.

Luego debemos crear una base de datos de *MySQL* en **PhpMyAdmin** que debe llamarse de la misma forma que se configuró en el archivo `.env`.


---
## ¿Como correr las migraciones (la base de datos) por primera vez?
```bash
php artisan migrate
```
Esto agregará tablas de usuarios, que por el momento no están en uso.

## Crear login de usuarios

### Instalar paquete Laravel/UI
```composer require laravel/ui "^3.0"```

### Generar Auth y estilos con Laravel
```php artisan ui bootstrap```
```php artisan ui bootstrap --auth```

### Instalar 
```npm install```
```npm run dev```
```npm run dev```


