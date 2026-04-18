# 00 - Especificacion de Producto

## Resumen

El producto sera una app de planificacion, despacho, seguimiento y prueba de entrega
para equipos de reparto, field service, preventa, cobranzas y visitas tecnicas. El
objetivo no es replicar todo Zeo Route Planner desde el primer release, sino cubrir
el valor operativo central con un costo mensual muy inferior al benchmark indicado:
aprox. USD 24, USD 42 o USD 50 por conductor/mes segun plan de Zeo.

La estrategia correcta para costo bajo es construir primero una plataforma que:

- Cree y optimice rutas multi-stop.
- Despache rutas a conductores.
- Muestre mapa, paradas, progreso y posicion actual.
- Registre estados por parada.
- Guarde prueba de entrega con foto, firma, nota y timestamp.
- Permita replanificar y reasignar desde un panel operativo.
- Abra navegacion externa en Google Maps, Waze o Apple Maps.

La navegacion embebida queda fuera del MVP salvo que el negocio demuestre que reduce
errores, mejora tiempos o justifica el costo variable adicional.

## Usuarios

| Usuario | Necesidad principal | Producto esperado |
| --- | --- | --- |
| Operador / dispatcher | Crear rutas, asignarlas y monitorear ejecucion | Panel web con mapa, estado y herramientas de replanificacion |
| Conductor / driver | Ejecutar una ruta sin friccion desde el movil | App simple, offline-tolerante, con stops, navegacion externa y POD |
| Supervisor / owner | Medir cumplimiento, costos y productividad | Historial, KPIs, evidencia y auditoria |
| Cliente final | Saber estado aproximado de visita/entrega | Link de tracking o notificacion opcional |

## Jobs to be Done

- Cuando tengo una lista de direcciones, quiero convertirlas en una ruta ordenada para reducir tiempo y kilometros.
- Cuando tengo varios conductores, quiero repartir paradas entre vehiculos para balancear carga y ventanas horarias.
- Cuando una ruta ya salio, quiero saber donde esta el conductor y que paradas fueron completadas.
- Cuando una entrega se completa, quiero tener evidencia verificable para resolver reclamos.
- Cuando una parada falla, quiero replanificar o reasignar sin rehacer todo manualmente.

## Principios de producto

- Primero operacion, despues navegacion embebida.
- Costo por conductor debe ser visible como metrica de negocio.
- Todo evento importante debe quedar auditado.
- El driver no debe depender de una conexion perfecta.
- Las integraciones de mapas/routing deben estar desacopladas para cambiar proveedor.
- La UX del conductor debe reducir taps y errores en contexto de calle.

## Alcance funcional

### Must Have

- Autenticacion y organizaciones multi-tenant.
- Roles: admin, dispatcher, driver, viewer.
- CRUD de clientes, direcciones, vehiculos y conductores.
- Importacion manual o CSV de paradas.
- Autocomplete/geocoding para direcciones.
- Optimizacion de una ruta multi-stop.
- Asignacion de ruta a conductor y vehiculo.
- App movil para ver ruta del dia.
- Accion de abrir navegacion externa por parada.
- Estados de parada: pendiente, en camino, llegada, completada, fallida, omitida.
- Tracking GPS en tiempo real con frecuencia configurable.
- Prueba de entrega: foto, firma, nota, timestamp, coordenadas.
- Panel operativo con rutas activas, mapa, progreso y reasignacion.
- Historial de rutas y paradas.
- Auditoria basica de acciones.

### Should Have

- Optimizacion multi-vehiculo.
- Ventanas horarias por parada.
- Prioridad por parada.
- Capacidades por vehiculo.
- Replanificacion parcial de ruta activa.
- Link publico de tracking para cliente.
- Notificaciones por email o push.
- Modo offline con cola de eventos.
- Dashboard de KPIs operativos.
- Integracion con WhatsApp/SMS solo si el costo esta aprobado.

### Could Have

- Captura de codigo de barras o QR.
- ETA por cliente.
- Chat operador-driver.
- Plantillas de rutas recurrentes.
- Zonas geograficas y territorios.
- Billing interno por conductor si se vende como SaaS.
- Webhooks para integracion con e-commerce o ERP.

### Won't Have en MVP

- Navegacion turn-by-turn embebida.
- Motor self-hosted de mapas desde el primer dia.
- Optimizacion compleja con restricciones avanzadas de flota.
- IA para prediccion de trafico propia.
- Marketplace de drivers.

## Requisitos funcionales

### Direcciones y geocoding

- `PRD-ADR-001`: El sistema debe permitir buscar direcciones con autocomplete.
- `PRD-ADR-002`: El sistema debe guardar coordenadas geocodificadas por parada.
- `PRD-ADR-003`: El sistema debe soportar reverse geocoding para eventos GPS y POD.
- `PRD-ADR-004`: Los resultados geocodificados permanentes deben respetar el SKU/licencia del proveedor.
- `PRD-ADR-005`: El sistema debe evitar geocoding repetido de la misma direccion normalizada.

### Rutas y optimizacion

- `PRD-RTE-001`: Un dispatcher debe poder crear una ruta con nombre, fecha, depot opcional y lista de paradas.
- `PRD-RTE-002`: Una ruta debe soportar parada inicial y final configurables.
- `PRD-RTE-003`: El sistema debe ordenar paradas usando el proveedor de optimizacion configurado.
- `PRD-RTE-004`: El sistema debe persistir el resultado de optimizacion con distancia, duracion estimada y proveedor.
- `PRD-RTE-005`: El sistema debe permitir bloquear manualmente el orden de una parada.
- `PRD-RTE-006`: El sistema debe permitir publicar una ruta para un driver.
- `PRD-RTE-007`: El sistema debe permitir reoptimizar paradas pendientes sin alterar paradas completadas.
- `PRD-RTE-008`: El sistema debe soportar multi-vehiculo como fase 2.

### Despacho y seguimiento

- `PRD-DSP-001`: El dispatcher debe ver rutas activas en un mapa.
- `PRD-DSP-002`: El dispatcher debe filtrar rutas por fecha, conductor, estado y zona.
- `PRD-DSP-003`: El dispatcher debe ver la ultima posicion conocida del driver.
- `PRD-DSP-004`: El dispatcher debe reasignar una ruta o una parada pendiente.
- `PRD-DSP-005`: El sistema debe registrar cada cambio de estado como evento inmutable.

### App del conductor

- `PRD-DRV-001`: El driver debe ver solo sus rutas asignadas.
- `PRD-DRV-002`: El driver debe iniciar una ruta.
- `PRD-DRV-003`: El driver debe abrir navegacion externa hacia la siguiente parada.
- `PRD-DRV-004`: El driver debe marcar llegada, completar, fallar u omitir una parada.
- `PRD-DRV-005`: El driver debe capturar POD antes de completar si la ruta lo requiere.
- `PRD-DRV-006`: La app debe guardar eventos offline y sincronizarlos cuando vuelva la red.
- `PRD-DRV-007`: La app debe mostrar errores accionables y permitir reintento.

### Prueba de entrega

- `PRD-POD-001`: La POD debe admitir foto.
- `PRD-POD-002`: La POD debe admitir firma digital.
- `PRD-POD-003`: La POD debe admitir nota de texto.
- `PRD-POD-004`: La POD debe guardar timestamp, coordenadas, usuario, ruta y parada.
- `PRD-POD-005`: La POD debe quedar asociada a un evento de completado o fallo.
- `PRD-POD-006`: Las fotos deben subirse a storage privado con URLs firmadas.

### Notificaciones

- `PRD-NTF-001`: El sistema debe soportar notificaciones por evento, aunque el MVP puede dejarlas desactivadas.
- `PRD-NTF-002`: SMS/WhatsApp deben ser feature flags por organizacion por impacto de costo.
- `PRD-NTF-003`: Los mensajes al cliente no deben incluir datos sensibles innecesarios.

## Estados

### Estados de ruta

| Estado | Significado |
| --- | --- |
| `draft` | Ruta editable no publicada |
| `optimized` | Ruta con secuencia calculada |
| `published` | Ruta asignada y visible al driver |
| `in_progress` | Driver inicio ejecucion |
| `paused` | Ruta detenida temporalmente |
| `completed` | Todas las paradas terminales |
| `cancelled` | Ruta anulada |

### Estados de parada

| Estado | Significado |
| --- | --- |
| `pending` | Aun no visitada |
| `next` | Siguiente parada sugerida |
| `en_route` | Driver va hacia la parada |
| `arrived` | Driver marco llegada |
| `completed` | Entrega/visita exitosa |
| `failed` | No se pudo completar |
| `skipped` | Omitida por decision operativa |
| `reassigned` | Movida a otro driver/ruta |

## Requisitos no funcionales

- Costo objetivo inicial: USD 0-25/mes para micro-flota; USD 25-60/mes para uso real chico/mediano, segun supuestos provistos.
- Latencia de panel operativo: cambios de estado visibles en menos de 5 segundos en condiciones normales.
- Tracking GPS: intervalo configurable por organizacion, por defecto 30-60 segundos durante ruta activa.
- Offline: eventos criticos del driver deben persistir localmente hasta confirmacion remota.
- Seguridad: RLS por organizacion; drivers no pueden leer rutas ajenas.
- Auditoria: cambios de asignacion, estado, POD y replanificacion deben quedar registrados.
- Privacidad: retencion limitada de ubicaciones crudas; agregar politicas por plan.
- Observabilidad: logs estructurados para geocoding, optimizacion, sync y POD.

## Metricas

### North Star

Rutas completadas con POD valida por conductor activo por semana.

### Operativas

- Porcentaje de paradas completadas.
- Tiempo promedio por parada.
- Kilometros estimados vs realizados.
- Rutas replanificadas.
- Paradas fallidas por motivo.
- Latencia de actualizacion GPS.

### Producto

- Drivers activos diarios.
- Rutas creadas por semana.
- Tiempo desde importacion a publicacion.
- Porcentaje de rutas optimizadas vs manuales.
- Errores de geocoding por cada 100 direcciones.
- Eventos offline sincronizados correctamente.

### Costos

- Costo mensual total por organizacion.
- Costo mensual por conductor activo.
- Requests de geocoding por ruta.
- Requests de optimization por ruta.
- Storage usado por POD.
- Peak realtime connections.

## Criterios de exito del MVP

- Un dispatcher puede crear una ruta de al menos 20 paradas, optimizarla y asignarla.
- Un driver puede ejecutar la ruta desde movil y completar paradas con POD.
- El panel muestra progreso y ubicacion actual sin refrescar manualmente.
- Una perdida de conexion movil no elimina eventos ni POD pendientes.
- El costo operativo por driver queda por debajo del benchmark de Zeo para una flota chica.
