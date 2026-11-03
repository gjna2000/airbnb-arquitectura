# 8. Propuesta de tecnologías y justificación

Las tecnologías se eligieron según los atributos de calidad priorizados
(seguridad, disponibilidad, escalabilidad, rendimiento y mantenibilidad) y
según su madurez, comunidad y disponibilidad de documentación.

## 8.1 Resumen por capa

| Capa | Tecnología propuesta | Alternativas evaluadas |
|---|---|---|
| Frontend web | React + Next.js | Angular, Vue |
| Backend (servicios) | Node.js con NestJS (TypeScript) | Java Spring Boot, Python FastAPI |
| API Gateway | Kong | NGINX, AWS API Gateway |
| Base de datos transaccional | PostgreSQL | MySQL, SQL Server |
| Base de datos documental | MongoDB | DynamoDB, Cassandra |
| Caché y sesiones | Redis | Memcached |
| Motor de búsqueda | OpenSearch | Elasticsearch, Algolia |
| Mensajería de eventos | Apache Kafka (o SQS) | RabbitMQ |
| Almacenamiento de archivos | Amazon S3 | Google Cloud Storage, Azure Blob |
| Contenedores | Docker | Podman |
| Orquestación | Kubernetes (EKS) | ECS, Docker Swarm |
| Nube | AWS | Azure, Google Cloud |
| CI/CD | GitHub Actions | Jenkins, GitLab CI |
| Monitoreo y logs | Prometheus, Grafana, ELK | Datadog, New Relic |
| Seguridad | OAuth 2.0, JWT, TLS, WAF, KMS | Keycloak, Auth0 |
| Pagos | Stripe / PayU | PayPal, Mercado Pago |
| Mapas | Google Maps API | Mapbox, OpenStreetMap |
| Correo y push | Amazon SES, Firebase Cloud Messaging | SendGrid, OneSignal |

## 8.2 Justificación de las decisiones

### Frontend: React + Next.js
- Permite renderizado en servidor (SSR), lo que mejora la velocidad de carga y el posicionamiento en buscadores (SEO), clave en una plataforma de alojamientos.
- Interfaz responsive y basada en componentes reutilizables (RNF-10).
- Gran comunidad y abundante documentación.

### Backend: Node.js con NestJS
- NestJS impone una estructura modular, lo que facilita separar los módulos M1 a M12 (RNF-09).
- TypeScript reduce errores y mejora el mantenimiento.
- Buen rendimiento en operaciones de entrada y salida, típicas de una plataforma con muchas consultas concurrentes.
- Un solo lenguaje en frontend y backend simplifica el equipo de desarrollo.

### API Gateway: Kong
- Centraliza autenticación, enrutamiento y límite de peticiones.
- Es de código abierto y se integra bien con Kubernetes.

### PostgreSQL (datos transaccionales)
- Soporta transacciones ACID, indispensables para reservas y pagos (RNF-08).
- Permite bloqueos y restricciones que evitan dobles reservas.
- Réplicas de lectura para descargar consultas.

### MongoDB (mensajes y reseñas)
- Esquema flexible y alto volumen de escritura, adecuados para chats y comentarios.
- Escala horizontalmente con facilidad.

### Redis (caché y sesiones)
- Respuestas en memoria, en milisegundos, que ayudan a cumplir el tiempo de búsqueda menor a 2 segundos (RNF-01).

### OpenSearch (búsqueda)
- Búsqueda por texto, filtros múltiples y geolocalización en una sola consulta.
- Evita sobrecargar la base de datos transaccional.

### Kafka / SQS (eventos)
- Comunicación asíncrona que desacopla los servicios.
- Permite que Notificaciones y Búsqueda reaccionen a eventos sin frenar a Reservas.
- Kafka para alto volumen; SQS si se prefiere un servicio administrado más simple.

### S3 (archivos)
- Almacenamiento barato, duradero y prácticamente ilimitado para fotos.
- Se integra con la CDN para entregar imágenes rápido.

### Docker + Kubernetes (EKS)
- Cada servicio se empaqueta igual en cualquier ambiente.
- Kubernetes escala automáticamente y reinicia contenedores caídos (RNF-02, RNF-03).

### AWS
- Cubre todos los servicios requeridos (bases administradas, CDN, WAF, KMS, balanceadores).
- Región São Paulo cercana a Colombia.
- Servicios administrados reducen la carga operativa del equipo.

### GitHub Actions (CI/CD)
- Integrado con el repositorio, sin servidores adicionales que mantener.
- Automatiza pruebas, construcción de imágenes y despliegue.

### Prometheus, Grafana y ELK (observabilidad)
- Estándar de la industria, de código abierto y compatible con Kubernetes (RNF-12).

### Stripe / PayU (pagos)
- Cumplen PCI DSS, así el sistema nunca almacena datos de tarjeta (RNF-06).
- PayU tiene cobertura sólida en Colombia y Latinoamérica.

## 8.3 Patrón arquitectónico elegido

Se adopta una **arquitectura de microservicios** con comunicación síncrona
(REST) para consultas inmediatas y asíncrona (eventos) para procesos
secundarios.

| Aspecto | Ventaja para Airbnb |
|---|---|
| Escalado independiente | Búsqueda y Reservas se escalan según su propia demanda |
| Despliegue independiente | Un cambio en Reseñas no obliga a redesplegar Pagos |
| Aislamiento de fallos | Si Mensajería falla, las reservas siguen funcionando |
| Tecnología por necesidad | Cada servicio puede usar la base de datos más adecuada |

**Contrapartida:** más complejidad operativa y consistencia eventual entre
servicios. Se mitiga con API Gateway, observabilidad y eventos bien definidos.
Para un equipo pequeño o un MVP inicial, se podría comenzar con un
**monolito modular** y separar los servicios a medida que crezca la demanda.