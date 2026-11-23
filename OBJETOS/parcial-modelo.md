# Parcial modelo de Objetos — Causas y funcionarios

Modelo de parcial resuelto en clase, explicado punto por punto: qué concepto evalúa cada parte, por qué se resolvió así y qué se podría mejorar.

- **Tema 1**: enunciado + resolución (de acá hasta "Tests que convendría escribir").
- **[Tema 2](#tema-2--sólo-la-resolución)**: sólo la resolución, sin consignas.
- **[Checklist para el parcial](#checklist-para-el-parcial)**: al final.

---

## Mapa rápido: qué evalúa cada punto

| Punto | Qué pide | Concepto / patrón | Ver |
| --- | --- | --- | --- |
| 1 | Perjuicio de distintas causas | **Herencia + Template Method**, **Composite** | [clase24.md](clase24.md), [clase26.md](clase26.md), [clase27.md](clase27.md) |
| 2 | Comerse una causa según el cargo | **Strategy** (composición), **excepciones** | [clase26.md](clase26.md), [clase22.md](clase22.md) (Módulo 10) |
| 3 | Filtrar causas / salir en medios | **Colecciones** (`filter`, `forEach`), **delegación** | [clase22.md](clase22.md) (Módulo 4) |
| 4 | Propuestas y promesas | **Strategy** otra vez, **fechas**, **mutabilidad** | [clase24.md](clase24.md) (Módulo 14) |

---

## Punto 1 — ¿Estamos en problemas?

### Enunciado (resumido)

Las causas tienen una carátula, una lista de jueces y un **perjuicio económico** (en millones de dólares):

- **Soborno**: monto base + 1 si hay arrepentidos, + 2 si no.
- **Obra pública**: monto base de 3; los jueces **siempre son los mismos**; + 2 si tiene hasta dos jueces, + 0 si tiene más.
- **Compleja**: formada por sub-causas de **cualquier tipo** (incluso otras complejas); monto base + la suma de los **montos extra** de cada sub-causa.

### Resolución

```wollok
class CausaJudicial {
  const juecesVinculados = []
  var montoBase

  // template method
  method perjuicioEconomico() = montoBase + self.ajusteEspecifico()

  // primitiva abstracta
  method ajusteEspecifico()

  method vincularJuez(juez) {
    juecesVinculados.add(juez)
  }

  method tieneAlJuez(juez) = juecesVinculados.contains(juez)

  method perjuicioMayorA(valor) = self.perjuicioEconomico() > valor

  method aumentarMontoBase(valor) {
    montoBase += valor
  }
}

class Soborno inherits CausaJudicial {
  const hayArrepentidos

  override method ajusteEspecifico() = if (hayArrepentidos) 1 else 2
}

object obraPublica inherits CausaJudicial(montoBase = 3) {
  override method ajusteEspecifico() = if (juecesVinculados.size() > 2) 0 else 2
}

class CausaCompleja inherits CausaJudicial {
  const causas = []

  override method ajusteEspecifico() = causas.sum({ causa => causa.ajusteEspecifico() })
}
```

### Por qué así

- **Template Method**: las tres causas calculan el perjuicio **igual** (`montoBase + algo`); sólo cambia el "algo". Entonces `perjuicioEconomico()` va en la superclase y `ajusteEspecifico()` queda **abstracto** para que cada subclase lo complete. `CausaJudicial` es **abstracta** (nunca se hace `new CausaJudicial(...)`).
- **¿Herencia o composición acá?** Herencia está bien: una causa de soborno **nunca se transforma** en obra pública. El tipo de causa es **intrínseco** ([clase24.md](clase24.md), sección 5.4).
- **Composite**: `CausaCompleja` tiene una colección de causas de **cualquier tipo**, incluida otra compleja. Para ella todas son polimórficas porque entienden `ajusteEspecifico()`. Es el mismo esquema que `TareaCompuesta` en [clase27.md](clase27.md).
- **`obraPublica` es un wko**: "los jueces que tratan esta causa **siempre son los mismos**" se interpretó como que hay **una sola** obra pública. El wko hereda de la clase pasándole `montoBase = 3`, igual que `theKing` en el ejemplo de pesistas.
- **Métodos que reifican preguntas**: `tieneAlJuez(juez)` y `perjuicioMayorA(valor)` evitan que otros objetos tengan que meterse en la lista de jueces o hacer la cuenta ellos ([clase26.md](clase26.md), acoplamiento).

### Para pensar

- **Falta la carátula.** El enunciado la menciona y no está modelada. En un parcial conviene modelar **todo lo que el enunciado nombra**, aunque no se use: `const property caratula`.
- **¿Una sola obra pública?** Es una decisión discutible. Si pudiera haber varias causas de obra pública (cada una con su monto), haría falta una clase `ObraPublica` con `montoBase = 3` por defecto. Lo importante en un parcial es **justificar la decisión**.
- **Consecuencia del wko**: como `obraPublica` es un único objeto, si dos funcionarios se la comen y uno sale en los medios (punto 3), **el monto sube para los dos**. Con instancias no pasaría.
- **Ojo en `CausaCompleja`**: suma el `ajusteEspecifico()` de las sub-causas, **no** su `perjuicioEconomico()`. Es lo que pide el enunciado ("montos extra"), pero es fácil confundirse.

---

## Punto 2 — Ñoqui al pesto

### Enunciado (resumido)

Un funcionario tiene causas y patrimonio. Para **comerse una causa**:

- tiene que tener patrimonio (> 0), y además:
- **poder ejecutivo**: tiene que tener un **juez amigo** en la causa;
- **ministro**: su patrimonio tiene que ser **mayor al 50%** del perjuicio de la causa.

Si no cumple, hay que definir **cómo responde**. Los funcionarios **pueden cambiar de puesto**: la solución tiene que ser **flexible**.

### Resolución

```wollok
class Funcionario {
  const property causasJudiciales = []
  const property promesas = []
  var property patrimonio
  var property cargo

  method validarPatrimonio() {
    if (patrimonio <= 0) {
      throw new DomainException(message = "Está incriminando a un chicho que no tiene plata")
    }
  }

  method comerseUnaCausa(causa) {
    self.validarPatrimonio()
    cargo.validarCausa(causa, self)
    causasJudiciales.add(causa)
  }
  // ...
}

class PoderEjecutivo {
  const juezAmigo
  const palabrasClave = []

  method validarCausa(causa, funcionario) {
    if (!causa.tieneAlJuez(juezAmigo)) {
      throw new DomainException(message = "Necesito tener un juez amigo para estar en esta causa")
    }
  }
  // ...
}

object ministro {
  method validarCausa(causa, funcionario) {
    if (funcionario.patrimonio() <= causa.perjuicioEconomico() * 0.5) {
      throw new DomainException(
        message = "El patrimonio debe ser mayor al 50% del perjuicio económico de la causa"
      )
    }
  }
  // ...
}
```

### Por qué así

- **"Pueden pasar de un puesto a otro" → composición (Strategy).** Es la frase clave del punto. Si `Ministro` y `PoderEjecutivo` fueran subclases de `Funcionario`, un ministro no podría pasar al ejecutivo (un objeto no cambia de clase). Con `var property cargo`, alcanza con `funcionario.cargo(ministro)`. Mismo razonamiento que pesistas ([clase24.md](clase24.md)) y complejidad de tareas ([clase27.md](clase27.md)).
- **La validación común queda en el funcionario** (`validarPatrimonio()`), y **la específica se delega en el cargo** (`cargo.validarCausa(...)`). No se repite lógica.
- **Excepciones, no booleanos.** `comerseUnaCausa` es una **acción**: si no se puede, se **tira un error** que corta la ejecución antes del `add` ([clase22.md](clase22.md), Módulo 10). Devolver `false` o no hacer nada sería "mirar a un costado".
- **¿Por qué `PoderEjecutivo` es clase y `ministro` es wko?** Cada funcionario del ejecutivo tiene **su propio** juez amigo y sus propias palabras clave → necesita una **instancia** propia. Un ministro no tiene datos propios → alcanza con **un solo objeto** compartido. Es la regla de `Fisicoculturismo` vs. `powerlifting`.
- **Polimorfismo**: los dos cargos entienden `validarCausa(causa, funcionario)` con la **misma firma**, aunque `PoderEjecutivo` no use el parámetro `funcionario`. Para el `Funcionario` son intercambiables.

### Para pensar

- **Separar pregunta de acción** ([clase22.md](clase22.md), sección 10.2): se podría agregar `puedeComerse(causa)` que devuelva un booleano, y que `comerseUnaCausa` use eso para validar. Hace falta sólo si el enunciado pide las dos cosas.
- Si se quisiera **reutilizar la estructura "validar y tirar error"**, se podría escribir `cargo.puedeComerse(causa, funcionario)` en cada cargo y un único `if` en el funcionario con un mensaje genérico. Se pierde el mensaje específico de cada caso: trade-off ([clase22.md](clase22.md), sección 10.3).

---

## Punto 3 — Si Neustadt te viera…

### Enunciado (resumido)

- **Causas que te dejan pegado**: las que tienen un perjuicio **mayor a 5 millones**.
- **Salir en medios**: cada vez, **todas** las causas que se comió aumentan su monto base en 0,1.

### Resolución

```wollok
// en Funcionario
method causasQueTeDejanPegado() = causasJudiciales.filter({ causa => causa.perjuicioMayorA(5) })

method salirEnLosMedios() {
  causasJudiciales.forEach({ causa => causa.aumentarMontoBase(0.1) })
}
```

### Por qué así

- `filter` para una **pregunta** (devuelve una colección nueva, sin efecto); `forEach` para una **acción** (efecto sobre cada causa).
- El funcionario **no toca el monto base** de la causa: le **pide** a la causa que se aumente (`aumentarMontoBase`). La responsabilidad de modificar el estado de la causa es de la causa.

### Para pensar

- **Reificar el concepto**: "perjuicio tramboliko" es un concepto del negocio. En vez de `perjuicioMayorA(5)` con el 5 en `Funcionario`, sería más expresivo `causa.esTrambolika()` con el 5 adentro de `CausaJudicial` ([clase26.md](clase26.md), sección 4.5). Así, si cambia el límite, se cambia en un solo lugar.
- **Composite y salir en medios**: si el funcionario se comió una causa compleja, sólo aumenta el monto base **de la compleja**, no el de sus sub-causas. Está bien, porque el enunciado habla de las causas que se comió.

---

## Punto 4 — Se acerca la campaña

### Enunciado (resumido)

Las propuestas tienen descripción, fecha de presentación y fecha de cumplimiento. Al escuchar una propuesta, el funcionario la convierte en **promesa**: la acepta **tal cual**, o la toma con la fecha de cumplimiento **4 años después**.

- **Poder ejecutivo**: acepta tal cual las que contienen alguna **palabra clave** de sus asesores ("aumento", "impuestos", "inflación"…).
- **Ministro**: acepta tal cual las **cortas** (menos de un año entre presentación y cumplimiento).

### Resolución

```wollok
// en Funcionario
method escucharPropuesta(propuesta) {
  if (!cargo.aceptaPropuesta(propuesta)) propuesta.postergar()
  promesas.add(propuesta)
}

// en PoderEjecutivo
method aceptaPropuesta(propuesta) =
  palabrasClave.any({ palabra => propuesta.contieneEnDescripcion(palabra) })

// en ministro
method aceptaPropuesta(propuesta) = propuesta.esCorta()

class Propuesta {
  const fechaPresentacion
  var fechaCumplimiento
  const descripcion

  method fechaCumplimiento() = fechaCumplimiento

  method postergar() {
    fechaCumplimiento = fechaCumplimiento.plusYears(4)
  }

  method contieneEnDescripcion(palabraClave) = descripcion.contains(palabraClave)

  method esCorta() = fechaCumplimiento < fechaPresentacion.plusYears(1)
}
```

### Por qué así

- **Se reutiliza el mismo Strategy** del punto 2: el cargo ahora también decide qué propuestas acepta. Por eso el cargo tiene dos responsabilidades polimórficas: `validarCausa` y `aceptaPropuesta`.
- **Lo común queda en el funcionario** (postergar o no, y agregar a promesas); **lo que varía, en el cargo**.
- **Bajo acoplamiento**: los cargos no leen la descripción ni las fechas, **le preguntan a la propuesta** (`contieneEnDescripcion`, `esCorta`).
- **Fechas inmutables**: `plusYears(4)` **no modifica** la fecha, devuelve una **nueva**; por eso hay que reasignar `fechaCumplimiento = ...` ([clase22.md](clase22.md), fechas; [clase24.md](clase24.md), Módulo 14).
- `any` devuelve `true` si **alguna** palabra clave aparece.

### Para pensar: el problema de mutar la propuesta

El enunciado dice que el funcionario **"la transforma en una promesa"**. La resolución **modifica la propuesta original** con `postergar()`. Problema:

```wollok
const propuesta = new Propuesta(descripcion = "bajar la inflación", ...)
unMinistro.escucharPropuesta(propuesta)       // la posterga 4 años
unEjecutivo.escucharPropuesta(propuesta)      // ¡ya viene postergada!
```

Si dos funcionarios escuchan **la misma propuesta**, el segundo recibe la fecha ya corrida, y si también la posterga, queda **8 años** adelante. Es exactamente el problema de compartir objetos **mutables** del Módulo 14.

Alternativa: que la propuesta sea **inmutable** y que postergar devuelva **una promesa nueva**.

```wollok
class Propuesta {
  const property fechaPresentacion
  const property fechaCumplimiento
  const property descripcion

  method postergada() = new Propuesta(
    descripcion = descripcion,
    fechaPresentacion = fechaPresentacion,
    fechaCumplimiento = fechaCumplimiento.plusYears(4)
  )
  // ...
}

// en Funcionario
method escucharPropuesta(propuesta) {
  promesas.add(self.promesaPara(propuesta))
}

method promesaPara(propuesta) =
  if (cargo.aceptaPropuesta(propuesta)) propuesta else propuesta.postergada()
```

Además separa la **pregunta** (`promesaPara`, sin efecto) de la **acción** (`escucharPropuesta`).

---

## Cómo se usaría en consola

```wollok
> const kometa = "Dr. Rodolfo Kometa"
> const unSoborno = new Soborno(montoBase = 10, hayArrepentidos = true)
> unSoborno.vincularJuez(kometa)
> unSoborno.perjuicioEconomico()                  // 11

> const ejecutivo = new PoderEjecutivo(juezAmigo = kometa, palabrasClave = ["aumento", "inflación"])
> const funcionario = new Funcionario(patrimonio = 100, cargo = ejecutivo)
> funcionario.comerseUnaCausa(unSoborno)          // ok: Kometa está en la causa
> funcionario.causasQueTeDejanPegado()            // [unSoborno] (11 > 5)
> funcionario.salirEnLosMedios()                  // el monto base pasa a 10.1

> funcionario.cargo(ministro)                     // cambia de puesto: mismo objeto, otro cargo
```

(Los jueces se modelan como strings con su nombre; también podrían ser wko.)

---

## Tests que convendría escribir

Siguiendo [clase23.md](clase23.md), una prueba por **clase de equivalencia**, incluidos los **valores límite** y los **errores**:

```wollok
describe "perjuicio de las causas" {
  test "un soborno con arrepentidos suma 1 al monto base" {
    assert.equals(11, new Soborno(montoBase = 10, hayArrepentidos = true).perjuicioEconomico())
  }

  test "un soborno sin arrepentidos suma 2 al monto base" {
    assert.equals(12, new Soborno(montoBase = 10, hayArrepentidos = false).perjuicioEconomico())
  }

  test "la obra pública con exactamente 2 jueces suma 2" {   // valor límite
    obraPublica.vincularJuez("Kometa")
    obraPublica.vincularJuez("Garantis")
    assert.equals(5, obraPublica.perjuicioEconomico())
  }

  test "la obra pública con 3 jueces no suma nada" {
    obraPublica.vincularJuez("Kometa")
    obraPublica.vincularJuez("Garantis")
    obraPublica.vincularJuez("Nacausa")
    assert.equals(3, obraPublica.perjuicioEconomico())
  }

  test "una causa compleja suma los ajustes de sus sub-causas" {
    const compleja = new CausaCompleja(
      montoBase = 1,
      causas = [new Soborno(montoBase = 10, hayArrepentidos = true), obraPublica]
    )
    assert.equals(4, compleja.perjuicioEconomico())   // 1 + 1 + 2
  }
}

describe "comerse una causa" {
  const causa = new Soborno(montoBase = 10, hayArrepentidos = true)   // perjuicio 11

  test "un funcionario sin patrimonio no se puede comer una causa" {
    const pobre = new Funcionario(patrimonio = 0, cargo = ministro)
    assert.throwsExceptionWithMessage(
      "Está incriminando a un chicho que no tiene plata",
      { pobre.comerseUnaCausa(causa) }
    )
  }

  test "un ministro con exactamente el 50% del perjuicio no se la puede comer" {   // límite
    const justo = new Funcionario(patrimonio = 5.5, cargo = ministro)
    assert.throwsException({ justo.comerseUnaCausa(causa) })
  }

  test "un ministro con más del 50% del perjuicio se la come" {
    const rico = new Funcionario(patrimonio = 6, cargo = ministro)
    rico.comerseUnaCausa(causa)
    assert.that(rico.causasJudiciales().contains(causa))
  }

  test "un ejecutivo sin su juez amigo en la causa no se la puede comer" {
    const ejecutivo = new Funcionario(patrimonio = 100, cargo = new PoderEjecutivo(juezAmigo = "Kometa"))
    assert.throwsException({ ejecutivo.comerseUnaCausa(causa) })
  }
}
```

> Ojo: `obraPublica` es un wko, pero **el ambiente se reinicia antes de cada test**, así que los jueces agregados en un test no aparecen en el siguiente.

---

## Tema 2 — Sólo la resolución

> No tenemos las consignas del Tema 2. Las reglas de esta sección están **inferidas del código** (nombres de métodos, mensajes de error y cálculos): pueden no coincidir exactamente con el enunciado real.

### Lo primero que salta: es el mismo esqueleto que el Tema 1

| | Tema 1 | Tema 2 |
| --- | --- | --- |
| Superclase de causas con template method | `CausaJudicial` + `ajusteEspecifico()` | **Igual** (ahora con `caratula`) |
| Causa en clase | `Soborno` | `DesvioDeFondos` |
| Causa como wko | `obraPublica` | `criptoDelito` |
| Composite | `CausaCompleja` | **Igual** |
| Funcionario con cargo intercambiable (Strategy) | `PoderEjecutivo` (clase), `ministro` (wko) | `legislador` (wko), `jefeDeGobierno` (wko) |
| Cargo valida causas con excepciones | `validarCausa(causa, funcionario)` | **Igual** |
| Cargo decide si acepta propuestas | `aceptaPropuesta(propuesta)` | **Igual** |
| Consultas sobre colecciones | `filter` | `all`, `filter`, `map` |
| Salir en medios | `forEach` + `aumentarMontoBase` | **Igual** |
| Dependencia de la fecha de hoy | — | **`calendario`** (nuevo) |

> **Conclusión para estudiar:** la cátedra arma los dos temas sobre **la misma estructura**. Si entendés por qué cada pieza es como es (template method en las causas, composite en las complejas, strategy en el cargo, excepciones al validar), el otro tema sale cambiando las reglas.

### Causas

```wollok
object calendario {
  method hoy() = new Date()
}

class CausaJudicial {
  const juecesVinculados = []
  var montoBase
  const property caratula

  method perjuicioEconomico() = montoBase + self.ajusteEspecifico()

  method ajusteEspecifico()

  method vincularJuez(juez) {
    juecesVinculados.add(juez)
  }

  method montoRaro() = montoBase.even()

  method aumentarMontoBase(valor) {
    montoBase += valor
  }

  method tieneMasDeUnJuez() = juecesVinculados.size() > 1
}

class DesvioDeFondos inherits CausaJudicial {
  const fechaCausa
  const calendarioActual = calendario

  method aniosPendiente() = calendarioActual.hoy().year() - fechaCausa.year()

  override method ajusteEspecifico() = self.aniosPendiente() * 1.5
}

object criptoDelito inherits CausaJudicial(montoBase = 5, caratula = "estafa") {
  override method ajusteEspecifico() = montoBase * 0.3
}

class CausaCompleja inherits CausaJudicial {
  const causas = []

  override method ajusteEspecifico() = causas.sum({ causa => causa.ajusteEspecifico() })
}
```

**Reglas inferidas:**

| Causa | Ajuste (se suma al monto base) |
| --- | --- |
| **Desvío de fondos** | 1,5 millones por cada **año que lleva pendiente** (año actual − año de la causa) |
| **Cripto delito** (única, monto base 5, carátula "estafa") | 30% del monto base |
| **Compleja** | Suma de los ajustes de sus sub-causas |

- Una causa tiene **monto raro** si su monto base es **par** (`even()`).
- Una causa tiene **más de un juez** si tiene 2 o más.

**Lo nuevo y más importante: el `calendario`**

`DesvioDeFondos` depende de **la fecha de hoy**. Eso es un problema para testear: un test que hoy da 1,5 el año que viene da 3. Es un proceso **no determinístico**, justo el caso que en [clase23.md](clase23.md) (sección 4.3) se resolvía con **objetos de resguardo (stubs)**.

La solución:

1. En vez de hacer `new Date()` adentro del cálculo, la causa le pregunta la fecha a **otro objeto**: `calendarioActual.hoy()`.
2. Por defecto ese objeto es `calendario`, que devuelve la fecha real.
3. En un test se le pasa **otro objeto polimórfico** que devuelve una fecha fija:

```wollok
test "un desvío de fondos de hace 2 años suma 3 millones" {
  const calendarioFijo = object { method hoy() = new Date(day = 1, month = 1, year = 2026) }
  const causa = new DesvioDeFondos(
    montoBase = 10,
    caratula = "desvío",
    fechaCausa = new Date(day = 1, month = 6, year = 2024),
    calendarioActual = calendarioFijo
  )
  assert.equals(13, causa.perjuicioEconomico())   // 10 + 2 * 1.5
}
```

Es **polimorfismo + composición** al servicio del testeo: `calendario` y el objeto anónimo entienden `hoy()`, y la causa no nota la diferencia. `Propuesta` hace lo mismo.

**Otras cosas para notar**

- **Se agregó la `caratula`** (en el Tema 1 faltaba). Como es `const property`, todas las causas la tienen y la exponen.
- **`criptoDelito` es un wko** que hereda pasándole **dos** valores a la superclase: `montoBase = 5` y `caratula = "estafa"`. Igual que `obraPublica`: es un único objeto compartido.
- **Cuidado con `montoRaro()` y salir en medios**: después de `salirEnLosMedios()`, el monto base pasa a tener decimales (por ejemplo, 10 → 10.1), y un número con decimales **nunca es par**. Así que salir en medios puede hacer que una causa **deje** de tener monto raro. Puede ser buscado o no; sin las consignas no lo sabemos.
- **Naming**: "monto raro" significa "monto par". En un parcial, el nombre del método tiene que coincidir con el del enunciado, aunque suene raro ([clase26.md](clase26.md), reificar conceptos).

### Funcionario

```wollok
class Funcionario {
  const property causasJudiciales = []
  const property promesas = []
  var property patrimonio
  var property cargo

  method escucharPropuesta(propuesta) {
    if (!cargo.aceptaPropuesta(propuesta)) propuesta.postergar()
    promesas.add(propuesta)
  }

  method llegoAlTopeDeCausas() = causasJudiciales.size() >= 3

  method validarPatrimonio() {
    if (patrimonio <= 0) {
      throw new DomainException(message = "Está incriminando a un chicho que no tiene plata")
    }
  }

  method comerseUnaCausa(causa) {
    self.validarPatrimonio()
    cargo.validarCausa(causa, self)
    causasJudiciales.add(causa)
  }

  method estaTecleando() = causasJudiciales.all({ causa => causa.tieneMasDeUnJuez() })

  method causasRaris() = causasJudiciales.filter({ causa => causa.montoRaro() })

  method caratulasRaris() = self.causasRaris().map({ causa => causa.caratula() })

  method salirEnLosMedios() {
    causasJudiciales.forEach({ causa => causa.aumentarMontoBase(0.1) })
  }
}
```

**Reglas inferidas:**

- Un funcionario **llegó al tope** si tiene 3 causas o más.
- **Está tecleando** si **todas** sus causas tienen más de un juez.
- Sus **causas raris** son las de monto raro, y sus **carátulas raris** son las carátulas de esas causas.

**Para notar:**

- **Reutilización**: `caratulasRaris()` no repite el `filter`: usa `self.causasRaris()` y le aplica un `map`. Evitar ideas repetidas ([clase26.md](clase26.md)).
- **Delegación**: el funcionario no mira la lista de jueces ni el monto de cada causa, le pregunta a la causa (`tieneMasDeUnJuez()`, `montoRaro()`).
- **Ojo con `all` sobre una colección vacía**: devuelve `true`. Un funcionario **sin causas** "está tecleando". Es un caso borde que conviene testear (o consultar si es lo esperado).
- `llegoAlTopeDeCausas()` está en el **funcionario**, aunque sólo lo usa el legislador: es información del funcionario (cuántas causas tiene), así que la pregunta le corresponde a él.

### Cargos

```wollok
object legislador {
  method validarCausa(causa, funcionario) {
    if (funcionario.llegoAlTopeDeCausas()) {
      throw new DomainException(message = "No le entra más una bala a este Congresista.")
    }
  }

  method aceptaPropuesta(propuesta) = propuesta.esDelAnioVigente()
}

object jefeDeGobierno {
  method validarCausa(causa, funcionario) {
    if (causa.montoRaro()) {
      throw new DomainException(message = "La causa tiene un monto raris")
    }
  }

  method aceptaPropuesta(propuesta) = propuesta.esTranqui()
}
```

**Reglas inferidas:**

| Cargo | Para comerse una causa | Acepta tal cual las propuestas… |
| --- | --- | --- |
| **Legislador** | No puede haber llegado al tope (3 causas) | Presentadas **este año** |
| **Jefe de gobierno** | La causa **no** puede tener monto raro | **Tranquis**: descripción de menos de 15 caracteres |

(Además, igual que en el Tema 1, cualquier funcionario necesita patrimonio > 0.)

**Para notar:**

- **Por qué la firma es `validarCausa(causa, funcionario)`**: el legislador sólo usa `funcionario` y el jefe de gobierno sólo usa `causa`. Para que sean **polimórficos** (intercambiables para el `Funcionario`), los dos tienen que recibir **ambos** parámetros, aunque cada uno ignore uno. Pasaba lo mismo con `PoderEjecutivo` en el Tema 1.
- **Los dos cargos son wko**: ninguno necesita datos propios de cada funcionario. Comparar con `PoderEjecutivo` del Tema 1, que era **clase** porque cada uno tenía **su** juez amigo.

### Propuesta

```wollok
class Propuesta {
  const fechaPresentacion
  var fechaCumplimiento
  const descripcion
  const calendarioActual = calendario

  method fechaCumplimiento() = fechaCumplimiento

  method postergar() {
    fechaCumplimiento = fechaCumplimiento.plusYears(4)
  }

  method esDelAnioVigente() = calendarioActual.hoy().year() == fechaPresentacion.year()

  method esTranqui() = descripcion.size() < 15
}
```

- Usa el mismo truco del `calendario` para que `esDelAnioVigente()` se pueda testear.
- **Tiene el mismo problema que en el Tema 1**: `postergar()` **modifica** la propuesta original, así que si dos funcionarios escuchan la misma, la segunda vez llega ya postergada. La alternativa inmutable (`postergada()` que devuelve una propuesta nueva) también aplica acá.

### Tests que convendría escribir (Tema 2)

```wollok
describe "Tema 2" {
  const calendarioFijo = object { method hoy() = new Date(day = 1, month = 1, year = 2026) }

  test "el cripto delito suma el 30% de su monto base" {
    assert.equals(6.5, criptoDelito.perjuicioEconomico())   // 5 + 1.5
  }

  test "una causa con monto base par tiene monto raro" {
    const causa = new DesvioDeFondos(montoBase = 10, caratula = "x",
      fechaCausa = new Date(day = 1, month = 1, year = 2026), calendarioActual = calendarioFijo)
    assert.that(causa.montoRaro())
  }

  test "un legislador con 3 causas no se puede comer otra" {   // valor límite
    const legis = new Funcionario(patrimonio = 100, cargo = legislador)
    3.times({ i => legis.comerseUnaCausa(criptoDelito) })
    assert.throwsExceptionWithMessage(
      "No le entra más una bala a este Congresista.",
      { legis.comerseUnaCausa(criptoDelito) }
    )
  }

  test "un jefe de gobierno no se puede comer una causa de monto raro" {
    const jefe = new Funcionario(patrimonio = 100, cargo = jefeDeGobierno)
    const causaPar = new DesvioDeFondos(montoBase = 10, caratula = "x",
      fechaCausa = new Date(day = 1, month = 1, year = 2026), calendarioActual = calendarioFijo)
    assert.throwsException({ jefe.comerseUnaCausa(causaPar) })
  }

  test "un funcionario sin causas está tecleando" {   // caso borde de all
    assert.that(new Funcionario(patrimonio = 100, cargo = legislador).estaTecleando())
  }

  test "una propuesta de menos de 15 caracteres es tranqui" {
    const propuesta = new Propuesta(descripcion = "bajar precios",
      fechaPresentacion = new Date(), fechaCumplimiento = new Date())
    assert.that(propuesta.esTranqui())   // "bajar precios" tiene 13 caracteres
  }
}
```

> Como en el Tema 1, estos tests los armé yo a partir del código; conviene correrlos en Wollok antes de confiar en ellos.

---

## Checklist para el parcial

Lo que este modelo muestra que se evalúa:

- [ ] **¿Algo puede cambiar durante la vida del objeto?** ("pueden pasar de un puesto a otro", "la familia puede cambiarlo") → **composición / Strategy** con `var property`.
- [ ] **¿Es intrínseco y nunca cambia?** (tipo de causa) → **herencia** está bien.
- [ ] **¿Varios tipos calculan igual salvo una parte?** → **Template Method** con método abstracto.
- [ ] **¿Algo contiene cosas de su mismo tipo?** ("sub-causas de cualquier tipo, incluso complejas") → **Composite**.
- [ ] **¿Cada objeto necesita datos propios?** → **clase**. **¿Es único y sin estado propio?** → **wko**.
- [ ] **¿Una acción no se puede realizar?** → **excepción** con un mensaje claro, antes de producir el efecto.
- [ ] **No repetir lógica**: lo común arriba (superclase o el objeto que delega), lo que varía abajo.
- [ ] **Delegar**: preguntarle a cada objeto lo que sabe (`tieneAlJuez`, `esCorta`) en vez de meterse en su estructura.
- [ ] **Modelar todo lo que nombra el enunciado** (la carátula, los jueces).
- [ ] **Cuidado con mutar objetos compartidos** (wko, propuestas): pensar si conviene que sean inmutables.
- [ ] **¿Algo depende de la fecha de hoy (o de algo no determinístico)?** → que lo pida a otro objeto (`calendario`) para poder reemplazarlo por un stub en los tests.
- [ ] **Si los cargos (estrategias) necesitan datos distintos**, la firma tiene que incluir todo lo que cualquiera necesite (`validarCausa(causa, funcionario)`), para que sigan siendo polimórficos.
- [ ] **Casos borde de colecciones**: `all` sobre vacía da `true`, `max` sobre vacía falla (`maxIfEmpty`).
