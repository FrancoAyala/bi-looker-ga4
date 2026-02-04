# Ejercicio Business Intelligence — GA4 Demo (Google Merch Shop)

Dashboard de performance **e-commerce** armado en **Looker Studio** usando la propiedad demo de **GA4 (Google Merch Shop)**.

> **Resumen:** KPIs principales (Compras, Sesiones, Tasa de conversión) + embudo de eventos (`add_to_cart → begin_checkout → purchase`) con filtros por período, dispositivo y producto.

---

## 🔗 Dashboard (Looker Studio)

- **Link:** https://lookerstudio.google.com/reporting/c430c960-8a93-494f-891a-ceeed838e8f8  
- **Fuente:** GA4 Demo (Google Merch Shop)  
- **Referencia:** documentación de Google sobre GA4 Demo (Merch Shop)

---

## 🎯 Objetivo del ejercicio

El líder de producto solicita un dashboard para conocer el rendimiento del e-commerce midiendo:

### Dimensiones / Selectores
- Rango de fechas
- Nombre del artículo

### Filtros
- Categoría de dispositivo (*device category*)
- Categoría del artículo

### Indicadores a medir
1. **Ratio de compras y sesiones totales** (Compras / Sesiones)
2. **Análisis de caída en cada paso del proceso de compra**  
   - `add_to_cart → begin_checkout → purchase` *(por eventos)*

---

## ✅ ¿Cómo lo resolvería?

1. Conectar la propiedad demo de **GA4** a **Looker Studio**.
2. Crear controles:
   - Selector de fechas
   - Device category
   - Item category
   - Item name
3. Crear KPIs:
   - **Compras** (*purchases*)
   - **Sesiones** (*sessions*)
   - **Tasa de conversión** = Compras / Sesiones *(campo calculado)*
4. Crear embudo de conversión:
   - `add_to_cart → begin_checkout → purchase`
5. Validar consistencia:
   - Comparar contra reportes estándar de GA4 (mismas fechas y filtros).

---

## 🧰 ¿Qué herramientas usaría? (costo / seguridad / gobernanza)

### Opción rápida y barata (para este caso)
**GA4 + Looker Studio**
- **Costo:** $0 (demo + Looker Studio estándar)
- **Seguridad:** control por cuenta Google / permisos de acceso
- **Ventaja:** simple, rápido, sin mover datos

### Opción enterprise (gobernanza y escalabilidad)
**GA4 → Export a BigQuery → Looker Studio**
- **Costo:** BigQuery (storage + queries)
- **Seguridad:** IAM, auditoría, control fino
- **Ventajas:** histórico sin límites, modelado SQL, performance, unificación con otras fuentes

---

## 📌 Notas importantes (por qué NO usar “Artículos comprados” en el ratio)

En e-commerce existen dos niveles distintos:

### 1) Compras (*purchases*) = cantidad de órdenes/transacciones
- 1 compra = 1 transacción (checkout confirmado)
- Ejemplo: comprás 3 remeras en un solo checkout → **1 compra**

### 2) Artículos comprados (*items purchased*) = unidades vendidas
- Cuenta ítems vendidos en total
- Ejemplo: 3 remeras → **3 artículos comprados**

✅ El enunciado pide **“Ratio de compras y sesiones”**, que se interpreta como:
- **Compras / Sesiones = tasa de conversión por sesión**

Si usara:
- **Artículos comprados / Sesiones**
estaría mezclando unidades con visitas, y el ratio puede subir si aumenta el tamaño del carrito aunque no aumenten las compras.

---

## 📈 Interpretación simple de la tasa de conversión

Si el ratio da **1,40%** significa:
- De todas las sesiones del período, **1,40% terminó en una compra**
- Por cada 100 sesiones → ~1,4 compras  
- Por cada 1.000 sesiones → ~14 compras

---

## 🧩 SQL (tabla anidada / nested) — BigQuery

El enunciado asume una tabla anidada `vuelos_por_piloto` con esta estructura conceptual:

- `id`, `nombre`, `apellido`, `experiencia`
- `viajes = ARRAY<STRUCT<id_vuelo, origen, destino, fecha_vuelo, duracion>>`


\### Query 1 — Pilotos que volaron a Buenos Aires en los últimos 2 años



```sql



SELECT DISTINCT

&nbsp; p.nombre,

&nbsp; p.apellido

FROM `vuelos\_por\_piloto` AS p,

UNNEST(p.viajes) AS v

WHERE v.destino = 'Buenos Aires'

&nbsp; AND v.fecha\_vuelo >= DATE\_SUB(CURRENT\_DATE(), INTERVAL 2 YEAR);







\### Query 2 — pilotos + total de vuelos (desc)



SELECT

&nbsp; p.nombre,

&nbsp; p.apellido,

&nbsp; COUNT(\*) AS total\_vuelos

FROM vuelos\_por\_piloto AS p,

UNNEST(p.viajes) AS v

GROUP BY p.nombre, p.apellido

ORDER BY total\_vuelos DESC;

```





