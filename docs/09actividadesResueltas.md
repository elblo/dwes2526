# Laravel Avanzado: Actividades resueltas

A continuación, vas a realizar una serie de ejercicios sobre cada uno de los apartados vistos en el tema. Puedes crear un proyecto nuevo o reutilizar uno existente.

### Manejo de ficheros

Para practicar con los ficheros vas a crear una galería de imágenes con posibilidad de subir nuevas, eliminar y acceder a su vista en detalle. También vas a implementar un "Mini Drive" para gestionar archivos. En ambos casos, vas a trabajar sin modelos para centrarte exclusivamente en el manejo de ficheros, pero en una app real, además sería recomendable trabajar con una BDD que almacene la información necesaria de los archivos.

901. **Formulario de subida**: Crea un formulario para subir una imagen. La ruta que lleva al formulario por GET será `imagen/create` y la vista del mismo será `imagen/create.blade.php`. El formulario se enviará por POST a la ruta `imagen/storage`. Además del propio input de la imagen, el formulario tendrá un radio button para seleccionar si la imagen se almacenará de forma privada o pública en el storage.

??? info "Solución"

    La ruta para abrir el formulario:

    ```php
    <?php
    
    ```

    La vista `imagen/create.blade.php` con el formulario de subida:

    ```html
        <form action="{{ route('subir.imagen') }}" method="POST" enctype="multipart/form-data">
        @csrf
        <input type="file" name="imagen" required>
        <button type="submit">Subir Imagen</button>
    </form>
    ```

902. **Almacenar archivos**: Crea la función correspondiente en el controlador para recibir la imagen del formulario anterior validando que sea del tipo imagen, requerida y con un tamaño máximo de 2MB. Y según la opción del formulario, almacena la imagen de forma privada o pública en el storage. Después redirecciona a la página anterior enviando un mensaje del tipo "Imagen NOMBRE almacenada correctamente en el storage privado|público" que mostrarás justo encima del formulario.

??? info "Solución"

    La función 'store' del controlador almacena la imagen en el directorio `storage/app/public/imagenes`:

    ```php
    <?php
    use Illuminate\Http\Request;

    class ArchivoController extends Controller
    {
        public function subirImagen(Request $request)
        {
            $request->validate([
                'imagen' => 'required|image|mimes:jpeg,png,jpg,gif,svg|max:2048',
            ]);

            if ($request->file('imagen')) {
                $path = $request->file('imagen')->store('imagenes', 'public');
                return back()->with('success', 'Imagen subida correctamente.')->with('path', $path);
            }

            return back()->withErrors('Error al subir la imagen.');
        }
    }
    ```

    Asegúrate de crear el enlace simbólico desde 'public' para que las imágenes sean accesibles públicamente. Ve a la raíz del proyecto y ejectua (sólo se hace una vez):

    ```console
    php artisan storage:link
    ```

903. **Mostrar archivos**: Mediante la ruta por GET `imagen` que lleva a la vista `imagen/index.blade.php` muestra una galería con todas las imágenes en miniatura. Puedes utilizar flexbox o grid layout para posicionarlas unas al lado de las otras. **Importante**: No olvides crear el enlace simbólico para poder acceder a las imágenes. Crea un enlace "Subir imagen" que lleve al formulario del primer punto y en dicho formulario, un enlace para volver aquí, al listado.

??? info "Solución"

    ```php
    <?php
    
    ```

904. **Mostrar imagen completa**: Mediante la ruta por GET `imagen/{name}` que lleva a la vista `imagen/show.blade.php` muestra la vista en tamaño completo de la imagen. Muestra un párrafo con su ruta completa y un enlace para volver al listado.

??? info "Solución"

    ```php
    <?php
    
    ```
    
905. **Eliminar archivos**: Mediante la ruta por GET `imagen/{name}/destroy` elimina la imagen que corresponda. A esta ruta podrás llegar mediante un enlace de la vista en detalle de la imagen. Una vez eliminada, se redirige automáticamente al listado enviando con 'with' un mensaje del tipo "Imagen NOMBRE eliminada correctamente del storage público".

??? info "Solución"

    ```php
    <?php
    use Illuminate\Support\Facades\Storage;

    public function eliminarImagen($filename)
    {
        $path = 'imagenes/' . $filename;

        if (Storage::disk('public')->exists($path)) {
            Storage::disk('public')->delete($path);
            return back()->with('success', 'Imagen eliminada correctamente.');
        }

        return back()->withErrors('El archivo no existe.');
    }
    ```

    Con la ruta:

    ```php
    <?php
    Route::delete('/eliminar-imagen/{filename}', [ArchivoController::class, 'eliminarImagen']);
    ```


906. **Almacenamiento en S3** (opcional): Con tu cuenta de estudiante de AWS, crea un bucket S3 con acceso público. Configura el proyecto actual para utilizar el disco 's3' por defecto o bien utilízalo de forma explícita en cada interacciíon que realices con el Storage.

??? info "Solución"

    Crea el bucket S3 en AWS y configura en `.env`:

    ```env
    AWS_ACCESS_KEY_ID=your-access-key-id
    AWS_SECRET_ACCESS_KEY=your-secret-access-key
    AWS_DEFAULT_REGION=your-region
    AWS_BUCKET=your-bucket-name   
    ```

    Asegúrate que en `config/filesystems.php` tengas configurado el disco 's3':

    ```php
    <?php
    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'url' => env('AWS_URL'),
    ],
    ```

    Ahora, en la función 'store' del controlador deberás indicar que se almacene en el disco 's3':

    ```php
    <?php
    public function subirImagen(Request $request)
    {
        $request->validate([
            'imagen' => 'required|image|mimes:jpeg,png,jpg,gif,svg|max:2048',
        ]);

        if ($request->file('imagen')) {
            $path = $request->file('imagen')->store('imagenes', 's3');
            return back()->with('success', 'Imagen subida correctamente.')->with('path', $path);
        }

        return back()->withErrors('Error al subir la imagen.');
    }
    ```

907. **Mini Drive**: De forma similar a lo que acabas de hacer con la galería, crea un sistema de almecenamiento de archivos que admita archivos de diferente tipo (imágenes, videos, documentos...). Deberás mostrar un listado con un icono según el tipo de archivo, su nombre, tamaño y opciones (eliminar), un formulario para subirlo con un campo en el que recojas el nombre con el que almacennarlo. Y en vez de la vista en detalle, al pulsar sobre el archivo en la vista del listado, se descargará directamente. Sigue las recomendaciones de los puntos anteriores.

??? info "Solución"

    En la función 'store' del formulario ahora deberás cambiar el nombre del archivo conservando su extensión mediante *getClientOriginalExtension()*:

    ```php
    <?php
    public function subirImagen(Request $request)
    {
        $request->validate([
            'imagen' => 'required|image|mimes:jpeg,png,jpg,gif,svg|max:2048',
        ]);

        if ($request->file('imagen')) {
            $nombreArchivo = time() . '.' . $request->file('imagen')->getClientOriginalExtension();
            $path = $request->file('imagen')->storeAs('imagenes', $nombreArchivo, 'public');
            return back()->with('success', 'Imagen subida correctamente.')->with('path', $path);
        }

        return back()->withErrors('Error al subir la imagen.');
    }
    ```

908. **Mini Drive con directorios** (opcional): Investiga cómo crear directorios y mover archivos entre ellos. Ofrece en la interfaz que has creado, las opciones correspondientes para crear un nuevo directorio, para cambiar el nombre a un archivo (si no existe uno ya con dicho nombre) y moverlo a un directorio determinado. 

??? info "Solución"

    Crear un nuevo directorio:

    ```php
    <?php
    // En el controlador
    use Illuminate\Support\Facades\Storage;

    Storage::makeDirectory('public/nueva_carpeta');
    ```

    Si no tienes creado el enlace simbólico desde 'public', ve a la raíz del proyecto y ejectua (sólo se hace una vez):

    ```console
    php artisan storage:link
    ```

    Ya es accesible la carpeta desde `http://tu-sitio.test/storage/nueva_carpeta`.

    Mover archivos entre directorios:

    ```php
    <?php
    public function moverArchivo($filename)
    {
        $pathOrigen = 'imagenes/' . $filename;
        $pathDestino = 'archivos/' . $filename;

        if (Storage::disk('public')->exists($pathOrigen)) {
            Storage::disk('public')->move($pathOrigen, $pathDestino);
            return back()->with('success', 'Archivo movido correctamente.');
        }

        return back()->withErrors('El archivo no existe.');
    }
    ```

    La ruta en `web.php` sería así:

    ```php
    <?php
    Route::post('/mover-archivo/{filename}', [ArchivoController::class, 'moverArchivo']);
    ```

### Request y response

En los siguientes ejercicios vas a trabajar con `Request`y `Response` en las funciones de los controladores. El primero ya lo has utilizado para recoger los datos recibidos de un formulario. Vas a repasar su uso y sobre todo, vas a conocer el segundo.

910. **Obtener y validar datos con Request**: Crea un pequeño formulario para recoger "Nombre" y "Email" del usuario. Recoge sus datos en una función del controlador y valídalos. Si pasan la validación, símplemente devuelve un mensaje "Se ha pasado la validación" y si no, captura los errores en la vista del formulario.

??? info "Solución"

    Función en el controlador para obtener los datos del formulario:

    ```php
    <?php
    use Illuminate\Http\Request;

    class UsuarioController extends Controller
    {
        public function guardar(Request $request)
        {
            $validated = $request->validate([
                'nombre' => 'required|string|max:255',
                'email' => 'required|email|unique:users,email',
            ]);

            // Si no pasa la validación, Laravel redirige al formulario con los errores

            $nombre = $request->input('nombre');
            $email = $request->input('email');

            return "Validación correcta: Nombre: $nombre, Email: $email";
        }
    }
    ```

    En la vista del formulario, mostrar los errores:

    ```html
     @if ($errors->any())
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    @endif
    ```

911. **Redireccionar la respuesta**: Modifica el ejercicio anterior para redireccionar la respuesta a otra ruta.

??? info "Solución"

    ```php
    <?php
    public function guardar(Request $request){
        // ...
        $mensaje = "Validación correcta: Nombre: $nombre, Email: $email";
        return redirect()->route('contacto.gracias')->with('success', $mensaje);
    }

    public function gracias(){ return session('success'); }
    ```

912. **Respuesta JSON**: Crea una ruta GET `/api/usuarios` que devuelva un array de usuarios (id, nombre, email) en formato JSON.

??? info "Solución"

    ```php
    <?php
    use Illuminate\Http\Request;

    class ApiController extends Controller
    {
        public function usuarios()
        {
            $usuarios = [
                ['id' => 1, 'nombre' => 'Juan', 'email' => 'juan@example.com'],
                ['id' => 2, 'nombre' => 'Ana', 'email' => 'ana@example.com'],
            ];

            return response()->json($usuarios);
        }
    }
    ```

913. **Respuesta JSON error**: Crea una ruta GET `/api/error` que devuelva un array (error y mensaje) en formato JSON y además, el código de estado 400 para indicar al cliente que ha enviado una petición inválida.

??? info "Solución"

    ```php
    <?php
    public function error()
    {
        return response()->json([
            'error' => 'Solicitud incorrecta',
            'message' => 'Faltan datos en la solicitud.',
        ], 400);
    }
    ```

914. **Modificar cabeceras de la respuesta**: Crea una ruta GET `/archivo/descargar` que devuelva un archivo descargable y modifique los encabezados de la respuesta. En el controlador, usa *response()->download()* para devolver el archivo estableciendo un encabezado personalizado.

??? info "Solución"

    ```php
    <?php
    public function descargar()
    {
        $file = public_path('archivos/ejemplo.pdf');
        return response()->download($file, 'mi_archivo.pdf', [
            'Content-Type' => 'application/pdf',
        ]);
    }
    ```

915. **Ejercicio completo: API Rest**: Con lo visto hasta ahora, implementa una API RESTful para manejar usuarios:

- **C**: Crear un usuario mediante POST `/api/usuarios`.
- **R**: Listar usuarios con GET `/api/usuarios`.
- **U**: Actualizar un usuario con PUT `/api/usuarios/{id}`.
- **D**: Eliminar un usuario con DELETE `/api/usuarios/{id}`.

Asegúrate de devolver respuestas adecuadas en JSON y manejar los errores correctamente.

### Eloquent: Relaciones

En este apartado vas a crear diferentes relaciones entre modelos.

920. **Relación 1 a 1**. 

- Crea los modelos `Usuario` y `Perfil`. Cada usuario tiene un perfil, y cada perfil pertenece a un único usuario.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `usuarios`: campos `id`, `nombre`, `email`.
  - `perfiles`: campos `id`, `usuario_id`, `telefono`, `direccion`.
- Define la relación en los modelos.

??? info "Solución"

    ```php
    // Modelo Usuario
    use Illuminate\Database\Eloquent\Model;

    class Usuario extends Model{
        public function perfil(){
            return $this->hasOne(Perfil::class);
        }
    }

    // Modelo Perfil
    use Illuminate\Database\Eloquent\Model;

    class Perfil extends Model{
        public function usuario(){
            return $this->belongsTo(Usuario::class);
        }
    }
    ```

- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta los datos de un usuario y muestra su perfil. Para ello, crea una ruta `usuario/{id}` que redirija a la función `show` del controlador y llame a la vista `usuario.show.blade.php` para los datos del usuario con su perfil.

??? info "Solución"

    ```php
    // En el controlador
    public function show(string $id){
        $usuario = Usuario::find($id);
        $perfil = $usuario->perfil; // También se podría recuperar el perfil en la vista
        return view('usuario.show', compact('usuario', 'perfil'));
    }
    ```

921. **Relación 1 a Muchos**. 

- Crea los modelos `Categoria` y `Producto`. Cada categoría tiene muchos productos, pero un producto sólo perteneca una determinada categoría.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `categorias`: campos `id`, `nombre`.
  - `productos`: campos `id`, `nombre`, `precio`, `categoria_id`.
- Define la relación en los modelos.

??? info "Solución"

    ```php
    // Modelo Categoria
    use Illuminate\Database\Eloquent\Model;

    class Categoria extends Model{
        public function productos(){
            return $this->hasMany(Producto::class);
        }
    }

    // Modelo Prodcuto
    use Illuminate\Database\Eloquent\Model;

    class Producto extends Model{
        public function categoria(){
            return $this->belongsTo(Categoria::class);
        }
    }
    ```

- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta el nombre de una categoría mostrando sus productos. Para ello, crea una ruta `categoria/{id}` que redirija a la función `show` del controlador y llame a la vista `categoria.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function show(string $id){
        $categoria = Categoria::find($id);
        $productos = $categoria->productos; // También se podría hacer esto en la vista
        return view('categoria.show', compact('categoria', 'productos'));
    }
    ```

- Mediante Eloquent agrega un nuevo producto a la categoría con id pasado por parámetro. Para ello, crea una ruta `categoria/{id}/addproduct/{nombre}` que redirija a la función `addProduct` del controlador y redirija a la ruta `show` anterior que llama a la vista `categoria.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function addProduct(string $id, string $nombre){
        $categoria = Categoria::find($id);
        $categoria->productos()->create(['nombre' => $nombre]);
        
        return redirect()->route('categoria.show', ['id' => $id]);
    }
    ```

922. **Relación Muchos a Muchos**. 

- Crea los modelos `Estudiante` y `Asignatura`. Cada estudiante puede estár matriculado en muchas asignaturas y una asignatura la cursan muchos estudiantes.
- En las migraciones asegúrate que las tablas tienen los siguientes campos:
  - `estudiantes`: campos `id`, `nombre`.
  - `asignaturas`: campos `id`, `nombre`.
  - `asignatura_estudiante` (tabla pivote): `estudiante_id`, `asignatura_id`.
- Define la relación en los modelos.

??? info "Solución"

    ```php
    // Modelo Estudiante
    use Illuminate\Database\Eloquent\Model;

    class Estudiante extends Model{
        public function asignaturas(){
            return $this->belongsToMany(Asignatura::class);
        }
    }

    // Modelo Asignatura
    use Illuminate\Database\Eloquent\Model;

    class Asignatura extends Model{
        public function estudiantes(){
            return $this->belongsToMany(Estudiante::class);
        }
    }
    ```

- Rellena con 2 ó 3 registros manualmente o mediante Eloquent en ambas tablas/modelos.
- Consulta todas las asignaturas de un estudiante por su id. Para ello, crea una ruta `estudiante/{id}` que redirija a la función `show` del controlador y llame a la vista `estudiante.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function show(string $id){
        $estudiante = Estudiante::find($id);
        $asignaturas = $estudiante->asignaturas; // También se podría hacer esto en la vista
        return view('estudiante.show', compact('estudiante', 'asignaturas'));
    }
    ```

- Consulta ahora todos los estudiantes que cursen una asignatura por su id. Para ello, crea una ruta `asignatura/{id}` que redirija a la función `show` del controlador y llame a la vista `asignatura.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function show(string $id){
        $asignatura = Asignatura::find($id);
        $estudiantes = $asignatura->estudiantes; // También se podría hacer esto en la vista
        return view('asignatura.show', compact('asignatura', 'estudiantes'));
    }
    ```

- Mediante Eloquent matricula a un estudiante en un curso determinado. Para ello, crea una ruta `estudiante/{id}/matricula/{idCurso}` que redirija a la función `matricula` del controlador y redirija a la ruta `show` de estudiante que llama a la vista `estudiante.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function matricula(string $id, string $idCurso){
        $estudiante = Estudiante::find($id);
        $estudiante->cursos()->attach($idCurso);
        
        return redirect()->route('estudiante.show', ['id' => $id]);
    }
    ```

- Mediante Eloquent desmatricula a un estudiante en un curso determinado. Para ello, crea una ruta `estudiante/{id}/desmatricula/{idCurso}` que redirija a la función `desmatricula` del controlador y redirija a la ruta `show` de estudiante que llama a la vista `estudiante.show.blade.php`.

??? info "Solución"

    ```php
    // En el controlador
    public function desmatricula(string $id, string $idCurso){
        $estudiante = Estudiante::find($id);
        $estudiante->cursos()->detach($idCurso);
        
        return redirect()->route('estudiante.show', ['id' => $id]);
    }
    ```

923. **Ejercicio completo**: CRUD con varias relaciones y formularios.

- Implementa un sistema de gestión de posts con sus respectivos comentarios mediante los modelos `Autor`, `Post` y `Comentario`.
- Piensa bien las relaciones a utilizar. Un autor puede escribir muchos posts y comentarios. Un post puede tener muchos comentarios. Un comentario sólo pertenece a un post y está escrito por un autor.
- De un *autor* interesa saber su imagen, nombre y email.
- De un *post* interesa saber su título, fecha y descripción.
- De un *comentario* interesa saber su texto y fecha.
- Crea los CRUDs necesarios para cada modelo con sus rutas específicas, controladores, vistas (con formularios)...

### Mutadores y accesores

930. **Formatear nombres y convertir números**: En el modelo `Producto` del ejercicio anterior, crea:

- Un mutador que almacene el nombre en minúsculas y un accesor que los devuelva con la primera letra en mayúscula.
- Un mutador que almacene el precio convertido a céntimos y un accesor que lo devuelva de nuevo en euros.

??? info "Solución"

    ```php
    namespace App\Models;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Casts\Attribute;

    class Producto extends Model{
        protected $fillable = ['nombre', 'precio'];

        protected function nombre(): Attribute{
            return Attribute::make(
                set: fn ($value) => strtolower($value), // Guardar en minúsculas
                get: fn ($value) => ucfirst($value) // Devolver con la 1ª en mayúsculas
            );
        }

        protected function precio(): Attribute{
            return Attribute::make(
                set: fn ($value) => $value * 100, // Guardar en céntimos
                get: fn ($value) => $value / 100  // Devolver en euros
            );
        }
    }
    ```

931. **Slug automático**: Un slug es una versión formateada de un texto, generalmente usada en URLs por gestores de contenidos. Un slug se crea eliminando caracteres especiales, convirtiendo espacios en guiones y pasando a minúsculas el texto. Por ejemplo: "Hola Mundo Laravel" tendría de slug "hola-mundo-laravel".

En el modelo `Post` del ejercicio anterior:

- Crea la migración correspondiente para añadir el campo `slug` de tipo string.
- Investiga cómo usar `Str::slug` para generar slugs.
- Crea un mutador que convierta el título a slug y lo almacene en el campo `slug`.

??? info "Solución"

    ```php
    <?php
    namespace App\Models;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Support\Str;

    class Post extends Model{
        protected $fillable = ['titulo', 'fecha', 'descripcion', 'slug'];

        protected function titulo(): Attribute{
            return Attribute::make(
                set: function ($value) {
                    return [
                        'titulo' => $value,
                        'slug' => Str::slug($value)
                    ];
                }
            );
        }
    }
    ```

932.  **Formatear fechas de creación**: En el modelo `Estudiante` del ejercicio anterior:

- Investiga cómo usar la biblioteca `Carbon` para trabajar con fechas incluida en Laravel.
- Crea un accesor que formatee la fecha de creación (created_at) en formato "d/m/Y - H:i".

??? info "Solución"

    ```php
    <?php
    namespace App\Models;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Carbon\Carbon;

    class Estudiante extends Model{
        protected $fillable = ['nombre'];

        protected function createdAt(): Attribute{
            return Attribute::make(
                get: fn ($value) => Carbon::parse($value)->format('d/m/Y - H:i')
            );
        }
    }

    // Uso en controlador o vista
    echo $estudiante->created_at; // 04/02/2025 - 15:30
    ```

### Seeders y factories

940. **Seeder básico**: Crea un nuevo modelo `Usuario` con campos `nombre`, `email` y `password` (en su migración) y crea un seeder `UsuarioSeeder` que inserte 3 usuarios de prueba en la base de datos.

??? info "Solución"

    Ejecuta: `php artisan make:seeder UsuarioSeeder`

    ```php
    <?php
    use App\Models\Usuario;
    use Illuminate\Database\Seeder;

    class UsuarioSeeder extends Seeder
    {
        public function run()
        {
            Usuario::create([
                'nombre' => 'Juan Pérez',
                'email' => 'juan@example.com',
                'password' => bcrypt('password123'),
            ]);

            Usuario::create([
                'nombre' => 'Ana Gómez',
                'email' => 'ana@example.com',
                'password' => bcrypt('password123'),
            ]);
            
            Usuario::create([
                'nombre' => 'Adrián Lara',
                'email' => 'adrian@example.com',
                'password' => bcrypt('password123'),
            ]);
        }
    }
    ```

    Añadir al `DatabaseSeeder`:

    ```php
    <?php
    class DatabaseSeeder extends Seeder
    {
        public function run()
        {
            $this->call([
                UsuarioSeeder::class,
                // ...
            ]);
        }
    }
    ```

    Ejecuta: `php artisan db:seed --class=UsuarioSeeder`

941. **Factoría con seeder**: Crea una factoría `UsuarioFactory` con datos fake y en el seeder `UsuarioSeeder` crea 10 usuarios mediante la factoría.

??? info "Solución"

    Ejecuta: `php artisan make:factory UsuarioFactory -m Usuario`

    ```php
    <?php
    namespace Database\Factories;
    use App\Models\Usuario;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class UsuarioFactory extends Factory{
        protected $model = Usuario::class; // Se podría obviar 

        public function definition(): array{
            return [
                'nombre' => $faker->name,
                'email' => $faker->unique()->safeEmail,
                'password' => bcrypt('password123'),
            ];
        }
    }
    ```

    En el modelo `Usuario` añadir el trait `use HasFactory` para asociarlos:

    ```php
    <?php
    class Usuario extends Model{
        use HasFactory;
        // ...
    }
    ```

    En el seeder `UsuarioSeeder` creado en el ejercicio anterior, añade:

    ```php
    <?php
    use App\Models\Usuario;
    use Illuminate\Database\Seeder;

    class UsuarioSeeder extends Seeder
    {
        public function run()
        {
            // ...
            User::factory(10)->create();
        }
    }
    ```

    Ejecuta: `php artisan db:seed --class=UsuarioSeeder`

942. **Seeders con modelos relacionados**: Crea el modelo `Publicacion` con los campos `titulo`, `contenido` y `usuario_id` (en su migración) y modifica los modelos para que un usuario se relacione con muchas publicaciones. Crea las factorías `UsuarioFactory` (ya la tienes) y `PublicacionFactory` con datos fake para utilizar en el seeder `UsuarioPublicacionSeeder` para crear 10 usuarios que tentan entre 1 y 5 publicaciones cada uno.

??? info "Solución"

    Ejecuta: `php artisan make:factory PublicacionFactory -m Publicacion`

    ```php
    <?php
    namespace Database\Factories;
    use App\Models\Publicacion;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class PublicacionFactory extends Factory{
        protected $model = Publicacion::class; // Se podría obviar 

        public function definition(): array{
            return [
                'titulo' => $faker->sentence,
                'contenido' => $faker->paragraph,
                'user_id' => 1, // Esto será modificado en el Seeder
            ];
        }
    }
    ```

    En el modelo `Publicacion` añadir el trait `use HasFactory` para asociarlos:

    ```php
    <?php
    class Usuario extends Model{
        use HasFactory;
        // ...
    }
    ```

    Ejecuta: `php artisan make:seeder UsuarioPublicacionSeeder`

    ```php
    <?php
    use App\Models\Usuario;
    use App\Models\Publicacion;
    use Illuminate\Database\Seeder;

    class UsuarioPublicacionSeeder extends Seeder
    {
        public function run()
        {
            // Crear 10 usuarios
            User::factory(10)->create()->each(function ($user) {
                // Crear entre 1 y 5 posts para cada usuario
                $user->posts()->createMany(Post::factory(rand(1, 5))->make()->toArray());
            });
        }
    }
    ```

    Añadir al `DatabaseSeeder`:

    ```php
    <?php
    class DatabaseSeeder extends Seeder
    {
        public function run()
        {
            $this->call([
                UsuarioPublicacionSeeder::class,
                // ...
            ]);
        }
    }
    ```

    Ejecuta: `php artisan db:seed --class=UsuarioPublicacionSeeder`