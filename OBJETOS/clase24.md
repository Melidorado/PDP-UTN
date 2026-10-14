# Clase 24 — Herencia, Super y Redefinición

### Módulos 13 y 14 — Paradigma Orientado a Objetos (Fernando Dodino)

---

## Módulo 13: Herencia. Super. Redefinición

### 1. Herencia

#### 1.1 Enunciado base

Un ornitólogo, además de las golondrinas tijerita (como pepita), quiere estudiar otras dos aves:

- el **petrel**
- la **torcaza**

Todas vuelan y comen igual que pepita, pero:

- del petrel interesa saber **cuántos kilómetros voló**,
- la torcaza es atolondrada: **antes de comer vuela en círculos 1 km**.

#### 1.2 Superclase

Las tres aves son polimórficas (entienden `comer` y `volar`) y además lo hacen de forma **muy parecida, pero no idéntica**. Como el comportamiento difiere, necesitamos tres clases (`GolondrinaTijerita`, `Petrel`, `Torcaza`), pero no queremos repetir el código común.

La solución es relacionar las clases: las tres heredan de una **superclase** (clase madre) `Ave`.

- La **superclase** representa un concepto más **general**.
- Las **subclases** representan **especializaciones** o casos particulares.

#### 1.3 Representación

En el diagrama de clases, la herencia se marca con una **flecha con triángulo cerrado** que va de la subclase a la superclase. Las subclases heredan los atributos y el comportamiento de `Ave`.

```
                    ┌─────────────────┐
                    │       Ave       │
                    ├─────────────────┤
                    │ energia         │
                    ├─────────────────┤
                    │ volar(km)       │
                    │ comer(gramos)   │
                    └────────▲────────┘
         ┌───────────────────┼──────────────────┐
┌────────┴─────────┐ ┌───────┴─────────┐ ┌───────┴─────────┐
│GolondrinaTijerita│ │     Petrel      │ │     Torcaza     │
├──────────────────┤ ├─────────────────┤ ├─────────────────┤
│                  │ │kilometrosVolados│ │                 │
├──────────────────┤ ├─────────────────┤ ├─────────────────┤
│ hacerAlgo(cosa)  │ │ volar(km)       │ │ comer(gramos)   │
└──────────────────┘ └─────────────────┘ └─────────────────┘
```

### 2. Herencia en Wollok: las aves

El comportamiento de una golondrina tijerita pasa a ser el comportamiento **por defecto** de un `Ave`:

- al volar, consume 1 joule por km + 10 joules fijos,
- al comer, gana 4 joules por gramo.

```wollok
class Ave {
  var energia = 50

  method volar(kilometros) {
    energia = energia - (kilometros + 10)
  }

  method comer(gramos) {
    energia = energia + gramos * 4
  }
}
```

#### 2.1 Golondrina tijerita

```wollok
class GolondrinaTijerita inherits Ave {
  method hacerAlgo(conCosa) { ... }   // comportamiento propio
}
```

Por heredar de `Ave`, una golondrina tijerita **tiene energía** y **sabe comer y volar**, y además puede agregar comportamiento propio.

```
> const pepita = new GolondrinaTijerita()
> pepita.volar(10)
```

#### 2.2 Method lookup con herencia

Se extiende el method lookup que veníamos usando:

1. Se busca el método en la **clase del objeto receptor** (`GolondrinaTijerita`).
2. Si no está, se sigue buscando en la **superclase** (`Ave`) → ahí está `volar(kilometros)`, y se evalúa con el estado de pepita.
3. Si tampoco está, se sigue subiendo hasta llegar a **`Object`** (la clase madre de todas las clases).
4. Si no está en `Object`, se produce un error:

```
> pepita.mugir()
wollok.lang.MessageNotUnderstoodException: un/a GolondrinaTijerita no entiende el mensaje mugir()
```

### 3. Segundo caso: el Petrel

El petrel come y vuela como un ave, y además registra los kilómetros que voló.

#### 3.1 Modelo y realidad

¿Es responsabilidad del petrel contar sus kilómetros, o del ornitólogo? Acá hay que **no confundir modelo y realidad**:

- En la **realidad**, el petrel vuela y el ornitólogo anota en un cuaderno.
- En el **modelo**, el petrel no es un petrel: es una **representación** simplificada (abstracción) que tiene sólo lo que nos sirve. Que "sepa" sus kilómetros es una decisión de diseño válida.

Ejemplos: el cuadro de Magritte _"Esto no es una pipa"_ (el dibujo de una pipa no es una pipa, es una representación), y Zoolander indignado porque la maqueta del centro para chicos "es para hormigas".

#### 3.2 Código del petrel

```wollok
class Petrel inherits Ave {
  var kilometrosVolados = 0

  override method volar(kilometros) {
    super(kilometros)
    kilometrosVolados = kilometrosVolados + kilometros
  }
}
```

```
> const unPetrel = new Petrel()
> unPetrel.volar(10)
> unPetrel.volar(20)     // kilometrosVolados = 30
```

El petrel tiene **`kilometrosVolados`** (definido en `Petrel`) **y también `energia`**, porque un Petrel **es un** Ave.

#### 3.3 Redefinición de métodos (`override`)

`override` indica que la subclase **redefine** un método que ya existe en la superclase: la nueva definición **pisa** a la de `Ave` para las instancias de `Petrel`.

Al enviar `unPetrel.volar(10)`, el lookup empieza en `Petrel`, que ahora sí tiene `volar` → se ejecuta ese.

#### 3.4 `super`

Queremos reutilizar el `volar` de `Ave` sin copiar su código. Para eso está la pseudo-variable **`super`**: hace que el method lookup **saltee la clase actual** y empiece directamente en la **superclase** de la clase donde está escrito el `super`.

Es especialmente útil dentro de un método redefinido: si en vez de `super(kilometros)` pusiéramos `self.volar(kilometros)`, el lookup volvería a encontrar el `volar` de `Petrel` → **loop infinito**.

#### 3.5 Lookup de `unPetrel.volar(10)`

1. Se busca `volar` en `Petrel` → lo encuentra (redefinido).
2. `super(kilometros)` → se busca `volar` a partir de `Ave` → lo encuentra y resta energía al petrel.
3. Vuelve al método de `Petrel` y suma los kilómetros.

### 4. Tercer caso: la torcaza

Vuela como un ave, pero antes de comer vuela 1 km. Entonces se redefine `comer`.

#### 4.1 `self`

Primera versión (mala): copiar la lógica de `volar` dentro de `comer`.

```wollok
class Torcaza inherits Ave {
  override method comer(gramos) {
    energia = energia - (1 + 10)   // copio el método volar 😬
    super(gramos)
  }
}
```

Mejor: mandarle el mensaje `volar(1)` **al mismo objeto receptor**, con `self`:

```wollok
class Torcaza inherits Ave {
  override method comer(gramos) {
    self.volar(1)
    super(gramos)
  }
}
```

| Pseudo-variable | ¿Dónde empieza el lookup?                                       | ¿Cuándo se usa?                                                     |
| --------------- | --------------------------------------------------------------- | ------------------------------------------------------------------- |
| `self`          | En la **clase del objeto receptor** (no cambia el lookup)       | Para enviarle mensajes al propio objeto                             |
| `super`         | En la **superclase de la clase donde está escrito** el `super`  | Para reutilizar el método de la superclase sin caer en loop infinito |

> En los dos casos **el objeto receptor es el mismo** (la torcaza). Lo único que cambia es desde dónde se empieza a buscar el método.

#### 4.2 Lookup de `comer` en la torcaza

- `self.volar(1)`: se busca `volar` en `Torcaza` → no está → se busca en `Ave` → lo encuentra.
- `super(gramos)`: se busca `comer` directamente en `Ave` → lo encuentra y lo aplica a la torcaza.

### 5. Más sobre herencia

#### 5.1 Clases abstractas y concretas

- **Clase abstracta**: sirve para agrupar comportamiento y atributos de sus subclases, pero **no se instancia** (como `Ave`).
- **Clase concreta**: tiene sentido instanciarla.

En Java se declara explícitamente (`public abstract class Ave`) y el compilador impide hacer `new Ave()`. En **Wollok no hace falta declararlo**: si nunca se hace `new Ave()`, `Ave` es abstracta "naturalmente".

¿Las superclases tienen que ser abstractas? **No**: una clase es concreta si tiene sentido instanciarla, sea superclase, subclase o una clase suelta. Pero una clase abstracta casi siempre tiene subclases (si no, no se puede instanciar ni agrupa ninguna jerarquía: no tendría sentido).

#### 5.2 Herencia vs. composición

|                | Herencia                                                 | Composición                                         |
| -------------- | -------------------------------------------------------- | --------------------------------------------------- |
| Relación       | Entre **clases** (estática)                              | Entre **instancias**: una conoce a la otra y le envía mensajes |
| Idea           | "**es un**"                                              | "**tiene un**"                                      |
| Interfaz       | La subclase hereda **toda** la interfaz de la superclase | Se expone sólo lo que se quiere                     |

Con **herencia simple**, la jerarquía tiene **un único punto de vista**: un `Perro`, ¿hereda de doméstico/salvaje, de vertebrado/invertebrado o de mamífero/ovíparo? Sólo se puede elegir uno. (¿Un `String` debería heredar de una colección o de algo comparable, como un `Ord` de Haskell?)

**Ejemplo: Pila.** ¿Una pila **es una** lista o **tiene una** lista?

- Si **hereda** de `Lista`, gana todo su comportamiento… y ese es el problema: una interfaz enorme cuando sólo necesita `push()`, `top()` y `pop()`.
- Si **tiene** una `Lista`, la implementación queda **encapsulada**: quien usa la pila sólo ve esos tres mensajes. Y queda libre la posibilidad de heredar de otra clase más afín a futuro, **sin gastar el único tiro** de la herencia.

#### 5.3 ¿Instancias o clases?

> "Hay tres tipos de descuento: el común es del 5%, el especial del 10% y para jubilados del 20%."

Primera idea: una clase abstracta `Descuento` con tres subclases.

```wollok
class Descuento {
  method porcentaje() = self.descuento() / 100
  method descuento()            // método abstracto: sin cuerpo
}

class DescuentoComun inherits Descuento {
  override method descuento() = 5
}
class DescuentoEspecial inherits Descuento {
  override method descuento() = 10
}
class DescuentoJubilados inherits Descuento {
  override method descuento() = 20
}
```

Un **método abstracto** no tiene cuerpo: sólo obliga a las subclases a implementarlo, y hace que la clase sea abstracta.

Pero las tres subclases **repiten la misma idea**; lo único que cambia es un número. Mejor: **una sola clase y tres instancias**.

```wollok
class Descuento {
  const descuento
  method porcentaje() = descuento / 100
}

const descuentoComun = new Descuento(descuento = 5)
const descuentoEspecial = new Descuento(descuento = 10)
const descuentoJubilados = new Descuento(descuento = 20)
```

Lo mismo con el ajedrez: ¿clases `Caballo`, `Alfil`, `Rey`… o 32 instancias de `Pieza`? Depende de si se mueven distinto (sí → clases).

> **Regla:** se usan **clases distintas cuando hay comportamiento distinto** (cuando el código es diferente). Si sólo cambian los valores, alcanza con **instancias distintas**.

#### 5.4 Cambiar la clase

Jerarquía `Persona` → `Alumno` / `Profesor`. Funciona… hasta que un alumno termina la cursada y pasa a ser ayudante: **una instancia no puede cambiar de clase** ni pertenecer a dos, y reemplazar todas las referencias al alumno por un profesor nuevo no es fácil.

Lo mismo con jerarquías basadas en características **temporales**: joven/adulto, feliz/aburrido/indignado, sin hijos/con hijos. **No conviene** subclasificar por eso.

> **Buena práctica:** armar la jerarquía según algo **intrínseco** del objeto, de forma que pertenezca a **una sola clase durante todo su ciclo de vida**.

### 6. Herencia de wko

Volvemos a pepita como wko. Aparece **chichita**, que vuela igual pero come distinto, y además hay **muchas otras aves** que se comportan igual que pepita. Pepita y chichita son importantes (queremos nombrarlas), pero las demás no tienen individualidad → alcanza con que sean instancias de `Ave`.

En Wollok **un wko puede heredar de una clase**:

```wollok
class Ave {
  var energia = 100

  method energia() = energia
  method volar(kms) { energia = energia - (kms + 10) }
  method comer(gramos) { energia = energia + (4 * gramos) }
}

object pepita inherits Ave { ... }

object chichita inherits Ave {
  override method comer(gramos) {
    energia = energia * gramos
  }
}
```

- `pepita` toma todo de `Ave` (y podría agregar comportamiento propio).
- `chichita` también, pero **redefine** `comer`. Como hereda de `Ave`, puede usar `energia` directamente.

#### 6.1 Method lookup de un wko con herencia

1. Se busca el método **en el propio wko**.
2. Si no está y el wko hereda de una clase, se busca **en esa clase**.
3. Si no está, se sigue por las **superclases** hasta encontrarlo o llegar a `Object` y fallar.

#### 6.2 Objetos y clases polimórficos

`pepita`, `chichita` y las instancias de `Ave` son **polimórficos**: todos entienden `energia()`, `volar(kms)` y `comer(gramos)`. Se les puede hablar sin importar si son wko o instancias:

```wollok
> pepita.volar(10)
> chichita.volar(5)
> const tweety = new Ave()
> tweety.volar(6)
> [pepita, new Ave(), tweety, chichita].forEach({ ave => ave.volar(5) })
```

### Resumen del módulo 13

Las clases se organizan **jerárquicamente**, de lo más general a lo más particular. La **herencia** permite **reutilizar** atributos y comportamiento y **redefinir** (`override`) en cada subclase lo que cambie. El **method lookup** empieza en la clase del receptor (o en el wko) y sube por las superclases hasta encontrar el método o fallar en `Object`. **`self`** envía un mensaje al propio objeto (lookup normal); **`super`** saltea la clase donde está escrito y arranca en su superclase.

Para modelar: subclasificar **sólo cuando hay comportamiento diferente**, elegir criterios **intrínsecos** (no temporales), y tener en cuenta la **composición** como alternativa a la herencia.

---

## Módulo 14: Mutabilidad. Igualdad e identidad

### 1. Mutabilidad / inmutabilidad

Para modelar un **punto** en un eje de coordenadas hay dos decisiones posibles:

- **Mutable**: con setters para `x` e `y`, el punto puede cambiar.
- **Inmutable**: una vez construido no cambia; representa **esa** ubicación y ninguna otra.

Con un punto inmutable, sumar dos puntos **construye un punto nuevo**:

```wollok
class Point {
  const property x
  const property y

  method +(otroPoint) =
    new Point(x = x + otroPoint.x(), y = y + otroPoint.y())
}
```

Es la misma estrategia que ya usan:

- los **números**: 2 + 3 no "cambia" al 2 en 5; el 5 es otro objeto,
- los **strings**: concatenar genera un string nuevo,
- los **booleanos**: existe un único `true` y un único `false`.

#### 1.1 Value objects

Todos estos son **value objects**: objetos que representan un **valor** del dominio. Otros ejemplos: un color (el rojo), un mail, una figura bidimensional (una lista de puntos), una fecha.

#### 1.2 Motivación

¿Por qué querer objetos inmutables? Porque **no tienen efecto colateral**. El paradigma lo permite, pero **elegimos** no usarlo: el paradigma está en quienes desarrollan.

Sin efecto colateral:

- el **testing** se simplifica (hay menos situaciones y contextos posibles),
- es más fácil **compartir** los objetos (incluso de forma concurrente): nadie puede modificarlos.

**Ejemplo:** dos personas que se llaman igual.

```wollok
class Persona {
  var property nombre
}

const pedro = new Persona(nombre = "pedro")
const pelusa = new Persona(nombre = "pedro")
```

```wollok
pedro.nombre("Pedro Tomás")
```

Si los strings fueran mutables y ambos compartieran el mismo `"pedro"`, ¡esto le cambiaría el nombre también a pelusa! Como los strings son inmutables, lo que pasa es que la referencia `nombre` de pedro **pasa a apuntar a otro string**, y pelusa sigue apuntando a `"pedro"`.

### 2. Igualdad e identidad

#### 2.1 Identidad (`===`)

Dos referencias son **idénticas** si apuntan **al mismo objeto**.

```wollok
class Mail {
  const property asunto
  const property emisor
  const property receptor
}

const invitacion10Oct = new Mail(asunto = "Asado?",
  emisor = "dodain@gmail.com", receptor = "alf@gmail.com")

const unMail = invitacion10Oct
```

`unMail` e `invitacion10Oct` son dos referencias al **mismo** objeto:

```
> unMail === invitacion10Oct
✓ true
```

#### 2.2 Igualdad (`==`)

A veces queremos decir que dos objetos **representan lo mismo** aunque **no sean el mismo objeto**. Eso es **igualdad**.

Otro día se manda un mail con el mismo asunto, emisor y receptor. ¿No es, en definitiva, el mismo mail?

```wollok
const invitacion17Oct = new Mail(asunto = "Asado?",
  emisor = "dodain@gmail.com", receptor = "alf@gmail.com")
```

```
> invitacion17Oct == invitacion10Oct
✓ false
```

**Por defecto, la igualdad funciona igual que la identidad**: dos referencias son iguales sólo si apuntan al mismo objeto.

| Consulta                                | Resultado | ¿Por qué?                                     |
| --------------------------------------- | --------- | --------------------------------------------- |
| `invitacion17Oct === invitacion10Oct`   | `false`   | Apuntan a objetos distintos                   |
| `unMail === invitacion10Oct`            | `true`    | Apuntan al mismo objeto                       |
| `invitacion17Oct == invitacion10Oct`    | `false`   | El `==` por defecto compara identidad 🤔      |

La gracia de tenerlos separados es que **el `==` se puede redefinir**: dos mails son iguales si tienen el mismo asunto, emisor y receptor.

```wollok
class Mail {
  const property asunto
  const property emisor
  const property receptor

  override method ==(otroMail) =
    asunto == otroMail.asunto() &&
    emisor == otroMail.emisor() &&
    receptor == otroMail.receptor()
}
```

```
> invitacion17Oct === invitacion10Oct
✓ false
> invitacion17Oct == invitacion10Oct
✓ true
```

No son el mismo objeto, pero **representan lo mismo**.

> **Identidad (`===`)**: ¿es el mismo objeto? Lo resuelve la VM, no se puede cambiar.
> **Igualdad (`==`)**: ¿representan lo mismo? Es un **punto de acuerdo** que podemos redefinir nosotros.

#### 2.3 ¿Igualdad o identidad?

La recomendación general es **preguntar siempre por igualdad**. Comparar identidad es raro, porque es un tema que maneja la máquina virtual. Casos borde:

- **Optimización**: si `==` es costoso (compara todos los valores) y se puede asegurar que existe una sola instancia, `===` es mucho más rápido. Ejemplo: el motor de Wollok Game, que procesa cambios visuales varias veces por segundo.
- **Objetos en dos ambientes** (aplicaciones distribuidas, navegador + servidor): el cliente y la "copia" que llegó por red son **dos objetos distintos** sí o sí, así que la identidad no sirve. Hace falta igualdad, y diseñar cómo saber que representan al mismo cliente.

### Resumen del módulo 14

Los objetos pueden ser **mutables** (sus referencias cambian) o **inmutables** (una vez construidos no cambian, no tienen efecto colateral). Los inmutables se pueden **compartir** sin cuidar modificaciones concurrentes y son más fáciles de **testear**. Los **value objects** (strings, números, fechas, puntos) son buenos candidatos a ser inmutables.

Cada objeto tiene una **identidad** que lo distingue del resto (`===`). La **igualdad** (`==`) por defecto es identidad, pero se puede redefinir para decir cuándo dos objetos representan lo mismo. Con value objects (strings, números, domicilios, puntos, booleanos) nunca conviene preguntar si son exactamente el mismo objeto: se pregunta por **igualdad**.

---

## Ideas que conectan los módulos

- La **herencia** (Módulo 13) agrega un paso más al **method lookup** que venimos ampliando desde [clase21.md](clase21.md) y [clase23.md](clase23.md): objeto → clase → superclases → `Object`.
- Redefinir `==` (Módulo 14) es un caso concreto de **`override`** (Módulo 13): `Mail` hereda el `==` de `Object` (que compara identidad) y lo pisa con su propia definición.
- La regla del Módulo 13 "**clases distintas sólo si hay comportamiento distinto**" (descuentos, piezas de ajedrez) es la misma idea del Módulo 8: si sólo cambian los valores, alcanza con instancias de una misma clase.
- La **inmutabilidad** (Módulo 14) conecta con el paradigma **funcional**: sin efecto colateral, como en Haskell, y con el **testing** (Módulo 11): menos estados posibles, tests más simples.
