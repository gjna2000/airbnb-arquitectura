# 3. Descripción de los módulos del sistema

La plataforma se organiza en módulos con responsabilidades bien definidas.
Cada módulo expone una API y es dueño de sus propios datos, lo que permite
desarrollarlos, desplegarlos y escalarlos de forma independiente.

## 3.1 Resumen de módulos

| # | Módulo | Responsabilidad principal | Requerimientos |
|---|---|---|---|
| M1 | Usuarios y autenticación | Registro, inicio de sesión, perfiles y verificación de identidad | RF-01, RF-02 |
| M2 | Alojamientos | Publicación y administración de propiedades | RF-03, RF-04 |
| M3 | Disponibilidad y calendario | Control de fechas libres, bloqueadas y reservadas | RF-05 |
| M4 | Precios dinámicos | Cálculo de tarifas según temporada, demanda y estadía | RF-06, RF-18 |
| M5 | Búsqueda y filtros | Búsqueda por destino, fechas, precio y tipo; vista en mapa | RF-07, RF-08 |
| M6 | Reservas | Creación, confirmación y cancelación de reservas | RF-09, RF-10, RF-11 |
| M7 | Pagos | Cobro al huésped, pago al anfitrión y reembolsos | RF-12, RF-13 |
| M8 | Reseñas y reputación | Calificaciones y comentarios entre huésped y anfitrión | RF-14 |
| M9 | Mensajería | Chat entre huésped y anfitrión | RF-15 |
| M10 | Notificaciones | Envío de correos y notificaciones push | RF-16 |
| M11 | Experiencias | Catálogo y reserva de experiencias adicionales | RF-17 |
| M12 | Administración y soporte | Moderación, reportes e incidencias | RF-19, RF-20 |

## 3.2 Detalle de cada módulo

### M1. Usuarios y autenticación
- **Descripción:** gestiona las cuentas de huéspedes, anfitriones y administradores.
- **Funciones:** registro, login (correo y cuentas de terceros), recuperación de contraseña, perfil, verificación de identidad, roles y permisos.
- **Datos que maneja:** usuarios, credenciales (hash), roles, documentos de verificación.
- **Depende de:** Notificaciones (correos de verificación).

### M2. Alojamientos
- **Descripción:** administra el catálogo de propiedades publicadas.
- **Funciones:** crear, editar, pausar y eliminar publicaciones; cargar fotos; definir reglas, servicios y ubicación.
- **Datos que maneja:** propiedades, fotos (en almacenamiento de objetos), reglas, tarifa base.
- **Depende de:** Usuarios (anfitrión dueño), servicio de mapas.

### M3. Disponibilidad y calendario
- **Descripción:** mantiene el estado de cada fecha de cada alojamiento.
- **Funciones:** bloquear fechas, liberar fechas, consultar disponibilidad, evitar dobles reservas.
- **Datos que maneja:** calendario por alojamiento (libre, bloqueado, reservado).
- **Depende de:** Alojamientos, Reservas.

### M4. Precios dinámicos
- **Descripción:** sugiere o calcula el precio final según condiciones del mercado.
- **Funciones:** ajuste por temporada, demanda y eventos; descuentos por estadías largas; cálculo de tarifas de servicio e impuestos.
- **Datos que maneja:** reglas de precio, historial de tarifas.
- **Depende de:** Alojamientos, Disponibilidad.

### M5. Búsqueda y filtros
- **Descripción:** permite encontrar alojamientos de forma rápida.
- **Funciones:** búsqueda por destino, fechas, huéspedes, precio y tipo; ordenamiento; resultados en lista y mapa.
- **Datos que maneja:** índice de búsqueda (copia optimizada de los alojamientos).
- **Depende de:** Alojamientos, Disponibilidad, Precios, servicio de mapas.

### M6. Reservas
- **Descripción:** coordina todo el proceso de reserva.
- **Funciones:** crear reserva, reserva instantánea o por solicitud, confirmación del anfitrión, cancelación según política.
- **Datos que maneja:** reservas, estados (pendiente, confirmada, cancelada, finalizada).
- **Depende de:** Disponibilidad, Precios, Pagos, Notificaciones.

### M7. Pagos
- **Descripción:** gestiona el dinero de forma segura a través de una pasarela externa.
- **Funciones:** cobro al huésped, retención del pago, transferencia al anfitrión tras el check-in, reembolsos.
- **Datos que maneja:** transacciones y estados de pago (nunca datos de tarjeta).
- **Depende de:** Pasarela de pagos externa, Reservas.

### M8. Reseñas y reputación
- **Descripción:** construye la confianza entre usuarios.
- **Funciones:** calificar y comentar tras la estadía, calcular promedios, moderar reseñas.
- **Datos que maneja:** reseñas y puntuaciones.
- **Depende de:** Reservas (solo se reseña una estadía finalizada).

### M9. Mensajería
- **Descripción:** comunicación directa entre huésped y anfitrión.
- **Funciones:** chat por reserva, historial, adjuntos, avisos de mensajes nuevos.
- **Datos que maneja:** conversaciones y mensajes.
- **Depende de:** Usuarios, Notificaciones.

### M10. Notificaciones
- **Descripción:** centraliza todos los avisos al usuario.
- **Funciones:** envío de correos y notificaciones push, plantillas, preferencias del usuario.
- **Datos que maneja:** plantillas, historial de envíos.
- **Depende de:** Servicio externo de correo/push.

### M11. Experiencias
- **Descripción:** ofrece actividades adicionales al alojamiento.
- **Funciones:** publicar, buscar y reservar experiencias (gastronómicas, culturales, aventuras).
- **Datos que maneja:** experiencias, cupos y horarios.
- **Depende de:** Usuarios, Reservas, Pagos.

### M12. Administración y soporte
- **Descripción:** herramientas internas para operar la plataforma.
- **Funciones:** moderar publicaciones y usuarios, atender reportes e incidencias, ver métricas.
- **Datos que maneja:** reportes, casos de soporte, registros de auditoría.
- **Depende de:** todos los módulos (lectura y acciones de moderación).

## 3.3 Relación entre módulos (resumen)

| Módulo | Se comunica con |
|---|---|
| Reservas | Disponibilidad, Precios, Pagos, Notificaciones |
| Búsqueda | Alojamientos, Disponibilidad, Precios |
| Pagos | Reservas, Pasarela externa |
| Reseñas | Reservas |
| Mensajería | Usuarios, Notificaciones |