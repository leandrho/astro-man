
---

## 9. Integraciones y Plugins

Astro permite extender sus capacidades mediante integraciones oficiales y de la comunidad. Estas integraciones se agregan fácilmente y permiten habilitar soporte para frameworks, herramientas de CSS, manejo de imágenes, y más.

### 9.1 ¿Qué es una integración?

Una integración en Astro es un paquete que modifica o amplía el comportamiento del framework. Algunas integraciones populares incluyen soporte para Tailwind CSS, React, Vue, imágenes optimizadas, sitemap, entre otros.

### 9.2 Agregar una integración

Usa el siguiente comando para agregar una integración:

```bash
npx astro add [nombre]
```

Por ejemplo, para agregar Tailwind CSS:

```bash
npx astro add tailwind
```

Astro instalará el paquete necesario y actualizará automáticamente `astro.config.mjs` con la integración correspondiente.

### 9.3 Ejemplo: Configuración con Tailwind CSS

Después de ejecutar `npx astro add tailwind`, verás que se crea el archivo `tailwind.config.cjs` y se actualiza tu archivo `astro.config.mjs` así:

```js
import { defineConfig } from 'astro/config';
import tailwind from "@astrojs/tailwind";

export default defineConfig({
  integrations: [tailwind()]
});
```

Y en tu CSS global (por ejemplo, `src/styles/global.css`), puedes usar las directivas:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 9.4 Otras integraciones útiles

* `@astrojs/react`, `@astrojs/vue`, `@astrojs/svelte`: soporte para frameworks.
* `@astrojs/image`: optimización de imágenes.
* `@astrojs/sitemap`: generación automática de sitemap.
* `@astrojs/mdx`: soporte para MDX (mezcla Markdown + JSX).

### 9.5 Crear tus propias integraciones

También puedes crear tus propias integraciones si necesitas comportamiento personalizado. Para ello, puedes consultar la [documentación oficial sobre Integrations API](https://docs.astro.build/en/guides/integrations-guide/).
