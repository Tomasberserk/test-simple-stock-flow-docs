# ADR-004: Valores congelados y agregación en motor de base de datos

## Estado
Aceptado.

## Contexto
El reporte de un período cerrado debe devolver hoy lo mismo que devolverá dentro de un año, incluso si el catálogo cambia o los productos se renombran (CA-06.4). Además, la agregación no debe cargarse a memoria (CA-06.5).

## Decisión
- La tabla `sale_item` guarda copias congeladas de `product_name`, `category_name` y `unit_price` en el instante de la venta.
- El puerto `SalesReportQuery` ejecuta una consulta `GROUP BY` directamente en MySQL, calculando `unitsSold` y `revenue` en el motor.
- Ni `total` ni `subtotal` se almacenan en columnas de base de datos (Artículo VII).
