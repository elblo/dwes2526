# Gestión de datos en Laravel: Actividades resueltas

A continuación, vas a realizar una serie de ejercicios sencillos sobre cada uno de los apartados vistos en el tema. Puedes crear un proyecto nuevo o reutilizar uno existente.

## Migraciones

En este apartado vas a trabajar creando migraciones. Es importante, que aparte del código en sí, apuntes los comandos que utilizas para crearlas, eliminarlas, ejecutarlas...

801. **Crear de una tabla básica**: Crea una tabla llamada productos con las siguientes columnas:

- *id* (entero, clave primaria, auto-incremental)
- *nombre* (string, longitud máxima de 255)
- *precio* (decimal, 8 dígitos en total, 2 decimales)

??? info "Solución"
    Ejecuta: `php artisan make:migration create_productos_table`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::create('productos', function (Blueprint $table) {
            $table->id();
            $table->string('nombre');
            $table->decimal('precio', 8, 2);
            $table->timestamps();
        });
    }

    public function down(){
        Schema::dropIfExists('productos');
    }
    ```

    Ejecuta: `php artisan migrate`

802. **Añadir columnas a una tabla existente**: Añade una columna *descripcion* (tipo texto) a la tabla productos.

??? info "Solución"
    Ejecuta: `php artisan make:migration add_descripcion_to_productos_table`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::table('productos', function (Blueprint $table) {
            $table->text('descripcion')->nullable();
        });
    }

    public function down(){
        Schema::table('productos', function (Blueprint $table) {
            $table->dropColumn('descripcion');
        });
    }
    ```

    Ejecuta: `php artisan migrate`

803. **Crear una tabla con claves foráneas**: Crea una tabla *categorias* y una tabla *productos* donde cada producto pertenece a una categoría.

??? info "Solución"
    Ejecuta: `php artisan make:migration create_categorias_table`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::create('categorias', function (Blueprint $table) {
            $table->id();
            $table->string('nombre');
            $table->timestamps();
        });
    }

    public function down(){
        Schema::dropIfExists('categorias');
    }
    ```

    Como la tabla *productos* ya la tenemos creada, hacemos una migración que añada el campo necesario con la clave foránea a la tabla *categorias*.

    Ejecuta: `php artisan make:migration add_categoria_id_to_productos_table`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::table('productos', function (Blueprint $table) {
            $table->foreignId('categoria_id')->constrained()->onDelete('cascade');
            // $table->foreignId('categoria_id')->constrained([table: 'categorias', indexName: 'id'])->onDelete('cascade');
        });
    }

    public function down(){
        Schema::table('productos', function (Blueprint $table) {
            $table->dropColumn('categoria_id');
        });
    }
    ```

    Ejecuta: `php artisan migrate`

804. **Modificar una tabla para añadir índices**: Añade un índice único a la columna *nombre* de la tabla *categorias*.

??? info "Solución"
    Ejecuta: `php artisan make:migration add_unique_to_nombre_in_categorias_table`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::table('categorias', function (Blueprint $table) {
            $table->unique('nombre');
        });
    }

    public function down(){
        Schema::table('categorias', function (Blueprint $table) {
            $table->dropUnique('categorias_nombre_unique'); // table_column_unique
        });
    }
    ```

    Ejecuta: `php artisan migrate`

805. **Eliminar una columna de una tabla**: Elimina la columna *descripcion* de la tabla *productos**.

??? info "Solución"
    Ejecuta: `php artisan make:migration drop_descripcion_from_productos_table`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::table('productos', function (Blueprint $table) {
            $table->dropColumn('descripcion');
        });
    }

    public function down(){
        Schema::table('productos', function (Blueprint $table) {
            $table->text('descripcion')->nullable();
        });
    }
    ```

    Ejecuta: `php artisan migrate`

806. **Renombrar una tabla**: Cambia el nombre de la tabla *productos* a *articulos*.

??? info "Solución"
    Ejecuta: `php artisan make:migration rename_productos_to_articulos`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::rename('productos', 'articulos');
    }

    public function down(){
        Schema::rename('articulos', 'productos');
    }
    ```

    Ejecuta: `php artisan migrate`

807. **Usar valores predeterminados en una columna**: Añade una columna *stock* con un valor por defecto de 0 a la tabla *productos*.

??? info "Solución"
    Ejecuta: `php artisan make:migration add_stock_to_productos_table`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::table('productos', function (Blueprint $table) {
            $table->integer('stock')->default(0);
        });
    }

    public function down(){
        Schema::table('productos', function (Blueprint $table) {
            $table->dropColumn('stock');
        });
    }
    ```

    Ejecuta: `php artisan migrate`

808. **Crear tabla con datos iniciales**: Crear tabla *usuarios* con los siguientes campos:

- *id*
- *nombre* (string)
- *email* (string, único)
- *password* (string)
- *created_at* y *updated_at*

Además, rellénala con datos iniciales mediante el seeder DatabaseSeeder (opcional, se ve en el tema siguiente).

??? info "Solución"
    Ejecuta: `php artisan make:migration create_usuarios_table`

    Contenido del archivo de la migración:

    ```php
    <?php
    public function up(){
        Schema::create('usuarios', function (Blueprint $table) {
            $table->id();
            $table->string('nombre');
            $table->string('email')->unique();
            $table->string('password');
            $table->timestamps();
        });
    }

    public function down(){
        Schema::dropIfExists('usuarios');
    }
    ```

    Inserta en la función `run` de `database/seeders/DatabaseSeeder.php`:

    ```php
    <?php
    DB::table('usuarios')->insert([
        ['nombre' => 'Juan', 'email' => 'juan@example.com', 'password' => bcrypt('123456')],
        ['nombre' => 'Ana', 'email' => 'ana@example.com', 'password' => bcrypt('123456')],
    ]);
    ```

    Ejecuta: `php artisan migrate --seed`

809. **Borrar y recrear la BDD**: Utiliza los comandos de Artisan necesarios para eliminar y volver a crear todas las tablas de la BDD.

??? info "Solución"
    Ejecuta: `php artisan migrate:fresh`

810. **Ejercicio completo: Crear un sistema de reservas**: Crea las siguientes tablas para un sistema de reservas:

- *usuarios* (id, nombre, email, password)
- *habitaciones* (id, nombre, capacidad)
- *reservas* (id, usuario_id, habitacion_id, fecha_reserva)

Incluye claves foráneas, valores predeterminados y relación de "cascade delete".

??? info "Solución"
    Ejecuta: ``

    Contenido del archivo de la migración:

    ```php
    <?php
    
    ```

    Ejecuta: `php artisan migrate`

## Query Builder

En este apartado vas a trabajar haciendo consultas directamente sobre la BDD mediante *Query Builder*. 

Para probar que funciona, se recomienda meter el código de cada ejercicio en una función independiente del controlador que se llamará con una ruta que te inventes (por ejemplo: `localhost/ejercicio820`). Nota: Dicha función del controlador debería llamar a la función correspondiente del modelo y ahí es donde insertarías el código de Query Builder, pero por simplificar, puedes trabajar únicamente sobre el controlador.

No olvides: `use Illuminate\Support\Facades\DB;` para que importe la BD.

Para todos los ejercicios se va a utilizar la tabla *productos*. Si no la tienes, créala mediante una migración con los campos *id*, *nombre*, *precio*, *descripcion*. O si la renombraste a *articulos*, vuelve a crear una migración en la que le cambies el nombre.

820. **Insertar registros**: Inserta un nuevo producto en la tabla. Puedes crear una ruta a la que se le pasen los parámetros *nombre*, *precio* y *descripcion*.

??? info "Solución"
    Las variables $nombre, $precio y $descripcion se han pasado por la ruta.

    ```php
    <?php
    DB::table('productos')->insert([
        'nombre' => $nombre,
        'precio' => $precio,
        'descripcion' => $descripcion,
        'created_at' => now(),
        'updated_at' => now(),
    ]);
    ```

821. **Actualizar registros**: Actualiza el *nombre* de un producto por su *id* (ambos de pasan por parámetro a la ruta).

??? info "Solución"
    Las variables $id y $nombre se han pasado por la ruta.

    ```php
    <?php
    DB::table('productos')
        ->where('id', $id)
        ->update(['nombre' => $nombre]);
    ```

822. **Actualizar registros**: Actualiza el *precio* de un producto por su *id* (ambos de pasan por parámetro a la ruta).

??? info "Solución"
    Las variables $id y $precio se han pasado por la ruta.

    ```php
    <?php
    DB::table('productos')
        ->where('id', $id)
        ->update(['precio' => $precio]);
    ```

823. **Actualizar registros**: Actualiza la *descripcion* de un producto por su *id* (ambos de pasan por parámetro a la ruta).

??? info "Solución"
    Las variables $id y $descripcion se han pasado por la ruta.

    ```php
    <?php
    DB::table('productos')
        ->where('id', $id)
        ->update(['descripcion' => $descripcion]);
    ```

824. **Eliminar registros**: Elimina un producto según el *id* pasado por la ruta.

??? info "Solución"
    Las variable $id se ha pasado por la ruta.

    ```php
    <?php
    DB::table('productos')->where('id', $id)->delete();
    ```

825. **Eliminar registros**: Elimina los productos cuyo *precio* sea inferior a 20.

??? info "Solución"
    Las variables $id y $precio se han pasado por la ruta.

    ```php
    <?php
    DB::table('productos')->where('precio', '<', 20)->delete();
    ```

826. **Obtener todos los registros**: Obtén todos los registros de la tabla *productos*. 

??? info "Solución"

    ```php
    <?php
    $productos = DB::table('productos')->get();
    ```

827. **Obtener registro por id**: Obtén un registro por su *id* pasado por la ruta.

??? info "Solución"
    Las variable $id se ha pasado por la ruta.

    ```php
    <?php
    $producto = DB::table('productos')->find($id);
    //$producto = DB::table('productos')->where('id', $id)->get();
    ```

828. **Seleccionar columnas específicas**: Obtén solo las columnas *nombre* y *precio* de todos los registros de *productos*.

??? info "Solución"
    Contenido del archivo de la migración:

    ```php
    <?php
    $productos = DB::table('productos')->select('nombre', 'precio')->get();
    ```

829. **Filtrar registros con where**: Obtén los productos cuyo precio sea mayor a 50.

??? info "Solución"

    ```php
    <?php
    $productos = DB::table('productos')->where('precio', '>', 50)->get();
    ```

830. **Filtrar registros con where**: Obtén productos cuyo *precio* esté entre 50 y 100, y cuya *descripción* no sea nula.

??? info "Solución"

    ```php
    <?php
    $productos = DB::table('productos')
        ->whereBetween('precio', [50, 100])
        ->whereNotNull('descripcion')
        ->get();
    ```

831. **Ordenar resultados**: Ordena los productos por precio de forma descendente.

??? info "Solución"

    ```php
    <?php
    $productos = DB::table('productos')->orderBy('precio', 'desc')->get();
    ```

832. **Paginar resultados**: Pagina los productos mostrando 5 por página.

??? info "Solución"

    ```php
    <?php
    $productos = DB::table('productos')->paginate(5);
    ```

833.  **Contar registros**: Cuenta cuántos productos tienen un **precio** mayor a 100.

??? info "Solución"

    ```php
    <?php
    $productosMayor100 = DB::table('productos')->where('precio', '>', 100)->count();
    ```

834. **Obtener el registro más caro**: Obtén el producto con el *precio* más alto.

??? info "Solución"

    ```php
    <?php
    $productoMasCaro = DB::table('productos')->orderBy('precio', 'desc')->first();
    ```

835. **Ejecutar consultas crudas**: Usa una consulta SQL "cruda" para obtener productos cuyo *nombre* contenga la palabra "Premium".

??? info "Solución"

    ```php
    <?php
    $productos = DB::select('SELECT * FROM productos WHERE nombre LIKE ?', ['%Premium%']);
    ```

836. **Consulta con uniones (join)**: Crea la migración corresponediente para crear la tabla *categorias* (*id*, *nombre*) y hacer que cada producto pertenezca a una categoría. Una vez hecho, mediante Query Builder obtén el *nombre del producto* junto al *nombre de su categoría*.

??? info "Solución"

    ```php
    <?php
    $productos = DB::table('productos')
        ->join('categorias', 'productos.categoria_id', '=', 'categorias.id')
        ->select('productos.nombre as producto', 'categorias.nombre as categoria')
        ->get();
    ```

837. **Agrupar resultados con groupBy y having**: Agrupa los productos por *categoría* y calcula el *precio promedio* por categoría, mostrando solo las categorías con un promedio mayor a 50.

??? info "Solución"

    ```php
    <?php
    $categoriasPromedioMayor50 = DB::table('productos')
        ->select(DB::raw('categoria, AVG(precio) as promedio_precio'))
        ->groupBy('categoria')
        ->having('promedio_precio', '>', 50)
        ->get();
    ```

838. **Consultas anidadas**: Encuentra el producto más caro dentro de cada categoría.

??? info "Solución"

    ```php
    <?php
    $productosMasCaros = DB::table('productos as p1')
        ->select('p1.*')
        ->whereRaw('precio = (SELECT MAX(precio) FROM productos WHERE categoria_id = p1.categoria_id)')
        ->get();
    ```

839. **Ejercicio completo: CRUD con Query Builder** Implementa un CRUD completo para la tabla *clientes* utilizando Query Builder:

- *C*: Inserta nuevos clientes.
- *R*: Obtén todos los clientes y filtra por email.
- *U*: Actualiza el nombre de un cliente específico.
- *D*: Elimina clientes con un email específico.

## Eloquent: Modelos

Antes has trabajado lanzando consultas mediante *Query Builder* directamente sobre la tabla *productos*. Ahora harás consultas parecidas, pero SIEMPRE desde el modelo mediante *Eloquent*.

Para probar que funciona, se recomienda meter el código de cada ejercicio en una función independiente del controlador que se llamará con una ruta que te inventes (por ejemplo: `localhost/ejercicio841`).

840. **Crear modelo y tabla asociada**: Crea el modelo *Producto* con su tabla asociada *productos* (ya la tienes del apartado anterior).

??? info "Solución"
    Recuerda que con el flag -m crea la migración asociada.

    Ejecuta: `php artisan make:model Producto`


841. **Inserta productos**: Inserta un producto con los campos *nombre*, *precio* y *descripcion* pasados mediante parámetro por la ruta.

??? info "Solución"
    En una función del controlador:

    ```php
    <?php
    use App\Models\Producto;

    // El siguiente código en una función del controlador a la que se llega mediante la ruta
    // Al utilizar 'create' no olvides definir en el modelo 'Producto':
    // protected $fillable = ['titulo', 'contenido', 'prioridad'];
    Producto::create([
        'nombre' => $nombre,
        'precio' => $precio,
        'descripcion' => $descripcion,
    ]);
    ```

842. **Inserta productos**: Inserta un producto con los campos *nombre*, *precio* y *descripcion* pasados mediante parámetro por la ruta, pero validando previamente que su *precio* sea mayor que 50 para insertarlo realmente.

??? info "Solución"
    En el controlador:

    ```php
    <?php
    if ($precio > 50) { 
        $producto = new Producto();
        $producto->nombre = $nombre;
        $producto->precio = $precio;
        $producto->descripcion = $descripcion;
        $producto->save();
        echo "Producto guardado correctamente.";
    } else {
        echo "El precio del producto es demasiado bajo.";
    }
    ```

843. **Actualizar productos**: Actualiza el *nombre* de un producto por su *id* (ambos de pasan por parámetro a la ruta).

??? info "Solución"

    ```php
    <?php
    $producto = Producto::find($id);
    $producto->nombre = $nombre;
    $producto->save();
    ```

844. **Actualizar productos**: Actualiza el *precio* de un producto por su *id* (ambos de pasan por parámetro a la ruta).

??? info "Solución"

    ```php
    <?php
    $producto = Producto::find($id);
    $producto->precio = $precio;
    $producto->save();
    ```


845. **Actualizar productos**: Actualiza la *descripcion* de un producto por su *id* (ambos de pasan por parámetro a la ruta).

??? info "Solución"

    ```php
    <?php
    $producto = Producto::find($id);
    $producto->descripcion = $descripcion;
    $producto->save();
    ```

846. **Actualizar múltiples productos**: Actualiza todos los productos cuyo precio sea menor que 50, cambiando su descripcion a 'producto económico'.

??? info "Solución"

    ```php
    <?php
    Producto::where('precio', '<', 50)->update(['descripcion' => 'Producto económico']);
    ```

847. **Eliminar productos**: Elimina un producto según el *id* pasado por la ruta.

??? info "Solución"

    ```php
    <?php
    Producto::destroy($id);
    // $producto = Producto::find($id);
    // $producto->delete();
    ```

848. **Eliminar productos**: Elimina los productos cuyo *precio* sea inferior a 20.

??? info "Solución"

    ```php
    <?php
    Producto::where('precio', '<', 20)->delete();
    ```

849. **Obtener todos los productos**: Obtén todos los productos de la tabla *productos*.

??? info "Solución"

    ```php
    <?php
    $productos = Producto::all();
    ```

850. **Obtener producto por id**: Obtén un producto por su *id* pasado por la ruta.

??? info "Solución"

    ```php
    <?php
    $producto = Producto::find($id);
    ```

851. **Filtrar con where**: Obtén los productos cuyo *precio* sea mayor a 50.

??? info "Solución"

    ```php
    <?php
    $productos = Producto::where('precio', '>', 50)->get();
    ```

852. **Contar el número de productos**: Cuenta cuántos productos tienen un *precio* mayor a 50.

??? info "Solución"

    ```php
    <?php
    $cantidadProductosMayor50 = Producto::where('precio', '>', 50)->count();
    ```

853. **Ordenar resultados**: Ordena los productos por *precio* de manera descendente.

??? info "Solución"

    ```php
    <?php
    $productos = Producto::orderBy('precio', 'desc')->get();
    ```

854. **Usar el método pluck** para obtener solo los nombres de los productos.

??? info "Solución"
    El método `pluck` recupera todos los valores para una clave dada de un array o, en este caso, los valores de una columna de una tabla.

    ```php
    <?php
    $nombres = Producto::pluck('nombre');
    ```

855. **Usar el método firstOrCreate** para buscar un producto por su *nombre*, y si no existe, crea un nuevo producto.

??? info "Solución"
    El método `firstOrCreate` busca un producto por su nombre, y si no existe, lo crea.

    ```php
    <?php
    $producto = Producto::firstOrCreate(
        ['nombre' => $nombre],
        ['precio' => 80.00, 'descripcion' => 'Nuevo producto']
    );
    ```

856. **Usar el método updateOrCreate** para actualizar un producto existente por su *nombre*, o crear uno nuevo si no existe.

??? info "Solución"
    El método `updateOrCreate` busca un producto por su nombre, y si no existe, lo crea.

    ```php
    <?php
    $producto = Producto::updateOrCreate(
        ['nombre' => $nombre],
        ['precio' => 120.00, 'descripcion' => 'Producto Y actualizado']
    );
    ```

857. **Limitar resultados**: Usa el método *take* para obtener solo los 5 primeros productos.

??? info "Solución"

    ```php
    <?php
    $productos = Producto::take(5)->get();
    ```

858. **Paginación de resultados**: Página los productos mostrando 5 por página.
    
??? info "Solución"

    ```php
    <?php
    $productos = Producto::paginate(5);
    ```

859. **Consulta con where y orWhere**: Recupera todos los productos cuyo precio sea mayor que 100 o cuyo *nombre* contenga la palabra "Premium".

??? info "Solución"

    ```php
    <?php
    $productos = Producto::where('precio', '>', 100)
        ->orWhere('nombre', 'LIKE', '%Premium%')
        ->get();
    ```

860. **Ejercicio completo: CRUD con Eloquent** Implementa un CRUD completo mediante Eloquent para el modelo *Cliente* que crees asociado a la tabla *clientes*:

- *C*: Crea nuevos clientes.
- *R*: Obtén todos los clientes y filtra por email.
- *U*: Actualiza el nombre de un cliente específico.
- *D*: Elimina clientes con un email específico.

## Formularios

En los siguientes ejercicios de formularios vas a realizar un CRUD de productos continuando lo que hiciste en el apartado anterior. Si tienes las clases de `Productos` (Modelo y Controlador) muy extensas y prefieres empezar de 0, puedes hacer los ejercicios siguientes para gestionar *usuarios en vez de productos*. Tendrías que crear previamente el modelo *Usuario* con su migración para crear su tabla asociada *usuarios* con los campos típicos: *nombre*, *email*, *password* y también el controlador *UsuarioController*.

En cualquier caso, recuerda nombrar correctamente las rutas, funciones en controladores y vistas siguiendo las recomendaciones de Laravel.

870. **Formulario para crear productos**: Crea un formulario en el que se pidan los campos necesarios para crear un producto. Se accederá mediante `GET /productos/create` y su vista estará en `productos/create.blade.php`. El formulario se procesará mediante `POST /productos/store` redirigiendo finalmente a `GET /productos` donde se muestra el listado de productos.

??? info "Solución"
    Formulario en `productos/create.blade.php`:

    ```html
    <form action="{{ route('productos.store') }}" method="POST">
        @csrf
        <p>
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" name="nombre" required>
        </p>
        <p>
            <label for="descripcion">Descripcion:</label>
            <input type="text" id="descripcion" name="descripcion" required>
        </p>
        <p>
            <label for="precio">Precio:</label>
            <input type="number" id="precio" name="precio" required>
        </p>
        <p><button type="submit">Crear Producto</button></p>
    </form>
    ```

    Función en `ProductoController.php` para almacenar el producto en la BDD:

    ```php
    <?php
    use App\Models\Producto;
    use Illuminate\Http\Request;

    class ProductoController extends Controller
    {
        public function store(Request $request)
        {
            $validated = $request->validate([
                'nombre' => 'required|string|max:255',
                'descripcion' => 'required|string|min:20',
                'precio' => 'required|decimal:2|min:0',
            ]);

            // Al utilizar 'create' no olvides definir en el modelo 'Producto':
            // protected $fillable = ['titulo', 'contenido', 'prioridad'];
            $producto = Producto::create([
                'nombre' => $validated['nombre'],
                'descripcion' => $validated['descripcion'],
                'precio' => $validated['precio'],
            ]);

            // Producto::create($validated);

            return redirect()->route('productos.index');
        }
    }
    ```

871. **Formulario para editar productos**: Crea el formulario de edición de un producto. Se accederá mediante `GET /productos/{id}/edit` y su vista estará en `productos/edit.blade.php`. El formulario se procesará mediante `PUT /productos/update` redirigiendo finalmente a `GET /productos` donde se muestra el listado de productos.

??? info "Solución"
    Formulario en `productos/edit.blade.php`:

    ```html
    <form action="{{ route('productos.update', $producto->id) }}" method="POST">
        @method('PUT') 
        @csrf
        <p>
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" name="nombre" value="{{ $producto->nombre }}" required>
        </p>
        <p>
            <label for="descripcion">Descripcion:</label>
            <input type="text" id="descripcion" name="descripcion" value="{{ $producto->descripcion }}" required>
        </p>
        <p>
            <label for="precio">Precio:</label>
            <input type="number" id="precio" name="precio" value="{{ $producto->precio }}" required>
        </p>
        <p><button type="submit">Actualizar Producto</button></p>
    </form>
    ```

    Función en `ProductoController.php` para actualizar el producto en la BDD:

    ```php
    <?php
    use App\Models\Producto;
    use Illuminate\Http\Request;

    class ProductoController extends Controller
    {
        public function update(Request $request, $id)
        {
            $producto = Producto::findOrFail($id);

            $validated = $request->validate([
                'nombre' => 'required|string|max:255',
                'descripcion' => 'required|string|min:20',
                'precio' => 'required|decimal:2|min:0',
            ]);

            $producto->nombre = $validated['nombre'];
            $producto->descripcion = $validated['descripcion'];
            $producto->precio = $validated['precio'];

            $producto->save();            

            return redirect()->route('productos.index');
        }
    }
    ```

872. **Validación de datos**: En las funciones correspondientes del controlador donde se reciben los datos de los formularios anteriores, añade validación a cada uno de los campos:

- *nombre*: Requerido, tipo cadena y valor máximo 255.
- *descripcion*: Tipo cadena y valor máximo 1000.
- *precio*: Requerido, tipo numérico con 2 decimales y valor mínimo 0.

En las vistas de los 2 formularios añade mensajes de error en el caso de que los campos no pasen la validación y asigna mediante *old* el valor antiguo del campo para que el usuario no tenga que volver a escribirlo.

??? info "Solución"
    Validación en el controlador ya añadida en las soluciones anteriores.

    En las vistas:

    Formulario en `productos/create.blade.php` con mensajes de error y valores antiguos de los campos:

    ```html
    <form action="{{ route('productos.store') }}" method="POST">
        @csrf
        <p>
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" name="nombre" value="{{ old('nombre') }}" required>
        </p>
        @error('nombre') <div class="error">{{ $message }}</div>@enderror
        <p>
            <label for="descripcion">Descripcion:</label>
            <input type="text" id="descripcion" name="descripcion" value="{{ old('descripcion') }}" required>
        </p>
        @error('descripcion') <div class="error">{{ $message }}</div>@enderror
        <p>
            <label for="precio">Precio:</label>
            <input type="number" id="precio" name="precio" value="{{ old('precio') }}" required>
        </p>
        @error('precio') <div class="error">{{ $message }}</div>@enderror
        <p><button type="submit">Crear Producto</button></p>
    </form>
    ```

    Formulario en `productos/edit.blade.php` con mensajes de error y valores antiguos de los campos:

    ```html
    <form action="{{ route('productos.update', $producto->id) }}" method="POST">
        @method('PUT')
        @csrf
        <p>
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" name="nombre" value="{{ old('nombre', $producto->nombre) }}" required>
        </p>
        @error('nombre') <div class="error">{{ $message }}</div>@enderror
        <p>
            <label for="descripcion">Descripcion:</label>
            <input type="text" id="descripcion" name="descripcion" value="{{ old('descripcion', $producto->descripcion) }}" required>
        </p>
        @error('descripcion') <div class="error">{{ $message }}</div>@enderror
        <p>
            <label for="precio">Precio:</label>
            <input type="number" id="precio" name="precio" value="{{ old('precio', $producto->precio) }}" required>
        </p>
        @error('precio') <div class="error">{{ $message }}</div>@enderror
        <p><button type="submit">Actualizar Producto</button></p>
    </form>
    ```

873. **Formulario de confirmación para eliminar productos**: Crear un formulario de confirmación para eliminar un recurso. Sólo contendrá un mensaje de "¿Estás seguro que quieres eliminar este producto?" y un botón para proceder a eliminarlo. Se accederá mediante `GET /productos/{id}/destroy` y su vista estará en `productos/destroy.blade.php`. El formulario se procesará mediante `DELETE /productos/{id}` redirigiendo finalmente a `GET /productos` donde se muestra el listado de productos.

??? info "Solución"
    Formulario en `productos/destroy.blade.php` para confirmar eliminación:

    ```html
    <form action="{{ route('productos.destroy', $producto->id) }}" method="POST">
        @method('DELETE')
        @csrf
        <p>¿Estás seguro de que deseas eliminar este producto?</p>
        <button type="submit">Eliminar Producto</button>
    </form>
    ```

    Función en `ProductoController.php` para eliminar el producto en la BDD:

    ```php
    <?php
    public function destroy($id)
    {
        $producto = Producto::findOrFail($id);
        $producto->delete();

        return redirect()->route('productos.index');
    }
    ```

874. **Ejercicio completo: CRUD con formularios**: Continúa el CRUD del apartado anterior para añadir funciones a la gestión de *clientes*:

- *C*: Crea nuevos clientes.
- *R*: Obtén todos los clientes y filtra por diferentes cmapos.
- *U*: Actualiza los campos de un cliente específico.
- *D*: Elimina clientes.

