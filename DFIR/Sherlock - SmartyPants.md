# 🕵️ Análisis de Incidente: Sherlock - SmartPants (DFIR)

## 📌 Descripción del Caso
Investigación de una intrusión crítica en el servidor de archivos del CTO de la empresa Forela. El atacante accedió vía RDP, utilizó herramientas de búsqueda rápida para localizar documentos confidenciales, exfiltró la información a la nube y finalmente utilizó un "shredder" para destruir los archivos originales y borrar los logs de seguridad.

## 🛡️ Resumen Ejecutivo
* **Vector de Entrada:** Acceso remoto mediante RDP comprometido.
* **Fecha del Incidente:** 24 de enero de 2025.
* **Impacto:** Violación de confidencialidad de documentos del Ministerio de Defensa y presupuestos, seguida de destrucción de datos.
* **Herramientas del Atacante:** WinRAR, Everything (búsqueda), MEGAsync (exfiltración) y File Shredder (sabotaje).

---

## 🛠️ Metodología de Análisis

### 1. Análisis de Autenticación (RDP)
Se examinaron los registros de eventos de Windows (`Security.evtx` y `Microsoft-Windows-RemoteDesktopServices-RdpCoreTS/Operational`). Se identificó el inicio de sesión exitoso del atacante el **2025-01-24 a las 10:15:14**, marcando el inicio de la actividad maliciosa.

### 2. Visibilidad mediante SmartScreen y Logs de Ejecución
Gracias a la habilitación de los registros de depuración de **SmartScreen**, se pudo rastrear la descarga y ejecución de herramientas externas:
* **Everything.exe:** El atacante utilizó esta herramienta portátil para indexar y buscar archivos PDF con palabras clave como "Audit" y "Budget" de forma instantánea.
* **Exfiltración:** Se detectó la instalación y ejecución de **MEGAsync** a las 10:22:19, utilizada como canal de salida para subir los archivos robados a la nube.

### 3. Acciones de Sabotaje y Anti-Forense
El atacante intentó cubrir sus huellas y asegurar la extorsión mediante dos métodos:
* **Destrucción de Datos:** Uso de la utilidad **File Shredder** para eliminar los documentos de manera que no puedan ser recuperados mediante software forense convencional.
* **Borrado de Evidencia:** El atacante limpió el **Registro de Seguridad** (Event ID 1102) a las 10:28:41, intentando ocultar los logs de su inicio de sesión inicial.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Acceso Inicial** | External Remote Services (RDP) | T1133 |
| **Descubrimiento** | File and Directory Discovery (Everything.exe) | T1083 |
| **Exfiltración** | Exfiltration to Cloud Storage (MEGAsync) | T1567.002 |
| **Impacto / Anti-Forense** | Data Destruction / Indicator Removal | T1485 / T1070.001 |

---

## 🔍 Recomendaciones de Remediación

* **Hardening de RDP:** Deshabilitar el acceso RDP directo desde internet. Implementar el uso de una VPN con **Autenticación de Múltiple Factor (MFA)** para cualquier acceso administrativo remoto.
* **Restricción de Software:** Implementar políticas de control de aplicaciones (como AppLocker) para impedir la ejecución de herramientas portátiles no autorizadas (`Everything.exe`) o instaladores de almacenamiento en la nube (`MEGAsync`).
* **Monitoreo de Logs:** Configurar el envío de logs a un servidor centralizado (SIEM). Aunque el atacante borre los logs locales, la evidencia ya habrá sido enviada al servidor central, haciendo que el intento de ocultación sea inútil.
