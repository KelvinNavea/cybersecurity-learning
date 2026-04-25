# 🕵️ Análisis de Inteligencia de Amenazas: Sandworm Team (Threat Intelligence)

## 📌 Descripción del Grupo
Sandworm es un actor de amenazas persistentes avanzadas (APT) de origen estatal ruso, activo desde al menos **2009**. Se especializa en ataques contra infraestructuras críticas, sectores gubernamentales y energía. Son conocidos por utilizar malware destructivo (*wipers*) y ataques de denegación de servicio (DoS) físico mediante la manipulación de sistemas SCADA.

## 🛡️ Resumen de Operaciones Clave
* **Campaña Eléctrica (2016):** Uso de técnicas de fuerza bruta (**T1110**) y acceso a memoria LSASS para movimiento lateral.
* **Campaña de Destrucción (2022):** Implementación de **CaddyWiper** y abuso de binarios legítimos de software industrial (`scilc.exe`) para sabotear subestaciones eléctricas.
* **Incidente Global NotPetya:** Explotación de la vulnerabilidad **MS17-010** (EternalBlue) para propagación masiva.

---

## 🛠️ Metodología de Análisis Técnico

### 1. Persistencia y Acceso Remoto en Servidores
Durante sus operaciones más recientes, el grupo ha demostrado gran habilidad para mantener el acceso mediante **Web Shells**. Utilizaron la herramienta **Neo-REGEORG** (T1505.003), lo que les permite tunelizar tráfico a través de servidores web comprometidos y evadir defensas de red perimetrales.

### 2. Manipulación de Entornos Industriales (SCADA)
A diferencia de otros grupos, Sandworm sabe operar software de control industrial. En 2022, ejecutaron comandos directamente sobre el binario `scilc.exe` para realizar acciones contra subestaciones:
* **Comando:** `C:\sc\prog\exec\scilc.exe -do pack\scil\s1.txt`
Esto demuestra que el atacante tiene conocimiento profundo del software de ingeniería que controla la red eléctrica.

### 3. Malware Destructivo (Anti-Forense e Impacto)
El uso de **CaddyWiper** es una firma del grupo para la destrucción de datos (T1485). Además, este malware utiliza la técnica **Native API (T1106)** para interactuar directamente con el sistema operativo, lo que dificulta su detección por parte de algunos antivirus que solo monitorean APIs de nivel superior.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Acceso Inicial** | Exploit Public-Facing Application | T1190 |
| **Acceso a Credenciales** | Brute Force / OS Credential Dumping | T1110 / T1003.001 |
| **Persistencia** | Server Software Component: Web Shell | T1505.003 |
| **Ejecución** | Native API | T1106 |
| **Impacto** | Data Destruction (Wiper) | T1485 |

---

## 🔍 Recomendaciones de Remediación y Defensa

* **Segmentación de Red IT/OT:** Es vital que las redes administrativas (donde llega el correo) estén físicamente o lógicamente separadas de las redes que controlan el equipo SCADA.
* **Parcheo Crítico (MS17-010):** Aunque es una vulnerabilidad antigua, Sandworm sigue buscando sistemas sin parchar para propagar malware tipo gusano como NotPetya.
* **Monitoreo de Puertos No Estándar:** Vigilancia estricta sobre puertos inusuales, como el **6789**, utilizado por el grupo para servidores SSH de comando y control (C2).
* **Control de Ejecución de Binarios:** Restringir la ejecución de scripts (`.vbs`) y monitorear el uso de herramientas de túnel como Neo-REGEORG en servidores web expuestos.
