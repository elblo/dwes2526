# Desarrollo Web en Entorno Servidor

??? info "Apuntes en construcción"
    Estos apuntes se están actualizando por [*Eladio Blanco*](https://x.com/eladioblanco) durante el curso 25/26 tomando como base los elaborados en el curso 21/22 por [*Aitor Medrano*](https://x.com/aitormedrano) y *Luis Alemañ*.

Aquí puedes encontrar los apuntes del módulo de ***Desarrollo web en entorno servidor***, que se imparte en el segundo curso del ciclo formativo de grado superior de *Desarrollo de Aplicaciones Web*.

La duración del módulo es de 168 horas lectivas, a razón de **7 horas semanales**, y se desarrolla a lo largo de los **dos primeros trimestres** del curso. Se ha [planificado](planning.md) basándose en 4 sesiones por semana.

## ¿Qué voy a aprender?

* A desarrollar aplicaciones web dinámicas, que obtienen la información a partir de una base de datos.
* Analizar la estructura de una aplicación cliente/servidor, separando el código de presentación de la lógica de negocio.
* Obtener información a partir de los datos almacenados, así como modificarlos.
* Utilizar frameworks de desarrollo web para agilizar el proceso de desarrollo.
* Crear servicios web para la comunicación entre aplicaciones.

## Resultados de aprendizaje

1. Selecciona las arquitecturas y tecnologías de programación web en entorno servidor, analizando sus capacidades y características propias.
2. Escribe sentencias ejecutables por un servidor web reconociendo y aplicando procedimientos de integración del código en lenguajes de marcas.
3. Escribe bloques de sentencias embebidos en lenguajes de marcas, seleccionando y utilizando las estructuras de programación.
4. Desarrolla aplicaciones web embebidas en lenguajes de marcas analizando e incorporando funcionalidades según especificaciones.
5. Desarrolla aplicaciones web identificando y aplicando mecanismos para separar el código de presentación de la lógica de negocio.
6. Desarrolla aplicaciones web de acceso a almacenes de datos, aplicando medidas para mantener la seguridad y la integridad de la información.
7. Desarrolla servicios web reutilizables y accesibles mediante protocolos web, verificando su funcionamiento.
8. Genera páginas web dinámicas analizando y utilizando tecnologías y frameworks del servidor web que añadan código al lenguaje de marcas.
9. Desarrolla aplicaciones web híbridas seleccionando y utilizando tecnologías, frameworks servidor y repositorios heterogéneos de información.

## Unidades didácticas y temporalización

A continuación se muestran las unidades didácticas y una estimación temporal de cada una de ellas.
La primera evaluación contendría las unidades comprendidas entre la 1 y 5, y parte de la unidad 6. Así pues, desde la mitad de la  unidad 6 a la unidad 9 se verán en la segunda evaluación.

### Primera evaluación

Duración estimada: 76 horas

1. [Arquitecturas Web](01arquitecturas.md) (4h)
    * Cliente/Servidor.
    * MVC.
    * Puesta en marcha con XAMPP y Docker.
2. [El lenguaje PHP](02php.md) (26h)
    * Condiciones y bucles.
    * Arrays.
3. [Orientación a objetos con PHP](03phpoo.md) (18h)
    * Clases y objetos
    * Namespaces
    * Excepciones
4. [Programación Web](04web.md) (12h)
    * Formularios.
    * Cookies.
    * Sesiones.
5. [Herramientas Web](05herramientas.md) (16h)
    * Gestión de librerías con *Composer*.
    * Envío de correos con *Resend*.
    * Gestión de logs con *Monolog*.
    * Documentación con *phpDocumentor*.
    * Web scraping con *Goutte*.
    * Pruebas con *PhpUnit*.
6. [Acceso a datos](06accesoDatos.md) (28h)
    * *SQL*
    * *PDO*
    * Ficheros CSV y PDF.

### Segunda evaluación

Duración estimada: 86 horas

7. [Frameworks PHP - Laravel](07frameworks.md) (28h)
    * Instalación y entornos de desarrollo
    * Rutas
    * Vistas y plantillas Blade
    * Controladores
8. [Gestion de datos en Laravel](08laravelDatos.md) (24h)
    * Migraciones e integración con el ORM *Eloquent*.
    * Query Builder
    * Modelos
    * Formulario sy validación
9. [Uso avanzado de Laravel](09laravelAvanzado.md) (22h)
    * Almacenamiento de ficheros (local y AWS S3)
    * Request y responses
    * Relaciones entre modelos
    * Seeders y factorías
    * Mutadores y accesores
10. [Autenticación y autorización](10laravelAutenticacion.md) (12h)
    * Autenticación
    * Autorización
    * Kits de inicio
    * Laravel Cloud
11. [Servicios REST](10laravelRest.md) (12h)
    * Producción y consumo.
    * *AJAX* con *JSON*.

## Evaluación

La nota de cada evaluación y también final se calcula mediante una suma ponderada de la nota de cada *Criterio de Evaluación* evaluado hasta el momento.

A cada criterio se le asigna un porcentaje del total de la nota final y va asociado a un tipo de actividad que puede ser:

- Examen Teoría
- Examen Prácticas
- Ejercicios de clase
- Prácticas obligatorias

### Recursos

!!! abstract "Recursos del módulo"

    === "Documentación"
        * Tutoriales
            * [Programación web en PHP - mclibre.org](https://www.mclibre.org/consultar/php/index.html)
            * [Tutorial para principiantes](https://www.ionos.es/digitalguide/paginas-web/creacion-de-paginas-web/tutorial-de-php-fundamentos-basicos-para-principiantes/)

        * Referencias
            * [Documentación oficial de PHP](https://www.php.net/manual/es/)
            * [Documentación oficial de Laravel](https://laravel.com/docs/11.x)
            * [LaravelDocs - Documentación no oficial de Laravel en español](https://laravel-docs.com/es/docs/10.x)

        * Recopilatorio
            * [Awesome PHP](https://github.com/ziadoz/awesome-php)
    
    === "Software"
        * IDEs
            * [Visual Studio Code](https://code.visualstudio.com/)
            * [PHPStorm](https://www.jetbrains.com/phpstorm/)
        * Administración de bases de datos
            * [DBeaver](https://dbeaver.io/)
        * Entorno de desarrollo
            * [Laragon](https://laragon.org/)
            * [XAMPP](https://www.apachefriends.org/es/index.html)

        * Entorno de desarrollo en contenedores
            * [Docker: Devilbox](https://devilbox.io/)
            * [Docker: Laradock](https://laradock.io/)
    
    === "Cursos PHP"
        * OpenWebinars:
            * [PHP: fundamentos](https://openwebinars.net/academia/aprende/curso-php-basico)
            * [PHP: Ampliando conceptos](https://openwebinars.net/academia/aprende/php-ampliar-conceptos)
            * [PHP y MySQL: Creando sitios dinámicos](https://openwebinars.net/academia/aprende/php-mysql)

    === "Cursos Laravel"
        * Laracasts (oficiales):
            * [30 días para aprender Laravel (inglés)](https://laracasts.com/series/30-days-to-learn-laravel-11)
        * El rincón de Isma (Youtube):
            * [Curso completo de Laravel 11](https://www.youtube.com/watch?v=Oa_pHsymaiA&list=PLbFjjy1sD3hr3ppWz9JndcXJErAQdpDHt)
        * OpenWebinars:
            * [Desarrollo Web Moderno con Laravel: De la teoría a la práctica](https://openwebinars.net/academia/portada/desarrollo-web-moderno-laravel-teoria-practica/)
