# Bitácora SmartH2O — Semana 3

**Proyecto:** SmartH2O – Sistema de Telemetría y Alertas para el Monitoreo Inteligente del Consumo de Agua
**Fase:** Fase 1 – Diseño de arquitectura y datos (cierre de Fase 1)
**Semana:** 3 · Período: 16 – 22 de junio de 2026
**Tutor:** Ing. Carlos G. Rodriguez
**Ciclo:** IV

---

## 1. Información general de la semana

| Campo | Detalle |
|-------|---------|
| Integrantes presentes | Todos los integrantes (5/5) |
| Roles activos | Product Owner · Ing. IoT Jr. · Ing. Cloud/Backend Jr. · Dev. Dashboard Jr. · Ing. Integración Jr. |
| Fase actual | Fase 1 – Diseño de arquitectura y datos (cierre de Fase 1) |
| Entregable objetivo | Payload JSON definitivo · `influxdb_smarth2o_v3.py` con 12 queries Flux · 5 reglas R01–R05 · 3 diagramas Draw.io · Documento de Análisis v2.0 |
| Acuerdos clave | Payload JSON confirmado con ID `AARD-EDIF-A-SAN1` · Measurement: `consumo_agua` · 12 queries Flux implementadas por Azucena · R05 (`anomaly_flag`) añadida por Karen · 3 diagramas Draw.io entregados |
| Estado general | ✅ Fase 1 completada. Todos los entregables del análisis y diseño listos. `influxdb_smarth2o_v3.py` funcional con 12 queries. 5 reglas de detección. 3 diagramas. Documento de Análisis v2.0 entregado. |

**Riesgos identificados:**
- MongoDB explorado como alternativa (D06) y descartado formalmente. InfluxDB confirmado.
- Diagramas Draw.io requerirán actualización si cambia el stack.

---

## 2. Actividades realizadas

| Actividad | Descripción técnica | Responsable | Evidencia verificable |
|-----------|---------------------|-------------|----------------------|
| Payload JSON definitivo confirmado | Estructura final: `sensor_id` (AARD-EDIF-A-SAN1), `timestamp` ISO 8601, `flow_rate`, `cumulative_volume`, `status`, `anomaly_flag`, `location{building,floor,zone}` | Christian Castro (IoT Jr.) | Sección payload en `Fase_1_Analisis_S2_S3.docx` con datos reales |
| Flujo de datos documentado | Simulador → MQTT → Servicio de ingesta → InfluxDB → Grafana → Alertas — 6 pasos con descripción | Christian Castro (IoT Jr.) | Sección 'Flujo de datos' en `Fase_1_Analisis_S2_S3.docx` |
| `influxdb_smarth2o_v3.py` — script completo | 600+ líneas: MQTT client con reconexión automática (1s–60s), validación schema guard, conversión ISO→nanosegundos, creación bucket con retención 30d, 12 queries Flux | Azucena Castillo (Cloud Jr.) | `smarth2o-esit/database/influxdb_smarth2o_v3.py` |
| 12 queries Flux implementadas | Q1: última lectura por sensor · Q3: promedio por zona · Q5: anomalías 24h · Q7: media móvil 15min · Q11: consumo por piso · Q12: tasa de anomalías con `join()` | Azucena Castillo (Cloud Jr.) | `influxdb_smarth2o_v3.py` — diccionario `CONSULTAS` con 12 queries nombradas |
| Catálogo de 5 sensores en código | `CATALOGO_SENSORES`: `AARD-EDIF-A-SAN1` a `SAN2` con `zone` y `floor` — usado para cargar datos de ejemplo | Azucena Castillo (Cloud Jr.) | `influxdb_smarth2o_v3.py` — constante `CATALOGO_SENSORES` |
| 5 reglas de detección R01–R05 | R01: caudal >20 L/min 10min · R02: >5 L/min nocturno · R03: `cumulative_volume` 150% promedio · R04: sin datos >5min · R05: `anomaly_flag=true` | Karen Caceres (Integración Jr.) | `smarth2o-esit/alerts/src/rules.py` + README con tabla de reglas |
| Estructura completa del repo `alerts/` | 7 módulos: `detector.py`, `rules.py`, `notifier.py`, `telegram_bot.py`, `email_sender.py`, `debounce.py`, `config.py` · `tests/` · `.env.example` | Karen Caceres (Integración Jr.) | `smarth2o-esit/alerts/README.md` con estructura documentada |
| Formato del mensaje Telegram documentado | Mensaje con emoji, sensor real `AARD-A-SAN-P1`, zona, caudal 24.3 L/min, duración 12 min, regla R01 y posible causa | Karen Caceres (Integración Jr.) | `smarth2o-esit/alerts/README.md` — sección 'Formato del mensaje de alerta' |
| 3 diagramas Draw.io generados | `01_arquitectura_sistema.drawio` · `02_flujo_de_datos.drawio` (12 pasos) · `03_responsabilidades_por_rol.drawio` (5 columnas con estado por fase) | Todo el equipo / Product Owner | `smarth2o-esit/docs/*.drawio` (raíz del repo) |
| Exploración y descarte formal de MongoDB (D06) | MongoDB evaluado como alternativa a InfluxDB. Descartado: no especializado en series temporales IoT. InfluxDB confirmado definitivamente. | David Rondon (PO) + Azucena | D06 registrada como ❌ descartada en `/Recursos/registro_de_decisiones.md` |
| Documento de Análisis v2.0 | Documento consolidado con aportes individuales de los 5 integrantes, scope actualizado, InfluxDB confirmado, reglas de Karen y KPIs de Mirna | Todo el equipo (coord. David) | `SmartH2O_Documento_Analisis_Fase1_v2.docx` — Entregable Fase 1 |

---

## 3. Herramientas y servicios utilizados

| Herramienta / Servicio | Propósito | Observaciones |
|-----------------------|-----------|---------------|
| `influxdb_smarth2o_v3.py` | Módulo central de integración MQTT + InfluxDB — 600+ líneas | Azucena: 12 queries, validación schema guard, reconexión MQTT automática, health check `ping()` |
| InfluxDB Cloud — Flux | BD de series temporales + lenguaje de consulta | 12 queries Flux: `aggregateWindow` 15min, `join()` para tasa de anomalías, consumo por piso con TAG `floor` |
| Telegram Bot API | Canal de alertas para las 5 reglas R01–R05 | Karen: formato de mensaje con datos reales, severidades y cooldown en `debounce.py` |
| Draw.io (`app.diagrams.net`) | 3 diagramas de arquitectura, flujo y responsabilidades | Archivos `.drawio` en raíz del repo `docs/` — abiertos con `app.diagrams.net` |
| GitHub `smarth2o-esit` | Control de versiones | Commits en `database/` (Azucena), `alerts/` (Karen). Diagramas en raíz de `docs/` |
| `broker.hivemq.com:1883` | Broker MQTT cloud | Frecuencia: 5 seg por sensor. Sin autenticación en free tier. Reconexión automática configurada. |

---

## 4. Problemas detectados

| Problema | Impacto | Causa probable | Resolución / Evidencia |
|----------|---------|----------------|------------------------|
| MongoDB evaluado como alternativa (D06) — genera confusión en el equipo | Medio — riesgo de scope creep en BD | Exploración legítima de alternativas durante el diseño | ✅ Resuelto: D06 descartada formalmente. InfluxDB confirmado. Documentado en `/Recursos/registro_de_decisiones.md` |
| Diagramas Draw.io requerirán actualización si cambia el stack | Bajo — costo de mantenimiento | Los diagramas reflejan el estado actual | 🔄 Mitigación: los archivos `.drawio` son editables. Actualizar en Semana 4 si hay cambios |
| Consulta Q12 (tasa anomalías) es compleja — usa `join()` Flux | Medio — curva de aprendizaje de Flux | `join()` en Flux requiere nomenclatura específica de tablas | ✅ Resuelto: Azucena implementó y documentó la consulta en `influxdb_smarth2o_v3.py` |

---

## 5. Soluciones aplicadas

| Solución aplicada | Problema asociado | Responsable | Evidencia |
|------------------|-------------------|-------------|-----------|
| Descarte formal de MongoDB (D06) con justificación técnica | Confusión sobre BD definitiva | David Rondon + Azucena Castillo | D06 en `/Recursos/registro_de_decisiones.md` — InfluxDB confirmado definitivamente |
| 12 queries Flux documentadas con nomenclatura clara | Complejidad de Flux query language | Azucena Castillo | `influxdb_smarth2o_v3.py` — diccionario `CONSULTAS` con nombres descriptivos |
| R05 añadida como regla de anomalía directa del simulador | Falta de regla para modo prueba | Karen Caceres | `src/rules.py` — R05: `anomaly_flag=true` → Critical → Telegram+Email |

---

## 6. Estado del proyecto

| Área | Estado | Comentarios |
|------|--------|-------------|
| Payload JSON definitivo | ✅ Completado | `AARD-EDIF-A-SAN1`, 7 campos, ISO 8601, `location` con `floor` como string |
| `influxdb_smarth2o_v3.py` | ✅ Completado | 600+ líneas: MQTT sub, validación, 12 queries Flux, reconexión automática, health check |
| Esquema InfluxDB confirmado | ✅ Completado | Measurement: `consumo_agua` · 4 TAGs · 4 FIELDs · Retención 30 días |
| 5 reglas de detección R01–R05 | ✅ Completado | R01-Critical, R02-Warning, R03-Warning, R04-Info, R05-Critical — todas con canal definido |
| Estructura repo `alerts/` (7 módulos) | ✅ Completado | `detector`, `rules`, `notifier`, `telegram_bot`, `email_sender`, `debounce`, `config` |
| 3 diagramas Draw.io | ✅ Completado | `01_arquitectura_sistema`, `02_flujo_de_datos`, `03_responsabilidades_por_rol` |
| D06 (MongoDB) descartada formalmente | ✅ Completado | InfluxDB confirmado como BD definitiva. D06 ❌ en registro de decisiones |
| Documento de Análisis v2.0 | ✅ Completado | 5 integrantes con aportes verificables. Entregable Fase 1 listo. |
| Dashboard Grafana | 🔄 En diseño | 6 KPIs especificados por Mirna. Configuración datasource pendiente para Fase 3 |

---

## 7. Plan de trabajo — Semana 4 (Fase 2)

| Tarea | Responsable | Fase | Entregable esperado |
|-------|-------------|------|---------------------|
| Implementar `simulator.py` con 5 sensores publicando en MQTT | Christian Castro (IoT Jr.) | Fase 2 – S4 | `src/simulator.py` con 5 threads + modo anomalía |
| Levantar `docker-compose`: Mosquitto + InfluxDB local | Azucena Castillo (Cloud Jr.) | Fase 2 – S4 | `docker-compose up -d` sin errores |
| Conectar simulador → MQTT → ingestión → InfluxDB y evidenciar | Azucena + Christian | Fase 2 – S4/S5 | Screenshot de datos en InfluxDB Data Explorer |
| Ejecutar `pytest` en simulator y en ingestion | Christian + Azucena | Fase 2 – S4/S5 | Captura de pytest passing |
| Verificar que `detector.py` evalúa reglas R01–R05 contra InfluxDB | Karen Caceres (Integración Jr.) | Fase 2 – S5 | Log del detector mostrando evaluación de reglas |
| Completar bitácoras Semana 4 y 5 | David Rondon (PO) | Fase 2 | Entradas en `/bitacoras/` |

---

## 8. Aportes individuales verificables

| Integrante | Rol | Tarea técnica específica | Entregable afectado | Evidencia verificable | Resultado |
|-----------|-----|--------------------------|---------------------|----------------------|-----------|
| Azucena Castillo | Cloud Jr. | `influxdb_smarth2o_v3.py` — script completo 600+ líneas con 12 queries Flux | `smarth2o-esit/database` | `influxdb_smarth2o_v3.py` en GitHub — MQTT + InfluxDB integrado | ✅ |
| Azucena Castillo | Cloud Jr. | Catálogo de 5 sensores (`CATALOGO_SENSORES`) con `zone` y `floor` para carga de datos de ejemplo | `influxdb_smarth2o_v3.py` | Constante `CATALOGO_SENSORES` con los 5 sensores del proyecto | ✅ |
| Azucena Castillo | Cloud Jr. | 12 queries Flux implementadas incluyendo Q11 (consumo por piso) y Q12 (tasa anomalías con `join()`) | `database/influxdb/queries/` | Diccionario `CONSULTAS` en `influxdb_smarth2o_v3.py` | ✅ |
| Christian Castro | IoT Jr. | Payload JSON definitivo con IDs reales `AARD-EDIF-A-SAN1` y flujo de datos documentado | Análisis S2-S3 | `Fase_1_Analisis_S2_S3.docx` — payload + flujo de datos | ✅ |
| Karen Caceres | Integración Jr. | 5 reglas R01–R05 con condición, severidad y canal. R03 nueva: `cumulative_volume` >150% promedio | `alerts/src/rules.py` | `README alerts/` — tabla de reglas | ✅ |
| Karen Caceres | Integración Jr. | Estructura completa repo `alerts/` con 7 módulos documentados y formato de mensaje Telegram real | `smarth2o-esit/alerts/` | `README.md` — estructura + formato de mensaje con `AARD-A-SAN-P1` | ✅ |
| Mirna Jimenez | Dashboard Jr. | 6 KPIs del dashboard con tipo de panel Grafana y valor operativo para la DAIR | `indicadores_dashboard.docx` | KPI 1 a 6 completos con justificación operativa | ✅ |
| David Rondon | Product Owner | 3 diagramas Draw.io (arquitectura, flujo de datos, responsabilidades) | `docs/*.drawio` | `smarth2o-esit/docs/` — 3 archivos `.drawio` | ✅ |
| David Rondon | Product Owner | Descarte formal D06 (MongoDB) y confirmación de InfluxDB como BD definitiva | `registro_de_decisiones.md` | D06 en `/Recursos/registro_de_decisiones.md` — ❌ descartada con razón | ✅ |
| David Rondon | Product Owner | Documento de Análisis v2.0 con aportes individuales de los 5 integrantes | `SmartH2O_Documento_Analisis_Fase1_v2.docx` | Entregable Fase 1 — documento consolidado completo | ✅ |

---

## 9. Cumplimiento individual

| Integrante | Tarea asignada | Estado | Observación | Seguimiento |
|-----------|----------------|--------|-------------|-------------|
| Azucena Castillo | Implementar `influxdb_smarth2o_v3.py` con 12 queries Flux y reconexión MQTT automática | ✅ Completado | Script de 600+ líneas con health check, validación, bucket con retención y 12 queries | No |
| Christian Castro | Confirmar payload JSON definitivo y documentar flujo de datos | ✅ Completado | `Fase_1_Analisis_S2_S3.docx` con payload y flujo de 6 pasos | No |
| Karen Caceres | Implementar 5 reglas R01–R05 y estructura completa del repo `alerts/` | ✅ Completado | 7 módulos en `alerts/src/` + tabla de reglas con formato de mensaje Telegram | No |
| Mirna Jimenez | Especificar 6 KPIs del dashboard con valor operativo para la DAIR | ✅ Completado | `indicadores_dashboard.docx` entregado con 6 KPIs justificados | No |
| David Rondon | Generar 3 diagramas Draw.io y Documento de Análisis v2.0 | ✅ Completado | 3 archivos `.drawio` en raíz de `docs/` + Documento de Análisis v2.0 entregado | No |

---

## 14. Registro de evidencias

| No. | Descripción de la evidencia | Tipo | Responsable | Enlace / Referencia |
|-----|----------------------------|------|-------------|---------------------|
| E13 | `influxdb_smarth2o_v3.py` — script MQTT + InfluxDB 600+ líneas | Python | Azucena Castillo | `smarth2o-esit/database/influxdb_smarth2o_v3.py` |
| E14 | 12 queries Flux implementadas (Q1–Q12) | Python/Flux | Azucena Castillo | `influxdb_smarth2o_v3.py` — diccionario `CONSULTAS` |
| E15 | Payload JSON definitivo con datos reales | Documento Word | Christian Castro | `Fase_1_Analisis_S2_S3.docx` — sección payload |
| E16 | 5 reglas R01–R05 con tabla de reglas | Documento / Python | Karen Caceres | `alerts/README.md` + `src/rules.py` |
| E17 | Capturas del repo `alerts/` — estructura y reglas de detección | Screenshots | Karen Caceres | `Evidencias/Fase 1/` — capturas del README de `alerts/` |
| E18 | Formato mensaje Telegram con sensor real `AARD-A-SAN-P1` | Screenshot | Karen Caceres | `Evidencias/Fase 1/` — captura del formato de alerta |
| E19 | 6 KPIs del dashboard especificados | Documento Word | Mirna Jimenez | `indicadores_dashboard.docx` |
| E20 | 3 diagramas Draw.io (arquitectura, flujo, roles) | Draw.io (`.drawio`) | David Rondon | `smarth2o-esit/docs/*.drawio` (raíz del repo) |
| E21 | D06 descartada — InfluxDB confirmado | Markdown | David Rondon | `/Recursos/registro_de_decisiones.md` — D06 ❌ |
| E22 | Documento de Análisis v2.0 — Entregable Fase 1 | Word (`.docx`) | Todo el equipo | `SmartH2O_Documento_Analisis_Fase1_v2.docx` |

---

*Bitácora generada al cierre de la Semana 3 · Fase 1 (cierre) · SmartH2O*
*Próxima actualización: al cierre de la Semana 4 · Fase 2*
