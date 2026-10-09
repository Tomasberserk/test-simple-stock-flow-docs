# ADR-009: Mitigación de Trampas de Laravel (Facades, Active Record, Auto-wiring)

## Estado
Aceptado.

## Contexto
Laravel fomenta patrones como Facades estáticos globales (`DB::`, `Auth::`), Active Record acoplado (`Product::create()`) y auto-wiring que pueden destruir los límites de la arquitectura Onion si se usan descuidadamente en el dominio o la aplicación.

## Decisión
- Prohibición absoluta de Facades e `Illuminate\` dentro de `Domain` y `Application`.
- Los modelos Eloquent viven únicamente en `Infrastructure/Persistence/Models` y se transforman mediante `Mappers` bidireccionales hacia/desde entidades de dominio puras.
- El IoC Container de Laravel se usa únicamente en `Bootstrap/PortBindingsServiceProvider` para enlazar puertos con sus implementaciones.
