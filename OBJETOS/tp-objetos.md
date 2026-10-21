# TP Objetos — La fiesta de Isabella (para la corrección)

> Apunte de estudio del TP grupal (entregas 1 y 2). Por cada parte: **qué pide la consigna**, **cómo lo resolvimos**, y sobre todo **por qué** — que es lo que se pregunta en la defensa. Todo lo que se usa en el código está en los resúmenes de clase ([clase21](clase21.md), [clase22](clase22.md), [clase23](clase23.md), [clase24](clase24.md)).

---

## Índice

| Entrega | Punto | Qué | Quién |
|---|---|---|---|
| 1 | [1 y 2](#entrega-1--puntos-1-y-2-leandro-y-matías) | barril, caterings y sectores | Leandro y Matías |
| 1 | [3](#entrega-1--punto-3-isabella-mi-punto-individual) | Isabella | **mío** |
| 1 | [Grupal](#entrega-1--punto-grupal-polimorfismo) | polimorfismo | todo el grupo |
| 2 | [1](#entrega-2--punto-1-las-fiestas-leandro) | fiestas y sectores como clases | Leandro |
| 2 | [2](#entrega-2--punto-2-los-proveedores-por-sector-matías) | proveedores por sector | Matías |
| 2 | [3](#entrega-2--punto-3-los-clientes-y-la-contratación-mi-punto-individual) | clientes y contratación | **mío** |
| 2 | [Grupal](#entrega-2--punto-grupal-la-agencia-el-fixture-y-el-readme) | agencia, fixture y README | todo el grupo |
| — | [Cierre](#cierre--conceptos-transversales-para-repasar-el-día-antes) | conceptos transversales | — |

Los puntos marcados como **míos** están explicados paso a paso; el resto, más resumido.

---
---

# Entrega 1 — Puntos 1 y 2 (Leandro y Matías)

**Consigna**: modelar el barril (rinde para 40 invitados, cuesta $300 y puede aumentar, se compran enteros), a Doña Rosa ($20 por invitado + bebida), a Pizzas Nico ($350 cada 10 invitados + bebida), a El Bodegón ($900 hasta 25 invitados, $35 por cada adicional, bebida incluida) y los dos sectores de la fiesta.

```wollok
object barril {
    var property precio = 300
    const property rindePara = 40

    method cuantosBarrilesPara(cantidadInvitados) = (cantidadInvitados / self.rindePara()).roundUp()
    method costoPara(cantidadInvitados) = self.cuantosBarrilesPara(cantidadInvitados) * precio
}

object doniaRosa {
    method cuantoCobraServicio(cantidadInvitados) = 20 * cantidadInvitados
    method cuantoCobraTotal(cantidadInvitados) = barril.costoPara(cantidadInvitados) + self.cuantoCobraServicio(cantidadInvitados)
}
```

Lo que hay que saber de estos puntos, porque los usa el mío:

- **Los tres caterings entienden `cuantoCobraTotal(cantidadInvitados)`**. Eso es lo que los hace **polimórficos** para Isabella.
- **El barril es un único objeto compartido** por Doña Rosa y Pizzas Nico (clase 21, "compartir valores"). Si sube el precio, a los dos les aumenta sin tocarlos, porque se lo piden al barril cada vez que calculan.
- `roundUp()` redondea hacia arriba: 42 invitados → 2 barriles; 42 invitados → 5 bloques de pizza.

---

# Entrega 1 — Punto 3: Isabella (MI PUNTO INDIVIDUAL)

**Consigna**: modelar a Isabella con sus ahorros, su presupuesto máximo (60% de sus ahorros), sus invitados (la suma de los dos sectores) y la pregunta de si puede contratar a un catering (si lo que le cobra en total no supera el presupuesto).

## 1. La versión final (después de la corrección)

```wollok
object isabella {
    var ahorros = 2750
    const porcentajeDisponible = 0.6
    var property sectorFamilia = familia
    var property sectorFacu = facultad

    method ahorros() = ahorros

    method ahorrar(monto) {
        ahorros = ahorros + monto
    }

    method presupuestoMaximo() = ahorros * porcentajeDisponible

    method cantidadInvitados() = sectorFamilia.cantidadInvitados() + sectorFacu.cantidadInvitados()

    method puedeContratar(catering) = catering.cuantoCobraTotal(self.cantidadInvitados()) <= self.presupuestoMaximo()
}
```

> En la entrega 2 este objeto **desaparece**: Isabella pasa a ser una instancia de `Cliente`. Pero todas las ideas de acá se mantienen, y la corrección del profe se trasladó tal cual (ver [Paso 2 de la entrega 2](#paso-2-el-estado-de-un-cliente)).

## 2. Por qué es un objeto (wko) y no otra cosa

En la entrega 1 todavía no habíamos visto clases. Isabella es **un concepto único** del negocio, así que es un *well-known object*: se define con `object isabella { ... }` y cualquiera le puede hablar por su nombre.

## 3. El estado: `var`, `const` y los accessors

| Atributo | Por qué así |
|---|---|
| `var ahorros = 2750` | Cambia (cuando ahorra). Arranca en 2750 porque es el **estado inicial** de los casos del enunciado. |
| `const porcentajeDisponible = 0.6` | No cambia nunca. Le damos nombre para que no sea un **número mágico**. |
| `var property sectorFamilia = familia` | Isabella **conoce** a sus sectores mediante una referencia propia. |

**Accessors** (clase 21, módulo 2, punto 4): `var property x` genera solo el getter `x()` y el setter `x(valor)`. Con `const property` solo se genera el getter.

**¿Por qué `ahorros` no es `property`?** Porque no queremos un setter: si cualquiera pudiera hacer `isabella.ahorros(5000)`, la cuenta de cuánto ahorra quedaría **afuera** de Isabella. El principio de la clase 21 es que **el estado lo modifica el propio objeto**. Por eso hay un getter `ahorros()` escrito a mano y una acción `ahorrar(monto)` que hace la suma adentro.

## 4. Por qué referencias propias y no los objetos globales

Se podría haber escrito:

```wollok
// lo que NO conviene
method cantidadInvitados() = familia.cantidadInvitados() + facultad.cantidadInvitados()
```

Funciona, pero la clase 21 (módulo 2, punto 9) dice que usar el wko directo **genera soluciones rígidas**: Isabella quedaría atada para siempre a esos dos sectores. Con una referencia propia (`sectorFamilia`) se puede cambiar el sector con el setter sin tocar el código de Isabella.

## 5. El polimorfismo: el catering entra por parámetro

```wollok
method puedeContratar(catering) = catering.cuantoCobraTotal(self.cantidadInvitados()) <= self.presupuestoMaximo()
```

Es el mismo patrón que `gimenez.pagarA(empleado)` de la clase 21: Isabella le manda `cuantoCobraTotal(...)` **sin saber** si es Doña Rosa, Pizzas Nico o El Bodegón. El método que se ejecuta lo decide el **method lookup** en tiempo de ejecución, según el objeto que llega.

- **Pregunta** (devuelve un valor): `puedeContratar`, `presupuestoMaximo`, `cantidadInvitados`.
- **Acción** (produce un efecto): `ahorrar`.

## 6. La historia del `null`

Wollok 4 **obliga a inicializar** los atributos de un `object`: sin valor, da *"Should initialize singleton attribute"*, y aunque se salteen las validaciones falla al ejecutar con *"attribute uninitialized"*. Mientras Matías no tenía los sectores, los inicializamos en `null`. Cuando los subió, los reemplazamos por `familia` y `facultad`. **`null` no lo vimos en clase**, así que no quedó en la entrega.

## 7. La corrección del profe (me costó un punto)

| Lo que marcó | Cómo quedó | Concepto |
|---|---|---|
| Ahorros arrancaba en 0, y el enunciado dice 2750 | `var ahorros = 2750` | Estado inicial |
| `0.6` es un número mágico | `const porcentajeDisponible = 0.6` | Darle nombre a las constantes |
| No había forma de ahorrar sin hacer la cuenta desde afuera | `method ahorrar(monto)` | El estado lo modifica el propio objeto |

> ⚠️ **Lección para la entrega 2**: revisar siempre los **valores iniciales** contra el enunciado, no dejar **números sueltos** y no dejar que la lógica del objeto se haga desde afuera con setters.

## 8. Preguntas probables

- **¿Qué es `property`?** Un atajo que genera getter y setter (o solo getter, con `const`).
- **¿Por qué `ahorrar` y no un setter?** Porque la cuenta le pertenece a Isabella. Con setter, quien la usa tendría que hacer la suma.
- **¿Qué es el method lookup?** La búsqueda del método a ejecutar, en el objeto que **recibe** el mensaje.
- **¿Qué hay que cambiarle a Isabella si aparece un cuarto catering?** Nada, mientras entienda `cuantoCobraTotal(cantidadInvitados)`.

---

# Entrega 1 — Punto grupal: polimorfismo

Ya está en el README. Lo esencial:

| Polimorfismo | Objetos | Mensaje | Quién lo envía |
|---|---|---|---|
| Caterings | `doniaRosa`, `pizzasNico`, `elBodegon` | `cuantoCobraTotal(cantidadInvitados)` | `isabella` en `puedeContratar` |
| Sectores | `familia`, `facultad` | `cantidadInvitados()` | `isabella` en `cantidadInvitados` |

- **Ventajas**: abstracción (Isabella no sabe el cómo), código más simple, fácil de extender, cambios encapsulados en cada objeto.
- **Cuarto catering**: no hay que cambiar a Isabella.
- **Sube el barril**: a Doña Rosa y Pizzas Nico les aumenta solo, porque comparten el mismo objeto `barril`. A El Bodegón no, porque no usa el barril.

---
---

# Entrega 2 — Punto 1: las fiestas (Leandro)

**Consigna**: modelar los sectores y la fiesta como **clases** (invitados, cantidad de sectores, si es complicada, si es larga) y adaptar los caterings para que cobren por una fiesta.

```wollok
class Fiesta {
    const property sectores = []
    var property duracion
    method cantidadDeSectores() = sectores.size()
    method cantidadInvitados() = sectores.sum({ s => s.invitadosSector() })
    method esComplicada() = self.cantidadDeSectores() > 3
    method esLarga() = duracion > 5
}

class SectorDeMesas {
    var property cantidadMesas
    var property invitadosPorMesa
    method invitadosSector() = cantidadMesas * invitadosPorMesa
}

class SectorDeGrupos { ... method invitadosSector() = ... }
```

Lo que importa para mi punto:

- **`Fiesta` entiende `esLarga()`**: lo uso en `Fiestero`.
- **Los sectores son polimórficos** para la fiesta: los dos entienden `invitadosSector()`, y la fiesta los suma con `sum` sin saber de qué clase es cada uno.
- **Los caterings ahora cobran por una fiesta**: se les agregó `cuantoCobra(fiesta)`, que **reutiliza** el cálculo viejo.

```wollok
method cuantoCobra(fiesta) = self.cuantoCobraTotal(fiesta.cantidadInvitados())
```

Así, `cuantoCobra(fiesta)` es **el mensaje que entienden los diez proveedores**, y es el único que usa mi `Cliente`.

---

# Entrega 2 — Punto 2: los proveedores por sector (Matías)

**Consigna**: salones, organizadores, equipos de mozos y seguridad, **evitando repetir código**.

```wollok
class Proveedor {
    var property precioPorSector

    method costoBase(unaFiesta) = precioPorSector * unaFiesta.cantidadDeSectores()
}

class Salon inherits Proveedor {
    method cuantoCobra(unaFiesta) = self.costoBase(unaFiesta) * unaFiesta.duracion()
}

class Organizador inherits Proveedor {
    const property recargo = 1.2 // 20%

    method cuantoCobra(unaFiesta) = if (unaFiesta.esComplicada()) { self.costoBase(unaFiesta) * recargo } else { self.costoBase(unaFiesta) }
}

class Seguridad inherits Proveedor {
    var property recargo //float ej 25% -> 1.25

    method cuantoCobra(unaFiesta) = self.costoBase(unaFiesta) * (if (unaFiesta.esLarga()) {recargo} else {1})
}

class EquipoDeMozos {
    var property mozosPorSector
    const property mozoPorHora = 50

    method cuantoCobra(unaFiesta) = mozosPorSector * mozoPorHora * unaFiesta.cantidadDeSectores() * unaFiesta.duracion()
}
```

- **`Proveedor` es la superclase** de `Salon`, `Organizador` y `Seguridad`. Comparten el atributo `precioPorSector` y el método `costoBase(fiesta)` ("precio por sector × cantidad de sectores"), escrito una sola vez. Cada subclase define su propio `cuantoCobra(fiesta)` a partir de `costoBase`, usando `self.costoBase(...)`: el lookup lo encuentra en `Proveedor`.
- **`Proveedor` nunca se instancia** con `new`, así que es una clase abstracta "naturalmente" (clase 24, punto 5.1).
- **`EquipoDeMozos` no hereda**. Matías lo justifica en el README con que no comparte `precioPorSector`. Ver la advertencia de abajo.

Y los proveedores concretos son **instancias**, como constantes al final de `example.wlk`:

```wollok
const losTilos = new Salon(precioPorSector = 500)
const marta = new Organizador(precioPorSector = 600)
const eliana = new Organizador(precioPorSector = 1500)
const losPibes = new EquipoDeMozos(mozosPorSector = 2)
const dodain = new EquipoDeMozos(mozosPorSector = 3)
const escudo = new Seguridad(precioPorSector = 100, recargo = 1.1)
const silvina = new Seguridad(precioPorSector = 100, recargo = 1.25)
```

**Una clase por comportamiento, una instancia por proveedor**: Escudo y Silvina calculan igual, solo cambia el recargo → misma clase, dos instancias (clase 24, el ejemplo de los descuentos).

### ⚠️ Lo que encontraron los tests al juntar este punto con el mío

| Problema | Cómo se vio | Arreglo |
|---|---|---|
| Le mandaban a la fiesta `cantidadSectores()` y `cantidadHoras()` | `a Fiesta does not understand cantidadSectores()` | `cantidadDeSectores()` y `duracion()` |
| `Seguridad` no multiplicaba por sectores | Isabella quedaba con $2650 en vez de $2550 | `× unaFiesta.cantidadDeSectores()` |
| `EquipoDeMozos` no multiplicaba por horas | Los Pibes cobraban $200 en vez de $800 | `× unaFiesta.duracion()` |

Es el mejor ejemplo para la pregunta del README sobre los **tests de regresión**.

### ⚠️ Lo que se le puede preguntar a Matías (cómo quedó en la entrega)

- **`EquipoDeMozos` fuera de la jerarquía.** Mozos también hace "algo × cantidad de sectores": su "precio por sector" sería `mozosPorSector × 50`. El corrector puede marcar que ahí quedó código repetido, y la consigna pedía "evitando repetir código". La justificación del README es que no comparte `precioPorSector`.
- **El `if` de `Organizador` repite `self.costoBase(unaFiesta)`** en las dos ramas. Lo que cambia es solo el recargo. El `1.2` sí quedó con nombre (`recargo`), así que no es un número mágico.
- **Los tests de proveedores por sector "con fiestas de resguardo" usan fiestas reales** (`new Fiesta(...)` con `SectorDeMesas`), no objetos anónimos como pide la consigna. Los de caterings sí son anónimos. El grupo decidió dejarlo así porque son tests de Matías.

---

# Entrega 2 — Punto 3: los clientes y la contratación (MI PUNTO INDIVIDUAL)

**Consigna**: modelar a los clientes, la contratación con su validación, cuánto gastaron, si son derrochones (incluido el caso de los fiesteros) y quiénes les trabajaron. Los tests de error deben verificar que el error se lance **y** que el estado del cliente no cambie.

> Esta sección está escrita como una **construcción paso a paso**. Si querés solo el código terminado, está en el **Paso 10**.

---

## Paso 0: partir la consigna en requerimientos

| # | Texto de la consigna | Mensaje |
|---|---|---|
| 1 | "puede contratar un proveedor si lo que le cobra no supera su presupuesto, que es un porcentaje de sus ahorros" | `presupuestoMaximo()`, `puedeContratar(proveedor)` |
| 2 | "contratar debe validar que haya suficiente presupuesto; en caso contrario… ¿qué se debe hacer?" | `contratar(proveedor)` + `validarPresupuesto(proveedor)` |
| 3 | "debe descontar de los ahorros lo que cobró el proveedor" | dentro de `contratar` |
| 4 | "debe quedar registrado, para saber cuánto gastó y cuántos servicios contrató" | `Servicio`, `totalGastado()`, `cantidadServicios()` |
| 5 | "es derrochón si pagó algún servicio de más de $3.000" | `esDerrochon()` |
| 6 | "para Damián y Tomás, fiesteros, solo vale si su fiesta no es larga" | `Fiestero` redefine `esDerrochon()` |
| 7 | "qué proveedores le trabajaron; si contrató dos veces al mismo, aparece una sola vez" | `proveedoresContratados()` |

Y el tip que hay que tener presente todo el tiempo: **"El porcentaje de presupuesto disponible no es el mismo para todos los clientes. Ojo con esto."**

---

## Paso 1: ¿clase, wko u objeto anónimo?

La tabla de la clase 23 (módulo 8, punto 8):

| Opción | Cuándo conviene |
|---|---|
| **Objeto anónimo** | Alcance acotado: un test. |
| **wko** | Un concepto **único** del negocio. |
| **Clase** | **Varios objetos** con el mismo comportamiento, que solo difieren en sus valores. |

Ahora hay cuatro clientes (Isabella, Milena, Damián y Tomás) que hacen **exactamente lo mismo**: tienen ahorros, presupuesto, contratan y registran. Solo cambian los valores. Entonces **`Cliente` es una clase**, y cada cliente es una **instancia**.

### ¿Y el 60% de Isabella? ¿Necesita su propia clase?

**No.** La regla de la clase 24 (módulo 13, punto 5.3, el ejemplo de los descuentos):

> Se usan **clases distintas cuando hay comportamiento distinto**. Si solo cambian los valores, alcanza con **instancias distintas**.

Isabella calcula el presupuesto **igual** que todos (`ahorros * porcentaje`), solo que con otro número. Así que es una instancia de `Cliente` creada con otro valor:

```wollok
new Cliente(ahorros = 2750, fiesta = fiestaIsabella, porcentajeDisponible = 0.6)
new Cliente(ahorros = 2000, fiesta = fiestaMilena)   // usa el 20% por defecto
```

Eso resuelve el "ojo con esto" del tip: el porcentaje es un **atributo con valor por defecto**. La mayoría usa el 20% habitual sin decir nada, e Isabella lo pisa al crearse.

---

## Paso 2: el estado de un cliente

```wollok
class Cliente {
    var ahorros
    const fiesta
    const porcentajeDisponible = 0.2
    const montoDerroche = 3000
    const servicios = []
    ...
}
```

| Atributo | `var` / `const` | Por qué |
|---|---|---|
| `ahorros` | `var` | Cambia al ahorrar y al contratar. **Sin valor por defecto**: cada cliente arranca con los suyos. |
| `fiesta` | `const` | Un cliente organiza una fiesta y no la cambia. **Sin getter**: nadie de afuera se la pide al cliente (clase 22, punto 7.5: no hace falta definir getters si no se necesitan desde afuera). |
| `porcentajeDisponible` | `const`, vale 0.2 | El 20% habitual. Isabella lo pisa con 0.6. |
| `montoDerroche` | `const`, vale 3000 | Para no tener un **número mágico** en `esDerrochon()`. |
| `servicios` | `const`, vale `[]` | La **referencia** no cambia, siempre es la misma lista. Lo que cambia es su **contenido**. |

**`const servicios = []`** es la misma idea que `const misiones = #{}` de Shrek (clase 22, punto 7.1): `const` dice que la referencia apunta siempre a la misma colección, no que la colección no pueda cambiar.

> ⚠️ **Pregunta trampa**: "si `servicios = []` está en la clase, ¿todos los clientes comparten la misma lista?". **No.** El valor inicial se evalúa **para cada instancia** al hacer `new`, así que cada cliente tiene su propia lista. Lo demuestran los tests: lo que contrata Isabella no aparece en Milena.

**Las correcciones de la entrega 1 siguen acá**:

| Corrección | Dónde quedó |
|---|---|
| Ahorros iniciales en 2750 | Al crear a Isabella: `new Cliente(ahorros = 2750, ...)` |
| Sin número mágico | `porcentajeDisponible` (y además `montoDerroche`) |
| `ahorrar(monto)` | Es un método de `Cliente`: lo entienden todos |

Y sigue sin haber setter de `ahorros`: solo cambian por `ahorrar(...)` o `contratar(...)`.

---

## Paso 3: el presupuesto y la pregunta

```wollok
method presupuestoMaximo() = ahorros * porcentajeDisponible

method puedeContratar(proveedor) = proveedor.cuantoCobra(fiesta) <= self.presupuestoMaximo()
```

- **Polimorfismo**: `proveedor` puede ser un catering (wko) o una instancia de `Salon`, `Seguridad`, etc. El cliente no lo distingue: solo le pide `cuantoCobra(fiesta)`. Es el "Comprador" de la clase 23 (módulo 8, punto 7): aparecen proveedores nuevos **sin tocar al cliente**.
- **Le pasa su propia fiesta**: el cliente conoce su fiesta y se la da al proveedor para que calcule. Ni el cliente sabe calcular lo que cobra un salón, ni el salón tiene que conocer al cliente.
- **Es una pregunta**: siempre devuelve un booleano, nunca lanza error (clase 22, módulo 10, punto 10.2). El resultado es la comparación `<=`, y una comparación siempre da `true` o `false`:
  - `isabella.puedeContratar(escudo)` → 200 ≤ 1650 → `true`
  - `isabella.puedeContratar(losTilos)` → 4000 ≤ 1650 → `false`

  Si no le alcanza, devuelve `false`: **no** lanza error. Si mezclara `true` con una excepción, quien la usa no sabría qué esperar. Ese booleano lo usan tres:

  | Quién | Para qué |
  |---|---|
  | `validarPresupuesto` | si da `false` (`!self.puedeContratar(...)`), lanza el error |
  | la agencia | en el `filter`, se queda con los proveedores que dan `true` |
  | los tests | `assert.that(...)` si se espera `true`, `assert.notThat(...)` si se espera `false` |
- **El presupuesto se calcula, no se guarda**: si se guardara como atributo, habría que acordarse de actualizarlo cada vez que cambian los ahorros. Calculándolo, **siempre** está al día. Por eso "contratar actualiza el presupuesto" no requiere hacer nada extra.

---

## Paso 4: contratar — la validación

La consigna pregunta: *"debe validar que haya suficiente presupuesto; en caso contrario… ¿qué se debe hacer?"*

Las alternativas de la clase 22 (módulo 10, punto 5):

| Alternativa | Qué pasa | ¿Sirve? |
|---|---|---|
| **Mirar a un costado** (`if` sin `else`) | No contrata, pero no avisa: quien llamó no se entera de que salió mal | ❌ La peor |
| **Códigos de error** (`return -1`) | Obliga a quien llama a chequear el resultado; números mágicos | ❌ |
| **Excepción** | Corta el flujo y avisa con un mensaje claro | ✅ |

La clase 22 tiene literalmente este caso en el punto 10.1, "To throw or not to throw": *"Una validación de negocio real (presupuesto insuficiente para contratar) sí amerita una excepción"*.

```wollok
method contratar(proveedor) {
    self.validarPresupuesto(proveedor)
    const monto = proveedor.cuantoCobra(fiesta)
    ahorros = ahorros - monto
    servicios.add(new Servicio(proveedor = proveedor, monto = monto))
}

method validarPresupuesto(proveedor) {
    if (!self.puedeContratar(proveedor)) {
        self.error("El presupuesto no alcanza para contratar al proveedor")
    }
}
```

**Las decisiones, una por una:**

1. **`self.error(...)`** lanza una excepción con ese mensaje (clase 22, módulo 10, punto 6). Es una **excepción de usuario/dominio**: la entiende el usuario y la puede corregir ahorrando más.
2. **La validación va primero.** Si falla, la excepción corta el método **antes** de descontar y de registrar. Por eso el estado del cliente no cambia, que es justo lo que la consigna pide verificar en los tests.
3. **La validación tiene su propio método** (`validarPresupuesto`), como `validarMontoPositivo` del monedero: `contratar` se lee como la consigna (validar, descontar, registrar).
4. **Reutiliza la pregunta** `puedeContratar`, en lugar de repetir la cuenta. Es la técnica de la clase 22 (punto 10.3) de **separar la pregunta de la acción**, que acá sí hace falta porque necesitamos las dos: la pregunta la usa la agencia, y la acción, el cliente al contratar.
5. **El cliente no atrapa la excepción** (no hay `try/catch`). La clase 22 (punto 7): los objetos de dominio **lanzan, no atrapan**. Solo la atrapan los tests (o una interfaz de usuario). Está explicado en detalle en el [Cierre, sección 7](#lanzar-y-atrapar-quién-hace-cada-cosa-y-por-qué).

> ⚠️ Fijate que **no** es "mirar a un costado": si no alcanza, **avisa**. Y fijate que la pregunta `puedeContratar` **nunca** lanza error, siempre devuelve `true` o `false`. El error es solo de la **acción**.

---

## Paso 5: registrar — la clase `Servicio`

```wollok
class Servicio {
    const property proveedor
    const property monto
}
```

**¿Por qué no guardar directamente los proveedores en una lista?** Porque la consigna pide saber **cuánto gastó**, y para eso hay que guardar **cuánto se pagó**. Y eso no se puede recalcular después:

> Si Isabella contrata a Doña Rosa por $1.440 y después el barril sube a $500, Doña Rosa pasaría a cobrar $1.840. Pero Isabella **pagó** $1.440. El servicio guarda el monto **del momento de la contratación**.

Por eso el tip dice *"Modelar el servicio que se contrata"*: es un objeto con dos datos, a quién se contrató y cuánto se pagó.

**¿Por qué todo `const`?** Un servicio ya contratado **no cambia**: es un **objeto inmutable**, como los *value objects* de la clase 24 (módulo 14). No tiene setters ni efecto colateral.

**¿Por qué una lista (`[]`) y no un conjunto (`#{}`)?** Porque si Milena contrata **dos veces** a Silvina, son **dos servicios** distintos, aunque tengan el mismo proveedor y el mismo monto. Un conjunto no admite repetidos y podría quedarse con uno solo. Ver [Paso 9](#paso-9-qué-proveedores-le-trabajaron--igualdad-e-identidad).

---

## Paso 6: cuánto gastó y cuántos servicios tuvo

```wollok
method cantidadServicios() = servicios.size()

method totalGastado() = servicios.sum({ servicio => servicio.monto() })
```

Igual que `cantidadDeMisiones()` y `totalPuntosDeRecompensa()` de Shrek (clase 22, punto 7). `sum` recibe un **bloque** que dice qué número sumar de cada servicio.

**¿Y si no contrató nada?** `size()` da 0 y `sum` da 0. **No es un error**: la clase 22 (punto 10.1) lo dice explícitamente, *"empezar sin servicios contratados es un estado normal, no un error"*. Por eso no hay ningún `if` ni excepción para la lista vacía.

---

## Paso 7: si es derrochón

```wollok
method esDerrochon() = !servicios.filter({ servicio => servicio.monto() > montoDerroche }).isEmpty()
```

Leelo de adentro hacia afuera:

1. `servicios.filter({ ... })` → los servicios de **más** de $3.000 (`>`, no `>=`: "más de").
2. `.isEmpty()` → ¿esa lista está vacía?
3. `!` → derrochón es que **no** esté vacía, o sea, que haya **alguno**.

- `filter` **no modifica** la lista original (clase 22, punto 7.7): devuelve una nueva. Preguntar si es derrochón no le borra servicios al cliente.
- `montoDerroche` en lugar de `3000` directo: sin números mágicos.

> **¿Por qué no `servicios.any(...)`?** Hace lo mismo, pero `any` **no está en los resúmenes de clase**. Decidimos usar solo lo que vimos, para poder defenderlo. `filter`, `isEmpty` y `!` sí están en la clase 22.

---

## Paso 8: los fiesteros — herencia

La consigna: *"Para Damián y Tomás, que son fiesteros, eso solo vale si su fiesta no es larga: gastar de más en una fiesta larga no es derrochar."*

### ¿Por qué una subclase?

Damián y Tomás hacen **todo** igual que un cliente, **menos una cosa**: cómo deciden si son derrochones. Eso **sí es comportamiento distinto** (no un valor), así que hace falta otra clase. Y como comparten todo lo demás, no queremos copiarlo: **heredamos**.

```wollok
class Fiestero inherits Cliente {
    override method esDerrochon() = !fiesta.esLarga() && super()
}
```

| Palabra | Qué hace | Clase 24 |
|---|---|---|
| `inherits Cliente` | `Fiestero` tiene todos los atributos y métodos de `Cliente` | 2 |
| `override` | Redefine `esDerrochon()`: para un fiestero, este método **pisa** al de `Cliente` | 3.3 |
| `super()` | Ejecuta el `esDerrochon()` **de `Cliente`**, para no repetir la regla de los $3.000 | 3.4 |
| `fiesta` | El fiestero usa directamente el atributo que heredó | 6 |

Se lee igual que la consigna: **"no es larga la fiesta, y además (`&&`) es derrochón como cualquier cliente"**.

### ¿Por qué `super()` y no `self.esDerrochon()`?

Si dentro de `esDerrochon()` de `Fiestero` pusiéramos `self.esDerrochon()`, el lookup empezaría otra vez en `Fiestero`, encontraría el mismo método y lo volvería a llamar: **loop infinito** (clase 24, punto 3.4). `super` saltea `Fiestero` y empieza a buscar en `Cliente`.

### ¿Por qué `super()` va sin argumentos? ¿Cuándo se le pasa algo?

`super(...)` llama **al mismo método** de la superclase, y los paréntesis son para pasarle **los argumentos que ese método necesita**. `super` no lleva nombre de método: siempre llama al método **que estás redefiniendo**, el mismo en el que está escrito. Por eso solo se escriben los argumentos.

**El ejemplo del Petrel (clase 24, punto 3.2)**:

```wollok
class Ave {
    method volar(kilometros) {
        energia = energia - (kilometros + 10)
    }
}

class Petrel inherits Ave {
    override method volar(kilometros) {
        super(kilometros)                                   // = "el volar de Ave, con estos kilómetros"
        kilometrosVolados = kilometrosVolados + kilometros
    }
}
```

El `volar` de `Ave` **necesita** saber cuántos kilómetros vuela para descontar la energía. Con `super()` sin argumento, el `volar` de `Ave` no sabría cuánto restar, y Wollok daría error porque falta un parámetro.

**Por qué nosotros no le pasamos nada**:

```wollok
class Cliente {
    method esDerrochon() = !servicios.filter({ servicio => servicio.monto() > montoDerroche }).isEmpty()
}

class Fiestero inherits Cliente {
    override method esDerrochon() = !fiesta.esLarga() && super()
}
```

`esDerrochon()` **no recibe parámetros**: para saber si el cliente es derrochón, le alcanza con sus propios atributos (`servicios` y `montoDerroche`). Entonces `super()` va vacío: no hay nada que pasarle.

**La regla**: `super` lleva **la misma cantidad de argumentos** que el método que estás redefiniendo.

| Método redefinido | Llamada a `super` |
|---|---|
| `volar(kilometros)` (Petrel, clase 24) | `super(kilometros)` |
| `comer(gramos)` (Torcaza, clase 24) | `super(gramos)` |
| `esDerrochon()` (Fiestero, nuestro TP) | `super()` |

Normalmente se le pasa **el mismo valor** que se recibió, pero podría ser otro: `super(kilometros * 2)` haría que el ave gaste energía como si volara el doble. Lo importante es que la **cantidad** coincida.

> **En la defensa**: "`super` llama al mismo método en la superclase, y entre paréntesis van los argumentos que ese método necesita. `esDerrochon()` no recibe parámetros, así que `super()` va vacío. En el Petrel, en cambio, `volar` recibe los kilómetros, y por eso es `super(kilometros)`."

### Method lookup, paso a paso

**`tomas.esDerrochon()`** (contrató a Los Tilos por $4.000, fiesta de 4 horas):

1. Se busca `esDerrochon` en `Fiestero` (la clase de `tomas`) → **está** (redefinido).
2. `fiesta.esLarga()` → 4 > 5 → `false` → `!false` = `true`.
3. `super()` → se busca `esDerrochon` **a partir de `Cliente`** → lo encuentra → ¿algún servicio de más de $3.000? → sí, el de $4.000 → `true`.
4. `true && true` → **es derrochón**.

**`damian.esDerrochon()`** (contrató a Los Tilos por $16.000, fiesta de 8 horas):

1. Se busca en `Fiestero` → está.
2. `fiesta.esLarga()` → 8 > 5 → `true` → `!true` = `false`.
3. `false && ...` → **no es derrochón**, aunque haya gastado $16.000.

**`damian.contratar(losTilos)`**: se busca `contratar` en `Fiestero` → **no está** → sube a `Cliente` → **lo encuentra** y lo ejecuta con el estado de Damián. Por eso los fiesteros saben contratar sin haberlo escrito.

### ¿"Fiestero" es un buen criterio para subclasificar?

La clase 24 (punto 5.4) advierte: **no subclasificar por cosas temporales** (joven/adulto, feliz/aburrido), porque una instancia **no puede cambiar de clase**. "Fiestero" está planteado en la consigna como algo **de la persona** ("Tomás es tan fiestero como Damián"), no como un estado que va y viene. Por eso es un criterio válido.

> 🔗 **Comparación con Isabella**: Isabella tiene un **valor** distinto (60%) → instancia. Los fiesteros tienen un **comportamiento** distinto (`esDerrochon`) → subclase. Es la regla de la clase 24 aplicada dos veces en el mismo punto.

### ¿Herencia o composición? Por qué hoy está bien que sea herencia

La clase 24 (punto 5.2) compara las dos formas de reutilizar comportamiento:

| | Herencia | Composición |
|---|---|---|
| Relación | entre **clases** (fija) | entre **instancias**: un objeto **conoce** a otro y le manda mensajes |
| Idea | "**es un**" | "**tiene un**" |
| ¿Se puede cambiar después? | **no**: una instancia no cambia de clase | **sí**: alcanza con cambiar la referencia |

Para `Fiestero` elegimos herencia porque se cumplen las tres condiciones que la hacen la opción natural:

1. **"Es un"**: un fiestero **es un** cliente, con todo lo que eso implica (ahorros, fiesta, contratar, servicios). No "tiene" un cliente adentro.
2. **Es algo intrínseco, no temporal**: la consigna lo presenta como una característica de la persona ("Damián y Tomás, que son fiesteros", "Tomás es tan fiestero como Damián"). En ningún requerimiento un cliente empieza o deja de ser fiestero. Es lo que pide la buena práctica de la clase 24 (punto 5.4): que el objeto pertenezca a **una sola clase durante toda su vida**.
3. **Cambia un solo comportamiento, y el resto se comparte entero**: `Fiestero` redefine `esDerrochon()` y hereda todo lo demás sin escribir nada. Con `super()` reutiliza hasta la parte de la regla que no cambia.

Además, la entrega evalúa "clases y herencia", y el README pide justificar dónde la usaron.

### ¿Cuándo pasaría a convenir composición?

**Si ser fiestero pudiera cambiar durante la vida del cliente.** Por ejemplo, si la consigna dijera *"un cliente pasa a ser fiestero cuando contrata más de 5 servicios"* o *"Damián decide calmarse y deja de ser fiestero"*.

Con herencia, eso **no se puede**: `damian` fue creado con `new Fiestero(...)` y va a ser un `Fiestero` para siempre. Es el mismo problema que la clase 24 (punto 5.4) muestra con el alumno que termina la cursada y pasa a ser ayudante: habría que crear un cliente nuevo y reemplazar todas las referencias al viejo (en los tests, en la agencia…), con el riesgo de perder sus servicios.

Con composición, el cliente **tiene** un objeto que decide cómo se evalúa el derroche, y ese objeto se puede cambiar:

```wollok
object criterioComun {
    method esDerrochon(cliente) = cliente.pagoAlgunServicioCaro()
}

object criterioFiestero {
    method esDerrochon(cliente) = !cliente.fiesta().esLarga() && cliente.pagoAlgunServicioCaro()
}

class Cliente {
    var criterioDerroche = criterioComun
    ...
    method criterioDerroche(nuevoCriterio) { criterioDerroche = nuevoCriterio }

    method pagoAlgunServicioCaro() = !servicios.filter({ servicio => servicio.monto() > montoDerroche }).isEmpty()

    method esDerrochon() = criterioDerroche.esDerrochon(self)
}
```

```wollok
damian.criterioDerroche(criterioComun)   // Damián deja de ser fiestero: es el mismo objeto, con sus servicios
```

Cómo funciona:

- **Es polimorfismo entre dos wkos**: `criterioComun` y `criterioFiestero` entienden el mismo mensaje `esDerrochon(cliente)`, y el cliente no sabe cuál tiene. Es lo mismo que Isabella con sus caterings en la entrega 1.
- **El cliente delega**: ya no decide él, se lo pregunta a su criterio pasándose a sí mismo (`self`).

Lo que cuesta:

| Costo | Por qué |
|---|---|
| Más objetos | Dos criterios para una sola regla que cambia. |
| Mensajes públicos extra | `pagoAlgunServicioCaro()` y el getter `fiesta()` tienen que existir para que los criterios puedan preguntarle al cliente. Con herencia no hacen falta: `Fiestero` usa directo el atributo `fiesta` que hereda. |
| Se pierde `super()` | Cada criterio tiene que volver a pedir "pagó algo caro"; ya no hay una superclase de la que reutilizarlo. |

**Conclusión**: la composición paga esos costos a cambio de **flexibilidad para cambiar en tiempo de ejecución**. Hoy la consigna no pide esa flexibilidad, así que sería complicar el modelo sin necesidad. Si la pidiera, convendría cambiar.

> **Otras señales de que convendría composición** (la clase 24, punto 5.2):
> - **Si hubiera más de un criterio para clasificar clientes.** Por ejemplo, fiestero/no fiestero **y además** VIP/común con otra regla. Con herencia simple hay un solo "tiro": no se puede heredar de `Fiestero` y de `ClienteVIP` a la vez, y habría que crear `FiesteroVIP`, `FiesteroComun`… Con composición, cada criterio es una referencia distinta.
> - **Si solo quisiéramos reutilizar una parte**: la herencia trae **toda** la interfaz de la superclase. Es el ejemplo de la pila, que no conviene que **sea** una lista sino que **tenga** una.

---

## Paso 9: qué proveedores le trabajaron — igualdad e identidad

```wollok
method proveedoresContratados() {
    const proveedores = #{}
    servicios.forEach({ servicio => proveedores.add(servicio.proveedor()) })
    return proveedores
}
```

1. Arma un **conjunto** vacío (`#{}`).
2. Recorre los servicios con `forEach` y agrega el proveedor de cada uno con `add`.
3. Como un conjunto **no admite repetidos** (clase 22, punto 6.4), si Silvina aparece en dos servicios, queda **una sola vez**.

### ¿Cómo sabe el conjunto que "es el mismo"?

Acá entra **igualdad e identidad** (clase 24, módulo 14):

- **Identidad (`===`)**: ¿es **el mismo objeto**?
- **Igualdad (`==`)**: ¿**representan lo mismo**? Por defecto, `==` funciona igual que la identidad, y se puede redefinir.

El conjunto usa `==`, y los proveedores no lo redefinen, así que compara **identidad**. Las dos veces que Milena contrata a Silvina, el servicio guarda una referencia al **mismo objeto** `silvina` (la constante global). Mismo objeto → para el conjunto es repetido → queda uno.

### ¿Y por qué los servicios **no** se colapsan?

Porque cada `contratar` hace un `new Servicio(...)`, que crea un objeto **nuevo**. Los dos servicios de Silvina son dos objetos **distintos** (`===` da falso), y como `Servicio` no redefine `==`, tampoco son iguales. Además, están en una **lista**, que admite repetidos.

| Colección | Elementos | ¿Repetidos? | Milena con 2× Silvina y 2× Escudo |
|---|---|---|---|
| `servicios` (lista) | 4 objetos `Servicio` distintos | se cuentan todos | `cantidadServicios()` → **4** |
| `proveedoresContratados()` (conjunto) | `silvina`, `silvina`, `escudo`, `escudo` | se colapsan | `#{silvina, escudo}` → **2** |

> ⚠️ **Si redefiniéramos `==` en `Servicio`** (por ejemplo, "iguales si tienen el mismo proveedor y monto") y los guardáramos en un conjunto, los dos de Silvina se colapsarían en uno, y `cantidadServicios()` daría **2 en lugar de 4**. Por eso a `Servicio` no le redefinimos `==`: cada contratación es un hecho distinto.

### ¿Por qué no guardar un conjunto de proveedores aparte?

La idea de fondo: **es mejor calcular un dato a partir de lo que ya tenés que guardarlo dos veces**.

#### Las dos opciones

**Opción A (la que usamos): calcularlo cada vez.** El cliente guarda **solo** los servicios. Cuando le preguntás qué proveedores le trabajaron, recorre los servicios y los arma en ese momento:

```wollok
const servicios = []

method contratar(proveedor) {
    ...
    servicios.add(new Servicio(proveedor = proveedor, monto = monto))
}

method proveedoresContratados() {
    const proveedores = #{}
    servicios.forEach({ servicio => proveedores.add(servicio.proveedor()) })
    return proveedores
}
```

**Opción B (la que descartamos): guardarlo aparte.** El cliente guarda **dos** colecciones y en cada contratación agrega en las dos:

```wollok
const servicios = []
const proveedores = #{}          // ← segunda colección

method contratar(proveedor) {
    ...
    servicios.add(new Servicio(proveedor = proveedor, monto = monto))
    proveedores.add(proveedor)   // ← hay que acordarse de esto también
}

method proveedoresContratados() = proveedores
```

Las dos funcionan y hoy dan el mismo resultado.

#### Por qué "guardar dos veces la misma información"

En la opción B, después de que Milena contrata dos veces a Silvina y dos veces a Escudo:

```
servicios   = [Servicio(silvina, 250), Servicio(silvina, 250), Servicio(escudo, 220), Servicio(escudo, 220)]
proveedores = #{silvina, escudo}
```

Que Silvina y Escudo le trabajaron **ya está** dentro de `servicios`: cada servicio sabe su proveedor. La colección `proveedores` repite algo que ya se sabía.

#### El problema: "mantenerlas sincronizadas"

Mientras las dos colecciones digan lo mismo, no pasa nada. El riesgo es que **algún día dejen de decir lo mismo**, y eso pasa cuando alguien toca una y se olvida de la otra.

Por ejemplo, si una entrega 3 agregara **"cancelar un servicio"**:

```wollok
method cancelar(servicio) {
    servicios.remove(servicio)
    // ¿y proveedores? ¡me olvidé de sacarlo de acá!
}
```

Si Milena cancela los dos servicios de Silvina:

```
servicios   = [Servicio(escudo, 220), Servicio(escudo, 220)]
proveedores = #{silvina, escudo}     ← sigue diciendo que Silvina le trabajó
```

Ahora el cliente **se contradice**: según `servicios`, Silvina no le trabajó, y según `proveedores`, sí. Eso es lo que significa que las colecciones estén **desincronizadas**.

Y no alcanza con sacarla de `proveedores` sin pensar: si hubiera cancelado **uno solo** de los dos servicios de Silvina, ella **sí** tendría que seguir. Mantener bien la segunda colección obliga a pensar cada caso.

**Con la opción A esto no puede pasar.** Hay una sola fuente de verdad, `servicios`. Si se cancela un servicio, la próxima vez que se pregunte `proveedoresContratados()` se recalcula a partir de lo que hay, y siempre da bien. **No hay nada que olvidarse de actualizar.**

#### La comparación con la clase 21

En la clase 21 (módulo 2, punto 7) estaba Juan, que tiene a Firulais como mascota, y Firulais, que tiene a Juan como dueño:

```wollok
juan.mascota(firulais)
firulais.duenio(juan)
```

La clase advierte que, si guardás la relación **en los dos lados**, tenés que mantenerlas **sincronizadas**. Si alguien cambia la mascota de Juan y no actualiza el dueño de Firulais, Juan dice "mi mascota es otra" y Firulais sigue diciendo "mi dueño es Juan". Es el mismo problema: **el mismo dato guardado en dos lugares**, que se puede contradecir.

#### En resumen

| | Opción A (calcular) | Opción B (guardar aparte) |
|---|---|---|
| Qué guarda el cliente | solo `servicios` | `servicios` **y** `proveedores` |
| Al contratar | agrega en un lugar | agrega en **dos** lugares |
| Riesgo | ninguno: siempre se calcula de lo que hay | que las dos digan cosas distintas si alguien se olvida de actualizar una |

Lo mismo hicimos con el presupuesto: no lo guardamos, lo calculamos (`ahorros * porcentajeDisponible`). Si lo guardáramos, habría que acordarse de actualizarlo cada vez que cambian los ahorros (ver [Paso 3](#paso-3-el-presupuesto-y-la-pregunta)).

> **En la defensa**: "No guardé los proveedores aparte porque ya están en los servicios. Si los guardara dos veces, tendría que mantener las dos colecciones de acuerdo, y si un día alguien modifica una y se olvida de la otra, el cliente se contradice. Calculándolos a partir de los servicios, siempre dan bien."

> **¿Por qué no `servicios.map(...).asSet()`?** Hace lo mismo en una línea, pero `asSet` **no está en los resúmenes**. Lo reemplazamos por `#{}` + `forEach` + `add`, que sí están (clases 22 y 24).

---

## Paso 10: la versión final

```wollok
class Cliente {
    var ahorros
    const fiesta
    const porcentajeDisponible = 0.2
    const montoDerroche = 3000
    const servicios = []

    method ahorros() = ahorros

    method ahorrar(monto) {
        ahorros = ahorros + monto
    }

    method presupuestoMaximo() = ahorros * porcentajeDisponible

    method puedeContratar(proveedor) = proveedor.cuantoCobra(fiesta) <= self.presupuestoMaximo()

    method contratar(proveedor) {
        self.validarPresupuesto(proveedor)
        const monto = proveedor.cuantoCobra(fiesta)
        ahorros = ahorros - monto
        servicios.add(new Servicio(proveedor = proveedor, monto = monto))
    }

    method validarPresupuesto(proveedor) {
        if (!self.puedeContratar(proveedor)) {
            self.error("El presupuesto no alcanza para contratar al proveedor")
        }
    }

    method cantidadServicios() = servicios.size()

    method totalGastado() = servicios.sum({ servicio => servicio.monto() })

    method esDerrochon() = !servicios.filter({ servicio => servicio.monto() > montoDerroche }).isEmpty()

    method proveedoresContratados() {
        const proveedores = #{}
        servicios.forEach({ servicio => proveedores.add(servicio.proveedor()) })
        return proveedores
    }
}

class Fiestero inherits Cliente {
    override method esDerrochon() = !fiesta.esLarga() && super()
}

class Servicio {
    const property proveedor
    const property monto
}
```

### Diagrama de clases

```
┌──────────────────────────────┐          ┌───────────────────┐
│           Cliente            │ servicios│     Servicio      │
├──────────────────────────────┤─────────▶├───────────────────┤
│ ahorros                      │    *     │ proveedor         │
│ fiesta                       │          │ monto             │
│ porcentajeDisponible = 0.2   │          └───────────────────┘
│ montoDerroche = 3000         │
│ servicios                    │
├──────────────────────────────┤
│ ahorrar(monto)               │
│ presupuestoMaximo()          │
│ puedeContratar(proveedor)    │
│ contratar(proveedor)         │
│ validarPresupuesto(proveedor)│
│ cantidadServicios()          │
│ totalGastado()               │
│ esDerrochon()                │
│ proveedoresContratados()     │
└──────────────▲───────────────┘
               │
┌──────────────┴───────────────┐
│           Fiestero           │
├──────────────────────────────┤
│ esDerrochon()   (override)   │
└──────────────────────────────┘
```

---

## Paso 11: traza de los casos de la consigna

### Isabella contrata a Escudo

| Paso | Cuenta | Resultado |
|---|---|---|
| Presupuesto inicial | 2750 × 0.6 | $1.650 |
| Escudo cobra | $100 × 2 sectores (fiesta de 4 horas, no es larga: sin recargo) | $200 |
| `puedeContratar(escudo)` | 200 ≤ 1650 | `true` → no hay error |
| Descuenta | 2750 − 200 | ahorros = **$2.550** |
| Presupuesto nuevo | 2550 × 0.6 | **$1.530** ✓ |

### Isabella contrata a Escudo y a El Bodegón

- Escudo: $200 → quedan $2.550, presupuesto $1.530.
- El Bodegón: 42 invitados → $900 + (42 − 25) × $35 = **$1.495** ≤ $1.530 ✓.
- **2 servicios**, gastó **$1.695** ✓. Ninguno pasa los $3.000 → **no es derrochona** ✓.

### Isabella intenta contratar a Los Tilos

- Los Tilos: $500 × 2 sectores × 4 horas = **$4.000** > $1.650 → **error**.
- La excepción corta `contratar` antes de descontar y de registrar → ahorros siguen en **$2.750** y **0 servicios** ✓.

### Isabella ahorra $5.000 y contrata a Los Tilos

- Ahorros: $7.750 → presupuesto $4.650 ≥ $4.000 ✓.
- Pagó $4.000 > $3.000 → **es derrochona** ✓.

### Damián y Tomás contratan a Los Tilos

| | Damián | Tomás |
|---|---|---|
| Fiesta | 4 sectores, 8 horas | 2 sectores, 4 horas (la de Isabella) |
| Los Tilos cobra | 500 × 4 × 8 = **$16.000** | 500 × 2 × 4 = **$4.000** |
| Presupuesto (250.000 × 0.2) | $50.000 ✓ | $50.000 ✓ |
| ¿Fiesta larga? | sí | no |
| ¿Derrochón? | **no** (la fiesta es larga) | **sí** |

### Milena ahorra $100.000

- Presupuesto: $102.000 × 0.2 = $20.400.
- Los Tilos: 500 × 2 × 6 = $6.000 → pagó más de $3.000 → **derrochona** ✓.
- Dos veces Silvina ($250 c/u) y dos veces Escudo ($220 c/u) → **4 servicios**, proveedores `#{silvina, escudo}` ✓.

---

## Paso 12: los tests

Están en `tests.wtest`, en dos `describe`: **"Punto 3: Contrataciones y derrochones"** y **"Regresión de la entrega 1: Isabella (punto 3)"**. Los clientes los toman del objeto `fixtureCompartido` (ver [punto grupal](#entrega-2--punto-grupal-la-agencia-el-fixture-y-el-readme)):

```wollok
describe "Punto 3: Contrataciones y derrochones" {
    const isabella = fixtureCompartido.isabella()
    const milena = fixtureCompartido.milena()
    const damian = fixtureCompartido.damian()
    const tomas = fixtureCompartido.tomas()
    ...
}
```

### El test de error

```wollok
test "Isabella no puede contratar a Los Tilos porque no le alcanza el presupuesto y su estado no cambia" {
    assert.throwsExceptionWithMessage("El presupuesto no alcanza para contratar al proveedor", { isabella.contratar(losTilos) })
    assert.equals(2750, isabella.ahorros())
    assert.equals(0, isabella.cantidadServicios())
}
```

- **El código que tiene que fallar va dentro de un bloque** `{ ... }` (clase 23, módulo 11, punto 3.1). Si tira la excepción esperada, el `assert` la atrapa y el test sigue; si **no** la tira, el test **falla**.
- **`throwsExceptionWithMessage`** exige que el error tenga **ese** mensaje (restricción media). Más seguro que `throwsException`, que acepta **cualquier** error (incluso un "does not understand" por un bug).
- **Los dos `assert` de después** son lo que pide la consigna: *"verificar que el error se lance **y** que el estado del cliente no cambie"*.

> ✅ **Lo comprobamos**: si se comenta la línea `self.validarPresupuesto(proveedor)`, este test **falla**. Es la prueba de que el test realmente verifica la validación (clase 23, punto 3.1).

### Cada test parte del estado inicial

La consigna dice *"Cada caso parte del estado inicial descripto para los clientes"*. No hay que hacer nada para lograrlo: Wollok **reinicia el ambiente y vuelve a crear el fixture antes de cada test** (clase 23, punto 2.5). Si un test hace que Isabella contrate a Escudo, el siguiente la vuelve a tener con $2.750.

### Regresión de Isabella de la entrega 1

| Caso | Esperado | Por qué |
|---|---|---|
| Presupuesto máximo inicial | $1.650 | 2750 × 0.6 |
| Puede contratar a Doña Rosa y a El Bodegón, no a Pizzas Nico | sí, sí, no | $1.440 y $1.495 ≤ $1.650; $2.350 no |
| El barril pasa a $500 | ya no puede contratar a Doña Rosa | $840 + 2 × $500 = $1.840 > $1.650 |
| Ahorra $2.000 más | puede contratar a Pizzas Nico | $4.750 × 0.6 = $2.850 ≥ $2.350 |

El caso del barril funciona **solo** porque el barril es un objeto **compartido**: lo cambia el test, y Doña Rosa lo ve al calcular.

---

## Preguntas probables de la corrección

**¿Por qué `Cliente` es una clase y no hay un `object isabella`?**
Porque hay cuatro clientes con el mismo comportamiento; solo cambian sus valores. La clase define comportamiento y estructura una vez, y cada cliente es una instancia con su propio estado.

**¿Cómo resolviste que el porcentaje no sea igual para todos?**
Es un atributo `const porcentajeDisponible = 0.2` con valor por defecto. Isabella se crea con `porcentajeDisponible = 0.6`. No hace falta una subclase porque no cambia el comportamiento, solo un valor.

**¿Qué pasa si el cliente no tiene presupuesto para contratar?**
Se lanza una excepción con `self.error(...)`. Es una validación de negocio: no se puede "mirar a un costado" porque quien contrata no se enteraría, y los códigos de error obligan a chequear el resultado.

**¿Cómo garantizás que el estado no cambie si falla?**
La validación es lo primero que se ejecuta en `contratar`. La excepción corta el método antes de descontar y registrar.

**¿Por qué el cliente no atrapa la excepción?**
Porque un objeto de dominio no tiene nada sensato que hacer con ella. La atrapan los tests (para verificarla) o una interfaz de usuario.

**¿Por qué existe la clase `Servicio`?**
Para registrar **cuánto se pagó** en el momento de contratar. Si después sube el barril, lo que el cliente ya gastó no tiene que cambiar.

**¿Por qué `Fiestero` es una subclase?**
Porque cambia un comportamiento (`esDerrochon`). Hereda todo lo demás de `Cliente`, redefine ese método con `override` y reutiliza la regla general con `super()`.

**¿Qué hace `super()`?**
Hace que el method lookup **saltee** la clase donde está escrito (`Fiestero`) y empiece a buscar en la superclase (`Cliente`). El receptor sigue siendo el mismo objeto.

**¿Por qué `super()` va sin nada entre paréntesis?**
Porque `esDerrochon()` no recibe parámetros. `super` lleva los mismos argumentos que el método que se redefine: en el Petrel es `super(kilometros)` porque `volar` recibe los kilómetros.

**¿Por qué no `self.esDerrochon()` en lugar de `super()`?**
Porque el lookup empezaría otra vez en `Fiestero` y se llamaría a sí mismo: loop infinito.

**¿Podrías haber usado composición en vez de herencia para el fiestero?**
Sí: el cliente podría tener un objeto "criterio de derroche" (común o fiestero) y delegarle la pregunta. Elegimos herencia porque un fiestero **es un** cliente, ser fiestero es algo **intrínseco** de la persona según la consigna, y cambia un solo comportamiento mientras el resto se comparte entero.

**¿Y si Damián deja de ser fiestero?**
Con herencia no se puede: una instancia no cambia de clase, habría que crear otro cliente y reemplazar todas las referencias. Si la consigna pidiera eso, convendría composición: cambiar el criterio del mismo cliente (`damian.criterioDerroche(criterioComun)`), sin perder sus servicios.

**¿Hay composición en el TP?**
Sí, en casi todo el modelo: el cliente **tiene** una fiesta y servicios, la fiesta **tiene** sectores, la agencia **tiene** proveedores, y Doña Rosa y Pizzas Nico **conocen** al barril. La herencia aparece solo donde algo **es un** caso particular de otra cosa.

**¿Cómo hacés que un proveedor aparezca una sola vez?**
Los agrego a un conjunto, que no admite repetidos. Como Silvina es el mismo objeto en los dos servicios, el conjunto la deja una vez.

**¿Igualdad o identidad?**
El conjunto compara con `==`. Los proveedores no lo redefinen, así que `==` compara identidad: el mismo objeto. Los servicios, en cambio, son objetos distintos (cada `contratar` hace un `new`), y por eso se cuentan los 4.

**¿Por qué `servicios` es `const` si se le agregan cosas?**
Porque `const` es sobre la **referencia**: apunta siempre a la misma lista. Lo que cambia es el contenido de la lista.

**¿Qué pasa con `totalGastado()` si no contrató nada?**
Devuelve 0. Empezar sin servicios es un estado normal, no un error.

---
---

# Entrega 2 — Punto grupal: la agencia, el fixture y el README

**Consigna**: modelar la agencia y armar el fixture compartido (fiestas, clientes y agencia) con los casos de la agencia. Justificar en el README dónde usaron herencia y qué comparten las clases. Responder: ¿fue necesario ajustar los casos de los caterings de la entrega 1? ¿Y los de Isabella? ¿Cómo ayudan los tests de regresión?

## 1. La agencia

```wollok
object agencia {
    const proveedores = #{}

    method incorporar(proveedor) {
        proveedores.add(proveedor)
    }

    method desvincular(proveedor) {
        proveedores.remove(proveedor)
    }

    method proveedoresQuePuedePagar(cliente) = proveedores.filter({ proveedor => cliente.puedeContratar(proveedor) })
}
```

| Decisión | Por qué |
|---|---|
| **wko** | Hay una sola agencia: es un concepto único del negocio. |
| **Conjunto `#{}`** | Un proveedor está o no está representado; no tiene sentido tenerlo dos veces. |
| **`const proveedores`** | La referencia es siempre la misma; cambia el contenido (`add`, `remove`). |
| **`incorporar` y `desvincular`** | La consigna: *"debe poder incorporar y desvincular proveedores"*. Son **acciones**. |
| **`proveedoresQuePuedePagar(cliente)`** | El requerimiento: *"a qué proveedores le podría pagar un cliente"*. Es una **pregunta**. |

### `proveedoresQuePuedePagar`: delegación y polimorfismo

- **La agencia pregunta, pero no decide**: la decisión de si puede pagar es del **cliente** (`cliente.puedeContratar(proveedor)`), porque él sabe sus ahorros y su porcentaje. Es la misma idea que `daniel.llevaA(pasajero)` en la clase 22: Daniel no calcula si el pasajero es joven, se lo pregunta.
- **Polimorfismo**: a la agencia no le importa si cada proveedor es un catering (wko) o una instancia de `Salon` o `Seguridad`. Todos entienden `cuantoCobra(fiesta)`.
- **`filter` no modifica** el conjunto de la agencia: preguntar a quién puede pagar Milena no le saca proveedores.

### Los casos de la agencia

| Caso | Presupuesto | Resultado |
|---|---|---|
| Isabella | $1.650 | Doña Rosa ($1.440), El Bodegón ($1.495), Marta ($1.200), Los Pibes ($800), Dodain ($1.200), Escudo ($200), Silvina ($200) |
| Milena | $400 | Escudo ($220), Silvina ($250) |
| Damián | $50.000 | los diez (el más caro, Los Tilos, cobra $16.000) |
| Se desvincula a Silvina, Milena | $400 | solo Escudo |

Quedan afuera para Isabella: Pizzas Nico ($2.350), Los Tilos ($4.000) y Eliana ($3.000).

## 2. Los proveedores como constantes globales

```wollok
const losTilos = new Salon(precioPorSector = 500)
const marta = new Organizador(precioPorSector = 600)
...
```

La agencia y los tests necesitan referirse a **cada** proveedor por su nombre. Como son instancias (no wkos), se declaran como **constantes** en el `.wlk`, igual que `descuentoComun = new Descuento(descuento = 5)` en la clase 24 (punto 5.3).

## 3. El fixture compartido

### ¿Qué es un fixture?

El **escenario inicial** que necesitan los tests: los objetos ya creados y configurados antes de que cada test empiece. Se arma **una sola vez**, en lugar de repetirlo en cada test (clase 23, módulo 11, punto 2).

### El problema: varios `describe` necesitan lo mismo

Los tests están en **un solo archivo** (`tests.wtest`) con **varios `describe`**, uno por tema:

| `describe` | ¿Usa fiestas o clientes del enunciado? |
|---|---|
| Punto 1: Respecto a las fiestas | sí, las 3 fiestas |
| Respecto a los proveedores, con fiestas de resguardo (puntos 1 y 2) | no, usa fiestas de prueba |
| Respecto a los proveedores, con las fiestas de los clientes (puntos 1 y 2) | sí, las 3 fiestas |
| Agencia (Punto Grupal) | sí, Isabella, Milena y Damián |
| Punto 3: Contrataciones y derrochones | sí, los 4 clientes |
| Regresión de la entrega 1: caterings por cantidad de invitados (punto 1) | no |
| Regresión de la entrega 1: Isabella (punto 3) | sí, Isabella |

Lo declarado en un `describe` **no se puede usar desde otro** (clase 23, punto 2.5). Si cada `describe` armara sus fiestas y clientes, la fiesta de Damián quedaría escrita varias veces. Eso pasaba en una versión anterior: estaba **4 veces**.

### La solución: un objeto aparte (clase 23)

La clase 23 (punto 2.5) lo resuelve así: *"Si hace falta compartir la creación de objetos entre varios [`describe`], conviene poner esa responsabilidad en un **objeto aparte**"*. Ese objeto es `fixtureCompartido`, en el archivo `fixtureCompartido.wlk`:

```wollok
import example.*

object fixtureCompartido {
    const property fiestaIsabella = new Fiesta(duracion = 4, sectores = [...])
    const property fiestaMilena = new Fiesta(duracion = 6, sectores = [...])
    const property fiestaDamian = new Fiesta(duracion = 8, sectores = [...])

    const property isabella = new Cliente(ahorros = 2750, fiesta = fiestaIsabella, porcentajeDisponible = 0.6)
    const property milena = new Cliente(ahorros = 2000, fiesta = fiestaMilena)
    const property damian = new Fiestero(ahorros = 250000, fiesta = fiestaDamian)
    const property tomas = new Fiestero(ahorros = 250000, fiesta = fiestaIsabella)

    method prepararAgencia() {
        agencia.incorporar(doniaRosa)
        ... (los diez)
    }
}
```

Cada `describe` toma **solo lo que necesita**, como variables del `describe` a partir de un mensaje (clase 23, punto 2.2: pueden ser *"resultados de mensajes"*):

```wollok
import example.*
import fixtureCompartido.*

describe "Agencia (Punto Grupal)" {
    const isabella = fixtureCompartido.isabella()
    const milena = fixtureCompartido.milena()
    const damian = fixtureCompartido.damian()

    method initialize() {
        fixtureCompartido.prepararAgencia()
    }

    test "Milena solo puede pagarle a Escudo y a Silvina" {
        assert.equals(#{escudo, silvina}, agencia.proveedoresQuePuedePagar(milena))
    }
}
```

> **¿Por qué no se llama `fixture`?** Porque `fixture` es una **palabra reservada** de Wollok: en versiones anteriores, `fixture { ... }` era la inicialización de un `describe` (lo que hoy es `method initialize()`). Wollok da error si se usa como nombre.

### ¿Por qué un solo archivo de tests?

La clase 22 (módulo 5, punto 8) dice que *"si corresponden a unidades bien distintas, conviene separarlos en archivos diferentes"*. Probamos separarlos (fiestas, proveedores, clientes y agencia), pero como en clase no se profundizó sobre eso, el grupo decidió **un solo archivo con varios `describe`**, uno por tema. El objeto `fixtureCompartido` funciona igual en los dos casos.

### Orden de ejecución de cada test (clase 23, punto 2.5)

1. Se **reinicia** el ambiente: también el objeto `fixtureCompartido` (sus fiestas y clientes vuelven a crearse) y la agencia (vuelve a estar vacía).
2. Se crean las **variables** del `describe` (`const isabella = fixtureCompartido.isabella()`).
3. Se ejecuta el **`initialize`** (por ejemplo, `fixtureCompartido.prepararAgencia()`).
4. Se corre el **test**.

Por eso los tests son **independientes**: el que hace que Isabella contrate a Escudo no afecta al siguiente, que la vuelve a tener con $2.750. Y el que desvincula a Silvina tampoco, porque la agencia se vuelve a preparar.

### Tomás: la fiesta "idéntica"

La consigna dice que la fiesta de Tomás *"es idéntica a la de Isabella"*. En el fixture, Tomás recibe **el mismo objeto** `fiestaIsabella`: son **idénticas** en el sentido de la clase 24 (`===` da `true`). Como contratar no modifica la fiesta, compartirla no trae problemas, igual que el barril compartido de la entrega 1.

> **Un detalle sobre el objeto aparte.** El ejemplo de la clase 23 es un objeto que **crea** objetos nuevos cada vez (*"uno que cree ciudades"*). El nuestro **guarda** los objetos ya creados (`const property`). Funciona igual porque el ambiente se reinicia antes de cada test, y así Tomás e Isabella comparten **la misma** fiesta sin hacer nada extra. Con un objeto que crea, cada llamada haría una fiesta nueva: serían **iguales**, pero no **idénticas**.

> **En la defensa**: "El fixture compartido es un objeto aparte que arma el escenario del enunciado: las fiestas, los clientes y la agencia. Cada `describe` toma de ahí lo que necesita, así no repetimos las declaraciones. Lo sugiere la clase 23 para compartir objetos entre `describe`. Como el ambiente se reinicia antes de cada test, cada test arranca desde el estado inicial."

## 4. Las respuestas del README

### ¿Dónde usaron herencia y qué comparten?

**`Fiestero` hereda de `Cliente`**: comparte todo (ahorros, fiesta, presupuesto, contratar, servicios, gasto, proveedores contratados) y redefine solo `esDerrochon()`, reutilizando la regla general con `super()`.

**`Salon`, `Organizador` y `Seguridad` heredan de `Proveedor`**: comparten el atributo `precioPorSector` y el método `costoBase(fiesta)` ("precio por sector × cantidad de sectores"). Cada una define su `cuantoCobra(fiesta)` a partir de `costoBase`. `EquipoDeMozos` no hereda: según el README, porque no comparte `precioPorSector`. El README también menciona la alternativa descartada: un `Proveedor` general, con `ProveedorPorSector` y mozos como hijas, que no usaron porque `Proveedor` quedaría vacía.

**Isabella no es subclase**: solo cambia un valor (el 60%), no un comportamiento.

### ¿Fue necesario ajustar los casos de los caterings de la entrega 1?

**No.** Siguen entendiendo `cuantoCobraTotal(cantidadInvitados)`, que no cambió. `cuantoCobra(fiesta)` se agregó **encima** y reutiliza ese cálculo. Los valores son los mismos ($1.100 por 40 invitados, etc.).

En `tests.wtest` están las dos cosas: el `describe` "Regresión de la entrega 1: caterings por cantidad de invitados" (los casos de la entrega 1, sin cambios) y los casos "por una fiesta de 40 invitados" con **objetos anónimos**, en el `describe` de fiestas de resguardo:

```wollok
test "Cuánto cobra Doña Rosa por una fiesta de 40 invitados" {
    const fiesta40 = object {
        method cantidadInvitados() = 40
    }
    assert.equals(1100, doniaRosa.cuantoCobra(fiesta40))
}
```

### ¿Y los de Isabella?

**Sí, pero solo en cómo se arma**, no en los resultados. Pasó de ser un wko con sus sectores a una instancia de `Cliente` con una `Fiesta`, y `puedeContratar` ahora le pregunta `cuantoCobra(fiesta)` a cualquier proveedor. Los valores esperados no cambiaron ($1.650, Doña Rosa sí, Pizzas Nico no, barril a $500, ahorra $2.000).

### ¿Cómo ayudan los tests de regresión?

Avisan si un cambio rompe algo que ya funcionaba, y dicen exactamente qué se esperaba y qué se obtuvo (clase 22, módulo 5). En este TP lo vivimos: al juntar los puntos, los tests de contratación detectaron los mensajes mal escritos y las cuentas mal hechas del punto 2, y los de Isabella confirmaron que el cambio de wko a `Cliente` no alteró los resultados.

---
---

# Cierre — Conceptos transversales (para repasar el día antes)

Todo el TP se apoya en un puñado de ideas que se repiten. Si entendés estas nueve, podés defender cualquier punto.

## 1. Clase, instancia, wko u objeto anónimo

### Si te preguntan qué es cada uno, en una oración

> **wko (well-known object)**: "Es un objeto único, definido con `object`, que existe desde el principio y al que cualquiera puede hablarle por su nombre, como la `agencia` o `doniaRosa`."

> **Clase**: "Es un molde que define el comportamiento y la estructura que comparten varios objetos; cada objeto creado con `new` es una instancia con su propio estado, como cada `Cliente`."

> **La diferencia**: "Uso un wko cuando el concepto es único, y una clase cuando hay varios objetos que se comportan igual y solo cambian sus valores."

### Cuándo usar cada uno

| Qué | Cuándo | En el TP |
|---|---|---|
| **wko** | Concepto único | `barril`, `doniaRosa`, `pizzasNico`, `elBodegon`, `agencia` |
| **Clase** | Varios objetos con el mismo comportamiento | `Cliente`, `Fiesta`, `Servicio`, `Salon`, ... |
| **Instancia** | Un objeto concreto de una clase, con su propio estado | `isabella`, `milena`, `losTilos`, `escudo`, ... |
| **Objeto anónimo** | Alcance acotado, típicamente un test | fiestas de resguardo del punto 2 |

**Regla de oro (clase 24)**: clases distintas solo si hay **comportamiento** distinto. Si solo cambian **valores**, instancias distintas de la misma clase.

## 2. Herencia: qué se comparte y qué se redefine

| | `Fiestero` → `Cliente` | `Salon`, `Organizador`, `Seguridad` → `Proveedor` |
|---|---|---|
| Qué comparten | todo el cliente | el atributo `precioPorSector` y el método `costoBase(fiesta)` |
| Qué cambia | `esDerrochon()` | cada una define su `cuantoCobra(fiesta)` |
| Cómo | `override` + `super()` | métodos nuevos en cada subclase que usan `self.costoBase(...)` |

```wollok
class Proveedor {
    var property precioPorSector
    method costoBase(unaFiesta) = precioPorSector * unaFiesta.cantidadDeSectores()
}

class Salon inherits Proveedor {
    method cuantoCobra(unaFiesta) = self.costoBase(unaFiesta) * unaFiesta.duracion()
}
```

**Method lookup de `losTilos.cuantoCobra(fiesta)`**: busca `cuantoCobra` en `Salon` → lo encuentra → adentro, `self.costoBase(...)` se busca en `Salon` → no está → sube a `Proveedor` → lo encuentra y lo ejecuta con el `precioPorSector` de Los Tilos (500).

**Clase abstracta**: `Proveedor` nunca se instancia con `new` (solo sus subclases), así que es abstracta "naturalmente". En Wollok no hace falta declararlo (clase 24, punto 5.1).

> **Una diferencia con `Fiestero`**: las subclases de `Proveedor` no **redefinen** nada (no usan `override`), porque `Proveedor` no tiene `cuantoCobra`. Agregan un método nuevo que **reutiliza** uno heredado (`costoBase`). `Fiestero`, en cambio, **redefine** `esDerrochon()` y usa `super()` para reutilizar la versión de `Cliente`. Son las dos formas de aprovechar la herencia: agregar comportamiento usando lo heredado, o redefinir lo heredado.

> ⚠️ **`EquipoDeMozos` no hereda de `Proveedor`.** Su cuenta también es "algo × cantidad de sectores" (`mozosPorSector × 50 × sectores × horas`), así que se podría discutir que quedó código repetido. La justificación del README es que no comparte `precioPorSector`.

### ¿Qué es herencia y qué es composición?

Las dos son formas de **reutilizar comportamiento** sin copiar código. La diferencia está en **entre quiénes** es la relación y en **cuándo** queda fija.

#### Herencia: "es un"

Una **clase** se define como un **caso particular** de otra. La subclase recibe **todos** los atributos y métodos de la superclase, y puede agregar los suyos o redefinir (`override`) los que cambian.

```wollok
class Cliente {
    method contratar(proveedor) { ... }
    method esDerrochon() = ...
}

class Fiestero inherits Cliente {       // un fiestero ES UN cliente
    override method esDerrochon() = !fiesta.esLarga() && super()
}
```

- La relación es **entre clases** y se escribe en el código (`inherits`). Queda **fija**: un objeto creado con `new Fiestero(...)` es un `Fiestero` para siempre.
- Lo que no está en la subclase, el **method lookup** lo va a buscar a la superclase. Por eso `damian.contratar(...)` funciona sin haberlo escrito en `Fiestero`.
- La subclase hereda la interfaz **completa**: si `Cliente` entiende un mensaje, `Fiestero` también, lo quiera o no.
- Con **herencia simple** (como en Wollok) una clase hereda de **una sola** superclase: hay que elegir un único punto de vista para la jerarquía.

#### Composición: "tiene un"

Un **objeto** guarda una **referencia** a otro objeto y le **delega** parte del trabajo mandándole mensajes. No hay ninguna relación entre las clases: es una relación **entre instancias**.

```wollok
class Cliente {
    const fiesta                       // un cliente TIENE UNA fiesta
    method puedeContratar(proveedor) = proveedor.cuantoCobra(fiesta) <= self.presupuestoMaximo()
}

object doniaRosa {                     // Doña Rosa CONOCE al barril y le pide el costo
    method cuantoCobraTotal(cantidadInvitados) = barril.costoPara(cantidadInvitados) + ...
}
```

- La relación es **entre objetos**, a través de un atributo. Si el atributo es `var`, se puede **cambiar en cualquier momento**: el mismo objeto pasa a delegar en otro.
- El objeto que compone decide **qué mostrar**: solo expone los mensajes que quiere, no toda la interfaz del objeto que conoce.
- Se puede tener **varias** referencias a la vez, cada una con un objeto distinto, sin el límite de "un solo padre".

#### La diferencia, de un vistazo

| | Herencia | Composición |
|---|---|---|
| Frase que la describe | "un fiestero **es un** cliente" | "un cliente **tiene una** fiesta" |
| Entre quiénes | **clases** | **objetos** (instancias) |
| Cómo se escribe | `class B inherits A` | un atributo que apunta a otro objeto |
| Cómo se reutiliza | el lookup sube a la superclase | el objeto le manda un mensaje a otro |
| ¿Se puede cambiar después? | **no**, un objeto no cambia de clase | **sí**, cambiando la referencia |
| Interfaz | se hereda **toda** | se expone **solo lo que se quiere** |
| Cuántos | **una** superclase (herencia simple) | **todas** las referencias que hagan falta |
| Ventaja | muy poco código: se redefine solo lo distinto, y `super` reutiliza el resto | flexibilidad, encapsulamiento |
| Costo | rigidez: se "gasta" el único tiro de la herencia | más objetos y más mensajes entre ellos |

#### El ejemplo de la clase: la pila

¿Una pila **es una** lista o **tiene una** lista? (clase 24, punto 5.2)

- Si **hereda** de `Lista`, entiende **todos** los mensajes de una lista: se le podría pedir el tercer elemento o sacar uno del medio, cosas que una pila no debería permitir.
- Si **tiene** una lista, la usa por dentro y solo muestra `push()`, `top()` y `pop()`. La implementación queda **encapsulada**, y queda libre la herencia por si a futuro hace falta.

Por eso la pila va con composición: no **es** una lista, solo **usa** una.

### Herencia vs. composición en el TP

| Relación | Tipo | Por qué |
|---|---|---|
| `Fiestero` → `Cliente` | **herencia** ("es un") | intrínseco, cambia un solo comportamiento |
| `Salon`, `Organizador`, `Seguridad` → `Proveedor` | **herencia** ("es un") | comparten `precioPorSector` y `costoBase`; un salón nunca se convierte en seguridad |
| `Cliente` → `Fiesta`, `Servicio` | **composición** ("tiene un") | el cliente conoce su fiesta y sus servicios |
| `Fiesta` → sectores | **composición** | la fiesta conoce sus sectores |
| `agencia` → proveedores | **composición** | la agencia conoce a sus proveedores |
| `doniaRosa`, `pizzasNico` → `barril` | **composición** | le piden el costo de la bebida |

**Regla para decidir**: herencia si es **"es un"** y **no va a cambiar** durante la vida del objeto. Composición si es **"tiene un"**, o si **puede cambiar** (con herencia, una instancia no cambia de clase). Ver el ejemplo del fiestero con criterio de derroche en el [Paso 8](#paso-8-los-fiesteros--herencia).

## 3. Polimorfismo: un mensaje, muchos objetos

**Polimorfismo** es cuando **dos o más objetos distintos entienden el mismo mensaje**, cada uno a su manera, y quien les manda el mensaje **los puede tratar igual** sin saber cuál es cuál.

### La idea (clase 21, módulo 3)

Hacen falta **tres** participantes:

1. Un objeto que **envía** el mensaje (el que usa).
2. Dos o más objetos que **entienden** ese mensaje, cada uno con **su propio método** (su propia forma de resolverlo).

Esos objetos son **polimórficos para quien los usa**. El beneficio no es para ellos, sino para el que les habla: puede cambiar de objeto **sin cambiar el código** que manda el mensaje.

La analogía de la clase es el botón **play**: un iPod, un DVD y una videocasetera entienden `play()`, cada uno hace algo distinto, y vos apretás play sin pensar en cómo funciona cada aparato.

### En el TP

```wollok
method puedeContratar(proveedor) = proveedor.cuantoCobra(fiesta) <= self.presupuestoMaximo()
```

El cliente le manda `cuantoCobra(fiesta)` a lo que le llegue, y cada proveedor responde con **su** cuenta:

| Proveedor | Qué hace al recibir `cuantoCobra(fiesta)` |
|---|---|
| `doniaRosa` | $20 × invitados + bebida |
| `elBodegon` | $900 + $35 por cada invitado después de 25 |
| `losTilos` (Salón) | $500 × sectores × horas |
| `escudo` (Seguridad) | $100 × sectores, + 10% si es larga |

El cliente **no sabe** cuál de ellos recibe y **no le importa**. El código no tiene ningún `if` del tipo "si es catering hacé tal cosa, si es salón hacé otra".

**Mensaje ≠ método**: el mensaje es el **qué** ("¿cuánto cobrás?"), el método es el **cómo** (la cuenta de cada uno). Lo que conecta uno con otro es el [method lookup](#4-method-lookup-cómo-se-encuentra-el-método).

### Lo que **no** hace falta para que haya polimorfismo

- **No hace falta herencia.** `doniaRosa` es un wko que no hereda de nada, y `losTilos` es una instancia de `Salon`. Igual son polimórficos, porque los dos entienden `cuantoCobra(fiesta)`. En Wollok el criterio es **entender el mensaje**, no pertenecer a una clase (clase 21, punto 4; clase 23, punto 7).
- **No hace falta que compartan toda la interfaz.** Alcanza con **el mensaje que usa** quien les habla. Doña Rosa entiende `cuantoCobraServicio(...)`, y Los Tilos no, pero al cliente no le importa porque nunca se lo pide.
- **Es en un contexto.** Son polimórficos **para el cliente y la agencia**, en el contexto de "cuánto cobrás". Fuera de eso pueden no tener nada en común.

### Todos los polimorfismos del TP

| Mensaje | Lo entienden | Quién lo envía |
|---|---|---|
| `cuantoCobra(fiesta)` | 3 caterings (wko) + 7 proveedores por sector (instancias) | `Cliente` |
| `invitadosSector()` | `SectorDeMesas`, `SectorDeGrupos` (y los wko `familia`, `facultad`) | `Fiesta`, en `cantidadInvitados()` |
| `esDerrochon()` | `Cliente`, `Fiestero` | los tests |
| `puedeContratar(proveedor)` | Isabella, Milena, Damián y Tomás | `agencia` |

En la entrega 1 eran `cuantoCobraTotal(cantidadInvitados)` entre los tres caterings y `cantidadInvitados()` entre los dos sectores.

### Ventajas (lo que van a preguntar)

1. **Se pueden agregar objetos nuevos sin tocar al que los usa.** Si la agencia suma un fotógrafo, alcanza con que entienda `cuantoCobra(fiesta)`: ni el cliente ni la agencia cambian. Es la pregunta de la entrega 1, "¿qué habría que cambiarle a Isabella si aparece un cuarto catering?": **nada**.
2. **El código del que usa queda simple:** no pregunta "¿qué sos?" antes de actuar.
3. **Bajo acoplamiento:** el cliente no depende de cómo calcula cada proveedor. Si Matías cambia la cuenta del salón, el cliente no se entera.
4. **División de tareas:** yo programé el `Cliente` y Matías los proveedores, en paralelo. Solo teníamos que ponernos de acuerdo en el nombre del mensaje. Por eso, cuando escribió `cantidadSectores()` y la fiesta entendía `cantidadDeSectores()`, se rompió: el polimorfismo depende de que **el nombre del mensaje sea exactamente el mismo**.
5. **Facilita testear:** un objeto anónimo que entienda `cuantoCobra(fiesta)` puede reemplazar a un proveedor real en un test (clase 22, módulo 6, y clase 23, los stubs).

> **En una frase**: "Polimorfismo es que distintos objetos entiendan el mismo mensaje, cada uno con su propio método, para que quien los usa pueda tratarlos igual sin saber cuál es. En el TP, el cliente le pregunta `cuantoCobra(fiesta)` a cualquier proveedor sin saber si es un catering o un salón."

## 4. Method lookup: cómo se encuentra el método

El **method lookup** es el mecanismo que usa Wollok para **encontrar qué método ejecutar** cuando un objeto recibe un mensaje. Es la búsqueda que conecta el **mensaje** (el qué) con el **método** (el cómo).

Cuando escribís `isabella.contratar(escudo)`, el código no dice **qué** `contratar` ejecutar. Lo decide el lookup **en el momento de ejecutar**, buscando en el objeto que recibe el mensaje.

### Cómo busca, según qué sea el receptor

En la materia lo fueron ampliando de a poco.

**1. Si el receptor es un wko (clase 21)**: busca el método **en el propio objeto**.

```wollok
doniaRosa.cuantoCobra(fiesta)
```

→ busca `cuantoCobra` en `doniaRosa` → lo encuentra → lo ejecuta.

**2. Si el receptor es una instancia de una clase (clase 23)**: las instancias no tienen métodos propios, el comportamiento está en la **clase**. Entonces:

- El **método** se busca en la **clase** del receptor.
- Los **atributos** se resuelven en la **instancia**: se ejecuta con **su** estado.

```wollok
losTilos.cuantoCobra(fiestaIsabella)
```

→ `losTilos` es un `Salon` → busca en `Salon` → encuentra `cuantoCobra` → lo ejecuta con el `precioPorSector` **de Los Tilos** (500). Si fuera otro salón con precio 800, encontraría **el mismo método**, pero con **su** precio.

**3. Si hay herencia (clase 24)**: si no lo encuentra en la clase, **sube** a la superclase, y así hasta llegar a `Object`, la madre de todas las clases.

```wollok
damian.contratar(losTilos)
```

1. `damian` es un `Fiestero` → busca `contratar` en `Fiestero` → **no está**.
2. Sube a `Cliente` → **lo encuentra** → lo ejecuta con el estado **de Damián**.

```wollok
damian.esDerrochon()
```

1. Busca en `Fiestero` → **lo encuentra** (está redefinido con `override`) → ejecuta ese y **no sigue subiendo**.

Por eso `override` funciona: el lookup encuentra primero el de la subclase.

**4. Si no lo encuentra en ningún lado**: llega hasta `Object`, no lo encuentra y falla.

```
wollok.lang.MessageNotUnderstoodException: a Fiesta does not understand cantidadSectores()
```

Es el error que tuvimos al juntar el punto 2: los proveedores le mandaban `cantidadSectores()` a la fiesta, el lookup lo buscó en `Fiesta`, después en `Object`, y no estaba en ninguno.

### `self` y `super`: desde dónde empieza a buscar

Dentro de un método se le puede mandar un mensaje al propio objeto. La diferencia entre `self` y `super` es **dónde arranca la búsqueda**:

| | Empieza a buscar en… | En el TP |
|---|---|---|
| `self.mensaje()` | la clase del **receptor**: lookup normal | `self.validarPresupuesto(proveedor)` en `contratar` |
| `super()` | la **superclase de la clase donde está escrito** | `super()` en `Fiestero.esDerrochon()` |

En los dos casos **el receptor es el mismo objeto**; solo cambia desde dónde se busca.

**Ejemplo con `super`**: `tomas.esDerrochon()`

1. Busca en `Fiestero` → lo encuentra → `!fiesta.esLarga()` da `true` (4 horas).
2. `super()` → **saltea** `Fiestero` y busca `esDerrochon` en `Cliente` → lo encuentra → ¿pagó algo de más de $3.000? → sí.
3. Resultado: es derrochón.

**Por qué no `self.esDerrochon()` ahí**: el lookup empezaría otra vez en `Fiestero`, encontraría el mismo método y se llamaría a sí mismo para siempre: **loop infinito**.

### Por qué importa: el lookup es lo que hace posible el polimorfismo

```wollok
method puedeContratar(proveedor) = proveedor.cuantoCobra(fiesta) <= self.presupuestoMaximo()
```

El mensaje es siempre el mismo, pero según qué llegue en `proveedor`, el lookup encuentra métodos distintos:

| `proveedor` es… | El lookup busca en… | Encuentra |
|---|---|---|
| `doniaRosa` | el wko `doniaRosa` | la cuenta de Doña Rosa |
| `losTilos` | la clase `Salon` | la cuenta del salón |
| `escudo` | la clase `Seguridad` | la cuenta de seguridad |

Si el método se decidiera al **escribir** el código y no al **ejecutarlo**, el cliente no podría tratar a todos igual.

### Resumen

1. Se busca en el **wko**, o en la **clase** del receptor.
2. Si no está, se **sube** por las superclases.
3. Si llega a `Object` sin encontrarlo: **"does not understand"**.
4. El método se busca en la clase, pero se ejecuta con el **estado de la instancia**.
5. `self` busca desde la clase del receptor; `super` busca desde la superclase de donde está escrito.

> **En una frase**: "El method lookup es cómo Wollok encuentra el método a ejecutar: lo busca en el objeto o en su clase, y si no está, sube por las superclases hasta `Object`. Por eso un fiestero sabe contratar sin haberlo escrito, y por eso el cliente puede mandarle `cuantoCobra` a cualquier proveedor."

## 5. Colecciones: listas y conjuntos

### Lista o conjunto (clase 22, punto 6.4)

| | Lista `[]` | Conjunto `#{}` |
|---|---|---|
| Orden | sí | no |
| Repetidos | **los admite** | **no los admite**: agregar uno que ya está no hace nada |
| Modela | una secuencia, un registro de hechos | el conjunto matemático: "quiénes" |

(En Wollok `[]` se llama **lista**, no array.)

**La regla para elegir**: ¿tienen sentido los repetidos? Si cada aparición cuenta → lista. Si solo importa **quiénes** están → conjunto.

### Por qué `servicios` es una lista

Cada contratación es un **hecho distinto**, aunque se repita el proveedor y el monto. La consigna pide:

> "Milena contrata dos veces a Silvina y dos veces a Escudo. Cuántos servicios tuvo" → **4**

Si contrató cuatro veces, tienen que quedar cuatro registros. Y `totalGastado()` tiene que sumar los cuatro pagos: Milena le pagó dos veces a Silvina, así que gastó dos veces esa plata. Con una lista no hay riesgo de perder ninguno. Además, cada `contratar` crea un `new Servicio(...)`, un objeto nuevo y distinto de los anteriores.

### Por qué los proveedores son conjuntos

Hay dos lugares, y en los dos interesa **quiénes**, no cuántas veces:

- **`proveedoresContratados()`** responde directamente la consigna: *"Si contrató dos veces al mismo, ese proveedor aparece **una sola vez**"*. El conjunto hace eso solo: al agregar `silvina` dos veces, queda una. Con una lista habría que sacar los repetidos a mano.
- **La agencia**: un proveedor está representado o no lo está; no tiene sentido tenerlo dos veces. Si por error se incorpora dos veces a Escudo, el conjunto lo deja una sola vez y no aparece repetido en `proveedoresQuePuedePagar`.

| | Colección | ¿Repetidos? | Pregunta que responde |
|---|---|---|---|
| `servicios` | lista `[]` | sí, cada contratación cuenta | ¿cuántas veces contrató? ¿cuánto pagó en total? |
| `sectores` (de la fiesta) | lista `[]` | — | ¿qué sectores tiene? |
| `proveedoresContratados()` | conjunto `#{}` | no | ¿quiénes le trabajaron? |
| proveedores de la agencia | conjunto `#{}` | no | ¿a quiénes representa? |

> **En la defensa**: "Elegí la colección según si los repetidos tienen sentido. En los servicios sí, porque cada contratación es un pago distinto. En los proveedores no, porque la consigna pide que cada uno aparezca una sola vez."

El conjunto decide "repetido" usando `==`, que por defecto es identidad. Ver [igualdad e identidad](#9-igualdad-e-identidad).

### Los mensajes que usamos (todos están en los apuntes)

| Mensaje | Qué hace | Dónde lo usamos |
|---|---|---|
| `add(x)` | agrega | `contratar`, `incorporar`, `proveedoresContratados` |
| `remove(x)` | saca | `desvincular` |
| `size()` | cuántos hay | `cantidadServicios`, `cantidadDeSectores` |
| `sum({ ... })` | suma lo que dice el bloque | `totalGastado`, `cantidadInvitados` |
| `filter({ ... })` | los que cumplen (colección nueva) | `esDerrochon`, `proveedoresQuePuedePagar` |
| `isEmpty()` | ¿está vacía? | `esDerrochon` |
| `forEach({ ... })` | hace algo con cada uno | `proveedoresContratados` |

`asSet()` y `any(...)` **no** los usamos porque no están en los resúmenes. `forEach` sí está: en la clase 24 (punto 6.2, `[pepita, new Ave(), tweety, chichita].forEach({ ave => ave.volar(5) })`) y mencionado en la clase 22 (punto 5.5).

### ¿Cuáles modifican la colección y cuáles no?

`map` y `filter` **no modifican** la colección que reciben: la dejan exactamente como estaba y **arman una nueva** con los resultados (clase 22, puntos 7.7 y 7.8: *"Tampoco tiene efecto colateral sobre la colección original"*).

```wollok
const servicios = [servicioSilvina, servicioEscudo]
const proveedores = servicios.map({ servicio => servicio.proveedor() })
// servicios   → sigue siendo [servicioSilvina, servicioEscudo]
// proveedores → [silvina, escudo], una lista NUEVA
```

"Transformar" no quiere decir que los servicios se conviertan en proveedores dentro de la lista original. Quiere decir que, para cada servicio, se calcula algo y ese resultado va a **otra** lista.

**Por qué importa** (clase 22, punto 7.7): preguntar *"¿cuáles clientes me deben plata?"* no puede hacerte perder a los que sí pagaron. En el TP:

- `esDerrochon()` hace `servicios.filter(...)`. Si modificara `servicios`, preguntar si Isabella es derrochona le borraría los servicios baratos, y `cantidadServicios()` daría mal.
- La agencia hace `proveedores.filter(...)`. Si modificara el conjunto, preguntar a quién puede pagar Milena le sacaría a la agencia los proveedores caros.

| Mensaje | ¿Modifica la colección? | ¿Qué devuelve? | Es una… |
|---|---|---|---|
| `map({ ... })` | **no** | una colección nueva con los resultados | pregunta |
| `filter({ ... })` | **no** | una colección nueva con los que cumplen | pregunta |
| `sum`, `size`, `isEmpty`, `contains` | **no** | un número o un booleano | pregunta |
| `forEach({ ... })` | **no** la colección, pero el bloque puede **hacer cosas** | nada útil | acción |
| `add(x)`, `remove(x)` | **sí** | nada útil | acción |

### `forEach`: no devuelve nada, ejecuta el bloque con cada uno

`forEach` ejecuta el bloque con cada elemento y **no devuelve nada útil**. El efecto lo produce **lo que hace el bloque**, y puede caer sobre los elementos o sobre **otro objeto**:

**El efecto cae sobre los elementos** (clase 24):

```wollok
[pepita, new Ave(), tweety, chichita].forEach({ ave => ave.volar(5) })
```

Cada ave **cambia su energía**: se modifican los elementos.

**El efecto cae sobre otro objeto** (nuestro TP):

```wollok
servicios.forEach({ servicio => proveedores.add(servicio.proveedor()) })
```

Los servicios **no cambian** (solo se les pregunta `proveedor()`). Lo que cambia es **otro** objeto, el conjunto `proveedores`, que recibe un `add` por cada servicio.

**En los dos casos la colección que se recorre no cambia**: la lista sigue teniendo las mismas aves, y `servicios`, los mismos servicios.

**Regla para elegir**: si querés **obtener** algo → `map`, `filter`, `sum`. Si querés **hacer** algo con cada uno → `forEach`. Un `forEach` cuyo bloque no produce ningún efecto no sirve para nada, porque no devuelve resultado.

### ¿Funcionan igual en listas y conjuntos?

**Todos los que usamos funcionan en los dos**, pero se comportan distinto según la colección. Lo verificamos con Wollok:

| Mensaje | En una lista | En un conjunto |
|---|---|---|
| `add(x)` | lo agrega, **aunque ya esté** | lo agrega **solo si no está** |
| `remove(x)` | saca **una** aparición | lo saca |
| `size()` | cuenta todos, con repetidos | cuenta los distintos |
| `contains(x)`, `isEmpty()`, `sum`, `forEach` | ✓ | ✓ |
| `filter({ ... })` | devuelve una **lista** | devuelve un **conjunto** |
| `map({ ... })` | devuelve una **lista** | devuelve una **lista** ⚠️ |

#### ⚠️ `map` sobre un conjunto devuelve una lista

```wollok
#{1, 2, 3}.map({ n => 0 })     // → [0, 0, 0]   (lista, con repetidos)
```

Tiene sentido: al transformar, dos elementos distintos pueden dar el mismo resultado, y `map` no los colapsa.

Esto confirma que en `proveedoresContratados()` hicimos bien en armar el conjunto con `#{}` + `forEach` + `add`. Con `servicios.map({ servicio => servicio.proveedor() })`, el resultado sería una **lista con Silvina y Escudo repetidos**, y el test de Milena fallaría.

#### Los que son solo de listas

Las listas tienen **orden**, así que entienden mensajes que dependen de la posición. Los conjuntos no, porque no tienen orden:

```wollok
[1, 2, 3].last()     // → 3
#{1, 2, 3}.last()    // → error
```

`last()` aparece en la clase 23 (`ciudadesFavoritas.last()` de pepita, que es una lista). En el TP no usamos ninguno de estos.

## 6. Estado: `var`, `const`, `property` y cuándo poner getters y setters

### `var` y `const` (clase 21, módulo 2, punto 6)

Las dos declaran una **referencia** (un nombre que apunta a un objeto). La diferencia es si esa referencia **puede pasar a apuntar a otro objeto** después.

| | Significado | ¿Se puede hacer `x = otraCosa`? |
|---|---|---|
| `var` | referencia **variable** | sí |
| `const` | referencia **fija** | no: da error |

```wollok
var edad = 10
edad = 11          // ✓ apunta a otro número

const edadAdulta = 21
edadAdulta = 18    // ✗ ESTO DA ERROR
```

#### La trampa: `const` no significa "no cambia nada"

`const` fija **la referencia**, no **el objeto** al que apunta. Si el objeto es una colección, se le pueden agregar y sacar elementos igual:

```wollok
const servicios = []

servicios.add(unServicio)    // ✓ cambia el CONTENIDO de la lista
servicios = []               // ✗ ERROR: intenta que la referencia apunte a OTRA lista
```

Es como una caja con tu nombre pegado: `const` dice que el nombre siempre está pegado a **esa** caja, pero lo que hay adentro puede cambiar. Es el ejemplo de Shrek (clase 22, punto 7.1): `const misiones = #{}` porque *"siempre va a apuntar al mismo conjunto; lo que cambia es el contenido"*.

**Con los números es al revés**: son **inmutables** (clase 22, punto 1), así que `ahorros + 100` no modifica el número, **crea uno nuevo**. Para que los ahorros cambien, la referencia tiene que pasar a apuntar al número nuevo, y eso solo se puede con `var`:

```wollok
var ahorros = 2750
ahorros = ahorros + 100    // la referencia pasa a apuntar al número 2850
```

#### Cuándo usar cada una

**La pregunta es: ¿alguna vez voy a escribir `nombre = ...` después de la declaración?**

- **Sí** → `var`.
- **No** → `const`.

Ante la duda, `const`: es más seguro, porque si por error intentás reasignar, Wollok te avisa.

#### En el TP

| Declaración | Por qué |
|---|---|
| `var ahorros` | Se reasigna: `ahorros = ahorros + monto` y `ahorros = ahorros - monto`. |
| `const fiesta` | El cliente siempre tiene la misma fiesta; nunca se reasigna. |
| `const porcentajeDisponible = 0.2` | Nunca cambia (Isabella nace con 0.6, pero no se reasigna después). |
| `const montoDerroche = 3000` | Nunca cambia. |
| `const servicios = []` | Siempre es **la misma lista**; solo cambia su contenido con `add`. |
| `const proveedores = #{}` (agencia) | Siempre el mismo conjunto; cambia con `add` y `remove`. |
| `var property precio = 300` (barril) | Se reasigna con el setter cuando aumenta el barril. |

También vale para las **variables locales** dentro de un método:

```wollok
method contratar(proveedor) {
    self.validarPresupuesto(proveedor)
    const monto = proveedor.cuantoCobra(fiesta)    // se calcula una vez y no cambia
    ...
}

method proveedoresContratados() {
    const proveedores = #{}                         // siempre el mismo conjunto, se le agregan cosas
    ...
}
```

Y en los **tests**: las variables del `describe` son `const` (`const isabella = new Cliente(...)`). Isabella cambia **por dentro** cuando contrata, pero la referencia `isabella` apunta siempre al mismo objeto.

> ⚠️ **Detalle del código del punto 1**: en `Fiesta` y en los sectores están declarados `var property duracion`, `var property cantidadMesas`, etc., pero nadie los reasigna nunca. Podrían ser `const` (o `const property`). No rompe nada, pero si preguntan "¿por qué es `var`?", no hay una razón.

> **En la defensa**: "`var` es una referencia que puede pasar a apuntar a otro objeto, y `const` una que no. Uso `var` solo si reasigno, como `ahorros`. Para `servicios` uso `const`, porque la lista siempre es la misma: lo que cambia es su contenido, y eso no es reasignar."

### Las herramientas (clase 21, módulo 2)

| | Qué es | ¿Se puede reasignar? |
|---|---|---|
| `var x` | referencia **variable** | sí |
| `const x` | referencia **fija** | no (pero si apunta a una colección, el **contenido** sí puede cambiar) |
| `var property x` | `var` + **getter** `x()` + **setter** `x(valor)` | sí, también desde afuera |
| `const property x` | `const` + **getter** `x()` (sin setter) | no |

- **Getter**: una **pregunta** que devuelve el valor del atributo, con el mismo nombre (`method ahorros() = ahorros`).
- **Setter**: una **acción** que lo cambia desde afuera (`method ahorros(nuevo) { ahorros = nuevo }`).
- `property` es solo un **atajo** para no escribirlos a mano.

### Dar un valor al crear ≠ setter

| | Cuándo | Cómo |
|---|---|---|
| **Inicializar** | **una vez**, en el `new` | `new Cliente(porcentajeDisponible = 0.6, ...)` |
| **Setter** | **en cualquier momento**, después de creado | `isabella.porcentajeDisponible(0.6)` |

Los parámetros con nombre del `new` **no son mensajes**: le dicen a Wollok con qué valores arrancan los atributos de esa instancia. Funcionan con `const` y con `var`, con o sin `property`. Lo vieron en clase: `new Ciudad(consumo = 60)` con `const consumo` (clase 23), `new Descuento(descuento = 5)` con `const descuento` (clase 24).

**El valor por defecto** (`const porcentajeDisponible = 0.2`) se usa solo si al crear no se dice otro. Isabella no lo "pisa" después: **nace** con 0.6, y como es `const`, ya no cambia.

### Por qué casi no usamos `property` en `Cliente`

`property` sirve cuando hay que **leer o cambiar el atributo desde afuera, después de crear el objeto**. En `Cliente`:

| Atributo | Declaración | Por qué |
|---|---|---|
| `ahorros` | `var` + **getter a mano**, sin setter | Lo leen los tests, pero solo cambia con `ahorrar` y `contratar`. Con `var property`, cualquiera podría hacer `isabella.ahorros(1000000)`. Es lo que corrigió el profe en la entrega 1. |
| `porcentajeDisponible` | `const` con valor por defecto, sin getter | Fijo para cada cliente; se define al crearlo. Con `var property` se podría hacer `isabella.porcentajeDisponible(1)` y gastar el 100%. Nadie de afuera lo pregunta: se usa adentro, en `presupuestoMaximo()`. |
| `fiesta` | `const` sin valor por defecto, sin getter | Cada cliente trae la suya al crearse. Nadie de afuera se la pide (el getter lo borramos por eso). |
| `montoDerroche` | `const` con valor, sin getter | Solo para no tener un número mágico. |
| `servicios` | `const` (la lista no se reemplaza; se le agregan elementos) | Se consulta con `cantidadServicios()`, `totalGastado()`, etc., no exponiendo la lista. |

**La regla** (clase 21 y clase 22, punto 7.5): el estado lo modifica **el propio objeto** con sus métodos, y **no hace falta definir getters ni setters si no se necesitan desde afuera**. Por eso:

- **Getter** solo si alguien de afuera necesita el valor (`ahorros()`, para los tests).
- **Setter** casi nunca: en su lugar, una **acción con nombre del dominio** que hace la cuenta adentro (`ahorrar(monto)`, `contratar(proveedor)`).

En `Servicio`, en cambio, sí usamos `const property`: el cliente necesita **leer** `monto()` y `proveedor()` desde afuera, y nunca cambian.

## 7. Excepciones

- **Se lanzan** con `self.error("mensaje")` cuando algo sale del flujo normal del negocio.
- **No se atrapan** en el dominio: solo en los tests y en la interfaz de usuario.
- **Pregunta ≠ acción**: `puedeContratar` siempre devuelve un booleano; `contratar` contrata o lanza error.
- **Validar antes de modificar**: así, si falla, el estado queda como estaba.
- **Lista vacía no es error**.

### Lanzar y atrapar: quién hace cada cosa y por qué

Hay que separar **dos momentos** de un error:

- **Lanzar** una excepción es **avisar** que algo salió mal. Es lo que hace `self.error(...)`: corta el método y el error empieza a "subir".
- **Atrapar** una excepción es **recibir** ese aviso y decidir qué hacer. Se hace con `try/catch`:

```wollok
try {
    // código que puede fallar
} catch e : Exception {
    // qué hacer si falla
}
```

#### La analogía: la alarma de incendio

- El **detector de humo** se da cuenta de que hay un problema y **suena**. No apaga el fuego, porque no sabe hacerlo: solo avisa. → **Lanzar**.
- El **bombero** escucha la alarma y **hace algo**: apaga el fuego. → **Atrapar**.
- Un **inspector** prende un fósforo cerca del detector para **comprobar que suene**. No apaga ningún fuego: quiere verificar que la alarma funciona. → Lo que hace un **test**.

El detector **no** se responde a sí mismo: si sonara y él mismo se apagara, nadie se enteraría del fuego.

#### Las dos preguntas para decidir

| Pregunta | Si la respuesta es **sí** | En el TP |
|---|---|---|
| ¿**Detecté** algo que no debería pasar y **no puedo resolverlo** yo? | **Lanzo** (`self.error`) | El cliente detecta que no le alcanza la plata. No puede inventarla → lanza. |
| ¿Soy **quien sabe qué hacer** con ese error, o **quiero comprobar** que ocurra? | **Atrapo** (`try/catch`) | El test quiere comprobar que, si no alcanza, se lance el error → atrapa. |

Casi siempre, **quien detecta el problema no es quien sabe resolverlo**. Por eso lanzar y atrapar ocurren en **lugares distintos**.

#### Qué pasa cuando nadie la atrapa: la excepción sube

Cuando un método lanza una excepción y él no la atrapa, la excepción **sube** al que lo llamó. Si ese tampoco la atrapa, sigue subiendo, hasta que alguien la atrape o el programa se detenga mostrando el error.

Cuando un test hace `isabella.contratar(losTilos)`:

```
test  →  isabella.contratar(losTilos)  →  self.validarPresupuesto(losTilos)  →  self.error("...")
                                                                                        │
test  ←  contratar se corta acá  ←  validarPresupuesto se corta acá  ←──────────────────┘
(lo atrapa el assert)  (no descuenta ni registra)
```

1. `self.error` lanza la excepción dentro de `validarPresupuesto`.
2. `validarPresupuesto` no la atrapa → sube a `contratar`.
3. `contratar` no la atrapa → **se corta ahí**, sin descontar ni registrar, y la excepción sube a quien lo llamó.
4. El test la atrapa con `assert.throwsExceptionWithMessage`.

Por eso el estado de Isabella no cambia: `contratar` se cortó antes de tocar nada.

#### Por qué el cliente no la atrapa

Si `contratar` sí la atrapara:

```wollok
// lo que NO conviene
method contratar(proveedor) {
    try {
        self.validarPresupuesto(proveedor)
        const monto = proveedor.cuantoCobra(fiesta)
        ahorros = ahorros - monto
        servicios.add(new Servicio(proveedor = proveedor, monto = monto))
    } catch e : Exception {
        // ¿¿¿ qué pongo acá ???
    }
}
```

**¿Qué pondría el cliente en el `catch`?** No tiene nada razonable que hacer:

- **No puede arreglarlo**: no puede inventar plata.
- **Si no hace nada**, el error desaparece. Quien llamó a `contratar` cree que salió bien, pero no se contrató nada. Es exactamente **"mirar a un costado"**, la peor opción de la clase 22 (punto 5.1), solo que escrita con más código.

La clase 22 (punto 7) muestra este mismo ejemplo con un `¿¿¿ Qué hacer ???` adentro del `catch`, y concluye: **dejar que la excepción siga su curso**. Y en el punto 10: *"Nunca atrapar una excepción y no hacer nada con ella: eso oculta el problema y hace que aparezca, disfrazado, más adelante"*.

#### Quién sí la atrapa (clase 22, punto 7)

| Quién | Por qué tiene sentido que la atrape |
|---|---|
| **Un test** | Para **verificar** que el error se produjo cuando correspondía. |
| **La interfaz de usuario** | Para **mostrarle al usuario** un mensaje entendible ("No te alcanza el presupuesto") en lugar de que la aplicación se caiga. El usuario decide qué hacer: ahorrar más o elegir otro proveedor. |

En el TP no hay interfaz de usuario, así que el único que la atrapa es el test.

#### ¿Dónde está el `try/catch` en el test?

No lo escribimos, pero está **adentro** de `assert.throwsExceptionWithMessage`. Ese mensaje hace algo así por dentro:

```wollok
// Lo que hace assert.throwsExceptionWithMessage(mensajeEsperado, bloque), simplificado
try {
    bloque.apply()                                       // ejecuta isabella.contratar(losTilos)
    // si llegó acá, NO hubo error → el test FALLA ("se esperaba un error")
} catch e : Exception {
    // hubo error → lo atrapó
    // si e tiene el mensaje esperado → el test PASA
    // si tiene otro mensaje → el test FALLA
}
```

Por eso le pasamos el código **dentro de un bloque `{ ... }`**: para que el `assert` lo ejecute él, adentro de su `try`, y pueda atrapar el error. Si lo escribiéramos sin bloque, el error saltaría antes de que el `assert` llegue a hacer nada, y el test se cortaría.

#### Todo junto

```
   CLIENTE (detector)                         TEST (inspector)
   ──────────────────                         ────────────────
   "No me alcanza la plata"                   "Voy a hacer que contrate algo caro
   → self.error("...")    ── la excepción ──▶   y comprobar que avise"
   (lanza y se corta)        sube              → assert.throwsExceptionWithMessage
                                                 (adentro tiene el try/catch)
```

| | Lanza | Atrapa |
|---|---|---|
| **Quién** | el objeto de dominio (`Cliente`) | el test (o una interfaz de usuario) |
| **Con qué** | `self.error("...")` | `assert.throwsExceptionWithMessage(...)`, que tiene el `try/catch` adentro |
| **Para qué** | avisar que algo salió mal | comprobar que se avisó (test) o decidir qué hacer (interfaz) |

> **En la defensa**: "El cliente lanza porque detecta el problema pero no puede resolverlo. No la atrapa, porque si la atrapara y no hiciera nada sería como mirar a un costado: quien contrata no se enteraría de que falló. El test atrapa porque su trabajo es comprobar que el error aparezca. El `try/catch` del test está adentro del `assert`, por eso no lo escribimos."

### `self.error(...)` o `throw new UserException(...)`

La clase 22 (módulo 10, punto 6, "Cómo generar una excepción") muestra dos formas de lanzar una excepción:

```wollok
// 1. Enviando un mensaje que entiende todo objeto
self.error("Pasó tal cosa")

// 2. Instanciando una excepción (usando el concepto de clase)
throw new UserException(message = "Pasó tal cosa")
```

**Usamos la primera.** `error(...)` es un mensaje que **entiende todo objeto**, así que el cliente se lo manda a sí mismo con `self`. Lanza una excepción con ese mensaje y **corta el método ahí**: lo que viene después (descontar, registrar) no se ejecuta.

```wollok
method validarPresupuesto(proveedor) {
    if (!self.puedeContratar(proveedor)) {
        self.error("El presupuesto no alcanza para contratar al proveedor")
    }
}
```

**Por qué esta y no `throw`:**

| | `self.error("...")` | `throw new UserException(...)` |
|---|---|---|
| Qué hace falta | nada: todo objeto entiende `error` | primero definir la clase: `class UserException inherits Exception { }` |
| Cómo se escribe | un mensaje más | `throw` + crear una instancia con `new` |
| Qué se puede verificar en el test | el **mensaje** del error | el mensaje **y el tipo** de excepción |

- **Es más simple**: no hay que crear ninguna clase de excepción.
- **Alcanza para lo que pide la consigna**: el test usa `assert.throwsExceptionWithMessage(...)`, que verifica que se lance un error **con ese mensaje**. No hace falta saber de qué tipo es.

**Cuándo convendría `throw new UserException(...)`**: si quisiéramos que el test verifique también el **tipo** de excepción, con `assert.throwsExceptionLike(new UserException(message = "..."), { ... })` (clase 23, módulo 11, punto 3: es la opción de "restricción máxima"). Por ejemplo, para distinguir un error de negocio de uno de programa. La consigna no lo pide.

**Lo que rodea al `self.error` también sale de los apuntes:**

- **La validación en su propio método** (`validarPresupuesto`), como `validarMontoPositivo` del monedero (clase 22, punto 6).
- **Excepción por presupuesto insuficiente**: es literalmente el ejemplo del punto 10.1, "To throw or not to throw".
- **Validar antes de modificar el estado**, para que si falla no cambie nada.
- **Un mensaje claro** para quien lo lea (clase 22, punto 10): dice qué pasó, no un código.

> **En la defensa**: "Usé `self.error`, que es un mensaje que entiende todo objeto y lanza una excepción con ese mensaje. Es más simple que `throw new UserException`, que requiere definir la clase de excepción, y alcanza porque el test verifica el mensaje con `throwsExceptionWithMessage`. Si quisiera verificar también el tipo, usaría `throw` con `throwsExceptionLike`."

## 8. Tests

| Herramienta | Para qué | En el TP |
|---|---|---|
| Objeto aparte (`fixtureCompartido`) | el fixture compartido entre varios `describe` | fiestas, clientes y `prepararAgencia()` |
| `describe` + variables | tomar del fixture lo que usa cada `describe` | `const isabella = fixtureCompartido.isabella()` |
| `method initialize()` | configuración antes de cada test | `fixtureCompartido.prepararAgencia()` |
| `assert.equals(esperado, obtenido)` | valores | $2.550, 4 servicios |
| `assert.that` / `notThat` | booleanos | derrochón, puede contratar |
| `assert.throwsExceptionWithMessage(msg, { ... })` | errores | contratar a Los Tilos |
| Objeto anónimo | dobles de prueba (stubs) | fiestas de resguardo de los caterings (`object { method cantidadInvitados() = 40 }`) |

- **Independientes**: el ambiente se reinicia antes de cada test.
- **Un escenario por test**: el nombre describe la situación, no el número.
- **Regresión**: correr todos los tests después de cada cambio para detectar lo que se rompió.

## 9. Igualdad e identidad

- **`===`**: ¿el mismo objeto? Lo decide la máquina virtual, no se puede cambiar.
- **`==`**: ¿representan lo mismo? Por defecto es identidad, pero se puede redefinir con `override method ==(otro)`.
- En el TP no redefinimos `==` en ningún lado, y eso es una **decisión**: los proveedores se comparan por identidad (Silvina es una sola), y los servicios también (cada contratación es distinta).
- La fiesta de Tomás es **idéntica** (el mismo objeto) a la de Isabella.

---

## ⚠️ Cómo quedó la entrega (para saber qué contestar si lo preguntan)

**Lo que se resolvió:**

| Tema | Cómo quedó |
|---|---|
| Superclase para no repetir código en proveedores | `Proveedor` con `precioPorSector` y `costoBase`, heredada por `Salon`, `Organizador` y `Seguridad` |
| `1.2` en `Organizador` | ahora tiene nombre: `const property recargo = 1.2` |
| Tests de proveedores (con fiestas de resguardo y con las fiestas de los clientes) | están en `tests.wtest` |
| Tests de caterings "por una fiesta" | están, con objetos anónimos |
| Fixture compartido | objeto `fixtureCompartido` en `fixtureCompartido.wlk`, usado por un solo archivo de tests con varios `describe` |
| Tests duplicados | se eliminaron: quedan 47 tests en `tests.wtest` |

**Lo que quedó así a propósito, pero lo pueden preguntar:**

| Tema | Qué contestar | De quién |
|---|---|---|
| `EquipoDeMozos` no hereda de `Proveedor` | Según el README, porque no comparte `precioPorSector`. Se podría discutir que su "precio por sector" es `mozosPorSector × 50` | Matías |
| Los tests de proveedores por sector "con fiestas de resguardo" usan `new Fiesta(...)`, no objetos anónimos | La consigna pedía objetos anónimos; los de caterings sí lo son | Matías |
| El `if` de `Organizador` repite `self.costoBase(unaFiesta)` en las dos ramas | Lo que cambia es solo el recargo | Matías |
| `familia` y `facultad` siguen definidos en `example.wlk` | Ya no los usa ningún test: la fiesta de Isabella se arma con `SectorDeMesas` y `SectorDeGrupos` en el fixture | Leandro |
| La tabla de reparto del README es la de la entrega 1 | La consigna pide declarar el reparto en el README | el grupo |
