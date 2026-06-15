# Sprint 3 — Documentación de la Iteración

## Resumen Ejecutivo

### Qué se añadió en esta iteración

- **Estadísticas visuales de movimientos**: los usuarios cuentan con una sección de estadísticas donde pueden ver gráficos de sus ingresos y egresos a lo largo del tiempo y rankings de cuánto movieron por categoría y por cuenta bancaria, con filtros para acotar el análisis.
- **Gastos recurrentes**: además de los ingresos fijos, los usuarios ahora pueden programar gastos recurrentes (servicios, suscripciones, etc.) desde el cronograma, verlos en un listado, editarlos y darlos de baja.
- **Comprobantes sin esperas**: al subir un comprobante el usuario ya no queda bloqueado esperando que se procese; puede seguir usando la aplicación y consultar en cualquier momento una lista con el estado de cada comprobante.
- **Confirmación desde la lista**: los comprobantes que quedan pendientes de revisión se pueden confirmar directamente desde la lista, retomando el flujo de validación cuando al usuario le resulte conveniente.

### Decisiones tomadas

- **Selección de librería de gráficos mediante evaluación**: antes de implementar las visualizaciones se compararon distintas librerías según su compatibilidad, facilidad para construir los gráficos necesarios e integración visual, y se eligió la más adecuada para el proyecto.
- **Generalización de los movimientos recurrentes**: en lugar de crear un modelo separado para los gastos, se amplió el concepto de "ingreso recurrente" a "movimiento recurrente", de modo que ingresos y gastos comparten la misma lógica de programación, edición y baja sin duplicar comportamiento.
- **Procesamiento de comprobantes en segundo plano**: la lectura automática de un comprobante puede demorar, por lo que se desacopló de la subida para que el usuario no tenga que esperar y pueda seguir trabajando mientras el comprobante se procesa.
- **OCR como servicio independiente**: la lectura automática de comprobantes se separó en un servicio independiente que el backend consulta cuando lo necesita, en lugar de tenerla embebida en la aplicación principal. Esto permite escalar y mantener el procesamiento de imágenes por separado, sin afectar al resto del sistema.
- **Estado visible de cada comprobante**: como el procesamiento es asíncrono, cada comprobante expone su estado (pendiente, a confirmar, procesado o fallido) para que el usuario sepa en todo momento qué pasó con cada carga.

---

## User Stories

### Visualización de Estadísticas con Gráficos

**Actor:** Usuario

**Funcionalidad:** Como usuario quiero visualizar gráficos de mis ingresos y egresos para tener un mejor análisis y entendimiento de mis movimientos. La sección de estadísticas combina un gráfico de evolución en el tiempo con rankings de cuánto se movió por categoría y por cuenta bancaria, todo acotado por filtros.

**Valor:** Permite entender de un vistazo dónde se concentran los ingresos y gastos, en qué períodos hay más actividad y qué categorías o cuentas pesan más, facilitando la toma de decisiones financieras.

**Criterios de aceptación:**
- Existe un nuevo tab de "Estadísticas" dentro de la pantalla de transacciones.
- El tab ofrece filtros de rango de fechas (por defecto, el último mes hasta hoy), granularidad temporal (día, semana o mes; por defecto semana), banco y categorías (multi-selección).
- Al cambiar cualquier filtro, los gráficos se actualizan automáticamente.
- Se muestra un gráfico de línea con la evolución de los montos enviados y recibidos por período, con dos series que el usuario puede mostrar u ocultar.
- Al pasar el cursor sobre el gráfico de línea se muestra el período, la cantidad de transacciones de ese período y el monto de cada serie.
- Se muestran dos rankings en forma de barras horizontales: uno por categoría y otro por cuenta bancaria.
- Cada barra indica el monto y su porcentaje sobre el total, y el ranking se ordena de mayor a menor.
- Las transacciones sin categoría se agrupan bajo "Sin categoría".
- Los rankings comparten los filtros de fecha y tipo del tab y se actualizan al cambiarlos.

---

### Gestión de Gastos Recurrentes

**Actor:** Usuario

**Funcionalidad:** Como usuario quiero programar gastos recurrentes (además de los ingresos) desde el cronograma, verlos en un listado, editarlos y darlos de baja, para llevar un control completo de mis movimientos periódicos.

**Valor:** Permite anticipar y ordenar tanto los ingresos como los gastos periódicos (servicios, suscripciones, alquileres), brindando una visión más realista de la planificación financiera.

**Criterios de aceptación:**
- Dentro del cronograma existe una sección para crear gastos recurrentes.
- El formulario de alta incluye monto, frecuencia (semanal, quincenal o mensual), cuenta, categoría, descripción y una fecha de fin opcional.
- Se valida que los campos requeridos estén completos y que el monto sea mayor a cero.
- Al guardar se confirma la creación con un mensaje de éxito.
- Se muestra un mensaje aclaratorio de que el monto podrá ajustarse al confirmar cada período, contemplando servicios de monto variable (por ejemplo, la luz).
- Se muestra un listado de los gastos recurrentes del usuario con su monto, frecuencia, cuenta y estado (activo o dado de baja).
- Cada movimiento recurrente se puede editar.
- Cada movimiento recurrente se puede dar de baja mediante un botón con confirmación, dejando de ejecutarse a futuro.
- Los movimientos recurrentes pueden tener una fecha de expiración opcional a partir de la cual dejan de ejecutarse.
- Un usuario solo puede crear y editar movimientos recurrentes sobre sus propias cuentas.

---

### Confirmación de Comprobantes sin Esperas

**Actor:** Usuario

**Funcionalidad:** Como usuario quiero que al subir un comprobante la aplicación no se bloquee mientras se procesa, poder ver una lista con el estado de mis comprobantes y confirmar desde ahí los que hayan quedado pendientes de revisión.

**Valor:** Mejora la experiencia al eliminar la espera durante el procesamiento automático y dar visibilidad del estado de cada comprobante, permitiendo retomar la confirmación cuando al usuario le resulte conveniente.

**Criterios de aceptación:**
- La pantalla de Upload se organiza en dos sub-tabs: "Comprobantes" y "Manual".
- El sub-tab "Comprobantes" contiene la subida de comprobantes y es el activo por defecto al entrar a la pantalla.
- El sub-tab "Manual" mantiene el formulario de transacción manual existente sin cambios.
- Al subir un comprobante, la ventana de carga se cierra de inmediato y se muestra un aviso de que el comprobante se está procesando.
- Mientras el comprobante se procesa, el usuario puede seguir navegando libremente por la aplicación.
- El sub-tab "Comprobantes" muestra la lista de comprobantes del usuario con su fecha y estado, con un indicador visual diferenciado en español para los estados pendiente, a confirmar, procesado y fallido.
- La lista se actualiza al entrar al sub-tab y mientras haya comprobantes en proceso.
- Al hacer clic en un comprobante a confirmar, se abre el formulario de confirmación existente con los datos extraídos precargados, sin modificar el flujo de validación.
