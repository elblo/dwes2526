# Introducción a la Programación

??? abstract "Duración y criterios de evaluación"

```
Duración estimada: 6 sesiones

<hr />

Resultado de aprendizaje:

1. Comprende los conceptos fundamentales de la programación, sus paradigmas y fases de desarrollo.
2. Conoce los distintos tipos de lenguajes de programación y su evolución histórica.
3. Comprende el proceso de compilación y ejecución de programas en Java.
4. Utiliza un entorno de desarrollo integrado para elaborar programas básicos.

Criterios de evaluación:

1. Se han identificado las acciones cotidianas que dependen de la programación.
2. Se han descrito los conceptos de programa, programación y algoritmo.
3. Se han caracterizado los principales paradigmas de programación y sus diferencias.
4. Se han descrito las fases del ciclo de vida de un programa con ejemplos prácticos.
5. Se han diferenciado los tipos de lenguajes de programación y sus ventajas e inconvenientes.
6. Se ha explicado el funcionamiento de Java y su máquina virtual.
7. Se han creado programas en Java utilizando un IDE de desarrollo.
```

---

## 1.1 Introducción

Vivimos rodeados de tecnología. Muchas de las acciones que realizamos a diario son posibles gracias a **programas informáticos**:

* La alarma del móvil que te despierta.
* El microondas que calienta tu desayuno.
* El ascensor que utilizas para salir de casa.
* Las noticias que consultas en un dispositivo digital.
* Los videojuegos, las aplicaciones de mensajería o incluso los cajeros automáticos.

Detrás de todo ello hay personas que diseñan, programan y mantienen el software. La programación es, por tanto, una habilidad fundamental en la sociedad actual.

!!! tip "Reflexiona"
Intenta pensar cuántas veces al día interactúas con un programa. Te sorprenderá ver hasta qué punto dependemos de ellos.

---

## 1.2 Programas y Programación

Un **programa** es un conjunto de instrucciones que indican a un ordenador cómo realizar una tarea. La **programación** es el proceso de diseñar y escribir dichos programas.

<figure>
  <img src="img/progt01-01.png" />
  <figcaption>Resolución de problemas mediante programación</figcaption>
</figure>

### Conceptos clave en la resolución de problemas

* **Abstracción**: centrarse en lo esencial, ignorando los detalles irrelevantes.
* **Divide y vencerás**: dividir un problema complejo en problemas más pequeños y manejables.
* **Encapsulación**: agrupar datos y procedimientos relacionados, de forma que se oculten los detalles internos.
* **Modularidad**: organizar el código en módulos reutilizables y fáciles de mantener.

### Algoritmo y programa

* **Algoritmo**: secuencia ordenada y no ambigua de pasos que llevan a la solución de un problema.
* **Programa**: implementación de un algoritmo en un lenguaje de programación concreto.

### Representación de algoritmos

Existen distintas técnicas para plasmar un algoritmo antes de programarlo:

* [Diagramas de flujo](https://www.lucidchart.com/pages/es/que-es-un-diagrama-de-flujo): representación gráfica de los pasos.
* [Pseudocódigo](https://es.wikipedia.org/wiki/Pseudoc%C3%B3digo): texto estructurado con instrucciones similares a un lenguaje de programación.
* [Tablas de decisión](https://eve-ingsistemas-u.blogspot.com/2012/05/tablas-de-decision-parte-1.html): representación tabular de condiciones y acciones.

!!! example "Ejemplo de algoritmo cotidiano"
Piensa en la receta de cocinar pasta:
- Poner agua a hervir.
- Añadir sal y pasta.
- Esperar 10 minutos.
- Escurrir y servir.

```
Este conjunto de pasos claros y ordenados es un **algoritmo**.
```

---

## 1.3 Paradigmas de Programación

Los **paradigmas** son formas de clasificar los lenguajes de programación según sus características.

### Clasificación general

* **Programación imperativa**: describe paso a paso cómo resolver un problema (*cómo* hacerlo). Ejemplo: C.
* **Programación declarativa**: describe el resultado que se quiere obtener (*qué* se desea). Ejemplo: SQL.

<figure>
  <img src="img/progt01-02.png" height="400" />
  <figcaption>Paradigmas de programación</figcaption>
</figure>

### Subtipos de paradigmas

* **Estructurada**: basada en secuencias, decisiones y bucles.
* **Orientada a objetos**: basada en clases, objetos, herencia y polimorfismo.
* **Funcional**: utiliza funciones matemáticas puras y evita estados cambiantes.
* **Lógica**: se centra en relaciones y reglas (ejemplo: Prolog).

### Lenguajes multiparadigma

Hoy en día, la mayoría de los lenguajes son multiparadigma. Ejemplo: **Java**, que es estructurado, orientado a objetos y funcional.

---

## 1.4 Fases de la Programación

El desarrollo de un programa no consiste solo en escribir código. Se siguen distintas fases:

1. **Resolución del problema**

   * **Análisis**: identificar los requisitos del cliente.
   * **Diseño**: definir cómo se resolverá el problema.
2. **Implementación**

   * **Codificación**: escribir el código en un lenguaje.
   * **Pruebas y validación**: comprobar que funciona correctamente.
3. **Explotación y mantenimiento**

   * Uso en producción.
   * Correcciones y mejoras.

### Ejemplo: programa "pares/impares"

**Análisis**

* El programa debe pedir un número entre 1 y 100.
* Si el número es 0 → error.
* Si está fuera del rango → error.
* Indicar si es par o impar.

**Diseño**

* Puede representarse mediante diagrama de flujo o pseudocódigo.

<figure>
  <img src="img/progt01-06.png" height="400" />
  <figcaption>Ejemplo de diagrama de flujo</figcaption>
</figure>

<figure>
  <img src="img/progt01-07.png" height="400" />
  <figcaption>Ejemplo de pseudocódigo</figcaption>
</figure>

**Codificación (Java)**

```java
import java.util.Scanner;

public class Main {
  public static void main(String[] args) {
    int modulo, numero;
    var scanner = new Scanner(System.in);
    System.out.println("Introduce un número");
    numero = Integer.parseInt(scanner.nextLine());

    if (numero == 0) {
      System.out.println("Valor incorrecto. El 0 no es válido");
    } else if (numero < 0 || numero > 100) {
      System.out.println("Número no válido, el rango es 1-100");
    } else {
      modulo = numero % 2;
      if (modulo == 0) {
        System.out.println("El número es par");
      } else {
        System.out.println("El número es impar");
      }
    }
  }
}
```

**Pruebas y validación**

* Probar con números válidos e inválidos.
* Documentar cómo usar el programa.

**Explotación y mantenimiento**

* Cuando el software se usa en la práctica, se corrigen errores y se actualiza.

---

## 1.5 Lenguajes de Programación

Los lenguajes han evolucionado mucho desde los inicios de la informática.

### 1. Lenguaje máquina

* Es el nivel más bajo.
* Se escribe directamente en código binario.
* Muy difícil de aprender.

### 2. Lenguaje ensamblador

* Usa instrucciones simbólicas (mnemónicos).
* Depende de la arquitectura del PC.
* Requiere gran conocimiento del hardware.

!!! info "Ejemplo en vídeo"
- [Lenguaje ensamblador (YouTube 1)](https://www.youtube.com/embed/GmtenWqfIaI)
- [Lenguaje ensamblador (YouTube 2)](https://www.youtube.com/embed/wQf0u8cTAcg)

### 3. Lenguajes compilados

* Traducidos a lenguaje máquina por un compilador.
* Rápidos en ejecución.
* Ejemplos: C, C++, Pascal, C#.

### 4. Lenguajes interpretados

* Cada instrucción se traduce y ejecuta en el momento.
* Más lentos, pero flexibles.
* Ejemplos: Python, PHP, JavaScript.

```python
numero1 = int(input("Ingresa un número: "))
numero2 = int(input("Ingresa otro número: "))
operacion = input("suma, resta, división, multiplicación: ")

if operacion == "suma":
    print(numero1 + numero2)
elif operacion == "resta":
    print(numero1 - numero2)
elif operacion == "división":
    print(numero1 / numero2)
elif operacion == "multiplicación":
    print(numero1 * numero2)
```

### Caso particular: Java

* Compilado a **bytecode**.
* Ejecutado por la **Máquina Virtual de Java (JVM)**.
* Permite portabilidad entre sistemas.

---

## 1.6 Programas en Java

Java es uno de los lenguajes más utilizados en el mundo. Sus características:

* Código independiente de la arquitectura.
* Totalmente orientado a objetos.
* Sintaxis similar a C/C++.
* Gran biblioteca de clases.
* Preparado para aplicaciones en red.
* Seguro y robusto.

### Proceso de trabajo

* **Compilación**: con el **JDK**.
* **Ejecución**: con el **JRE**.

### Estructura básica de un programa

```java
public class HolaMundo {
  public static void main(String[] args) {
    System.out.println("Hola Mundo!");
  }
}
```

Tipos de programas en Java:

* Consola
* Aplicaciones gráficas
* Applets
* Servlets
* Midlets

---

## 1.7 Entornos de Desarrollo Integrado (IDE)

Un IDE integra editor, compilador, depurador y otras herramientas.

Los más conocidos para Java son:

* [IntelliJ IDEA](https://www.jetbrains.com/es-es/idea/): potente y muy usado en entornos profesionales.
* [NetBeans](https://netbeans.apache.org/): gratuito y versátil.

!!! tip "Consejo"
Empieza con un IDE sencillo, pero no olvides aprender también a compilar desde la terminal. Eso te ayudará a entender mejor cómo funciona Java.

---

## 1.8 Actividades

101. Enumera 5 ejemplos de tu vida diaria que dependen de la programación.

102. Explica con tus palabras la diferencia entre un **algoritmo** y un **programa**.

103. Representa en **pseudocódigo** un algoritmo que determine si un número es divisible por 5.

104. Busca 3 lenguajes de programación **compilados** y 3 **interpretados**. Comenta en qué situaciones se usaría cada uno.

105. Dibuja un **diagrama de flujo** que represente el cálculo del área de un triángulo.

106. Descarga e instala un IDE de Java. Crea un programa “Hola Mundo” y describe los pasos realizados.

107. Investiga la diferencia entre **OpenJDK** y **Oracle JDK** y explica cuál utilizarías en un proyecto educativo.

108. Busca una oferta de trabajo en programación y analiza: lenguaje solicitado, paradigma, herramientas y nivel de experiencia requerido.
