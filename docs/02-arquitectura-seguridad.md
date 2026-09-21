# Arquitectura de seguridad: Cloud Secure

## Visión general

La arquitectura resuelve cuatro frentes de seguridad sobre la cuenta de AWS Academy Learner Lab usada por Blue Wave: protección de datos, auditoría, cumplimiento y monitoreo. Los cuatro controles son independientes entre sí pero se refuerzan mutuamente: CloudTrail audita lo que pasa con el bucket S3, Config verifica que el bucket siga cumpliendo la política de seguridad, y CloudWatch alerta si algo se sale de lo esperado.

## Componentes

### 1. Amazon S3 — Bucket de datos protegido

- **Cifrado en reposo**: SSE-S3 (`AES-256`) activado por defecto a nivel de bucket (ver ADR 0001).
- **Block Public Access**: las 4 opciones activadas (`BlockPublicAcls`, `IgnorePublicAcls`, `BlockPublicPolicy`, `RestrictPublicBuckets`).
- Este bucket representa los datos "core bancario" de Blue Wave que originalmente estaban mal configurados.

### 2. AWS CloudTrail — Auditoría de eventos

- Trail habilitado en la región activa de Academy.
- Los logs se entregan a un **segundo bucket S3, cifrado**, dedicado exclusivamente a logs (separado del bucket de datos de la lección 1, buena práctica para no mezclar datos de negocio con logs de auditoría).
- Registra todas las llamadas a la API de la cuenta (management events), incluyendo cambios de configuración de seguridad.

### 3. AWS Config — Cumplimiento continuo

- Config habilitado, apuntando a un bucket de configuración (puede reutilizar el bucket de logs de CloudTrail o uno propio, según cuota disponible en Academy).
- Regla administrada activa: `s3-bucket-public-read-prohibited` (ver ADR 0003), que evalúa continuamente si el bucket de la lección 1 permite lectura pública.
- El resultado (`COMPLIANT` / `NON_COMPLIANT`) es la evidencia central de la Lección 3.

### 4. Amazon CloudWatch — Monitoreo y alarma

- Una alarma configurada sobre un evento o métrica de seguridad (ver ADR 0004 para la justificación de cuál elegir).
- Estado observable en `OK` / `ALARM`, con evidencia de al menos un cambio de estado documentado.

## Flujo de datos (resumen)

```
Usuarios / Servicios AWS
        │
        ├── interactúan con ──► Bucket S3 (datos) ──[cifrado + Block Public Access]
        │
        ├── generan eventos de API ──► CloudTrail ──► Bucket S3 (logs, cifrado)
        │                                   │
        │                                   └──► (opcional) CloudWatch Logs ──► Métrica ──► Alarma
        │
        └── AWS Config evalúa el Bucket S3 (datos) ──► Regla: s3-bucket-public-read-prohibited ──► COMPLIANT / NON_COMPLIANT
```

## Mapeo con las lecciones de la evaluación

| Lección | Objetivo | Componente de esta arquitectura |
|---|---|---|
| 1 — Identidad y privilegio mínimo | Modelo de responsabilidad compartida + Block Public Access | Bucket S3 (datos) |
| 2 — Auditoría de eventos | Trazabilidad de acciones | CloudTrail + bucket S3 (logs) |
| 3 — Cumplimiento básico | Evaluación de configuraciones | AWS Config |
| 4 — Monitoreo y alarmas | Alertas de seguridad | CloudWatch |
| 5 — Documentación y diagrama | Consolidación | Este repositorio + `entregable-word/` |

## Ver también

- Guía de implementación paso a paso: [`03-guia-implementacion-paso-a-paso.md`](./03-guia-implementacion-paso-a-paso.md)
- Decisiones de arquitectura: [`../adr/`](../adr)
- Guía del diagrama: [`../diagrams/GUIA-DIAGRAMA.md`](../diagrams/GUIA-DIAGRAMA.md)
