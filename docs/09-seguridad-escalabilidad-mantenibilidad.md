# 9. Consideraciones de seguridad, escalabilidad y mantenibilidad

## 9.1 Seguridad

### 9.1.1 Autenticación y autorización

| Medida | Descripción | Requerimiento |
|---|---|---|
| OAuth 2.0 + JWT | Tokens firmados de corta duración, con renovación mediante refresh token | RNF-05 |
| Inicio de sesión con terceros | Google y Facebook mediante OAuth 2.0 | RF-01 |
| Autenticación multifactor (MFA) | Opcional para usuarios, obligatoria para administradores | RNF-05 |
| Control de acceso por roles (RBAC) | Huésped, anfitrión y administrador con permisos distintos | RNF-05 |
| Contraseñas | Almacenadas con hash seguro (bcrypt o Argon2), nunca en texto plano | RNF-04 |

### 9.1.2 Protección de datos

| Medida | Descripción | Requerimiento |
|---|---|---|
| Cifrado en tránsito | HTTPS/TLS 1.2 o superior en toda comunicación | RNF-04 |
| Cifrado en reposo | Bases de datos y S3 cifrados con claves de KMS | RNF-04 |
| Gestión de secretos | Credenciales en Secrets Manager, nunca en el código | RNF-04 |
| Datos de tarjeta | No se almacenan: solo tokens de la pasarela (PCI DSS) | RNF-06 |
| Protección de datos personales | Consentimiento, derecho a consulta, corrección y eliminación (Ley 1581 de 2012) | RNF-07 |
| Minimización de datos | Solo se guarda lo estrictamente necesario | RNF-07 |

### 9.1.3 Protección de la aplicación

| Amenaza | Mitigación |
|---|---|
| Inyección SQL | Consultas parametrizadas y ORM |
| XSS | Escape de salidas y política de seguridad de contenido (CSP) |
| CSRF | Tokens anti-CSRF y cookies SameSite |
| DDoS y bots | WAF y CDN en el borde, límite de peticiones en el API Gateway |
| Fuerza bruta en login | Bloqueo temporal tras intentos fallidos y MFA |
| Fraude en reservas y pagos | Verificación de identidad, reglas antifraude de la pasarela y revisión de actividad sospechosa |
| Suplantación entre servicios | Comunicación interna autenticada (mTLS) y redes privadas |
| Dependencias vulnerables | Análisis automático en el pipeline (Dependabot, escaneo de imágenes) |

### 9.1.4 Auditoría y trazabilidad
- Registro de acciones sensibles (inicios de sesión, cambios de precio, pagos, moderación).
- Logs centralizados e inmutables, consultables por el equipo de seguridad.
- Alertas ante comportamientos anómalos (RNF-12).

## 9.2 Escalabilidad

### 9.2.1 Estrategias aplicadas

| Estrategia | Cómo se aplica | Requerimiento |
|---|---|---|
| Escalado horizontal | Kubernetes agrega réplicas de cada servicio según CPU, memoria o peticiones | RNF-02 |
| Servicios sin estado | Las sesiones viven en Redis, no en el servidor, así cualquier instancia atiende cualquier petición | RNF-02 |
| Balanceo de carga | Reparte tráfico entre zonas de disponibilidad | RNF-03 |
| Caché en varios niveles | CDN (estáticos), Redis (consultas frecuentes) | RNF-01 |
| Réplicas de lectura | PostgreSQL distribuye las lecturas | RNF-01 |
| Índice de búsqueda separado | OpenSearch absorbe la carga de consultas | RNF-01 |
| Procesamiento asíncrono | Eventos para notificaciones e indexación | RNF-02 |
| Particionamiento (sharding) | Si crece el volumen, las reservas se particionan por región o fecha | RNF-02 |

### 9.2.2 Manejo de picos de tráfico
- **Autoescalado** anticipado en temporadas altas (vacaciones, festivos).
- **Colas** que absorben ráfagas de eventos sin perder mensajes.
- **Degradación controlada:** si un servicio secundario falla (reseñas, experiencias), la búsqueda y la reserva siguen operando.
- **Circuit breaker:** corta las llamadas a un servicio caído para evitar fallos en cadena.

### 9.2.3 Disponibilidad y recuperación

| Aspecto | Medida |
|---|---|
| Alta disponibilidad | Despliegue en dos zonas de disponibilidad |
| Base de datos | PostgreSQL Multi-AZ con conmutación automática |
| Copias de seguridad | Automáticas y diarias, con retención de 30 días |
| Recuperación ante desastres | Copias replicadas en otra región; objetivo de recuperación (RTO) de 1 hora y pérdida máxima de datos (RPO) de 15 minutos |
| Pruebas de resiliencia | Simulacros periódicos de falla de una zona |

## 9.3 Mantenibilidad

### 9.3.1 Diseño del código y de la arquitectura

| Práctica | Beneficio | Requerimiento |
|---|---|---|
| Módulos desacoplados | Cada servicio cambia sin afectar a los demás | RNF-09 |
| Una base de datos por servicio | Evita dependencias ocultas entre módulos | RNF-09 |
| APIs documentadas (OpenAPI/Swagger) | Contratos claros entre servicios y equipos | RNF-09 |
| Versionado de APIs | Los cambios no rompen a los clientes existentes | RNF-11 |
| Arquitectura por capas dentro de cada servicio | Separa controladores, lógica de negocio y acceso a datos | RNF-09 |
| TypeScript y convenciones de estilo | Código uniforme y con menos errores | RNF-09 |

### 9.3.2 Calidad y automatización

| Práctica | Descripción |
|---|---|
| Pruebas automatizadas | Unitarias, de integración y de extremo a extremo en cada cambio |
| CI/CD | GitHub Actions ejecuta pruebas, construye imágenes y despliega |
| Revisión de código | Pull requests con aprobación antes de unir a la rama principal |
| Análisis estático | Linters y herramientas de calidad (por ejemplo, SonarQube) |
| Infraestructura como código | Terraform describe la infraestructura de forma versionada y repetible |
| Despliegues graduales | Canary o blue-green para reducir riesgos al publicar |

### 9.3.3 Observabilidad y documentación

| Elemento | Herramienta | Requerimiento |
|---|---|---|
| Métricas | Prometheus y Grafana | RNF-12 |
| Logs centralizados | ELK | RNF-12 |
| Trazabilidad distribuida | OpenTelemetry | RNF-12 |
| Alertas | Alertmanager | RNF-12 |
| Documentación de la arquitectura | Este repositorio, versionado con Git | RNF-09 |

## 9.4 Matriz de trazabilidad: atributo de calidad y decisión

| Atributo | Decisiones principales |
|---|---|
| Seguridad | OAuth 2.0, JWT, RBAC, TLS, KMS, WAF, PCI DSS vía pasarela |
| Disponibilidad | Dos zonas, Multi-AZ, balanceador, copias de seguridad |
| Escalabilidad | Kubernetes, servicios sin estado, caché, eventos, réplicas |
| Rendimiento | Redis, OpenSearch, CDN, réplicas de lectura |
| Mantenibilidad | Microservicios, APIs documentadas, CI/CD, IaC, observabilidad |

## 9.5 Riesgos y mitigación

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Doble reserva por concurrencia | Alto | Transacciones y bloqueo de fechas en una sola base (RNF-08) |
| Caída de la pasarela de pagos | Alto | Reintentos, segunda pasarela de respaldo y estados pendientes |
| Fuga de datos personales |