# 🕵️ Análisis de Incidente: Sherlock - PhishNet (SOC)

## 📌 Descripción del Caso
Análisis de un intento de fraude financiero dirigido al equipo de contabilidad. El ataque utiliza técnicas de suplantación de identidad (spoofing) y un archivo adjunto malicioso para comprometer la estación de trabajo del usuario. El análisis se basó en el estudio del archivo de correo crudo (`.eml`).

## 🛡️ Resumen Ejecutivo
* **Vector de Entrada:** Phishing por correo electrónico (Spear-phishing).
* **IP de Origen:** `45.67.89.10`.
* **Técnica de Engaño:** Uso de doble extensión en archivos (`.pdf.bat`) y suplantación de dominio.
* **Estado de Seguridad:** El correo logró pasar el filtro SPF (`pass`), lo que indica que el atacante configuró correctamente su servidor para parecer legítimo.

---

## 🛠️ Metodología de Análisis

### 1. Análisis de Encabezados (Headers)
Se inspeccionó el archivo `.eml` para rastrear el salto del correo. Se identificó la IP del remitente original y el servidor de correo intermedio (`203.0.113.25`) que retransmitió el mensaje. A pesar de ser un correo falso, el registro **SPF** resultó positivo, lo que sugiere que el atacante utilizó un dominio propio bien configurado para evadir filtros básicos.

### 2. Inspección del Cuerpo y URL
El correo simulaba una urgencia de pago de la empresa **Business Finance Ltd.** Se detectó una URL que dirigía al dominio `secure.business-finance.com`. Este dominio está diseñado para generar confianza mediante el uso de palabras como "secure" y el nombre de la empresa suplantada.

### 3. Forense del Archivo Adjunto
Se analizó el archivo adjunto `Invoice_2025_Payment.zip`. 
* **Hash SHA-256:** `8379C41239E9AF845B2AB6C27A7509AE8804D7D73E455C800A551B22BA25BB4A`.
* **Carga Útil (Payload):** Dentro del ZIP se encontró el archivo `invoice_document.pdf.bat`. Este archivo es un script de Windows que, al ejecutarse, inicia la infección mientras el usuario cree estar abriendo un documento PDF.

---

## 📊 Mapeo de Técnicas (MITRE ATT&CK)

| Táctica | Técnica | ID |
| :--- | :--- | :--- |
| **Acceso Inicial** | Phishing: Spearphishing Attachment | T1566.001 |
| **Ejecución** | User Execution: Malicious File | T1204.002 |
| **Defensa de Evasión** | Masquerading: Double File Extension | T1036.007 |

---

## 🔍 Recomendaciones de Remediación

* **Bloqueo de Indicadores (IOCs):** Bloquear la IP `45.67.89.10` y el dominio `business-finance.com` en el firewall y en el filtro de correo de la empresa.
* **Configuración de Windows:** Implementar por política de grupo (GPO) que Windows **siempre muestre las extensiones de archivo**. Esto permitiría al usuario ver el `.bat` al final del nombre y sospechar del archivo.
* **Capacitación de Usuarios:** Reforzar la importancia de desconfiar de archivos `.zip` que contengan ejecutables o scripts, incluso si el remitente parece conocido y el correo "pasa" las validaciones de seguridad.
