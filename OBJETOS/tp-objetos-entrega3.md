# TP Objetos — Entrega 3: niveles, criterio de derroche y eventos por etapas (para la corrección)

> Apunte de estudio de la **entrega 3** del TP "La fiesta de Isabella". Las entregas 1 y 2 están en [tp-objetos.md](tp-objetos.md). Por cada parte: **qué pide la consigna**, **cómo lo resolvimos** y sobre todo **por qué**, que es lo que se pregunta en la defensa. Los patrones salen de [clase26](clase26.md) (Strategy y Template Method) y [clase27](clase27.md) (Composite); herencia, clases abstractas y composición, de [clase24](clase24.md).

**Conceptos evaluados** (dice la consigna): diseño, Template Method, Strategy, Composite, refactoring.

---

## Índice

| Punto | Qué | Patrón | Quién |
|---|---|---|---|
| [0](#0-la-idea-de-la-entrega) | La idea de la entrega | — | — |
| [1](#punto-1--los-niveles-de-experiencia-leandro) | Niveles de experiencia | **Template Method** | Leandro |
| [2](#punto-2--la-experiencia-y-el-criterio-de-derroche-matías) | Experiencia y criterio de derroche | **Strategy** | Matías |
| [3](#punto-3--los-eventos-por-etapas-mi-punto-individual) | Eventos por etapas | **Composite** | **mío** |
| [Grupal](#punto-grupal--diagrama-readme-y-regresión) | Diagrama, README y regresión | — | todo el grupo |
| [Cierre](#cierre--conceptos-para-repasar-el-día-antes) | Conceptos para repasar | los tres | — |

Mi punto está explicado paso a paso; los de Leandro y Matías, más resumidos pero con lo que me pueden preguntar igual.

---
---

# 0. La idea de la entrega

La consigna lo dice al principio: *"Ninguno es un requerimiento nuevo aislado: cada uno pone a prueba una decisión de diseño de las entregas anteriores."*

| Cambio | Qué decisión anterior pone a prueba | Qué pasó |
|---|---|---|
| **Niveles** | ¿Los proveedores tenían una superclase donde poner lo común? | Se agregó el nivel **una sola vez**, en `Proveedor` (Template Method). |
| **Criterio de derroche** | En la entrega 2 `Fiestero` era una **subclase** de `Cliente`. | La herencia no permite cambiar de criterio: pasó a ser un **objeto aparte** (Strategy). |
| **Eventos por etapas** | ¿Los proveedores le preguntaban cosas a la fiesta o metían mano en sus sectores? | Como solo le mandaban mensajes, alcanzó con un objeto nuevo que entienda esos mensajes (Composite). **Los proveedores no cambiaron.** |

Es la idea de "refactoring" de la consigna: cambiar la estructura del código **sin cambiar lo que hace** (los tests viejos lo comprueban), para que el requerimiento nuevo entre fácil.

---
---

# Punto 1 — Los niveles de experiencia (Leandro)

**Consigna**: según la experiencia acumulada, un proveedor es **aprendiz** (hasta 500, no suma nada), **experimentado** (501 a 1000, +20%) o **maestro** (más de 1000, +30%). Hay que modelarlo **"sin repetir ese cálculo en cada rubro"**. Y además: *"El nivel de un proveedor depende sólo de su experiencia: nunca puede quedar desactualizado respecto de ella."*

## El código

```wollok
class Proveedor {
    var experiencia = 0

    method experiencia() = experiencia

    // Template method: el nivel se aplica una sola vez, para todos los rubros
    method cuantoCobra(unEvento) = self.costoServicio(unEvento) * self.nivel().porcentajeAumento()

    method costoServicio(unEvento)   // Primitiva: el valor del trabajo según el rubro
    method complejidadDelRol()       // Primitiva

    method sumarExperiencia(unEvento) {
        experiencia += unEvento.complejidadDelEvento() * self.complejidadDelRol()
    }

    // El nivel se determina siempre a partir de la experiencia actual
    method nivel() = [aprendiz, experimentado, maestro].find({ nivel => nivel.aplicaA(experiencia) })
}

object aprendiz {
    method porcentajeAumento() = 1
    method aplicaA(experiencia) = experiencia <= 500
}

object experimentado {
    method porcentajeAumento() = 1.2
    method aplicaA(experiencia) = experiencia.between(501, 1000)
}

object maestro {
    method porcentajeAumento() = 1.3
    method aplicaA(experiencia) = experiencia > 1000
}
```

## Template Method en `cuantoCobra`

Es exactamente la **variante 2** de la clase 26 (`Contenido` / `Serie` / `Pelicula`):

| Clase 26 | Nuestro TP |
|---|---|
| `Contenido` (superclase abstracta) | `Proveedor` |
| `esInteresante()` = el **template method**, el esqueleto | `cuantoCobra(unEvento)` |
| `presupuesto > 5000000` = la parte **común** | `* self.nivel().porcentajeAumento()` |
| `cumpleCriterioInteresante()` = la **primitiva** abstracta | `costoServicio(unEvento)` |

- **La parte común** (aplicar el nivel) está **una sola vez**, en la superclase.
- **La parte que varía** (cuánto vale el trabajo según el rubro) la completa cada subclase.
- **Ninguna subclase redefine `cuantoCobra`.** Por eso ninguna se puede "olvidar" de aplicar el nivel: es la ventaja de la variante 2 sobre la de `super` (tabla de la clase 26, 5.2).

**¿Por qué funciona?** Por el method lookup y `self` (clase 26: *"`self` siempre es el objeto receptor"*):

1. `marta.cuantoCobra(fiestaIsabella)` → `Organizador` no lo tiene → `ProveedorPorSector` tampoco → lo encuentra en `Proveedor`.
2. Adentro, `self.costoServicio(...)` → `self` es Marta → el lookup **vuelve a empezar en `Organizador`** → no está → lo encuentra en `ProveedorPorSector`.
3. Ahí, `self.adicionalProveedor(...)` → vuelve a empezar en `Organizador` → lo encuentra ahí.

Un solo objeto (Marta), tres clases para tener el comportamiento completo.

## Los niveles: objetos polimórficos, no `if`s

En vez de una cadena de `if (experiencia <= 500) ... else if ...`, cada nivel **sabe su regla** (`aplicaA`) y **su porcentaje** (`porcentajeAumento`). Es lo mismo que los **colores de los Meeseeks** en la clase 26: cada color sabe si le corresponde a un meeseeks, y se busca el primero que corresponda.

- **Son polimórficos**: los tres entienden `aplicaA(experiencia)` y `porcentajeAumento()`.
- **Son wko, no clases**: no tienen estado propio; hay uno solo de cada uno y lo comparten todos los proveedores. Es lo mismo que las complejidades del repo de la clase 27 (`complejidadMinima`, `complejidadMedia`, `complejidadMaxima` son wko).
- **Reifican un concepto del negocio** (clase 26, 4.5): "nivel" es una palabra del enunciado, y ahora es un objeto.

### ⚠️ `find` no está en los apuntes

`find(bloque)` devuelve el **primer** elemento de la colección para el que el bloque da `true`. Lo más parecido en los apuntes es `findOrDefault` (clase 26, Meeseeks), que hace lo mismo pero devuelve un valor por defecto si no encuentra ninguno. `find`, si no encuentra ninguno, **tira error**. Acá no pasa porque los tres rangos cubren todos los valores enteros desde 0.

Si te preguntan: *"recorre los niveles en orden y devuelve el primero que dice que le corresponde esa experiencia"*.

## "Nunca puede quedar desactualizado": el nivel se **calcula**, no se guarda

`nivel()` es un método que se calcula cada vez a partir de `experiencia`. **No hay un atributo `nivel`.**

Si guardáramos el nivel en una variable, cada vez que cambia la experiencia habría que acordarse de actualizarlo también: dos datos que dicen lo mismo y que hay que **mantener sincronizados**. Es el mismo problema que vimos en la entrega 2 con guardar un conjunto de proveedores aparte de los servicios (ver [tp-objetos.md, Paso 9](tp-objetos.md#paso-9-qué-proveedores-le-trabajaron--igualdad-e-identidad)). La clase 26 (4.6) lo dice como heurística: *"Diferenciar lo que es **estado** de lo que se puede **calcular**."*

## Los casos de niveles

| Experiencia | Nivel | Por qué es un buen caso |
|---|---|---|
| 0 | aprendiz | el valor inicial |
| 500 | aprendiz | **valor límite** (clase 23): el último de aprendiz |
| 501 | experimentado | el primero de experimentado |
| 1000 | experimentado | el último de experimentado |
| 1001 | maestro | el primero de maestro |

Los tests crean un `new Salon(precioPorSector = 100, experiencia = ...)` por cada caso. Se puede pasar `experiencia` al crear aunque sea un `var` sin `property`: dar un valor al crear no es un setter (ver [tp-objetos.md](tp-objetos.md), "Dar un valor al crear ≠ setter").

---
---

# Punto 2 — La experiencia y el criterio de derroche (Matías)

## 2.1 La experiencia

**Consigna**: `experiencia ganada = complejidad del evento * complejidad del rol`.

| Concepto | Dónde quedó | Por qué ahí |
|---|---|---|
| **Complejidad del evento**: 1 si dura hasta 5 horas, 3 si dura más; +2 si es complicado | `Evento.complejidadDelEvento()` | Es una pregunta sobre el evento: la sabe responder el evento. |
| **Complejidad del rol**: salón 5, organizador 3, seguridad y catering 2, mozos 1 | `complejidadDelRol()`, primitiva abstracta en `Proveedor`, cada rubro la define | Cada rubro sabe la suya. |
| **Sumar la experiencia** | `Proveedor.sumarExperiencia(unEvento)` | Una sola vez para todos. |

```wollok
method complejidadDelEvento() = (if (self.esLarga()) 3 else 1) + (if (self.esComplicada()) 2 else 0)
```

### El orden en `contratar` importa

```wollok
method contratar(proveedor) {
    self.validarPresupuesto(proveedor)
    const monto = proveedor.cuantoCobra(fiesta)      // 1. se calcula el precio con el nivel ACTUAL
    ahorros = ahorros - monto
    servicios.add(new Servicio(proveedor = proveedor, monto = monto))
    proveedor.sumarExperiencia(fiesta)                // 2. recién después suma experiencia
}
```

La consigna dice: *"La experiencia se suma al finalizar el trabajo: el precio del servicio se calcula con el nivel que el proveedor tenía al momento de contratarlo."* Por eso `sumarExperiencia` va **al final**. Si fuera antes, el contrato que hace pasar a Dodain de nivel ya se cobraría con el aumento.

### Traza: Damián contrata a Dodain tres veces

Dodain: 3 mozos por sector × $50 = $150 por sector. Fiesta de Damián: 4 sectores, 8 horas, complejidad 5. Rol de mozos: 1.

| Contrato | Experiencia antes | Nivel | Cobra | Experiencia después |
|---|---|---|---|---|
| 1° | 495 | aprendiz | 150 × 4 × 8 × 1 = **4800** | 495 + 5 × 1 = 500 |
| 2° | 500 | aprendiz (500 todavía es aprendiz) | **4800** | 505 |
| 3° | 505 | experimentado | 4800 × 1.2 = **5760** | 510 |

Total: 4800 + 4800 = **9600** después de dos; 9600 + 5760 = **15360** después de tres. ✅ Coincide con la consigna.

## 2.2 El criterio de derroche: Strategy

**Consigna**: *"el criterio de derroche es algo que cualquier cliente puede cambiar en cualquier momento"*. Milena pasa al criterio fiestero y Damián puede volver al clásico.

```wollok
class CriterioDerroche {
    method esDerrochon(unCliente)    // abstracto
}

class CriterioClasico inherits CriterioDerroche {
    const montoDerroche = 3000
    override method esDerrochon(unCliente) = !unCliente.servicios().filter({ servicio => servicio.monto() > montoDerroche }).isEmpty()
}

class CriterioFiestero inherits CriterioDerroche {
    override method esDerrochon(unCliente) = criterioClasico.esDerrochon(unCliente) && !unCliente.fiesta().esLarga()
}

const criterioClasico = new CriterioClasico()
const criterioFiestero = new CriterioFiestero()
```

Y en `Cliente`:

```wollok
var property criterioDeDerroche
method esDerrochon() = criterioDeDerroche.esDerrochon(self)
method cambiarCriterioDerroche(unCriterio) { criterioDeDerroche = unCriterio }
```

Es **igual** al ejemplo del alumno de la clase 26 (2.2 y 5.1):

| Clase 26 | Nuestro TP |
|---|---|
| `Alumno` | `Cliente` |
| `criterioParaEstudiar` | `criterioDeDerroche` |
| `Estudioso`, `Vago`, `HijoDelRigor` | `CriterioClasico`, `CriterioFiestero` |
| `estudiaPara(parcial, alumno)` | `esDerrochon(unCliente)` |
| el alumno **delega** pasándose a sí mismo | el cliente **delega** pasándose a sí mismo (`self`) |

### ¿Qué es Strategy?

La definición de la clase 26 (5.1):

> Un objeto **delega** una parte de su comportamiento en otro objeto **intercambiable** (la estrategia). Todas las estrategias son **polimórficas** entre sí.

Dicho más simple: **en vez de que el objeto sepa resolver algo él mismo, tiene guardado otro objeto que sabe hacerlo, y se lo pregunta.** Como ese otro objeto está en una variable, se puede **cambiar por otro** que lo resuelva distinto, y el objeto original sigue siendo el mismo.

**Una analogía: cómo vas a la facu.** Vos sos siempre la misma persona, pero un día vas en colectivo, otro en subte y otro en bici. "Ir a la facu" lo resuelve **el medio de transporte** que elegiste ese día. Algunas cosas cambian: cuánto tardás, cuánto pagás. Pero:

- **Cambiar de medio no te cambia a vos**: seguís siendo la misma persona, con tus cosas.
- **Podés cambiar cuando quieras**: hoy colectivo, mañana bici.
- **Si aparece un medio nuevo** (un monopatín), no tenés que cambiar vos: alcanza con que el monopatín sepa llevarte.
- **A vos no te importa cómo funciona cada medio**: solo le pedís que te lleve.

En el TP: el cliente es la persona, el criterio de derroche es el medio de transporte, y "¿soy derrochón?" es "ir a la facu".

### Las partes del patrón

| Parte | Qué es | Clase 26 | Nuestro TP |
|---|---|---|---|
| **El que delega** | el objeto que tiene guardada una estrategia y le pregunta | `Alumno` | `Cliente` |
| **La referencia** | el atributo donde guarda la estrategia; se puede cambiar | `criterioParaEstudiar` | `criterioDeDerroche` |
| **El mensaje común** | lo que entienden todas las estrategias | `estudiaPara(parcial, alumno)` | `esDerrochon(unCliente)` |
| **Las estrategias** | los objetos intercambiables, cada uno con su forma de resolver | `Estudioso`, `Vago`, `HijoDelRigor` | `CriterioClasico`, `CriterioFiestero` |

```
 ┌──────────┐  criterioDeDerroche  ┌────────────────────────┐
 │ Cliente  │─────────────────────▶│ esDerrochon(unCliente) │  (mensaje común)
 └──────────┘          1           └───────────▲────────────┘
                              ┌────────────────┴────────────────┐
                    ┌─────────┴────────┐              ┌─────────┴────────┐
                    │ CriterioClasico  │              │ CriterioFiestero │
                    └──────────────────┘              └──────────────────┘
```

### Cómo funciona: el cliente delega y se pasa a sí mismo

```wollok
method esDerrochon() = criterioDeDerroche.esDerrochon(self)
```

El cliente **no decide** si es derrochón: le pregunta a su criterio. Y le pasa `self` (a sí mismo) porque el criterio necesita **datos del cliente** para decidir: sus servicios y su fiesta.

**Traza con Milena** (ahorra $100.000 y contrata a Los Tilos por $6.000):

```
milena.esDerrochon()
└─ criterioDeDerroche es criterioClasico
   └─ criterioClasico.esDerrochon(milena)
      └─ ¿algún servicio de milena cuesta más de $3.000?  Los Tilos: $6.000 → sí
→ true
```

**Ahora Milena cambia de criterio:**

```wollok
milena.cambiarCriterioDerroche(criterioFiestero)
```

```
milena.esDerrochon()                              ← la misma milena, con los mismos servicios
└─ criterioDeDerroche ahora es criterioFiestero
   └─ criterioFiestero.esDerrochon(milena)
      ├─ criterioClasico.esDerrochon(milena) → true    (pagó $6.000)
      └─ !milena.fiesta().esLarga()          → false   (su fiesta dura 6 horas: es larga)
→ true && false = false
```

**Mismo objeto, misma pregunta, otra respuesta**, porque cambió la estrategia que tiene guardada. Milena no se creó de nuevo: conserva su identidad, sus ahorros y sus servicios.

### ¿Por qué es composición y no herencia?

Strategy se basa en **composición** (clase 26): el cliente **tiene** un criterio; no **es** un tipo de cliente.

| | Herencia (entrega 2: `Fiestero inherits Cliente`) | Composición (entrega 3: Strategy) |
|---|---|---|
| Relación | el cliente **es un** fiestero | el cliente **tiene un** criterio |
| Es entre | **clases**: se decide al escribir `new Fiestero(...)` | **objetos**: se decide guardando una referencia |
| ¿Se puede cambiar? | **No**: un objeto no puede cambiar de clase | **Sí**: se cambia la referencia |
| Para cambiar de criterio | crear otro cliente y perder el anterior | `cambiarCriterioDerroche(...)` |

La regla de la clase 27: **herencia para lo que no cambia, composición para lo que sí cambia**. El rubro de un proveedor no cambia → herencia. El criterio de un cliente cambia → composición.

### La otra alternativa: un `if` adentro del cliente

También se podría permitir el cambio sin Strategy, guardando un string y preguntando con `if`:

```wollok
class Cliente {
    var tipoDerroche = "clasico"      // o "fiestero"
    method esDerrochon() =
        if (tipoDerroche == "clasico") self.pagoAlgoCaro()
        else self.pagoAlgoCaro() && !fiesta.esLarga()
}
```

Se puede cambiar (`tipoDerroche = "fiestero"`), pero es justo la primera versión del alumno de la clase 26 (2.1), la que después se mejora con Strategy:

- **Strings mágicos**: `"clasico"` y `"fiestero"` no significan nada para Wollok. Un error de tipeo (`"fiestro"`) no avisa: cae en el `else`.
- **Agregar un criterio obliga a modificar `Cliente`** y sumar otro `if`.
- **`Cliente` pierde cohesión** (clase 26, 2): además de ahorrar y contratar, sabe todas las reglas de derroche.
- **No se pueden testear las reglas por separado** del cliente.

Con Strategy, cada criterio es un objeto que sabe su regla, y el cliente solo delega. La clase 26 dice que al alumno *"pasa de hacer 3 cosas a hacer 1: delegar"*.

### ¿Qué cuesta? (clase 27)

- **Más objetos**: el cliente y su criterio.
- **Construir un cliente es más complejo**: hay que pasarle un criterio (`new Cliente(..., criterioDeDerroche = criterioClasico)`).
- **El comportamiento queda repartido** en otro objeto, y a primera vista parece menos natural.
- **El cliente tiene que dar acceso a sus datos** (`servicios()`, `fiesta()`) para que el criterio pueda preguntarle.

Se paga a cambio de poder cambiar el criterio. Como la consigna lo pide, conviene.

### Strategy vs. Template Method, en corto

| | Strategy | Template Method |
|---|---|---|
| Mecanismo | **composición**: el objeto **tiene** otro y le delega | **herencia**: la subclase completa un hueco |
| ¿Cuántos objetos? | dos (el cliente y el criterio) | uno (la fiesta, el proveedor) |
| ¿Se puede cambiar después? | ✅ sí, se cambia la referencia | ❌ no, la clase es fija |
| En el TP | criterio de derroche | `Proveedor.cuantoCobra`, `Evento.esComplicada` |

### Si te preguntan "¿qué es Strategy?"

**1. La definición, en una oración:**

> "Es un patrón donde un objeto delega una parte de su comportamiento en otro objeto, la estrategia, que tiene guardado en un atributo. Todas las estrategias entienden el mismo mensaje, así que se pueden intercambiar."

**2. La clave:**

> "Como la estrategia está en una referencia, se puede cambiar en cualquier momento sin cambiar el objeto. Por eso se basa en composición y no en herencia: con herencia, la clase de un objeto no se puede cambiar."

**3. El ejemplo del TP:**

> "El cliente tiene un `criterioDeDerroche`. Cuando le preguntan si es derrochón, le pregunta a su criterio pasándose a sí mismo. `CriterioClasico` y `CriterioFiestero` entienden `esDerrochon(unCliente)`, y el cliente puede cambiar de uno a otro con `cambiarCriterioDerroche`, sin dejar de ser el mismo objeto."

**Si piden más:**

- **Por qué no herencia:** "En la entrega 2 el fiestero era una subclase, pero un objeto no puede cambiar de clase: para que Milena pasara a fiestera habría que crear otro cliente."
- **Qué se gana:** "Cambiar el criterio en cualquier momento, agregar criterios nuevos sin tocar al cliente y un cliente más cohesivo."
- **Qué cuesta:** "Más objetos, y construir un cliente pide pasarle un criterio."

**La versión más corta:**

> "En vez de resolverlo él, el objeto se lo pregunta a otro objeto que tiene guardado, y ese otro se puede cambiar."

### ¿Por qué ya no puede ser como en la entrega 2?

En la entrega 2, `Fiestero` era una **subclase** de `Cliente`. Con herencia, **la clase de un objeto se decide al crearlo y no cambia nunca**: `damian` fue creado con `new Fiestero(...)` y es fiestero para siempre. Para que Milena "pase a ser fiestera" habría que crear **otro objeto** y perder la identidad (sus servicios, las referencias que la apuntan…).

> En la entrega 2 lo dejamos anotado en [tp-objetos.md, "¿Cuándo pasaría a convenir composición?"](tp-objetos.md#cuándo-pasaría-a-convenir-composición): *"Si ser fiestero pudiera cambiar durante la vida del cliente"*. **Eso es exactamente lo que pide la entrega 3.**

La clase 27 lo dice con la tarea y su complejidad: con Strategy *"el cambio es dinámico: la tarea sigue siendo la misma (conserva su identidad) y sólo cambia su complejidad. Con herencia habría que crear otro objeto."*

### ¿Qué se ganó?

- **Cambiar en cualquier momento**: `milena.cambiarCriterioDerroche(criterioFiestero)`. Es el **mismo** cliente, con sus servicios.
- **Agregar criterios nuevos sin tocar `Cliente`**: alcanza con otra clase que entienda `esDerrochon(unCliente)`.
- **`Cliente` es más cohesivo** (clase 26, 2): ya no sabe las reglas del derroche, solo delega.
- **Se pueden testear los criterios por separado.**

### Lo que se puede preguntar

| Pregunta | Qué contestar |
|---|---|
| ¿Por qué `CriterioFiestero` usa a `criterioClasico`? | Porque la regla fiestera es "la clásica, salvo que el evento sea largo". Reutiliza la clásica en vez de repetir el `filter`. Antes eso se hacía con `super()`; ahora es delegación. |
| ¿Por qué hay una superclase `CriterioDerroche` si no tiene código? | Solo declara el mensaje abstracto: documenta qué tiene que entender un criterio. Para el polimorfismo no hace falta (ver [tp-objetos.md](tp-objetos.md), "Lo que no hace falta para que haya polimorfismo"). |
| ¿Los criterios podrían ser wko? | Sí: no tienen estado que cambie, y hay uno solo de cada uno. Se hicieron clases y se crea una instancia de cada una. |
| ¿Por qué hay `var property` y además `cambiarCriterioDerroche`? | `property` ya genera el setter `criterioDeDerroche(x)`; `cambiarCriterioDerroche` es un segundo método que hace lo mismo con un nombre más del negocio. Los tests usan este último. |
| ¿Hay más objetos que antes? | Sí: es el costo del Strategy (clase 27, "Consecuencias"): construir un cliente pide pasarle un criterio. |

### Los casos

| Caso | Criterio | Cuenta | ¿Derrochón? |
|---|---|---|---|
| Milena ahorra $100.000 y contrata a Los Tilos | clásico | Los Tilos cobra 500 × 2 × 6 = $6.000 > $3.000 | **Sí** |
| Ídem y cambia a fiestero | fiestero | su fiesta dura 6 horas: es larga | **No** |
| Damián contrata a Los Tilos | fiestero | fiesta de 8 horas: larga | **No** |
| Ídem y cambia a clásico | clásico | pagó 500 × 4 × 8 = $16.000 | **Sí** |
| Isabella cambia a fiestero, ahorra $5.000 y contrata a Los Tilos | fiestero | pagó $4.000 y su fiesta dura 4 horas: no es larga | **Sí** |

---
---

# Punto 3 — Los eventos por etapas (MI PUNTO INDIVIDUAL)

**Consigna**: Lucía organiza un casamiento con tres etapas: el civil, la fiesta y el after. *"Cada etapa es un evento como los que ya conocemos, con sus sectores y su duración. Y nada impide que una etapa sea, a su vez, un evento por etapas."* *"Los proveedores deben poder cobrar por un evento por etapas exactamente igual que por una fiesta simple. Un evento por etapas tiene tantos invitados, sectores y horas como la suma de sus etapas, y con eso se determina si es complicado y cuál es su complejidad, con las mismas reglas de siempre."*

Y lo que pide el punto individual: *"Modelar los eventos por etapas de manera que los proveedores cobren por ellos **sin cambiar**."*

## Paso 0: partir la consigna en requerimientos

| Frase de la consigna | Requerimiento |
|---|---|
| "Cada etapa es un evento como los que ya conocemos" | Una etapa puede ser una `Fiesta`. |
| "nada impide que una etapa sea, a su vez, un evento por etapas" | Una etapa también puede ser un evento por etapas → **anidamiento de n niveles**. |
| "tantos invitados, sectores y horas como la suma de sus etapas" | `cantidadInvitados`, `cantidadDeSectores` y `duracion` = suma de las etapas. |
| "con las mismas reglas de siempre" | `esComplicada`, `esLarga` y `complejidadDelEvento` **no se reescriben**: son las mismas de la fiesta. |
| "los proveedores cobren por ellos sin cambiar" | Un evento por etapas tiene que entender **los mismos mensajes** que una fiesta. |

## Paso 1: ¿qué le piden los proveedores (y el cliente) a un evento?

Antes de diseñar nada, la pregunta de la clase 27 (5.4): *¿a quién le mando qué mensaje?* Busqué en `example.wlk` todo lo que se le manda a la fiesta:

| Mensaje | Quién lo manda |
|---|---|
| `cantidadDeSectores()` | `ProveedorPorSector.costoServicio` |
| `duracion()` | `Salon` y `EquipoDeMozos` (`adicionalProveedor`) |
| `esComplicada()` | `Organizador` (`adicionalProveedor`) |
| `esLarga()` | `Seguridad` (`adicionalProveedor`) y `CriterioFiestero` |
| `cantidadInvitados()` | `Catering.costoServicio` |
| `complejidadDelEvento()` | `Proveedor.sumarExperiencia` |

Esa es **la interfaz** de un evento. Es la heurística *"Interfaz > implementación"* de la clase 26 (4.2): primero pensar **qué se le pide** a un objeto, después cómo lo hace.

**Ningún proveedor le pide la lista de sectores** (`sectores()`) para contarla él: le pregunta `cantidadDeSectores()` y la fiesta resuelve. Por eso un evento que **no tiene sectores propios** (los tiene adentro de sus etapas) puede entrar sin que los proveedores se enteren. Si algún proveedor hiciera `unaFiesta.sectores().size()`, sí habría que cambiarlo.

## Paso 2: la estructura es un Composite

### 2.1 El problema que resuelve

Hasta la entrega 2 había un solo tipo de evento: la **fiesta**, con sectores y una duración. En la entrega 3 aparecen eventos **hechos de otros eventos**:

- El **casamiento** está hecho de tres eventos: el civil, la fiesta y el after.
- El **fin de semana** está hecho del casamiento y el brunch. O sea, de un evento que **a su vez** está hecho de otros eventos.

Eso forma un **árbol**:

```
finDeSemana
├── casamiento
│   ├── civil          (fiesta)
│   ├── fiestaDamian   (fiesta)
│   └── after          (fiesta)
└── brunch             (fiesta)
```

Y la consigna pide que **los proveedores cobren igual** por cualquier punto de ese árbol: una fiesta suelta, el casamiento o el fin de semana. El Composite es la forma de armar ese árbol para que eso sea posible.

### 2.2 La analogía de la clase 27: carpetas y archivos

En el explorador de archivos:

- Un **archivo** es una pieza simple: no contiene nada adentro.
- Una **carpeta** contiene cosas, y esas cosas pueden ser **archivos u otras carpetas**.

Si le preguntás a algo **"¿cuánto pesás?"**:

- Un archivo responde **solo**: "peso 2 MB".
- Una carpeta no tiene un peso propio: le pregunta a cada cosa que tiene adentro y **suma**. Si adentro hay otra carpeta, esa hace lo mismo con lo suyo.

Para quien usa la compu **no hay diferencia**: puede preguntarle el peso, copiar o borrar tanto un archivo como una carpeta, sin importarle cuál es.

**Eso es el Composite**: una estructura de **piezas simples** (archivos) y **piezas compuestas** (carpetas) que **responden a los mismos mensajes**, donde las compuestas pueden contener **cualquier tipo de pieza**, incluso otras compuestas.

### 2.3 Las tres partes del patrón

| Parte | Qué es | Carpetas | Nuestro TP |
|---|---|---|---|
| **Componente** | lo común a todas las piezas: qué mensajes entienden | "elemento del explorador" | `Evento` |
| **Hoja** | pieza simple: responde sola, no contiene otras | archivo | `Fiesta` |
| **Rama** (compuesto) | pieza que contiene otras piezas y responde preguntándoles a ellas | carpeta | `EventoPorEtapas` |

"Un archivo nunca se convierte en directorio" (clase 27): una fiesta nunca pasa a ser un evento por etapas. Por eso hoja y rama pueden ser **subclases** de `Evento`: lo que no cambia va con herencia.

### 2.4 Las dos claves

**Clave 1: la hoja y la rama entienden los mismos mensajes.**

`Fiesta` y `EventoPorEtapas` entienden `duracion()`, `cantidadDeSectores()`, `cantidadInvitados()`, `esLarga()`, `esComplicada()` y `complejidadDelEvento()`. Son **polimórficas**. Por eso un salón puede hacer `unEvento.duracion()` sin saber si `unEvento` es una fiesta o un casamiento: **quien usa el árbol no distingue hojas de ramas**.

**Clave 2: la rama contiene *Eventos*, no *Fiestas*.**

```wollok
class EventoPorEtapas inherits Evento {
    const etapas = []      // ← cada etapa es un Evento: puede ser Fiesta o EventoPorEtapas
    ...
}
```

Las etapas no son "fiestas": son **cualquier cosa que sea un evento**. Como un `EventoPorEtapas` **también es un evento**, puede ser la etapa de otro `EventoPorEtapas`. Eso permite meter el casamiento adentro del fin de semana, y así todos los niveles que se quieran.

> **Detalle de Wollok**: una lista no tiene tipo, así que `etapas` podría guardar cualquier objeto. Lo que hace que el diseño sea un Composite no es lo que dice la lista, sino **qué mensajes le manda la rama a sus etapas**: tiene que usar solo mensajes que entienda **cualquier** evento, sea hoja o rama (ver Paso 10, alternativa 1).

### 2.5 Las dos flechas entre `EventoPorEtapas` y `Evento`

En el diagrama hay **dos flechas distintas** entre esas dos clases, y dicen cosas distintas:

```
            ┌──────────┐
            │  Evento  │◀─────────────┐
            └────▲─────┘              │  ② asociación: "tiene"
                 │ ① herencia: "es un" │     (atributo etapas, *)
         ┌───────┴───────┐            │
         │EventoPorEtapas│────────────┘
         └───────────────┘
```

| | ① Herencia | ② Asociación |
|---|---|---|
| Cómo se dibuja | flecha con **triángulo vacío** | flecha **abierta**, con el **nombre del atributo** y la cantidad (`*`) |
| Qué dice | `EventoPorEtapas` **es un** `Evento` | un `EventoPorEtapas` **tiene** eventos adentro |
| En el código | `class EventoPorEtapas inherits Evento` | `const etapas = []` |
| Entre quiénes es | entre **clases** (el código) | entre **objetos** (las instancias) |
| Qué le da | **los métodos** de `Evento`: `esLarga()`, `esComplicada()`, `complejidadDelEvento()` | **los objetos** a los que les va a preguntar: sus etapas |

**① Herencia, qué *es***: le dice a Wollok que un evento por etapas **es** un evento, así que entiende todo lo que entiende un evento. Por eso `casamiento.esLarga()` funciona aunque `EventoPorEtapas` no tenga ese método escrito. Es la misma flecha que va de `Fiesta` a `Evento` o de `Salon` a `ProveedorPorSector`: una relación **entre clases**, que se decide al escribir el código y no cambia.

**② Asociación, qué *tiene***: dice que cada objeto `EventoPorEtapas` **guarda una lista de objetos**, y que esos objetos son `Evento`s. No le da ningún método: le da **a quién preguntarle**. Es la misma clase de flecha que `Cliente → Servicio` (atributo `servicios`) o `Cliente → CriterioDerroche`: un cliente no *es* un servicio, *tiene* servicios. Igual, un casamiento no *es* sus etapas, *tiene* etapas.

Se ve mejor mirando los **objetos** en vez de las clases:

```
casamiento (un EventoPorEtapas)
  etapas ──▶ [ civil , fiestaDamian , after ]      ← ② cada uno es un objeto Fiesta

finDeSemana (un EventoPorEtapas)
  etapas ──▶ [ casamiento , brunch ]               ← ② casamiento es un EventoPorEtapas
```

La herencia no aparece en este dibujo, porque es de las clases y no de los objetos. Lo que aparece son las referencias que guarda cada evento.

**¿Por qué la asociación apunta a `Evento` y no a `Fiesta`?** Porque la flecha indica **qué tipo de cosa** puede haber en `etapas`. Si apuntara a `Fiesta`, las etapas solo podrían ser fiestas y el fin de semana no podría tener al casamiento adentro. Apuntando a `Evento`, una etapa puede ser **cualquier evento**.

Y ahí se juntan las dos flechas: como `EventoPorEtapas` **es un** `Evento` (①), puede ser una de las etapas que **tiene** otro `EventoPorEtapas` (②). Con una sola no alcanza:

| Solo herencia | Solo asociación (a `Fiesta`) | Las dos |
|---|---|---|
| Entendería los mensajes, pero no tendría etapas a quién preguntarles. | Tendría etapas, pero solo fiestas: no se podría anidar. | Entiende los mensajes **y** puede contener otros eventos por etapas → **Composite** |

### 2.6 Composite no es lo mismo que composición

Se parecen en el nombre, pero (clase 27):

- **Composición**: un objeto **conoce a otro** y le delega algo. El cliente conoce a su criterio de derroche (Strategy).
- **Composite**: un **árbol** de piezas simples y compuestas que entienden los mismos mensajes, donde las compuestas contienen piezas del mismo tipo general.

En el TP están las dos: `Cliente → criterioDeDerroche` es composición; `EventoPorEtapas → etapas` es Composite.

### En la defensa

> "Usé Composite: `Fiesta` es la hoja, `EventoPorEtapas` es la rama y `Evento` es lo común. Las dos entienden los mismos mensajes, así que los proveedores no distinguen una de otra. Y como las etapas son `Evento`s, una etapa puede ser otra rama: por eso el fin de semana puede contener al casamiento. La rama responde preguntándole lo mismo a cada etapa y sumando."

> "En el diagrama hay dos flechas entre `EventoPorEtapas` y `Evento`. La de herencia dice que un evento por etapas **es** un evento, y por eso los proveedores lo pueden usar igual que una fiesta. La otra es el atributo `etapas`: dice que **tiene** eventos adentro. Como esos eventos pueden ser a su vez eventos por etapas, se pueden anidar."

## Paso 3: el código

```wollok
class Evento {
    method cantidadInvitados()
    method cantidadDeSectores()
    method duracion()

    method esComplicada() = self.cantidadDeSectores() > 3
    method esLarga() = self.duracion() > 5
    method complejidadDelEvento() = (if (self.esLarga()) 3 else 1) + (if (self.esComplicada()) 2 else 0)
}

class Fiesta inherits Evento {
    const property sectores = []
    const duracion

    override method duracion() = duracion
    override method cantidadDeSectores() = sectores.size()
    override method cantidadInvitados() = sectores.sum({ sector => sector.invitadosSector() })
}

class EventoPorEtapas inherits Evento {
    const etapas = []

    override method duracion() = etapas.sum({ etapa => etapa.duracion() })
    override method cantidadDeSectores() = etapas.sum({ etapa => etapa.cantidadDeSectores() })
    override method cantidadInvitados() = etapas.sum({ etapa => etapa.cantidadInvitados() })
}
```

## Paso 4: ¿por qué una superclase `Evento`? (Template Method adentro del Composite)

"Con las mismas reglas de siempre" quiere decir que `esComplicada`, `esLarga` y `complejidadDelEvento` son **iguales** para una fiesta y para un evento por etapas. Si estuvieran en las dos clases, sería una **idea repetida** (clase 26, 4.1).

Entonces:

- **`Evento` define las reglas una sola vez**, a partir de tres métodos **abstractos**: `cantidadDeSectores()`, `duracion()` y `cantidadInvitados()`.
- **Cada subclase completa solo esos tres**: la fiesta con sus sectores y su duración; el evento por etapas sumando sus etapas.

Eso es un **Template Method**. Abajo se explica qué es, qué son el esqueleto y la primitiva, y por qué `esComplicada()` es uno.

### 4.1 ¿Qué es un Template Method?

La definición de la clase 26 (5.2):

> El **template method** define el **esqueleto de un algoritmo general** en una superclase, y le da a cada subclase la posibilidad de **redefinir una parte** de ese algoritmo.

Dicho más simple: **la superclase escribe los pasos de una cuenta, pero deja un paso sin resolver**. Ese paso lo resuelve cada subclase a su manera. La cuenta se escribe una sola vez y lo único que cambia es el paso que falta.

**Una analogía: una receta con un hueco.** "Torta: mezclar harina y huevos, **agregar el sabor**, hornear 40 minutos." La receta es la misma para todas las tortas; lo único que cambia es "agregar el sabor": la de chocolate le pone cacao y la de limón, ralladura. Nadie reescribe la receta entera para cada torta: solo dice cuál es su sabor.

**El ejemplo de la clase 26**: un contenido es interesante si tiene presupuesto de más de 5 millones **y** cumple una condición propia. La serie la cumple si tiene menos de 5 temporadas; la película, si se estrenó este año.

```wollok
class Contenido {
  const presupuesto
  method esInteresante() = presupuesto > 5000000 && self.cumpleCriterioInteresante()   // el esqueleto
  method cumpleCriterioInteresante()                                                     // la primitiva (el hueco)
}

class Serie inherits Contenido {
  const temporadas
  override method cumpleCriterioInteresante() = temporadas < 5                           // completa el hueco
}
```

### 4.2 El esqueleto y la primitiva

Son las **dos partes** de un Template Method:

| | **Esqueleto** (el template method) | **Primitiva** (el hueco) |
|---|---|---|
| Qué es | el método con **la cuenta completa**, la parte que es igual para todos | el **paso que falta**, el que cambia según la subclase |
| Dónde está escrito | **una sola vez**, en la superclase | declarado en la superclase **sin cuerpo** (método abstracto) e implementado en **cada** subclase |
| ¿Las subclases lo redefinen? | **No**: lo heredan tal cual | **Sí**: cada una tiene que implementarlo |
| En la receta | "mezclar, agregar el sabor, hornear 40 minutos" | "agregar el sabor" |
| En la clase 26 | `esInteresante()` | `cumpleCriterioInteresante()` |

El nombre "esqueleto" es porque es la **estructura** de la cuenta, a la que le falta una parte para estar completa. El nombre "primitiva" es porque es la **pieza básica** que el esqueleto necesita y que no sabe calcular él mismo. En nuestro código las marcamos con el comentario `//Primitiva`.

**¿Por qué se completa en la subclase?** Por `self`. Cuando el esqueleto hace `self.primitiva()`, `self` es el objeto que recibió el mensaje (una serie, una fiesta…), y el method lookup **empieza a buscar en su clase**. Por eso, aunque el esqueleto esté escrito en la superclase, encuentra la primitiva de la subclase. La clase 26: *"`self` siempre es el objeto receptor: aunque el código esté en la superclase, `self.algo()` busca primero en la subclase."*

### 4.3 ¿Por qué `esComplicada()` es un Template Method?

Porque tiene exactamente esas dos partes:

```wollok
class Evento {
    method cantidadDeSectores()                                  // PRIMITIVA: sin cuerpo

    method esComplicada() = self.cantidadDeSectores() > 3        // ESQUELETO: la regla, una sola vez
}

class Fiesta inherits Evento {
    override method cantidadDeSectores() = sectores.size()       // completa el hueco: cuenta sus sectores
}

class EventoPorEtapas inherits Evento {
    override method cantidadDeSectores() = etapas.sum({ etapa => etapa.cantidadDeSectores() })   // completa el hueco: suma sus etapas
}
```

- **El esqueleto** es la regla "es complicado si tiene más de 3 sectores". Es **igual** para una fiesta y para un evento por etapas ("las mismas reglas de siempre"), así que está una sola vez, en `Evento`.
- **La primitiva** es "cuántos sectores tengo". Es lo que **cambia**: la fiesta los cuenta y el evento por etapas los suma.
- **Ninguna subclase redefine `esComplicada()`**: solo completan `cantidadDeSectores()`.

Con `esLarga()` pasa lo mismo, con otra primitiva:

| Esqueleto (en `Evento`) | Primitiva que usa | `Fiesta` la completa con… | `EventoPorEtapas` la completa con… |
|---|---|---|---|
| `esComplicada() = self.cantidadDeSectores() > 3` | `cantidadDeSectores()` | `sectores.size()` | la suma de las etapas |
| `esLarga() = self.duracion() > 5` | `duracion()` | su atributo `duracion` | la suma de las etapas |
| `complejidadDelEvento()` | usa `esLarga()` y `esComplicada()`, que a su vez usan las primitivas | — | — |

### 4.4 ¿Por qué dice "**otro**" Template Method?

Porque en el TP hay dos lugares con el mismo patrón:

| | En los proveedores (punto 1) | En los eventos (mi punto) |
|---|---|---|
| Superclase | `Proveedor` | `Evento` |
| Esqueleto | `cuantoCobra(unEvento) = self.costoServicio(unEvento) * self.nivel().porcentajeAumento()` | `esComplicada() = self.cantidadDeSectores() > 3` |
| Lo que es igual para todos | aplicar el nivel | la regla de "más de 3 sectores" |
| Primitiva | `costoServicio(unEvento)` | `cantidadDeSectores()` |
| Lo que cambia | cuánto vale el trabajo de cada rubro | cómo se obtienen los sectores |
| Subclases | `ProveedorPorSector`, `Catering` | `Fiesta`, `EventoPorEtapas` |

La clase 27 dice que el repo de manejo de proyectos tiene Template Method y Composite **juntos** (`Tarea.costo()` + `TareaSimple`/`TareaCompuesta`): acá pasa lo mismo. El Composite arma el árbol de eventos, y el Template Method hace que las reglas estén una sola vez en la superclase.

### 4.5 ¿Qué ganamos?

- **La regla está una sola vez.** Si mañana "complicado" pasa a ser "más de 4 sectores", se cambia en un solo lugar y vale para fiestas y eventos por etapas.
- **Las subclases no se pueden olvidar de la regla**: no la escriben, la heredan. Solo tienen que decir cuántos sectores tienen.
- **Un tipo de evento nuevo** solo tiene que completar las tres primitivas, y ya entiende `esComplicada()`, `esLarga()` y `complejidadDelEvento()`.

### Si te preguntan "¿qué es Template Method?"

**1. La definición, en una oración:**

> "Es un patrón donde la superclase define el esqueleto de un algoritmo, la parte que es igual para todos, y deja uno o más pasos como métodos abstractos para que cada subclase los complete con lo suyo."

**2. Las dos partes, con nombre:**

> "El esqueleto es el template method: está escrito una sola vez en la superclase y las subclases no lo redefinen. Los pasos que faltan son las primitivas: la superclase los declara sin cuerpo y cada subclase los implementa."

**3. El ejemplo de mi punto:**

> "En el TP, `esComplicada()` en `Evento` es el esqueleto: la regla de más de 3 sectores, igual para todos los eventos. `cantidadDeSectores()` es la primitiva: la fiesta cuenta sus sectores y el evento por etapas suma los de sus etapas. Funciona porque `self` es el objeto que recibió el mensaje, así que el lookup encuentra la primitiva en su clase."

**Si piden más:**

- **Qué se gana:** "La regla no se repite, y una subclase no se puede olvidar de aplicarla, porque no la escribe: la hereda."
- **El otro lugar del TP donde está:** "En `Proveedor`, `cuantoCobra` es el esqueleto que aplica el nivel, y `costoServicio` es la primitiva que completa cada rubro."

**La versión más corta:**

> "La superclase escribe la cuenta una vez y deja un hueco; cada subclase completa solo el hueco."

### En la defensa

> "En `Evento` usé Template Method: `esComplicada()` es el esqueleto, la regla de más de 3 sectores, que es igual para todos los eventos y está escrita una sola vez. `cantidadDeSectores()` es la primitiva: un método abstracto que cada subclase completa. La fiesta cuenta sus sectores y el evento por etapas suma los de sus etapas. Funciona porque `self` es el evento que recibió el mensaje, así que el lookup encuentra la primitiva de su clase."

**`Evento` es abstracta**: nunca se hace `new Evento()`. Según la clase 24 (5.1), en Wollok no hace falta declararlo: tiene métodos abstractos (sin cuerpo) y no se instancia.

### Method lookup: `casamiento.esComplicada()`

1. `casamiento` es un `EventoPorEtapas` → no tiene `esComplicada` → lo encuentra en `Evento`.
2. Adentro, `self.cantidadDeSectores()` → `self` es el casamiento → vuelve a empezar en `EventoPorEtapas` → lo encuentra ahí.
3. Ese método le manda `cantidadDeSectores()` a cada etapa. Cada etapa hace **su propio** lookup: las fiestas lo encuentran en `Fiesta`; si una etapa fuera un evento por etapas, lo encontraría otra vez en `EventoPorEtapas` y bajaría un nivel más.
4. Suma: 1 + 4 + 1 = 6 → `6 > 3` → `true`.

## Paso 5: la recursión sin `if`

`EventoPorEtapas` **nunca pregunta** si una etapa es una fiesta o un evento por etapas. Le manda el mismo mensaje a cada una y cada una responde a su manera. **Eso es polimorfismo** (clase 21): el "caso base" de la recursión (la fiesta, que cuenta sus sectores) y el "caso recursivo" (el evento por etapas, que suma) los decide el **method lookup**, no un `if`.

Para el fin de semana:

```
finDeSemana.cantidadDeSectores()
 ├─ casamiento.cantidadDeSectores()          ← rama: suma sus etapas
 │   ├─ civil.cantidadDeSectores()     = 1   ← hoja
 │   ├─ fiestaDamian.cantidadDeSectores() = 4 ← hoja
 │   └─ after.cantidadDeSectores()     = 1   ← hoja
 │                                     = 6
 └─ brunch.cantidadDeSectores()        = 1   ← hoja
                                       = 7
```

La pregunta **baja** por el árbol hasta las hojas, y las respuestas **suben** sumándose. La rama no tiene sectores propios: le pregunta lo mismo a cada etapa y suma. La hoja responde con lo suyo. Nadie usa un `if` para decidir qué hacer.

## Paso 6: qué **no** cambió

| Parte | ¿Cambió? | Por qué |
|---|---|---|
| Proveedores | **No** | Solo le mandan mensajes de la interfaz de `Evento`. Solo se renombró el parámetro `unaFiesta` → `unEvento`, que es un nombre, no comportamiento. |
| `Cliente` | **No** | `fiesta` puede apuntar a una `Fiesta` o a un `EventoPorEtapas`: el cliente le manda `cuantoCobra(fiesta)` al proveedor y nada más. Lucía es un `Cliente` común. |
| `Fiesta` | Casi | Ahora hereda de `Evento`; sus reglas (`esLarga`, `esComplicada`) se movieron a la superclase. Responde **exactamente lo mismo** que antes: los tests de la entrega 2 lo comprueban. |
| Criterios de derroche | **No** | `CriterioFiestero` le pregunta `esLarga()` a la fiesta del cliente, sea cual sea. |

Esto es lo que contesta la tercera pregunta del README: **no hubo que modificar los proveedores**.

## Paso 7: los datos en el fixture

```wollok
const property casamiento = new EventoPorEtapas(etapas = [
    new Fiesta(duracion = 2, sectores = [new SectorDeMesas(cantidadMesas = 6, invitadosPorMesa = 5)]),   // el civil
    fiestaDamian,                                                                                         // la fiesta
    new Fiesta(duracion = 3, sectores = [new SectorDeMesas(cantidadMesas = 2, invitadosPorMesa = 5)])    // el after
])
const property finDeSemana = new EventoPorEtapas(etapas = [
    casamiento,
    new Fiesta(duracion = 2, sectores = [new SectorDeMesas(cantidadMesas = 4, invitadosPorMesa = 5)])    // el brunch
])

method lucia() = new Cliente(ahorros = 100000, fiesta = casamiento, criterioDeDerroche = criterioClasico)
```

- **La fiesta de Damián es el mismo objeto** (`fiestaDamian`), no una copia: la consigna dice *"la fiesta de Damián (la misma de la entrega 2)"*. Es lo mismo que Tomás en la entrega 2: **identidad** (clase 22). No hay problema en compartirla porque una fiesta no cambia (todo es `const`).
- **El fin de semana contiene al casamiento** como una etapa más: es el anidamiento que pide la consigna.
- **Las etapas son una lista**: tienen un orden (civil, fiesta, after) y, en teoría, la misma fiesta podría ser dos etapas.
- **Lucía es un `Cliente` común**, sin nada especial: su `fiesta` es un `EventoPorEtapas`. El atributo se sigue llamando `fiesta` porque así estaba desde la entrega 2.

## Paso 8: traza de los casos de la consigna

### Invitados, sectores y horas del casamiento

| Etapa | Invitados | Sectores | Horas |
|---|---|---|---|
| Civil | 6 × 5 = 30 | 1 | 2 |
| Fiesta de Damián | 30 + 20 + 10 + 16 = 76 | 4 | 8 |
| After | 2 × 5 = 10 | 1 | 3 |
| **Casamiento** | **116** | **6** | **13** |

### Complicado y complejidad

- 6 sectores > 3 → **complicado**.
- 13 horas > 5 → es largo → 3; complicado → +2 → **complejidad 5**.

### Lo que cobra cada proveedor por el casamiento

| Proveedor | Cuenta | Nivel | Cobra |
|---|---|---|---|
| Los Tilos | 500 × 6 sectores × 13 horas = 39000 | aprendiz (× 1) | **$39.000** |
| Marta | 600 × 6 × 1.2 (complicado) = 4320 | maestra (× 1.3) | **$5.616** |
| Escudo | 100 × 6 × 1.1 (largo) = 660 | aprendiz | **$660** |
| Doña Rosa | 20 × 116 = 2320 + 3 barriles (116 / 40 → 3) × 300 = 900 | aprendiz | **$3.220** |

Marta es el caso que junta las tres cosas de la entrega: el evento por etapas (mío), el recargo por complicado (entrega 2) y el nivel (punto 1).

### El fin de semana

| Etapa | Invitados | Sectores | Horas |
|---|---|---|---|
| Casamiento | 116 | 6 | 13 |
| Brunch | 4 × 5 = 20 | 1 | 2 |
| **Fin de semana** | **136** | **7** | **15** |

### Lucía

- Presupuesto: 100000 × 0.2 = **$20.000**.
- Escudo cobra $660 ≤ 20000 → **puede**.
- Los Tilos cobra $39.000 > 20000 → **no puede**.
- Contrata a Escudo: gastó **$660**, y Escudo gana 5 (complejidad del casamiento) × 2 (rol de seguridad) = **10** de experiencia.

## Paso 9: los tests

Están en el `describe "Entrega 3 - Punto 3: Eventos por etapas"`, al final de `tests.wtest`. Son 9:

| Test | Qué comprueba |
|---|---|
| 116 invitados, 6 sectores, 13 horas | la suma de las etapas (la recursión) |
| complicado y complejidad 5 | que las reglas de `Evento` funcionan con la rama |
| Los Tilos $39.000 | proveedor por sector con duración |
| Marta $5.616 | recargo por complicado + nivel |
| Escudo $660 | recargo por largo |
| Doña Rosa $3.220 | catering (usa `cantidadInvitados`) |
| Fin de semana 136 / 7 / 15 | **el anidamiento**: una etapa que es un evento por etapas |
| Lucía puede Escudo, no Los Tilos | el cliente con un evento por etapas |
| Lucía contrata a Escudo | contratar + experiencia con un evento por etapas |

- Elegí un proveedor de **cada rubro que usa un mensaje distinto** del evento (`duracion`, `esComplicada`, `esLarga`, `cantidadInvitados`), para probar que todos funcionan con la rama.
- El caso del fin de semana es el que prueba el Composite de verdad: sin él, alcanzaría con una lista de fiestas.
- Los objetos se toman del `fixtureCompartido` con variables del `describe` (clase 23).

## Paso 10: qué pasaría sin Composite (las alternativas descartadas)

Las alternativas se entienden mejor viendo **dónde se rompen**. Para cada una: el código que tendríamos y qué pasa con el **fin de semana**, que es el caso que pone a prueba el diseño.

### Alternativa 1: la rama mira adentro de las etapas

```wollok
class EventoPorEtapas inherits Evento {
    const etapas = []
    override method cantidadDeSectores() = etapas.sum({ etapa => etapa.sectores().size() })
    ...
}
```

Le pide a cada etapa **su lista de sectores** y la cuenta ella. Es pensar "las etapas son fiestas".

- **Con el casamiento funciona**: sus tres etapas son fiestas y todas tienen `sectores()`. Da 1 + 4 + 1 = 6.
- **Con el fin de semana se rompe**:

```
finDeSemana.cantidadDeSectores()
├─ casamiento.sectores()   💥 error: casamiento no entiende sectores()
└─ brunch.sectores()
```

El casamiento es un `EventoPorEtapas`: no tiene sectores propios, los tienen sus etapas. Wollok tira el error de que el objeto no entiende el mensaje.

**En nuestro código**, la rama le pregunta a cada etapa `etapa.cantidadDeSectores()`, que lo entienden las fiestas y los eventos por etapas.

> Es la "opción 1" de la clase 27: *"una subtarea no puede tener subtareas"*. Acá, una etapa no podría ser un evento por etapas.

### Alternativa 2: el evento por etapas con mensajes propios

Una clase aparte, sin heredar de `Evento` y con otros nombres:

```wollok
class Casamiento {
    const etapas = []
    method duracionTotal() = etapas.sum({ etapa => etapa.duracion() })
    method sectoresTotales() = etapas.sum({ etapa => etapa.cantidadDeSectores() })
    method invitadosTotales() = etapas.sum({ etapa => etapa.cantidadInvitados() })
}
```

Parece razonable, pero:

1. **Los proveedores no lo pueden cobrar.** Los Tilos hace `unEvento.cantidadDeSectores()` y `unEvento.duracion()`, y el casamiento no entiende esos mensajes, porque se llaman distinto. Habría que **cambiar todos los proveedores** con algo así:

   ```wollok
   // en cada proveedor 😬
   override method adicionalProveedor(unEvento) =
       if (unEvento es un casamiento) unEvento.duracionTotal() else unEvento.duracion()
   ```

   Además de que Wollok no tiene una forma linda de preguntar "¿es un casamiento?", la consigna dice que los proveedores tienen que cobrar **sin cambiar**.
2. **No se puede anidar.** El fin de semana le pediría `etapa.duracion()` al casamiento, que solo entiende `duracionTotal()`. 💥
3. **No tiene `esLarga()` ni `esComplicada()`.** Habría que copiar esas reglas en `Casamiento`: quedarían escritas dos veces.

**En nuestro código**, `EventoPorEtapas` **hereda de `Evento`** y redefine **los mismos mensajes**. Por eso los proveedores, el cliente y los criterios lo usan igual que a una fiesta.

### Alternativa 3: una sola clase `Fiesta` que a veces tiene etapas

```wollok
class Fiesta {
    const sectores = []
    const duracion = 0
    const etapas = []

    method cantidadDeSectores() =
        if (etapas.isEmpty()) sectores.size()
        else etapas.sum({ etapa => etapa.cantidadDeSectores() })

    method duracion() =
        if (etapas.isEmpty()) duracion
        else etapas.sum({ etapa => etapa.duracion() })

    method cantidadInvitados() =
        if (etapas.isEmpty()) sectores.sum({ sector => sector.invitadosSector() })
        else etapas.sum({ etapa => etapa.cantidadInvitados() })
    ...
}
```

**Funciona**, incluso con el fin de semana. Pero tiene problemas de diseño:

- **El mismo `if` aparece en tres métodos** ("¿tengo etapas?"): idea repetida (clase 26, 4.1). Si se agrega otra pregunta sobre el evento, hay que acordarse de poner el `if` otra vez.
- **Cada objeto tiene atributos que no usa.** Una fiesta simple tiene `etapas = []` vacía; el casamiento tiene `sectores = []` y `duracion = 0` que no significan nada. Da la falsa idea de que todo evento tiene sectores **y** etapas.
- **Se puede armar algo sin sentido**: una fiesta con sectores **y** etapas a la vez. El `if` hace que valgan las etapas y los sectores se ignoren sin que nadie avise.
- **Pierde cohesión** (clase 26, 2): la clase hace dos trabajos distintos según cómo la crearon.

> Es la "opción 2" de la clase 27: *"hay que preguntar si tiene subtareas en tres lugares"* y *"da la falsa idea de que todas las tareas tienen subtareas"*.

**En nuestro código**, cada clase hace una sola cosa: `Fiesta` siempre cuenta sus sectores, `EventoPorEtapas` siempre suma sus etapas. **No hay ningún `if`**: el "¿tengo etapas?" lo resuelve el **method lookup**.

### Alternativa 4: aplanar todo en una lista de fiestas

```wollok
const finDeSemana = new EventoPorEtapas(etapas = [civil, fiestaDamian, after, brunch])
```

**Los números dan bien** (136 invitados, 7 sectores, 15 horas). Pero:

- **Se pierde el casamiento como objeto.** La consigna dice que el fin de semana *"incluye el casamiento"*. Si el casamiento es la fiesta de Lucía y también parte del fin de semana, tiene que ser **el mismo objeto** en los dos lugares, no sus fiestas copiadas.
- **Hay que repetir las etapas.** Si cambia el casamiento (se le agrega una etapa), hay que acordarse de cambiar también el fin de semana: dos datos que dicen lo mismo y que hay que mantener sincronizados.

**En nuestro código**, el fin de semana tiene al casamiento **como una etapa más** (`etapas = [casamiento, brunch]`): el casamiento es uno solo.

### Otras dos que también descartamos

| Alternativa | Problema |
|---|---|
| **`EventoPorEtapas` hereda de `Fiesta`** | Heredaría `sectores` y `duracion` como atributos, que no tiene: sus sectores y su duración salen de las etapas. Un evento por etapas **no es una** fiesta. |
| **Copiar `esLarga`/`esComplicada` en `EventoPorEtapas`** | Idea repetida; si cambia la regla, hay que cambiarla en dos lugares. Por eso están en `Evento` (Paso 4). |

### Resumen

| Alternativa | ¿Casamiento? | ¿Fin de semana? | Problema |
|---|---|---|---|
| 1. La rama mira los sectores de las etapas | ✅ | 💥 | La rama pide algo que solo entienden las hojas |
| 2. Clase aparte con otros mensajes | 💥 con los proveedores | 💥 | No es polimórfica: hay que cambiar los proveedores |
| 3. Una sola clase con `if`s | ✅ | ✅ | `if` repetido, atributos sin sentido, poca cohesión |
| 4. Aplanar en una lista | ✅ | ✅ en números | Se pierde el casamiento como objeto; datos duplicados |
| **Composite (el nuestro)** | ✅ | ✅ | Más clases, pero cada una hace una sola cosa |

**Las dos condiciones que tiene que cumplir el diseño**, y que el Composite cumple:

1. **El evento por etapas entiende los mismos mensajes que una fiesta.** Si no, los proveedores no lo pueden cobrar (falla la alternativa 2).
2. **El evento por etapas solo les manda a sus etapas mensajes que entiende cualquier evento.** Si no, una etapa no puede ser otro evento por etapas (falla la alternativa 1).

Las alternativas 3 y 4 dan los números, pero son peores diseños: repiten ideas o duplican datos.

## Preguntas probables de la corrección (mi punto)

| Pregunta | Respuesta |
|---|---|
| ¿Qué patrón usaste? | **Composite**: `Fiesta` es la hoja, `EventoPorEtapas` la rama, `Evento` el componente. Además un Template Method en `Evento`. |
| ¿Por qué los proveedores no cambiaron? | Porque solo le mandan mensajes al evento y `EventoPorEtapas` entiende los mismos: son **polimórficos**. |
| ¿Cómo sabe `EventoPorEtapas` si una etapa es simple o compuesta? | **No lo sabe ni le importa**: le manda el mensaje y el lookup de cada etapa resuelve. |
| ¿Dónde está el caso base de la recursión? | En `Fiesta`: cuenta sus propios sectores y devuelve su duración, sin delegar en otros eventos. |
| ¿Por qué `Evento` y no repetir el código? | Para que las reglas de complicado, largo y complejidad estén **una sola vez** ("las mismas reglas de siempre"). |
| ¿`Evento` se puede instanciar? | No tiene sentido: tiene tres métodos abstractos. Es abstracta. |
| ¿Es lo mismo Composite que composición? | No. Composición = un objeto conoce a otro. Composite = ramas y hojas polimórficas, donde las ramas contienen componentes del mismo tipo. |
| ¿Qué pasa con un evento por etapas sin etapas? | Daría 0 invitados, 0 sectores y 0 horas (`sum` de una lista vacía es 0): no es largo ni complicado, complejidad 1. La consigna no lo pide. |
| ¿Por qué la fiesta de Damián está dos veces (en Damián y en el casamiento)? | No está dos veces: es **el mismo objeto** referenciado desde dos lugares. Se puede porque una fiesta no cambia. |
| ¿Y si un proveedor necesitara los sectores en sí, no la cantidad? | Habría que agregarle a `Evento` un mensaje `sectores()` y que el evento por etapas junte los de sus etapas. Hoy nadie lo necesita. |

---
---

# Punto grupal — Diagrama, README y regresión

## 1. El diagrama de clases

Está en el README (`diagrama-clases.drawio.png`, con el link a draw.io). Muestra:

- **Herencia** (flecha con triángulo vacío): `Evento`, `Proveedor`, `ProveedorPorSector`, `Catering`, `CriterioDerroche` con sus subclases; `doniaRosa`, `pizzasNico` y `elBodegon` son wko que heredan de `Catering`.
- **Asociación** (flecha con el nombre del atributo y `1` o `*`): `Cliente → Evento` (`fiesta`), `Cliente → Servicio` (`servicios`), `Cliente → CriterioDerroche`, `Servicio → Proveedor`, `agencia → Proveedor`, `EventoPorEtapas → Evento` (`etapas`), `Fiesta → sectores`.
- **"Usa"** (punteada): lo que se pide pero no se guarda. `Proveedor` usa a los niveles (se calculan), `CriterioFiestero` usa a `criterioClasico`, los caterings usan a `barril`.
- **Una nota por patrón**: Template Method, Strategy y Composite.

Lo que **no** está: las instancias (`marta`, `losTilos`…, son objetos, no clases) y `familia`/`facultad` (no se usan).

Las tres flechas clave para la defensa:

| Patrón | Dónde se ve en el diagrama |
|---|---|
| Template Method | `Proveedor` con `cuantoCobra` concreto y `costoServicio` abstracto (en cursiva) |
| Strategy | `Cliente → CriterioDerroche` |
| Composite | `EventoPorEtapas → Evento` (la flecha que vuelve a la superclase) |

## 2. Las respuestas del README

### ¿Qué repetición desaparece al incorporar el nivel? ¿Cómo quedó distribuido el cálculo?

- **Desaparece**: aplicar el porcentaje del nivel en cada rubro. Está **una sola vez**, en `Proveedor.cuantoCobra`.
- **Cómo quedó repartido** (lo que está en el código):

| Clase | Qué hace |
|---|---|
| `Proveedor` | `cuantoCobra = costoServicio × nivel().porcentajeAumento()` (template method). `costoServicio` y `complejidadDelRol` abstractos. |
| `ProveedorPorSector` | `costoServicio = precioPorSector × cantidadDeSectores × adicionalProveedor`. `adicionalProveedor` abstracto. |
| `Salon`, `EquipoDeMozos` | `adicionalProveedor` = la duración |
| `Organizador` | `adicionalProveedor` = recargo si es complicada |
| `Seguridad` | `adicionalProveedor` = recargo si es larga |
| `Catering` | `costoServicio = cuantoCobraTotal(cantidadInvitados)`. `cuantoCobraTotal` abstracto. |
| `doniaRosa`, `pizzasNico`, `elBodegon` | `cuantoCobraTotal` |

> ⚠️ El README quedó escrito por un compañero con algunas diferencias con el código (dice que `Proveedor` multiplica por `adicionalProveedor`, que el método se llama `complejidad()` y que los caterings tienen `adicionalProveedor = 1`). Decidimos dejarlo así. **Si lo preguntan, contestar con la tabla de arriba, que es lo que hace el código.**

Son **dos niveles de Template Method**: `Proveedor` deja el hueco `costoServicio`, y `ProveedorPorSector` lo llena dejando otro hueco, `adicionalProveedor`.

### ¿Por qué el criterio de derroche ya no puede ser como en la entrega 2?

Ver [2.2](#22-el-criterio-de-derroche-strategy): con herencia la clase es fija; el requerimiento pide cambiar el criterio en cualquier momento → Strategy.

### ¿Qué tuvieron que modificar en los proveedores para cobrar por un evento por etapas?

**Nada** (ver [Paso 6](#paso-6-qué-no-cambió)).

## 3. La regresión: los tests de las entregas 1 y 2

Corrimos los tests que estaban en `master` (entrega 2) contra el código nuevo.

**Primero no corrían**: el fixture viejo creaba los clientes con la clase `Fiestero`, que ya no existe, y sin `criterioDeDerroche`, que ahora es obligatorio. Se cambiaron solo esas 4 líneas, sin tocar ningún test.

**Resultado: de 47 tests, fallan 5 y pasan 42.**

| Fallan | Esperado (entrega 2) | Da ahora | Por qué |
|---|---|---|---|
| Marta, fiesta de 3 sectores | 1800 | 2340 | maestra: × 1.3 |
| Marta, fiesta complicada de 5 sectores | 3600 | 4680 | × 1.3 |
| Marta, fiesta de Isabella | 1200 | 1560 | × 1.3 |
| Marta, fiesta de Damián | 2880 | 3744 | × 1.3 |
| Eliana, fiesta de Damián | 7200 | 8640 | experimentada: × 1.2 |

**No es un error**: la consigna de la entrega 3 cambia cuánto cobran (son los "tests de regresión que deben modificarse" del punto 1). En el `tests.wtest` actual ya están con los valores nuevos.

**¿Por qué los demás siguen dando lo mismo?**

- **El resto de los proveedores son aprendices** (experiencia ≤ 500): multiplican por 1.
- **`Fiesta` responde lo mismo**, aunque ahora herede de `Evento`.
- **La agencia da los mismos conjuntos**: Marta le cobra a Isabella $1.560 y el presupuesto de Isabella es $1.650, así que le sigue alcanzando.
- **Los derrochones dan igual**: los criterios calculan lo mismo; solo cambió dónde está la lógica.

> Es lo que la clase 22 llama **test de regresión**: correr los mismos tests después de un cambio para ver qué se rompió. Y es la idea del **refactoring**: cambiar la estructura (herencia → Strategy, `Fiesta` → `Evento`) sin cambiar el comportamiento.

---
---

# Cierre — Conceptos para repasar el día antes

## 1. Los tres patrones, en una oración

| Patrón | En una oración | En el TP |
|---|---|---|
| **Template Method** | La superclase define el esqueleto del algoritmo y deja huecos (métodos abstractos) que cada subclase completa. | `Proveedor.cuantoCobra` (hueco: `costoServicio`); `ProveedorPorSector.costoServicio` (hueco: `adicionalProveedor`); `Evento.esComplicada`/`esLarga` (huecos: `cantidadDeSectores`, `duracion`) |
| **Strategy** | Un objeto delega parte de su comportamiento en otro objeto intercambiable. | `Cliente` delega `esDerrochon` en su `criterioDeDerroche` |
| **Composite** | Ramas y hojas polimórficas: quien las usa no distingue si son simples o compuestas. | `Fiesta` (hoja) y `EventoPorEtapas` (rama) |

## 2. Strategy vs. Template Method (clase 26, 5.3) aplicado al TP

| | Strategy (criterio de derroche) | Template Method (niveles) |
|---|---|---|
| Mecanismo | **Composición**: el cliente tiene un criterio | **Herencia**: las subclases completan un hueco |
| ¿Cuántos objetos? | Dos: el cliente y su criterio | Uno: el proveedor |
| ¿Se puede cambiar en runtime? | ✅ `cambiarCriterioDerroche(...)` | ❌ Un salón es salón para siempre |
| ¿Por qué ese y no el otro? | El criterio **cambia** durante la vida del cliente | El rubro **no cambia** nunca |

**La regla de la clase 27**: *herencia para lo que no cambia, composición para lo que sí cambia*. La pregunta clave es **qué puede cambiar** durante la vida del objeto.

¿Y el nivel? **Cambia** durante la vida del proveedor, pero no se guarda: se **calcula** desde la experiencia. El proveedor delega en el objeto nivel (`nivel().porcentajeAumento()`), parecido a un Strategy, pero el nivel no se elige de afuera: lo decide la experiencia.

## 3. Herencia vs. composición: la historia del fiestero

| Entrega | Diseño | Por qué |
|---|---|---|
| 2 | `class Fiestero inherits Cliente` con `override method esDerrochon() = !fiesta.esLarga() && super()` | Ser fiestero era algo fijo de la persona. Herencia era lo natural. |
| 3 | `Cliente` tiene un `criterioDeDerroche` | El criterio **puede cambiar** → composición (Strategy). |

Lo que se ganó: flexibilidad para cambiar en cualquier momento y agregar criterios sin tocar `Cliente`.
Lo que costó (clase 27): **más objetos**, construir un cliente pide un criterio, y la lógica queda **repartida** en otro objeto. Se pierde `super()`: ahora `CriterioFiestero` reutiliza al clásico **delegando** en él.

## 4. Clases y métodos abstractos (clase 24, 5.1)

- **Método abstracto**: no tiene cuerpo (`method duracion()`). Obliga a las subclases a implementarlo.
- **Clase abstracta**: agrupa comportamiento de sus subclases pero no se instancia. En Wollok no se declara: alcanza con no hacer `new`.
- En el TP: `Evento`, `Proveedor`, `ProveedorPorSector`, `Catering`, `CriterioDerroche`.
- En el código los marcamos con el comentario `//Primitiva`: es el nombre que la clase 26 le da a los huecos del template method.

## 5. `self` es lo que hace funcionar al Template Method

Aunque el código esté en la superclase, `self.algo()` **empieza a buscar en la clase del objeto receptor**. Por eso `Proveedor` puede llamar a `self.costoServicio(...)` sin saber qué rubro es: el lookup lo resuelve en la subclase. Si se usara `super`, empezaría a buscar más arriba y no encontraría la implementación.

## 6. Polimorfismo: los nuevos del TP

| Mensaje | Quiénes lo entienden | Quién lo usa sin saber cuál es |
|---|---|---|
| `cantidadDeSectores()`, `duracion()`, `cantidadInvitados()`, `esLarga()`, `esComplicada()`, `complejidadDelEvento()` | `Fiesta`, `EventoPorEtapas` | los proveedores, el cliente, los criterios, el propio `EventoPorEtapas` |
| `esDerrochon(unCliente)` | `CriterioClasico`, `CriterioFiestero` | `Cliente` |
| `aplicaA(experiencia)`, `porcentajeAumento()` | `aprendiz`, `experimentado`, `maestro` | `Proveedor` |
| `costoServicio(unEvento)` | `ProveedorPorSector`, `Catering` (y sus subclases) | `Proveedor.cuantoCobra` |
| `adicionalProveedor(unEvento)` | `Salon`, `Organizador`, `EquipoDeMozos`, `Seguridad` | `ProveedorPorSector.costoServicio` |
| `complejidadDelRol()` | todos los rubros | `Proveedor.sumarExperiencia` |

## 7. Estado vs. cálculo (clase 26, 4.6)

| Se guarda (estado) | Se calcula |
|---|---|
| `experiencia` del proveedor | `nivel()` |
| `servicios` del cliente | `totalGastado()`, `proveedoresContratados()`, `esDerrochon()` |
| `etapas` del evento por etapas | `duracion()`, `cantidadDeSectores()`, `cantidadInvitados()` |
| `sectores` y `duracion` de la fiesta | `esLarga()`, `esComplicada()`, `complejidadDelEvento()` |

Regla: si algo se puede calcular a partir de otro dato, **no se guarda**, para que nunca quede desactualizado.

## 8. Mensajes nuevos que aparecieron

| Mensaje | Qué hace | ¿Está en los apuntes? |
|---|---|---|
| `coleccion.find({ x => condición })` | devuelve el **primer** elemento que cumple; error si ninguno | ⚠️ No: está `findOrDefault` (clase 26) |
| `numero.between(a, b)` | ¿está entre `a` y `b`, incluidos? | Sí (clase 22) |
| `variable += valor` | suma y reasigna | Sí (clase 22 y 24) |
| `coleccion.sum({ x => ... })` | suma lo que devuelve el bloque para cada uno; 0 si está vacía | Sí (clase 22) |
| `object x inherits Clase` | un wko que hereda de una clase (los caterings, como `crossfit inherits Disciplina`) | Sí (clase 24 y 27) |

## 9. Las preguntas que van a hacer seguro

1. **¿Dónde usaron Template Method?** En `Proveedor.cuantoCobra`: el nivel se aplica una sola vez y cada rubro completa `costoServicio`.
2. **¿Dónde usaron Strategy y por qué no herencia?** En el criterio de derroche, porque el cliente lo tiene que poder cambiar y un objeto no puede cambiar de clase.
3. **¿Dónde usaron Composite?** En los eventos: `EventoPorEtapas` contiene `Evento`s, que pueden ser fiestas u otros eventos por etapas.
4. **¿Qué cambiaron en los proveedores para los eventos por etapas?** Nada: son polimórficos.
5. **¿Qué tests de regresión fallaron y por qué?** Los 5 de Marta y Eliana, porque ahora cobran según su nivel. Los demás proveedores son aprendices (× 1).
6. **¿Cómo se asegura que el nivel no quede desactualizado?** No se guarda: se calcula desde la experiencia cada vez.
7. **¿Por qué la experiencia se suma al final de `contratar`?** Para que el precio se calcule con el nivel que tenía al contratarlo.
