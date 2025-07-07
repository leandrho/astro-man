
Turso y cómo se integra con AstroDB.

### ¿Qué es Turso?

**Turso** es una plataforma de base de datos distribuida, basada en **libSQL**, una bifurcación de código abierto de SQLite. Piensa en Turso como el **backend de producción para tu AstroDB**.

  * **AstroDB local**: Cuando trabajas en tu computadora (`npm run dev`), AstroDB crea un archivo de base de datos local (un archivo `.db` en tu proyecto) para que desarrolles de forma rápida y sin conexión.
  * **Turso en producción**: Cuando despliegas tu sitio web, no puedes usar ese archivo local. Turso te proporciona una **base de datos remota y distribuida** donde vivirá tu información. Esto asegura que tu sitio web sea rápido para usuarios de todo el mundo, ya que Turso puede replicar tu base de datos en diferentes ubicaciones geográficas.

En resumen, **Turso es el servicio que aloja tu AstroDB en la nube**, permitiendo que tu aplicación Astro funcione en un entorno de producción real.

-----

### ¿Cómo se usa Turso con AstroDB?

Usar Turso implica crear una base de datos en su plataforma, obtener las credenciales y conectarla a tu proyecto a través de **Astro Studio**, el panel de control de Astro.

Aquí tienes un ejemplo paso a paso:

#### Paso 1: Instalar la CLI de Turso

Primero, necesitas la herramienta de línea de comandos de Turso para crear y gestionar tus bases de datos.

```bash
# En macOS/Linux con Homebrew
brew install tursodatabase/tap/turso

# O con el script de instalación
curl -sSfL https://get.tur.so/install.sh | bash
```

#### Paso 2: Autenticarse y Crear una Base de Datos

Una vez instalada la CLI, inicia sesión en tu cuenta de Turso (te pedirá que te registres si no tienes una) y crea una nueva base de datos.

```bash
# Inicia sesión en tu cuenta de Turso
turso auth login

# Crea una nueva base de datos (p. ej., 'mi-biblioteca')
turso db create mi-biblioteca
```

#### Paso 3: Obtener la URL y el Token de Autenticación

Necesitarás dos piezas de información de tu base de datos Turso para conectarla con Astro: la **URL** y un **token de autenticación**.

```bash
# Muestra la URL de tu base de datos
turso db show mi-biblioteca

# Crea un token de autenticación para tu base de datos
turso db tokens create mi-biblioteca
```

Copia ambos valores, los necesitarás en el siguiente paso.

#### Paso 4: Conectar la Base de Datos en Astro Studio

Astro Studio es la plataforma que une tu proyecto de Astro con servicios en la nube como Turso.

1.  Ve a [Astro Studio](https://studio.astro.build) y crea un nuevo proyecto.
2.  Dentro de tu proyecto en Studio, ve a la pestaña **"Settings"** (Configuración).
3.  Busca la sección **"Database"** (Base de datos) y haz clic en **"Connect a DB"** (Conectar una BD).
4.  Selecciona la opción para conectar una base de datos **Turso existente**.
5.  Pega la **URL** y el **Token de autenticación** que obtuviste en el paso anterior.

#### Paso 5: Vincular tu Proyecto Local con Astro Studio

Finalmente, vincula tu proyecto de Astro local al proyecto que acabas de configurar en Astro Studio. Esto permite que tu proyecto local sepa qué base de datos remota usar cuando se despliegue.

En la terminal, dentro del directorio de tu proyecto Astro, ejecuta:

```bash
npx astro link
```

Este comando te guiará para conectar tu repositorio local con tu proyecto de Astro Studio.

-----

### Ejemplo de Uso

Una vez que has completado la conexión, **tu código no cambia en absoluto**. Esta es la mayor ventaja de la integración.

El código que usas para consultar la base de datos es el mismo, ya sea en desarrollo local o en producción. AstroDB se encarga de la magia por debajo.

Por ejemplo, esta consulta en una de tus páginas Astro funcionará perfectamente tanto en local como en producción con Turso:

**`src/pages/index.astro`**

```astro
---
import { db, Book, Author, eq } from 'astro:db';

// Esta misma consulta funciona en local (contra el archivo .db)
// y en producción (contra tu base de datos Turso)
const latestBooks = await db
  .select({
    title: Book.title,
    authorName: Author.name
  })
  .from(Book)
  .innerJoin(Author, eq(Book.authorId, Author.id))
  .orderBy(Book.published_at)
  .limit(5);
---

<h1>Últimos Libros Añadidos</h1>
<ul>
  {latestBooks.map(book => (
    <li>{book.title} por {book.authorName}</li>
  ))}
</ul>
```

Cuando ejecutas `npm run dev`, AstroDB consulta el archivo local. Cuando tu sitio se despliega en una plataforma como Vercel o Netlify, Astro usará las credenciales de Astro Studio para conectar y consultar tu base de datos **Turso** remota.
