# MongoDB

### ELEMENTOS PRINCIPALES EN LA PLATAFORMA DE MONGODB

Mongodb está disponible en dos versiones: Community o Enterprise

Tipo de aplicaciones:

* CLOUD
  * mongoshell \(cli-tool\)
  * Compass    \(gui-tool\)
  * Atlas     
  * Ops Manager
  * BI Connector
  * Charts
* SERVER
  * mongoshell \(cli tools\)

## Cloud

{% tabs %}
{% tab title="Ops-Manager" %}
```
Available with MongoDB Enterprise Advanced, Ops Manager is the best way to run MongoDB on your own infrastructure, making it fast and easy for your team to deploy, monitor, back up and scale MongoDB.
```
{% endtab %}

{% tab title="Compass" %}
```
Search, visualize, and work with your data through an intuitive GUI.
Manipulate your data with a powerful visual editing tool.
Understand performance issues with visual explain plans and manage your indices.
```
{% endtab %}

{% tab title="BI Connector" %}
```
Allow any BI tool that can speak the MySQL protocol to work with your MongoDB data.
Leverage the BI tools your organization already uses.
Perform federated analytics, combining data from MongoDB and other databases.
```
{% endtab %}

{% tab title="Charts" %}
```text
The fastest way to create visualizations of MongoDB data.
Built for the document model.
Visualize live data from any of your MongoDB instances. Available on MongoDB Atlas.
```
{% endtab %}
{% endtabs %}

\_\_

## MongoDB Connectors

Conectores para integrar MongoDB con el entorno. Por ejemplo: MongoDB connectors for BI MongoDB connectors for Apache Kafka MongoDB connectors for

#### PRINCIPIOS BASICOS DE MONGODB

* MongoDB almacena la información en forma de documentos \(son pares clave-valor en formato JSON\)
* MongoDB almacena todos los documentos en colecciones
* Una colección es un grupo de documentos relacionados semánticamente.
* Todos los documentos tendrán un campo \_id
* Los documentos en MongoDB tienen un esquema flexible
* Existen dos formas de relacionar los datos:
  * Referencias a otros documentos.
  * Modelo basado en Subdocumentos \(Se puede recuperar toda la información de una sola vez\)
    * El modelo de datos está altamente relacionado con el uso que hacemos de los datos

### Tipo de datos:

* Los más comunes: int32, double, date, string



  * String − This is the most commonly used datatype to store the data. String in MongoDB must be UTF-8 valid.
  * Integer − This type is used to store a numerical value. Integer can be 32 bit or 64 bit depending upon your server.
  * Boolean − This type is used to store a boolean \(true/ false\) value.
  * Double − This type is used to store floating point values.
  * Min/ Max keys − This type is used to compare a value against the lowest and highest BSON elements.
  * Arrays − This type is used to store arrays or list or multiple values into one key.
  * Timestamp − ctimestamp. This can be handy for recording when a document has been modified or added.
  * Object − This datatype is used for embedded documents.
  * Null − This type is used to store a Null value.
  * Symbol − This datatype is used identically to a string; however, it's generally reserved for languages that use a specific symbol type.
  * Date − This datatype is used to store the current date or time in UNIX time format. You can specify your own date time by creating object of Date and passing day, month, year into it.
  * Object ID − This datatype is used to store the document’s ID.
  * Binary data − This datatype is used to store binary data.
  * Code − This datatype is used to store JavaScript code into the document.
  * Regular expression − This datatype is used to store regular expression.

### Acceso a MongoDB

Ejemplo si usamos el mongoshell:

mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass

### Consultas tipicas CRUD.

* Consultas de busqueda find\(\)
  * Buscar en un documento que no tiene el campo &gt;&gt;xxxx&lt;&lt;

    db.data.find\( {atmosphericPressureChange: {$exists : false}}\).count\(\);

### Comandos útiles en mongo shell

* Logs del sistema
  * Ver los logs del sistema

```text
db.getLogComponents()
db.adminCommand({ "getLog": "global" })
```

* Cambiar el nivel de log de un componente

```text
db.setLogLevel(0, "index")
```

* Añadir Profiling para capturar eventos de sistema.

```text
db.db.getProfilingLevel()   -- 0 Desactivado 
                            -- 1 Captura operaciones que superen un umbral (Slowms)
                            -- 2 Captura todas las operacione
```

[Documentación sobre logs y métodos](https://docs.mongodb.com/manual/reference/log-messages/index.html)

* Creación de usuario y Roles

  ```text
  db.createUser( { user: "security_officer",  pwd: "h3ll0th3r3", roles: [ { db: "admin", role: "userAdmin" } ]})
  db.createUser( { user: "dba",  pwd: "c1lynd3rs", roles: [ { db: "admin", role: "dbAdmin" } ] })
  db.grantRolesToUser( "dba",  [ { db: "playground", role: "dbOwner"  } ] )

  Ver privilegios de un usuario.
  db.runCommand( { rolesInfo: { role: "dbOwner", db: "playground" }, showPrivileges: true} )
  ```

