# ADR 0003: Selección de la regla de AWS Config: `s3-bucket-public-read-prohibited`

## Estado
Aceptado

## Contexto
AWS Config permite evaluar el cumplimiento de los recursos de la cuenta contra reglas administradas (predefinidas por AWS) o reglas personalizadas (implementadas con una función Lambda). La rúbrica pide activar Config y revisar al menos una regla, con el ejemplo explícito de "S3 bucket público".

## Decisión
Usar la regla administrada por AWS **`s3-bucket-public-read-prohibited`**, aplicada sobre el bucket de datos creado en la Lección 1.

## Alternativas consideradas

| Alternativa | Por qué no se eligió |
|---|---|
| **Regla personalizada vía AWS Lambda** | Permitiría lógica de validación a medida, pero requiere crear una función Lambda con permisos de ejecución propios. En AWS Academy Learner Lab no se pueden crear roles IAM nuevos (solo se dispone de `LabRole`), lo que complica innecesariamente el despliegue de una Lambda con los permisos exactos que necesitaría Config para invocarla. Para el alcance "básico" de esta evaluación, no aporta valor proporcional al riesgo de que la sesión del Lab expire antes de completarlo. |
| **`s3-bucket-server-side-encryption-enabled`** (regla administrada alternativa) | También es una regla administrada válida y relevante (verifica cifrado en vez de acceso público). Se descarta como *regla principal* porque la rúbrica da explícitamente el ejemplo de "S3 bucket público", y esta opción está más directamente conectada con el control de Block Public Access de la Lección 1, generando trazabilidad clara entre lecciones. Queda documentada como candidata a **regla adicional** si se dispone de tiempo extra en la sesión de Academy. |

## Consecuencias
- La evaluación de esta regla depende directamente de la configuración hecha en la Lección 1 (ADR 0001): si Block Public Access se mantiene activo, el recurso se reporta `COMPLIANT`; si alguien lo desactivara, Config lo marcaría `NON_COMPLIANT` de forma automática, sin intervención manual.
- Esto demuestra el valor de Config como control de **cumplimiento continuo** (detecta desviaciones de configuración a lo largo del tiempo), en contraste con una revisión manual puntual.
- No se requiere gestión de código (Lambda) ni permisos IAM adicionales, compatible con las restricciones de `LabRole` en Academy.

## Referencias
- [AWS Config - Getting Started](https://docs.aws.amazon.com/config/latest/developerguide/getting-started.html)
- [AWS Config - Regla administrada `s3-bucket-public-read-prohibited`](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-public-read-prohibited.html)
