# 2. Requerimientos del sistema

## 2.1 Actores

| Actor | Descripción |
|---|---|
| Huésped | Busca, reserva y paga alojamientos |
| Anfitrión | Publica y administra sus propiedades y reservas |
| Administrador | Modera contenido, usuarios y resuelve incidencias |
| Pasarela de pagos | Sistema externo que procesa cobros y pagos |
| Servicio de mapas | Sistema externo para geolocalización |
| Servicio de correo/push | Sistema externo para notificaciones |

## 2.2 Requerimientos funcionales

| ID | Módulo | Requerimiento |
|---|---|---|
| RF-01 | Usuarios | El sistema debe permitir registrarse e iniciar sesión con correo o cuentas de terceros (Google, Facebook). |
| RF-02 | Usuarios | El sistema debe permitir editar el perfil y verificar la identidad del usuario. |
| RF-03 | Alojamientos | El anfitrión debe poder publicar un alojamiento con fotos, descripción, tarifas, reglas y ubicación. |
| RF-04 | Alojamientos | El anfitrión debe poder editar, pausar o eliminar sus publicaciones. |
| RF-05 | Disponibilidad | El anfitrión debe poder gestionar el calendario y bloquear fechas. |
| RF-06 | Precios | El sistema debe ofrecer herramientas de precios dinámicos según temporada y demanda. |
| RF-07 | Búsqueda | El huésped debe poder buscar alojamientos por destino, fechas, número de huéspedes, precio y tipo. |
| RF-08 | Búsqueda | El sistema debe mostrar los resultados en lista y en mapa. |
| RF-09 | Reservas | El huésped debe poder reservar un alojamiento disponible en las fechas elegidas. |
| RF-10 | Reservas | El anfitrión debe poder aceptar o rechazar solicitudes, según el tipo de reserva. |
| RF-11 | Reservas | El sistema debe aplicar la política de cancelación definida por el anfitrión. |
| RF-12 | Pagos | El sistema debe procesar el pago del huésped de forma segura mediante una pasarela. |
| RF-13 | Pagos | El sistema debe transferir el pago al anfitrión tras el check-in y gestionar reembolsos. |
| RF-14 | Reseñas | Huésped y anfitrión deben poder calificarse y comentar tras finalizar la estadía. |
| RF-15 | Mensajería | Huésped y anfitrión deben poder comunicarse dentro de la plataforma. |
| RF-16 | Notificaciones | El sistema debe enviar notificaciones de reservas, pagos y mensajes por correo y push. |
| RF-17 | Experiencias | El usuario debe poder consultar y reservar experiencias adicionales. |
| RF-18 | Estancias largas | El sistema debe aplicar descuentos mensuales en estadías prolongadas. |
| RF-19 | Administración | El administrador debe poder moderar publicaciones, usuarios y reportes. |
| RF-20 | Soporte | El usuario debe poder reportar incidencias y contactar con soporte. |

## 2.3 Requerimientos no funcionales

| ID | Categoría | Requerimiento |
|---|---|---|
| RNF-01 | Rendimiento | Las búsquedas deben responder en menos de 2 segundos en el 95 % de los casos. |
| RNF-02 | Escalabilidad | El sistema debe escalar horizontalmente para soportar picos de tráfico (temporadas altas). |
| RNF-03 | Disponibilidad | La disponibilidad objetivo es del 99,9 % mensual. |
| RNF-04 | Seguridad | Toda comunicación debe usar HTTPS/TLS y los datos sensibles deben almacenarse cifrados. |
| RNF-05 | Seguridad | La autenticación debe usar tokens (JWT/OAuth 2.0) y control de acceso por roles. |
| RNF-06 | Seguridad | Los pagos deben cumplir el estándar PCI DSS; los datos de tarjeta nunca se almacenan en el sistema. |
| RNF-07 | Privacidad | El tratamiento de datos personales debe cumplir la normativa aplicable (p. ej., Ley 1581 de 2012 en Colombia). |
| RNF-08 | Consistencia | No debe permitirse una doble reserva de las mismas fechas para un mismo alojamiento. |
| RNF-09 | Mantenibilidad | El sistema debe estar organizado en módulos desacoplados con APIs documentadas. |
| RNF-10 | Usabilidad | La interfaz debe ser responsive y accesible desde navegador de escritorio y móvil. |
| RNF-11 | Interoperabilidad | La integración con servicios externos debe hacerse mediante APIs REST. |
| RNF-12 | Observabilidad | El sistema debe registrar logs, métricas y alertas para detectar fallos. |
| RNF-13 | Recuperación | Debe haber copias de seguridad automáticas y un plan de recuperación ante desastres. |
| RNF-14 | Internacionalización | La plataforma debe soportar múltiples idiomas y monedas. |