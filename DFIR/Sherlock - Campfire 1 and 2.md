# Análisis Forense: Campfire Series (Kerberoasting & AS-REP Roasting)

## 📌 Descripción del Caso
Este repositorio documenta el análisis forense de dos casos de ataques contra **Active Directory**: **Kerberoasting** (Campfire-1) y **AS-REP Roasting** (Campfire-2). El objetivo fue reconstruir la cadena de ataque (Kill Chain) analizando registros del Controlador de Dominio (DC) y artefactos forenses de una estación de trabajo comprometida para entender cómo los atacantes abusan de protocolos legítimos.

## 🛠️ Metodología de Análisis
Para realizar el análisis utilicé herramientas de análisis forense digital (DFIR) como **PECmd**, **Timeline Explorer** y el **Visor de Eventos de Windows**, aplicando un enfoque lógico para correlacionar la actividad maliciosa.

### 1. Correlación de Eventos Kerberos (DC Security Logs)
Filtré los logs de seguridad del Controlador de Dominio (Security.evtx) buscando patrones específicos:
- **Event ID 4769:** Solicitud de ticket de servicio (identificación de Kerberoasting mediante cifrado RC4 - 0x17).
- **Event ID 4768:** Solicitud de ticket de autenticación (TGT) para identificar el AS-REP Roasting en cuentas con pre-autenticación desactivada.

### 2. Análisis de Enumeración (PowerShell Logs)
Analicé el log `Microsoft-Windows-PowerShell/Operational.evtx` (Event ID 4104) para observar el código ejecutado en la estación de trabajo:
- **Script identificado:** `PowerView.ps1`
- **Acción:** Escaneo de objetos de Active Directory buscando cuentas con SPN configurado.

### 3. Análisis Forense de Prefetch
Procesé los archivos `.pf` encontrados en la estación de trabajo mediante **PECmd** para determinar la ejecución local de herramientas. Esto permitió extraer:
- **Herramienta:** `Rubeus.exe`
- **Ruta de ejecución:** `C:\Users\Alonzo.Spire\Downloads\Rubeus.exe`
- **Marca de tiempo de ejecución:** `2024-05-21 03:18:08`

## 📊 Resumen de Hallazgos

| Sherlock | Ataque | Cuenta(s) Afectada(s) | IP de Origen | Herramienta |
| :--- | :--- | :--- | :--- | :--- |
| Campfire-1 | Kerberoasting | - | 172.17.79.129 | Rubeus.exe |
| Campfire-2 | AS-REP Roasting | arthur.kyle | 172.17.79.129 | - |

## 🛡️ Mapeo de Técnicas (MITRE ATT&CK)

- **T1558.003 - Kerberoasting:** Solicitud de tickets de servicio con cifrado débil (RC4) para intentar romper contraseñas offline.
- **T1558.004 - AS-REP Roasting:** Explotación de cuentas de usuario con la pre-autenticación de Kerberos deshabilitada.
- **T1059.001 - PowerShell:** Uso de scripts para enumeración avanzada de objetos en el dominio de Active Directory.

## 💡 Conclusión
El ejercicio permitió comprender que gran parte de los ataques modernos no dependen de malware complejo, sino del abuso de funciones legítimas de Windows. La correlación entre logs de DC y artefactos locales (como Prefetch) es fundamental para cualquier analista de SOC al momento de reconstruir incidentes de seguridad.
