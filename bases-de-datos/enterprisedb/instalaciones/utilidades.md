# Dblinks

## Extensiones

* [Dblinks](utilidades.md#dblinks)
* [Extensión FDW](utilidades.md#extension-fdw)

## Dblinks

* [Crear dblink de Postgresql a Oracle](utilidades.md#Crear-dblink-de-Postgresql-to-Oracle)
* [Crear dblink de Oracle a Postgresql](utilidades.md#crear-dblink-de-oracle-a-postgresql)
* [Crear dblink de Postgresql a Postgresql](utilidades.md#crear-dblink-de-postgresql-a-postgresql)
* [Configurar dblink\_ora ](utilidades.md#crear-dblink-de-postgresql-a-postgresql)

### Crear dblink de Postgresql to Oracle

Es necesario tener instalado un cliente Oracle y tener instalada la extención [Oracle\_FDW](https://github.com/laurenz/oracle_fdw), que permite un acceso a un base de datos Oracle desde PostgreSQL

Los pasos que tenemos que realizar son los siguientes: 

1. Instalar los binarios y compilar Oracle\_fdw en el servidor postgresql 

2. Crear la extensión en el servidor postgresql. 

3. Crear el dblink hacia la base de datos Oracle.

#### Instalar Oracle\_FDW

* Descargamos los fuentes desde el sitio [https://github.com/laurenz/oracle\_fdw/releases/tag/ORACLE\_FDW\_2\_1\_0](https://github.com/laurenz/oracle_fdw/releases/tag/ORACLE_FDW_2_1_0) en el directorio /opt/
* Revisar que pg\_config’ esté en PATH \(se puede testear con el comando ‘pg\_config –pgxs’\).
* Revisar que tenemos la variable ORACLE\_HOME en la ubicación de la instalación de Oracle.
* Descomprimimos el .tar.gz en /opt/ con el comando

  `tar -xvf ORACLE_FDW_2_1_0.tar.gz`

* Compilamos la versión del Oracle\_fdw, ejecutando lo siguiente:

```text
$ make
$ make install
```

Una vez se haya generado bien el binario, procedemos a crear la extensión. Pero antes de nada debemos configurar las variables de entorno para el proceso de PostgreSQL. Esto lo debemos hacer creando o editando el archivo /etc/systemd/system/postgresql-11.service con las siguientes variables:

```text
.include /usr/lib/systemd/system/postgresql-11.service
[Service]
Environment=ORACLE_HOME=<Directorio de Oracle Home>
Environment=LD_LIBRARY_PATH=<Directorio lib de Oracle Home>
Environment=TNS_ADMIN=<Directorio admin de Oracle Home>
Environment=PATH=$PATH:<pgsql-11/bin>
```

* Reiniciamos el servicio de PostgreSQL:

```text
systemctl restart postgresql-11
systemctl status postgresql-11
```

#### Crear la extensión Oracle\_fdw en Postgresql

* Crear la extensión en PostgreSQL

```sql
postgres=# create extension oracle_fdw ;
```

_Nota: Esto crea todas las funciones para el dblink._

* Configurar Oracle\_fdw con el usuario postgres:

```sql
pgdb=# CREATE EXTENSION oracle_fdw;
pgdb=# CREATE SERVER oradb FOREIGN DATA WRAPPER oracle_fdw
OPTIONS (dbserver '//dbserver.mydomain.com:1521/ORADB');
pgdb=# GRANT USAGE ON FOREIGN SERVER oradb TO pguser;
```

1. Creamos el mapeo para el usuario que utilizará el link

```sql
pgdb=> CREATE USER MAPPING FOR pguser SERVER oradb
OPTIONS (user 'orauser', password 'orapwd');
```

* A continuación creamos los mapeos de las tablas.

```sql
pgdb=> CREATE FOREIGN TABLE oratab (
id integer OPTIONS (key 'true') NOT NULL,
text character varying(30),
floating double precision NOT NULL
) SERVER oradb OPTIONS (schema 'ORAUSER', table 'ORATAB');
```

Por lo general el nombre del schema y la tabla deben ir mayúsculas.

Finalmente se puede utilizar la tabla como si fuera de PostgreSQL.

Referencias:

* [Using Foreign Data Wrappers to access remote PostgreSQL and Oracle databases](https://www.enterprisedb.com/postgres-tutorials/using-foreign-data-wrappers-access-remote-postgresql-and-oracle-databases) [Edb ](https://www.enterprisedb.com/edb-docs/d/edb-postgres-advanced-server/reference/database-compatibility-for-oracle-developers-reference-guide/11/Database_Compatibility_for_Oracle_Developers_Reference_Guide.1.035.html)
* [Create public database link](https://www.enterprisedb.com/edb-docs/d/edb-postgres-advanced-server/reference/database-compatibility-for-oracle-developers-reference-guide/11/Database_Compatibility_for_Oracle_Developers_Reference_Guide.1.035.html) 
* [Como crear un dblink postgres a Oracle](https://www.codificandola.net/2019/02/23/como-crear-un-dblink-entre-postgresql-11-y-oracle-10g/%20)

### Crear dblink de Oracle a Postgresql

Para realizar el acceso desde una base de datos Oracle a Postgresql \(o otro tipo de base de datos\), es necesario usar la conexión heterogénea de Oracle.  
Los siguientes pasos mostramos como configurar una conexión de Oracle a PostgreSQL a través de ODBC \(conectividad de base de datos abierta\) y Oracle heterogeneous Service\(hs\)

Para estas pruebas partimos de una instalación de Oracle Database 12c y y PostgreSQL 12.3 \(EnterpriseDB Advanced Server 12.3.4\).

1. Instalar controladores unixODBC  \(como root\) \``yum install unixODBC*`
2. Configurar controladores ODBC
   1. Configurar fichero **odbcinst.ini**
   2. Configurar fichero **odbc.ini**
3. Configurar el Oracle heterogeneous service \(HS\)
   1. Configurar fichero listener.ora
   2. Configurar fichero tnsnames.ora
   3. Recargar configuración del listener
4. Realizar pruebas de conectividad.

#### Configurar controladores ODBC

* Editar fichero **/etc/odbcinst.ini**

```sql
# Example driver definitions

# Driver from the postgresql-odbc package
# Setup from the unixODBC package
[PostgreSQL]
Description    = ODBC for PostgreSQL
Driver        = /usr/lib/psqlodbcw.so
Setup        = /usr/lib/libodbcpsqlS.so
Driver64    = /usr/lib64/psqlodbcw.so
Setup64        = /usr/lib64/libodbcpsqlS.so
FileUsage    = 1
```

* Editar fichero **/etc/odbc.ini**

```sql
[PostgreSQL1]
Driver = PostgreSQL   ### <== El mismo nombre asigando en odbcinst.ini
Description = PostgreSQL Data Source
Servername = 127.0.0.1
Port = 5444
UserName = MAPP
Password = mapp
Database = postgres
ReadOnly = no
ServerType = Postgres
ConnSettings = UseServerSidePrepare=1
ByteaAsLongVarBinary=1
Optimizer=0
Ksqo=0
Trace=yes
TraceFile = /var/log/odbc-TSMPGSQL-trace.log
Debug = No
DebugFile = /var/log/odbc-TSMPGSQL-debug.log
```

#### Configurar Oracle heterogeneous service \(HS\)

El Oracle heterogeneous service debería estar instalado en el directorio $ORACLE\_HOME/hs

* Crear el fichero **initPostgreSQL1.ora** con la siguientes información: 

```sql
HS_FDS_CONNECT_INFO = PostgreSQL1
HS_FDS_TRACE_LEVEL = ON
HS_FDS_SHAREABLE_NAME = /usr/lib64/psqlodbcw.so
HS_LANGUAGE = AMERICAN_AMERICA.WE8ISO8859P9
set ODBCINI=/etc/odbc.ini
```

* Configurar el fichero **"listener.ora"**

Añadir la parte correspondiente al nuevo SID \(creado para el acceso a postgres\) en el SID\_LIST\_LISTENER

```sql
#### Se añade esta parte ###
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME = EXTPROC1)
      (ORACLE_HOME = /opt/oracle/product/12.2.0.1/dbhome_1)
      (PROGRAM = extproc)
    )
    (SID_DESC =
      (SID_NAME = PostgreSQL1)
      (ORACLE_HOME = /opt/oracle/product/12.2.0.1/dbhome_1)
      (PROGRAM = dg4odbc)
    )
  )


LISTENER = 
(DESCRIPTION_LIST = 
  (DESCRIPTION = 
    (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1)) 
    (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521)) 
  ) 
) 

DEDICATED_THROUGH_BROKER_LISTENER=ON
DIAG_ADR_ENABLED = off
```

* Configurar **tnsnames.ora**

```sql
#### Se añade esta parte ###
ORCLCDB=localhost:1521/ORCLCDB
ORCLPDB1= 
(DESCRIPTION = 
  (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
  (CONNECT_DATA =
    (SERVER = DEDICATED)
    (SERVICE_NAME = ORCLPDB1)
  )
)
### Se añade esta parte
PostgreSQL1 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA =
      (SID = PostgreSQL1)
    )
   (HS=OK)
  )
###
EXTPROC_CONNECTION_DATA =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1))
    )
    (CONNECT_DATA =
      (SID = EXTPROC1)
      (PRESENTATION = RO)
    )
  )
```

{% hint style="info" %}
Fichero listener.ora y tnsnames.ora tienen que estar en $ORACLE\_HOME/network
{% endhint %}

### Instalación

```sql
# Creación de usuario "mapp" en postgres para prueba del dblink 
-bash-4.2$ psql postgres
psql (12.3)
Type "help" for help.


postgres=# create user "mapp" with password 'mapp'  superuser;
CREATE ROLE

postgres=# create table test(col1 int);
CREATE TABLE

postgres=# insert into test values (100);
INSERT 0 1

postgres=# select * from test;
 con1 
------
  100
(1 row)

# Nos conetamos a Oracle y damos permiso al usuario para crear dblinks.
sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Fri Aug 28 14:16:11 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Last Successful login time: Fri Aug 28 2020 11:43:20 +02:00

Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
SQL> grant create database link to subastas;
```

{% hint style="success" %}
Configurar los ficheros **odbcinst.ini y odbc.ini** tal y como se indican en los apartados anteriores. Además de los ficheros de listener.ora, tnsnames.ora y $ORACLE\_HOME/hs/admin/initPostgreSQL1.ora
{% endhint %}

```sql
# Probar conectividad por ODBC a Postgresql

[root@oranode-122 ~]# isql -v PostgreSQL1
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> select * from test;
+------------+
| con1       |
+------------+
| 100        |
+------------+
SQLRowCount returns 1
```

```sql
# Probar conectividad a Oracle

[root@oranode-122 ~]# su - oracle
Last login: Fri Aug 28 14:00:18 +02 2020 on pts/0
[oracle@oranode-122 ~]$ tnsping PostgreSQL1

TNS Ping Utility for Linux: Version 12.2.0.1.0 - Production on 28-AUG-2020 15:16:43

Copyright (c) 1997, 2016, Oracle.  All rights reserved.

Used parameter files:
/opt/oracle/product/12.2.0.1/dbhome_1/network/admin/sqlnet.ora


Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521)) (CONNECT_DATA = (SID = PostgreSQL1)) (HS=OK))
OK (0 msec)


# Recarcar el listener (o parar y arrancar el listener)

[oracle@oranode-122 ~]$ lsnrctl

LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 28-AUG-2020 15:17:29

Copyright (c) 1991, 2016, Oracle.  All rights reserved.

Welcome to LSNRCTL, type "help" for information.

LSNRCTL> status
Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
Start Date                28-AUG-2020 10:23:48
Uptime                    0 days 4 hr. 53 min. 42 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /opt/oracle/product/12.2.0.1/dbhome_1/network/admin/listener.ora
Listener Log File         /opt/oracle/diag/tnslsnr/oranode-122/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=oranode-122)(PORT=5500))(Security=(my_wallet_directory=/opt/oracle/product/12.2.0.1/dbhome_1/admin/ORCLCDB/xdb_wallet))(Presentation=HTTP)(Session=RAW))
Services Summary...
Service "EXTPROC1" has 1 instance(s).
  Instance "EXTPROC1", status UNKNOWN, has 1 handler(s) for this service...
Service "ORCLCDB" has 1 instance(s).
  Instance "ORCLCDB", status READY, has 1 handler(s) for this service...
Service "ORCLCDBXDB" has 1 instance(s).
  Instance "ORCLCDB", status READY, has 1 handler(s) for this service...
Service "PostgreSQL1" has 1 instance(s).
  Instance "PostgreSQL1", status UNKNOWN, has 1 handler(s) for this service...
The command completed successfully
LSNRCTL> reload
Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1)))
The command completed successfully

##
#
```

```sql
# Probar la conectividad por dblink de Oracle a Postgresql
SQL> create database link test 
connect to "mapp" identified by "mapp" using 'PostgreSQL1';

Database link created.

SQL> select * from "test"@test;

      con1
----------
       100
```

## Crear dblink de Postgresql a Postgresql

Ejemplo de como crear un dblink entre bases de datos Postgresql. Nota: Es necesario tener instalado el postgresql-contrib

1. Instalarán los archivos de la extensión "dblink" si no están instalados. 

ls -l /usr/share/postgresql/9.1/extension/dblink--1.0.sql

1. Instalamos los sql en la bbdd. 

psql postgres &lt; /usr/share/postgresql/9.1/extension/dblink--1.0.sql

1. Creamos la extensión psql postgres postgres=\# CREATE EXTENSION dblink;
2. Revisar si está instalada la extensión.

postgres=\# \dx

```text
                            List of installed extensions
```

Name \| Version \| Schema \| Description  
---------+---------+------------+-------------------------------------------------------------- dblink \| 1.0 \| public \| connect to other PostgreSQL databases from within a database plpgsql \| 1.0 \| pg\_catalog \| PL/pgSQL procedural language \(2 rows\)

1. Revisar si nos funciona el dblink a otra bbdd.

Select \* from dblink\( 'dbname=test host=172.27.38.7 user=user\_db password=password\_db', 'select nombre from personas'\) as \(persona varchar\);

La salida sería algo así

jesus andres eleazar orlando marcos carlos \(6 rows\)

DETALLE IMPORTANTE: hay que colocar los tipos de datos por c/u de las columnas que queremos que nos devuelva la consulta en el servidor remoto. Ejemplo:

select id, nombre from otras\_personas UNION\( select \* from  
dblink\( 'dbname=inventario host=200.25.10.11 user=user\_db password=password\_db', 'select id, nombre from usuarios'\) as otros\_usuarios \(id integer, nombre varchar\) \) order by id ASC

## Extensión FDW

FDW es una implementación de dblink, es más útil, así que para usarlo:

1. Crear una extensión:

CREATE EXTENSION postgres\_fdw;

1. Crear SERVIDOR:

CREATE SERVER name\_srv FOREIGN DATA WRAPPER postgres\_fdw OPTIONS \(host 'hostname', dbname 'bd\_name', port '5432'\);

1. Crear mapeo de usuario para servidor postgres

CREATE USER MAPPING FOR postgres SERVER name\_srv OPTIONS\(user 'postgres', password 'password'\);

1. Crear tabla extranjera o importar esquema del servidor: 

CREATE FOREIGN TABLE table\_foreign \(id INTEGER, code character varying\) SERVER name\_srv OPTIONS\(schema\_name 'schema', table\_name 'table'\);

IMPORT FOREIGN SCHEMA schema\_name\_to\_import\_from\_remote\_db FROM SERVER server\_name INTO schema\_name;

1. Usa esta tabla externa como está en tu base de datos o acceder al contenerdor de datos:

SELECT \* FROM table\_foreign;

SELECT \* FROM schema\_name.table;



## Configurarar DBLINK\_ORA

El siguiente procedimiento sirve para configurar el dblink_ora para prepara el uso del copyviadblink o dblink\_ora \(en EDB postgres\)_

Pasos para la configuración de PostgreSQL para usar dblink\_ora:

1. Instalación cliente Oracle en servidor Postgres.
2. Descargar el driver OCI [http://www.oracle.com/technology/tech/oci/instantclient/index.html](http://www.oracle.com/technology/tech/oci/instantclient/index.html)
3. Crear symbolic link named libclntsh.so

   ln -s libclntsh.so libclntsh.so

   ln -s libclntsh.so.11.1 libclntsh.so

4. Configurar oracle\_home = '' donde está la librería

Crear el dblink SELECT `dblink_ora_connect('oralink_fmsnr_img','172.31.2.3','FAMITEST','mmurillo','Dba$2018.',1521);` Ver status 

`SELECT dblink_ora_status('oralink_fmsnr_img');`

Insertar datos de una tabla a otra.

`INSERT INTO IMAGENES_AUTOLIQUIDACION SELECT id, file_name,image FROM dblink('dbname=dbtest', 'SELECT id, time FROM tblB') AS t(id integer, time integer) WHERE time > 1000;`

Sacar registro:

`SELECT * FROM dblink_ora_record('oralink_fmsnr_img','select id, file_name,image from fmsnr_img.imagenes_autoliquidacion') AS t1(id NUMERIC,file_name VARCHAR, image bytea) where rownum < 10;`

Eliminar el dblink 

`SELECT dblink_ora_disconnect('oralink_fmsnr_img');`



