# Upgrade

Los siguientes documentos explican los diferentes procedimientos a seguir para realizar el upgrade de una versión de postgresql, dependiendo del tipo de upgrade \(minor-version, major-version\)

Las versiones de PostgreSQL consisten en un major-version y un minor-version. Por ejemplo, para la versión 10.1, \(10 sería la major - version y 1 sería la minor - version\). Para las versiones anteriores a la versión 10.0 de PostgreSQL, los números de versión consisten en tres números, por ejemplo, 9.6.22 \(9.6 sería major-version y 22 sería la minor versión\).

Información para los diferentes métodos de migración de postgresql:

* [https://www.postgresql.org/docs/current/upgrading.html](https://www.postgresql.org/docs/current/upgrading.html)
* [https://www.postgresql.org/docs/current/pgupgrade.html](https://www.postgresql.org/docs/current/pgupgrade.html)

## Upgrade postgresql minor-version

Las minor-version nunca cambian el formato de almacenamiento interno y siempre son compatibles con versiones de la misma major-version. Por ejemplo, 9.5.3 es compatible con 9.5.0, 9.5.1 y 9.5.6. Para actualizar entre versiones compatibles: 1. Parar el servidor de base de datos. 2. Instalar los binarios de la nueva versión. 3. Arrancar el servidor con los nuevos binarios.

Nota: El directorio de datos permanece sin cambios

## Upgrade postgresql major-version

Podemos seguir los diferentes métodos para la migración de una major-version de postgresql:

* pg\_dump
* pg\_upgrade

### Upgrading vía pg\_dumpall

Una vez instalado el software de la nueva versión, es recomendable usar el binario del pg\_dump o pg\_dumpall de la nueva versión.

Realizar los siguientes pasos para migrar la versión:

1. Parar aplicativo o deshabilitar la conexión de usuarios a la base de datos \(vía pg\_hba.conf reject\)
2. Realizar el backup full de la base de datos

`pg_dumpall > outputfile`

1. Shutdown de old.server `pg_ctl stop`
2. Renombrar el antiguo directorio de datos

`mv /pgdata/pg_data /pg data/pg_data.old`

1. Crear el nuevo cluster de postgres con la versión nueva.

`/usr/local/pgsql/bin/initdb -D /pgdata/pg_data`

1. Resturar los ficheros de configuración de la versión old. pg\_hba.conf y postgresql.conf.
2. Arrancar el nuevo cluster.

`/usr/local/pgsql/bin/postgres -D /pgdata/pg_data`

1. Restaurar el backup realizado de la versión anterior en el nuevo cluster.

`/usr/local/pgsql/bin/psql -d postgres -f outputfile`

