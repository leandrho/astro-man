
Imaginemos que queremos construir una pequeña biblioteca virtual. Un `Autor` puede escribir muchos `Libros`, estableciendo una relación de uno a muchos.

-----

### \#\# 1. Definición del Esquema de la Base de Datos

Primero, definimos la estructura de nuestras tablas. Esto se hace en el archivo `db/config.ts`. Usaremos las utilidades que nos provee Drizzle ORM, que viene integrado con AstroDB.

En `db/config.ts`, define las tablas `Author` y `Book`:

```typescript
import { defineDb, defineTable, column, NOW } from 'astro:db';

// Define la tabla de Autores
const Author = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    name: column.text(),
    birth_year: column.number({ optional: true }), // Año de nacimiento del autor
  }
});

// Define la tabla de Libros
const Book = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    authorId: column.number({ references: () => Author.columns.id }), // Clave foránea que referencia al autor
    title: column.text(),
    publication_year: column.number(),
    published_at: column.date({ default: NOW }), // Fecha de publicación con valor por defecto
  }
});

// https://astro.build/db/config
export default defineDb({
  tables: { Author, Book }
});
```

**Explicación:**

  * `defineTable`: Crea una nueva tabla en la base de datos.
  * `column`: Define las columnas de cada tabla con sus tipos de datos (`number`, `text`, `date`).
  * `primaryKey: true`: Marca la columna `id` como la clave primaria única.
  * `references`: En la tabla `Book`, la columna `authorId` hace referencia a la columna `id` de la tabla `Author`. Esto crea la **relación** entre ambas.

-----

### \#\# 2. Ingresando Datos de Ejemplo (Seeding)

Para tener datos con los que trabajar, podemos usar el archivo `db/seed.ts`. Este script se ejecuta para poblar la base de datos, lo cual es ideal para el desarrollo.

Agrega lo siguiente en `db/seed.ts`:

```typescript
import { db, Author, Book } from 'astro:db';

// https://astro.build/db/seed
export default async function seed() {
  // Inserta datos en la tabla de Autores
  await db.insert(Author).values([
    { id: 1, name: 'Gabriel García Márquez', birth_year: 1927 },
    { id: 2, name: 'Jorge Luis Borges', birth_year: 1899 },
    { id: 3, name: 'Julio Cortázar', birth_year: 1914 },
  ]);

  // Inserta datos en la tabla de Libros, asociándolos a los autores
  await db.insert(Book).values([
    { id: 1, authorId: 1, title: 'Cien años de soledad', publication_year: 1967 },
    { id: 2, authorId: 1, title: 'El amor en los tiempos del cólera', publication_year: 1985 },
    { id: 3, authorId: 2, title: 'Ficciones', publication_year: 1944 },
    { id: 4, authorId: 2, title: 'El Aleph', publication_year: 1949 },
    { id: 5, authorId: 3, title: 'Rayuela', publication_year: 1963 },
  ]);
}
```

**Explicación:**

  * Importamos `db` y nuestras tablas `Author` y `Book`.
  * Usamos `db.insert(TableName).values([...])` para insertar arreglos de objetos. Cada objeto representa una fila en la tabla.
  * Es crucial que los `authorId` en la tabla `Book` coincidan con los `id` de la tabla `Author`.

Para ejecutar este "seeder", abre tu terminal y corre el siguiente comando:

```bash
npx astro db seed
```

-----

### \#\# 3. Consultando y Mostrando los Datos en una Página Astro

Ahora que tenemos la base de datos definida y con datos, vamos a consultarla desde un componente o una página de Astro. Crearemos una página en `src/pages/biblioteca.astro` para mostrar todos los libros junto con el nombre de su autor.

```astro
---
import { db, Book, Author, eq } from 'astro:db';
import Layout from '../layouts/Layout.astro';

// Realizamos una consulta para unir las tablas Book y Author.
// Queremos obtener todos los libros y el nombre del autor correspondiente.
const booksWithAuthors = await db
  .select({
    title: Book.title,
    publication_year: Book.publication_year,
    authorName: Author.name,
  })
  .from(Book)
  .innerJoin(Author, eq(Book.authorId, Author.id));
---

<Layout title="Mi Biblioteca">
  <main>
    <h1>📚 Mi Biblioteca Virtual</h1>
    <p>Una colección de grandes obras literarias.</p>

    <div class="book-grid">
      {booksWithAuthors.map((book) => (
        <div class="book-card">
          <h2>{book.title}</h2>
          <p><strong>Autor:</strong> {book.authorName}</p>
          <p><em>Publicado en: {book.publication_year}</em></p>
        </div>
      ))}
    </div>
  </main>
</Layout>

<style>
  main {
    margin: auto;
    padding: 1.5rem;
    max-width: 80ch;
  }
  h1 {
    font-size: 2.5rem;
    font-weight: 700;
  }
  .book-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 1rem;
    margin-top: 2rem;
  }
  .book-card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 1rem;
    background-color: #f9f9f9;
  }
  .book-card h2 {
    font-size: 1.25rem;
    margin-top: 0;
  }
</style>
```

**Explicación de la consulta:**

  * `import { db, Book, Author, eq } from 'astro:db';`: Importamos las herramientas necesarias. `eq` es una función de Drizzle que significa "equals" (igual a), y se usa para definir la condición de la unión.
  * `db.select({...})`: Indicamos qué columnas queremos seleccionar. Para evitar conflictos de nombres, renombramos `Author.name` a `authorName`.
  * `.from(Book)`: Especificamos que la consulta principal parte de la tabla `Book`.
  * `.innerJoin(Author, eq(Book.authorId, Author.id))`: Aquí ocurre la magia. Unimos `Book` con `Author` donde el `authorId` del libro sea **igual** al `id` del autor. Esto nos permite acceder a los datos de ambas tablas en una sola consulta.

Al visitar `/biblioteca` en tu navegador, verás una lista de tarjetas, cada una con el título del libro, el año de publicación y, lo más importante, el nombre del autor obtenido desde la tabla `Author`.
