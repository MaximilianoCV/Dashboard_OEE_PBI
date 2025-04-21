# 📘 DAX_Explicado.md - Dashboard OEE Coflex

Este documento contiene la explicación detallada de las medidas DAX utilizadas en el modelo Power BI para Coflex, con foco en indicadores de OEE, calidad, eficiencia, disponibilidad, forecast, paros y mejora continua. Ideal para compartir con equipos técnicos o de negocio que consultan el dashboard.

---

## 🟢 Indicadores Principales de OEE

### ✳️ `2.0 OEE`
**¿Qué mide?** El desempeño total del equipo combinando disponibilidad, eficiencia y calidad. Es el KPI principal del tablero.
**Fórmula:** `[2.0 OEE] = [2.1 Disponibilidad] * [2.2 Eficiencia] * [2.3 Calidad]`

---

### ✳️ `2.1 Disponibilidad`
**¿Qué mide?** Qué tanto del tiempo programado realmente se utilizó para producir. Considera paros programados y no programados.
**Lógica:**
- Calcula el tiempo total disponible = tiempo de producción + paros con OT nula.
- Restamos los paros no programados.
- Se normaliza contra el tiempo programado.

---

### ✳️ `2.2 Eficiencia`
**¿Qué mide?** Qué tan eficiente fue la producción contra el estándar. Se topea en 1 (100%) para evitar valores irreales.
**Fórmula:** Piezas reales / piezas estándar.

---

### ✳️ `2.3 Calidad`
**¿Qué mide?** Porcentaje de piezas buenas sobre el total producido.
**Fórmula:** (Producción total - piezas defectuosas) / producción total.

---

## 🟢 Periodo Base (PB) - Benchmark histórico

### ✳️ `2.0 OEE PB`, `2.1 Disponibilidad PB`, `2.2 Eficiencia PB`, `2.3 Calidad PB`
**¿Qué miden?** El valor histórico (benchmark) de cada indicador clave. Se calculan tomando un promedio de los últimos 2 a 4 meses antes del mes actual.
**Usos:** Comparar la situación actual contra el pasado reciente.

---

### ✳️ `% Mejora` medidas (OEE, eficiencia, calidad, disponibilidad)
**¿Qué miden?** Qué tanto ha mejorado (o empeorado) el desempeño actual vs el periodo base.
**Fórmula típica:** `(Indicador Actual / Indicador PB) - 1`

---

## 🟢 Paros y Uso del Tiempo

### ✳️ `4.0 Horas Paro Almacén`
**¿Qué mide?** Total de horas perdidas por causas específicas de almacén (FALTA OT, SIN SURTIR, INCOMPLETO).

### ✳️ `% Paros por Almacén`
**¿Qué mide?** Qué porcentaje representan los paros de almacén respecto al total de paros no programados.

### ✳️ `4.1 Tiempo neto trabajado`
**¿Qué mide?** Horas efectivas de trabajo (producción real - paros).

### ✳️ `4.1 Tiempo invisible`
**¿Qué mide?** Tiempo total disponible menos tiempo trabajado y paros. Ayuda a detectar “vacíos” no documentados.

---

## 🟢 Forecast y Producción

### ✳️ `3.0 FC PROYECTADO`
**¿Qué mide?** El acumulado del forecast desde inicio hasta la fecha actual, usando una hoja de planeación externa.

### ✳️ `3.0 Meta vs Producción`
**¿Qué mide?** Qué tanto se ha cumplido la meta (forecast) con la producción real.

### ✳️ `3.0 Porcentaje Cumplimiento vs SO&P`
**¿Qué mide?** Relación entre producción real y forecast proyectado (Sales & Operations Planning).

### ✳️ `3.0 Faltante Req Diario`
**¿Qué mide?** Cuánto falta para alcanzar el requerimiento diario (forecast - producción).

---

## 🟢 Rolling Averages (Promedios móviles)

### ✳️ Producción acumulada, semanal, mensual y 3M
**¿Qué miden?** Tendencias de producción en diferentes cortes (día, semana, mes) para análisis de estabilidad y proyección.

---

## 🟢 Otros KPIs Relevantes

### ✳️ `5.0 TC Propuesta`
**¿Qué mide?** Tiempo de ciclo ideal con base en la producción actual y un factor de eficiencia del 1.08.

### ✳️ `2.4 Capacidad`
**¿Qué mide?** Capacidad de producción estimada a partir del OEE real y un OEE meta (0.73).

---

📌 *Este documento debe acompañar a `DAX_Utilizados.md` y servir como explicación ejecutiva del modelo Power BI. Puedes complementarlo con ejemplos visuales y enlaces al dashboard en producción.*
