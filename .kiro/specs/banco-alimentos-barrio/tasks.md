# Implementation Plan: Sistema de Gestión del Banco de Alimentos de Barrio

## Overview

Implementación incremental de la PWA (React + Vite + TypeScript + Firebase Firestore + Tailwind CSS) que cubre los seis módulos funcionales (Voluntarios, Donaciones, Almacén, Entregas, Catálogo y Dashboard) más la capa de accesibilidad y usabilidad. Cada tarea construye sobre las anteriores y termina con integración; no queda código huérfano.

---

## Tasks

- [~] 1. Configurar el proyecto base y la infraestructura Firebase
  - Inicializar proyecto Vite + React + TypeScript con plugin PWA (vite-plugin-pwa / Workbox)
  - Instalar y configurar Tailwind CSS con preflight
  - Instalar Firebase SDK, Vitest, React Testing Library y fast-check
  - Crear `src/firebase/config.ts` con las claves públicas del proyecto Firebase
  - Crear `src/firebase/persistence.ts` que habilite la persistencia offline de Firestore
  - Crear `src/main.tsx` como punto de entrada que inicialice Firebase y renderice la SPA
  - Configurar `vitest.config.ts` con `environment: 'jsdom'` y `setupFiles`
  - _Requisitos: 7.5, 7.6_

- [ ] 2. Implementar tipos, interfaces y utilidades compartidas
  - [~] 2.1 Definir todas las interfaces TypeScript del diseño (`Voluntario`, `Donacion`, `Articulo`, `FamiliaBeneficiaria`, `RegistroEntrega`, `PeriodoDistribucion`, `HerramientaDigital`, `MetricasDashboard`, `IndicadorConfig`, `MovimientoAlmacen`, `UmbralEscasez`, tipos auxiliares)
    - Ubicar en `src/shared/types/index.ts`
    - _Requisitos: 1.1, 2.1, 3.1, 4.1, 5.1, 6.2_
  - [~] 2.2 Implementar `src/shared/utils/fefo.ts` con la función `ordenarFEFO`
    - _Requisitos: 3.2, 3.5_
  - [ ]* 2.3 Escribir test de propiedad para `ordenarFEFO`
    - **Propiedad 11: Ordenación FEFO del inventario y sugerencia de entrega**
    - **Valida: Requisitos 3.2, 3.5**
  - [~] 2.4 Implementar `src/shared/utils/fechas.ts` con la función `clasificarEstadoCaducidad`
    - _Requisitos: 3.3, 3.4, 6.4_
  - [ ]* 2.5 Escribir test de propiedad para `clasificarEstadoCaducidad`
    - **Propiedad 12: Clasificación correcta del estado de caducidad de un artículo**
    - **Valida: Requisitos 3.3, 3.4, 6.4**
  - [~] 2.6 Implementar `src/shared/utils/validaciones.ts` con las funciones de validación de formularios (`validarNotas`, validación de campos requeridos y rangos, mensajes de error por campo)
    - _Requisitos: 1.8, 7.4_
  - [ ]* 2.7 Escribir test de propiedad para validación de notas
    - **Propiedad 6: La validación de notas rechaza textos de más de 500 caracteres**
    - **Valida: Requisitos 1.8**
  - [ ]* 2.8 Escribir test de propiedad para mensajes de error de formularios
    - **Propiedad 28: La validación de formularios retorna mensajes de error específicos por campo**
    - **Valida: Requisitos 7.4**

- [ ] 3. Implementar componentes compartidos de UI
  - [~] 3.1 Crear `src/shared/components/BotonPrimario.tsx` (mínimo 44×44 px, etiqueta de texto obligatoria, sin dependencia de solo icono)
    - _Requisitos: 7.2_
  - [~] 3.2 Crear `src/shared/components/MensajeConfirmacion.tsx` (texto cotidiano tras guardar/borrar/entregar)
    - _Requisitos: 7.3_
  - [~] 3.3 Crear `src/shared/components/MensajeError.tsx` (indica campo incorrecto y cómo corregirlo, sin códigos técnicos)
    - _Requisitos: 7.4_
  - [~] 3.4 Crear `src/shared/hooks/useOfflineStatus.ts` (suscripción a `navigator.onLine` + eventos `online`/`offline`)
    - _Requisitos: 2.6, 2.7_
  - [~] 3.5 Crear `src/shared/hooks/useFirestore.ts` (wrapper tipado de lecturas/escrituras Firestore con manejo de errores estandarizado)
    - _Requisitos: 2.3, 3.7_
  - [~] 3.6 Crear `src/shared/hooks/useSessionTimeout.ts` (detección de inactividad 30 min → aviso; 35 min → cierre de sesión + guardado de borrador en IndexedDB)
    - _Requisitos: 7.7_
  - [ ]* 3.7 Escribir test de propiedad para preservación del borrador al cerrar sesión por inactividad
    - **Propiedad 29: Preservación del borrador de formulario al cerrar sesión por inactividad**
    - **Valida: Requisitos 7.7**

- [~] 4. Checkpoint — Verificar infraestructura base
  - Asegurarse de que todos los tests pasan, que el proyecto compila sin errores TypeScript y que la configuración Firebase está conectada. Preguntar al usuario si surgen dudas.

- [ ] 5. Implementar módulo de Voluntarios
  - [~] 5.1 Implementar `src/features/voluntarios/voluntariosService.ts` con las funciones `detectarDuplicado`, `filtrarPorNivel`, `validarNotas` y el CRUD de voluntarios en Firestore
    - _Requisitos: 1.1, 1.2, 1.6, 1.7, 1.8_
  - [ ]* 5.2 Escribir test de propiedad para round-trip de persistencia de voluntarios
    - **Propiedad 1: Round-trip de persistencia de voluntarios**
    - **Valida: Requisitos 1.1, 1.2**
  - [ ]* 5.3 Escribir test de propiedad para unicidad de identificadores de voluntarios
    - **Propiedad 2: Unicidad de identificadores de voluntarios**
    - **Valida: Requisitos 1.2**
  - [ ]* 5.4 Escribir test de propiedad para detección de duplicados
    - **Propiedad 4: Detección de duplicados de voluntarios**
    - **Valida: Requisitos 1.6**
  - [ ]* 5.5 Escribir test de propiedad para filtrado por nivel tecnológico
    - **Propiedad 5: Filtrado de voluntarios por nivel tecnológico**
    - **Valida: Requisitos 1.7**
  - [~] 5.6 Crear `src/features/voluntarios/FormularioVoluntario.tsx` (crear y editar voluntarios; detecta duplicados; botones ≥44 px; mensajes de error no técnicos)
    - _Requisitos: 1.1, 1.2, 1.6, 1.8_
  - [~] 5.7 Crear `src/features/voluntarios/ListaVoluntarios.tsx` (tabla filtrable por nivel tecnológico; etiqueta textual del nivel junto a cada voluntario)
    - _Requisitos: 1.4, 1.5, 1.7_
  - [~] 5.8 Crear `src/features/voluntarios/SemanaVoluntarios.tsx` (vista semanal: voluntarios activos, nombre, nivel y tareas asignadas; permite actualizar la lista)
    - _Requisitos: 1.3, 1.4, 1.5_
  - [ ]* 5.9 Escribir test de propiedad para el panel semanal
    - **Propiedad 3: El panel semanal refleja nombre y nivel de cada voluntario activo**
    - **Valida: Requisitos 1.4, 1.5**

- [ ] 6. Implementar módulo de Donaciones
  - [~] 6.1 Implementar `src/features/donaciones/donacionesService.ts` con las funciones `validarArticulosCaducados`, `obtenerHistorialDonante`, `ordenarPorFechaDescendente` y el CRUD de donaciones en Firestore
    - _Requisitos: 2.1, 2.4, 2.5, 2.8, 2.9_
  - [ ]* 6.2 Escribir test de propiedad para detección de artículos caducados en donación entrante
    - **Propiedad 7: Detección correcta de artículos caducados en donación entrante**
    - **Valida: Requisitos 2.4**
  - [ ]* 6.3 Escribir test de propiedad para integridad del registro de donación guardada
    - **Propiedad 8: Integridad del registro de donación guardada**
    - **Valida: Requisitos 2.5, 3.1**
  - [ ]* 6.4 Escribir test de propiedad para ordenación descendente del historial
    - **Propiedad 9: Ordenación descendente del historial de donaciones**
    - **Valida: Requisitos 2.8**
  - [ ]* 6.5 Escribir test de propiedad para historial de donante recurrente
    - **Propiedad 10: Historial de donante recurrente devuelve ≤10 registros del donante correcto**
    - **Valida: Requisitos 2.9**
  - [~] 6.6 Crear `src/features/donaciones/AlertaCaducidad.tsx` (modal de confirmación cuando algún artículo ya caducó; requiere confirmación explícita o permite cancelar)
    - _Requisitos: 2.4_
  - [~] 6.7 Crear `src/features/donaciones/HistorialDonante.tsx` (muestra últimas 10 donaciones si el donante tiene ≥2 donaciones previas)
    - _Requisitos: 2.9_
  - [~] 6.8 Crear `src/features/donaciones/WizardDonacion.tsx` (formulario multipaso ≤5 pasos; integra `AlertaCaducidad` e `HistorialDonante`; registra voluntarioReceptorId y timestamp UTC; soporta modo offline)
    - _Requisitos: 2.1, 2.2, 2.3, 2.5, 2.6_
  - [~] 6.9 Crear `src/features/donaciones/ListaDonaciones.tsx` (historial paginado 50 registros/página, ordenado por fecha descendente)
    - _Requisitos: 2.8_

- [~] 7. Checkpoint — Verificar módulos Voluntarios y Donaciones
  - Asegurarse de que todos los tests pasan y que el modo offline funciona para donaciones. Preguntar al usuario si surgen dudas.

- [ ] 8. Implementar módulo de Almacén
  - [~] 8.1 Implementar `src/features/almacen/almacenService.ts` con las funciones `ordenarFEFO` (wrapper del util), `clasificarEstadoCaducidad` (wrapper), `detectarEscasez` y `generarInformeCaducados`; CRUD de artículos y movimientos en Firestore
    - _Requisitos: 3.1, 3.2, 3.5, 3.7, 3.8, 3.9, 3.10_
  - [ ]* 8.2 Escribir test de propiedad para alerta de escasez
    - **Propiedad 13: Alerta de escasez cuando el stock está bajo el umbral**
    - **Valida: Requisitos 3.8, 6.3**
  - [ ]* 8.3 Escribir test de propiedad para round-trip de movimiento de almacén
    - **Propiedad 14: Round-trip de movimiento de almacén**
    - **Valida: Requisitos 3.9**
  - [ ]* 8.4 Escribir test de propiedad para filtrado del informe de artículos caducados
    - **Propiedad 15: Filtrado correcto del informe de artículos caducados**
    - **Valida: Requisitos 3.10**
  - [~] 8.5 Crear `src/features/almacen/EtiquetaCaducidad.tsx` (badge textual: "Urgente" ≤3 días, "Próximo a caducar" 4–7 días, "Caducado"; visualmente diferenciables)
    - _Requisitos: 3.3, 3.4, 6.4_
  - [~] 8.6 Crear `src/features/almacen/MovimientoAlmacen.tsx` (registra entrada / salida / descarte con motivo; actualiza stock en Firestore)
    - _Requisitos: 3.7, 3.9_
  - [~] 8.7 Crear `src/features/almacen/InventarioFEFO.tsx` (lista de artículos ordenada FEFO; muestra etiquetas de caducidad; artículo con stock 0 con aviso; integra `EtiquetaCaducidad` y `MovimientoAlmacen`)
    - _Requisitos: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6_
  - [~] 8.8 Crear `src/features/almacen/InformeCaducados.tsx` (genera listado de descartados por caducidad en rango 1–365 días seleccionable)
    - _Requisitos: 3.10_

- [ ] 9. Implementar módulo de Entregas
  - [~] 9.1 Implementar `src/features/entregas/entregasService.ts` con las funciones `buscarFamilia`, `tieneEntregaEnPeriodo`, `calcularProgresoPeriodo`, `obtenerPendientesUrgentes` y `validarSegundaEntrega`; CRUD de RegistrosEntrega y FamiliasBeneficiarias en Firestore
    - _Requisitos: 4.1, 4.2, 4.4, 4.5, 4.7, 4.9_
  - [ ]* 9.2 Escribir test de propiedad para round-trip de familia beneficiaria
    - **Propiedad 16: Round-trip de persistencia de familia beneficiaria**
    - **Valida: Requisitos 4.1**
  - [ ]* 9.3 Escribir test de propiedad para búsqueda de familia por nombre o identificador
    - **Propiedad 17: Búsqueda de familia por nombre o identificador**
    - **Valida: Requisitos 4.2**
  - [ ]* 9.4 Escribir test de propiedad para unicidad de entrega por familia y período
    - **Propiedad 18: Unicidad de entrega por familia y período sin intervención del coordinador**
    - **Valida: Requisitos 4.4, 4.9**
  - [ ]* 9.5 Escribir test de propiedad para lista de familias pendientes urgentes
    - **Propiedad 19: Lista de familias pendientes urgentes antes del cierre de período**
    - **Valida: Requisitos 4.5**
  - [ ]* 9.6 Escribir test de propiedad para integridad del RegistroEntrega
    - **Propiedad 20: Integridad del RegistroEntrega creado al confirmar una entrega**
    - **Valida: Requisitos 4.6**
  - [ ]* 9.7 Escribir test de propiedad para cálculo del progreso de entregas
    - **Propiedad 21: Cálculo correcto del progreso de entregas del período**
    - **Valida: Requisitos 4.7**
  - [~] 9.8 Crear `src/features/entregas/BuscadorFamilia.tsx` (búsqueda por nombre parcial o identificador exacto)
    - _Requisitos: 4.2_
  - [~] 9.9 Crear `src/features/entregas/EstadoEntrega.tsx` (indicador visual diferenciado: con entrega / sin entrega en el período)
    - _Requisitos: 4.3_
  - [~] 9.10 Crear `src/features/entregas/ListaPendientes.tsx` (familias sin entrega a ≤24 h del cierre de período)
    - _Requisitos: 4.5_
  - [~] 9.11 Crear `src/features/entregas/FormularioEntrega.tsx` (prepara entrega con sugerencia FEFO automática; bloquea segunda entrega sin confirmación del coordinador; crea RegistroEntrega; integra `BuscadorFamilia`, `EstadoEntrega` y sugerencia del almacenService)
    - _Requisitos: 4.2, 4.3, 4.4, 4.6, 4.9_
  - [~] 9.12 Crear `src/features/entregas/InformePeriodo.tsx` (resumen de entregas realizadas, familias pendientes y artículos distribuidos; contador actualizable en ≤30 s)
    - _Requisitos: 4.7, 4.8_

- [~] 10. Checkpoint — Verificar módulos Almacén y Entregas
  - Asegurarse de que todos los tests pasan y que las restricciones de tiempo (≤3 s para inventario, ≤5 s para stock, ≤30 s para progreso) se verifican con Firebase Emulator Suite. Preguntar al usuario si surgen dudas.

- [ ] 11. Implementar módulo de Catálogo de Herramientas
  - [~] 11.1 Implementar `src/features/catalogo/catalogoService.ts` con las funciones `filtrarCatalogoPorNivel`, `ordenarCatalogoPorNivel`, `validarHerramienta` y `verificarEnlaceDisponible`; CRUD de herramientas en Firestore
    - _Requisitos: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 5.7_
  - [ ]* 11.2 Escribir test de propiedad para cobertura de los tres niveles del catálogo
    - **Propiedad 22: El catálogo cubre los tres niveles de dificultad**
    - **Valida: Requisitos 5.1**
  - [ ]* 11.3 Escribir test de propiedad para ordenación del catálogo (Básico primero)
    - **Propiedad 23: Ordenación del catálogo coloca primero las herramientas Básicas**
    - **Valida: Requisitos 5.2**
  - [ ]* 11.4 Escribir test de propiedad para validación de estructura de herramienta
    - **Propiedad 24: Validación de estructura de herramienta del catálogo**
    - **Valida: Requisitos 5.3**
  - [ ]* 11.5 Escribir test de propiedad para filtrado del catálogo y sección recomendada
    - **Propiedad 25: Filtrado del catálogo y sección recomendada son consistentes**
    - **Valida: Requisitos 5.5, 5.6**
  - [~] 11.6 Crear `src/features/catalogo/TarjetaHerramienta.tsx` (nombre, descripción ≤80 palabras, nivel, enlace, guía de primeros pasos 3–5 pasos; muestra herramienta alternativa si el enlace falla)
    - _Requisitos: 5.3, 5.4, 5.7_
  - [~] 11.7 Crear `src/features/catalogo/SeccionRecomendada.tsx` (herramientas del nivel del voluntario autenticado bajo etiqueta "Recomendado para tu nivel")
    - _Requisitos: 5.6_
  - [~] 11.8 Crear `src/features/catalogo/CatalogoHerramientas.tsx` (lista filtrable por nivel; sección recomendada en primera posición; mensaje cuando no hay herramientas para el nivel seleccionado; integra `TarjetaHerramienta` y `SeccionRecomendada`)
    - _Requisitos: 5.1, 5.2, 5.5, 5.6_

- [ ] 12. Implementar módulo de Dashboard
  - [~] 12.1 Implementar `src/features/dashboard/dashboardService.ts` con la función de cálculo de `MetricasDashboard` integrando datos de almacén, entregas y voluntarios; actualización automática cada 5 minutos
    - _Requisitos: 6.1, 6.2, 6.3, 6.4_
  - [ ]* 12.2 Escribir test de propiedad para métricas del dashboard
    - **Propiedad 26: Las métricas del dashboard reflejan el estado real del sistema**
    - **Valida: Requisitos 6.2**
  - [ ]* 12.3 Escribir test de propiedad para etiquetas de indicadores (máximo 4 palabras)
    - **Propiedad 27: Las etiquetas de indicadores del dashboard tienen como máximo 4 palabras**
    - **Valida: Requisitos 6.6**
  - [~] 12.4 Crear `src/features/dashboard/AlertaEscasez.tsx` (lista de tipos de alimento con stock bajo umbral; nombre + cantidad actual)
    - _Requisitos: 3.8, 6.3_
  - [~] 12.5 Crear `src/features/dashboard/AlertaUrgente.tsx` (artículos con caducidad ≤3 días; fondo visualmente diferenciado; etiqueta textual "Urgente")
    - _Requisitos: 3.3, 6.4_
  - [~] 12.6 Crear `src/features/dashboard/FallbackDatos.tsx` (conserva últimos valores conocidos; aviso textual cuando los datos no están actualizados)
    - _Requisitos: 6.8_
  - [~] 12.7 Crear `src/features/dashboard/IndicadorMetrica.tsx` (icono + etiqueta ≤4 palabras + valor numérico; navega a la sección correspondiente con filtro al hacer clic)
    - _Requisitos: 6.6, 6.7_
  - [~] 12.8 Crear `src/features/dashboard/PanelPrincipal.tsx` (cuadrícula responsiva de indicadores; accesible en 1 clic desde cualquier sección; legible en móvil 5"; integra todos los sub-componentes del dashboard; muestra 5 métricas)
    - _Requisitos: 6.1, 6.2, 6.5, 6.6, 6.7, 6.8_

- [ ] 13. Implementar enrutamiento y navegación principal
  - [~] 13.1 Configurar React Router con rutas para cada módulo (Dashboard, Voluntarios, Donaciones, Almacén, Entregas, Catálogo) y autenticación Firebase (correo/contraseña) con protección de rutas por rol (`coordinador` / `voluntario` via `customClaims`)
    - _Requisitos: 6.1, 7.5_
  - [~] 13.2 Crear barra de navegación superior responsiva con acceso en 1 clic al Dashboard; badge "Sin conexión" cuando `useOfflineStatus` detecta desconexión; integrar `useSessionTimeout`
    - _Requisitos: 6.1, 7.7_
  - [ ]* 13.3 Escribir tests de ejemplo para flujos de navegación críticos (clic en indicador del dashboard navega con filtro correcto; protección de rutas por rol)
    - _Requisitos: 6.7_

- [~] 14. Checkpoint — Verificar todos los módulos integrados
  - Asegurarse de que todos los tests de propiedad y de ejemplo pasan, que la navegación entre módulos funciona correctamente y que el flujo offline→sync es correcto. Preguntar al usuario si surgen dudas.

- [ ] 15. Añadir datos iniciales del catálogo y configuración de umbrales
  - [~] 15.1 Crear script de seed (`src/firebase/seed.ts`) que pueble Firestore con al menos una herramienta por nivel (Básico, Intermedio, Avanzado), incluyendo la herramienta Básica orientada a registro en papel, con sus herramientas alternativas
    - _Requisitos: 5.1, 5.4_
  - [~] 15.2 Crear `src/features/almacen/ConfiguracionUmbrales.tsx` (permite al coordinador definir el umbral de escasez por tipo de alimento, entre 1 y 9.999 unidades)
    - _Requisitos: 3.8_

- [~] 16. Checkpoint final — Verificar cobertura completa y construir la PWA
  - Ejecutar la suite completa de tests (`vitest --run`); verificar que todos los tests de propiedades (29 propiedades × 100 iteraciones) y tests de ejemplo pasan. Construir el proyecto con `vite build` y comprobar que el Service Worker se genera correctamente. Preguntar al usuario si surgen dudas.

---

## Notes

- Las sub-tareas marcadas con `*` son opcionales y pueden omitirse para un MVP más rápido, pero se recomienda su implementación para garantizar la corrección formal del sistema.
- Cada tarea referencia los requisitos específicos para trazabilidad completa.
- Los checkpoints garantizan validación incremental antes de avanzar al siguiente bloque funcional.
- Los tests de propiedad usan **fast-check** con `numRuns: 100` mínimo y el comentario de etiqueta `// Feature: banco-alimentos-barrio, Propiedad N: <descripción>`.
- Los tests unitarios y de integración usan **Vitest** + **React Testing Library** + **Firebase Emulator Suite** (gratuito).
- Las restricciones de tiempo (≤3 s incorporación de artículos, ≤5 s actualización de stock, ≤30 s progreso de entregas) se verifican con Firebase Emulator Suite en los tests de integración.
- El proyecto se despliega en **Firebase Hosting** (nivel gratuito Spark); no se requiere ningún paso de despliegue en este plan de tareas.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1"] },
    { "id": 1, "tasks": ["2.1", "3.4", "3.5", "3.6"] },
    { "id": 2, "tasks": ["2.2", "2.4", "2.6", "3.1", "3.2", "3.3"] },
    { "id": 3, "tasks": ["2.3", "2.5", "2.7", "2.8", "3.7"] },
    { "id": 4, "tasks": ["5.1", "6.1", "8.1", "9.1", "11.1", "12.1"] },
    { "id": 5, "tasks": ["5.2", "5.3", "5.4", "5.5", "6.2", "6.3", "6.4", "6.5", "8.2", "8.3", "8.4", "9.2", "9.3", "9.4", "9.5", "9.6", "9.7", "11.2", "11.3", "11.4", "11.5", "12.2", "12.3"] },
    { "id": 6, "tasks": ["5.6", "6.6", "6.7", "8.5", "8.6", "9.8", "9.9", "9.10", "11.6", "11.7", "12.4", "12.5", "12.6", "12.7"] },
    { "id": 7, "tasks": ["5.7", "5.8", "6.8", "6.9", "8.7", "9.11", "11.8", "12.8"] },
    { "id": 8, "tasks": ["5.9", "8.8", "9.12", "13.1"] },
    { "id": 9, "tasks": ["13.2", "15.1", "15.2"] },
    { "id": 10, "tasks": ["13.3"] }
  ]
}
```
