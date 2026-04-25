# 🕵️ Análisis de Incidentes: Sherlock - Vantage (DFIR)

## 📌 Descripción del Caso
Investigación de una filtración de datos en una infraestructura de nube privada (OpenStack). Debido a una configuración de red incorrecta, un panel de administración quedó expuesto a internet, permitiendo que un atacante realizara un escaneo, ganara acceso y robara información sensible de los contenedores de almacenamiento.

## 🛡️ Resumen Ejecutivo
* **Activo Afectado:** Panel de control de nube privada (OpenStack).
* **Vector de Entrada:** Exposición de subdominio crítico y ataque de fuerza bruta.
* **Impacto:** Filtración de base de datos de usuarios y creación de una cuenta administrativa no autorizada.
* **Herramientas Detectadas:** Herramientas de escaneo de directorios (Fuzzing).

---

## 🛠️ Metodología de Análisis

### 1. Reconocimiento y Descubrimiento
El análisis de los registros de acceso (*Access Logs*) del servidor web reveló que un atacante estaba buscando subdominios ocultos. Se identificó el uso de una herramienta automatizada que probó miles de nombres hasta encontrar el subdominio `cloud`, el cual devolvió una respuesta positiva y permitió al atacante localizar el panel de acceso.

### 2. Acceso y Uso de la API
Tras probar contraseñas y lograr entrar al panel, el atacante descargó archivos de configuración que contenían credenciales de acceso. Con estas llaves, comenzó a realizar peticiones directamente a la "maquinaria" de la nube (la API), permitiéndole listar todos los recursos disponibles sin necesidad de usar la interfaz visual.

### 3. Robo de Información (Almacenamiento de Objetos)
El atacante exploró los contenedores de almacenamiento (similares a carpetas en la nube) y localizó archivos de bases de datos. Se confirmó la descarga de estos archivos, lo que explica la notificación de filtración de datos que recibió la empresa.

### 4. Persistencia en la Nube
Para no perder el acceso, el atacante creó un nuevo usuario con permisos de administrador dentro del sistema de identidad de la nube. Esto significa que, aunque la empresa cerrara el panel o cambiara la clave original, el atacante podría seguir entrando con su propio "usuario secreto".

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Reconocimiento** | Escaneo Activo (Fuzzing) | T1595 |
| **Acceso a Credenciales** | Fuerza Bruta (Brute Force) | T1110 |
| **Exfiltración** | Transferencia de datos a cuenta externa | T1537 |
| **Persistencia** | Creación de Cuenta en la Nube | T1136.003 |

---

## 🔍 Recomendaciones de Remediación

* **Cerrar Paneles de Administración:** No permitir que los paneles de gestión de la nube sean accesibles desde cualquier lugar de internet. Se deben proteger detrás de una VPN o restringir el acceso solo a las oficinas de la empresa.
* **Limpieza de Usuarios:** Revisar el listado de usuarios de la nube y eliminar inmediatamente cualquier cuenta que no haya sido creada por el equipo de IT (como la cuenta detectada en este análisis).
* **Principio de Menor Privilegio:** Revisar los permisos de los usuarios actuales. Un usuario no debería poder ver o descargar todas las bases de datos de la empresa a menos que sea estrictamente necesario para su trabajo.
