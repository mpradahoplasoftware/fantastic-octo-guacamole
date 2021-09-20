# Configurar acceso SSH

## Configurar acceso por SSH entre dos servidores

Se disponen de dos servidores:

* masterepas12 con IP: 192.168.80.10
* slaveepas12 con IP: 192.168.80.11

Pasos a realizar:

1. Editar el fichero /etc/ssh/sshd\_config y descomentar las entradas
   * "PubkeyAuthentication yes"
   * "PasswordAuthentication yes"
2. Recargar el daemon del sshd 
3. Generar las claves publicas y privadas en cada servidor
4. Copiar las claves privadas entre los servidores

> En este caso se hace con el usuario _enterprisedb_ que es el usado para la instalación del EPAS-12

1. Editar el fichero /etc/ssh/sshd\_config

   ```text
    [root@slaveepas12 ~]#sudo vi /etc/ssh/sshd_config
    ...
    PubkeyAuthentication yes
    ...
    [root@slaveepas12 ~]# systemctl daemon-reload
    [root@slaveepas12 ~]# systemctl restart sshd
   ```

2. Generamos las claves en el **masterepas12**

   `[enterprisedb@masterepas12 enterprisedb]$` **`ssh-keygen -t rsa`**

3. Generamos las claves en el **slaveepas12 ``**`[enterprisedb@masterepas12 slaveepas12]$ ssh-keygen -t rsa  [enterprisedb@slaveepas12 enterprisedb]$ ls -l  .ssh total 8  -rw-------. 1 enterprisedb enterprisedb 1679 Feb 5 17:36 id_rsa  -rw-r--r--. 1 enterprisedb enterprisedb 406 Feb 5 17:36 id_rsa.pub` ****
4. Copiar e intercambiar las claves entre los servidores.

   `[enterprisedb@masterepas12 enterprisedb]$ ssh-copy-id enterprisedb@192.168.80.11` 

> Recordar modificar los permisos a 600 de todos los archivos dentro del directorio .ssh   
> cd .ssh   
> chmod 600 \*

1. Probar conectividad entre los servidores:

   `[enterprisedb@masterepas12 .ssh]$ ssh enterprisedb@192.168.80.11 enterprisedb@192.168.80.11's password: Last failed login: Wed Feb 5 17:57:27 UTC 2020 from 192.168.80.10 on ssh:notty There were 2 failed login attempts since the last successful login. Last login: Wed Feb 5 17:55:34 2020 from 192.168.80.10 [enterprisedb@slaveepas12 ~]$`

