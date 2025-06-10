Los **API Routes** (o Rutas de API) en Astro son una de las características más potentes para añadir funcionalidades de backend a tu sitio web. Te permiten crear endpoints de servidor que pueden manejar lógica, conectarse a bases de datos, procesar formularios, autenticar usuarios, y mucho más, sin necesidad de renderizar una interfaz de usuario en HTML.

En esencia, son la forma en que Astro te permite construir una API RESTful o cualquier otro tipo de servicio web directamente dentro de tu proyecto.

---
### ¿Qué son y Cómo Funcionan los API Routes?

A diferencia de los componentes de página `.astro` que generan HTML, los API Routes son archivos `.js` o `.ts` que se colocan en la carpeta `src/pages/`. La ubicación del archivo determina la URL del endpoint, siguiendo las mismas reglas de enrutamiento basadas en archivos que las páginas normales.

**Características Clave:**

1.  **Ejecución en el Servidor:** Su código se ejecuta exclusivamente en el servidor (o en un entorno de funciones serverless/edge), nunca en el navegador del cliente.
2.  **Exportación de Métodos HTTP:** En lugar de código HTML, exportas funciones que corresponden a los métodos HTTP que quieres manejar: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, etc.
3.  **Contexto de la API (`APIContext`):** Cada una de estas funciones recibe un objeto de contexto (`APIContext`) que contiene información y utilidades sobre la solicitud entrante.
4.  **Retorno de una `Response` Estándar:** La función debe devolver un objeto `Response` estándar de la Web API. Esto te da control total sobre el código de estado, las cabeceras y el cuerpo de la respuesta.
5.  **Requisito de Despliegue:** Para que los API Routes funcionen, tu sitio debe estar desplegado en un entorno que soporte renderizado del lado del servidor (SSR) o funciones de servidor. Esto significa usar un [adaptador de Astro](https://docs.astro.build/es/guides/integrations-guide/node/) (como Node.js, Vercel, Netlify, Cloudflare, etc.). No funcionan en un despliegue puramente estático.

---
### El Objeto de Contexto (`APIContext`)

Este es el único argumento que reciben tus funciones de método HTTP y es tu puerta de entrada a toda la información de la solicitud. Sus propiedades más importantes son:

* `request`: El objeto `Request` estándar. Contiene toda la información de la solicitud entrante, como las cabeceras (`request.headers`), el método, la URL y, lo más importante, el cuerpo (`await request.json()` o `await request.formData()`).
* `params`: Un objeto que contiene los valores de los segmentos dinámicos de la ruta. Por ejemplo, para una ruta `src/pages/api/productos/[id].ts`, si se accede a `/api/productos/123`, `params` será `{ id: '123' }`.
* `cookies`: Una utilidad para leer (`cookies.get('nombre')`) y establecer (`cookies.set('nombre', 'valor', { ...options })`) cookies.
* `locals`: Un objeto para compartir datos entre el middleware y el endpoint de la API. Muy útil para pasar información del usuario autenticado.
* `redirect`: Una función de ayuda para devolver una respuesta de redirección (ej: `return redirect('/login', 307)`).

---
## Ejemplo Detallado: Creando una API de "Tareas"

Vamos a crear una pequeña API REST para gestionar una lista de tareas. La API tendrá los siguientes endpoints:

* `GET /api/tareas`: Devuelve todas las tareas.
* `GET /api/tareas/[id]`: Devuelve una tarea específica por su ID.
* `POST /api/tareas`: Crea una nueva tarea.

**1. Preparar los Datos (Mock)**

Para mantener el ejemplo simple, usaremos un array en memoria en lugar de una base de datos real.

```typescript
// src/data/tareas.ts
// Este archivo simulará nuestra base de datos.
export interface Tarea {
  id: number;
  texto: string;
  completada: boolean;
}

export const tareas: Tarea[] = [
  { id: 1, texto: "Aprender sobre API Routes en Astro", completada: true },
  { id: 2, texto: "Crear un ejemplo práctico", completada: false },
  { id: 3, texto: "Desplegar el proyecto en un entorno SSR", completada: false },
];
```

**2. Crear el Endpoint `GET` y `POST` para la Colección**

Este archivo manejará las solicitudes a `/api/tareas`.

```typescript
// src/pages/api/tareas/index.ts
import type { APIRoute } from 'astro';
import { tareas, type Tarea } from '../../../data/tareas';

// Maneja solicitudes GET a /api/tareas
export const GET: APIRoute = ({ request }) => {
  // Devolvemos la lista completa de tareas en formato JSON
  return new Response(JSON.stringify(tareas), {
    status: 200,
    headers: {
      'Content-Type': 'application/json',
    },
  });
};

// Maneja solicitudes POST a /api/tareas
export const POST: APIRoute = async ({ request }) => {
  try {
    const nuevaTareaData = await request.json();

    // Validación simple
    if (!nuevaTareaData.texto || typeof nuevaTareaData.texto !== 'string') {
      return new Response(JSON.stringify({ error: 'El campo "texto" es requerido y debe ser un string.' }), { status: 400 });
    }

    // Creamos la nueva tarea
    const nuevaTarea: Tarea = {
      id: Math.max(...tareas.map(t => t.id), 0) + 1, // Generamos un nuevo ID
      texto: nuevaTareaData.texto,
      completada: false,
    };

    // La "guardamos" en nuestra base de datos en memoria
    tareas.push(nuevaTarea);

    // Devolvemos la nueva tarea creada con un estado 201 Created
    return new Response(JSON.stringify(nuevaTarea), {
      status: 201, // 201 Created
      headers: {
        'Content-Type': 'application/json',
      },
    });
  } catch (error) {
    // Si el cuerpo de la solicitud no es un JSON válido
    return new Response(JSON.stringify({ error: 'Cuerpo de la solicitud inválido.' }), { status: 400 });
  }
};
```

**3. Crear el Endpoint `GET` para un Recurso Específico**

Este archivo manejará las solicitudes a `/api/tareas/[id]`, donde `[id]` es un segmento dinámico.

```typescript
// src/pages/api/tareas/[id].ts
import type { APIRoute } from 'astro';
import { tareas } from '../../../data/tareas';

// Maneja solicitudes GET a /api/tareas/[id]
export const GET: APIRoute = ({ params, request }) => {
  // `params.id` contiene el valor del segmento [id] de la URL
  const id = parseInt(params.id ?? '', 10);

  if (isNaN(id)) {
    return new Response(JSON.stringify({ error: 'ID inválido.' }), { status: 400 });
  }

  // Buscamos la tarea en nuestra "base de datos"
  const tarea = tareas.find(t => t.id === id);

  // Si no se encuentra la tarea, devolvemos un 404 Not Found
  if (!tarea) {
    return new Response(JSON.stringify({ error: 'Tarea no encontrada.' }), {
      status: 404,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  // Si se encuentra, la devolvemos como JSON
  return new Response(JSON.stringify(tarea), {
    status: 200,
    headers: {
      'Content-Type': 'application/json',
    },
  });
};
```

**4. Cómo Consumir la API desde el Frontend**

Ahora puedes usar `fetch` desde el lado del cliente en cualquier página `.astro` para interactuar con tu nueva API.

```astro
---
// src/pages/index.astro
import Layout from '../layouts/Layout.astro';
---
<Layout title="Gestor de Tareas">
  <main>
    <h1>Lista de Tareas</h1>
    <ul id="lista-tareas">
      <li>Cargando...</li>
    </ul>

    <h2>Añadir Nueva Tarea</h2>
    <form id="form-nueva-tarea">
      <input type="text" name="texto" placeholder="Nueva tarea..." required />
      <button type="submit">Añadir</button>
    </form>
  </main>
</Layout>

<script>
  // Script del lado del cliente para interactuar con la API

  // Función para obtener y mostrar todas las tareas
  async function cargarTareas() {
    const response = await fetch('/api/tareas');
    const tareas = await response.json();
    const lista = document.getElementById('lista-tareas');
    if (lista) {
      lista.innerHTML = tareas.map(tarea =>
        `<li style="${tarea.completada ? 'text-decoration: line-through;' : ''}">
           ${tarea.id}: ${tarea.texto}
         </li>`
      ).join('');
    }
  }

  // Función para manejar el envío del formulario
  async function handleFormSubmit(event) {
    event.preventDefault();
    const form = event.target;
    const formData = new FormData(form);
    const texto = formData.get('texto');

    const response = await fetch('/api/tareas', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ texto }),
    });

    if (response.ok) {
      form.reset(); // Limpiar el formulario
      await cargarTareas(); // Recargar la lista de tareas
    } else {
      const errorData = await response.json();
      alert(`Error: ${errorData.error}`);
    }
  }

  // Cargar las tareas iniciales al cargar la página
  document.addEventListener('DOMContentLoaded', () => {
    cargarTareas();
    const form = document.getElementById('form-nueva-tarea');
    form?.addEventListener('submit', handleFormSubmit);
  });
</script>
```

Con esta estructura, has creado una API completamente funcional dentro de tu proyecto Astro, separando limpiamente la lógica del servidor de la presentación del cliente, pero manteniendo todo dentro del mismo codebase.
