# 6. Diagrama de despliegue

El despliegue propuesto usa **computación en la nube** (ejemplo con AWS), con
contenedores en Kubernetes, bases de datos administradas y dos zonas de
disponibilidad para cumplir la disponibilidad del 99,9 % (RNF-03).

Se presenta en tres vistas: infraestructura principal, capa de datos y
entrega continua.

## 6.1 Vista de infraestructura

```mermaid
flowchart TB
    U(["Usuarios<br/>navegador web / móvil"])
    DNS["DNS<br/>Route 53"]
    CDN["CDN + WAF<br/>CloudFront"]
    LB["Balanceador de carga<br/>Application Load Balancer"]

    subgraph K["Cluster Kubernetes (EKS) - región sa-east-1"]
        direction LR
        subgraph AZ1["Zona A"]
            P1["API Gateway<br/>+ Servicios M1 a M12"]
        end
        subgraph AZ2["Zona B"]
            P2["API Gateway<br/>+ Servicios M1 a M12"]
        end
    end

    DATA[("Capa de datos<br/>ver 6.2")]
    EXT[/"Servicios externos<br/>pagos, mapas, correo, OAuth"/]

    U -->|HTTPS| DNS --> CDN --> LB
    LB --> P1
    LB --> P2
    P1 --> DATA
    P2 --> DATA
    P1 --> EXT
    P2 --> EXT

    classDef usuario fill:#fff3cd,stroke:#b8860b,color:#000
    classDef borde fill:#d1ecf1,stroke:#0c5460,color:#000
    classDef app fill:#cfe2ff,stroke:#084298,color:#000
    classDef datos fill:#d1e7dd,stroke:#0f5132,color:#000
    classDef externo fill:#f8d7da,stroke:#842029,color:#000

    class U usuario
    class DNS,CDN,LB borde
    class P1,P2 app
    class DATA datos
    class EXT externo
```

## 6.2 Vista de la capa de datos

```mermaid
flowchart LR
    SV["Servicios M1 a M12"]

    subgraph REL["Datos transaccionales"]
        PGP[("PostgreSQL<br/>primaria")]
        PGR[("PostgreSQL<br/>réplica de lectura")]
    end

    subgraph NOSQL["Datos flexibles y rápidos"]
        MDB[("MongoDB<br/>mensajes y reseñas")]
        RED[("Redis<br/>caché y sesiones")]
        ES[("OpenSearch<br/>búsqueda")]
    end

    subgraph FILES["Archivos y eventos"]
        S3[("S3<br/>fotos y documentos")]
        MQ{{"Kafka / SQS<br/>bus de eventos"}}
    end

    BKP["Copias de seguridad<br/>automáticas"]

    SV --> PGP
    PGP -. replicación .-> PGR
    SV --> MDB
    SV --> RED
    SV --> ES
    SV --> S3
    SV -.-> MQ
    PGP -.-> BKP
    MDB -.-> BKP

    classDef app fill:#cfe2ff,stroke:#084298,color:#000
    classDef datos fill:#d1e7dd,stroke:#0f5132,color:#000
    classDef ops fill:#e2d9f3,stroke:#59359a,color:#000

    class SV app
    class PGP,PGR,MDB,RED,ES,S3,MQ datos
    class BKP ops
```

## 6.3 Vista de entrega continua (CI/CD)

```mermaid
flowchart LR
    DEV(["Desarrollador"]) -->|push| GH["GitHub<br/>repositorio"]
    GH --> GA["GitHub Actions<br/>pruebas y build"]
    GA --> REG["Registro de<br/>imágenes Docker"]
    REG -->|despliegue| STG["Staging"]
    STG -->|aprobación| PRD["Producción<br/>Kubernetes"]

    classDef usuario fill:#fff3cd,stroke:#b8860b,color:#000
    classDef cicd fill:#e9ecef,stroke:#495057,color:#000
    classDef app fill:#cfe2ff,stroke:#084298,color:#000

    class DEV usuario
    class GH,GA,REG cicd
    class STG,PRD app
```

## 6.4 Descripción de los nodos

| Nodo | Tecnología | Función |
|---|---|---|
| DNS | Route 53 | Resuelve el dominio y dirige el tráfico |
| CDN + WAF | CloudFront | Entrega contenido estático y filtra ataques (inyección SQL, XSS, DDoS) |
| Balanceador de carga | Application Load Balancer | Reparte las peticiones entre las zonas |
| Cluster Kubernetes | EKS | Ejecuta contenedores y escala según la carga |
| API Gateway | Kong / NGINX | Valida tokens, enruta y limita peticiones |
| Servicios M1 a M12 | Contenedores Docker | Lógica de negocio de cada módulo |
| PostgreSQL | RDS Multi-AZ | Usuarios, alojamientos, reservas y pagos |
| MongoDB | Atlas / DocumentDB | Mensajes y reseñas (alto volumen, esquema flexible) |
| Redis | ElastiCache | Caché de búsquedas y sesiones |
| OpenSearch | Amazon OpenSearch | Búsqueda por texto, filtros y geolocalización |
| S3 | Amazon S3 | Fotos de propiedades y documentos |
| Bus de eventos | Kafka / SQS | Comunicación asíncrona entre servicios |
| Monitoreo y logs | Prometheus, Grafana, ELK | Métricas y alertas (RNF-12) |
| Secretos y claves | Secrets Manager, KMS | Protege credenciales y cifrado (RNF-04) |
| Copias de seguridad | Snapshots automáticos | Recuperación ante desastres (RNF-13) |

## 6.5 Decisiones de despliegue

- **Dos zonas de disponibilidad:** si una falla, la otra sigue atendiendo.
- **Región São Paulo:** la más cercana a Colombia, menor latencia.
- **Escalado horizontal:** Kubernetes agrega o quita contenedores según el tráfico (RNF-02).
- **Réplica de lectura:** descarga las consultas de la base principal.
- **CDN y WAF delante de todo:** protegen y aceleran antes de llegar a los servidores.
- **CI/CD con GitHub Actions:** cada cambio se prueba, se empaqueta y se despliega de forma automática.

## 6.6 Ambientes

| Ambiente | Propósito |
|---|---|
| Desarrollo | Pruebas de cada desarrollador |
| Staging | Réplica reducida de producción para pruebas finales |
| Producción | Sistema real con alta disponibilidad |