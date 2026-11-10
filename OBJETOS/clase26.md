# Clase 26 — Elementos de Diseño, Strategy y Template Method

### Módulo 15 + presentaciones "Elementos de Diseño" e "Implementando un Template method" + ejemplo Meeseeks

> La clase 25 fue la entrega oral del TP (ver [tp-objetos.md](tp-objetos.md)).

---

## Módulo 15: Elementos de Diseño

### 1. ¿Qué es diseñar?

Diseñar es:

1. **Encontrar los componentes.**
2. **Definir qué responsabilidades tiene cada uno**: su **alcance**, lo que hace y lo que **no** hace.
3. Y sobre todo, **definir las relaciones entre los componentes.**

> **Principio holístico:** el todo es más que la suma de las partes, porque **las relaciones** son las que enriquecen y dan vida a un sistema.

**Qué no es diseñar:** no es "hacer cajitas" ni saber UML. El diseño ocurre **en nuestra cabeza**; código, documentos o diagramas UML son sólo formas de **comunicarlo**.

#### 1.1 Componentes

Un componente puede verse en distintos niveles:

| Nivel                | Qué es un componente                                                           | Ejemplo                                                       |
| -------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------- |
| **Micro**            | Un objeto, una clase (en PDP también: una función, un predicado, un procedimiento) | `Alumno`, `Parcial`                                           |
| **Meso** (arquitectura) | Un conjunto de objetos/clases que cumplen un objetivo                       | Un alumno con sus notas modela su carrera académica           |
| **Macro**            | Módulos o sistemas que se comunican entre sí                                   | Módulo de correlatividades ↔ módulo de inscripciones          |

En un objeto, sus **responsabilidades** se marcan a partir de los **métodos** que implementa.

#### 1.2 ¿Qué es un "buen" diseño?

No hay una receta ("no tenemos la papa"), pero sí **heurísticas** y **atributos de calidad** con los que evaluarlo: **simple**, **mantenible**, **flexible**, **robusto**, **coherente**, entre otros. Dos de los más importantes son **cohesión** y **acoplamiento**.

### 2. Cohesión

- Una **clase es cohesiva** si tiene un **objetivo claro y definido**: su comportamiento y su estructura giran en torno a él.
- Un **método es cohesivo** si se concentra en **una sola cosa** a la vez.

Buenas prácticas para ganar cohesión:

- **Separar métodos con efecto** (emitir una factura, cambiarle el sueldo a un empleado) de los **sin efecto** (calcular el total facturado, conocer el promedio).
- **Extraer ideas repetidas** a un método con nombre propio: se evita duplicar código y cada método hace una sola cosa.

#### 2.1 Ejemplo: ¿cuándo estudia un alumno?

Hay tres tipos de alumno:

- **Estudioso**: estudia siempre.
- **Vago**: estudia sólo si le fue mal en el último examen (nota < 6).
- **Hijo del rigor**: estudia si el parcial es difícil (más de 5 preguntas).

Primera versión:

```wollok
class Alumno {
  const notas = []
  var tipoAlumno = "E"   // E = Estudioso, V = Vago, H = Hijo del rigor

  method rendir(nota) {
    notas.add(nota)
  }

  method estudiaPara(parcial) {
    if (tipoAlumno == "E") {
      // Estudioso
      return true
    }
    if (tipoAlumno == "V") {
      // Vago
      return notas.last() < 6
    }
    if (tipoAlumno == "H") {
      // Hijo del rigor
      return parcial.cantidadPreguntas() > 5
    }
    return false
  }
}

class Parcial {
  const property cantidadPreguntas
}
```

`estudiaPara` **no es cohesivo**: tiene varias responsabilidades mezcladas. El alumno:

1. sabe cuándo estudia un estudioso,
2. sabe cuándo estudia un vago,
3. sabe que él mismo guarda una **lista** de notas a la que le pide el último elemento,
4. sabe cuándo estudia un hijo del rigor,
5. sabe **cuándo un parcial es difícil**.

¡Cinco responsabilidades! Eso lo hace **más difícil de testear y de mantener**. Otras señales de alarma:

- Los tipos de alumno son **strings mágicos** (`"E"`, `"V"`, `"H"`) que necesitan un comentario para entenderse.
- Agregar un tipo nuevo obliga a **modificar** este método y sumar otro `if`.

#### 2.2 Refactor: reificar los criterios

**Reificar** = convertir una idea en un objeto. Cada criterio pasa a ser un objeto **polimórfico**, y cada responsabilidad queda en quien corresponde:

| Pregunta                                   | ¿Quién sabe responderla?                                |
| ------------------------------------------ | ------------------------------------------------------- |
| ¿Es difícil el parcial?                    | El **parcial**                                          |
| ¿Cuál fue la nota del último examen?       | El **alumno**                                           |
| ¿Estudia el alumno para este parcial?      | El **alumno**… **delegando** en su criterio de estudio  |

```wollok
class Estudioso {
  method estudiaPara(parcial, alumno) = true
}

class Vago {
  method estudiaPara(parcial, alumno) = alumno.notaUltimoExamen() < 6
}

class HijoDelRigor {
  method estudiaPara(parcial, alumno) = parcial.esDificil()
}

class Alumno {
  const notas = []
  var property criterioParaEstudiar = new Estudioso()

  method rendir(nota) {
    notas.add(nota)
  }

  method estudiaPara(parcial) =
    criterioParaEstudiar.estudiaPara(parcial, self)

  method notaUltimoExamen() = notas.last()
}

class Parcial {
  const property cantidadPreguntas

  method esDificil() = cantidadPreguntas > 5
}
```

- El alumno pasa de hacer 3 cosas a hacer **1**: delegar.
- Los criterios son **intercambiables**: `alumno.criterioParaEstudiar(new Vago())`.
- Agregar un criterio nuevo no toca al `Alumno`.
- `Parcial` es un **value object inmutable** (Módulo 14).

> Esta idea tiene nombre: es el patrón **Strategy** (ver sección 5).

### 3. Acoplamiento

> **Acoplamiento**: el grado de conocimiento que un componente tiene de los demás.

- **Acoplamiento cero no tiene sentido**: diseñar exige que los componentes se relacionen.
- Lo que se busca es **bajo acoplamiento**: que un cambio **interno** en un componente no obligue a cambiar a todos los que lo usan.

#### 3.1 Ejemplo: Vago

```wollok
class Vago {
  method estudiaPara(parcial, alumno) = alumno.notas().last() < 6
}
```

`Vago` sabe que el alumno guarda sus notas en una **colección ordenada**. Si el alumno cambia la lista por un conjunto, `Vago` se rompe: un `Set` no entiende `last()` porque no tiene orden.

Mejor: que sea **responsabilidad del alumno** decir la nota de su último examen:

```wollok
class Vago {
  method estudiaPara(parcial, alumno) = alumno.notaUltimoExamen() < 6
}
```

O, bajando todavía más el acoplamiento, que el alumno responda directamente si aprobó:

```wollok
class Vago {
  method estudiaPara(parcial, alumno) = !alumno.aproboUltimoExamen()
}
// en Alumno: method aproboUltimoExamen() = self.notaUltimoExamen() >= 6
```

> El apunte usa `aproboUltimoExamen()` pero no muestra su definición en `Alumno`; la línea comentada es una forma posible.

#### 3.2 Ejemplo: Hijo del rigor

```wollok
// Antes
class HijoDelRigor {
  method estudiaPara(parcial, alumno) = parcial.cantidadPreguntas() > 5
}

// Después
class HijoDelRigor {
  method estudiaPara(parcial, alumno) = parcial.esDificil()
}
```

Ventajas de delegarle la pregunta al parcial:

- Si cambia la regla de "parcial difícil", se modifica **en un solo lugar**.
- El concepto "**parcial difícil**" queda **reificado** en un método con nombre: es fácil de encontrar y de reutilizar. Con `cantidadPreguntas() > 5` suelto, nada dice que eso significa "difícil".

### 4. Heurísticas de buen diseño

| Heurística | Idea |
| --- | --- |
| **Evitar ideas repetidas** | Extraer lo redundante a un método con nombre |
| **Interfaz > implementación** | Pensar primero **cómo se usa** un objeto (qué servicios da) y después cómo se implementa |
| **Objetos chicos con responsabilidades marcadas** | Métodos largos son un *smell*; cuidado con los **Middle Man** (objetos que sólo delegan sin aportar); cada objeto y método debe aportar valor |
| **Composición > herencia** | Pensar primero en delegar en otro objeto antes que en heredar |
| **Reificar conceptos del negocio** | Todo concepto importante para el usuario debe ser una abstracción |
| **Acotar el efecto** | Controlar las mutaciones; separar estado de cálculo, y acciones de preguntas |
| **No todo tiene que ser flexible** | La flexibilidad agrega complejidad: usarla donde haga falta |

#### 4.1 Evitar ideas repetidas

```wollok
// Antes: costoPorMetroCuadrado aparece dos veces
method costoPorSuperficie(superficie) =
  if (superficie <= 25) costoPorMetroCuadrado * 2 else costoPorMetroCuadrado * 1.5

// Después: se extrae lo que varía
method costoPorSuperficie(superficie) =
  costoPorMetroCuadrado * self.costoExtraPorSuperficie(superficie)

method costoExtraPorSuperficie(superficie) = if (superficie <= 25) 2 else 1.5
```

#### 4.2 Interfaz > implementación

```wollok
class Cliente {
  var saldo = 0
  method pagar(factura) {
    saldo = saldo - factura.montoPendiente()
    // ...
    factura.montoPendiente(0)   // 🤔
  }
}
```

¿Por qué no mejor **`factura.cancelar()`**? El cliente no debería saber que "cancelar una factura" se implementa poniendo el monto en 0: eso es decisión de la factura. Pensar en la interfaz (qué le pido) y no en la implementación (cómo lo hace por dentro) baja el acoplamiento.

#### 4.3 Muchos objetos chicos > pocos objetos grandes

Es preferible un diseño con **muchos objetos intercambiables**, cada uno con una responsabilidad clara, que **pocos objetos con muchas responsabilidades**.

#### 4.4 Composición > herencia

- La herencia trabaja sobre **definiciones estáticas**: es **rígida**.
- Se puede usar **una sola vez** ("one shot only").
- Obliga a pensar todo desde **una única taxonomía** (un solo punto de vista).
- La composición abre la puerta a **jerarquías polimórficas** intercambiables.
- **Pero**: la composición requiere más objetos → hay una relación **costo/beneficio**.

(Ver el ejemplo de pesistas en [clase24.md, sección 7](clase24.md).)

#### 4.5 Reificar conceptos, encontrar abstracciones

Todo concepto importante para el usuario se lleva a una abstracción: un **objeto**, una **clase**, un **método** o una **constante**. El nombre en el código tiene que coincidir con el del negocio.

**Bad smell:** necesitar comentarios para aclarar lo que el código podría decir solo.

```wollok
method total(casa) =
  precioPorAmbiente * casa.cantidadDeAmbientes() * (if (casa.pisos() > 2) 20 else 0)
  //                                                   ↑ "es una casa complicada"
```

Ese `casa.pisos() > 2` es un concepto ("casa complicada") que merece su propio método: `casa.esComplicada()`. Suele asociarse con **falta de delegación** (querer resolver todo de una).

#### 4.6 Acotar el efecto

- En objetos se trabaja con **estado**, pero se puede elegir modelar objetos **inmutables** (valores: fechas, números, strings, o cosas que en el dominio no cambian, como un mail — Módulo 14).
- Diferenciar lo que es **estado** de lo que se puede **calcular**.
- **Separar mensajes que producen efecto de los que responden preguntas.**

### 5. Patrones de diseño

Muchas ideas de diseño frecuentes ya están **analizadas y catalogadas**, con sus ventajas, consecuencias e implementaciones posibles, en el libro **_Design Patterns: Elements of Reusable Object-Oriented Software_** (Gamma, Helm, Johnson y Vlissides, conocidos como el *Gang of Four*).

En esta clase aparecen dos: **Strategy** y **Template Method**.

#### 5.1 Strategy

Un objeto **delega** una parte de su comportamiento en otro objeto **intercambiable** (la estrategia). Todas las estrategias son **polimórficas** entre sí.

- **Ejemplo del alumno:** el `Alumno` delega en su `criterioParaEstudiar` (`Estudioso`, `Vago`, `HijoDelRigor`).
- **Ejemplo de pesistas** ([clase24.md](clase24.md)): el `Pesista` delega en su `disciplina`.
- Se basa en **composición**: la estrategia se puede **cambiar en cualquier momento**.

```
 ┌──────────┐  criterioParaEstudiar  ┌──────────────────────────┐
 │  Alumno  │───────────────────────▶│ estudiaPara(parcial, al) │  (interfaz común)
 └──────────┘                        └────────────▲─────────────┘
                              ┌───────────────────┼────────────────────┐
                        ┌─────┴─────┐       ┌─────┴─────┐       ┌──────┴──────┐
                        │ Estudioso │       │   Vago    │       │ HijoDelRigor│
                        └───────────┘       └───────────┘       └─────────────┘
```

#### 5.2 Template Method

> El **template method** define el **esqueleto de un algoritmo general** en una superclase, y le da a cada subclase la posibilidad de **redefinir una parte** de ese algoritmo.

**Ejemplo:** queremos saber si un contenido es interesante.

- Una **serie** es interesante si el presupuesto es de **más de 5 millones** y tiene **menos de 5 temporadas**.
- Una **película** es interesante si el presupuesto es de **más de 5 millones** y fue **estrenada este año**.

La condición del presupuesto es **común**; la otra es **específica** de cada uno.

**Casos de prueba:**

| | Serie | Película |
| --- | --- | --- |
| Presupuesto bajo | ❌ | ❌ |
| Falla la condición específica | ❌ muchas temporadas | ❌ estrenada hace más de un año |
| Presupuesto alto + condición específica | ✅ | ✅ |

Al principio alcanza con una clase `Serie`. Cuando aparece la película, abstraemos una superclase **abstracta** `Contenido`. Hay dos variantes:

> Las presentaciones sólo muestran diagramas; el código de abajo es una implementación posible en Wollok.

**Variante 1: usar `super`.** `Contenido` valida el presupuesto y cada subclase redefine `esInteresante`, llamando a `super` y agregando su validación:

```wollok
class Contenido {
  const presupuesto

  method esInteresante() = presupuesto > 5000000
}

class Serie inherits Contenido {
  const temporadas

  override method esInteresante() = super() && temporadas < 5
}

class Pelicula inherits Contenido {
  const anioEstreno

  override method esInteresante() = super() && anioEstreno == new Date().year()
}
```

**Variante 2: template method.** `Contenido` define **el algoritmo completo** (`esInteresante`) y deja un "hueco" que cada subclase **debe** completar: `cumpleCriterioInteresante()`, un **método abstracto**.

```wollok
class Contenido {
  const presupuesto

  // template method: el esqueleto del algoritmo
  method esInteresante() = presupuesto > 5000000 && self.cumpleCriterioInteresante()

  // primitiva: la parte que cada subclase completa
  method cumpleCriterioInteresante()
}

class Serie inherits Contenido {
  const temporadas

  override method cumpleCriterioInteresante() = temporadas < 5
}

class Pelicula inherits Contenido {
  const anioEstreno

  override method cumpleCriterioInteresante() = anioEstreno == new Date().year()
}
```

| | Variante 1 (`super`) | Variante 2 (template method) |
| --- | --- | --- |
| ¿Quién define el algoritmo completo? | Cada subclase (llamando a `super`) | La superclase |
| ¿Qué redefinen las subclases? | `esInteresante()` entero | Sólo `cumpleCriterioInteresante()` |
| Si una subclase se olvida de algo | Puede olvidarse de llamar a `super` y saltear la validación del presupuesto | No puede: el esqueleto no se toca, y si no implementa el método abstracto, falla |

**¿Cuántos objetos hay?** **Uno solo**: una serie o una película. Pero ese objeto necesita **al menos dos clases** para tener el comportamiento completo. El method lookup lo muestra:

1. `serie.esInteresante()` → no está en `Serie` → se encuentra en `Contenido`.
2. Dentro, `self.cumpleCriterioInteresante()` → `self` es la serie → el lookup **vuelve a empezar en `Serie`** → la encuentra ahí.

> La clave del template method es que **`self` siempre es el objeto receptor**: aunque el código esté en la superclase, `self.algo()` busca primero en la subclase.

Estructura general del patrón:

```
 ┌───────────────────────────┐
 │     AbstractTemplate      │
 ├───────────────────────────┤
 │ templateMethod()  ────────┼──▶ ... self.primitiva1() ... self.primitiva2() ...
 │ primitiva1()  (abstracto) │
 │ primitiva2()  (abstracto) │
 └─────────────▲─────────────┘
               │
 ┌─────────────┴─────────────┐
 │     ConcreteTemplate      │
 ├───────────────────────────┤
 │ primitiva1()              │
 │ primitiva2()              │
 └───────────────────────────┘
```

#### 5.3 Strategy vs. Template Method

| | Strategy | Template Method |
| --- | --- | --- |
| Mecanismo | **Composición** (delegar en otro objeto) | **Herencia** (subclases completan un hueco) |
| ¿Cuántos objetos? | Dos: el que delega y la estrategia | Uno: la instancia de la subclase |
| ¿Se puede cambiar en runtime? | ✅ Sí, se cambia la referencia | ❌ No, la clase es fija |
| Ejemplos | Criterio de estudio del alumno, disciplina del pesista, criterio de colapso de la caja | `Contenido`/`Serie`/`Pelicula`, `Disciplina.cantidadDeHoras()` en pesistas |

Se pueden **combinar**: en el ejemplo de pesistas con composición, `Pesista` usa **Strategy** con su disciplina, y `Disciplina` usa **Template Method** (`entrenaFierros` define el algoritmo y `cantidadDeHoras` es el hueco).

---

## Ejemplo de clase: Meeseeks destructores

Adaptación a objetos de un parcial de funcional (repo `meeseeksObjetosos`). Sirve para ver todos los elementos de diseño juntos.

### Enunciado (resumido)

Rick le regaló a la familia Smith una **caja de Meeseeks**. Cada Meeseeks tiene:

- **nombre** (arranca en `"Mr. Meeseeks"`, puede recibir sufijos),
- **color**: `celeste` → `azul` → `violeta` → `negro` (de más claro a más oscuro),
- **horas de existencia**,
- **tarea actual**,
- **historial** de intentos (cada uno con descripción y horas).

**Punto 1b — Color que merece**, según sus horas:

- `< 1` hora **y** nombre de longitud par → `celeste`
- entre 1 y 5 (inclusive) → `azul`
- más de 5 y hasta 24 → `violeta`
- cualquier otro caso → `negro`

**Punto 1c — Está sufriendo** si su color no coincide con el que merece, o si su tarea contiene `"golf"`.

**Punto 3 — Interacciones:**

- **Pedir ayuda**: +1 hora y cambia la tarea.
- **Golpear**: agrega `" (enojado)"` al nombre y duplica las horas (si tenía 0, queda en 1).
- **Agradecer**: guarda un intento con la tarea y las horas actuales, y reinicia horas a 0 y tarea a `"ninguna"`.
- **Frustrarse**: igual que golpear, pero dos veces.

**Punto 4 — La caja:**

- Tiene una **secuencia de castigos** (interacciones del punto 3) que la familia puede cambiar. **Apretar el botón** aplica, en orden, todos los castigos a cada Meeseeks.
- Tiene un **criterio de colapso** que también se puede cambiar: todos sufren (por defecto), más de la mitad sufre, o alguno sufre.
- **Apretar hasta colapsar**: aprieta al menos una vez, y sigue mientras no haya colapsado.

### Solución y decisiones de diseño

#### Colores: cada color sabe su regla

```wollok
object celeste {
  method correspondeA(meeseeks) = meeseeks.esRecienSalido() && meeseeks.tieneNombrePar()
}

object azul {
  method correspondeA(meeseeks) = meeseeks.horasDeExistencia().between(1, 5)
}

object violeta {
  method correspondeA(meeseeks) =
    meeseeks.horasDeExistencia() > 5 && meeseeks.horasDeExistencia() <= 24
}

object negro {}

object escalaDeColores {
  const colores = [celeste, azul, violeta]
  const colorPorDefecto = negro

  method colorPara(meeseeks) =
    colores.findOrDefault({ color => color.correspondeA(meeseeks) }, colorPorDefecto)
}
```

- **Reificación**: los colores son **objetos**, no strings. Cada uno sabe cuándo le corresponde a un Meeseeks (**cohesión**).
- No hay una cadena de `if`: `escalaDeColores` recorre los colores y devuelve el **primero** que corresponda, o `negro` por defecto (`findOrDefault`).
- `negro` no necesita `correspondeA` porque nunca se le pregunta: es el caso "por defecto".
- Agregar un color nuevo es agregar un objeto a la lista, sin tocar al Meeseeks.

#### Meeseeks

```wollok
class Meeseeks {
  var nombre = "Mr. Meeseeks"
  const color = celeste
  var horasDeExistencia = 0
  var tareaActual
  const historial = []
  const escala = escalaDeColores

  method nombre() = nombre
  method tareaActual() = tareaActual
  method horasDeExistencia() = horasDeExistencia
  method cantidadDeIntentos() = historial.size()
  method ultimoIntento() = historial.last()

  // Punto 1b
  method esRecienSalido() = horasDeExistencia < 1
  method tieneNombrePar() = nombre.size().even()
  method colorQueMerece() = escala.colorPara(self)

  // Punto 1c
  method estaSufriendo() = self.tieneColorEquivocado() || self.odiaSuTarea()
  method tieneColorEquivocado() = color != self.colorQueMerece()
  method odiaSuTarea() = tareaActual.contains("golf")

  // Punto 3
  method pedirAyuda(tarea) {
    horasDeExistencia = horasDeExistencia + 1
    tareaActual = tarea
  }

  method golpear() {
    nombre = nombre + " (enojado)"
    horasDeExistencia = self.horasDuplicadas()
  }

  method horasDuplicadas() = if (self.noTieneHoras()) 1 else horasDeExistencia * 2
  method noTieneHoras() = horasDeExistencia == 0

  method agradecer() {
    historial.add(new Intento(descripcion = tareaActual, horas = horasDeExistencia))
    horasDeExistencia = 0
    tareaActual = "ninguna"
  }

  method frustrarse() {
    self.golpear()
    self.golpear()
  }
}

class Intento {
  const property descripcion
  const property horas
}
```

Para notar:

- **Conceptos reificados en métodos con nombre**: `esRecienSalido()`, `tieneNombrePar()`, `tieneColorEquivocado()`, `odiaSuTarea()`, `noTieneHoras()`. Cada condición del enunciado tiene su nombre, sin comentarios.
- **Bajo acoplamiento**: los colores le preguntan al Meeseeks `esRecienSalido()` o `tieneNombrePar()`, sin meterse en cómo guarda el nombre.
- **No repetir lógica**: `frustrarse()` reutiliza `golpear()`.
- **Separar efecto de pregunta**: `golpear()` tiene efecto; `horasDuplicadas()` sólo calcula.
- `Intento` es un **value object inmutable**.
- Sólo se exponen los getters que el diseño **realmente necesita**.

#### Castigos: interacciones reificadas

```wollok
object golpe {
  method aplicarA(meeseeks) { meeseeks.golpear() }
}

object agradecimiento {
  method aplicarA(meeseeks) { meeseeks.agradecer() }
}

object frustracion {
  method aplicarA(meeseeks) { meeseeks.frustrarse() }
}

class PedidoDeAyuda {
  const tarea
  method aplicarA(meeseeks) { meeseeks.pedirAyuda(tarea) }
}
```

- Las interacciones del punto 3 son **mensajes**; para poder **guardarlas en una lista, pasarlas y cambiarlas**, hay que convertirlas en **objetos**. Su valor no está en el comportamiento (que ya tiene el Meeseeks) sino en **ser valores**.
- Todos son **polimórficos**: entienden `aplicarA(meeseeks)`.
- `golpe`, `agradecimiento` y `frustracion` son **wko** porque no tienen estado. `PedidoDeAyuda` es **clase** porque cada pedido lleva **su propia tarea** (regla de [clase24.md, sección 5.3](clase24.md)).

#### Criterios de colapso: Strategy

```wollok
object todosSufriendo {
  method colapsada(caja) = caja.cantidadDeSufridos() == caja.cantidadDeMeeseeks()
}

object mayoriaSufriendo {
  method colapsada(caja) = caja.cantidadDeSufridos() > caja.cantidadDeMeeseeks() / 2
}

object algunoSufriendo {
  method colapsada(caja) = caja.cantidadDeSufridos() > 0
}
```

La caja **delega** en su criterio, que es **intercambiable** ("la familia puede cambiarlo en cualquier momento" → **composición**, no herencia). El criterio le **pregunta cosas a la caja** (`cantidadDeSufridos()`, `cantidadDeMeeseeks()`) en vez de recorrer sus Meeseeks: bajo acoplamiento.

#### Caja

```wollok
class Caja {
  const meeseeks = #{}
  var property castigos
  var property criterioDeColapso = todosSufriendo

  method apretarBoton() {
    meeseeks.forEach { unMeeseeks => self.castigar(unMeeseeks) }
  }

  method castigar(unMeeseeks) {
    castigos.forEach { castigo => castigo.aplicarA(unMeeseeks) }
  }

  method colapso() = criterioDeColapso.colapsada(self)
  method cantidadDeSufridos() = meeseeks.count { unMeeseeks => unMeeseeks.estaSufriendo() }
  method cantidadDeMeeseeks() = meeseeks.size()

  // Se aprieta al menos una vez; la recursión sigue hasta el colapso.
  method apretarHastaColapsar() {
    self.apretarBoton()
    if (!self.colapso()) {
      self.apretarHastaColapsar()
    }
  }
}
```

- `castigos` y `criterioDeColapso` son `var property`: la familia los puede **cambiar en cualquier momento**.
- `apretarBoton()` y `castigar()` están **separados** (cada método hace una cosa): uno recorre los Meeseeks, el otro aplica la secuencia de castigos a uno.
- **¿Dónde va la recursividad?** Aplicar los castigos en orden es un simple `forEach` (los castigos son una lista ya conocida). Lo que **no se sabe cuántas veces** pasa es el botoneo hasta colapsar → ahí va la **recursión**.

#### En consola (punto 3)

```wollok
> const unMeeseeks = new Meeseeks(tareaActual = "nada")
> unMeeseeks.pedirAyuda("abrir un frasco")
> unMeeseeks.golpear()
> unMeeseeks.frustrarse()
> unMeeseeks.agradecer()
```

#### Tests

El repo incluye tests de todo lo anterior, organizados en un `describe` por tema (color, sufrimiento, interacciones, caja, criterio de colapso). Algunos detalles que repasan [clase23.md](clase23.md):

- Se testean los **valores límite**: exactamente 1 hora, exactamente 5, 5.5, 24 y 25 (**clases de equivalencia**).
- Hay variables compartidas en el `describe` (`const meeseeks = new Meeseeks(...)`) que **se reinician antes de cada test**.
- Se verifica que **cambiar la estrategia cambia el resultado** sobre el mismo objeto (`caja.criterioDeColapso(mayoriaSufriendo)`).

---

## Resumen de la clase

- **Diseñar** es encontrar componentes, asignarles responsabilidades y, sobre todo, definir cómo se relacionan.
- **Alta cohesión**: cada objeto tiene un objetivo claro; cada método hace una cosa.
- **Bajo acoplamiento**: cada objeto conoce lo mínimo de los demás; pide servicios (`esDificil()`, `notaUltimoExamen()`) en vez de meterse en su implementación.
- El **polimorfismo** y la **delegación** son las herramientas para lograr las dos cosas.
- **Strategy**: delegar en un objeto intercambiable (composición).
- **Template Method**: la superclase define el esqueleto del algoritmo y las subclases completan las partes que varían (herencia + método abstracto + `self`).

## Ideas que conectan con clases anteriores

- **Reificar** criterios en objetos polimórficos (alumno, colores, castigos, criterios de colapso) es la misma idea de **polimorfismo** desde [clase21.md](clase21.md): quien envía el mensaje no necesita saber qué objeto lo recibe.
- **Strategy** es la versión con nombre de la **composición** del ejemplo de pesistas ([clase24.md](clase24.md)). **Template Method** es lo que hacía `Disciplina` con su método abstracto `cantidadDeHoras`.
- El template method funciona gracias al **method lookup con `self`** ([clase24.md](clase24.md), sección 4.1): `self` siempre es el receptor, así que el lookup arranca en la subclase aunque el código esté en la superclase.
- **Acotar el efecto** retoma los value objects y la inmutabilidad (Módulo 14, [clase24.md](clase24.md)) y la separación entre acciones y preguntas (Módulo 10, [clase22.md](clase22.md)).
- **"Composición > herencia"** retoma herencia vs. composición (Módulo 13) y el problema de no poder cambiar de clase.
