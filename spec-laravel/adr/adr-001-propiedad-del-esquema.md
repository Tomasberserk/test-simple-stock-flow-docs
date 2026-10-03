# ADR-001: Propiedad del esquema en el servicio (Laravel)

## Estado
Aceptado.

## Contexto
El Artículo V de la Constitución establece que el esquema relacional es propiedad del servicio y no de la infraestructura.

## Decisión
Las tablas, índices y restricciones son gestionadas exclusivamente mediante migraciones de Laravel en el repositorio `api`. El repositorio `infra` provee un contenedor MySQL 8.4 LTS vacío sin DDL.

## Consecuencias
- Despliegue sincronizado entre código y esquema.
- El repositorio `infra` no contiene archivos SQL ni DDL.
