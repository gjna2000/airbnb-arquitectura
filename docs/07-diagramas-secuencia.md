# 7. Diagramas de secuencia

Se presentan tres casos de uso representativos:

1. Realizar una reserva con pago seguro
2. Buscar alojamientos por destino y fechas
3. Publicar un alojamiento

## 7.1 Caso de uso 1: Reserva con pago seguro

**Actor:** Huésped
**Requerimientos relacionados:** RF-09, RF-11, RF-12, RNF-06, RNF-08
**Precondición:** el huésped ha iniciado sesión y eligió un alojamiento con fechas.

```mermaid
sequenceDiagram
    autonumber
    actor H as Huésped
    participant W as Aplicación web
    participant GW as API Gateway
    participant R as Servicio Reservas
    participant D as Servicio Disponibilidad
    participant P as Servicio Precios
    participant PG as Servicio Pagos
    participant PS as Pasarela de pagos
    participant Q as Bus de eventos
    participant N as Servicio Notificaciones
    actor A as Anfitrión

    H->>W: Selecciona fechas y pulsa "Reservar"
    W->>GW: POST /reservas (token JWT)
    GW->>GW: Valida token y límite de peticiones
    GW->>R: Crea solicitud de reserva
    R->>P: Calcula precio total (noches, tarifas, impuestos)
    P-->>R: Precio total
    R->>D: Bloquea fechas temporalmente (transacción)

    alt Fechas disponibles
        D-->>R: Fechas bloqueadas
        R->>PG: Solicita cobro por el monto total
        PG->>PS: Crea transacción (token de pago, sin datos de tarjeta)
        alt Pago aprobado
            PS-->>PG: Pago aprobado
            PG-->>R: Pago confirmado
            R->>D: Confirma fechas como reservadas
            R->>R: Estado de la reserva = CONFIRMADA
            R-)Q: Publica evento "ReservaConfirmada"
            Q-)N: Entrega evento
            N-)H: Correo y push de confirmación
            N-)A: Aviso de nueva reserva
            R-->>GW: Reserva confirmada
            GW-->>W: 201 Created
            W-->>H: Muestra comprobante de reserva
        else Pago rechazado
            PS-->>PG: Pago rechazado
            PG-->>R: Pago fallido
            R->>D: Libera las fechas bloqueadas
            R-->>GW: Error de pago
            GW-->>W: 402 Payment Required
            W-->>H: Informa que el pago fue rechazado
        end
    else Fechas no disponibles
        D-->>R: Conflicto de fechas
        R-->>GW: Error de disponibilidad
        GW-->>W: 409 Conflict
        W-->>H: Informa que las fechas ya no están disponibles
    end
```

**Puntos clave:**
- Las fechas se **bloquean antes de cobrar**, así dos huéspedes no pueden reservar lo mismo (RNF-08).
- La aplicación nunca maneja datos de tarjeta: solo un **token** que entrega la pasarela (RNF-06).
- Notificaciones se envía por **evento asíncrono**, para no demorar la respuesta al huésped.
- Si el pago falla, las fechas se **liberan** de inmediato.

## 7.2 Caso de uso 2: Búsqueda de alojamientos

**Actor:** Huésped
**Requerimientos relacionados:** RF-07, RF-08, RNF-01

```mermaid
sequenceDiagram
    autonumber
    actor H as Huésped
    participant W as Aplicación web
    participant GW as API Gateway
    participant B as Servicio Búsqueda
    participant C as Caché (Redis)
    participant I as Índice (OpenSearch)
    participant D as Servicio Disponibilidad
    participant P as Servicio Precios

    H->>W: Ingresa destino, fechas, huéspedes y filtros
    W->>GW: GET /busqueda?destino=...&fechas=...
    GW->>B: Reenvía la consulta
    B->>C: Consulta resultados en caché

    alt Resultado en caché
        C-->>B: Resultados guardados
    else Sin caché
        C-->>B: No encontrado
        B->>I: Búsqueda por texto, filtros y geolocalización
        I-->>B: Alojamientos candidatos
        B->>D: Verifica disponibilidad en las fechas
        D-->>B: Alojamientos disponibles
        B->>P: Obtiene precio por noche y total
        P-->>B: Precios
        B->>C: Guarda resultado (tiempo de vida corto)
    end

    B-->>GW: Lista de resultados
    GW-->>W: 200 OK
    W-->>H: Muestra resultados en lista y en mapa
```

**Puntos clave:**
- La **caché** evita repetir consultas costosas y ayuda a cumplir el tiempo menor a 2 segundos (RNF-01).
- La búsqueda usa un **índice especializado**, no la base de datos transaccional.
- La disponibilidad se verifica **después** de filtrar, para consultar solo los candidatos.

## 7.3 Caso de uso 3: Publicar un alojamiento

**Actor:** Anfitrión
**Requerimientos relacionados:** RF-03, RF-05, RF-06

```mermaid
sequenceDiagram
    autonumber
    actor A as Anfitrión
    participant W as Aplicación web
    participant GW as API Gateway
    participant AL as Servicio Alojamientos
    participant S3 as Almacenamiento (S3)
    participant MP as Servicio de mapas
    participant Q as Bus de eventos
    participant B as Servicio Búsqueda
    participant AD as Administración

    A->>W: Completa datos, fotos, tarifas y reglas
    W->>GW: POST /alojamientos (token JWT, rol anfitrión)
    GW->>GW: Valida token y rol
    GW->>AL: Crea publicación
    AL->>MP: Convierte dirección en coordenadas
    MP-->>AL: Latitud y longitud
    AL->>S3: Guarda las fotos
    S3-->>AL: URLs de las imágenes
    AL->>AL: Guarda alojamiento (estado = EN REVISIÓN)
    AL-)Q: Publica evento "AlojamientoCreado"
    AL-->>GW: Publicación creada
    GW-->>W: 201 Created
    W-->>A: Muestra "Publicación en revisión"

    Q-)AD: Entrega evento para moderación
    AD->>AL: Aprueba la publicación
    AL->>AL: Estado = PUBLICADO
    AL-)Q: Publica evento "AlojamientoPublicado"
    Q-)B: Entrega evento
    B->>B: Indexa el alojamiento para búsquedas
```

**Puntos clave:**
- Toda publicación pasa por **moderación** antes de ser visible (RF-19).
- La indexación en Búsqueda es **asíncrona**: no hace esperar al anfitrión.
- Las fotos van a almacenamiento de objetos, no a la base de datos.