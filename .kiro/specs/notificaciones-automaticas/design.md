# Design Document

## Overview

Los eventos del dominio producen notificaciones idempotentes en Firestore. El cliente autenticado con rol administrador presenta los banners y el centro de notificaciones; un proceso servidor confiable consolida eventos y, si está habilitado, solicita el envío Web Push.

## Componentes

| Componente | Responsabilidad |
|---|---|
| `notificacionesService` | Crear eventos idempotentes, consultar y actualizar estado |
| `CentroNotificaciones` | Mostrar hasta 50 registros ordenados y estado de lectura |
| `BannerNotificacion` | Aviso no bloqueante con acción contextual |
| `PreferenciasNotificaciones` | Configurar eventos habilitados |
| Servicio servidor | Validar eventos, agruparlos y solicitar envío push |
| Service Worker | Recibir y mostrar Web Push con la app en segundo plano |

## Eventos y datos

Eventos: `escasez_stock`, `caducidad_urgente`, `segunda_entrega_coordinador` y `conflicto_sincronizacion`.

`notificaciones/{id}` guarda tipo, descripción, fecha/hora, estado (`no_leida`, `leida`, `descartada`), referencia de colección/documento, indicador de agrupación, conteo y clave idempotente. `configuracion/notificaciones` guarda preferencias del administrador. Las suscripciones push se guardan por usuario/dispositivo, con acceso limitado al propietario y al proceso servidor.

## Flujo

1. Un evento de dominio se valida y crea con una clave idempotente.
2. El proceso de agrupación combina eventos del mismo tipo dentro de 10 minutos y mantiene el conteo/referencias.
3. Firestore distribuye cambios del centro de notificaciones a las sesiones autorizadas.
4. El administrador puede marcar o descartar; el estado se persiste.
5. Web Push es opcional: solo se intenta con permiso y token vigente; el fallo no afecta el registro in-app.

## Seguridad y operación

- Lectura Firestore solo para administradores; escrituras de notificaciones desde servidor confiable.
- El cliente solicita permiso de forma explícita y permite revocar la suscripción.
- No almacenar secretos de FCM/Admin SDK en archivos frontend.
- Elegir proveedor/backend tras validar coste, cuota y operación; probar reglas y eventos en Emulator Suite antes del despliegue.
