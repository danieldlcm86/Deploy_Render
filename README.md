# Render Deploy - SPRING API

## Github Repo

Es necesario crear un nuevo repositorio en GitHub donde vamos a subir todo el proycto de Eclipse.
Pero antes de subirlo hay que modificar algunas cositas en nuestro proyecto Gradle.

## Render

- Entrar al sitio oficial de Render [https://dashboard.render.com]
  - Login con Github
- Presionar el botón New -> Service -> PostgeSQL
- Configurar los parámetros de la base de datos y presionar el botón `Create Database`:

```json
{
  "name": "ProjectName",
  "user": "root"
}
```

5. Presionar el botón `Create Database` y esperar que termine la configuración de la base de datos.
6. En un bloc de notas, guardar la información de la base de datos de Render ubicada en Base de datos -> Info -> Connections. Esta información incluye el `hostname, port, name_database, username y password`
7. La información que obtendremos de Render se guardará dentro de variables específicas de la siguiente manera:

```sh
${PROD_DB_HOSTNAME} Hostname
${PROD_DB_PORT} Port
${PROD_DB_NAME} Database
${PROD_DB_USERNAME} Username
${PROD_DB_PASSWORD} Password
```

## Eclipse

Iniciamos la modificación en Eclipse para el Deploy.

1. En el archivo `build.gradle` eliminar la dependencia de MySQL y agregar la dependencia de PostgreSQL:

```java
implementation 'org.postgresql:postgresql:42.7.1'
```

2. Crear un archivo llamado `Dockerfile`, click derecho sobre la carpeta del Project -> New -> File. Dentro del archivo `Dockerfile` agregamos lo siguiente:

```sh
FROM azul/zulu-openjdk:17-latest
VOLUME /tmp
COPY build/libs/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
EXPOSE 8080
```

3. Modificar el achivo `applications.properties` que se encuentra en la carpeta `src/main/resources` y por seguridad, colocar las variables de entorno de cada dato.

```sh
spring.datasource.url=jdbc:postgresql://${PROD_DB_HOSTNAME}:${PROD_DB_PORT}/${PROD_DB_NAME}
spring.datasource.username=${PROD_DB_USERNAME}
spring.datasource.password=${PROD_DB_PASSWORD}

#Agregamos la configuración de `hibernate.dialect` aunque no es necesaria
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=create

logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type=TRACE
```

_Recuerda refrescar el proyecto de Gradle después de haber creado y modificado los archivos._

4. En las tareas de Gradle (`Gradle Tasks`) seleccionar `build` y doble click en el archivo `build`. Una vez que haya terminado la ejecución, validar en la carpeta del proyecto que los archivos `.jar` fueron creados.

   - Estos archivos `.jar` son los que podremos subir para no exponer las contraseñas.

5. En la carpeta del proyecto ubicar el archivo `.gitignore`, comentar el directorio ``build` y `src/main/**/build/` y guardar

```sh
#build
#!**/src/main/**/build/
```

6. Subir el proyecto al repositorio que creamos.

## Render servicio

1. Presionar el botón `New -> WebService`.
2. Conectar con el repositorio que se acaba de crear en Github.
3. Escribir un nombre para la aplicación.
4. Seleccionar el `Tipo de instancia` como `Free`.
5. Ubicar la sección de `Variables de entorno`, en donde copiaremos las variables de entorno que definimos en el paso 3 de Eclipse, sin incluir los caracteres `$` y `{}`.
   Para ello, presionamos el botón `Add Enviroment Variable` y llenamos con los valores que copiamos en nuestro bloc de notas.
6. Presionar el botón `Create Web Service`
7. Inicia el deploy y esperar a que la aplicación termine de publicarse.
   Para saber si el deploy finalizó con éxito, hay que localizar el mensaje `Your service is live 🎉` en la consola del Dashboard.
8. Inmediamente, comenzar a crear productos utilizando postman, ya que la versión gratuita de Render solo otorga un tiempo limitado de vida del deploy y después entra en suspensión, siendo imposible reactivarlo (a menos que contrates un plan).

- Copiamos la URL que se encuentra en la parte superior del dashboard, la cual tiene dominio `.onrender` y complementamos con el path configurado en spring boot para 'postear' registros desde Postman.

9. Modificar la url del fetch en el frontend con la nueva url que nos proporciona Render.
10. En la carpeta `static` que se encuentra en el directorio `src/main/resources` del proyecto de Spring boot, copiamos el frontend del proyecto.
    _No olvides refrescar el proyecto_
11. En `application.properties` cambiar `create` por `validate` y repetir el paso 4 de Eclipse en el `build`.

```sh
spring.jpa.hibernate.ddl-auto=validate
```

12. Realizar commit y push al repositorio y esperar que termine el deploy.

**En el enlace principal de render `.onder` podemos acceder al fronted.**

