# Especificación de Traducción Arquitectónica: De Hexagonal a Onion
## Proyecto: Simple Stock Flow — Ficha ADSO 3413974

> **Estado:** Documentación Previa al Desarrollo (Commit Cero de Documentación)  
> **Rige bajo:** Constitución del Proyecto (Artículos I, II, III, V, VI, VII, VIII, IX, X, XI) y ADR-001 a ADR-004.  
> **Rama:** `docs/onion-decision`

---

## 1. Contexto y Justificación de la Traducción

El documento base [`spec-python/architecture.md`](spec-python/architecture.md) se titula **«El hexágono del servicio»** y describe una arquitectura **Hexagonal (Puertos y Adaptadores)** con adaptadores primarios/inbound (`adapters/inbound/api`) y secundarios/outbound (`adapters/outbound/{persistence,storage,security}`).

Por mandato de la prueba técnica, esta especificación traduce las decisiones y reglas del hexágono a **Arquitectura Onion (Cebolla)** en **PHP 8.1+ con Laravel** para el backend (`api`) y **React** para el frontend (`app`), manteniendo intactas todas las invariantes de negocio, los contratos de la API y el modelo de datos.

---

## 2. Decisiones Arquitectónicas del Equipo

### Decisión 1 — Manejo de Importes: `Brick\Math\BigDecimal`
- **Fidelidad al spec:** El spec manda expresamente: *«El importe es decimal en el dominio y nunca float»*.
- **Implementación:** Se utiliza la librería PHP pura `Brick\Math\BigDecimal` (disponible de forma nativa en el ecosistema).
- **Reglas del Value Object `Money`:** Escala fija de 2 decimales, modo de redondeo `RoundingMode::HALF_UP` (aleja de cero en el empate), y monomoneda (`COP` / D-05).
- **Prohibición:** El uso de tipos primitivos `float` queda prohibido en la aritmética de dominio. El formateo lo realiza el objeto de valor, nunca el adaptador de persistencia.

### Decisión 2 — Estrategia de Ramas y Commits (SDD Riguroso)
- Se trabaja con ramas cortas por fase y Pull Requests hacia `main`:
  1. `docs/onion-decision` (Fase P0: Documentación inicial y arquitectura, merge previo a cualquier código).
  2. `phase/1-domain`
  3. `phase/2-schema-infra`
  4. `phase/3-use-cases`
  5. `phase/4-robustness`
  6. `phase/5-app`
  7. `phase/6-satellites`
  8. `phase/7-delivery`
- **Regla innegociable:** El commit de documentación es siempre el primero. El instructor evalúa la trazabilidad documental antes que las líneas de código.

---

## 3. Definición Estricta de las Capas Onion (Backend `api`)

La cebolla se organiza en 4 anillos concéntricos estrictos más 1 punto de ensamblaje (`Bootstrap`):

```text
                             ANILLO 4: PRESENTATION
                         ┌─────────────────────────────┐
                         │   Controllers / Requests    │
                         │   Resources / Middleware    │
                         │ ┌─────────────────────────┐ │
                         │ │  ANILLO 2: APPLICATION  │ │
                         │ │ ┌─────────────────────┐ │ │
                         │ │ │  ANILLO 1: DOMAIN   │ │ │
                         │ │ │                     │ │ │
                         │ │ │ Entities / Model    │ │ │
                         │ │ │ ValueObjects (Math) │ │ │
                         │ │ │ Business Exceptions │ │ │
                         │ │ └──────────┬──────────┘ │ │
                         │ │            │            │ │
                         │ │ Inbound Ports (5)       │ │
                         │ │ Outbound Ports (10)     │ │
                         │ │ Use Cases / Services    │ │
                         │ └────────────▲────────────┘ │
                         └──────────────┼──────────────┘
                                        │ (implementa)
                         ┌──────────────┴──────────────┐
                         │   ANILLO 3: INFRASTRUCTURE  │
                         │   Eloquent Models / Mappers │
                         │   Eloquent Repositories     │
                         │   UnitOfWork (DB::trans)    │
                         │   Security / FileStorage    │
                         └─────────────────────────────┘
                                        ▲
                                        │ (ensambla)
                         ┌──────────────┴──────────────┐
                         │          BOOTSTRAP          │
                         │ PortBindingsServiceProvider │
                         └─────────────────────────────┘
```

### Reglas de Dependencia:
1. **`Domain` (Anillo 1):** PHP puro nativo. Cero dependencias de Laravel o librerías de terceros (salvo `Brick\Math\BigDecimal`). No conoce HTTP, SQL ni controladores.
2. **`Application` (Anillo 2):** Solo importa `Domain`. Declara los 5 puertos Inbound y los 10 puertos Outbound. Orquesta casos de uso.
3. **`Infrastructure` (Anillo 3):** Implementa los puertos Outbound de `Application` y contratos de `Domain`. Encapsula Eloquent, migraciones y servicios técnicos. **`Presentation` tiene prohibido importar `Infrastructure`**.
4. **`Presentation` (Anillo 4):** Solo importa `Application` (Inbound Ports) y `Domain`. Recibe peticiones HTTP y devuelve JSON según contrato.
5. **`Bootstrap`:** Único punto de composición (Artículo III). Amarra los puertos Inbound y Outbound con sus implementaciones de infraestructura.

---

## 4. Estructura Oficial de Archivos (`test-simple-stock-flow-api`)

Los nombres se alinean estrictamente con la especificación funcional y `tasks.md`:

```text
app/
│
├── Domain/                                          # ANILLO 1: NÚCLEO PURO
│   ├── Model/
│   │   ├── Product.php                              # Raíz de agregado (catálogo), sin `version`
│   │   ├── Sale.php                                 # Raíz de agregado (ventas), sin `total`
│   │   ├── SaleItem.php                             # Entidad interna, sin `subtotal`
│   │   ├── Category.php                             # Entidad de referencia fija
│   │   └── User.php                                 # Raíz de agregado (identidad)
│   ├── ValueObject/
│   │   ├── Money.php                                # Brick\Math\BigDecimal, escala 2, HALF_UP
│   │   ├── Quantity.php                             # Entero > 0
│   │   ├── ProductId.php, SaleId.php, CategoryId.php, UserId.php
│   │   ├── Username.php                             # Normaliza a minúsculas y sin espacios
│   │   └── Role.php                                 # 'admin' | 'seller'
│   ├── Exception/                                   # Mensajes en ESPAÑOL (para usuario / HTTP 422)
│   │   ├── BusinessRuleViolation.php                # Base abstracta
│   │   ├── InsufficientStockException.php
│   │   ├── InvalidPriceException.php
│   │   ├── InvalidQuantityException.php
│   │   ├── EmptySaleException.php
│   │   ├── RepeatedProductException.php
│   │   ├── ProductNotFoundException.php
│   │   ├── UnknownCategoryException.php
│   │   ├── DuplicateUsernameException.php
│   │   ├── InvalidRoleException.php
│   │   └── InvalidCredentialsException.php
│   └── Service/                                     # Invariantes en el agregado (Art. VI)
│       └── .gitkeep
│
├── Application/                                     # ANILLO 2: CASOS DE USO
│   ├── Ports/
│   │   ├── Inbound/                                 # 5 PUERTOS DE ENTRADA + DTOs
│   │   │   ├── PlaceSale.php
│   │   │   ├── ManageProducts.php
│   │   │   ├── GetSales.php
│   │   │   ├── GetSalesReport.php
│   │   │   ├── Authenticate.php
│   │   │   ├── PlaceSaleCommand.php
│   │   │   ├── ProductView.php, SaleView.php, SaleItemView.php
│   │   │   ├── SalesReport.php, SalesReportRow.php
│   │   │   ├── AuthResult.php, PagedResult.php
│   │   └── Outbound/                                # 10 PUERTOS DE SALIDA
│   │       ├── ProductRepository.php
│   │       ├── SaleRepository.php
│   │       ├── CategoryRepository.php
│   │       ├── UserRepository.php
│   │       ├── FileStorage.php
│   │       ├── PasswordHasher.php
│   │       ├── TokenGenerator.php
│   │       ├── Clock.php
│   │       ├── UnitOfWork.php                       # Puerto para transacciones atómicas
│   │       └── SalesReportQuery.php                 # Agregación en base de datos
│   ├── UseCase/                                     # 5 SERVICIOS EXACTOS DE TASKS.MD
│   │   ├── PlaceSaleService.php                     # T-10
│   │   ├── ProductCatalogService.php                # T-04
│   │   ├── GetSalesService.php                      # T-07
│   │   ├── SalesReportService.php                   # T-08
│   │   └── AuthenticationService.php                # T-06 (Login + Registro de Vendedor)
│   ├── Model/
│   │   ├── PageRequest.php                          # Límite a 100 elementos
│   │   └── DateRange.php                            # Rango de fechas inicio <= fin
│   └── Exception/
│       └── ConcurrencyConflict.php                  # Conflicto optimista de aplicación (ADR-002)
│
├── Infrastructure/                                  # ANILLO 3: DETALLES TÉCNICOS
│   ├── Persistence/
│   │   ├── Model/                                   # Modelos Eloquent de MySQL
│   │   │   ├── ProductModel.php                     # Contiene columna `version` y `deleted_at`
│   │   │   ├── SaleModel.php
│   │   │   ├── SaleItemModel.php
│   │   │   ├── CategoryModel.php
│   │   │   └── UserModel.php
│   │   ├── Mapper/                                  # ProductMapper, SaleMapper, etc.
│   │   ├── Repository/                              # EloquentProductRepository, etc.
│   │   │   └── EloquentSalesReportQuery.php         # Resuelve agrupaciones con SQL nativo
│   │   └── LaravelUnitOfWork.php                    # Implementa UnitOfWork llamando a DB::transaction()
│   ├── Security/
│   │   ├── JwtTokenGenerator.php
│   │   └── BcryptPasswordHasher.php
│   ├── Storage/
│   │   └── LocalFileStorage.php                     # Claves opacas, nunca rutas
│   └── Configuration/
│       └── Settings.php                             # Cero valores por defecto para secretos (Art. IX)
│
├── Presentation/                                    # ANILLO 4: HTTP API
│   ├── Http/
│   │   ├── Controller/                              # Controladores REST
│   │   ├── Request/                                 # FormRequests (solo forma, sin reglas de negocio)
│   │   ├── Resource/                                # JsonResources (CamelCase según api-contract.md)
│   │   ├── Serialization/                           # Serializadores para Money y UTC
│   │   └── ProblemDetails/                          # Formatos de error 400, 409, 422
│   └── Middleware/                                  # Autenticación JWT y roles
│
└── Bootstrap/                                       # PUNTO DE ENSAMBLAJE (NO ES ANILLO)
    └── PortBindingsServiceProvider.php              # Amarre único de puertos e implementaciones (Art. III)
```

---

## 5. Arquitectura del Frontend (`test-simple-stock-flow-app`)

En React (Vite) se aplica también la separación limpia de capas:

```text
src/
├── domain/                                          # ANILLO 1: Sin React, tipos e invariantes puras
│   └── model/                                       # Product, Cart, SaleItem, Money, Quantity
├── application/                                     # ANILLO 2: Casos de uso
│   ├── ports/                                       # ProductRepository, CartRepository, SessionRepository
│   └── use-cases/                                   # BrowseCatalog, AddToCart, Checkout, Login, ViewSalesReport
├── infrastructure/                                  # ANILLO 3: Conexión HTTP
│   ├── http/                                        # Cliente Axios / Fetch, DTOs del contrato
│   ├── mappers/
│   └── providers.ts                                 # Ensamblaje de servicios
└── features/                                        # ANILLO 4: Componentes visuales y páginas
    ├── auth/
    ├── catalog/
    ├── cart/
    ├── sales/
    └── reports/
```

---

## 6. Comprobaciones Mecánicas de Arquitectura

Para garantizar que el código cumpla con los principios constitucionales antes de cada entrega:
1. `grep -rn "Illuminate\\\\" app/Domain` → **0 resultados** (Dominio no depende de Laravel).
2. `grep -rn "DB::" app/Application` → **0 resultados** (Aplicación no llama a la base de datos directamente).
3. `grep -rn "total" database/migrations` → **0 columnas** en `sale` y `sale_item` (Valores derivados se calculan, Artículo VII).
4. `grep -rn "float" app/Domain/ValueObject/Money.php` → **0 ocurrencias** (Manejo de dinero exacto con `BigDecimal`).
