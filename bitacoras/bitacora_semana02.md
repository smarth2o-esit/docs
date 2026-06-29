# Bitácora SmartH2O — Semana 2

**Proyecto:** SmartH2O – Sistema de Telemetría y Alertas para el Monitoreo Inteligente del Consumo de Agua
**Fase:** Fase 1 – Análisis del escenario hídrico
**Semana:** 2 · Período: 09 – 15 de junio de 2026
**Tutor:** Ing. Carlos G. Rodriguez
**Ciclo:** IV

---

## 1. Información general de la semana

| Campo | Detalle |
|-------|---------|
| Integrantes presentes | Todos los integrantes (5/5) |
| Roles activos | Product Owner · Ing. IoT Jr. · Ing. Cloud/Backend Jr. · Dev. Dashboard Jr. · Ing. Integración Jr. |
| Fase actual | Fase 1 – Análisis del escenario hídrico y diseño de la solución |
| Entregable objetivo | Documento de Análisis: puntos de monitoreo, variables, umbrales, flujo de datos y payload JSON |
| Acuerdos clave | Broker MQTT: `broker.hivemq.com:1883` · Topic: `smarth2o/sensors` · IDs duales confirmados: `AARD-EDIF-A-SAN1` / `SH2O-ZA-001` · Umbrales por zona definidos |
| Estado general | ✅ Completada. 5 puntos de monitoreo, 7 variables del sistema, umbrales por zona y flujo de datos documentados |

**Riesgos identificados:** Sincronización entre IDs del simulador (Christian) y el esquema de InfluxDB (Azucena). Mitigación: acuerdo de payload JSON único como fuente de verdad.

---

## 2. Actividades realizadas

| Actividad | Descripción técnica | Responsable | Evidencia verificable |
|-----------|---------------------|-------------|----------------------|
| Documentar 5 puntos de monitoreo | IDs, zonas, datos enviados y observación de integración para cada sensor | Christian Castro (IoT Jr.) | Tabla de sensores en `Fase_1_Analisis_S2_S3.docx` |
| Definir variables del sistema | 7 variables del payload: `sensor_id`, `timestamp`, `flow_rate`, `cumulative_volume`, `status`, `anomaly_flag`, `location` — con tipo, unidad y descripción | Christian Castro (IoT Jr.) | Sección Variables en `Fase_1_Analisis_S2_S3.docx` |
| Definir umbrales por zona | Umbrales normales y críticos para las 4 zonas funcionales (sanitarios, cocina, riego, cisterna) | Christian Castro + Todo el equipo | Sección Umbrales en `Fase_1_Analisis_S2_S3.docx` |
| Confirmar broker MQTT y topic | Broker: `broker.hivemq.com:1883` — Topic: `smarth2o/sensors` — Frecuencia: 5 seg por sensor | Azucena Castillo (Cloud Jr.) | Config en `influxdb_smarth2o_v3.py`: `MQTT_BROKER`, `MQTT_PORT`, `MQTT_TOPIC` |
| Registrar decisión D01 con alternativas descartadas | MySQL, MongoDB y PostgreSQL documentados como alternativas evaluadas y descartadas con razones técnicas | Azucena Castillo (Cloud Jr.) | `/Recursos/registro_de_decisiones.md` — D01 completa |
| Especificación de 6 KPIs del dashboard | 6 indicadores con tipo de panel Grafana, descripción y valor operativo para la DAIR | Mirna Jimenez (Dashboard Jr.) | `indicadores_dashboard.docx` — 6 KPIs con justificación |
| Inicio de estructuración del repo alerts | Estructura de carpetas y 4 reglas de detección iniciales definidas | Karen Caceres (Integración Jr.) | `smarth2o-esit/alerts/src/` — 7 módulos definidos |

---

## 3. Herramientas y servicios utilizados

| Herramienta / Servicio | Propósito | Observaciones |
|-----------------------|-----------|---------------|
| `broker.hivemq.com:1883` | Broker MQTT cloud gratuito para transmisión de datos del simulador | Topic: `smarth2o/sensors`. Frecuencia: 5 seg/sensor. Seleccionado sobre Mosquitto local |
| InfluxDB Cloud Free | BD de series temporales. Bucket: `water_telemetry`. Measurement: `consumo_agua` | Azucena configuró esquema: tags (`sensor_id`, `building`, `floor`, `zone`) + fields (`flow_rate`, `cumulative_volume`, `anomaly_flag`, `status`) |
| Python 3.11 + paho-mqtt | Simulador de sensores e integración MQTT → InfluxDB | Azucena inició `influxdb_smarth2o_v3.py` con validación de payload y queries Flux |
| Grafana Cloud Free | Dashboard de visualización — 6 KPIs especificados | Mirna definió los 6 indicadores con tipo de gráfico y valor para el cliente |
| Telegram Bot API | Canal de alertas automáticas | Karen definió estructura de 4 reglas R01–R04 con severidad y canal de notificación |
| GitHub `smarth2o-esit` | Control de versiones de código y documentación | Commits de Azucena en `database/`, Karen en `alerts/src/` |

---

## 4. Problemas detectados

| Problema | Impacto | Causa probable | Resolución / Evidencia |
|----------|---------|----------------|------------------------|
| Inconsistencia de IDs de sensores entre miembros | Medio — dificulta integración MQTT→InfluxDB | Diferentes miembros usaban formatos distintos (`AARD-` vs `SH2O-`) | ✅ Resuelto: IDs duales adoptados — `AARD-EDIF-A-SAN1` / `SH2O-ZA-001` |
| `status` definido inicialmente como TAG en InfluxDB | Alto — degradaría rendimiento por alta cardinalidad | Falta de claridad en el modelo de datos | ✅ Resuelto: Azucena definió `status` como FIELD (corrección técnica correcta) |
| Reglas de alerta R01–R04 con condiciones por afinar | Medio — umbrales iniciales provisionales | Umbrales definidos sin datos reales de simulación | 🔄 Mitigación: umbrales ajustables en `rules.py` sin modificar el motor |

---

## 5. Soluciones aplicadas

| Solución aplicada | Problema asociado | Responsable | Evidencia |
|------------------|-------------------|-------------|-----------|
| Adoptar IDs duales para sensores (`AARD-` y `SH2O-`) | Inconsistencia de nomenclatura entre miembros | Christian Castro + Azucena Castillo | Tabla de puntos de monitoreo con ambos formatos de ID |
| `status` como FIELD en InfluxDB (no TAG) | Error de modelo que degradaría rendimiento | Azucena Castillo | `influxdb_smarth2o_v3.py` — comentario explicando la decisión técnica |
| Payload JSON único como fuente de verdad | Desincronización entre simulador e ingestión | Todo el equipo | Estructura de payload en `Fase_1_Analisis_S2_S3.docx` y en `influxdb_smarth2o_v3.py` |

---

## 6. Estado del proyecto

| Área | Estado | Comentarios |
|------|--------|-------------|
| Puntos de monitoreo (5 sensores) | ✅ Completado | IDs, zonas, umbrales y observaciones documentados para los 5 puntos |
| Variables del sistema | ✅ Completado | 7 variables con tipo, unidad y descripción en Doc. de Análisis |
| Umbrales por zona | ✅ Completado | Sanitarios >20, Cocina >15, Riego >25, Cisterna >30 L/min (crítico) |
| Broker MQTT confirmado | ✅ Completado | `broker.hivemq.com:1883` — topic: `smarth2o/sensors` |
| Esquema InfluxDB | ✅ Completado | Measurement: `consumo_agua` — Tags: `sensor_id`, `building`, `floor`, `zone` — Fields: `flow_rate`, `cumulative_volume`, `anomaly_flag`, `status` |
| 6 KPIs del dashboard | ✅ Completado | Mirna: 6 indicadores con tipo de panel y valor operativo para la DAIR |
| Estructura repo alerts | 🔄 En progreso | 4 reglas R01–R04 definidas, 7 módulos `src/` planificados |

---

## 7. Plan de trabajo — Semana 3

| Tarea | Responsable | Fase | Entregable esperado |
|-------|-------------|------|---------------------|
| Implementar flujo completo Simulador→MQTT→InfluxDB | Azucena Castillo (Cloud Jr.) | Fase 1 – S3 | `influxdb_smarth2o_v3.py` funcional con 12 queries Flux |
| Diseñar arquitectura de extremo a extremo (diagrama) | Todo el equipo | Fase 1 – S3 | Diagrama en `/Diseno/` (`.drawio`) |
| Confirmar payload JSON definitivo | Christian Castro (IoT Jr.) | Fase 1 – S3 | Sección payload en `Fase_1_Analisis_S2_S3.docx` |
| Ampliar reglas de detección a R01–R05 | Karen Caceres (Integración Jr.) | Fase 1 – S3 | `src/rules.py` con 5 reglas + formato mensaje Telegram |
| Documentar Documento de Análisis v1 | Todo el equipo (David coord.) | Fase 1 – S3 | `Analisis/documento_analisis_fase1_v1.md` con arquitectura, modelo de datos y reglas |

---

## 8. Aportes individuales verificables

| Integrante | Rol | Tarea técnica específica | Entregable afectado | Evidencia verificable | Resultado |
|-----------|-----|--------------------------|---------------------|----------------------|-----------|
| Christian Castro | IoT Jr. | Documentar 5 puntos de monitoreo con IDs duales y zonas | Análisis S2 — tabla de sensores | `Fase_1_Analisis_S2_S3.docx` — Sección 2 | ✅ |
| Christian Castro | IoT Jr. | Definir 7 variables del sistema con tipo, unidad y descripción | Doc. de Análisis — sección variables | `Fase_1_Analisis_S2_S3.docx` — Sección 3 | ✅ |
| Christian Castro | IoT Jr. | Proponer umbrales por zona (sanitarios, cocina, riego, cisterna) | Doc. de Análisis — umbrales | `Fase_1_Analisis_S2_S3.docx` — Sección umbrales | ✅ |
| Azucena Castillo | Cloud Jr. | Confirmar broker MQTT: `broker.hivemq.com:1883` y topic `smarth2o/sensors` | `influxdb_smarth2o_v3.py` — config MQTT | `MQTT_BROKER`, `MQTT_PORT`, `MQTT_TOPIC` en el script | ✅ |
| Azucena Castillo | Cloud Jr. | Definir `status` como FIELD (no TAG) en el esquema InfluxDB | Modelo de datos InfluxDB | `influxdb_smarth2o_v3.py` — comentario línea modelo | ✅ |
| Azucena Castillo | Cloud Jr. | Documentar D01 con alternativas descartadas (MySQL, MongoDB, PostgreSQL) | `registro_de_decisiones.md` | `/Recursos/registro_de_decisiones.md` | ✅ |
| Mirna Jimenez | Dashboard Jr. | Especificar 6 KPIs del dashboard con tipo de panel y valor para DAIR | `indicadores_dashboard.docx` | 6 indicadores completos con justificación | ✅ |
| Karen Caceres | Integración Jr. | Definir 4 reglas de detección R01–R04 con condición y canal | `alerts/src/rules.py` | Documento de análisis de alertas — Semana 2 | ✅ |
| David Rondon | Product Owner | Coordinar sesiones de avance y actualizar Documento Preliminar v0.2 | Documento Preliminar | Documento Preliminar v0.2 con equipo real y D01 | ✅ |

---

## 9. Cumplimiento individual

| Integrante | Tarea asignada | Estado | Observación | Seguimiento |
|-----------|----------------|--------|-------------|-------------|
| Christian Castro | Documentar 5 puntos de monitoreo y variables del sistema | ✅ Completado | Entregado en `Fase_1_Analisis_S2_S3.docx` con tabla y umbrales | No |
| Azucena Castillo | Confirmar broker MQTT y esquema InfluxDB (`status` como FIELD) | ✅ Completado | Configuración en `influxdb_smarth2o_v3.py` y D01 documentada | No |
| Mirna Jimenez | Especificar 6 KPIs del dashboard Grafana | ✅ Completado | `indicadores_dashboard.docx` entregado con justificación por KPI | No |
| Karen Caceres | Definir reglas R01–R04 con condición y canal | ✅ Completado | Reglas documentadas con severidad y acción para cada caso | No |
| David Rondon | Coordinar avances y actualizar Documento Preliminar | ✅ Completado | Documento Preliminar v0.2 con equipo real y D01 registrada | No |

---

## 14. Registro de evidencias

| No. | Descripción de la evidencia | Tipo | Responsable | Enlace / Referencia |
|-----|----------------------------|------|-------------|---------------------|
| E07 | Tabla de 5 puntos de monitoreo con IDs duales | Documento Word | Christian Castro | `Fase_1_Analisis_S2_S3.docx` — Sección 2 |
| E08 | Tabla de variables del sistema (7 campos) | Documento Word | Christian Castro | `Fase_1_Analisis_S2_S3.docx` — Sección 3 (Variables) |
| E09 | Tabla de umbrales por zona (4 zonas) | Documento Word | Christian Castro | `Fase_1_Analisis_S2_S3.docx` — Sección umbrales |
| E10 | Registro de decisiones D01 con alternativas | Markdown | Azucena Castillo | `/Recursos/registro_de_decisiones.md` |
| E11 | Especificación de 6 KPIs del dashboard | Documento Word | Mirna Jimenez | `indicadores_dashboard.docx` |
| E12 | Reglas R01–R04 de detección iniciales | Documento / src | Karen Caceres | `smarth2o-esit/alerts` — análisis de alertas S2 |

---

*Bitácora generada al cierre de la Semana 2 · Fase 1 · SmartH2O*
*Próxima actualización: al cierre de la Semana 3*
