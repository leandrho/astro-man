
---

## Layouts y Props en Astro con TypeScript

### ¿Qué es un Layout en Astro?

Un **Layout** es un componente `.astro` reutilizable que envuelve otras páginas o componentes. Sirve para definir una estructura común: encabezados, pies de página, barras laterales, etc.

Es similar a un "template base" donde otras páginas se insertan en una ranura (`<slot />`).

---

### Crear un Layout

Supongamos que quieres que todas tus páginas tengan un header y un footer. Puedes crear un layout así:

📄 `src/layouts/BaseLayout.astro`

```astro
---
interface Props {
  title: string;
}
const { title } = Astro.props;
---

<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <title>{title}</title>
  </head>
  <body>
    <header>
      <h1>{title}</h1>
    </header>

    <main>
      <slot />  <!-- Aquí se inyecta el contenido de la página -->
    </main>

    <footer>
      <p>© 2025 - Mi sitio Astro</p>
    </footer>
  </body>
</html>
```

---

### Usar el Layout en una página

📄 `src/pages/index.astro`

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout title="Inicio">
  <p>Bienvenido a la página principal de Astro.</p>
</BaseLayout>
```

> El contenido dentro de `<BaseLayout>...</BaseLayout>` será inyectado en el `<slot />`.

---

### Tipado de Props con TypeScript

En Astro podés usar TypeScript para definir tipos seguros. Esto se hace dentro del bloque `---` usando una `interface`, como hicimos arriba.

Ejemplo de props con múltiples valores tipados:

```astro
---
interface Props {
  title: string;
  subtitle?: string;  // opcional
  showHeader?: boolean;
}

const { title, subtitle = "", showHeader = true } = Astro.props;
---
```

De esta manera:

* Sabés exactamente qué espera tu componente/layout.
* Tenés autocompletado y chequeo de tipos.
* Evitás errores al usar el componente.

---

### Validar props opcionales con valores por defecto

Podés definir valores por defecto al desestructurar `Astro.props`, como se muestra arriba (`subtitle = ""`, `showHeader = true`). También podés hacerlo así:

```astro
const props = Astro.props;
const title = props.title;
const showHeader = props.showHeader ?? true;
```

---

### Props en otros componentes `.astro`

Lo mismo aplica para componentes que no son layouts:

📄 `src/components/Alert.astro`

```astro
---
interface Props {
  type: 'success' | 'error' | 'warning';
  message: string;
}

const { type, message } = Astro.props;
---

<div class={`alert alert-${type}`}>
  {message}
</div>
```

Y lo usás así:

```astro
<Alert type="success" message="Todo salió bien" />
```

---
