El siguiente es un punto crucial para entender cómo crear experiencias de usuario fluidas con Astro.

Por defecto, cuando navegas entre páginas en un sitio Astro (incluso si usas componentes de framework como React en Islas), cada página es una nueva renderización. Esto significa que:

* El HTML de la nueva página se carga.
* Las Islas en esa nueva página se inicializan e hidratan desde cero.
* **Cualquier estado interno de un componente de Isla (como el `count` en tu `Counter` de React) se perdería y se reiniciaría a su valor inicial.**

Esto se debe a que Astro prioriza el envío de HTML ligero y la hidratación selectiva, y la navegación tradicional entre páginas implica descartar el estado de la página anterior.

---
## Manteniendo el Estado con Transiciones de Vista (View Transitions)

Para resolver esto y permitir que el estado de los componentes persista entre navegaciones, Astro introduce las **Transiciones de Vista (View Transitions)**. Esta es una API experimental del navegador que Astro utiliza (con un fallback para navegadores que no la soportan) para permitir transiciones animadas y, lo más importante para tu pregunta, **la persistencia de elementos (y por lo tanto, el estado de las Islas) entre páginas.**

**¿Cómo funciona con respecto al estado?**

La clave está en la directiva `transition:persist` que puedes aplicar a tus componentes Isla.

1.  **Habilitar View Transitions:**
    Primero, necesitas habilitar las View Transitions en tu proyecto. Generalmente, esto se hace añadiendo el componente `<ViewTransitions />` a tu layout principal (o a las páginas específicas donde las necesites):

    ```astro
    // src/layouts/Layout.astro
    import { ViewTransitions } from 'astro:transitions';

    ---
    // ... tu código de layout ...
    ---
    <html lang="es">
      <head>
        <meta charset="utf-8" />
        <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
        <meta name="viewport" content="width=device-width" />
        <meta name="generator" content={Astro.generator} />
        <title>{Astro.props.title}</title>
        <ViewTransitions /> {/* ¡Aquí se habilita! */}
      </head>
      <body>
        <slot />
      </body>
    </html>
    ```

2.  **Usar `transition:persist` en tu Isla:**
    Ahora, en la página donde usas tu componente `Counter`, puedes añadirle la directiva `transition:persist`:

    ```astro
    ---
    // src/pages/pagina-uno.astro
    import Layout from '../layouts/Layout.astro';
    import Counter from '../components/Counter.jsx'; // Tu componente React
    ---
    <Layout title="Página Uno">
      <h1>Página Uno</h1>
      <p>Este es el contador en la página uno:</p>
      <Counter client:load transition:persist />
      <nav>
        <a href="/pagina-dos">Ir a Página Dos</a>
      </nav>
    </Layout>
    ```

    Y asegúrate de que el mismo componente (o uno que Astro pueda identificar como "el mismo" para la transición) exista en la página de destino, también con `transition:persist`:

    ```astro
    ---
    // src/pages/pagina-dos.astro
    import Layout from '../layouts/Layout.astro';
    import Counter from '../components/Counter.jsx'; // Tu componente React
    ---
    <Layout title="Página Dos">
      <h1>Página Dos</h1>
      <p>Este es el MISMO contador, ahora en la página dos:</p>
      <Counter client:load transition:persist />
      <nav>
        <a href="/pagina-uno">Ir a Página Uno</a>
      </nav>
    </Layout>
    ```

**¿Qué sucede ahora?**

Cuando navegas de `pagina-uno` a `pagina-dos` (o viceversa):

* Astro, gracias a `ViewTransitions` y `transition:persist`, identificará que el componente `Counter` está presente en ambas páginas y está marcado para persistir.
* En lugar de destruir la instancia del DOM y el componente `Counter` de la página uno y crear uno nuevo en la página dos, Astro **intentará mantener viva la instancia existente del componente `Counter` y la moverá (o la "transformará morfológicamente") a su nueva posición en el DOM de la página dos.**
* Como la instancia del componente React (su árbol de DOM y su estado JavaScript asociado) se conserva, **el estado `count` dentro de tu componente `Counter` persistirá.** Si habías hecho clic 5 veces en la página uno, seguirá mostrando 5 en la página dos.

**Consideraciones Importantes:**

* **Identificación del Componente:**
    * Por defecto, Astro puede identificar componentes para `transition:persist` basados en su tipo y orden en el DOM.
    * Si tienes múltiples instancias del mismo tipo de componente en una página y quieres persistir una específica, o si quieres más control, puedes darles un nombre único con `transition:name` a los componentes o a sus contenedores:
        ```astro
        <Counter client:load transition:persist="mi-contador-global" />
        ```
        O en un elemento contenedor:
        ```astro
        <div transition:name="contador-principal" transition:persist>
            <Counter client:load />
        </div>
        ```
    * El componente debe estar presente en **ambas páginas** (la de origen y la de destino de la navegación) y tener la directiva `transition:persist` (y el mismo `transition:name` si se usa) para que el estado persista.
* **Props:** Si el componente persistido recibe props diferentes en la nueva página, React (o el framework que estés usando) re-renderizará el componente con esas nuevas props, pero el estado interno que no depende directamente de las props (como el `useState` de tu contador) debería mantenerse.
* **Directivas de Cliente:** La directiva de cliente (como `client:load`, `client:visible`) sigue siendo importante. El componente necesita estar hidratado para que su estado sea significativo. `transition:persist` funciona en conjunto con estas directivas.
* **No es un Estado Global:** `transition:persist` preserva el estado de una instancia de componente específica durante una transición de página. No es una solución de gestión de estado global para toda la aplicación (para eso, seguirías usando herramientas como Zustand, Redux, Context API de React, etc., y tendrías que pensar en cómo ese estado global interactúa o se inicializa en las islas).

---
## Ejemplo Detallado con el Contador de React

**1. Asegúrate de tener `<ViewTransitions />` en tu `Layout.astro`:**

```astro
// src/layouts/Layout.astro
import { ViewTransitions } from 'astro:transitions';
---
<html>
  <head>
    <title>{Astro.props.title}</title>
    <ViewTransitions />
  </head>
  <body>
    <nav>
      <a href="/">Inicio</a> |
      <a href="/otra-pagina">Otra Página</a>
    </nav>
    <hr />
    <slot />
  </body>
</html>
```

**2. Tu componente `Counter.jsx` (sin cambios):**

```jsx
// src/components/Counter.jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  console.log("Renderizando/Actualizando Counter. Estado actual:", count);

  return (
    <div style={{ border: '1px solid blue', padding: '1em', margin: '1em 0' }}>
      <p>Contador (React): {count}</p>
      <button onClick={() => setCount(count + 1)}>Incrementar</button>
    </div>
  );
}
export default Counter;
```
*He añadido un `console.log` para ver cuándo se renderiza y cuál es su estado.*

**3. Página de Inicio (`src/pages/index.astro`):**

```astro
---
import Layout from '../layouts/Layout.astro';
import Counter from '../components/Counter.jsx';
---
<Layout title="Página de Inicio">
  <h1>Página de Inicio</h1>
  <p>Este contador debería persistir su estado al navegar.</p>
  <Counter client:load transition:persist />
  <p>
    <a href="/otra-pagina">Ir a Otra Página</a>
  </p>
</Layout>
```

**4. Otra Página (`src/pages/otra-pagina.astro`):**

```astro
---
import Layout from '../layouts/Layout.astro';
import Counter from '../components/Counter.jsx';
---
<Layout title="Otra Página">
  <h1>Otra Página</h1>
  <p>Si el estado persistió, el contador debería tener el mismo valor.</p>
  <Counter client:load transition:persist />
  <p>
    <a href="/">Volver a Inicio</a>
  </p>
</Layout>
```

**Funcionamiento Esperado:**

1.  Carga la página de inicio. El contador muestra 0.
2.  Haz clic en el botón "Incrementar" varias veces. Digamos que llega a 5.
3.  Haz clic en el enlace "Ir a Otra Página".
4.  La "Otra Página" carga. Gracias a `ViewTransitions` y `transition:persist`, el componente `Counter` **debería seguir mostrando 5**. En la consola del navegador, no deberías ver el `console.log` indicando una nueva inicialización del estado a 0 para este componente específico (aunque puede haber logs de re-render si React lo considera necesario, pero el estado `count` se mantiene).
5.  Si vuelves a la página de inicio, el estado de 5 seguirá ahí.

Sin `transition:persist`, cada vez que navegas a una nueva página, el contador se reiniciaría a 0 porque se montaría una nueva instancia del componente.

Esta combinación de Astro Islands con View Transitions y `transition:persist` es muy poderosa para crear sitios que se sienten tan fluidos y con estado como las SPAs, pero manteniendo los beneficios de rendimiento de la arquitectura de Astro.
