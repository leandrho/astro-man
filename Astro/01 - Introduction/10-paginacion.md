La **paginación estática** en Astro te permite dividir grandes cantidades de contenido en múltiples páginas HTML discretas durante el proceso de construcción (build time). Esto significa que cada página paginada existe como un archivo `.html` individual, listo para ser servido directamente a los usuarios sin necesidad de procesamiento del lado del servidor o del cliente en tiempo de ejecución para generar la paginación. ⚙️

Es una característica poderosa para sitios con mucho contenido, como blogs, catálogos de productos o galerías, ya que mejora el rendimiento de carga y el SEO.

---
## ¿Cómo funciona?

En esencia, Astro toma una colección de datos (por ejemplo, artículos de un blog o productos) y, basándose en una configuración que tú defines, genera un conjunto de páginas HTML. Cada página mostrará un subconjunto de esos datos.

El proceso generalmente involucra estos pasos:

1.  **Obtención de Datos**: Primero, necesitas una fuente de datos. Esto puede ser archivos Markdown locales, una API externa, una base deatos, etc. Astro te permite obtener estos datos usando `Astro.glob()` para archivos locales o `Workspace()` para APIs dentro de tus archivos `.astro`.
2.  **Función `getStaticPaths`**: Esta es la función clave para la generación de rutas dinámicas y la paginación estática en Astro. Dentro de esta función, le dices a Astro cómo debe generar las páginas.
3.  **Lógica de Paginación**: En `getStaticPaths`, implementas la lógica para dividir tus datos en "trozos" o páginas. Esto usualmente implica calcular cuántas páginas se necesitan y qué datos irán en cada una. Astro provee una función `paginate()` para simplificar este proceso.
4.  **Generación de Rutas**: La función `paginate()` (o tu lógica personalizada) devuelve un array de objetos. Cada objeto describe una ruta (una página HTML a generar) y los datos (props) que se pasarán a esa página. Estos props incluirán los datos específicos para esa página (ej. los 10 artículos del blog para la página 2) e información de paginación (ej. número de página actual, URL de la página siguiente/anterior, total de páginas).
5.  **Renderizado de la Página**: En la parte HTML de tu archivo `.astro`, usas los props recibidos para renderizar el contenido de la página actual y los controles de navegación de la paginación (ej. enlaces "Siguiente", "Anterior", números de página).

---
## Implementación Básica con `paginate()`

Astro facilita enormemente la paginación estática con su función `paginate()`. Aquí tienes un ejemplo conceptual de cómo se vería en un archivo como `src/pages/blog/[page].astro` o `src/pages/blog/pagina/[page].astro` para un blog:

```astro
---
// src/pages/blog/[...page].astro o src/pages/blog/pagina/[...page].astro
// (Usar [...page].astro permite que la primera página sea /blog/ en lugar de /blog/1/)

// 1. Obtener todos los artículos del blog (por ejemplo, de archivos Markdown)
const allPosts = await Astro.glob('../posts/*.md');
// Ordenar posts por fecha, por ejemplo
const sortedPosts = allPosts.sort((a, b) => new Date(b.frontmatter.pubDate).valueOf() - new Date(a.frontmatter.pubDate).valueOf());

export async function getStaticPaths({ paginate }) {
  // 2. Usar la función paginate()
  // pageSize indica cuántos artículos por página
  return paginate(sortedPosts, { pageSize: 5 });
}

// 3. Los props `page` son inyectados automáticamente por `paginate()`
const { page } = Astro.props;
// `page.data` contiene los artículos para la página actual
// `page.url.prev` y `page.url.next` contienen las URLs para la navegación
// `page.currentPage`, `page.totalPages`, `page.start`, `page.end`, `page.total`
// también están disponibles.
---

<h1>Mi Blog</h1>
<p>Página {page.currentPage} de {page.totalPages}</p>

<ul>
  {page.data.map(post => (
    <li>
      <a href={post.url}>{post.frontmatter.title}</a>
      <p>{post.frontmatter.description}</p>
    </li>
  ))}
</ul>

<div>
  {page.url.prev && <a href={page.url.prev}>&laquo; Anterior</a>}
  {page.url.next && <a href={page.url.next}>Siguiente &raquo;</a>}
</div>

<style>
  div {
    display: flex;
    justify-content: space-between;
    margin-top: 2rem;
  }
</style>
```

**Explicación del Ejemplo:**

* **`[...page].astro`**: El uso de `...` (spread syntax) en el nombre del archivo junto con `paginate` permite que la primera página de la paginación tenga una URL "limpia" (ej. `/blog/`) y las subsiguientes sean `/blog/2`, `/blog/3`, etc. Si usaras `[page].astro`, la primera página sería `/blog/1`.
* **`Astro.glob('../posts/*.md')`**: Carga todos los archivos Markdown de la carpeta `posts`.
* **`export async function getStaticPaths({ paginate })`**: Aquí es donde defines cómo se generarán las páginas. Astro pasa la función `paginate` como argumento.
* **`paginate(sortedPosts, { pageSize: 5 })`**: Esta función toma tu array de datos (`sortedPosts`) y un objeto de opciones. `pageSize: 5` significa que cada página mostrará 5 artículos. `paginate` automáticamente calcula cuántas páginas se necesitan y qué datos van en cada una.
* **`const { page } = Astro.props;`**: Dentro de la parte de renderizado del componente, `Astro.props` contendrá un objeto `page`. Este objeto es proporcionado por la función `paginate` y contiene:
    * `page.data`: Un array con los elementos de datos para la página actual (los 5 artículos en este caso).
    * `page.url.prev`: La URL de la página anterior (si existe).
    * `page.url.next`: La URL de la página siguiente (si existe).
    * `page.currentPage`: El número de la página actual.
    * `page.totalPages`: El número total de páginas.
    * `page.size`: El `pageSize` que definiste.
    * `page.total`: El número total de ítems.
    * `page.start`: El índice (basado en 0) del primer ítem en la página actual.
    * `page.end`: El índice (basado en 0) del último ítem en la página actual.
* **Renderizado**: El HTML utiliza `page.data` para mostrar los artículos y `page.url.prev`/`page.url.next` para los enlaces de navegación.

---
## Ventajas de la Paginación Estática

* 🚀 **Mejor Rendimiento**: Las páginas se generan en el momento de la compilación (build time), por lo que se sirven como archivos HTML estáticos. Esto es extremadamente rápido y eficiente.
* 👍 **Excelente SEO**: Cada página paginada es una URL distinta con su propio contenido, lo que facilita que los motores de búsqueda las rastreen e indexen.
* 🔒 **Mayor Seguridad y Escalabilidad**: Al ser archivos estáticos, se reduce la superficie de ataque y son inherentemente escalables a través de CDNs.
* 💡 **Experiencia de Usuario Mejorada**: Los usuarios obtienen tiempos de carga rápidos y una navegación clara entre secciones de contenido.
* 🌍 **Despliegue Sencillo**: Puedes desplegar tu sitio en cualquier hosting de archivos estáticos.

---
## Casos de Uso Comunes

* **Blogs**: Mostrar una lista de artículos dividida en múltiples páginas.
* **Catálogos de Productos**: Presentar productos en una tienda online, con un número determinado de productos por página.
* **Galerías de Imágenes**: Dividir una gran colección de imágenes.
* **Listados de Directorios**: Por ejemplo, un listado de negocios o servicios.
* **Resultados de Búsqueda (si los datos son estáticos o se pueden generar en el build)**: Aunque menos común para búsquedas dinámicas, podría usarse para categorías predefinidas.

---
## Paginación con Rutas Nombradas (Named Routes)

También puedes personalizar más la URL de tus páginas paginadas. Por ejemplo, si en lugar de `/blog/2`, `/blog/3`, quisieras `/blog/pagina/2`, `/blog/pagina/3`, podrías estructurar tus archivos así: `src/pages/blog/pagina/[page].astro`.

El parámetro `params` en `paginate` te permite pasar datos adicionales a cada ruta generada si fuera necesario, y el `prop` te permite nombrar la propiedad que contendrá los datos de la página (por defecto es `page`).

```astro
// src/pages/mis-items/pagina-[pageNum].astro
export async function getStaticPaths({ paginate }) {
  const items = ['A', 'B', 'C', 'D', 'E', 'F'];
  return paginate(items, {
    pageSize: 2,
    params: { tag: 'popular' }, // Parámetro adicional para todas las páginas
    props: { customData: 'Hola Mundo' } // Prop adicional para todas las páginas
  });
}

const { page, tag, customData } = Astro.props;
// page.params.pageNum contendrá el número de página
---
<h1>Items - Página {page.params.pageNum}</h1>
<p>Tag: {tag}</p>
<p>Custom Prop: {customData}</p>
<ul>
  {page.data.map(item => (
    <li>{item}</li>
  ))}
</ul>
{page.url.prev && <a href={page.url.prev}>Anterior</a>}
{page.url.next && <a href={page.url.next}>Siguiente</a>}
```

En este caso, las URLs generadas serían `/mis-items/pagina-1`, `/mis-items/pagina-2`, etc. Y `page.params.pageNum` te daría el número de página actual.

---

En resumen, la paginación estática en Astro es una forma eficiente y amigable con el SEO para manejar grandes conjuntos de contenido, generando páginas HTML individuales en el momento de la compilación. La función `paginate()` simplifica enormemente este proceso, proporcionando todos los datos necesarios para renderizar tanto el contenido de la página actual como los controles de navegación. ✨
