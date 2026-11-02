# Implementation Plan: Alertas y notificaciones automáticas

## Tasks

- [ ] 1. Confirmar backend/emisor push y coste de operación.
- [ ] 2. Definir tipos, esquema Firestore, claves idempotentes y preferencias por evento.
- [ ] 3. Implementar detección para los cuatro eventos críticos.
- [ ] 4. Implementar agrupación por tipo en ventanas de 10 minutos con conteo y referencias.
- [ ] 5. Implementar centro de notificaciones limitado a 50 y ordenado por fecha.
- [ ] 6. Implementar banners no bloqueantes con navegación contextual.
- [ ] 7. Implementar acciones leído/descartado y preferencias de eventos.
- [ ] 8. Implementar reglas Firestore para lectura admin y escritura servidor.
- [ ] 9. Implementar permiso, suscripción, revocación y recepción Web Push en Service Worker.
- [ ] 10. Añadir manejo de tokens caducados y fallos de push manteniendo el aviso in-app.
- [ ] 11. Probar idempotencia, agrupación, permisos, latencia y desconexión en Emulator Suite/staging.
- [ ] 12. Validar métricas y desplegar tras confirmar proveedor, cuotas y coste.

## Verification

Antes del despliegue, comprobar los cuatro eventos críticos, los accesos denegados, los casos de agrupación/reintento y el fallback in-app cuando Web Push no está disponible.
