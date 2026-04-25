# 🕵️ Análisis de Incidentes: Sherlock - MangoBleed (DFIR)

## 📌 Descripción del Caso
Investigación de una alerta en el servidor `mongodbsync` por el uso de un exploit conocido como **MongoBleed**. El objetivo fue analizar los logs del sistema para identificar cómo entró el atacante, qué buscaba dentro de la base de datos y cómo intentó llevarse la información.

## 🛡️ Resumen Ejecutivo
* **Activo Afectado:** Servidor de base de datos MongoDB.
* **Vulnerabilidad:** CVE-2025-14847 (MongoBleed).
* **Impacto:** Lectura no autorizada de la memoria del servidor y posible robo de credenciales.
* **Actividad Detectada:** Gran volumen de conexiones automatizadas y uso de un servidor web temporal para mover archivos.

---

## 🛠️ Metodología de Análisis

### 1. Identificación de la Vulnerabilidad
El análisis comenzó verificando la versión de MongoDB instalada (8.0.16), la cual es vulnerable a **MongoBleed**. Esta falla permite que un atacante "lea" partes de la memoria del servidor que no debería ver, exponiendo potencialmente datos que están en uso en ese momento.

### 2. Análisis de Logs de Conexión
Se revisaron los logs en `/var/log/mongodb/` para rastrear la actividad sospechosa:
* **Identificación de la IP:** Se aisló una dirección IP externa que realizó miles de peticiones en un tiempo muy corto. Este comportamiento no es humano y coincide con el uso de herramientas automatizadas para explotar la base de datos.
* **Timeline:** Se estableció que el ataque comenzó con ráfagas de conexiones rápidas y culminó con un acceso exitoso al sistema de archivos minutos después.

### 3. Actividad Post-Compromiso
Una vez que el atacante logró manipular el servicio, se detectaron acciones para extraer información:
* **Ejecución de Scripts:** Se identificó el uso de comandos en la terminal para intentar subir de nivel de privilegios.
* **Exfiltración de Datos:** El atacante levantó un servidor web muy básico usando **Python** (una herramienta que ya viene en casi todos los Linux) para intentar descargar una carpeta específica del servidor hacia su propia máquina.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Acceso Inicial** | Explotación de Aplicación Pública (MongoBleed) | T1190 |
| **Ejecución** | Intérprete de Comandos y Scripts (Unix Shell) | T1059.004 |
| **Exfiltración** | Exfiltración sobre servicio Web (Python HTTP Server) | T1567 |

---

## 🔍 Recomendaciones de Remediación

* **Actualización Urgente:** Parchear o actualizar MongoDB a una versión que no sea vulnerable a MongoBleed. Esta es la única forma definitiva de cerrar la puerta que usó el atacante.
* **Restricción de Red (Firewall):** Configurar el firewall para que la base de datos no sea visible desde todo Internet. Solo deberían poder conectarse las IP de los servidores de la empresa que realmente necesiten usar la base de datos.
* **Monitoreo de Procesos Inusuales:** Estar atentos a la ejecución de comandos como `python -m http.server` en servidores que no deberían estar compartiendo archivos por web. Si un servidor de base de datos empieza a actuar como servidor web, es una señal de alerta inmediata.
