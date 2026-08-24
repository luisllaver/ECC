# Arquitectura Tecnica — StockShield Platform

## Vision

No somos un "seguro digital". Somos una **capa de inteligencia financiera
continua** entre el inventario fisico y el mercado asegurador.

La plataforma opera como un **gemelo digital del riesgo de inventario**:
un modelo vivo que refleja en tiempo real que hay en cada deposito, cuanto
vale, que riesgo representa, y cuanto deberia costar protegerlo.

Esto no existe en ningun mercado del mundo con esta profundidad.

## Principios de diseno

1. **Event-driven**: todo es un evento (ingreso de mercaderia, venta, cambio
   de precio, alerta de sensor). El sistema reacciona, no consulta.
2. **Data as moat**: cada dato que fluye construye un modelo actuarial que
   ningun competidor puede replicar sin anios de operacion.
3. **Plugin architecture**: agregar un ERP nuevo o una aseguradora nueva
   es escribir un adaptador, no modificar el core.
4. **Zero-trust en datos**: el sistema verifica, cruza y audita. Nunca
   confia ciegamente en lo que declara el cliente ni en lo que reporta el ERP.

## Arquitectura de alto nivel

```
                    CAPA DE INGESTION
                    ==================
  +----------+  +----------+  +----------+  +----------+
  |  Tango   |  |  Colppy  |  | Bejerman |  |  Excel   |
  |  Adapter |  |  Adapter |  |  Adapter |  |  Upload  |
  +----+-----+  +----+-----+  +----+-----+  +----+-----+
       |              |              |              |
       v              v              v              v
  +----------------------------------------------------------+
  |              EVENT BUS (Apache Kafka)                     |
  |   Topics: stock.changes | stock.snapshots | pricing.req  |
  |            endorsement.req | alerts | audit.log          |
  +---------------------------+------------------------------+
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
     +--------------+  +-------------+  +---------------+
     |  VALUATION   |  |   RISK      |  |   ANOMALY     |
     |  ENGINE      |  |   SCORING   |  |   DETECTOR    |
     |              |  |   ENGINE    |  |   (ML)        |
     | Art.87 rules |  |             |  |               |
     | cost/sale    |  | location    |  | fraud detect  |
     | price caps   |  | building    |  | data quality  |
     |              |  | fire safety |  | outlier alert |
     +------+-------+  +------+------+  +-------+-------+
            |                  |                  |
            v                  v                  v
     +----------------------------------------------------------+
     |              PRICING ENGINE (core IP)                     |
     |                                                          |
     |  valorAsegurable * tasaDinamica * riskScore * anomalyAdj |
     |                                                          |
     |  Outputs: primaDiaria, sumaAsegurada, endosoRequerido    |
     +---------------------------+------------------------------+
                                 |
                +----------------+----------------+
                |                                 |
                v                                 v
     +-------------------+            +---------------------+
     |  ENDORSEMENT      |            |   INSURER GATEWAY   |
     |  ENGINE           |            |                     |
     |                   |            |  API aseguradora 1  |
     | auto-endoso       |            |  API aseguradora 2  |
     | approval rules    |            |  Socotra/custom     |
     | regulatory check  |            |  Billing trigger    |
     +-------------------+            +---------------------+
                |                                 |
                v                                 v
     +----------------------------------------------------------+
     |              EXPERIENCE LAYER                             |
     |                                                          |
     |  Dashboard cliente | Dashboard aseguradora | API publica |
     |  Mobile (PWA)      | Notificaciones        | Webhooks    |
     +----------------------------------------------------------+
```

## Capa 1: Ingestion — Conectores ERP

### Investigacion de APIs reales disponibles

**Tango Gestion (Axoft) — 60.000+ empresas en Argentina**

- API REST disponible via Axoft Cloud
- Endpoint de stock: `tiendas.axoft.com/api/Aperture/Stock`
- Parametros: filtro por codigo producto, numero sucursal, codigo deposito
- Consulta datos en tiempo real (no copia paralela)
- GitHub: `TangoSoftware/ApiTiendas`

**Colppy — PyMEs cloud-native**

- API REST documentada en `apidocs.colppy.com`
- Portal de desarrolladores: `dev.colppy.com`
- Endpoint produccion: `login.colppy.com/lib/frontera2/service.php`
- Operaciones de inventario: `listar_itemsinventario`, `alta_iteminventario`,
  `editar_iteminventario`, `alta_ajusteinventario`
- Formato: HTTP POST con JSON

**Xubio — PyMEs cloud**

- API REST disponible
- Documentacion en portal de desarrolladores

**Excel/CSV — micro-comercios**

- Upload manual via dashboard web
- Template estandarizado descargable
- Parseo automatico con validacion de esquema
- Opcion Google Sheets con Apps Script para sync periodico

### Patron de conector (Adapter pattern)

```javascript
// Cada ERP implementa esta interfaz
class ERPConnector {
  // Autenticacion
  async authenticate(credentials) -> Session
  async refreshToken(session) -> Session

  // Lectura de inventario
  async getFullSnapshot() -> InventorySnapshot
  async getChanges(since: timestamp) -> InventoryDelta[]

  // Metadata del negocio
  async getProductCatalog() -> Product[]
  async getWarehouses() -> Warehouse[]
  async getSalesHistory(period) -> Sale[]

  // Health check
  async ping() -> { connected: boolean, latency: ms }
}

// Los datos se normalizan a un formato canonico
InventorySnapshot {
  clientId: string
  timestamp: ISO8601
  source: 'tango' | 'colppy' | 'bejerman' | 'xubio' | 'manual'
  warehouses: [{
    id: string
    name: string
    address: string
    geoLocation: { lat, lng }
    items: [{
      sku: string
      description: string
      quantity: number
      unitCost: number
      acquisitionCost: number
      manufacturingCost: number | null
      salePrice: number
      category: string
      riskClass: 'standard' | 'flammable' | 'perishable' | 'high_value' | 'hazardous'
      lastMovement: ISO8601
    }]
  }]
  totalValue: number
  checksum: SHA256    // integridad del snapshot
}
```

### Frecuencia y modos de sincronizacion

| Modo | Frecuencia | Tecnologia | Uso |
|------|-----------|------------|-----|
| **Push (webhook)** | Tiempo real | ERP dispara evento en cada movimiento de stock | Ideal, cuando el ERP lo soporte |
| **Polling** | Cada 1-4 horas | Cron job consulta API del ERP | Default para ERPs con API |
| **Batch nocturno** | 1x/dia (03:00 AM) | Job que toma snapshot completo | Consolidacion diaria para pricing |
| **Upload manual** | A demanda | Usuario sube CSV/Excel | Fallback, micro-comercios |

### Validacion cruzada de datos (anti-fraude, capa 1)

El sistema no confia ciegamente en el ERP. Cruza informacion:

```
Datos del ERP (stock declarado)
  x  Datos de facturacion (AFIP / factura electronica)
  x  Datos de compras (ordenes de compra del ERP)
  x  Patrones historicos (ML: "este cliente nunca tuvo stock < $2M,
     pero ahora declara $500K justo antes de renovar")

Si hay discrepancia > umbral -> FLAG para revision
```

## Capa 2: Event Bus — Apache Kafka

Todo cambio en el sistema es un **evento inmutable** que fluye por Kafka.

### Topics principales

```
stock.raw.{clientId}          Datos crudos del ERP (pre-normalizacion)
stock.normalized.{clientId}   Snapshot normalizado y validado
stock.anomaly                 Alertas de deteccion de anomalias
pricing.request               Solicitud de recalculo de prima
pricing.result                Resultado del calculo de prima
endorsement.request           Solicitud de endoso
endorsement.approved          Endoso aprobado
endorsement.rejected          Endoso que requiere revision manual
billing.event                 Evento de facturacion/cobro
audit.trail                   Todo cambio con timestamp y actor
```

### Por que Kafka y no una cola simple

- **Inmutabilidad**: cada evento queda registrado para siempre (audit trail
  regulatorio, la SSN puede auditar cualquier cambio historico)
- **Replay**: si cambiamos el motor de pricing, podemos re-procesar todo
  el historico con las nuevas reglas
- **Multiples consumidores**: el mismo evento de stock alimenta al motor
  de valuacion, al detector de anomalias, al dashboard, y al data lake
  en paralelo
- **Orden garantizado**: por particion (clientId), los eventos de un cliente
  se procesan en orden

## Capa 3: Motores de procesamiento

### 3A. Motor de Valuacion (Valuation Engine)

Transforma datos de inventario en **valor asegurable** segun Ley 17.418.

```javascript
function valuate(snapshot) {
  return snapshot.warehouses.flatMap(wh =>
    wh.items.map(item => {
      let value

      if (item.manufacturingCost !== null) {
        // Art. 87: mercaderia producida -> costo de fabricacion
        value = item.manufacturingCost * item.quantity
      } else {
        // Art. 87: otra mercaderia -> precio de adquisicion
        value = item.acquisitionCost * item.quantity
      }

      // Art. 87: tope = precio de venta al momento
      const maxValue = item.salePrice * item.quantity
      value = Math.min(value, maxValue)

      // Factor de depreciacion por antiguedad de stock
      const daysSinceLastMovement = daysBetween(item.lastMovement, now())
      if (daysSinceLastMovement > 180) {
        value *= 0.85  // stock estancado pierde valor de mercado
      }
      if (daysSinceLastMovement > 365) {
        value *= 0.70
      }

      return {
        sku: item.sku,
        warehouse: wh.id,
        insuredValue: value,
        riskClass: item.riskClass,
        concentration: value / snapshot.totalValue  // % del total
      }
    })
  )
}
```

### 3B. Motor de Scoring de Riesgo (Risk Scoring Engine)

Calcula un **riskScore** por cliente que modifica la tasa base.

```
riskScore = f(
  // Riesgo del establecimiento
  zonaGeografica,           // inundable, sismico, zona roja de robos
  tipoConstruccion,         // hormigon, chapa, madera
  sistemaAntiIncendio,      // rociadores, extintores, hidrantes
  sistemaSeguridad,         // alarma, camaras, vigilancia
  distanciaBomberos,        // km a cuartel mas cercano

  // Riesgo del inventario
  composicionRiesgo,        // % inflamable, % perecedero, % alto valor
  concentracionSKU,         // un solo SKU = 80% del valor -> mayor riesgo
  rotacionInventario,       // stock estancado = mayor riesgo
  estacionalidad,           // picos predecibles

  // Historial
  siniestrosCliente,        // frecuencia y severidad
  siniestrosSegmento,       // benchmark del rubro
  aniosComoCliente          // fidelidad

  // Datos externos (fase 2+)
  climaLocal,               // alerta meteorologica
  reportesBomberos,         // incidentes en la zona
  datosAFIP                 // consistencia fiscal
)
```

**Output**: score de 0.5 (riesgo muy bajo) a 3.0 (riesgo muy alto)
que multiplica la tasa base.

### 3C. Detector de Anomalias (ML)

Sistema de machine learning que detecta comportamientos sospechosos o
riesgosos en los datos de stock.

**Tipos de anomalias detectadas**:

| Anomalia | Indicador | Accion |
|----------|-----------|--------|
| **Fraude por subdeclaracion** | Stock reportado cae 60% sin ventas correspondientes | Flag + auditoria |
| **Fraude por sobredeclaracion** | Stock sube 200% sin compras registradas | Flag + auditoria |
| **Inconsistencia ERP** | Datos del ERP no matchean facturacion AFIP | Alerta al cliente |
| **Riesgo de concentracion** | 1 SKU = 70%+ del valor total | Ajuste de riskScore |
| **Patron pre-siniestro** | Stock sube dramaticamente antes de un reclamo | Investigacion |
| **Anomalia de rotacion** | Stock que no se mueve hace 12+ meses | Ajuste de valuacion |
| **Calidad de datos** | Campos vacios, valores negativos, duplicados | Rechazo + alerta |

**Modelos ML a implementar**:

```
Fase 1 (reglas): Umbrales fijos y reglas de negocio
  - Cambio > 50% sin transacciones correspondientes
  - Costo unitario fuera de rango historico
  - Stock negativo o imposible

Fase 2 (estadistico): Isolation Forest + Z-score
  - Deteccion de outliers multidimensional
  - Perfil de comportamiento normal por cliente
  - Alertas cuando se desvia del perfil

Fase 3 (deep learning): Autoencoder + LSTM
  - Modelo de series temporales que predice stock esperado
  - Desviacion significativa entre predicho y reportado = anomalia
  - Deteccion de patrones complejos de fraude
```

## Capa 4: Pricing Engine (core IP de la plataforma)

El motor de pricing es el corazon del sistema y la propiedad intelectual
mas valiosa.

### Formula de pricing dinamico

```
primaDiaria(t) = SA(t) * TD(t) * RS * AA * FC

donde:
  SA(t)  = Suma Asegurada en el momento t (del Valuation Engine)
  TD(t)  = Tasa Dinamica (tasa base ajustada por condiciones de mercado)
  RS     = Risk Score del cliente (del Risk Scoring Engine)
  AA     = Anomaly Adjustment (1.0 normal, >1.0 si hay flags)
  FC     = Factor Contractual (deducible, franquicia, sublimites)

primaPeriodo = SUM(primaDiaria(t)) para t en [inicio, fin]
```

### Tasa dinamica (innovacion clave)

La tasa no es fija como en seguros tradicionales. Se ajusta por:

```
tasaDinamica(t) = tasaBase
  * factorEstacionalidad(t)      // temporada alta = mas riesgo
  * factorClima(t)               // alerta meteo = mas riesgo
  * factorConcentracion(t)       // inventario concentrado = mas riesgo
  * factorMercado(t)             // condiciones del mercado asegurador
  * bonificacionHistorial(t)     // buen historial = descuento

Limites: tasaDinamica no puede variar mas de +-30% de tasaBase
(restriccion regulatoria probable de la SSN)
```

### Ejemplo de calculo real

```
FERRETERIA "EL TORNILLO" - Mes de Junio

Dia 1:  stock $8.5M  * 0.28% / 365 * 1.1 (riesgo)  = $72.04/dia
Dia 5:  stock $9.2M  * 0.28% / 365 * 1.1            = $77.97/dia
Dia 10: stock $7.1M  * 0.28% / 365 * 1.1            = $60.18/dia
Dia 15: stock $6.8M  * 0.28% / 365 * 1.1            = $57.64/dia
Dia 20: stock $11.3M * 0.30% / 365 * 1.1 (+ estac.) = $102.25/dia
Dia 25: stock $12.1M * 0.30% / 365 * 1.1            = $109.50/dia
Dia 30: stock $10.5M * 0.28% / 365 * 1.1            = $89.01/dia

Prima de Junio = suma ponderada de primas diarias
Promedio ponderado ~ $81.23/dia * 30 = $2,436.90

vs Poliza tradicional: $8.5M * 0.3% / 12 = $2,125/mes (fijo, 
   pero si se quema dia 25 con $12.1M, cobra como si tuviera $8.5M
   = regla proporcional = pierde $2.5M de cobertura)
```

## Capa 5: Endorsement Engine

### Tipos de endosos automatizados

```
MICRO-ENDOSO (automatico, sin intervencion)
  Trigger: cambio de SA < 15%
  Accion: actualizar poliza, ajustar prima
  Frecuencia: diaria/semanal
  Aprobacion: algoritmo

ENDOSO ESTANDAR (semi-automatico)
  Trigger: cambio de SA entre 15-40%
  Accion: generar endoso, notificar al cliente
  Frecuencia: semanal
  Aprobacion: cliente confirma via app

ENDOSO MAYOR (requiere aprobacion aseguradora)
  Trigger: cambio de SA > 40% o cambio de composicion de riesgo
  Accion: generar solicitud, enviar a underwriter de la aseguradora
  Frecuencia: variable
  Aprobacion: underwriter humano
```

### Integracion con sistemas de aseguradoras

```
// Gateway para multiples aseguradoras
class InsurerGateway {
  // Socotra (plataforma cloud para aseguradoras modernas)
  async socotraEndorsement(policyId, changes) -> EndorsementResult

  // API propietaria de aseguradora argentina
  async customApiEndorsement(policyId, changes) -> EndorsementResult

  // Fallback: email estructurado + PDF
  async emailEndorsement(policyId, changes) -> { sent: true }
}
```

**Nota sobre Socotra**: plataforma cloud-native de core de seguros usada por
aseguradoras modernas. API-first, permite crear productos de seguro
configurables. Ya tiene AI underwriting assistant. Potencial partner
tecnologico para aseguradoras argentinas que quieran modernizarse.

## Capa 6: Inteligencia predictiva (diferenciador exponencial)

Esto es lo que transforma la plataforma de "herramienta util" a
"infraestructura imposible de replicar".

### 6A. Prediccion de inventario

Con 6+ meses de datos de un cliente, el sistema puede:

```
- Predecir el stock de las proximas 2-4 semanas
- Identificar patrones estacionales (navidad, dia del padre, etc.)
- Anticipar picos de riesgo y notificar proactivamente
- Ofrecer "pre-ajustes" de cobertura antes de que el stock suba
```

**Modelo**: LSTM (Long Short-Term Memory) entrenado por cliente + segmento.

### 6B. Benchmark de industria

Con 100+ clientes del mismo rubro, el sistema genera:

```
- Ratio de stock promedio por rubro y tamano
- Patron de rotacion tipico
- Estacionalidad por segmento
- Tasa de siniestralidad por tipo de mercaderia

-> Esto se convierte en un PRODUCTO que se puede vender a las
   aseguradoras como data actuarial (revenue adicional)
```

### 6C. Score crediticio alternativo

El comportamiento de inventario revela la salud financiera del negocio:

```
Stock que crece sostenidamente = negocio saludable
Stock que se desploma sin ventas = problemas de liquidez
Rotacion alta y constante = operacion eficiente
Concentracion extrema en pocos SKUs = riesgo de negocio

-> Esto se puede ofrecer como score crediticio alternativo
   a bancos, fintechs, y otros financiadores (revenue adicional)
```

### 6D. Gemelo digital del deposito

Representacion virtual del deposito que integra:

```
Datos del ERP (que hay)
  + Datos IoT futuros (temperatura, humedad, movimiento)
  + Datos de ubicacion (geolocalizacion del deposito)
  + Datos de riesgo externo (clima, zona, construccion)

= Modelo 3D del riesgo que la aseguradora puede visualizar
  para decidir si suscribir y a que tasa
```

## Integracion de datos externos

### Fuentes de datos para enriquecer el risk scoring

| Fuente | Dato | Uso | Fase |
|--------|------|-----|------|
| AFIP (factura electronica) | Ventas y compras reales | Validacion cruzada anti-fraude | V1 |
| SMN (Servicio Meteorologico) | Alertas climaticas | Ajuste de tasa por tormenta/inundacion | V1 |
| Google Maps API | Distancia a bomberos, zona | Risk scoring de ubicacion | MVP |
| BCRA (Banco Central) | Tipo de cambio, inflacion | Ajuste de valor en pesos | MVP |
| Datos catastrales | Tipo de construccion | Risk scoring estructural | V1 |
| Bomberos / Defensa Civil | Historial incidentes zona | Risk scoring geografico | V2 |
| Mercado de commodities | Precios de materias primas | Valuacion de stock de commodities | V2 |

## Stack tecnologico

| Capa | Tecnologia | Por que |
|------|-----------|---------|
| **Event streaming** | Apache Kafka (managed: Confluent Cloud) | Inmutabilidad, replay, multi-consumer |
| **Backend API** | Node.js + Fastify | Performance, async nativo, ecosistema ERP |
| **Pricing Engine** | Python (microservicio) | ML libraries (scikit-learn, PyTorch) |
| **Base de datos** | PostgreSQL + TimescaleDB | Transaccional + time-series nativo |
| **Cache** | Redis | Ultimo snapshot de stock en memoria |
| **Object storage** | S3 | Snapshots historicos, documentos de poliza |
| **Frontend** | Next.js + React | SSR, dashboard interactivo |
| **Mobile** | PWA (Progressive Web App) | Cross-platform sin app store |
| **ML pipeline** | MLflow + Python | Entrenamiento, versionado, deploy de modelos |
| **Infra** | AWS (Sao Paulo region) | Latencia LATAM, compliance |
| **CI/CD** | GitHub Actions | Deploy automatizado |
| **Monitoring** | Datadog o Grafana | Observabilidad de todos los pipelines |
| **Secrets** | AWS Secrets Manager | Credenciales ERP encriptadas |

## Modelo de datos completo

```sql
-- Core
CREATE TABLE clients (
  id UUID PRIMARY KEY,
  business_name TEXT NOT NULL,
  cuit VARCHAR(13) UNIQUE NOT NULL,
  industry_code VARCHAR(10),         -- AFIP activity code
  erp_type VARCHAR(20) NOT NULL,
  erp_config JSONB,                  -- encrypted connection details
  onboarding_date DATE,
  status VARCHAR(20) DEFAULT 'active'
);

CREATE TABLE warehouses (
  id UUID PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  name TEXT,
  address TEXT,
  geo_lat DECIMAL(10,7),
  geo_lng DECIMAL(10,7),
  construction_type VARCHAR(20),     -- concrete, steel, wood, mixed
  fire_system VARCHAR(20),           -- sprinklers, extinguishers, hydrants, none
  security_system VARCHAR(20),       -- alarm, cameras, guard, none
  area_sqm DECIMAL(10,2),
  risk_score DECIMAL(4,2)
);

CREATE TABLE policies (
  id UUID PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  insurer_id UUID REFERENCES insurers(id),
  policy_number VARCHAR(50),
  current_insured_value DECIMAL(18,2),
  current_annual_premium DECIMAL(18,2),
  base_rate DECIMAL(8,6),
  deductible_pct DECIMAL(5,2),
  start_date DATE,
  end_date DATE,
  status VARCHAR(20) DEFAULT 'active',
  terms JSONB                        -- condiciones particulares
);

-- Time-series (TimescaleDB hypertable)
CREATE TABLE inventory_snapshots (
  time TIMESTAMPTZ NOT NULL,
  client_id UUID NOT NULL,
  warehouse_id UUID,
  total_value DECIMAL(18,2),
  item_count INTEGER,
  risk_composition JSONB,            -- % by risk class
  top_skus JSONB,                    -- top 10 SKUs by value
  source VARCHAR(20),                -- erp sync, manual, etc
  checksum VARCHAR(64),
  FOREIGN KEY (client_id) REFERENCES clients(id)
);
SELECT create_hypertable('inventory_snapshots', 'time');

CREATE TABLE daily_premiums (
  time TIMESTAMPTZ NOT NULL,
  policy_id UUID NOT NULL,
  insured_value DECIMAL(18,2),
  daily_premium DECIMAL(12,4),
  base_rate DECIMAL(8,6),
  dynamic_rate DECIMAL(8,6),
  risk_score DECIMAL(4,2),
  anomaly_adjustment DECIMAL(4,2) DEFAULT 1.0,
  FOREIGN KEY (policy_id) REFERENCES policies(id)
);
SELECT create_hypertable('daily_premiums', 'time');

-- Endorsements
CREATE TABLE endorsements (
  id UUID PRIMARY KEY,
  policy_id UUID REFERENCES policies(id),
  type VARCHAR(20),                  -- increase, decrease, composition_change
  previous_value DECIMAL(18,2),
  new_value DECIMAL(18,2),
  previous_premium DECIMAL(18,2),
  new_premium DECIMAL(18,2),
  auto_approved BOOLEAN DEFAULT false,
  approval_status VARCHAR(20),       -- pending, approved, rejected
  approved_by VARCHAR(50),           -- system, client, underwriter
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ML & Analytics
CREATE TABLE anomaly_events (
  id UUID PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  detected_at TIMESTAMPTZ DEFAULT NOW(),
  anomaly_type VARCHAR(30),
  severity VARCHAR(10),              -- low, medium, high, critical
  details JSONB,
  resolved BOOLEAN DEFAULT false,
  resolution_notes TEXT
);

CREATE TABLE risk_scores_history (
  time TIMESTAMPTZ NOT NULL,
  client_id UUID NOT NULL,
  overall_score DECIMAL(4,2),
  components JSONB,                  -- breakdown by factor
  FOREIGN KEY (client_id) REFERENCES clients(id)
);
SELECT create_hypertable('risk_scores_history', 'time');

-- Audit (inmutable)
CREATE TABLE audit_log (
  id BIGSERIAL PRIMARY KEY,
  timestamp TIMESTAMPTZ DEFAULT NOW(),
  actor VARCHAR(50),                 -- system, user email, api key
  action VARCHAR(50),
  entity_type VARCHAR(30),
  entity_id UUID,
  changes JSONB,
  ip_address INET
);
```

## Seguridad y compliance

### Proteccion de datos

- **Encriptacion en reposo**: AES-256 para credenciales ERP y datos sensibles
- **Encriptacion en transito**: TLS 1.3 en todas las comunicaciones
- **Tokenizacion**: datos de clientes tokenizados en la base de datos
- **Aislamiento**: cada cliente en su propio schema o con row-level security

### Autenticacion y autorizacion

- **OAuth2 + PKCE** para usuarios del dashboard
- **API keys rotables** con scopes para integraciones ERP
- **RBAC** (Role-Based Access Control):
  - `client.admin`: ve y configura su propio negocio
  - `client.viewer`: ve dashboards, no configura
  - `insurer.underwriter`: ve cartera, aprueba endosos
  - `insurer.admin`: configura tasas y parametros
  - `platform.admin`: acceso total

### Compliance regulatorio

- **Audit trail**: todo cambio con timestamp, actor, IP (requerido SSN)
- **Retencion 10 anios**: polizas y endosos (Ley 17.418)
- **Backup**: diario, retencion 7 anios, geo-redundante
- **Derecho al olvido**: datos personales eliminables, datos de poliza anonimizables
- **Habeas data**: cumplimiento Ley 25.326 de Proteccion de Datos Personales

## APIs publicas (plataforma como servicio)

En fase V2+, la plataforma expone APIs para que terceros construyan encima:

```
POST   /api/v1/quotes                  Cotizacion instantanea
GET    /api/v1/policies/{id}/coverage   Cobertura actual
GET    /api/v1/policies/{id}/premium    Prima actual y proyectada
POST   /api/v1/claims                  Iniciar reclamo
GET    /api/v1/analytics/benchmark     Benchmark de industria
GET    /api/v1/analytics/risk-score    Score de riesgo del cliente

Webhooks:
  stock.updated       Cambio de stock procesado
  premium.adjusted    Prima recalculada
  endorsement.issued  Endoso emitido
  anomaly.detected    Anomalia detectada
  claim.status        Cambio de estado de reclamo
```

Esto habilita:
- **Embedded insurance**: un ERP integra el seguro como feature nativo
- **Comparadores**: pueden cotizar en tiempo real
- **Brokers digitales**: pueden ofrecer el producto a sus clientes
- **Fintech**: pueden usar el risk score como variable crediticia

## Fases de desarrollo (actualizado)

### MVP (3-4 meses) — Probar la hipotesis

- Conector Tango (API REST) + Colppy (API REST) + CSV manual
- Motor de valuacion (Art. 87)
- Motor de pricing con tasa fija configurable por la aseguradora
- Dashboard web cliente (valor stock + prima + historial basico)
- Endoso por notificacion (email estructurado a la aseguradora)
- Ajuste: semanal
- **Meta**: 10 clientes piloto con 1 aseguradora

### V1 (6-9 meses) — Producto viable

- + Xubio, Bejerman, Google Sheets
- Event bus (Kafka)
- Risk scoring basico (zona + construccion + historial)
- Deteccion de anomalias por reglas (umbral + consistencia)
- Dashboard aseguradora
- Endoso semi-automatico (micro-endosos automaticos)
- Ajuste: diario
- PWA mobile
- **Meta**: 100 clientes, 2 aseguradoras

### V2 (12-18 meses) — Plataforma inteligente

- Pricing dinamico con tasa variable
- ML: Isolation Forest para deteccion de anomalias
- ML: prediccion de stock (LSTM)
- Benchmark de industria
- API publica
- Multi-aseguradora con gateway
- **Meta**: 500+ clientes, API publica, primeros clientes API

### V3 (18-36 meses) — Escala exponencial

- Score crediticio alternativo (venta a bancos/fintechs)
- IoT ready (sensores de temperatura, humedad, movimiento)
- Gemelo digital del deposito
- Expansion LATAM (Brasil, Chile, Colombia, Mexico)
- Embedded insurance via API en ERPs
- Data actuarial como producto (venta a reaseguradoras)
- **Meta**: infraestructura de referencia del mercado asegurador LATAM
