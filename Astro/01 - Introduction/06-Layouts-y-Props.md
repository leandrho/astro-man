
---

## 🧱 Layouts en Astro

### ¿Qué es un Layout?

Un **Layout** es un componente `.astro` que actúa como plantilla envolviendo otras páginas o componentes. Se usa para evitar duplicar secciones comunes como headers, footers, o metaetiquetas `<head>`.

### Ubicación típica

Los layouts suelen colocarse en la carpeta `src/layouts/`.

### Ejemplo básico de un layout: `BaseLayout.astro`

```astro
---
// src/layouts/BaseLayout.astro
const { title } = Astro.props;
---
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <title>{title}</title>
  </head>
  <body>
    <header><h1>{title}</h1></header>
    <main>
      <slot /> <!-- Aquí va el contenido de la página -->
    </main>
    <footer>© 2025 Mi sitio</footer>
  </body>
</html>
```

### Cómo se usa en una página `.astro`

```astro
---
// src/pages/about.astro
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout title="Sobre Nosotros">
  <p>Somos un equipo apasionado por la web.</p>
</BaseLayout>
```

> 🔁 Todo lo que esté entre `<BaseLayout> ... </BaseLayout>` se inyecta dentro del `<slot />` del layout.

---

## 🎁 Props en Astro

### ¿Qué son?

Las **props** son datos que se le pasan a un componente (o layout) desde otro componente o página.

### ¿Cómo se acceden?

En un archivo `.astro`, las props se acceden mediante `Astro.props`.

```astro
---
const { title, descripcion } = Astro.props;
---
<h2>{title}</h2>
<p>{descripcion}</p>
```

### Paso de props desde el padre

```astro
---
// src/pages/index.astro
import Card from '../components/Card.astro';
---
<Card title="Bienvenido" descripcion="Gracias por visitar nuestro sitio." />
```

---

## 🧩 Slots + Props

Los layouts y componentes pueden recibir tanto props como contenido dinámico (slots):

```astro
---
// src/layouts/ConSidebar.astro
const { titulo } = Astro.props;
---
<section>
  <aside>Barra lateral</aside>
  <main>
    <h2>{titulo}</h2>
    <slot />
  </main>
</section>
```

Y en su uso:

```astro
<ConSidebar titulo="Noticias">
  <p>Esta es la última noticia publicada.</p>
</ConSidebar>
```

---
