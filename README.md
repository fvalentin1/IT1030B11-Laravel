# IT1030B11-Laravel
Tutorial para el curso de Programación Backend - IT UCSC

## Carpeta inicial
Debemos abrir desde Visual Studio Code la carpeta de XAMPP donde almacenamos todos nuestros proyectos. Esto es en `C://xampp/htdocs/xampp` en Windows.
Luego de abrir la carpeta de nuestros proyectos, desde la terminal de VS Code ya podemos crear nuestro proyecto de Laravel mediante un comando.

## ¿Como crear un proyecto en blanco en Laravel 8
```bash
composer create-project --prefer-dist laravel/laravel nombredesuproyecto "8.*"
```

Forzar instalación en caso de error de versión de PHP:
```bash
composer create-project --prefer-dist laravel/laravel nombredesuproyecto "8.*" --ignore-platform-reqs
```

Este comando creará una carpeta llamada `nombredesuproyecto` dentro de `C://xampp/htdocs/xampp`. Esta carpeta contiene un proyecto en blanco de Laravel 8.

Si se marca un error al descomprimir los archivos necesarios del proyecto, es necesario ir a la dirección `C://xampp/php` y editar con el Bloc de Notas o con VS Code el archivo `php.ini`.
Una vez abierto el editor, buscamos con `F3` o `Ctrl + F` el texto **extension=zip**.
Debería encontrar algo como ésto:
```ini
;extension=soap
;extension=sockets
;extension=sodium
;extension=sqlite3
;extension=tidy
;extension=xsl
;extension=zip
```

Si la línea ```extension=zip``` se encuentra con el punto y coma (```;```) al inicio, se debe borrar el punto y coma y luego guardar el archivo de texto. (Es aproximádamente, la línea 962).

---

Si el proyecto fue creado exitosamente, veremos el mensaje:
```Application key set successfully.```

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

### Instalar paquete Laravel/UI:
```bash
composer require laravel/ui "^3.0"
```

Forzar instalación en caso de error de versión de PHP:
```bash
composer require laravel/ui:^3.0 --dev --ignore-platform-reqs
```

### Generar Auth y estilos con Laravel:

Esto instalará Bootstrap en nuestro proyecto 
```bash
php artisan ui bootstrap
```
Esto instalará un login funcional básico en nuestro proyecto
```bash
php artisan ui bootstrap --auth
```

### Compilar Laravel Mix

Para esto debemos abrir una consola de NodeJS en la ruta de nuestro proyecto, en `C://xampp/htdocs/xampp/nombredesuproyecto`.
La consola/terminal de NodeJS se llama `Node.js command prompt`.

Instalar dependencias JS/CSS necesarias para compilar.
```bash
npm install
```
Compilar los archivos y dejarlos listos para el navegador.
```bash
npm run dev
```
Reduntante, pero asegura la ejecución de Laravel Mix.
```bash
npm run dev
```

### Actualizar la base de datos
Esto borrará la base de datos por completo, y aplicará la última configuración realizada.
```bash
php artisan migrate:fresh
```

### Optimize
En caso de que no se vean cambios en alguna de las vistas, es necesario eliminar la caché y algunas configuraciones temporales mediante:
```bash
php artisan optimize
```


