## iDempiere para dummies

### Cómo instalar y usar iDempiere en Fedora 36 y no morir en el intento

En este repositorio iré creando los materiales necesarios para revelar los detalles técnicos que permitan a un usuario común con conocimientos básicos de Linux, instalar y usar [iDempiere](https://github.com/idempiere/idempiere), que es, en mi opinión, el mejor sistema empresarial disponible para empresas al menor costo de adquisición.

## Instalar iDempiere en Fedora 36

### Prerrequisitos

1. Actualizar el sistema    
```$ sudo dnf upgrade --refresh``` 

2. Instalar PostgreSQL 

   Instalar el repositorio RPM:  
   ```$ sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/F-36-x86_64/pgdg-fedora-repo-latest.noarch.rpm```

   Instalar postgresql:  
   ```$ sudo dnf install -y postgresql14-server```  
   
   Inicializar la base de datos y habilita la inicialización automática:  
   ```$ sudo /usr/pgsql-14/bin/postgresql-14-setup initdb```  

   Habilitar Postgresql como servicio:  
   ```$ sudo systemctl enable postgresql-14```

   Iniciar el servicio:  
   ```$ sudo systemctl start postgresql-14```  
   
   Verificar el servicio:  
   ```$ sudo systemctl status postgresql-14```  
   
   Opcionalmente, puede detener o reiniciar el servicio, de ser necesario:  
   ```
   $ sudo systemctl stop postgresql-14  
   $ sudo systemctl restart postgresql-14
   ```  
   
   Acceder con el usuario administrador de PostgreSQL (postgres):  
   ```$ sudo su - postgres```  
   
   Definir la contraseña del usuario postgres:  
   ```psql -c "alter user postgres with password 'contraseña'"```
   
   Abandonar el entorno de psql del usuario postgres:  
   ```exit```  

3. Configurar pgAdmin4

   Instalar el repositorio:  
   ```$ sudo rpm -i https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-fedora-repo-2-1.noarch.rpm```
   
   Instalar versión de escritorio y web de pgAdmin4:  
   ```$ sudo dnf install pgadmin4```
   
   Localizar el  archivo pg_hba.conf:  
   ```$ sudo find / -name pg_hba.conf```  
   
   Editar el archivo pg_hba.conf:  
   ```$ sudo vim /var/lib/pgsql/14/data/pg_hba.conf```  
   
   Cambiar la siguiente línea para que se acceda universalmente con contraseña:  
   ```
   #host    all             all             all            peer  
   host    all             all             all            md5
       
   # IPv4 local connections:  
   #host    all             all             127.0.0.1/32            scram-sha-256  
   host    all             all             0.0.0.0/0               md5
   ```  
   
   Localizar el  archivo postgresql.conf:  
   ```$ sudo find / -name postgresql.conf```  
   
   Editar el archivo postgresql.conf:  
   ```$ sudo vim /var/lib/pgsql/14/data/postgresql.conf```  
   
   Cambiar la siguiente línea para que se acceda universalmente desde cualquier dirección IP:  
   ```
   #listen_addresses = 'localhost'         # what IP address(es) to listen on;  
                                           # comma-separated list of addresses;  
                                           # defaults to 'localhost'; use '*' for all  
                                           # (change requires restart)  
   # @jzavalar:  
   listen_addresses = '*'          # what IP address(es) to listen on;  
     
   #port = 5432                            # (change requires restart)  
     
   # @jzavalar:  
   port = 5432                             # (change requires restart)  
   ```  

   Reiniciar postgresql:  
   ```$ sudo systemctl restart postgresql-14```  

   Comprobar el estatus del sevidor postgresql:  
   ```$ sudo systemctl status postgresql-14```  

   Fuente: *PostgreSQL en Red Hat*: [https://www.postgresql.org/download/linux/redhat/](https://www.postgresql.org/download/linux/redhat/)


4. Resetear la contraseña del usuario postgres (de ser necesario)

   Acceder a PostgreSQL con el usuario postgres:  
   ```$ psql -U postgres```  

   Si al ejecutar el comando anterior, el acceso se reporta el siguiente error:  
   ```FATAL:  password authentication failed for user "postgres"```  
   hay que seguir el siguiente procedimiento para restablecer la contraseña:  

   Localizar el archivo pg_hba.conf:  
   ```$ sudo find / -name pg_hba.conf```  

   Editar el archivo pg_hba.conf:  
   ```$ sudo vim /var/lib/pgsql/14/data/pg_hba.conf```  

   Cambiar de md5 a trust para las conexiones locales, para que se acceda sin contraseña y hacer el cambio:  
   ```
   #host    all             all             all            md5  
   host    all             all             all            trust  
   ```

   Reiniciar el servidor:  
   ```$ sudo systemctl restart postgresql```  

   Comprobar el estado del sevidor postgresql:  
   ```$ sudo systemctl status postgresql-14```  

   Acceder a PostgreSQL con el usuario postgres:  
   ```$ psql -U postgres```  

   Cambiar la contraseña:  
   ```# \password postgres```  

   Salir de psql:  
   ```# \q```  

   Acceder al archivo ```pg_hba.conf``` y regresar a su estado previo (autenticación por ```md5```), editando el archivo ```pg_hba.conf```:  
   ```$ sudo vim /var/lib/pgsql/14/data/pg_hba.conf```  
 
   Cambiar la línea:  
   ```
   #host    all             all             all            trust  
   host    all             all             all            md5  
   ```

   Reiniciar el servidor:   
   ```$ sudo systemctl restart postgresql```  

   Ahora nos podremos conectar a PostgreSQL con el usuario postgres:  
   ```$ psql -U postgres```  

   Fuente:  
   [https://www.codingnotebook.eu/postgresql-reset-password/](https://www.codingnotebook.eu/postgresql-reset-password/)


5. Creación de la bases de datos idempiere

   Acceder a PostgreSQL:  
   ```$ sudo -u postgres psql```  

   Crear la base de datos idempiere:  
   ```postgres=# create database idempiere;```  

   Crear el usuario con su contraseña:  
   ```postgres=# create user adempiere with encrypted password 'adempiere';```  

   Otorgar todos los privilegios sobre la base de datos, previamente creada:  
   ```postgres=# grant all privileges on database idempiere to adempiere;```  

   Salir:  
   ```postgres=# \q```


6. Instalar Java 11

   En Fedora 36 debe verfificarse los paquetes instalados de Java, porque instala por defecto el paquete más reciente de Java (Java 17) y no  Java 11.  

   Identificar los paquetes instalados de Java:  
   ```$ sudo yum list installed "java*"```  

   ```
   Installed Packages
   java-17-openjdk.x86_64                  1:17.0.3.0.7-2.fc36            @updates 
   java-17-openjdk-devel.x86_64            1:17.0.3.0.7-2.fc36            @updates 
   java-17-openjdk-headless.x86_64         1:17.0.3.0.7-2.fc36            @updates 
   javapackages-filesystem.noarch          6.0.0-7.fc36                   @anaconda  
   ```

   Verificar la versión activa de Java:  
   ```$ java -version```  

   Si tiene instalado algún paquete Java 17, similar a esto:  
   ```
   openjdk version "17.0.3" 2022-04-19  
   OpenJDK Runtime Environment 21.9 (build 17.0.3+7)  
   OpenJDK 64-Bit Server VM 21.9 (build 17.0.3+7, mixed mode, sharing)  
   ```
   
   Desinstalar Java 17:  
   ```
   $ sudo dnf remove jre-17-openjdk  
   $ sudo dnf remove java-17-openjdk  
   $ sudo dnf remove java-17-openjdk-headless
   ```  

   Instalar Java 11:  
   ```$ sudo dnf install java-11-openjdk java-11-openjdk-devel java-11-openjdk-headless```  

   Comprobar paquetes instalados:  
   ```$ sudo yum list installed "java*"```  

   Hasta que quede similar a esto:  
   ```
   Installed Packages  
   java-11-openjdk.x86_64                  1:11.0.15.0.10-1.fc36          @updates  
   java-11-openjdk-devel.x86_64            1:11.0.15.0.10-1.fc36          @updates  
   java-11-openjdk-headless.x86_64         1:11.0.15.0.10-1.fc36          @updates  
   javapackages-filesystem.noarch          6.0.0-7.fc36                   @anaconda  
   ```
   Comprobar los paquetes instalados y la versión de Java 11:  
   ```
   $ sudo yum list installed "java*"  
   $ java -version
   ```  

   Identificar el JAVA_HOME para configurar la ruta de acceso para su correcta ejecución:  
   ```$ java -XshowSettings:properties -version 2>&1 > /dev/null | grep 'java.home'```  

   Editar el archivo .bash_profile de root:  
   ```$ sudo vim /root/.bash_profile```  

   Insertar el siguiente texto:  
   ```
   JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.15.0.10-1.fc36.x86_64  
   PATH=$PATH:$HOME/bin:$JAVA_HOME/bin  
     
   export PATH
   ```  

   Recargar las definiciones en __~/.bash_profile__ de root:  
   ```$ sudo source /root/.bash_profile```  

   Probar Java  
   ```$ java -version```


7. Crear el usuario idempiere en Fedora 36

   Crear el usuario con el directorio ```home``` en ```/opt/idempiere-server```:  
   ```$ sudo useradd idempiere -m -d /opt/idempiere-server```  

   Asignar una contraseña:  
   ```$ sudo passwd idempiere```  

   Tomar el rol del usuario idempiere:  
   ```$ su - idempiere```  

   Cambiarse al usuario idempiere:  
   ```$ cd /opt/idempiere-server```  


### Instalar el servidor de aplicaciones iDempiere (Application Server)

1. Instalación gráfica  

   Ejecutar el instalador gráfico:  
   ```sh setup.sh```  
   o  
   ```sh setup-alt.sh```  

   *Parámetros:*  
   ```
   Java:  
      Java Home: ```/usr/lib/jvm/java-11-openjdk-11.0.15.0.10-1.fc36.x86_64```  

   iDempiere:  
       iDempiere Home: ```/opt/idempiere-server```  
       KeyStore Password: ```(creada por defecto)```  
 
   Application Server:  
       Application Server: ```0.0.0.0```  
       Web Port: ```8080```  
       SSL: ```8443```  

   Database Server:  
       DB Already Exists: ```(unchecked)```  
       Database Name: ```idempiere```  
       Database Port: ```5432```  
       Database User: ```adempiere```  
       DB Admin Password: ```postgres```  
       Database User: ```adempiere```  
       Database Password: ```adempiere```  
   ```
`
2. Importar la base de datos

   Como prerrequisito, después de instalar el servidor, se debe ejecutar el siquiente script, como usuario idempiere. Si no, debe cambiarse de usuario:  
   ```$ su - idempiere```  

   Cambiar de directorio:  
   ```$ cd /opt/idempiere-server/utils```  

   Ejecutar el script:  
   ```$ sh RUN_ImportIdempiere.sh```  


3. Actualizar la base de datos

   Se debe ejecutar el siquiente script, como usuario idempiere. Si no, debe cambiarse de usuario:  
   ```$ su - idempiere```  

   Cambiar de directorio:  
   ```$ cd /opt/idempiere-server/utils```  

   Ejecutar el script:  
   ```$ sh RUN_SyncDB.sh```  

   Abandonar el usuario:  
   ```$ exit```  


4. Registrar el código de la versión en la base de datos

   Se debe ejecutar el siquiente script, como usuario idempiere. Si no, debe cambiarse de usuario:  
   ```$ su - idempiere```  

   Cambiar de directorio:  
   ```$ cd /opt/idempiere-server```  

   Firmar la base de datos con el siguiente script:  
   ```$ sh sign-database-build-alt.sh```  

   Abandonar el usuario:  
   ```$ exit```  

   Fuente:  
   [https://wiki.idempiere.org/en/Installing_from_Installers](https://wiki.idempiere.org/en/Installing_from_Installers)
