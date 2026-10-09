# ADR-008: Verificación Estática de Dependencias entre Capas con Deptrac

## Estado
Aceptado.

## Contexto
El cumplimiento de las reglas de dependencia (R-01 a R-06) en Onion no puede depender de la buena voluntad del desarrollador. En Python se utilizaba `import-linter`.

## Decisión
- Utilizar Deptrac / tests de reflexión arquitectónica en PHP para garantizar:
  1. `Domain` no importa nada de `Application`, `Infrastructure`, `Presentation` ni Laravel.
  2. `Application` solo importa `Domain`.
  3. `Presentation` nunca importa `Infrastructure`.
  4. Los controladores de `Presentation` solo inyectan puertos `Inbound`, nunca `Outbound`.
