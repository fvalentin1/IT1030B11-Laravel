# Relación uno a muchos `(Region 1:N Comunas)`

## 1. CRUD de Regiones
Es necesario tener el CRUD de Regiones ya creado, según lo visto la clase pasada.
Recordar que las comunas tenian las siguientes columnas:
- id
- code
- name

## 2. Crear el CRUD de Comunas
Las comunas deben tener las columnas:
- id
- nombre
- region_id

Como buena práctica de programación se utilizará "Communes" (en inglés) para referirnos a las Comunas.

### 2.1. Crear el modelo y migración de las comunas
Ejecutar en consola:
```php artisan make:model Commune -m```
Esto crea el modelo `app/Models/Commune.php` y la migración.

### 2.2. Configurar migración de las comunas
Abrir `database/migrations/xxxx_xx_xx_create_communes_table.php` y definir las columnas de la tabla:

```php
public function up()
{
    Schema::create('communes', function (Blueprint $table) {
        $table->id();
        $table->string('name', 100);
        $table->foreignId('region_id')->constrained('regions')->onDelete('cascade');
        $table->timestamps();
    });
}
```
Aquí `region_id` es la clave foránea que conecta con la tabla `regions`.

### 2.3 Configurar el modelo de comunas
En `app/Models/Commune.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Commune extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'region_id'];

    // Una comuna pertenece a una región
    public function region()
    {
        return $this->belongsTo(Region::class);
    }
}

```

### 2.4 Configurar el modelo de regiones
En `app/Models/Region.php` es necesario agregar:
```php
// Una región tiene muchas comunas
public function communes()
{
    return $this->hasMany(Commune::class);
}

```

### 2.5 Crear el controlador para comunas
En la terminal ingresamos:
```php artisan make:controller CommuneController --resource --model=Commune```

### 2.6 Configurar el controlador para comunas
En `app/Http/Controllers/CommuneController.php` colocar:
```php
<?php

namespace App\Http\Controllers;

use App\Models\Commune;
use App\Models\Region;
use Illuminate\Http\Request;
use Illuminate\Validation\Rule;

class CommuneController extends Controller
{
    public function index()
    {
        // Traemos todas las comunas con su región asociada
        $communes = Commune::with('region')->get();
        return view('communes.index', compact('communes'));
    }

    public function create()
    {
        // Necesitamos las regiones para el <select>
        $regions = Region::orderBy('name')->get();
        return view('communes.create', compact('regions'));
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'name'      => ['required', 'string', 'max:100'],
            'region_id' => ['required', 'exists:regions,id'],
        ]);

        Commune::create($data);

        return redirect()->route('communes.index')
            ->with('success', 'Commune created.');
    }

    public function show(Commune $commune)
    {
        return view('communes.show', compact('commune'));
    }

    public function edit(Commune $commune)
    {
        $regions = Region::orderBy('name')->get();
        return view('communes.edit', compact('commune', 'regions'));
    }

    public function update(Request $request, Commune $commune)
    {
        $data = $request->validate([
            'name'      => ['required', 'string', 'max:100', Rule::unique('communes', 'name')->ignore($commune->id)],
            'region_id' => ['required', 'exists:regions,id'],
        ]);

        $commune->update($data);

        return redirect()->route('communes.index')
            ->with('success', 'Commune updated.');
    }

    public function destroy(Commune $commune)
    {
        $commune->delete();

        return redirect()->route('communes.index')
            ->with('success', 'Commune deleted.');
    }
}
```

### 2.7 Crear vistas para comunas
Es necesario crear las vistas en `resources/views/communes`.

#### 2.7.1 Vista de Index
Crear el archivo `resources/views/communes/index.blade.php`.
```php
@extends('layouts.app')

@section('content')
<div class="container">
    <h1>Listado de Comunas</h1>
    <a href="{{ route('communes.create') }}" class="btn btn-primary mb-3">Agregar Nueva Comuna</a>

    @if(session('success'))
        <div class="alert alert-success">{{ session('success') }}</div>
    @endif

    <table class="table table-striped">
        <thead>
            <tr>
                <th>Nombre</th>
                <th>Región</th>
                <th>Acciones</th>
            </tr>
        </thead>
        <tbody>
            @forelse($communes as $commune)
            <tr>
                <td>{{ $commune->name }}</td>
                <td>{{ $commune->region->name }}</td>
                <td>
                    <a href="{{ route('communes.show', $commune) }}" class="btn btn-info btn-sm">Ver</a>
                    <a href="{{ route('communes.edit', $commune) }}" class="btn btn-warning btn-sm">Editar</a>
                    <form action="{{ route('communes.destroy', $commune) }}" method="POST" style="display:inline;">
                        @csrf
                        @method('DELETE')
                        <button type="submit" class="btn btn-danger btn-sm" onclick="return confirm('¿Eliminar esta comuna?')">Eliminar</button>
                    </form>
                </td>
            </tr>
            @empty
            <tr>
                <td colspan="3" class="text-center">No hay comunas registradas.</td>
            </tr>
            @endforelse
        </tbody>
    </table>
</div>
@endsection
```


#### 2.7.2 Vista de Create
Crear el archivo `resources/views/communes/create.blade.php`.
```php
@extends('layouts.app')

@section('content')
<div class="container">
    <h1>Agregar Comuna</h1>

    <form action="{{ route('communes.store') }}" method="POST">
        @csrf

        <div class="mb-3">
            <label for="name" class="form-label">Nombre</label>
            <input type="text" name="name" id="name" class="form-control" value="{{ old('name') }}">
            @error('name') <div class="text-danger">{{ $message }}</div> @enderror
        </div>

        <div class="mb-3">
            <label for="region_id" class="form-label">Región</label>
            <select name="region_id" id="region_id" class="form-select">
                <option value="">-- Seleccione una Región --</option>
                @foreach($regions as $region)
                    <option value="{{ $region->id }}" {{ old('region_id') == $region->id ? 'selected' : '' }}>
                        {{ $region->name }}
                    </option>
                @endforeach
            </select>
            @error('region_id') <div class="text-danger">{{ $message }}</div> @enderror
        </div>

        <button type="submit" class="btn btn-success">Guardar</button>
        <a href="{{ route('communes.index') }}" class="btn btn-secondary">Cancelar</a>
    </form>
</div>
@endsection
```


#### 2.7.3 Vista de Edit
Crear el archivo `resources/views/communes/edit.blade.php`.
```php
@extends('layouts.app')

@section('content')
<div class="container">
    <h1>Editar Comuna</h1>

    <form action="{{ route('communes.update', $commune) }}" method="POST">
        @csrf
        @method('PUT')

        <div class="mb-3">
            <label for="name" class="form-label">Nombre</label>
            <input type="text" name="name" id="name" class="form-control" value="{{ old('name', $commune->name) }}">
            @error('name') <div class="text-danger">{{ $message }}</div> @enderror
        </div>

        <div class="mb-3">
            <label for="region_id" class="form-label">Región</label>
            <select name="region_id" id="region_id" class="form-select">
                @foreach($regions as $region)
                    <option value="{{ $region->id }}" {{ old('region_id', $commune->region_id) == $region->id ? 'selected' : '' }}>
                        {{ $region->name }}
                    </option>
                @endforeach
            </select>
            @error('region_id') <div class="text-danger">{{ $message }}</div> @enderror
        </div>

        <button type="submit" class="btn btn-primary">Actualizar</button>
        <a href="{{ route('communes.index') }}" class="btn btn-secondary">Volver</a>
    </form>
</div>
@endsection
```

#### 2.7.4 Vista de Show
Crear el archivo `resources/views/communes/show.blade.php`.
```php
@extends('layouts.app')

@section('content')
<div class="container">
    <h1>Detalle de Comuna</h1>

    <div class="card">
        <div class="card-body">
            <h5 class="card-title">{{ $commune->name }}</h5>
            <p class="card-text"><strong>Región:</strong> {{ $commune->region->name }}</p>
        </div>
    </div>

    <a href="{{ route('communes.index') }}" class="btn btn-secondary mt-3">Volver</a>
</div>
@endsection
```

### 2.8 Configurar el archivo de rutas
En `routes/web.php` es importante agregar los controladores y generar funciones de ambas entidades relacionadas (Regiones y Comunas).

Importar controladores
```php
use App\Http\Controllers\RegionController;
use App\Http\Controllers\CommuneController;
```

Generar funciones de los controladores
```php
Route::resource('regions', RegionController::class);
Route::resource('communes', CommuneController::class);
```


### 2.9 Correr las migraciones
Finalmente se debe actualizar la base de datos actualizando las tablas con:
```php artisan migrate:fresh```

## 2.10 CRUD de comunas creado
Se debe levantar el proyecto con `php artisan serve`.
Será posible ingresar a nuestro CRUD en: ```http://127.0.0.1:8000/communes```, y podremos ver que de forma dinámica se pueden seleccionar las regiones ya ingresadas.

# Semillas o seeders
Se podrán crear semillas (o seeders). Las semillas son datos que se podrán ingresar por defecto al sistema, sin necesidad de crear estos registro en su respectivo CRUD. Esto es ideal en sistemas que necesiten ingresar datos precargados, ahorrando tiempo en ingresar los registros uno a uno, y además estandarizando el sistema.

## 3. Crear semillas para Regiones
Crearemos las regiones de la división política de Chile.

### 3.1 Crear el archivo de semilla
En la terminal, deben ingresar:
```php artisan make:seeder RegionSeeder```

Esto creará un archivo en ```database/seeders/RegionSeeder.php```.

### 3.2 Editar el archivo de semilla
Editar el archivo ```database/seeders/RegionSeeder.php```, ingresando un conjunto de datos en formato clave y valor.
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Region; // Llamada al modelo de Región

class RegionSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $regions = [
            ['code' => 'XV',   'name' => 'Arica and Parinacota'],
            ['code' => 'I',    'name' => 'Tarapaca'],
            ['code' => 'II',   'name' => 'Antofagasta'],
            ['code' => 'III',  'name' => 'Atacama'],
            ['code' => 'IV',   'name' => 'Coquimbo'],
            ['code' => 'V',    'name' => 'Valparaiso'],
            ['code' => 'VI',   'name' => "O'Higgins"],
            ['code' => 'VII',  'name' => 'Maule'],
            ['code' => 'XVI',  'name' => 'Ñuble'],
            ['code' => 'VIII', 'name' => 'Biobio'],
            ['code' => 'IX',   'name' => 'Araucania'],
            ['code' => 'X',    'name' => 'Los Lagos'],
            ['code' => 'XI',   'name' => 'Aysen'],
            ['code' => 'XII',  'name' => 'Magallanes and Chilean Antarctica'],
            ['code' => 'XIV',  'name' => 'Los Rios'],
            ['code' => 'RM', 'name' => 'Santiago Metropolitan'],
        ];

        foreach ($regions as $region) {
            Region::create($region);
        }
    }
}
```

### 3.3 Registrar la semilla
Abrir `database/seeders/DatabaseSeeder.php` y agrega la llamada dentro de `run()`:
```php
public function run()
    {
        $this->call([
            RegionSeeder::class,
        ]);
    }
```

### 3.4 Ejecutar la semilla
Para ejecutar la semilla es necesario tener creado y funcionando el CRUD de regiones.

Para ejecutar solo las semillas:
```php artisan db:seed```

Para ejecutar migraciones y semillas (borra todo y lo vuelve a crear):
```php artisan migrate:fresh --seed```

### 3.4 Verificar semillas
Si vamos a la vista de index de regiones en: `http://127.0.0.1:8000/regions`, podremos ver todas nuestras regiones ya creadas.

## 4. Crear semillas para Comunas
Crearemos algunas comunas pertenecientes a las regiones de la división política de Chile.

### 4.1 Crear el archivo de semilla
En la terminal, deben ingresar:
```php artisan make:seeder CommuneSeeder```

Esto creará un archivo en ```database/seeders/CommuneSeeder.php```.

### 4.2 Editar el archivo de semilla
Editar el archivo ```database/seeders/CommuneSeeder.php```, ingresando un conjunto de datos en formato clave y valor.

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Commune;

class CommuneSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $communes = [

            //Region de Arica y Parinacota
            ['name' => 'Arica', 'region_id' => 1],
            ['name' => 'Camarones', 'region_id' => 1],
            ['name' => 'Putre', 'region_id' => 1],
            ['name' => 'General Lagos', 'region_id' => 1],

            //Region de Tarapaca
            ['name' => 'Iquique', 'region_id' => 2],
            ['name' => 'Alto Hospicio', 'region_id' => 2],
            ['name' => 'Pozo Almonte', 'region_id' => 2],
            ['name' => 'Camiña', 'region_id' => 2],
            ['name' => 'Colchane', 'region_id' => 2],
            ['name' => 'Huara', 'region_id' => 2],
            ['name' => 'Pica', 'region_id' => 2],

            //Region de Antofagasta
            ['name' => 'Antofagasta', 'region_id' => 3],
            ['name' => 'Mejillones', 'region_id' => 3],
            ['name' => 'Sierra Gorda', 'region_id' => 3],
            ['name' => 'Taltal', 'region_id' => 3],
            ['name' => 'Calama', 'region_id' => 3],
            ['name' => 'Ollague', 'region_id' => 3],
            ['name' => 'San Pedro de Atacama', 'region_id' => 3],
            ['name' => 'Tocopilla', 'region_id' => 3],
            ['name' => 'María Elena', 'region_id' => 3],

            //Region de Atacama
            ['name' => 'Copiapó', 'region_id' => 4],
            ['name' => 'Caldera', 'region_id' => 4],
            ['name' => 'Tierra Amarilla', 'region_id' => 4],
            ['name' => 'Chañaral', 'region_id' => 4],
            ['name' => 'Diego de Almagro', 'region_id' => 4],
            ['name' => 'Vallenar', 'region_id' => 4],
            ['name' => 'Alto del Carmen', 'region_id' => 4],
            ['name' => 'Freirina', 'region_id' => 4],
            ['name' => 'Huasco', 'region_id' => 4],

            //Region de Coquimbo
            ['name' => 'La Serena', 'region_id' => 5],
            ['name' => 'Coquimbo', 'region_id' => 5],
            ['name' => 'Andacollo', 'region_id' => 5],
            ['name' => 'La Higuera', 'region_id' => 5],
            ['name' => 'Paiguano', 'region_id' => 5],
            ['name' => 'Vicuña', 'region_id' => 5],
            ['name' => 'Illapel', 'region_id' => 5],
            ['name' => 'Canela', 'region_id' => 5],
            ['name' => 'Los Vilos', 'region_id' => 5],
            ['name' => 'Salamanca', 'region_id' => 5],
            ['name' => 'Ovalle', 'region_id' => 5],
            ['name' => 'Combarbalá', 'region_id' => 5],
            ['name' => 'Monte Patria', 'region_id' => 5],
            ['name' => 'Punitaqui', 'region_id' => 5],
            ['name' => 'Río Hurtado', 'region_id' => 5],
            ['name' => 'Andacollo', 'region_id' => 5],
            ['name' => 'La Higuera', 'region_id' => 5],
            ['name' => 'Paiguano', 'region_id' => 5],
            ['name' => 'Vicuña', 'region_id' => 5],
            ['name' => 'Illapel', 'region_id' => 5],
            ['name' => 'Canela', 'region_id' => 5],
            ['name' => 'Los Vilos', 'region_id' => 5],
            ['name' => 'Salamanca', 'region_id' => 5],
            ['name' => 'Ovalle', 'region_id' => 5],
            ['name' => 'Combarbalá', 'region_id' => 5],
            ['name' => 'Monte Patria', 'region_id' => 5],
            ['name' => 'Punitaqui', 'region_id' => 5],
            ['name' => 'Río Hurtado', 'region_id' => 5],

            //Region de Valparaiso
            ['name' => 'Valparaíso', 'region_id' => 6],
            ['name' => 'Casablanca', 'region_id' => 6],
            ['name' => 'Concón', 'region_id' => 6],
            ['name' => 'Juan Fernández', 'region_id' => 6],
            ['name' => 'Puchuncaví', 'region_id' => 6],
            ['name' => 'Quintero', 'region_id' => 6],
            ['name' => 'Viña del Mar', 'region_id' => 6],
            ['name' => 'Isla de Pascua', 'region_id' => 6],
            ['name' => 'Los Andes', 'region_id' => 6],
            ['name' => 'Rinconada', 'region_id' => 6],
            ['name' => 'La Ligua', 'region_id' => 6],
            ['name' => 'Cabildo', 'region_id' => 6],
            ['name' => 'Papudo', 'region_id' => 6],
            ['name' => 'Petorca', 'region_id' => 6],
            ['name' => 'Zapallar', 'region_id' => 6],
            ['name' => 'Quillota', 'region_id' => 6],
            ['name' => 'Calera', 'region_id' => 6],
            ['name' => 'Hijuelas', 'region_id' => 6],
            ['name' => 'Nogales', 'region_id' => 6],
            ['name' => 'San Antonio', 'region_id' => 6],
            ['name' => 'Algarrobo', 'region_id' => 6],
            ['name' => 'Cartagena', 'region_id' => 6],
            ['name' => 'El Quisco', 'region_id' => 6],
            ['name' => 'El Tabo', 'region_id' => 6],
            ['name' => 'Santo Domingo', 'region_id' => 6],
            ['name' => 'San Felipe', 'region_id' => 6],
            ['name' => 'Catemu', 'region_id' => 6],
            ['name' => 'Concón', 'region_id' => 6],
            ['name' => 'Juan Fernández', 'region_id' => 6],
            ['name' => 'Puchuncaví', 'region_id' => 6],
            ['name' => 'Quintero', 'region_id' => 6],
            ['name' => 'Viña del Mar', 'region_id' => 6],
            ['name' => 'Isla de Pascua', 'region_id' => 6],
            ['name' => 'Los Andes', 'region_id' => 6],
            ['name' => 'Rinconada', 'region_id' => 6],
            ['name' => 'La Ligua', 'region_id' => 6],
            ['name' => 'Cabildo', 'region_id' => 6],
            ['name' => 'Papudo', 'region_id' => 6],
            ['name' => 'Petorca', 'region_id' => 6],
            ['name' => 'Zapallar', 'region_id' => 6],
            ['name' => 'Quillota', 'region_id' => 6],
            ['name' => 'Calera', 'region_id' => 6],
            ['name' => 'Hijuelas', 'region_id' => 6],
            ['name' => 'La Cruz', 'region_id' => 6],
            ['name' => 'Nogales', 'region_id' => 6],
            ['name' => 'San Antonio', 'region_id' => 6],
            ['name' => 'Algarrobo', 'region_id' => 6],
            ['name' => 'Cartagena', 'region_id' => 6],
            ['name' => 'El Quisco', 'region_id' => 6],
            ['name' => 'El Tabo', 'region_id' => 6],
            ['name' => 'Santo Domingo', 'region_id' => 6],
            ['name' => 'San Felipe', 'region_id' => 6],
            ['name' => 'Llaillay', 'region_id' => 6],
            ['name' => 'Panquehue', 'region_id' => 6],
            ['name' => 'Putaendo', 'region_id' => 6],

            //Region de O'Higgins
            ['name' => 'Rancagua', 'region_id' => 7],
            ['name' => 'Codegua', 'region_id' => 7],
            ['name' => 'Coinco', 'region_id' => 7],
            ['name' => 'Coltauco', 'region_id' => 7],
            ['name' => 'Doñihue', 'region_id' => 7],
            ['name' => 'Graneros', 'region_id' => 7],

            //Region de Maule
            ['name' => 'Talca', 'region_id' => 8],
            ['name' => 'Constitución', 'region_id' => 8],
            ['name' => 'Curepto', 'region_id' => 8],
            ['name' => 'Empedrado', 'region_id' => 8],
            ['name' => 'Maule', 'region_id' => 8],
            ['name' => 'Pelarco', 'region_id' => 8],
            ['name' => 'Pencahue', 'region_id' => 8],
            ['name' => 'Río Claro', 'region_id' => 8],
            ['name' => 'San Clemente', 'region_id' => 8],
            ['name' => 'San Rafael', 'region_id' => 8],
            ['name' => 'Cauquenes', 'region_id' => 8],
            ['name' => 'Chanco', 'region_id' => 8],
            ['name' => 'Pelluhue', 'region_id' => 8],
            ['name' => 'Curicó', 'region_id' => 8],
            ['name' => 'Hualañé', 'region_id' => 8],
            ['name' => 'Constitución', 'region_id' => 8],
            ['name' => 'Curepto', 'region_id' => 8],
            ['name' => 'Empedrado', 'region_id' => 8],
            ['name' => 'Maule', 'region_id' => 8],
            ['name' => 'Pelarco', 'region_id' => 8],
            ['name' => 'Pencahue', 'region_id' => 8],
            ['name' => 'Río Claro', 'region_id' => 8],
            ['name' => 'San Clemente', 'region_id' => 8],
            ['name' => 'San Rafael', 'region_id' => 8],
            ['name' => 'Cauquenes', 'region_id' => 8],
            ['name' => 'Chanco', 'region_id' => 8],
            ['name' => 'Pelluhue', 'region_id' => 8],
            ['name' => 'Curicó', 'region_id' => 8],
            ['name' => 'Hualañé', 'region_id' => 8],

            //Region de Ñuble
            ['name' => 'Chillán', 'region_id' => 9],
            ['name' => 'Chillán Viejo', 'region_id' => 9],
            ['name' => 'El Carmen', 'region_id' => 9],
            ['name' => 'Pemuco', 'region_id' => 9],
            ['name' => 'Pinto', 'region_id' => 9],
            ['name' => 'Quillón', 'region_id' => 9],

            //Region de Biobio
            ['name' => 'Concepción', 'region_id' => 10],
            ['name' => 'Coronel', 'region_id' => 10],
            ['name' => 'Chiguayante', 'region_id' => 10],
            ['name' => 'Florida', 'region_id' => 10],
            ['name' => 'Hualpén', 'region_id' => 10],
            ['name' => 'Hualqui', 'region_id' => 10],
            ['name' => 'Lota', 'region_id' => 10],
            ['name' => 'Penco', 'region_id' => 10],
            ['name' => 'San Pedro de la Paz', 'region_id' => 10],
            ['name' => 'Santa Juana', 'region_id' => 10],
            ['name' => 'Talcahuano', 'region_id' => 10],
            ['name' => 'Tomé', 'region_id' => 10],

        ];

        foreach ($communes as $commune) {
            Commune::create($commune);
        }
    }
}
```

### 4.3 Registrar la semilla
Abrir `database/seeders/DatabaseSeeder.php` y agrega la llamada dentro de `run()`:
```php
public function run()
    {
        $this->call([
            RegionSeeder::class,
            CommuneSeeder::class,
        ]);
    }
```

### 4.4 Ejecutar la semilla
Para ejecutar la semilla es necesario tener creado y funcionando el CRUD de regiones y el CRUD de comunas.

Para ejecutar solo las semillas:
```php artisan db:seed```

Para ejecutar migraciones y semillas (borra todo y lo vuelve a crear):
```php artisan migrate:fresh --seed```

### 4.4 Verificar semillas
Si vamos a la vista de index de comunas en: `http://127.0.0.1:8000/communes`, podremos ver todas nuestras comunas ya creadas.