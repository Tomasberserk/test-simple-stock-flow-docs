# Especificación de Contenedores e Infraestructura — Docker

## 1. El Principio del Motor Vacío (Artículo V)

El repositorio de infraestructura (`test-simple-stock-flow-infra`) tiene una responsabilidad única y estrictamente delimitada: **levantar el motor de base de datos vacío**.

- **No contiene migraciones:** Cero scripts DDL (`.sql`) dentro de `infra`.
- **Propiedad del esquema:** El esquema relacional le pertenece exclusivamente al backend (`api`) y se gestiona mediante migraciones de Laravel.

---

## 2. Definición del Servicio MySQL 8.4 LTS

```yaml
services:
  db:
    image: mysql:8.4
    container_name: stockflow-db
    restart: unless-stopped
    command: >
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_0900_ai_ci
      --default-time-zone=+00:00
      --sql-mode=STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-rootsecret}
      MYSQL_DATABASE: ${MYSQL_DATABASE:-stockflow}
      MYSQL_USER: ${MYSQL_USER:-stockflow}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:-stockflow}
    ports:
      - "${MYSQL_PORT:-3306}:3306"
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -u $$MYSQL_USER --password=$$MYSQL_PASSWORD || exit 1"]
      interval: 5s
      timeout: 5s
      retries: 10
      start_period: 15s

volumes:
  db_data:
    name: stockflow_db_data
```

### Características Técnicas:
1. **Colación:** `utf8mb4_0900_ai_ci` (insensible a mayúsculas y acentos para búsquedas y unicidad).
2. **Zona horaria:** UTC (`+00:00`). Todas las fechas se registran como `DATETIME(6)` en UTC.
3. **Healthcheck:** Utiliza `mysqladmin ping` para garantizar que la API solo intente conectarse cuando el motor esté listo para recibir peticiones.
