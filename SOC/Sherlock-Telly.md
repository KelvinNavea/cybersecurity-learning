# 🕵️ Análisis de Incidente: Sherlock - Telly (SOC)

## 📌 Descripción del Caso
Investigación de una alerta crítica de **exfiltración de datos** detectada por una solución DLP (*Data Loss Prevention*) en un servidor de respaldo. El análisis se basó en el estudio de capturas de tráfico de red (`.pcapng`) para identificar el compromiso inicial y reconstruir las acciones del adversario.

## 🛡️ Resumen Ejecutivo
* **Activo Afectado:** Servidor de respaldo (`backup-secondary`).
* **Vector de Entrada:** Explotación de vulnerabilidades en el servicio de **Telnet** (puerto 23).
* **Tipo de Incidente:** Exfiltración de Información PII y datos financieros.
* **Timeline:** El compromiso y la extracción se completaron en menos de 10 minutos, lo que indica un ataque automatizado o un actor con objetivos claros.

---

## 🛠️ Metodología de Análisis

### 1. Detección de Explotación en Protocolos Legacy
Mediante el análisis de flujos TCP en **Wireshark**, se identificó tráfico en texto claro a través de Telnet. La reconstrucción de la sesión permitió verificar la explotación del **CVE-2026-24061**, que permitió al atacante evadir la autenticación y obtener privilegios de `root` de manera inmediata.

### 2. Análisis de Persistencia y Movimiento Lateral
Se observó que el atacante, una vez dentro del sistema, ejecutó comandos para garantizar el acceso futuro:
* **Creación de Cuentas:** Se detectó la creación de un usuario local con privilegios elevados para actuar como *backdoor*.
* **Transferencia de Herramientas (Ingress):** Se identificó el uso de herramientas de transferencia de archivos (`curl`/`wget`) para descargar scripts desde una infraestructura de Comando y Control (C2) externa.

### 3. Forense de Red y Exfiltración de Datos
El análisis de los paquetes de datos confirmó la transferencia de un archivo de base de datos hacia una IP externa sospechosa. Mediante la **inspección profunda de paquetes (DPI)**, se validó que la información contenía registros de clientes y números de tarjetas de crédito, lo que disparó la alerta original del DLP.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Acceso Inicial** | Explotación de Aplicación Pública (Telnet) | T1190 |
| **Persistencia** | Creación de Cuenta Local | T1136.001 |
| **Mando y Control** | Transferencia de Herramientas | T1105 |
| **Exfiltración** | Exfiltración sobre Canal de C2 | T1041 |

---

## 🔍 Recomendaciones de Remediación

* **Desactivación de Protocolos Inseguros:** Deshabilitar Telnet en toda la infraestructura de servidores de respaldo.
* **Hardening de Accesos:** Implementar **SSH (v2)** con autenticación exclusiva por llaves públicas y desactivar el acceso `root` directo.
* **Monitoreo de Red:** Configurar alertas en el **SIEM** para cualquier intento de conexión saliente desde servidores de base de datos hacia IPs externas no autorizadas.
