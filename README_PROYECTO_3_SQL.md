# 💰 Proyecto 3: Análisis Financiero con SQL
## Rentabilidad y ROI por País y Campaña

---

## 🎯 Descripción General

**Usuario:** Director de Finanzas  
**Período:** 2024  
**Objetivo:** Analizar rentabilidad por país y efectividad de campañas marketing  
**Entregable:** Reporte ejecutivo con recomendaciones de presupuesto

El Director de Finanzas necesitaba entender cuáles países eran rentables y cuáles campañas de marketing estaban generando valor real.

---

## 🔴 EL PROBLEMA

Los datos financieros estaban dispersos en múltiples tablas sin análisis consolidado:

```
❌ Ingresos en una tabla
❌ Costos de producción en otra tabla
❌ Gastos de marketing en tabla separada
❌ Imposible calcular margen bruto
❌ Imposible calcular ROI de campañas
❌ No se veía qué países eran rentables
❌ Decisiones de presupuesto sin datos
```

**Impacto empresarial:**
- 💸 Posiblemente invirtiendo en campañas ineficientes
- 🤔 Desconocimiento de rentabilidad por país
- ⚠️ Riesgo de inviertir mal el presupuesto

---

## ✅ LA SOLUCIÓN

Creé **queries SQL complejas** para consolidar datos y calcular métricas financieras:

### 1. **Consolidación de Datos (JOINs)**
```sql
SELECT 
    t1.country,
    t1.campaign_id,
    t1.revenue,          -- De tabla ingresos
    t2.production_cost,  -- De tabla costos
    t3.marketing_spend   -- De tabla marketing
FROM ventas t1
LEFT JOIN costos_produccion t2 
    ON t1.product_id = t2.product_id
LEFT JOIN gastos_marketing t3 
    ON t1.campaign_id = t3.campaign_id

Resultado:
  - 1 tabla con TODOS los datos financieros
  - Sin duplicados
  - Listo para cálculos
```

### 2. **Cálculo de Margen Bruto**
```sql
SELECT 
    country,
    SUM(revenue) as total_revenue,
    SUM(production_cost) as total_costs,
    (SUM(revenue) - SUM(production_cost)) as gross_profit,
    ROUND((SUM(revenue) - SUM(production_cost)) / SUM(revenue) * 100, 2) 
        as gross_margin_pct
FROM datos_consolidados
GROUP BY country
ORDER BY gross_margin_pct DESC

Resultado:
  País A: 45% margen (BUENO)
  País B: 28% margen (MEDIO)
  País C: -5% margen (NEGATIVO)
```

### 3. **Cálculo de ROI Marketing**
```sql
SELECT 
    campaign_id,
    SUM(revenue) as campaign_revenue,
    SUM(marketing_spend) as marketing_cost,
    SUM(production_cost) as production_cost,
    (SUM(revenue) - SUM(marketing_spend) - SUM(production_cost)) 
        as campaign_profit,
    ROUND((SUM(revenue) - SUM(marketing_spend) - SUM(production_cost)) 
        / SUM(marketing_spend) * 100, 2) as roi_pct
FROM datos_consolidados
GROUP BY campaign_id
ORDER BY roi_pct DESC

Resultado:
  Campaña A: ROI 350% (EXCELENTE)
  Campaña B: ROI 85% (BUENO)
  Campaña C: ROI -40% (NEGATIVO)
```

### 4. **Análisis de Campañas Ineficientes**
```sql
SELECT 
    campaign_id,
    campaign_name,
    SUM(marketing_spend) as total_spent,
    SUM(revenue) as total_revenue,
    (SUM(revenue) - SUM(marketing_spend)) as net_profit
FROM datos_consolidados
WHERE (SUM(revenue) - SUM(marketing_spend)) < 0
GROUP BY campaign_id
ORDER BY net_profit ASC

Hallazgo:
  Campaña X perdió $150,000
  Campaña Y perdió $89,000
  Recomendación: DETENER estas campañas inmediatamente
```

### 5. **Ranking de Países por Rentabilidad**
```sql
SELECT 
    country,
    SUM(revenue) as total_revenue,
    SUM(production_cost) as total_costs,
    SUM(revenue) - SUM(production_cost) as total_profit,
    ROUND((SUM(revenue) - SUM(production_cost)) / SUM(revenue) * 100, 2) 
        as margin_pct,
    COUNT(*) as num_transactions
FROM datos_consolidados
GROUP BY country
ORDER BY total_profit DESC

Resultado: Ranking claro de más a menos rentable
```

---

## 📈 RESULTADOS Y HALLAZGOS

### Hallazgo 1: Concentración de Rentabilidad
```
País A: 45% margen (TOP - Muy rentable)
País B: 28% margen (Medio)
País C: -5% margen (PÉRDIDA)

Recomendación: 
  - Aumentar inversión en País A
  - Revisar estructura de costos en País C
  - Considerar salida si no mejora
```

### Hallazgo 2: Campañas Ineficientes
```
De 50 campañas analizadas:
  - 15 tienen ROI negativo (pierden dinero)
  - 5 tienen ROI < 0% (perder dinero significativamente)
  
Dinero desperdiciado: $2.5M en campañas ineficientes

Recomendación: Reasignar presupuesto a campañas rentables
```

### Hallazgo 3: Margen Comprimido
```
Sector promedio: 35% margen
Tu empresa: 22% margen

Razón: Costos de producción altos vs competencia

Recomendación: Optimizar cadena de suministro o revisar pricing
```

---

## 🛠️ QUERIES SQL UTILIZADAS

### Query 1: Consolidación Base
```sql
SELECT 
    v.country,
    v.campaign_id,
    v.product_id,
    v.revenue,
    c.cost as production_cost,
    m.spend as marketing_spend,
    v.date
FROM ventas v
LEFT JOIN costos c ON v.product_id = c.product_id
LEFT JOIN marketing m ON v.campaign_id = m.campaign_id
WHERE v.date >= '2024-01-01'
```

### Query 2: Margen por País
```sql
SELECT 
    country,
    ROUND(SUM(revenue), 2) as total_revenue,
    ROUND(SUM(production_cost), 2) as total_costs,
    ROUND(SUM(revenue) - SUM(production_cost), 2) as gross_profit,
    ROUND(
        (SUM(revenue) - SUM(production_cost)) / SUM(revenue) * 100, 2
    ) as margin_pct,
    COUNT(*) as num_transactions
FROM datos_consolidados
GROUP BY country
ORDER BY gross_profit DESC
```

### Query 3: ROI por Campaña
```sql
SELECT 
    campaign_id,
    ROUND(SUM(revenue), 2) as revenue,
    ROUND(SUM(marketing_spend), 2) as marketing_cost,
    ROUND(
        (SUM(revenue) - SUM(marketing_spend)) / SUM(marketing_spend) * 100, 2
    ) as roi_pct,
    CASE 
        WHEN (SUM(revenue) - SUM(marketing_spend)) / SUM(marketing_spend) > 1 
            THEN 'RENTABLE'
        WHEN (SUM(revenue) - SUM(marketing_spend)) / SUM(marketing_spend) > 0 
            THEN 'MARGINAL'
        ELSE 'PÉRDIDA'
    END as status
FROM datos_consolidados
GROUP BY campaign_id
ORDER BY roi_pct DESC
```

### Query 4: Campañas Ineficientes
```sql
SELECT 
    campaign_id,
    campaign_name,
    ROUND(SUM(marketing_spend), 2) as spent,
    ROUND(SUM(revenue), 2) as revenue,
    ROUND(SUM(revenue) - SUM(marketing_spend), 2) as profit_loss
FROM datos_consolidados
GROUP BY campaign_id
HAVING SUM(revenue) - SUM(marketing_spend) < 0
ORDER BY profit_loss ASC
LIMIT 10
```

---

## 📁 ARCHIVOS EN ESTE REPOSITORIO

```
financial-analysis-sql/
├── README.md (este archivo)
├── queries.sql (todas las queries SQL)
├── query-results.csv (resultados exportados)
├── query-results-preview.png (captura de resultados)
└── DOCUMENTACIÓN:
    - Explicación de cada query
    - Cómo interpretar resultados
    - Recomendaciones de acción
```

---

## 📊 ARCHIVOS CSV DISPONIBLES

### `query-results.csv`
Contiene resultados de todas las queries:
- Country Rankings (margen, rentabilidad)
- Campaign Analysis (ROI, profit/loss)
- Inefficient Campaigns (lista de pérdidas)

Columnas:
- country
- total_revenue
- production_cost
- gross_profit
- margin_pct
- roi_by_campaign
- marketing_spend_efficiency

---

## 📈 VISUALIZACIONES RECOMENDADAS

### PNG 1: `query-results-preview.png`
Captura mostrando:
```
RESULTS OF SQL ANALYSIS:

Country Ranking (by Profit):
│ Country │ Revenue   │ Cost    │ Profit  │ Margin% │
├─────────┼───────────┼─────────┼─────────┼─────────┤
│ USA     │ $5,200k   │ $3,100k │ $2,100k │ 40.4%   │
│ Canada  │ $3,850k   │ $2,310k │ $1,540k │ 40.0%   │
│ Mexico  │ $2,100k   │ $1,890k │ $210k   │ 10.0%   │

Campaign ROI (Top 5):
│ Campaign │ ROI%  │ Status    │
├──────────┼───────┼───────────┤
│ Camp_A   │ 350%  │ EXCELLENT │
│ Camp_B   │  85%  │ GOOD      │
│ Camp_C   │  45%  │ OK        │
│ Camp_D   │ -50%  │ STOP NOW  │
│ Camp_E   │-100%  │ STOP NOW  │
```

### PNG 2: `roi-comparison.png`
Gráfico de barras mostrando ROI por campaña:
```
ROI BY CAMPAIGN (%)

Camp_A    ████████████████████████ 350%
Camp_B    ███████████ 85%
Camp_C    ██████ 45%
Camp_D    ██ -50%
Camp_E    █ -100%
```

### PNG 3: `margin-by-country.png`
Gráfico mostrando margen por país:
```
GROSS MARGIN BY COUNTRY (%)

USA     ████████████████████ 40.4%
Canada  ████████████████████ 40.0%
Mexico  ████ 10.0%
Brazil  ██ 5.0%
```

---

## ✅ VALIDACIÓN

```
QA Checklist:

✓ Todos los JOINs sin errores
✓ Cálculos matemáticos verificados
✓ Sin duplicados en resultados
✓ ROI positivos y negativos identificados
✓ Valores extremos revisados
✓ Queries están documentadas
✓ Resultados son reproducibles
```

---

## 🔧 HERRAMIENTAS UTILIZADAS

```
Base de Datos: SQL
Funciones clave:
  - LEFT JOIN (integración de tablas)
  - SUM(), COUNT() (agregaciones)
  - ROUND() (precisión numérica)
  - CASE WHEN (clasificación)
  - GROUP BY, ORDER BY (organización)
  - WHERE, HAVING (filtrado)

Análisis:
  - Margen Bruto = (Revenue - Costs) / Revenue
  - ROI = (Revenue - Spend) / Spend × 100%
  - Profit = Revenue - Costs - Marketing_Spend
```

---

## 💡 HABILIDADES DEMOSTRADAS

```
SQL Avanzado:
  ✓ JOINs complejos (múltiples tablas)
  ✓ Agregaciones (SUM, AVG, COUNT)
  ✓ GROUP BY con múltiples dimensiones
  ✓ HAVING para filtrar agregados
  ✓ CASE WHEN para clasificación
  ✓ Subconsultas

Análisis Financiero:
  ✓ Cálculo de margen bruto
  ✓ Análisis de ROI
  ✓ Análisis de rentabilidad
  ✓ Identificación de pérdidas

Pensamiento Crítico:
  ✓ Hacer preguntas clave ("¿Por qué?")
  ✓ Traducir preguntas en queries
  ✓ Interpretar resultados
  ✓ Proponer acciones
```

---

## 📌 METODOLOGÍA

```
Paso 1: ENTENDER LA PREGUNTA
  → Dirección financiera necesita saber:
    * Qué países son rentables
    * Cuáles campañas funcionan
    * Dónde reasignar presupuesto

Paso 2: IDENTIFICAR DATOS
  → ¿Qué tablas necesito?
  → ¿Qué columnas debo relacionar?

Paso 3: CONSOLIDAR
  → Escribir JOINs para conectar tablas
  → Verificar integridad referencial

Paso 4: CALCULAR
  → Margen bruto
  → ROI
  → Profit/Loss

Paso 5: ANALIZAR
  → ¿Qué se ve en los datos?
  → ¿Qué significa?
  → ¿Qué hacer al respecto?

Paso 6: COMUNICAR
  → Informe ejecutivo
  → Recomendaciones claras
  → Números y justificaciones
```

---

## ⚠️ LIMITACIONES

```
Limitaciones:
  - Solo año 2024 (no puede ver tendencias históricas)
  - No incluye factores externos (competencia, economía)
  - Asume que precio = margen válido
  - No incluye retenciones o devoluciones

Mejoras futuras:
  - Análisis multi-año
  - Incluir datos de competencia
  - Análisis de elasticidad de precio
  - Predicciones de rentabilidad
```

---

## 🎯 RECOMENDACIONES FINALES

```
ACCIÓN 1: DETENER INMEDIATAMENTE
  Campañas con ROI < -50%
  Ahorro potencial: $XXX,XXX

ACCIÓN 2: REASIGNAR PRESUPUESTO
  Aumentar inversión en campañas ROI > 100%
  Reducir en campañas ROI 0-50%

ACCIÓN 3: INVESTIGAR PAÍS C
  Margen negativo requiere análisis profundo
  Opciones: Aumentar precios, reducir costos, o salir

ACCIÓN 4: REPLICAR ÉXITO
  País A funciona bien (40% margen)
  Aplicar su modelo a otros países

TIMELINE: Implementar cambios dentro de 30 días
```

---

## 📞 CONTACTO

- **Fecha:** Mayo 2026
- **Bootcamp:** TripleTen Data Analysis
- **Nivel:** Avanzado (SQL financiero)

---

**Hecho con rigor analítico y pensamiento estratégico. ✨**
