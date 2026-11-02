# Design Document

## Overview

El módulo ofrece un formulario de emisión, un historial administrativo y descarga del PDF. El folio se reserva y el recibo se guarda en Firestore antes de generar la descarga; así, volver a descargar no emite otro documento.

## Componentes

| Componente | Responsabilidad |
|---|---|
| `GeneradorRecibo` | Seleccionar donante y período, validar campos y solicitar emisión |
| `HistorialRecibos` | Filtrar por donante/año y consultar estados |
| `VisorRecibo` | Revisar, descargar, anular y generar correcciones |
| `recibosService` | Transacción de folio, persistencia, consultas y correcciones |
| Generador PDF | Crear el documento a partir de los datos persistidos |

## Datos Firestore

- `recibos/{folio}`: folio, donante, fechas, donaciones incluidas, artículos/cantidades, datos del banco, fecha de emisión, estado, folio anterior y UID emisor.
- `contadores/recibos-AAAA`: siguiente número disponible del año.

La operación de emisión lee el contador, comprueba colisiones y escribe contador/recibo en una única transacción. Los recibos emitidos no se borran; una corrección crea un nuevo recibo y conserva la referencia previa.

## Seguridad

- Lectura y escritura de `recibos` y `contadores` solo para el claim `administrador`.
- Borrado de recibos denegado.
- Comprobaciones de cliente mejoran la UX, pero las reglas Firestore deciden el acceso.
- El folio se asigna en transacción para soportar concurrencia.

## Consideraciones

El PDF puede generarse en cliente con una librería gratuita; el registro y folio permanecen en Firestore. Antes de uso oficial, validar los campos requeridos para la jurisdicción fiscal correspondiente.
