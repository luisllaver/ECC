# Puntos Clave del Contrato con la Aseguradora

> Este documento es una guia estrategica, NO asesoramiento legal.
> Todo contrato debe ser redactado y revisado por un abogado
> especialista en derecho de seguros antes de firmar.

## Objetivo del contrato

Establecer una relacion donde:
- La aseguradora aporta la capacidad (el capital que respalda el riesgo)
- Nosotros aportamos la tecnologia, los clientes y la operacion
- Ninguna parte puede "quedarse con todo" y dejar afuera a la otra

## 1. Propiedad Intelectual (clausula critica)

```
DEBE decir:
  - La plataforma, el codigo fuente, los algoritmos de pricing, los
    conectores ERP y los modelos de ML son propiedad exclusiva de
    NUESTRA empresa.
  - La aseguradora recibe una LICENCIA DE USO limitada, revocable, y
    solo mientras dure el contrato.
  - Los datos de comportamiento de stock y los modelos derivados son
    de NUESTRA propiedad.
  - Al terminar el contrato, NO se transfiere codigo fuente ni modelos.

NUNCA aceptar:
  - Clausula que obligue a entregar codigo fuente al terminar
  - Clausula que de a la aseguradora derecho a usar el algoritmo
  - "Work for hire" que haga que lo que desarrollamos sea de ellos
```

## 2. Titularidad del cliente

```
DEBE decir:
  - El cliente final (comercio) es cliente de NUESTRA plataforma.
  - El contrato de servicio tecnologico es entre el comercio y nosotros.
  - La poliza es entre el comercio y la aseguradora, pero intermediada
    y administrada por nosotros.
  - Los datos de contacto y comportamiento del cliente son nuestros.
  - Al terminar, podemos migrar el cliente a otra aseguradora
    (respetando las polizas vigentes hasta su vencimiento).
```

## 3. Estructura economica

```
- Comision sobre prima: 15-25% de la prima neta suscrita
- Fee de plataforma: cobrado directamente por nosotros al cliente
- Quien cobra al cliente: NOSOTROS (y giramos su parte a la aseguradora)
  -> esto refuerza la titularidad del cliente
- Profit commission: bonus si el loss ratio es bajo (alineacion de
  incentivos: si seleccionamos bien el riesgo, ganamos mas)
```

## 4. Facultades delegadas (si somos agente institorio)

```
- Suscripcion dentro de parametros definidos por la aseguradora
- Fijacion de prima dentro de bandas autorizadas
- Emision de endosos automaticos hasta cierto umbral
- Gestion de cobros
- Sobre cierto umbral: aprobacion del underwriter de la aseguradora
```

## 5. Duracion y terminacion

```
- Plazo inicial: 2-3 anios (da estabilidad para desarrollar)
- Renovacion automatica salvo aviso previo (90-180 dias)
- Terminacion con causa: solo por incumplimiento grave y comprobable
- Terminacion sin causa: preaviso largo (180 dias minimo)
- Run-off: administramos las polizas vigentes hasta su vencimiento
- Post-terminacion: la aseguradora NO puede replicar nuestro sistema
  por 24 meses (no-competencia)
```

## 6. Confidencialidad y no-competencia

```
- NDA mutuo sobre toda informacion tecnica y comercial
- La aseguradora no puede desarrollar un producto identico usando
  el conocimiento adquirido de nuestra plataforma
- La aseguradora no puede contactar directamente a nuestros clientes
  para ofrecerles el mismo producto por fuera de la plataforma
- Vigencia post-terminacion: 24 meses
```

## 7. Nivel de servicio (SLA) mutuo

```
De nosotros hacia la aseguradora:
  - Uptime de la plataforma
  - Precision de valuacion
  - Reportes actuariales periodicos

De la aseguradora hacia nosotros:
  - Tiempos de aprobacion de endosos mayores
  - Capacidad comprometida (limite de suma asegurada total)
  - Pago de comisiones en plazo
```

---

# El dilema de la EXCLUSIVIDAD

## El escenario

La primera aseguradora casi seguro va a pedir exclusividad. Es logico
desde su lado: invierte en desarrollar el producto con nosotros, lo
presenta a la SSN, pone su capital, y no quiere que despues llevemos
la misma tecnologia a su competencia.

Pero para nosotros, exclusividad total = capacity risk maximo =
dependencia de un solo actor = el escenario que queremos evitar.

## Por que NO conviene exclusividad total

- Nos deja expuestos al capacity risk (nos echan y perdemos todo)
- Limita nuestro crecimiento a lo que UNA aseguradora quiera/pueda
- Nos quita poder de negociacion en el futuro
- Si la aseguradora tiene problemas (rating, liquidez), caemos con ella

## La clave: exclusividad NO es binaria

No es "exclusividad si" o "exclusividad no". Hay muchas dimensiones para
negociar. Se le puede dar a la aseguradora una exclusividad ACOTADA que
la satisfaga sin encadenarnos.

### Opcion A: Exclusividad temporal (la mas comun y sana)

```
"Exclusividad por 12-18 meses desde el lanzamiento."

- Durante ese periodo, no trabajamos con otra aseguradora
- A cambio, pedimos contraprestaciones (ver mas abajo)
- Pasado el plazo, quedamos libres para sumar aseguradoras
- Le da tiempo a la aseguradora de recuperar su inversion
- Nos da un horizonte claro de cuando diversificar
```

Este es el estandar internacional: "la duracion debe ser razonablemente
necesaria para proteger los intereses de las partes sin restringir
indebidamente el futuro de ninguna."

### Opcion B: Exclusividad por segmento/ramo

```
"Exclusividad solo para seguro de stock/mercaderia, no para otros ramos."

- La aseguradora es exclusiva en NUESTRO producto especifico
- Nosotros quedamos libres para desarrollar otros productos
  (ej: seguro de transporte, seguro tecnico) con otras aseguradoras
- Acota la exclusividad a lo que realmente le importa a la aseguradora
```

### Opcion C: Exclusividad por territorio

```
"Exclusividad en una region/provincia, no en todo el pais."

- La aseguradora tiene exclusiva en su zona fuerte (ej: CABA + GBA)
- Nosotros podemos sumar otra aseguradora para el interior
- Util si la aseguradora es regional
```

### Opcion D: Right of First Refusal (derecho de preferencia)

```
"Sin exclusividad, pero la aseguradora tiene derecho de preferencia."

- Podemos hablar con otras aseguradoras
- Pero antes de cerrar con otra, se lo ofrecemos primero a la actual
- Si iguala la oferta, se queda ella
- Le da seguridad sin encadenarnos
```

Estandar internacional confirmado: "desde la perspectiva del MGA, esto
asegura que el carrier no trabaje con un competidor durante el periodo
relevante, o al menos que el MGA tenga derecho de preferencia en
productos relacionados."

### Opcion E: Exclusividad reciproca condicionada a volumen

```
"Exclusividad mientras la aseguradora cumpla ciertos compromisos."

- Le damos exclusividad SI se compromete a:
  - Una capacidad minima (limite de suma asegurada disponible)
  - Tiempos de aprobacion de endosos
  - Un volumen minimo de polizas suscriptas
  - Objetivos comerciales conjuntos
- Si NO cumple, la exclusividad se cae automaticamente y quedamos libres
```

Esta es la MEJOR opcion combinada: la exclusividad tiene "dientes".
Si la aseguradora no rema, perdemos la atadura pero no el negocio.

## Que pedir A CAMBIO de dar exclusividad (contraprestaciones)

Si damos exclusividad (aunque sea acotada), NUNCA gratis. Pedimos:

```
1. Capacidad garantizada
   - Un limite minimo de suma asegurada disponible
   - Que no nos frenen el crecimiento por "concentracion"

2. Compromiso comercial
   - Que la aseguradora aporte su red de PAS para vender
   - Objetivos de cantidad de polizas

3. Mejores condiciones economicas
   - Comision mas alta (ej: 25% en vez de 20%)
   - Profit commission mas agresiva

4. Inversion o adelanto
   - Que aporten capital para el desarrollo
   - O un fee fijo mensual minimo garantizado

5. Blindaje de la exclusividad
   - Que la exclusividad sea RECIPROCA: ellos tampoco desarrollan
     un producto competidor por su cuenta durante el periodo
   - Que si nos incumplen, la exclusividad cae

6. Clausula de salida clara
   - Fecha cierta de fin de exclusividad
   - O condiciones objetivas que la terminen
```

## Guion para la conversacion de exclusividad

Cuando la aseguradora pida exclusividad, la respuesta NO es "no". Es:

> "Entiendo perfectamente que quieran proteger su inversion, y tiene
> todo el sentido. Podemos darles exclusividad, pero estructurada de una
> forma que funcione para los dos: exclusividad por 12 meses en el ramo
> de stock, condicionada a que se comprometan con una capacidad minima y
> objetivos de volumen. Asi ustedes tienen la ventana para consolidar el
> producto, y nosotros tenemos la certeza de que el proyecto va a crecer.
> Pasado ese periodo, revisamos juntos como seguimos."

Esto muestra que:
- No somos ingenuos (no damos exclusividad eterna y gratis)
- Somos socios razonables (entendemos su necesidad)
- Pensamos en el largo plazo (queremos que funcione para ambos)

## Linea roja: lo que NUNCA aceptar

```
- Exclusividad perpetua o indefinida
- Exclusividad sin contraprestacion
- Exclusividad que incluya la propiedad de la tecnologia
- Exclusividad que impida migrar clientes al terminar
- Cualquier clausula que entregue codigo fuente o algoritmos
- Exclusividad sin condicion de volumen/capacidad minima
```

## Resumen ejecutivo del dilema

La exclusividad no es el enemigo. La exclusividad MAL ESTRUCTURADA si.

Formula recomendada para la primera aseguradora:

```
Exclusividad temporal (12-18 meses)
  + acotada al ramo de stock
  + condicionada a capacidad y volumen minimo
  + reciproca (ellos tampoco compiten)
  + con la IP y los clientes siempre nuestros
  + con fecha cierta de fin
  + a cambio de mejor comision y compromiso comercial
```

Esto le da a la aseguradora la seguridad que necesita para invertir, y
a nosotros nos deja un camino claro hacia el modelo multi-aseguradora
que es nuestra verdadera proteccion de largo plazo.
