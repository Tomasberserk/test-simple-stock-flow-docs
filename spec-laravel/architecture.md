# Arquitectura Onion (Cebolla) — Simple Stock Flow (Laravel)

## 1. Los Cuatro Anillos de la Cebolla

El diseño del backend desacopla completamente el núcleo de negocio de cualquier detalle tecnológico o framework. Las dependencias apuntan estrictamente hacia el centro:

```text
                        ┌────────────────────────────────────────────────────────┐
                        │                ANILLO 4: PRESENTATION                  │
                        │           Controllers / Requests / Resources           │
                        │ ┌────────────────────────────────────────────────────┐ │
                        │ │              ANILLO 2: APPLICATION                 │ │
                        │ │ ┌────────────────────────────────────────────────┐ │ │
                        │ │ │                ANILLO 1: DOMAIN                │ │ │
                        │ │ │                                                │ │ │
                        │ │ │  Entities (Product, Sale, SaleItem, User, Cat) │ │ │
                        │ │ │  Value Objects (Money BigDecimal, Quantity)    │ │ │
                        │ │ │  Business Rule Violations (Exceptions)         │ │ │
                        │ │ └───────────────────────┬────────────────────────┘ │ │
                        │ │                         │                          │ │
                        │ │  Inbound Ports (5)      │                          │ │
                        │ │  Outbound Ports (10)    │                          │ │
                        │ │  Use Cases / Services   │                          │ │
                        │ └─────────────────────────▲──────────────────────────┘ │
                        └───────────────────────────┼────────────────────────────┘
                                                    │ (implementa puertos)
                        ┌───────────────────────────┴────────────────────────────┐
                        │                ANILLO 3: INFRASTRUCTURE                │
                        │         Eloquent Models / Repositories / Mappers       │
                        │         UnitOfWork (DB::transaction) / Security        │
                        └────────────────────────────────────────────────────────┘
                                                    ▲
                                                    │ (ensambla dependencias)
                        ┌───────────────────────────┴────────────────────────────┐
                        │                       BOOTSTRAP                        │
                        │             PortBindingsServiceProvider                │
                        └────────────────────────────────────────────────────────┘
```

---

## 2. Reglas de Dependencia Innegociables

1. **`Domain` (Anillo 1):** Código PHP 8.1+ puro y nativo. Cero dependencias de Laravel o librerías de terceros (salvo `Brick\Math\BigDecimal`). No conoce a Eloquent, controladores ni la base de datos.
2. **`Application` (Anillo 2):** Solo importa `Domain`. Declara los puertos de entrada (Inbound) y de salida (Outbound). Orquesta los casos de uso. Prohibido llamar a `DB::transaction()` o modelos Eloquent directamente.
3. **`Infrastructure` (Anillo 3):** Implementa los puertos Outbound y satisface los contratos del Dominio. Encapsula los modelos Eloquent (`ProductModel`, `SaleModel`), las migraciones y la persistencia física en MySQL. `Presentation` tiene prohibido importar `Infrastructure`.
4. **`Presentation` (Anillo 4):** Solo conoce `Application` (Inbound Ports) y `Domain`. Recibe peticiones HTTP, valida el esquema de entrada y devuelve respuestas JSON según contrato.
5. **`Bootstrap`:** Único punto de composición (Artículo III). Amarra las interfaces con sus implementaciones de infraestructura mediante un ServiceProvider.
