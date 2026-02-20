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

# API REST con Laravel

> Duración estimada: 5 sesiones

------------------------------------------------------------------------

## 11.1 Introducción a las APIs REST

### Introducción

Hoy en día, gran parte de las aplicaciones modernas no renderizan HTML
directamente desde el servidor. En su lugar, el backend expone **APIs
REST**, y distintos clientes (frontend SPA, apps móviles, otros
servidores...) consumen esos datos.

Laravel es especialmente potente para este tipo de arquitecturas, ya que
facilita:

-   Definición de rutas para APIs
-   Serialización automática a JSON
-   Validación de datos
-   Autenticación mediante tokens
-   Autorización mediante policies

Una **API REST** se basa en:

- Recursos (usuarios, productos, pedidos...)
- Verbos HTTP (GET, POST, PUT/PATCH, DELETE)
- Respuestas en JSON\
- Operaciones CRUD

------------------------------------------------------------------------

### ¿Qué es REST?

REST (*Representational State Transfer*) no es un protocolo, sino un
conjunto de principios:

-   Cada recurso tiene una URL única
-   Se utilizan correctamente los verbos HTTP
-   Las peticiones son *stateless* (sin sesión)
-   Las respuestas representan el estado del recurso

Ejemplo conceptual:
| Operación | Verbo HTTP | Endpoint |
|---|---|---|
| Listar recursos | GET | /api/posts |
| Ver un recurso | GET | /api/posts/1 |
| Crear recurso | POST | /api/posts |
| Actualizar recurso | PUT/PATCH | /api/posts/1 |
| Eliminar recurso | DELETE | /api/posts/1 |


## 11.2 Rutas API en Laravel

Laravel separa claramente las rutas web de las rutas API.

Archivo:

``` bash
routes/api.php
```

Las rutas definidas aquí:

- Devuelven JSON por defecto
- No utilizan sesiones
- Están bajo el prefijo `/api`

Ejemplo:

``` php
<?php
Route::get('/posts', [PostController::class, 'index']);
```

Endpoint real: ```/api/posts```

### Resource Controllers (CRUD automático)

Laravel permite generar controladores REST completos:

``` bash
php artisan make:controller PostController --api
```

Y registrar el CRUD:

``` php
Route::apiResource('posts', PostController::class);
```

- Código limpio
- Convención REST correcta
- Menos errores

## 11.3 Creación de un CRUD REST

Supongamos un recurso **Post**.

### Modelo + Migración

``` bash
php artisan make:model Post -m
```

Migración:

``` php hl_lines="5-6" 
<?php
public function up(): void{

    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('content');
        $table->timestamps();
    });
}
```

``` bash
php artisan migrate
```

### Controlador API

``` bash
php artisan make:controller PostController --api
```

Métodos CRUD típicos:

- index() --> Listar recursos

```php
<?php
public function index()
{
    return Post::all(); // Laravel serializa automáticamente a JSON
}    
```

- store() --> Crear recurso

```php
<?php
public function store(Request $request){

    $validated = $request->validate([
        'title' => 'required',
        'content' => 'required'
    ]);

    return Post::create($validated);
}
```

- show() --> Ver recurso

```php
<?php
public function show(Post $post){

    return $post; // Route Model Binding automático
} 
```

- update() --> Actualizar recurso

```php
<?php
public function update(Request $request, Post $post){

    $validated = $request->validate([
        'title' => 'required',
        'content' => 'required'
    ]);

    $post->update($validated);

    return $post;
}
```
- destroy() --> Eliminar recurso

```php
<?php
public function destroy(Post $post){

    $post->delete();

    return response()->json([
        'message' => 'Post eliminado'
    ]);
}
```

## 11.4 Códigos HTTP

Una API REST correcta no solo devuelve JSON, también debe usar correctamente los códigos HTTP.

| Código | Significado |
|---|---|
| 200 | OK |
| 201 | Created |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 422 | Validation Error |

Ejemplo:

```php
<?php
return response()->json($post, 201);
```

Utilizando los códigos HTTP conseguirás una API profesional que se integrará mejor con los clientes que la utilicen.

## 11.5 Autenticación mediante Tokens (Sanctum)

Las APIs no usan sesiones.

Se autentican mediante:

- Tokens
- Cabeceras HTTP
- Stateless authentication

Laravel utiliza principalmente: **Laravel Sanctum*, que proporciona autenticación ligera para SPAs, aplicaciones móviles y API, lo que permite a los usuarios generar múltiples tokens con capacidades específicas.

### Instalación

``` bash
composer require laravel/sanctum
php artisan sanctum:install
php artisan migrate
```

### Generar Token

``` php
$user->createToken('api-token')->plainTextToken;
```

Respuesta típica:

``` json
{
  "token": "1|asdkjasdkljasdkljasd"
}
```

De esta forma el cliente ya tiene disponible el token para poder realizar peticiones incluyéndolo en la cabecera de las mismas: ```Authorization: Bearer TOKEN```.

### Proteger rutas

Sólo los usuarios autenticados pueden realizar peticiones a la API de posts:

``` php
<?php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('posts', PostController::class);
});
```

## 11.6 Login API

Igual que el login del tema anterior, sólo que aquí se devuelve el token:

```php hl_lines="9 15 17-19"
<?php
public function login(Request $request){

    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required'
    ]);

    if (!Auth::attempt($credentials)) {
        return response()->json([
            'message' => 'Credenciales incorrectas'
        ], 401);
    }

    $user = Auth::user();

    return response()->json([
        'token' => $user->createToken('api-token')->plainTextToken
    ]);
}
```

## 11.7 Autorización en APIs (Policies)

- Autenticado ≠ Autorizado

Un usuario puede estar logueado, pero no tener permiso.

Laravel reutiliza:

- Policies
- Gates
- Middleware `can`

Ejemplo de Policy:

```bash
php artisan make:policy PostPolicy --model=Post
```

``` php
<?php
// PostPolicy.php
public function update(User $user, Post $post){
    return $user->id === $post->user_id;
}
```

Uso en API Controller:

``` php
<?php
public function update(Request $request, Post $post){

    if ($request->user()->cannot('update', $post)) {
        abort(403);
    }

    // ...
}
```

## 11.8 API Resources

No siempre queremos devolver el modelo tal cual. Laravel permite transformar la salida, por ejemplo a JSON.

```bash
php artisan make:resource PostResource
```

``` php
<?php
// PostResource.php
public function toArray(Request $request): array{

    return [
        'id' => $this->id,
        'title' => $this->title,
        'summary' => substr($this->content, 0, 50)
    ];
}
```

Uso:

``` php
<?php
return PostResource::collection(Post::all());
```

## 11.9 Buenas prácticas

- Validar siempre
- Usar HTTP codes correctos
- No exponer datos sensibles
- Versionar API (`/api/v1/...`)
- Manejar errores correctamente
- Transformar respuestas con Resources
- Autenticación stateless
- Autorización mediante policies

## 11.10 Resumen

- CRUD REST\
- Tokens Sanctum\
- Policies\
- Resources

## 11.11 Actividades

### API Rest de productos

Construir una API REST para un recurso **Producto**: *name*, *price* y *stock*.

Ten en cuenta para crear la relación, que un usuario tendrá muchos productos y un producto pertenecerá a un usuario.

Debe incluir:

- CRUD completo
- Validación
- Respuestas HTTP correctas

### Autenticación

A la API anterior añadele **login** para generar el token y protege las rutas con Sanctum.

### Autorización

Crea una **policy** para restringir la actualización y eliminación para que sólo el usuario propietario del producto pueda eliminarlo.

### Cliente API

Crea en **Postman** una colección con los diferentes endpoints para probar la API. No olvides enviar el token cuando sea necesario y verificar los posiles errores.