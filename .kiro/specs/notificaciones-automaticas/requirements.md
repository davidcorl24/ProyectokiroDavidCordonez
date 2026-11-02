# Requirements Document

## Introduction

Esta SPEC define la detección, agrupación, persistencia y presentación de alertas automáticas para administradores del Banco de Alimentos. Las notificaciones dentro de la app funcionan independientemente de que el administrador habilite Web Push.

## Glossary

- **Administrador**: Usuario con custom claim `rol` o `role` igual a `administrador`.
- **Evento crítico**: Escasez de stock, caducidad urgente, segunda entrega autorizada o conflicto de sincronización.
- **Notificación**: Registro persistente de un evento que requiere atención.
- **Notificación agrupada**: Resumen de eventos del mismo tipo ocurridos dentro de una ventana de 10 minutos.
- **Web Push**: Notificación del navegador que puede mostrarse con la app en segundo plano.

## Criterios de aceptación

### Requisito 1: Detección y almacenamiento de eventos

1. WHEN ocurre un evento crítico, THE Sistema SHALL crear en Firestore una notificación con tipo, descripción, fecha/hora, estado y referencia al recurso afectado.
2. THE Sistema SHALL cubrir estos eventos: (a) stock bajo el umbral configurado, (b) caducidad en 3 días o menos, (c) segunda entrega autorizada por coordinación, (d) conflicto de sincronización offline.
3. THE Sistema SHALL evitar notificaciones duplicadas cuando el mismo evento se reintenta o se procesa más de una vez.
4. THE Sistema SHALL conservar las últimas 50 notificaciones ordenadas de más reciente a más antigua.

### Requisito 2: Centro y banners

1. WHEN el administrador tiene la aplicación abierta, THE Sistema SHALL mostrar un banner no bloqueante con descripción y acción "Ver detalle" que navega al recurso afectado.
2. THE Sistema SHALL ofrecer un centro de notificaciones accesible desde la barra superior con indicador de no leídas.
3. WHEN el administrador marca una notificación como leída o la descarta, THE Sistema SHALL persistir el nuevo estado y reflejarlo en las demás sesiones autorizadas.
4. WHEN eventos del mismo tipo ocurren dentro de 10 minutos, THE Sistema SHALL agruparlos en una sola notificación con conteo exacto y referencias afectadas.
5. THE Sistema SHALL permitir activar o desactivar cada tipo de evento desde configuración.

### Requisito 3: Web Push y seguridad

1. IF el administrador habilita las notificaciones del navegador y existe una suscripción válida, THEN THE Sistema SHALL permitir enviar Web Push mediante el Service Worker aunque la aplicación no esté en primer plano.
2. IF el permiso no se concede, la suscripción caduca o el proveedor push falla, THEN THE Sistema SHALL conservar la notificación in-app y explicar que el push no está disponible.
3. Firestore SHALL permitir leer notificaciones solo a administradores; la creación y el envío SHALL realizarse desde un proceso confiable, nunca con secretos dentro del navegador.
4. THE Sistema SHALL permitir al administrador consultar o revocar la suscripción push del dispositivo.

## Métricas de éxito

1. **Cobertura**: 100% de los cuatro eventos críticos definidos generan una notificación en pruebas automatizadas.
2. **Latencia in-app**: 95% de los eventos de prueba aparecen en el centro en menos de 30 segundos con conexión estable.
3. **Agrupación**: 100% de los casos elegibles en una ventana de 10 minutos se consolidan una sola vez con conteo correcto.
4. **Duplicación**: 0 notificaciones duplicadas después de reintentos del mismo evento.
5. **Estado compartido**: 95% de actualizaciones leído/descartado se reflejan en otras sesiones en menos de 5 segundos.
6. **Acceso**: 100% de intentos de lectura desde roles no administradores se deniegan en las reglas Firestore.
7. **Resiliencia push**: un fallo o falta de permiso push deja intacta la notificación in-app en el 100% de las pruebas.
8. **Entrega push**: con permiso, token vigente y proveedor disponible, 95% de envíos de staging llegan al proveedor en menos de 60 segundos.

## Restricciones y decisiones pendientes

- El envío FCM requiere un emisor confiable. Confirmar plan y coste de despliegue antes de elegir Cloud Functions u otro backend; no asumir que el envío automatizado puede ejecutarse desde el nivel gratuito.
- No colocar claves de servidor, credenciales Admin SDK ni tokens privados en el cliente.
