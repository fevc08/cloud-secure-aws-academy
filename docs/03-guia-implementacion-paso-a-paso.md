# Guía de implementación paso a paso - AWS Academy Learner Lab

> Ejecutar en una única sesión activa del Learner Lab (los recursos y el rol `LabRole` se reinician entre sesiones). Tomar la captura de pantalla indicada al final de cada bloque y guardarla en la carpeta `docs/evidencias/` correspondiente.

## 0. Antes de empezar

1. Iniciar el Learner Lab y esperar a que el círculo junto a "AWS" se ponga verde.
2. Abrir la consola de AWS (botón "AWS Details" → "AWS Console Access" o el enlace directo).
3. Confirmar la región activa (arriba a la derecha) — normalmente `us-east-1 (N. Virginia)`. Usar siempre esa misma región en todos los pasos.

## Lección 1: Bucket S3 cifrado y bloqueado

**Objetivo:** crear el bucket de datos protegido y aplicar el principio de privilegio mínimo a nivel de recurso.

1. Consola → **S3** → **Create bucket**.
2. Nombre sugerido: `bluewave-core-data-<tus-iniciales>-<random>` (los nombres de bucket son globalmente únicos).
3. Región: la misma región activa del Lab.
4. **Block Public Access settings**: dejar las 4 casillas activadas (es el valor por defecto — no desactivar ninguna).
5. **Bucket Versioning**: opcional, se puede activar como buena práctica adicional (no es requisito de la rúbrica).
6. **Default encryption**: seleccionar **Server-side encryption with Amazon S3 managed keys (SSE-S3)**.
7. Crear el bucket.
8. Verificar: entrar al bucket → pestaña **Properties** → confirmar que "Default encryption" muestra `AES-256`, y pestaña **Permissions** → confirmar que "Block public access" muestra las 4 reglas activas.
9. 📸 **Evidencia (`leccion-1-s3-block-public-access/`)**: captura de la pestaña Permissions mostrando Block Public Access ON, y captura de la pestaña Properties mostrando el cifrado por defecto.

## Lección 2: CloudTrail

**Objetivo:** activar trazabilidad de todas las acciones sobre la cuenta.

1. Primero crear el bucket de logs: **S3** → **Create bucket** → nombre sugerido `bluewave-cloudtrail-logs-<tus-iniciales>-<random>`, misma región, **cifrado SSE-S3 activado** y **Block Public Access activado** (mismo criterio que el bucket de datos).
2. Consola → **CloudTrail** → **Create trail**.
3. Nombre del trail: `bluewave-security-trail`.
4. Storage location: **Create new S3 bucket** o **Use existing** → seleccionar el bucket de logs creado en el paso 1.
5. Dejar **Log file SSE-KMS encryption** desactivado (usamos el cifrado SSE-S3 ya configurado en el bucket; KMS requeriría gestión de llaves fuera del alcance de Academy) — ver ADR 0002 para el detalle de esta decisión.
6. Management events: **Read** y **Write** ambos activados (por defecto).
7. Crear el trail y esperar 1-2 minutos a que quede `Logging: ON`.
8. Generar al menos un evento (por ejemplo, entrar y salir de la consola de S3) y esperar ~5-15 minutos a que CloudTrail entregue el primer log al bucket.
9. Verificar: **CloudTrail** → **Event history** debe mostrar eventos recientes; y el bucket de logs debe empezar a poblarse con objetos bajo `AWSLogs/<account-id>/CloudTrail/...`.
10. 📸 **Evidencia (`leccion-2-cloudtrail/`)**: captura del trail con estado `Logging: ON`, captura de **Event history** con eventos registrados, y captura del bucket de logs mostrando al menos un archivo de log entregado.

## Lección 3: AWS Config

**Objetivo:** evaluar el cumplimiento de configuraciones de seguridad.

1. Consola → **AWS Config** → si es la primera vez, **Get started** / **Settings**.
2. Recording: **All resources supported in this region** (o, si el Lab limita el alcance, al menos incluir S3).
3. Delivery method: usar un bucket S3 propio para Config (puede ser un tercer bucket `bluewave-config-<tus-iniciales>` o, si la cuota de Academy lo permite, uno gestionado automáticamente por AWS Config).
4. IAM role: usar el rol sugerido por AWS Config basado en `LabRole` (Academy no permite crear roles nuevos — si el asistente pide crear un rol personalizado, usar la opción de rol existente / `LabRole`).
5. Confirmar y habilitar Config.
6. Ir a **Rules** → **Add rule** → buscar `s3-bucket-public-read-prohibited` (regla administrada por AWS) → **Next** → dejar los parámetros por defecto → **Add rule**.
7. Esperar unos minutos a que Config evalúe el recurso (el bucket de la Lección 1).
8. Verificar el resultado: debería mostrar **Compliant** (porque el bucket tiene Block Public Access activo).
9. (Opcional, para evidenciar el caso `NON_COMPLIANT`): crear un bucket de prueba desechable con acceso público permitido, observar cómo Config lo marca `Non-compliant`, y luego eliminarlo. Documentar ambos estados si se hace esta prueba.
10. 📸 **Evidencia (`leccion-3-aws-config/`)**: captura de la regla `s3-bucket-public-read-prohibited` con el resultado de cumplimiento (Compliant/Non-compliant) del bucket evaluado.

## Lección 4: CloudWatch (alarma)

**Objetivo:** configurar una alerta de seguridad.

Ver ADR 0004 para la justificación de la métrica elegida. Dos caminos posibles (elegir uno según el tiempo disponible en la sesión de Academy):

### Opción A: Alarma de seguridad basada en CloudTrail (recomendada, más alineada al proyecto)

1. **CloudTrail** → confirmar el trail de la Lección 2 → **Edit** → en "CloudWatch Logs", habilitar el envío a un **CloudWatch Logs log group** (crear uno nuevo, ej. `bluewave-cloudtrail-logs`), usando el rol sugerido (basado en `LabRole`).
2. **CloudWatch** → **Log groups** → abrir `bluewave-cloudtrail-logs` → **Metric filters** → **Create metric filter**.
3. Filter pattern sugerido (uso de la cuenta root, evento sensible):
   ```
   { $.userIdentity.type = "Root" }
   ```
4. Asignar nombre de métrica, ej. `RootAccountUsageCount`, namespace `BlueWaveSecurityMetrics`.
5. **Create alarm** a partir de esa métrica: threshold `>= 1` en 5 minutos, sin acción de notificación obligatoria (SNS es opcional en Academy).
6. Probar generando el evento (iniciar sesión como root si es posible, o documentar el mecanismo) y verificar que la alarma pase a `ALARM`; luego confirmar que vuelve a `OK`.

### Opción B: Alarma simple de utilización de CPU (fallback pragmático)

1. Lanzar una instancia EC2 (t2.micro, usando `LabRole`) si no hay una disponible ya en el proyecto.
2. **CloudWatch** → **Alarms** → **Create alarm** → seleccionar la métrica `CPUUtilization` de esa instancia.
3. Threshold sugerido: `>= 70%` durante 5 minutos.
4. Generar carga (ej. un `stress` simple o simplemente dejar documentado el umbral) y verificar los estados `OK` / `ALARM` en el historial de la alarma.

5. 📸 **Evidencia (`leccion-4-cloudwatch-alarm/`)**: captura de la alarma creada mostrando su configuración, y captura del historial de estados (`OK` / `ALARM`).

## Lección 5: Documentación y diagrama

1. Completar el diagrama en draw.io siguiendo `diagrams/GUIA-DIAGRAMA.md`, exportarlo como PNG a `diagrams/arquitectura-seguridad.png`.
2. Revisar que todas las carpetas de `docs/evidencias/` tengan al menos una captura.
3. Completar `entregable-word/Cloud-Secure-Evaluacion-Modulo9.docx` con las capturas, el diagrama y una sección de conclusiones y aprendizajes personales.
4. (Opcional) Preparar una presentación breve de 5-6 diapositivas resumiendo el proyecto para el portafolio.
