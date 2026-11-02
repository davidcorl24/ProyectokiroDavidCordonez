# Requirements Document

## Introduction

Esta SPEC define la emisión, consulta, anulación y corrección de recibos fiscales digitales por donante para el Banco de Alimentos. La generación se autoriza exclusivamente al administrador y los documentos se conservan en Firestore.

## Glossary

- **Administrador**: Usuario con custom claim `rol` o `role` igual a `administrador`.
- **Recibo fiscal**: Documento PDF que acredita las donaciones de un donante durante un período.
- **Folio**: Identificador único secuencial por año, con formato `AÑO-NNN`.
- **Recibo corregido**: Nuevo recibo emitido después de anular uno previo, con referencia al folio anterior.

## Criterios de aceptación

### Requisito 1: Selección y generación del recibo

1. WHEN un administrador selecciona un donante y un período, THE Sistema SHALL generar un recibo PDF con nombre del donante, fecha de cada donación, artículos y cantidades, nombre y dirección del banco y folio.
2. IF falta información obligatoria del donante, período o banco, THEN THE Sistema SHALL explicar qué campo falta y SHALL NOT emitir el recibo ni consumir un folio.
3. THE Sistema SHALL generar el PDF con una herramienta gratuita y permitir su descarga desde el navegador.
4. WHEN se vuelve a descargar un recibo existente, THE Sistema SHALL conservar el mismo contenido y folio; la descarga SHALL NOT emitir un recibo adicional.

### Requisito 2: Permisos y folios

1. THE Sistema SHALL permitir generar, consultar, descargar, anular y corregir recibos únicamente a usuarios con rol administrador.
2. Firestore SHALL denegar operaciones sobre recibos y contadores a otros roles, aunque se altere el estado de la interfaz cliente.
3. WHEN se emite un recibo, THE Sistema SHALL asignar un folio único y secuencial por año en formato `AÑO-NNN` mediante una transacción.
4. WHEN varios administradores emiten recibos al mismo tiempo, THE Sistema SHALL evitar folios duplicados.
5. THE Sistema SHALL conservar el folio del recibo anulado en el registro de corrección y SHALL NOT borrar recibos emitidos.

### Requisito 3: Historial y correcciones

1. IF una donación incluida en un recibo vigente se modifica, THEN THE Sistema SHALL marcar como anulado el recibo asociado y permitir emitir uno corregido que muestre el folio anterior.
2. THE Sistema SHALL mantener un historial filtrable por donante y año.
3. WHEN el administrador consulta el historial, THE Sistema SHALL mostrar folio, donante, fecha de emisión, estado y enlace de descarga.

## Métricas de éxito

1. **Control de acceso**: 100% de las pruebas de acceso no administrador son rechazadas por las reglas Firestore.
2. **Unicidad**: 0 folios duplicados en pruebas concurrentes del Firebase Emulator Suite.
3. **Exactitud del PDF**: 100% de PDFs de prueba incluyen los datos del recibo persistido y el folio correcto.
4. **Descarga repetible**: múltiples descargas del mismo recibo producen 0 registros nuevos y mantienen el folio original.
5. **Validación previa**: 100% de solicitudes con datos obligatorios incompletos se rechazan antes de reservar un folio.
6. **Disponibilidad**: 95% de recibos autorizados se generan y descargan correctamente en pruebas de staging.

## Restricciones y decisiones pendientes

- Revisar con la jurisdicción aplicable si el contenido del recibo satisface los requisitos fiscales locales; esta SPEC no constituye certificación legal.
- No incluir credenciales privilegiadas en el cliente. Firestore y las reglas de seguridad son la frontera de autorización.
