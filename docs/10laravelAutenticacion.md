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

# Seguridad en Laravel

> Duración estimada: 4 sesiones

## 10.1 Autenticación

### Introducción

La mayoría de aplicaciones web permiten que sus usuarios se autentiquen mediante un sistema de "Login" o "Inicio de sesión". Implementar esta función de forma manual puede ser complejo y arriesgado. Por eso, Laravel se esfuerza por proporcionar herramientas para implementar la autenticación de manera rápida, segura y fácil.

En su núcleo, las instalaciones de autenticación de Laravel se componen de "guards" (guardias) y "providers" (proveedores). Los guardias definen cómo se autentican los usuarios para cada solicitud. Por ejemplo, el guardia session mantiene el estado utilizando almacenamiento de sesión y cookies.

El archivo de configuración de autenticación de tu aplicación se encuentra en `config/auth.php`. Contiene varias opciones documentadas para ajustar el comportamiento de los servicios de autenticación de Laravel.

### Kits de inicio

Laravel proporciona unos kits de inicio que estructuran automáticamente la aplicación con las rutas, controladores y vistas necesarios para registrar y autenticar a los usuarios de la aplicación. Los más utilizados son **Breeze** y **JetStream**.

**Laravel Breeze** es una implementación simple y mínima de todas las funciones de autenticación de Laravel, que incluye inicio de sesión, registro, restablecimiento de contraseña, verificación de correo electrónico y confirmación de contraseña. La capa de vista de Laravel Breeze está compuesta por plantillas Blade con estilos de Tailwind CSS. 

??? tip "Instalación de Laravel Breeze"
    
    1. Instalar Breeze: Navega a la carpeta del proyecto у ejecuta:
   
    ```bash
    composer require laravel/breeze --dev
    ```

    2. Instalar el Kit: Ejecuta el comando para publicar los archivos de autenticación:
   
    ```bash
    php artisan breeze:install
    ```

    3. Migrar y Compilar: Prepara la base de datos y los recursos frontend:

    ```bash
    php artisan migrate
    npm install
    npm run dev
    ```

**Laravel Jetstream** es un sólido kit de inicio de aplicaciones que consume y expone los servicios del backend de autenticación Laravel Fortify con una interfaz de usuario basada en Tailwind CSS, Livewire y/o Inertia. Laravel Jetstream incluye soporte opcional para la autenticación de dos factores, soporte para equipos, gestión de sesiones de navegador, gestión de perfiles e integración incorporada con Laravel Sanctum para ofrecer autenticación de token de API. 

Puedes probar a crear un proyecto y utilizar uno de estos kits, sobre todo Breeze, para navegar por sus archivos y aprender cómo funciona un sistema de autenticación.

### Sistema autenticación manual

Mejor incluso que empezar utilizando un kit de inicio, es **crear un sistema de autenticación de forma manual**. Así, aprenderás paso a paso cómo funciona la autenticación. Y es lo que vamos a hacer en este punto. Una vez que entiendas todos los puntos, para futuras aplicaciones será más práctico empezar el desarrollo con un kit de inicio.

Vamos a necesitar unas vistas con los formularios de login y registro, unas rutas y controladores con sus funciones asociadas.

#### 1. Rutas para login, registro y logout

En `web.php` agregamos las rutas necesarias para:

- Mostrar el formulario de registro y procesar sus datos.
- Mostrar el formulario de login y procesar sus datos.
- Cerrar sesión.

```php
<?php
// Rutas auth
Route::get('/register', [RegisterController::class, 'create']);  //Formulario de registro
Route::post('/register', [RegisterController::class, 'store']);  //Registrar usuario

Route::get('/login', [SessionController::class, 'create']);  //Formulario de login
Route::post('/login', [SessionController::class, 'store'])->name('login');  //Iniciar sesión

Route::post('/logout', [SessionController::class, 'destroy']);  //Cerrar sesión 
```

#### 2. Controladores necesarios

Creamos 2 controladores:

- `RegisterController`, encargado del registro de usuarios.
- `SessionController`, encargado del inicio y cierre de sesión.

```bash
php artisan make:controller RegisterController
php artisan make:controller SessionController
```

Empezamos con `RegisterController`, que va a tener 2 funciones:

- `create`: simplemente muestra la vista del formulario de registro.
- `store`: recibe los datos del formulario de registro y si son válidos, crea el usuario en la BDD, inicia su sesión y redirige a la página de inicio.

```php
<?php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\Rules\Password;

class RegisterController extends Controller
{
    // Muestra la vista del formulario de registro 
    public function create(){ return view('auth.register'); }

    // Procesa los datos del formulario de registro
    public function store(Request $request){
        // Validación
        $validatedAttributes = $request->validate([
            'name' => 'required',
            'email' => 'required|email',
            'password' => ['required', Password::min(8)->mixedCase()->numbers()->symbols(), 'confirmed:password_confirmation'],
        ]);

        // Creación del usuario
        $user = User::create($validatedAttributes); 

        // Inicio de sesión con la instancia del usuario (login)
        Auth::login($user);

        // Redirección a página de inicio
        return redirect('/');
    }
}
```

Es el turno de `SessionController`, que va a tener 3 funciones:

- `create`: simplemente muestra la vista del formulario de login.
- `store`: recibe los datos del formulario de login y si son válidos, intenta iniciar sesión con ellos y redirige a la página de inicio.
- `destroy`: cierra la sesión, invalidándola y redirigiendo a la página de inicio.

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\ValidationException;

class SessionController extends Controller
{
    // Muestra la vista del formulario de login 
    public function create(){
        return view('auth.login');
    }

    // Procesa los datos del formulario de login
    public function store(Request $request){ 
       //Validar los datos email y password
       $request->validate([
            'email' => 'required|email',
            'password' => 'required',
       ]);

       $attributes = $request->only(['email', 'password']);

       // Intentar iniciar sesión con esos datos (email y password)
       if (!Auth::attempt($attributes)) {
            throw ValidationException::withMessages([
                'email' => 'Esas credenciales no son correctas.'
            ]);
        }

       // Registrar sesión regenerando su id 
       $request->session()->regenerate();

       // Redirección a la página de inicio
       return redirect('/');
    }

    // Cierra sesión
    public function destroy(Request $request){
        // Cerrar sesión
        Auth::logout();

        // Invalidar la sesión
        $request->session()->invalidate();

        // Redirección a la página de inicio
        return redirect('/');
    }
}
```

#### 3. Vistas login y register

Vamos a crear las vistas para los formularios de inicio de sesión y registro dentro de una carpeta común `auth` para su mejor organización.

Vista para el fomulario de registro en `/resources/views/auth/register.blade.php`:

```html
@extends('layout.app')
@section('content')
<div class="flex min-h-full items-center justify-center py-12 px-4 sm:px-6 lg:px-8">
    <div class="w-full max-w-md space-y-8">
      <div>
        <h2 class="mt-6 text-center text-3xl font-bold tracking-tight text-gray-900">Registro</h2>
      </div>
      <form method="POST" action="/register">
        @csrf

        <div class="space-y-12">
          <div class="border-b border-gray-900/10 pb-12">
            <div class="flex flex-col gap-3">

                <div class="mb-4">
                    <label for="name" class="block text-sm font-medium text-gray-700">Nombre</label>
                    <div class="mt-2">
                        <input type="text" name="name" id="name" value="{{ old('name')}}" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('name')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>

                <div class="mb-4">
                    <label for="email" class="block text-sm font-medium text-gray-700">Email</label>
                    <div class="mt-2">
                        <input type="email" name="email" id="email" value="{{ old('email')}}" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('email')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>

                <div class="mb-4">
                    <label for="password" class="block text-sm font-medium text-gray-700">Password</label>
                    <div class="mt-2">
                        <input type="password" name="password" id="password" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('password')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>

                <div class="mb-4">
                    <label for="password_confirmation" class="block text-sm font-medium text-gray-700">Repite el Password</label>
                    <div class="mt-2">
                        <input type="password" name="password_confirmation" id="password_confirmation" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('password_confirmation')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>


            </div>
          </div>
        </div>

        <div class="mt-6 flex items-center justify-end gap-x-6">
          <a href="/" class="text-sm font-semibold leading-6 text-gray-900">Cancelar</a>
          <input type="submit" value="Registro" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" />
        </div>
      </form>
    </div>
  </div>
@endsection
```

Vista para el fomulario de login en `/resources/views/auth/login.blade.php`:

```html
@extends('layout.app')
@section('content')
<div class="flex min-h-full items-center justify-center py-12 px-4 sm:px-6 lg:px-8">
    <div class="w-full max-w-md space-y-8">

      <div>
        <h2 class="mt-6 text-center text-3xl font-bold tracking-tight text-gray-900">Log In</h2>
      </div>

      <form class="flex-[0.5]" method="POST" action="/login">
        @csrf

        <div class="space-y-12">
          <div class="border-b border-gray-900/10 pb-12">
            <div class="flex flex-col gap-3">

                <div class="mb-4">
                    <label for="email" class="block text-sm font-medium text-gray-700">Email</label>
                    <div class="mt-2">
                        <input type="email" name="email" id="email" value="{{ old('email') }}" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('email')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>

                <div class="mb-4">
                    <label for="password" class="block text-sm font-medium text-gray-700">Password</label>
                    <div class="mt-2">
                        <input type="password" name="password" id="password" required 
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        @error('password')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                </div>

            </div>
          </div>
        </div>

        <div class="mt-6 flex items-center justify-end gap-x-6">
          <a href="/" class="text-sm font-semibold leading-6 text-gray-900">Cancelar</a>
          <input type="submit" value="Log In" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" />
        </div>
      </form>
    </div>
  </div>
@endsection
```

#### 4. Vistas de plantilla base y menu

Crear la vista con la plantilla base en `/resources/views/layout/app.blade.php`:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <title>Laravel</title>

        <!-- Fonts -->
        <link rel="preconnect" href="https://fonts.bunny.net">
        <link href="https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap" rel="stylesheet" />

        <!-- Styles / Scripts -->
        @vite(['resources/css/app.css', 'resources/js/app.js'])

    </head>
    <body class="flex flex-col min-h-screen">
       <header class="bg-blue-800 text-white">
            @include('partials.menu')
       </header>
       <main class="flex-grow">
            @yield('content')
       </main>
       <footer class="pl-10 text-center bg-blue-800 text-white py-4">
            Aplicación desarrollada por 🦫 en 2026
       </footer>
    </body>
</html>
```

Con la directiva **@auth** podemos mostrar contenido si el usuario está logueado y con **@guest** si no lo está. Realmente son un if encubierto, donde simplemente se comprueba si existe un usuairo logueado o no. En la siguiente vista los utilizamos para mostrar unos menús u otros según el estado de la sesión.

Crear la vista con el menú con la navegación y opciones en `/resources/views/partials/menu.blade.php`:

```html
<div class="flex h-16 items-center justify-between">
    <div class="flex items-center">
        <div class="ml-10 flex items-baseline space-x-4">

            @php
                $currentRoute = request()->path();
            @endphp

            <nav class="flex space-x-4">
                <a href="/" class="{{ $currentRoute === '/' ? 'text-orange-300 font-semibold' : ' hover:text-orange-300' }}">
                    Inicio
                </a>
                <a href="/usuarios" class="{{ $currentRoute === 'usuarios' ? 'text-orange-300 font-semibold' : 'hover:text-orange-300' }}">
                    Usuarios
                </a>
                <!--
                    ...
                -->
            </nav>

        </div>
    </div>

    <!-- Enlaces para login/register -->
    <div class="ml-4 mr-10 flex items-center md:ml-6">
        <!-- Visible para invitados -->
        @guest
            <nav class="flex space-x-4">
                <a href="/login" class="{{ $currentRoute === 'login' ? 'text-orange-300 font-semibold' : ' hover:text-orange-300' }}">
                    Log In
                </a>
                <a href="/register" class="{{ $currentRoute === 'register' ? 'text-orange-300 font-semibold' : ' hover:text-orange-300' }}">
                    Registro
                </a>
            </nav>
        @endguest

        <!-- Visible para usuarios autenticados -->
        @auth
            <form method="POST" action="/logout">
                @csrf
                <button class-name="text-gray-300 hover:text-white bg-transparent hover:bg-transparent">Cerrar sesión</button>
            </form>
        @endauth
        </div>
  </div>
```

#### 5. Personalización de la página de inicio

Vamos a personalizar la página de inicio mostrando un mensaje de bienvenida si el usuario está logueado o un mensaje genérico si no lo está. Para ello vamos a utilizar el helper **auth()** para acceder a los datos del usuario logueado.

En primer lugar, creamos la ruta en `web.php`:

```php
<?php
Route::get('/', function () { return view('sections.home'); });
```

Y ahora la vista en `/resources/views/sections/home.blade.php`:

```html
@extends('layout.app')
@section('content')
<div class="flex min-h-full items-center justify-center py-12 px-4 sm:px-6 lg:px-8">
    <div class="w-full max-w-md space-y-8">

      <div>
        <h2 class="mt-6 text-center text-3xl font-bold tracking-tight text-gray-900">Home</h2>
      </div>

        @auth
            <p class="text-center text-xl text-green-600">Bienvenido {{ auth()->user()->name }}!</p>
        @endauth

        @guest
            <p class="text-center text-xl text-orange-400">Hola invitado/a.</p>
            <p>Inicia sesión desde le menú superior o regístrate si todavía no tienes cuenta.</p>
        @endguest
    </div>
</div>
@endsection
```

#### 6. Proteger rutas

Mediante el middleware `auth` de Laravel, podemos proteger rutas para que solo los usuarios autenticados puedan acceder a ellas:

```php
<?php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth');
```

En el siguiente punto se amplía el uso de middlewares.

### Resumen

- Mediante `auth()` se acceden a los datos del usuario. Ej: `auth()->user()->name;`
- Con `Auth::login($usuario)` se inicia sesión con una instancia de un usuario.
- Con `Auth::attempt(['email', 'password'])` se intenta iniciar sesión con email y password devolviendo - un boolean si ha tenido o no éxito.
- Con `Auth::logout()` se cierra la sesión.
- Se puede utilizar indistintamente el helper global `request()` o `$request` para trabajar con las peticiones. Si se hace con el segundo, hay que pasarlo explícitamente como parámetro `Request $request` en la función del controlador que se trate. Mejor práctica esta última porque mejora la legibilidad, el testing y mantenimiento.
- Con las directivas `@auth` y `@guest` en plantillas se puede mostrar contenido visible para usuarios autenticados o para invitados respectivamente.
- Mediante el middleware `auth` hacemos que ciertas rutas sólo sean accesibles a usuarios autenticados.
- Utilizar `request()->session()->invalidate()` al cerrar la sesión o `request()->session()->regenerate()` en el login, son buenas prácticas que mejoran la seguridad.

## 10.2 Autorización

### Introducción

Laravel ofrece un sistema de autorización flexible y potente basado en policies y gates, permitiendo restringir el acceso a diferentes partes de la aplicación según los permisos del usuario.

### Gates

Los Gates son funciones de cierre (closures) que determinan si un usuario autenticado está autorizado para realizar una acción específica.

#### Definir un Gate

Los Gates se definen en `app/Providers/AppServiceProvider.php` dentro del método `boot()`. Los Gates siempre reciben la instancia del usuario como primer argumento y opcionalmente argumentos adicionales como otros modelos Eloquent que sean necesarios:

```php
<?php
use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Gate;
use App\Models\User;

class AppServiceProvider extends ServiceProvider
{
    // ...

    public function boot()
    {
        Gate::define('ver-admin', function (User $user) {
            return $user->role === 'admin';
        });

        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id;
        });

        // Como en las rutas, los Gates también se pueden definir utilizando un array con la clase y método
        Gate::define('update-post', [PostPolicy::class, 'update']);
    }
}
```

#### Usar un Gate

Para verificar el permiso en un controlador:

```php
<?php
if (Gate::allows('ver-admin')) {
    // El usuario tiene autorización
}

if (Gate::denies('update-post', $post)) {
    // El usuario no tiene autorización
}
```

En una vista Blade:

```html
@can('ver-admin')
    <p>Eres administrador.</p>
@endcan
```

??? info "Métodos de Gates"
    Existen más métodos en las Gates. Anímate a investigarlos en la [documentación oficial](https://laravel.com/docs/12.x/authorization). Tanto los métodos de Gate para autorizar acciones como `allows`, `denies`, `check`, `any`, `none`, `authorize`, `can`, `cannot` y las directivas de autorización Blade `@can`, `@cannot`, `@canany` pueden recibir un array como segundo argumento, cuyos elementos se pasan como parámetros a la función closure de la Gate y se pueden uilizar utilizar para dar contexto adicional.

    ```php
    <?php
    // En AppServiceProvider -> boot()
    Gate::define('create-post', function (User $user, Category $category, bool $pinned) {
        if (! $user->canPublishToGroup($category->group)) {
            return false;
        } elseif ($pinned && ! $user->canPinPosts()) {
            return false;
        }
    
        return true;
    });
    
    // En un controlador donde sea necesario comprobar si el usuario tiene la autorización
    if (Gate::check('create-post', [$category, $pinned])) {
        // The user can create the post...
    }
    ```

### Policies

Las Policies son clases específicas para manejar la autorización del usuario autenticado sobre modelos.

#### Crear una Policy

Ejecutar el comando:

```bash
php artisan make:policy PostPolicy
```

Opcionalmente puedes indicarle el flag `--model=Modelo` para generar la policy con métodos de ejemplo sobre el modelo propuesto. 

Esto crea `app/Policies/PostPolicy.php`, donde se definen los métodos de autorización:

```php
<?php
use App\Models\User;
use App\Models\Post;

class PostPolicy
{
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

#### Registrar una Policy

Por defecto Laravel descubre y asocia las policies si están correctamente nombradas y ubicadas, por ejemplo *PostPolicy* en `app/Policies` para una clase *Post* en `app/Models`. Si no fuera el caso, es posible registrar la asociación entre modelo y policy de forma manual en el método `boot()` de `AppServiceProvider.php`:

```php
<?php
use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

// ...

public function boot(): void
{
    // Clase Post con policy PostPolicy (esto, Laravel lo saca solo, no sería necesario)
    Gate::policy(Post::class, PostPolicy::class);
}
```

#### Usar una Policy

En un controlador:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    // Actualizar el post pasado como argumento
    public function update(Request $request, Post $post): RedirectResponse
    {
        if ($request->user()->cannot('update', $post)) {
            abort(403);
        }      

        // Aquí el código para actualizar el post...

        return redirect('/posts');
    }
}
```

En una vista Blade:

```html
@can('update', $post)
    <button>Editar</button>
@endcan
```

En algunos casos es necesario pasar la propia definición de la clase en lugar de una instancia de la misma, como hacíamos con `post` en el ejemplo anterior.

En un controlador:

```php
<?php
public function usuarios(Request $request){
    // Las 3 opciones siguientes son válidas
    // if ($request->user()->cannot('show', $request->user())) { 
    // if ($request->user()->cannot('show', Auth::user())) { 
    if ($request->user()->cannot('show', User::class)) { 
        abort(403);
    }

    return view('sections.usuarios');
}
```

En una vista Blade:

```php
// Las 3 opciones siguientes son válidas
// @can('show', auth()->user())
// @can('show', Auth::user())
@can('show', 'App\Models\User')
    <p>Tienes autorización para ver este contenido.</p>
@endcan
```

??? tip "Gates vs policies"
    
  Ambas permiten gestionar la autorización del usuario autenticado en la aplicación, pero **¿cuándo utilizar unas u otras?**

  **Policies** → autorización ligada a MODELOS (recursos concretos)
  **Gates** → autorización ligada a ACCIONES (reglas generales)

  Si la decisión depende de una entidad concreta, **policy**. 
  Ejemplos:

  - ¿Puede editar este Post?
  - ¿Puede borrar este Pedido?
  - ¿Puede ver este Usuario?

  Si la decisión es global o conceptual, **gate**. 
  Ejemplos:

  - ¿Es administrador?
  - ¿Puede acceder al panel?
  - ¿Puede exportar informes?
  - ¿Puede ver estadísticas?

### Middlewares

Los middlewares permiten restringir el acceso a rutas en función de la autenticación y permisos del usuario.

#### Middleware `auth`

Para proteger rutas y asegurarse de que solo los usuarios autenticados puedan acceder:

```php
<?php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth');
```

#### Middleware `can`

Para aplicar autorización específica en rutas mediante gates o policies definidas:

```php
<?php
Route::get('/admin', [AdminController::class, 'view'])->middleware('can:ver-admin');

// OJO! Para que coja 'post' como objeto del tipo Post y no como id (string), en la función edit hay 
// que pasar el parámetro como Post y así Laravel recuperará automáticamente el Post con ese id
Route::get('/post/{post}/edit', [PostController::class, 'edit'])->middleware('can:update, post');

Route::get('/usuarios', [PagesController::class, 'usuarios'])->middleware('can:show, App\Models\User'); 

```

#### Middleware personalizado

Se puede crear un middleware propio con Artisan:

```bash
php artisan make:middleware CheckRole
```

Esto genera `app/Http/Middleware/CheckRole.php`, donde se puede definir la lógica de autorización. Los middlewares tienen un único método llamado `handle`, que recibe una petición y una función para llamar como continuación si pasa la verificación que especifiquemos. Y si la verificación falla, podemos devolver una respuesta o redirigir a otra ruta.

```php
<?php
use Closure;
use Illuminate\Http\Request;

class CheckRole
{
    public function handle(Request $request, Closure $next, $role)
    {
        if($request->user()->role !== $role) {
            abort(403);
        }
        return $next($request);
    }
}
```

Y se usa en rutas:

```php
<?php
Route::get('/admin', [AdminController::class, 'view'])->middleware(CheckRole::class.":admin");
```

<!-- ## 10.3 Laravel Cloud
Crear cuenta Sandbox
Crear entorno con repo de GitHub y asociar BDD
Desplegar
Dominios personalizados
Opciones: Hibernar, despliegues automáticos al hacer commit en el repo
...

## 10.4 Kits de inicio
React mediante Innertia.js -->

## 10.3 Resumen

En este tema sobre **Seguridad en Laravel**, se aborda la implementación de sistemas de acceso y permisos.

### I. Introducción y Conceptos Clave (10.1)

*   **Fundamentos:** Laravel proporciona herramientas para implementar la autenticación de forma rápida, segura y fácil. En su núcleo, este sistema utiliza **"guards"** (definen cómo se autentican los usuarios en cada solicitud, como las sesiones y cookies) y **"providers"**. Toda la configuración se encuentra en `config/auth.php`.
*   **Kits de Inicio:** Son herramientas que estructuran automáticamente rutas, controladores y vistas de autenticación. 
    *   **Laravel Breeze:** Implementación simple y mínima con plantillas Blade y Tailwind CSS.
    *   **Laravel Jetstream:** Kit robusto que incluye soporte para autenticación de dos factores, gestión de equipos y perfiles, e integración con API mediante Sanctum.

### II. Sistema de Autenticación Manual (10.1)

Crear el sistema manualmente es la mejor forma de aprender cómo funciona paso a paso. Los componentes esenciales son:

1.  **Rutas:** Se deben definir rutas para mostrar y procesar el registro (`/register`), el inicio de sesión (`/login`) y el cierre de sesión (`/logout`).
2.  **Controladores:** 
    *   **RegisterController:** Encargado de validar los datos, crear el usuario en la base de datos e iniciar su sesión automáticamente con `Auth::login($user)`.
    *   **SessionController:** Gestiona el login mediante `Auth::attempt($credentials)` y el cierre de sesión con `Auth::logout()`.
3.  **Seguridad de Sesión:** Es una buena práctica usar `session()->regenerate()` al iniciar sesión para evitar ataques de fijación de sesión y `session()->invalidate()` al cerrarla.
4.  **Vistas:** Se utilizan las directivas **`@auth`** (contenido para usuarios logueados) y **`@guest`** (contenido para invitados) para condicionar la interfaz. Los datos del usuario actual se obtienen mediante el helper **`auth()`**.

### III. Autorización: Gates y Policies (10.2)

Mientras que la autenticación identifica al usuario, la **autorización** determina si tiene permiso para realizar una acción.

*   **Gates:** Son funciones de cierre (closures) ideales para acciones que no están ligadas a un modelo específico. Se definen en `AppServiceProvider.php` y se comprueban con `Gate::allows()`, `Gate::denies()` o la directiva de Blade `@can`.
*   **Policies:** Son clases dedicadas a organizar la lógica de autorización en torno a un modelo de Eloquent (ej. `PostPolicy` para el modelo `Post`). Se crean con `php artisan make:policy` y permiten definir métodos como `update` o `delete` para verificar la propiedad o permisos sobre un recurso.

### IV. Middlewares de Seguridad (10.2)

Los middlewares restringen el acceso a las rutas antes de que lleguen al controlador.

*   **Middleware `auth`:** Asegura que solo usuarios autenticados accedan a una ruta.
*   **Middleware `can`:** Aplica autorizaciones de Gates o Policies directamente en la definición de la ruta (ej. `middleware('can:update, post')`).
*   **Middlewares Personalizados:** Permiten definir lógica propia, como verificar roles de usuario (`admin`, `user`), mediante el método `handle` de la clase generada.

***

**Resumen de funciones:**

*   `Auth::attempt()`: Intenta iniciar sesión y devuelve un booleano.
*   `Auth::login()`: Inicia sesión con una instancia de usuario.
*   `Auth::logout()`: Cierra la sesión activa.
*   `auth()->user()`: Accede al objeto del usuario autenticado.

## 10.4 Actividades

A continuación se presentan 2 prácticas sobre autorización y autenticación. Escoge una de las 2 e impleméntala.

### Gestión de usuarios

Aplicación que permita gestionar usuarios con autenticación y autorización, implementando roles de usuario y permisos específicos.

- Implementar autenticación manual con Laravel.
- Crear dos roles: admin y user.
- Los administradores pueden gestionar (crear, editar y eliminar) usuarios.
- Los usuarios pueden actualizar su propio perfil pero no el de otros.
- Restringir acceso a las vistas según el rol del usuario.

### Mini Instagram

Aplicación que permita a los usuarios subir fotos, ver las de otros, pero modificar únicamente las suyas.

- Implementar autenticación manual con Laravel.
- Cada usuario podrá subir, editar y eliminar únicamente sus propias fotografías.
- Todos los usuarios pueden ver las fotografías de los demás.
- Implementar middlewares y policies para restringir la modificación y eliminación.










