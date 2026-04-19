# Reporte de Análisis: Sherlock - MangoBleed (HTB)

## 1. Escenario del Incidente
Se recibió una alerta sobre el servidor `mongodbsync` (MongoDB secundario) por una posible explotación de la vulnerabilidad conocida como **MongoBleed**. 
Como analista DFIR, el objetivo es analizar el triaje del sistema para identificar el acceso inicial, la actividad del atacante y los posibles intentos de exfiltración de datos.

## 2. Resumen de Hallazgos

| Ítem | Información Identificada |
| :--- | :--- |
| **CVE Identificado** | `CVE-2025-14847` |
| **Versión de MongoDB** | `8.0.16` |
| **IP del Atacante** | `65.0.76.43` |
| **Inicio del Ataque (UTC)** | `2025-12-29 05:25:52` |
| **Conexiones Maliciosas** | `75260` total |
| **Acceso Remoto Exitoso** | `2025-12-29 05:40:03` |
| **Directorio Objetivo** | `/var/lib/mongodb` |

## 3. Análisis Técnico (Metodología)

### Fase 1: Identificación de la Vulnerabilidad
El análisis comenzó investigando la vulnerabilidad **MongoBleed**. Esta vulnerabilidad afecta a versiones específicas de MongoDB y permite la lectura de memoria del proceso, 
lo que puede exponer credenciales o datos sensibles. Se confirmó la versión instalada en el sistema para validar la superficie de ataque.

### Fase 2: Análisis de Logs de MongoDB
Se analizaron los logs ubicados en la carpeta `system` / `var/log/mongodb/`. 
* **Identificación de la IP:** Se filtraron las conexiones entrantes para aislar la IP de origen que realizó peticiones anómalas consistentes con el exploit de MongoBleed.
* **Timeline:** Se reconstruyó la línea de tiempo detectando el primer evento malicioso, seguido de una serie de conexiones rápidas que indican un proceso automatizado o de fuerza bruta.

### Fase 3: Escalada de Privilegios y Persistencia
Tras el compromiso del servicio MongoDB, el atacante logró obtener acceso interactivo al sistema. El análisis de los artefactos de `live_response` y logs de comandos permitió identificar:
* La ejecución de un **script in-memory** (sin escribir en disco) para elevar privilegios.
* El uso de un servidor web temporal en **Python** para preparar la exfiltración de un directorio específico identificado en el sistema de archivos.

## 4. Mapeo MITRE ATT&CK
* **Initial Access:** T1190 - Exploit Public-Facing Application (MongoBleed).
* **Privilege Escalation:** T1059.004 - Command and Scripting Interpreter: Unix Shell.
* **Exfiltration:** T1567 - Exfiltration Over Web Service (Python SimpleHTTPServer).

---
**Nota de Analista:** Es imperativo parchear la instancia de MongoDB a una versión no vulnerable y revisar las 
reglas de firewall para limitar el acceso al puerto de la base de datos solo a IPs autorizadas.
