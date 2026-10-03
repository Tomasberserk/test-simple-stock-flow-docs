# Plan de Tareas y Evidencias — Simple Stock Flow (Laravel + React)

## Fase P0 — Documentación y Arquitectura
- [x] **T-00:** Definición de arquitectura Onion y especificación `spec-laravel/`. Evidencia: commit y merge en `docs`.

## Fase 1 — Dominio Puro (Anillo 1)
- [x] **T-01:** Value Objects (`Money` con `BigDecimal`, `Quantity`, IDs). Evidencia: `MoneyTest.php`.
- [x] **T-02:** Entidades de dominio (`Product`, `Sale`, `SaleItem`, `Category`, `User`). Evidencia: `ProductTest.php`, `SaleTest.php`.
- [x] **T-03:** Verificación mecánica de pureza (`grep -rn "Illuminate" app/Domain` = 0).

## Fase 2 — Infraestructura y Persistencia (Anillo 3)
- [ ] **T-04:** Migraciones de base de datos (5 tablas en singular con 9 CHECKs exactos). Evidencia: `php artisan migrate`.
- [ ] **T-05:** Semilla de 5 categorías fijas y admin inicial idempotente. Evidencia: `php artisan db:seed`.
- [ ] **T-06:** Modelos Eloquent y Mappers bidireccionales. Evidencia: tests de integración.

## Fase 3 — Casos de Uso y Puertos (Anillo 2)
- [ ] **T-07:** Definición de 5 puertos Inbound y 10 Outbound en `Application/Ports/`.
- [ ] **T-08:** `ProductCatalogService` (T-04 del spec).
- [ ] **T-09:** `AuthenticationService` (T-06 del spec: login + alta de vendedor).
- [ ] **T-10:** `GetSalesService` (T-07 del spec).
- [ ] **T-11:** `SalesReportService` (T-08 del spec: agregación en motor SQL).
- [ ] **T-12:** `PlaceSaleService` (T-10 del spec: registro atómico y concurrencia optimista).

## Fase 4 — Presentación HTTP API (Anillo 4 y Bootstrap)
- [ ] **T-13:** Controladores REST, FormRequests y JsonResources.
- [ ] **T-14:** `PortBindingsServiceProvider` (Artículo III: único punto de composición).
- [ ] **T-15:** Verificación de contrato con Newman o tests de integración HTTP.

## Fase 5 — Frontend React
- [ ] **T-16:** Setup de Vite + Tailwind y capas desacopladas.
- [ ] **T-17:** Vistas de Login, Catálogo, Carrito de Ventas y Reporte.

## Fase 6 — Satélites y Entrega
- [ ] **T-18:** Herramienta sembradora demo (`tool`) vía HTTP.
- [ ] **T-19:** Landing estática (`page`).
