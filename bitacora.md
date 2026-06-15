# Bitácora SmartH2O — Semana 1

**Proyecto:** SmartH2O – Sistema de Telemetría y Alertas para el Monitoreo Inteligente del Consumo de Agua
**Fase:** Fase 0 – Onboarding y planificación
**Semana:** 1 · Período: 03 – 08 de junio de 2026
**Tutor:** Ing. Carlos G. Rodriguez
**Ciclo:** IV

---

## Asistentes

| # | Nombre | Rol | Contacto |
|---|--------|-----|----------|
| 1 | Azucena del Carmen Castillo Zepeda | Ingeniero Cloud/Backend Jr. | castillozepeda0@gmail.com |
| 2 | Karen Elizabeth Caceres Alvarado | Ingeniero de Integración Jr. | eliza.2309alvarado@gmail.com |
| 3 | Mirna Yesenia Jimenez Moreno | Desarrollador Dashboard Jr. | mirnita.jimenez@gmail.com |
| 4 | David Ali Rondon Majano | Product Owner / Líder de Proyecto | Ali.majano87@gmail.com |
| 5 | Christian Ezequiel Castro Marquez | Ingeniero IoT Jr. | cmarq1987@gmail.com |

✅ Todos los integrantes presentes.

---

## Decisiones tomadas

### D01 · Stack tecnológico
**Decisión:** Se acuerda utilizar **MQTT** como canal de transmisión e **InfluxDB** como base de datos de series temporales.
**Alternativas descartadas:** API REST (FastAPI), MongoDB.
**Razón:** MQTT es el protocolo nativo para IoT con menor latencia; InfluxDB está optimizado para series temporales con políticas de retención nativas.
**Acordado por:** Todo el equipo en reunión grupal de kickoff.

### D02 · Estructura de repositorios
**Decisión:** Un repositorio por componente bajo la organización `smarth2o-esit`.
**Repositorios creados:**
- https://github.com/smarth2o-esit/simulator
- https://github.com/smarth2o-esit/Ingestion
- https://github.com/smarth2o-esit/database
- https://github.com/smarth2o-esit/dashboard
- https://github.com/smarth2o-esit/alerts
- https://github.com/smarth2o-esit/docs

### D03 · Dinámica de trabajo
**Decisión:** Sesiones 1-on-1 por rol para definir tareas y responsabilidades particulares de cada integrante, adicionales a la reunión grupal.
**Responsable de coordinar:** David Ali Rondon Majano (Product Owner).

### D04 · Cuentas cloud individuales
**Decisión:** Cada integrante crea sus propias cuentas en los servicios cloud de su rol para maximizar los créditos gratuitos disponibles.

---

## Actividades realizadas esta semana

| Actividad | Responsable | Estado | Evidencia |
|-----------|-------------|--------|-----------|
| Lectura y análisis del Documento Preliminar v0.1 y PDF del proyecto | Todos | ✅ Completado | Todos los integrantes analizaron sus tareas |
| Definición y asignación de roles del equipo | Product Owner (David) | ✅ Completado | Registrado en esta bitácora |
| Creación de repositorios en la organización smarth2o-esit | Product Owner (David) | ✅ Completado | 6 repositorios activos en GitHub |
| Selección del stack tecnológico (D01) | Todo el equipo | ✅ Completado | MQTT + InfluxDB acordado |
| Creación de cuentas en servicios cloud por rol | Cada responsable | ✅ Iniciado | Cuentas individuales creadas |
| Primera entrada de bitácora en /docs/bitacora.md | Product Owner (David) | ✅ Completado | Este archivo |
| Inicio de archivos .py para reglas del bot de Telegram | Karen (Integración Jr.) | ✅ Iniciado | Archivos base creados |
| Creación de cuenta Grafana Cloud | Mirna (Dashboard Jr.) | ✅ Completado | Cuenta activa |
| Creación de cuenta InfluxDB Cloud | Azucena (Cloud Jr.) | ✅ Completado | Cuenta activa |
| Creación de cuenta InfluxDB Cloud | Karen (Integración Jr.) | ✅ Completado | Cuenta activa |

---

## Acuerdos registrados

1. Realizar sesiones 1-on-1 por rol para delegar tareas específicas de cada fase.
2. Cada integrante crea cuentas individuales en los servicios de su rol para aprovechar los créditos gratuitos.
3. Continuar investigación individual sobre las tecnologías asignadas antes de la Semana 2.
4. Actualizar el repositorio de GitHub con los archivos y evidencias de cada rol.
5. Stack tecnológico definido: **MQTT + InfluxDB** (Decisión D01 registrada).

---

## Estado del proyecto al cierre de la semana

| Área | Estado | Notas |
|------|--------|-------|
| Documentación / punto de partida | ✅ Iniciado | Documento Preliminar v0.1 disponible en /docs/ |
| Repositorios GitHub | ✅ Iniciado | 6 repos creados en smarth2o-esit |
| Definición de roles | ✅ Completado | 5 integrantes con rol asignado |
| Stack tecnológico (D01) | ✅ Definido | MQTT + InfluxDB |
| Cuentas cloud | ✅ Iniciado | InfluxDB, Grafana, GitHub activos por rol |
| Plan de Recursos Tecnológicos | 🔄 En progreso | Completar con URLs y responsables |

---

## Próximos pasos — Semana 2 (Fase 1 · Análisis)

| Tarea | Responsable | Entregable |
|-------|-------------|------------|
| Definir puntos de monitoreo (mínimo 5 zonas) | Christian (IoT Jr.) | `/docs/analisis.md` — tabla de sensores |
| Definir variables a monitorear y unidades | Christian (IoT Jr.) | Sección de variables en Doc. de Análisis |
| Proponer umbrales de comportamiento normal y anómalo | Todo el equipo | Tabla de umbrales en Doc. de Análisis |
| Redactar Documento de Análisis v1 | Todo el equipo (coordinado por David) | `/docs/analisis.md` publicado en repo |
| Registrar decisión D01 formalmente en el Documento Preliminar | David (Product Owner) | D01 en `/docs/registro_de_decisiones.md` |
| Sesiones 1-on-1 con cada integrante | David (Product Owner) | Acuerdos individuales documentados |

---

## Problemas identificados y resolución

| Problema | Resolución | Pendiente |
|----------|------------|-----------|
| Stack tecnológico sin definir al inicio de la semana | ✅ Resuelto — D01: MQTT + InfluxDB acordado en reunión grupal | No |
| Roles sin asignar a personas específicas | ✅ Resuelto — Roles asignados a los 5 integrantes | No |
| Cuentas cloud no creadas | ✅ Parcialmente resuelto — InfluxDB y Grafana creadas | Faltan: EMQX/Mosquitto por configurar |
| Verificación de estructura del repositorio | 🔄 Pendiente — Product Owner debe confirmar estructura de carpetas | Sí |
| Definición de puntos de monitoreo (IoT) | 🔄 Pendiente — Tarea de la Semana 2 | Sí |

---

*Bitácora generada al cierre de la Semana 1 · Fase 0 · SmartH2O*
*Próxima actualización: al cierre de la Semana 2*
