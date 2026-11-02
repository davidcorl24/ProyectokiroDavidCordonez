# Requirements Document

## Introduction

Esta SPEC define la persistencia de los datos operativos del Banco de Alimentos en Firebase Firestore, la sincronización entre sesiones, el soporte offline disponible y el control de acceso basado en roles. El modo demo conserva datos ficticios locales y no debe escribir en el proyecto Firebase.

## Glossary

- **Usuario autenticado**: Persona cuya sesión ha sido validada por Firebase Authentication.
- **Voluntario**: Usuario con el claim `rol` o `role` igual a `voluntario`.
- **Coordinador**: Usuario con el claim `rol` o `role` igual a `coordinador`.
- **Administrador**: Usuario con el claim `rol` o `role` igual a `administrador`.
- **Modo demo**: Acceso explícito mediante `?demo=1`, con datos de ejemplo exclusivamente locales.
- **Escritura offline**: Cambio aceptado por la caché local de Firestore que queda pendiente de sincronización.

## Criterios de aceptación

### Requisito 1: Persistencia y sincronización

1. WHEN un usuario autenticado abre la aplicación, THE Sistema SHALL cargar desde Firestore las colecciones `voluntarios`, `donaciones`, `almacen` y `familiasBeneficiarias`; solo un administrador SHALL cargar `recibos`.
2. WHEN una operación crea o actualiza un voluntario, donación, artículo de almacén o familia beneficiaria, THE Sistema SHALL guardar el cambio en la colección correspondiente y conservarlo tras recargar la página.
3. WHEN se genera un recibo, THE Sistema SHALL persistirlo en `recibos` y actualizar en una transacción el contador anual de `contadores`.
4. THE Sistema SHALL generar folios únicos con formato `AÑO-NNN`, incluso cuando dos administradores creen recibos concurrentemente.
5. WHEN una escritura compatible con la caché offline se confirma localmente sin conexión, THE Sistema SHALL mantenerla pendiente y sincronizarla al recuperar conexión; un error posterior de permisos SHALL informar al usuario y revertir la vista a los datos autorizados del servidor.
6. WHEN el modo demo está activo, THE Sistema SHALL usar solo datos de ejemplo locales y SHALL NOT leer ni escribir datos de Firestore.
7. IF Firestore no está disponible o rechaza una operación, THEN THE Sistema SHALL mostrar un mensaje comprensible y no presentar una operación rechazada como guardada.

### Requisito 2: Integridad de inventario y entregas

1. WHEN se descuenta stock, THE Sistema SHALL actualizar cantidad y estado en una transacción, rechazar cantidades superiores al stock disponible y evitar cantidades negativas.
2. WHEN una familia ya tiene una entrega en el período, THE Sistema SHALL permitir una segunda actualización únicamente a un coordinador o administrador autenticado; el permiso SHALL comprobarse también en las reglas de Firestore.
3. THE Sistema SHALL mantener el identificador del período activo en `configuracion/global`; al iniciar sesión, un administrador SHALL inicializarlo o avanzarlo a la semana actual.
4. WHEN se registra una entrega, THE Sistema SHALL esperar a cargar `configuracion/global` y guardar el identificador de su período activo en la familia y en el registro; las reglas SHALL rechazar entregas para períodos distintos y el cliente SHALL NOT sustituir un período aún no cargado por la fecha local.
5. IF una transacción de stock o entrega no puede completarse, THEN THE Sistema SHALL conservar el estado confirmado previamente y mostrar un mensaje de error.

### Requisito 3: Autorización en Firestore

1. THE Sistema SHALL exigir una sesión autenticada para leer las colecciones operativas definidas en las reglas.
2. THE Sistema SHALL restringir la lectura, creación y actualización de `recibos` y el uso de contadores de folio al administrador; SHALL NOT permitir borrar recibos.
3. THE Sistema SHALL restringir la gestión de voluntarios, herramientas y umbrales a coordinadores o administradores; cualquier usuario autenticado SHALL poder registrar familias, mientras que solo coordinadores o administradores SHALL poder eliminarlas.
4. THE Sistema SHALL restringir los cambios de inventario a los campos de cantidad y estado, con cantidad entera no negativa.
5. THE Sistema SHALL denegar por defecto las colecciones que no tengan reglas explícitas.
6. WHEN una cuenta no autorizada intenta acceder a recibos o modificar un recurso restringido, THEN Firestore SHALL denegar la operación aunque el cliente modifique la interfaz.

### Requisito 4: Acceso con varias cuentas

1. WHEN una persona ya tiene una cuenta de Google activa, THE Sistema SHALL permitir elegir otra cuenta desde la pantalla de acceso sin redirigirla automáticamente a la cuenta anterior.
2. THE Sistema SHALL mantener la expiración de sesión de cada usuario separada por su identificador de Firebase Authentication.

## Métricas de éxito

1. **Retención de datos**: 100% de las escrituras válidas de cada colección operativa probadas en Firebase Emulator SHALL seguir disponibles tras recargar la aplicación y volver a iniciar sesión.
2. **Sincronización online**: 95% de las escrituras aceptadas SHALL aparecer en otra sesión autorizada en menos de 5 segundos bajo una conexión estable.
3. **Recuperación offline**: 100% de las escrituras offline admitidas por Firestore SHALL sincronizarse una sola vez tras reconectar; los casos rechazados por reglas SHALL mostrar error y no quedar como confirmados.
4. **Control de acceso**: 100% de las pruebas negativas de rol SHALL ser denegadas por Firestore; 100% de las pruebas positivas SHALL permitir únicamente las operaciones definidas para ese rol.
5. **Integridad transaccional**: 0 folios duplicados, 0 cantidades negativas y 0 segundas entregas no autorizadas en pruebas concurrentes.
6. **Modo demo**: 0 lecturas y 0 escrituras de Firestore al usar `?demo=1`.
7. **Calidad de validación**: todas las reglas SHALL compilar con Firebase CLI y los criterios críticos SHALL contar con pruebas del Emulator Suite antes de desplegar.

## Fuera de alcance

- Migración automática de los datos ficticios del modo demo a Firestore.
- Historial completo de movimientos y entregas cuando la interfaz no genera todavía esos registros.
- Asignación de custom claims desde el cliente; los claims se asignan con Firebase Admin SDK en un entorno confiable.
