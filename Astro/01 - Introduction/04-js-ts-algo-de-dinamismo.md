
## Uso de JavaScript/TypeScript para interactividad

Astro está diseñado para generar HTML estático, pero permite añadir interactividad en el navegador mediante componentes de frameworks como React, Vue, Svelte, Solid, etc., o incluso con JavaScript/TypeScript nativo.

### Opciones para agregar dinamismo

1. **Componentes con frameworks (usando `client:*`)**
   Astro permite integrar componentes de frameworks modernos para manejar interactividad. Ejemplo con React:

   ```jsx
   // src/components/Contador.jsx
   import { useState } from 'react';

   export default function Contador() {
     const [cuenta, setCuenta] = useState(0);
     return (
       <button onClick={() => setCuenta(cuenta + 1)}>
         Contar: {cuenta}
       </button>
     );
   }
   ```

   Y lo usás desde un archivo `.astro` con una directiva:

   ```astro
   ---
   import Contador from '../components/Contador.jsx';
   ---
   <Contador client:load />
   ```

2. **JavaScript/TypeScript nativo (Vanilla)**
   También podés escribir scripts manuales para cosas simples, como toggles, menús, animaciones, etc.

   ```astro
   <button id="toggle">Mostrar/Ocultar</button>
   <div id="contenido" hidden>Hola mundo</div>

   <script type="module">
     const btn = document.getElementById('toggle');
     const contenido = document.getElementById('contenido');
     btn.addEventListener('click', () => {
       contenido.hidden = !contenido.hidden;
     });
   </script>
   ```

   * Si usás TypeScript, podés cambiar `type="module"` por `type="module" lang="ts"` y configurar `tsconfig.json`.

3. **Eventos personalizados**
   Podés comunicar componentes Astro con componentes interactivos usando eventos. Por ejemplo, disparar un evento desde un componente React hacia el DOM.

### Buenas prácticas

* Usá `client:*` solo en los componentes que realmente necesitan JS.
* Preferí `client:idle` o `client:visible` antes que `client:load` si no es urgente.
* Evitá escribir lógica muy compleja en `<script>`s si podés encapsularla en componentes.
* Mantené separados los scripts por archivo cuando crezca su tamaño o complejidad.

---

## Entendiendo JavaScript y la Reactividad en Astro: La Arquitectura de Islas

Astro es un framework web que se ha ganado un lugar en el ecosistema por su enfoque único en el **rendimiento** y la **velocidad de carga**. A diferencia de los Single Page Applications (SPAs) que envían una gran cantidad de JavaScript al navegador, Astro opera bajo un principio conocido como **"Arquitectura de Islas"**. Este enfoque cambia fundamentalmente cómo se maneja JavaScript en tus aplicaciones web.

### La Filosofía de Astro: Menos JavaScript por Defecto

La premisa central de Astro es enviar la menor cantidad de JavaScript posible al navegador. Esto significa que la mayoría de tu sitio se renderiza en el servidor como HTML puro. Tu usuario descarga HTML y CSS optimizado, lo que resulta en una experiencia de carga casi instantánea y un excelente rendimiento de Core Web Vitals, crucial para el SEO.

Pero, ¿qué sucede cuando necesitas interactividad? Aquí es donde entra en juego la gestión inteligente de JavaScript por parte de Astro.

### La Etiqueta `<script>` en Astro: Contextos y Comportamiento

Cuando incluyes una etiqueta `<script>` en un archivo `.astro`, su comportamiento no es tan simple como un script tradicional en un archivo HTML plano. Astro procesa estos scripts de formas específicas, dependiendo de dónde se encuentren y de cómo los declares.

#### 1. Scripts en Componentes `.astro` (Scripts del Lado del Cliente)

Cuando colocas un `<script>` directamente dentro de un componente `.astro` (ya sea un componente re-utilizable o un componente de página), Astro lo trata como un script del lado del cliente.

* **Ejecución Inicial:** Estos scripts se incluyen en el HTML final y se ejecutan **una única vez** en el navegador cuando la página se carga inicialmente y el navegador los parsea.
* **Comportamiento en Navegación (View Transitions):** Astro, por defecto, utiliza **View Transitions** para una navegación fluida entre páginas sin recargar completamente el navegador. En este escenario:
    * Si el script afecta elementos que son parte de la "transición" de Astro, esos elementos (y por ende los scripts que los controlan) no se re-ejecutan automáticamente como si fuera una recarga completa de página. Astro intenta mantener el estado y la interactividad donde sea posible.
    * Para scripts que necesitan inicializarse o re-inicializarse después de una navegación de View Transition, deberás escuchar eventos específicos como `astro:after-swap` en el `document`. Esto te permite adjuntar lógica que se ejecutará cada vez que Astro realice una transición de página, evitando duplicar listeners o comportamientos no deseados.
* **`addEventListener` y Duplicación:** En un script de componente `.astro` (no hidratado), si adjuntas un `addEventListener`, y la página se recarga completamente (por ejemplo, al presionar F5 o navegar a una URL externa), el DOM se destruye y reconstruye. El script se ejecuta nuevamente y adjunta un **nuevo** listener, sin duplicación, ya que el anterior ya no existe. Sin embargo, como mencionamos, con **View Transitions**, debes ser proactivo y manejar la re-ejecución de tu lógica si no quieres que el listener se duplique al "re-aparecer" el elemento en el DOM.

#### 2. Scripts en Componentes de UI (Islas de Interactividad)

La verdadera potencia de Astro para la interactividad reside en su capacidad para integrar **componentes de frameworks de UI populares** como React, Vue, Svelte, o Preact. Cuando usas estos componentes en tus archivos `.astro` y les aplicas una **directiva de cliente** (como `client:load`, `client:idle`, `client:visible`, etc.), estás creando una **"isla de interactividad"**.

* **¿Qué es una "Isla"?** Una isla es una pequeña porción de tu página donde el JavaScript del framework de UI se "hidrata" (es decir, se carga y ejecuta) en el navegador. Fuera de estas islas, el resto de la página sigue siendo HTML puro, sin JavaScript.
* **Ejecución y Control:** Cuando un componente de UI se hidrata:
    * El JavaScript asociado a ese componente (y su lógica de eventos, estado, etc.) se carga y ejecuta **una única vez** en el cliente, bajo el control del framework específico (React, Vue, etc.).
    * **Manejo de `addEventListener`:** Dentro de estos componentes de UI, el framework se encarga de gestionar el ciclo de vida de los listeners de eventos. Ellos se aseguran de que los listeners se adjunten correctamente cuando el componente se monta y se limpien cuando el componente se desmonta, evitando duplicaciones de forma automática.
* **Comportamiento en Navegación (View Transitions):** Cuando navegas usando View Transitions, los componentes de UI ya hidratados **mantienen su estado y su interactividad**. No se "re-hidratan" a menos que haya una recarga completa de la página. Esto contribuye a una experiencia de usuario fluida y persistente.

#### 3. Scripts Globales (`<script>` en el `head` o `body` principal)

Puedes incluir scripts en el `<head>` de tu layout principal o directamente en el `<body>` fuera de cualquier componente de Astro.

* **Ejecución:** Estos scripts se ejecutan en el orden en que aparecen en el HTML, tan pronto como el navegador los encuentra y los parsea.
* **Comportamiento en Navegación (View Transitions):** Este es el escenario donde debes ser más cuidadoso. Si tienes un script global que adjunta `addEventListener` y usas View Transitions, el script global **puede ejecutarse nuevamente** con cada navegación. Si no tomas precauciones, esto **puede llevar a la duplicación de listeners** si no se limpian adecuadamente. Es por ello que, para la lógica interactiva que debe persistir o re-inicializarse con View Transitions, se recomienda:
    * Encapsularla en un componente de UI hidratado.
    * O bien, escuchar el evento `astro:after-swap` y asegurarse de que la lógica de adjuntar listeners sea idempotente (que se pueda ejecutar varias veces sin efectos secundarios negativos, como duplicar listeners).

### Tipos de Scripts y Directivas Importantes

* **`<script>`:** La etiqueta estándar. Astro procesa y "bundlea" este script.
* **`<script is:inline>`:** Astro *no* procesa ni bundlea este script. Lo inserta directamente en el HTML final tal cual. Es útil para scripts muy pequeños que no necesitan optimización o que deben ejecutarse muy temprano en el proceso de carga.
* **`<script type="module">`:** Declara el script como un módulo JavaScript, permitiendo el uso de `import` y `export` dentro del script.
* **`<script type="module" src="...">`:** Carga un módulo de script externo.

### Conclusión: El Paradigma de Astro

Astro te anima a pensar en tu aplicación como un conjunto de componentes estáticos, y solo "activar" (hidratar) las partes que **realmente necesitan JavaScript para la interactividad**. Esta es la clave para entender por qué Astro es tan rápido y cómo gestionar su JavaScript. Al comprender la diferencia entre scripts que se ejecutan una vez en la carga inicial y el comportamiento de las islas interactivas (y cómo View Transitions afecta a ambos), estarás bien equipado para construir sitios web rápidos y robustos con Astro.
