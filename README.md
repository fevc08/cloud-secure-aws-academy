# Cloud Secure — Evaluación Módulo 9 (SOFOFA)

Proyecto de aplicación de controles básicos de seguridad cloud sobre AWS Academy Learner Lab, desarrollado para la evaluación del Módulo 9 del programa de Arquitectura Cloud de SOFOFA.

## Situación inicial

La fintech ficticia **Blue Wave** migró su core bancario a AWS Academy Learner Lab. Una revisión preliminar detectó configuraciones débiles de seguridad y falta de monitoreo. Este proyecto aplica medidas básicas de seguridad cloud con los recursos disponibles en AWS Academy, documentando la arquitectura y las evidencias.

## Objetivo

Aplicar principios de seguridad cloud en AWS Academy, priorizando:

- Protección de datos en Amazon S3.
- Monitoreo y auditoría con AWS CloudTrail y Amazon CloudWatch.
- Cumplimiento básico con AWS Config.
- Representación y documentación clara de los controles aplicados.

## Arquitectura de seguridad aplicada

| Control | Servicio AWS | Propósito |
|---|---|---|
| Cifrado y bloqueo de acceso público | Amazon S3 (SSE-S3 + Block Public Access) | Proteger datos en reposo y evitar exposición accidental |
| Auditoría de eventos | AWS CloudTrail → S3 (bucket cifrado) | Trazabilidad de todas las acciones sobre la cuenta |
| Cumplimiento continuo | AWS Config (regla `s3-bucket-public-read-prohibited`) | Evaluar de forma continua que los recursos cumplan la política de seguridad |
| Monitoreo y alertas | Amazon CloudWatch (alarma) | Detectar y notificar eventos de seguridad o de uso anómalo |

El detalle de cada decisión está documentado como ADR en [`adr/`](./adr), y el diseño conceptual completo en [`docs/02-arquitectura-seguridad.md`](./docs/02-arquitectura-seguridad.md).

## Estructura del repositorio

```
cloud-secure-aws-academy/
├── README.md
├── docs/
│   ├── 01-shared-responsibility-model.md   # Modelo de responsabilidad compartida aplicado al proyecto
│   ├── 02-arquitectura-seguridad.md        # Diseño conceptual de la arquitectura de seguridad
│   ├── 03-guia-implementacion-paso-a-paso.md  # Guía de ejecución en la consola de AWS Academy
│   └── evidencias/                         # Capturas de pantalla por lección
│       ├── leccion-1-s3-block-public-access/
│       ├── leccion-2-cloudtrail/
│       ├── leccion-3-aws-config/
│       └── leccion-4-cloudwatch-alarm/
├── adr/                                    # Decisiones de arquitectura (Architecture Decision Records)
│   ├── 0001-cifrado-y-block-public-access-en-s3.md
│   ├── 0002-cloudtrail-multi-region-vs-single-region.md
│   ├── 0003-seleccion-regla-aws-config.md
│   └── 0004-metrica-y-umbral-alarma-cloudwatch.md
├── diagrams/
│   ├── arquitectura-seguridad.drawio       # Fuente editable (draw.io)
│   ├── arquitectura-seguridad.png          # Export (generar desde draw.io)
│   └── GUIA-DIAGRAMA.md                    # Guía paso a paso para completar el diagrama
└── entregable-word/
    └── Cloud-Secure-Evaluacion-Modulo9.docx  # Documento final exigido por la rúbrica
```

## Restricciones de AWS Academy Learner Lab consideradas

- No se crean roles IAM nuevos: se usa el rol `LabRole` ya existente para los servicios que lo requieran.
- La sesión de Academy expira y reinicia los recursos: los pasos de implementación (`docs/03-guia-implementacion-paso-a-paso.md`) están pensados para ejecutarse y evidenciarse dentro de una misma sesión activa.
- Región de trabajo: la región disponible por defecto en el Learner Lab (normalmente `us-east-1`).
- Los nombres de Security Group no pueden empezar con `sg-` (no aplica directamente a este proyecto, ya que no se crean Security Groups, pero se deja documentado como estándar del programa).

## Referencias oficiales

- [Modelo de Responsabilidad Compartida de AWS](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [Amazon S3 — Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [Amazon S3 — Cifrado del lado del servidor](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html)
- [AWS CloudTrail — Getting Started](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-getting-started.html)
- [AWS Config — Getting Started](https://docs.aws.amazon.com/config/latest/developerguide/getting-started.html)
- [Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)

## Portafolio

Proyecto registrado como **"Cloud Secure en AWS Academy"**, destacando:

- Uso combinado de S3, CloudTrail, Config y CloudWatch para reforzar la postura de seguridad de una cuenta cloud.
- Aplicación del principio de privilegio mínimo con los recursos accesibles en un entorno de laboratorio (Academy Learner Lab).
- Evidencias de cumplimiento documentadas con capturas y diagrama de arquitectura.

## Autor

Fidel — [github.com/fevc08](https://github.com/fevc08)
