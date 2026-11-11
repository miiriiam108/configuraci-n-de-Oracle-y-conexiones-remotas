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

El valor original de `open_cursors` era 300. Un aumento del 25 % es 375. Usé `ALTER SYSTEM SET ... SCOPE=BOTH`: `MEMORY` lo aplica en la instancia en ejecución sin reiniciar y `SPFILE` lo escribe en el archivo de parámetros, y `BOTH` hace las dos cosas. Después del cambio, `SHOW PARAMETER` y `V$SYSTEM_PARAMETER` muestran 375. Además, la fecha de `SPFILEXE.ORA` cambió, lo que confirma que se reescribió.

  ![Captura 16](img/Captura%20de%20pantalla%202026-11-11%20114303.png)


### 13. Aumenta processes en 20 de forma permanente. Muestra el valor en uso y el guardado antes y después de reiniciar, y explica la diferencia

**Explicación:**
El valor original de `processes` era 300, tanto en uso como guardado. Con un aumento de 20, es 320. Para hacerlo permanente he usado `ALTER SYSTEM SET processes=320 SCOPE=SPFILE;`, que escribe el valor en el SPFILE sin aplicarlo a la instancia en marcha.
Antes de reiniciar, `SHOW PARAMETER` seguía en 300, y `SHOW SPPARAMETER` ya era 320. Después lo he reiniciado con `SHUTDOWN IMMEDIATE` y `STARTUP`. Después del reinicio, la PDB `XEPDB1` quedó en `READ WRITE` sin necesidad de abrirla, y el valor en uso y el guardado pasaron a ser 320. La diferencia se debe a que `processes` dimensiona recursos que Oracle reserva al arrancar, por lo que el cambio solo tiene efecto tras reiniciar. Aunque el diccionario indicaba `IMMEDIATE`, la prueba real mostró que exige reinicio. `open_cursors`.

  ![Captura 17](img/Captura%20de%20pantalla%202026-11-11%20115411.png)
  
  ![Captura 18](img/Captura%20de%20pantalla%202026-11-11%20115433.png)
  
  ![Captura 19](img/Captura%20de%20pantalla%202026-11-11%20115449.png)
  
  ![Captura 20](img/Captura%20de%20pantalla%202026-11-11%20120451.png)
  
  ![Captura 21](img/Captura%20de%20pantalla%202026-11-11%20120528.png)


### 14. Cambia el formato de fecha a DD/MM/YYYY HH24:MI:SS solo para tu sesión. Demuestra que el cambio funciona y que se pierde al volver a conectar

Con `ALTER SESSION SET NLS_DATE_FORMAT` cambié el formato de fecha solo para mi sesión. Antes del cambio, `SYSDATE` se mostraba como formato original. Después, aparece como formato nuevo, con año de cuatro cifras y hora. El cambio solo afecta a mi sesión porque los parámetros NLS de sesión se guardan en memoria mientras dura la conexión y no se escriben en el SPFILE. Al reconectar con `CONNECT`, Oracle crea una sesión nueva con los valores por defecto, y `SYSDATE` volvió a mostrarse con el formato original, con lo que el cambio se ha perdido.

  ![Captura 22](img/Captura%20de%20pantalla%202026-11-11%20121938.png)
  
  ![Captura 23](img/Captura%20de%20pantalla%202026-11-11%20122039.png)


### 15. Deja open_cursors y processes con sus valores originales y demuéstralo

Los valores originales, eran `open_cursors` = 300 y `processes` = 300.Luego han pasado a 375 y 320. `open_cursors` se restauró con `ALTER SYSTEM ... SCOPE=BOTH`, que lo ha aplicado al momento y lo ha guardado. `processes` se restauró con `SCOPE=SPFILE`: antes de reiniciar el valor en uso seguía en 320 y el guardado ya era 300. Tras `SHUTDOWN IMMEDIATE` y `STARTUP`, la instancia leyó el SPFILE y aplicó 300. Ahora los dos parámetros valen 300 en uso y guardados, igual que al principio

  ![Captura 24](img/Captura%20de%20pantalla%202026-11-11%20122825.png)
  ![Captura 25](img/Captura%20de%20pantalla%202026-11-11%20123108.png)
  ![Captura 26](img/Captura%20de%20pantalla%202026-11-11%20123233.png)
  ![Captura 27](img/Captura%20de%20pantalla%202026-11-11%20123313.png)
  ![Captura 28](img/Captura%20de%20pantalla%202026-11-11%20123337.png)


## Bloque 4: Cuentas de administración

### 16. Muestra en qué contenedores existen las cuentas SYS, SYSTEM y PDBADMIN, su estado y si son comunes o locales. Explica la diferencia entre las tres.

- SYS:  Todas las tablas y vistas para el diccionario de datos de la base de datos están almacenados en el esquema SYS. Estas tablas y vistas son críticas para el funcionamiento de la base de datos ORACLE. Para mantener la integridad del diccionario de datos, las tablas del esquema SYS son manipulados solo por la base de datos. Nunca se debería modificar algo o crear tablas en el esquema del usuario SYS

- SYSTEM: El usuario SYSTEM se utiliza para crear tablas y vistas adicionales que muestran información administrativa, tablas internas y vistas utilizado por varias opciones y herramientas de la base de datos ORACLE. No se recomienda utilizar el esquema SYSTEM para almacenar tablas de interés para usuarios no administrativos. Esta cuenta puede realizar todas las funciones administrativas excepto las siguientes: •Copia de seguridad y recuperación. •Actualización de la base de datos.

- PDBADMIN, este es un rol específico de Oracle que se utiliza en un entorno de base de datos contenedor con múltiples bases de datos contenedor plugables (PDB). Este rol se crea automáticamente cuando se crea una base de datos contenedor. Los usuarios que tienen el rol PDBAADMN pueden administrar todas las bases de datos contenedor, pero no tienen acceso a las operaciones de SYSDBA o SYSOPER en la base de datos contenedor raíz. Este rol facilita la gestión de entornos de base de datos con múltiples PDB. PDBAADMN es el administrador de la base de datos local (PDB) y tiene privilegios para administrar esa base de datos específica dentro del contenedor. Sus privilegios están restringidos al ámbito de la PDB asignada.

  ![Captura 29](img/Captura%20de%20pantalla%202026-11-11%20124611.png)

### 17. Comprueba si SYSTEM, conectado sin privilegio SYSDBA, puede parar la base de datos. Muestra el resultado y explícalo. Atención: asegúrate de que no te has conectado AS SYSDBA antes de intentarlo.

Me he conectado con `CONNECT SYSTEM`. Al ejecutar `SHUTDOWN IMMEDIATE`, Oracle respondió con un error y la base de datos no se ha parado: `V$INSTANCE` sigue mostrando estado `OPEN`. Esto ocurre porque parar y arrancar la base de datos exige el privilegio SYSDBA o SYSOPER, y SYSTEM, aunque es una cuenta administrativa, no lo tiene en una conexión normal. SYSTEM puede hacer todas las funciones administrativas excepto arrancar y parar, copias de seguridad y recuperación, y actualización de la base de datos.

  ![Captura 30](img/Captura%20de%20pantalla%202026-11-11%20125700.png)
  ![Captura 31](img/Captura%20de%20pantalla%202026-11-11%20125710.png)


### 18. En el servidor, conéctate como SYSTEM AS SYSDBA escribiendo una contraseña incorrecta. ¿Has podido entrar? ¿Con qué usuario? Explica las dos cosas

- **¿Has podido entrar?** Sí. Oracle se ha conectado aunque la contraseña era incorrecta. Con `AS SYSDBA`, desde el propio servidor, Oracle usa la autenticación del sistema operativo: mi usuario de Windows (`mv-miriam\miria`) pertenece al grupo `ORA_DBA`, por lo que recibe el privilegio SYSDBA y Oracle no valida la contraseña escrita.


- **¿Con qué usuario?** Con `SYS`, no con `SYSTEM`. `SHOW USER` y `SESSION_USER` devuelven `SYS`. Con el privilegio SYSDBA siempre se entra como `SYS`, sea cual sea el nombre escrito.

  ![Captura 32](img/Captura%20de%20pantalla%202026-11-11%20130611.png)


## Bloque 5: Servidor accesible por red


### 19. Indica la IP del servidor y el tipo de red de la máquina virtual. Demuestra que servidor y cliente están en la misma red

He añadido un Adaptador 2 en modo "solo anfitrión", que tiene la IP `192.168.56.101` con máscara `255.255.255.0`. Esa es
la IP del servidor para la práctica. El cliente (equipo anfitrión) tiene la IP `192.168.56.1` y la misma máscara
en el adaptador "VirtualBox Host-Only". Ambas direcciones comparten la red `192.168.56.0/24`, por lo que están en la misma red.
El `ping` desde el cliente agotó el tiempo de espera, pero eso no indica redes distintas: lo más probable es que el cortafuegos de Windows del
servidor bloquee el eco ICMP. Para confirmar la conectividad, he visto la tabla ARP del cliente: `arp -a 192.168.56.101` devuelve la MAC
`08-00-27-ce-cf-c2`, que coincide con la del Adaptador 2 de la VM. Esto demuestra que el cliente alcanza al servidor a
nivel de red. 

  ![Captura 33](img/Captura%20de%20pantalla%202026-11-11%20172001.png)
  ![Captura 34](img/Captura%20de%20pantalla%202026-11-11%20171950.png)
  ![Captura 35](img/Captura%20de%20pantalla%202026-11-11%20172334.png)
  ![Captura 36](img/Captura%20de%20pantalla%202026-11-11%20172357.png)
  ![Captura 37](img/Captura%20de%20pantalla%202026-11-11%20173055.png)
 
### 20. Comprueba que el listener está arrancado, en qué dirección y puerto escucha y qué servicios conoce. Si solo admite conexiones locales, corrígelo y documenta el antes y el después

El listener está iniciado y utiliza el archivo: `C:\app\miria\product\21c\homes\OraDB21Home1\network\admin\listener.ora`. 
Escucha las conexiones por el puerto 1521. Para comprobar si acepta conexiones desde otros equipos, he ejecutado el comando: `netstat -an | findstr 1521` y de resultado me ha salido `0.0.0.0:1521`.
Esto significa que el listener está escuchando en todas las interfaces de red del servidor, incluida la dirección 192.168.56.101 de la red solo anfitrión.
Por tanto, el listener permite conexiones desde otros equipos de la red y no ha sido necesario realizar ninguna modificación en la configuración, ya que funciona correctamente.

  ![Captura 38](img/Captura%20de%20pantalla%202026-11-11%20173841.png)
  ![Captura 39](img/Captura%20de%20pantalla%202026-11-11%20173857.png)
  ![Captura 40](img/Captura%20de%20pantalla%202026-11-11%20173934.png)
  ![Captura 41](img/Captura%20de%20pantalla%202026-11-11%20174103.png)
  ![Captura 42](img/Captura%20de%20pantalla%202026-11-11%20174112.png)

### 21. Configura el cortafuegos del servidor para que acepte conexiones al listener solo desde la red de las máquinas virtuales.

He comprobado la configuración con: `netsh advfirewall firewall show rule name="Oracle 1521 red VM"`. Con esta configuración, solo los equipos de la red 192.168.56.0/24 pueden conectarse al puerto 1521.

  ![Captura 43](img/Captura%20de%20pantalla%202026-11-11%20175511.png)

### 22. Desde el propio servidor, conéctate a la PDB usando su IP (no localhost ) y comprueba que el listener responde

En el servidor ejecuto: `sqlplus system@//192.168.56.101:1521/XEPDB1` y me pidió la contraseña de SYSTEM. Una vez que ya estaba dentro para ver la PBD que utiliza lo hice con: `show con_name;`

  ![Captura 44](img/Captura%20de%20pantalla%202026-11-11%20180219.png)


## Bloque 6: Cliente remoto


### 23. Desde el cliente, comprueba que el puerto del listener es accesible

En cliente he ejecutado: `telnet 192.168.56.101 1521`

  ![Captura 45](img/Captura%20de%20pantalla%202026-11-11%20182029.png)


### 24. Conéctate desde el cliente con SQL*Plus a la PDB como SYSTEM. Demuestra con una consulta en qué contenedor estás y desde qué IP te ve el servidor.

Desde cliente he ejecutado: `sqlplus system/8888@//192.168.56.101:1521/XEPDB1`, y una vez que estaba dentro he comprobado el contenedor con `show con_name` y por último he comprobado la dirección ip que me ve el servidor `SELECT SYS*CONTEXT('USERENV','IP_ADDRESS*)*FROM dual;`

  ![Captura 50](img/Captura%20de%20pantalla%202026-11-11%20192456.png)


### 25. Conéctate desde el cliente a la PDB como SYS. ¿Qué te hace falta que en la tarea 5 no necesitabas?

Desde cliente he ejecutado: `sqlplus sys/8888@//192.168.56.101:1521/XEPDB1 as sysdba` y la conexión se ha realizado correctamente.

En la tarea 5 me conecté utilizando `sqlplus / as sysdba` sin necesidad de indicar usuario ni contraseña, ya que la autenticación se realizaba mediante el sistema operativo del servidor.

Sin embargo, al conectarme desde un equipo cliente ha sido necesario indicar, el usuario (SYS), la contraseña, la dirección IP del servidor, el servicio (XEPDB1) y el privilegio AS SYSDBA.

Esto se debe a que la conexión se realiza de forma remota y Oracle no puede utilizar la autenticación del sistema operativo del servidor.

  ![Captura 51](img/Captura%20de%20pantalla%202026-11-11%20193150.png)


### 26. Repite la tarea 18 desde el cliente. Compara el resultado con el de la tarea 18 y explica la diferencia.

Desde cliente he ejecutado: `sqlplus system/1234@//192.168.56.101:1521/XEPDB1 as sysdba`utilizando una contraseña que no es correcta y como se esperaba Oracle rechazó la conexión mostrando un error. 

En comparación de la tarea 8, que la realicé en el servidor, accedí con una contraseña incorrecta porque Oracle utilizada la autenticación del sistema operativo del propio servidor.

  ![Captura 52](img/Captura%20de%20pantalla%202026-11-11%20193758.png)


  ### 27. Configura el cliente para conectarte mediante alias: uno para la CDB y otro, con el nombre que quieras, para la PDB. Conéctate con los dos y demuestra en qué contenedor entras con cada uno.

Modifiqué el fichero tnsnames.ora y añadí dos alias:

- MIRIAM_CDB para la base de datos XE.
- MIRIAM_PDB para la PDB XEPDB1.

Luego para conectarme a MIRIAM_CDB he utilizado: `sqlplus system/8888@MIRIAM_CDB`y para comprobar el contenedor `show con_name;`.

He seguido los mismo pasos para MIRIAM_PDB

  ![Captura 53](img/Captura%20de%20pantalla%202026-11-11%20194249.png)
  ![Captura 54](img/Captura%20de%20pantalla%202026-11-11%20194321.png)
  ![Captura 55](img/Captura%20de%20pantalla%202026-11-11%20194344.png)


## ERRORES
1. BLOQUE 1: He comprobado que las variables de Oracle no estaban definidas en las variables de entorno de Windows, por lo que las he localizado en el Registro de Windows, donde Oracle almacena parte de su configuración.
   
![Captura 3](img/Captura%20de%20pantalla%202026-11-05%20110325.png)


2. BLOQUE 6 EJERCICIO 24: Al intentar conectarme desde el cliente a la base de datos mediante SQL*Plus con el comando: `sqlplus system/8888@//192.168.56.101:1521/XEPDB1` y me aparecía el error `ORA-12638: Fallo de recuperación de credenciales` y era porque el cliente Oracle estaba configurado para utilizar autenticación de Windows con la línea `SQLNET.AUTHENTICATION_SERVICES = (NTS)`.
Para solucionarlo he cambiado esa línea por: `SQLNET.AUTHENTICATION_SERVICES = (NONE)` y con esta modificación se deshailita la autenticación de Windows.

  ![Captura 46](img/Captura%20de%20pantalla%202026-11-11%20191723.png)
  ![Captura 47](img/Captura%20de%20pantalla%202026-11-11%20191621.png)
  ![Captura 48](img/Captura%20de%20pantalla%202026-11-11%20192858.png)
  ![Captura 49](img/Captura%20de%20pantalla%202026-11-11%20192322.png)




## BIBLIOGRAFÍA

- [Variables de entorno](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqpug/configuring-SQL-Plus.html#GUID-5A2953BF-9E2F-450B-AFBA-EE2846C59B5E)

- [Inicio y Detener Oracle Database XE](https://docs.oracle.com/en/database/oracle/oracle-database/21/xeinw/starting-and-stopping-oracle-database-xe.html)

- [Archivos del servidor](https://es.scribd.com/document/229452992/Ejemplo-Ficheros-SQL-Net)

- [SYS, SYSTEM, PDBADMIN](https://www.collegesidekick.com/study-docs/15068978)



