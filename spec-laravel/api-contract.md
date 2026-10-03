# Contrato de la API HTTP — Simple Stock Flow

Todas las respuestas exitosas y de error se serializan en JSON con convención `camelCase`.

## 1. Autenticación (`/api/auth`)

### `POST /api/auth/login`
- **Request:**
  ```json
  {
    "username": "admin",
    "password": "secretpassword"
  }
  ```
- **Response 200 OK:**
  ```json
  {
    "token": "eyJhbGciOi...",
    "expiresAt": "2026-10-03T20:00:00.000Z",
    "userId": "11111111-2222-3333-4444-555555555555",
    "username": "admin",
    "role": "admin"
  }
  ```
- **Response 401 Unauthorized:**
  ```json
  {
    "error": "Credenciales inválidas"
  }
  ```

---

## 2. Catálogo de Productos (`/api/products`)

### `GET /api/products`
- **Query params:** `query` (opcional), `categoryId` (opcional), `page` (default 1), `perPage` (default 20, max 100).
- **Response 200 OK:**
  ```json
  {
    "items": [
      {
        "id": "uuid",
        "name": "Martillo",
        "price": 25.50,
        "stock": 10,
        "categoryId": "22222222-2222-4222-8222-222222222222",
        "imageKey": null
      }
    ],
    "total": 1,
    "page": 1,
    "perPage": 20,
    "totalPages": 1
  }
  ```

### `POST /api/products` (Requiere rol `admin`)
- **Request:**
  ```json
  {
    "name": "Taladro",
    "price": 120.00,
    "stock": 5,
    "categoryId": "22222222-2222-4222-8222-222222222222"
  }
  ```
- **Response 201 Created:** `ProductView`
- **Response 422 Unprocessable Entity:** Validaciones de negocio en español.

### `PUT /api/products/{id}` (Requiere rol `admin`)
- **Request:** Modifica nombre, precio, categoría.
- **Response 200 OK:** `ProductView`

### `DELETE /api/products/{id}` (Requiere rol `admin`)
- **Response 204 No Content** (Baja lógica).

---

## 3. Ventas (`/api/sales`)

### `POST /api/sales` (Requiere autenticación: `admin` o `seller`)
- **Request:**
  ```json
  {
    "items": [
      {
        "productId": "uuid-producto-1",
        "quantity": 2
      }
    ]
  }
  ```
- **Response 201 Created:**
  ```json
  {
    "id": "uuid-venta",
    "soldAt": "2026-10-03T17:00:00.000Z",
    "soldByUserId": "uuid-usuario",
    "soldBy": "carlos",
    "items": [
      {
        "id": "uuid-linea",
        "productId": "uuid-producto-1",
        "productName": "Taladro",
        "categoryName": "Herramientas",
        "quantity": 2,
        "unitPrice": 120.00,
        "subtotal": 240.00
      }
    ],
    "total": 240.00
  }
  ```
- **Response 409 Conflict:** Conflicto de concurrencia optimista al modificar stock.
- **Response 422 Unprocessable Entity:** Stock insuficiente o producto duplicado.

### `GET /api/sales/{id}`
- **Response 200 OK:** Venta con sus líneas y total calculado.

---

## 4. Reporte de Ventas (`/api/reports/sales`)

### `GET /api/reports/sales`
- **Query params:** `startDate` (YYYY-MM-DD), `endDate` (YYYY-MM-DD).
- **Response 200 OK:**
  ```json
  {
    "startDate": "2026-10-01",
    "endDate": "2026-10-31",
    "totalSalesCount": 15,
    "grandTotal": 3600.00,
    "items": [
      {
        "productId": "uuid",
        "productName": "Taladro",
        "categoryName": "Herramientas",
        "unitsSold": 10,
        "revenue": 1200.00
      }
    ]
  }
  ```
