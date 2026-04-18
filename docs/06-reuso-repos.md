# 06 - Reuso de Repos Open Source

## Objetivo

Evaluar repos existentes que ataquen partes del problema para acelerar entrega sin
comprometer arquitectura, costos o mantenibilidad.

Regla base: **adoptar patrones y componentes aislables**, no acoplar el core del
producto a una demo o a un repo sin mantenimiento activo.

## Matriz de candidatos

| Repo | Lo que aporta | Riesgo principal | Decision |
| --- | --- | --- | --- |
| `googlemaps/js-route-optimization-app` | Modelo VRP serio: shipments, vehicles, constraints, visualizacion | El README aclara uso exploratorio, no para ruta critica productiva | Adoptar como referencia tecnica de modelado y pruebas de constraints |
| `Kasheftin/route-planner-vue` | Base visual de planificacion, capas, waypoints, export/import JSON | Stack viejo y acoplado a Google Maps UI clasica | Reusar ideas de UX/estructura, no base de codigo |
| `biz3e/QuickRoute-WebApp` | MVP web simple de optimizacion con inicio optimizable y criterio distancia/duracion | Alcance funcional limitado, sin operacion live | Reusar flujo de onboarding de ruta para MVP |
| `vraa/route-planner` | Ejemplo React simple de waypoints con distancia/tiempo | Limite bajo de waypoints; arquitectura antigua | Solo referencia para UI minima de planificador |
| `eranabir/react-native-navigation-apps` | Handoff a apps externas (Waze, Google Maps, iOS Maps) con direccion o lat/lon | Libreria antigua con poco movimiento reciente | Crear adapter interno; evaluar wrapper propio con deep links |
| `CarlosSTS/OpenMapsApp` | Demo React Native reciente para abrir Uber/Waze/Google Maps + notas Android 11 | Es demo, no planner | Tomar checklist Android package queries y patron de apertura segura |
| `fabiocaccamo/FCMapsApp` | Wrapper iOS para Apple/Google/Waze/Yandex | Legacy (iOS muy antiguo), no apto uso moderno directo | Solo referencia de esquemas URL y fallback entre apps |

## Evidencia validada (README)

### 1) `googlemaps/js-route-optimization-app`

- Indica explicitamente que resuelve VRP con envios, vehiculos, costos y restricciones.
- Indica explicitamente que es herramienta exploratoria y no para production critical path.

Implicacion:

- Sirve para definir el **modelo de dominio de optimizacion** y casos de prueba.
- No se debe incrustar como modulo operativo en produccion.

### 2) `Kasheftin/route-planner-vue`

- Declara marcadores/capas ilimitados.
- Declara hasta 20 waypoints por ruta.
- Declara export/import JSON.

Implicacion:

- Sirve para diseñar UX del builder y formato portable de escenarios.
- No conviene usar su stack base como foundation de producto.

### 3) `biz3e/QuickRoute-WebApp`

- Declara optimizacion de multi-stop.
- Declara punto de inicio elegible u optimizable.
- Declara calculo por distancia o duracion.

Implicacion:

- Muy util para definir flujo “CSV -> optimize -> compare”.

### 4) `vraa/route-planner`

- Declara hasta 8 waypoints entre origen y destino.
- Muestra distancia y tiempo por waypoint y ruta total.

Implicacion:

- Referencia de experiencia basica.
- No cubre escenarios de “muchas paradas”.

### 5) `react-native-navigation-apps` (eranabir)

- Describe apertura y navegacion con Waze, Google Maps e iOS Maps.
- Describe navegacion por direccion o lat/lon.

Implicacion:

- Encaja perfecto con nuestra decision de navegacion externa en MVP.
- Por antiguedad, conviene encapsular y poder reemplazar.

### 6) `CarlosSTS/OpenMapsApp`

- Demuestra apertura de Uber, Waze y Google Maps.
- Incluye nota tecnica Android 11+ sobre `queries` en `AndroidManifest`.

Implicacion:

- Referencia practica para robustecer integracion Android.

### 7) `FCMapsApp`

- Soporta Apple Maps, Google Maps, Waze y Yandex Maps.
- Foco iOS y apertura de apps externas.

Implicacion:

- Referencia util de fallback y esquemas URL.
- No adoptar directamente por obsolescencia.

## Decisiones de adopcion

### ADP-001: Reusar modelo VRP de Google como benchmark interno

- Crear fixtures de shipments/vehicles/time windows inspirados en su data model.
- Validar que nuestros adapters de optimizacion soporten esos casos.

### ADP-002: Estandarizar formato JSON de escenario de ruta

- Tomar inspiracion de export/import de `route-planner-vue`.
- Definir contrato propio versionado para import/export de rutas y stops.

### ADP-003: Implementar adapter propio de navegacion externa

- API interna:
  - `canOpen(app)`
  - `openNavigate({lat,lng,address,appPreference})`
  - `openChooser({lat,lng,address})`
- Primer backend RN: deep links directos + validacion de apps instaladas.
- No acoplar negocio a una libreria de tercero sin mantenimiento.

### ADP-004: Mantener repos externos como referencia, no como core

- Ninguno de los repos evaluados reemplaza nuestros modulos de:
  - multi-tenant
  - dispatch live
  - tracking realtime
  - POD
  - auditoria

## Plan de integracion por iteraciones

### Iteracion A - Planning spike

- Importar 3 escenarios tipo VRP con restricciones reales.
- Probarlos contra provider `mapbox` y provider `ors`.
- Medir costo y tiempo por optimizacion.

### Iteracion B - Route Builder UX spike

- Implementar builder con:
  - lista de stops
  - orden manual
  - optimize
  - export/import JSON versionado

### Iteracion C - External Navigation spike (mobile)

- Implementar adapter navegacion externa en RN.
- Validar iOS + Android:
  - Google Maps
  - Waze
  - Apple Maps (iOS)
- Validar fallback cuando no hay app instalada.

## Criterios de aceptacion de reuso

- Cualquier componente de tercero debe estar encapsulado detras de interfaz interna.
- Debe existir prueba E2E que pase igual si se reemplaza la libreria externa.
- Debe haber plan de salida (exit strategy) por repo adoptado.
- Si un repo no tiene actividad reciente, se permite solo como referencia documental.

## Riesgos de adopcion

| Riesgo | Impacto | Mitigacion |
| --- | --- | --- |
| Acoplarse a demo no productiva | Deuda tecnica alta | Extraer solo patrones, no montar core encima |
| Librerias viejas de navegacion | Fallos en OS modernos | Adapter propio + pruebas en dispositivos reales |
| Licencias o terminos incompatibles | Riesgo legal | Revisar licencia antes de copiar codigo |
| Copiar modelo sin contexto de costo | Sobreconsumo API | Benchmarks de costo por escenario |
| Portar UI legacy | UX inconsistente y lenta | Rediseño moderno sobre nuestros componentes |
