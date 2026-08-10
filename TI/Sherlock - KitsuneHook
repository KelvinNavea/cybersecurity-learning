# 🕵️ Análisis de Inteligencia de Amenazas: Winnti / APT41 (Threat Intelligence)

## 📌 Descripción del Grupo
Winnti (también rastreado como APT41 o Blackfly) es un grupo de ciberespionaje patrocinado por el estado, activo desde al menos 2012. Se enfoca principalmente en operaciones dirigidas contra sectores industriales, energéticos, de manufactura y de materiales, combinando el espionaje comercial con el espionaje tradicional.

---

## 🛡️ Resumen de Operaciones Clave
* **Campaña RevivalStone:** Operación dirigida a organizaciones de manufactura y materiales, utilizando cadenas de infección complejas y componentes personalizados.
* **Operation CuckooBees (2021):** Actividad detectada con cargas útiles y DLLs específicas (como `prntvpt.dll`) recopiladas en ventanas clave de tiempo durante el año 2021.
* **Filtraciones de Contratistas (I-Soon):** Exposición de documentación interna que reveló herramientas de control y controladores Linux asociados a su ecosistema de malware.

---

## 🛠️ Metodología de Análisis Técnico y TTPs

### 1. Acceso Inicial y Explotación Web
Durante las campañas analizadas, el grupo aprovecha vulnerabilidades en aplicaciones orientadas a Internet para lograr su entrada. Utilizaron de forma recurrente ataques de inyección SQL (SQL injection) para comprometer los sistemas iniciales.

### 2. Despliegue de Web Shells y Credenciales
Una vez dentro, los atacantes despliegan múltiples web shells para mantener el control, destacando herramientas como "China Chopper" y "Behinder" (también conocido como Bingxia). 
* **Nota técnica:** Behinder utiliza una llave de cifrado embebida fija que corresponde a los primeros 16 caracteres del hash MD5 de la palabra `rebeyond`.

### 3. Movimiento Lateral y Abuso de Servicios del Sistema
Para la persistencia y ejecución de fases posteriores, el grupo abusa de servicios legítimos de Windows mediante técnicas de *DLL side-loading*. Un ejemplo documentado es el uso del servicio `SessionEnv` abusando de `TSMSISrv.DLL`.

### 4. Carga de Malware, Rootkits y Criptografía
La cadena de infección involucra componentes modulares:
* **Loader y Rootkit:** Se despliega el cargador `PRIVATELOG` en conjunto con el rootkit a nivel de kernel `WINNKIT`.
* **Cifrado de Archivos DAT:** El análisis de los contenedores de datos del malware demostró que el proceso de descifrado de archivos `.DAT` utiliza el algoritmo AES bajo el modo de operación **OFB (Output Feedback)**.
* **Componentes de Red y C2:** Se identificó infraestructura con nombres en clave geológicos como el panel de control *TreadStone* (Linux) y el uso de herramientas como el uploader de `sqlmap`.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| Acceso Inicial | Exploit Public-Facing Application / SQL Injection | T1190 |
| Persistencia / Ejecución | Server Software Component: Web Shell | T1505.003 |
| Defense Evasion | Hijack Execution Flow: DLL Side-Loading | T1574.002 |
| Rootkit / Privilegios | Boot or Logon Rootkit (Kernel-level) | T1014 |

---
