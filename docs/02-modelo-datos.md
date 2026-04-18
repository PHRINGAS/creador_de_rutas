# 02 - Modelo de Datos

## Base recomendada

PostgreSQL en Supabase con UUIDs, `TIMESTAMPTZ`, soft delete selectivo y Row Level
Security por `organization_id`. Para coordenadas se recomienda PostGIS si esta
disponible; si no, usar columnas `lat` y `lng` con indices B-tree y bounding boxes
basicas.

## Convenciones

- Todas las tablas multi-tenant incluyen `organization_id`.
- Todas las tablas principales incluyen `created_at`, `updated_at`.
- Las tablas auditables no se borran fisicamente salvo politica de retencion.
- IDs: UUID generados por base o aplicacion.
- Estados: enums PostgreSQL o checks controlados por migraciones.
- Campos `metadata` JSONB solo para atributos variables, no para relaciones centrales.

## Entidades principales

### `organizations`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk | Tenant |
| `name` | text | Nombre comercial |
| `plan` | text | free, pro, enterprise, internal |
| `settings` | jsonb | Tracking interval, POD rules, provider config |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

### `profiles`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk | Igual a auth user id |
| `full_name` | text |  |
| `phone` | text | Opcional |
| `avatar_url` | text | Opcional |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

### `memberships`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `profile_id` | uuid fk |  |
| `role` | text | admin, dispatcher, driver, viewer |
| `status` | text | invited, active, disabled |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

Restriccion: unique `organization_id, profile_id`.

### `drivers`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `profile_id` | uuid fk nullable | Si tiene login |
| `display_name` | text |  |
| `phone` | text |  |
| `status` | text | active, inactive |
| `default_vehicle_id` | uuid nullable |  |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

### `vehicles`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `label` | text | Ej: Moto 01 |
| `type` | text | car, van, bike, motorcycle, truck |
| `capacity_units` | numeric nullable | Fase 2 |
| `status` | text | active, maintenance, inactive |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

### `customers`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `name` | text |  |
| `phone` | text nullable |  |
| `email` | text nullable |  |
| `external_ref` | text nullable | ID ERP/e-commerce |
| `notes` | text nullable |  |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

### `addresses`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `customer_id` | uuid fk nullable |  |
| `label` | text | Casa, deposito, local |
| `raw_address` | text | Entrada original |
| `normalized_address` | text | Direccion normalizada |
| `lat` | numeric(10,7) |  |
| `lng` | numeric(10,7) |  |
| `geohash` | text nullable | Para busqueda espacial simple |
| `provider` | text | mapbox, ors, nominatim, manual |
| `provider_place_id` | text nullable | Respetar licencia |
| `confidence` | numeric nullable |  |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

### `routes`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `name` | text |  |
| `service_date` | date | Dia operativo |
| `status` | text | draft, optimized, published, in_progress, paused, completed, cancelled |
| `depot_address_id` | uuid nullable | Punto inicial |
| `end_address_id` | uuid nullable | Punto final |
| `assigned_driver_id` | uuid nullable | Driver principal |
| `assigned_vehicle_id` | uuid nullable | Vehiculo principal |
| `optimization_provider` | text nullable |  |
| `estimated_distance_meters` | integer nullable |  |
| `estimated_duration_seconds` | integer nullable |  |
| `started_at` | timestamptz nullable |  |
| `completed_at` | timestamptz nullable |  |
| `version` | integer | Para concurrencia optimista |
| `created_by` | uuid | profile id |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

### `route_stops`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk | Denormalizado para RLS y queries |
| `route_id` | uuid fk |  |
| `customer_id` | uuid fk nullable |  |
| `address_id` | uuid fk |  |
| `sequence` | integer | Orden actual |
| `locked_sequence` | boolean | No mover al optimizar |
| `status` | text | pending, next, en_route, arrived, completed, failed, skipped, reassigned |
| `service_minutes` | integer nullable | Tiempo esperado |
| `time_window_start` | timestamptz nullable |  |
| `time_window_end` | timestamptz nullable |  |
| `priority` | integer | 0 normal, mayor mas prioritario |
| `instructions` | text nullable |  |
| `failure_reason` | text nullable |  |
| `arrived_at` | timestamptz nullable |  |
| `completed_at` | timestamptz nullable |  |
| `created_at` | timestamptz |  |
| `updated_at` | timestamptz |  |

Restriccion: unique `route_id, sequence`.

### `route_events`

Tabla append-only.

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `route_id` | uuid fk |  |
| `route_stop_id` | uuid nullable |  |
| `driver_id` | uuid nullable |  |
| `event_type` | text | route_started, stop_completed, pod_uploaded, etc. |
| `payload` | jsonb | Snapshot compacto |
| `client_event_id` | uuid nullable | Idempotencia movil |
| `occurred_at` | timestamptz | Tiempo de origen |
| `received_at` | timestamptz | Tiempo servidor |
| `created_by` | uuid nullable | profile id |

Restriccion recomendada: unique nullable parcial por `organization_id, client_event_id`.

### `driver_locations`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `driver_id` | uuid fk |  |
| `route_id` | uuid nullable |  |
| `lat` | numeric(10,7) |  |
| `lng` | numeric(10,7) |  |
| `accuracy_meters` | numeric nullable |  |
| `heading` | numeric nullable |  |
| `speed_mps` | numeric nullable |  |
| `battery_level` | numeric nullable | Opcional |
| `recorded_at` | timestamptz | En dispositivo |
| `received_at` | timestamptz | En servidor |

Retencion sugerida: crudo 30-90 dias; ultima ubicacion indefinida o agregada.

### `proofs_of_delivery`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `route_id` | uuid fk |  |
| `route_stop_id` | uuid fk |  |
| `driver_id` | uuid fk |  |
| `status` | text | submitted, accepted, rejected |
| `note` | text nullable |  |
| `recipient_name` | text nullable |  |
| `signature_path` | text nullable | Storage path |
| `photo_paths` | text[] | Storage paths |
| `lat` | numeric(10,7) nullable |  |
| `lng` | numeric(10,7) nullable |  |
| `captured_at` | timestamptz |  |
| `created_at` | timestamptz |  |

### `customer_tracking_links`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `route_stop_id` | uuid fk |  |
| `token_hash` | text | No guardar token plano |
| `expires_at` | timestamptz |  |
| `last_viewed_at` | timestamptz nullable |  |
| `created_at` | timestamptz |  |

### `provider_requests`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `provider` | text | mapbox, ors, nominatim |
| `operation` | text | geocode, directions, optimize |
| `request_hash` | text | Deduplicacion |
| `status` | text | success, failed |
| `cost_units` | numeric nullable | Para estimar costo |
| `latency_ms` | integer nullable |  |
| `error_code` | text nullable |  |
| `created_at` | timestamptz |  |

### `audit_log`

| Campo | Tipo | Notas |
| --- | --- | --- |
| `id` | uuid pk |  |
| `organization_id` | uuid fk |  |
| `actor_profile_id` | uuid nullable |  |
| `action` | text |  |
| `entity_type` | text | route, stop, driver, etc. |
| `entity_id` | uuid |  |
| `before` | jsonb nullable |  |
| `after` | jsonb nullable |  |
| `created_at` | timestamptz |  |

## Indices recomendados

```sql
create index idx_memberships_profile_org on memberships (profile_id, organization_id);
create index idx_drivers_org_status on drivers (organization_id, status);
create index idx_vehicles_org_status on vehicles (organization_id, status);
create index idx_customers_org_external_ref on customers (organization_id, external_ref);
create index idx_addresses_org_customer on addresses (organization_id, customer_id);
create index idx_addresses_org_geohash on addresses (organization_id, geohash);

create index idx_routes_org_date_status on routes (organization_id, service_date, status);
create index idx_routes_org_driver_date on routes (organization_id, assigned_driver_id, service_date);
create index idx_route_stops_route_sequence on route_stops (route_id, sequence);
create index idx_route_stops_org_status on route_stops (organization_id, status);
create index idx_route_stops_org_customer on route_stops (organization_id, customer_id);

create index idx_route_events_route_time on route_events (route_id, occurred_at desc);
create index idx_route_events_org_type_time on route_events (organization_id, event_type, occurred_at desc);
create unique index idx_route_events_client_event
  on route_events (organization_id, client_event_id)
  where client_event_id is not null;

create index idx_driver_locations_driver_time on driver_locations (driver_id, recorded_at desc);
create index idx_driver_locations_route_time on driver_locations (route_id, recorded_at desc);
create index idx_pod_stop on proofs_of_delivery (route_stop_id);
create index idx_provider_requests_org_time on provider_requests (organization_id, created_at desc);
```

Si se usa PostGIS:

```sql
alter table addresses add column location geography(Point, 4326);
alter table driver_locations add column location geography(Point, 4326);
create index idx_addresses_location on addresses using gist (location);
create index idx_driver_locations_location on driver_locations using gist (location);
```

## Politicas RLS

Reglas base:

- Un usuario autenticado solo ve filas de organizaciones donde tenga membership activa.
- `admin` puede administrar todo dentro de su organizacion.
- `dispatcher` puede gestionar rutas, clientes, direcciones, drivers y vehiculos.
- `driver` solo puede leer su perfil, sus rutas asignadas, sus stops y crear eventos/locations/POD propios.
- `viewer` solo puede leer.

Ejemplo conceptual:

```sql
create policy "members can read org routes"
on routes for select
using (
  exists (
    select 1
    from memberships m
    where m.organization_id = routes.organization_id
      and m.profile_id = auth.uid()
      and m.status = 'active'
  )
);
```

Para drivers, agregar condicion de asignacion:

```sql
exists (
  select 1
  from drivers d
  where d.id = routes.assigned_driver_id
    and d.profile_id = auth.uid()
)
```

## Storage

Buckets:

- `pod-photos`: privado, imagenes de entrega.
- `pod-signatures`: privado, firmas.
- `imports`: privado, CSV de paradas importadas.

Reglas:

- Upload con path `organization_id/route_id/stop_id/file_id`.
- URLs firmadas para lectura temporal.
- Validar MIME type y tamano.
- Generar thumbnails si el volumen de fotos crece.

## Migraciones futuras

- Multi-depot.
- Rutas recurrentes.
- Turnos y disponibilidad de drivers.
- Capacidades por vehiculo y demanda por parada.
- Zonas/territorios con poligonos.
- Facturacion SaaS si se comercializa a terceros.
