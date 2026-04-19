# Reporte de Análisis: Sherlock - Telly (HTB)

## 1. Escenario del Incidente
Como analista DFIR junior en un MSSP, se investigó una alerta de exfiltración de datos detectada por una solución DLP en un servidor de respaldo. El análisis se centró en un archivo de captura de red (`.pcapng`) para reconstruir el compromiso inicial a través de protocolos legacy y la posterior extracción de información confidencial.

## 2. Resumen de Hallazgos

| Ítem | Información Identificada |
| :--- | :--- |
| **CVE Relacionado** | `CVE-2026-24061` |
| **Éxito del Exploit (UTC)** | `2026-01-27 10:39:28` |
| **Hostname del Objetivo** | `backup-secondary` |
| **Backdoor (User:Pass)** | `cleanupsvc:YouKnowWhoiam69` |
| **IP del C2** | `91.99.25.54` |
| **Hora de Exfiltración** | `2026-01-27 10:49:54` |
| **Tarjeta Quinn Harris** | `5312269047781209` |

## 3. Análisis Técnico (Metodología)

### Fase 1: Análisis de Tráfico de Red (Telnet)
El análisis comenzó filtrando el tráfico por el protocolo **Telnet (puerto 23)**. Al ser un protocolo de texto claro, 
se utilizó la función `Follow TCP Stream` en Wireshark para visualizar la interacción entre el atacante y el servidor. 
Se identificó el uso de una vulnerabilidad específica que permitió el acceso remoto con privilegios de **root** sin necesidad de credenciales legítimas iniciales.

### Fase 2: Persistencia y Comando y Control (C2)
Tras obtener acceso, el atacante ejecutó comandos para asegurar su retorno:
* **Creación de Usuario:** Se observó la creación de una cuenta con credenciales estáticas para persistencia manual.
* **Despliegue de Implante:** Se identificó un comando completo (utilizando `curl`/`wget`) para descargar un script desde una infraestructura externa.
* El flujo de red confirmó la dirección IP del servidor C2 y la ejecución del script en el sistema comprometido.

### Fase 3: Análisis de la Base de Datos Exfiltrada
Se identificó el momento exacto en que el atacante transfirió un archivo de base de datos (`.sql` o `.csv`) fuera de la red. Mediante el análisis del contenido de los paquetes, 
se recuperó información sensible de clientes, incluyendo datos financieros (tarjetas de crédito), validando la alerta inicial del DLP.

## 4. Mapeo MITRE ATT&CK
* **Initial Access:** T1190 - Exploit Public-Facing Application (Telnet).
* **Persistence:** T1136.001 - Create Account: Local Account.
* **Command and Control:** T1105 - Ingress Tool Transfer.
* **Exfiltration:** T1041 - Exfiltration Over C2 Channel.

---
**Nota de Analista:** El hallazgo de protocolos como Telnet en servidores de respaldo subraya la necesidad de auditorías de servicios activos. 
Se recomienda deshabilitar Telnet y forzar el uso de SSH con autenticación de clave pública.
