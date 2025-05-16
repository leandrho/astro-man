## Manual de Astro

### 1. Introducción a Astro

Astro es un generador de sitios estáticos moderno centrado en el rendimiento y la simplicidad. Permite construir sitios web rápidos aprovechando el rendering estático por defecto, con la opción de inyectar componentes y lógica dinámica sólo cuando sea necesario.

#### 1.1 ¿Qué es un generador de sitios estáticos?

Un SSG (Static Site Generator) toma tus archivos de contenido (Markdown, MDX, etc.) y plantillas (HTML, JSX, Vue, Svelte, etc.) y produce páginas estáticas (.html, .css, .js) que se pueden servir desde cualquier CDN o servidor.

#### 1.2 Filosofía de Astro

* **Contenido primero**: prioriza el contenido estático para maximizar la velocidad de carga.
* **Islas de interactividad**: sólo hidrata (carga JavaScript) en los componentes que realmente lo necesitan.
* **Framework-agnostic**: puedes usar React, Vue, Svelte, Solid, Preact, etc., en el mismo proyecto.
* **Zero JavaScript by default**: si no añades componentes interactivos, no se incluye JavaScript en la página.

### 2. Ventajas principales

* **Alta velocidad**: páginas estáticas optimizadas, con HTML pre-generado.
* **Menor coste de hosting**: al servir contenido estático se reduce el uso de cómputo.
* **SEO mejorado**: el HTML completo ya está generado al desplegar.
* **Flexibilidad**: mezcla de frameworks y sintaxis.
* **Construcción incremental**: sólo reconstruye lo que cambia.

### 3. Arquitectura y flujo de trabajo

1. **Contenido**: Markdown, MDX u otras fuentes.
2. **Componentes**: archivos `.astro` o de otros frameworks.
3. **Build**: Astro lee contenido y componentes, genera HTML estático.
4. **Deploy**: subir los archivos resultantes a cualquier CDN/servidor.

### 4. Casos de uso comunes

* Blogs y documentación
* Landing pages
* Sitios de portafolio
* Documentación de proyectos de código abierto

### 5. Primeros pasos en un proyecto Astro

1. Instalar Astro:

   ```bash
   npm create astro@latest
   ```
2. Elegir plantilla: blog, docs, minimal, etc.
3. Ejecutar el servidor de desarrollo:

   ```bash
   npm run dev
   ```
4. Crear páginas en `src/pages/` y componentes en `src/components/`.
5. Generar la versión de producción:

   ```bash
   npm run build
   ```

---

