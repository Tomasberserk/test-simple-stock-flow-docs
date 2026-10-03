# Guía de Traducción Conceptual: De Hexagonal a Onion en Laravel

## 1. Mapeo de Conceptos Arquitectónicos

| Hexagonal (Python / .NET) | Onion (Laravel) | Justificación y Regla |
|---|---|---|
| `domain/model` | `Domain/Model/` | Entidades puras sin dependencias externas. |
| `domain/value_objects` | `Domain/ValueObject/` | `BigDecimal` con `RoundingMode::HALF_UP` sustituye a `decimal.Decimal`. |
| `domain/exceptions` | `Domain/Exception/` | Excepciones en español heredando de `BusinessRuleViolation`. |
| `application/ports/inbound` | `Application/Ports/Inbound/` | 5 interfaces que definen los casos de uso. |
| `application/ports/outbound` | `Application/Ports/Outbound/` | 10 interfaces de repositorios y servicios técnicos. |
| `application/services` | `Application/UseCase/` | Los 5 servicios oficiales de `tasks.md`. |
| `adapters/outbound/persistence` | `Infrastructure/Persistence/` | Modelos Eloquent (`ProductModel`, etc.), Repositorios y Mappers. |
| `adapters/inbound/api` | `Presentation/Http/` | Controladores API, FormRequests y JsonResources. |
| `bootstrap/composition` | `Bootstrap/` | `PortBindingsServiceProvider` (Artículo III). |

---

## 2. Decisiones Técnicas Clave Adaptadas

- **D-01 (Motor):** MySQL 8.4 LTS, `utf8mb4_0900_ai_ci`.
- **D-04 (Concurrencia):** Columna física `version INT` gestionada por el mapper/repositorio en `Infrastructure`. El dominio no la conoce.
- **D-05 (Monomoneda):** COP, 2 decimales exactos.
- **D-06 (Valores Congelados):** `product_name`, `category_name`, `unit_price` se copian en `sale_item` en el instante de la venta.
- **D-07 (Baja Lógica):** `deleted_at DATETIME(6)` filtrado en repositorio.
- **D-09 (Seguridad):** Hash irreversible con Bcrypt / Argon2. JWT para tokens de sesión.
- **D-10 (Semilla):** 5 categorías fijas con UUIDs predeterminados y admin desde variables de entorno.
