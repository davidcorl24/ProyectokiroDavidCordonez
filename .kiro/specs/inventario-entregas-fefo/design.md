# Design Document

## Overview

Cada donación crea lotes de inventario independientes para conservar su caducidad. Las entregas aplican FEFO y se registran con transacciones Firestore. La prioridad para vencimientos se calcula al presentar el inventario y el dashboard usa los mismos datos, evitando alertas estáticas.

## Colecciones

| Colección | Contenido | Uso |
|---|---|---|
| `almacen` | ID de lote, donación, nombre, cantidad, caducidad y estado | Inventario disponible por lote |
| `familiasBeneficiarias` | Datos de la familia, período actual, conteo y última fecha | Estado visible de atención en el período |
| `registrosEntrega` | Familia, período, artículos/cantidades, fecha y voluntario | Historial de entregas |
| `movimientosAlmacen` | Lote, cantidad, motivo, fecha y responsable | Trazabilidad de entradas/salidas |
| `umbralesEscasez` | Tipo de alimento y umbral | Alertas de escasez |
| `configuracion/global` | `periodoActivoId` | Fuente autorizada del período semanal |

## Flujos

1. La donación se guarda con sus detalles y lotes de almacén en un batch.
2. El inventario recalcula su clasificación por caducidad y se ordena FEFO al consultar/sugerir.
3. Antes de una entrega, la UI oculta lotes sin stock o caducados; las transacciones vuelven a validar stock y fecha.
4. La transacción de entrega lee familia y lotes, valida período/rol/cantidad, y actualiza stock, movimientos, familia e historial como una sola operación.
5. El estado de entrega se compara con `configuracion/global.periodoActivoId`; los registros previos permanecen como historial.
6. Los cambios concurrentes se resuelven mediante reintentos de transacción; si no se puede completar, no se presenta como guardado.

## Seguridad

- Lectura del inventario y familias para usuarios autenticados.
- Actualización del stock limitada a cantidad y estado, sin cantidades negativas.
- Cambios de período y segunda entrega restringidos por rol y reglas Firestore.
- Movimientos append-only; no se modifican ni borran desde el cliente.
- Las reglas consultan el período activo global. Un administrador lo inicializa/avanza al iniciar sesión.
- Ejecutar Emulator Suite para validar permisos, transacciones y concurrencia antes de despliegue.

## Limitaciones conocidas

La interfaz actual permite seleccionar una unidad por lote en la entrega desde familias; falta un selector de cantidad por lote. Aún faltan pantallas para configurar umbrales, descartar alimentos y generar informes de caducados. Esos puntos permanecen como tareas pendientes.
