# Caso de Estudio: Análisis DFIR - CrownJewel (Parte 1 y Parte 2)

## 📋 Descripción del Caso
Este repositorio documenta el análisis forense de respuesta a incidentes (DFIR) realizado sobre dos retos consecutivos de Hack The Box (*CoronaJewel Part 1* y *Part 2*). El objetivo fue investigar un ataque dirigido al Controlador de Dominio (DC) de Forela, donde el actor de amenaza logró comprometer cuentas de Administrador de Dominio y abusar de utilidades nativas (*LOLBINs*) para realizar un volcado de la base de datos de Active Directory (**NTDS.dit**).

---

## 🛠️ Metodología de Análisis
Para realizar la investigación utilicé herramientas de análisis forense digital y correlación de eventos, enfocándome en el análisis de logs de seguridad, marcas de tiempo y artefactos de sistema para reconstruir la cadena de ataque.

### 1. Análisis de Artefactos y Servicios del DC (Parte 1: Abuso de VSSAdmin)
* **Monitoreo de Servicios:** Seguimiento del estado del servicio de instantáneas de volumen para identificar el inicio del volcado.
* **Correlación de Identidades:** Identificación del uso de la cuenta de máquina (`DC01$`) y la validación de privilegios de los grupos *Administrators* y *Backup Operators* (Proceso PID 4496).
* **Extracción de Rutas y Volúmenes:** Localización del GUID de la instantánea montada (`{06c4a997-cca8-11ed-a90f-000c295644f9}`) y la ruta del archivo NTDS junto al registro `SYSTEM` volcado de forma simultánea.

### 2. Respuesta a Persistencia y Nuevo Volcado (Parte 2: Abuso de Ntdsutil)
* **Detección de Nuevos Vectores:** Tras restaurar el entorno, el atacante reutilizó acceso persistente empleando `ntdsutil.exe`.
* **Análisis de Eventos ESENT:** Uso de fuentes de eventos `ESENT` para rastrear el estado, creación y separación de la base de datos.
* **Trazabilidad de Sesiones:** Identificación de la hora exacta de inicio de la sesión maliciosa a través del ID de inicio de sesión del atacante.

---

## 📊 Resumen de Hallazgos

| Sherlock / Caso | Vector / Herramienta | Artefacto Clave | Ruta del Volcado / Hallazgo |
| :--- | :--- | :--- | :--- |
| **CoronaJewel Part 1** | `vssadmin` (LOLBIN) | Eventos de Servicio / PID 4496 | `C:\Users\Administrator\Documents\backup_sync_Dc\Ntds.dit` |
| **CoronaJewel Part 2** | `ntdsutil.exe` | Registros ESENT / Sesión Activa | `C:\Windows\Temp\dump_tmp\Active Directory\ntds.dit` |

---

## 🛡️ Mapeo de Técnicas (MITRE ATT&CK)
* **T1003.003 - OS Credential Dumping: NTDS:** Extracción de credenciales mediante el volcado directo de la base de datos de Active Directory (`NTDS.dit`).
* **T1562.001 - Impair Defenses: Disable or Modify Tools:** Abuso de utilidades administrativas legítimas del sistema operativo (*vssadmin* y *ntdsutil*) para fines maliciosos (LOLBINs).
* **T1078 - Valid Accounts:** Uso continuo de credenciales y cuentas de administrador comprometidas para mantener persistencia en el dominio.

[CrownJewel Parte 1]([url](https://labs.hackthebox.com/achievement/sherlock/2858413/737))

[CrownJewel Parte 2 ]([url](https://labs.hackthebox.com/achievement/sherlock/2858413/736))
