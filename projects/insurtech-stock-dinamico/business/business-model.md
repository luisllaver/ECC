# Modelo de Negocio Detallado

## Propuesta de valor

### Para el comercio/industria (cliente final)

- **Nunca mas infraseguro**: si se quema el deposito, cobra el 100% del valor real
- **Nunca mas sobreseguro**: no paga prima por cobertura que no necesita
- **Automatico**: sin declaraciones juradas mensuales, sin formularios
- **Transparente**: ve en tiempo real cuanto tiene asegurado y cuanto paga
- **Justo**: paga proporcional a lo que realmente tiene en stock

### Para la aseguradora (partner)

- **Mejor pricing del riesgo**: datos reales en vez de declaraciones estimadas
- **Menor riesgo moral**: el cliente no tiene incentivo a subdeclarar
- **Nuevo canal de distribucion**: acceso a PyMEs via tecnologia
- **Retención de clientes**: la integracion ERP genera lock-in natural
- **Datos actuariales**: historico granular de valuaciones para modelar mejor

## Opciones de estructura legal

### Opcion A: Agente Institorio (MGA) — RECOMENDADA

```
NUESTRA EMPRESA (Agente Institorio)
  |
  |-- Registrada en RAI (SSN) + IGJ
  |-- Mandato de la aseguradora partner
  |-- Facultades: suscribir, fijar prima dentro de parametros, administrar
  |
  |-- INGRESOS:
  |     Comision sobre prima: 15-25%
  |     Fee tecnologico al cliente: $X/mes
  |
  |-- ASEGURADORA PARTNER
        Pone el capital (capacidad)
        Define los limites de suscripcion
        Respalda el riesgo
        Puede reasegurar con Munich Re u otros
```

**Ventajas**:
- Mayor margen (comision + fee)
- Control del cliente y la experiencia
- Marca propia
- Escalable: podemos sumar aseguradoras

**Desventajas**:
- Requiere registro SSN (2+ anios de experiencia en actividad principal)
- Mayor responsabilidad regulatoria
- Necesita acuerdo formal con aseguradora

### Opcion B: Proveedor tecnologico (SaaS)

```
NUESTRA EMPRESA (SaaS)
  |
  |-- Vende plataforma a la aseguradora
  |-- No suscribe polizas
  |-- No requiere registro SSN
  |
  |-- INGRESOS:
  |     Licencia SaaS a la aseguradora: $X/mes + % sobre prima
  |     Implementacion: fee unico
  |
  |-- ASEGURADORA
        Opera el producto bajo su marca
        Suscribe las polizas
        Gestiona la relacion con el cliente
```

**Ventajas**:
- Sin carga regulatoria
- Inicio mas rapido
- Puede vender a multiples aseguradoras

**Desventajas**:
- Menor margen
- No controla al cliente final
- Dependencia de la aseguradora para vender
- La aseguradora podria internalizar la solucion

### Opcion C: Hibrida (empezar SaaS, migrar a MGA)

Fase 1 (meses 1-12): operar como SaaS con 1 aseguradora. Validar producto.
Fase 2 (meses 12-24): registrarse como agente institorio. Escalar.

**Esta es probablemente la ruta mas pragmatica.**

## Segmentos de cliente

### Segmento 1: Comercio minorista con stock rotativo (PRIORITARIO)

- Ferreterias, bazares, librerias, electronica, indumentaria
- Stock que cambia semanalmente
- Usan Tango, Colppy o Excel
- Prima tipica: USD 500-2.000/anio
- Volumen: ~80.000 comercios en Argentina

### Segmento 2: Distribuidores y mayoristas

- Distribuidoras de alimentos, bebidas, limpieza, construccion
- Stock de alto volumen con rotacion variable
- Usan Tango, Bejerman o SAP B1
- Prima tipica: USD 2.000-15.000/anio
- Volumen: ~20.000 empresas

### Segmento 3: Industrias con stock de materia prima y producto terminado

- Fabricas que acumulan materia prima y producto terminado
- Valuacion compleja (costo de fabricacion)
- Usan SAP, Bejerman o sistemas propios
- Prima tipica: USD 5.000-50.000/anio
- Volumen: ~15.000 empresas

### Segmento 4: E-commerce con deposito propio

- Tiendas online con fulfillment propio
- Integracion con Tiendanube, Shopify, MercadoLibre
- Stock altamente variable (promos, temporadas)
- Prima tipica: USD 300-5.000/anio
- Volumen: ~30.000 tiendas activas

## Modelo de ingresos

### Como Agente Institorio (Opcion A)

| Concepto | Monto | Frecuencia |
|----------|-------|------------|
| Comision sobre prima | 20% de la prima neta | Por cobro de prima |
| Fee de plataforma | USD 15-50/mes por cliente | Mensual |
| Setup de integracion | USD 100-500 (unico) | Una vez |
| Servicios adicionales | Variable | Por demanda |

**Ejemplo unitario (comercio mediano)**:
- Prima anual del cliente: USD 2.000
- Comision anual: USD 400
- Fee plataforma: USD 25/mes x 12 = USD 300
- **Ingreso anual por cliente: USD 700**

### Proyeccion de ingresos (escenario conservador)

| Metrica | Anio 1 | Anio 2 | Anio 3 |
|---------|--------|--------|--------|
| Clientes activos | 50 | 300 | 1.200 |
| Prima promedio | USD 2.000 | USD 2.500 | USD 3.000 |
| Prima total | USD 100K | USD 750K | USD 3.6M |
| Comision (20%) | USD 20K | USD 150K | USD 720K |
| Fee plataforma | USD 15K | USD 90K | USD 360K |
| **Ingreso total** | **USD 35K** | **USD 240K** | **USD 1.08M** |

## Estructura de costos

### Costos fijos (mensuales, anio 1)

| Concepto | USD/mes |
|----------|---------|
| CTO / dev principal (socio) | USD 0 (equity) |
| 1 desarrollador | USD 1.500 |
| Infraestructura cloud | USD 200 |
| Legal / contable | USD 300 |
| Oficina / cowork | USD 200 |
| **Total** | **USD 2.200** |

### Costos variables (por cliente)

| Concepto | USD/cliente/mes |
|----------|-----------------|
| Soporte | USD 5 |
| Infraestructura incremental | USD 2 |
| Comision de venta (si hay PAS) | 5% prima |

## Metricas clave (KPIs)

- **CAC** (Costo de adquisicion de cliente): target < USD 200
- **LTV** (Lifetime value): target > USD 2.100 (3 anios x USD 700)
- **LTV/CAC**: target > 10x
- **Churn mensual**: target < 2% (la integracion ERP genera stickiness)
- **GWP** (Gross Written Premium): prima total suscrita
- **Loss Ratio**: siniestros pagados / primas cobradas (responsabilidad aseguradora)
- **Tiempo de integracion**: target < 48 horas por cliente

## Canales de adquisicion

### Canal 1: Camara Insurtech Argentina (LinkUp)
- Acceso a aseguradoras partner
- Visibilidad en el ecosistema
- Costo: bajo (aplicacion)

### Canal 2: Productores Asesores de Seguros (PAS)
- Red existente de intermediarios
- Ya tienen cartera de clientes con polizas de mercaderia
- Comision: 5% de la prima para el PAS
- Volumen potencial: alto

### Canal 3: Contadores y estudios contables
- Asesoran a PyMEs sobre seguros
- Conocen el problema de infra/sobreseguro de sus clientes
- Referral fee o acuerdo de colaboracion

### Canal 4: ERPs como canal de distribucion
- Partnership con Tango, Colppy, Xubio
- Plugin o integracion nativa dentro del ERP
- El ERP refiere clientes, cobra comision
- Win-win: el ERP agrega valor a su producto

### Canal 5: Camaras de comercio y asociaciones empresarias
- Acceso a miles de PyMEs agremiadas
- Posibilidad de poliza colectiva o convenio

## Go-to-market (primeros 12 meses)

### Mes 1-2: Validacion
- Entrevistas con 20+ comercios para validar dolor
- Definir MVP con 1 aseguradora (via LinkUp o contacto directo)
- Iniciar tramite de marca INPI

### Mes 3-5: Desarrollo MVP
- Conector Tango/Colppy + Excel
- Motor de valuacion y pricing basico
- Dashboard minimo
- Beta con 5-10 clientes de prueba

### Mes 6-8: Lanzamiento piloto
- 20-50 clientes reales
- Iterar producto segun feedback
- Medir metricas (churn, satisfaccion, precision de valuacion)

### Mes 9-12: Escalamiento inicial
- Sumar conectores ERP
- Activar canal PAS
- Llegar a 50+ clientes
- Evaluar registro como agente institorio

## Riesgos y mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigacion |
|--------|-------------|---------|------------|
| Aseguradora no quiere innovar | Media | Alto | Tener 2-3 aseguradoras en pipeline |
| SSN rechaza el producto | Baja | Alto | Consultar con SSN antes de desarrollar |
| Integracion ERP mas compleja de lo esperado | Alta | Medio | Empezar con Excel + 1 ERP simple |
| Clientes no confian en compartir datos de stock | Media | Alto | Encriptacion, NDA, certificaciones |
| Competidor copia el modelo | Baja (corto plazo) | Medio | Velocidad de ejecucion, acuerdos exclusivos |
| Fraude del cliente (reportar menos stock) | Media | Medio | Auditorias aleatorias, cruce con facturacion |

## Equipo necesario (anio 1)

| Rol | Dedicacion | Perfil |
|-----|-----------|--------|
| CEO / Negocio | Full-time | Conocimiento del mercado asegurador |
| CTO / Dev | Full-time | Backend, integraciones, APIs |
| Actuario consultor | Part-time | Matriculado, diseno de producto tecnico |
| Abogado seguros | Part-time | Especialista en derecho de seguros |
| Comercial | Part-time (mes 6+) | Experiencia en venta a PyMEs |
