\# Ejercicio Business Intelligence — GA4 Demo (Google Merch Shop)



Dashboard de performance e-commerce armado en \*\*Looker Studio\*\* usando la propiedad demo de \*\*GA4 (Google Merch Shop)\*\*.



\## 🔗 Dashboard

\- Link: \*\*https://lookerstudio.google.com/reporting/c430c960-8a93-494f-891a-ceeed838e8f8\*\*



> Fuente: GA4 Demo (Google Merch Shop)  

> Referencia: soporte de Google sobre la demo de GA4.



---



\## 🎯 Objetivo del ejercicio



El líder de producto quiere un dashboard para conocer el rendimiento del e-commerce con:



\### Dimensiones

\- Rango de fechas

\- Nombre del artículo



\### Filtros

\- Categoría de dispositivo (device category)

\- Categoría del artículo



\### Indicadores a medir

1\) \*\*Ratio de compras y sesiones totales\*\*  

2\) \*\*Análisis de caída en cada paso del proceso de compra\*\*  

&nbsp;  - add\_to\_cart → begin\_checkout → purchase  

&nbsp;  ( por eventos)



---



\## ✅ ¿Cómo lo resolvería?



1\. Conectar la propiedad demo de GA4 a Looker Studio.

2\. Crear controles:

&nbsp;  - Selector de fechas

&nbsp;  - Device category

&nbsp;  - Item category

&nbsp;  - Item name

3\. Crear KPIs:

&nbsp;  - Compras (purchases)

&nbsp;  - Sesiones

&nbsp;  - Ratio compras/sesiones (campo calculado)

4\. Embudo de conversión:

&nbsp;  - add\_to\_cart → begin\_checkout → purchase

5\. Validar consistencia:

&nbsp;  - comparar valores contra reportes estándar de GA4.



---



\## 🧰 ¿Qué herramientas usaría? (costo / seguridad / gobernanza)



\### Opción rápida y barata (recomendada para este caso)

\- \*\*GA4 + Looker Studio\*\*

&nbsp; - Costo: $0 (demo + Looker Studio estándar)

&nbsp; - Seguridad: control por cuenta Google / permisos de acceso

&nbsp; - Ventaja: rápido, simple, sin mover datos



\### Opción enterprise 

\- \*\*GA4 → Export a BigQuery → Looker Studio\*\*

&nbsp; - Costo: BigQuery (storage + queries)

&nbsp; - Seguridad: IAM, auditoría

&nbsp; - Ventajas: histórico sin límites, modelado SQL, performance, datos unificados



---



\## 📌 Notas importantes (por qué NO uso “Artículos comprados” para el ratio)



En e-commerce existen dos niveles distintos:



\### 1) “Compras” (purchases) = cantidad de pedidos/órdenes

\- 1 compra = 1 transacción (checkout confirmado).

\- Ejemplo: comprás 3 remeras en un solo checkout → \*\*1 compra\*\*.



\### 2) “Artículos comprados” (items purchased) = unidades totales vendidas

\- Cuenta cuántos ítems se vendieron en total.

\- Ejemplo: 3 remeras → \*\*3 artículos comprados\*\*.



✅ El enunciado pide \*\*“Ratio de compras y sesiones”\*\*, que se interpreta como:

\- \*\*Compras / Sesiones = tasa de conversión por sesión\*\*



Si usara:

\- \*\*Artículos comprados / Sesiones\*\*

obtendría otra cosa (mezcla unidades con visitas) y puede subir si la gente compra más ítems por pedido aunque no aumenten las compras.



---



\## 📈 Interpretación simple del ratio (ejemplo)

Si el ratio da \*\*1,40%\*\* significa:



\- De todas las sesiones (visitas) del período, \*\*1,40% terminaron en compra\*\*.

\- Por cada 100 sesiones → ~1,4 compras  

\- Por cada 1.000 sesiones → ~14 compras



---



\## 🧩 SQL (tabla anidada / nested) — BigQuery



El enunciado asume una tabla anidada `vuelos\_por\_piloto`:



\- id, nombre, apellido, experiencia

\- viajes = `ARRAY<STRUCT<id\_vuelo, origen, destino, fecha\_vuelo, duracion>>`



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





