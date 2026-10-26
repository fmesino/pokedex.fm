# Guía de Despliegue: PokeDex en Azure App Service

Este documento detalla el procedimiento paso a paso utilizado para compilar, configurar y desplegar la aplicación web PokeDex en la nube pública de Microsoft Azure. Además, se documenta la configuración de seguridad implementada para obtener la calificación **A+** exigida por los estándares de Pueblo Paleta Inc.

## 1. Preparación y Compilación Local

La aplicación fue desarrollada en Angular, por lo que requirió un proceso de construcción (build) antes de su publicación.

1. Se abrió la terminal de comandos (CMD) en Windows y se navegó hasta la ruta del código fuente:

   ```
   cd "sistemas-distribuidos\poke-dex-lab\source\pokedex-angular"
   
   ```

2. Se instalaron las dependencias necesarias de Node.js:

   ```
   npm install
   
   ```

3. Se generó la versión de producción de la aplicación:

   ```
   npm run build
   
   ```

   *Este comando creó automáticamente la carpeta `dist/pokedex-angular` con los archivos estáticos optimizados y listos para producción.*

## 2. Configuración de Encabezados de Seguridad

Para cumplir con las normativas de seguridad web y proteger la aplicación contra ciberataques, se inyectaron encabezados HTTP. Dado que el despliegue se realizó en un entorno Windows de Azure (IIS), se creó un archivo `web.config` en la raíz de la carpeta `dist/pokedex-angular` con el siguiente contenido:

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <httpProtocol>
      <customHeaders>
        <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains; preload" />
        <add name="X-Frame-Options" value="DENY" />
        <add name="X-Content-Type-Options" value="nosniff" />
        <add name="Referrer-Policy" value="strict-origin-when-cross-origin" />
        <add name="Permissions-Policy" value="geolocation=(), microphone=(), camera=()" />
        <!-- Política estricta para garantizar calificación A+ en Security Headers -->
        <add name="Content-Security-Policy" value="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com data:; connect-src 'self' https://beta.pokeapi.co; img-src 'self' data: https:;" />
      </customHeaders>
    </httpProtocol>
    <rewrite>
      <rules>
        <rule name="Angular Routes" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="./index.html" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>

```

## 3. Empaquetado y Despliegue en Azure

Una vez configurado el servicio App Service en Azure (alojado en la región *Brazil South* para cumplir con la disponibilidad de la suscripción de estudiantes), se procedió a subir los archivos:

1. Se seleccionó todo el contenido del directorio `dist/pokedex-angular` (incluyendo el nuevo `web.config`) y se comprimió en un archivo `.zip`.

2. En el portal de Azure, se navegó al recurso de **App Service**.

3. En el menú lateral, se ingresó a **Herramientas de desarrollo > Herramientas avanzadas** y se abrió el entorno de **Kudu**.

4. En el menú de Kudu, se seleccionó **Tools > Zip Deploy**.

5. Se arrastró y soltó el archivo `.zip` en la interfaz. El servidor extrajo automáticamente los archivos en el directorio `site/wwwroot` y puso la aplicación en línea.

## 4. Verificación y Resultados

Tras finalizar el despliegue:

* La aplicación se encuentra totalmente accesible y funcional desde la URL pública generada por Azure.

* Se realizó una auditoría en **securityheaders.com**. Gracias a la configuración del archivo `web.config`, la aplicación obtuvo exitosamente la máxima calificación (**A+**), cumpliendo íntegramente con los requisitos de la compañía.