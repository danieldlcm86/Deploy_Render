### Repositorio GitHub
Es necesario crear un nuevo repositorio en GitHub donde vamos a subir todo nuestro archivo de Eclipse, pero antes de subirlo al repo hay que modificar algunas cositas en nuestro proyecto Gradle.

### Render
1. Entrar al sitio oficial de Render [https://dashboard.render.com]
2. Login con Github
3. Seleccionar el botón New -> PostgeSQL
4. Configurar los parámetros de la base de datos y presionar el botón `Create Database`:
```sh
Name: nombreProyecto
User: root
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

### Eclipse
Iniciamos la modificación en Eclipse para el Deploy. 
1. En el archivo `build.gradle` eliminar la dependencia de MySQL y agregar la dependencia de PostgreSQL
```java
//https://mvnrepository.com/artifact/org.postgresql/postgresql
implementation 'org.postgresql:postgresql'
```

2. Crear un archivo llamado `Dockerfile`, click derecho sobre la carpeta del proyecto -> New -> File. Dentro del archivo `Dockerfile` agregamos lo siguiente:
```sh
FROM azul/zulu-openjdk:17-latest
VOLUME /tmp
COPY build/libs/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
EXPOSE 8080
```

3. Modificar el achivo `applications.properties` que se encuentra en la carpeta src/main/resources y por seguridad, colocar las variables de entorno de cada dato. 
```sh
spring.datasource.url=jdbc:postgresql://${PROD_DB_HOSTNAME}:${PROD_DB_PORT}/${PROD_DB_NAME}
spring.datasource.username=${PROD_DB_USERNAME}
spring.datasource.password=${PROD_DB_PASSWORD}

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQL10Dialect
#create, create-drop, validate, update
spring.jpa.hibernate.ddl-auto=update

logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type=TRACE
```
*Recuerda refrescar el proyecto de Gradle después de haber creado y modificado los archivos.

4. En las tareas de Gradle (`Gradle Tasks`) seleccionar `build` y doble click en el archivo `build`. Una vez que haya terminado la ejecución, validar en la carpeta del proyecto que los archivos `.jar` fueron creados.
    - Estos archivos `.jar` son los que podremos subir para no exponer las contraseñas.

5. En la carpeta del proyecto ubicar el archivo `.gitignore` y comentar el directorio build y src/main/**/build/ y guardar
```sh
#build
#!**/src/main/**/build/
```
6. Subir el proyecto al repositorio que creamos.

### Render
1. Seleccionar el botón New -> WebService
2. Conectar con el repositorio que se acaba de crear en Github.
3. Escribir un nombre para la aplicación y ubicar la lista desplegable `Advance`. Dentro de Advance creamos las variables de entorno
4. Crear todas las variables de entorno de Eclipse paso 3, sin incluir los símbolos de $ y {}. Para ello, presionamos el botón `Add Enviroment Variable` y llenamos con los valores que copiamos en nuestro bloc de notas. 
5. Presionar el botón `Create Web Service`
6. Inicia el deploy y esperar a que la aplicación termine de publicarse.
7. Para saber si el deploy finalizó con éxito, hay que localizar el mensaje `Your service is live 🎉` en la consola del Dashboard.
8. Inmediamente, comenzar a crear productos utilizando postman, ya que la versión gratuita de Render solo otorga un tiempo limitado de vida del deploy y después entra en suspensión, siendo imposible reactivarlo (a menos que contrates un plan).
    Copiamos el URL que se encuentra en la parte superior del dashboard, el cual tiene dominio `.onrender` y complementamos con nuestro path configurado en spring boot para 'postear' registros.

9. En la carpeta `static` que se encuentra en el directorio src/main/resources copiar el proyecto del frontend.
10. En `application.properties` cambiar `create` por `validate` y repetir el paso 4 de Eclipse en el build.
10. Realizar commit y push al repositorio y esperar que termine el deploy. En el enlace principal se encuentra el frontend desplegado.