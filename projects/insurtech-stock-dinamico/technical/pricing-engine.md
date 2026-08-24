# Pricing Engine — Especificacion tecnica

## El motor de pricing es el core IP de la plataforma

Este documento detalla la logica del motor de calculo de prima dinamica,
el componente mas valioso y diferenciador de la plataforma.

## Principios

1. **Justo**: el cliente paga por lo que tiene, no por lo que declaro hace meses
2. **Verificable**: cada prima calculada se puede auditar y explicar
3. **Regulable**: cumple con Ley 17.418 y limites de la SSN
4. **Adaptable**: la aseguradora puede configurar sus propios parametros

## Formula maestra

```
primaDiaria(t) = SA(t) * TD(t) * RS(c) * AA(c,t) * FC(p)
                 --------------------------   ----------
                      variable diaria         fijo por poliza

SA(t)   = Suma Asegurada en el dia t
          (calculada por el Valuation Engine)

TD(t)   = Tasa Dinamica en el dia t
          = tasaBase * ajusteEstacional(t) * ajusteClimatico(t)
            * ajusteConcentracion(t)

RS(c)   = Risk Score del cliente c
          (calculado por el Risk Scoring Engine, actualizado mensual)

AA(c,t) = Anomaly Adjustment
          (1.0 normal, >1.0 si hay alertas activas del ML)

FC(p)   = Factor Contractual de la poliza p
          = f(deducible, franquicia, sublimites, cobertura)
```

## Componente 1: Suma Asegurada SA(t)

Viene del Valuation Engine. Es el valor total asegurable del stock
del cliente en el dia t.

```
SA(t) = SUM(
  para cada item i en inventario:
    min(
      costoAdquisicion(i) * cantidad(i),   // Art. 87
      precioVenta(i) * cantidad(i)          // tope Art. 87
    )
    * factorDepreciacion(i)                 // por antiguedad
)
```

### Factor de depreciacion por antiguedad

```
diasSinMovimiento    factor
0 - 90               1.00    (stock fresco)
91 - 180              0.95    (stock lento)
181 - 365             0.85    (stock estancado)
> 365                 0.70    (stock viejo, valor de liquidacion)
```

## Componente 2: Tasa Dinamica TD(t)

La tasa base la fija la aseguradora. Los factores dinamicos la ajustan
dia a dia dentro de limites configurables.

### Tasa base (por ramo/rubro)

```
Rubro                         Tasa base anual
Ferreteria / construccion     0.25% - 0.40%
Electronica / tecnologia      0.30% - 0.50%
Alimentos no perecederos      0.15% - 0.25%
Alimentos perecederos         0.35% - 0.60%
Indumentaria / textil         0.20% - 0.35%
Farmacia                      0.20% - 0.30%
Quimicos / inflamables        0.50% - 1.00%
Distribucion general          0.20% - 0.40%
```

### Ajuste estacional

```
ajusteEstacional(t) =
  si mesActual in [noviembre, diciembre]:
    1.15   (temporada alta comercial, mayor acumulacion)
  si mesActual in [enero, febrero]:
    0.90   (verano, menor actividad comercial)
  si esVispera(diaDelPadre, navidad, diaDelNino):
    1.20   (pico de acumulacion de stock)
  else:
    1.00

Limitable por config: max +/- 20% sobre tasa base
```

### Ajuste climatico

```
ajusteClimatico(t) =
  si alertaMeteorologica(zona, t) == 'tormenta_severa':
    1.10   (mayor riesgo de inundacion/dano)
  si alertaMeteorologica(zona, t) == 'ola_de_calor':
    1.05   (riesgo incendio, riesgo perecederos)
  else:
    1.00

Fuente: API del Servicio Meteorologico Nacional (SMN)
Limitable por config: max +10%
```

### Ajuste por concentracion de inventario

```
ajusteConcentracion(t) =
  top1_pct = valor del SKU mas valioso / valor total
  
  si top1_pct > 0.50:
    1.15   (mas del 50% en un solo producto = riesgo alto)
  si top1_pct > 0.30:
    1.05
  else:
    1.00
```

### Banda de variacion

```
TD(t) esta limitada a:
  tasaBase * 0.70 <= TD(t) <= tasaBase * 1.30

Esto protege al cliente de saltos bruscos y cumple
con expectativas regulatorias de la SSN.
```

## Componente 3: Risk Score RS(c)

Score compuesto que refleja el perfil de riesgo del cliente.
Se recalcula mensualmente o cuando cambian las condiciones.

```
RS(c) = w1 * scoreUbicacion
      + w2 * scoreConstruccion
      + w3 * scoreSeguridad
      + w4 * scoreSiniestralidad
      + w5 * scoreComportamiento

Pesos default:
  w1 = 0.20  (ubicacion)
  w2 = 0.20  (construccion)
  w3 = 0.20  (medidas de seguridad)
  w4 = 0.25  (historial de siniestros)
  w5 = 0.15  (comportamiento de stock)

Rango final: 0.60 (excelente) a 2.50 (riesgo muy alto)
```

### Score de ubicacion

```
scoreUbicacion =
  zonaInundable?          +0.3
  zonaRoboAlto?           +0.2
  distanciaBomberos > 5km? +0.2
  zonaIndustrial?         +0.1 (mayor riesgo incendio vecinos)
  zonaCentrica?           -0.1 (mejor acceso emergencias)

Fuente: geolocalizacion del deposito + datos publicos
```

### Score de construccion

```
scoreConstruccion =
  hormigon:    0.8 (mas resistente)
  steel frame: 0.9
  mixto:       1.0
  chapa:       1.3
  madera:      1.5

Fuente: declaracion del cliente + verificacion
```

### Score de seguridad

```
scoreSeguridad =
  rociadores automaticos?   -0.3
  detectores humo?          -0.1
  extintores vigentes?      -0.1
  alarma monitoreada?       -0.1
  camaras?                  -0.05
  vigilancia 24h?           -0.15
  ninguna medida:           +0.4

Fuente: declaracion + inspeccion inicial
Incentivo: el cliente que mejora su seguridad baja su prima
```

### Score de siniestralidad

```
scoreSiniestralidad =
  0 siniestros en 5 anios:  0.7  (bonificacion)
  1 siniestro menor:        1.0
  1 siniestro mayor:        1.3
  2+ siniestros:            1.5 - 2.0
  fraude detectado:         3.0 (rechazo probable)

Fuente: historial con aseguradoras + base SSN
```

### Score de comportamiento (unico nuestro)

```
scoreComportamiento =
  rotacionSaludable?        -0.1
  datosConsistentes?        -0.1  (ERP matchea facturacion)
  actualizacionFrecuente?   -0.05
  anomaliasDetectadas?      +0.2 por cada anomalia no resuelta
  stockEstancado > 50%?     +0.1

Fuente: datos propios de la plataforma (esto es nuestro moat)
```

## Componente 4: Anomaly Adjustment AA(c,t)

Multiplicador que se activa cuando el detector de anomalias
levanta alertas no resueltas.

```
AA(c,t) =
  sin anomalias activas:                1.00
  anomalia de calidad de datos:         1.00 (no afecta prima, pero alerta)
  anomalia de subdeclaracion leve:      1.05
  anomalia de subdeclaracion fuerte:    1.15 + auditoria obligatoria
  patron pre-siniestro detectado:       SUSPENSION -> revision manual

Maximo: 1.20 (mas alla, se escala a revision humana)
```

## Ciclo de calculo

```
Hora     Proceso                      Output
------   -------------------------    --------------------------------
03:00    Batch de snapshots nocturnos  inventory_snapshots actualizado
03:30    Valuation Engine             SA(t) calculada por cliente
04:00    Risk Score (si corresponde)  RS(c) actualizado
04:15    Anomaly Detection            AA(c,t) actualizado
04:30    Pricing Engine               primaDiaria(t) por poliza
05:00    Endorsement check            Endosos generados si corresponde
06:00    Dashboard actualizado         Cliente ve su prima del dia
08:00    Notificaciones enviadas       Alertas de cambio significativo
```

## Facturacion y cobro

### Modelo de cobro al cliente final

```
Opcion A: Debito automatico mensual (recomendado)
  - Se cobra la suma de primas diarias del mes anterior
  - Factura detallada con desglose diario
  - CBU/tarjeta configurado en onboarding

Opcion B: Cobro semanal
  - Para clientes que prefieren menor monto por vez
  - 4-5 cobros por mes

Opcion C: Cobro quincenal
  - Compromiso entre frecuencia y practicidad
```

### Ejemplo de factura mensual

```
FACTURA - POLIZA #SP-2026-0042
Cliente: Ferreteria El Tornillo SRL
Periodo: 01/06/2026 - 30/06/2026
Aseguradora: Hipotecario Seguros S.A.

Detalle de cobertura diaria:
  Fecha      Stock asegurado   Tasa efectiva   Prima del dia
  01/06      $8,500,000        0.0308%         $71.67
  02/06      $8,520,000        0.0308%         $71.84
  ...
  15/06      $6,800,000        0.0308%         $57.32
  ...
  25/06      $12,100,000       0.0330%         $109.36  (*)
  ...
  30/06      $10,500,000       0.0308%         $88.50

  (*) Tasa ajustada por estacionalidad pre-Dia del Padre

  Subtotal primas:                              $2,436.90
  Fee de plataforma:                            $3,500.00
  IVA (21%):                                    $1,246.75
  --------------------------------------------------
  TOTAL:                                        $7,183.65

  Cobertura maxima del periodo: $12,100,000
  Cobertura minima del periodo: $6,200,000
  Ahorro vs poliza fija tradicional: $812.40 (estimado)
```

## Testing y validacion del pricing

### Tests automatizados

```
1. Test de consistencia: mismos inputs -> mismo output (determinista)
2. Test de limites: TD nunca fuera de banda +-30%
3. Test de Art. 87: valuacion nunca excede precio de venta
4. Test de regresion: cambios en el engine no alteran calculos historicos
5. Test de stress: 10.000 clientes simultaneos
6. Test de edge cases: stock = 0, stock negativo, 1 solo SKU, etc.
7. Test de auditoria: todo calculo tiene trail completo
```

### Validacion actuarial

- Cada cambio en la formula requiere aprobacion del actuario
- Backtesting con datos historicos antes de deploy
- A/B testing con grupo de control en produccion
- Reporte mensual de loss ratio al comite actuarial
