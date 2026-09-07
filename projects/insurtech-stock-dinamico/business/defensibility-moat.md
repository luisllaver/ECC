# Estrategia de Defensibilidad (Moat)

## El riesgo central: "Capacity Risk"

El mayor miedo de toda insurtech MGA tiene nombre en la industria:
**capacity risk** (riesgo de capacidad).

> "Una sola llamada del directorio de la aseguradora puede terminar tu MGA
> de la noche a la manana: reduccion de capacidad, salida del ramo,
> reestructuracion de comisiones, o un downgrade de rating. Decisiones
> tomadas fuera de tu control que pueden frenar todo tu libro de polizas."

Este documento define como NO quedar expuestos a eso.

## Que puede pasar cuando una aseguradora termina el contrato

Segun la practica internacional de contratos MGA:

1. **Run-off**: el MGA puede seguir administrando renovaciones de polizas
   vigentes, confirmando cobertura, remitiendo primas.
2. **Transferencia de datos**: si el MGA no puede hacer run-off, debe
   entregar TODOS los datos del programa, incluyendo backups y registros
   de polizas.
3. **Peligro clave**: algunos contratos mal negociados obligan a entregar
   el CODIGO FUENTE y a licenciar el software propietario a la aseguradora.
   **Esto es exactamente lo que debemos evitar en nuestro contrato.**

## Las 6 capas de proteccion (defensa en profundidad)

Ninguna capa sola alcanza. La fortaleza viene de combinarlas todas.

### Capa 1: Propiedad Intelectual en el contrato (fuerza: media)

Clausulas obligatorias:
- Todo lo desarrollado por una parte le pertenece a esa parte
- Lo desarrollado conjuntamente es de propiedad conjunta, pero la
  aseguradora NO puede usarlo sin nuestra licencia
- NDA: la aseguradora no puede revelar la tecnologia a terceros
- Si terminan el contrato, NO transferimos codigo fuente
- Si terminan el contrato, NO pueden usar nuestro algoritmo de pricing
- El codigo, los modelos, los conectores ERP: siempre nuestros

Standard internacional: "anything developed by one party remains owned
by the developing party."

### Capa 2: Patente del metodo tecnico (fuerza: media-alta)

- Patentar en INPI el metodo tecnico especifico (no la idea)
- "Sistema y metodo para ajuste automatico de prima de seguro basado en
  integracion con sistema de gestion de inventario"
- Tarda 3-5 anios pero establece prioridad desde el dia 1
- Referencia: ya existen patentes similares en EEUU (US8271308, US8046243)
  que validan que es patentable

### Capa 3: Multi-aseguradora (fuerza: muy alta) — LA MAS IMPORTANTE

**NUNCA depender de una sola aseguradora.**

```
Escenario fragil:
  1 aseguradora -> te echan -> perdiste todo

Escenario robusto:
  3-4 aseguradoras -> 1 te echa -> mudas clientes a las otras
  -> la que te echo pierde los clientes
```

Practica confirmada de MGAs exitosas: "Con dos o mas carriers, el MGA
distribuye riesgo. Si uno reduce apetito, el MGA traslada nuevos negocios
a carriers alternativos manteniendo las polizas existentes."

Meta: llegar a 2 aseguradoras en los primeros 12 meses.

### Capa 4: El cliente es NUESTRO, no de la aseguradora (fuerza: muy alta)

```
Modelo fragil:
  Comercio -> Aseguradora (nosotros somos proveedor tech)
  La aseguradora tiene la relacion. Te echa y se queda el cliente.

Modelo robusto:
  Comercio -> NUESTRA PLATAFORMA -> Aseguradora
  Nosotros tenemos la relacion. El cliente usa nuestra app.
  Si la aseguradora se va, el cliente se queda con nosotros.
```

Como se logra en la practica:
- El cliente firma contrato de servicio con NUESTRA empresa
- El cliente usa NUESTRA app/dashboard, no la de la aseguradora
- La integracion ERP es NUESTRA; la aseguradora recibe datos procesados
- El certificado sale con NUESTRA marca (white-label de la aseguradora)
- El cobro lo hacemos NOSOTROS y giramos a la aseguradora su parte

Efecto: echarnos les cuesta perder toda la cartera que les trajimos.

### Capa 5: Los datos como moat imposible de copiar (fuerza: altisima)

Despues de 12 meses de operacion tenemos:
- Historial de valuacion diaria de cientos de comercios
- Patrones de rotacion de stock por rubro
- Estacionalidad real por segmento
- Correlacion entre patrones de stock y siniestralidad
- Modelos predictivos entrenados con datos reales
- Benchmark de industria que ninguna aseguradora tiene

**Una aseguradora puede copiar el codigo. No puede copiar los datos.**
El moat crece exponencialmente con cada cliente y cada dia de operacion.

### Capa 6: Integraciones ERP como barrera tecnica (fuerza: alta)

Cada conector (Tango, Colppy, Bejerman) representa:
- 2-3 meses de desarrollo
- Testing con datos reales
- Edge cases descubiertos en produccion
- Mantenimiento continuo ante cambios de API
- Posible partnership formal/exclusivo con el ERP

Si somos partner oficial de Tango (Axoft) y Colppy para el caso de uso
de seguros, un competidor tiene que desarrollar todo de cero y ademas
negociar con cada ERP.

## Lo que NO protege (no confiarse)

- "Ser el primero": no alcanza sin construir las barreras
- "La idea": no se patenta ni se protege por si sola
- "Un NDA solo": restringe informacion, no impide construir algo similar
- "La relacion personal": un cambio de gerente y el contacto desaparece

## La estructura que nos hace imbancables

```
                    NUESTRA PLATAFORMA
              (duena del cliente, del dato, de la tech,
               de las integraciones, y de la patente)
                         |
          +--------------+--------------+
          |              |              |
     Aseguradora 1  Aseguradora 2  Aseguradora 3

     Si una se va, los clientes se quedan y pasan a otra.

          +--------------+--------------+
          |              |              |
       Tango API     Colppy API    Bejerman API
       (conector propio, posible exclusividad)
```

## Roadmap de proteccion

```
Semana 1-2:
  - Consulta con abogado de propiedad intelectual
  - Iniciar solicitud de patente (metodo tecnico)
  - Registrar marca en INPI (clase 36 + 42)

Mes 1-3 (primera aseguradora):
  - Contrato: la IP es NUESTRA
  - El cliente firma con NUESTRA empresa
  - Exclusividad limitada (ver contract-key-points.md)
  - Clausula de no-competencia post-terminacion (24 meses)

Mes 6-12:
  - Sumar segunda aseguradora
  - Partnership con Tango/Colppy
  - Acumular datos

Mes 12+:
  - Tercera aseguradora
  - API publica (somos infraestructura, no vendor)
  - Productos derivados (score crediticio, data actuarial)
```
