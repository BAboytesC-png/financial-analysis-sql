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
    v.numero_pedido, 
    v.clave_producto, 
    p.nombre_producto, 
    pc.clave_categoria, 
    -- Reemplazo de nulos (COALESCE devuelve el primer valor no nulo, en este caso 0)
    COALESCE(p.precio_producto, 0) AS precio_producto, 
    COALESCE(v.cantidad_pedido, 0) AS cantidad_pedido, 
    COALESCE(p.costo_producto, 0) AS costo_producto,
    t.pais, 
    t.continente, 
    v.clave_territorio
FROM ventas_2017 AS v
-- Unión 1: Ventas con Productos (Clave Celeste)
INNER JOIN productos AS p 
    ON v.clave_producto = p.clave_producto
-- Unión 2: Productos con Categorías (Clave Verde)
INNER JOIN productos_categorias AS pc 
    ON p.clave_subcategoria = pc.clave_subcategoria
-- Unión 3: Ventas con Territorios (Clave Amarilla)
INNER JOIN territorios AS t 
    ON v.clave_territorio = t.clave_territorio;

Resultado:
  - 1 tabla con TODOS los datos financieros
  - Sin duplicados
  - Listo para cálculos
```

### 2. **Cálculo de Margen Bruto**
```sql
SELECT 
    v.numero_pedido, 
    v.clave_producto, 
    p.nombre_producto, 
    pc.clave_categoria, 
    COALESCE(p.precio_producto, 0) AS precio_producto, 
    COALESCE(v.cantidad_pedido, 0) AS cantidad_pedido, 
    COALESCE(p.costo_producto, 0) AS costo_producto,
    t.pais, 
    t.continente, 
    v.clave_territorio,
    -- Cálculo de ingreso_total: precio x cantidad
    (COALESCE(p.precio_producto, 0) * COALESCE(v.cantidad_pedido, 0)) AS ingreso_total,
    -- Cálculo de costo_total: costo x cantidad
    (COALESCE(p.costo_producto, 0) * COALESCE(v.cantidad_pedido, 0)) AS costo_total
FROM ventas_2017 AS v
JOIN productos AS p 
    ON v.clave_producto = p.clave_producto
LEFT JOIN productos_categorias AS pc 
    ON p.clave_subcategoria = pc.clave_subcategoria
LEFT JOIN territorios AS t 
    ON v.clave_territorio = t.clave_territorio;


```

### 3. **Cálculo de ROI Marketing**
```sql
SELECT 
    p.pais, 
    p.clave_territorio,
    -- 1. Métricas Base
    SUM(p.ingresos)::integer AS ingresos,
    SUM(p.costos)::integer AS costos,
    COALESCE(SUM(c.costo_campana::integer), 0) AS costo_campana,

    -- 2. Beneficio Bruto: Ingresos totales menos Costos totales
SUM(p.ingresos)::integer - SUM(p.costos)::integer AS beneficio_bruto,
-- margen_pct = (Ingresos – Costos) / Ingresos * 100
        ((SUM(p.ingresos) - SUM(p.costos)) * 100.0)
        / NULLIF(SUM(p.ingresos), 0) AS margen_pct,
-- roi_pct = Sumas (Ingresos – Costos) / CostoCampanas 100 y usa nullif para evitar dividir entre cero
    ((SUM(p.ingresos) - SUM(p.costos)) * 100.0)
    / NULLIF(SUM(c.costo_campana), 0) AS roi_pct


FROM pais_ingreso_costo AS p
LEFT JOIN pais_campanas AS c
  ON p.clave_territorio = c.clave_territorio
GROUP BY 
    p.pais, 
    p.clave_territorio
ORDER BY 
    p.clave_territorio ASC, 
    ingresos DESC;

```


---



## 📁 ARCHIVOS EN ESTE REPOSITORIO

```
financial-analysis-sql/
├── README.md (este archivo)
├── query-results.csv (resultados exportados)
└── query-results-preview.pdf
```

---

## 📊 ARCHIVOS CSV DISPONIBLES

### `query-results.csv`
Contiene resultados de todas las queries:
- Country Rankings (margen, rentabilidad)
- Campaign Analysis (ROI, profit/loss)
- Inefficient Campaigns (lista de pérdidas)

Columnas:
numero_pedido
clave_producto
nombre_producto
clave_categoria
precio_producto
cantidad_pedido
costo_producto
pais
continente
clave_territorio



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


```

---

## 📞 CONTACTO

- **Fecha:** Marzo 2026
- **Bootcamp:** TripleTen Data Analysis
- **Nivel:** Avanzado (SQL financiero)

---

**Hecho con rigor analítico y pensamiento estratégico. ✨**
