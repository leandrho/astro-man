
La forma más común de implementar esto en Astro es usando una combinación de **páginas y componentes Astro** para la interfaz (el frontend) y **rutas de API (API Routes)** para manejar la lógica de la base de datos (el backend).

-----

### \#\# 1. Leer (Read) 📖

Ya cubrimos la lectura de *todos* los libros. Ahora, creemos la página principal que no solo los lista, sino que también incluye enlaces para crear, editar y eliminar.

**`src/pages/biblioteca.astro` (Versión mejorada)**

```astro
---
import { db, Book, Author, eq } from 'astro:db';
import Layout from '../layouts/Layout.astro';
import AddBookForm from '../components/AddBookForm.astro';

// Obtenemos todos los libros con el nombre de su autor
const booksWithAuthors = await db
  .select({
    id: Book.id,
    title: Book.title,
    authorName: Author.name,
  })
  .from(Book)
  .innerJoin(Author, eq(Book.authorId, Author.id))
  .orderBy(Book.id);

// Obtenemos todos los autores para pasarlos al formulario de creación
const allAuthors = await db.select().from(Author);
---

<Layout title="Mi Biblioteca">
  <main>
    <h1>📚 Mi Biblioteca Virtual</h1>

    <AddBookForm authors={allAuthors} />

    <hr class="section-divider">

    <h2>Listado de Libros</h2>
    <ul class="book-list">
      {booksWithAuthors.map((book) => (
        <li class="book-item">
          <span><strong>{book.title}</strong> - {book.authorName}</span>
          <div class="actions">
            <a href={`/libros/${book.id}/editar`} class="edit-btn">Editar</a>

            <form action="/api/libros/delete" method="POST">
              <input type="hidden" name="bookId" value={book.id} />
              <button type="submit" class="delete-btn">Eliminar</button>
            </form>
          </div>
        </li>
      ))}
    </ul>
  </main>
</Layout>

<style>
/* Estilos para una mejor presentación */
main { max-width: 800px; margin: auto; padding: 1.5rem; }
.section-divider { margin: 2rem 0; border: 1px solid #eee; }
.book-list { list-style: none; padding: 0; }
.book-item { display: flex; justify-content: space-between; align-items: center; padding: 0.75rem; border-bottom: 1px solid #ddd; }
.actions { display: flex; gap: 0.5rem; }
.edit-btn, .delete-btn { padding: 0.3rem 0.6rem; border: none; border-radius: 4px; color: white; cursor: pointer; text-decoration: none; font-size: 0.9em; }
.edit-btn { background-color: #007bff; }
.delete-btn { background-color: #dc3545; }
</style>
```

-----

### \#\# 2. Crear (Create) 📝

Para crear un nuevo libro, necesitamos un formulario en el frontend y una ruta de API que reciba los datos y los inserte en la base de datos.

#### **Frontend: El Formulario (`src/components/AddBookForm.astro`)**

Este componente recibe la lista de autores para mostrarlos en un menú desplegable.

```astro
---
import type { Author } from 'astro:db';

interface Props {
  authors: (typeof Author.$inferSelect)[];
}

const { authors } = Astro.props;
---
<h2>Añadir Nuevo Libro</h2>
<form action="/api/libros/create" method="POST" class="form-container">
  <div class="form-group">
    <label for="title">Título</label>
    <input type="text" id="title" name="title" required />
  </div>
  <div class="form-group">
    <label for="publication_year">Año de Publicación</label>
    <input type="number" id="publication_year" name="publication_year" required />
  </div>
  <div class="form-group">
    <label for="authorId">Autor</label>
    <select id="authorId" name="authorId" required>
      <option value="" disabled selected>Selecciona un autor</option>
      {authors.map(author => (
        <option value={author.id}>{author.name}</option>
      ))}
    </select>
  </div>
  <button type="submit" class="submit-btn">Guardar Libro</button>
</form>

<style>
/* Estilos para el formulario */
.form-container { display: flex; flex-direction: column; gap: 1rem; padding: 1.5rem; border: 1px solid #eee; border-radius: 8px; }
.form-group { display: flex; flex-direction: column; }
.form-group label { margin-bottom: 0.5rem; font-weight: bold; }
.form-group input, .form-group select { padding: 0.5rem; border: 1px solid #ccc; border-radius: 4px; }
.submit-btn { padding: 0.75rem; background-color: #28a745; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 1em; }
</style>
```

#### **Backend: La Ruta de API (`src/pages/api/libros/create.ts`)**

Esta ruta procesa los datos del formulario.

```typescript
import type { APIRoute } from 'astro';
import { db, Book } from 'astro:db';

export const POST: APIRoute = async ({ request, redirect }) => {
  const formData = await request.formData();

  const title = formData.get('title')?.toString();
  const publication_year = formData.get('publication_year')?.toString();
  const authorId = formData.get('authorId')?.toString();

  // Validación simple
  if (!title || !publication_year || !authorId) {
    return new Response("Faltan campos requeridos", { status: 400 });
  }

  // Insertar en la base de datos
  await db.insert(Book).values({
    title,
    publication_year: parseInt(publication_year),
    authorId: parseInt(authorId),
  });

  // Redirigir de vuelta a la biblioteca
  return redirect('/biblioteca');
};
```

-----

### \#\# 3. Borrar (Delete) 🗑️

La acción de borrar se inicia desde el botón "Eliminar" en la lista de libros.

#### **Frontend: El Formulario (ya está en `biblioteca.astro`)**

Cada libro tiene su propio mini-formulario con un campo oculto (`hidden`) que contiene su `id` y un botón de envío.

#### **Backend: La Ruta de API (`src/pages/api/libros/delete.ts`)**

```typescript
import type { APIRoute } from 'astro';
import { db, Book, eq } from 'astro:db';

export const POST: APIRoute = async ({ request, redirect }) => {
  const formData = await request.formData();
  const bookId = formData.get('bookId')?.toString();

  if (!bookId) {
    return new Response("ID del libro no proporcionado", { status: 400 });
  }

  // Borrar el libro de la base de datos
  await db.delete(Book).where(eq(Book.id, parseInt(bookId)));

  return redirect('/biblioteca');
};
```

-----

### \#\# 4. Actualizar (Update) 🔄

La actualización es un proceso de dos pasos:

1.  Mostrar una página con un formulario pre-rellenado con los datos del libro a editar.
2.  Procesar el envío de ese formulario para actualizar el registro.

#### **Paso 1: Página de Edición (`src/pages/libros/[id]/editar.astro`)**

Esta es una **ruta dinámica**. Astro usará el `[id]` en la URL para saber qué libro buscar.

```astro
---
import { db, Book, Author, eq } from 'astro:db';
import Layout from '../../../layouts/Layout.astro';

// Obtenemos el ID del libro desde los parámetros de la URL
const { id } = Astro.params;

if (!id) return Astro.redirect('/404');

// Buscamos el libro específico
const book = await db.select().from(Book).where(eq(Book.id, parseInt(id))).get();

// Si el libro no existe, redirigimos
if (!book) return Astro.redirect('/404');

// Obtenemos todos los autores para el selector
const allAuthors = await db.select().from(Author);
---

<Layout title={`Editar: ${book.title}`}>
  <main>
    <h1>🔄 Editando: {book.title}</h1>
    <form action="/api/libros/update" method="POST" class="form-container">
      <input type="hidden" name="bookId" value={book.id} />

      <div class="form-group">
        <label for="title">Título</label>
        <input type="text" id="title" name="title" value={book.title} required />
      </div>

      <div class="form-group">
        <label for="publication_year">Año de Publicación</label>
        <input type="number" id="publication_year" name="publication_year" value={book.publication_year} required />
      </div>

      <div class="form-group">
        <label for="authorId">Autor</label>
        <select id="authorId" name="authorId" required>
          {allAuthors.map(author => (
            <option value={author.id} selected={author.id === book.authorId}>
              {author.name}
            </option>
          ))}
        </select>
      </div>

      <button type="submit" class="submit-btn">Actualizar Libro</button>
    </form>
  </main>
</Layout>

<style is:global>
  /* Importar o duplicar estilos del formulario */
  .form-container { display: flex; flex-direction: column; gap: 1rem; padding: 1.5rem; border: 1px solid #eee; border-radius: 8px; max-width: 600px; margin: 2rem auto; }
  .form-group { display: flex; flex-direction: column; }
  .form-group label { margin-bottom: 0.5rem; font-weight: bold; }
  .form-group input, .form-group select { padding: 0.5rem; border: 1px solid #ccc; border-radius: 4px; }
  .submit-btn { padding: 0.75rem; background-color: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 1em; }
</style>
```

#### **Paso 2: Backend de Actualización (`src/pages/api/libros/update.ts`)**

```typescript
import type { APIRoute } from 'astro';
import { db, Book, eq } from 'astro:db';

export const POST: APIRoute = async ({ request, redirect }) => {
  const formData = await request.formData();

  const bookId = formData.get('bookId')?.toString();
  const title = formData.get('title')?.toString();
  const publication_year = formData.get('publication_year')?.toString();
  const authorId = formData.get('authorId')?.toString();

  if (!bookId || !title || !publication_year || !authorId) {
    return new Response("Faltan campos requeridos", { status: 400 });
  }

  // Actualizar el registro en la base de datos
  await db.update(Book)
    .set({
      title,
      publication_year: parseInt(publication_year),
      authorId: parseInt(authorId),
    })
    .where(eq(Book.id, parseInt(bookId)));

  return redirect('/biblioteca');
};
```

Con estos archivos, tendrás un sistema CRUD completamente funcional para la tabla `Book`, interactuando con la tabla `Author` para mostrar y seleccionar autores.
