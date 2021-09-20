# Instalaciones

## Como instalar EPAS11

Para instalar EDB 11 a partir del repositorio de EDB, los comandos son los siguientes:

Nota: Asumimos que los directorios /db1/data y /var/lib/edb/as11/data/ estan vacios.

### Instalar repositorio:

```text
yum install http://yum.enterprisedb.com/edbrepos/edb-repo-latest.noarch.rpm
```

> Editar archivo para agregar usuario y contraseña validos de EDB: /etc/yum.repos.d/edb.repo

### instalar EDB AS 11

```text
yum install -y edb-as11-server edb-as11-server-client edb-as11-server-contrib
```

> Por defecto el directorio de datos se encuentra en: /var/lib/edb/as11/data/. e Eliminamos el directorio, que en este momento debe estar vacio rm -r /var/lib/edb/as11/data/

### Cambiamos el propietario del directorio

`chown enterprisedb:enterprisedb /db1/data`

### Creamos un link simbólico a la ubicación PGDATA deseada:

ln -s /db1/data /var/lib/edb/as11

### Inicializamos PGDATA

```text
/usr/edb/as11/bin/edb-as-11-setup initdb
```

### Arrancamos el servicio

```text
systemctl start edb-as-11
```

