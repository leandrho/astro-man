
## Estructura de un proyecto Astro

Cuando creas un proyecto con `npm create astro@latest`, Astro genera una estructura de carpetas y archivos similar a la siguiente:

```
my-astro-project/
├── astro.config.mjs        # Configuración principal de Astro
├── package.json            # Dependencias y scripts de npm
├── tsconfig.json           # (Opcional) Configuración de TypeScript
├── public/                 # Archivos estáticos servidos directamente (favicon, imágenes, robots.txt...)
├── src/
│   ├── components/         # Componentes reutilizables (.astro, .jsx, .svelte, etc.)
│   ├── layouts/            # Plantillas o layouts para envolver páginas
│   ├── pages/              # Páginas de la aplicación (ruta automapeada)
│   ├── content/            # (Opcional) Markdown, MDX u otras fuentes de contenido
│   ├── styles/             # Archivos CSS globales o de utilidades
│   └── assets/             # Recursos (imágenes, fuentes, íconos) que procesará Vite
├── .gitignore              # Archivos y carpetas que Git debe ignorar
└── README.md               # Guía rápida del proyecto
```

### Explicación de archivos raíz

* **astro.config.mjs**: aquí defines las integraciones (plugins), rutas de contenido, personalizaciones de build, adapters para deploy y más.
* **package.json**: listas las dependencias (`dependencies`, `devDependencies`) y defines scripts útiles (`dev`, `build`, `preview`).
* **tsconfig.json**: si usas TypeScript, configura los paths, target, strictness, etc.

### Carpetas clave

* **public/**: todo lo que pongas aquí se copia tal cual al build final. Ideal para favicons, `robots.txt`, y archivos que no necesitan procesamiento.
* **src/components/**: componentes aislados que pueden recibir props y generar marcación. Ej: botones, cards, formularios.
* **src/layouts/**: wrappers generales para páginas. Permiten evitar duplicar encabezados, pies de página y estilos comunes.
* **src/pages/**: cada archivo o directorio aquí corresponde a una ruta en tu sitio. Ej: `src/pages/index.astro` → `/`, `src/pages/blog/[slug].astro` → ruta dinámica.
* **src/content/**: si gestionas contenido Markdown o MDX, aquí van tus archivos de contenido. Astro ofrecerá acceso a estos archivos en tiempo de build.
* **src/styles/**: puedes incluir CSS global o variables CSS personalizadas. También aquí van archivos Tailwind u otros preprocesadores.
* **src/assets/**: recursos importables desde tus componentes. Astro utiliza Vite, por lo que al importar imágenes o fuentes, se optimizan automáticamente.

### Flujo de trabajo interno

1. **Desarrollo (`npm run dev`)**: Astro arranca un servidor local con recarga en caliente via Vite.
2. **Build (`npm run build`)**: se generan los archivos estáticos optimizados.
3. **Preview (`npm run preview`)**: simula el servidor de producción para verificar el build.

---

