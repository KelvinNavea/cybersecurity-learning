# 🕵️ Análisis de Incidente: Sherlock - Forty Seven (CTI)

## 📌 Descripción del Caso
Investigación de una campaña de ciberespionaje dirigida por el actor de amenazas **APT-K-47** (también conocido como *Mysterious Elephant*). El análisis se centra en la cadena de infección detectada en enero de 2024, la cual utiliza técnicas de ingeniería social y explotación de vulnerabilidades en herramientas de compresión para comprometer objetivos específicos.

## 🛡️ Resumen Ejecutivo
* **Actor de Amenaza:** APT-K-47 / Mysterious Elephant.
* **Vector de Entrada:** Spear-phishing vía mensajería instantánea (WhatsApp) y correo electrónico.
* **Vulnerabilidad Explotada:** CVE-2023-38831 (WinRAR).
* **Objetivo del Ataque:** Despliegue de troyanos de acceso remoto (RAT) para espionaje y exfiltración de documentos.

---

## 🛠️ Metodología de Análisis

### 1. Análisis del Vector de Acceso Inicial
Se identificó el uso de archivos comprimidos (`.zip`) con nombres temáticos para engañar al usuario. La investigación reveló que el atacante aprovechó la vulnerabilidad **CVE-2023-38831**, la cual permite ejecutar código malicioso cuando un usuario intenta abrir un archivo legítimo dentro de un archivo ZIP especialmente manipulado.

### 2. Cadena de Ejecución y Evasión
Una vez ejecutado el vector inicial, se activó una secuencia de comandos (`.bat` y `.lnk`) diseñada para:
* **Evasión de Sandbox:** El malware (MemLoader) cuenta los procesos activos del sistema; si detecta un número bajo (típico de entornos virtuales de análisis), detiene su ejecución para no ser descubierto.
* **Carga de Payload:** Despliegue de **Asyncshell-v2**, un implante que evolucionó sus comunicaciones de protocolos TCP crudos hacia HTTPS para mimetizarse con el tráfico web legítimo.

### 3. Persistencia y Comando y Control (C2)
El análisis forense en el host mostró que el atacante estableció persistencia mediante la modificación de llaves de registro y carpetas de inicio. El implante mantenía una comunicación constante con servidores externos para recibir instrucciones y exfiltrar archivos sensibles del equipo comprometido.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Acceso Inicial** | Spearphishing Link / Attachment | T1566 |
| **Ejecución** | User Execution: Malicious File | T1204.002 |
| **Evasión de Defensa** | Virtualization/Sandbox Evasion | T1497 |
| **Persistencia** | Registry Run Keys / Startup Folder | T1547.001 |
| **Exfiltración** | Exfiltration Over C2 Channel | T1041 |

---

## 🔍 Recomendaciones de Remediación

* **Gestión de Parches:** Actualizar WinRAR a versiones superiores a la 6.23 para mitigar el **CVE-2023-38831**. Priorizar siempre la actualización de software de terceros que maneje archivos externos.
* **Restricción de Ejecución:** Implementar políticas que bloqueen la ejecución de scripts (`.bat`, `.vbs`, `.ps1`) desde directorios temporales o carpetas de descarga de usuarios.
* **Concientización (Awareness):** Capacitar al personal sobre los riesgos de descargar y descomprimir archivos de fuentes no verificadas, incluso si provienen de aplicaciones de mensajería "confiables".
