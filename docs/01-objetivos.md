# 1. Objetivos del diseño arquitectónico

## 1.1 Objetivo general

Diseñar y documentar una arquitectura de software escalable, segura y modular
para una plataforma de gestión de alojamientos tipo Airbnb, que conecte
anfitriones con huéspedes y soporte el proceso completo de publicación,
búsqueda, reserva, pago y valoración.

## 1.2 Objetivos específicos

1. Identificar los requerimientos funcionales y no funcionales que guían las
   decisiones de diseño de la plataforma.
2. Definir una arquitectura modular que separe las responsabilidades del
   sistema (usuarios, alojamientos, búsqueda, reservas, pagos, reseñas,
   mensajería y notificaciones).
3. Representar la arquitectura mediante diagramas de contexto, componentes,
   despliegue y secuencia.
4. Seleccionar y justificar las tecnologías más adecuadas para cada capa del
   sistema.
5. Garantizar la seguridad de los datos personales y de los pagos, la
   capacidad de crecer con la demanda y la facilidad de mantenimiento.

## 1.3 Alcance

El diseño cubre las funcionalidades principales de la plataforma web: gestión
de usuarios, publicación y administración de alojamientos, búsqueda con
filtros, reservas, pagos, reseñas, mensajería, notificaciones y
administración. No se implementa código: el entregable es la documentación
arquitectónica.

## 1.4 Atributos de calidad priorizados

| Prioridad | Atributo | Razón |
|---|---|---|
| 1 | Seguridad | Se manejan datos personales y transacciones de pago |
| 2 | Disponibilidad | Las reservas no pueden fallar en temporadas altas |
| 3 | Escalabilidad | Millones de usuarios y búsquedas concurrentes |
| 4 | Rendimiento | La búsqueda debe responder en tiempos muy cortos |
| 5 | Mantenibilidad | El sistema debe evolucionar sin afectar todos los módulos |