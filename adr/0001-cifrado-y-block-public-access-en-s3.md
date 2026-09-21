# ADR 0001: Cifrado SSE-S3 y Block Public Access en el bucket de datos

## Estado
Aceptado

## Contexto
El bucket S3 que representa los datos "core bancario" de Blue Wave debe protegerse contra acceso público y contra exposición de datos en reposo. AWS ofrece varias opciones de cifrado del lado del servidor (SSE-S3, SSE-KMS, SSE-C) y controles de acceso público independientes del cifrado (Block Public Access, bucket policies, ACLs).

## Decisión
Se activa:
- **Cifrado por defecto SSE-S3** (`AES-256`, llaves gestionadas por AWS) para todos los objetos del bucket.
- **Block Public Access** con las 4 configuraciones activas (`BlockPublicAcls`, `IgnorePublicAcls`, `BlockPublicPolicy`, `RestrictPublicBuckets`).

## Alternativas consideradas

| Alternativa | Por qué no se eligió |
|---|---|
| **SSE-KMS** (llave gestionada por el cliente o AWS) | Ofrece auditoría más granular (cada uso de la llave queda en CloudTrail) y control de rotación, pero introduce gestión de llaves KMS y política de llaves adicional. Para el alcance de esta evaluación (controles *básicos* de seguridad) es complejidad innecesaria; además, en AWS Academy Learner Lab la creación de recursos KMS puede estar sujeta a las mismas limitaciones de `LabRole` que otros servicios, agregando riesgo a la ejecución dentro del tiempo limitado de sesión. |
| **Solo Block Public Access, sin cifrado explícito** | No cumple el requisito de la rúbrica ("Buckets S3 cifrados **y** bloqueados") ni las buenas prácticas de protección de datos en reposo para un caso de uso fintech. |
| **Bucket policy restrictiva en vez de Block Public Access** | Una policy bien escrita puede lograr un efecto similar, pero es más propensa a errores de configuración (JSON mal formado, condiciones incompletas) y no es la herramienta que AWS recomienda como primera línea de defensa contra exposición pública accidental. Block Public Access es un control a nivel de cuenta/bucket más simple y menos propenso a errores, adecuado para el nivel de esta evaluación. |

## Consecuencias
- No es posible compartir el bucket con otras cuentas AWS externas sin cambiar esta configuración explícitamente (efecto deseado: privilegio mínimo por defecto).
- Con SSE-S3, AWS gestiona la rotación de llaves automáticamente; no se requiere administración adicional, pero tampoco se tiene visibilidad granular del uso de la llave en CloudTrail (trade-off aceptado dado el alcance del proyecto).
- Este control es la base sobre la que se apoya la regla de AWS Config `s3-bucket-public-read-prohibited` (ver ADR 0003): al mantener Block Public Access activo, el recurso se evalúa como `COMPLIANT`.

## Referencias
- [Amazon S3 - Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [Amazon S3 - Server-side encryption](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html)
