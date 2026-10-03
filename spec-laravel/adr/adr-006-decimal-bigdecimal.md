# ADR-006: Manejo monetario exacto con `Brick\Math\BigDecimal`

## Estado
Aceptado.

## Contexto
El spec establece que el importe es decimal y nunca `float`. El uso de tipos primitivos `float` en PHP produce errores de redondeo de punto flotante IEEE 754.

## Decisión
- El Value Object `Money` utiliza internamente `Brick\Math\BigDecimal`.
- Escala estricta de 2 decimales con modo de redondeo `RoundingMode::HALF_UP`.
- Prohibición de aritmética con `float` en todo el núcleo de dominio.
