# Implementation Plan: Recibos fiscales por donante

## Tasks

- [ ] 1. Confirmar con la organización y la jurisdicción los campos requeridos en el recibo.
- [ ] 2. Definir tipos y esquema Firestore para recibos y contadores anuales.
- [ ] 3. Implementar transacción de folio único y secuencial, incluyendo reintentos concurrentes.
- [ ] 4. Implementar emisión persistida, anulación y corrección con referencia al folio anterior.
- [ ] 5. Implementar generación PDF gratuita desde los datos guardados.
- [ ] 6. Crear formulario y previsualización para administradores.
- [ ] 7. Crear historial filtrable por donante/año y descarga repetible.
- [ ] 8. Añadir reglas Firestore y pruebas de acceso por rol.
- [ ] 9. Probar datos incompletos, descargas repetidas, correcciones y concurrencia en Emulator Suite.
- [ ] 10. Validar las métricas de éxito en staging antes del despliegue.

## Verification

No desplegar hasta que los accesos no administrativos sean rechazados, los folios concurrentes sean únicos y el contenido del PDF haya sido revisado para la jurisdicción aplicable.
