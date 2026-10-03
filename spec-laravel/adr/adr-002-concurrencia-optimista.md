# ADR-002: Concurrencia optimista con columna `version`

## Estado
Aceptado.

## Contexto
Se requiere garantizar que dos ventas simultáneas del último ejemplar de un producto no dejen el stock en negativo (CA-04.6) y una de ellas sea rechazada con HTTP 409 Conflict.

## Decisión
MySQL no provee testigos de versión nativos por fila como PostgreSQL (`xmin`). Por tanto, se utiliza una columna física `version INT NOT NULL` en la tabla `product`.
- La columna vive exclusivamente en la persistencia (`ProductModel`).
- El Dominio (`Product`) no conoce la columna `version`.
- La actualización en el repositorio ejecuta:
  `UPDATE product SET stock = ?, version = version + 1 WHERE id = ? AND version = ?`
- Si las filas afectadas son 0, se lanza `ConcurrencyConflictException` y la API responde HTTP 409.
