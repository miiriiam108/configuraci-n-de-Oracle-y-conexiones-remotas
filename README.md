# configuraci-n-de-Oracle-y-conexiones-remotas

## Bloque 1. Entorno

### 1. Localizar los valores
1. Abrir las variables de entorno
   1. Pulso a la vez la tecla windows + r.
   2. Escribo: regedit y entro.
   3. He localizado los valores de Oracle en el registro de Windows con la ruta: `HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\KEY_OraDB21Home1`
      
  ![Captura 1](img/Captura%20de%20pantalla%202026-11-05%20105556.png)
     
2. Buscar las variables.
   1. Las variables encontradas son:

      - ORACLE_HOME = C:\app\X1704\product\21c\dbhomeXE -> Variable de entorno para especificar dónde está instalado SQL*Plus. 
      - ORACLE_BASE = C:\app\X1704\product\21c -> contiene la carpeta raíz de Oracle.
      - ORACLE_SID = XE -> Variable de entorno para especificar la instancia de la base de datos.

  ![Captura 2](img/Captura%20de%20pantalla%202026-11-05%20105910.png)

### 2. ¿Por qué se puede ejecutar sqlplus desde cualquier carpeta del dispositivo?

He ejecutado en el CMD el comando:

`path` 

y he comprobado que la carpeta:

`C:\app\X1704\product\21c\dbhomeXE\bin` 

está incluida en la variable PATH de Windows.

Esto permite que el sistema operativo encuentre el ejecutable sqlplus.exe sin necesidad de indicar la ruta completa, por lo que puede ejecutarse desde cualquier carpeta.

  ![Captura 4](img/Captura%20de%20pantalla%202026-11-10%20200438.png)


### 3. Localiza las entradas que controlan el arranque y la parada automáticos de tu instancia. Indica su valor y qué significa cada una.

Las entradas las encontramos en:

`HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\KEY_OraDB21Home1`

Los valores encontrados son:

- ORA_XE_AUTOSTART = TRUE

Indica la intancia Oracle XE se inicia automáticamente cuando arranca el sistema operativo.

- ORA_XE_SHUTDOWN = TRUE

Indica que Oracle XE se cierra automáticamente cuando se apaga el sistema operativo.

- ORA_XE_SHUTDOWN_TIMEOUT = 90

Indica el tiempo máximo de espera, en segundos, para que Oracle se cierre correctamente antes de forzar el cierre.

- ORA_XE_SHUTDOWNTYPE = immediate

Indica el tiempo de parada que realizará Oracle. El valor "immediate" ordena el cierre inmediato  de la instancia, finalizando las sesiones activas y realizando las operaciones necesarias antes de detener la base de datos.

  ![Captura 5](img/Captura%20de%20pantalla%202026-11-07%20104302.png)


### 4. Localiza los archivos listener.ora , sqlnet.ora y tnsnames.ora del servidor. ¿En qué carpeta están? ¿Para qué sirve cada uno?

He localizado los archivos en:

`C:\app\miria\product\21c\dbhomeXE\network\admin`

Los archivos contienen:

- listener.ora

Este fichero contiene la configuración del Listener de Oracle. El Listener es el servicio encargado de escuchar las peticiones de conexión de los clientes y dirigirlas a la instancia correspondiente de la base de datos.

- sqlnet.ora

Este fichero contiene parámetros de configuración de Oracle Net Services y permite definir aspectos relacionados con la comunicación entre cliente y servidor, así como la resolución de nombres utilizada durante las conexiones.

- tnsnames.ora

Este fichero contiene los alias y parámetros de conexión utilizados por los clientes para localizar y conectarse a una base de datos Oracle. En él se define el nombre del host, el puerto y el servicio de la base de datos.


## Bloque 2. Primera conexión y PDBs


### 5. Conéctate como administrador a la base de datos sin escribir usuario ni contraseña. Explica por qué es posible.

primero me identifico con el comando 

`whoami`

Para conectarme he utilizado

`sqlplus / as sysdba`

Y una vez dentro de sql me identifico 

`SHOW USER`

¿Por qué es posible?
Porque al escribir / en vez de el usuario y la contraseña, Oracle no valida la contraseña.
Con AS SYSDBA, Oracle comprueba que el usuario de Windows pertenece al grupo local, ORA_DBA, si pertenece, le concede el privilegio SYSDBA y entra como SYS


  ![Captura 6](img/Captura%20de%20pantalla%202026-11-10%20203134.png)


### 6. Muestra en qué contenedor estás, con qué usuario y cuál es el nombre de la base de datos.

- Para ver el contenedor:

He utilizado 

`show con_name`

y me ha salido el contendor CDB&ROOT que es el contenedor raíz de la base de datos.

- Para ver el usuario:

He vuelto a utilizar el comando

`show user`

- Para ver el nombre de la base de datos:

He utilizado el comando:

`select name from v&database;`

el XE es el nombre de la base de datos que corresponde con la CDB completa, no a una PDB.

  ![Captura 7](img/Captura%20de%20pantalla%202026-11-10%20203653.png)


### 7. Muestra las PDBs de tu base de datos y explica el estado de cada una.
La base de datos tiene dos PDBs:

- `PDB$SEED` (CON_ID 2): es la PDB plantilla que Oracle usa para crear
  nuevas PDBs. Está en `READ ONLY`, es decir, abierta solo para lectura,
  y no se trabaja en ella.
- `XEPDB1` (CON_ID 3): es la PDB de trabajo que crea el instalador de XE.
  Está en `READ WRITE`: abierta, y los usuarios pueden leer y escribir
  en ella.
Ambas tienen `RESTRICTED = NO`, lo que significa que no están en modo
restringido y pueden conectarse todos los usuarios con permisos, no solo
los administradores.


















## ERRORES
1. BLOQUE 1: He comprobado que las variables de Oracle no estaban definidas en las variables de entorno de Windows, por lo que las he localizado en el Registro de Windows, donde Oracle almacena parte de su configuración.
   
![Captura 3](img/Captura%20de%20pantalla%202026-11-05%20110325.png)

## BIBLIOGRAFÍA

- [Variables de entorno](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqpug/configuring-SQL-Plus.html#GUID-5A2953BF-9E2F-450B-AFBA-EE2846C59B5E)

- [Inicio y Detener Oracle Database XE](https://docs.oracle.com/en/database/oracle/oracle-database/21/xeinw/starting-and-stopping-oracle-database-xe.html)

- [Archivos del servidor](https://es.scribd.com/document/229452992/Ejemplo-Ficheros-SQL-Net)


