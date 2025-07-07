Los **Server Actions** (o Acciones de Astro) son una característica poderosa que te permite escribir y llamar funciones de backend directamente desde tus componentes de frontend (`.astro`), eliminando la necesidad de crear rutas de API separadas para manejar la lógica del servidor, como el envío de formularios o la interacción con la base de datos.

En esencia, te permiten **colocar tu lógica de servidor junto al código que la utiliza**, simplificando enormemente el desarrollo.

-----

### \#\# ¿Cómo Funcionan? ⚙️

El funcionamiento se basa en el concepto de **RPC (Remote Procedure Call)** y el principio de **Mejora Progresiva (Progressive Enhancement)**.

1.  **Definición**: Defines una o más funciones asíncronas dentro de un objeto `export const actions = { ... }` en tu archivo `.astro`. Cada función es una "acción". Para mayor seguridad y validación, se usa la utilidad `defineAction`.

2.  **Invocación en el Frontend**: En el mismo archivo, dentro de tu HTML, invocas la acción en un elemento `<form>`. Astro te proporciona un objeto `Astro.locals.actions` que contiene manejadores para tus acciones. Al hacer `<form {...Astro.locals.actions.tuAccion()}>`, Astro automáticamente configura los atributos del formulario (`action`, `method`, etc.) para apuntar a un endpoint único y seguro que él mismo genera.

3.  **El Puente Mágico**: Cuando el formulario se envía, Astro intercepta la petición en el servidor. Gracias al endpoint único, sabe exactamente qué función de acción ejecutar.

4.  **Mejora Progresiva**:

      * **Sin JavaScript**: Si el navegador del usuario no tiene JavaScript habilitado, el formulario funciona como un formulario HTML estándar. Se envía, la página se recarga por completo y el usuario ve el resultado.
      * **Con JavaScript**: Si JavaScript está habilitado, Astro utiliza `fetch` para enviar los datos del formulario en segundo plano. Esto permite una experiencia de usuario mucho más fluida, ya que puedes actualizar partes de la página sin una recarga completa, mostrar notificaciones, etc.

El resultado es un código más limpio, seguro y que funciona para todos los usuarios, independientemente de su navegador.

-----

### \#\# Ejemplo: Añadir un "Me Gusta" a un Libro

Continuemos con el ejemplo de la biblioteca. Vamos a añadir un botón de "Me Gusta" ❤️ a cada libro. Esto actualizará un contador en nuestra base de datos.

#### **Paso 1: Actualizar el Esquema de la Base de Datos**

Primero, necesitamos una columna para guardar los "me gusta".

**`db/config.ts`**

```typescript
import { defineDb, defineTable, column, NOW } from 'astro:db';

const Author = defineTable({ /* ... sin cambios ... */ });

const Book = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    authorId: column.number({ references: () => Author.columns.id }),
    title: column.text(),
    publication_year: column.number(),
    // Añadimos la nueva columna para los "me gusta"
    likes: column.number({ default: 0 }),
    published_at: column.date({ default: NOW }),
  }
});

export default defineDb({
  tables: { Author, Book }
});
```

*No olvides migrar tu base de datos después de cambiar el esquema (`npx astro db push`).*

#### **Paso 2: Definir e Invocar la Acción**

Ahora, en nuestra página principal de la biblioteca, definiremos y usaremos la acción `likeBook`.

**`src/pages/biblioteca.astro`**

```astro
---
import { db, Book, Author, eq, sql } from 'astro:db';
import { defineAction, z } from 'astro:actions';
import Layout from '../layouts/Layout.astro';

// 1. DEFINICIÓN DE LA ACCIÓN
export const actions = {
  likeBook: defineAction({
    // Validamos la entrada usando Zod para más seguridad
    accept: 'form',
    input: z.object({
      bookId: z.coerce.number(), // coerce convierte el string del form a número
    }),
    handler: async ({ bookId }) => {
      try {
        // Actualizamos el contador en la base de datos de forma segura
        await db.update(Book)
          .set({ likes: sql`${Book.likes} + 1` })
          .where(eq(Book.id, bookId));

        return { success: true, bookId };
      } catch (error) {
        console.error(error);
        return { success: false, error: 'No se pudo dar me gusta.' };
      }
    },
  }),
};

// Obtenemos los libros para mostrarlos
const booksWithAuthors = await db
  .select({
    id: Book.id,
    title: Book.title,
    authorName: Author.name,
    likes: Book.likes, // Obtenemos el contador de likes
  })
  .from(Book)
  .innerJoin(Author, eq(Book.authorId, Author.id));

// 2. OBTENEMOS EL MANEJADOR DE LA ACCIÓN
const { likeBook } = Astro.locals.actions;
---

<Layout title="Mi Biblioteca">
  <main>
    <h1>📚 Mi Biblioteca Virtual</h1>

    <ul class="book-list">
      {booksWithAuthors.map((book) => (
        <li class="book-item">
          <span>
            <strong>{book.title}</strong> - {book.authorName}
          </span>
          <div class="actions">
            <form {...likeBook()}>
              <input type="hidden" name="bookId" value={book.id} />
              <button type="submit" class="like-btn">
                ❤️ {book.likes} Me gusta
              </button>
            </form>
          </div>
        </li>
      ))}
    </ul>
  </main>
</Layout>

<style>
/* Estilos similares a los ejemplos anteriores */
main { max-width: 800px; margin: auto; padding: 1.5rem; }
.book-list { list-style: none; padding: 0; }
.book-item { display: flex; justify-content: space-between; align-items: center; padding: 0.75rem; border-bottom: 1px solid #ddd; }
.actions { display: flex; gap: 0.5rem; }
.like-btn { background: #f0f0f0; border: 1px solid #ccc; border-radius: 20px; padding: 0.3rem 0.8rem; cursor: pointer; display: flex; align-items: center; gap: 0.3rem; }
.like-btn:hover { background: #e0e0e0; }
</style>
```

**Explicación del código:**

1.  **`defineAction`**: Creamos la acción `likeBook`. Usamos `z` (Zod) para asegurarnos de que el `bookId` que recibimos del formulario es un número, evitando errores y ataques maliciosos.
2.  **`handler`**: Esta es la función que se ejecuta en el servidor. Recibe el `bookId` ya validado.
3.  **Lógica del handler**: Usamos `db.update` para incrementar el contador `likes`. La expresión `sql`${Book.likes} + 1\`\` es una forma segura de realizar operaciones atómicas en la base de datos.
4.  **`Astro.locals.actions`**: Obtenemos el manejador `likeBook` que Astro nos proporciona.
5.  **`<form {...likeBook()}>`**: Aplicamos las propiedades del manejador directamente al formulario. Astro se encarga del resto. Cada vez que hagas clic en el botón "Me gusta", el formulario se enviará, la acción se ejecutará en el servidor, el contador se actualizará y la página se recargará con el nuevo número de "me gusta".
