# 🕵️ Análisis de Incidente: Sherlock - LLMNR Poisoning

### 📌 Descripción del Caso
Investigación de una interceptación de credenciales en una red local. El análisis se basó en el estudio de capturas de tráfico (.pcap) para identificar cómo un fallo en la resolución de nombres permitió el compromiso de un usuario del dominio.

### 🛡️ Resumen Ejecutivo
* **Vector de Entrada:** Envenenamiento del protocolo LLMNR (Link-Local Multicast Name Resolution).
* **Tipo de Incidente:** Captura de hash Net-NTLMv2 y cracking offline.
* **Timeline:** El compromiso se produce de forma inmediata tras un error de resolución de nombres por parte de la víctima.

### 🛠️ Metodología de Análisis

#### 1. Detección de Envenenamiento en Protocolos Legacy
Mediante el análisis en Wireshark, se identificaron consultas multicast fallidas que fueron respondidas por un actor malicioso. La inspección del tráfico NTLMSSP permitió verificar la captura de los desafíos de autenticación.

#### 2. Forense de Red y Extracción de Hashes
Se procedió a la reconstrucción manual del hash Net-NTLMv2 extrayendo los valores de *Server Challenge* y *NTProofStr*. Se realizó la limpieza de la respuesta para asegurar la compatibilidad con herramientas de auditoría.

#### 3. Validación de la Seguridad de Credenciales
Se utilizó Hashcat para realizar un ataque de diccionario. La recuperación de la contraseña en texto claro confirmó que las políticas de complejidad vigentes eran insuficientes para mitigar este vector de ataque.

### 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| Acceso Inicial | Adversary-in-the-Middle (LLMNR/NetBIOS) | T1557.001 |
| Acceso a Credenciales | Brute Force (Offline Cracking) | T1110.003 |

### 🔍 Recomendaciones de Remediación
* **Desactivación de Protocolos Inseguros:** Deshabilitar LLMNR y NetBIOS mediante GPO en todo el dominio.
* **Hardening de Cuentas:** Implementar políticas de contraseñas robustas y fomentar el uso de MFA para reducir el impacto de la captura de hashes.
