# ADR-007: Ubicación de los Puertos en Application y no en Domain

## Estado
Aceptado.

## Contexto
En la arquitectura Onion clásica de Jeffrey Palermo, las interfaces de repositorio a menudo se ubican en el dominio. Sin embargo, los Artículos II y IV (DIP) de la Constitución del proyecto establecen que todo lo que la aplicación necesita del exterior se declara como puerto en `application/ports/outbound/`.

## Decisión
- Los 5 puertos Inbound y los 10 puertos Outbound residen exclusivamente en `Application/Ports/`.
- `Domain` permanece 100% libre de puertos, interfaces de persistencia y dependencias externas.
