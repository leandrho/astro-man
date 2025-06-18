Los **Adaptadores SSR (Server-Side Rendering)** son una pieza fundamental en el ecosistema de Astro que te permite llevar tu sitio más allá de las páginas estáticas y desplegar aplicaciones dinámicas en una gran variedad de plataformas de hosting.

Vamos a explicarlo en detalle.

---
### 1. La Base: Static (SSG) vs. Server (SSR) en Astro

Por defecto, Astro es un generador de sitios **estáticos (Static Site Generation - SSG)**. Esto significa que cuando ejecutas el comando `astro build`, Astro genera archivos HTML, CSS y JavaScript puros.

* **Ventajas del modo estático (`output: 'static'`):**
    * **Rendimiento Extremo:** Servir archivos HTML es lo más rápido posible.
    * **Seguridad:** No hay un servidor en vivo que pueda ser atacado.
    * **Hosting Sencillo y Barato:** Puedes desplegarlo en cualquier servidor web simple (como GitHub Pages, Netlify, Vercel, etc.).

Sin embargo, hay muchas cosas que no puedes hacer con un sitio puramente estático, como:
* Procesar formularios en el servidor.
* Tener páginas protegidas por autenticación de usuario.
* Mostrar contenido personalizado para cada visitante.
* Crear **API Routes** para que tu frontend consuma datos.

Aquí es donde entra el **Renderizado del Lado del Servidor (Server-Side Rendering - SSR)**. Al cambiar Astro al modo SSR (`output: 'server'`), le dices que en lugar de generar archivos HTML en el momento de la compilación, debe generar el HTML de una página **en el momento en que un usuario la solicita**.

---
### 2. ¿Qué es un Adaptador SSR? El Puente hacia el Despliegue

Ahora, el problema es que cada plataforma de hosting (Vercel, Netlify, Cloudflare, un servidor Node.js propio, etc.) tiene una forma diferente de ejecutar código del lado del servidor.

* **Vercel y Netlify** usan "funciones serverless".
* **Cloudflare** usa "edge workers".
* Un **servidor Node.js** necesita un script que inicie un servidor (como Express o Fastify).

Un **Adaptador de Astro** es un plugin que actúa como un **puente** o **traductor**. Su trabajo es tomar el código de servidor genérico que Astro produce y "adaptarlo" al formato específico que la plataforma de despliegue de destino entiende.

En resumen, el adaptador es el responsable de que tu aplicación Astro dinámica pueda ejecutarse correctamente en el entorno de producción que elijas.

---
### 3. ¿Cómo Funcionan los Adaptadores?

El proceso es bastante directo:

1.  **Configurar el Modo SSR:** En tu archivo `astro.config.mjs`, cambias la opción `output` a `'server'`.
2.  **Instalar y Añadir el Adaptador:** Instalas el paquete del adaptador para tu plataforma de destino y lo añades a la configuración de Astro.
3.  **El Proceso de `astro build`:** Cuando ejecutas `astro build`, en lugar de generar un montón de archivos `.html`, Astro crea una estructura diferente en la carpeta `dist/`:
    * `dist/client/`: Contiene todos los assets estáticos (CSS, JavaScript del lado del cliente, imágenes, fuentes).
    * `dist/server/`: Contiene un único punto de entrada (por ejemplo, `entry.mjs`) que es el servidor de Astro empaquetado y listo para ser ejecutado.
4.  **Ejecución en el Servidor:** El adaptador se asegura de que el script en `dist/server/` se ejecute de la manera que la plataforma espera. Por ejemplo:
    * El adaptador de Node.js crea un script que puedes ejecutar con `node dist/server/entry.mjs`.
    * El adaptador de Vercel empaqueta ese script dentro de una estructura de función serverless que Vercel entiende y despliega automáticamente.

---
### Ejemplos Prácticos

Veamos cómo configurar dos de los adaptadores más comunes.

#### Ejemplo 1: Despliegue en un Servidor Node.js (`@astrojs/node`)

Usa este adaptador si quieres tener control total y desplegar tu aplicación en tu propio servidor (un VPS, un contenedor Docker, etc.).

**a. Instalación:**

```bash
# El comando 'add' instala el paquete y actualiza tu configuración automáticamente
npx astro add node
```

**b. Configuración (`astro.config.mjs`):**

El comando anterior debería haber modificado tu archivo de configuración para que se vea así:

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node'; // Importa el adaptador

export default defineConfig({
  output: 'server', // Cambia el modo a SSR
  adapter: node({   // Añade el adaptador
    mode: 'standalone' // O 'middleware' si quieres integrarlo con un servidor Express existente
  }),
});
```

**c. Crear una Página Dinámica:**

Vamos a crear una página que muestre la hora actual del servidor, algo que es imposible en un sitio estático.

```astro
---
// src/pages/hora.astro
import Layout from '../layouts/Layout.astro';

// Este código se ejecuta en el servidor en cada solicitud
const horaActual = new Date().toLocaleTimeString('es-AR', { timeZone: 'America/Argentina/Buenos_Aires' });
---
<Layout title="Hora del Servidor">
  <h1>Página Dinámica</h1>
  <p>Esta página fue generada en el servidor.</p>
  <p>La hora actual en el servidor es: <strong>{horaActual}</strong></p>
  <p>Refresca la página para ver cómo la hora cambia en cada solicitud.</p>
</Layout>
```

**d. Compilar y Ejecutar:**

1.  **Compila la aplicación:**
    ```bash
    npm run build
    ```
2.  **Ejecuta el servidor:**
    ```bash
    node dist/server/entry.mjs
    ```

¡Listo! Si visitas `http://localhost:4321/hora` (o el puerto que te indique la terminal), verás la hora del servidor, y cambiará cada vez que refresques la página.

---
#### Ejemplo 2: Despliegue en Vercel (`@astrojs/vercel`)

Usa este adaptador para desplegar en la plataforma serverless de Vercel.

**a. Instalación:**

```bash
npx astro add vercel
```

**b. Configuración (`astro.config.mjs`):**

El comando `add` configurará tu archivo automáticamente:

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel/serverless'; // Importa el adaptador

export default defineConfig({
  output: 'server', // Modo SSR
  adapter: vercel(),  // Añade el adaptador de Vercel
});
```
*Nota: Vercel también soporta Edge Functions, en cuyo caso el import sería de `@astrojs/vercel/edge`.*

**c. Crear una API Route (un caso de uso común para SSR):**

Las API Routes solo funcionan en modo SSR.

```typescript
// src/pages/api/saludo.json.ts
import type { APIRoute } from 'astro';

export const GET: APIRoute = ({ request }) => {
  const url = new URL(request.url);
  const nombre = url.searchParams.get('nombre') || 'Mundo';

  const data = {
    mensaje: `¡Hola, ${nombre}!`,
    timestamp: new Date().toISOString(),
  };

  return new Response(JSON.stringify(data), {
    status: 200,
    headers: { 'Content-Type': 'application/json' },
  });
};
```

**d. Compilar y Desplegar:**

1.  **No necesitas ejecutar un servidor localmente de la misma manera que con Node.** El flujo de trabajo principal es desplegar en Vercel.
2.  Simplemente sube tu código a un repositorio de GitHub/GitLab/Bitbucket y conéctalo a un proyecto de Vercel.
3.  Vercel detectará automáticamente que estás usando el adaptador de Astro, ejecutará `astro build`, y desplegará tu sitio con las funciones serverless configuradas correctamente. Tu API Route estará disponible en `https://tu-sitio.vercel.app/api/saludo.json`.
4.  Para pruebas locales, puedes usar la CLI de Vercel: `vercel dev`.

### Resumen: ¿Cuándo Necesitas un Adaptador?

Necesitas configurar tu proyecto en modo `server` y usar un adaptador siempre que quieras utilizar características dinámicas de Astro, como:

* **API Routes** para crear un backend.
* **Páginas protegidas por contraseña** o sesión de usuario.
* **Contenido que se obtiene de una base de datos en tiempo real** por cada solicitud.
* **Pruebas A/B** o personalización de contenido.

Los adaptadores son la pieza clave que le da a Astro la flexibilidad para pasar de ser un simple generador de sitios estáticos a un framework full-stack capaz de ejecutarse en cualquier entorno de hosting moderno.
