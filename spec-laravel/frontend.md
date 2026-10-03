# Especificación del Frontend — React (Vite)

## 1. Principio Arquitectónico

El frontend en React se estructura desacoplando la lógica de negocio de la vista visual, garantizando que los componentes de React no se comuniquen directamente con Axios o Fetch sin pasar por un caso de uso o servicio:

```text
src/
├── domain/                                          # ANILLO 1: Modelos e invariantes en TypeScript puro
│   └── model/                                       # Product, Cart, SaleItem, Money, Quantity
├── application/                                     # ANILLO 2: Casos de uso del cliente
│   ├── ports/                                       # ProductRepository, CartRepository, SessionRepository
│   └── use-cases/                                   # BrowseCatalog, AddToCart, Checkout, Login, ViewSalesReport
├── infrastructure/                                  # ANILLO 3: Adaptadores HTTP y Storage
│   ├── http/                                        # Cliente HTTP, Interceptores JWT, DTOs del contrato
│   ├── mappers/                                     # Traductores de respuestas JSON a modelos de vista
│   └── providers.ts                                 # Composition root del frontend
└── features/                                        # ANILLO 4: Interfaz de usuario (Presentación)
    ├── auth/                                        # Formulario de inicio de sesión
    ├── catalog/                                     # Catálogo con búsqueda, filtro de categoría y paginación
    ├── cart/                                        # Carrito de ventas y checkout
    ├── sales/                                       # Historial de ventas registradas
    └── reports/                                     # Tabla del reporte de ventas con totales
```

## 2. Reglas de Interfaz (UI/UX)
- Textos orientados al usuario completamente en **español** con sus respectivas tildes y eñes (Artículo XI).
- Manejo de estados de carga (skeletons / spinners) y mensajes de error amigables basados en las respuestas 422 y 409 de la API.
- Carrito inmutable que valida existencias locales antes de disparar el checkout.
