# Insurtech Stock Dinamico

Seguro de mercaderia con prima variable atada al valor real del inventario en tiempo real.

## Problema

Los comercios e industrias aseguran mercaderia a un valor fijo, pero el stock fluctua
constantemente. Esto genera:

- **Infraseguro**: al momento del siniestro hay mas mercaderia que la asegurada.
  La aseguradora aplica regla proporcional (Art. 65, Ley 17.418) y paga menos.
- **Sobreseguro**: se paga prima por cobertura que no se necesita.

## Solucion

Integracion automatica con el sistema de gestion de stock (ERP/WMS) del cliente
que ajusta la cobertura y la prima diaria o semanalmente, sin intervencion humana.

## Modelo de negocio

Agente Institorio (equivalente argentino de MGA) que:

- Desarrolla la plataforma tecnologica (integracion ERP -> motor de pricing)
- Suscribe polizas en nombre de una aseguradora partner
- Cobra comision de gestion (15-25%) + fee tecnologico

## Estado del arte

| Empresa | Pais | Modelo | Frecuencia ajuste |
|---------|------|--------|-------------------|
| Loadsure (Huron) | UK/USA | Stock throughput dinamico | Cotizacion dinamica por envio |
| StartSure | USA | Inventario e-commerce (Shopify) | Mensual (30 dias) |
| Polizas tradicionales | Argentina | Declaracion jurada manual | Mensual/anual |
| **Este proyecto** | **Argentina** | **ERP/WMS automatico** | **Diario/semanal** |

## Marco regulatorio

- Ley 17.418 (Ley de Seguros): soporta primas variables y ajustes frecuentes
- SSN: permite polizas con ajuste de suma asegurada
- Agente Institorio: regulado por SSN, registro en RAI + IGJ

## Canal de entrada

Camara Insurtech Argentina - Programa LinkUp (conecta startups con aseguradoras)

## Aseguradoras target

- Hipotecario Seguros (partner activo LinkUp)
- HDI Seguros (partner LinkUp 2024)
- Galicia Seguros (partner LinkUp)
- Sancor Seguros (producto mercaderia existente)
- Federacion Patronal (fuerte en PyMEs)

## Estructura del proyecto

```
insurtech-stock-dinamico/
  README.md                  - Este archivo
  research/
    market-analysis.md       - Analisis de mercado y competencia
    regulatory.md            - Marco regulatorio detallado
    ip-protection.md         - Estrategia de propiedad intelectual
  business/
    business-model.md        - Modelo de negocio detallado
    pitch-deck-outline.md    - Estructura del pitch para LinkUp
    financial-projections.md - Proyecciones financieras
  technical/
    architecture.md          - Arquitectura tecnica
    erp-integrations.md      - Integraciones ERP/WMS
    pricing-engine.md        - Motor de calculo de prima
```
