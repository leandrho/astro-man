Las **Colecciones de Contenido (Content Collections)** en Astro son una forma poderosa y organizada de gestionar archivos locales de Markdown, MDX, JSON o YAML dentro de tu proyecto. Te permiten definir una estructura (schema) para tus datos, obtener validación y autocompletado en TypeScript, y consultar fácilmente este contenido para generar páginas o mostrar datos.

Son especialmente útiles para blogs, documentación, portafolios, listados de productos, o cualquier conjunto de contenido estructurado.

---
## ¿Qué son y por qué usar Colecciones de Contenido?

Imagina que estás construyendo un blog. Tendrás múltiples artículos, cada uno con un título, fecha de publicación, autor, etiquetas, y por supuesto, el contenido principal. Las colecciones te ayudan a:

1.  **Organizar tu Contenido:** Agrupa archivos de contenido relacionados en carpetas dedicadas (por ejemplo, `src/content/blog/`, `src/content/proyectos/`).
2.  **Definir una Estructura (Schema):** Para cada colección, defines un "schema" que especifica los campos que cada entrada *debe* o *puede* tener en su frontmatter (para Markdown/MDX) o en su estructura (para JSON/YAML). Por ejemplo, puedes hacer que el `title` y `pubDate` sean obligatorios.
3.  **Validación de Datos:** Astro valida automáticamente tus archivos de contenido contra el schema definido. Si falta un campo obligatorio o tiene un tipo incorrecto, Astro te avisará durante el desarrollo o la compilación.
4.  **Tipado Automático (TypeScript):** Basándose en tu schema, Astro genera automáticamente tipos de TypeScript. Esto significa que cuando consultes tus colecciones, tendrás autocompletado y seguridad de tipos para los datos de cada entrada, reduciendo errores.
5.  **APIs de Consulta Simplificadas:** Astro proporciona funciones (`getCollection`, `getEntry`) para leer y filtrar fácilmente las entradas de tus colecciones.
6.  **Renderizado de Contenido:** Para Markdown y MDX, Astro facilita la obtención del contenido HTML renderizado junto con los datos del frontmatter.

---
## ¿Cómo Funcionan las Colecciones?

**1. Estructura de Carpetas:**

Todo el contenido gestionado por colecciones reside dentro de la carpeta `src/content/`. Cada subcarpeta dentro de `src/content/` representa una colección diferente.

```
src/
└── content/
    ├── blog/                 <-- Colección "blog"
    │   ├── primer-post.md
    │   ├── segundo-post.mdx
    │   └── ...
    ├── autores/              <-- Colección "autores"
    │   ├── autor-uno.json
    │   ├── autor-dos.yaml
    │   └── ...
    └── config.ts             <-- Archivo de configuración de colecciones (o .js)
```

**2. Configuración de Colecciones (`src/content/config.ts` o `.js`):**

Aquí es donde defines los schemas para cada una de tus colecciones. Utilizas la utilidad `defineCollection` de Astro y `zod` (una biblioteca de validación de schemas) para especificar la estructura de los datos.

```typescript
// src/content/config.ts
import { defineCollection, z, reference } from 'astro:content';

// Schema para la colección "autores" (usando archivos JSON/YAML)
const autoresCollection = defineCollection({
  type: 'data', // Indica que son archivos de datos (JSON, YAML)
  schema: z.object({
    nombre: z.string(),
    bio: z.string().optional(), // Campo opcional
    avatar: z.string().url().optional(),
    redesSociales: z.object({
      twitter: z.string().url().optional(),
      github: z.string().url().optional(),
    }).optional(),
  }),
});

// Schema para la colección "blog" (usando archivos Markdown/MDX)
const blogCollection = defineCollection({
  type: 'content', // Indica que son archivos de contenido (MD, MDX)
  schema: ({ image }) => z.object({ // `image` es un helper para optimización de imágenes
    title: z.string().max(60, "¡El título es demasiado largo!"),
    description: z.string().min(10, "La descripción es muy corta.").max(160),
    pubDate: z.date(),
    updatedDate: z.date().optional(),
    heroImage: image().optional(), // Campo para imagen, optimizado por Astro
    tags: z.array(z.string()).default([]), // Un array de strings, con valor por defecto
    draft: z.boolean().default(false),
    autor: reference('autores'), // Referencia a una entrada de la colección "autores"
    // `reference` asegura que el valor coincida con un `slug` de una entrada en 'autores'.
  }),
});

// Exporta tus colecciones
export const collections = {
  blog: blogCollection,
  autores: autoresCollection,
};
```

**Explicación del Schema con `zod`:**

* `z.string()`: Espera un string.
* `z.date()`: Espera una fecha (Astro la parseará desde el frontmatter).
* `z.boolean()`: Espera un booleano (`true` o `false`).
* `z.array(z.string())`: Espera un array de strings (para las etiquetas).
* `z.object({...})`: Para campos anidados.
* `.optional()`: Hace que un campo no sea obligatorio.
* `.default(...)`: Proporciona un valor por defecto si el campo no está presente.
* `.min(x)`, `.max(x)`: Validaciones de longitud para strings.
* `.url()`: Valida que un string sea una URL.
* `reference('nombreOtraColeccion')`: Crea una relación entre colecciones. El valor debe ser el `slug` (identificador único, usualmente el nombre del archivo sin extensión) de una entrada en la colección referenciada.
* `image()`: Helper de Astro para procesar y optimizar imágenes referenciadas en el frontmatter. Necesita `astro add image` y configuración.
* `type: 'content'`: Para colecciones con cuerpo de contenido principal (Markdown, MDX).
* `type: 'data'`: Para colecciones que solo contienen datos estructurados (JSON, YAML).

**3. Creación de Contenido:**

* **Para `type: 'content'` (ej: `blog`):**
    Crea archivos `.md` o `.mdx` dentro de `src/content/blog/`. El frontmatter de estos archivos debe cumplir con el schema `blogCollection`.

    ```markdown
    ---
    // src/content/blog/mi-primer-post.md
    title: "Mi Primer Post en Astro"
    description: "Un emocionante comienzo con colecciones de contenido en Astro."
    pubDate: 2025-05-30
    heroImage: "./images/primer-post-cover.jpg" # Relativo al archivo .md
    tags: ["astro", "blogging", "tutorial"]
    draft: false
    autor: "autor-uno" # Este debe ser el slug de un archivo en src/content/autores/
    ---

    ## ¡Hola Mundo!

    Este es el contenido de mi primer post...
    ```

* **Para `type: 'data'` (ej: `autores`):**
    Crea archivos `.json` o `.yaml` dentro de `src/content/autores/`. El nombre del archivo (sin extensión) será el `slug` de la entrada.

    ```json
    // src/content/autores/autor-uno.json
    {
      "nombre": "Ana Pérez",
      "bio": "Desarrolladora web apasionada por Astro y las nuevas tecnologías.",
      "avatar": "https://example.com/avatars/ana.jpg",
      "redesSociales": {
        "twitter": "https://twitter.com/anaperezdev",
        "github": "https://github.com/anaperez"
      }
    }
    ```

    ```yaml
    # src/content/autores/autor-dos.yaml
    nombre: Carlos López
    bio: Escritor técnico y entusiasta de la documentación.
    # avatar es opcional, así que lo omitimos aquí.
    redesSociales:
      github: https://github.com/carloslopez
    ```

**4. Consultar Colecciones en tus Páginas Astro:**

Astro proporciona APIs para obtener tus datos de colección:

* `getCollection('nombreDeLaColeccion', filterCallback?)`: Obtiene todas las entradas de una colección. Opcionalmente, puedes pasar una función de filtro.
* `getEntry('nombreDeLaColeccion', 'slugDeLaEntrada')`: Obtiene una única entrada por su `slug` desde una colección de tipo `data`.
* `getEntryBySlug('nombreDeLaColeccion', 'slugDeLaEntrada')`: Obtiene una única entrada por su `slug` desde una colección de tipo `content`.
* `render()`: Un método en las entradas de contenido (`type: 'content'`) para obtener el HTML renderizado y otra información útil.

**Ejemplo: Mostrar una lista de posts del blog (`src/pages/blog/index.astro`)**

```astro
---
import Layout from '../../layouts/Layout.astro';
import { getCollection } from 'astro:content';

// 1. Obtener todas las entradas de la colección 'blog'
//    Filtramos para excluir borradores (drafts)
//    y ordenamos por fecha de publicación descendente
const posts = (await getCollection('blog', ({ data }) => {
  return data.draft === false; // o !data.draft
})).sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---
<Layout title="Mi Blog">
  <h1>Mi Blog</h1>
  <p>Aquí están mis últimos artículos.</p>
  <ul>
    {posts.map(post => (
      <li>
        <a href={`/blog/${post.slug}/`}>
          <img
            src={post.data.heroImage ? post.data.heroImage().src : '/placeholder-image.jpg'}
            alt={post.data.heroImage ? '' : 'Imagen de reemplazo'}
            width="100"
          />
          <h2>{post.data.title}</h2>
          <p>{post.data.description}</p>
          <p>
            <time datetime={post.data.pubDate.toISOString()}>
              {post.data.pubDate.toLocaleDateString('es-ES', {
                year: 'numeric', month: 'long', day: 'numeric'
              })}
            </time>
          </p>
        </a>
      </li>
    ))}
  </ul>
</Layout>
<style>
  ul { list-style: none; padding: 0; }
  li { margin-bottom: 2rem; border: 1px solid #eee; padding: 1rem; }
  li img { max-width: 100px; float: left; margin-right: 1rem; }
  li h2 { margin-top: 0; }
</style>
```

**Ejemplo: Generar páginas dinámicas para cada post del blog (`src/pages/blog/[...slug].astro`)**

Esta página usará el `slug` del archivo Markdown como parte de la URL.

```astro
---
import Layout from '../../layouts/Layout.astro';
import { getCollection, getEntry, type CollectionEntry } from 'astro:content';

// 1. getStaticPaths define todas las rutas que se generarán
export async function getStaticPaths() {
  const posts = await getCollection('blog', ({data}) => !data.draft);
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

// 2. Obtenemos el post específico para esta ruta
interface Props {
  post: CollectionEntry<'blog'>;
}
const { post } = Astro.props;

// 3. Renderizamos el contenido Markdown/MDX
const { Content, headings } = await post.render();

// 4. (Opcional) Obtener datos del autor referenciado
let autorData = null;
if (post.data.autor) {
  autorData = await getEntry(post.data.autor); // 'post.data.autor' es un objeto de referencia { collection, id }
}
---
<Layout title={post.data.title}>
  <article>
    {post.data.heroImage && (
      <img
        src={post.data.heroImage().src}
        width={post.data.heroImage().attributes.width}
        height={post.data.heroImage().attributes.height}
        alt=""
        class="hero-image"
      />
    )}
    <h1>{post.data.title}</h1>
    <p><em>{post.data.description}</em></p>
    <p>
      Publicado el: <time datetime={post.data.pubDate.toISOString()}>
        {post.data.pubDate.toLocaleDateString('es-ES', {
          year: 'numeric', month: 'long', day: 'numeric'
        })}
      </time>
      {post.data.updatedDate && (
        <> | Actualizado el:
          <time datetime={post.data.updatedDate.toISOString()}>
            {post.data.updatedDate.toLocaleDateString('es-ES', {
              year: 'numeric', month: 'long', day: 'numeric'
            })}
          </time>
        </>
      )}
    </p>

    {autorData && (
      <div class="autor-info">
        Escrito por: <strong>{autorData.data.nombre}</strong>
        {autorData.data.avatar && <img src={autorData.data.avatar} alt={autorData.data.nombre} width="50" />}
      </div>
    )}

    <div class="tags">
      Etiquetas:
      {post.data.tags.map(tag => <span class="tag">{tag}</span>)}
    </div>

    <hr />

    <Content />

    {headings.length > 0 && (
      <aside class="toc">
        <h2>En esta página</h2>
        <ul>
          {headings.map(heading => (
            <li class={`toc-depth-${heading.depth}`}>
              <a href={`#${heading.slug}`}>{heading.text}</a>
            </li>
          ))}
        </ul>
      </aside>
    )}
  </article>
</Layout>

<style is:global>
  .hero-image { width: 100%; max-height: 300px; object-fit: cover; margin-bottom: 1rem; }
  .autor-info { margin-bottom: 1rem; display: flex; align-items: center; gap: 0.5rem; }
  .autor-info img { border-radius: 50%; }
  .tags .tag { background-color: #f0f0f0; padding: 0.2em 0.5em; margin-right: 0.5em; border-radius: 4px; font-size: 0.9em; }
  .toc { border-left: 2px solid #eee; padding-left: 1rem; margin-top: 2rem; }
  .toc ul { list-style: none; padding: 0; }
  .toc-depth-3 { margin-left: 1rem; }
  .toc-depth-4 { margin-left: 2rem; }
</style>
```

**Explicación del Ejemplo de Página Dinámica:**

* `getStaticPaths`: Esta función es fundamental en Astro para la generación de sitios estáticos. Devuelve un array de objetos, donde cada objeto define una ruta (`params`) y las props que se pasarán a esa página. Aquí, creamos una ruta para cada `slug` de post.
* `Astro.props`: Contiene las `props` definidas en `getStaticPaths` para la instancia actual de la página.
* `post.render()`: Esta función asíncrona es crucial para las entradas de tipo `'content'`. Devuelve:
    * `Content`: Un componente que renderiza el cuerpo principal de tu archivo Markdown/MDX.
    * `headings`: Un array de los encabezados (`<h1>`, `<h2>`, etc.) encontrados en tu contenido, útil para generar una tabla de contenidos.
* `getEntry(post.data.autor)`: Cuando usas `reference` en tu schema, el campo (`post.data.autor`) contiene un objeto `{ collection: 'autores', id: 'autor-uno' }`. `getEntry` puede tomar este objeto directamente para obtener la entrada referenciada.

---
## Beneficios Clave Resumidos:

* **Organización Centralizada:** `src/content/` es la única fuente de verdad para tu contenido.
* **Seguridad de Tipos:** Menos errores gracias a la validación de schemas y el autocompletado de TypeScript.
* **Consultas Simplificadas:** APIs amigables para obtener y filtrar contenido.
* **Escalabilidad:** Ideal para sitios con mucho contenido que necesita ser estructurado y gestionado de manera consistente.
* **Integración con Imágenes:** El helper `image()` en los schemas permite la optimización automática de imágenes referenciadas en el frontmatter.

Las colecciones de contenido son una de las características más potentes y bien pensadas de Astro, haciendo que la gestión de contenido sea una experiencia mucho más agradable y robusta para los desarrolladores.
