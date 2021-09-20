---
description: >-
  Documentos y otros procedimientos de interés para realizar el proceso de
  migración a PostgreSql
---

# Migraciones

## Migración de Oracle a PostgreSql

* Usando la herramienta de enterprisedb
* Usando la herramienta de Amazon \(AWS\)



### Migración usando AWS



En el siguiente enlace se puede consultar el proceso de instalación y se puede descargar el software para realizar la instalación.

[https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP\_Installing.html](https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP_Installing.html) 

{% embed url="https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP\_Installing.html" %}

[https://s3.amazonaws.com/publicsctdownload/Ubuntu/aws-schema-conversion-tool-1.0.latest.zip](https://s3.amazonaws.com/publicsctdownload/Ubuntu/aws-schema-conversion-tool-1.0.latest.zip)

1. Descargar el software para el sistema operativo adecuado \(Windows, Linux, Mac...\).
2. Descomprimir e instalar el package \(.rpm , .deb, .msi\)
3. Descargar e instalar los drivers jdbc para Oracle y Postgresql

```
PROMPT>sudo mkdir –p /usr/local/jdbc-drivers
PROMPT>sudo cd /usr/local/jdbc-drivers
PROMPT>sudo mkdir oracle-jdbc

# Copiar los drivers para oracle 
PROMPT>sudo cp /tmp/ojdbc7.jar /usr/local/jdbc-drivers/oracle-jdbc

# Copiar los drivers para Postgresql
PROMPT>sudo cp /tmp/postgresql-X.X.X.jre7.tar /usr/local/jdbc-drivers
```

{% hint style="info" %}
 Una vez instalados los drivers configurar la aplicación AWS Schema Conversion -&gt; Global Settings -&gt; Drivers con el path correspondiente a Oracle y Postgres
{% endhint %}

