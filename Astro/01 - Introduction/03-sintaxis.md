
# Sintaxis de Astro

Astro utiliza una sintaxis propia en los archivos `.astro`, que combina HTML con bloques de JavaScript/TypeScript. La sintaxis está diseñada para ser familiar, especialmente si has trabajado con frameworks como React, Vue o Svelte.

### Estructura básica de un archivo `.astro`

```astro
---
// Bloque de script en la parte superior
const nombre = "Astro";
---

<!-- HTML estándar con soporte para expresiones -->
<html>
  <head>
    <title>Hola {nombre}</title>
  </head>
  <body>
    <h1>Bienvenido a {nombre}</h1>
  </body>
</html>
```

### El bloque `---`

Todo lo que se coloca entre las líneas `---` al inicio del archivo es JavaScript (o TypeScript si lo habilitas). Aquí puedes importar componentes, definir variables, funciones, etc.

```astro
---
import Layout from '../layouts/Base.astro';
const saludo = "Hola Mundo";
---
```

### Interpolación de variables

Puedes insertar variables dentro del HTML con llaves `{}`:

```astro
<h2>{saludo}</h2>
```

### Componentes

Puedes usar componentes `.astro` o de otros frameworks:

```astro
---
import Card from '../components/Card.astro';
import Counter from '../components/Counter.jsx';
---

<Card title="Ejemplo" />
<Counter client:load />
```

### Directivas de cliente

Permiten controlar cuándo se carga un componente con interactividad (normalmente de React, Vue, etc.). Algunas directivas comunes:

* `client:load`: carga el componente en cuanto la página carga.
* `client:idle`: espera hasta que el navegador esté inactivo.
* `client:visible`: carga cuando el componente entra en el viewport.
* `client:only="react"`: carga solo en el cliente, usando el framework especificado.

### Condicionales

Astro permite condicionales simples dentro del HTML:

```astro
---
const logueado = true;
---

{logueado ? <p>Bienvenido</p> : <a href="/login">Iniciar sesión</a>}
```

### Iteraciones

Puedes usar `.map()` para iterar sobre listas:

```astro
---
const productos = ["Pelota", "Zapatillas", "Camiseta"];
---

<ul>
  {productos.map(item => <li>{item}</li>)}
</ul>
```

### Slots

Los componentes `.astro` pueden tener slots para contenido dinámico:

```astro
<!-- En el componente padre -->
<Card>
  <h2>Contenido dentro del slot</h2>
</Card>

<!-- En el componente Card.astro -->
<div class="card">
  <slot />
</div>
```

---
