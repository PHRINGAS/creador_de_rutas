# 04 - UX y Flujos

## Enfoque

La experiencia debe separar claramente dos productos:

- Panel operativo: alta densidad de informacion, mapa, filtros, replanificacion y auditoria.
- App driver: pocas decisiones, botones grandes, tolerancia offline y navegacion externa.

El driver esta en movimiento, con red variable, bateria limitada y atencion dividida.
La app movil no debe copiar patrones de escritorio.

## Evaluacion MFRI

| Variante | Riesgo | Lectura |
| --- | --- | --- |
| MVP con navegacion externa | Moderado | Interacciones simples, pero tracking/offline/POD agregan complejidad |
| Navegacion embebida | Alto | GPS, mapa activo, instrucciones, bateria, permisos y sesion larga |
| Self-host OSM desde dia uno | Alto | Reduce costo variable, pero aumenta riesgo operacional |

Mitigaciones del MVP:

- Botones tactiles de 44-48px minimo.
- Flujos lineales por parada.
- Sin gestos obligatorios.
- Estados y errores recuperables.
- Cola offline visible.
- Upload de POD con reintento.
- Mapa como apoyo, no como unica interfaz.

## App driver

### Navegacion principal

Estructura recomendada:

- `Hoy`: ruta asignada y progreso.
- `Historial`: rutas completadas recientes.
- `Cuenta`: perfil, permisos, sync, soporte.

Evitar demasiadas tabs. El trabajo diario del driver debe estar en la primera pantalla.

### Pantalla `Hoy`

Contenido:

- Nombre de ruta y fecha.
- Estado: publicada, en progreso, pausada, completada.
- Progreso: `8/24 paradas`.
- Siguiente parada destacada.
- Boton primario: iniciar ruta / continuar / navegar.
- Lista virtualizada de paradas.
- Indicador offline/sync.

Acciones:

- Iniciar ruta.
- Pausar.
- Ver detalle de parada.
- Abrir navegacion externa.

### Componente de parada

Informacion visible:

- Secuencia.
- Nombre del cliente o etiqueta.
- Direccion corta.
- Estado.
- Ventana horaria si existe.
- Indicacion especial si hay POD requerida.

Acciones:

- Ver detalle.
- Navegar.

No mostrar demasiados campos en la lista; el detalle contiene el resto.

### Detalle de parada

Contenido:

- Direccion completa.
- Cliente y contacto.
- Instrucciones.
- Historial minimo de eventos de esa parada.
- Mapa pequeno o preview.
- Botones segun estado.

Estados y CTAs:

| Estado | CTA principal | CTA secundaria |
| --- | --- | --- |
| `pending` | Navegar | Marcar omitida |
| `en_route` | Llegue | Reportar problema |
| `arrived` | Completar con POD | Fallida |
| `completed` | Ver POD | Ninguna |
| `failed` | Ver motivo | Reintentar si dispatcher permite |

### Flujo de POD

1. Driver toca `Completar`.
2. App muestra requisitos: foto, firma, nota, receptor.
3. Driver captura foto.
4. Driver captura firma si aplica.
5. Driver confirma.
6. App guarda localmente y sube.
7. Si no hay red, queda como `pendiente de sincronizacion`.
8. Cuando el backend confirma, la parada pasa a `completed`.

Reglas UX:

- Nunca perder una foto por falla de red.
- Mostrar progreso de upload.
- Permitir reintento.
- Comprimir imagen antes de subir.
- Confirmar antes de descartar firma/foto.

### Navegacion externa

Opciones:

- Google Maps.
- Waze.
- Apple Maps en iOS.

Deep link generado desde coordenadas, no desde texto de direccion cuando sea posible.

Ejemplo conceptual:

```text
google.navigation:q={lat},{lng}
waze://?ll={lat},{lng}&navigate=yes
maps://?daddr={lat},{lng}
```

La app debe registrar que el driver inicio navegacion hacia una parada, pero no
necesita controlar el turn-by-turn.

### Offline

Estados visibles:

- `Online`: sincronizado.
- `Sin conexion`: guardando localmente.
- `Sincronizando`: enviando eventos.
- `Requiere accion`: fallaron eventos.

La app debe permitir:

- Ver ruta ya descargada.
- Abrir navegacion externa con coordenadas guardadas.
- Cambiar estado de parada.
- Capturar POD.
- Enviar cola al recuperar red.

No debe permitir:

- Optimizar ruta offline.
- Reasignar rutas offline.
- Descargar rutas no cacheadas.

### Permisos moviles

Permisos necesarios:

- Ubicacion durante uso para tracking.
- Camara para POD.
- Fotos/archivos si se permite adjuntar desde galeria.
- Notificaciones push si se activan.

La app debe explicar por que pide cada permiso antes del prompt nativo.

## Panel operativo

### Navegacion recomendada

- Dashboard.
- Rutas.
- Mapa en vivo.
- Clientes.
- Drivers.
- Vehiculos.
- Historial.
- Configuracion.

### Dashboard

KPIs:

- Rutas activas.
- Drivers online.
- Paradas completadas / pendientes / fallidas.
- Rutas retrasadas.
- POD pendientes de revision.
- Costo estimado de proveedor del mes.

### Route Builder

Flujo:

1. Crear ruta con fecha y nombre.
2. Agregar paradas manualmente o importar CSV.
3. Geocodificar direcciones.
4. Corregir direcciones con baja confianza.
5. Optimizar.
6. Revisar orden en mapa/lista.
7. Asignar driver y vehiculo.
8. Publicar.

Estados necesarios:

- Direcciones sin coordenadas.
- Geocoding ambiguo.
- Paradas duplicadas.
- Proveedor de optimizacion sin capacidad suficiente.
- Ruta excede limites del plan/proveedor.

### Mapa en vivo

Debe mostrar:

- Rutas activas por color.
- Drivers con ultima ubicacion y freshness.
- Stops por estado.
- Panel lateral con progreso.
- Filtros por driver, ruta y estado.

No debe saturar:

- Agrupar markers en zoom bajo.
- Mostrar etiquetas solo al acercar.
- Actualizar ubicacion con throttling.

### Replanificacion

Casos:

- Driver se queda sin disponibilidad.
- Parada urgente entra en ruta activa.
- Cliente cancela.
- Zona bloqueada o demora.

Reglas:

- No mover stops completados.
- No borrar eventos historicos.
- Crear evento de replanificacion.
- Avisar al driver afectado.
- Mantener version de ruta para resolver concurrencia.

### Importacion CSV

Columnas minimas:

- `customer_name`
- `address`
- `phone`
- `notes`

Columnas opcionales:

- `external_ref`
- `time_window_start`
- `time_window_end`
- `service_minutes`
- `priority`
- `requires_photo`
- `requires_signature`

La pantalla debe mostrar preview y errores antes de crear stops.

## Pagina publica de tracking

Debe ser opcional y limitada.

Mostrar:

- Estado general.
- Ventana ETA.
- Cantidad de paradas previas si se desea.
- Mensaje de contacto de la empresa.

No mostrar:

- Ubicacion exacta continua del driver por defecto.
- Otros clientes.
- Telefono personal del driver.
- Ruta completa.

## Accesibilidad

- Targets tactiles minimo 44px/48dp.
- Contraste AA.
- Texto escalable.
- Estados no deben depender solo de color.
- Botones destructivos requieren confirmacion.
- La firma debe tener alternativa: nombre recibido + foto/nota si el usuario no puede firmar.

## Performance

Movil:

- Listas con virtualizacion.
- No usar mapa pesado como home si la ruta tiene muchas paradas.
- Compresion de imagen antes de upload.
- GPS solo durante ruta activa.
- Intervalo de tracking remoto configurable.

Web:

- Cluster de markers.
- Paginacion de historial.
- Suscripcion realtime solo a rutas activas.
- Fetch incremental de eventos.

## Textos de estado sugeridos

- `Sin conexion. Guardamos tus cambios en este telefono.`
- `Hay 3 eventos pendientes de sincronizar.`
- `No pudimos subir la foto. Reintentar.`
- `Esta parada ya fue completada. Pide al operador que la reabra si hubo un error.`
- `Direccion ambigua. Elegi una opcion antes de optimizar.`
