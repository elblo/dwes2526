<style>
    img { margin: 20px 0; border-radius: 8px; }

    .alert { color: #BD1550; }
    .warning { color: #E97F02; }
    .success { color: #8A9B0F; }

    .center { text-align: center; }
    .right { text-align: right; }

    .img-small { max-width: 200px; margin: auto; }
    .img-medium { max-width: 400px; margin: auto; }
    .img-large { max-width: 800px; margin: auto; }

    .leyenda {
        font-size: small;
        margin: 10px 0;
    }
</style>

# Gestión de datos en Laravel

> Duración estimada: 20 sesiones

## 8.1 Introducción

Laravel es un framework PHP moderno que simplifica el desarrollo de aplicaciones web, incluyendo la gestión de bases de datos. La integración con Eloquent, su ORM (Object Relational Mapping), permite trabajar con bases de datos de forma intuitiva y eficiente.

## 8.2 Configuración inicial

Laravel soporta varios motores de bases de datos como MySQL, PostgreSQL, SQLite y SQL Server. La configuración principal se realiza en el archivo `.env`. 

**Ejemplo de configuración en .env:**

```console
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nombre_base_datos
DB_USERNAME=usuario
DB_PASSWORD=password
```

Nota: El servidor MySQL debe estar funcionando con la base de datos <span class="alert">***ya creada***</span>.

**Probar conexión:**

Probamos la conexión ejecutando las migraciones:

```console
php artisan migrate
```

Si todo ha salido bien obtendremos el siguiente resultado donde podremos observar que todas las migraciones se han insertado correctamente en la base de datos.

<div class="center img-large">
    <img src="imagenes/07/migraciones.png">
</div>

Si nos vamos al cliente que utilicemos para manejar la base de datos (phpMyAdmin por ejemplo) veremos que en nuestra base de datos se han creado todas las tablas de la migración que hemos ejecutado y **además** una tabla que se llama <span class="success">***migrations***</span>.

La tabla `migrations` es simplemente un registro de todas las migraciones llevadas a cabo.

**Posibles problemas:**

- La extensión del driver de la base de datos (como *pdo_mysql* o *pdo_pgsql*) debe estar habilitada en el `php.ini`.
- Utilizar `php artisan config:clear` para borrar la caché de configuraciones si los cambios del `.env` no se reflejan.

## 8.3 Migraciones

### Introducción

Las migraciones son un sistema de control de versiones para bases de datos que permite trabajar de forma colaborativa, manteniendo un histórico de los cambios realizados en el esquema. Con las migraciones se puede: 

- Crear, modificar y borrar tablas. 
- Gestionar el esquema de forma programática utilizando **Artisan** y el **Schema Builder**. 
- Revertir cambios mediante **rollback** o volver a aplicar todos los cambios con **refresh**.

### Estructura de las migraciones

Las migraciones de un proyecto Laravel se guardan en el directorio *database/migrations* en archivos `.php` y siguen una estructura predefinida con dos métodos principales: 

- **up**: Define las operaciones que deben aplicarse en la base de datos (crear tablas, añadir columnas, etc.). 
- **down**: Define las operaciones inversas para revertir (rollback) los cambios aplicados por up.

Ejemplo:

```php
<?php
public function up()
{
    Schema::create('usuarios', function (Blueprint $tabla) {
        $tabla->id();
        $tabla->string('nombre');
        $tabla->string('email')->unique();
        $tabla->timestamps();
    });
}

public function down()
{
    Schema::dropIfExists('usuarios');
}
```

Por defecto, Laravel añade un campo autonumérico *id* y si se llama al método `timestamps()`, dos columnas *created_at* y *updated_at* que se actualizan automáticamente para saber cuándo se creó y actualizó un registro.

### Crear una migración

Mediante el comando `make:migration` de Artisan generamos una migración, un archivo con las instrucciones (Schema Builder) para construir o cambiar las tablas de la base de datos. En el nombre de dicho archivo se incluye un timestamp para asegurar el orden cronológico.

**Ejemplos:**

```bash
# Migración en blanco
php artisan make:migration nombre_migración 

# Migración para crear una tabla
php artisan make:migration create_table_usuarios --create=usuarios  

# Migración para modificar una tabla
php artisan make:migration add_fields_to_usuarios --table=usuarios 
```

Laravel puede inferir acciones del nombre de la migración gracias a la clase **TableGuesser**. Por ejemplo, si el nombre contiene *create* o *to*, Artisan deducirá si es para crear o modificar tablas.

### Ejecutar migraciones

- `php artisan migrate`: Ejecuta las migraciones pendientes.
- `php artisan migrate:status`: Muestra el estado de las migraciones.
- `php artisan migrate:fresh`: Borra todas las tablas de la BDD (sin ejecutar rollback) y ejecuta todas las migraciones.
- `php artisan migrate:refresh`: Hace un rollback de todas las migraciones y las vuelve a ejecutar. Para rellenar la BDD con datos de prueba, usar el flag --seed.
- `php artisan migrate:reset`: Hace un rollback de todas las migraciones.
- `php artisan migrate:rollback`: Revierte la la última migración.

### Schema Builder

La clase **Schema** es el kernel para definir y modificar el esquema de las bases de datos. Incluye constructores para crear, modificar y eliminar tablas y columnas. Y es lo que utilizaremos dentro de los archivos de migraciones.

#### Crear y eliminar tablas

```php
<?php
Schema::create('usuarios', function (Blueprint $table) {
    $table->id();
    $table->string('nombre', 32);
    $table->timestamps();
});

Schema::dropIfExists('usuarios');
```

#### Añadir y eliminar columnas

```php
<?php
Schema::table('usuarios', function (Blueprint $table) {
    $table->string('telefono')->after('nombre')->nullable();
});

Schema::table('usuarios', function (Blueprint $table) {
    $table->dropColumn('telefono');
});
```

#### Tipos de columnas

Laravel ofrece una amplia variedad de tipos de columnas que puedes consultar en la [documentación oficial](https://laravel.com/docs/11.x/migrations#available-column-types).

#### Índices

```php
<?php
$table->primary('id'); // Campo id como clave primaria
$table->primary(['nombre', 'apellidos']); // Clave primaria compuesta
$table->unique('email'); // Campo email único
$table->index('localidad'); // Campo localidad como índice
```

#### Claves foráneas

```php
<?php
Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');
    $table->foreign('user_id')->references('id')->on('usuarios');
});
```

Laravel proporciona mediante el método `foreignId` una forma más concisa de hacer lo anterior, creando automáticamente el *unsignedBigInteger* y determinando la tabla a la que hace referencia (user) por el nombre del campo.

```php
<?php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

Si la tabla a la que hace referencia no sigue las convenciones de nombres de Laravel, se puede indicar a mano la referencia.

```php
<?php
$table->foreignId('user_id')->constrained('usuarios', 'id_usuario');
```

Y también se puede especificar si queremos que los registros de la tabla actual se actualicen o borren en cascada según lo haga el registro de la tabla principal.

```php
<?php
$table->foreignId('user_id')
      ->constrained()
      ->onUpdate('cascade')
      ->onDelete('cascade');
```

## 8.4 Query Builder

El **Query Builder** de Laravel proporciona una interfaz fluida para construir y ejecutar consultas de bases de datos. Permite trabajar con varias bases de datos de forma sencilla sin escribir código SQL.

Es ideal para crear *consultas personalizadas* en las que el rendimiento es una prioridad y *consultas complejas* que no se pueden expresar fácilmente con Eloquent.

##### Ejemplos de consultas

```php
<?php
// Los siguientes ejemplos irían en la función correspondiente del MODELO
// Para pruebas, de momento también puedes ubicarlas en el CONTROLADOR
use Illuminate\Support\Facades\DB;

// Obtener todos los registros de users
$users = DB::table('users')->get(); 

// Filtrar registros
$users = DB::table('users')->where('type', 'customer')->get();

// Seleccionar columnas
$users = DB::table('users')->select('name', 'email')->get();

// Ordenar resultados
$users = DB::table('users')->orderBy('name', 'asc')->get();

// Contar registros
$count = DB::table('users')->count();

// Agregados
$maxSalary = DB::table('employees')->max('salary');

// Subconsultas
$users = DB::table('users')
    ->whereExists(function($query) {
        $query->select(DB::raw(1))
              ->from('orders')
              ->whereColumn('orders.user_id', 'users.id');
    })
    ->get();
```

##### Ejemplos de manipulación de datos

```php
<?php
use Illuminate\Support\Facades\DB;

// Insertar registro
DB::table('users')->insert([
    'name' => 'John Doe',
    'email' => 'john@example.com',
]);

// Actualizar registro
DB::table('users')
    ->where('id', 1)
    ->update(['name' => 'Updated Name']);

// Eliminar registro
DB::table('users')
    ->where('id', 1)
    ->delete();

// Borrar todos los registros
DB::table('users')->truncate();
```

## 8.5 Eloquent: Modelos

### Introducción

Un **ORM (Object-Relational Mapping)** es una técnica de programación que permite eliminar la disparidad entre el modelo de datos de una base de datos relacional y el modelo de objetos de una aplicación. Mientras que en una BDD pensamos en tablas y campos, en el mundo de desarrollo pensamos en objetos y propiedades.

Ventajas de un ORM:

- **Abstracción de la base de datos**: No es necesario escribir SQL, ya que el ORM se encarga de traducir las operaciones de la base de datos a objetos.
- **Nombres de campos y tablas**: No es necesario recordar los nombres de las tablas y campos, ya que el ORM se encarga de ello. Si cambiamos el nombre de un campo, solo tenemos que cambiarlo en un lugar, en el modelo.
- **Relaciones**: Las relaciones entre tablas se pueden definir en los modelos, y el ORM se encarga de gestionarlas. Atravesar relaciones es tan sencillo como acceder a una propiedad de un objeto.

**Eloquent** es el ORM de Laravel, y nos permite interactuar con la base de datos de una forma sencilla y elegante. Eloquent es una capa de abstracción de la base de datos, que nos permite interactuar con ella utilizando objetos. 

Cada tabla de la base de datos tiene un modelo asociado, que es una clase que representa a la tabla. Por tanto:

- las tablas son modelos
- los registros de la tabla son instancias de ese modelo
- los campos de una tabla son propiedades del modelo

<figure style="align: center;">
    <img src="imagenes/08/eloquent-orm.png" width="700">
    <figcaption>Relación entre tabla y modelo</figcaption>
</figure>

### Modelos

Los modelos son uno de los componentes más importantes de Laravel, son los responsables de interactuar con nuestra base de datos de una manera orientada a objetos. Representan las tablas de la base de datos como clases en la aplicación, permiten realizar operaciones para seleccionar, crear, actualizar y eliminar datos de una manera más sencilla y estructurada.

#### Crear un modelo

Los modelos se definen dentro de la carpeta *app/Models* y se pueden crear mediante Artisan:

```console
php artisan make:model Nota -m
```

!!! danger "Nombrar correctamente"
    El nombre del modelo empieza por Mayúscula y siempre se escribe en **SINGULAR**. 
    
    Si le pasamos el parámetro **-m** además creará la migración con el código para crear la tabla correspondiente en la BDD, cuyo nombre irá en minúscula y plural.

??? info "Opciones al crear el modelo"
    Podemos añadir las siguientes opciones al comando para crear otros elementos relacionados con el modelo:

    - **-c**, --controller: Crea un controlador.
    - **-m**, --migration: Crea una migración.
    - **-r**, --resource: Crea un controlador de recursos.
    - **-f**, --factory: Crea un factory.
    - **-s**, --seed: Crea un seeder.
    - **-a**, --all: Crea todo: un controlador de recursos, una migración, factoría...
  
    Por ejemplo, si queremos crear un modelo, una migración y un controlador, ejecutamos:

    ```console
    php artisan make:model Nota -cm 
    ```

Si todo ha salido bien, veremos en nuestro directorio de migraciones `database/migrations` un nuevo archivo con un nombre similar a `2025_01_21_111237_create_notas_table.php` en el que se encuentra la tabla relacionada y que podemos abrir para seguir añadiendo campos mediante el **Schema Builder** como se ha visto anteriormente. Por ejemplo:

```php
<?php

Schema::create('notas', function (Blueprint $table) {
  $table->id();
  $table->timestamps();
  // Campos añadidos
  $table->string('titulo'); 
  $table->text('descripcion');
  $table->integer('prioridad');
});
```

Una vez tengamos listo nuestro esquema debemos lanzar `php artisan migrate` para que ejecute las migraciones pendientes introduciendo la nueva información en la base de datos.

#### Uso básico de un modelo

##### Recuperar datos

```php
<?php
// Todos los registros
$notas = Nota::all();

// Registros filtrados
$notas = Nota::where('prioridad', '>', 5)->get();

// Registro único
$nota = Nota::find($id); // devuelve el objeto o null
$nota = Nota::findOrFail($id); // devuelve el objeto o una excepción, que por ejemplo redirige a una página 404 no encontrado en el caso de acceder a alguno de sus métodos como 'delete'
```

##### Insertar datos
```php
<?php
// Método 1
$nota = new Nota();
$nota->titulo = "Proyecto Laravel";
$nota->descripcion = "Programar la parte de los modelos de la práctica de Fútbol Femenino.";
$nota->prioridad = 10;
$nota->save();

// IMPORTANTE, para los siguientes métodos hay que definir en el modelo 'Nota':
// protected $fillable = ['titulo', 'descripcion', 'prioridad'];

// Método 2
$nota = new Nota(['titulo' => $titulo, 'descripcion' => $descripcion, 'prioridad' => $prioridad]);
$nota->save();

// Método 3: Guardado automático en BDD
Nota::create(['titulo' => $titulo, 'descripcion' => $descripcion, 'prioridad' => $prioridad]);
```

##### Actualizar datos
```php
<?php
// Método 1
$nota = Nota::find($id);
$nota->titulo = "Nuevo título";
$nota->save();

// Método 2: Guardado automático en BDD
Nota::find($id)->update(['titulo' => 'Nuevo título']);
```

##### Eliminar datos
```php
<?php
// Método 1: 
$nota = Nota::find($id);
$nota->delete(); // Devuelve true/false

// Método 2: Devuelve el número de registros eliminados
Nota::destroy($id); // admite un array de ids a eliminar: Nota::destroy([1, 2, 3]);
```

??? info "Otros métodos: updateOrCreate, firstOrCreate, firstOrNew..."
    Anímate a buscar en la documentación oficial, hay muchos más métodos para trabajar con los modelos de Eloquent, como:

    - `updateOrCreate()`: Busca un registro, si lo encuentra lo actualiza, si no lo crea.
    - `firstOrCreate()`: Busca un registro, si lo encuentra lo devuelve, si no lo crea.
    - `firstOrNew()`: Busca un registro, si lo encuentra lo devuelve, si no devuelve una instancia nueva sin guardar en la BD.

    Por ejemplo: Si ya existe un usuario con ese email, actualizará name y password. Si no, creará un nuevo usuario con esos datos.

    ```
    <?php
    User::updateOrCreate(
        ['email' => 'ejemplo@email.com'], // Buscar usuario con este email
        ['name' => 'Nuevo Nombre', 'password' => bcrypt('secreto')]
    );
    ```

#### Propiedades comunes de los modelos Eloquent

En los modelos podemos definir varias propiedades para configurar el comportamiento de la interacción con la base de datos. A continuación se detallan las más importantes:

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Example extends Model
{
    // Especifica el nombre de la tabla si no sigue la convención de nombres de Laravel
    protected $table = 'custom_table_name';

    // Define la clave primaria de la tabla
    protected $primaryKey = 'custom_id';
    // Indica si la clave primaria es autoincremental
    public $incrementing = false;
    // Especifica el tipo de la clave primaria (si no es integer)
    protected $keyType = 'string';

    // Define qué atributos pueden ser asignados masivamente (a través de métodos como create o update)
    protected $fillable = ['name', 'email', 'password'];
    // Contrario a $fillable. Define qué atributos no pueden ser asignados masivamente
    protected $guarded = ['is_admin'];

    // Define los atributos a ocultar al serializar el modelo (a JSON o array)
    protected $hidden = ['password', 'remember_token'];
    // Contrario a $hidden, define los atributos que serán visibles al serializar
    protected $visible = ['name', 'email'];

    // Transformación automática de los atributos a un tipo específico
    protected $casts = [
        'is_admin' => 'boolean',
        'settings' => 'array',
    ];

    // Indica si la tabla tiene los campos `created_at` y `updated_at`
    public $timestamps = true;

    // Define la conexión a la BDD
    protected $connection = 'mysql';
}
```

### Ejemplo recuperar datos

Ya tenemos nuestra base de datos creada con las tablas migradas, ahora sólo falta rellenarlas con algunos datos de prueba a través del cliente de MySQL que más nos guste:

- PHPMyAdmin
- [MySQL Workbench](https://www.mysql.com/products/workbench/)
- [HeidiSQL](https://www.heidisql.com/download.php)

Vamos a ver cómo recuperar esos datos desde una vista y lo primero que vamos a necesitar va a ser un controlador para gestionar esas notas:

```bash
php artisan make:controller NotaController
```

Es recomendable seguir la convención de Laravel de nombres de rutas y funciones de controladores que vimos [aquí](https://elblo.github.io/dwes2425/07frameworks.html#controlador-de-recursos).

##### 1. Rutas 

Creamos la ruta en `web.php` que redirige a la función correspondiente del controlador para mostrar todas las notas o una en particular.

```php
<?php
// estamos en ▓▓▓ web.php 
Route::get('notas', [ PagesController::class, 'index' ])->name('notas.index');
Route::get('notas/{id?}', [ PagesController::class, 'show' ])->name('notas.show');
```

##### 2. Controlador

Desde la función del controlador se llama al modelo para recuperar las notas y devuelve la vista pasándoselas.

```php
<?php
// estamos en ▓▓▓ NotaController.php 

// Muestra listado de notas
public function index() {
  $notas = Nota::all();
  return view('notas.index', compact('notas'));
}

// Muestra una nota en específico
public function show($id) {
  $nota = Nota::findOrFail($id);
  return view('notas.show', compact('nota'));
}
```

##### 3. Vistas

`notas/index.blade.php`: Vista con la tabla que pinta los datos mediante las notas pasadadas como parámetro.

```html
<h1>Notas desde base de datos</h1>

@if(session('mensaje'))
    <div>{{ session('mensaje') }}</div>
@endif

<table border="1">
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Descripción</th>
            <th>Prioridad</th>
            <th>Editar</th>
            <th>Eliminar</th>
        </tr>
    </thead>
    
    @foreach ($notas as $nota)
        <tr>
            <td>{{$nota->titulo}}</td>
            <td>{{$nota->descripcion}}</td>
            <td>{{$nota->prioridad}}</td>
            <td>📝</td>
            <td>❌</td>
        </tr>
    @endforeach
</table>
<p><a href="">Nueva nota</a></p>

```

`notas/show.blade.php`: Vista con el detalle de una nota en particular. Hace uso de la plantilla que teníamos y está dentro de la subcarpeta *notas*.

```php
<?php
// estamos en ▓▓▓ notas/show.blade.php
@extends('plantilla')

@section('apartado')
  <h1>Detalle de la nota</h1>

  <p>ID: {{ $nota->id }}</p>
  <p>Nombre: {{ $nota->titulo }}</p>
  <p>Descripción: {{ $nota->descripcion }}</p>    
  <p>Prioridad: {{ $nota->prioridad }}</p>     
@endsection
```

!!! info "Ojo con los nombres"
    Hay que fijarse bien en los nombres de las columnas que tienen nuestras tablas, porque serán los nombres de los atributos de los objetos del modelo que utilizaremos.

### Modificar tablas sin perder datos

A veces cometemos errores de diseño y queremos introducir una nueva columna dentro de nuestra tabla o modificar una de esas columnas <span class="alert">***SIN PERDER LOS DATOS DE LA BASE DE DATOS***</span>.

Imaginemos que en nuestra tabla `notas` queremos agregar una columna con el nombre `autor`.

Lo primero de todo es crear una nueva migración para realizar este cambio mediante *Artisan* con el nombre `add_fields_to_` seguido del nombre de la tabla a modificar.

```console
php artisan migrate add_fields_to_nota
```

Seguidamente, abrimos el archivo de la migración que acabamos de crear y en la función `up()` ponemos el cambio que queremos realizar y en `down()` lo eliminamos para que en caso de hacer una migración rollback, se vuelva a quedar todo como estaba.

```php
<?php

public function up()
{
  Schema::table('notas', function (Blueprint $table) {
      $table->string('autor');
  });
}

public function down()
{
  Schema::table('notas', function (Blueprint $table) {
      $table->dropColumn('autor');
  });
}
```

## 8.6 Formularios

Ya sabemos cómo recuperar datos de una base de datos. Ahora toca ver cómo insertarlos, actualizaros y eliminarlos con Laravel y sin escribir ni una sola línea de SQL.

Es recomendable seguir la convención de Laravel de nombres de rutas y funciones de controladores que viemos [aquí](https://elblo.github.io/dwes2425/07frameworks.html#controlador-de-recursos).

### Insertar datos

Para insertar datos vamos a necesitar 2 rutas, 2 funciones en el controlador y 1 vista con el formulario:

- Con la primera ruta `notas/create` llamaremos a la función *create* del controlador que abrirá el formulario para crear una nueva nota.
- El formulario enviará los datos a la segunda ruta `notas` mediante **POST**, la cual llamará a la función *store* del controlador para crear la nota mediante el método *save()*.

##### 1. Rutas

Creamos las rutas GET y POST con sus alias correspondientes en nuestro archivo de rutas `web.php`. Situar la ruta `notas/create` antes de la ruta `notas/{id?}` porque si no entraría siempre en esta última.

```php
<?php
// estamos en ▓▓▓ web.php
Route::get('notas/create', [ NotaController::class, 'create' ])->name('notas.create');
Route::post('notas', [ NotaController::class, 'store' ])->name('notas.store');
```

##### 2. Controlador

En el controlador creamos los 2 métodos:

- `create` para abrir el formulario.
- `store` para crear la nueva nota con los datos que le llegan del formulario mediante *Request* y almacenarla medidante *save()* y volvemos a la página del formulario con el método *back()* añadiendo un mensaje con *with()*.

```php
<?php
// estamos en ▓▓▓ NotaController.php

// Muestra el formulario para crear una nueva nota
public function create() {
    return view('notas.create');
}

// Crea una nota con la info del formulario
public function store(Request $request) {
    $notaNueva = new Nota();
    $notaNueva->titulo = $request->titulo;
    $notaNueva->descripcion = $request->descripcion;
    $notaNueva->prioridad = $request->prioridad;
    $notaNueva -> save();

    // Volver al formulario para seguir insertando
    return back()->with('mensaje', 'Nota insertada');
}
```

##### 3. Vista

`notas/create.blade.php`: Vista con el formulario para crear una nueva nota. En el *action* se indica la ruta a la que enviar los datos por POST.

- En el *action* se indica la ruta a la que enviar los datos por POST.
- El atributo *name* de los inputs tiene que ser igual al del campo correspondiente de la tabla.
- Se usa la cláusula de seguridad `@csrf` para evitar ataques desde otros sitios. [Más info](https://www.ionos.es/digitalguide/servidores/seguridad/cross-site-request-forgery/) sobre este ataque.
- Con *session('mensaje')* mostramos el mensaje que viene del controlador.

```html
<h2>Crear nueva nota</h2>
@if (session('mensaje'))
    <div class="mensaje-nota-creada">{{ session('mensaje') }}</div>
@endif

<form action="{{ route('notas.store') }}" method="POST">
    @csrf {{-- Cláusula para obtener un token de formulario al enviarlo --}}
    <div>
        <input type="text" name="titulo" placeholder="Título de la nota" autofocus />
        <input type="text" name="descripcion" placeholder="Descripción de la nota" />
        <input type="number" name="prioridad" placeholder="5" />

        <button type="submit">Crear nueva nota</button>
    </div>
</form>

<div><a href="{{ route('notas.index') }}">Volver</a></div>
```

##### 4. Incluir enlace para insertar

En la vista `notas/index.blade.php` añadimos un enlace o botón que abra el formulario para crear una nueva nota.

```html
<p><a href="{{ route('notas.create') }}">Nueva nota</a></p>
```

### Actualizar datos

Para actualizar, al igual que para insertar datos, vamos a necesitar 2 rutas, 2 funciones en el controlador y 1 vista con el formulario:

- Con la primera ruta `notas/{id}/edit` llamaremos a la función *edit* del controlador que abrirá el formulario para modificar la nota.
- El formulario enviará los datos a la segunda ruta `notas/{id}` mediante **PUT**, la cual llamará a la función *update* del controlador para actualizar la nota mediante el método *save()*.

##### 1. Rutas

Creamos las rutas GET y PUT con sus alias correspondientes en nuestro archivo de rutas `web.php`.

```php
<?php
// estamos en ▓▓▓ web.php
Route::get('notas/{id}/edit', [ NotaController::class, 'edit' ])->name('notas.edit');
Route::put('notas/{id}', [ NotaController::class, 'update' ])->name('notas.update');
```

##### 2. Controlador

En el controlador creamos los 2 métodos:
- `edit` para abrir el formulario.
- `update` para actualizar la nota con los datos que le llegan del formulario mediante *Request* y almacenarla medidante *save()* y volvemos a la página del formulario con el método *back()* añadiendo un mensaje con *with()*.

```php
<?php
// estamos en ▓▓▓ NotaController.php

// Muestra el formulario para editar una nota
public function edit($id) {
    $nota = Nota::findOrFail($id);
    return view('notas.edit', compact('nota'));
}

// Almacena la info recibida del formulario de edición
public function update(Request $request, $id) {
    $notaUpdate = Nota::findOrFail($id);
    $notaUpdate->titulo = $request->titulo;
    $notaUpdate->descripcion = $request->descripcion;
    $notaUpdate->prioridad = $request->prioridad;
    $notaUpdate->save();

    // Volver al listado de notas
    //return redirect('/notas')->with('mensaje','Nota actualizada');
    return redirect()->route('notas.index')->with('mensaje','Nota actualizada');
}
```

##### 3. Vista

`notas/edit.blade.php`: Vista con el formulario para actualizar la nota. En el *action* se indica la ruta a la que enviar los datos por POST.

- En el *action* se indica la ruta a la que enviar los datos junto al *id* de la nota.
- Mediante `@method('PUT')` indicamos que se haga la petición a la url del formulario mediante el método **PUT**, que es como la recogemos en las rutas.
- Se usa la cláusula de seguridad `@csrf` para evitar ataques desde otros sitios. [Más info](https://www.ionos.es/digitalguide/servidores/seguridad/cross-site-request-forgery/) sobre este ataque.
- El atributo *name* de los inputs tiene que ser igual al del campo correspondiente de la tabla.
- Con *session('mensaje')* mostramos el mensaje que viene del controlador.

```html
<h2>Editando la nota {{ $nota -> id }}</h2>

<form action="{{ route('notas.update', $nota->id) }}" method="POST">
    @method('PUT') {{-- Necesitamos cambiar al método PUT para editar --}}
    @csrf {{-- Cláusula para obtener un token de formulario al enviarlo --}}

    @error('nombre')
        <div class="text-red-500 text-lg mt-2">El nombre es obligatorio</div>
    @enderror

    @error('descripcion')
        <div class="text-red-500 text-lg mt-2">La descripción es obligatoria</div>
    @enderror

    @error('prioridad')
        <div class="text-red-500 text-lg mt-2">La descripción es obligatoria</div>
    @enderror

  <input
      type="text"
      name="titulo"
      value="{{ $nota->titulo }}"
      placeholder="Título de la nota"
      autofocus
  />
  <input
      type="text"
      name="descripcion"
      placeholder="Descripción de la nota"
      value="{{ $nota->descripcion }}"
  />
  <input
      type="number"
      name="prioridad"
      placeholder="5"
      value="{{ $nota->prioridad }}"
  />

  <button type="submit">Guardar cambios</button>
</form>

<div><a href="{{ route('notas.index') }}">Volver</a></div>
```

##### 4. Incluir enlace para editar

En la vista `notas/index.blade.php` añadimos a cada nota que se muestra en la tabla un enlace para poder editarla.

```html
<td><a href="{{ route('notas.edit', $nota->id) }}">📝</a></td>
```

### Eliminar datos

Para insertar datos vamos a necesitar 1 ruta y 1 función en el controlador:

- Con la ruta `notas/{id}` que llega mediante **DELETE** llamaremos a la función *destroy* del controlador.

##### 1. Ruta

Creamos las rutas DELETE con su alia correspondiente en nuestro archivo de rutas `web.php`.

```php
<?php
// estamos en ▓▓▓ web.php
Route::delete('notas/{id}', [ NotaController::class, 'destroy' ])->name('notas.destroy');
```

##### 2. Controlador

En el controlador creamos el método:
- `destroy` que mediante el método *delete()* elimina la nota y redirige al listado de notas añadiendo un mensaje con *with()*.

```php
<?php
// estamos en ▓▓▓ NotaController.php

public function destroy($id) {
    $notaEliminar = Nota::findOrFail($id);
    $notaEliminar->delete();
  
    // Volver al listado de notas
    return back()->with('mensaje','Nota eliminada');
}
```

##### 3. Vista

No es necesaria ninguna vista específica 🥳

##### 4. Incluir enlace para eliminar

En la vista `notas/index.blade.php` añadimos un botón para cada nota que lance la ruta con el id de la nota a eliminar.

```html
<td>
    <form action="{{ route('notas.destroy', $nota->id) }}" method="POST">
        @method('DELETE')
        @csrf
        <button type="submit">❌</button>
    </form>
</td>
```

!!! info "Enhorabuena!"
    Si todo ha salido bien, habrás creado un sitio en Laravel y Eloquent capaz de hacer un ***CRUD*** con datos reales en una base de datos.

## 8.7 Validación

Laravel nos proporciona herramientas para poder validar en el lado del servidor los datos que el usuario introduce en los campos del formulario.

### Validación básica

La validación se hace de los campos recibidos del formulario, por lo que tenemos que utilizar los nombres (atributos *name*) que utilizamos en los campos del formulario.

```php
<?php
// estamos en ▓▓▓ NotaController.php

// Crea una nota con la info del formuluario
public function store(Request $request) {
    // Validar datos recibidos del formulario mediante Request
    $request -> validate([
      'titulo' => 'required|string|min:5',
      'descripcion' => 'required|max:255',
    ]);

    // ... creación de la nota

    // Volver al formulario para seguir insertando
    return back()->with('mensaje', 'Nota insertada');
}
```

### Reglas comunes

El listado completo de reglas se puede consultar en la [documentación oficial](https://laravel.com/docs/11.x/validation#available-validation-rules), aunque aquí se muestran las más comunes: 

- **required**: Campo obligatorio.
- **email**: Validación de un correo electrónico.
- **min:value**: Mínimo de caracteres o valor numérico.
- **max:value**: Máximo de caracteres o valor numérico.
- **unique:table,column**: Debe ser único en una tabla/columna. 

### Mensajes personalizados

La función *validate()* admite 2 arrays:

1. Listado de elementos con sus reglas de validación.
2. Listado de mensaje personalizados según el elemento y regla.

```php
<?php
$request -> validate([
    'titulo' => 'required|string|min:5',
    'descripcion' => 'required|max:255',
  ],[
    'titulo.required' => 'El campo título es requerido',
    'titulo.min' => 'El campo título debe tener al menos 5 caracteres',
    'titulo.string' => 'El campo título debe ser de tipo texto',
  ]);
          
```

### Mostrar mensajes error

Mediante la directiva *@error* en la vista del formulario, mostramos los mensajes de error (por defecto o personalizados) asociados a cada campo. Se le pasa el nombre del campo que queremos validar, y si hay un error, se mostrará el mensaje de error utilizando la variable *$message*. Útil para mostrar el mensaje de error junto a su campo correspondiente.

```html
<!-- estamos en ▓▓▓ notas/create.blade.php (donde está el formulario) -->
@error('titulo')
    <p class="text-red-500 text-xs mt-2"> {{ $message }}</p>
@enderror

<!-- También es posible escribir directamente el mensaje -->
@error('descripcion')
    <p class="text-red-500 text-xs mt-2">No olvides rellenar la descripción.</p>
@enderror
```

### Mantener valor

En caso de que haya un error podemos mantener el valor que tuviera el campo del formulario mediante el método *old()* de Laravel. Recibe el nombre del campo, y si hay un error, mostrará el valor que había introducido el usuario. Admite como segundo parámetro, el valor inicial del campo.

```html
<!-- estamos en ▓▓▓ notas.blade.php -->
<input
  type="text"
  name="titulo"
  value="{{ old('titulo') }}"
  placeholder="Título de la nota"
  autofocus
/>

<!-- En formularios de actualización es interesante indicar el valor inicial del campo -->
<input
  type="text"
  name="titulo"
  value="{{ old('titulo', $nota->titulo) }}"
  placeholder="Título de la nota"
  autofocus
/>
```

!!! info "Validación en front y back"
    Debemos siempre **validar los datos tanto en el backend** (como acabamos de ver) **como en el frontend** con las opciones que nos da html5 mediante *required*, *pattern* o estableciedno el tipo de input apropiado: number, email, date...

## 8.8 Paginación

Para añadir paginación a nuestros resultados, Eloquent tiene el método `paginate()` donde le pasamos un número entero como parámetro para indicarle el número de resultados que queremos por página.

```php
<?php
// estamos en ▓▓▓ NotaController.php

// Muestra listado de notas
public function index() {
    $notas = Nota::paginate(3); // 3 notas por página
    return view('notas.index', compact('notas'));
}
```

Y en la vista `notas/index.blade.php`, para que muestre los enlaces para pasar de página incluimos:

```html
<p>{{ $notas->links() }}</p>
```

La librería de paginación que utiliza Laravel está situada en la carpeta `vendor/laravel/framework/src/illuminate/Pagination`. Abriendo su archivo `resources/views/tailwind.blade.php` se puede ver la estructura HTML del sistema de paginación. 

Aunqeu no es recomendable tocar la carpeta *vendor*, por practicar podríais modificar ese archivo (guardando antes una copia del mismo).

---

## 8.9 Actividades

A continuación, vas a realizar una serie de ejercicios sencillos sobre cada uno de los apartados vistos en el tema. Puedes crear un proyecto nuevo o reutilizar uno existente.

### Migraciones

En este apartado vas a trabajar creando migraciones. Es importante, que aparte del código en sí, apuntes los comandos que utilizas para crearlas, eliminarlas, ejecutarlas...

801. **Crear de una tabla básica**: Crea una tabla llamada productos con las siguientes columnas:

- *id* (entero, clave primaria, auto-incremental)
- *nombre* (string, longitud máxima de 255)
- *precio* (decimal, 8 dígitos en total, 2 decimales)

802. **Añadir columnas a una tabla existente**: Añade una columna *descripcion* (tipo texto) a la tabla productos.
803. **Crear una tabla con claves foráneas**: Crea una tabla *categorias* y una tabla *productos* donde cada producto pertenece a una categoría.
804. **Modificar una tabla para añadir índices**: Añade un índice único a la columna *nombre* de la tabla *categorias*.
805.  **Eliminar una columna de una tabla**: Elimina la columna *descripcion* de la tabla *productos*.
806. **Renombrar una tabla**: Cambia el nombre de la tabla *productos* a *articulos*.
807. **Usar valores predeterminados en una columna**: Añade una columna *stock* con un valor por defecto de 0 a la tabla *productos*.
808. **Crear tabla con datos iniciales**: Crear tabla *usuarios* con los siguientes campos:

- *id*
- *nombre* (string)
- *email* (string, único)
- *password* (string)
- *created_at* y *updated_at*

Además, rellénala con datos iniciales mediante el seeder DatabaseSeeder (opcional, se ve en el tema siguiente).

809. **Borrar y recrear la BDD**: Utiliza los comandos de Artisan necesarios para eliminar y volver a crear todas las tablas de la BDD.

810. **Ejercicio completo: Crear un sistema de reservas**: Crea las siguientes tablas para un sistema de reservas:

- *usuarios* (id, nombre, email, password)
- *habitaciones* (id, nombre, capacidad)
- *reservas* (id, usuario_id, habitacion_id, fecha_reserva)

Incluye claves foráneas, valores predeterminados y relación de "cascade delete".

### Query Builder

En este apartado vas a trabajar haciendo consultas directamente sobre la BDD mediante *Query Builder*. 

Para probar que funciona, se recomienda meter el código de cada ejercicio en una función independiente del controlador que se llamará con una ruta que te inventes (por ejemplo: `localhost/ejercicio820`).

Para todos los ejercicios se va a utilizar la tabla *productos*. Si no la tienes, créala mediante una migración con los campos *id*, *nombre*, *precio*, *descripcion*.

820. **Insertar registros**: Inserta un nuevo producto en la tabla. Puedes crear una ruta a la que se le pasen los parámetros *nombre*, *precio* y *descripcion*.
821. **Actualizar registros**: Actualiza el *nombre* de un producto por su *id* (ambos de pasan por parámetro a la ruta).
822. **Actualizar registros**: Actualiza el *precio* de un producto por su *id* (ambos de pasan por parámetro a la ruta).
823. **Actualizar registros**: Actualiza la *descripcion* de un producto por su *id* (ambos de pasan por parámetro a la ruta).
824. **Eliminar registros**: Elimina un producto según el *id* pasado por la ruta.
825. **Eliminar registros**: Elimina los productos cuyo *precio* sea inferior a 20.
826. **Obtener todos los registros**: Obtén todos los registros de la tabla *productos*. 
827. **Obtener registro por id**: Obtén un registro por su *id* pasado por la ruta.
828. **Seleccionar columnas específicas**: Obtén solo las columnas *nombre* y *precio* de todos los registros de *productos*.
829. **Filtrar registros con where**: Obtén los productos cuyo precio sea mayor a 50.
830. **Ordenar resultados**: Ordena los productos por precio de forma descendente.
831. **Paginar resultados**: Pagina los productos mostrando 5 por página.
832. **Consultas con varios where**: Obtén productos cuyo *precio* esté entre 50 y 100, y cuya *descripción* no sea nula.
833. **Contar registros**: Cuenta cuántos productos tienen un **precio** mayor a 100.
834. **Obtener el registro más caro**: Obtén el producto con el *precio* más alto.
835. **Ejecutar consultas crudas**: Usa una consulta SQL cruda para obtener productos cuyo *nombre* contenga la palabra "Premium".
836. **Consulta con uniones (join)**: Crea la migración corresponediente para crear la tabla *categorias* (*id*, *nombre*) y hacer que cada producto pertenezca a una categoría. Una vez hecho, mediante Query Builder obtén el *nombre del producto* junto al *nombre de su categoría*.
837. **Agrupar resultados con groupBy y having**: Agrupa los productos por *categoría* y calcula el *precio promedio* por categoría, mostrando solo las categorías con un promedio mayor a 50.
838. **Consultas anidadas**: Encuentra el producto más caro dentro de cada categoría.
839. **Ejercicio completo: CRUD con Query Builder** Implementa un CRUD completo para la tabla *clientes* utilizando Query Builder:

- *C*: Inserta nuevos clientes.
- *R*: Obtén todos los clientes y filtra por email.
- *U*: Actualiza el nombre de un cliente específico.
- *D*: Elimina clientes con un email específico.

### Eloquent: Modelos

Antes has trabajado lanzando consultas mediante *Query Builder* directamente sobre la tabla *productos*. Ahora harás consultas parecidas, pero SIEMPRE desde el modelo mediante *Eloquent*.

Para probar que funciona, se recomienda meter el código de cada ejercicio en una función independiente del controlador que se llamará con una ruta que te inventes (por ejemplo: `localhost/ejercicio841`).

840. **Crear modelo y tabla asociada**: Crea el modelo *Producto* con su tabla asociada *productos* (ya la tienes del apartado anterior).
841. **Inserta productos**: Inserta un producto con los campos *nombre*, *precio* y *descripcion* pasados mediante parámetro por la ruta.
842. **Inserta productos**: Inserta un producto con los campos *nombre*, *precio* y *descripcion* pasados mediante parámetro por la ruta, pero validando previamente que su *precio* sea mayor que 50 para insertarlo realmente.
843. **Actualizar productos**: Actualiza el *nombre* de un producto por su *id* (ambos de pasan por parámetro a la ruta).
844.  **Actualizar productos**: Actualiza el *precio* de un producto por su *id* (ambos de pasan por parámetro a la ruta).
845. **Actualizar productos**: Actualiza la *descripcion* de un producto por su *id* (ambos de pasan por parámetro a la ruta).
846. **Actualizar múltiples productos**: Actualiza todos los productos cuyo precio sea menor que 50, cambiando su descripcion a 'producto económico'.
847. **Eliminar productos**: Elimina un producto según el *id* pasado por la ruta.
848. **Eliminar productos**: Elimina los productos cuyo *precio* sea inferior a 20.
849. **Obtener todos los productos**: Obtén todos los productos de la tabla *productos*.
850. **Obtener producto por id**: Obtén un producto por su *id* pasado por la ruta.
851. **Filtrar con where**: Obtén los productos cuyo *precio* sea mayor a 50.
852. **Contar el número de productos**: Cuenta cuántos productos tienen un *precio* mayor a 50.
853. **Ordenar resultados**: Ordena los productos por *precio* de manera descendente.
854. **Usar el método pluck** para obtener solo los nombres de los productos.
855. **Usar el método firstOrCreate** para buscar un producto por su *nombre*, y si no existe, crea un nuevo producto
856. **Usar el método updateOrCreate** para actualizar un producto existente por su *nombre*, o crear uno nuevo si no existe.
857. **Limitar resultados**: Usa el método *take* para obtener solo los 5 primeros productos.
858. **Paginación de resultados**: Página los productos mostrando 5 por página.
859. **Consulta con where y orWhere**: Recupera todos los productos cuyo precio sea mayor que 100 o cuyo *nombre* contenga la palabra "Premium".
860. **Ejercicio completo: CRUD con Eloquent** Implementa un CRUD completo mediante Eloquent para el modelo *Cliente* que crees asociado a la tabla *clientes*:

- *C*: Crea nuevos clientes.
- *R*: Obtén todos los clientes y filtra por email.
- *U*: Actualiza el nombre de un cliente específico.
- *D*: Elimina clientes con un email específico.

### Formularios

En los siguientes ejercicios de formularios vas a realizar un CRUD de productos continuando lo que hiciste en el apartado anterior. Si tienes las clases de Productos (Modelo y Controlador) muy extensas y prefieres empezar de 0, puedes hacer los ejercicios siguientes para gestionar *usuarios en vez de productos*. Tendrías que crear previamente el modelo *Usuario* y tabla asociada *usuarios* con los campos típicos: *nombre*, *email*, *password*.

En cualquier caso, recuerda nombrar correctamente las rutas, funciones en controladores y vistas siguiendo las recomendaciones de Laravel.

870. **Formulario para crear productos**: Crea un formulario en el que se pidan los campos necesarios para crear un producto. Se accederá mediante `GET /productos/create` y su vista estará en `productos/create.blade.php`. El formulario se procesará mediante `POST /productos/store` redirigiendo finalmente a `GET /productos` donde se muestra el listado de productos.
871. **Formulario para editar productos**: Crea el formulario de edición de un producto. Se accederá mediante `GET /productos/{id}/edit` y su vista estará en `productos/edit.blade.php`. El formulario se procesará mediante `PUT /productos/update` redirigiendo finalmente a `GET /productos` donde se muestra el listado de productos.
872. **Validación de datos**: En las funciones correspondientes del controlador donde se reciben los datos de los formularios anteriores, añade validación a cada uno de los campos:

- *nombre*: Requerido, tipo cadena y valor máximo 255.
- *precio*: Requerido, tipo numérico y valor mínimo 0.
- *descripcion*: Tipo cadena y valor máximo 1000.

En las vistas de los 2 formularios añade mensajes de error en el caso de que los campos no pasen la validación y asigna mediante *old* el valor antiguo del campo para que el usuario no tenga que volver a escribirlo.

873. **Formulario de confirmación para eliminar productos**: Crear un formulario de confirmación para eliminar un recurso. Sólo contendrá un mensaje de "¿Estás seguro que quieres eliminar el producto ID?" y un botón para proceder a eliminarlo. Se accederá mediante `GET /productos/{id}/destroy` y su vista estará en `productos/destroy.blade.php`. El formulario se procesará mediante `DELETE /productos/{id}` redirigiendo finalmente a `GET /productos` donde se muestra el listado de productos.
874. **Ejercicio completo: CRUD con formularios**: Continúa el CRUD del apartado anterior para añadir funciones a la gestión de *clientes*:

- *C*: Crea nuevos clientes.
- *R*: Obtén todos los clientes y filtra por diferentes cmapos.
- *U*: Actualiza los campos de un cliente específico.
- *D*: Elimina clientes.

### Práctica: Gestión libros

Desarrolla una app para gestionar la biblioteca personal de libros del usuario. La aplicación permitirá registrar los libros que ha leído o tiene pendientes, junto con información relevante como su opinión, el formato en el que lo posee, si lo han prestado a alguien...

A continuación se detallan los requisitos. Deberás hacer las migraciones correspondientes, rutas, controlador, vistas... que necesites para su implementación.

#### Requisitos

1.	Modelo Libro con los siguientes campos:

- *titulo* (string): Nombre del libro.
- *autor* (string): Nombre del autor.
- *portada* (string): URL con la imagen del libro.
- *genero* (string): Género: novela, ciencia ficción, ensayo...
- *año_publicacion* (integer): Año en que se publicó el libro.
- *formato* (string): Formato del libro: físico, ebook, pdf...
- *estado_lectura* (string): Estado actual del libro: pendiente, leyendo, leído, abandonado...
- *puntuacion* (integer, 1-10): Puntuación personal sobre el libro.
- *favorito* (boolean): Indica si está en la lista de favoritos del usuario.
- *opinion* (string, nullable): Opinión personal del libro.
- *prestado_a* (string, nullable): Nombre de la persona a la que ha prestado el libro (si aplica).
- *fecha_prestamo* (date, nullable): Fecha en la que lo prestó (si aplica).

2.	Funciones CRUD:

- Agregar nuevos libros a la biblioteca.
- Editar la información de un libro.
- Eliminar libros.
- Listar todos los libros con opciones de filtrado.

3.	Vistas con Blade:

- Listado de libros con opciones de borrado, búsqueda y filtros (por estado de lectura, género, formato, puntuación, favoritos, prestados...).
- Formulario para añadir y editar libros.
- Página de detalle de cada libro con su información completa.

4.	Extras opcionales:

- Mostrar una alerta si un libro lleva prestado más de 30 días.
- Opción de paginar el listado de libros.
- Exportar datos a fichero CSV descargable.
- Gráfico simple con estadísticas de libros leídos vs pendientes.
- Gráfico de barras de libros leídos por año.

### Práctica: Diario personal

Desarrolla una app para llevar un diario personal digital. En esta aplicación, el usuario podrá realizar anotaciones en cualquier momento, organizarlas por categorías y visualizar un listado con opciones de filtrado. Una anotación puede ser cualquier cosa que se le pase por la cabeza, una idea, reflexión...

A continuación se detallan los requisitos. Deberás hacer las migraciones correspondientes, rutas, controlador, vistas... que necesites para su implementación.

#### Requisitos

1.	Modelo Anotacion con los siguientes campos:

- *titulo* (string): Título de la anotación.
- *contenido* (text): Cuerpo de la anotación.
- *categoria* (enum: "Personal", "Trabajo", "Ideas", "Otros"): Categoría de la anotación. INVESTIGA sobre cómo utilizar un tipo enumerado
- *fecha* (date): Fecha en la que se creó la anotación.
- *favorito* (boolean): Indica si es una anotación destacada.

2.	Funciones CRUD:

- Crear nuevas anotaciones.
- Editar y actualizar anotaciones existentes.
- Eliminar anotaciones.
- Listar todas las anotaciones con opciones de filtrado.

3.	Vistas con Blade:

- Listado de anotaciones, con búsqueda y filtros por categoría, fecha o si está marcada como favorita.
- Formulario para añadir anotaciones con fecha por defecto, la de hoy.
- Formulario para editar anotaciones.
- Vista de detalle de una anotación.

4.	Extras opcionales:

- Posibilidad de marcar una anotación como "favorita" y que se muestre destacada.
- Ordenar las anotaciones por fecha (más recientes primero).
- Implementar un calendario donde el usuario pueda ver qué días tiene anotaciones.
