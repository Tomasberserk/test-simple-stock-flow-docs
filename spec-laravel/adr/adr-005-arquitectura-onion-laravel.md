# ADR-005: Adopción de Arquitectura Onion en Laravel

## Estado
Aceptado.

## Contexto
El reto exige implementar Arquitectura Onion en Laravel aislando el dominio de Eloquent y del framework HTTP para evitar la "cebolla deforme".

## Decisión
- 4 anillos concéntricos: `Domain` (PHP puro nativo), `Application` (Casos de uso y puertos), `Infrastructure` (Eloquent Models y Mappers), `Presentation` (Controllers y Resources).
- 1 punto de ensamblaje (`Bootstrap/PortBindingsServiceProvider`).
- Las transacciones de base de datos se inyectan a través del puerto `UnitOfWork` / `TransactionManagerInterface` para que `Application` no conozca `DB::transaction()`.
