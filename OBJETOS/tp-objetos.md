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
class Salon {
    var property precioPorSector
    method cuantoCobra(unaFiesta) = precioPorSector * unaFiesta.cantidadDeSectores() * unaFiesta.duracion()
}
// Organizador, EquipoDeMozos y Seguridad: misma idea, cada uno con su cuenta
```

Y los proveedores concretos son **instancias** (están al final de `example.wlk`):

```wollok
const losTilos = new Salon(precioPorSector = 500)
const escudo = new Seguridad(recargo = 1.1)
...
```

**Una clase por comportamiento, una instancia por proveedor**: Escudo y Silvina calculan igual, solo cambia el recargo → misma clase, dos instancias (clase 24, el ejemplo de los descuentos).

### ⚠️ Lo que encontraron los tests al juntar este punto con el mío

| Problema | Cómo se vio | Arreglo |
|---|---|---|
| Le mandaban a la fiesta `cantidadSectores()` y `cantidadHoras()` | `a Fiesta does not understand cantidadSectores()` | `cantidadDeSectores()` y `duracion()` |
| `Seguridad` no multiplicaba por sectores | Isabella quedaba con $2650 en vez de $2550 | `× unaFiesta.cantidadDeSectores()` |
| `EquipoDeMozos` no multiplicaba por horas | Los Pibes cobraban $200 en vez de $800 | `× unaFiesta.duracion()` |

Es el mejor ejemplo para la pregunta del README sobre los **tests de regresión**.

### ⚠️ Lo que queda pendiente (de Matías)

- **Código repetido**: las cuatro clases repiten "precio × sectores". La solución es una superclase `ProveedorPorSector` con la cuenta escrita una vez y dos métodos abstractos (`montoPorSector()` y `ajuste(fiesta)`). Ver el [Cierre, punto 2](#2-herencia-qué-se-comparte-y-qué-se-redefine).
- **`1.2` en `Organizador`** es un número mágico, lo mismo que nos marcaron a nosotros.
- **El `if` repite la cuenta entera** en las dos ramas. Lo correcto es que el `if` decida solo el recargo.

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
| `fiesta` | `const` | Un cliente organiza una fiesta y no la cambia. |
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
- **Es una pregunta**: siempre devuelve un booleano, nunca lanza error (clase 22, módulo 10, punto 10.2).
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
5. **El cliente no atrapa la excepción** (no hay `try/catch`). La clase 22 (punto 7): los objetos de dominio **lanzan, no atrapan**. Solo la atrapan los tests (o una interfaz de usuario).

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

Se podría tener `const proveedores = #{}` como atributo y hacer `proveedores.add(proveedor)` en `contratar`. Pero sería **guardar dos veces la misma información** (los proveedores ya están dentro de los servicios), y habría que mantener las dos colecciones sincronizadas. Es el mismo problema que la clase 21 marca con las relaciones bidireccionales. Calculándolo a partir de `servicios`, nunca se desincroniza.

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

    method fiesta() = fiesta

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

Están en `fixtureCompartido.wtest`, junto con los de la agencia (ver [punto grupal](#entrega-2--punto-grupal-la-agencia-el-fixture-y-el-readme)).

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

**¿Por qué no `self.esDerrochon()` en lugar de `super()`?**
Porque el lookup empezaría otra vez en `Fiestero` y se llamaría a sí mismo: loop infinito.

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

El **escenario inicial** que necesitan los tests: los objetos ya creados y configurados antes de que cada test empiece. Se arma **una sola vez** en el `describe`, en lugar de repetirlo en cada test (clase 23, módulo 11, punto 2).

```wollok
describe "Fixture compartido: fiestas, clientes y agencia" {
  // 1. Variables del describe
  const fiestaIsabella = new Fiesta(duracion = 4, sectores = [...])
  const fiestaMilena = new Fiesta(duracion = 6, sectores = [...])
  const fiestaDamian = new Fiesta(duracion = 8, sectores = [...])

  const isabella = new Cliente(ahorros = 2750, fiesta = fiestaIsabella, porcentajeDisponible = 0.6)
  const milena = new Cliente(ahorros = 2000, fiesta = fiestaMilena)
  const damian = new Fiestero(ahorros = 250000, fiesta = fiestaDamian)
  const tomas = new Fiestero(ahorros = 250000, fiesta = fiestaIsabella)

  // 2. initialize: la configuración que no es solo crear objetos
  method initialize() {
    agencia.incorporar(doniaRosa)
    ... (los diez)
  }

  // 3. Los tests
  test "..." { ... }
}
```

### Orden de ejecución de cada test (clase 23, punto 2.5)

1. Se **reinicia** el ambiente (incluida la agencia, que vuelve a estar vacía).
2. Se crean las **variables** del `describe`.
3. Se ejecuta el **`initialize`** (se incorporan los diez proveedores).
4. Se corre el **test**.

Por eso los tests son **independientes**: el que desvincula a Silvina no afecta al siguiente, que la vuelve a tener.

### ¿Por qué "compartido"?

Antes, cada integrante tenía sus propias fiestas en sus tests. La consigna pide **uno solo** para fiestas, clientes y agencia. Lo declarado en un `describe` **no se puede usar desde otro** (clase 23, punto 2.5), así que los tests que necesitan esas fiestas y clientes (agencia, contratación y regresión de Isabella) están todos en el mismo `describe`.

### Tomás: la fiesta "idéntica"

La consigna dice que la fiesta de Tomás *"es idéntica a la de Isabella"*. En el fixture, Tomás recibe **el mismo objeto** `fiestaIsabella`: son **idénticas** en el sentido de la clase 24 (`===` da `true`). Como contratar no modifica la fiesta, compartirla no trae problemas, igual que el barril compartido de la entrega 1.

## 4. Las respuestas del README

### ¿Dónde usaron herencia y qué comparten?

**`Fiestero` hereda de `Cliente`**: comparte todo (ahorros, fiesta, presupuesto, contratar, servicios, gasto, proveedores contratados) y redefine solo `esDerrochon()`, reutilizando la regla general con `super()`.

**Si Matías hace la superclase**, también `Salon`, `Organizador`, `EquipoDeMozos` y `Seguridad` heredan de `ProveedorPorSector`: comparten la cuenta "monto por sector × sectores × ajuste" y cada una define `montoPorSector()` y `ajuste(fiesta)`.

**Isabella no es subclase**: solo cambia un valor (el 60%), no un comportamiento.

### ¿Fue necesario ajustar los casos de los caterings de la entrega 1?

**No.** Siguen entendiendo `cuantoCobraTotal(cantidadInvitados)`, que no cambió. `cuantoCobra(fiesta)` se agregó **encima** y reutiliza ese cálculo. Los valores son los mismos ($1.100 por 40 invitados, etc.).

> ⚠️ La tabla de la entrega 2 pide probarlos "por una fiesta de 40 invitados" (con fiestas de resguardo). Hoy `testExample.wtest` prueba `cuantoCobraTotal(40)` directo. Si el corrector lo pregunta, hay que reconocerlo.

### ¿Y los de Isabella?

**Sí, pero solo en cómo se arma**, no en los resultados. Pasó de ser un wko con sus sectores a una instancia de `Cliente` con una `Fiesta`, y `puedeContratar` ahora le pregunta `cuantoCobra(fiesta)` a cualquier proveedor. Los valores esperados no cambiaron ($1.650, Doña Rosa sí, Pizzas Nico no, barril a $500, ahorra $2.000).

### ¿Cómo ayudan los tests de regresión?

Avisan si un cambio rompe algo que ya funcionaba, y dicen exactamente qué se esperaba y qué se obtuvo (clase 22, módulo 5). En este TP lo vivimos: al juntar los puntos, los tests de contratación detectaron los mensajes mal escritos y las cuentas mal hechas del punto 2, y los de Isabella confirmaron que el cambio de wko a `Cliente` no alteró los resultados.

---
---

# Cierre — Conceptos transversales (para repasar el día antes)

Todo el TP se apoya en un puñado de ideas que se repiten. Si entendés estas siete, podés defender cualquier punto.

## 1. Clase, instancia, wko u objeto anónimo

| Qué | Cuándo | En el TP |
|---|---|---|
| **wko** | Concepto único | `barril`, `doniaRosa`, `pizzasNico`, `elBodegon`, `agencia` |
| **Clase** | Varios objetos con el mismo comportamiento | `Cliente`, `Fiesta`, `Servicio`, `Salon`, ... |
| **Instancia** | Un objeto concreto de una clase, con su propio estado | `isabella`, `milena`, `losTilos`, `escudo`, ... |
| **Objeto anónimo** | Alcance acotado, típicamente un test | fiestas de resguardo del punto 2 |

**Regla de oro (clase 24)**: clases distintas solo si hay **comportamiento** distinto. Si solo cambian **valores**, instancias distintas de la misma clase.

## 2. Herencia: qué se comparte y qué se redefine

| | `Fiestero` → `Cliente` | `ProveedorPorSector` (pendiente, Matías) |
|---|---|---|
| Qué comparten | todo el cliente | la cuenta de `cuantoCobra` |
| Qué cambia | `esDerrochon()` | `montoPorSector()` y `ajuste(fiesta)` |
| Cómo | `override` + `super()` | métodos abstractos (sin cuerpo) + `override` |

```wollok
// Cómo quedaría la superclase de los proveedores
class ProveedorPorSector {
    method cuantoCobra(unaFiesta) = self.montoPorSector() * unaFiesta.cantidadDeSectores() * self.ajuste(unaFiesta)
    method montoPorSector()
    method ajuste(unaFiesta)
}

class Salon inherits ProveedorPorSector {
    const precioPorSector
    override method montoPorSector() = precioPorSector
    override method ajuste(unaFiesta) = unaFiesta.duracion()
}
```

**Clase abstracta**: `ProveedorPorSector` nunca se instancia con `new`, y tiene métodos sin cuerpo. En Wollok no hace falta declararla abstracta.

## 3. Method lookup, la versión completa

1. Se busca el método en el **wko** o en la **clase del receptor**.
2. Si no está, se sube a la **superclase**, y así hasta `Object`.
3. Si no está en ningún lado: `MessageNotUnderstoodException` ("does not understand").

| | ¿Dónde empieza a buscar? | Para qué |
|---|---|---|
| `self.mensaje()` | En la clase del **receptor** | Mandarse un mensaje a sí mismo |
| `super()` | En la **superclase** de la clase donde está escrito | Reutilizar el método que se redefinió |

En los dos casos **el receptor es el mismo objeto**; solo cambia desde dónde se busca.

## 4. Polimorfismo: un mensaje, muchos objetos

| Mensaje | Lo entienden | Lo envía |
|---|---|---|
| `cuantoCobra(fiesta)` | los 3 caterings (wko) + las 7 instancias de proveedores | `Cliente` (y a través de él, la agencia) |
| `invitadosSector()` | `SectorDeMesas`, `SectorDeGrupos` (y los wko `familia`, `facultad`) | `Fiesta` |
| `esDerrochon()` | `Cliente`, `Fiestero` | los tests |

**Polimorfismo no requiere herencia**: los caterings no heredan de nada y son polimórficos con los salones. Alcanza con **entender el mismo mensaje** (clase 23, módulo 8, punto 7).

## 5. Colecciones (solo lo que está en los apuntes)

| Mensaje | Qué hace | Dónde lo usamos |
|---|---|---|
| `add(x)` | agrega | `contratar`, `incorporar`, `proveedoresContratados` |
| `remove(x)` | saca | `desvincular` |
| `size()` | cuántos hay | `cantidadServicios`, `cantidadDeSectores` |
| `sum({ ... })` | suma lo que dice el bloque | `totalGastado`, `cantidadInvitados` |
| `filter({ ... })` | los que cumplen (colección nueva) | `esDerrochon`, `proveedoresQuePuedePagar` |
| `isEmpty()` | ¿está vacía? | `esDerrochon` |
| `forEach({ ... })` | hace algo con cada uno | `proveedoresContratados` |

**Lista `[]`** (orden, admite repetidos): `servicios`, `sectores`. **Conjunto `#{}`** (sin repetidos): proveedores de la agencia, `proveedoresContratados`.

`filter` y `map` **no modifican** la colección original: devuelven una nueva.

## 6. Excepciones

- **Se lanzan** con `self.error("mensaje")` cuando algo sale del flujo normal del negocio.
- **No se atrapan** en el dominio: solo en los tests y en la interfaz de usuario.
- **Pregunta ≠ acción**: `puedeContratar` siempre devuelve un booleano; `contratar` contrata o lanza error.
- **Validar antes de modificar**: así, si falla, el estado queda como estaba.
- **Lista vacía no es error**.

## 7. Tests

| Herramienta | Para qué | En el TP |
|---|---|---|
| `describe` + variables | el fixture | fiestas, clientes |
| `method initialize()` | configuración antes de cada test | incorporar los diez proveedores |
| `assert.equals(esperado, obtenido)` | valores | $2.550, 4 servicios |
| `assert.that` / `notThat` | booleanos | derrochón, puede contratar |
| `assert.throwsExceptionWithMessage(msg, { ... })` | errores | contratar a Los Tilos |
| Objeto anónimo | dobles de prueba (stubs) | fiestas de resguardo del punto 2 |

- **Independientes**: el ambiente se reinicia antes de cada test.
- **Un escenario por test**: el nombre describe la situación, no el número.
- **Regresión**: correr todos los tests después de cada cambio para detectar lo que se rompió.

## 8. Igualdad e identidad

- **`===`**: ¿el mismo objeto? Lo decide la máquina virtual, no se puede cambiar.
- **`==`**: ¿representan lo mismo? Por defecto es identidad, pero se puede redefinir con `override method ==(otro)`.
- En el TP no redefinimos `==` en ningún lado, y eso es una **decisión**: los proveedores se comparan por identidad (Silvina es una sola), y los servicios también (cada contratación es distinta).
- La fiesta de Tomás es **idéntica** (el mismo objeto) a la de Isabella.

---

## ⚠️ Lo que queda abierto en el grupo (para saber qué contestar si lo preguntan)

| Tema | Estado | De quién |
|---|---|---|
| Superclase `ProveedorPorSector` (código repetido en proveedores) | pendiente | Matías |
| `1.2` en `Organizador` (número mágico) | pendiente | Matías |
| Tests de proveedores (con objetos anónimos y con las fiestas de los clientes) | pendiente | Matías |
| Tests de caterings "por una fiesta" en lugar de por cantidad | pendiente | Leandro / Matías |
| `familia` y `facultad` siguen como wkos en los tests del punto 1 | funciona (son polimórficos con los sectores), pero el punto pedía clases | Leandro |
