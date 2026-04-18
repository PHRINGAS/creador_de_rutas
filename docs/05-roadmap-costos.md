# 05 - Roadmap, Costos y Validacion

## Supuestos

Los costos y limites externos se toman como supuestos provistos por el proyecto. Antes
de contratar o lanzar, se deben verificar en la pagina oficial de cada proveedor.

Benchmarks de referencia provistos:

- Zeo Route Optimization: aprox. USD 24 por conductor/mes.
- Zeo Route Management: aprox. USD 42 por conductor/mes.
- Zeo Fleet Management: aprox. USD 50 por conductor/mes.

Meta inicial: operar micro-flotas por debajo de esos valores, idealmente USD 0-25/mes
total al inicio y USD 25-60/mes para flota chica/mediana con navegacion externa.

## Roadmap

### Fase 0 - Descubrimiento y prototipo

Objetivo: validar flujo operativo sin invertir en navegacion embebida.

Entregables:

- Wireframes de panel y app driver.
- Prueba tecnica de geocoding + optimization con 20-50 stops.
- Modelo de datos inicial.
- Prototipo de deep links a Google Maps/Waze/Apple Maps.
- Estimacion de requests por ruta.
- Spike de reuso de repos candidatos documentado en `docs/06-reuso-repos.md`.
- Benchmark de escenarios VRP contra 2 proveedores de optimizacion.

Criterios de salida:

- Una ruta puede importarse, geocodificarse y optimizarse.
- El equipo acepta navegacion externa como primera version.
- Se define proveedor inicial: Mapbox managed u ORS barato.
- Existe decision explicita de adopcion por repo (adoptar, adaptar o descartar).

### Fase 1 - MVP operativo

Objetivo: cubrir planificacion, despacho, tracking y POD para una flota chica.

Incluye:

- Auth y roles.
- Clientes, direcciones, drivers, vehiculos.
- Route Builder.
- Optimizacion single-vehicle.
- Asignacion y publicacion.
- App driver con ruta del dia.
- Estados de parada.
- Tracking GPS.
- POD con foto/firma/nota.
- Panel de rutas activas.
- Historial basico.

No incluye:

- Navegacion embebida.
- Multi-vehiculo avanzado.
- Notificaciones pagas por defecto.

Criterios de aceptacion:

- Crear y publicar ruta de 20 paradas en menos de 5 minutos de trabajo operativo.
- Driver completa 20 paradas con al menos 95% de eventos sincronizados automaticamente.
- POD queda visible desde panel.
- Dashboard refleja cambios en menos de 5 segundos en condiciones normales.
- Costo de proveedor se puede auditar por organizacion.

### Fase 2 - Gestion avanzada

Objetivo: acercarse al valor de Route Management.

Incluye:

- Multi-vehiculo.
- Ventanas horarias.
- Replanificacion parcial.
- Reasignacion de stops.
- Tracking publico por link.
- Notificaciones configurables.
- Dashboard KPIs.
- Revision/validacion de POD.
- Plantillas de rutas recurrentes.

Criterios de aceptacion:

- Reoptimizar solo paradas pendientes sin alterar completadas.
- Reasignar parada y notificar al driver.
- Ver fallas por motivo y exportar historial.

### Fase 3 - Escala y control de costo

Objetivo: bajar costo variable si el volumen lo justifica.

Opciones:

- Migrar routing a OSRM o Valhalla.
- Migrar geocoding a Nominatim self-host si hay capacidad DevOps.
- Mantener Supabase o migrar backend segun necesidades.
- Agregar observabilidad, backups y tuning.

Criterio economico:

- Considerar self-host cuando el gasto managed mensual supere de forma estable el costo total de servidor + mantenimiento + tiempo tecnico.

### Fase 4 - Navegacion embebida

Objetivo: controlar turn-by-turn dentro de la app si aporta ROI.

Entrar solo si:

- Drivers se equivocan frecuentemente saliendo a apps externas.
- El negocio necesita telemetria de viaje mas granular.
- El costo por trip/usuario esta presupuestado.
- Hay capacidad para probar bateria, permisos, interrupciones y rendimiento real.

## Backlog epico

| Epic | Prioridad | Fase |
| --- | --- | --- |
| Auth multi-tenant | P0 | 1 |
| Route Builder | P0 | 1 |
| Geocoding adapter | P0 | 1 |
| Optimization adapter | P0 | 1 |
| Scenario import/export JSON | P0 | 1 |
| Driver mobile route execution | P0 | 1 |
| External navigation adapter (Waze/Google/Apple) | P0 | 1 |
| POD | P0 | 1 |
| Live operations dashboard | P0 | 1 |
| Offline outbox | P0 | 1 |
| Multi-vehicle optimization | P1 | 2 |
| Customer tracking link | P1 | 2 |
| Notifications | P1 | 2 |
| KPI dashboard | P1 | 2 |
| Self-host routing | P2 | 3 |
| Embedded navigation | P3 | 4 |

## Spikes tecnicos desde repos

### Spike S1 - VRP modeling benchmark

Objetivo: verificar que nuestro dominio represente restricciones reales de ruteo.

Entrada:

- Escenarios inspirados en `googlemaps/js-route-optimization-app`:
  - shipments
  - vehicles
  - costos
  - restricciones

Salida:

- Matriz de soporte por proveedor (`mapbox`, `ors`, futuro `valhalla`).
- Riesgos funcionales y de costo por tipo de restriccion.

### Spike S2 - Route Builder import/export

Objetivo: acelerar carga y colaboracion de escenarios.

Entrada:

- Aprendizajes de `route-planner-vue` sobre export/import JSON.

Salida:

- Especificacion `route-scenario.v1.json`.
- Endpoint de import y export.
- Validaciones de compatibilidad de versiones.

### Spike S3 - Mobile navigation handoff

Objetivo: robustecer apertura de apps externas en iOS/Android.

Entrada:

- `react-native-navigation-apps` (eranabir) como referencia funcional.
- `OpenMapsApp` para compatibilidad Android 11+ (`queries` manifest).
- `FCMapsApp` para fallback de esquemas en iOS.

Salida:

- `NavigationAdapter` propio con fallback.
- Pruebas en dispositivos reales y emuladores.
- Tabla de compatibilidad por plataforma y app instalada.

## Historias iniciales

### Crear ruta desde CSV

Como dispatcher, quiero importar un CSV de paradas para crear una ruta rapidamente.

Criterios:

- El sistema muestra preview antes de guardar.
- El sistema indica filas con direccion invalida.
- El sistema no crea paradas duplicadas si `external_ref` ya existe en la ruta.

### Optimizar ruta

Como dispatcher, quiero optimizar el orden de paradas para reducir tiempo estimado.

Criterios:

- El sistema no optimiza si faltan coordenadas.
- El sistema muestra distancia y duracion estimada.
- El sistema permite conservar stops bloqueados.
- El sistema guarda proveedor y parametros usados.

### Ejecutar parada

Como driver, quiero completar una parada con evidencia para que la empresa pueda auditar la entrega.

Criterios:

- Puedo abrir navegacion externa.
- Puedo marcar llegada.
- Puedo capturar foto/firma/nota.
- Si no hay red, la evidencia queda guardada localmente.
- La parada se sincroniza cuando vuelve la red.

### Monitorear ruta

Como dispatcher, quiero ver progreso y ubicacion del driver para intervenir si hay problemas.

Criterios:

- Veo rutas activas en mapa.
- Veo ultima ubicacion con freshness.
- Veo conteo completadas/pendientes/fallidas.
- Puedo reasignar una parada pendiente.

## Escenarios de costo

### Escenario A - Micro-flota

Perfil:

- 1-3 drivers.
- 5-20 rutas por semana.
- 20-50 stops por ruta.
- Navegacion externa.

Stack:

- Supabase Free o Pro segun storage/POD.
- ORS o Mapbox dentro de cuotas gratuitas.

Costo esperado segun supuestos: USD 0-25/mes.

### Escenario B - Flota chica

Perfil:

- 5-20 drivers.
- Rutas diarias.
- Tracking durante jornada.
- POD con fotos.

Stack:

- Supabase Pro probable.
- Mapbox managed.
- Notificaciones limitadas.

Costo esperado segun supuestos: USD 25-60/mes si se evita navegacion embebida y se cachea geocoding correctamente.

### Escenario C - Escala con mucho routing

Perfil:

- Muchas rutas por dia.
- Alto volumen de optimization/directions.
- Necesidad de control de datos.

Stack:

- MapLibre.
- Valhalla/OSRM.
- Nominatim self-host.
- Supabase o backend propio.

Costo cash base potencial: bajo, pero con costo operativo real por DevOps, actualizacion de datos OSM, monitoreo y backups.

## Guardrails de costo

- No persistir geocoding permanente si el SKU no lo permite o encarece.
- Cachear direcciones internas normalizadas cuando la licencia lo permita.
- Evitar optimization en cada edicion menor; pedir confirmacion.
- Reoptimizar solo pendientes en rutas activas.
- Configurar tracking GPS por intervalo y solo en ruta activa.
- Comprimir imagenes POD.
- Feature flag para SMS/WhatsApp.
- Medir requests por organizacion desde el dia uno.

## Riesgos

| Riesgo | Impacto | Mitigacion |
| --- | --- | --- |
| Direcciones ambiguas | Rutas malas | Confidence score, revision manual, mapa |
| Limites de ORS | Bloquea multi-vehiculo | Provider adapter y fallback Mapbox |
| Costo por geocoding persistente | Sube OPEX | Politica de persistencia por proveedor |
| GPS consume bateria | Driver abandona app | Tracking solo activo, intervalos configurables |
| Offline pierde POD | Reclamos | Outbox local, storage temporal y reintentos |
| Realtime excesivo | Costos/conexiones | Suscripcion solo a rutas activas |
| WhatsApp/SMS caro | Margen bajo | Feature flag y presupuesto |
| Self-host prematuro | Retrasa producto | Managed primero, self-host solo con volumen |

## Plan de pruebas

### Unitarias

- Reglas de estado de rutas y paradas.
- Normalizacion de direcciones.
- Calculo de progreso.
- Idempotencia de eventos.
- Validacion de POD requerida.

### Integracion

- Geocoding adapter con mocks.
- Optimization adapter con fixtures.
- Supabase RLS por rol.
- Upload de POD con storage privado.
- Realtime event publication.

### E2E web

- Crear ruta desde CSV.
- Resolver direcciones ambiguas.
- Optimizar y publicar.
- Reasignar parada activa.
- Ver POD en historial.

### E2E movil

- Login driver.
- Descargar ruta del dia.
- Abrir navegacion externa.
- Completar parada con POD.
- Simular offline y sincronizar.
- Permisos de ubicacion/camara.

### Pruebas en dispositivo real

- Android gama baja.
- iPhone con permisos restrictivos.
- Red lenta/intermitente.
- App en background.
- Bateria durante tracking continuo.

## Go / No-Go MVP

Go si:

- Ruta de 20+ stops funciona end-to-end.
- POD no se pierde offline.
- RLS impide acceso cruzado entre organizaciones.
- Panel ve progreso en tiempo razonable.
- Costo por driver proyectado queda bajo benchmark.

No-Go si:

- Geocoding produce demasiadas paradas erroneas sin revision.
- La app movil pierde eventos.
- Tracking drena bateria de forma inaceptable.
- No hay auditoria de cambios manuales.
- El costo de proveedor no se puede atribuir por organizacion.
