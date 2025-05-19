
## 8. Uso de JavaScript/TypeScript para interactividad

Astro está diseñado para generar HTML estático, pero permite añadir interactividad en el navegador mediante componentes de frameworks como React, Vue, Svelte, Solid, etc., o incluso con JavaScript/TypeScript nativo.

### 8.1 Opciones para agregar dinamismo

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

### 8.2 Buenas prácticas

* Usá `client:*` solo en los componentes que realmente necesitan JS.
* Preferí `client:idle` o `client:visible` antes que `client:load` si no es urgente.
* Evitá escribir lógica muy compleja en `<script>`s si podés encapsularla en componentes.
* Mantené separados los scripts por archivo cuando crezca su tamaño o complejidad.
