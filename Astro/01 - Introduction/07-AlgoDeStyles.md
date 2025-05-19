
---

## 10. CSS en Astro: Estilos locales y globales

Astro te permite escribir estilos en varios niveles:

* **Locales** (por componente)
* **Globales** (para toda la app)
* Usar **preprocesadores** (como SCSS)
* Integrar **frameworks de CSS** (como Tailwind o Bootstrap)

Aquí nos enfocamos en los estilos **locales** y **globales** usando CSS puro.

---

### 10.1 Estilos por componente (locales)

Podés escribir estilos directamente dentro de un archivo `.astro`, y por defecto serán **scoped**, es decir, **solo afectarán a ese componente**.

📄 `src/components/Card.astro`

```astro
---
const { title, content } = Astro.props;
---

<style>
  .card {
    border: 1px solid #ccc;
    padding: 1rem;
    border-radius: 8px;
  }

  h2 {
    color: #333;
  }
</style>

<div class="card">
  <h2>{title}</h2>
  <p>{content}</p>
</div>
```

✅ En este caso, el `<style>` está **scopeado automáticamente** al componente. Astro transforma los selectores internamente para evitar conflictos con otros estilos.

---

### 10.2 Estilos globales en un componente

Si querés que un estilo sea global aunque esté dentro de un componente, podés usar el atributo `:global`.

```astro
<style>
  :global(body) {
    margin: 0;
    font-family: system-ui;
  }

  :global(.btn) {
    padding: 0.5rem 1rem;
    background: blue;
    color: white;
  }
</style>
```

> `:global(...)` indica a Astro que no scopee esos estilos, por lo tanto serán aplicados en toda la app.

También podés hacer global solo una parte del selector:

```astro
<style>
  .card :global(h2) {
    color: red;
  }
</style>
```

---

### 10.3 Estilos globales en archivos separados

Para definir estilos que se apliquen a toda la aplicación, podés crear un archivo CSS global (por ejemplo, `src/styles/global.css`) y usarlo en tu layout o en el `src/pages/_app.astro` si lo usás.

📄 `src/styles/global.css`

```css
body {
  background-color: #f4f4f4;
  color: #222;
  font-family: sans-serif;
}

a {
  color: blue;
  text-decoration: none;
}
```

Luego lo importás en tu layout o directamente en `src/pages/index.astro` si querés:

```astro
---
import '../styles/global.css';
---
```

📌 **Nota**: Si lo importás desde un layout, se aplicará a todas las páginas que usen ese layout.

---

### 10.4 Usar archivos CSS separados por componente

Otra opción es tener un archivo `.css` por cada componente:

📄 `src/components/Button.astro`

```astro
---
import './Button.css';
---

<button class="btn">
  <slot />
</button>
```

📄 `src/components/Button.css`

```css
.btn {
  background-color: black;
  color: white;
  padding: 0.5rem 1rem;
}
```

⚠️ Si hacés esto, los estilos **no estarán scopeados automáticamente**, por lo que pueden afectar otras partes si no usás nombres únicos o metodologías como BEM.

---

### 10.5 Buenas prácticas

* Usá estilos scopeados (`<style>` en `.astro`) siempre que sea posible para evitar conflictos.
* Usá `:global(...)` solo para resets, estilos base o clases reutilizables (como `.btn`, `.container`).
* Organizá los estilos globales en archivos separados (`global.css`, `reset.css`).
* Podés usar preprocesadores como SCSS si lo necesitás con una integración.

---
