# configuraci-n-de-Oracle-y-conexiones-remotas

## Bloque 1. Entorno

### 1. Localizar los valores
1. Abrir las variables de entorno
   1. Pulso a la vez la tecla windows + r.
   2. Escribo: regedit y entro.
   3. He localizado los valores de Oracle en el registro de Windows con la ruta: HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\KEY_OraDB21Home1
      
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

  ![Captura 4](img/Captura%20de%20pantalla%202026-11-05%20115324.png)


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

  ![Captura 5](img/Captura%20de%20pantalla%202026-11-07%20115324.png)

## ERRORES
1. BLOQUE 1: He comprobado que las variables de Oracle no estaban definidas en las variables de entorno de Windows, por lo que las he localizado en el Registro de Windows, donde Oracle almacena parte de su configuración.
   
![Captura 3](img/Captura%20de%20pantalla%202026-11-05%20110325.png)

## BIBLIOGRAFÍA

[Variables de entorno](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqpug/configuring-SQL-Plus.html#GUID-5A2953BF-9E2F-450B-AFBA-EE2846C59B5E)


