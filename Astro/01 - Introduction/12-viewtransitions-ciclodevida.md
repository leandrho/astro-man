Las **View Transitions de Astro** facilitan la creación de animaciones suaves y transiciones entre diferentes páginas de tu sitio web, mejorando la experiencia del usuario. El ciclo de vida de una View Transition en Astro se compone de varias etapas clave que te permiten controlar y personalizar la animación.

---
## Ciclo de Vida de las View Transitions en Astro 🔄

El ciclo de vida de una View Transition en Astro se gestiona a través de una serie de eventos que se disparan en el documento durante el proceso de navegación. Estos eventos te permiten ejecutar código JavaScript en momentos específicos de la transición.

Aquí están las etapas principales:

1.  **Inicio de la Navegación:**
    * El usuario hace clic en un enlace que desencadena una navegación a una nueva página.
    * Astro intercepta esta navegación si las View Transitions están habilitadas.

2.  **Preparación (`astro:before-preparation`):**
    * Este evento se dispara **antes** de que Astro comience a preparar la página entrante y a capturar el estado de los elementos compartidos (elementos con el mismo `view-transition-name`).
    * Es un buen momento para realizar cambios de último minuto en la página actual antes de que se tomen las "instantáneas" de los elementos.

3.  **Captura del Estado Antiguo y Nuevo:**
    * Astro captura el estado (tamaño, posición, etc.) de los elementos con `view-transition-name` en la página actual (la "vieja" página).
    * Luego, carga la nueva página en segundo plano.
    * Una vez cargada, captura el estado de los elementos correspondientes con el mismo `view-transition-name` en la nueva página.

4.  **Preparación Completada (`astro:after-preparation`):**
    * Este evento se dispara **después** de que Astro ha capturado el estado de los elementos en ambas páginas y está listo para comenzar la animación de transición.
    * Puedes usar este evento para configurar animaciones personalizadas o modificar los elementos justo antes de que comience la transición visual.

5.  **Intercambio del DOM (`astro:before-swap`):**
    * Este evento se dispara **justo antes** de que Astro reemplace el contenido del DOM de la página actual con el contenido de la nueva página.
    * En este punto, las "instantáneas" de los elementos ya han sido tomadas. El DOM antiguo todavía está visible.

6.  **El DOM se Actualiza:**
    * Astro reemplaza el contenido del DOM antiguo con el nuevo.
    * El navegador ahora realiza la animación de transición basándose en las diferencias entre el estado "antiguo" y "nuevo" de los elementos compartidos. Por defecto, esto suele ser una animación de fundido cruzado (cross-fade) para los elementos no compartidos y una transformación para los elementos compartidos.

7.  **Animación en Curso:**
    * El navegador ejecuta las animaciones CSS definidas para la transición. Puedes personalizar estas animaciones usando CSS (por ejemplo, `@keyframes` y `animation`).

8.  **Intercambio Completado (`astro:after-swap`):**
    * Este evento se dispara **después** de que el nuevo contenido del DOM ha sido insertado y las transiciones han comenzado (o completado, si son instantáneas).
    * Es útil para realizar limpieza, inicializar nuevo JavaScript en la página entrante, o realizar ajustes finales después de que la nueva página sea visible.

---
## Ejemplo Práctico 📝

Aquí tienes un ejemplo sencillo para ilustrar cómo podrías usar estos eventos.

**Requisitos:**
* Asegúrate de tener las View Transitions habilitadas en tu `astro.config.mjs`:

    ```javascript
    // astro.config.mjs
    import { defineConfig } from 'astro/config';

    export default defineConfig({
      // Habilita las View Transitions
      viewTransitions: true
    });
    ```

**Estructura del Proyecto:**

```
/src
  /components
    Header.astro
  /layouts
    Layout.astro
  /pages
    index.astro
    about.astro
```

**1. Layout (`src/layouts/Layout.astro`):**

Asegúrate de que tu layout principal incluya el componente `<ViewTransitions />` de Astro en el `<head>`.

```html+astro
---
import { ViewTransitions } from 'astro:transitions';
import Header from '../components/Header.astro';
---
<html lang="es">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Astro View Transitions</title>
    <ViewTransitions /> <script>
        // Escucha los eventos del ciclo de vida de View Transitions
        document.addEventListener('astro:before-preparation', (event) => {
            console.log('🚀 Antes de la preparación:', event.to);
            // Ejemplo: Añadir una clase al body antes de la captura
            document.body.classList.add('is-preparing-transition');
        });

        document.addEventListener('astro:after-preparation', (event) => {
            console.log('✅ Preparación completada:', event.to);
            // Aquí podrías, por ejemplo, modificar algo antes de que empiece la animación
        });

        document.addEventListener('astro:before-swap', (event) => {
            console.log('🔄 Antes del intercambio del DOM:', event.to);
            // El DOM antiguo todavía está presente.
            // Puedes guardar algún estado si es necesario.
            document.body.classList.remove('is-preparing-transition');
            document.body.classList.add('is-swapping-dom');
        });

        document.addEventListener('astro:after-swap', (event) => {
            console.log('✨ Intercambio completado:', event.to);
            // El nuevo DOM está en su lugar.
            // Perfecto para inicializar scripts de la nueva página.
            document.body.classList.remove('is-swapping-dom');
            alert(`Navegaste a: ${event.to}`);
        });
    </script>
    <style>
        /* Estilo opcional para ver el efecto de las clases */
        .is-preparing-transition {
            /* Podrías aplicar un filtro o algo visual */
            opacity: 0.8;
        }
        .is-swapping-dom {
            /* Podrías aplicar un efecto mientras el DOM cambia */
            /* border: 2px solid red; */
        }

        /* Animación por defecto (fade) para todo el cuerpo */
        ::view-transition-old(root),
        ::view-transition-new(root) {
          animation-duration: 0.5s;
          animation-timing-function: ease-in-out;
        }
    </style>
</head>
<body>
    <Header />
    <main>
        <slot />
    </main>
</body>
</html>
```

**2. Componente de Cabecera (`src/components/Header.astro`):**

Podemos darle un `view-transition-name` a un elemento que queramos que se anime de forma persistente entre páginas.

```html+astro
---
// src/components/Header.astro
---
<header>
    <h1 transition:name="main-title">Mi Sitio con Transiciones</h1>
    <nav>
        <a href="/">Inicio</a>
        <a href="/about">Acerca de</a>
    </nav>
</header>

<style>
    header {
        background: #f0f0f0;
        padding: 1em;
        text-align: center;
        margin-bottom: 2em;
    }
    nav a {
        margin: 0 1em;
        text-decoration: none;
        color: #333;
    }
    /* transition:name debe estar en un elemento que persista o tenga un par en la otra página */
    [transition:name="main-title"] {
        color: rebeccapurple;
        /* No necesita `view-transition-name` en CSS aquí, Astro lo maneja.
           Pero si quieres personalizar su animación específica: */
    }

    /* Animación específica para el título */
    ::view-transition-old(main-title),
    ::view-transition-new(main-title) {
      animation-duration: 0.7s;
      animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
    }
</style>
```

**3. Página de Inicio (`src/pages/index.astro`):**

```html+astro
---
import Layout from '../layouts/Layout.astro';
---
<Layout>
    <section>
        <h2>Página de Inicio</h2>
        <p>Bienvenido a la página principal. Intenta navegar a la página "Acerca de".</p>
        <img src="https://via.placeholder.com/300" alt="Placeholder" transition:name="hero-image" />
    </section>
</Layout>
<style>
    /* Animación específica para la imagen en esta página */
    /* Nota: el nombre "hero-image" debe coincidir con el de la página "about" para una transición suave del elemento */
    ::view-transition-old(hero-image),
    ::view-transition-new(hero-image) {
      animation-duration: 1s;
      mix-blend-mode: normal; /* Asegúrate de que el modo de mezcla sea el esperado */
    }
</style>
```

**4. Página "Acerca de" (`src/pages/about.astro`):**

```html+astro
---
import Layout from '../layouts/Layout.astro';
---
<Layout>
    <section>
        <h2>Acerca de Nosotros</h2>
        <p>Esta es la página "Acerca de". Observa la transición del título y la imagen.</p>
        <img src="https://via.placeholder.com/300/0000FF/808080?Text=AboutPage" alt="Placeholder About" transition:name="hero-image" />
    </section>
    <style>
        section {
            background-color: #e6f7ff;
            padding: 20px;
        }
    </style>
</Layout>
```

**Funcionamiento:**

1.  Cuando navegas entre "Inicio" y "Acerca de", el componente `<ViewTransitions />` intercepta la navegación.
2.  Se disparan los eventos `astro:before-preparation`, `astro:after-preparation`, `astro:before-swap`, y `astro:after-swap` en ese orden. Podrás ver los `console.log` en la consola de tu navegador y el `alert` al final.
3.  El `<h1>` con `transition:name="main-title"` se animará suavemente entre las dos páginas porque tiene el mismo nombre de transición.
4.  La imagen con `transition:name="hero-image"` también se animará entre su estado en la página de inicio y su estado en la página "Acerca de".
5.  El resto del contenido de la página (el `<slot />`) tendrá una transición de fundido cruzado por defecto (controlada por `::view-transition-old(root)` y `::view-transition-new(root)`).

Este ejemplo te da una base para entender el flujo. Puedes usar los eventos para lógica más compleja, como cargar datos, actualizar el estado de la UI, o integrar con otras bibliotecas de animación si es necesario. La clave está en cómo Astro maneja la captura del estado antes y después, y te da puntos de enganche para tu propio código.
