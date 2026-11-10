# Clase 27 — El proceso de diseño: Manejo de proyectos

### Módulo 16 — Paradigma Orientado a Objetos (Fernando Dodino) + ejemplo `manejoProyectos`

---

## Módulo 16: El proceso de diseño

### 1. Enunciado

La compañía Kendari Corporation quiere una herramienta de **manejo de proyectos**.

- Los **proyectos** se componen de **tareas**, y las tareas de **subtareas** (que a su vez pueden tener subtareas).
- Cada tarea tiene un **tiempo** (en días) y un **costo**.

**Costo según la complejidad** (se usa sólo el tiempo propio de la tarea, no el de sus subtareas):

| Complejidad | Costo |
| --- | --- |
| **Mínima** | `tiempo * $25` |
| **Media** | `tiempo * $25` + 5% |
| **Máxima** | `tiempo * $25` + 7%, y si dura más de 10 días, + $10 por cada día después del décimo |

**Extras:**

- **Overhead**: las tareas con **más de 3 subtareas** tienen un 4% extra.
- **Impuestos**: existe el **impuesto A (3%)** y el **impuesto B (5%)**. Una tarea puede tener los dos, uno o ninguno. Se aplican sólo a la tarea que los tiene, no a sus subtareas.

**Completitud:**

- Las tareas **sin subtareas** están completas o no: 0% o 100%.
- Las tareas **con subtareas** calculan el promedio ponderado de sus subtareas.

**Atraso posible** (depende de la complejidad y el tiempo):

| Complejidad | Atraso |
| --- | --- |
| Mínima | 5 días |
| Media | 10% del tiempo |
| Máxima | 20% del tiempo + 8 días |

**Se pide:**

1. Obtener el costo de una tarea.
2. Obtener el costo total de un proyecto.
3. Obtener los días máximos de atraso de un proyecto.
4. Saber o asignar el porcentaje de completitud de una tarea.

### 2. Objetivo de la clase

Lo importante **no es el resultado final** sino **el proceso** que lleva a elegir entre alternativas:

- encontrar formas **objetivas** de comparar soluciones y decidir,
- repasar por qué la **composición** es más flexible que la herencia,
- decidir cuándo modelar con **clases** y cuándo con **instancias**,
- usar una metodología **iterativa** que tolere equivocarse.

### 3. Qué es diseñar

Diseñar es encontrar **los componentes**, **qué hacen** y **cómo se relacionan** (ver [clase26.md](clase26.md)). En objetos, un componente puede ser un objeto o un conjunto de objetos que brindan un servicio. Por ejemplo: el alumno y su criterio de estudio forman juntos **un componente** que sabe decir si un alumno estudia para un parcial.

### 4. Algunas cuestiones sobre el diseño

#### 4.1 El diseño en otros paradigmas

En funcional y en lógico **también diseñamos**: los componentes se construyen de otra forma (funciones, predicados), pero las ideas de cohesión, acoplamiento y abstracción son las mismas.

#### 4.2 ¿Cuánto diseñar?

Algunas metodologías piden diseñar todo antes de programar. Acá se prefiere **tomar algunas decisiones y postergar otras**, porque:

- no siempre se tiene toda la información,
- usuario y desarrollador **van aprendiendo** el dominio mientras se construye,
- **los requerimientos cambian**.

Cuando aparece una idea, se **valida con algo de código** (no todo, ni todas las alternativas): alcanza para saber si es implementable. Al no diseñar todo de antemano, es más fácil **aceptar el error** y repensar.

> El error es parte de la naturaleza humana: conviene una metodología que **lo tolere**, no que lo esconda.

#### 4.3 Diseño y diagramas

El diseño ocurre **en la mente** al generar abstracciones. Saber UML sirve para **comunicar** el diseño, pero el diseño existe aunque no haya documentación formal.

### 5. Manejo de proyectos: el proceso

#### 5.1 ¿Por dónde arrancar?

1. **No perder de vista los requerimientos**: son los que guían el proceso y evitan hacer cosas de más.
2. Elegir una herramienta para encontrar los componentes (cualquiera sirve):
   - **diagrama de clases** (estático),
   - **diagrama de objetos** (dinámico),
   - lista de **objetos candidatos** y sus responsabilidades,
   - escribir **un test primero** e implementar lo mínimo para que pase, iterando (**TDD**),
   - **tirar código** directamente… sabiendo frenar a tiempo para que no se vuelva "code & fix".

**Cuando me trabo, cambio la perspectiva:**

- Si estoy con un diagrama, paso al código (o a un test, o a un diagrama de objetos).
- Si tengo dudas sobre los requerimientos, **le pregunto a quien entiende del negocio**.
- O anoto la duda en una **lista de pendientes** y **postergo** la decisión.

#### 5.2 Objetos candidatos

Lista tentativa (no todos van a quedar): **Proyecto**, **Tarea**, **Subtarea**, **Complejidad**, **Impuesto**.

#### 5.3 Diagrama estático: primeros intentos

El costo de una tarea depende de:

- si **tiene subtareas** (overhead),
- su **complejidad**,
- sus **impuestos**.

Y la completitud también cambia según si la tarea **tiene subtareas o no**: sólo las que no tienen se pueden marcar como cumplidas, y se calcula distinto en cada caso.

La primera decisión importante: **¿cómo representar tareas y subtareas?**

**Opción 1 — `Tarea` tiene `Subtarea`s:**

```
┌───────┐  subtareas  ┌──────────┐
│ Tarea │────────────▶│ Subtarea │
└───────┘      *      └──────────┘
```

❌ Una subtarea no puede tener subtareas.

**Opción 2 — `Tarea` tiene una colección de `Tarea`s** (referencia recursiva; si está vacía, no tiene subtareas):

```
┌───────┐
│ Tarea │◀──┐
└───┬───┘   │ subtareas *
    └───────┘
```

Es flexible, pero:

- hay que **preguntar si tiene subtareas** en tres lugares (consultar completitud, asignarla y calcular el overhead),
- da la falsa idea de que **todas** las tareas tienen subtareas,
- `Tarea` queda con **baja cohesión**: representa a la vez a las que tienen y a las que no tienen subtareas.

**Opción 3 — `Tarea` y `Subtarea` separadas**, cada una con sus subtareas:

❌ Es la más compleja, no reutiliza conceptos parecidos y **tampoco** distingue tareas con y sin subtareas.

> **La trampa del enunciado:** lo importante **no es distinguir tarea de subtarea**, sino distinguir **tareas que tienen subtareas** de **tareas que no tienen**.

**Opción 4 — subclasificar en `TareaSimple` y `TareaCompuesta`:**

```
            ┌───────┐
            │ Tarea │◀────────────┐
            └───▲───┘             │ subtareas *
        ┌───────┴────────┐        │
┌───────┴─────┐ ┌────────┴───────┐│
│ TareaSimple │ │ TareaCompuesta │┘
└─────────────┘ └────────────────┘
```

Al subclasificar aparecieron **mejores nombres**: encontramos una abstracción.

#### El conflicto: dos criterios de clasificación

La complejidad **también** cambia el comportamiento (costo y atraso). Si además quisiéramos `TareaComplejidadMinima`, `TareaComplejidadMedia`, `TareaComplejidadMaxima`… **la herencia permite un solo punto de vista**. Tenemos dos:

1. Tarea **simple / compuesta**.
2. Tarea de complejidad **mínima / media / máxima**.

Como no pueden convivir las dos jerarquías, usamos **composición** para una de ellas:

| Opción | Herencia para… | Composición para… | Consecuencia |
| --- | --- | --- | --- |
| 1 | Simple / compuesta | **Complejidad** (objeto aparte) | Cambiar la complejidad es fácil (se cambia la referencia); pasar de simple a compuesta es costoso |
| 2 | Complejidad | Simple / compuesta (con `if (self.esSimple())`) | Cambiar la complejidad es costoso; `Tarea` pierde cohesión (sabe hacer lo de simples y compuestas) |
| 3 | Complejidad | Simple / compuesta (objeto aparte) | Pasar de simple a compuesta es fácil; cambiar la complejidad, no |

**¿Cuál elegir?** No lo puedo decidir yo: el proceso de diseño me lleva a **hacerle una pregunta valiosa al usuario**. Respuesta:

> "Las tareas **pueden cambiar su complejidad**, en cambio una tarea simple **no se transforma** en compuesta ni viceversa."

→ **Opción 1**: herencia para simple/compuesta (no cambia), composición para la complejidad (sí cambia). La idea de "Subtarea" desaparece: queda sólo como candidato descartado.

Pendientes que quedan anotados: costo (de tarea y de proyecto), overhead, impuestos, completitud.

#### 5.4 Identificando los requerimientos: ¿a quién le mando qué mensaje?

| Requerimiento | Mensaje |
| --- | --- |
| Costo de una tarea | `tarea.costo()` |
| Costo total de un proyecto | `proyecto.costoTotal()` |
| Asignar completitud a una tarea simple | `tareaSimple.porcentajeCompletitud(x)` |
| Completitud de un proyecto | `proyecto.porcentajeCompletitud()` |
| Tiempo total de un proyecto | `proyecto.tiempoTotal()` |
| Días máximos de atraso de un proyecto | `proyecto.diasMaximoDeAtraso()` |

**Modelo vs. realidad (otra vez):** "quiero saber el saldo de un cliente" → ¿se lo pregunto al cliente? En la realidad no, pero el `Cliente` del sistema **no es el cliente físico**, es una abstracción. Está bien que el saldo sea comportamiento suyo (como el petrel que cuenta sus kilómetros, [clase24.md](clase24.md)).

**¿Para qué pensar en componentes?** Tarea + complejidad + impuestos forman el componente "costo de una tarea". Si se hacen cambios **sin tocar la interfaz de `Tarea`**, el `Proyecto` no se ve afectado.

#### 5.5 Costo de una tarea

El costo tiene tres partes, cada una con su propia abstracción:

```wollok
method costo() = self.costoBase() + self.costoPorOverhead() + self.costoImpositivo()
```

**Overhead**: hay dos opciones:

- Definirlo **abstracto** en `Tarea` y redefinirlo en cada subclase → cualquier subclase nueva está **obligada** a definirlo (ventaja o desventaja, según el caso). `Tarea` queda abstracta.
- Definirlo con un valor por defecto (0) en `Tarea` y redefinirlo sólo en `TareaCompuesta` → `Tarea` se podría instanciar. ¿Lo queremos?

Se elige la primera:

```wollok
class Tarea {
  method costoPorOverhead()   // abstracto
}

class TareaSimple inherits Tarea {
  override method costoPorOverhead() = 0
}

class TareaCompuesta inherits Tarea {
  const subtareas = []

  override method costoPorOverhead() =
    if (self.tieneMuchasSubtareas()) self.costoBase() * 0.04 else 0

  method tieneMuchasSubtareas() = subtareas.size() > 3
}
```

- `tieneMuchasSubtareas()` **reifica** la condición en un método con nombre: se puede reutilizar.
- Desventaja: `costoBase()` se calcula dos veces (se podría guardar en una variable local y pasarlo).

> Esto es un **Template Method** ([clase26.md](clase26.md)): `costo()` está definido en la superclase abstracta y delega una parte (`costoPorOverhead()`) en cada subclase.

#### 5.6 Agregando los impuestos

Se analizan cuatro alternativas:

**Opción 1 — una clase por impuesto:**

```wollok
class ImpuestoA {
  method costo(costo) = costo * 0.03
}

class ImpuestoB {
  method costo(costo) = costo * 0.05
}
```

No hace falta una superclase: comparten una **interfaz común** (`costo(costo)`) que en Wollok no se escribe. Pero sólo difieren en el porcentaje.

**Opción 2 — un wko por impuesto:**

```wollok
object impuestoA {
  method costo(costo) = costo * 0.03
}

object impuestoB {
  method costo(costo) = costo * 0.05
}
```

- Se acceden **globalmente**.
- Se pueden **compartir** entre muchas tareas, porque **no tienen estado** propio de una tarea.
- Pero siguen siendo dos objetos que **repiten la misma idea**.

**Opción 3 — wko que heredan de una superclase común:**

```wollok
class Impuesto {
  const porcentaje
  method costo(costo) = costo * porcentaje / 100
}

object impuestoA inherits Impuesto(porcentaje = 3) {}
object impuestoB inherits Impuesto(porcentaje = 5) {}
```

O, mejor, como **constantes** (deja claro que **no hay comportamiento diferencial**, son sólo instancias distintas):

```wollok
const impuestoA = new Impuesto(porcentaje = 3)
const impuestoB = new Impuesto(porcentaje = 5)
```

El cálculo está en un solo lugar y se mantienen las referencias globales. Pero **no escala** si la cantidad de impuestos crece.

**Opción 4 — una clase `Impuesto` y simplemente instancias:**

```wollok
class Impuesto {
  const porcentaje
  method costo(costo) = costo * porcentaje / 100
}
```

Cada tarea tiene 0, 1 o 2 instancias de `Impuesto`.

- ✅ Se pueden tener muchos impuestos: sólo hay que instanciarlos.
- ❌ Se puede perder la abstracción "impuesto A" / "impuesto B", si fuera importante.

**Preguntas para el usuario:** ¿cuántos impuestos surgieron en los últimos años? ¿Hubo alguno con otro cálculo? ¿Siempre fueron A y B? Mientras tanto, se elige la **opción 4** por simple.

| Opción | ¿Repite la idea? | ¿Escala con muchos impuestos? | ¿Conserva los nombres A y B? |
| --- | --- | --- | --- |
| 1. Una clase por impuesto | Sí | No | Sí |
| 2. Un wko por impuesto | Sí | No | Sí |
| 3. Superclase + wko/constantes | No | No | Sí |
| 4. Una clase, muchas instancias | No | **Sí** | No necesariamente |

> Es la regla de [clase24.md, sección 5.3](clase24.md): **clases distintas sólo si hay comportamiento distinto**. Si sólo cambia un valor, alcanza con instancias.

Costo impositivo de la tarea:

```wollok
class Tarea {
  const impuestos = #{}

  method costoImpositivo() {
    const costoBase = self.costoBase()   // se calcula una sola vez
    return impuestos.sum { impuesto => impuesto.costo(costoBase) }
  }
}
```

#### 5.7 Costo total de un proyecto

```wollok
class Proyecto {
  const tareas = []

  method costoTotal() = tareas.sum { tarea => tarea.costoTotal() }
}

// TareaSimple
method costoTotal() = self.costo()

// TareaCompuesta
method costoTotal() = self.costo() + subtareas.sum { tarea => tarea.costoTotal() }
```

El código de `Proyecto` y de `TareaCompuesta` es **parecido**, pero **no igual**: representan conceptos distintos. Ojo con cambiar todo el diseño sólo porque dos cosas se parecen.

Se podría hacer que el proyecto tenga una **tarea raíz** y delegue en ella:

```wollok
class Proyecto {
  var tareaRaiz
  method costoTotal() = tareaRaiz.costoTotal()
}
```

> **Cuidado con el sobrediseño:** hacer cosas de más que nunca se usan tiene un costo y vuelve la aplicación más compleja. Trabajar de forma **iterativa** permite empezar simple e ir sumando complejidad a medida que se gana confianza.

#### 5.8 Complejidad de una tarea

¿Por qué separar la complejidad en otro objeto?

1. La herencia ofrece **una sola chance** de clasificar, y ya la usamos para simple/compuesta.
2. Si se modelara con un número (1 = mínima, 2 = media, 3 = máxima), aparecería un `if` múltiple **en dos lugares**:

```wollok
method costoBase() {
  if (complejidad == 1) { ... }   // mínima
  if (complejidad == 2) { ... }   // media
  if (complejidad == 3) { ... }   // máxima
}

method diasMaximosDeAtraso() {
  if (complejidad == 1) { ... }   // mínima
  if (complejidad == 2) { ... }   // media
  if (complejidad == 3) { ... }   // máxima
}
```

Lo más grave no es tener que agregar un `if` si aparece una complejidad "crítica", sino que **la misma decisión está en dos lugares** con números mágicos que tienen que estar **sincronizados**, y que no se pueden exponer hacia afuera.

Con un objeto complejidad:

```wollok
// en Tarea
method costoBase() = complejidad.costo(self)
```

**Discusión 1: ¿`Complejidad` debería ser clase abstracta?**

- Si hay **comportamiento en común** (acá sí: `tiempo * $25`), una superclase abstracta evita repetir código.
- Si no lo hubiera, alcanza con una **interfaz** (los mensajes que todas entienden). En Wollok la interfaz no se escribe: es una abstracción de diseño.

**Discusión 2: ¿le paso la tarea o sólo el tiempo?**

| | `complejidad.costo(self)` (la tarea) | `complejidad.costo(tiempo)` (el tiempo) |
| --- | --- | --- |
| Acoplamiento | La complejidad podría pedirle **más cosas** a la tarea → a priori más acoplamiento | La complejidad no conoce a la tarea → menos mensajes posibles |
| Conocimiento de la tarea | La tarea no sabe qué usa la complejidad para calcular | La tarea **sabe** que el costo depende de su tiempo |
| Si cambian los requerimientos | Si hace falta otro dato, **no cambia la interfaz** | Si hace falta otro dato, **hay que cambiar** `costo(tiempo)` |

No hay una respuesta única: es un trade-off.

#### 5.9 Resolviendo el costo de la complejidad

En las tres complejidades se repite `tiempo * 25`. Y en la mínima, `tiempo * 25` es lo mismo que `tiempo * 25 + 0`. → Otro **Template Method**:

```wollok
class Complejidad {
  // template method
  method costo(tarea) {
    const costoPorTiempo = tarea.tiempo() * 25
    return costoPorTiempo + self.costoExtra(tarea.tiempo(), costoPorTiempo)
  }

  // hook (primitiva): cada subclase está obligada a definirla
  method costoExtra(tiempo, costoPorTiempo)
}

class ComplejidadMinima inherits Complejidad {
  override method costoExtra(tiempo, costoPorTiempo) = 0
}

class ComplejidadMedia inherits Complejidad {
  override method costoExtra(tiempo, costoPorTiempo) = 0.05 * costoPorTiempo
}

class ComplejidadMaxima inherits Complejidad {
  override method costoExtra(tiempo, costoPorTiempo) =
    costoPorTiempo * 0.07 + self.costoAdicionalPorRetraso(tiempo)

  method costoAdicionalPorRetraso(tiempo) = 10 * 0.max(tiempo - 10)
}
```

- `costo(tarea)` es el **template method**.
- `costoExtra(...)` es el **hook** o **primitiva** que cada subclase completa.
- `0.max(tiempo - 10)`: si dura 10 días o menos da 0; si dura más, da los días que pasaron del décimo.

### 6. Patrones de diseño que aparecieron

Además de **Template Method** (en `Tarea.costo()` y en `Complejidad.costo()`), aparecieron dos más.

#### 6.1 Strategy

Para **separar la complejidad de la tarea**: cada algoritmo de cálculo se encapsula en un objeto polimórfico.

```
┌───────┐  complejidad  ┌──────────────────┐
│ Tarea │──────────────▶│   Complejidad    │
└───────┘               ├──────────────────┤
                        │ costo(tarea)     │
                        │ diasAtraso(tarea)│
                        └────────▲─────────┘
               ┌─────────────────┼──────────────────┐
     ┌─────────┴───────┐ ┌───────┴────────┐ ┌───────┴─────────┐
     │ComplejidadMinima│ │ComplejidadMedia│ │ComplejidadMaxima│
     └─────────────────┘ └────────────────┘ └─────────────────┘
```

**Ventajas:**

- Frente a los **números mágicos e `if`s**: el cálculo de costo y de atraso queda encapsulado en objetos polimórficos; tarea y complejidad tienen responsabilidades separadas y **se pueden testear por separado**.
- Frente a **subclasificar la tarea por complejidad**:
  - aparece una **abstracción nueva** (la complejidad) que antes estaba escondida dentro de la tarea,
  - el cambio es **dinámico**: la tarea sigue siendo **la misma** (conserva su identidad) y sólo cambia su complejidad. Con herencia habría que crear otro objeto.

**Consecuencias:**

- Hay **más objetos** (tarea + complejidad).
- La tarea conoce a la complejidad; la complejidad puede o no conocer a la tarea (como variable, como parámetro, o recibiendo sólo el tiempo).

**Desventajas** (frente a ifs o subclases):

- Construir una tarea es más complejo.
- El comportamiento queda **distribuido** en otro objeto, lo que a primera vista parece menos natural.

**Otro ejemplo:** una colección que puede ordenarse con distintos algoritmos (burbujeo, QuickSort, MergeSort): cada algoritmo es una estrategia.

#### 6.2 Composite

Para trabajar una **estructura jerárquica** donde **ramas y hojas** son **polimórficas**: quien las usa no distingue si son simples o compuestas.

**Ejemplo:** archivos y directorios en el explorador del sistema operativo.

- Con los dos se puede hacer lo mismo: abrir, copiar, eliminar (son polimórficos).
- Un archivo nunca se convierte en directorio, ni al revés.

**Aplicado al ejercicio:**

```
            ┌────────────────┐
            │     Tarea      │◀────────────┐
            │ costoTotal()   │             │
            └───────▲────────┘             │ subtareas *
        ┌───────────┴──────────┐           │
┌───────┴──────┐      ┌────────┴───────┐   │
│ TareaSimple  │      │ TareaCompuesta │───┘
│   (hoja)     │      │    (rama)      │
└──────────────┘      └────────────────┘
```

Como `TareaCompuesta` tiene una colección de `Tarea` (simples **o** compuestas), se pueden **anidar n niveles**. Y `costoTotal()` funciona igual en todos: la hoja devuelve su costo, la rama suma el de sus hijas.

> **Ojo: _Composite_ ≠ _composición_.**
>
> - **Composición**: un objeto **conoce a otro** (un cliente tiene proyectos, una tarea tiene su complejidad).
> - **Composite**: una **jerarquía de ramas y hojas polimórficas**, donde las ramas contienen otros componentes del mismo tipo.

### 7. Resumen del módulo 16

A partir de un ejemplo se tomaron decisiones de diseño **no triviales**. La **composición** aparece como forma de armar jerarquías de objetos polimórficos para comportamientos específicos. Elegir entre alternativas requiere analizar **objetivamente** ventajas y desventajas, y ese análisis además **dispara preguntas al usuario**. Iterar entre pensar y bajar a código permite que las decisiones puedan **cambiar y escalar**.

---

## Ejemplo de clase: repositorio `manejoProyectos`

La solución publicada por la cátedra (repo `wollok/manejoProyectos`). Está separada en tres archivos.

### `impuestos.wlk`

```wollok
class Impuesto {
  const porcentaje

  method calcularValor(costo) = costo * porcentaje / 100
}

const impuestoA = new Impuesto(porcentaje = 3)
const impuestoB = new Impuesto(porcentaje = 5)
```

Una clase con constantes globales (una mezcla de las opciones 3 y 4 de la sección 5.6). El mensaje se llama `calcularValor` en vez de `costo`.

### `tareas.wlk`

```wollok
class Complejidad {
  method costoBase(tiempo) = tiempo * 25

  // template method
  method costo(tarea) = self.costoBase(tarea.tiempo()) + self.costoExtra(tarea.tiempo())

  method costoExtra(tiempo)   // abstracto
}

object complejidadMinima inherits Complejidad {
  override method costoExtra(tiempo) = 0
  method diasAtraso(tarea) = 5
}

object complejidadMedia inherits Complejidad {
  override method costoExtra(tiempo) = 0.05 * self.costoBase(tiempo)
  method diasAtraso(tarea) = 1.1 * tarea.tiempo()
}

object complejidadMaxima inherits Complejidad {
  override method costoExtra(tiempo) =
    self.costoBase(tiempo) * 0.07 + self.costoAdicionalPorRetraso(tiempo)

  method costoAdicionalPorRetraso(tiempo) = 10 * 0.max(tiempo - 10)

  method diasAtraso(tarea) = 1.2 * tarea.tiempo() + 8
}

class Tarea {
  var property complejidad = complejidadMinima
  var property tiempo
  const impuestos = #{}

  method agregarImpuesto(impuesto) {
    impuestos.add(impuesto)
  }

  method cumplir()   // abstracto

  method costo() = self.costoComplejidad() + self.costoPorOverhead() + self.costoImpositivo()

  method costoComplejidad() = complejidad.costo(self)

  method diasAtraso() = complejidad.diasAtraso(self)

  method costoPorOverhead()   // abstracto

  method costoImpositivo() {
    const costoBase = self.costoComplejidad()
    return impuestos.sum { impuesto => impuesto.calcularValor(costoBase) }
  }
}

class TareaSimple inherits Tarea {
  var property porcentajeCumplimiento = 0

  override method costoPorOverhead() = 0

  override method cumplir() {
    porcentajeCumplimiento = 100
  }

  method costoTotal() = self.costo()
}

class TareaCompuesta inherits Tarea {
  const subtareas = []

  override method cumplir() {
    throw new DomainException(message = "Solo puede marcar como cumplida una tarea simple")
  }

  method agregarSubtarea(subtarea) {
    subtareas.add(subtarea)
  }

  override method costoPorOverhead() =
    if (self.tieneMuchasSubtareas()) self.costoComplejidad() * 0.04 else 0

  method tieneMuchasSubtareas() = subtareas.size() > 3

  method costoTotal() = subtareas.sum { tarea => tarea.costoTotal() }

  method porcentajeCumplimiento() =
    subtareas.sum { subtarea => subtarea.porcentajeCumplimiento() } / subtareas.size()
}
```

### `proyectos.wlk`

```wollok
class Proyecto {
  const tareas = []

  method agregarTarea(tarea) {
    tareas.add(tarea)
  }

  method costoTotal() = tareas.sum { tarea => tarea.costoTotal() }

  method diasMaximosDeAtraso() = tareas.map({ tarea => tarea.diasAtraso() }).maxIfEmpty({ 0 })
}
```

### Qué se ve en el código

- **Strategy**: `Tarea` delega costo y atraso en su `complejidad`, que se puede cambiar con `tarea.complejidad(complejidadMaxima)`.
- **Template Method, dos veces**: `Tarea.costo()` (con `costoPorOverhead()` abstracto) y `Complejidad.costo(tarea)` (con `costoExtra(tiempo)` abstracto).
- **Composite**: `TareaSimple` y `TareaCompuesta` entienden `costoTotal()` y `porcentajeCumplimiento()`, y una compuesta puede contener tanto simples como compuestas.
- **Las complejidades son wko**, no clases como en el apunte: no tienen estado, así que alcanza con un único objeto compartido por todas las tareas.
- **Completitud**: `cumplir()` es abstracto en `Tarea`. En una simple pone el 100%; en una compuesta **tira una excepción**, porque esa acción no tiene sentido para ella (Módulo 10, [clase22.md](clase22.md)).
- **`maxIfEmpty({ 0 })`**: el máximo de una colección, o 0 si está vacía (si no, `max()` fallaría con un proyecto sin tareas).

### Diferencias entre el repo, el apunte y el enunciado

Conviene tenerlas en cuenta para no confundirse:

| Tema | Enunciado / apunte | Repo |
| --- | --- | --- |
| Clases de complejidad | Clases (`ComplejidadMinima`, …) | wko (`complejidadMinima`, …) |
| Hook de complejidad | `costoExtra(tiempo, costoPorTiempo)` | `costoExtra(tiempo)`, recalcula `costoBase(tiempo)` |
| Atraso complejidad media | **10%** del tiempo (`0.1 * tiempo`) | `1.1 * tiempo` (110%) |
| Atraso complejidad máxima | **20%** del tiempo + 8 (`0.2 * tiempo + 8`) | `1.2 * tiempo + 8` |
| `costoTotal()` de compuesta | `self.costo()` + suma de subtareas | **Sólo** la suma de subtareas (no suma su propio costo) |
| Completitud de compuesta | Promedio **ponderado** | Promedio **simple** |
| Mensaje del impuesto | `costo(costo)` | `calcularValor(costo)` |

> En un parcial, guiarse por el **enunciado**. Las fórmulas de atraso del repo dan más del doble que las del enunciado (por ejemplo, para una tarea media de 1 día: enunciado → 0.1; repo → 1.1).

### Tests

El repo tiene dos `describe`. Algunos valores para verificar a mano:

| Tarea | Tiempo | Complejidad | Impuestos | Costo |
| --- | --- | --- | --- | --- |
| `escribirTests` | 4 | mínima | — | 4 × 25 = **100** |
| `armarFixture` | 2 | mínima | A y B | 50 + 1.5 + 2.5 = **54** |
| `programarObjetoPepita` | 1 | media | — | 25 + 1.25 = **26.25** |
| `programarObjetoPepona` | 2 | máxima | — | 50 + 3.5 = **53.5** |
| Tarea larga | 12 | máxima | — | 300 + 21 + 20 = **341** |

Repasan lo de [clase23.md](clase23.md):

- `override method initialize()` arma el escenario: agrega subtareas, impuestos y cambia complejidades **antes de cada test**.
- Se testea el **error** con `assert.throwsExceptionWithMessage(...)` al hacer `programarTP0.cumplir()`.
- El segundo `describe` testea las complejidades **por separado** de las tareas (una ventaja del Strategy), incluido el **valor límite** de 10 días (4 días → sin recargo; 12 días → +$20).

---

## Ideas que conectan con clases anteriores

- Es la aplicación completa de **herencia vs. composición** ([clase24.md](clase24.md)): herencia para lo que **no cambia** (simple/compuesta), composición para lo que **sí cambia** (complejidad). La pregunta clave al usuario es justamente **qué puede cambiar** durante la vida del objeto.
- **Strategy** y **Template Method** ([clase26.md](clase26.md)) aparecen juntos: `Tarea` usa Strategy con su complejidad, y tanto `Tarea` como `Complejidad` tienen un template method.
- **Clases vs. instancias** ([clase24.md](clase24.md), sección 5.3): los impuestos sólo difieren en un porcentaje → instancias; las complejidades difieren en el cálculo → objetos distintos.
- **Cohesión y acoplamiento** ([clase26.md](clase26.md)): la opción 2 de tareas (una sola clase con `if`s) tiene baja cohesión; pasar `self` o `tiempo` a la complejidad es una discusión de acoplamiento.
- **Composite** es **polimorfismo** ([clase21.md](clase21.md)) aplicado a una estructura recursiva: el proyecto le pide `costoTotal()` a cada tarea sin saber si es simple o compuesta.
