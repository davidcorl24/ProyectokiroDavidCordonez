---
name: banco-alimentos-barrio
description: Guía completa para desarrollar e implementar el Sistema de Gestión del Banco de Alimentos de Barrio: una PWA con React, Firebase y diseño accesible para voluntarios.
---

# Skill: Banco de Alimentos de Barrio

## Propósito

Esta skill proporciona instrucciones, contexto y directrices para implementar el **Sistema de Gestión del Banco de Alimentos de Barrio** — una aplicación web progresiva (PWA) de coste cero, orientada a voluntarios de distintos niveles tecnológicos, incluyendo personas mayores con poca experiencia digital.

---

## Contexto del proyecto

- **Sin presupuesto**: todo el stack debe ser gratuito (Firebase Spark, React/Vite, Tailwind CSS, jsPDF, fast-check)
- **Usuarios objetivo**: voluntarios con nivel tecnológico Básico, Intermedio o Avanzado; el diseño prioriza simplicidad extrema
- **Offline obligatorio**: Firestore Persistence + IndexedDB para borradores
- **Idioma**: español en toda la interfaz y los documentos

---

## Stack tecnológico

| Capa | Tecnología | Notas |
|------|-----------|-------|
| Frontend | React 18 + Vite | SPA / PWA |
| Estilos | Tailwind CSS | Clases utilitarias, responsivo |
| Backend / DB | Firebase Firestore (Spark) | Offline persistence, tiempo real |
| Autenticación | Firebase Auth + Google OAuth | Sin contraseña local |
| Hosting | Firebase Hosting | HTTPS automático, CDN |
| PDF | jsPDF (gratuito) | Generación de recibos en cliente |
| Notificaciones push | Firebase Cloud Messaging (FCM) | Service Worker |
| Iconos | Heroicons | SVG, compatibles con Tailwind |
| Tipografía | Inter (Google Fonts) | Legible, moderna |
| Testing PBT | fast-check | ≥100 iteraciones por propiedad |
| Testing unitario | Vitest + React Testing Library | |

---

## Design System

### Paleta de colores

```
Verde Esperanza (primario) : #2D7D46  — botones, encabezados
Verde Claro (secundario)   : #52C27A  — hover, badges activos
Naranja Cálido (alerta)    : #F4A023  — escasez, próximo a caducar
Rojo Suave (urgencia)      : #E05252  — caducados, urgente, errores
Azul Tranquilo (info)      : #3B82F6  — notificaciones informativas
Crema (fondo)              : #FAFAF5  — fondo general
Blanco tarjeta             : #FFFFFF  — tarjetas y modales
Gris Texto                 : #374151  — texto principal
Gris Suave                 : #9CA3AF  — placeholders, texto secundario
Degradado login            : linear-gradient(135deg, #2D7D46 0%, #52C27A 100%)
```

### Tipografía

- Fuente: `Inter` (Google Fonts)
- Títulos: `text-2xl font-semibold` | Subtítulos: `text-lg font-medium`
- Cuerpo: `text-base font-normal` | Botones: `text-base font-semibold`
- Line-height cuerpo: 1.6 | Line-height títulos: 1.3

### Badges de estado de caducidad

| Estado | Fondo | Texto | Extra |
|--------|-------|-------|-------|
| Normal | #E8F5E9 | #2D7D46 | — |
| Próximo a caducar | #FEF3C7 | #D97706 | — |
| Urgente | #FEE2E2 | #E05252 | animate-pulse |
| Caducado | #F3F4F6 | #6B7280 | line-through |

---

## Módulos del sistema

### 1. Voluntarios (`features/voluntarios/`)
- Listado semanal con etiquetas de nivel tecnológico (Básico / Intermedio / Avanzado)
- Detección de duplicados por nombre + contacto
- Filtro por nivel, notas de 500 caracteres máximo

### 2. Donaciones (`features/donaciones/`)
- Wizard de registro en ≤5 pasos
- Alerta para artículos ya caducados al recibir
- Soporte offline + sincronización con resolución de conflictos local-primero
- Historial de donantes recurrentes (≥2 donaciones previas → muestra últimas 10)

### 3. Almacén FEFO (`features/almacen/`)
- Inventario ordenado por fecha de caducidad ascendente (FEFO)
- Etiquetas: Normal / Próximo a caducar (≤7 días) / Urgente (≤3 días, animate-pulse) / Caducado
- Sugerencia automática FEFO al preparar entregas
- Alertas de escasez configurables por coordinador (umbral 1–9.999 unidades)
- Registro de movimientos con motivo

### 4. Entregas (`features/entregas/`)
- Bloqueo de doble entrega por familia por período
- Confirmación del coordinador para excepciones
- Lista de familias pendientes cuando faltan ≤24 h para cierre del período
- Progreso en tiempo real (actualización ≤30 s)

### 5. Catálogo de herramientas (`features/catalogo/`)
- Herramientas gratuitas ordenadas Básico → Intermedio → Avanzado
- Sección "Recomendado para tu nivel" según perfil del voluntario
- Al menos 1 herramienta por nivel; fallback si enlace no disponible

### 6. Dashboard (`features/dashboard/`)
- Actualización automática cada 5 minutos
- 5 métricas clave con icono + etiqueta ≤4 palabras
- Alertas de escasez y urgencia
- Fallback a últimos valores conocidos si no hay datos
- Navegación directa a cada sección desde el indicador

### 7. Accesibilidad (`shared/`)
- Botones ≥44×44 px con etiqueta textual visible
- Mensajes de error en lenguaje cotidiano (sin códigos técnicos)
- Aviso de inactividad a 30 min; cierre de sesión a 35 min con recuperación de borrador

### 8. Autenticación Google (`features/auth/`)
- Inicio de sesión con Google (Firebase Auth + GoogleAuthProvider)
- Roles: voluntario (defecto) → coordinador → administrador
- ProtectedRoute para rutas restringidas
- Sesión persistente 7 días; foto y nombre en barra superior

### 9. Recibos Fiscales (`features/recibos/`)
- Solo accesible para el rol administrador
- Generación de PDF con jsPDF en el cliente (sin servidor)
- Folio secuencial: AÑO-NNN (ej: 2026-001, reinicia cada año)
- Historial filtrable por donante y año
- Anulación y corrección de recibos

### 10. Notificaciones (`features/notificaciones/`)
- Eventos que disparan notificación: escasez de stock, caducidad urgente, segunda entrega, conflicto de sincronización
- Banners no bloqueantes (esquina superior derecha, deslizamiento desde arriba)
- Centro de notificaciones con las últimas 50 (icono BellIcon en barra superior)
- Agrupación de notificaciones del mismo tipo en ≤10 minutos
- Web Push via FCM + Service Worker (notificaciones fuera de primer plano)
- Configurable por el administrador

---

## Estructura de directorios

```
src/
├── firebase/          # config.ts, persistence.ts
├── features/
│   ├── voluntarios/
│   ├── donaciones/
│   ├── almacen/
│   ├── entregas/
│   ├── catalogo/
│   ├── dashboard/
│   ├── auth/
│   ├── recibos/
│   └── notificaciones/
├── shared/
│   ├── components/    # BotonPrimario, MensajeConfirmacion, MensajeError
│   ├── hooks/         # useFirestore, useOfflineStatus, useSessionTimeout
│   └── utils/         # fefo.ts, fechas.ts, validaciones.ts
├── service-worker.ts  # Workbox + FCM
└── tests/
    ├── unit/
    └── property/      # fast-check (32 propiedades)
```

---

## Reglas de implementación

1. **Coste cero siempre**: no introducir ninguna dependencia de pago, ni siquiera en tier gratuito con límites que el banco pueda superar fácilmente.
2. **Mensajes en español y sin tecnicismos**: todos los textos visibles al usuario usan lenguaje cotidiano.
3. **Botones mínimo 44×44 px**: aplicar `min-h-[44px] min-w-[44px]` en todos los botones de acción.
4. **FEFO siempre que se sugieran artículos**: nunca sugerir un artículo ignorando la fecha de caducidad.
5. **Unicidad de entrega**: la función `tienEntregaEnPeriodo` debe consultarse antes de crear cualquier `RegistroEntrega`.
6. **Roles via customClaims**: no almacenar el rol solo en Firestore del lado cliente; verificar siempre el token de Firebase.
7. **Folios secuenciales**: usar transacciones de Firestore para generar folios de recibo y evitar duplicados en escrituras concurrentes.
8. **Tests de propiedades**: cada nueva función de dominio debe tener al menos una propiedad fast-check con ≥100 iteraciones.
9. **Offline-first**: todas las escrituras deben funcionar con Firestore en modo offline; la UI nunca debe bloquearse esperando conexión.
10. **Accesibilidad WCAG 2.1 AA**: mantener contraste mínimo AA en todos los colores de texto sobre fondo.

---

## Propiedades de corrección (resumen)

El sistema define 32 propiedades de corrección verificables con fast-check. Las más críticas son:

- **P10**: Ordenación FEFO — los artículos sugeridos tienen siempre la caducidad más próxima primero.
- **P12**: Clasificación de caducidad — cada artículo tiene exactamente uno de los 4 estados (Normal / Próximo / Urgente / Caducado).
- **P17**: Búsqueda de familia — sin falsos positivos ni falsos negativos.
- **P18**: Unicidad de entrega — no puede existir una segunda entrega sin `confirmadaPorCoordinador = true`.
- **P30**: Folios secuenciales — cada folio del año es único y el número incrementa en 1.
- **P32**: Agrupación de notificaciones — N notificaciones del mismo tipo en ≤10 min producen exactamente 1 notificación con `conteoAgrupado = N`.

---

## Colecciones Firestore (resumen)

```
voluntarios / usuarios / donantes / donaciones
tiposAlimento / almacen / movimientosAlmacen
familiasBeneficiarias / periodosDistribucion / registrosEntrega
umbralesEscasez / herramientasDigitales
recibos / notificaciones / configuracion/global
```

---

## Notas para el agente

- Lee siempre `.kiro/specs/banco-alimentos-barrio/requirements.md` y `design.md` antes de implementar cualquier módulo.
- Sigue el Design System definido: colores, tipografía Inter, Heroicons.
- Implementa los tests de propiedades fast-check **junto con** cada función de dominio, no al final.
- Si añades una colección nueva a Firestore, actualiza también las reglas de seguridad.
- Cuando generes un folio de recibo, usa una transacción Firestore para evitar duplicados en concurrencia.
