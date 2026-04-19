# Reporte de Análisis: Sherlock - Vantage (HTB)

## 1. Escenario del Incidente
Una empresa migró recursos a una nube privada. Debido a una mala configuración (redirección expuesta al panel de control), 
un atacante logró infiltrarse. El equipo de seguridad recibió una notificación de filtración de datos de usuario. 
El objetivo es reconstruir el ataque desde el reconocimiento inicial hasta la exfiltración y persistencia en el entorno OpenStack.

## 2. Resumen de Hallazgos

| Ítem | Información Identificada |
| :--- | :--- |
| **Herramienta de Fuzzing** | ffuf@2.1.0 |
| **Subdominio Descubierto** | cloud |
| **Fuerza Bruta al Panel** | 3 intentos fallidos |
| **ID de Proyecto OpenStack** | 9fb84977ff7c4a0baf0d5dbb57e235c7 |
| **Servicio de Autenticación** | keystone |
| **Contenedores Descubiertos** | 3 |
| **Usuario de Persistencia** | jellibean |

## 3. Análisis Técnico (Metodología)

### Fase 1: Reconocimiento y Descubrimiento
El atacante comenzó realizando un "fuzzing" de directorios y subdominios. Mediante el análisis de los logs del servidor web (Access Logs), 
se identificó el User-Agent de la herramienta utilizada y el subdominio específico que devolvió un código de estado `200 OK`, permitiendo al atacante localizar el panel de gestión.

### Fase 2: Explotación y Acceso a la API
Tras obtener acceso al panel mediante fuerza bruta, el atacante descargó credenciales de acceso remoto (`openrc`). 
El análisis de los timestamps en los logs de descarga y las posteriores llamadas a la API permiten determinar 
el momento exacto en que el atacante comenzó a interactuar con el nodo controlador de OpenStack.

### Fase 3: Exfiltración de Objetos (Object Storage)
Utilizando el servicio **Swift** (almacenamiento de objetos de OpenStack), el atacante enumeró contenedores y localizó archivos sensibles. 
Se identificó la URL del endpoint y se confirmó la descarga de una base de datos de usuarios, validando la notificación de filtración recibida por la empresa.

### Fase 4: Persistencia en la Nube
Para asegurar el acceso futuro, el atacante utilizó sus privilegios actuales para crear un nuevo usuario administrativo dentro del servicio de identidad (**Keystone**).
Esto le permite mantener el control total del entorno aunque las credenciales originales sean comprometidas o cambiadas.

## 4. Mapeo MITRE ATT&CK
* **Reconnaissance:** T1595 - Active Scanning.
* **Credential Access:** T1110 - Brute Force.
* **Exfiltration:** T1537 - Transfer Data to Cloud Account.
* **Persistence:** T1136.003 - Cloud Account.

---
**Nota de Analista:** Este incidente resalta la importancia de no exponer paneles de administración a internet y de implementar políticas de "Least Privilege" en las políticas de IAM de la nube.
