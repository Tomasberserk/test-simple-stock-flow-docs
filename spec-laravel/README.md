# Especificación Técnica para Laravel + React — Simple Stock Flow

> **Prueba técnica · Ficha ADSO 3413974**  
> **Línea de base de Arquitectura Onion en Laravel**

Esta carpeta contiene la **especificación formal completa (SDD)** para la implementación del sistema *Simple Stock Flow* utilizando el stack tecnológico exigido:

- **Backend:** PHP 8.1+ con Laravel bajo **Arquitectura Onion (Cebolla)**.
- **Frontend:** React (Vite) desacoplado en capas.
- **Infraestructura:** Docker con MySQL 8.4 LTS.

---

## Índice de Documentos de la Especificación

| Documento | Propósito |
|---|---|
| [`architecture.md`](architecture.md) | Definición formal de los 4 anillos de la Arquitectura Onion, límites de dependencia y puerto de ensamblaje (`Bootstrap`). |
| [`backend.md`](backend.md) | Especificación técnica del backend: entidades puras, Value Objects con `BigDecimal`, puertos Inbound/Outbound, Eloquent Mappers y transacciones. |
| [`frontend.md`](frontend.md) | Especificación del frontend en React: arquitectura por capas, features (auth, catálogo, carrito, reporte) y consumo del API. |
| [`docker.md`](docker.md) | Especificación de contenedores e infraestructura: servicio MySQL 8.4 LTS, red, volúmenes y cumplimiento del Artículo V. |
| [`migration.md`](migration.md) | Guía de traducción conceptual: cómo se adaptan las decisiones del spec base (Python/.NET Hexagonal) a Laravel Onion. |
| [`tasks.md`](tasks.md) | Plan de tareas por fases con criterios de aceptación y evidencias de verificación (T-01 a T-12). |
| [`api-contract.md`](api-contract.md) | Contrato formal de la API HTTP: rutas, verbos, formatos JSON (CamelCase), y códigos de error (400, 409, 422). |
| [`adr/`](adr/) | Registros de Decisiones de Arquitectura (ADR-001 a ADR-006). |
