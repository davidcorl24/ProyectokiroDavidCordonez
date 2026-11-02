# Implementation Plan: Inventario y entregas FEFO

## Tasks

- [x] 1. Guardar detalles de donación y crear lotes de inventario con cantidad y caducidad.
- [x] 2. Mostrar estado de caducidad actual y ordenar lotes por FEFO.
- [x] 3. Excluir y bloquear lotes caducados o sin stock para entrega.
- [x] 4. Actualizar stock, familia, movimientos e historial en una transacción de entrega.
- [x] 5. Asociar el estado de familia al período activo y requerir rol coordinador para repetir entrega.
- [ ] 6. Añadir selector de cantidad por artículo/lote en el flujo de entrega a familia.
- [ ] 7. Implementar configuración de umbrales y alertas de escasez a partir de datos Firestore.
- [ ] 8. Implementar descarte por caducidad con motivo y movimiento inmutable.
- [ ] 9. Implementar informe de descartes por períodos de 1 a 365 días.
- [ ] 10. Probar reglas, operaciones offline compatibles, concurrencia, FEFO y métricas en Emulator Suite.
- [ ] 11. Desplegar reglas y verificar el período activo con cuentas de cada rol.

## Verification

Antes de producción, comprobar con Emulator Suite que las transacciones no permiten stock negativo ni segundas entregas sin autorización, y que los registros conservan movimientos e historial completos.
