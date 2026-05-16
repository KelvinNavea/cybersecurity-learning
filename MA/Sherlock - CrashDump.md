# 🕵️ Análisis de Incidente: Sherlock - CrashDump (Malware Analysis)

## 📌 Descripción del Caso
Se identificó un archivo ejecutable sospechoso que se estaba corriendo en una de las computadoras comprometidas de la red. Al principio se sospechó que el atacante usó este programa para meterse y quedarse dentro del sistema. Para poder investigar bien qué hacía y cómo se comportaba, se descargó un volcado de memoria (un archivo dump de tipo CrashDump.zip de 61 MB) del proceso para analizarlo a fondo.

## 🛡️ Resumen Ejecutivo
Después de revisar el volcado de memoria del proceso, pude confirmar que el sistema fue comprometido con éxito. El programa malicioso no solo se ejecutó, sino que logró conectarse con un servidor externo (C2) controlado por el atacante para recibir órdenes. En la investigación se encontró la ruta de dónde salió el virus, cómo se comunicaba internamente y la forma en que ocultó su código adentro de otro programa del sistema operativo que sí era confiable para no ser detectado.

## 🛠️ Metodología de Análisis

### 1. Datos del Sistema y de dónde salió el archivo
Lo primero que hice fue revisar los datos básicos del sistema dentro del archivo dump. Pude identificar que la computadora usaba la versión **10.0.10240.16384** de Windows. Buscando el origen del problema, encontré la ruta completa desde donde se ejecutó el archivo malicioso:
* `C:\Users\s1rx\Downloads\update.exe`

Al mirar más adentro, vi que este programa levantó un total de **6 hilos** (subprocesos) para realizar sus tareas al mismo tiempo.

### 2. Comunicación interna del programa (IPC)
Para comunicarse entre sus propios componentes internos o sincronizarse, el ejecutable creó una tubería con nombre (Named Pipe). Descubrí que usaba este canal específico:
* `\\.\pipe\MSSE-1641-server`

### 3. Ocultamiento e Inyección de Código
Para que el antivirus o el usuario no sospechen, el malware usó una técnica para meter su código adentro de otro proceso legítimo que ya estaba corriendo. Al investigar esto encontré:
* **El proceso afectado (PID):** El código se inyectó en el proceso con el identificador **2336** (en formato decimal).
* **Tiempo de ejecución:** Pude ver que el último hilo o tarea que se creó para este código inyectado se armó el **2025-11-05 01:09:12 UTC**.

### 4. Encontrando la IP del atacante y el Framework utilizado
Mirando con mucha atención la memoria del proceso inyectado, encontré la dirección exacta de memoria donde arrancaba el código oculto (el shellcode):
* **Dirección Base:** `b1`20870000`

Al hacer un volcado visual de esa zona de memoria para leer el texto plano que había adentro, logré extraer los datos clave que nos dicen a dónde se comunicaba:
* **Dirección IP del Servidor C2:** `101.10.25.4`
* **Herramienta del atacante:** Por la forma en que estaban ordenados los datos y por el "User-Agent" que simulaba ser un navegador viejo, se pudo confirmar que el atacante usó un framework de post-explotación muy conocido llamado **Cobalt Strike**.

## 📊 Mapeo de Técnicas (MITRE ATT&CK)
* **T1055 - Inyección de Procesos:** El malware metió su código en otro proceso (`PID 2336`) para esconderse y que el sistema confiara en él.
* **T1071.001 - Protocolos de la Capa de Aplicación (Tráfico Web):** El programa simulaba ser un navegador web común y corriente para mandar señales y hablar con el servidor del atacante.
* **T1573 - Canal Cifrado:** El uso de Cobalt Strike para asegurarse de que las órdenes que mandaba el atacante viajen encriptadas y no se puedan leer fácil en la red.

## 🔍 Recomendaciones de Remediación
1. **Desconectar el equipo:** Aislar la computadora afectada de la red para que el atacante pierda el control y no intente meterse en otras máquinas.
2. **Bloquear la IP en el Firewall:** Configurar el firewall o proxy para bloquear cualquier conexión que vaya o venga de la IP `101.10.25.4`.
3. **Borrar el archivo:** Eliminar por completo el ejecutable `update.exe` de la carpeta de descargas del usuario y realizar un escaneo de antivirus/EDR.
4. **Protección de memoria:** Revisar las políticas de seguridad del sistema para evitar que programas desconocidos tengan permisos de escribir y ejecutar código directamente en la memoria de otros procesos del sistema.
