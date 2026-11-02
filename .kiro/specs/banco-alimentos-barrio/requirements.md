# Requirements Document

## Introduction

Este documento describe los requisitos del **Sistema de Gestión del Banco de Alimentos de Barrio**, una solución diseñada para ser operada por voluntarios de distintas edades y niveles tecnológicos, sin presupuesto asignado. El sistema cubre la gestión de voluntarios, donaciones, almacén con control de caducidad (FIFO/FEFO), registro de entregas a familias, catálogo de herramientas ordenadas por dificultad, un panel de control (dashboard) centralizado, autenticación segura mediante Google OAuth, generación de recibos fiscales digitales para donantes y alertas y notificaciones automáticas al administrador.

---

## Glossary

- **Sistema**: El sistema de gestión del banco de alimentos de barrio.
- **Voluntario**: Persona que colabora en las operaciones del banco de alimentos sin remuneración económica.
- **Coordinador**: Voluntario con rol de supervisión general que gestiona a otros voluntarios y accede a todas las funciones del sistema.
- **Donante**: Persona o entidad que aporta alimentos al banco.
- **Familia_Beneficiaria**: Unidad familiar registrada que puede recibir entregas de alimentos.
- **Donación**: Conjunto de alimentos recibidos de un donante en un momento determinado.
- **Artículo**: Unidad de alimento individual registrada en el almacén con fecha de caducidad.
- **Lote**: Grupo de artículos del mismo tipo recibidos en la misma donación.
- **Registro_Entrega**: Documento digital que acredita que una familia beneficiaria ha recibido alimentos en un período determinado.
- **Período_Distribución**: Intervalo de tiempo (normalmente semanal) durante el cual se realizan entregas a familias.
- **FIFO**: First In, First Out — los artículos que entraron primero se distribuyen primero.
- **FEFO**: First Expired, First Out — los artículos con fecha de caducidad más próxima se distribuyen primero.
- **Dashboard**: Panel de control visual que muestra el estado general del banco de alimentos en tiempo real.
- **Herramienta_Digital**: Aplicación, plantilla o recurso gratuito recomendado para apoyar la gestión del banco.
- **Nivel_Tecnológico**: Clasificación del voluntario según su dominio de herramientas digitales: Básico, Intermedio o Avanzado.
- **Google_OAuth**: Protocolo de autenticación mediante cuenta de Google que permite acceso seguro sin contraseña local.
- **Recibo_Fiscal**: Documento digital que certifica una donación realizada por un donante, con los datos necesarios para efectos fiscales o tributarios.
- **Notificación_Push**: Mensaje enviado automáticamente al dispositivo del administrador para alertarle de eventos críticos del sistema.
- **Administrador**: Rol con los mismos privilegios que el Coordinador más acceso a la configuración global del sistema y recepción de alertas automáticas.

---

## Requirements

### Requisito 1: Gestión de Voluntarios

**Historia de usuario:** Como coordinador del banco de alimentos, quiero gestionar el listado semanal de voluntarios y conocer su nivel tecnológico, para asignar tareas adecuadas a cada persona y garantizar que haya cobertura suficiente cada semana.

#### Criterios de Aceptación

1. THE Sistema SHALL mantener un registro de voluntarios que incluya nombre completo (máximo 100 caracteres), al menos un dato de contacto (teléfono de hasta 15 dígitos o dirección de correo electrónico de hasta 254 caracteres), nivel tecnológico (Básico, Intermedio o Avanzado) y disponibilidad semanal expresada como selección de uno o más días de la semana (lunes a domingo).
2. WHEN un coordinador registra un nuevo voluntario, THE Sistema SHALL almacenar los datos del voluntario y asignarle un identificador único numérico autogenerado.
3. WHEN comienza una nueva semana, THE Sistema SHALL permitir al coordinador actualizar la lista de voluntarios activos para ese período.
4. WHILE un voluntario está registrado como activo en la semana en curso, THE Sistema SHALL mostrar su nombre, nivel tecnológico y tareas asignadas en el panel de la semana.
5. THE Sistema SHALL mostrar una etiqueta textual con el valor del nivel tecnológico (Básico, Intermedio o Avanzado) junto a cada voluntario en el listado semanal, de forma que el nivel sea identificable sin depender únicamente del color.
6. IF un coordinador intenta registrar un voluntario cuyo nombre completo coincide con el de un registro existente y comparte al menos un dato de contacto con ese registro, THEN THE Sistema SHALL mostrar un aviso de posible duplicado; si el coordinador confirma, el Sistema SHALL guardar el nuevo registro; si el coordinador cancela, el Sistema SHALL descartar el nuevo registro y mantener el formulario abierto con los datos introducidos.
7. WHEN el coordinador filtra el listado por nivel tecnológico, THE Sistema SHALL mostrar únicamente los voluntarios que coincidan con el filtro seleccionado.
8. THE Sistema SHALL permitir al coordinador añadir notas sobre la experiencia o disponibilidad de cada voluntario, con un máximo de 500 caracteres por nota.

---

### Requisito 2: Gestión de Donaciones

**Historia de usuario:** Como voluntario receptor de donaciones, quiero registrar rápidamente cualquier donación que llegue sin previo aviso, para que el almacén refleje siempre el inventario real disponible.

#### Criterios de Aceptación

1. WHEN un voluntario inicia el registro de una donación, THE Sistema SHALL solicitar: fecha y hora de recepción, nombre del donante (opcional, máximo 100 caracteres), y una lista de artículos de hasta 50 ítems, cada uno con cantidad (mínimo 1, máximo 9.999 unidades) y fecha de caducidad.
2. THE Sistema SHALL permitir registrar una donación en un máximo de 5 pasos en pantalla.
3. WHEN una donación es guardada, THE Sistema SHALL incorporar los artículos al inventario del almacén con sus fechas de caducidad en un plazo máximo de 3 segundos tras la confirmación.
4. IF uno o más artículos de una donación tienen fecha de caducidad anterior a la fecha actual, THEN THE Sistema SHALL mostrar una alerta indicando los artículos afectados y requerir confirmación explícita del voluntario antes de continuar; el voluntario podrá cancelar el guardado o confirmar para proceder igualmente.
5. WHEN se guarda una donación, THE Sistema SHALL registrar el nombre del voluntario que la recibió y la fecha y hora exactas del registro en UTC.
6. THE Sistema SHALL permitir completar y guardar el registro de una donación sin conexión a internet.
7. WHEN la conexión a internet se restablece, THE Sistema SHALL sincronizar automáticamente las donaciones registradas sin conexión; IF existen datos en conflicto entre el registro local y el servidor, THEN THE Sistema SHALL preservar el registro local y notificar al coordinador del conflicto para su resolución manual.
8. WHEN el coordinador consulta el historial de donaciones, THE Sistema SHALL mostrar la lista ordenada por fecha de recepción de más reciente a más antigua, con un máximo de 50 registros por página.
9. WHEN un voluntario inicia el registro de una nueva donación, IF el nombre del donante coincide con el de un donante que tiene 2 o más donaciones previas registradas en el Sistema, THEN THE Sistema SHALL mostrar el historial de las últimas 10 donaciones de ese donante.

---

### Requisito 3: Gestión del Almacén y Control de Caducidad (FIFO/FEFO)

**Historia de usuario:** Como voluntario de almacén, quiero ver qué alimentos caducan antes y gestionarlos con el método FEFO, para reducir el desperdicio y garantizar que se entreguen alimentos en buen estado.

#### Criterios de Aceptación

1. THE Sistema SHALL mantener un inventario de artículos organizado por tipo de alimento, cantidad disponible y fecha de caducidad.
2. WHEN un voluntario consulta el inventario, THE Sistema SHALL mostrar los artículos ordenados por fecha de caducidad de más próxima a más lejana (FEFO).
3. IF la fecha de caducidad de un artículo es igual o inferior a 7 días desde la fecha actual y el artículo no ha caducado aún, THEN THE Sistema SHALL mostrar la etiqueta "Próximo a caducar" junto a ese artículo en el inventario.
4. IF la fecha de caducidad de un artículo es anterior a la fecha actual, THEN THE Sistema SHALL mostrar la etiqueta "Caducado" junto a ese artículo en el inventario, visualmente distinguible de la etiqueta "Próximo a caducar".
5. WHEN se prepara una entrega a una familia beneficiaria y el stock disponible del tipo de alimento seleccionado es mayor a 0, THE Sistema SHALL sugerir automáticamente los artículos con fecha de caducidad más próxima para ser incluidos en la entrega (aplicando FEFO).
6. IF el stock disponible de un tipo de alimento es igual a 0 durante la preparación de una entrega, THEN THE Sistema SHALL mostrar un aviso indicando que no hay existencias disponibles de ese alimento y no incluirlo en la sugerencia de entrega.
7. WHEN un artículo es retirado del almacén (por entrega o descarte), THE Sistema SHALL actualizar la cantidad disponible de ese artículo en un plazo máximo de 5 segundos.
8. IF el stock de un tipo de alimento cae por debajo de un umbral definido por el coordinador (entre 1 y 9.999 unidades), THEN THE Sistema SHALL mostrar una alerta en el dashboard indicando el nombre del artículo afectado y la cantidad actual en almacén.
9. WHEN un artículo entra o sale del almacén, THE Sistema SHALL registrar un movimiento con los campos: fecha y hora, cantidad, tipo de artículo y motivo (donación recibida, entrega o descarte por caducidad).
10. WHEN el coordinador solicita un informe de caducados, THE Sistema SHALL generar un listado de artículos descartados por caducidad en el período seleccionado (entre 1 y 365 días).

---

### Requisito 4: Registro de Entregas a Familias

**Historia de usuario:** Como voluntario de distribución, quiero registrar cada entrega a una familia beneficiaria y consultar el historial del período actual, para asegurarme de que todas las familias reciban exactamente una entrega por período y evitar conflictos en la comunidad.

#### Criterios de Aceptación

1. THE Sistema SHALL mantener un padrón de familias beneficiarias con nombre del titular (máximo 100 caracteres), número de miembros del hogar (entre 1 y 20) e identificador único numérico.
2. WHEN un voluntario inicia una entrega, THE Sistema SHALL permitir buscar a la familia beneficiaria por nombre o identificador.
3. WHEN se selecciona una familia beneficiaria para entrega, THE Sistema SHALL mostrar un indicador de estado de entrega visible y diferenciado visualmente del estado sin entrega, indicando si esa familia ya ha recibido una entrega en el período de distribución actual.
4. IF una familia beneficiaria ya ha recibido una entrega en el período actual, THEN THE Sistema SHALL mostrar un aviso de alerta, bloquear el registro de la entrega y requerir confirmación explícita del coordinador para proceder; IF la confirmación del coordinador no es proporcionada, THEN THE Sistema SHALL cancelar la operación y conservar el Registro_Entrega existente sin modificaciones.
5. IF una familia beneficiaria no ha recibido entrega en el período actual y faltan 24 horas o menos para la fecha de cierre del período, THEN THE Sistema SHALL incluir a esa familia en una lista de pendientes visible para el coordinador.
6. WHEN se confirma una entrega, THE Sistema SHALL crear un Registro_Entrega con: identificador de la familia, fecha y hora, lista de artículos entregados (máximo 50 artículos), nombre del voluntario que realizó la entrega.
7. WHILE el período de distribución está activo, THE Sistema SHALL mostrar el número de familias que han recibido entrega frente al total registrado, actualizándose en un plazo máximo de 30 segundos tras cada entrega confirmada.
8. WHEN el coordinador solicita el informe del período, THE Sistema SHALL generar un listado con todas las entregas realizadas, familias pendientes y artículos distribuidos.
9. THE Sistema SHALL garantizar que cada familia beneficiaria aparece como máximo una vez como receptora confirmada sin intervención del coordinador en cualquier período de distribución.
10. WHEN cualquier usuario autenticado registra una familia beneficiaria, THE Sistema SHALL permitir su alta; la autorización para registrar familias SHALL NOT requerir el rol de coordinador ni de administrador.

---

### Requisito 5: Catálogo de Herramientas Digitales

**Historia de usuario:** Como voluntario de cualquier nivel tecnológico, quiero acceder a un listado de herramientas ordenado de más fácil a más compleja, para poder elegir la que mejor se adapte a mis capacidades y mantener el orden sin coste adicional.

#### Criterios de Aceptación

1. THE Sistema SHALL mostrar un catálogo de herramientas digitales gratuitas o de coste cero, clasificadas en tres niveles de dificultad: Básico, Intermedio y Avanzado, con al menos una herramienta disponible en cada nivel.
2. WHEN un voluntario accede al catálogo, THE Sistema SHALL presentar primero las herramientas de nivel Básico.
3. THE Sistema SHALL incluir para cada herramienta: nombre, descripción de máximo 80 palabras sin términos técnicos no definidos en la propia guía de primeros pasos, nivel de dificultad, enlace de acceso y una guía de primeros pasos de entre 3 y 5 puntos.
4. THE Sistema SHALL incluir al menos una herramienta de nivel Básico orientada al registro en papel o formularios simples en línea, para voluntarios sin acceso a dispositivos avanzados.
5. WHEN un voluntario filtra el catálogo por nivel de dificultad, THE Sistema SHALL mostrar únicamente las herramientas que correspondan al nivel seleccionado; IF no existen herramientas registradas para ese nivel, THEN THE Sistema SHALL mostrar un mensaje indicando que no hay herramientas disponibles para el nivel seleccionado.
6. WHERE el voluntario ha indicado su nivel tecnológico en su perfil, THE Sistema SHALL mostrar las herramientas correspondientes a ese nivel bajo una sección etiquetada como "Recomendado para tu nivel" al abrir el catálogo.
7. IF un enlace de herramienta no está disponible en el momento de acceso, THEN THE Sistema SHALL mostrar un mensaje de alternativa o herramienta sustituta del mismo nivel.

---

### Requisito 6: Panel de Control (Dashboard)

**Historia de usuario:** Como coordinador del banco de alimentos, quiero un panel de control visual con la información más relevante en una sola pantalla, para tomar decisiones rápidas sin tener que navegar por múltiples secciones.

#### Criterios de Aceptación

1. THE Sistema SHALL presentar un dashboard como pantalla principal accesible desde cualquier sección del sistema en un máximo de un clic o toque.
2. THE Dashboard SHALL mostrar, con actualización automática cada 5 minutos: número de voluntarios activos en la semana, cantidad total de artículos en almacén, número de artículos con etiqueta "Próximo a caducar" (≤ 7 días), número de familias con entrega completada en los 7 días naturales en curso frente al total registrado, y número de artículos descartados por caducidad en el período.
3. WHEN el nivel de stock de cualquier artículo cae por debajo del umbral definido, THE Dashboard SHALL mostrar una alerta de escasez con el nombre del artículo afectado y su cantidad actual en almacén.
4. WHEN hay artículos con etiqueta "Próximo a caducar" con caducidad igual o inferior a 3 días, THE Dashboard SHALL mostrar esos artículos con fondo de color diferenciado y la etiqueta textual "Urgente".
5. THE Dashboard SHALL ser legible en dispositivos móviles con pantalla de 5 pulgadas o superior, sin necesidad de desplazamiento horizontal.
6. THE Dashboard SHALL mostrar cada indicador con un icono y una etiqueta de texto de no más de 4 palabras, sin requerir interacción adicional para identificar su significado.
7. WHEN el coordinador hace clic en cualquier indicador del dashboard, THE Sistema SHALL navegar a la sección correspondiente aplicando automáticamente el mismo filtro de período que se muestra en el dashboard.
8. IF los datos del dashboard no pueden cargarse, THEN THE Dashboard SHALL conservar los últimos valores conocidos y mostrar un aviso de que los datos pueden no estar actualizados.

---

### Requisito 7: Accesibilidad y Usabilidad para Voluntarios No Técnicos

**Historia de usuario:** Como voluntario mayor con poca experiencia tecnológica, quiero que el sistema sea sencillo e intuitivo, para poder usarlo sin formación técnica ni ayuda constante.

#### Criterios de Aceptación

1. THE Sistema SHALL utilizar lenguaje claro y cotidiano en todos los textos, evitando términos técnicos sin definición contextual.
2. THE Sistema SHALL presentar botones de acción principales con un tamaño mínimo de 44x44 píxeles y etiquetas de texto visibles, sin depender únicamente de iconos.
3. THE Sistema SHALL proporcionar mensajes de confirmación en lenguaje no técnico después de cada acción de guardado, eliminación o registro de entrega, indicando el resultado de la acción realizada.
4. IF un voluntario comete un error de entrada de datos, THEN THE Sistema SHALL mostrar un mensaje de error que indique qué campo está incorrecto y cómo corregirlo, sin usar códigos de error técnicos.
5. THE Sistema SHALL estar disponible como aplicación web accesible desde las dos últimas versiones estables de Chrome, Firefox, Safari y Edge, en ordenadores, tabletas y teléfonos móviles, sin necesidad de instalación.
6. THE Sistema SHALL funcionar con herramientas de coste cero, incluyendo únicamente soluciones gratuitas o de nivel gratuito suficiente para las necesidades descritas.
7. WHEN un voluntario no ha realizado ninguna acción durante 30 minutos, THE Sistema SHALL mostrar un aviso preguntando si desea continuar la sesión activa; IF el voluntario no responde en 5 minutos adicionales, THEN THE Sistema SHALL cerrar la sesión, redirigir a la pantalla de inicio y conservar los datos del formulario en progreso para recuperarlos al volver a iniciar sesión.

---

### Requisito 8: Autenticación con Google

**Historia de usuario:** Como coordinador o administrador del banco de alimentos, quiero poder iniciar sesión con mi cuenta de Google, para acceder al sistema de forma segura y rápida sin necesidad de recordar una contraseña adicional.

#### Criterios de Aceptación

1. THE Sistema SHALL ofrecer un botón de "Iniciar sesión con Google" en la pantalla de acceso, sin requerir registro previo con usuario y contraseña local.
2. WHEN un usuario hace clic en "Iniciar sesión con Google", THE Sistema SHALL redirigir al flujo OAuth de Google y, tras la autenticación exitosa, crear o actualizar automáticamente el perfil del usuario en el sistema con su nombre y correo de Google.
3. WHEN un usuario se autentica por primera vez con Google, THE Sistema SHALL asignarle el rol "voluntario" por defecto; IF un administrador existente eleva el rol del usuario a "coordinador" o "administrador", THEN el Sistema SHALL aplicar los nuevos permisos en la siguiente sesión del usuario.
4. IF la autenticación con Google falla o es cancelada por el usuario, THEN THE Sistema SHALL mostrar un mensaje claro en lenguaje cotidiano indicando que el acceso no se completó y ofrecer la opción de intentarlo de nuevo.
5. WHEN un usuario cierra sesión, THE Sistema SHALL revocar el acceso y redirigir a la pantalla de inicio de sesión.
6. THE Sistema SHALL mantener la sesión activa entre recargas de página durante un máximo de 7 días; pasado ese tiempo, requerirá nueva autenticación con Google.
7. IF un usuario intenta acceder a una ruta protegida sin sesión activa, THEN THE Sistema SHALL redirigirlo automáticamente a la pantalla de inicio de sesión.
8. THE Sistema SHALL mostrar el nombre y foto de perfil de Google del usuario autenticado en la barra de navegación superior.

---

### Requisito 9: Recibos Fiscales por Donante

**Historia de usuario:** Como administrador del banco de alimentos, quiero generar recibos fiscales digitales para cada donante, para que los donantes puedan justificar sus aportaciones ante las autoridades fiscales y así incentivar futuras donaciones.

#### Criterios de Aceptación

1. WHEN el administrador selecciona un donante y un período, THE Sistema SHALL generar un recibo fiscal digital en formato PDF que incluya: nombre del donante, fecha de la donación, lista de artículos donados con cantidades, y nombre y dirección del banco de alimentos.
2. THE Sistema SHALL permitir generar recibos fiscales únicamente a usuarios con rol "administrador".
3. WHEN se genera un recibo fiscal, THE Sistema SHALL asignarle un número de folio único y secuencial dentro del año en curso (formato: AÑO-NÚMERO, ejemplo: 2026-001).
4. THE Sistema SHALL permitir al administrador descargar el recibo en formato PDF desde el navegador sin necesidad de software adicional.
5. IF una donación asociada a un recibo ya emitido es modificada, THEN THE Sistema SHALL marcar el recibo como "anulado" y permitir generar un recibo nuevo corregido con el folio anterior visible.
6. THE Sistema SHALL mantener un historial de todos los recibos generados, filtrable por donante y por año, accesible solo para el administrador.
7. WHEN el administrador consulta el historial de recibos, THE Sistema SHALL mostrar para cada recibo: folio, nombre del donante, fecha de generación, estado (vigente / anulado) y enlace de descarga.
8. THE Sistema SHALL funcionar con herramientas de coste cero para la generación de PDF (por ejemplo, usando la API de impresión del navegador o una librería gratuita como jsPDF).

---

### Requisito 10: Alertas y Notificaciones Automáticas al Administrador

**Historia de usuario:** Como administrador del banco de alimentos, quiero recibir alertas automáticas cuando ocurran eventos críticos en el sistema, para poder reaccionar a tiempo sin tener que consultar el dashboard manualmente.

#### Criterios de Aceptación

1. THE Sistema SHALL enviar una notificación automática al administrador WHEN cualquiera de los siguientes eventos ocurra: (a) el stock de un tipo de alimento cae por debajo del umbral de escasez definido, (b) hay artículos con caducidad igual o inferior a 3 días ("Urgente"), (c) una familia recibe una segunda entrega en el mismo período con confirmación del coordinador, (d) hay un conflicto de sincronización offline sin resolver.
2. WHEN el administrador tiene el sistema abierto en el navegador, THE Sistema SHALL mostrar las notificaciones como banners no bloqueantes en la esquina superior derecha, con un botón de "Ver detalle" que navega a la sección afectada.
3. THE Sistema SHALL mantener un centro de notificaciones accesible desde la barra superior, mostrando las últimas 50 notificaciones ordenadas de más reciente a más antigua, con indicador visual de no leídas.
4. WHEN el administrador marca una notificación como leída o la descarta, THE Sistema SHALL actualizar el estado de esa notificación de forma inmediata.
5. IF el administrador tiene activadas las notificaciones del navegador (Web Push API), THEN THE Sistema SHALL enviar notificaciones push incluso cuando el sistema no está abierto en primer plano, usando el Service Worker existente.
6. THE Sistema SHALL agrupar notificaciones del mismo tipo ocurridas en un intervalo de 10 minutos en una sola notificación consolidada (por ejemplo: "5 artículos nuevos con caducidad urgente").
7. THE Sistema SHALL permitir al administrador configurar qué tipos de eventos generan notificaciones y cuáles no, desde la sección de configuración.
8. WHEN se genera una notificación, THE Sistema SHALL registrarla en Firestore con: tipo de evento, descripción, fecha y hora, estado (no leída / leída / descartada) y referencia al recurso afectado.
