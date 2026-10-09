# ADR-010: Docker Compose con Volúmenes con Nombre para Dependencias

## Estado
Aceptado.

## Contexto
En entornos Windows / WSL2, montar directamente carpetas `vendor/` y `node_modules/` a través de volúmenes de host genera problemas severos de permisos y rendimiento de E/S. Además, el spec original usaba Testcontainers en Python.

## Decisión
- El orquestador `infra/docker-compose.yml` gestiona MySQL 8.4 LTS y los servicios de aplicación con repositorios hermanos.
- `vendor` y `node_modules` se aíslan como volúmenes con nombre de Docker (`named volumes`) para garantizar portabilidad y evitar conflictos de permisos de filesystem.
- Las migraciones y semillas se ejecutan directamente dentro del contenedor de la API.
