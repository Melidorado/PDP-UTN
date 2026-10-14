# Clase 23 — Clases, Method lookup con clases y Testeo unitario avanzado

### Módulos 8, 10 y 11 — Paradigma Orientado a Objetos (Fernando Dodino y colaboradores)

> El **Módulo 10 (Manejo de errores)** también se vio en esta clase, pero ya está resumido completo en [clase22.md](clase22.md#módulo-10-manejo-de-errores). Acá sólo se retoma lo necesario para el testeo de errores (Módulo 11, punto 3).

---

## Módulo 8: Clases. Method lookup y polimorfismo con clases

### 1. Introducción al concepto de clase

Ejemplo: un comercio que vende libros (_El Principito_, _Martín Fierro_, _El Reino del Revés_). Si modelamos cada libro como un wko, queda así:

```wollok
object elPrincipito {
  var property precio = 100
  method esCaro() = precio > 150
  method aumentar(aumento) { precio = precio + aumento }
}

object martinFierro {
  var property precio = 100
  method esCaro() = precio > 150
  method aumentar(aumento) { precio = precio + aumento }
}

object elReinoDelReves { ... exactamente lo mismo ... }
```

Cada libro tiene **su propio precio**, pero el **comportamiento es idéntico** (cómo decide si es caro, cómo aumenta). Eso es código duplicado. La solución es definir ese comportamiento **una sola vez**, en una **clase**:

```wollok
class Libro {
  var property precio = 100
  method esCaro() = precio > 150
  method aumentar(aumento) { precio = precio + aumento }
}
```

#### Diagrama de clases (UML)

Una clase se representa con un rectángulo dividido en 3 partes:

```
┌──────────────────────┐
│        Libro         │  ← nombre de la clase
├──────────────────────┤
│ precio               │  ← referencias (atributos / variables de instancia)
├──────────────────────┤
│ esCaro()             │  ← métodos
│ aumentar(aumento)    │
└──────────────────────┘
```

### 2. Clase e instancia

La clase funciona como un **molde** (o un **sello**): a partir de ella se generan los objetos concretos, que se llaman **instancias**. Se crean con `new`:

```wollok
> const elPrincipito = new Libro()
✓
> elPrincipito.aumentar(80)
✓
> elPrincipito.esCaro()
✓ true
> const martinFierro = new Libro()
✓
```

La referencia `elPrincipito` apunta a **una instancia de `Libro`**.

### 3. Estructura de un objeto

Al instanciar, el objeto recibe **todas las referencias definidas en su clase** (su **estructura**), pero con **sus propios valores**. Por eso los atributos se llaman **variables de instancia**: cada instancia guarda su propio `precio`.

```wollok
> const martinFierro = new Libro()
> martinFierro.aumentar(50)
```

Ahora `elPrincipito` tiene precio 180 y `martinFierro` tiene 150 — misma estructura, distintos valores.

> **Clase** = comportamiento + estructura compartidos.
> **Instancia** = un objeto concreto, con su propio estado.

### 4. Method lookup con clases

Ahora las instancias **no tienen métodos propios**: el comportamiento está en la clase. Entonces el method lookup cambia:

1. Se envía `elPrincipito.esCaro()`.
2. Se busca el método **en la clase del receptor** (`Libro`) → encuentra `method esCaro() = precio > 150`.
3. Se ejecuta **con el estado del receptor**: el `precio` de `elPrincipito` es 180 → `true`.

Con `martinFierro.esCaro()` se encuentra **el mismo método**, pero se evalúa con **su** `precio` (150) → `false`.

> El método se busca en la **clase**; los atributos se resuelven en la **instancia**.

### 5. Clases y tipos

> Un **tipo** es un conjunto de mensajes que entiende un objeto.

Si consideramos **todos** los mensajes que entiende, el tipo lo define la clase. `Libro` define un tipo con:

- `precio()` y `precio(_precio)` (implícitos por `var property`)
- `esCaro()`
- `aumentar(aumento)`

Pero un mismo objeto puede tener **muchos tipos**. _El Principito_ puede verse como:

- un **libro**
- un **objeto que sabe decir si es caro** (tipo que comparte con un celular o un auto, aunque lo implementen distinto)
- un **objeto Wollok genérico** que sabe mostrarse por consola

Los tipos permiten encontrar similitudes entre clases que aparentemente no tienen conexión.

### 6. Polimorfismo con clases

El local agrega **DVDs** y **láminas**. Todos tienen precio, pero lo calculan distinto:

- **DVD**: precio = precio por minuto × cantidad de minutos. Es caro si dura más de 100 minutos.
- **Lámina**: precio = ancho × alto × precio base del material. Es cara si el precio base del material supera $50.

Como el **comportamiento es distinto**, necesitamos **clases distintas**:

```wollok
class DVD {
  var property precioMinuto = 10
  var property cantidadMinutos = 90

  method precio() = precioMinuto * cantidadMinutos
  method esCaro() = cantidadMinutos > 100
}

class Lamina {
  var property ancho = 29.7
  var property alto = 42
  var property material = new Material()

  method precio() = alto * ancho * self.precioBase()
  method esCaro() = self.precioBase() > 50
  method precioBase() = material.precio()
}

class Material {
  var property precio = 5
}
```

`self` se refiere al **objeto receptor** del mensaje: la instancia de `Lamina` que se está evaluando.

```wollok
const jurassicPark = new DVD()
const laGioconda = new Lamina()

jurassicPark.precio()
laGioconda.precio()
```

Ambos entienden `precio()`, aunque cada clase lo implemente distinto.

### 7. Tipos y polimorfismo

Libro, DVD y lámina son **polimórficos**: cada uno tiene su implementación, pero **quien les envía mensajes no nota la diferencia**:

```wollok
class Comprador {
  method comprar(algo) {
    if (algo.esCaro()) {
      ...
```

`algo` puede ser un libro, un DVD, una lámina, o cualquier otro objeto (wko o instancia de cualquier clase) que entienda `esCaro()`. Beneficios:

- **Bajo acoplamiento**: aparecen nuevos productos sin tocar al `Comprador`.
- **División de tareas**: una persona programa el `Comprador` y otra(s) los productos.
- El polimorfismo es **en un contexto** (la compra): no implica que se comporten igual en todo. Se inicializan distinto, y en este modelo sólo los libros pueden `aumentar` su precio.

### 8. Repaso: ¿objeto anónimo, wko o clase?

| Opción             | Cuándo conviene                                                                                                                                                                    | Ejemplo                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **Objeto anónimo** | Alcance acotado: un test, o resolver un requerimiento puntual. No importa que tenga nombre.                                                                                        | El bloque `{ taxista => taxista.viveEn(zona) }` de un `filter` |
| **wko**            | Representa un concepto de negocio **único**, con comportamiento específico que queremos modelar por separado.                                                                     | `pepita`, `monedero`                                 |
| **Clase**          | Hay **múltiples objetos** que comparten comportamiento y no tiene sentido nombrarlos por separado. Sólo difieren en los valores de sus referencias → el comportamiento va en un solo lugar. | Los viajes en colectivo, los libros                  |

### Resumen del módulo 8

La **clase** agrupa objetos con el **mismo comportamiento y la misma estructura**, aunque cada instancia guarde sus propios valores. Todo objeto pertenece a una clase durante toda su vida, y una clase puede tener **varios tipos** según los mensajes que define. Cuando objetos de **distintas clases** comparten un tipo, son **polimórficos en ese contexto**: un observador les envía los mismos mensajes sin percibir diferencias, aunque cada clase tenga su propio método.

---

## Módulo 11: Testeo unitario automatizado avanzado

### 1. Introducción

Ya conocemos `describe`, `test` y los tres resultados posibles (ok, falla, error — ver [clase22.md](clase22.md#módulo-5-introducción-al-testeo-unitario-automatizado)). Ahora vemos herramientas para mejorar la calidad de los tests.

### 2. Código compartido entre tests

Cuando los tests crecen, muchos comparten situaciones iniciales o acciones. Para **no repetir lógica** sin perder la independencia de cada test, Wollok ofrece tres herramientas dentro del `describe`.

#### 2.1 Ejemplo: Pepita con ciudades

Pepita ahora vuela a **ciudades**, cada una con un consumo distinto. Tiene una lista de ciudades favoritas y puede **cumplir su deseo**: volar a la más reciente de sus favoritas, que después deja de serlo.

```wollok
class Ciudad {
  const consumo
  method consumo() = consumo
}

object pepita {
  var energia = 100
  const ciudadesFavoritas = []

  method energia() = energia
  method esFuerte() = energia > 50

  method volar(ciudad) {
    energia = energia - (ciudad.consumo() + 10)
  }

  method comer(gramos) {
    energia = energia + 4 * gramos
  }

  method cumplirDeseo() {
    const destino = ciudadesFavoritas.last()
    self.volar(destino)
    ciudadesFavoritas.remove(destino)
  }

  method agregarCiudad(ciudad) { ciudadesFavoritas.add(ciudad) }
  method esFavorita(ciudad) = ciudadesFavoritas.contains(ciudad)
}
```

Escenario: pepita tiene como favoritas a Campana (consume 60) y San Martín (consume 5). En algún momento puede agregar a Quilmes (consume 1).

#### 2.2 Inicialización del `describe`

Antes de los tests se pueden declarar **variables y constantes**, con valores fijos, instancias nuevas o resultados de mensajes. Su alcance son todos los tests (y el `initialize` y los métodos del `describe`). Se pueden declarar constantes sin valor, siempre que se inicialicen en el `initialize`.

> **Importante:** estas referencias **se reinician antes de cada test**, igual que todos los objetos del ambiente. Los tests siguen siendo independientes.

#### 2.3 `initialize`

Para configuraciones más complejas que una simple asignación, se define un `method initialize()` (sin parámetros). Va después de las variables y antes de los tests, y **se ejecuta antes de cada test**.

#### 2.4 Métodos auxiliares

Para acciones que se repiten en **algunos** tests (no en todos) se definen métodos comunes, con cualquier nombre, con o sin parámetros, que se invocan con `self` desde un test, otro método o el `initialize`.

#### 2.5 Orden de ejecución de cada test

1. Reseteo del ambiente
2. Inicialización de las variables del `describe`
3. `initialize`
4. El test propiamente dicho

#### Solución final: dos `describe` separados

No todos los tests necesitan la misma inicialización: los de energía no necesitan las ciudades favoritas. Separarlos evita mezclar casos de prueba distintos y no penaliza con inicializaciones innecesarias.

```wollok
import pepita.*

describe "Tests de Pepita - ciudades" {
  const campana = new Ciudad(consumo = 60)
  const sanMartin = new Ciudad(consumo = 5)

  method initialize() {
    pepita.agregarCiudad(campana)
    pepita.agregarCiudad(sanMartin)
  }

  method realizarViajeLoco() {
    // quilmes sólo se usa acá: mejor definirla localmente
    const quilmes = new Ciudad(consumo = 1)
    pepita.cumplirDeseo()
    pepita.agregarCiudad(quilmes)
    pepita.cumplirDeseo()
    pepita.cumplirDeseo()
  }

  test "al inicio pepita tiene a San Martin por ciudad favorita" {
    assert.that(pepita.esFavorita(sanMartin))
  }

  test "pepita cumple su deseo y cambia su ciudad favorita" {
    pepita.cumplirDeseo()
    assert.notThat(pepita.esFavorita(sanMartin))
    assert.that(pepita.esFavorita(campana))
  }

  test "pepita hace su viaje loco y no tiene a campana como favorita" {
    self.realizarViajeLoco()
    assert.notThat(pepita.esFavorita(campana))
  }

  test "pepita cumple su deseo, al volar queda con menos energía" {
    pepita.cumplirDeseo()
    assert.equals(85, pepita.energia())
  }

  test "pepita hace su viaje loco y queda con una cierta energía" {
    self.realizarViajeLoco()
    assert.equals(4, pepita.energia())
  }
}

describe "Tests de Pepita - energía" {
  test "pepita comienza con 100 unidades de energía" {
    assert.equals(100, pepita.energia())
  }

  test "pepita vuela a una ciudad con consumo mayor a cero y ese consumo le quita energia" {
    const sanMartin = new Ciudad(consumo = 5)
    pepita.volar(sanMartin)
    assert.equals(85, pepita.energia())
  }

  test "pepita vuela a una ciudad que no tiene consumo, la energía baja un valor constante" {
    const ciudadSinConsumo = new Ciudad(consumo = 0)
    pepita.volar(ciudadSinConsumo)
    assert.equals(90, pepita.energia())
  }

  test "pepita vuela a una ciudad que le consume mucho, ya no es fuerte" {
    const campana = new Ciudad(consumo = 60)
    pepita.volar(campana)
    assert.notThat(pepita.esFuerte())
  }
  // ... comer 120 gramos → 580, comer 0 → 100, comienza siendo fuerte
}
```

Cuenta del viaje loco: 100 − (5+10) − (1+10) − (60+10) = **4**.

> Los métodos de un `describe` **no se pueden reutilizar desde otro `describe`**. Si hace falta compartir la creación de objetos entre varios, conviene poner esa responsabilidad en un **objeto aparte** (por ejemplo, uno que cree ciudades con poco o mucho consumo).

### 3. Testeo de errores

Retomando el monedero del Módulo 10 (arranca con $500, sólo montos positivos, no se puede sacar más de lo que hay):

| Acción                  | Verificación                                                     |
| ----------------------- | ---------------------------------------------------------------- |
| ninguna                 | El monedero tiene $500                                           |
| sacar $15               | Quedan $485                                                      |
| sacar $1000             | Error con mensaje "Debe retirar menos de 500"                    |
| sacar $-20              | Error con mensaje "La cantidad a retirar debe ser positiva"      |
| sacar "A"               | Error (cualquiera)                                               |
| poner $-20              | Error con mensaje "La cantidad a ingresar debe ser positiva"     |

```wollok
import monedero.*

test "estado inicial del monedero tiene 500 $" {
  assert.equals(500, monedero.plata())
}

test "cuando saco una cantidad posible de pesos, resta plata" {
  monedero.sacar(15)
  assert.equals(485, monedero.plata())
}

test "cuando quiero sacar más plata de la que tengo tira error" {
  assert.throwsExceptionWithMessage("Debe retirar menos de 500", { monedero.sacar(1000) })
}

test "cuando quiero sacar un monto negativo tira error" {
  assert.throwsExceptionLike(
    new UserException(message = "La cantidad a retirar debe ser positiva"),
    { monedero.sacar(-20) }
  )
}

test "cuando quiero sacar un valor alfabetico tira error" {
  assert.throwsException({ monedero.sacar("A") })
}

test "cuando quiero poner un monto negativo tira error" {
  assert.throwsExceptionWithMessage("La cantidad a ingresar debe ser positiva", { monedero.poner(-20) })
}
```

#### 3.1 Esperando un error

El código que debería fallar se pasa **dentro de un bloque** al `assert`:

- Si el bloque **tira la excepción esperada** → el `assert` la atrapa y el test **pasa**.
- Si el bloque **no tira excepción** → el test **falla** (es una mala señal: la validación no está funcionando).

Para comprobarlo, si se comenta la validación de montos negativos en `sacar`, el test de `sacar(-20)` falla.

| Mensaje                                         | Qué exige                                         | Restricción |
| ----------------------------------------------- | ------------------------------------------------- | ----------- |
| `throwsException(block)`                        | Que ocurra **algún** error                        | Mínima      |
| `throwsExceptionWithMessage(mensaje, block)`    | Error con **ese mensaje**                         | Media       |
| `throwsExceptionLike(excepcionEsperada, block)` | Mismo **tipo** de excepción **y** mismo mensaje   | Máxima      |

### 4. ¿Qué probar? ¿Cuánto probar?

Testear lleva tiempo y esfuerzo, así que la pregunta es si vale la pena probar absolutamente todo.

#### 4.1 Code coverage

Métrica (0 a 100) de **qué porcentaje del código** está cubierto por tests. No se busca el 100%: al acercarse, los tests **pierden valor** (se termina testeando getters, setters, lógica trivial). No hay número mágico: es cuestión de **equilibrio**. Se busca un porcentaje alto, pero llega un punto en que el esfuerzo extra es contraproducente.

#### 4.2 Clases de equivalencia

Si ya hay un test donde pepita vuela 5 km, ¿tiene sentido otro con 10 km? ¿Y con 6? **No**, mientras pepita quede con energía positiva: son el mismo caso. En cambio volar 91 km la deja en −1, una situación distinta que sí vale la pena probar.

Los casos de prueba forman **clases de equivalencia**: grupos de entradas que se comportan igual. Alcanza con un test representativo por clase. El **análisis de valores límite** ayuda a encontrarlas:

- `comer`: cantidad mayor a cero vs. cantidad nula.
- `esFuerte()` (`energia > 50`): una clase con energía **50 o menos**, otra con **51 o más**.
- Los **casos de error** también son una clase de equivalencia importante (como en el monedero).

#### 4.3 ¿Pruebas unitarias?

Una prueba es unitaria en tanto pruebe **una unidad funcional**. No siempre equivale a "una clase/objeto": pepita, el empleado o el monedero no dependían de otros objetos, pero ¿qué pasa al probar un cliente con facturas, que tienen ítems, cada uno con cantidad y producto?

Cuando hay **dependencias entre objetos** (los tests siguen siendo independientes entre sí), hay que elegir **hasta dónde** se testea: por ejemplo, separar cliente, factura y producto como tres unidades independientes. Aun así, armar el juego de datos no es fácil: ¿puedo testear el cliente si todavía no construí la factura? ¿Y la factura, si no terminé el producto?

Lo mismo pasa cuando:

- hay **efecto colateral** (bases de datos, envío de mails),
- hay procesos **no determinísticos** (errores de red, la hora del día, una temperatura),
- el **juego de datos es complejo** de construir,
- hay objetos **poco performantes**.

En esos casos conviene usar objetos **polimórficos** con los reales, pero con un comportamiento **simple, rápido, determinístico y fácil de configurar**.

#### Tests sociales vs. solitarios (Martin Fowler)

| Tipo          | Qué prueba                                                                 | En el ejemplo                                                         |
| ------------- | -------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **Sociales**  | La unidad **junto con sus dependencias reales**                            | El cliente usa facturas y productos reales (hay que esperar a que existan) |
| **Solitarios**| La unidad sola, usando **implementaciones de resguardo** para las dependencias | El cliente usa una factura "de prueba" → el test sigue siendo unitario |

#### Ejemplo: cliente con una factura de prueba

```wollok
class Cliente {
  const facturas = []

  method agregarFactura(factura) { facturas.add(factura) }
  method saldo() = facturas.sum { factura => factura.saldo() }
}

class FacturaPrueba {
  method saldo() = 100
}
```

La `FacturaPrueba` existe sólo para poder testear al cliente: si tiene una sola factura de prueba, su saldo tiene que ser 100.

```wollok
import cliente.*

describe "Test de clientes" {
  const cliente = new Cliente()
  const facturaPrueba = new FacturaPrueba()

  method initialize() {
    cliente.agregarFactura(facturaPrueba)
  }

  test "el saldo del cliente de una sola factura de prueba es el saldo de esa factura" {
    assert.equals(100, cliente.saldo())
  }
}
```

Se puede hacer **configurable** el saldo al instanciar, para tener varias facturas con montos distintos:

```wollok
class FacturaPrueba {
  const property saldo
  method estaPendiente() = true  // o self.saldo() > 0
}
```

```wollok
describe "Test de clientes" {
  const cliente = new Cliente()
  const facturaPrueba1 = new FacturaPrueba(saldo = 150)
  const facturaPrueba2 = new FacturaPrueba(saldo = 250)

  method initialize() {
    cliente.agregarFactura(facturaPrueba1)
    cliente.agregarFactura(facturaPrueba2)
  }

  test "el saldo del cliente suma el saldo de las facturas pendientes" {
    assert.equals(400, cliente.saldo())
  }
}
```

El test pasa **aunque todavía no exista la implementación real de `Factura`**.

#### Con objetos anónimos

También se pueden armar los objetos de resguardo con **objetos anónimos** (como en el Módulo 6), sin crear una clase:

```wollok
describe "Test de clientes" {
  const cliente = new Cliente()
  const facturaPrueba1 = self.facturaPrueba(150)
  const facturaPrueba2 = self.facturaPrueba(250)

  method facturaPrueba(saldo) = object {
    method estaPendiente() = true
    method saldo() = saldo
  }

  method initialize() { ... (idem) }
}
```

> Estas facturas de prueba son **objetos impostores** o **stubs**: dan una implementación de resguardo para poder testear al cliente de forma aislada.

### Resumen del módulo 11

El `describe` no sólo agrupa tests: permite definir **variables compartidas**, inicializadas en su definición, en un `initialize` o mediante **métodos auxiliares**, y que se reinician antes de cada test. También vimos cómo testear cuando lo esperado es un **error** (`throwsException` y variantes).

Diseñar casos de prueba no es mecánico: no hay metodología que garantice buenos tests. Métricas como el **code coverage** ayudan, pero son una medida, no un fin. Las **clases de equivalencia** y los **valores límite** permiten elegir pocos tests representativos. Y para mantener la **unitariedad** frente a dependencias, se usan **objetos de resguardo** (stubs o mocks), polimórficos con los reales.

---

## Ideas que conectan los módulos

- Las **clases** (Módulo 8) cambian el **method lookup**: ya no se busca el método en el objeto sino en su clase, pero el estado sigue siendo de cada instancia. Es la misma idea de [clase21.md](clase21.md) (mensaje ≠ método), con un paso más.
- El **polimorfismo** ya no es sólo entre wkos: instancias de **distintas clases** pueden ser polimórficas si comparten un tipo, y el que las usa (el `Comprador`) no necesita saber cuál es cuál.
- En el testeo (Módulo 11) las clases simplifican armar el juego de datos: en vez de definir un wko por cada ciudad, se instancian con `new Ciudad(consumo = ...)` dentro del `describe` o del test.
- El **manejo de errores** (Módulo 10) y el **testeo de errores** (Módulo 11) van de la mano: los tests son uno de los pocos lugares donde tiene sentido atrapar excepciones, y los casos de error son una clase de equivalencia que siempre conviene cubrir.
- Los **stubs** (Módulo 11) son una aplicación directa del **polimorfismo**: el `Cliente` no distingue una factura real de una de prueba, porque ambas entienden `saldo()`. Se pueden armar con una clase (`FacturaPrueba`) o con un **objeto anónimo**, según lo que convenga (ver la tabla del Módulo 8, punto 8).
