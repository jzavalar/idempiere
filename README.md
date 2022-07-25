# idempiere
## iDempiere para dummies
### Cómo instalar y usar iDempiere en Fedora 36 y no morir en el intento.

En este repositorio iré creando los materiales necesarios para revelar los detalles técnicos que permitan a un usuario común con conocimientos básicos de Linux, instalar y usar [iDempiere](https://github.com/idempiere/idempiere), que es, en mi opinión, el mejor sistema empresarial disponible para empresas al menor costo de adquisición.

### Instalar iDempiere en Fedora 36

1. Creación de la bases de datos

Acceder a PostgreSQL:
sudo -u postgres psql

Crear la base de datos idempiere:
postgres=# 
create database idempiere;

Crear el usuario con su contraseña:
postgres=# 
create user adempiere with encrypted password 'adempiere';

Otorgar todos los privilegios sobre la base de datos, previamente creada:
postgres=# 
grant all privileges on database idempiere to adempiere;

Salir:
postgres=# 
\q

2. Instalación Gráfica de iDempiere

Cambiarse al usuario idempiere:
su - idempiere

Cambiarse al usuario idempiere:
cd /opt/idempiere-server

Ejecutar el instalador gráfico:
sh setup.sh

o 
sh setup-alt.sh

Parámetros

Java:
    Java Home: 
/usr/lib/jvm/java-11-openjdk-11.0.15.0.10-1.fc36.x86_64

iDempiere:
    iDempiere Home: 
/opt/idempiere-server
    KeyStore Password: 
(creada por defecto)
 
Application Server: 
    Application Server: 
0.0.0.0
    Web Port: 
8080
    SSL: 
8443

Database Server:
     DB Already Exists: 
unchecked
     Database Name: 
idempiere
     Database Port: 
5432
     Database User: 
adempiere
     DB Admin Password: 
postgres
     Database User: 
adempiere
     Database Password: 
adempiere

3. Importar la base de datos

Como prerrequisito, después de instalar el servidor, se debe ejecutar el siquiente script, como usuario idempiere. Si no, debe cambiarse de usuario: 
su - idempiere

Cambiar de directorio: 
cd /opt/idempiere-server/utils

Ejecutar el script:
sh RUN_ImportIdempiere.sh

4. Actualizar la base de datos

Se debe ejecutar el siquiente script, como usuario idempiere. Si no, debe cambiarse de usuario: 
su - idempiere

Cambiar de directorio: 
cd /opt/idempiere-server/utils

Ejecutar el script:
sh RUN_SyncDB.sh

Abandonar el usuario:
exit

5. Registrar el código de la versión en la base de datos

Se debe ejecutar el siquiente script, como usuario idempiere. Si no, debe cambiarse de usuario: 
su - idempiere

Cambiar de directorio: 
cd /opt/idempiere-server

Firmar la base de datos con el siguiente script:
sh sign-database-build-alt.sh

Abandonar el usuario:
exit

Fuente:
https://wiki.idempiere.org/en/Installing_from_Installers
