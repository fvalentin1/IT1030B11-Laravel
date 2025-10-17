# CRUD Relación Muchos a Muchos (n:m)

### Relación Estudiantes, Cursos y Matrícula
Haremos una relación muchos a muchos entre dos entidades y una tabla intermedia (pivote).

**Estudiantes:**

- `Nombre (string)`
- `Año de ingreso (year)`
- `Estado (enum)` Valor debe pertenecer al conjunto: `Regular`, `Suspendido`, `Egresado`, `Titulado`.

**Cursos:**

- `Código (string)`
- `Nombre (string)`
- `Créditos (integer)`

**Matrícula:**

- `ID Estudiante (FK)`
- `ID Curso (FK)`
- `Año (year)`
- `Semestre (integer)`


## 1. Crear las migraciones

### 1.1. Crear migración para `students`
```bash
php artisan make:migration create_students_table
```
Luego se debe configurar la migración creada:
```php
public function up()
{
    Schema::create('students', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->year('year_admission');
        $table->enum('status', ['Regular', 'Suspendido', 'Egresado', 'Titulado']);
        $table->timestamps();
    });
}
```

### 1.2. Crear migración para `courses`
```bash
php artisan make:migration create_courses_table
```
Luego se debe configurar la migración creada:
```php
public function up()
{
    Schema::create('courses', function (Blueprint $table) {
        $table->id();
        $table->string('code')->unique();
        $table->string('name');
        $table->integer('credits');
        $table->timestamps();
    });
}
```

### 1.3. Crear migración para el pivote `course_student`
```bash
php artisan make:migration create_course_student_table
```
Luego se debe configurar la migración creada:
```php
public function up()
{
    Schema::create('course_student', function (Blueprint $table) {
        $table->id();
        $table->foreignId('student_id')->constrained()->onDelete('cascade');
        $table->foreignId('course_id')->constrained()->onDelete('cascade');
        $table->year('year');
        $table->integer('semester'); // 1 o 2
        $table->timestamps();

        $table->unique(['student_id', 'course_id', 'year', 'semester']);
    });
}
```

## 2. Crear modelos

Creamos los modelos para ambas entidades, pero no para el pivote:
```bash
php artisan make:model Student
php artisan make:model Course
```

### 2.1 Configurar modelo `Student`
```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Student extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'year_admission', 'status'];

    public function courses()
    {
        return $this->belongsToMany(Course::class)
                    ->withPivot('year', 'semester')
                    ->withTimestamps();
    }
}
```


### 2.2 Configurar modelo `Course`
```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Course extends Model
{
    use HasFactory;

    protected $fillable = ['code', 'name', 'credits'];

    public function students()
    {
        return $this->belongsToMany(Student::class)
                    ->withPivot('year', 'semester')
                    ->withTimestamps();
    }
}
```

## 3. Generar Controladores
Generamos los 3 controladores necesarios, incluyendo el pivote:
```bash
php artisan make:controller StudentController --resource
php artisan make:controller CourseController --resource
php artisan make:controller CourseStudentController --resource
```

### 3.1. StudentController
```php
<?php
namespace App\Http\Controllers;

use App\Models\Student;
use Illuminate\Http\Request;

class StudentController extends Controller
{
    public function index()
    {
        $students = Student::all();
        return view('students.index', compact('students'));
    }

    public function create()
    {
        return view('students.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'year_admission' => 'required|integer',
            'status' => 'required|in:Regular,Suspendido,Egresado,Titulado',
        ]);

        Student::create($request->all());
        return redirect()->route('students.index');
    }

    public function edit(Student $student)
    {
        return view('students.edit', compact('student'));
    }

    public function update(Request $request, Student $student)
    {
        $request->validate([
            'name' => 'required',
            'year_admission' => 'required|integer',
            'status' => 'required|in:Regular,Suspendido,Egresado,Titulado',
        ]);

        $student->update($request->all());
        return redirect()->route('students.index');
    }

    public function destroy(Student $student)
    {
        $student->delete();
        return redirect()->route('students.index');
    }
}
```


### 3.2. CourseController
```php
<?php
namespace App\Http\Controllers;

use App\Models\Course;
use Illuminate\Http\Request;

class CourseController extends Controller
{
    public function index()
    {
        $courses = Course::all();
        return view('courses.index', compact('courses'));
    }

    public function create()
    {
        return view('courses.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'code' => 'required|unique:courses',
            'name' => 'required',
            'credits' => 'required|integer|min:1',
        ]);

        Course::create($request->all());
        return redirect()->route('courses.index');
    }

    public function edit(Course $course)
    {
        return view('courses.edit', compact('course'));
    }

    public function update(Request $request, Course $course)
    {
        $request->validate([
            'code' => 'required|unique:courses,code,' . $course->id,
            'name' => 'required',
            'credits' => 'required|integer|min:1',
        ]);

        $course->update($request->all());
        return redirect()->route('courses.index');
    }

    public function destroy(Course $course)
    {
        $course->delete();
        return redirect()->route('courses.index');
    }
}
```

### 3.3. CourseStudentController
```php
<?php
namespace App\Http\Controllers;

use App\Models\Course;
use App\Models\Student;
use Illuminate\Http\Request;

class CourseStudentController extends Controller
{
    public function index()
    {
        $enrollments = \DB::table('course_student')
            ->join('students', 'students.id', '=', 'course_student.student_id')
            ->join('courses', 'courses.id', '=', 'course_student.course_id')
            ->select('course_student.*', 'students.name as student', 'courses.name as course')
            ->get();

        return view('enrollments.index', compact('enrollments'));
    }

    public function create()
    {
        $students = Student::all();
        $courses = Course::all();
        return view('enrollments.create', compact('students', 'courses'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'student_id' => 'required|exists:students,id',
            'course_id' => 'required|exists:courses,id',
            'year' => 'required|integer',
            'semester' => 'required|integer|between:1,2',
        ]);

        $student = Student::find($request->student_id);
        $student->courses()->attach($request->course_id, [
            'year' => $request->year,
            'semester' => $request->semester
        ]);

        return redirect()->route('enrollments.index');
    }

    public function destroy($id)
    {
        \DB::table('course_student')->where('id', $id)->delete();
        return redirect()->route('enrollments.index');
    }
}
```

## 4. Definir Rutas
Se deben definir las rutas e invocar a sus respectivos modelos:
```php
use App\Http\Controllers\StudentController;
use App\Http\Controllers\CourseController;
use App\Http\Controllers\CourseStudentController;

Route::resource('students', StudentController::class);
Route::resource('courses', CourseController::class);
Route::resource('enrollments', CourseStudentController::class);
```

No olvidar que cuando se hacen modificaciones a las rutas, debemos ejecutar:
```bash
php artisan optimize
```

## 5. Crear las vistas
Deberiamos tener la siguiente estructura para las vistas:
```pgsql
resources/views/
 ├── students/
 │    ├── index.blade.php
 │    ├── create.blade.php
 │    └── edit.blade.php
 ├── courses/
 │    ├── index.blade.php
 │    ├── create.blade.php
 │    └── edit.blade.php
 └── enrollments/
      ├── index.blade.php
      └── create.blade.php
```

### 5.1. Vistas de Students

#### 5.1.1 Index:
```blade
@extends('layouts.admin')

@section('content')
<div class="container mt-4">
    <div class="d-flex justify-content-between align-items-center mb-3">
        <h2>Students</h2>
        <a href="{{ route('students.create') }}" class="btn btn-primary">Add Student</a>
    </div>

    <table class="table table-striped align-middle">
        <thead class="table-dark">
            <tr>
                <th>Name</th>
                <th>Year Admission</th>
                <th>Status</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @forelse ($students as $student)
                <tr>
                    <td>{{ $student->name }}</td>
                    <td>{{ $student->year_admission }}</td>
                    <td><span class="badge bg-info">{{ $student->status }}</span></td>
                    <td>
                        <a href="{{ route('students.edit', $student) }}" class="btn btn-warning btn-sm">Edit</a>
                        <form action="{{ route('students.destroy', $student) }}" method="POST" class="d-inline">
                            @csrf
                            @method('DELETE')
                            <button class="btn btn-danger btn-sm" onclick="return confirm('Are you sure?')">Delete</button>
                        </form>
                    </td>
                </tr>
            @empty
                <tr><td colspan="4" class="text-center">No students found.</td></tr>
            @endforelse
        </tbody>
    </table>
</div>
@endsection
```

#### 5.1.2 Create:
```blade
@extends('layouts.admin')

@section('content')
<div class="container mt-4">
    <h2>Add Student</h2>
    <form action="{{ route('students.store') }}" method="POST" class="mt-3">
        @csrf
        <div class="mb-3">
            <label class="form-label">Name</label>
            <input type="text" name="name" class="form-control" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Year Admission</label>
            <input type="number" name="year_admission" class="form-control" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Status</label>
            <select name="status" class="form-select" required>
                <option value="">Select status...</option>
                <option>Regular</option>
                <option>Suspendido</option>
                <option>Egresado</option>
                <option>Titulado</option>
            </select>
        </div>

        <button type="submit" class="btn btn-success">Save</button>
        <a href="{{ route('students.index') }}" class="btn btn-secondary">Back</a>
    </form>
</div>
@endsection
```

#### 5.1.3 Edit:
```blade
@extends('layouts.admin')

@section('content')
<div class="container mt-4">
    <h2>Edit Student</h2>
    <form action="{{ route('students.update', $student) }}" method="POST" class="mt-3">
        @csrf
        @method('PUT')

        <div class="mb-3">
            <label class="form-label">Name</label>
            <input type="text" name="name" class="form-control" value="{{ $student->name }}" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Year Admission</label>
            <input type="number" name="year_admission" class="form-control" value="{{ $student->year_admission }}" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Status</label>
            <select name="status" class="form-select" required>
                <option {{ $student->status == 'Regular' ? 'selected' : '' }}>Regular</option>
                <option {{ $student->status == 'Suspendido' ? 'selected' : '' }}>Suspendido</option>
                <option {{ $student->status == 'Egresado' ? 'selected' : '' }}>Egresado</option>
                <option {{ $student->status == 'Titulado' ? 'selected' : '' }}>Titulado</option>
            </select>
        </div>

        <button type="submit" class="btn btn-success">Update</button>
        <a href="{{ route('students.index') }}" class="btn btn-secondary">Back</a>
    </form>
</div>
@endsection
```



### 5.2. Vistas de Courses

#### 5.1.1 Index:
```blade
@extends('layouts.admin')

@section('content')
<div class="container mt-4">
    <div class="d-flex justify-content-between align-items-center mb-3">
        <h2>Courses</h2>
        <a href="{{ route('courses.create') }}" class="btn btn-primary">Add Course</a>
    </div>

    <table class="table table-striped align-middle">
        <thead class="table-dark">
            <tr>
                <th>Code</th>
                <th>Name</th>
                <th>Credits</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @forelse ($courses as $course)
                <tr>
                    <td>{{ $course->code }}</td>
                    <td>{{ $course->name }}</td>
                    <td>{{ $course->credits }}</td>
                    <td>
                        <a href="{{ route('courses.edit', $course) }}" class="btn btn-warning btn-sm">Edit</a>
                        <form action="{{ route('courses.destroy', $course) }}" method="POST" class="d-inline">
                            @csrf
                            @method('DELETE')
                            <button class="btn btn-danger btn-sm" onclick="return confirm('Are you sure?')">Delete</button>
                        </form>
                    </td>
                </tr>
            @empty
                <tr><td colspan="4" class="text-center">No courses found.</td></tr>
            @endforelse
        </tbody>
    </table>
</div>
@endsection
```

#### 5.1.2 Create:
```blade
@extends('layouts.admin')

@section('content')
<div class="container mt-4">
    <h2>Add Course</h2>
    <form action="{{ route('courses.store') }}" method="POST" class="mt-3">
        @csrf
        <div class="mb-3">
            <label class="form-label">Code</label>
            <input type="text" name="code" class="form-control" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Name</label>
            <input type="text" name="name" class="form-control" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Credits</label>
            <input type="number" name="credits" class="form-control" required min="1">
        </div>

        <button type="submit" class="btn btn-success">Save</button>
        <a href="{{ route('courses.index') }}" class="btn btn-secondary">Back</a>
    </form>
</div>
@endsection
```

#### 5.1.3 Edit:
```blade
@extends('layouts.admin')

@section('content')
<div class="container mt-4">
    <h2>Edit Course</h2>
    <form action="{{ route('courses.update', $course) }}" method="POST" class="mt-3">
        @csrf
        @method('PUT')

        <div class="mb-3">
            <label class="form-label">Code</label>
            <input type="text" name="code" class="form-control" value="{{ $course->code }}" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Name</label>
            <input type="text" name="name" class="form-control" value="{{ $course->name }}" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Credits</label>
            <input type="number" name="credits" class="form-control" value="{{ $course->credits }}" required min="1">
        </div>

        <button type="submit" class="btn btn-success">Update</button>
        <a href="{{ route('courses.index') }}" class="btn btn-secondary">Back</a>
    </form>
</div>
@endsection
```

### 5.3. Vistas de Enrollments

#### 5.1.1 Index:
```blade
@extends('layouts.admin')

@section('content')
<div class="container mt-4">
    <div class="d-flex justify-content-between align-items-center mb-3">
        <h2>Enrollments</h2>
        <a href="{{ route('enrollments.create') }}" class="btn btn-primary">Add Enrollment</a>
    </div>

    <table class="table table-striped align-middle">
        <thead class="table-dark">
            <tr>
                <th>Student</th>
                <th>Course</th>
                <th>Year</th>
                <th>Semester</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @forelse ($enrollments as $enrollment)
                <tr>
                    <td>{{ $enrollment->student }}</td>
                    <td>{{ $enrollment->course }}</td>
                    <td>{{ $enrollment->year }}</td>
                    <td>{{ $enrollment->semester }}</td>
                    <td>
                        <form action="{{ route('enrollments.destroy', $enrollment->id) }}" method="POST" class="d-inline">
                            @csrf
                            @method('DELETE')
                            <button class="btn btn-danger btn-sm" onclick="return confirm('Delete this enrollment?')">Delete</button>
                        </form>
                    </td>
                </tr>
            @empty
                <tr><td colspan="5" class="text-center">No enrollments found.</td></tr>
            @endforelse
        </tbody>
    </table>
</div>
@endsection
```

#### 5.1.2 Create:
```blade
@extends('layouts.admin')

@section('content')
<div class="container mt-4">
    <h2>Add Enrollment</h2>
    <form action="{{ route('enrollments.store') }}" method="POST" class="mt-3">
        @csrf

        <div class="mb-3">
            <label class="form-label">Student</label>
            <select name="student_id" class="form-select" required>
                <option value="">Select student...</option>
                @foreach ($students as $student)
                    <option value="{{ $student->id }}">{{ $student->name }}</option>
                @endforeach
            </select>
        </div>

        <div class="mb-3">
            <label class="form-label">Course</label>
            <select name="course_id" class="form-select" required>
                <option value="">Select course...</option>
                @foreach ($courses as $course)
                    <option value="{{ $course->id }}">{{ $course->name }}</option>
                @endforeach
            </select>
        </div>

        <div class="mb-3">
            <label class="form-label">Year</label>
            <input type="number" name="year" class="form-control" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Semester</label>
            <select name="semester" class="form-select" required>
                <option value="">Select semester...</option>
                <option value="1">1</option>
                <option value="2">2</option>
            </select>
        </div>

        <button type="submit" class="btn btn-success">Save</button>
        <a href="{{ route('enrollments.index') }}" class="btn btn-secondary">Back</a>
    </form>
</div>
@endsection
```

## 6. Finalizar migraciones
Podemos ejecutar todas nuestras migraciones y semillas con:
```bash
php artisan migrate:fresh --seed
```

Luego correr nuestro proyecto con:
```bash
php artisan serve
```
