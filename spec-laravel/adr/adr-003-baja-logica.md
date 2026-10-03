# ADR-003: Baja lógica en productos con clave foránea restrictiva

## Estado
Aceptado.

## Contexto
Un producto que ya se vendió no puede borrarse físicamente porque rompería el histórico contable de ventas (CA-02.5).

## Decisión
- Columna `deleted_at DATETIME(6) NULL` en la tabla `product`.
- La clave foránea en `sale_item.product_id` tiene `ON DELETE RESTRICT`.
- El repositorio de productos filtra centralizadamente `WHERE deleted_at IS NULL`.
- La entidad de dominio `Product` no porta el atributo `deleted_at`: el agregado representa existencia activa.
