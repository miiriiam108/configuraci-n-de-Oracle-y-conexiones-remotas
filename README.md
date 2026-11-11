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

  ![Captura 8](img/Captura%20de%20pantalla%202026-11-10%20204614.png)


### 8. Sin desconectarte, entra en tu PDB de trabajo, demuestra que estás dentro y vuelve a la raíz. ¿Qué nombre de base de datos se obtiene dentro de la PDB? Explícalo


Con `ALTER SESSION SET CONTAINER=XEPDB1` cambio de contenedor sin cerrar
la sesión. `SHOW CON_NAME` demuestra que estoy dentro (`XEPDB1`), y con
el mismo comando apuntando a `CDB$ROOT` vuelvo a la raíz. El usuario
sigue siendo `SYS`.
Dentro de la PDB, `SELECT NAME FROM V$DATABASE` devuelve `XE`, no
`XEPDB1`. Esto es porque `V$DATABASE` muestra el nombre de la base de
datos, la CDB, que es única y contiene a todas las PDBs. La PDB es
un contenedor lógico dentro de ella, y su nombre se ve con `SHOW CON_NAME`.

  ![Captura 9](img/Captura%20de%20pantalla%202026-11-10%20205339.png)
  ![Captura 10](img/Captura%20de%20pantalla%202026-11-10%20205350.png)


## Bloque 3: La instancia

### 9. Localiza el SPFILE de tu instancia. ¿Cómo se llama y dónde está? ¿Por qué no debe editarse con un editor de texto?

El SPFILE se llama `SPFILEXE.ORA` y está en `C:\APP\MIRIA\PRODUCT\21C\DATABASE\`. Es el archivo de parámetros de inicialización que la instancia lee al arrancar.
No debe editarse con un editor de texto porque es un archivo binario: un
editor puede alterar su contenido al guardarlo y corromperlo, y la
instancia no podría leer sus parámetros al arrancar. Los cambios se hacen
desde Oracle con `ALTER SYSTEM SET`, que valida el parámetro y lo escribe
de forma segura. Para ver su contenido en texto se genera una copia legible
con `CREATE PFILE FROM SPFILE`.

  ![Captura 11](img/Captura%20de%20pantalla%202026-11-10%20205947.png)
  ![Captura 12](img/Captura%20de%20pantalla%202026-11-10%20210110.png)


### 10. Genera una copia legible del SPFILE en una carpeta tuya y muestra en ella los valores de db_name , processes y open_cursors .

La orden CREATE PFILE='C:\practica\init_xe_copia.ora' FROM SPFILE; genera un archivo de texto plano (PFILE) con todos los parámetros de inicialización que contiene el SPFILE de la instancia. El SPFILE (SPFILEXE.ORA) es binario y no se puede leer ni editar con un editor de texto, así que esta copia legible permite ver su contenido sin tocar el original. El SPFILE no se modifica: sigue teniendo el mismo tamaño y la misma fecha.
Con findstr se buscan en la copia los tres parámetros pedidos:
- db_name = 'XE': nombre de la base de datos (la CDB).


- open_cursors = 300: número máximo de cursores abiertos por sesión.


- processes = 300: número máximo de procesos del sistema operativo que pueden conectarse a la instancia.


  ![Captura 13](img/Captura%20de%20pantalla%202026-11-10%20211003.png)
  ![Captura 14](img/Captura%20de%20pantalla%202026-11-10%20211208.png)


### 11. Indica cuál de los parámetros open_cursors y processes puede cambiarse sin reiniciar la instancia. Justifícalo con una consulta al diccionario.

La columna `ISSYS_MODIFIABLE` indica si un parámetro puede cambiarse con
`ALTER SYSTEM` sin reiniciar: `IMMEDIATE` se aplica ya. En mi instancia `open_cursors`
aparece como `IMMEDIATE`, por lo que puede cambiarse sin reiniciar.
`processes` también aparece como `IMMEDIATE` en el diccionario, lo cual no
es lo normal.

  ![Captura 15](img/Captura%20de%20pantalla%202026-11-10%20212801.png)

### 12. Aumenta open_cursors un 25 % de forma inmediata y permanente. Demuestra que el cambio está en uso y guardado

El valor original de `open_cursors` era 300. Un aumento del 25 % es 375. Usé `ALTER SYSTEM SET ... SCOPE=BOTH`: `MEMORY` lo aplica en la instancia en ejecución sin reiniciar y `SPFILE` lo escribe en el archivo de parámetros, y `BOTH` hace las dos cosas. Tras el cambio, `SHOW PARAMETER` y `V$SYSTEM_PARAMETER` muestran 375
y `SHOW SPPARAMETER` y `V$SPPARAMETER` muestran 375. Además, la fecha de `SPFILEXE.ORA` cambió, lo que confirma que se reescribió.



### 13. Aumenta processes en 20 de forma permanente. Muestra el valor en uso y el guardado antes y después de reiniciar, y explica la diferencia


### 14. Cambia el formato de fecha a DD/MM/YYYY HH24:MI:SS solo para tu sesión. Demuestra que el cambio funciona y que se pierde al volver a conectar


### 15. Deja open_cursors y processes con sus valores originales y demuéstralo





























## ERRORES
1. BLOQUE 1: He comprobado que las variables de Oracle no estaban definidas en las variables de entorno de Windows, por lo que las he localizado en el Registro de Windows, donde Oracle almacena parte de su configuración.
   
![Captura 3](img/Captura%20de%20pantalla%202026-11-05%20110325.png)

## BIBLIOGRAFÍA

- [Variables de entorno](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqpug/configuring-SQL-Plus.html#GUID-5A2953BF-9E2F-450B-AFBA-EE2846C59B5E)

- [Inicio y Detener Oracle Database XE](https://docs.oracle.com/en/database/oracle/oracle-database/21/xeinw/starting-and-stopping-oracle-database-xe.html)

- [Archivos del servidor](https://es.scribd.com/document/229452992/Ejemplo-Ficheros-SQL-Net)


