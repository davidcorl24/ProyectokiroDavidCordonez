# Requirements Document

## Introduction

Esta SPEC define el inventario por lote y caducidad, la selección FEFO y el registro consistente de entregas a familias beneficiarias. El inventario debe reflejar las donaciones recibidas y cada entrega debe actualizar existencias e historial sin dejar cambios parciales.

## Glossary

- **Lote**: Cantidad de un alimento recibida en una donación, con una fecha de caducidad propia.
- **FEFO**: First Expired, First Out; se priorizan los lotes que caducan primero.
- **Período activo**: Semana de distribución indicada en `configuracion/global.periodoActivoId`.
- **Entrega**: Registro de una distribución de artículos a una familia durante un período.
- **Movimiento**: Registro inmutable de una entrada o salida de inventario y su motivo.

## Criterios de aceptación

### Requisito 1: Inventario y caducidades

1. WHEN se registra una donación, THE Sistema SHALL conservar sus artículos, cantidades y caducidades, y crear un lote separado en `almacen` por cada artículo.
2. WHEN se consulta el almacén, THE Sistema SHALL mostrar los lotes ordenados por fecha de caducidad ascendente y cantidad disponible.
3. IF un lote tiene cantidad cero, THEN THE Sistema SHALL mostrarlo como "Sin stock" y no sugerirlo para entrega.
4. IF un lote ya caducó, THEN THE Sistema SHALL etiquetarlo "Caducado", diferenciarlo visualmente y bloquear su entrega.
5. IF un lote vence dentro de los próximos 7 días y aún no caducó, THEN THE Sistema SHALL etiquetarlo "Próximo a caducar"; si vence en 3 días o menos, SHALL mostrar además la prioridad "Urgente".
6. WHEN cambia la fecha o cantidad disponible, THE Sistema SHALL recalcular el estado y actualizar métricas/alertas sin conservar etiquetas obsoletas.
7. WHEN el stock queda por debajo de un umbral configurado, THE Sistema SHALL mostrar el alimento y la cantidad restante en el dashboard.

### Requisito 2: Preparación y registro de entregas FEFO

1. WHEN se prepara una entrega, THE Sistema SHALL permitir buscar una familia por nombre o identificador y mostrar si ya recibió entrega en el período activo.
2. WHEN hay lotes disponibles y vigentes, THE Sistema SHALL sugerir primero los lotes con fecha de caducidad más próxima.
3. IF no hay existencias vigentes del alimento, THEN THE Sistema SHALL avisar que no hay stock y SHALL NOT incluir el lote en la sugerencia.
4. WHEN se confirma una entrega, THE Sistema SHALL actualizar en una transacción el stock de todos los artículos, el estado/período de la familia, los movimientos y el `Registro_Entrega`.
5. THE Sistema SHALL rechazar cantidades inválidas, cantidades superiores al stock disponible y lotes caducados; ningún error SHALL dejar solo una parte de la operación guardada.
6. IF la familia ya recibió una entrega en el período activo, THEN solo un coordinador o administrador podrá autorizar otra; la autorización se validará en la UI, la transacción y las reglas Firestore.
7. THE Sistema SHALL marcar la familia como pendiente al cambiar `periodoActivoId` y conservar los registros de períodos anteriores.
8. WHEN una entrega termina, THE Sistema SHALL mostrar un mensaje de confirmación y actualizar el avance de familias en un máximo de 30 segundos.
9. WHEN se prepara una entrega, THE Sistema SHALL permitir indicar por separado la cantidad de cada artículo seleccionado, limitada a las unidades disponibles; solo esas cantidades SHALL descontarse del almacén y guardarse en el registro de entrega.
10. WHEN se registra una entrega, THE Sistema SHALL generar un comprobante imprimible para la familia con folio, fecha, artículos y cantidades; este comprobante SHALL distinguirse del recibo fiscal de donación.

### Requisito 3: Movimientos y trazabilidad

1. WHEN ingresan o salen artículos, THE Sistema SHALL crear un movimiento inmutable con fecha/hora, lote, cantidad, motivo y usuario responsable.
2. THE Sistema SHALL distinguir al menos estos motivos: `donacion_recibida`, `entrega` y `descarte_caducidad`.
3. WHEN un coordinador solicita un informe de descartes, THE Sistema SHALL listar los lotes descartados por caducidad dentro de un período seleccionable de 1 a 365 días.

## Métricas de éxito

1. **Integridad de stock**: 0 cantidades negativas y 0 discrepancias entre movimientos confirmados y stock en pruebas concurrentes.
2. **FEFO**: 100% de sugerencias ordenadas de menor a mayor fecha de caducidad y 0 lotes caducados sugeridos.
3. **Atomicidad**: 100% de fallos simulados durante una entrega dejan sin cambios el stock, la familia y el historial.
4. **Unicidad por período**: 0 segundas entregas no autorizadas por familia/período.
5. **Trazabilidad**: 100% de entradas, entregas y descartes probados generan exactamente un movimiento por lote afectado.
6. **Actualización**: 95% de cambios confirmados aparecen en las métricas de otra sesión en menos de 5 segundos con conexión estable.
7. **Caducidad**: 100% de lotes de prueba cambian a "Caducado" después de su fecha y quedan bloqueados para entrega.

## Fuera de alcance

- Migrar automáticamente inventario ficticio de la demo a Firestore.
- Emitir documentos fiscales o notificaciones push; esos módulos tienen SPEC independientes.
- Reglas de valoración monetaria o certificación legal de donaciones.
