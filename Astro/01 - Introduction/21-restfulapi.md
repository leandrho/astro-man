Es importante aclarar que Astro no *es* una API RESTful, sino que **provee las herramientas para construir una API RESTful de manera muy eficiente y natural** a través de sus **API Routes**.

Vamos a desglosar primero qué es una API RESTful y luego cómo las características de Astro se alinean perfectamente para construir una, con un ejemplo completo.

---
### ¿Qué es una API RESTful?

**REST (Representational State Transfer)** es un estilo de arquitectura de software para diseñar aplicaciones en red. No es un estándar estricto, sino un conjunto de principios y restricciones. Una API que sigue estos principios se considera **RESTful**.

Los principios clave son:

1.  **Arquitectura Cliente-Servidor:** El cliente (frontend) y el servidor (backend) están separados. Se comunican a través de una red. Astro lo logra separando las páginas (`.astro`) del frontend y las Rutas de API (`.ts`/`.js`) del backend.
2.  **Sin Estado (Stateless):** Cada solicitud del cliente al servidor debe contener toda la información que el servidor necesita para entenderla y procesarla. El servidor no almacena ningún estado del cliente entre solicitudes. Las API Routes de Astro son funciones sin estado por naturaleza.
3.  **Interfaz Uniforme:** Este es el corazón de REST y se compone de cuatro restricciones:
    * **Identificación de Recursos (URIs):** Cada recurso (un usuario, un producto, un post) tiene un identificador único, que es su URL (o URI). En Astro, esto se mapea directamente al **enrutamiento basado en archivos**. `src/pages/api/productos/123` es el identificador del producto con ID 123.
    * **Manipulación a través de Representaciones:** Los clientes no interactúan con el recurso directamente, sino con una "representación" de él, comúnmente en formato **JSON** o XML.
    * **Uso de Métodos HTTP Estándar:** Usas los verbos HTTP para indicar la acción que quieres realizar sobre un recurso:
        * `GET`: Leer un recurso.
        * `POST`: Crear un nuevo recurso.
        * `PUT`: Reemplazar/actualizar un recurso existente.
        * `PATCH`: Actualizar parcialmente un recurso.
        * `DELETE`: Eliminar un recurso.
    * **Mensajes Autodescriptivos:** Las respuestas deben ser claras, usando **códigos de estado HTTP** correctos (como `200 OK`, `201 Created`, `404 Not Found`) y cabeceras (`Content-Type: application/json`) para describir el contenido.

---
### ¿Cómo se Construye una API RESTful con Astro?

Astro implementa los principios REST de la siguiente manera:

| Principio RESTful | Implementación en Astro |
| :--- | :--- |
| **Identificación de Recursos (URIs)** | **Enrutamiento basado en archivos** en `src/pages/`. La ruta del archivo define la URL del recurso. Ej: `src/pages/api/productos/[id].ts` se convierte en `/api/productos/:id`. |
| **Métodos HTTP** | **Exportación de funciones** con el nombre del verbo HTTP (`GET`, `POST`, `PUT`, `DELETE`) desde un archivo de API Route. |
| **Representaciones (JSON)** | Se crean y devuelven objetos `Response` estándar, usando `JSON.stringify()` para el cuerpo y estableciendo la cabecera `Content-Type: application/json`. |
| **Códigos de Estado HTTP** | Se especifican en el objeto `Response` con la propiedad `status`. Ej: `{ status: 200 }`. |
| **Sin Estado** | Cada llamada a una función (`GET`, `POST`, etc.) es una invocación independiente que recibe toda la información que necesita a través del objeto `APIContext` (`request`, `params`). |

---
### Ejemplo Completo: API RESTful para "Productos"

Vamos a construir una API RESTful completa para gestionar productos.

**Recurso:** Producto (`id`, `nombre`, `precio`, `descripcion`)
**URL Base:** `/api/productos`

**Operaciones (CRUD):**
* `GET /api/productos`: Obtener todos los productos.
* `POST /api/products`: Crear un nuevo producto.
* `GET /api/productos/[id]`: Obtener un producto por su ID.
* `PUT /api/productos/[id]`: Actualizar un producto por su ID.
* `DELETE /api/productos/[id]`: Eliminar un producto por su ID.

#### 1. Datos de Ejemplo (Simulando una Base de Datos)

```typescript
// src/data/productos.ts
export interface Producto {
  id: number;
  nombre: string;
  precio: number;
  descripcion: string;
}

// Nuestra "base de datos" en memoria.
export let productos: Producto[] = [
  { id: 1, nombre: "Laptop AstroBook Pro", precio: 1200, descripcion: "Potente laptop para desarrolladores." },
  { id: 2, nombre: "Mouse Ergonómico", precio: 45, descripcion: "Diseñado para largas horas de trabajo." },
  { id: 3, nombre: "Teclado Mecánico RGB", precio: 95, descripcion: "Teclas silenciosas y luces personalizables." },
];
```

#### 2. Endpoints para la Colección (`/api/productos`)

Este archivo manejará la obtención de todos los productos y la creación de uno nuevo.

```typescript
// src/pages/api/productos/index.ts
import type { APIRoute } from 'astro';
import { productos, type Producto } from '../../../data/productos';

// GET: Obtener todos los productos
export const GET: APIRoute = () => {
  return new Response(JSON.stringify(productos), {
    status: 200,
    headers: { 'Content-Type': 'application/json' },
  });
};

// POST: Crear un nuevo producto
export const POST: APIRoute = async ({ request }) => {
  try {
    const body = await request.json();
    if (!body.nombre || !body.precio) {
      return new Response(JSON.stringify({ error: 'Nombre y precio son requeridos.' }), { status: 400 });
    }

    const nuevoProducto: Producto = {
      id: Math.max(...productos.map(p => p.id), 0) + 1,
      nombre: body.nombre,
      precio: body.precio,
      descripcion: body.descripcion || '',
    };
    productos.push(nuevoProducto);

    return new Response(JSON.stringify(nuevoProducto), {
      status: 201, // 201 Created
      headers: { 'Content-Type': 'application/json' },
    });
  } catch (e) {
    return new Response(JSON.stringify({ error: 'Solicitud inválida.' }), { status: 400 });
  }
};
```

#### 3. Endpoints para un Recurso Específico (`/api/productos/[id]`)

Este archivo con un nombre de ruta dinámico manejará las operaciones sobre un único producto.

```typescript
// src/pages/api/productos/[id].ts
import type { APIRoute } from 'astro';
import { productos, type Producto } from '../../../data/productos';

// GET: Obtener un producto por ID
export const GET: APIRoute = ({ params }) => {
  const id = parseInt(params.id!, 10);
  const producto = productos.find(p => p.id === id);

  if (!producto) {
    return new Response(JSON.stringify({ error: 'Producto no encontrado.' }), { status: 404 });
  }
  return new Response(JSON.stringify(producto), { status: 200 });
};

// PUT: Actualizar un producto por ID (reemplazo completo)
export const PUT: APIRoute = async ({ params, request }) => {
  const id = parseInt(params.id!, 10);
  const productoIndex = productos.findIndex(p => p.id === id);

  if (productoIndex === -1) {
    return new Response(JSON.stringify({ error: 'Producto no encontrado.' }), { status: 404 });
  }

  try {
    const body = await request.json();
    if (!body.nombre || !body.precio) {
      return new Response(JSON.stringify({ error: 'Nombre y precio son requeridos.' }), { status: 400 });
    }

    const productoActualizado: Producto = { ...productos[productoIndex], ...body };
    productos[productoIndex] = productoActualizado;

    return new Response(JSON.stringify(productoActualizado), { status: 200 });
  } catch (e) {
    return new Response(JSON.stringify({ error: 'Solicitud inválida.' }), { status: 400 });
  }
};

// DELETE: Eliminar un producto por ID
export const DELETE: APIRoute = ({ params }) => {
  const id = parseInt(params.id!, 10);
  const productoIndex = productos.findIndex(p => p.id === id);

  if (productoIndex === -1) {
    return new Response(JSON.stringify({ error: 'Producto no encontrado.' }), { status: 404 });
  }

  productos.splice(productoIndex, 1);

  // Una respuesta DELETE exitosa no debe tener cuerpo.
  return new Response(null, { status: 204 }); // 204 No Content
};

// PATCH podría implementarse de manera similar a PUT, pero solo actualizando los campos presentes.
```

### Resumen del Flujo RESTful en el Ejemplo

1.  **Recursos y URIs:**
    * La colección de productos es `/api/productos`.
    * Un producto individual es `/api/productos/1`, `/api/productos/2`, etc.

2.  **Verbos HTTP:**
    * Para leer, usas `GET`.
    * Para crear, usas `POST`.
    * Para actualizar, usas `PUT`.
    * Para eliminar, usas `DELETE`.

3.  **Códigos de Estado:**
    * `200 OK` para solicitudes `GET` y `PUT` exitosas.
    * `201 Created` para un `POST` exitoso.
    * `204 No Content` para un `DELETE` exitoso.
    * `404 Not Found` si el ID no existe.
    * `400 Bad Request` si los datos enviados son incorrectos.

4.  **Representación:**
    * Toda la comunicación se realiza con representaciones de los productos en formato **JSON**.

En conclusión, Astro te ofrece un marco de trabajo minimalista pero extremadamente poderoso para construir APIs RESTful. Su enrutamiento basado en archivos y la exportación directa de métodos HTTP hacen que el código sea intuitivo, organizado y se alinee perfectamente con los principios de diseño de REST.
