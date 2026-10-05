# Guía completa: Conectar una interfaz web con MySQL usando Java + Spring Boot + JPA

## 1. Objetivo de la práctica

El objetivo de esta práctica es construir desde cero una aplicación
pequeña que permita entender, de forma práctica, cómo una página web
termina obteniendo información almacenada en una base de datos.

La arquitectura final será:

``` text
┌──────────────────────┐
│      FRONTEND        │
│    HTML + CSS + JS   │
└──────────┬───────────┘
           │
           │ HTTP / JSON
           │ fetch()
           ▼
┌──────────────────────┐
│       BACKEND        │
│    Spring Boot       │
│       REST API       │
└──────────┬───────────┘
           │
           │ JPA / Hibernate
           ▼
┌──────────────────────┐
│       MySQL          │
│    conexion_db       │
└──────────────────────┘
```

La práctica termina mostrando en una interfaz web los usuarios que
originalmente están almacenados en MySQL.

------------------------------------------------------------------------

# 2. Qué conocimientos se necesitan antes de empezar

Para esta práctica no es necesario dominar Spring Boot ni las APIs.

Sí conviene saber:

-   Crear una base de datos.
-   Crear tablas en MySQL.
-   Hacer consultas SQL básicas.
-   Tener Java instalado.
-   Saber utilizar VS Code.
-   Entender de manera básica qué es una API y qué es HTTP.

La práctica sirve precisamente para aprender la conexión entre estos
conceptos.

------------------------------------------------------------------------

# 3. Tecnologías utilizadas

## Backend

-   Java 21
-   Spring Boot
-   Spring Web
-   Spring Data JPA
-   Hibernate
-   Maven
-   MySQL Connector/J

## Base de datos

-   MySQL Server 8
-   MySQL Workbench es opcional

## Frontend

-   HTML
-   CSS
-   JavaScript
-   `fetch()`
-   Live Server para ejecutar la página durante la práctica

------------------------------------------------------------------------

# 4. Comprobación inicial de Java

Antes de comenzar, comprobar Java desde la terminal:

``` bash
java -version
```

En la práctica se utilizó Java 21 LTS.

Ejemplo:

``` text
openjdk version "21.0.12.1" 2026-08-18 LTS
```

Lo importante es que Java esté correctamente instalado y disponible
desde la terminal.

------------------------------------------------------------------------

# 5. Comprobación de MySQL

Inicialmente el comando:

``` bash
mysql --version
```

mostraba:

``` text
bash: mysql: command not found
```

Aunque MySQL sí estaba instalado en Windows.

La instalación estaba ubicada en:

``` text
C:\Program Files\MySQL\MySQL Server 8.0\bin
```

y allí estaba:

``` text
mysql.exe
```

Esto demuestra que tener el programa instalado y tener su ejecutable
disponible en el `PATH` de la terminal son cosas diferentes.

Finalmente se pudo ejecutar:

``` bash
mysql -u root -p
```

y apareció:

``` text
Welcome to the MySQL monitor.
Server version: 8.0.46 MySQL Community Server
```

Para salir del monitor de MySQL:

``` sql
exit;
```

o:

``` sql
quit;
```

------------------------------------------------------------------------

# 6. Crear la base de datos

La base de datos utilizada se llamó:

``` text
conexion_db
```

Desde MySQL:

``` sql
CREATE DATABASE conexion_db;
```

Después:

``` sql
USE conexion_db;
```

------------------------------------------------------------------------

# 7. Crear la tabla usuarios

La tabla utilizada fue:

``` sql
CREATE TABLE usuarios (
    idUsuario INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    correo VARCHAR(100)
);
```

La estructura final es:

``` text
usuarios
├── idUsuario
├── nombre
└── correo
```

Se agregaron dos registros:

``` sql
INSERT INTO usuarios (nombre, correo)
VALUES
('Juan Pérez', 'juan@gmail.com'),
('María López', 'maria@gmail.com');
```

Para comprobar:

``` sql
SELECT * FROM usuarios;
```

Resultado:

``` text
+-----------+-------------+-----------------+
| idUsuario | nombre      | correo          |
+-----------+-------------+-----------------+
|         1 | Juan Pérez  | juan@gmail.com  |
|         2 | María López | maria@gmail.com |
+-----------+-------------+-----------------+
```

------------------------------------------------------------------------

# 8. Comprobar la estructura real de la tabla

Una comprobación muy importante durante el debugging fue:

``` sql
DESCRIBE usuarios;
```

Resultado:

``` text
+-----------+--------------+------+-----+---------+----------------+
| Field     | Type         | Null | Key | Default | Extra          |
+-----------+--------------+------+-----+---------+----------------+
| idUsuario | int          | NO   | PRI | NULL    | auto_increment |
| nombre    | varchar(100) | YES  |     | NULL    |                |
| correo    | varchar(100) | YES  |     | NULL    |                |
+-----------+--------------+------+-----+---------+----------------+
```

Esto confirmó que la columna se llama exactamente:

``` text
idUsuario
```

y no:

``` text
id_usuario
```

Esta diferencia será muy importante posteriormente.

------------------------------------------------------------------------

# 9. Crear el proyecto Spring Boot

Se creó un proyecto Maven/Spring Boot llamado:

``` text
conexion-db
```

La idea es que este proyecto sea nuestro backend.

La estructura básica será:

``` text
conexion-db/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/
│       │       └── ejemplo/
│       │           └── conexiondb/
│       └── resources/
│           └── application.properties
├── pom.xml
└── ...
```

Dentro del paquete principal tendremos:

``` text
com.ejemplo.conexiondb
├── ConexionDbApplication.java
├── controller/
├── model/
└── repository/
```

------------------------------------------------------------------------

# 10. Dependencias necesarias

El proyecto necesita principalmente:

-   Spring Web
-   Spring Data JPA
-   MySQL Driver

La dependencia de MySQL permite que Java pueda comunicarse con MySQL.

Spring Data JPA permite trabajar con repositorios y entidades.

Spring Web permite crear endpoints REST.

------------------------------------------------------------------------

# 11. Configurar la conexión con MySQL

Archivo:

``` text
src/main/resources/application.properties
```

Configuración utilizada:

``` properties
# Nombre de la aplicación
spring.application.name=conexion-db

# Mi base de datos está en este computador, en el puerto 3306
# y el nombre de la base de datos es conexion_db
spring.datasource.url=jdbc:mysql://localhost:3306/conexion_db

# Usuario y contraseña de la base de datos
spring.datasource.username=root
spring.datasource.password=TU_CONTRASEÑA

# Driver de la base de datos
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Configuración de JPA:
# Spring no debe modificar o crear nuestras tablas automáticamente
spring.jpa.hibernate.ddl-auto=none

# Mostrar las consultas SQL en la consola
spring.jpa.show-sql=true

# Mantener exactamente los nombres de columnas definidos
# en las entidades
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

## Importante sobre la contraseña

Nunca se debe compartir públicamente la contraseña real de MySQL.

Para una práctica local se puede utilizar directamente en
`application.properties`, pero en proyectos reales se recomienda
utilizar variables de entorno o una configuración segura.

------------------------------------------------------------------------

# 12. Qué significa la URL de conexión

Esta línea:

``` properties
spring.datasource.url=jdbc:mysql://localhost:3306/conexion_db
```

se puede dividir:

``` text
jdbc:mysql://localhost:3306/conexion_db
│    │       │         │
│    │       │         └── Base de datos
│    │       └──────────── Puerto
│    └──────────────────── Motor
└───────────────────────── Java Database Connectivity
```

Por lo tanto:

``` text
localhost
```

significa que MySQL está en el mismo computador.

``` text
3306
```

es el puerto habitual de MySQL.

``` text
conexion_db
```

es la base de datos a la que queremos conectarnos.

------------------------------------------------------------------------

# 13. Primer error: DataSource

Inicialmente Spring Boot mostró:

``` text
Failed to configure a DataSource:
'url' attribute is not specified
```

y:

``` text
Failed to determine a suitable driver class
```

Esto ocurría porque todavía no estaban configuradas correctamente las
propiedades de conexión a MySQL y/o el driver.

La solución fue configurar:

``` properties
spring.datasource.url=jdbc:mysql://localhost:3306/conexion_db
spring.datasource.username=root
spring.datasource.password=TU_CONTRASEÑA
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

Después Spring Boot pudo iniciar correctamente.

------------------------------------------------------------------------

# 14. Confirmar que Spring Boot inicia

Una ejecución correcta mostró:

``` text
Tomcat started on port 8080
```

y:

``` text
Started ConexionDbApplication
```

También apareció:

``` text
Initialized JPA EntityManagerFactory
```

Esto confirma que:

-   Spring Boot arrancó.
-   Tomcat está funcionando.
-   JPA/Hibernate se inicializó.
-   La configuración de persistencia está disponible.

El servidor queda disponible en:

``` text
http://localhost:8080
```

------------------------------------------------------------------------

# 15. Crear la entidad Usuario

Archivo:

``` text
src/main/java/com/ejemplo/conexiondb/model/Usuario.java
```

Código:

``` java
package com.ejemplo.conexiondb.model;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import jakarta.persistence.Column;

@Entity
@Table(name = "usuarios")
public class Usuario {

    @Id
    @Column(name = "idUsuario")
    private Integer idUsuario;

    private String nombre;

    private String correo;

    public Integer getIdUsuario() {
        return idUsuario;
    }

    public void setIdUsuario(Integer idUsuario) {
        this.idUsuario = idUsuario;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public String getCorreo() {
        return correo;
    }

    public void setCorreo(String correo) {
        this.correo = correo;
    }
}
```

------------------------------------------------------------------------

# 16. Explicación de Usuario.java

## `@Entity`

``` java
@Entity
```

Le dice a JPA que esta clase representa una entidad que se relacionará
con la base de datos.

------------------------------------------------------------------------

## `@Table`

``` java
@Table(name = "usuarios")
```

Indica que la entidad corresponde a la tabla:

``` text
usuarios
```

------------------------------------------------------------------------

## `@Id`

``` java
@Id
```

Indica cuál es la clave primaria.

En nuestro caso:

``` text
idUsuario
```

------------------------------------------------------------------------

## `@Column`

``` java
@Column(name = "idUsuario")
```

Esta línea fue fundamental.

El atributo Java se llama:

``` java
idUsuario
```

y la columna MySQL también se llama:

``` text
idUsuario
```

Al especificarlo explícitamente evitamos que Hibernate aplique una
conversión de nombres que pudiera transformar:

``` text
idUsuario
```

en:

``` text
id_usuario
```

------------------------------------------------------------------------

# 17. Segundo error: archivo `usuario.java`

En un momento apareció:

``` text
class Usuario is public, should be declared in a file named Usuario.java
```

El archivo se llamaba:

``` text
usuario.java
```

pero la clase era:

``` java
public class Usuario
```

En Java, una clase pública debe estar en un archivo con exactamente el
mismo nombre, respetando mayúsculas y minúsculas.

Correcto:

``` text
Usuario.java
```

Incorrecto:

``` text
usuario.java
```

La solución fue renombrar el archivo.

------------------------------------------------------------------------

# 18. Crear el Repository

Archivo:

``` text
src/main/java/com/ejemplo/conexiondb/repository/UsuarioRepository.java
```

Código:

``` java
package com.ejemplo.conexiondb.repository;

import com.ejemplo.conexiondb.model.Usuario;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UsuarioRepository extends JpaRepository<Usuario, Integer> {
}
```

------------------------------------------------------------------------

# 19. ¿Qué hace el Repository?

El Repository es la capa que nos permite trabajar con los datos.

Al extender:

``` java
JpaRepository<Usuario, Integer>
```

Spring Data JPA proporciona métodos como:

``` java
findAll()
findById()
save()
deleteById()
```

En nuestra primera operación usamos:

``` java
findAll()
```

que significa:

> Obtener todos los usuarios.

No necesitamos escribir manualmente:

``` sql
SELECT * FROM usuarios;
```

para esta operación.

JPA/Hibernate genera la consulta.

------------------------------------------------------------------------

# 20. Tercer error: `id_usuario`

Al probar la API apareció:

``` text
Unknown column 'u1_0.id_usuario' in 'field list'
```

Hibernate estaba generando:

``` sql
select u1_0.id_usuario,
       u1_0.correo,
       u1_0.nombre
from usuarios u1_0;
```

Pero nuestra tabla tenía:

``` text
idUsuario
```

no:

``` text
id_usuario
```

Se comprobó la tabla con:

``` sql
DESCRIBE usuarios;
```

y se confirmó:

``` text
idUsuario
```

También se revisó que `Usuario.java` tuviera:

``` java
@Column(name = "idUsuario")
```

Finalmente se agregó:

``` properties
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

y se ejecutó:

``` bash
./mvnw clean spring-boot:run
```

La consulta finalmente apareció como:

``` sql
Hibernate: select u1_0.idUsuario,u1_0.correo,u1_0.nombre from usuarios u1_0
```

Esto confirmó que Hibernate estaba utilizando el nombre correcto.

------------------------------------------------------------------------

# 21. Crear el Controller

Archivo:

``` text
src/main/java/com/ejemplo/conexiondb/controller/UsuarioController.java
```

Código:

``` java
package com.ejemplo.conexiondb.controller;

import com.ejemplo.conexiondb.model.Usuario;
import com.ejemplo.conexiondb.repository.UsuarioRepository;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/usuarios")
@CrossOrigin(origins = "http://127.0.0.1:5500")
public class UsuarioController {

    private final UsuarioRepository usuarioRepository;

    public UsuarioController(UsuarioRepository usuarioRepository) {
        this.usuarioRepository = usuarioRepository;
    }

    @GetMapping
    public List<Usuario> obtenerUsuarios() {
        return usuarioRepository.findAll();
    }
}
```

------------------------------------------------------------------------

# 22. Explicación del Controller

## `@RestController`

``` java
@RestController
```

Indica que esta clase manejará peticiones HTTP y devolverá datos,
normalmente en formato JSON.

------------------------------------------------------------------------

## `@RequestMapping`

``` java
@RequestMapping("/usuarios")
```

Define la ruta base.

------------------------------------------------------------------------

## `@GetMapping`

``` java
@GetMapping
```

Indica que el método responderá a una petición:

``` http
GET /usuarios
```

------------------------------------------------------------------------

## Método

``` java
public List<Usuario> obtenerUsuarios() {
    return usuarioRepository.findAll();
}
```

Aquí sucede:

``` text
Controller
    ↓
Repository
    ↓
findAll()
    ↓
JPA/Hibernate
    ↓
MySQL
```

El resultado es una lista de objetos `Usuario`.

Spring Boot convierte automáticamente esos objetos a JSON para la
respuesta HTTP.

------------------------------------------------------------------------

# 23. Primera prueba de la API

Con Spring Boot ejecutándose:

``` bash
./mvnw spring-boot:run
```

se puede abrir:

``` text
http://localhost:8080/usuarios
```

El resultado fue:

``` json
[
  {
    "correo": "juan@gmail.com",
    "idUsuario": 1,
    "nombre": "Juan Pérez"
  },
  {
    "correo": "maria@gmail.com",
    "idUsuario": 2,
    "nombre": "María López"
  }
]
```

Esto demuestra que la API ya podía:

1.  Recibir una petición.
2.  Consultar MySQL.
3.  Obtener los usuarios.
4.  Convertirlos a JSON.
5.  Devolverlos al navegador.

------------------------------------------------------------------------

# 24. Arquitectura hasta este punto

``` text
Navegador
    │
    │ GET /usuarios
    ▼
UsuarioController
    │
    │ findAll()
    ▼
UsuarioRepository
    │
    ▼
JPA / Hibernate
    │
    │ SQL
    ▼
MySQL
    │
    ▼
usuarios
```

Respuesta:

``` text
MySQL
   ↓
Hibernate
   ↓
Repository
   ↓
Controller
   ↓
JSON
   ↓
Navegador
```

------------------------------------------------------------------------

# 25. Crear el frontend

Se decidió separar frontend y backend.

Estructura general:

``` text
Conexion_API_DB/
│
├── conexion-db/
│   └── Backend Spring Boot
│
└── interfaz-usuarios/
    ├── index.html
    ├── css/
    │   └── styles.css
    └── js/
        └── app.js
```

Esto representa una arquitectura sencilla de frontend separado del
backend.

------------------------------------------------------------------------

# 26. Crear `index.html`

Código:

``` html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Usuarios</title>

    <link rel="stylesheet" href="./css/styles.css">
</head>

<body>

    <main class="contenedor">

        <h1>Usuarios</h1>

        <p>
            Usuarios registrados en nuestra base de datos
        </p>

        <button id="btnCargar">
            Cargar usuarios
        </button>

        <section id="listaUsuarios"></section>

    </main>

    <script src="./js/app.js"></script>

</body>
</html>
```

------------------------------------------------------------------------

# 27. Crear el CSS

Archivo:

``` text
css/styles.css
```

Código:

``` css
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
}

.contenedor {
    width: 90%;
    max-width: 700px;
    margin: 50px auto;
    background-color: white;
    padding: 30px;
    border-radius: 10px;
}

h1 {
    margin-bottom: 5px;
}

button {
    margin-top: 20px;
    padding: 10px 20px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
}

#listaUsuarios {
    margin-top: 25px;
}

.usuario {
    padding: 15px;
    margin-bottom: 10px;
    border: 1px solid #ddd;
    border-radius: 8px;
}
```

------------------------------------------------------------------------

# 28. Crear el JavaScript

Archivo:

``` text
js/app.js
```

Código final:

``` javascript
const btnCargar = document.getElementById("btnCargar");
const listaUsuarios = document.getElementById("listaUsuarios");

btnCargar.addEventListener("click", obtenerUsuarios);

async function obtenerUsuarios() {

    const respuesta = await fetch("http://localhost:8080/usuarios");

    const usuarios = await respuesta.json();

    mostrarUsuarios(usuarios);
}

function mostrarUsuarios(usuarios) {

    listaUsuarios.innerHTML = "";

    usuarios.forEach(usuario => {

        const tarjeta = document.createElement("div");

        tarjeta.classList.add("usuario");

        tarjeta.innerHTML = `
            <h3>${usuario.nombre}</h3>
            <p>ID: ${usuario.idUsuario}</p>
            <p>Correo: ${usuario.correo}</p>
        `;

        listaUsuarios.appendChild(tarjeta);
    });
}
```

------------------------------------------------------------------------

# 29. Explicación del JavaScript

## Obtener el botón

``` javascript
const btnCargar = document.getElementById("btnCargar");
```

Busca en HTML:

``` html
<button id="btnCargar">
```

y guarda una referencia al botón.

------------------------------------------------------------------------

## Obtener el contenedor

``` javascript
const listaUsuarios = document.getElementById("listaUsuarios");
```

Busca:

``` html
<section id="listaUsuarios"></section>
```

Ese elemento será donde aparecerán las tarjetas.

------------------------------------------------------------------------

# 30. Escuchar el clic

``` javascript
btnCargar.addEventListener("click", obtenerUsuarios);
```

Significa:

> Cuando se haga clic en el botón, ejecutar `obtenerUsuarios`.

Flujo:

``` text
Clic
 ↓
obtenerUsuarios()
```

------------------------------------------------------------------------

# 31. `fetch()`

La parte fundamental:

``` javascript
const respuesta = await fetch("http://localhost:8080/usuarios");
```

`fetch()` realiza una petición HTTP.

En este caso:

``` http
GET http://localhost:8080/usuarios
```

Esto permite que el frontend consuma nuestra API.

------------------------------------------------------------------------

# 32. `async` y `await`

La función es:

``` javascript
async function obtenerUsuarios()
```

Se utiliza `async` porque vamos a trabajar con una operación asíncrona.

Después:

``` javascript
const respuesta = await fetch(...)
```

`await` permite esperar el resultado de la petición antes de continuar.

Después:

``` javascript
const usuarios = await respuesta.json();
```

esperamos a que la respuesta se convierta a JSON.

------------------------------------------------------------------------

# 33. Qué contiene `usuarios`

Después de:

``` javascript
const usuarios = await respuesta.json();
```

la variable contiene aproximadamente:

``` javascript
[
    {
        idUsuario: 1,
        nombre: "Juan Pérez",
        correo: "juan@gmail.com"
    },
    {
        idUsuario: 2,
        nombre: "María López",
        correo: "maria@gmail.com"
    }
]
```

Estos datos comenzaron originalmente en MySQL.

------------------------------------------------------------------------

# 34. Mostrar usuarios

Después:

``` javascript
mostrarUsuarios(usuarios);
```

se envía la lista a la función encargada de construir las tarjetas.

------------------------------------------------------------------------

# 35. Recorrer el arreglo

``` javascript
usuarios.forEach(usuario => {
```

Esto recorre cada usuario.

Primera vuelta:

``` text
Juan Pérez
```

Segunda vuelta:

``` text
María López
```

------------------------------------------------------------------------

# 36. Crear una tarjeta

``` javascript
const tarjeta = document.createElement("div");
```

JavaScript crea dinámicamente un elemento:

``` html
<div></div>
```

Después:

``` javascript
tarjeta.classList.add("usuario");
```

le asigna la clase CSS:

``` text
usuario
```

------------------------------------------------------------------------

# 37. Insertar los datos

``` javascript
tarjeta.innerHTML = `
    <h3>${usuario.nombre}</h3>
    <p>ID: ${usuario.idUsuario}</p>
    <p>Correo: ${usuario.correo}</p>
`;
```

Para Juan se genera algo equivalente a:

``` html
<div class="usuario">
    <h3>Juan Pérez</h3>
    <p>ID: 1</p>
    <p>Correo: juan@gmail.com</p>
</div>
```

Para María:

``` html
<div class="usuario">
    <h3>María López</h3>
    <p>ID: 2</p>
    <p>Correo: maria@gmail.com</p>
</div>
```

------------------------------------------------------------------------

# 38. Agregar la tarjeta a la página

``` javascript
listaUsuarios.appendChild(tarjeta);
```

Añade la tarjeta dentro de:

``` html
<section id="listaUsuarios"></section>
```

Por eso finalmente aparecen las tarjetas.

------------------------------------------------------------------------

# 39. Primer problema del frontend: no aparecía nada

Al hacer clic en:

``` text
Cargar usuarios
```

no aparecían los usuarios.

Sin embargo, en el backend aparecía:

``` text
Hibernate: select u1_0.idUsuario,u1_0.correo,u1_0.nombre from usuarios u1_0
```

Esto fue una pista muy importante.

Significaba que:

``` text
Frontend
   ↓
fetch()
   ↓
Spring Boot
   ↓
Hibernate
   ↓
MySQL
```

sí estaba ocurriendo.

Pero la respuesta no estaba llegando correctamente a JavaScript.

------------------------------------------------------------------------

# 40. Error de CORS

La consola del navegador mostró:

``` text
Access to fetch at 'http://localhost:8080/usuarios'
from origin 'http://127.0.0.1:5500'
has been blocked by CORS policy
```

También:

``` text
GET http://localhost:8080/usuarios net::ERR_FAILED 200 (OK)
```

y:

``` text
TypeError: Failed to fetch
```

------------------------------------------------------------------------

# 41. ¿Qué significa CORS?

CORS significa:

``` text
Cross-Origin Resource Sharing
```

El navegador aplica políticas de seguridad cuando una página intenta
consumir recursos desde otro origen.

Nuestro frontend se estaba ejecutando en:

``` text
http://127.0.0.1:5500
```

y el backend en:

``` text
http://localhost:8080
```

Aunque ambos apuntan al mismo computador, no son el mismo origen.

Existen diferencias en:

``` text
127.0.0.1 ≠ localhost
5500 ≠ 8080
```

Por eso el navegador bloqueó el acceso desde JavaScript.

------------------------------------------------------------------------

# 42. Algo importante sobre el error `200 OK`

El error mostraba:

``` text
ERR_FAILED 200 (OK)
```

Esto es muy útil para entender CORS.

El backend sí había procesado correctamente la petición y había
consultado MySQL.

El problema estaba en que el navegador no permitía que JavaScript
utilizara esa respuesta debido a la política CORS.

Por eso:

``` text
Backend: 200 OK
Frontend: bloqueado por CORS
```

No era un problema de MySQL.

------------------------------------------------------------------------

# 43. Solucionar CORS

Se agregó al controller:

``` java
@CrossOrigin(origins = "http://127.0.0.1:5500")
```

Por eso el Controller final quedó:

``` java
package com.ejemplo.conexiondb.controller;

import com.ejemplo.conexiondb.model.Usuario;
import com.ejemplo.conexiondb.repository.UsuarioRepository;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/usuarios")
@CrossOrigin(origins = "http://127.0.0.1:5500")
public class UsuarioController {

    private final UsuarioRepository usuarioRepository;

    public UsuarioController(UsuarioRepository usuarioRepository) {
        this.usuarioRepository = usuarioRepository;
    }

    @GetMapping
    public List<Usuario> obtenerUsuarios() {
        return usuarioRepository.findAll();
    }
}
```

Esto le dice a Spring:

> Permitir peticiones provenientes de `http://127.0.0.1:5500`.

------------------------------------------------------------------------

# 44. No utilizar `no-cors`

El navegador sugería:

``` text
mode: 'no-cors'
```

No se utilizó.

Para esta aplicación queremos que JavaScript pueda leer la respuesta
JSON.

El objetivo es:

``` text
fetch()
   ↓
respuesta HTTP
   ↓
JSON
   ↓
JavaScript
```

Por eso se configura correctamente CORS en el backend.

------------------------------------------------------------------------

# 45. Resultado final

Después de configurar CORS, ejecutar nuevamente el backend y abrir el
frontend con Live Server, el botón:

``` text
Cargar usuarios
```

funciona.

La página muestra dos tarjetas:

``` text
Juan Pérez
ID: 1
Correo: juan@gmail.com
```

y:

``` text
María López
ID: 2
Correo: maria@gmail.com
```

------------------------------------------------------------------------

# 46. Flujo completo final

Este es el concepto más importante de toda la práctica:

``` text
┌─────────────────────────────┐
│          FRONTEND           │
│       HTML + CSS + JS       │
│                             │
│  Botón "Cargar usuarios"    │
└──────────────┬──────────────┘
               │
               │ fetch()
               │ GET /usuarios
               ▼
┌─────────────────────────────┐
│           BACKEND           │
│        Spring Boot          │
│                             │
│    UsuarioController        │
└──────────────┬──────────────┘
               │
               │ findAll()
               ▼
┌─────────────────────────────┐
│        REPOSITORY           │
│      UsuarioRepository      │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│        JPA / HIBERNATE      │
│                             │
│      Genera SQL             │
└──────────────┬──────────────┘
               │
               │ SELECT
               ▼
┌─────────────────────────────┐
│            MYSQL            │
│                             │
│         conexion_db         │
│                             │
│          usuarios           │
└──────────────┬──────────────┘
               │
               │ Datos
               ▼
┌─────────────────────────────┐
│        JPA / HIBERNATE      │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│        SPRING BOOT          │
│                             │
│       Convierte a JSON      │
└──────────────┬──────────────┘
               │
               │ HTTP Response
               ▼
┌─────────────────────────────┐
│          JAVASCRIPT         │
│                             │
│       respuesta.json()      │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│            HTML             │
│                             │
│      Tarjetas de usuarios   │
└─────────────────────────────┘
```

------------------------------------------------------------------------

# 47. Resumen de responsabilidades

## MySQL

Su responsabilidad es:

-   Guardar datos.
-   Organizar tablas.
-   Ejecutar consultas.
-   Mantener la información persistente.

No debería ser consumido directamente por el navegador.

------------------------------------------------------------------------

## Spring Boot

Su responsabilidad es:

-   Recibir peticiones HTTP.
-   Procesar la lógica del backend.
-   Comunicarse con la base de datos.
-   Exponer endpoints.
-   Devolver respuestas.

------------------------------------------------------------------------

## JPA / Hibernate

Su responsabilidad es facilitar el trabajo entre:

``` text
Java ↔ Base de datos
```

Por ejemplo:

``` java
usuarioRepository.findAll();
```

termina generando una consulta SQL.

------------------------------------------------------------------------

## Repository

Su responsabilidad es proporcionar operaciones para trabajar con los
datos.

Ejemplo:

``` java
findAll()
```

------------------------------------------------------------------------

## Controller

Su responsabilidad es recibir las peticiones HTTP.

Ejemplo:

``` http
GET /usuarios
```

------------------------------------------------------------------------

## JavaScript

Su responsabilidad en el frontend es:

-   Detectar eventos.
-   Hacer peticiones HTTP.
-   Recibir JSON.
-   Procesar los datos.
-   Modificar el HTML.

------------------------------------------------------------------------

## HTML

Su responsabilidad es representar la estructura de la interfaz.

------------------------------------------------------------------------

## CSS

Su responsabilidad es la apariencia.

------------------------------------------------------------------------

# 48. Conceptos que quedaron aprendidos

Al repetir esta práctica desde cero, se deben dominar especialmente
estos conceptos:

## Base de datos

-   Crear una base de datos.
-   Seleccionar una base de datos.
-   Crear tablas.
-   Insertar datos.
-   Consultar datos.
-   `DESCRIBE`.
-   Claves primarias.
-   `AUTO_INCREMENT`.

## Java

-   Clases.
-   Atributos.
-   Métodos.
-   Getters y setters.
-   Archivos y nombres de clases.
-   Mayúsculas y minúsculas.

## Spring Boot

-   Proyecto Maven.
-   `application.properties`.
-   Controller.
-   Repository.
-   Entity.
-   Endpoint REST.
-   Inyección de dependencias.

## JPA/Hibernate

-   `@Entity`
-   `@Table`
-   `@Id`
-   `@Column`
-   `JpaRepository`
-   `findAll()`
-   Mapeo entre Java y SQL.

## API

-   HTTP.
-   GET.
-   Endpoint.
-   JSON.
-   Request.
-   Response.
-   Puerto.
-   `localhost`.

## JavaScript

-   `addEventListener`.
-   Funciones.
-   `async`.
-   `await`.
-   `fetch()`.
-   `response.json()`.
-   Arrays.
-   `forEach()`.
-   `document.createElement()`.
-   `innerHTML`.
-   `appendChild()`.

## CORS

-   Origen.
-   Frontend y backend en puertos diferentes.
-   Restricciones del navegador.
-   `@CrossOrigin`.

------------------------------------------------------------------------

# 49. Errores encontrados y cómo diagnosticarlos

## Error 1: `mysql: command not found`

Causa:

MySQL estaba instalado, pero el ejecutable no estaba disponible desde
esa terminal mediante el PATH.

Solución:

Comprobar la instalación y configurar correctamente el entorno.

------------------------------------------------------------------------

## Error 2: `Failed to configure a DataSource`

Causa:

Spring Boot no tenía una configuración válida para conectarse a MySQL.

Solución:

Configurar:

``` properties
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...
spring.datasource.driver-class-name=...
```

------------------------------------------------------------------------

## Error 3: `class Usuario is public...`

Causa:

El archivo se llamaba:

``` text
usuario.java
```

pero la clase era:

``` java
public class Usuario
```

Solución:

Renombrar:

``` text
Usuario.java
```

------------------------------------------------------------------------

## Error 4: `Unknown column 'id_usuario'`

Causa:

Hibernate estaba generando un nombre diferente al nombre real de la
columna.

Base de datos:

``` text
idUsuario
```

Hibernate:

``` text
id_usuario
```

Solución:

Mapear explícitamente:

``` java
@Column(name = "idUsuario")
```

y utilizar:

``` properties
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

------------------------------------------------------------------------

## Error 5: Whitelabel Error Page / HTTP 500

La página:

``` text
Whitelabel Error Page
```

no era la causa real.

La causa real se encontró revisando la terminal de Spring Boot.

Regla importante:

> Cuando una API devuelve 500, revisar primero los logs del backend.

------------------------------------------------------------------------

## Error 6: No aparecían las tarjetas

La terminal mostraba:

``` text
Hibernate: select ...
```

Esto demostraba que el backend estaba consultando la base de datos.

Después se revisó la consola del navegador y apareció CORS.

------------------------------------------------------------------------

## Error 7: CORS

Causa:

``` text
Frontend:
http://127.0.0.1:5500

Backend:
http://localhost:8080
```

El navegador bloqueaba la petición.

Solución:

``` java
@CrossOrigin(origins = "http://127.0.0.1:5500")
```

------------------------------------------------------------------------

# 50. Método de debugging utilizado

Una de las enseñanzas más importantes de la práctica fue aprender a no
adivinar.

Cuando algo fallaba:

``` text
1. Mirar el navegador.
2. Mirar la consola del navegador.
3. Mirar la terminal del backend.
4. Encontrar la excepción real.
5. Identificar en qué capa ocurrió.
6. Comprobar la base de datos si era necesario.
7. Corregir solamente la causa.
8. Volver a probar.
```

Ejemplo:

``` text
500
 ↓
Revisar terminal
 ↓
Unknown column
 ↓
DESCRIBE usuarios
 ↓
Comparar con Usuario.java
 ↓
Corregir mapeo
```

Otro ejemplo:

``` text
No aparecen usuarios
 ↓
Revisar terminal
 ↓
Hibernate sí consulta MySQL
 ↓
Revisar consola del navegador
 ↓
CORS
 ↓
Configurar @CrossOrigin
```

------------------------------------------------------------------------

# 51. Repetición recomendada desde cero

Para consolidar el conocimiento, repetir exactamente este proceso sin
copiar directamente la solución final.

## Fase 1 --- Base de datos

1.  Instalar/comprobar MySQL.

2.  Entrar con:

    ``` bash
    mysql -u root -p
    ```

3.  Crear `conexion_db`.

4.  Crear `usuarios`.

5.  Insertar dos usuarios.

6.  Ejecutar:

    ``` sql
    SELECT * FROM usuarios;
    ```

7.  Ejecutar:

    ``` sql
    DESCRIBE usuarios;
    ```

## Fase 2 --- Backend

1.  Crear proyecto Spring Boot.
2.  Agregar Web.
3.  Agregar Spring Data JPA.
4.  Agregar MySQL Driver.
5.  Configurar `application.properties`.
6.  Crear `Usuario`.
7.  Crear `UsuarioRepository`.
8.  Crear `UsuarioController`.
9.  Ejecutar Spring Boot.
10. Probar: `text     http://localhost:8080/usuarios`

## Fase 3 --- Frontend

1.  Crear `interfaz-usuarios`.
2.  Crear `index.html`.
3.  Crear `styles.css`.
4.  Crear `app.js`.
5.  Ejecutar con Live Server.
6.  Implementar `fetch()`.
7.  Probar el botón.

## Fase 4 --- Debugging

Si no funciona:

1.  Revisar consola del navegador.
2.  Revisar terminal.
3.  Revisar SQL generado.
4.  Revisar tabla.
5.  Revisar CORS.
6.  Corregir.
7.  Volver a probar.

------------------------------------------------------------------------

# 52. Resultado que se debe conseguir al repetir la práctica

Al finalizar nuevamente, se debe poder hacer esto:

``` text
1. Iniciar MySQL.
2. Iniciar Spring Boot.
3. Abrir el frontend.
4. Pulsar "Cargar usuarios".
5. Ver Juan Pérez.
6. Ver María López.
```

Y poder explicar sin memorizar:

``` text
¿De dónde salieron esos datos?
```

Respuesta:

``` text
MySQL
 ↓
JPA/Hibernate
 ↓
Repository
 ↓
Controller
 ↓
JSON
 ↓
fetch()
 ↓
JavaScript
 ↓
HTML
```

------------------------------------------------------------------------

# 53. La idea que debes llevarte para proyectos futuros

La página web normalmente **no se conecta directamente a MySQL**.

La arquitectura recomendada es:

``` text
Frontend
   │
   │ HTTP
   ▼
Backend / API
   │
   │ JPA / JDBC / ORM
   ▼
Base de datos
```

Para este proyecto:

``` text
HTML + CSS + JavaScript
          ↓
     Spring Boot API
          ↓
       JPA/Hibernate
          ↓
         MySQL
```

Esta misma idea se puede ampliar posteriormente para proyectos más
grandes:

``` text
Frontend
   ↓
API
   ↓
Controllers
   ↓
Services
   ↓
Repositories
   ↓
JPA/Hibernate
   ↓
MySQL
```

En proyectos más grandes se agregan autenticación, validaciones, DTOs,
manejo de errores, seguridad, variables de entorno, documentación de
API, etc.

Pero esta práctica constituye la base.

------------------------------------------------------------------------

# 54. Checklist final

Antes de considerar la práctica terminada:

-   [ ] Java funciona.
-   [ ] MySQL funciona.
-   [ ] Se puede entrar con `mysql -u root -p`.
-   [ ] Existe `conexion_db`.
-   [ ] Existe la tabla `usuarios`.
-   [ ] Existen Juan y María.
-   [ ] `DESCRIBE usuarios` muestra `idUsuario`.
-   [ ] Spring Boot arranca.
-   [ ] Tomcat utiliza el puerto 8080.
-   [ ] JPA/Hibernate inicia.
-   [ ] `Usuario.java` existe correctamente.
-   [ ] `Usuario.java` usa `@Entity`.
-   [ ] `Usuario.java` usa `@Table(name = "usuarios")`.
-   [ ] `idUsuario` utiliza `@Id`.
-   [ ] `idUsuario` utiliza `@Column(name = "idUsuario")`.
-   [ ] Existe `UsuarioRepository`.
-   [ ] Existe `UsuarioController`.
-   [ ] Existe `GET /usuarios`.
-   [ ] La API devuelve JSON.
-   [ ] Existe el frontend separado.
-   [ ] El frontend utiliza `fetch()`.
-   [ ] El frontend utiliza `async/await`.
-   [ ] CORS está configurado.
-   [ ] El botón funciona.
-   [ ] Las tarjetas aparecen.
-   [ ] Se entiende el recorrido completo.

------------------------------------------------------------------------

# 55. Resumen final en una sola imagen mental

Quédate con esto:

``` text
                         USUARIO
                            │
                            │ hace clic
                            ▼
                    ┌───────────────┐
                    │   FRONTEND    │
                    │ HTML/CSS/JS   │
                    └───────┬───────┘
                            │
                         fetch()
                            │
                       GET /usuarios
                            │
                            ▼
                    ┌───────────────┐
                    │  SPRING BOOT  │
                    │      API      │
                    └───────┬───────┘
                            │
                       Controller
                            │
                       Repository
                            │
                            ▼
                    ┌───────────────┐
                    │ JPA/HIBERNATE │
                    └───────┬───────┘
                            │
                       SQL SELECT
                            │
                            ▼
                    ┌───────────────┐
                    │     MYSQL     │
                    │ conexion_db   │
                    │   usuarios    │
                    └───────┬───────┘
                            │
                       datos
                            │
                            ▼
                    ┌───────────────┐
                    │  SPRING BOOT  │
                    │     JSON      │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ JAVASCRIPT    │
                    │  recibe JSON  │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │     HTML      │
                    │   tarjetas    │
                    └───────────────┘
```

Si puedes reconstruir esta práctica desde cero y explicar cada flecha de
este diagrama, ya tienes una base real para empezar a trabajar con
aplicaciones web conectadas a bases de datos.
