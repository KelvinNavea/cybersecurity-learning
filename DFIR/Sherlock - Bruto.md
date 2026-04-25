# 🕵️ Análisis de Incidentes: Sherlock - Bruto (DFIR)

## 📌 Descripción del Caso
Investigación de un compromiso de seguridad en un servidor **Confluence** basado en Unix. El análisis se centró en identificar el origen de un ataque de fuerza bruta exitoso sobre el servicio SSH, documentando las acciones del atacante para asegurar su acceso y la descarga de herramientas sospechosas.

## 🛡️ Resumen Ejecutivo
* **Activo Afectado:** Servidor Confluence (Linux).
* **Vector de Entrada:** Ataque de fuerza bruta (Brute Force) sobre el puerto 22 (SSH).
* **Cuenta Comprometida:** `root`.
* **Impacto:** Creación de una cuenta de acceso secundaria y descarga de scripts externos.

---

## 🛠️ Metodología de Análisis

### 1. Identificación de Acceso Inicial
El análisis comenzó revisando el archivo de logs de autenticación `/var/log/auth.log`. Se detectó un volumen inusual de intentos fallidos de inicio de sesión seguidos de un evento `Accepted password` para el usuario `root`, lo que confirma que el atacante logró adivinar la contraseña mediante un ataque automatizado.
* **Validación:** Se utilizó el archivo `wtmp` para identificar el tiempo de conexión y confirmar que la sesión fue interactiva (el atacante estuvo operando manualmente el servidor).

### 2. Análisis de Persistencia
Una vez dentro, el atacante buscó una forma de no perder el acceso:
* **Creación de Cuenta Secundaria:** Se identificó en los logs la creación de un nuevo usuario con permisos de administrador. Esto permite al atacante volver a entrar incluso si se cambia la contraseña de `root`.

### 3. Descarga de Archivos Sospechosos
Mediante la revisión de los comandos ejecutados (`history`) y los logs de `sudo`, se detectó que el atacante utilizó la herramienta `curl` para descargar un script desde un repositorio externo. El script fue alojado en carpetas temporales para evitar su detección inmediata.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Acceso a Credenciales** | Brute Force (SSH) | T1110 |
| **Persistencia** | Local Account (Creación de usuario) | T1136.001 |
| **Mando y Control** | Ingress Tool Transfer (Descarga de scripts) | T1105 |

---

## 🔍 Recomendaciones de Remediación

* **Gestión de Cuentas:** Realizar un cambio inmediato de la contraseña de `root` por una clave robusta. Es fundamental identificar y eliminar la cuenta de usuario creada por el atacante para cortar su vía de acceso secundaria.
* **Restricción de Acceso SSH:** Configurar el servidor para prohibir el inicio de sesión directo con el usuario `root`. Se recomienda forzar el uso de una cuenta de usuario estándar para la conexión inicial, reduciendo la exposición de la cuenta con máximos privilegios.
* **Limpieza del Sistema:** Localizar y borrar cualquier archivo o script descargado por el atacante en directorios como `/tmp/` o carpetas de usuario. Esto asegura que no queden herramientas maliciosas residentes en el disco.
