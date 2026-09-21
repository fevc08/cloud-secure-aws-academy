# ADR 0002: Trail de CloudTrail multi-región (en vez de single-región)

## Estado
Aceptado

## Contexto
AWS CloudTrail permite crear un trail que capture eventos de una sola región o de todas las regiones (multi-region trail). AWS Academy Learner Lab normalmente restringe el trabajo práctico a una única región activa (p. ej. `us-east-1`), lo que en la práctica limita cuántos recursos "regionales" se pueden observar. Sin embargo, algunos eventos son de servicios **globales** (IAM, STS, Route 53, etc.) y solo se capturan completamente con un trail multi-región.

## Decisión
Configurar el trail como **multi-region trail** (opción por defecto al crear un trail desde la consola).

## Alternativas consideradas

| Alternativa | Por qué no se eligió |
|---|---|
| **Trail de una sola región** | Es más simple de explicar, pero no captura eventos de servicios globales (por ejemplo, cambios relacionados con IAM), lo cual es relevante para un caso de seguridad como el de Blue Wave. Además, no genera costo adicional usar multi-región para el primer trail de la cuenta (el primer trail por cuenta es gratuito en el nivel de "management events"), por lo que no hay ninguna ventaja real en limitarlo a una sola región. |

## Consecuencias
- El trail queda alineado con la [buena práctica recomendada por AWS](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-getting-started.html) de habilitar CloudTrail a nivel de cuenta completa, no solo de una región.
- En el contexto de Academy, aunque solo se trabaje activamente en una región, el trail multi-región no tiene costo adicional para el volumen de eventos de este proyecto y demuestra comprensión del comportamiento real de CloudTrail en una cuenta de producción.
- Los logs de todos los eventos (de cualquier región donde ocurriera actividad) se consolidan en el mismo bucket S3 de logs, simplificando el punto de revisión.

## Referencias
- [AWS CloudTrail - Getting Started](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-getting-started.html)
- [AWS CloudTrail - Multi-region trails](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-multi-region.html)
