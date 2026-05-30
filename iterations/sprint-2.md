# Sprint 2 — Documentación de la Iteración

## Resumen Ejecutivo

### Qué se añadió en esta iteración

- **Inicio de sesión seguro**: los usuarios inician sesión con su cuenta de Google y toda la aplicación está protegida para que cada persona pueda acceder solamente a sus propios datos financieros.
- **Categorías personales**: los usuarios pueden crear sus propias categorías y asignarlas a transacciones e ingresos fijos, además de poder filtrar por cada una.
- **Revisión de carga de recibo**: después de subir un recibo, el sistema muestra lo que interpretó para que el usuario pueda verificarlo y corregirlo antes de guardarlo, así se reducen los errores de la lectura automática.
- **Fechas locales correctas**: las transacciones se registran en la propia zona horaria del usuario y no se permiten fechas futuras.
- **Gestión de ingresos fijos**: los usuarios pueden ver, editar y filtrar sus ingresos programados en una sección dedicada.
- **Vista dedicada de transacciones y categorías**: las transacciones ahora tienen su propia sección a parte del Dashboard, compartida con el manejo de categorías.

### Decisiones tomadas

- **Delegación de Inicio de Sesión**: el inicio de sesión con Google evita tener otra contraseña y almacenar credenciales, lo que mejora la seguridad.
- **Revisión antes de guardar para los recibos**: la lectura automática puede malinterpretar un recibo, por lo que un paso de confirmación y edición mantiene la precisión de los datos.
- **Identidad vinculada al usuario conectado**: cada acción está ligada al usuario autenticado, de modo que las personas solo ven y modifican sus propios datos.
- **Soporte para zonas horarias**: las fechas se almacenan en la hora local de cada usuario para que los registros sigan siendo precisos independientemente del servidor.
- **Categorías por usuario**: las categorías pertenecen a cada usuario y se comportan de la misma manera tanto en las transacciones como en los ingresos recurrentes.

---

## User Stories

### Autenticación con Google OAuth

**Actores:** Usuario no registrado, usuario registrado

**Funcionalidad:** Permite autenticarse mediante una cuenta de Google, sin necesidad de crear credenciales locales. El flujo contempla tanto el login de usuarios existentes como el registro automático de nuevos usuarios, solicitando el CUIL para completar el alta.

**Valor:** Mejora la seguridad al delegar la autenticación a Google, y facilita la incorporación de nuevos usuarios eliminando el formulario de registro tradicional. 

**Criterios de aceptación:**
- En la pantalla de inicio de sesión existe un botón para autenticarse con Google.
- Si el usuario ya existe, el login con Google lo autentica y redirige al dashboard.
- Si el usuario no existe, el sistema inicia un flujo de pre-registro y solicita el CUIL para completar el alta.
- No se puede crear el usuario definitivo sin un CUIL válido y único.
- No puede existir más de un usuario con el mismo email o CUIL.
- Al completar el registro, el usuario queda autenticado automáticamente y se redirige al dashboard.
- Al refrescar la página, la sesión persiste.
- El usuario puede cerrar sesión.
- Si no está autenticado, el usuario es redirigido al login y no puede acceder a otras secciones.
- Si falla el login, se muestra un mensaje de error claro y se permite reintentar.
  
---

### Gestión de Ingresos Fijos

**Actor:** Usuario

**Funcionalidad:** Nueva pestaña dedicada a ingresos recurrentes (sueldos, alquileres, etc.) donde el usuario puede dar de alta nuevos ingresos fijos, visualizar los existentes y editar sus datos.

**Valor:** Permite llevar un registro ordenado de ingresos periódicos, facilitando la planificación financiera personal.

**Criterios de aceptación:**
- Existe un nuevo acceso en el NavBar (ícono de calendario) que navega a la pestaña de Ingresos Fijos.
- La pestaña incluye un formulario de alta con campos de monto, descripción, frecuencia, fechas y categoría.
- Se muestra un listado de los ingresos fijos activos del usuario autenticado.
- El listado mantiene el mismo diseño visual que el listado de transacciones.
- Cada ítem del listado tiene un botón de edición que abre un modal para modificar los datos.
- Solo se pueden editar ingresos fijos que pertenezcan al usuario autenticado.
- Los cambios se persisten correctamente en la base de datos.
- Se valida que los datos sean correctos (monto positivo).

---

### Categorización de Transacciones e Ingresos Fijos

**Actor:** Usuario

**Funcionalidad:** Sistema de categorías personalizables para clasificar transacciones e ingresos fijos. El usuario puede crear, editar y eliminar categorías propias con nombre e ícono, asignarlas al registrar o editar movimientos, visualizarlas en los listados y filtrar por ellas.

**Valor:** Mejora la organización financiera personal al permitir agrupar y filtrar movimientos por tipo (ej: comida, transporte, salud), facilitando el análisis de gastos e ingresos.

**Criterios de aceptación:**
- Existe una sección para crear, editar y eliminar categorías personalizadas.
- Cada categoría tiene nombre e ícono seleccionable desde un conjunto predefinido.
- Los formularios de creación y edición de transacciones e ingresos fijos incluyen un selector de categoría.
- Los listados muestran el ícono y/o nombre de la categoría asignada en cada ítem.
- Se puede filtrar el listado de transacciones por una o varias categorías.
- Solo se puede asignar una categoría que pertenezca al usuario autenticado.
- La categoría es un campo opcional en transacciones e ingresos fijos.

---

### Validación Manual de Comprobantes Procesados por OCR

**Actor:** Usuario

**Funcionalidad:** Luego de la carga de un comprobante, se muestra al usuario la imagen del comprobante y un formulario de revisión y edición de los datos extraídos para corregir campos antes de que la transacción se guarde.

**Valor:** Garantiza la integridad y exactitud de los datos financieros registrados, evitando que errores del OCR se persistan sin control.

**Criterios de aceptación:**
- Tras el procesamiento OCR, el comprobante queda en estado pendiente de confirmación y la transacción no se persiste aún.
- Se muestra un formulario con los datos extraídos por el OCR y una vista previa de la imagen del comprobante.
- Todos los campos del formulario son editables por el usuario.
- Los campos que el OCR no pudo extraer se muestran vacíos/invalidados para completar manualmente.
- Existe un botón de confirmación explícita para finalizar la carga.
- La transacción solo se crea y persiste tras la confirmación del usuario.
- Se mantiene registro de los datos originales del OCR y los datos finales confirmados.
- Si ocurre un error al confirmar, se muestra un mensaje claro y se permite reintentar sin volver a cargar el comprobante.

---

### Pestaña de Transacciones con Gestión de Categorías

**Actor:** Usuario autenticado

**Funcionalidad:** Nueva pestaña de Transacciones accesible desde el NavBar que centraliza el historial de transacciones con opción de edición y la administración de categorías personalizadas, organizadas en tabs internos para una navegación clara.

**Valor:** Centraliza en un único lugar toda la gestión relacionada con transacciones y categorías, mejorando la usabilidad y organización de la interfaz.

**Criterios de aceptación:**
- El NavBar incluye un nuevo botón que navega a la pestaña de Transacciones.
- La pestaña muestra el listado histórico de transacciones con opción de edición de cada registro.
- Dentro de la misma pestaña hay una sección para gestionar categorías (crear y modificar).
- El layout está organizado en tabs internos: listado de transacciones y panel de categorías.

---

### Manejo Consistente de Fechas y Zonas Horarias

**Actores:** Usuario autenticado, sistema

**Funcionalidad:** Implementación de una política estricta donde la zona horaria y las fechas se muestran al usuario en su huso horario local.

**Valor:** Garantiza consistencia e integridad en los datos temporales de contexto financiero/contable, evitando errores por diferencias de zona horaria, relojes mal configurados o discrepancias entre OCR e ingreso manual.

**Criterios de aceptación:**
- Todos los timestamps se almacenan en UTC en la base de datos.
- El frontend envía fechas en formato ISO 8601 con offset explícito (Z o +HH:MM).
- El payload incluye la zona horaria IANA del usuario (ej: America/Argentina/Buenos_Aires).
- El backend rechaza cualquier fecha enviada sin offset explícito.
- La validación de fecha futura es responsabilidad del backend, no del reloj del cliente.
- La UI muestra las fechas en el huso horario del usuario.
- El esquema de transacciones incluye los campos: transaction_date, transaction_date_source, uploaded_at, created_at y user_timezone.

