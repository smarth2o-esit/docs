# Registro de Decisiones Técnicas — SmartH2O

**Proyecto:** SmartH2O – Sistema de Telemetría y Alertas para el Monitoreo Inteligente del Consumo de Agua
**Repositorio:** `smarth2o-esit/docs`
**Responsable de mantenimiento:** David Ali Rondon Majano (Product Owner)

> Este documento registra todas las decisiones arquitectónicas y tecnológicas tomadas durante el proyecto, incluyendo las alternativas evaluadas y las razones de descarte.

---

## Resumen de decisiones

| ID | Fecha | Decisión | Estado |
|----|-------|----------|--------|
| D01 | Jun 2026 | MQTT + InfluxDB como stack de telemetría | ✅ Confirmada |
| D02 | Jun 2026 | Un repositorio por componente en `smarth2o-esit` | ✅ Confirmada |
| D03 | Jun 2026 | Sesiones 1-on-1 por rol para delegar tareas | ✅ Completada |
| D04 | Jun 2026 | Cuentas cloud individuales por integrante | ✅ Confirmada |
| D05 | Jun 2026 | Umbrales de anomalía por zona definidos | ✅ Completada |
| D06 | Jun 2026 | Exploración de MongoDB como alternativa a InfluxDB | ❌ Descartada |

---

## D01 · Stack de telemetría: MQTT + InfluxDB

**Fecha:** Semana 1 — junio 2026
**Estado:** ✅ Confirmada
**Acordado por:** Todo el equipo en reunión grupal de kickoff

**Decisión:** Utilizar **MQTT 5.0** como protocolo de transmisión e **InfluxDB 2.x** como base de datos de series temporales.

**Razón de selección:**
- MQTT: protocolo IoT nativo con modelo pub/sub, QoS configurable, bajo consumo y reconexión automática.
- InfluxDB: optimizado para series temporales, Flux query language, retención nativa de 30 días, Data Explorer integrado.

**Alternativas evaluadas y descartadas:**

| Alternativa | Razón de descarte |
|-------------|-------------------|
| API REST (FastAPI) | Mayor latencia; no diseñada para transmisión continua de sensores IoT |
| MySQL | Base de datos relacional; no apta para métricas IoT de alta frecuencia |
| MongoDB | No especializado en series temporales; sin Flux ni retención nativa (ver D06) |
| PostgreSQL | Requiere extensión TimescaleDB adicional; mayor complejidad de setup |

**Configuración confirmada:**
- Broker: `broker.hivemq.com:1883` (cloud free, sin autenticación en free tier)
- Topic: `smarth2o/sensors`
- Frecuencia: 5 segundos por sensor
- Bucket: `water_telemetry` · Org: `smarth2o` · Retención: 30 días
- Measurement: `consumo_agua`

---

## D02 · Estructura de repositorios: uno por componente

**Fecha:** Semana 1 — junio 2026
**Estado:** ✅ Confirmada
**Acordado por:** Product Owner (David Rondon)

**Decisión:** Un repositorio por componente bajo la organización `smarth2o-esit` en GitHub.

**Razón:** Trazabilidad individual por rol; ownership claro de cada repositorio; facilita evaluación individual de aportes.

**Repositorios creados:**

| Repositorio | Responsable | Descripción |
|-------------|-------------|-------------|
| `smarth2o-esit/simulator` | Christian Castro | Simulador de sensores IoT |
| `smarth2o-esit/Ingestion` | Azucena Castillo | Servicio de ingestión MQTT → InfluxDB |
| `smarth2o-esit/database` | Azucena Castillo | Scripts y queries InfluxDB |
| `smarth2o-esit/dashboard` | Mirna Jimenez | Dashboard Grafana |
| `smarth2o-esit/alerts` | Karen Caceres | Sistema de detección y alertas |
| `smarth2o-esit/docs` | David Rondon | Documentación central del proyecto |

---

## D03 · Dinámica de trabajo: sesiones 1-on-1 por rol

**Fecha:** Semana 1 — junio 2026
**Estado:** ✅ Completada
**Responsable:** David Rondon (Product Owner)

**Decisión:** Realizar sesiones 1-on-1 individuales con cada integrante para delegar tareas específicas por fase (20 tareas por integrante en 6 fases).

**Razón:** Permitió definir entregables individuales y alinear expectativas sin ambigüedad. Aumenta la trazabilidad individual en la evaluación.

---

## D04 · Cuentas cloud: individuales por integrante

**Fecha:** Semana 1 — junio 2026
**Estado:** ✅ Confirmada

**Decisión:** Cada integrante crea y gestiona sus propias cuentas en los servicios cloud de su rol.

**Razón:** Maximizar créditos del free tier disponibles; cada integrante mantiene control de sus credenciales; facilita la evaluación individual de uso de plataformas.

**Cuentas activas:**

| Servicio | Responsable(s) |
|----------|---------------|
| InfluxDB Cloud | Azucena Castillo, Karen Caceres |
| Grafana Cloud | Mirna Jimenez |
| GitHub (`smarth2o-esit`) | Todos los integrantes |

---

## D05 · Umbrales de anomalía por zona

**Fecha:** Semana 2 — junio 2026
**Estado:** ✅ Completada
**Responsable:** Christian Castro (IoT Jr.) + Todo el equipo

**Decisión:** Definir umbrales de caudal normales y críticos por zona funcional, configurables en `src/rules.py`.

**Umbrales definidos:**

| Zona | Sensor ID | Umbral normal | Umbral crítico | Observación |
|------|-----------|---------------|----------------|-------------|
| Sanitarios Piso 1 | `AARD-EDIF-A-SAN1` / `SH2O-ZA-001` | 0–15 L/min | >20 L/min | Alto consumo en horario laboral |
| Cocina / Comedor | `AARD-EDIF-A-COCINA` / `SH2O-ZB-001` | 0–10 L/min | >15 L/min | Picos al mediodía |
| Áreas verdes / Riego | `AARD-EDIF-A-RIEGO` / `SH2O-ZC-001` | 0–20 L/min | >25 L/min | Detectar consumo fuera de horario de riego |
| Cisterna / Mantenimiento | `AARD-EDIF-A-CIST` / `SH2O-ZD-001` | 5–25 L/min | >30 L/min | Punto crítico para fugas y rebalses |
| Sanitarios Piso 2 | `AARD-EDIF-A-SAN2` / `SH2O-ZE-001` | 0–15 L/min | >20 L/min | Bajo consumo esperado en horario nocturno |

**Razón:** Una fuga de 0.5 L/min sin detectar durante 30 días equivale a 21,600 litros desperdiciados. SmartH2O detecta la anomalía en < 15 minutos desde el inicio del evento.

---

## D06 · Exploración de MongoDB como alternativa a InfluxDB

**Fecha:** Semana 3 — junio 2026
**Estado:** ❌ Descartada
**Responsable:** David Rondon (PO) + Azucena Castillo

**Decisión:** MongoDB fue evaluado formalmente como alternativa a InfluxDB y **descartado**.

**Razón de descarte:**
- MongoDB no está especializado en series temporales IoT.
- No incluye Flux query language ni políticas de retención nativas.
- InfluxDB ofrece `aggregateWindow`, `join()` en Flux, y retención configurable por bucket — características esenciales para telemetría de sensores.
- El descarte formal fortalece y justifica la elección de InfluxDB como la solución técnicamente superior.

**Resultado:** InfluxDB 2.x confirmado como **base de datos definitiva** del proyecto. La decisión D01 permanece vigente.

> ✅ **Scope confirmado:** MongoDB fue evaluado correctamente como parte del proceso de toma de decisiones. Su descarte formal no constituye un error sino una buena práctica de arquitectura.

---

*Registro de Decisiones · SmartH2O · ESIT – Estancia Profesional · Ciclo IV*
*Última actualización: 28 de junio de 2026 — Semana 3 · Fase 1 completada*
