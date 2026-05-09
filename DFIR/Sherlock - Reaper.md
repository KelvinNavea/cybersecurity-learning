# 🛡️ Investigación de Incidente: Movimiento Extraño en la Red (Reaper)

## 📌 1. Descripción del Caso
El sistema de alertas del equipo de seguridad avisó que algo no andaba bien: una computadora en la red aparecía con un nombre que no coincidía con su dirección IP. Mi tarea fue investigar si se trataba de un error técnico o de un atacante intentando esconderse para robar información de los servidores.

## 📋 2. Resumen Ejecutivo
* **Problema detectado:** Un atacante entró a la red y se hizo pasar por una computadora de la empresa.
* **Impacto:** Lograron robar la contraseña (el hash) de un usuario legítimo y entraron a carpetas con información sensible en el servidor.
* **Estado:** Incidente analizado y rutas de ataque identificadas para su bloqueo.

## ⚙️ 3. Metodología de Análisis
Para entender qué pasó, no me quedé solo con la alerta, sino que investigué a fondo usando estas herramientas:
1. **Análisis de Red (Wireshark):** Revisé el tráfico para ver quién estaba hablando realmente. Ahí encontré una dirección IP "intrusa" que no pertenecía a ninguna máquina autorizada.
2. **Revisión de Logs (Windows):** Entré al visor de eventos para ver quién había iniciado sesión. Crucé los datos y confirmé que el atacante estaba usando la cuenta de un usuario para navegar por el servidor.
3. **Correlación:** Uní los puntos entre la IP sospechosa, el nombre de equipo falso que usaba y las carpetas a las que intentó entrar.

## 📊 4. Mapeo de Técnicas (MITRE ATT&CK)
Para hablar el mismo idioma que otros analistas, identifiqué estas maniobras del atacante:
* **Acceso de Credenciales (T1557):** El atacante se metió en medio de la comunicación para atrapar la contraseña del usuario.
* **Defensa Evasiva (T1036):** Cambió su nombre lógico para que parezca una computadora de la oficina y no sospecharan de él.
* **Movimiento Lateral (T1021):** Una vez que tuvo la contraseña, saltó de su máquina hacia las carpetas del servidor.

## 🛡️ 5. Recomendaciones de Remediación
Como analista, sugiero estos cambios para que no nos vuelva a pasar:
* **Revisión de Políticas de Grupo (GPOs):** Es fundamental auditar las políticas del servidor para asegurar que los protocolos de red antiguos (como LLMNR) estén desactivados,
  evitando que un atacante engañe a los equipos de la red.
* **Control de Cuentas Privilegiadas:** Aplicar estrictamente el Principio de Menor Privilegio.
  Los usuarios solo deben tener acceso a las carpetas que realmente necesitan para su trabajo diario.
