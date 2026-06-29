# Documento de Análisis — Fase 1

**SmartH2O – Sistema de Telemetría y Alertas para el Monitoreo Inteligente del Consumo de Agua**
**Fase 1 – Análisis y Diseño · Semanas 2 y 3**
**Versión:** 2.0 — Avances individuales + Scope actualizado (InfluxDB confirmado)
**Fecha:** 28 de junio de 2026
**Estado:** ✅ Entregado – Fase 1 completada

---

## Información del proyecto

| Campo | Detalle |
|-------|---------|
| Institución | Escuela Superior de Innovación y Tecnología (ESIT) |
| Programa | Estancia Profesional – Línea IoT, Analítica y Sostenibilidad Digital |
| Cliente | Dirección de Administración de Infraestructura y Recursos (DAIR) – AARD |
| Tutor | Ing. Carlos G. Rodriguez |
| Ciclo | IV |

---

## Equipo

| # | Nombre | Rol | Entregable Fase 1 |
|---|--------|-----|-------------------|
| 1 | David Ali Rondon Majano | Product Owner / Líder | Coordinación, registros de decisiones, bitácora |
| 2 | Christian E. Castro Marquez | Ingeniero IoT Jr. | Análisis escenario hídrico, puntos de monitoreo, payload |
| 3 | Azucena del C. Castillo Zepeda | Ingeniero Cloud/Backend Jr. | `influxdb_smarth2o_v3.py` — integración MQTT + InfluxDB |
| 4 | Mirna Y. Jimenez Moreno | Desarrollador Dashboard Jr. | Especificación de 6 KPIs del dashboard en Grafana |
| 5 | Karen E. Caceres Alvarado | Ingeniero de Integración Jr. | Estructura repo alerts, 5 reglas de detección, formato Telegram |

---

## Historial de versiones

| Versión | Fecha | Descripción |
|---------|-------|-------------|
| 1.0 | 2026-06-28 | Documento inicial – Entregable Fase 1 |
| 1.1 | 2026-06-28 | Actualización scope: Decisión D06 (MongoDB explorado y descartado) |
| 2.0 | 2026-06-28 | Versión consolidada: avances individuales reales del equipo + scope con InfluxDB confirmado como BD definitiva |

---

> ### ✅ SCOPE CONFIRMADO — DECISIÓN D06 REVERTIDA
>
> **Base de datos: InfluxDB 2.x** (opción definitiva del proyecto)
>
> MongoDB fue evaluado como alternativa y descartado formalmente. Esta versión refleja el scope definitivo.
>
> Ver `/Recursos/registro_de_decisiones.md` — D06 ❌

---

## 1. Requerimientos del cliente

### 1.1 Descripción de la entidad receptora

La AARD gestiona instalaciones gubernamentales con consumo hídrico constante: servicios sanitarios, limpieza, cocinas, mantenimiento de áreas verdes. El sistema SmartH2O responde a la necesidad de visibilidad operativa en tiempo casi real sobre el consumo de agua, que actualmente se gestiona de forma reactiva con facturación mensual y revisiones manuales.

### 1.2 Requerimientos funcionales

| ID | Requerimiento | Prioridad |
|----|---------------|-----------|
| RF-01 | Monitorear caudal y consumo por punto de medición en tiempo casi real | Alta |
| RF-02 | Detectar comportamientos anómalos compatibles con fugas o uso ineficiente | Alta |
| RF-03 | Disparar alertas automáticas al superar umbrales definidos por zona | Alta |
| RF-04 | Visualizar historial de consumo por zona, edificio y franja horaria en Grafana | Alta |
| RF-05 | Analizar patrones históricos para apoyar mantenimiento preventivo | Media |
| RF-06 | Comparar consumo entre zonas, períodos e instalaciones | Media |
| RF-07 | Simular lecturas de 5 sensores IoT por punto de medición | Alta |
| RF-08 | Enviar notificaciones automáticas por Telegram y/o email ante anomalías | Alta |

---

## 2. Problemática actual

### 2.1 Situación y brechas identificadas

| Brecha | Estado actual | Estado deseado con SmartH2O |
|--------|--------------|------------------------------|
| Visibilidad | Factura mensual (30 días de retraso) | Dashboard Grafana con refresh cada 30 segundos |
| Detección de fugas | Detección manual post-facto | Detección automática < 15 min via regla R01 |
| Alertamiento | Sin alertas automáticas | Telegram Bot en < 2 min desde la anomalía |
| Historial | Sin base de datos de consumo | InfluxDB: 30 días raw + `aggregateWindow` 15 min |
| Cobertura por zona | Un medidor global por edificio | 5 sensores: sanitarios, cocina, riego, cisterna, piso 2 |

> **Riesgo cuantificado:** Una fuga de 0.5 L/min sin detectar durante 30 días = **21,600 litros desperdiciados**. SmartH2O detecta la anomalía en < 15 minutos desde el inicio del evento.

---

## 3. Justificación tecnológica

### 3.1 Stack confirmado — D01

| Componente | Tecnología | Razón de selección | Alternativas descartadas |
|------------|------------|--------------------|-----------------------------|
| Mensajería | MQTT 5.0 (HiveMQ / Mosquitto) | Protocolo IoT nativo; pub/sub; QoS 1; bajo consumo; reconexión automática | API REST — mayor latencia, no diseñada para IoT |
| Base de datos | InfluxDB 2.x | Optimizado para series temporales; Flux query; retención nativa; Data Explorer integrado; 12 queries implementadas | MySQL — relacional; MongoDB — no especializado (D06 ❌); PostgreSQL — requiere TimescaleDB |
| Dashboard | Grafana 10+ | Integración nativa con InfluxDB; 6 KPIs definidos; Alert Rules; variables de plantilla; exportación JSON | Kibana, custom React — mayor complejidad de setup |
| Alertas | Telegram Bot API + SMTP | Gratuita; latencia < 1 seg; 5 reglas con severidad; debounce anti-spam implementado | Solo email — menor inmediatez |
| Simulador | Python 3.11 + paho-mqtt | Broker HiveMQ (`broker.hivemq.com:1883`); 5 sensores en paralelo; payload JSON validado | Node.js — menor integración con stack científico Python |
| Infraestructura | Docker + Docker Compose | Reproducibilidad; un comando levanta el stack; aislamiento de servicios | — |

### 3.2 Tecnologías emergentes aplicadas

**Series temporales con InfluxDB 2.x y Flux**

Azucena implementó 12 queries Flux en `influxdb_smarth2o_v3.py`, incluyendo:
- Q1: última lectura por sensor
- Q7: promedio móvil cada 15 minutos con `aggregateWindow`
- Q11: consumo total por piso usando el TAG `floor`
- Q12: tasa de anomalías por sensor con `join()` de tablas

El script incluye health check automático (`ping()`), creación dinámica del bucket con política de retención de 30 días, y reconexión MQTT automática con back-off exponencial (1s–60s).

**Detección de anomalías por capas — 5 reglas**

Karen expandió el modelo de detección a 5 reglas (R01–R05), incorporando R03 que evalúa `cumulative_volume` superando el 150% del promedio semanal. Las reglas se definen en `src/rules.py` y son configurables por zona sin modificar el motor `detector.py`.

**Dashboard orientado a KPIs con valor para el cliente**

Mirna definió 6 indicadores en el documento de especificación, cada uno justificado por su valor operativo para la DAIR. El Indicador 4 (caudal promedio en horario no laboral) es el más crítico para fugas nocturnas. El Indicador 6 (SLA de telemetría) actúa como semáforo de disponibilidad de sensores.

---

## 4. Propuesta arquitectónica

### 4.1 Pipeline del sistema — confirmado

```
Simulador IoT  →  MQTT (HiveMQ / Mosquitto)  →  Ingestión (influxdb_smarth2o_v3.py)  →  InfluxDB 2.x  →  Grafana  →  Telegram Bot
```

### 4.2 Capas del sistema

| Capa | Componente | Tecnología | Responsable | Estado |
|------|------------|------------|-------------|--------|
| 1 | Simulador de sensores | Python 3.11 + paho-mqtt + HiveMQ | Christian Castro | ✅ Implementado |
| 2 | Broker MQTT | `broker.hivemq.com:1883` (cloud free) | Azucena Castillo | ✅ Configurado |
| 3 | Ingestión + BD | `influxdb_smarth2o_v3.py` | Azucena Castillo | ✅ Implementado |
| 4 | Base de datos | InfluxDB 2.x — bucket: `water_telemetry` | Azucena Castillo | ✅ Esquema definido |
| 5 | Dashboard | Grafana — 6 KPIs especificados | Mirna Jimenez | 🔄 En diseño |
| 6 | Alertas | Telegram Bot + SMTP — 5 reglas | Karen Caceres | ✅ Estructura lista |

### 4.3 Diagramas Draw.io entregados

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| `01_arquitectura_sistema.drawio` | 5 capas swimlane con todos los componentes, responsables y flujo | ✅ Entregado |
| `02_flujo_de_datos.drawio` | 12 pasos del mensaje: sensor → MQTT → validación → InfluxDB → Grafana → Telegram | ✅ Entregado |
| `03_responsabilidades_por_rol.drawio` | Matriz de responsabilidades con estado de tareas por fase | ✅ Entregado |

> Los archivos `.drawio` se encuentran en la raíz del repositorio `smarth2o-esit/docs` y pueden abrirse con [app.diagrams.net](https://app.diagrams.net).

---

## 5. Modelo de datos y pipeline

### 5.1 Modelo de datos InfluxDB — confirmado

- **Measurement:** `consumo_agua`
- **Bucket:** `water_telemetry`
- **Org:** `smarth2o`
- **Retención:** 30 días

| Elemento | Campo | Tipo | Ejemplo | Descripción |
|----------|-------|------|---------|-------------|
| TAG | `sensor_id` | String | `AARD-EDIF-A-SAN1` | ID único. Indexado para filtros rápidos. Cardinalidad baja. |
| TAG | `building` | String | `A` | Edificio. Permite agrupar por inmueble en Grafana. |
| TAG | `floor` | String | `1` | Piso (siempre string para evitar mezcla int/str en series). |
| TAG | `zone` | String | `Sanitarios Piso 1` | Zona funcional. Viene de `location.zone` del payload. |
| FIELD | `flow_rate` | Float | `12.5` | Caudal en L/min. Rango válido: 0.0 – 50.0. |
| FIELD | `cumulative_volume` | Float | `120.3` | Litros acumulados del día. Reset a medianoche. |
| FIELD | `anomaly_flag` | Boolean | `false` | `true` si el sensor detecta anomalía. FIELD, no TAG. |
| FIELD | `status` | String | `normal` | `normal` \| `warning` \| `critical`. FIELD (varía por lectura). |
| TIMESTAMP | `_time` | UTC ns | `2026-06-14T10:00:00Z` | ISO 8601 convertido a nanosegundos. `WritePrecision.NANOSECONDS`. |

> **Nota de diseño:** `status` es **FIELD**, no TAG. Esta decisión es correcta: `status` varía en cada lectura (alta cardinalidad si fuera TAG), lo que degradaría el rendimiento de InfluxDB.

### 5.2 Payload JSON del simulador — confirmado

```json
{
  "sensor_id":         "AARD-EDIF-A-SAN1",
  "timestamp":         "2026-06-14T10:00:00Z",
  "flow_rate":         12.5,
  "cumulative_volume": 120.3,
  "status":            "normal",
  "anomaly_flag":      false,
  "location": {
    "building": "A",
    "floor":    1,
    "zone":     "Sanitarios Piso 1"
  }
}
```

- **ID de sensor real en la implementación:** `AARD-EDIF-A-SAN1` (Zona A)
- **Topic MQTT:** `smarth2o/sensors`

### 5.3 Puntos de monitoreo — confirmados

| ID Sensor | Zona | Punto de medición | Datos clave | Umbral normal | Umbral crítico | Observación |
|-----------|------|-------------------|-------------|---------------|----------------|-------------|
| `AARD-EDIF-A-SAN1` / `SH2O-ZA-001` | Zona A | Sanitarios Piso 1 | `flow_rate`, `cumulative_volume`, `status` | 0–15 L/min | >20 L/min | Alto consumo en horario laboral |
| `AARD-EDIF-A-COCINA` / `SH2O-ZB-001` | Zona B | Cocina / Comedor | `flow_rate`, `cumulative_volume`, `status` | 0–10 L/min | >15 L/min | Picos al mediodía |
| `AARD-EDIF-A-RIEGO` / `SH2O-ZC-001` | Zona C | Áreas verdes / Riego | `flow_rate`, `cumulative_volume`, `status` | 0–20 L/min | >25 L/min | Detectar consumo fuera de horario de riego |
| `AARD-EDIF-A-CIST` / `SH2O-ZD-001` | Zona D | Cuarto mantenimiento / Cisterna | `flow_rate`, `cumulative_volume`, `status`, `anomaly_flag` | 5–25 L/min | >30 L/min | Punto crítico para fugas y rebalses |
| `AARD-EDIF-A-SAN2` / `SH2O-ZE-001` | Zona E | Sanitarios Piso 2 | `flow_rate`, `cumulative_volume`, `status` | 0–15 L/min | >20 L/min | Bajo consumo esperado en horario nocturno |

### 5.4 Variables del sistema

| Variable | Tipo | Unidad | Descripción |
|----------|------|--------|-------------|
| `sensor_id` | String | N/A | Identificador único del sensor. Distingue cada punto de medición. |
| `timestamp` | String | ISO 8601 | Fecha y hora de la medición en UTC. |
| `flow_rate` | Float | L/min | Caudal instantáneo. Campo principal para detección de fugas. |
| `cumulative_volume` | Float | Litros | Consumo acumulado del día. Se resetea a medianoche. |
| `status` | String | N/A | Estado calculado: `normal` \| `warning` \| `critical` \| `offline`. |
| `anomaly_flag` | Boolean | N/A | `true` si existe anomalía detectada localmente o en el motor. |
| `location` | Object | N/A | Subdocumento: `{ building, floor, zone }`. Metadatos del punto. |

### 5.5 Reglas de detección — 5 reglas (Karen Caceres)

| Regla | Condición | Severidad | Canal | Descripción |
|-------|-----------|-----------|-------|-------------|
| R01 | `flow_rate` > 20 L/min durante más de 10 minutos | 🔴 Critical | Telegram + Email | Posible fuga activa o consumo excesivo sostenido |
| R02 | `flow_rate` > 5 L/min fuera de horario laboral (20h–06h) | 🟠 Warning | Telegram | Consumo fuera del horario esperado — posible fuga nocturna |
| R03 | `cumulative_volume` del día supera el 150% del promedio semanal | 🟠 Warning | Email | Consumo acumulado anormal para la zona — tendencia de sobreuso |
| R04 | Ausencia de datos de un sensor por más de 5 minutos | 🔵 Info | Telegram | Posible desconexión o falla del sensor — revisar conectividad |
| R05 | `anomaly_flag = true` recibido directamente del simulador | 🔴 Critical | Telegram + Email | Anomalía inyectada en modo prueba o detectada en sensor físico |

> Los umbrales son configurables en `src/rules.py` sin necesidad de modificar el motor `detector.py`.

---

## 6. Avances individuales — Fase 1

### David Ali Rondon Majano — Product Owner / Líder de Proyecto
*Repo: `smarth2o-esit/docs`*

- Creación de la organización `smarth2o-esit` y 6 repositorios en GitHub
- Documento Preliminar de Solución v0.1 → v0.2 → v1.1 (con D06)
- READMEs de todos los repositorios (simulator, ingestion, database, dashboard, alerts, docs)
- Bitácora Semana 1 en `/bitacoras/` — 5 asistentes, 4 decisiones registradas
- Documentos de delegación 1-on-1 por rol (20 tareas por integrante en 6 fases)
- Guía de GitHub paso a paso para Windows (6 capítulos, tarjeta de referencia rápida)
- Registro de decisiones D01–D05 y exploración/descarte formal de D06 (MongoDB)
- Coordinación de sesiones 1-on-1 con cada integrante del equipo
- 3 diagramas Draw.io (arquitectura, flujo de datos, responsabilidades)
- Documento de Análisis v2.0 — Entregable Fase 1

### Christian E. Castro Marquez — Ingeniero IoT Jr.
*Repo: `smarth2o-esit/simulator`*

- Análisis del escenario hídrico — Semana 2: documento de puntos de monitoreo
- 5 puntos de monitoreo definidos con IDs duales: `AARD-EDIF-A-SAN1` / `SH2O-ZA-001`
- Tabla de variables del sistema: 7 campos con tipo, unidad y descripción
- Umbrales por zona definidos: sanitarios (>20 L/min), cocina (>15), riego (>25), cisterna (>30)
- Estructura del payload JSON confirmada — Semana 3
- Flujo de datos documentado: Simulador → MQTT → Ingesta → InfluxDB → Grafana → Alertas
- Topic MQTT real: `smarth2o/sensors` — Broker: `broker.hivemq.com:1883`

### Azucena del C. Castillo Zepeda — Ingeniero Cloud/Backend Jr.
*Repo: `smarth2o-esit/ingestion` + `database`*

- `influxdb_smarth2o_v3.py` — script completo de integración MQTT + InfluxDB (600+ líneas)
- 12 queries Flux implementadas: última lectura, promedio 15min, tasa anomalías, consumo por piso
- Validación de payload con schema guard — 8 campos requeridos verificados
- Conversión de timestamp ISO 8601 → nanosegundos UTC para `WritePrecision.NANOSECONDS`
- Health check automático de conexión a InfluxDB (`ping()` antes de operar)
- Creación dinámica de bucket con política de retención de 30 días
- Reconexión MQTT automática con back-off exponencial (1s min – 60s max)
- Carga de datos de ejemplo: 5 sensores con timestamps escalonados para pruebas
- Measurement: `consumo_agua` | `status` como FIELD (no TAG) — decisión técnica correcta
- Broker externo configurado: `broker.hivemq.com:1883` (cloud free, sin servidor local)
- Registro de decisiones D01 documentado con alternativas descartadas

### Mirna Y. Jimenez Moreno — Desarrollador Dashboard Jr.
*Repo: `smarth2o-esit/dashboard`*

- Documento de especificación de 6 KPIs del dashboard — orientado a valor para el cliente DAIR
- **KPI 1:** Consumo Acumulado por Punto — Bar Chart / Stat — impacto financiero por sede
- **KPI 2:** Caudal Instantáneo y Tendencia — Time Series — detección visual de picos atípicos
- **KPI 3:** Registro de Eventos Anómalos — State Timeline / Alert List — evidencia para mantenimiento
- **KPI 4:** Caudal Promedio en Horario No Laboral — Gauge/Stat — indicador crítico de fugas nocturnas
- **KPI 5:** Pico Máximo por Franja Horaria — Histogram — identificar comportamientos ineficientes
- **KPI 6:** Disponibilidad de Sensores (SLA de Telemetría) — Status History / semáforo verde-rojo
- Cada KPI justificado con valor operativo directo para la DAIR

### Karen E. Caceres Alvarado — Ingeniero de Integración Jr.
*Repo: `smarth2o-esit/alerts`*

- Estructura completa del repositorio `alerts/` documentada en README: 7 módulos `src/`
- `src/detector.py` — Motor de detección: evalúa reglas contra la BD
- `src/rules.py` — Definición de 5 reglas con umbrales configurables por zona/sensor
- `src/notifier.py` — Envío de alertas Telegram + email
- `src/telegram_bot.py` — Integración con Telegram Bot API
- `src/email_sender.py` — Envío vía SMTP
- `src/debounce.py` — Control anti-spam (cooldown configurable por sensor)
- 5 reglas de detección (R01–R05) con severidad, canal y condición definidas
- R03 nueva: `cumulative_volume` supera 150% del promedio semanal → Warning via Email
- Formato de mensaje Telegram documentado con sensor real `AARD-A-SAN-P1`, caudal 24.3 L/min

---

## 7. Resumen de entregables — Fase 1

| Entregable | Responsable | Semana | Estado | Ubicación |
|-----------|-------------|--------|--------|-----------|
| Organización GitHub + 6 repos | David Rondon | S1 | ✅ | `github.com/smarth2o-esit` |
| Documento Preliminar v0.2 | Todo el equipo | S1 | ✅ | `smarth2o-esit/docs` |
| Bitácora Semana 1 | David Rondon | S1 | ✅ | `/bitacoras/Bitacora_Fase0.md` |
| Registro de decisiones D01–D05 | David Rondon | S1 | ✅ | `/Recursos/registro_de_decisiones.md` |
| Análisis escenario hídrico — puntos de monitoreo | Christian Castro | S2 | ✅ | `smarth2o-esit/docs` |
| Variables del sistema + payload JSON | Christian Castro | S2–S3 | ✅ | Semana2 doc + README simulator |
| Umbrales por zona (5 sensores) | Christian Castro | S2 | ✅ | Documento análisis semana 2 |
| `influxdb_smarth2o_v3.py` (600+ líneas) | Azucena Castillo | S2–S3 | ✅ | `smarth2o-esit/database` |
| 12 queries Flux + `schema.md` | Azucena Castillo | S3 | ✅ | `database/influxdb/queries/` |
| Registro de decisiones D01 con alternativas | Azucena Castillo | S2 | ✅ | `/Recursos/registro_de_decisiones.md` |
| Especificación de 6 KPIs del dashboard | Mirna Jimenez | S3 | ✅ | `indicadores_dashboard.docx` |
| Estructura repo `alerts/` + 7 módulos | Karen Caceres | S3 | ✅ | `smarth2o-esit/alerts/README.md` |
| 5 reglas detección R01–R05 | Karen Caceres | S3 | ✅ | `smarth2o-esit/alerts/src/rules.py` |
| Formato mensaje Telegram con caso real | Karen Caceres | S3 | ✅ | `smarth2o-esit/alerts/README.md` |
| 3 diagramas Draw.io | Todo el equipo | S3 | ✅ | `smarth2o-esit/docs/*.drawio` |
| D06 descartada — InfluxDB confirmado | David + Azucena | S3 | ✅ | `/Recursos/registro_de_decisiones.md` |
| Bitácora Semana 2 | David Rondon | S2 | ✅ | `/bitacoras/bitacora_semana02.md` |
| Bitácora Semana 3 | David Rondon | S3 | ✅ | `/bitacoras/bitacora_semana03.md` |
| Este Documento de Análisis v2.0 | Todo el equipo | S3 | ✅ | `/Analisis/documento_analisis_fase1_v2.md` |

---

> **Logro de la Fase 1:** Todos los entregables fueron completados con aportes verificables de los 5 integrantes. El código de Azucena (`influxdb_smarth2o_v3.py`) representa el avance técnico más significativo: 600+ líneas de integración MQTT + InfluxDB lista para la Fase 2.

---

*SmartH2O · Documento de Análisis v2.0 · Fase 1 · 28 de junio de 2026*
*ESIT – Estancia Profesional · Tutor: Ing. Carlos G. Rodriguez · Ciclo IV*
