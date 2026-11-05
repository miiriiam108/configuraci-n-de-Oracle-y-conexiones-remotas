# configuraci-n-de-Oracle-y-conexiones-remotas

## Bloque 1. Entorno
1. Abrir las variables de entorno
   1. Pulso a la vez la tecla windows + r.
   2. Escribo: regedit y entro.
      
  ![Captura 1](img/Captura%20de%20pantalla%202026-11-05%20105556.png)

   3. He localizado los valores de Oracle en el registro de Windows con la ruta:
     HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\KEY_OraDB21Home1
2. Buscar las variables.
   1. Las variables encontradas son:

      - ORACLE_HOME = C:\app\X1704\product\21c\dbhomeXE -> contiene dónde está instalado Oracle.
      - ORACLE_BASE = C:\app\X1704\product\21c -> contiene la carpeta raíz de Oracle.
      - ORACLE_SID = XE -> contiene el identificador de la instancia

  ![Captura 2](img/Captura%20de%20pantalla%202026-11-05%20105910.png)





## ERRORES
1. BLOQUE 1: He comprobado que las variables de Oracle no estaban definidas en las variables de entorno de Windows, por lo que las he localizado en el Registro de Windows, donde Oracle almacena parte de su configuración.
