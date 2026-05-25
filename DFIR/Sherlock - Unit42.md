# 🕵️ Análisis de Incidente: Sherlock - Unit42 (Malware Analysis)

## 📌 Descripción del Caso
Este laboratorio está inspirado en una investigación real de la Unidad 42 de Palo Alto sobre una campaña que utilizaba versiones modificadas de UltraVNC para mantener el acceso a los sistemas comprometidos. Para este ejercicio, se proporcionó un archivo comprimido (`unit42.zip`) que contenía el registro de eventos de Sysmon `Microsoft-Windows-Sysmon-Operational.evtx`, abarcando la etapa del acceso inicial del atacante en la máquina de la víctima.

## 🛡️ Resumen Ejecutivo
A través del análisis de los logs de Sysmon, se confirmó la ejecución de un archivo malicioso disfrazado que el usuario descargó desde un servicio de almacenamiento en la nube. El ejecutable realizó varias acciones defensivas para ocultar su presencia, incluyendo la alteración de fechas de creación de archivos (*Time Stomping*) y la creación de scripts ocultos en carpetas profundas del perfil de usuario. Finalmente, el proceso realizó validaciones de red consultando un dominio de prueba antes de finalizar su ejecución de manera controlada, dejando instalada la puerta trasera.

## 🛠️ Metodología de Análisis

Para resolver este caso, utilicé el **Visor de Eventos de Windows** aplicando filtros específicos basados en los EventIDs de Sysmon para seguir de forma lógica los pasos del malware.

### 1. Auditoría de Archivos Creados
Comencé analizando el impacto general en el disco utilizando el **Event ID 11 (File Created)**. Al filtrar por este identificador, encontré un total de **56 registros** de creación de archivos. Esto me dio el panorama inicial de la gran cantidad de elementos que el malware estaba desplegando en el sistema.

### 2. Identificación del Proceso Malicioso y Origen
Para descubrir qué programa inició la infección, utilicé el **Event ID 1 (Process Creation)**, el cual registra las rutas y líneas de comando ejecutadas. Detecté el proceso malicioso principal corriendo desde la carpeta de descargas con una doble extensión sospechosa para engañar al usuario:
* `C:\Users\CyberJunkie\Downloads\Preventivo24.02.14.exe.exe`

Cruzando la información de este proceso y revisando el tráfico o los eventos relacionados con el origen de la descarga, se descubrió que el atacante utilizó la plataforma de almacenamiento en la nube **Dropbox** para alojar y distribuir el archivo dañino.

### 3. Técnicas de Evasión: Manipulación de Tiempos (Time Stomping)
El malware intentó esconder los archivos que creaba aplicando *Time Stomping*. Esta técnica consiste en modificar las fechas reales de un archivo para que parezca antiguo y pase desapercibido entre los archivos legítimos del sistema operativo. 
Filtrando por el **Event ID 2 (File creation time changed)**, descubrí que el malware alteró la marca de tiempo de un archivo PDF ficticio, cambiándola exactamente al valor:
* `2024-01-14 08:10:06`

### 4. Persistencia y Despliegue de Scripts
Siguiendo el rastro de los archivos creados (Event ID 11), busqué dónde se estaban alojando scripts de ejecución automática. Localicé el archivo de comandos `once.cmd` escondido en una ruta muy profunda dentro de las carpetas de aplicación del usuario:
* `C:\Users\CyberJunkie\AppData\Roaming\Photo and Fax Vn\Photo and vn 1.1.2\install\F97891C\WindowsVolume\Games\once.cmd`

### 5. Actividad de Red y Cierre del Proceso
Para verificar si la máquina tenía salida a internet antes de desplegar el resto de sus funciones, el proceso malicioso intentó conectarse a un dominio de prueba legítimo (utilizado comúnmente para validar conectividad).
* **Filtro Usado:** **Event ID 22 (DNS Query)**
* **Dominio Consultado:** `www.example.com`
* **Dirección IP de Destino:** Al revisar la resolución y la conexión (**Event ID 3**), el proceso apuntó a la IP `93.184.216.34`.

Una vez que el malware cumplió su objetivo de infectar el equipo con la variante vulnerada de UltraVNC y asegurar la persistencia, el proceso principal se cerró solo. Filtrando por el **Event ID 5 (Process Terminated)**, registré la hora exacta del fin de la infección:
* `2024-02-14 03:41:58`

## 📊 Mapeo de Técnicas (MITRE ATT&CK)
* **T1204.002 - User Execution (Malicious File):** El usuario descargó y ejecutó el archivo fraudulento `Preventivo24.02.14.exe.exe` desde Dropbox.
* **T1070.006 - Indicator Removal (Timestomp):** Modificación del tiempo de creación de los archivos PDF generados para evadir la detección forense basada en antigüedad.
* **T1059.003 - Command and Scripting Interpreter (Windows Command Shell):** Uso de scripts `.cmd` (`once.cmd`) alojados en directorios del usuario para ejecutar comandos del sistema.
