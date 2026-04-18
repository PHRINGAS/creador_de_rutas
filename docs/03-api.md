# 03 - Especificacion de API

## Estilo

API REST versionada bajo `/v1`, con eventos realtime para cambios operativos. La
API debe ser resource-oriented, consistente, paginada e idempotente en operaciones
moviles o costosas.

## Autenticacion

- Supabase Auth emite JWT.
- Todas las llamadas privadas envian `Authorization: Bearer <token>`.
- Todas las operaciones multi-tenant requieren `X-Organization-Id`.
- Clientes moviles envian `X-App-Version`, `X-Platform` y `X-Device-Id`.
- Operaciones mutantes soportan `Idempotency-Key`.

## Formato de error

```json
{
  "error": {
    "code": "ROUTE_ALREADY_STARTED",
    "message": "La ruta ya fue iniciada.",
    "details": {
      "route_id": "uuid"
    },
    "request_id": "req_123"
  }
}
```

Codigos HTTP:

- `400`: input invalido o regla de negocio simple.
- `401`: no autenticado.
- `403`: sin permisos.
- `404`: recurso inexistente o no visible para el usuario.
- `409`: conflicto de estado o version.
- `422`: validacion semantica.
- `429`: rate limit.
- `500`: error no esperado.
- `502`: proveedor externo fallo.

## Paginacion

Parametros:

- `limit`: default 50, max 100.
- `cursor`: cursor opaco.
- `sort`: campo permitido.

Respuesta:

```json
{
  "data": [],
  "page": {
    "next_cursor": "opaque",
    "has_more": true
  }
}
```

## Endpoints

### Organizaciones y usuarios

| Metodo | Ruta | Uso |
| --- | --- | --- |
| `GET` | `/v1/me` | Perfil, organizaciones y roles |
| `GET` | `/v1/organizations/{id}` | Detalle de organizacion |
| `PATCH` | `/v1/organizations/{id}/settings` | Configuracion operativa |
| `GET` | `/v1/memberships` | Miembros de la organizacion |
| `POST` | `/v1/memberships/invite` | Invitar usuario |
| `PATCH` | `/v1/memberships/{id}` | Cambiar rol/estado |

### Drivers y vehiculos

| Metodo | Ruta | Uso |
| --- | --- | --- |
| `GET` | `/v1/drivers` | Listar conductores |
| `POST` | `/v1/drivers` | Crear conductor |
| `GET` | `/v1/drivers/{id}` | Detalle |
| `PATCH` | `/v1/drivers/{id}` | Editar |
| `GET` | `/v1/vehicles` | Listar vehiculos |
| `POST` | `/v1/vehicles` | Crear vehiculo |
| `PATCH` | `/v1/vehicles/{id}` | Editar vehiculo |

### Clientes y direcciones

| Metodo | Ruta | Uso |
| --- | --- | --- |
| `GET` | `/v1/customers` | Listar clientes |
| `POST` | `/v1/customers` | Crear cliente |
| `PATCH` | `/v1/customers/{id}` | Editar cliente |
| `GET` | `/v1/addresses` | Buscar direcciones internas |
| `POST` | `/v1/geocode/search` | Autocomplete/geocoding |
| `POST` | `/v1/geocode/reverse` | Reverse geocoding |
| `POST` | `/v1/geocode/batch` | Geocoding por lote |

Ejemplo geocoding:

```json
{
  "query": "Av Corrientes 1234, CABA",
  "country": "AR",
  "limit": 5,
  "persist": false
}
```

Respuesta:

```json
{
  "data": [
    {
      "label": "Av. Corrientes 1234, Buenos Aires, Argentina",
      "lat": -34.603722,
      "lng": -58.381592,
      "confidence": 0.92,
      "provider": "mapbox",
      "provider_place_id": "place.123"
    }
  ]
}
```

Nota: `persist=true` solo puede usarse si el proveedor/licencia permite guardar el
resultado.

### Rutas

| Metodo | Ruta | Uso |
| --- | --- | --- |
| `GET` | `/v1/routes` | Listar rutas por fecha/estado/driver |
| `POST` | `/v1/routes` | Crear ruta draft |
| `GET` | `/v1/routes/{id}` | Detalle de ruta con stops |
| `PATCH` | `/v1/routes/{id}` | Editar metadata |
| `DELETE` | `/v1/routes/{id}` | Cancelar o borrar draft |
| `POST` | `/v1/routes/{id}/stops` | Agregar parada |
| `PATCH` | `/v1/routes/{id}/stops/reorder` | Reorden manual |
| `POST` | `/v1/routes/{id}/optimize` | Optimizar |
| `POST` | `/v1/routes/{id}/publish` | Publicar |
| `POST` | `/v1/routes/{id}/assign` | Asignar driver/vehiculo |
| `POST` | `/v1/routes/{id}/replan` | Reoptimizar pendientes |
| `POST` | `/v1/routes/{id}/complete` | Cierre manual controlado |

Crear ruta:

```json
{
  "name": "Reparto zona norte",
  "service_date": "2026-04-20",
  "depot_address_id": "uuid",
  "stops": [
    {
      "customer_id": "uuid",
      "address_id": "uuid",
      "service_minutes": 8,
      "instructions": "Tocar timbre B"
    }
  ]
}
```

Optimizar:

```json
{
  "provider": "mapbox",
  "mode": "single_vehicle",
  "respect_locked_sequence": true,
  "return_geometry": true
}
```

Respuesta:

```json
{
  "data": {
    "route_id": "uuid",
    "status": "optimized",
    "provider": "mapbox",
    "estimated_distance_meters": 42300,
    "estimated_duration_seconds": 9800,
    "stops": [
      {
        "id": "uuid",
        "sequence": 1,
        "eta_offset_seconds": 0
      }
    ]
  }
}
```

### Operacion en vivo

| Metodo | Ruta | Uso |
| --- | --- | --- |
| `GET` | `/v1/operations/active-routes` | Rutas activas para dashboard |
| `GET` | `/v1/operations/drivers/last-locations` | Ultima posicion de drivers |
| `POST` | `/v1/operations/stops/{id}/reassign` | Reasignar parada |
| `POST` | `/v1/operations/routes/{id}/pause` | Pausar |
| `POST` | `/v1/operations/routes/{id}/resume` | Reanudar |

### API movil driver

| Metodo | Ruta | Uso |
| --- | --- | --- |
| `GET` | `/v1/driver/sync` | Snapshot compacto para app movil |
| `GET` | `/v1/driver/routes/today` | Rutas del dia |
| `POST` | `/v1/driver/routes/{id}/start` | Iniciar ruta |
| `POST` | `/v1/driver/routes/{id}/pause` | Pausar ruta |
| `POST` | `/v1/driver/routes/{id}/finish` | Finalizar ruta |
| `POST` | `/v1/driver/stops/{id}/status` | Cambiar estado |
| `POST` | `/v1/driver/locations` | Enviar ubicacion |
| `POST` | `/v1/driver/offline-events/batch` | Sincronizar cola local |

Enviar ubicacion:

```json
{
  "client_event_id": "uuid",
  "route_id": "uuid",
  "lat": -34.603722,
  "lng": -58.381592,
  "accuracy_meters": 12,
  "heading": 180,
  "speed_mps": 4.1,
  "recorded_at": "2026-04-20T14:35:00Z"
}
```

Cambiar estado de parada:

```json
{
  "client_event_id": "uuid",
  "status": "arrived",
  "occurred_at": "2026-04-20T14:40:00Z",
  "lat": -34.603722,
  "lng": -58.381592
}
```

### Proof of Delivery

| Metodo | Ruta | Uso |
| --- | --- | --- |
| `POST` | `/v1/pod/upload-url` | Crear URL firmada para foto/firma |
| `POST` | `/v1/driver/stops/{id}/pod` | Registrar POD |
| `GET` | `/v1/stops/{id}/pod` | Ver POD |
| `POST` | `/v1/pod/{id}/review` | Aceptar/rechazar POD |

Registrar POD:

```json
{
  "client_event_id": "uuid",
  "recipient_name": "Juan Perez",
  "note": "Entregado en recepcion",
  "photo_paths": [
    "org/route/stop/photo.jpg"
  ],
  "signature_path": "org/route/stop/signature.png",
  "lat": -34.603722,
  "lng": -58.381592,
  "captured_at": "2026-04-20T14:45:00Z"
}
```

### Tracking publico

| Metodo | Ruta | Uso |
| --- | --- | --- |
| `GET` | `/v1/public/tracking/{token}` | Estado publico limitado |

Respuesta publica:

```json
{
  "data": {
    "status": "en_route",
    "eta_window": {
      "from": "2026-04-20T15:00:00Z",
      "to": "2026-04-20T15:30:00Z"
    },
    "stop_sequence": 8,
    "remaining_stops_before_you": 2
  }
}
```

No exponer telefono del driver, lista completa de paradas ni datos de otros clientes.

## Eventos realtime

Evento de ruta:

```json
{
  "type": "stop.status_changed",
  "organization_id": "uuid",
  "route_id": "uuid",
  "stop_id": "uuid",
  "status": "completed",
  "occurred_at": "2026-04-20T14:45:00Z"
}
```

Evento de ubicacion:

```json
{
  "type": "driver.location_updated",
  "driver_id": "uuid",
  "route_id": "uuid",
  "lat": -34.603722,
  "lng": -58.381592,
  "recorded_at": "2026-04-20T14:45:00Z"
}
```

## Rate limits iniciales

| Operacion | Limite sugerido |
| --- | --- |
| Geocode search | 60/min por usuario |
| Batch geocode | 5/min por organizacion |
| Optimize route | 30/h por organizacion en MVP |
| Location ping | 1 cada 15s por driver en ruta activa |
| POD upload URL | 20/min por driver |

## Idempotencia

Obligatoria para:

- `POST /v1/routes/{id}/optimize`
- `POST /v1/driver/locations`
- `POST /v1/driver/stops/{id}/status`
- `POST /v1/driver/stops/{id}/pod`
- `POST /v1/driver/offline-events/batch`

Si llega la misma key, devolver el resultado original sin duplicar eventos.

## Versionado movil

El backend debe poder responder:

```json
{
  "error": {
    "code": "APP_VERSION_UNSUPPORTED",
    "message": "Actualiza la app para continuar.",
    "details": {
      "minimum_version": "1.3.0"
    }
  }
}
```

Esto evita romper clientes ya instalados cuando cambian contratos criticos.
