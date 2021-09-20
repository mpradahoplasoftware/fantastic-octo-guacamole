---
description: Guías y procesos de instalación de herramientas.
---

# Instalaciones y configuración

* Instalación del servidor de base de datos
* Instalación de herramientas de migración
* Instalación de aplicaciones de administración
* Creación de una base de datos con una codificación especifica
* Instalación de **pgbench**

## Instalación de servidor de base de datos.

## Creación de una base de datos con una codificación especifica

El _encoding_ de una base de datos se puede definir en varios niveles:

* Al iniciar el cluster
* Al crear la base de datos
* A nivel de sesión 
* De manera permanente, modificando template1.

Ejemplos:

```text
# Al inicializar el cluster
initdb -D /pgdata -E UTF-8 --locale=es_ES.UTF-8

# Al crear la bbdd
CREATE DATABASE mi_bbdd LC_COLLATE 'es_ES.UTF-8' LC_CTYPE 'es_ES.UTF-8' ENCODING UTF8 TEMPLATE template0;

# De manera permanente, al crear la bbdd se puede modificar el template1

UPDATE pg_database SET datistemplate = FALSE WHERE datname = 'template1';
DROP DATABASE template1;
CREATE DATABASE template1 WITH TEMPLATE = template0 LC_COLLATE 'es_ES.UTF-8' LC_CTYPE 'es_ES.UTF-8' ENCODING = 'UTF8' ;
UPDATE pg_database SET datistemplate = TRUE WHERE datname = 'template1';
```

### Instalación de pgbench



