# Creador de Rutas

Repositorio de especificaciones para construir una app de planificacion, despacho,
tracking y prueba de entrega inspirada en Zeo Route Planner, optimizada para operar
con bajo costo inicial.

## Documentos

- [00 - Producto](docs/00-producto.md): vision, alcance, usuarios, requisitos y metricas.
- [01 - Arquitectura](docs/01-arquitectura.md): stack recomendado, modulos, flujos, integraciones y decisiones tecnicas.
- [02 - Modelo de datos](docs/02-modelo-datos.md): esquema Supabase/PostgreSQL, relaciones, indices y politicas RLS.
- [03 - API](docs/03-api.md): endpoints, eventos realtime, errores, idempotencia y contratos.
- [04 - UX y flujos](docs/04-ux-flujos.md): app del driver, panel operativo, estados, offline y accesibilidad.
- [05 - Roadmap y costos](docs/05-roadmap-costos.md): fases, criterios de aceptacion, pruebas, riesgos y escenarios de costo.
- [06 - Reuso de repos](docs/06-reuso-repos.md): analisis de candidatos open source, decision de adopcion y plan de integracion.

## Decision base

La app no debe clonar toda la navegacion embebida de Zeo desde el dia uno. El MVP
debe cubrir planificacion, optimizacion, despacho, tracking, prueba de entrega y
panel operativo, delegando la navegacion a Google Maps, Waze o Apple Maps mediante
deep links.

Stack recomendado para Fase 1:

- Supabase para auth, PostgreSQL, realtime y almacenamiento de fotos/POD.
- Mapbox como proveedor managed principal para mapas, geocoding, directions y optimization.
- Adaptador compatible con openrouteservice para reducir costo en MVP o pruebas.
- MapLibre si se decide desacoplar render de mapas del proveedor.
- Navegacion externa hasta que el volumen justifique Navigation SDK o self-host.
