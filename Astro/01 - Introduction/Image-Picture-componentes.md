Astro ha puesto un gran énfasis en el manejo optimizado de imágenes, ya que son cruciales para el rendimiento web. Para esto, Astro ofrece los componentes `<Image />` y `<Picture />` (a través de su sistema de **Assets `astro:assets`**) que facilitan la optimización, transformación y entrega de imágenes de manera eficiente.

Primero, es importante entender que estas funcionalidades son parte del manejo de assets incorporado en Astro, disponible desde Astro 3.0. Ya no necesitas instalar un paquete separado como `@astrojs/image` para imágenes locales; `astro:assets` lo maneja todo.

---
## `astro:assets`: La Base del Manejo de Imágenes

Antes de sumergirnos en los componentes, hablemos de `astro:assets`. Esta característica permite:

1.  **Importar Imágenes Directamente:** Puedes importar imágenes (`.jpg`, `.png`, `.webp`, `.avif`, `.svg`, etc.) directamente en tus componentes `.astro` o archivos de frontmatter Markdown.
    ```javascript
    import miImagenLocal from '../assets/paisaje.jpg';
    import iconoSVG from '../assets/icono.svg';
    ```
2.  **Optimización en Tiempo de Compilación:** Astro procesa y optimiza estas imágenes durante el `build`. Esto puede incluir:
    * Redimensionar imágenes.
    * Convertir a formatos modernos como WebP o AVIF.
    * Generar múltiples variantes para `srcset` (imágenes responsivas).
    * Minimizar el tamaño del archivo.
3.  **Metadatos de Imagen:** Al importar una imagen, obtienes un objeto con metadatos útiles (ancho, alto, formato original, etc.).
4.  **Servicio de Imágenes:** Astro utiliza un servicio de imágenes (como Sharp, que es el predeterminado para procesamiento local, o Squoosh) para realizar estas transformaciones. No necesitas configurar nada complejo para empezar con imágenes locales.

**¿Dónde colocar tus imágenes?**
Puedes colocar tus imágenes en cualquier lugar de tu proyecto, pero una práctica común es usar la carpeta `src/assets/`. Sin embargo, puedes importarlas desde `src/pages/`, etc., si están junto a tu contenido.

---
## El Componente `<Image />`

El componente `<Image />` se utiliza para renderizar una etiqueta `<img>` optimizada. Es ideal para la mayoría de los casos de uso donde necesitas mostrar una imagen con optimizaciones automáticas y responsividad básica.

**Importación:**
```astro
---
import { Image } from 'astro:assets';
import miImagen from '../assets/mi-foto.jpg'; // Asegúrate de que esta ruta sea correcta
---
```

**Props Clave:**

* `src` (obligatorio): El objeto de imagen importado o una ruta de string a una imagen en `public/` (aunque para optimizaciones, la importación es preferida).
* `alt` (obligatorio): Texto alternativo descriptivo para la imagen.
* `width` y `height` (muy recomendados): Dimensiones de la imagen renderizada. Proporcionarlos ayuda a prevenir el Cumulative Layout Shift (CLS). Astro puede inferirlos de la imagen original si no se especifican, pero para la imagen renderizada es bueno tener control.
* `format`: Formato de salida deseado (ej: `'webp'`, `'avif'`, `'png'`, `'jpeg'`). Astro convertirá la imagen.
* `quality`: Un número entre `0` y `100` o una preselección como `'low'`, `'mid'`, `'high'`, `'max'`. Controla la compresión.
* `densities`: Un array de descriptores de densidad (ej: `[1.5, 2]`) para generar imágenes para pantallas de alta resolución (`1.5x`, `2x`).
* `widths`: Un array de anchos en píxeles (ej: `[200, 400, 800]`) para generar múltiples versiones de la imagen para diferentes tamaños de viewport, usadas en el atributo `srcset`.
* `sizes`: (Opcional, pero recomendado con `widths`) Un string que describe cómo se mostrará la imagen en diferentes anchos de viewport (ej: `"(max-width: 600px) 100vw, 50vw"`).
* Cualquier otro atributo HTML estándar para `<img>` (ej: `class`, `loading`, `decoding`). `loading="lazy"` y `decoding="async"` suelen ser valores predeterminados inteligentes.

**Ejemplos:**

1.  **Uso Básico (con inferencia de dimensiones de la fuente):**
    ```astro
    ---
    import { Image } from 'astro:assets';
    import imagenPerfil from '../assets/perfil.png';
    ---
    <Image src={imagenPerfil} alt="Foto de perfil de Juan Pérez" />
    ```
    *Astro usará las dimensiones de `perfil.png` e intentará optimizarla.*

2.  **Especificando Dimensiones y Formato de Salida:**
    ```astro
    ---
    import { Image } from 'astro:assets';
    import portadaArticulo from '../assets/portada.jpg';
    ---
    <Image
      src={portadaArticulo}
      alt="Portada del artículo sobre viajes"
      width={800}
      height={400}
      format="webp"
      quality={80}
      class="img-portada"
    />
    ```
    *Generará un `<img src="ruta/a/portada.webp" width="800" height="400" ... />`.*

3.  **Imágenes Responsivas con `widths` y `sizes`:**
    ```astro
    ---
    import { Image } from 'astro:assets';
    import productoDestacado from '../assets/producto.jpg';
    ---
    <Image
      src={productoDestacado}
      alt="Producto destacado XYZ"
      widths={[200, 400, 800, 1200]}
      sizes="(max-width: 600px) 100vw, (max-width: 1024px) 50vw, 800px"
      format="avif"
      alt="Un producto muy chulo"
    />
    ```
    *Esto generará un `<img>` con un atributo `srcset` que lista las versiones de la imagen en 200px, 400px, 800px y 1200px de ancho en formato AVIF. El atributo `sizes` ayuda al navegador a elegir cuál descargar.*

---
## El Componente `<Picture />`

El componente `<Picture />` se utiliza para renderizar un elemento HTML `<picture>`. Es más avanzado y se usa principalmente para dos escenarios:

1.  **Ofrecer Múltiples Formatos de Imagen:** Puedes especificar varios formatos (como AVIF, WebP, y un fallback a JPEG/PNG), y el navegador elegirá el primer formato que soporte. Esto se conoce como "format negotiation".
2.  **Dirección de Arte (Art Direction):** Aunque el componente `<Picture />` de `astro:assets` se centra principalmente en la negociación de formatos, el elemento HTML `<picture>` en sí es para la dirección de arte (mostrar imágenes completamente diferentes o diferentes recortes en distintos tamaños de pantalla). Para una dirección de arte compleja con `astro:assets`, podrías necesitar construir manualmente las etiquetas `<source>` dentro de un `<Picture>`, usando el componente `<Image />` (o sus propiedades de transformación) para generar cada fuente. Sin embargo, el uso más directo del componente `<Picture />` es para la selección de formatos.

**Importación:**
```astro
---
import { Picture } from 'astro:assets';
import miHeroImage from '../assets/hero-banner.jpg';
---
```

**Props Clave:**

* `src` (obligatorio): El objeto de imagen importado para la imagen base/fallback.
* `alt` (obligatorio): Texto alternativo.
* `formats` (obligatorio para `Picture`): Un array de strings de formatos a generar (ej: `['avif', 'webp']`). Astro creará elementos `<source>` para cada uno. El formato del `src` original se usará como fallback si no se especifica uno explícitamente o si `includeFallbackFormat` no es `false`.
* `fallbackFormat`: (Opcional) Especifica el formato para la etiqueta `<img>` de fallback (ej: `'jpeg'`, `'png'`). Por defecto, usa el formato del `src` o el último formato viable.
* `widths`, `sizes`, `quality`: Funcionan de manera similar al componente `<Image />`, pero se aplican a cada `<source>` generado.
* `aspectRatio`: Puedes especificar un aspect ratio (ej: `16/9` o `1.777`).
* `pictureAttributes`: Un objeto con atributos para pasar al elemento `<picture>` mismo (ej: `{ class: 'mi-clase-picture' }`).
* Otros atributos HTML estándar para `<img>` (ej: `class`, `loading`, `decoding`) se aplican a la etiqueta `<img>` de fallback dentro del `<picture>`.

**Ejemplos:**

1.  **Ofreciendo Múltiples Formatos:**
    ```astro
    ---
    import { Picture } from 'astro:assets';
    import galeriaFoto from '../assets/galeria-01.png';
    ---
    <Picture
      src={galeriaFoto}
      widths={[400, 800, 1200]}
      sizes="(max-width: 768px) 100vw, 800px"
      formats={['avif', 'webp']}
      alt="Una hermosa foto de una galería de arte"
      loading="lazy"
    />
    ```
    Esto generaría algo como:
    ```html
    <picture>
      <source type="image/avif" srcset="ruta/a/galeria-01-400.avif 400w, ruta/a/galeria-01-800.avif 800w, ruta/a/galeria-01-1200.avif 1200w" sizes="(max-width: 768px) 100vw, 800px">
      <source type="image/webp" srcset="ruta/a/galeria-01-400.webp 400w, ruta/a/galeria-01-800.webp 800w, ruta/a/galeria-01-1200.webp 1200w" sizes="(max-width: 768px) 100vw, 800px">
      <img src="ruta/a/galeria-01.png" alt="Una hermosa foto de una galería de arte" loading="lazy" decoding="async" width="originalWidth" height="originalHeight">
    </picture>
    ```
    El navegador elegirá el primer formato de `<source>` que soporte (AVIF, luego WebP) y, si no soporta ninguno, usará el `<img>` (PNG en este caso).

2.  **Controlando el Fallback y Atributos del `picture`:**
    ```astro
    ---
    import { Picture } from 'astro:assets';
    import logoEmpresa from '../assets/logo-empresa.svg'; // SVG también puede ser un src, pero los formatos rasterizados son más comunes para Picture
    import logoEmpresaPng from '../assets/logo-empresa.png'; // Si el original es SVG, necesitas un raster para formatos de Picture
    ---
    <Picture
      src={logoEmpresaPng} {/* Usamos un PNG como base para transformaciones rasterizadas */}
      formats={['webp']}
      fallbackFormat="png" {/* Asegura que el fallback sea PNG */}
      alt="Logo de la Empresa"
      width={200} {/* Para el img de fallback */}
      pictureAttributes={{ class: 'logo-principal' }}
    />
    ```

**Dirección de Arte más Avanzada (Manual):**
Si necesitas mostrar imágenes completamente diferentes (no solo diferentes formatos/resoluciones de la *misma* imagen) para distintos breakpoints (art direction), usarías el elemento `<picture>` con múltiples `<source media="...">` y el componente `<Image />` para cada fuente.

```astro
---
import { Image } from 'astro:assets';
import heroDesktop from '../assets/hero-desktop.jpg';
import heroMobile from '../assets/hero-mobile.jpg';
---
<picture>
  <source media="(min-width: 768px)" srcset={(await Image.getImage({src: heroDesktop, format: 'webp', width: 1200})).src} />
  <source media="(min-width: 768px)" srcset={(await Image.getImage({src: heroDesktop, format: 'jpeg', width: 1200})).src} />

  <source media="(max-width: 767px)" srcset={(await Image.getImage({src: heroMobile, format: 'webp', width: 600})).src} />
  <source media="(max-width: 767px)" srcset={(await Image.getImage({src: heroMobile, format: 'jpeg', width: 600})).src} />

  <Image src={heroDesktop} alt="Banner principal" format="jpeg" width={1200} height={400} />
</picture>
```
*Nota: El uso de `Image.getImage()` es una forma de obtener la URL de la imagen procesada programáticamente. Esto es más manual que el componente `<Picture />` directo para formatos.*
Una forma más declarativa y alineada con `astro:assets` sería:
```astro
---
import { Image } from 'astro:assets';
import heroDesktop from '../assets/hero-desktop.jpg';
import heroMobile from '../assets/hero-mobile.jpg';
---
<picture>
  <source media="(min-width: 768px)" >
    <Image src={heroDesktop} format="webp" width={1200} alt="" class="hidden-source-img" />
  </source>
  <source media="(min-width: 768px)" >
    <Image src={heroDesktop} format="jpeg" width={1200} alt="" class="hidden-source-img" />
  </source>

  <source media="(max-width: 767px)" >
    <Image src={heroMobile} format="webp" width={600} alt="" class="hidden-source-img" />
  </source>
  <source media="(max-width: 767px)" >
    <Image src={heroMobile} format="jpeg" width={600} alt="" class="hidden-source-img" />
  </source>

  <Image src={heroDesktop} alt="Banner principal" format="jpeg" width={1200} height={400} />
</picture>
<style is:global>
  .hidden-source-img img { display: none; } /* Para evitar que los Image dentro de source se muestren por sí mismos */
</picture>
```
Este enfoque de anidar `<Image>` dentro de `<source>` para dirección de arte no es el caso de uso principal para el que `astro:assets` fue diseñado y puede tener peculiaridades. La documentación de Astro se enfoca en `<Picture>` para la selección de `formats`. Para art direction compleja, a menudo se recurre a soluciones CSS o a la construcción más manual de los `srcset` dentro de los `<source>`.

**La fortaleza principal del componente `<Picture />` de `astro:assets` es la fácil generación de múltiples formatos para la *misma imagen de origen*.**

---
## `<Image />` vs `<Picture />`

* **Usa `<Image />` si:**
    * Solo necesitas un formato de salida (aunque puedes generar `srcset` para responsividad).
    * Quieres la forma más sencilla de mostrar una imagen optimizada.
    * La mayoría de tus necesidades de imagen.

* **Usa `<Picture />` si:**
    * Necesitas ofrecer múltiples formatos de imagen (ej: AVIF, WebP, y un fallback JPEG/PNG) para la misma imagen, permitiendo que el navegador elija el mejor que soporte.
    * (Con construcción más manual) Quieres implementar "art direction" donde la imagen en sí cambia o tiene diferentes recortes en diferentes tamaños de pantalla. El componente `<Picture />` de `astro:assets` ayuda principalmente con los formatos, no con fuentes de imagen completamente diferentes para art direction de forma directa.

---
## Beneficios Clave

* **Rendimiento Web Mejorado:** Imágenes optimizadas, formatos modernos, lazy loading.
* **Prevención de CLS:** Al especificar `width` y `height`, se reserva el espacio para la imagen antes de que cargue.
* **Mejor Experiencia de Desarrollador:** Importaciones simples, API de componentes clara, seguridad de tipos con TypeScript.
* **Automático y Configurable:** Gran parte de la optimización es automática, pero tienes control fino a través de las props.

En resumen, los componentes `<Image />` y `<Picture />` de `astro:assets` son herramientas esenciales para construir sitios web rápidos y modernos con Astro, manejando de forma eficiente uno de los aspectos más críticos del rendimiento web: las imágenes.
