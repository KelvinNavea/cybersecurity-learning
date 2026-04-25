# 🕵️ Análisis de Incidente: Sherlock - RomCom (DFIR)

## 📌 Descripción del Caso
Investigación de un compromiso inicial en el Laboratorio de Investigación del Hospital Forela. La empleada Susan reportó errores inusuales al abrir un archivo comprimido recibido por correo. El análisis confirma que el grupo de amenazas **RomCom** explotó una vulnerabilidad de WinRAR para desplegar persistencia y una puerta trasera (*backdoor*) en el sistema.

## 🛡️ Resumen Ejecutivo
* **Actor de Amenaza:** RomCom.
* **Vector de Entrada:** Spear-phishing con archivo malicioso (`RomCom.zip`).
* **Vulnerabilidad Explotada:** CVE-2025-8088 (Path Traversal en WinRAR).
* **Impacto:** Ejecución de código remoto y establecimiento de persistencia mediante la carpeta de Inicio (*Startup*).

---

## 🛠️ Metodología de Análisis

### 1. Análisis del Vector de Infección
El atacante envió un archivo ZIP que contenía un archivo `.rar` especialmente diseñado (`Pathology-Department-Research-Records.rar`). Al abrirlo, la vulnerabilidad **Path Traversal** permitió que, además de extraer el PDF legítimo, se extrajeran archivos maliciosos en rutas críticas del sistema sin que el usuario lo notara.

### 2. Identificación del Documento Señuelo
Para distraer a la víctima, el archivo extrajo un documento PDF real llamado `Genotyping_Results_B57_Positive.pdf`. Mientras Susan revisaba este documento (a las 08:15:05), los componentes maliciosos ya se habían instalado en segundo plano.

### 3. Persistencia y Ejecución de Backdoor
Se identificaron dos artefactos clave que confirman el control del atacante sobre el equipo:
* **El Backdoor:** Un ejecutable llamado `ApbxHelper.exe` ubicado en el directorio `Local` de la usuaria.
* **Mecanismo de Persistencia:** Se creó un acceso directo malicioso (`Display Settings.lnk`) en la carpeta de **Startup** (Inicio) de Windows. Esto garantiza que cada vez que Susan encienda su computadora, la puerta trasera se ejecute automáticamente.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Acceso Inicial** | Explotación de Aplicación Pública / Phishing | T1190 / T1566 |
| **Persistencia** | Boot or Logon Autostart Execution: Shortcut Modification | T1547.009 |
| **Ejecución** | User Execution: Malicious File | T1204.002 |

---

## 🔍 Recomendaciones de Remediación

* **Actualización de Software:** Es crítico actualizar WinRAR a la última versión disponible para parchar el **CVE-2025-8088**. La versión actual del personal es vulnerable a ataques de "salto de directorio" (Path Traversal).
* **Limpieza de Persistencia:** Eliminar el archivo `Display Settings.lnk` de la ruta de *Startup* y el ejecutable `ApbxHelper.exe`. Es necesario realizar un escaneo completo con el antivirus actualizado para asegurar que no queden otros restos del malware RomCom.
* **Monitoreo de Carpetas Críticas:** Configurar alertas de seguridad para cualquier creación de archivos `.exe` o `.lnk` en carpetas de `AppData` o `Startup`, ya que no es un comportamiento normal para un usuario de laboratorio.
