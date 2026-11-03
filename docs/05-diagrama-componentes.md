# 5. Diagrama de componentes

La arquitectura propuesta es **modular por servicios**: un cliente web, un API
Gateway y servicios independientes, cada uno con su propia base de datos.

```mermaid
flowchart TB
    subgraph Clientes
        WEB[Aplicación web<br/>responsive]
    end

    GW[API Gateway<br/>autenticación, rutas, límite de peticiones]

    subgraph Servicios["Servicios de negocio"]
        USR[M1 Usuarios<br/>y autenticación]
        ALO[M2 Alojamientos]
        DIS[M3 Disponibilidad]
        PRE[M4 Precios dinámicos]
        BUS[M5 Búsqueda]
        RES[M6 Reservas]
        PAG[M7 Pagos]
        REV[M8 Reseñas]
        MSG[M9 Mensajería]
        NOT[M10 Notificaciones]
        EXP[M11 Experiencias]
        ADM[M12 Administración]
    end

    subgraph Datos["Almacenamiento"]
        DBU[(BD Usuarios)]
        DBA[(BD Alojamientos)]
        DBR[(BD Reservas<br/>y Disponibilidad)]
        DBP[(BD Pagos)]
        DBM[(BD Mensajes<br/>y Reseñas)]
        IDX[(Índice de búsqueda)]
        OBJ[(Almacenamiento<br/>de fotos)]
        CAC[(Caché)]
    end

    BUS_MQ{{Bus de eventos<br/>cola de mensajes}}

    subgraph Externos["Servicios externos"]
        PGW[/Pasarela de pagos/]
        MAP[/Servicio de mapas/]
        MAIL[/Correo y push/]
        IDP[/Google y Facebook/]
    end

    WEB --> GW
    GW --> USR & ALO & BUS & RES & REV & MSG & EXP & ADM

    RES --> DIS
    RES --> PRE
    RES --> PAG
    BUS --> DIS
    BUS --> PRE
    ALO --> DIS

    USR --> DBU
    ALO --> DBA
    ALO --> OBJ
    DIS --> DBR
    RES --> DBR
    PAG --> DBP
    REV --> DBM
    MSG --> DBM
    BUS --> IDX
    BUS --> CAC

    RES -. eventos .-> BUS_MQ
    PAG -. eventos .-> BUS_MQ
    ALO -. eventos .-> BUS_MQ
    MSG -. eventos .-> BUS_MQ
    BUS_MQ -.-> NOT
    BUS_MQ -.-> BUS

    PAG --> PGW
    BUS --> MAP
    NOT --> MAIL
    USR --> IDP
```

## 5.1 Descripción de los componentes

| Componente | Función |
|---|---|
| Aplicación web | Interfaz para huéspedes, anfitriones y administradores |
| API Gateway | Punto único de entrada: valida tokens, enruta y limita peticiones |
| Servicios M1 a M12 | Lógica de negocio de cada módulo, desplegables por separado |
| Bus de eventos | Comunicación asíncrona entre servicios (reserva creada, pago confirmado) |
| Índice de búsqueda | Copia optimizada de alojamientos para consultas rápidas |
| Caché | Guarda resultados frecuentes para bajar la latencia |
| Almacenamiento de fotos | Guarda imágenes y archivos fuera de la base de datos |

## 5.2 Decisiones clave

- **Una base de datos por servicio:** cada módulo es dueño de sus datos, así se reduce el acoplamiento.
- **Comunicación asíncrona por eventos:** Notificaciones y Búsqueda se actualizan sin frenar al servicio que origina el evento.
- **Reservas y Disponibilidad comparten base de datos:** es lo que permite garantizar, con transacciones, que no haya dobles reservas (RNF-08).
- **API Gateway:** centraliza la seguridad y simplifica el cliente.