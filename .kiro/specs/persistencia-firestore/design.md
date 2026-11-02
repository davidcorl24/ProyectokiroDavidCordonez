# Design Document

## Overview

La aplicación sigue siendo HTML/JavaScript estático. Firebase Authentication identifica al usuario; Firestore almacena las colecciones operativas y mantiene una caché local persistente. El modo demo permanece aislado y usa únicamente los arreglos de ejemplo en memoria.

## Colecciones

| Colección | Uso | Acceso previsto |
|---|---|---|
| `voluntarios` | Registros de voluntariado | Lectura autenticada; gestión por coordinación |
| `donaciones` | Donaciones registradas | Lectura y alta autenticadas; cambios administrativos |
| `almacen` | Cantidad, caducidad y estado | Lectura y movimientos autenticados; cambios de stock limitados por reglas |
| `familiasBeneficiarias` | Estado actual de atención | Lectura y alta autenticadas; eliminación por coordinación; cambios de entrega controlados |
| `recibos` | Recibos fiscales | Administrador; sin borrado |
| `contadores` | Secuencia anual de folios | Administrador, transacción |
| `configuracion/global` | Identificador del período de distribución activo | Lectura autenticada; actualización por administrador |
| `registrosEntrega` | Registros futuros de entregas | Lectura y escritura conforme al claim y las reglas |
| `movimientosAlmacen` | Historial futuro de movimientos | Lectura y alta autenticadas; sin modificación ni borrado |
| `herramientasDigitales` | Catálogo de herramientas | Lectura autenticada; gestión por coordinación |
| `umbralesEscasez` | Umbrales de inventario | Lectura autenticada; gestión por coordinación |
| `notificaciones` | Alertas generadas por sistema | Lectura por administradores; escritura de servidor |

## Flujos

1. Tras validar el usuario, el cliente obtiene el token Firebase y lee `rol` o `role`; si no existe, aplica `voluntario`.
2. La pantalla de acceso permite abrir el selector de cuentas de Google aunque exista una sesión persistente; la expiración local se guarda por UID.
3. Las colecciones permitidas se suscriben mediante `onSnapshot`. Las vistas se actualizan con datos iniciales, cambios remotos y escrituras optimistas de Firestore.
4. Las escrituras documentales usan caché persistente. Las operaciones que requieren consistencia concurrente usan transacciones y necesitan conexión.
5. Al iniciar sesión, un administrador inicializa o avanza `configuracion/global.periodoActivoId`; las reglas validan las entregas contra ese valor.
6. Los folios se reservan leyendo y avanzando `contadores/recibos-AÑO` dentro de una transacción; si el folio propuesto ya existe, se avanza hasta encontrar uno disponible.
7. Las reglas de `firestore.rules` son la frontera de seguridad. Las comprobaciones de interfaz solo mejoran la experiencia y no sustituyen las reglas.
8. En `?demo=1` no se inicializa el adaptador de datos Firestore.

## Seguridad y despliegue

- `firebase.json` apunta a `firestore.rules` y `.firebaserc` selecciona `banco-alimentos-barrio` como proyecto por defecto.
- Los custom claims se asignan mediante Admin SDK, nunca desde la aplicación cliente.
- El despliegue de reglas requiere revisión y validación con Firebase CLI/Emulator Suite.
- Las colecciones sin regla explícita quedan denegadas por defecto.

## Limitaciones conocidas

- La aplicación no tiene todavía UI para crear usuarios, movimientos de almacén o registros históricos completos de entregas.
- Los datos de muestra no se migran automáticamente y la persistencia real requiere iniciar sesión con un usuario del proyecto.
- Las reglas cubren los documentos actuales; deberán ampliarse y probarse al conectar nuevas operaciones o colecciones.
