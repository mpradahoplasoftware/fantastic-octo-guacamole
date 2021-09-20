---
description: >-
  La siguiente documentación es una guía para la configurar postgresql 12 en
  streaming replication en una arquitectura master - slave - slace
---

# Configuración postgresql streaming replication

## Prerequisitos

Tener instalados las servidores de postgresql 12 o superior

* Tener instalados las servidores de postgresql 12 o superior
* Tener creado el directorio /pddata/pg\_data y /pgdata/archived
* Tener conectividad por ssh para los usuarios postgres y root

| Nombre servidor | Dirección IP |
| :--- | :--- |
| pg12nodo1 \(master\) | 192.168.80.20 |
| pg12nodo2 \(slave\) | 192.168.80.21 |
| pg12nodo3 \(slave\) | 192.168.80.22 |

## Procedimiento de configuración.

1. Configurar parámetros para la replicación en el fichero de configuración "postgresql.conf"
2. Crear un usuario para la replicación
3. Modificar el fichero de configuracion **pg\_hba.conf** para permitir replicacion
4. Reiniciar el servidor postgresql para que todos los cambios tomen efecto.
5. Realizar un backup del servidor master a cada uno de los servidores slave
6. Revisar en el PGDATA del servidor slave que ser ha creado el fichero **standby.signal** y la configuración de conexión al nodo master \(primary\_conninfo\) se ha agregado al fichero de configuración postgresql.auto.conf
7. Revisar replicación.

{% hint style="info" %}
A partir de la versión 12 de postgresql, el fichero de configuración "_recovery.conf_" en los servidores standby, **ya no existe** y se han integrado como parámetros en el fichero de configuración _"postgresql.conf"_
{% endhint %}

```text
# 1. Modificación en el fichero de configuración "postgresql.conf"
############# master/postgresql.conf #############
wal_level = replica
archive_mode = on
max_wal_senders = 10 
wal_keep_segments = 10
hot_standby = on
archive_command = 'cp %p /pgdata/archived/%f'
port = 5432
wal_log_hints = on
restore_command = 'cp /pgdata/archived/%f %p'
archive_cleanup_command = 'pg_archivecleanup /pgdata/archived %r'
```

```text
# Crear usuario para la replicación entre los servidores master - standby4
psql> CREATE USER repuser WITH REPLICATION ENCRYPTED PASSWORD 'repuser';

# Alternativamente se puede crear desde sistema operativo

postgres# createuser repuser -s --replication
```

```text
# Añadir la siguiente línea al fichero de configuración "pga_hba.conf"

host    replication     repuser      192.168.80.20/32     md5
host    replication     repuser      192.168.80.21/32     md5
host    replication     repuser      192.168.80.22/32     md5

# Recargar la configuración del cluster:

pg_ctl -D /pgdata/pg_data -l logfile reload

```

```text
# Crear una copia del servidor master al slave1 y al slave2
pg12nodo1#pg_basebackup -h 192.168.80.21 -U repuser -p 5432 -D $PGDATA -Fp -Xs -P -R
pg12nodo1#pg_basebackup -h 192.168.80.22 -U repuser -p 5432 -D $PGDATA -Fp -Xs -P -R

# Nota: Previamente borrar el directorio PGDATA en todos los standby server (rm -fr $PGDATA)
```

```text
# Verificar proceso de replicación

# En el standby:
$ psql -c "\x" -c "SELECT * FROM pg_stat_wal_receiver;"

# En el master
$ psql -c "\x" -c "SELECT * FROM pg_stat_replication;"
$ psql -c "\x" -c "SELECT pg_current_xlog_location();"
$ psql -c "\x" -c "SELECT pg_last_xlog_receive_location();"
```

