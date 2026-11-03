# 6. Diagrama de despliegue

El despliegue propuesto usa **computación en la nube** (ejemplo con AWS), con
contenedores orquestados en Kubernetes, bases de datos administradas y
distribución en **varias zonas de disponibilidad** para cumplir la
disponibilidad del 99,9 % (RNF-03).

```mermaid
flowchart TB
    U(["Usuarios<br/>Navegador web / móvil"])

    subgraph EDGE["Borde de la red"]
        DNS["DNS<br/>Route 53"]
        CDN["CDN + WAF<br/>CloudFront"]
    end

    subgraph CLOUD["Nube - Región sa-east-1 (São Paulo)"]
        LB["Balanceador de carga<br/>Application Load Balancer"]

        subgraph AZ1["Zona de disponibilidad A"]
            subgraph K1["Cluster Kubernetes - nodos A"]
                GW1["API Gateway"]
                SV1["Servicios M1 a M12<br/>(contenedores Docker)"]
            end
        end

        subgraph AZ2["Zona de disponibilidad B"]
            subgraph K2["Cluster Kubernetes - nodos B"]
                GW2["API Gateway"]
                SV2["Servicios M1 a M12<br/>(contenedores Docker)"]
            end
        end

        subgraph DATA["Capa de datos administrada"]
            PGP[("PostgreSQL<br/>primaria")]
            PGR[("PostgreSQL<br/>réplica")]
            MDB[("MongoDB<br/>mensajes y reseñas")]
            RED[("Redis<br/>caché")]
            ES[("OpenSearch<br/>índice de búsqueda")]
            S3[("S3<br/>fotos y archivos")]
            MQ{{"Bus de eventos<br/>Kafka / SQS"}}
        end

        subgraph OPS["Operación y seguridad"]
            MON["Monitoreo y logs<br/>Prometheus, Grafana, ELK"]
            SEC["Gestor de secretos<br/>y claves KMS"]
            BKP["Copias de seguridad<br/>automáticas"]
        end
    end

    subgraph EXT["Servicios externos"]
        PAY["Pasarela de pagos<br/>Stripe / PayU"]
        MAPS["Mapas<br/>Google Maps API"]
        MAIL["Correo y push<br/>SES / Firebase"]
        IDP["Google y Facebook<br/>OAuth 2.0"]
    end

    subgraph CICD["Integración y entrega continua"]
        GH["GitHub<br/>repositorio"]
        GA["GitHub Actions<br/>pruebas y build"]
        REG["Registro de imágenes<br/>Docker"]
    end

    U -->|HTTPS| DNS --> CDN --> LB
    CDN -.->|estáticos| S3
    LB --> GW1 & GW2
    GW1 --> SV1
    GW2 --> SV2

    SV1 & SV2 --> PGP
    PGP -. replicación .-> PGR
    SV1 & SV2 --> MDB
    SV1 & SV2 --> RED
    SV1 & SV2 --> ES
    SV1 & SV2 --> S3
    SV1 & SV2 -.-> MQ

    SV1 & SV2 --> PAY
    SV1 & SV2 --> MAPS
    SV1 & SV2 --> MAIL
    SV1 & SV2 --> IDP

    SV1 & SV2 -.-> MON
    SV1 & SV2 -.-> SEC
    PGP -.-> BKP
    MDB -.-> BKP

    GH --> GA --> REG -->|despliegue| K1 & K2

    classDef usuario fill:#fff3cd,stroke:#b8860b,color:#000
    classDef borde fill:#d1ecf1,stroke:#0c5460,color:#000
    classDef app fill:#cfe2ff,stroke:#084298,color:#000
    classDef datos fill:#d1e7dd,stroke:#0f5132,color:#000
    classDef ops fill:#e2d9f3,stroke:#59359a,color:#000
    classDef externo fill:#f8d7da,stroke:#842029,color:#000
    classDef cicd fill:#e9ecef,stroke:#495057,color:#000

    class U usuario
    class DNS,CDN,LB borde
    class GW1,GW2,SV1,SV2 app
    class PGP,PGR,MDB,RED,ES,S3,MQ datos
    class MON,SEC,BKP ops
    class PAY,MAPS,MAIL,IDP externo
    class GH,GA,REG cicd
```

## 6.1 Descripción de los nodos

| Nodo | Tecnología | Función |
|---|---|---|
| DNS | Route 53 | Resuelve el dominio y dirige el tráfico |
| CDN + WAF | CloudFront | Entrega contenido estático rápido y filtra ataques (inyección SQL, XSS, DDoS) |
| Balanceador de carga | Application Load Balancer | Reparte las peticiones entre las zonas de disponibilidad |
| Cluster Kubernetes | EKS | Ejecuta los contenedores y escala automáticamente según la carga |
| API Gateway | Kong / NGINX | Valida tokens, enruta y limita peticiones |
| Servicios M1 a M12 | Contenedores Docker | Lógica de negocio de cada módulo |
| PostgreSQL | RDS Multi-AZ | Datos transaccionales: usuarios, alojamientos, reservas, pagos |
| MongoDB | Atlas / DocumentDB | Mensajes y reseñas (alto volumen, esquema flexible) |
| Redis | ElastiCache | Caché de búsquedas y sesiones |
| OpenSearch | Amazon OpenSearch | Búsqueda por texto, filtros y geolocalización |
| S3 | Amazon S3 | Fotos de propiedades y documentos |
| Bus de eventos | Kafka / SQS | Comunicación asíncrona entre servicios |
| Monitoreo | Prometheus, Grafana, ELK | Métricas, logs y alertas (RNF-12) |
| Secretos y claves | Secrets Manager, KMS | Protege credenciales y cifrado de datos (RNF-04) |
| Copias de seguridad | Snapshots automáticos | Recuperación ante desastres (RNF-13) |

## 6.2 Decisiones de despliegue

- **Dos zonas de disponibilidad:** si una falla, la otra sigue atendiendo (alta disponibilidad).
- **Región São Paulo:** es la más cercana a Colombia, lo que reduce la latencia.
- **Escalado horizontal:** Kubernetes agrega o quita contenedores según el tráfico (RNF-02).
- **Réplica de lectura de PostgreSQL:** descarga las consultas de lectura de la base principal.
- **CDN y WAF delante de todo:** protege y acelera antes de llegar a los servidores.
- **CI/CD con GitHub Actions:** cada cambio se prueba, se empaqueta en una imagen Docker y se despliega de forma automática.

## 6.3 Ambientes

| Ambiente | Propósito |
|---|---|
| Desarrollo | Pruebas de cada desarrollador |
| Staging | Réplica reducida de producción para pruebas finales |
| Producción | Sistema real con alta disponibilidad |