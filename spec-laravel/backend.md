# Especificación Técnica del Backend — Laravel (PHP 8.1+)

## 1. El Núcleo Puro: `Domain/`

### Entidades de Dominio
- **`Product`:** Raíz de agregado del catálogo. Encapsula stock, precio e identificador. **No contiene la columna `version`** (ADR-002) ni `deleted_at` (ADR-003). Invariantes: `withdrawStock(Quantity $qty)` prohíbe saldo negativo; `changePrice(Money $price)` exige precio positivo.
- **`Sale`:** Raíz de agregado de ventas. Inmutable una vez confirmada. Mínimo 1 línea y prohíbe productos duplicados. **No almacena total ni subtotal**: método `getTotal(): Money` calculado sumando líneas (Artículo VII).
- **`SaleItem`:** Línea con valores congelados (`unitPrice`, `productName`, `categoryName`). Subtotal dinámico `unitPrice * quantity`.
- **`Category`:** Entidad de referencia fija (5 categorías de solo lectura).
- **`User`:** Identidad, username normalizado en minúsculas y sin espacios, `role` en `['admin', 'seller']`, hash irreversible de contraseña.

### Value Objects
- **`Money`:** Precisión matemática exacta utilizando `Brick\Math\BigDecimal` y `RoundingMode::HALF_UP` a 2 decimales. Monomoneda (`COP` / D-05). Cero `float`.
- **`Quantity`:** Entero estrictamente mayor a 0.
- **`ProductId`, `SaleId`, `CategoryId`, `UserId`:** Identificadores fuertemente tipados.
- **`Username`, `Role`:** Validaciones integradas.

---

## 2. Casos de Uso y Puertos: `Application/`

### 5 Puertos Inbound (Entrada)
1. `PlaceSale`: Registro atómico de ventas.
2. `ManageProducts`: Mantenimiento y consulta del catálogo.
3. `GetSales`: Consulta de ventas por ID y rango.
4. `GetSalesReport`: Reporte de ventas por rango.
5. `Authenticate`: Autenticación y alta de vendedor (DP-04).

### 10 Puertos Outbound (Salida)
1. `ProductRepository`: Consulta y persistencia de catálogo.
2. `SaleRepository`: Persistencia de ventas.
3. `CategoryRepository`: Consulta de categorías de solo lectura.
4. `UserRepository`: Consulta y persistencia de usuarios.
5. `UnitOfWork`: Transacciones atómicas desacopladas de Laravel.
6. `SalesReportQuery`: Agregación en motor de base de datos.
7. `Clock`: Instante de tiempo en UTC.
8. `PasswordHasher`: Hash y verificación de contraseñas.
9. `TokenGenerator`: Generación y validación de tokens JWT.
10. `FileStorage`: Almacenamiento local de imágenes mediante claves opacas.

### 5 Servicios Oficiales de `tasks.md`
- `PlaceSaleService.php` (T-10)
- `ProductCatalogService.php` (T-04)
- `GetSalesService.php` (T-07)
- `SalesReportService.php` (T-08)
- `AuthenticationService.php` (T-06)

---

## 3. Adaptadores y Persistencia: `Infrastructure/`

- **Modelos Eloquent:** Aislados dentro de `Infrastructure/Persistence/Model/` (`ProductModel`, `SaleModel`, `SaleItemModel`, `CategoryModel`, `UserModel`).
- **Concurrencia Optimista (ADR-002):** La columna `version INT` vive exclusivamente en `ProductModel`. Si un `UPDATE` afecta 0 filas, el adaptador traduce el fallo a `ConcurrencyConflictException`.
- **Unit of Work:** `LaravelUnitOfWork` implementa el puerto `UnitOfWork` llamando internamente a `DB::transaction()`.
