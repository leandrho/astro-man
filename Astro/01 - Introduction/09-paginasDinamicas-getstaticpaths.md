
## 📘 ¿Qué son las rutas dinámicas en Astro?

Las rutas dinámicas te permiten crear páginas que dependen de parámetros en la URL, como:

```
/blog/mi-articulo
/blog/otro-articulo
```

Para esto usás **archivos con nombres dinámicos** en `src/pages`, por ejemplo:

```
src/pages/blog/[slug].astro
```

El `[slug]` indica que es una ruta dinámica, y el valor real (`mi-articulo`, `otro-articulo`, etc.) será generado usando `getStaticPaths`.

---

## 📍 Paso a paso

### 1. Crear el archivo dinámico

```bash
src/pages/blog/[slug].astro
```

### 2. Usar `getStaticPaths` para generar las rutas

```astro
---
// [slug].astro
export async function getStaticPaths() {
  const posts = [
    { slug: 'mi-articulo', title: 'Mi artículo' },
    { slug: 'otro-articulo', title: 'Otro artículo' }
  ];

  return posts.map(post => ({
    params: { slug: post.slug },
    props: { title: post.title }
  }));
}

const { slug, title } = Astro.props;
---
<html>
  <head>
    <title>{title}</title>
  </head>
  <body>
    <h1>{title}</h1>
    <p>Este es el artículo con slug: {slug}</p>
  </body>
</html>
```

---

## 🔎 Explicación de `getStaticPaths`

`getStaticPaths` le dice a Astro:

1. Qué rutas dinámicas hay que generar.
2. Qué props usar en cada una.

Retorna un array de objetos:

```ts
type StaticPath = {
  params: Record<string, string>; // los valores dinámicos, como slug
  props?: Record<string, any>;    // opcional: datos para pasarle a la página
}
```

---

## 🗂 Ejemplo completo con archivo JSON

### `src/data/posts.json`

```json
[
  { "slug": "mi-articulo", "title": "Mi primer artículo" },
  { "slug": "otro-articulo", "title": "Otro gran post" }
]
```

### `src/pages/blog/[slug].astro`

```astro
---
import posts from '../../data/posts.json';

export async function getStaticPaths() {
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { title: post.title }
  }));
}

const { slug, title } = Astro.props;
---
<html>
  <head><title>{title}</title></head>
  <body>
    <h1>{title}</h1>
    <p>Estás leyendo el artículo con slug: {slug}</p>
    <a href="/blog">Volver al blog</a>
  </body>
</html>
```

---

## ✅ Resultado

Astro generará automáticamente:

* `/blog/mi-articulo/index.html`
* `/blog/otro-articulo/index.html`

Cada una con el contenido adecuado.

---

## 📝 Importante

* Si no usás `getStaticPaths`, Astro no sabrá qué rutas debe pre-renderizar y fallará en el build.
* También podés usar `getStaticPaths` para hacer fetch a una API o leer archivos del sistema (`fs.readdirSync()`).

