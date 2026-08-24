# Integraciones ERP/WMS — Especificaciones tecnicas

## Tango Gestion (Axoft)

**Mercado**: 60.000+ empresas en Argentina. El ERP mas usado en PyMEs.

### API disponible

- Tipo: REST API
- Repositorio: `github.com/TangoSoftware/ApiTiendas`
- Endpoint stock: `tiendas.axoft.com/api/Aperture/Stock`
- Autenticacion: API key / OAuth (segun version)
- Formato: JSON

### Datos accesibles via API

```
GET /api/Aperture/Stock
  Parametros:
    - codigoProducto: filtro por SKU
    - numeroSucursal: filtro por sucursal
    - codigoDeposito: filtro por deposito
  
  Retorna:
    - Articulo, descripcion
    - Stock actual por deposito
    - Stock comprometido (en ordenes)
    - Stock pendiente de ingreso
    - Costo unitario
```

### Estrategia de integracion

1. Cliente genera API key en Tango Cloud
2. Nuestra plataforma registra las credenciales (encriptadas)
3. Polling cada 2 horas via REST API
4. Snapshot completo nocturno a las 03:00 AM
5. Mapeo automatico: categorias Tango -> riskClass nuestro

### Limitaciones conocidas

- No todas las versiones de Tango tienen API REST habilitada
- Tango Desktop (on-premise) requiere Tango Sync o agente local
- Rate limits de la API a verificar con Axoft
- Posible necesidad de partnership formal con Axoft

## Colppy

**Mercado**: PyMEs cloud-native. Crecimiento acelerado.

### API disponible

- Tipo: REST API (HTTP POST con JSON)
- Documentacion: `apidocs.colppy.com`
- Portal dev: `dev.colppy.com`
- Endpoint produccion: `login.colppy.com/lib/frontera2/service.php`
- Staging: `staging.colppy.com/lib/frontera2/service.php`
- Autenticacion: usuario + clave API (sesion)

### Operaciones de inventario

```
Operacion                  Descripcion
listar_itemsinventario     Lista todos los items de inventario
alta_iteminventario        Crea un nuevo item
editar_iteminventario      Modifica un item existente
alta_ajusteinventario      Registra un ajuste de inventario
```

### Estrategia de integracion

1. Registrar app en dev.colppy.com
2. Cliente autoriza acceso a su empresa en Colppy
3. Sync via `listar_itemsinventario` cada 2-4 horas
4. Enriquecer con datos de facturacion para validacion cruzada
5. Colppy es cloud -> menor complejidad que on-premise

### Ventajas

- API bien documentada con Postman collection
- 100% cloud, no requiere agente local
- Incluye datos de facturacion (validacion cruzada)
- Target demografico alineado con nuestro ICP

## Xubio

**Mercado**: PyMEs cloud, fuerte en contadores.

### API disponible

- Tipo: REST API
- Documentacion: portal de desarrolladores
- Autenticacion: OAuth2

### Estrategia de integracion

- Similar a Colppy (cloud-native)
- Menor prioridad porque tiene menos market share en gestion de stock
- Fuerte en contabilidad, complementario

## Bejerman

**Mercado**: empresas medianas, mas complejo.

### API disponible

- Tipo: API propietaria + acceso a base de datos (SQL Server)
- Complejidad: mayor que Tango/Colppy

### Estrategia de integracion

- Opcion A: API propietaria si esta disponible
- Opcion B: Agente local que lee la base de datos SQL Server
- Opcion C: Export programado a CSV + upload automatico
- Prioridad media: mercado mas chico pero primas mas altas

## SAP Business One

**Mercado**: empresas medianas-grandes. Fase 2.

### API disponible

- Tipo: REST API (Service Layer) + OData
- Autenticacion: usuario + password + company DB
- Muy bien documentada

### Datos accesibles

```
GET /b1s/v1/Items              Maestro de articulos
GET /b1s/v1/Items('{code}')    Detalle de articulo
GET /b1s/v1/StockTransfers     Movimientos de stock
GET /b1s/v1/InventoryGenExits  Salidas de inventario
GET /b1s/v1/Warehouses         Depositos
```

### Estrategia

- Fase 2 (mes 9+)
- Clientes con SAP B1 tienen primas mas altas -> mayor revenue por cliente
- Integracion mas compleja pero datos mas ricos

## Excel / CSV / Google Sheets (fallback universal)

### Para micro-comercios sin ERP

1. **Template descargable**: Excel con columnas predefinidas
   ```
   SKU | Descripcion | Cantidad | Costo Unitario | Precio Venta | Deposito | Categoria
   ```

2. **Upload via dashboard**: arrastrar y soltar el archivo
3. **Validacion automatica**: verificar tipos, detectar errores
4. **Google Sheets con webhook**:
   - Cliente completa la planilla
   - Apps Script envia webhook a nuestra plataforma en cada cambio
   - Sync cuasi-real-time sin desarrollo del lado del cliente

### Limitaciones

- Dependencia del usuario para actualizar
- Sin validacion cruzada automatica
- Propenso a errores manuales
- Solo para MVP y micro-comercios

## Tiendanube / Shopify / MercadoLibre (e-commerce)

### Fase 2: segmento e-commerce

Estos marketplaces y plataformas tienen APIs maduras para inventario:

```
Tiendanube: GET /v1/{store_id}/products  (stock por variante)
Shopify:    GET /admin/api/2024-01/inventory_levels.json
MELI:       GET /users/{user_id}/items (stock por publicacion)
```

La ventaja del e-commerce es que los datos de venta son 100% trazables
(cada venta genera una transaccion en la plataforma), lo que hace
la validacion cruzada trivial.

## Arquitectura del conector universal

```javascript
// Patron: cada conector es un plugin que implementa la interfaz
// Se registra en un registry y se selecciona por erpType del cliente

const connectorRegistry = {
  'tango': TangoConnector,
  'colppy': ColppyConnector,
  'xubio': XubioConnector,
  'bejerman': BejermanConnector,
  'sap_b1': SapB1Connector,
  'excel': ExcelConnector,
  'google_sheets': GoogleSheetsConnector,
  'tiendanube': TiendanubeConnector,
  'shopify': ShopifyConnector,
  'mercadolibre': MeliConnector
}

function getConnector(client) {
  const ConnectorClass = connectorRegistry[client.erpType]
  return new ConnectorClass(client.erpConfig)
}

// Sync job (ejecutado por scheduler)
async function syncClient(clientId) {
  const client = await db.clients.findById(clientId)
  const connector = getConnector(client)

  await connector.authenticate(client.erpConfig.credentials)
  const snapshot = await connector.getFullSnapshot()

  // Normalizar al formato canonico
  const normalized = normalize(snapshot, client.erpType)

  // Publicar evento en Kafka
  await kafka.produce('stock.raw.' + clientId, normalized)

  // Actualizar ultimo sync
  await db.clients.update(clientId, { lastSync: new Date() })
}
```

## Prioridad de desarrollo

| Fase | Conectores | Justificacion |
|------|-----------|---------------|
| MVP | Tango + Colppy + CSV | Cubren 70%+ de PyMEs AR con ERP |
| V1 | + Xubio + Google Sheets | Amplian a PyMEs cloud y micro |
| V1.5 | + Bejerman | Empresas medianas, primas mas altas |
| V2 | + SAP B1 + Tiendanube | Empresas grandes + e-commerce |
| V2.5 | + Shopify + MercadoLibre | E-commerce completo |
