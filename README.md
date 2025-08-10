# IT1030B11-Laravel
Tutorial para el curso de Programación Backend - IT UCSC

## Carpeta inicial
Debemos abrir desde Visual Studio Code la carpeta de XAMPP donde almacenamos todos nuestros proyectos. Esto es en `C://xampp/htdocs/xampp` en Windows.
Luego de abrir la carpeta de nuestros proyectos, desde la terminal de VS Code ya podemos crear nuestro proyecto de Laravel mediante un comando.

## ¿Como crear un proyecto en blanco en Laravel 8
```
composer create-project --prefer-dist laravel/laravel nombredesuproyecto "8.*"
```

Este comando creará una carpeta llamada `nombredesuproyecto` dentro de `C://xampp/htdocs/xampp`. Esta carpeta contiene un proyecto en blanco de Laravel 8.

Debemos ingresar a esta carpeta dentro de la terminal. Para esto debemos hacer:
```
cd nombredesuproyecto
```


---
## ¿Como levantar el proyecto creado en localhost en Laravel 8?
```
php artisan serve
```

---
## ¿Como correr las migraciones (la base de datos) por primera vez?
```
php artisan migrate
```