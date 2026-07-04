# Sprint 4 — Documentación de la Iteración

## Resumen Ejecutivo

### Qué se añadió en esta iteración

- **Carga de operaciones por voz**: los usuarios ahora pueden registrar una operación hablando. Graban un audio desde la aplicación, el sistema lo transcribe y extrae los datos automáticamente, y luego pueden revisar y corregir la operación sugerida antes de confirmarla, sin que se genere ninguna transacción sola.
- **Reservas de dinero y saldo disponible**: los usuarios pueden apartar dinero para gastos que tendrán en el futuro creando reservas, y ver cuánto les queda realmente disponible por cuenta descontando lo reservado, sin que su saldo real cambie.
- **Proyección de gastos futuros**: nueva vista dentro del cronograma que proyecta a futuro los gastos recurrentes activos, mostrando los totales a gastar por período tanto en una tabla como en un gráfico de línea.
- **Mejoras de accesibilidad**: se realizan correcciones contra los criterios WCAG 2.1 nivel AA en las páginas principales, mejorando el contraste, la semántica, los formularios y la navegación por teclado.
- **Base de testing del frontend**: se incorporó la infraestructura de pruebas automatizadas y se cubrieron con tests las funcionalidades y componentes más importantes de la aplicación.

### Decisiones tomadas

- **Speech-to-Text como servicio gestionado**: antes de implementar la carga por voz se realizó un spike comparando alternativas (Whisper, Faster-Whisper y otros proveedores), priorizando opciones open source o de modelos abiertos consumibles como API, para evitar sobrecargar la memoria RAM de los equipos locales.
- **Procesamiento de voz asíncrono y sin transacciones automáticas**: al igual que con los comprobantes, el audio se procesa en segundo plano (transcripción + extracción con un LLM) y nunca genera una transacción por sí solo; el resultado siempre queda pendiente de revisión y confirmación del usuario.
- **Reutilización del patrón de confirmación de comprobantes**: la revisión de operaciones por voz reusa el mismo flujo ya establecido para los comprobantes OCR (consulta periódica del backend + modal de validación con datos precargados), manteniendo consistencia en la experiencia de uso.
- **Reservas sin mover el saldo real**: en lugar de descontar el dinero del saldo, las reservas introducen el concepto de "dinero disponible" (saldo real − lo reservado). Así el saldo real nunca cambia y el usuario ve con claridad cuánto puede gastar sin tocar lo que ya apartó.
- **Monto de la reserva atado al gasto futuro**: el monto no se ingresa manualmente, sino que se calcula como N × monto del gasto futuro asociado, manteniendo la reserva coherente con la planificación y evitando montos arbitrarios.
- **Adopción de Vitest + React Testing Library + MSW**: se estandarizó el stack de testing del frontend y sus convenciones (estructura AAA, helper de render con providers, mock de contextos a nivel de módulo e interceptación de red con MSW) para que las pruebas crezcan de forma consistente en las próximas iteraciones.

---

## User Stories

### Carga de Operaciones por Voz

**Actores:** Usuario

**Funcionalidad:** Como usuario quiero registrar una operación hablando en lugar de completar un formulario o subir un comprobante. La aplicación incorpora una tercera vía de carga junto a los comprobantes y carga de formulario manual en la que el usuario graba un audio, el sistema lo transcribe mediante un motor Speech-to-Text y extrae los datos de la transferencia con un LLM, y finalmente el usuario revisa, corrige y confirma la operación sugerida. Todo el procesamiento ocurre en segundo plano y sin generar transacciones automáticamente.

**Valor:** Ofrece la forma más rápida y natural de cargar una operación, especialmente desde el celular, reduciendo la fricción del ingreso manual y manteniendo la precisión gracias al paso de confirmación previo al guardado.

**Criterios de aceptación:**
- La pantalla de carga incorpora una nueva pestaña "Voz", con un layout propio para la grabación y una sección que refleja los estados de procesamiento (pendiente, procesado, fallido), sin alterar los flujos existentes.
- El usuario puede iniciar, finalizar y cancelar una grabación de audio desde el navegador antes de enviarla.
- El audio grabado se envía al backend, que lo persiste junto con sus metadatos, genera el registro de procesamiento en estado pendiente y responde de inmediato sin esperar el resultado.
- La navegación no se bloquea durante el procesamiento; se informa el envío exitoso y se muestran errores cuando la carga falla.
- El audio se transcribe con un proveedor STT y la transcripción, junto con el esquema de la transferencia, se envía a un LLM que extrae los datos estructurados; el registro pasa a "a confirmar" al finalizar correctamente o a "fallido" ante un error.
- La aplicación consulta periódicamente el backend para detectar cargas por voz pendientes de confirmación mientras la pantalla permanece abierta, deteniendo la consulta al abandonarla y sin generar solicitudes duplicadas.
- El backend expone únicamente las cargas del usuario autenticado en estado pendiente de confirmación, incluyendo la transcripción, los datos sugeridos y la referencia al audio original.
- Cuando existe al menos una carga pendiente se muestra un modal de validación que permite reproducir el audio, leer la transcripción, revisar y corregir los datos sugeridos, y confirmar o descartar la operación.
- Al confirmar o editar una sugerencia, ésta deja de mostrarse como pendiente; cuando hay varias, el usuario puede revisarlas de forma secuencial.
- La experiencia mantiene consistencia visual y funcional con el flujo de validación de comprobantes procesados.

---

### Reservas de Dinero y Saldo Disponible

**Actor:** Usuario

**Funcionalidad:** Como usuario quiero apartar dinero para gastos que tendré en el futuro, para evitar gastar lo que voy a necesitar. El usuario crea reservas asociadas a un gasto futuro y a una fuente de fondeo (una cuenta propia y/o un ingreso futuro), y la aplicación expone por cada cuenta el "dinero disponible" (saldo real menos lo reservado) sin modificar el saldo real. Las reservas se gestionan desde una sección dedicada donde se pueden crear, editar y cancelar.

**Valor:** Permite planificar con realismo cuánto dinero se puede gastar sin comprometer los compromisos futuros, distinguiendo con claridad entre el saldo total y el efectivamente disponible.

**Criterios de aceptación:**
- Existe una sección dedicada a las reservas, accesible desde la navegación, con el listado de reservas del usuario y una acción visible para crear una nueva.
- La creación se realiza mediante un formulario con descripción, fuente de fondeo, gasto futuro destino y cantidad de iteraciones N (≥ 1); la fuente es única (una cuenta propia y/o un ingreso futuro a esa misma cuenta).
- El monto no se ingresa manualmente: se muestra calculado como N × monto del gasto futuro asociado.
- Por cada cuenta se expone el "dinero disponible para usar" = saldo real − lo reservado sobre esa cuenta, distinguiéndolo del saldo real, que no cambia.
- Una reserva fondeada con una cuenta descuenta del disponible de esa cuenta desde su creación; una fondeada con un ingreso futuro no afecta el disponible hasta que ese ingreso se ejecuta.
- El disponible se actualiza al crear, editar, cancelar o cerrar una reserva, y se validan las reglas (fuente única, N ≥ 1, no exceder el disponible) tanto en el frontend como en el backend.
- El usuario puede editar sus reservas reusando el mismo formulario y validaciones, y listar cada una con su monto calculado, fuente, gasto futuro asociado, iteraciones y estado.
- El usuario puede cancelar una reserva: deja de impactar el disponible y queda inactiva, sin borrarse de la base.
- Cuando el ingreso futuro que la fondea se desactiva o nunca se ejecuta, la reserva se cancela; cuando el gasto futuro asociado se ejecuta y genera su transacción, la reserva se cierra (disminuyendo iteración y monto en gastos recurrentes hasta agotarse), tratándose en ambos casos como cumplida e inactiva.
- La vista contempla el estado vacío (con llamada a crear la primera reserva), los estados de carga y error, y distingue visualmente el estado de cada reserva (activa / inactiva / cumplida).
- Un usuario solo puede crear, editar y gestionar reservas sobre sus propias cuentas.

---

### Proyección de Gastos Futuros

**Actor:** Usuario

**Funcionalidad:** Como usuario quiero visualizar el desglose y la evolución de mis gastos recurrentes proyectados a futuro, para anticipar cuánto voy a gastar en un período determinado. La página de cronograma incorpora una vista "Gastos futuros" que consume un nuevo endpoint de proyección y presenta la información en una tabla agrupada por período y en un gráfico evolutivo, con controles de rango de fechas y granularidad compartidos.

**Valor:** Brinda una visión anticipada y clara de los compromisos de gasto futuros, apoyando la planificación financiera con datos y una representación visual.

**Criterios de aceptación:**
- El backend expone un endpoint que, dada una ventana temporal y una granularidad (diaria / semanal / mensual), proyecta las ejecuciones futuras de los movimientos recurrentes activos de tipo egreso del usuario autenticado, a partir de su próxima ejecución y hasta la fecha de fin o su expiración.
- La respuesta agrupa las ejecuciones por período e incluye, por cada uno, la fecha de inicio, el monto total proyectado y el detalle de cada ejecución (descripción, monto, categoría, cuenta destino); un usuario no puede consultar proyecciones ajenas.
- Se agrega una sección "Gastos futuros" dentro de la página de Schedule, respetando la estructura de tabs de la aplicación.
- El usuario selecciona el horizonte temporal mediante un rango de fechas y la granularidad, y la vista responde a esos cambios sin recargar la página.
- La información se presenta en una tabla agrupada por período (fecha, monto total y, al expandir cada fila, el detalle de ejecuciones), con el total acumulado del horizonte como resumen en la parte superior.
- Se incluye un gráfico de evolución en el tiempo, integrado en la misma sección y con el mismo estilo visual que el de estadísticas de transacciones, compartiendo los controles de rango y granularidad.
- La carga de datos utiliza los patrones de Suspense ya establecidos y muestra un estado vacío informativo cuando el usuario no tiene gastos recurrentes de egreso activos.

---

### Mejoras de Accesibilidad (WCAG 2.1 AA)

**Actores:** Usuario

**Funcionalidad:** Se aplicaron las correcciones a partir de los criterios WCAG 2.1 nivel AA en las páginas principales (Dashboard, Upload, Transactions, Schedule y User), priorizando contraste de color, semántica HTML, accesibilidad de formularios y navegación por teclado. Al usar Ant Design, la corrección se centralizó en los tokens del tema siempre que fue posible.

**Valor:** Mejora la usabilidad de la aplicación y la calidad general de la interfaz, cumpliendo un estándar reconocido de accesibilidad para los usuarios.

**Criterios de aceptación:**
- Los textos sobre fondos personalizados (estados, etiquetas, montos, mensajes de error, controles en estado de error, botones de acción) cumplen el ratio de contraste mínimo de 4.5:1 (texto normal) y 3:1 (texto grande y componentes de UI), incluidas las etiquetas y series de los gráficos.
- Las correcciones de paleta están centralizadas en los tokens del tema, el archivo de variables o constantes de color, evitando estilos inline dispersos.
- Cada página tiene un único `h1` y una jerarquía de headings sin saltos; las regiones (`main`, `nav`) usan elementos semánticos y existe un skip-link al contenido principal.
- Los íconos decorativos están ocultos a la tecnología asistiva y los funcionales tienen un nombre accesible; los campos de formulario tienen label asociado programáticamente y los controles compuestos que solo exponían `title` reciben un nombre accesible explícito.
- Los campos requeridos están marcados como tales, sin atributos ARIA inválidos para el rol del elemento.
- Los elementos interactivos son alcanzables y operables con teclado (incluidos elementos clickeables personalizados y controles de tablas y tabs), y los modales mantienen el foco atrapado devolviéndolo al disparador al cerrarse.
- Se integró `@axe-core` en modo desarrollo y se auditaron las páginas principales con axe-core y Lighthouse, alcanzando 0 incidencias automáticas por página, complementado con una verificación manual de navegación por teclado.

---

### Base de Testing del Frontend

**Actor:** Devs

**Funcionalidad:** El frontend no contaba con pruebas automatizadas. Se incorporó Vitest junto con React Testing Library y MSW como infraestructura base de testing, definiendo la configuración y las convenciones, y se escribieron los tests unitarios y de integración que cubren las funcionalidades y componentes más críticos de las páginas principales (Subir, Usuario, Dashboard, Transacciones y Movimientos recurrentes).

**Valor:** Establece una red de seguridad que permite evolucionar la aplicación con confianza, detectando regresiones de forma temprana y documentando el comportamiento esperado de los componentes clave.

**Criterios de aceptación:**
- Vitest queda configurado como test runner integrado con Vite (con jsdom como entorno, matchers de `jest-dom` globales, `globals: true` y setup compartido), y MSW interceptando las llamadas HTTP de los tests que dependen de la API.
- Se definen los scripts `test` y `test:coverage`, con el reporte de cobertura sobre `src/` excluyendo configuración y mocks, y los archivos de test conviven con su componente bajo la convención `NombreComponente.test.tsx`.
- Los tests siguen convenciones consistentes: estructura AAA, render con el helper `renderWithProviders`, mock de contextos y hooks a nivel de módulo con fixtures reutilizables, selección por rol/label/testid y textos vía i18next, y red mockeada con MSW en lugar del backend real.
- La sección Subir queda cubierta en sus tres vías (comprobantes, manual y voz), incluyendo servicios, listas y estados, modales de subida y confirmación, el grabador de audio y sus estados de error.
- La autenticación queda cubierta (servicio de token y JWT, ruta protegida en sus tres estados de sesión, página de usuario autenticada/no autenticada y ciclo de sesión del contexto).
- El Dashboard queda cubierto en el renderizado de datos, estados de carga y error, y el flujo completo de creación de cuenta bancaria (validaciones, éxito y cancelación).
- La página de Transacciones queda cubierta en sus solapas, el listado y sus estados, el panel de filtros (comportamiento de borrador y cada filtro) y el flujo de edición con payload parcial.
- La página de Movimientos recurrentes queda cubierta en la creación (validaciones y flujo exitoso), la edición con payload parcial, el listado y sus estados, y el flujo de desactivación con confirmación.
