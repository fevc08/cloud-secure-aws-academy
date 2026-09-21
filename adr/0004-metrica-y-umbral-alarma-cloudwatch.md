# ADR 0004: Métrica y umbral de la alarma de CloudWatch

## Estado
Aceptado

## Contexto
La rúbrica pide crear al menos una alarma en CloudWatch, sugiriendo dos ejemplos posibles: "cambios en seguridad" o "uso de CPU en una instancia". El proyecto busca reforzar específicamente la postura de **seguridad** de la cuenta (no el rendimiento de la infraestructura), por lo que la elección de métrica debe alinearse con ese objetivo siempre que el tiempo disponible en la sesión de AWS Academy lo permita.

## Decisión
Priorizar una **alarma de seguridad basada en un metric filter sobre los logs de CloudTrail** (por ejemplo, uso de la cuenta root, o llamadas a la API no autorizadas), documentada como **Opción A** en la guía de implementación. Se documenta explícitamente una **Opción B** (alarma de `CPUUtilization` sobre una instancia EC2) como fallback pragmático si la sesión de Academy no alcanza para completar la integración CloudTrail → CloudWatch Logs → Metric Filter → Alarm.

## Alternativas consideradas

| Alternativa | Por qué se documenta como fallback (Opción B) y no como decisión principal |
|---|---|
| **Alarma de `CPUUtilization` en EC2** | Es más simple de configurar (métrica nativa, sin pasos intermedios), pero no está directamente relacionada con el objetivo de seguridad del proyecto ("Cloud Secure"); mide carga de trabajo, no un evento de seguridad. Además, el proyecto no requiere obligatoriamente una instancia EC2 en el resto de las lecciones, por lo que crear una solo para esta alarma agrega un componente de infraestructura fuera del alcance central (S3 + CloudTrail + Config + CloudWatch). |
| **Alarma sobre métricas de S3 (ej. `4xxErrors`)** | Relevante pero más indirecta como señal de seguridad (mezcla errores de aplicación con intentos de acceso no autorizado); requiere métricas de solicitudes de S3 que no están habilitadas por defecto (hay que activar métricas de solicitud, con costo asociado). Se descarta por complejidad/costo adicional no justificado para el alcance "básico" de la evaluación. |

## Consecuencias
- La Opción A depende de que CloudTrail entregue logs a **CloudWatch Logs** (paso adicional a la sola entrega a S3 de la Lección 2), y de crear un **metric filter** que interprete el patrón de log deseado. Esto es más trabajo de configuración, pero produce una alarma directamente trazable a un evento de seguridad real, coherente con el resto del proyecto.
- Si el tiempo de sesión del Learner Lab es limitado, la Opción B garantiza cumplir igualmente el requisito mínimo de la rúbrica ("1 alarma creada en CloudWatch"), documentando la decisión y el trade-off tomado.
- Cualquiera de las dos opciones debe evidenciarse con al menos un cambio de estado observado (`OK` ↔ `ALARM`), tal como exige la rúbrica.

## Referencias
- [Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [Enviar eventos de CloudTrail a CloudWatch Logs](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/send-cloudtrail-events-to-cloudwatch-logs.html)
- [Crear un metric filter en CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html)
