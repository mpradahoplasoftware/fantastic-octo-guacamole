---
description: Ejemplos de como actuar para resolver las  incidencias típicas con postgresql
---

# Incidencias

## Incidencias

#### Listado de incidencias

* [Archivo WAL eliminado o rotado.](resolucion-incidencias.md#archivo-wal-eliminado-o-rotado1)
* [Métodos de bloquear conexiones.](resolucion-incidencias.md#metodos-bloquear-conexiones)
* Datos a pedir cuando ocurre una incidencia.
* Reasignar Pgpool-II al cluster
* Instalar repositorio yum de EDB sin acceso a internet.

### Archivo WAL eliminado o rotado. <a id="archivo-wal-eliminado-o-rotado1"></a>

**Descripción del problema**

Caso real con una base de datos de Iberdrola, que tiene configurado EFM y se ha detectado que un fichero de WAL no se encuentra porque ha rotado o se ha eliminado.  
Los errores del log que aparecen son los siguientes:

```yaml
cp: cannot create regular file '/postgresql/logs/archives/0000000D0000015E00000018': No such file or directory
2020-08-10 12:37:15 CEST:::: LOG: archive command failed with exit code 1

Reviso hoy y no existe la carpeta donde se archiva '/postgresql/logs/archives/. La creo manualmente con los permisos necesarios y el error ahora es

cp: cannot stat 'pg_xlog/0000000D0000015E00000018': No such file or directory
2020-08-20 12:52:47 CEST:::: LOG: archive command failed with exit code 1
2020-08-20 12:52:47 CEST:::: DETAIL: The failed archive command was: cp pg_xlog/0000000D0000015E00000018 /postgresql/logs/archives/0000000D0000015E00000018
```

**Datos solicitados**

Para intentar solventar la incidencia se le solicitan los siguientes datos:

* pg\_controldata -D &lt;datadir&gt; \([https://www.postgresql.org/docs/9.6/app-pgcontroldata.html](https://www.postgresql.org/docs/9.6/app-pgcontroldata.html)\). 
* ls -lrta /postgresql/logs/archives/
* ls -lrta pg\_xlog
* ls -lrta pg\_xlog/archive\_status/

{% hint style="info" %}
pg\_controldata para saber el timeline y el último WAL de la bbdd.
{% endhint %}

**Resolución propuesta**

El servicio de postgres está buscando un archivo wal \(0000000D0000015E00000018\) que ha sido eliminado o rotado.  
Acciones:

1. Buscar dicho archivo de wal en alguna otra ubicación o recuperarlo de algún backup y copiarlo a pg\_xlog
2. Si no lo encuentra para que postgres se olvide de archivar ese wal debe realizar lo siguiente: 
   1. Para el servicio de postgres
   2. Desactivar el parámetro archive\_mode \(off\)
   3. Iniciar el servicio de postgres, comprobar el log y la carpeta pg\_xlog/archive\_status/

Una vez se asegure de que el servicio está funcionando correctamente y no haya ningún archivo .ready de un wal antiguo, puede volver a activar el parámetro archive\_mode y reiniciar el servicio.

Para que cuando apague el maestro no se produzca un failover tiene dos opciones:

* Apagar los agentes EFM
* Establecer el parámetro auto.failover a false

Información de la documentación relevante \([https://www.enterprisedb.com/edb-docs/static/docs/efm/3.10/edb\_efm\_user.pdf](https://www.enterprisedb.com/edb-docs/static/docs/efm/3.10/edb_efm_user.pdf)\)

{% hint style="info" %}
The auto.failover property enables automatic failover. By default, auto.failover is set to true. \#Whether or not failover will happen automatically when the master fails. Set to false if you \#want to receive the failover notifications but not have EFM actually perform the failover steps. \#The value of this property must be the same across all agents. auto.failover=true
{% endhint %}

{% hint style="info" %}
Los agentes EFM permiten que un nodo master realice un failover automático a un esclavo en caso de fallo, pero para la sincronización del maestro y el esclavo se utiliza la funcionalidad del streaming replication \([https://www.postgresql.org/docs/12/warm-standby.html\#STREAMING-REPLICATION](https://www.postgresql.org/docs/12/warm-standby.html#STREAMING-REPLICATION)\), que permite transferir los registros WAL sobre la marcha. Por tanto, apagar los agentes EFM no afecta a la sincronización de los nodos. El orden recomendado para apagar los agentes EFM sería: 1º\) el esclavo, 2º\) el witness, 3º\) el master.

Por otro lado, cuando apague el servicio de postgres del maestro, el esclavo se quedará en recovery, esperando a dicho maestro. Si realizan rápido la tarea indicada no deberían tener problemas, no obstante, si surgiera algún problema tendrán que sincronizar los nodos manualmente con pg\_rewind o pg\_basebackup.
{% endhint %}

### Métodos para bloquear conexiones. <a id="metodos-bloquear-conexiones"></a>

Hay varias maneras de negar la conexion a los usuarios:

1.- Utilizando el metodo reject en el archivo pg\_hba.conf, para el segmento que corresponda.

```text
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             127.0.0.1/32            reject
```

2.- Revocando los pemisos de conexion al usuario:

```text
REVOKE CONNECT  ON DATABASE {database} FROM {usuario};
```

3.- Dependiendo de como este migrando las bases de datos, puede configurar el servidor para que solo acepte peticiones en localhost, o alguna ip privada que tenga el servidor.

```text
ALTER SYSTEM SET listen_addresses = '127.0.0.1';
```

{% hint style="info" %}
Debido a que el role PUBLIC otorga permisos de conexion, es necesario revocar explicitamente los permisos de conexion para el role public con:

```text
REVOKE CONNECT ON DATABASE {database} FROM PUBLIC;
```
{% endhint %}

## Datos a pedir cuando ocurre una incidencia.

Cuando ocurra una incidencia que no sabemos la arquitectura actual del cliente, solicitar la siguiente información:

Por favor, incluir una descripción de su arquitectura actual:

* Numero total de servidores, nombre de host y dirección IP, cuales son esclavos, cual es el maestro.
* Configuracion de pgpool \(watchdog, balanceo, numero de instancias de pgpool, direccion VIP, pgpool se encuentra instalado en cada servidor de base de datos o en servidores independientes.\)
* Versión de posgtresql ,  de pgpool , otras... y tipo de sistema operativo.
* Archivos de configuracion de pgpool y postgres
* Salida del comando: show all;

## Reasignar Pgpool-II al cluster

Ocurre en ocasiones que las bases de datos están funcionando correctamente pero el pgpool-ii no arranca porque no detecta que las bases de datos están levantadas y funcionando correctamente. Por ejemplo después de un reinicio del servidor o de las bases de datos.

Si hay un servidor maestro y un esclavo y la replicacion está funcionando correctamente, tenemos que hacer lo siguiente: 1. Detener el servicio de pgpool en los 3 nodos. 2. Borrar el archivo /tmp/pgpool\_status de los 3 nodos. 3. Arrancar el servicio de pgpool en los 3 nodos. 4. Conectarse al puerto de pgpool y ejecutar show pool\_nodes para validar el estatus.

También se puede intentar en primer lugar realizar un attach del nodo manualmente con el comando:

```text
pcp_attach_node -h 127.0.0.1 -U {USER} -p 9898 -n <node_id>
```



### Instalar repositorio yum de EDB sin acceso a internet

Si en un servidor se quiere instalar un servidor o software de EDB que se quiere tiene que acceder al repositorio de YUM de EDB pero no se tiene acceso a internet. Existe la opción de hacer un mirror del repositorio desde un equipo con acceso a internet y hacer disponible ese repositorio a las máquinas a instalar. Ya sea montando una unidad compartida , un pendrive .. , o copiando el repositorio entre los distintos servidores.  
Los pasos necesarios para este tipo de instalación vienen descritos en:

[https://www.enterprisedb.com/edb-docs/d/edb-postgres-advanced-server/installation-getting-started/installation-guide-for-linux/12/EDB\_Postgres\_Advanced\_Server\_Installation\_Guide\_Linux.1.14.html\#pID0E0HG0HA](https://www.enterprisedb.com/edb-docs/d/edb-postgres-advanced-server/installation-getting-started/installation-guide-for-linux/12/EDB_Postgres_Advanced_Server_Installation_Guide_Linux.1.14.html#pID0E0HG0HA)  
También otra opción es ir descargando paquete a paquete los necesarios para la instalación desde el repositorio, pero desde nuestro punto de vista, esto es bastante tedioso cuando la instalación es de varios productos.

```text
yum install epel-reslease
yum install createrepo
mkdir /srv/repos
reposync -r edbas12 -p /srv/repos 
createrepo /srv/repos
#Edit /etc/yum.repos.d/edb-repo
[edbas12]
name=EnterpriseDB Advanced Server 12
baseurl=https://yum.your_domain.com/edbas12
enabled=1
gpgcheck=0

$yum install edb-as12-server #Centos7
$dnf install edb-as12-server #Centos8

```

