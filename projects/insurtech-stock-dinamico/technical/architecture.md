# Arquitectura Tecnica

## Vision general

Plataforma que conecta el sistema de gestion de inventario del cliente (ERP/WMS)
con el motor de pricing de la aseguradora, ajustando cobertura y prima
automaticamente segun el valor real del stock.

## Diagrama de componentes

```
+------------------+      +-----------------+      +------------------+
|   CLIENTE        |      |   PLATAFORMA    |      |   ASEGURADORA    |
|                  |      |                 |      |                  |
| ERP/WMS          | ---> | Conectores ERP  |      |                  |
| (Tango, Colppy,  |  API | (adaptadores)   |      |                  |
|  Bejerman, SAP,  |      |       |         |      |                  |
|  Xubio, Excel)   |      |       v         |      |                  |
|                  |      | Motor de        |      |                  |
|                  |      | Valuacion       |      |                  |
|                  |      |   (valor stock) |      |                  |
|                  |      |       |         |      |                  |
|                  |      |       v         |      |                  |
|                  |      | Motor de        | ---> | Sistema de       |
|                  |      | Pricing         |  API | polizas          |
|                  |      |   (prima)       |      |                  |
|                  |      |       |         |      | Endoso automatico|
|                  |      |       v         |      |                  |
| Dashboard        | <--- | Portal cliente  |      | Facturacion      |
| (valor, prima,   |      | + Notificaciones|      | variable         |
|  historial)      |      |                 |      |                  |
+------------------+      +-----------------+      +------------------+
```

## Componentes principales

### 1. Conectores ERP (Capa de integracion)

Adaptadores para cada sistema de gestion de stock del mercado argentino.

**ERPs target (por orden de prioridad)**:

| ERP | Mercado | Integracion | Prioridad |
|-----|---------|-------------|-----------|
| Tango Gestion | PyMEs Argentina, muy extendido | API REST (Axoft Cloud) | Alta |
| Colppy | PyMEs cloud-native | API REST documentada | Alta |
| Bejerman | Medianas empresas | API / base de datos | Media |
| Xubio | PyMEs cloud | API REST | Media |
| SAP Business One | Empresas medianas-grandes | API REST / OData | Baja (fase 2) |
| Excel/Google Sheets | Micro-comercios | Import manual + webhook | Alta (MVP) |

**Patron de integracion**:

```
interface ERPConnector {
  // Configuracion inicial
  connect(credentials): Connection
  
  // Lectura de stock
  getInventorySnapshot(): InventoryItem[]
  getInventoryChanges(since: Date): InventoryDelta[]
  
  // Metadata
  getProductCatalog(): Product[]
  getWarehouses(): Warehouse[]
}

interface InventoryItem {
  sku: string
  description: string
  quantity: number
  unitCost: number          // costo unitario
  totalValue: number        // cantidad x costo
  warehouse: string
  lastUpdated: Date
  category: string          // para segmentacion de riesgo
}
```

**Frecuencia de sincronizacion**:
- **Tiempo real** (webhook/push): cuando el ERP lo soporte
- **Polling programado**: cada 4-8 horas como minimo
- **Batch diario**: consolidacion nocturna para calculo de prima
- **Snapshot semanal**: para ajuste formal de poliza

### 2. Motor de Valuacion

Calcula el valor asegurable del stock respetando Art. 87 de Ley 17.418.

**Reglas de valuacion**:

```
function calcularValorAsegurable(items: InventoryItem[]): ValuationResult {
  return items.map(item => {
    if (item.isManufactured) {
      // Art. 87: mercaderia producida -> costo de fabricacion
      value = item.manufacturingCost * item.quantity
    } else {
      // Art. 87: otra mercaderia -> precio de adquisicion
      value = item.acquisitionCost * item.quantity
    }
    
    // Art. 87: no puede exceder precio de venta
    maxValue = item.salePrice * item.quantity
    value = Math.min(value, maxValue)
    
    return { sku: item.sku, insuredValue: value }
  })
}
```

**Salida**: valor total asegurable del stock, desglosado por:
- Deposito/ubicacion
- Categoria de producto
- Nivel de riesgo (inflamable, perecedero, alto valor, etc.)

### 3. Motor de Pricing

Calcula la prima basandose en el valor asegurable y factores de riesgo.

**Inputs**:
- Valor asegurable del stock (del motor de valuacion)
- Perfil de riesgo del establecimiento (zona, tipo construccion, medidas seguridad)
- Historial de siniestros del cliente
- Factores de mercado (tablas de la aseguradora partner)

**Formula base**:

```
primaAnual = valorAsegurable * tasaBase * factorRiesgo * factorSiniestralidad

donde:
  tasaBase          = tasa de la aseguradora para el ramo (ej: 0.15% - 0.50%)
  factorRiesgo      = f(zona, construccion, seguridad, tipo mercaderia)
  factorSiniestralidad = f(historial del cliente, historial del segmento)

primaDiaria = primaAnual / 365
primaSemanal = primaAnual / 52
primaMensual = primaAnual / 12
```

**Ajuste dinamico**:

```
Dia 1:  stock = $10M  -> prima diaria = $10M * 0.3% / 365 = $82.19
Dia 15: stock = $7M   -> prima diaria = $7M  * 0.3% / 365 = $57.53
Dia 30: stock = $12M  -> prima diaria = $12M * 0.3% / 365 = $98.63

Prima del mes = suma de primas diarias = variable
```

### 4. Motor de Endosos

Genera endosos automaticos (modificaciones a la poliza) cuando el valor
del stock cambia significativamente.

**Reglas de trigger**:
- Cambio > 10% del valor asegurado -> endoso automatico
- Cambio > 25% -> endoso + notificacion al cliente
- Cambio > 50% -> endoso + alerta a la aseguradora + revision manual

**Tipos de endoso**:
- Aumento de suma asegurada (stock sube)
- Disminucion de suma asegurada (stock baja)
- Cambio de composicion de riesgo (cambia tipo de mercaderia)

### 5. Portal del cliente (Dashboard)

**Funcionalidades**:
- Valor actual del stock asegurado (tiempo real)
- Prima actual (diaria/semanal/mensual)
- Historial de valuaciones (grafico temporal)
- Historial de primas pagadas
- Certificado de cobertura vigente (descargable)
- Alertas y notificaciones
- Configuracion de integracion ERP

### 6. Portal de la aseguradora

**Funcionalidades**:
- Cartera de clientes con stock dinamico
- Exposicion total en tiempo real
- Alertas de concentracion de riesgo
- Reportes actuariales
- Gestion de endosos automaticos
- Configuracion de tasas y factores

## Stack tecnologico sugerido

| Capa | Tecnologia | Justificacion |
|------|-----------|---------------|
| Backend API | Node.js (Express/Fastify) | Ecosistema rico, async nativo |
| Base de datos | PostgreSQL | Transaccional, time-series con TimescaleDB |
| Cache | Redis | Snapshots de stock en memoria |
| Cola de mensajes | RabbitMQ o SQS | Procesamiento asincrono de syncs |
| Frontend | React + Next.js | Dashboard interactivo |
| Mobile | React Native o PWA | Notificaciones push |
| Infra | AWS o GCP | Escalabilidad, compliance |
| Integraciones | Webhooks + REST adapters | Conectores ERP |

## Modelo de datos simplificado

```
Cliente
  - id, razonSocial, cuit, rubro
  - polizaId, aseguradoraId
  - erpType, erpCredentials (encriptadas)

Poliza
  - id, clienteId, aseguradoraId
  - sumaAseguradaActual, primaAnualActual
  - fechaInicio, fechaVencimiento
  - condicionesParticulares

SnapshotStock
  - id, clienteId, fecha
  - valorTotal, cantidadItems
  - detalleJson (por SKU)

AjustePrima
  - id, polizaId, fecha
  - valorAnterior, valorNuevo
  - primaAnterior, primaNueva
  - endosoGenerado (bool)

Endoso
  - id, polizaId, fecha, tipo
  - cambioSumaAsegurada
  - cambioprima
  - aprobadoAutomatico (bool)
```

## Seguridad

- Credenciales ERP encriptadas con AES-256 en reposo
- Comunicacion TLS 1.3 en transito
- OAuth2 para autenticacion de usuarios
- API keys rotables para integraciones ERP
- Audit log de todos los cambios de poliza
- Cumplimiento con normativa SSN de proteccion de datos
- Backups diarios con retencion 7 anios (requisito legal seguros)

## Fases de desarrollo

### MVP (3-4 meses)
- Conector para 1 ERP (Tango o Colppy) + Excel/CSV
- Motor de valuacion basico
- Motor de pricing con tasa fija configurable
- Dashboard cliente minimo (valor stock + prima)
- Endoso manual (notificacion a aseguradora)

### V1 (6-8 meses)
- 3-4 conectores ERP
- Motor de pricing con factores de riesgo
- Endoso semi-automatico
- Dashboard completo cliente + aseguradora
- App mobile (PWA)

### V2 (12-18 meses)
- Todos los ERPs principales
- Pricing con ML (prediccion de stock, deteccion anomalias)
- Endoso 100% automatico
- API abierta para aseguradoras
- Expansion a otros paises LATAM
