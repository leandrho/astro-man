
# Alias de rutas

Configurar "path aliases" (alias de rutas) en TypeScript te permite crear atajos o nombres más amigables para las rutas de importación de tus módulos. Esto es especialmente útil en proyectos grandes para evitar rutas relativas largas y confusas como `../../../../components/Button` y en su lugar usar algo como `@components/Button`.

Se configuran principalmente a través del archivo `tsconfig.json` en tu proyecto TypeScript. Aquí te explico en detalle cómo hacerlo:

---

## Configuración en `tsconfig.json`

Los dos campos principales que necesitas configurar en la sección `compilerOptions` de tu `tsconfig.json` son:

1.  **`baseUrl`**:
    * Este campo le indica al compilador de TypeScript dónde encontrar los módulos.
    * Normalmente, se establece en `"."` (el directorio raíz de tu proyecto, donde se encuentra `tsconfig.json`) o `"src"` (si todo tu código fuente está dentro de una carpeta `src`).
    * **Es un requisito para que `paths` funcione correctamente.**

2.  **`paths`**:
    * Este campo es un objeto donde defines tus alias.
    * Cada clave en el objeto `paths` es el patrón del alias que quieres usar (por ejemplo, `"@components/*"`).
    * El valor asociado a cada clave es un array de rutas físicas a las que ese alias debe resolver. Usualmente, solo necesitas una ruta por alias.
    * El `*` actúa como un comodín que captura cualquier cosa que siga al alias.

---

## Ejemplo Práctico

Supongamos que la estructura de tu proyecto es la siguiente:

```
mi-proyecto/
├── src/
│   ├── components/
│   │   └── Button.tsx
│   ├── utils/
│   │   └── helpers.ts
│   ├── services/
│   │   └── userService.ts
│   └── App.tsx
├── tsconfig.json
└── package.json
```

Y quieres crear los siguientes alias:

* `@components/*` para que apunte a `src/components/*`
* `@utils/*` para que apunte a `src/utils/*`
* `@services/*` para que apunte a `src/services/*`
* `@app/*` para que apunte a `src/*` (para archivos directamente en `src`)

Tu `tsconfig.json` se vería así:

```json
{
  "compilerOptions": {
    // --- Opciones básicas del compilador ---
    "target": "esnext",             // O la versión de ECMAScript a la que compilas
    "module": "esnext",             // O el sistema de módulos que uses (commonjs, etc.)
    "jsx": "react-jsx",             // Si usas JSX (React, Preact, etc.)
    "strict": true,                 // Habilita todas las comprobaciones estrictas de tipos
    "esModuleInterop": true,        // Permite la interoperabilidad con módulos CommonJS
    "skipLibCheck": true,           // Omite la comprobación de tipos de los archivos de declaración
    "forceConsistentCasingInFileNames": true, // Asegura la consistencia en el case de los nombres de archivo

    // --- Configuración de Path Aliases ---
    "baseUrl": "./",                // Directorio base para resolver los módulos no relativos.
                                    // Si todo tu código está en 'src', podrías poner "./src".
                                    // En este ejemplo, usamos "./" porque los paths comienzan desde la raíz.

    "paths": {
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@services/*": ["src/services/*"],
      "@app/*": ["src/*"]
      // También podrías tener alias para archivos específicos:
      // "@config": ["src/config.ts"]
    },

    // --- Otras opciones ---
    "moduleResolution": "node",     // Cómo TypeScript resuelve los módulos. 'node' es común.
    "resolveJsonModule": true,      // Permite importar archivos .json como módulos
    "isolatedModules": true,        // Asegura que cada archivo pueda ser transpilado sin depender de otras importaciones
    "noEmit": true                  // Si otro bundler (como Webpack, Parcel, Vite) se encarga de la emisión de archivos JS
                                    // Si TypeScript es el que compila a JS, ponlo en false o quítalo.
  },
  "include": ["src"]                // Especifica qué archivos TypeScript debe incluir el compilador.
                                    // "src/**/*" también es común.
}
```

**Explicación Detallada de `baseUrl` y `paths`:**

* **`"baseUrl": "./"`**:
    * Le dice a TypeScript que cuando vea una importación que no sea relativa (que no comience con `./` o `../`), debe buscarla comenzando desde el directorio raíz del proyecto (donde está `tsconfig.json`).
    * Si hubieras puesto `"baseUrl": "./src"`, entonces los `paths` deberían definirse relativos a `src`. Por ejemplo, `@components/*` sería `["components/*"]` en lugar de `["src/components/*"]`. Elegir `./` o `./src` depende de tu preferencia y estructura. Usar `./` es a menudo más claro cuando los `paths` incluyen `src/`.

* **`"paths": { ... }`**:
    * **`"@components/*": ["src/components/*"]`**:
        * Cuando TypeScript encuentre una importación como `import Button from '@components/Button';`
        * El `*` en `@components/*` captura `Button`.
        * TypeScript reemplazará el alias y el comodín para formar la ruta `src/components/Button`.
    * **`"@app/*": ["src/*"]`**:
        * Permite importar archivos directamente desde la carpeta `src` con un alias, por ejemplo: `import { someFunction } from '@app/someModule';` que resolvería a `src/someModule.ts`.

---

## Uso en el Código

Con la configuración anterior, ahora puedes importar módulos usando los alias:

```typescript
// En src/App.tsx

import Button from '@components/Button'; // Resuelve a src/components/Button.tsx
import { helperFunction } from '@utils/helpers'; // Resuelve a src/utils/helpers.ts
import { getUser } from '@services/userService'; // Resuelve a src/services/userService.ts

function App() {
  helperFunction();
  getUser('1');
  return (
    <div>
      <h1>Mi Aplicación</h1>
      <Button />
    </div>
  );
}

export default App;
```

---

## Consideraciones Adicionales

1.  **Herramientas de Bundling (Webpack, Vite, Parcel, Rollup, etc.)**:
    * El `tsconfig.json` le dice al *compilador de TypeScript* cómo resolver estos alias. Sin embargo, si estás usando un bundler para empaquetar tu código para el navegador o Node.js, **ese bundler también necesita entender estos alias**.
    * **Vite**: Usualmente lee `tsconfig.json` automáticamente para los alias a través de plugins como `@vitejs/plugin-react` (si usas React) o configurando `resolve.alias` en `vite.config.ts` basándose en tu `tsconfig.json`.
    * **Webpack**: Necesitarás configurar `resolve.alias` en tu `webpack.config.js`. Puedes usar paquetes como `tsconfig-paths-webpack-plugin` para leer automáticamente la configuración de `paths` desde `tsconfig.json`.
    * **Parcel**: Generalmente tiene un buen soporte para `tsconfig.json` `paths` de forma nativa.
    * **Rollup**: Puedes usar plugins como `@rollup/plugin-alias` y `rollup-plugin-typescript2` que puede leer los `paths`.
    * **Jest / Vitest (Testing)**: Tus frameworks de testing también necesitarán ser configurados para entender estos alias.
        * **Jest**: Configura la opción `moduleNameMapper` en `jest.config.js`.
        * **Vitest**: Usualmente funciona bien si Vite está configurado correctamente, ya que se basa en la configuración de Vite.

    *Ejemplo para `moduleNameMapper` en `jest.config.js` (o `package.json` si usas Jest ahí):*
    ```javascript
    // jest.config.js
    module.exports = {
      // ... otras configuraciones de Jest
      moduleNameMapper: {
        '^@components/(.*)$': '<rootDir>/src/components/$1',
        '^@utils/(.*)$': '<rootDir>/src/utils/$1',
        '^@services/(.*)$': '<rootDir>/src/services/$1',
        '^@app/(.*)$': '<rootDir>/src/$1',
      },
    };
    ```

2.  **Importaciones Relativas vs. Alias**:
    * Sigue siendo bueno usar importaciones relativas (`./MyComponent`) para archivos dentro del mismo módulo o directorio muy cercano.
    * Los alias son más beneficiosos para cruzar límites de módulos o directorios más lejanos.

3.  **Nombres de Alias**:
    * Usar el prefijo `@` es una convención común (ej. `@components`, `@utils`) porque indica claramente que no es un paquete de `node_modules` ni una ruta relativa.

4.  **Reiniciar el Servidor de TypeScript/IDE**:
    * Después de modificar `tsconfig.json`, es posible que necesites reiniciar tu editor de código (VS Code, WebStorm, etc.) o el servidor de TypeScript para que los cambios surtan efecto y el autocompletado funcione correctamente. VS Code usualmente te pide recargar la ventana de TypeScript.

---

Configurar path aliases puede mejorar significativamente la legibilidad y mantenibilidad de tus importaciones en proyectos TypeScript a medida que crecen. ¡Solo asegúrate de que tanto TypeScript como tus herramientas de construcción/testing estén al tanto de ellos!
