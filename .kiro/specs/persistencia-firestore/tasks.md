# Implementation Plan: Persistencia Firestore y permisos por rol

## Tasks

- [x] 1. Añadir configuración Firebase del proyecto y carga de Firestore con caché persistente.
- [x] 2. Aislar los datos ficticios en modo demo y cargar las colecciones operativas mediante snapshots.
- [x] 3. Persistir las operaciones existentes de voluntarios, donaciones, almacén, estado de familias y recibos.
- [x] 4. Usar transacciones para folios de recibos, descuentos de stock y actualización del estado de entrega por período activo.
- [x] 5. Añadir reglas de acceso por rol y denegación por defecto para colecciones no configuradas.
- [ ] 6. Instalar o habilitar Firebase CLI y compilar las reglas localmente.
- [ ] 7. Crear o avanzar `configuracion/global.periodoActivoId` como administrador y probar persistencia, permisos, desconexión y concurrencia con Firebase Emulator Suite.
- [ ] 8. Configurar custom claims mediante Admin SDK, autorizar dominios de acceso y desplegar las reglas revisadas.
- [ ] 9. Migrar datos reales únicamente si el propietario confirma su origen y formato; no importar datos de demostración.

## Verification

Antes del despliegue, verificar los criterios de aceptación y las métricas de éxito definidos en `requirements.md`. No desplegar reglas sin probar acceso positivo y negativo para cada rol.
