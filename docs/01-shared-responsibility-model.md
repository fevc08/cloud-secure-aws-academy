# Modelo de Responsabilidad Compartida aplicado a Cloud Secure

## Concepto general

El [Modelo de Responsabilidad Compartida de AWS](https://aws.amazon.com/compliance/shared-responsibility-model/) divide las obligaciones de seguridad en dos planos:

- **AWS es responsable de la seguridad *de* la nube**: infraestructura física, hardware, software base, redes y las instalaciones que operan los servicios administrados (S3, CloudTrail, Config, CloudWatch).
- **El cliente es responsable de la seguridad *en* la nube**: configuración de los servicios, gestión de identidades y accesos, clasificación de los datos, cifrado, y controles de red que el cliente decide activar.

En servicios administrados como S3, CloudTrail, Config y CloudWatch, AWS gestiona la infraestructura subyacente, pero **la configuración de seguridad (cifrado, bloqueo de acceso público, políticas, alarmas) siempre es responsabilidad del cliente**. Este es el eje central del proyecto: la "revisión preliminar" del caso Blue Wave detectó justamente fallas del lado del cliente (bucket sin cifrar, sin auditoría, sin monitoreo), no de la infraestructura de AWS.

## Aplicación al proyecto

| Capa | Responsable | Controles aplicados en este proyecto |
|---|---|---|
| Infraestructura física, hipervisor, red global | AWS | No aplica configuración por parte del proyecto |
| Configuración del servicio S3 (cifrado, Block Public Access, políticas de bucket) | Cliente (Blue Wave / nosotros) | ADR 0001: cifrado SSE-S3 + Block Public Access |
| Registro y trazabilidad de la actividad de la cuenta | Cliente | ADR 0002: CloudTrail habilitado, logs en S3 cifrado |
| Verificación continua de cumplimiento | Cliente | ADR 0003: regla de AWS Config activa |
| Monitoreo y respuesta a eventos anómalos | Cliente | ADR 0004: alarma de CloudWatch |
| Gestión de identidades y privilegios | Cliente (limitado por Academy) | Uso del `LabRole` existente; no se crean usuarios/roles IAM nuevos, pero sí se restringe el acceso a nivel de recurso (bucket policy + Block Public Access) |

## Principio de privilegio mínimo en el contexto de AWS Academy

AWS Academy Learner Lab no permite crear roles ni políticas IAM personalizadas: todos los servicios se ejecutan bajo el rol `LabRole` preconfigurado. Esto significa que el privilegio mínimo **no se aplica a nivel de identidad** (no podemos crear un rol acotado solo a S3, por ejemplo), pero sí se puede y se debe aplicar **a nivel de recurso**:

- **Block Public Access en S3** es en sí mismo una expresión del principio de mínimo privilegio a nivel de recurso, por defecto, nadie fuera de la cuenta puede acceder al bucket, sin necesidad de tocar IAM.
- **Las reglas de AWS Config** actúan como una verificación continua de que ese privilegio mínimo se mantiene en el tiempo (evita "drift" de configuración).
- **Las alarmas de CloudWatch** detectan cuándo alguien intenta ampliar el acceso más allá de lo previsto (por ejemplo, cambios de configuración de seguridad).

Este enfoque: mínimo privilegio a nivel de recurso, ya que a nivel de identidad está limitado por el laboratorio, es el criterio que se sigue en todo el proyecto y se documenta explícitamente en cada ADR cuando aplica.

## Diagrama conceptual (ver `diagrams/`)

El diagrama de arquitectura incluye una franja superior que representa la línea divisoria del modelo de responsabilidad compartida, con "AWS" arriba (infraestructura) y "Cliente / Blue Wave" abajo (S3, CloudTrail, Config, CloudWatch configurados). Ver `diagrams/GUIA-DIAGRAMA.md` para el detalle de cómo construirlo.
