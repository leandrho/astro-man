### ¿Qué es AstroDB?

**AstroDB** es una base de datos SQL totalmente gestionada, diseñada específicamente para el framework de desarrollo web **Astro**. Su objetivo es simplificar el trabajo con bases de datos en proyectos de Astro, proporcionando una experiencia fluida tanto para el desarrollo local como para el despliegue en producción.

A diferencia de las bases de datos tradicionales que requieren una configuración y gestión complejas, AstroDB se integra directamente en tu proyecto de Astro. Utiliza **libSQL**, una bifurcación de SQLite, lo que le permite ser ligera y rápida, ideal para sitios web centrados en el contenido.

Entre sus características principales se encuentran:

  * **Integración Sencilla:** Se añade a un proyecto de Astro con un único comando.
  * **Desarrollo Local:** Crea una base de datos local para un desarrollo rápido y sin necesidad de conexión a internet.
  * **ORM Incluido:** Viene con Drizzle ORM preconfigurado, lo que permite realizar consultas a la base de datos de una manera segura y con tipado estricto en TypeScript.
  * **Astro Studio:** Ofrece una plataforma web para gestionar tus bases de datos alojadas en producción.
  * **Basada en SQL:** Al ser una base de datos SQL, puedes utilizar la sintaxis y los conceptos que ya conoces.

Es importante no confundir AstroDB con otras herramientas de nombres similares como *TkAstroDb* (un programa en Python para astrología) o *astrodbkit* (una librería de Python para bases de datos astronómicas). AstroDB está enfocado exclusivamente en el desarrollo web con el framework Astro.

-----

### ¿Cómo Instalar AstroDB?

La instalación de AstroDB en un proyecto de Astro es un proceso sencillo y automatizado gracias a las herramientas que provee el propio framework. A continuación se detallan los pasos generales:

#### Requisitos Previos

  * Tener un proyecto de Astro existente o crear uno nuevo.
  * Astro v4.5 o superior.
  * Node.js en una versión compatible con tu versión de Astro.

#### Pasos para la Instalación

1.  **Abre tu terminal:** Navega hasta el directorio raíz de tu proyecto de Astro.

2.  **Ejecuta el comando de instalación:** Utiliza la interfaz de línea de comandos (CLI) de Astro para añadir la integración de la base de datos. Este comando se encargará de instalar las dependencias necesarias y de crear los archivos de configuración iniciales.

    ```bash
    npx astro add db
    ```

3.  **Confirma la instalación:** La terminal te hará algunas preguntas para confirmar la instalación y la configuración de los archivos necesarios. Generalmente, solo necesitarás confirmar las acciones propuestas.

4.  **Archivos de Configuración:** Una vez finalizado el proceso, notarás que se han creado nuevos archivos y directorios en tu proyecto:

      * `astro.config.mjs`: Se actualizará para incluir la integración de la base de datos.
      * `db/config.ts`: En este archivo definirás el esquema de tu base de datos, es decir, las tablas y las columnas que la componen.
      * `db/seed.ts`: (Opcional) Este archivo te permite definir datos iniciales (semillas) para tu base de datos, muy útil para el desarrollo y las pruebas.

5.  **Inicia tu servidor de desarrollo:** Al ejecutar `npm run dev` (o el comando que uses para iniciar tu servidor), Astro creará la base de datos local en un archivo dentro del directorio `.astro/`.

Una vez completados estos pasos, ya tendrás AstroDB instalado y configurado en tu proyecto, listo para que empieces a definir tus tablas y a realizar consultas desde tus componentes y páginas de Astro.

-----
