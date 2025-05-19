En **Astro**, los **View Transitions** (transiciones de vistas) permiten animar el cambio de páginas sin necesidad de un framework frontend como React o Vue. Se basan en la API nativa de View Transitions de los navegadores, y Astro los integra para que funcionen fácilmente al navegar entre páginas del sitio.

---

### 🧠 ¿Cómo funcionan?

1. **La API de View Transitions** permite capturar visualmente el estado actual de la página.
2. Astro **intercepta la navegación** de enlaces internos (`<a>`) para realizar una transición animada en lugar de un recargo completo.
3. Se usa CSS para definir **cómo deben animarse** los elementos al cambiar de página.

---

### ✅ Requisitos

1. Tener Astro 3.0 o superior.
2. Activar `viewTransitions` en `astro.config.mjs`.

```js
// astro.config.mjs
export default {
  experimental: {
    viewTransitions: true
  }
};
```

---

### 📁 Estructura de ejemplo

Supongamos que tienes dos páginas:

* `/index.astro`
* `/about.astro`

Y quieres hacer que haya una animación suave al pasar de una a otra.

---

### 🧪 Ejemplo básico

#### `src/pages/index.astro`

```astro
---
---
<html>
  <head>
    <title>Inicio</title>
  </head>
  <body>
    <h1 style="view-transition-name: page-title">Bienvenido</h1>
    <a href="/about">Ir a About</a>
  </body>
</html>
```

#### `src/pages/about.astro`

```astro
---
---
<html>
  <head>
    <title>About</title>
  </head>
  <body>
    <h1 style="view-transition-name: page-title">Sobre nosotros</h1>
    <a href="/">Volver al Inicio</a>
  </body>
</html>
```

> El atributo `view-transition-name` permite que el navegador sepa qué elementos son equivalentes entre páginas, y los anime entre ellos.

---

### 🎨 Agregar animación con CSS

```css
::view-transition-old(page-title),
::view-transition-new(page-title) {
  animation-duration: 0.4s;
}

::view-transition-old(page-title) {
  animation-name: fade-out;
}

::view-transition-new(page-title) {
  animation-name: fade-in;
}

@keyframes fade-out {
  from { opacity: 1; }
  to   { opacity: 0; }
}

@keyframes fade-in {
  from { opacity: 0; }
  to   { opacity: 1; }
}
```

Puedes colocar este CSS global en `src/styles/global.css` y asegurarte de importarlo en el layout o en las páginas.

---

### 📝 Notas

* Solo funciona en navegadores que soporten la API (Chrome, Edge, etc.).
* Solo se activa cuando los elementos tienen el mismo `view-transition-name`.
* Es ideal para **navegaciones suaves**, headers, imágenes o títulos que se repiten entre páginas.


