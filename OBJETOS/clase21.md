# Clase 1 — Objeto, Mensaje y Polimorfismo

### Módulos 1, 2 y 3 — Paradigma Orientado a Objetos (Fernando Dodino)

---

## Módulo 1: Objeto, mensaje, métodos

### 1. ¿Qué es un objeto?

Un **objeto** es la abstracción principal del paradigma. Representa una entidad a través de un modelo mental: no trabajamos con la "cosa real" sino con una **simplificación** que hacemos de ella según lo que nos interesa resolver. Por eso dos personas pueden mirar el mismo objeto del mundo real y construir representaciones mentales distintas: cada una abstrae lo que le sirve para su objetivo.

Esto vale también para **objetos conceptuales**, no solo para cosas físicas: una deuda entre dos personas es un objeto, un `String` es un objeto, un thread corriendo en la PC es un objeto. Todo lo que puede pensarse como una idea puede representarse como un objeto.

De acá surge la definición central del paradigma:

> **"Un sistema es un conjunto de objetos que se envían mensajes para alcanzar un determinado objetivo."**

Es una idea muy cercana a la Teoría General de Sistemas ("conjunto de partes que se relacionan para un objetivo común"): lo importante no es cada objeto aislado, sino la red de interacciones entre ellos.

### 2. Composición de un objeto: la interfaz

Lo que nos importa de un objeto de cara al exterior es **qué mensajes le puedo enviar** — eso es su **interfaz**. Por ejemplo, el gato Félix "sabe" maullar, comer y dormir: esos tres mensajes conforman su interfaz pública. No nos interesa (desde afuera) cómo hace cada una de esas cosas, sino que las puede hacer.

### 3. Mensaje

Los objetos interactúan enviándose **mensajes**. Para que un objeto A le pueda enviar un mensaje a un objeto B:

1. A tiene que **conocer** a B (tener una referencia hacia él).
2. Conociéndolo, A sabe qué mensajes puede mandarle (su interfaz).
3. A **no necesita saber cómo** B resuelve ese mensaje.
   Ese último punto es clave y tiene un nombre: **abstracción**. Si yo supiera (y dependiera de) cómo Pepita vuela, quedaría **acoplado** a esa implementación: si Pepita cambia su forma interna de volar, yo podría romperme. Al no asumir nada sobre el "cómo", mi código se vuelve más resistente a los cambios internos de los objetos con los que hablo.

Ejemplo (Pepita, un pájaro):

- `energia` → ¿cuál es tu energía?
- `volá` → volá
- `comé` → comé
- `reiniciate` → volvé a tu estado inicial
  Hay **dos posiciones** posibles frente a estos mensajes:
- **Como observador** (quien envía el mensaje `volá` a Pepita): me importa _que_ vuele, no _cómo_ lo hace. Estoy abstrayendo.
- **Como Pepita** (quien recibe el mensaje): sé cómo volar, pero esa lógica es mía, interna. No se la cuento a nadie; simplemente respondo cuando me la piden. Esto es **encapsulamiento**: agrupo mi propia funcionalidad. Importante: encapsular _no_ es "ocultar para proteger" — la implementación puede verse si hace falta —, sino **simplificarle las cosas al cliente** (cliente = quien me envía mensajes).

### 4. Ambiente

Los objetos necesitan "vivir" en algún lado: ese lugar es el **ambiente** (también llamado _imagen_, _virtual machine_ o _engine_ en otras tecnologías). Es el espacio donde coexisten los objetos y donde, para poder enviarse mensajes entre sí, primero deben conocerse.

### 5. Referencias

El mecanismo por el cual un objeto "conoce" a otro es la **referencia**, y de ahí sale el concepto de **variable**: una variable apunta (referencia) a un objeto, y tiene un nombre — la forma en la que el observador se dirige al objeto observado.

Un mismo objeto puede tener **varios nombres distintos** según quién lo mire: para mí puede ser `pepita`, para otro observador puede ser `ave`, pero ambas referencias apuntan **al mismo objeto**. Este punto se retoma y profundiza en el Módulo 2 con el concepto de identidad.

### 6. Mensaje y método

Cuando se envía un mensaje, en el software **se ejecuta código**. Ese código vive en el objeto que **recibe** el mensaje, y se llama **método**.

> **Mensaje** ≠ **método**.
>
> - El **mensaje** es la orden que el emisor le manda al receptor: el emisor no sabe (ni le importa) cómo se resuelve, solo pide que se resuelva. Es el **qué**, no el **cómo** (la misma idea de abstracción del punto 3).
> - El **método** es el código concreto que el receptor ejecuta para responder a ese mensaje. Es el **cómo**.

#### 6.1 Method lookup

**Method lookup** es la estrategia que usa el lenguaje para encontrar, dado un mensaje, cuál es el método que hay que ejecutar. En el caso más simple (el que vimos hasta acá), el lookup busca directamente el método en el propio objeto receptor.

#### 6.2 Acciones y preguntas

Los mensajes se pueden clasificar en dos tipos, según qué es lo que nos importa de la respuesta:

| Tipo         | Ejemplo                                      | Qué importa                                         |
| ------------ | -------------------------------------------- | --------------------------------------------------- |
| **Pregunta** | `pepita.energia()` → `10`                    | El **resultado** que devuelve                       |
| **Acción**   | `pepita.vola(10)` → Pepita se desplaza 10 km | El **efecto colateral** que produce sobre el objeto |

Ejemplos de preguntas: `pepita, ¿cuál es tu energía?`, `pepita, ¿sos feliz?`, `pepita, ¿cuál es tu nombre?`.
Ejemplos de acciones: `pepita, ¡volá!`, `pepita, ¡comé!`.

### Resumen del módulo 1

Un objeto representa un concepto y define su **interfaz** (los mensajes que entiende). Los objetos se comunican enviándose **mensajes**, sin conocer la implementación del otro (abstracción/encapsulamiento). Viven dentro de un **ambiente**, y se conocen entre sí a través de **referencias**. Cada mensaje dispara la ejecución de un **método** en el receptor, y la forma de encontrar ese método se llama **method lookup**.

---

## Módulo 2: Referencias, Estado, Compartir objetos, Identidad

### 1. Repaso

- **Mensaje / método**: define de qué lado estoy parada. Si envío un mensaje, soy usuaria del objeto (no me interesa el cómo). Si ejecuto un método, soy el objeto mismo, corriendo código propio, más allá de quién me lo haya pedido.
- **Interfaz / implementación**: la interfaz es lo que el objeto publica para que lo usen (el qué); la implementación es lo que encapsula para resolverlo (el cómo).

### 2. Estado de un objeto

El **estado** de un objeto es el conjunto de referencias que tiene guardadas. Cada referencia es un **atributo**: tiene un nombre y un valor, que es el objeto al que apunta.

Un principio de diseño importante: el estado de un objeto debería ser modificado **únicamente por métodos definidos en el propio objeto** — nadie de afuera debería poder ir a "tocar" directamente un atributo.

### 3. Introducción a Wollok

**Wollok** es un lenguaje con fines didácticos:

- No obliga a declarar tipos.
- Es interpretado.
- Tiene un IDE propio.
  Se inicializa un proyecto con:

```
wollok init --project aves
```

Esto genera un `example.wlk` que se puede renombrar.

#### Definir un objeto en Wollok

```wollok
object pepita {
  var energia = 100

  method volar() {
    energia = energia - 10
  }

  method comer(cantidad) {
    energia = energia + 2 * cantidad
  }
}
```

- `object pepita { ... }`: define un objeto llamado `pepita`. Adentro van sus atributos y su comportamiento.
- No hay tipos declarados ni en parámetros ni en métodos. Si un objeto no entiende un mensaje que le mandan, **falla el method lookup** y salta un error.
- `var energia = 100`: define un atributo `energia` con valor inicial `100`.
- `method volar()`: método sin parámetros, que modifica `energia`.
- `method comer(cantidad)`: recibe un parámetro `cantidad`, usado para calcular cuánto aumenta la energía.

### 4. Accessors

Como el estado no debería tocarse desde afuera, se usan mensajes especiales llamados **accessors**:

- **getter**: publica el valor de la referencia. Es un método de una sola línea que **devuelve** algo.
- **setter**: modifica la referencia. **No** necesita devolver nada.

```wollok
object pepita {
  ...
  // getter
  method energia() { return energia }

  // setter
  method energia(_energia) { energia = _energia }
}
```

Esta es la convención de accessors en Wollok: mismo nombre que el atributo, y el setter se distingue por recibir un parámetro.

### 5. Primeras pruebas en Wollok

Se pueden probar los objetos desde el REPL (`F1 > Wollok: Start a new REPL session`, o _Run in REPL_):

```
> pepita.energia()
✓ 100
> pepita.volar()
✓
> pepita.energia()
✓ 90
> pepita.energia(170)
✓
> pepita.energia()
✓ 170
```

Sintaxis general: `objeto.mensaje()`, `objeto.mensaje(parametro1)`, `objeto.mensaje(parametro1, parametro2)`.

### 6. Referencias en Wollok

Hay dos formas de declarar una referencia:

| Palabra clave | Significado                                                      |
| ------------- | ---------------------------------------------------------------- |
| `var`         | Referencia **variable**: puede reasignarse a otro objeto         |
| `const`       | Referencia **fija**: una vez inicializada, no se puede reasignar |

```wollok
var age = 10
age = 11
age = age + 1

const adultAge = 21
adultAge = 18  // ESTO DA ERROR
```

### 7. Relaciones bidireccionales

Escenario: Juan tiene una mascota, Firulais. Si solo `juan` guarda una referencia a `firulais`, la relación es **unidireccional**: Juan conoce a Firulais, pero Firulais no conoce a Juan.

```wollok
object juan {
  var mascota

  method mascota(_mascota) { mascota = _mascota }
}

object firulais {
  var duenio

  method duenio(_duenio) { duenio = _duenio }
}
```

```
> juan.mascota(firulais)
> firulais.duenio(juan)
```

Para que la relación sea **bidireccional**, necesitamos que ambos objetos tengan una referencia mutua. Esto es cómodo (cualquiera de los dos puede enviarle un mensaje al otro), pero **exige mantener ambas referencias sincronizadas** para no caer en inconsistencias (por ejemplo, que Juan tenga a Firulais como mascota, pero Firulais no tenga a Juan como dueño).

Una forma de reducir ese riesgo es que **uno de los objetos sea responsable de actualizar también la referencia del otro**, usando `self` (la referencia al propio objeto dentro de un método):

```wollok
object juan {
  var mascota

  method mascota(_mascota) {
    mascota = _mascota
    mascota.duenio(self)
  }
}

object firulais {
  var duenio

  method duenio(_duenio) { duenio = _duenio }
}
```

```
> juan.mascota(firulais)
```

Con una sola llamada (`juan.mascota(firulais)`) quedan sincronizadas ambas referencias. La desventaja: si alguien le manda directamente `duenio` a una mascota (sin pasar por `juan.mascota(...)`), puede quedar una referencia sin definir del otro lado — la sincronización sigue dependiendo de _cómo_ se invoque.

### 8. Compartir valores

Un mismo objeto puede ser referenciado por **más de un objeto** a la vez. Si Juan y Tefi quieren compartir actividades con Firulais, simplemente le pasamos la misma referencia a ambos:

```wollok
object tefi {
  method jugarCon(_mascota) { ... }
}

object juan {
  method alimentarA(_mascota) { ... }
}
```

```
> tefi.jugarCon(firulais)
> juan.alimentarA(firulais)
```

`firulais` es un único objeto compartido por dos referencias distintas.

### 9. Referencias globales

Objetos como `juan`, `tefi` y `firulais`, al estar definidos con `object nombre { ... }`, son objetos autodefinidos o **well-known objects (wko)**: cualquier otro objeto puede enviarles un mensaje directamente, sin necesidad de guardar una referencia propia a ellos:

```wollok
object firulais {
  method jugar() { ... }
}

object juan {
  method jugarConMascota() {
    firulais.jugar()
  }
}
```

Esto **no es buena práctica**: al acceder directamente por el nombre global, Juan queda "atado" para siempre a jugar únicamente con `firulais` — no hay forma de cambiar esa referencia dinámicamente. Acceder a un objeto global es tentador porque ahorra código, pero **genera soluciones rígidas**. Por eso, aunque parezca más burocrático, siempre es preferible tener una referencia propia al objeto (como parámetro, o como atributo `var`/`const`).

### 10. Identidad

Cada objeto, dentro de un ambiente, **sabe que es él mismo y ningún otro** — por eso es el único responsable de resolver los mensajes que recibe. Esto da lugar al concepto de **identidad**:

> Dos referencias son **idénticas** si apuntan al mismo objeto.

En el ejemplo de Juan y Firulais con relación bidireccional, las referencias `juan` y `duenio` (dentro de Firulais) son idénticas, porque ambas apuntan al mismo objeto Juan.

### Resumen del módulo 2

Los objetos tienen **estado interno**, formado por atributos que son referencias hacia otros objetos. Esas referencias pueden ser **variables** (`var`, reasignables) o **constantes** (`const`, fijas). Para que dos objetos se conozcan mutuamente hace falta una referencia en cada dirección (relación **bidireccional**), lo cual exige mantenerlas sincronizadas. Un mismo objeto puede ser **compartido** por varias referencias. Usar objetos autodefinidos como **referencias globales** genera acoplamiento rígido y conviene evitarlo. Y cada objeto tiene **identidad**: dos referencias son idénticas si apuntan al mismo objeto.

---

## Módulo 3: Introducción a Polimorfismo

### 1. La idea central

¿Qué pasa si **dos objetos distintos son capaces de responder al mismo mensaje**? Entonces quien envía ese mensaje puede cambiar de qué objeto está hablando (cambiar la referencia) **sin que el envío del mensaje tenga que cambiar**.

Esto requiere **tres objetos**:

- Uno que **envía** el mensaje (el observador).
- Dos (o más) que **implementan ese mensaje cada uno a su manera** (cada uno con su propio método).
  Decimos que esos dos objetos, que comparten al menos una parte de su interfaz, son **polimórficos** para ese observador.

> **Importante:** no hace falta que los objetos polimórficos definan exactamente el mismo conjunto de métodos — alcanza con que compartan **al menos un mensaje en común**.

### 2. Analogía: el botón Play

Un iPod, un reproductor de DVD y una videocasetera entienden todos el mensaje `play()`:

```
> ipod.play()
> dvdLiving.play()
> vhsRecorder.play()
```

Cada uno hace algo distinto al recibir ese mensaje (sus implementaciones difieren), y ninguno puede necesariamente hacer lo que hacen los otros (un reproductor de video no graba, no se puede "ir a la pista 8" de un cassette). Lo valioso del polimorfismo **no es para los objetos en sí**, sino **para quien los usa**: puede tratarlos de manera uniforme sin preocuparse por sus diferencias internas — la misma razón de fondo que la abstracción del Módulo 1 (no me interesa el "cómo").

### 3. Ejemplo: empleados de una empanadería

Dos empleados con formas de cobro distintas:

- **Galván**: sueldo fijo (hoy $15.000, modificable).
- **Baigorria**: cobra $15 por cada empanada vendida.

```wollok
object galvan {
  var sueldo = 15000

  method sueldo() = sueldo
  method sueldo(_sueldo) { sueldo = _sueldo }
}

object baigorria {
  var cantidadEmpanadasVendidas = 100
  var montoPorEmpanada = 15

  method venderEmpanada() {
    cantidadEmpanadasVendidas = cantidadEmpanadasVendidas + 1
  }

  method sueldo() = cantidadEmpanadasVendidas * montoPorEmpanada
}
```

```
> baigorria.sueldo()
✓ 1500
> baigorria.venderEmpanada()
✓
> baigorria.sueldo()
✓ 1515
> galvan.sueldo()
✓ 15000
```

Ambos objetos entienden `sueldo()` sin parámetros — comparten esa parte de su interfaz —, aunque lo calculan de maneras completamente distintas. Notar que **no** comparten toda la interfaz: a `baigorria` no se le puede asignar el sueldo directamente, porque no define `sueldo(_sueldo)`:

```
> baigorria.sueldo(29)
✗ Evaluation Error!
  wollok.lang.MessageNotUnderstoodException: baigorria does not understand sueldo(29)
```

Agregamos un tercer objeto, `gimenez`, que le paga a cualquiera de los dos empleados:

```wollok
object gimenez {
  var dinero = 300000

  method dinero() = dinero
  method pagarA(empleado) { dinero = dinero - empleado.sueldo() }
}
```

El método `pagarA(empleado)` envía el mensaje `sueldo()` sobre lo que recibe como parámetro, **sin saber de antemano** si `empleado` va a ser `baigorria` o `galvan`. Eso se decide recién en tiempo de ejecución, según qué objeto concreto se le pase:

```
> gimenez.pagarA(baigorria)
✓
> gimenez.dinero()
✓ 298485
```

Si en cambio se llamara `gimenez.pagarA(galvan)`, el mismo código `empleado.sueldo()` terminaría ejecutando el método `sueldo()` de `galvan`. El **mensaje enviado** (`empleado.sueldo()`) es siempre el mismo; el **método que efectivamente se ejecuta** depende del objeto real que ocupe ese lugar — eso es polimorfismo en acción.

### 4. Polimorfismo y tipos

Volviendo a `method pagarA(empleado) { dinero = dinero - empleado.sueldo() }`: ¿de qué tipo es `empleado`? En principio, de **cualquier objeto que entienda el mensaje `sueldo()` sin parámetros y devuelva un número**. No se trata de una jerarquía de tipos declarada explícitamente: el sistema de tipos de Wollok, en este ejemplo, reconoce que hoy son dos los objetos posibles que cumplen esa condición (`baigorria` y `galvan`), pero el criterio de fondo es **comportamental**: "entender el mensaje", no "pertenecer a una clase declarada".

### Resumen del módulo 3

El **polimorfismo** permite agrupar objetos que tienen comportamientos (implementaciones/métodos) distintos pero comparten al menos parte de su **interfaz**. Gracias a esto, puedo enviar un mensaje sin saber cómo lo resuelve el objeto receptor, e incluso sin saber de antemano **a qué objeto concreto** le estoy hablando — puedo cambiar la referencia sin tener que tocar el código que envía el mensaje.

---

## Ideas que conectan los tres módulos

- Todo el paradigma gira alrededor de separar **el qué** (mensaje, interfaz) **del cómo** (método, implementación) — esto es lo que permite la **abstracción**.
- Las **referencias** son el mecanismo que conecta objetos entre sí, y determinan tanto el **estado** (Módulo 2) como la posibilidad de que una misma referencia apunte, en distintos momentos, a distintos objetos que responden al mismo mensaje de formas distintas (**polimorfismo**, Módulo 3).
- El **method lookup** (Módulo 1) es lo que hace posible el polimorfismo: en tiempo de ejecución, el lenguaje busca el método correspondiente **en el objeto real que está recibiendo el mensaje**, no en base a lo que "dice" el código en el punto de envío.
