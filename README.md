# Render Deploy - SPRING API

<!--toc:start-->

- [Render Deploy - SPRING API](#render-deploy-spring-api)

  - [Github Repo](#github-repo)
  - [Render](#render)
  - [Eclipse](#eclipse)
    - [Dependencias](#dependencias)
    - [Dockerfile](#dockerfile)
    - [Variables](#variables)
      - [Opcion 1](#opcion-1)
      - [Opcion 2](#opcion-2)
    - [Gradle](#gradle)
    - [gitignore](#gitignore)
  - [Servicio Java](#servicio-java)
    - [Pruebas](#pruebas)
  - [Ajustes finales](#ajustes-finales) - [Front adicional](#front-adicional)

  <!--toc:end-->

## Github Repo

Es necesario crear un nuevo repositorio en GitHub donde vamos a subir todo el proycto de Eclipse.
Pero antes de subirlo hay que modificar algunas cositas en nuestro proyecto Gradle.

## Render

- Entrar al sitio oficial de Render [https://dashboard.render.com]
  - Login con Github
- Presionar el botón New -> Service -> PostgeSQL
- Configurar los parámetros de la base de datos y presionar el botón `Create Database`:

```properties
"name": "ProjectName",
"user": "root"
```

- Presionar el botón `Create Database` y esperar que termine la configuración.
- En un bloc de notas, guardar la información de la base de datos de Render
  - Ubicada en `Database` -> `Info`-> `Connections`.
  - Esta información incluye: `hostname, port, name_database, username` y `password`.
- La información obtenida se guardará dentro de variables de entorno de la siguiente manera:

```properties
${PROD_DB_HOSTNAME}=Hostname
${PROD_DB_PORT}=Port
${PROD_DB_NAME}=Database
${PROD_DB_USERNAME}=Username
${PROD_DB_PASSWORD}=Password
```

## Eclipse

### Dependencias

- En el archivo `build.gradle` eliminar la dependencia de MySQL
- Agregar la dependencia de PostgreSQL:

```*.properties
implementation 'org.postgresql:postgresql:42.7.1'
```

### Dockerfile

- Crear un archivo llamado `Dockerfile`
  - click derecho sobre la carpeta del proyecto:
    - Project -> New -> File.
      -Dentro del archivo `Dockerfile` agregamos lo siguiente:

```docker
FROM azul/zulu-openjdk:17-latest
VOLUME /tmp
COPY build/libs/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
EXPOSE 8080
```

### Variables

#### Opcion 1

- Modificar el achivo `applications.properties`:
  - Se encuentra en la carpeta `src/main/resources`
  - Por seguridad, colocar las variables de entorno de la base de dato

```properties
spring.datasource.url=jdbc:postgresql://${PROD_DB_HOSTNAME}:${PROD_DB_PORT}/${PROD_DB_NAME}
spring.datasource.username=${PROD_DB_USERNAME}
spring.datasource.password=${PROD_DB_PASSWORD}

#Agregamos la configuración de `hibernate.dialect` aunque no es necesaria
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=create

logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type=TRACE
```

#### Opcion 2

- Crear un nuevo archivo `application-prod.properties`
  - Este servirá exclusivamente para configuración de producción.
  - No es necesario modificar `application.properties`.
- Pega las mismas lineas de configuracion que la [Opcion 1](#opcion-1)

> Recuerda refrescar el proyecto de Gradle después creación y modificación.

### Gradle

- En las tareas de Gradle (`Gradle Tasks`) seleccionar `build`.

  - Accede a la carpeta `build` en root del proyecto.
  - Una vez que haya terminado la ejecución
    - validar en la carpeta del proyecto que los archivos `.jar` fueron creados.
  - Estos archivos `.jar` son los que podremos subir para no exponer las contraseñas.

- En la carpeta del proyecto ubicar el archivo `.gitignore`.
  - Comentar el directorio `build` y `src/main/**/build/`.
  - Guarda cambios.

### gitignore

```properties
# build
# !**/src/main/**/build/
```

- Subir el proyecto al repositorio que creamos.

## Servicio Java

- Presionar el botón `New -> WebService`.
- Conectar con el repositorio que se acaba de crear en Github.
  - Escribir un nombre para la aplicación.
- Seleccionar el `Tipo de instancia` como `Free`.
- Ubicar la sección de `Variables de entorno`, en donde copiaremos las variables de entorno que definimos en el paso 3 de Eclipse, sin incluir los caracteres `$` y `{}`.
  - Si se creó un `application-prod.properties`:
  - Agregar una variable de entorno:
  - `SPRING_PROFILES_ACTIVE` con valor: `prod`
- Para ello, presionamos el botón `Add Enviroment Variable`
  - Llenamos con los valores en nuestro bloc de notas.
- Presionar el botón `Create Web Service`
- Inicia el deploy y espera a que la aplicación termine de publicarse.

### Pruebas

Para saber si el deploy finalizó con éxito, hay que localizar el mensaje `Your service is live 🎉` en la consola del Dashboard.

Inmediamente, comenzar a crear datos de las entidades por medio de postman.

> La versión gratuita de Render solo otorga un tiempo limitado de vida del deploy
> y después entra en suspensión, siendo imposible reactivarlo (a menos que contrates un plan).

- Copiamos la URL que se encuentra en la parte superior del dashboard.
- La cual tiene dominio `.onrender`
  - Complementamos con el `endpoint` configurado en spring boot para 'postear' registros.

## Ajustes finales

### Front adicional

1. Modificar la url del fetch en el frontend con la nueva url que nos proporciona Render.
2. Dentro de `/static` que se encuentra en el directorio `src/main/resources` del proyecto:
   2.1 Copiamos el frontend del proyecto.
   > _No olvides refrescar el proyecto_
3. En `application.properties` cambiar `create` por `validate`
   3.1 y repetir el paso 4 de Eclipse en el `build`:

```properties
spring.jpa.hibernate.ddl-auto=validate
```

- Realizar commit y push
- Esperar que termine el deploy.
- **En el enlace principal de `.onrder` podemos acceder al fronted.**

