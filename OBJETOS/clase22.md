# Clase 22 — Objetos básicos, Testeo unitario, Objetos anónimos y Manejo de errores

### Módulos 4, 5, 6 y 10 — Paradigma Orientado a Objetos (Fernando Dodino y colaboradores)

---

## Módulo 4: Objetos básicos — Números, Strings, Booleanos, Fechas, Bloques, Colecciones

Wollok viene con una serie de **objetos básicos** ya definidos, que sirven como bloques de construcción para nuestras propias soluciones.

### 1. Números

Los números son **objetos inmutables**: no cambian su estado interno. `1 + 2` no modifica a `1`, sino que produce **un nuevo número**, `3`.

```wollok
const a = 1
var b = a + 10  // suma
b = b - 1       // resta
b = b * 2       // multiplicación
b = b / 2       // división
b = b % 2       // resto
b = b ** 3      // potencia

5.between(2, 7)        // ¿5 está entre 2 y 7? ==> true
3.min(6)                // menor entre 3 y 6 ==> 3
3.max(6)                // mayor entre 3 y 6 ==> 6
(3.1416).truncate(0)    // parte entera ==> 3
(3.1416).truncate(2)    // ==> 3.14
(3.1416).roundUp(0)     // primer entero mayor ==> 4
(3.1416).roundUp(2)     // ==> 3.15
```

#### Números con decimales

Por defecto Wollok trabaja con **5 decimales como máximo**, y redondea hacia arriba si el último decimal es ≥ 5 (si no, hacia abajo). Al imprimir, se eliminan los ceros no significativos:

```
> 5.2912413
✓ 5.29124
> 5.2912488
✓ 5.29125
> 1.000005 - 1.000004
✓ 0.00001
> 1.000002 - 1.000001
✓ 0
```

### 2. Booleanos

Hay dos objetos booleanos: `true` y `false`, también **inmutables**. Operadores lógicos: `and`, `or`, `not` (o sus equivalentes con símbolos `&&`, `||`, `!`, para quienes vienen de otros lenguajes).

```wollok
const hecho = true and true
const seraFalse = true and false
const seraTrue = true or false
const seraTrue2 = not false
```

### 3. Strings

Se delimitan con comillas simples o dobles, y también son **inmutables**: concatenar `"hola"` y `"mundo"` produce un String **nuevo**, no modifica a los originales.

```wollok
const unString = "hola"
const otroString = 'mundo'
const holaMundo = unString + " " + otroString + " !"
```

### 4. Fechas

Una fecha es un objeto **inmutable** que representa día, mes y año (sin horas ni minutos). Se puede crear con la fecha actual o con una fecha específica:

```wollok
const hoy = new Date()
const unDiaCualquiera = new Date(day = 24, month = 11, year = 2017)
```

Operaciones típicas (todas devuelven una fecha **nueva**, o algún otro valor, sin modificar la original):

```
> hoy.plusYears(1)    // suma un año
> hoy.plusMonths(2)   // suma 2 meses
> hoy.plusDays(20)    // suma 20 días
> hoy.isLeapYear()    // ¿es bisiesto?
> hoy.dayOfWeek()     // día de la semana
> hoy.month()
> hoy.year()
> const ayer = hoy.minusDays(1)
> ayer < hoy          // comparación
> ayer - hoy          // diferencia en días
> ayer.between(haceUnMes, hoy)
```

### 5. Lambdas: objetos bloque

Wollok permite representar un **bloque de código como un objeto**, de manera de poder guardarlo en una referencia, pasarlo como parámetro, y decidir en qué momento ejecutarlo (más adelante, condicionalmente, repetidas veces, etc.).

#### 5.1 Ejemplo básico

```wollok
> const abs4 = { => 4.abs() }
✓
> abs4.apply()
✓ 4
```

Se crea el bloque (la lambda) en un momento, y se ejecuta —con `apply()`— en otro.

#### 5.2 Bloques con un parámetro

```wollok
> const abs = { numero => numero.abs() }
> abs.apply(-8)
✓ 8
```

#### 5.3 Bloques con dos parámetros

```wollok
> { num1, num2 => num1.max(num2) }.apply(4, 2)
✓ 4
> { num1, num2 => num1 + num2 }.apply(4, 2)
✓ 6
```

También se puede currificar, como en el paradigma funcional:

```wollok
> { num1 => { num2 => num1 + num2 }}.apply(4)
✓ { num2 => num1 + num2 }
> { num1 => { num2 => num1 + num2 }}.apply(4).apply(2)
✓ 6
```

#### 5.4 Contexto de los bloques

Un bloque (closure) no solo accede a sus propios parámetros, sino también a **cualquier referencia del contexto** donde fue definido. Si esa referencia externa cambia, el efecto se propaga al bloque:

```wollok
> var to = "world"
> const helloWorld = { "hello " + to }
> helloWorld.apply()
✓ "hello world"
> to = "someone else"
> helloWorld.apply()
✓ "hello someone else"
```

#### 5.5 Comparación con el paradigma funcional

| Haskell                         | Wollok                          |
| ------------------------------- | ------------------------------- |
| `( \num1 num2 -> num1 + num2 )` | `{ num1, num2 => num1 + num2 }` |

Similitudes y diferencias clave:

- Ambas abstraen comportamiento que no vale la pena nombrar/reutilizar en otro contexto.
- Las lambdas de Haskell **siempre devuelven un valor**; en Wollok un bloque puede devolver un valor **o** producir efecto colateral (en ese caso, el valor de retorno puede no importar).
- Ambas permiten evitar código repetitivo y ganar **declaratividad**: resolver con `map`/`filter`/`fold` (funcional) o con `map`/`filter`/`forEach` (objetos) requiere conocer menos del algoritmo interno y del orden de ejecución que hacerlo con recursividad o con iteración manual.

### 6. Introducción a colecciones

#### 6.1 ¿Qué es una colección?

Una colección representa un **conjunto de objetos relacionados** (jugadores de un equipo, cosas de una lista de compras, objetos en la mochila de un héroe), y en general modela una **relación 1 a N** (una factura tiene muchas líneas, un escritor publicó varios libros).

Importante: una colección **no contiene** a los objetos dentro suyo — más precisamente, es un **conjunto de referencias** hacia otros objetos. Los elementos existen independientemente; la colección simplemente los conoce.

#### 6.2 y 6.3 Interfaz de una colección

Pensando en un álbum de fotos como ejemplo de colección, ¿qué operaciones tienen sentido?

| Operación sobre el álbum                  | Concepto                                            |
| ----------------------------------------- | --------------------------------------------------- |
| Mirar las fotos una por una               | **Iterar** la colección                             |
| Saber cuántas fotos hay                   | **Tamaño** de la colección                          |
| Saber si una foto está en el álbum        | **Pertenencia** de un elemento                      |
| Pegar una foto nueva                      | **Agregar** un elemento                             |
| Regalar una foto                          | **Eliminar** un elemento                            |
| Seleccionar las fotos de un viaje puntual | **Filtrar** elementos                               |
| Anotar las fechas de cada foto            | **Transformar** (mapear) elementos                  |
| Saber si hay alguna foto de Navidad       | Determinar si **algún** elemento cumple un criterio |

#### 6.4 Tipos de colecciones en Wollok

- **Conjuntos** (`#{ }`): sin orden, sin elementos repetidos — modelan el conjunto matemático.
  ```wollok
  const numeros = #{1, 2, 3, 4}
  ```
- **Listas** (`[ ]`): con orden, admiten repetidos.
  ```wollok
  const fila = [francisco, mirta, hector]
  ```

### 7. Ejemplo integrador: Misiones de un héroe (Shrek)

Un héroe tiene una colección de misiones, y cada misión tiene una recompensa de respeto.

#### 7.1 El héroe y su colección

```wollok
object shrek {
  const misiones = #{}
}
```

La referencia `misiones` es `const` porque siempre va a apuntar **al mismo conjunto** — lo que cambia es el contenido de ese conjunto (se agregan o quitan elementos), no la referencia en sí.

#### 7.2 Agregar la primera misión

Se define la misión como un objeto (por ahora sin atributos ni comportamiento — no nos adelantamos a decisiones de diseño que todavía no hacen falta):

```wollok
object liberarAFiona { }
```

```wollok
object shrek {
  const misiones = #{}
  method agregarMision(_mision) { misiones.add(_mision) }
}
```

```
> shrek.agregarMision(liberarAFiona)
```

#### 7.3 Cantidad de misiones

```wollok
object shrek {
  ...
  method cantidadDeMisiones() = misiones.size()
}
```

#### 7.4 Segunda misión

Se agrega `buscarPiedraFilosofal` de la misma manera.

#### 7.5 Filtrar misiones difíciles

Se necesita que ambas misiones entiendan el mismo mensaje `esDificil()` — deben ser **polimórficas**, no para sí mismas sino **para quien las usa** (Shrek):

```wollok
object liberarAFiona {
  var cantidadTrolls = 5
  method esDificil() = cantidadTrolls.between(4, 5)
}

object buscarPiedraFilosofal {
  var kilometrosDistancia = 40
  method esDificil() = kilometrosDistancia > 100
}
```

Nótese que no hace falta definir getters/setters para estos atributos si no se necesitan desde afuera.

#### 7.6 Filter

```wollok
object shrek {
  ...
  method misionesDificiles() =
    misiones.filter({ mision => mision.esDificil() })
}
```

`filter` recibe un **bloque** que representa el criterio de selección, y devuelve los elementos que lo cumplen. Shrek conoce a sus misiones, pero **delega** en cada una la respuesta de si es difícil — no le importa cómo la calcula cada una.

#### 7.7 Filter y efecto colateral

`filter` (al igual que `map`, más abajo) **no modifica** la colección original: devuelve una colección **nueva** con los elementos que cumplen la condición. Esto es fundamental: preguntar "¿cuáles clientes me deben plata?" no puede hacerme perder la referencia a los que sí pagaron.

#### 7.8 Map

```wollok
object shrek {
  ...
  method solicitantesDeMisMisiones() =
    misiones.map({ mision => mision.solicitante() })
}
```

`map` **transforma** cada elemento de la colección según el criterio del bloque, devolviendo una colección nueva (de otro tipo de elementos, incluso — de misiones a solicitantes). Tampoco tiene efecto colateral sobre la colección original.

#### 7.9 Sumar puntos de recompensa

```wollok
object liberarAFiona {
  ...
  method puntosRecompensa() = cantidadTrolls * 2
}

object buscarPiedraFilosofal {
  ...
  method puntosRecompensa() =
    if (kilometrosDistancia > 50) 10 else 5
}

object shrek {
  ...
  method totalPuntosDeRecompensa() =
    misiones.sum({ mision => mision.puntosRecompensa() })
}
```

`sum` recibe un bloque que indica **qué número sumar** por cada elemento. Una vez más, Shrek aprovecha el **polimorfismo** entre las misiones sin importarle cómo cada una calcula su recompensa.

### Resumen del módulo 4

Wollok ofrece objetos básicos **inmutables** (números, booleanos, strings, fechas) y **bloques** (lambdas) que permiten abstraer comportamiento y decidir cuándo ejecutarlo, accediendo además al contexto donde fueron definidos. Las **colecciones** (conjuntos y listas) representan relaciones 1 a N como un conjunto de referencias, con una interfaz rica (`filter`, `map`, `sum`, entre otras) que, combinada con **polimorfismo**, permite resolver requerimientos sin bajar al detalle del algoritmo ni de cómo cada objeto implementa lo que se le pide.

---

## Módulo 5: Introducción al Testeo unitario automatizado

### 1. Introducción

Para saber si el software que construimos funciona bien, lo **probamos**. En Wollok hay varias formas manuales de ejecutar y observar código (REPL, programas, Wollok Game), pero a medida que la solución crece, esas formas manuales dejan de ser prácticas. Frente a eso, Wollok ofrece **tests unitarios automatizados**.

### 2. Casos de prueba

Antes incluso de escribir código, conociendo el problema ya podemos anticipar situaciones a probar: desde los casos típicos hasta los límites o poco frecuentes. Lo importante es **anticipar** el resultado correcto esperado para cada situación — no alcanza con ver que el código "no tira error" y asumir que está bien.

En objetos, lo que queremos probar de un objeto es su **comportamiento**: enviarle mensajes y verificar que pase lo esperado.

### 3. Ejemplo: Pepita

Enunciado: Pepita arranca con 100 de energía. Comer _g_ gramos de alpiste aumenta la energía en `4 * g`. Volar _k_ kilómetros la disminuye en `k + 10`. Es "fuerte" cuando su energía **supera** 50.

Casos de prueba identificados:

| Acciones                             | Resultado esperado                                                |
| ------------------------------------ | ----------------------------------------------------------------- |
| ninguna                              | energía = 100                                                     |
| ninguna                              | es fuerte                                                         |
| volar 5 km                           | energía = 85                                                      |
| comer 120 g                          | energía = 580                                                     |
| comer 0 g                            | energía = 100 (caso límite/trivial)                               |
| volar 0 km                           | energía = 90 (caso límite/trivial)                                |
| volar 60 km                          | **no** es fuerte                                                  |
| volar 40 km (energía exactamente 50) | **no** es fuerte (porque "fuerte" exige _superar_ 50, no igualar) |

Este último caso es un ejemplo de **caso límite**: probar exactamente el valor frontera de una condición, para verificar que el `>` (y no un `>=`) esté bien implementado.

### 4. Solución en Wollok

```wollok
object pepita {
  var energia = 100

  method energia() { return energia }
  method esFuerte() { return energia > 50 }

  method volar(kms) {
    energia = energia - (kms + 10)
  }

  method comer(gramos) {
    energia = energia + 4 * gramos
  }
}
```

### 5. Pruebas manuales

- **REPL**: se envían mensajes uno por uno y se verifica manualmente el resultado. Problema: el estado se **acumula** en la consola, así que para volver a probar desde el estado inicial hay que **reiniciar la consola** cada vez.
- **Programas** (archivos `.wpgm`): una secuencia de mensajes que se ejecuta de punto a punto sin interacción, mostrando resultados por consola con `console.println(...)`.
- **Wollok Game**: representación gráfica de los objetos, útil para verificar visualmente el efecto de los mensajes.

### 6. Problemas de las pruebas manuales

- Es tedioso recordar y repetir siempre el mismo conjunto de casos.
- La verificación manual es propensa a errores (aceptar por descuido un resultado que no es exactamente el esperado).
- Puede ser costoso llevar a un objeto al estado necesario para una prueba puntual.
- Es difícil reproducir una secuencia larga para verificar una corrección.
- En un programa con muchos mensajes encadenados, es difícil ubicar en cuál falló algo.
- Cada cambio de código implica volver a invertir el mismo esfuerzo de prueba.
- Si sólo se prueba lo que cambió, no hay garantía de que el resto del sistema siga funcionando — habría que rehacer _todas_ las pruebas anteriores.

### 7. Testeo unitario automatizado

Frente a estos problemas, Wollok ofrece una herramienta de **tests unitarios automatizados**: un test es una porción de código Wollok que describe una situación (enviando mensajes) y especifica el resultado esperado.

#### Unitario

Se llama así porque en vez de probar un programa completo de punta a punta, se identifican **unidades significativas** de código y se prueban casos puntuales sobre ellas. Es clave que **cada test pruebe una sola cosa**: si un test mezcla muchas verificaciones y falla, es más difícil saber cuál fue el problema real.

#### Automático

La validación la hace la máquina, no una persona — más rápido y confiable. Los tests quedan guardados como código: se pueden volver a correr todos juntos cuando se corrige algo o se refactoriza, para confirmar que todo sigue funcionando.

#### Independencia

Cada test se concibe **independiente** de los demás: se asume que corre desde el estado inicial del sistema (el ambiente se reinicia entre test y test). Si un test falla, los demás pueden seguir corriendo sin problema.

#### Información sintética y precisa

Cuando un test pasa, la información es mínima (simplemente "funcionó"). Cuando **falla**, en cambio, debe informar con precisión qué se esperaba y qué se obtuvo realmente — ahí está el verdadero valor de un test.

### 8. Definición de tests

Los tests se escriben en archivos separados (extensión `.wtest`). El objeto clave es `assert`, un objeto autodefinido de la biblioteca `lib` que permite declarar expectativas.

```wollok
import pepita.*

describe "Tests de Pepita" {

  test "energía inicial de pepita" {
    assert.equals(100, pepita.energia())
  }

  test "pepita comienza siendo fuerte" {
    assert.that(pepita.esFuerte())
  }

  test "pepita vuela y baja su energía" {
    pepita.volar(5)
    assert.equals(85, pepita.energia())
  }

  test "pepita come y sube su energía" {
    pepita.comer(120)
    assert.equals(580, pepita.energia())
  }

  test "pepita empieza a volar, no vuela y la energía baja" {
    pepita.volar(0)
    assert.equals(90, pepita.energia())
  }

  test "pepita come 0 gramos y la energía se mantiene" {
    pepita.comer(0)
    assert.equals(100, pepita.energia())
  }

  test "pepita vuela muchos kilómetros, y ya no es fuerte" {
    pepita.volar(60)
    assert.notThat(pepita.esFuerte())
  }

  test "pepita vuela pocos kilómetros, y ya no es fuerte" {
    pepita.volar(40)
    assert.notThat(pepita.esFuerte())
  }
}
```

#### `describe`

Agrupa un conjunto de tests bajo un nombre expresivo, entre `{ }`. Se puede tener más de un `describe` por archivo, pero si corresponden a unidades bien distintas (aves, entrenadores) conviene separarlos en archivos diferentes.

#### `test`

Cada `test` empieza con una descripción textual — importante que sea clara, porque **esa misma leyenda aparece en el informe cuando falla**. Conviene describir el _escenario_ (la clase de equivalencia), no detalles redundantes del resultado (si describo "energía queda en 85" y luego cambian las reglas de cálculo, tengo que actualizar el nombre en dos lugares).

Estructura típica de un test (patrón **Arrange-Act-Assert**):

1. Se envían los mensajes que configuran la situación ("acciones").
2. Se envía un mensaje a `assert` para validar ("validación").

Tres formas principales de `assert`:

- `assert.equals(esperado, obtenido)` — el más común.
- `assert.that(mensajeBooleano)` — cuando se espera `true`.
- `assert.notThat(mensajeBooleano)` — cuando se espera `false`.

Dentro de un test conviene tener un `assert` por caso de prueba (pueden ser varios `assert`, siempre que respondan al mismo escenario).

### 9. Ejecución de los tests

Se pueden correr desde el menú contextual del IDE (un test, un describe, o el archivo entero), o todos los archivos de test juntos con `Ctrl + R`.

#### Éxito

Si todos los casos pasan, se muestra un tilde verde por cada test, con su descripción — "8 de 8 pasaron", por ejemplo.

#### Fallas

Si se introduce un error en la lógica (p. ej. `volar` resta 5 en vez de 10), algunos tests fallan con **`AssertionException`**: el test terminó de ejecutar, pero el resultado obtenido difiere del esperado. El informe detalla justamente esa diferencia (esperado vs. obtenido).

#### Errores

Si el error es más grave (p. ej. una división por cero al comer 0 gramos), aparece **`EvaluationError`**: el test no pudo terminar de ejecutarse. El informe indica en qué línea de código se produjo el error.

### 10. Resultados posibles de un test

| Resultado                      | Significado                                           |
| ------------------------------ | ----------------------------------------------------- |
| ✓ Ok                           | El código hace lo esperado                            |
| ✗ Falla (`AssertionException`) | El código corrió, pero el resultado no es el esperado |
| ✗ Error (`EvaluationError`)    | Un error impidió completar la ejecución del test      |

Un punto clave: al detectar fallas/errores **no se modifica el código del test**, sólo el de la solución — y correr de nuevo los mismos tests permite verificar el arreglo. Esto es un **test de regresión**.

### Resumen del módulo 5

El testeo unitario garantiza que cada objeto cumple con su responsabilidad. A diferencia de las pruebas manuales, los tests automatizados son **unitarios** (prueban una sola cosa), **automáticos** (la validación no depende de una persona), **independientes** entre sí, y dan **información precisa** cuando fallan. Tener muchos tests no garantiza buen diseño, pero es un excelente punto de partida — y habilita a modificar y refactorizar código con confianza, sin miedo a romper algo sin darse cuenta.

### Apéndice: interfaz de `assert`

| Mensaje                                                  | Verifica                                                                           |
| -------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `assert.that(condition)`                                 | Que la condición se cumple                                                         |
| `assert.notThat(condition)`                              | Que la condición sea falsa                                                         |
| `assert.equals(actual, expected)`                        | Que dos valores sean iguales                                                       |
| `assert.notEquals(actual, expected)`                     | Que dos valores sean distintos                                                     |
| `assert.throwsException(block)`                          | Que al ejecutar el bloque se produzca algún error                                  |
| `assert.throwsExceptionLike(exceptionExpected, block)`   | Que se produzca la misma excepción (tipo y mensaje) que la esperada                |
| `assert.throwsExceptionWithMessage(errorMessage, block)` | Que se produzca un error con ese mensaje exacto, sin importar el tipo de excepción |

---

## Módulo 6: Objetos anónimos. Repaso de polimorfismo

### 1. Ejercicio motivador: viajes en taxi

Se quiere modelar choferes de taxi; en particular a **Daniel**, que lleva a todo pasajero que **no** sea joven.

### 2. Repaso de objetos

¿Qué necesita `llevaA()`? No la edad directamente — determinar si alguien es joven **no es responsabilidad de Daniel**, sino de cada pasajero. Daniel debe **delegar** esa pregunta:

```wollok
object daniel {
  method llevaA(pasajero) = !pasajero.esJoven()
}
```

### 3. Pruebas

| Caso de prueba    | Resultado esperado |
| ----------------- | ------------------ |
| Pasajero joven    | Daniel no lo lleva |
| Pasajero no joven | Daniel sí lo lleva |

Para poder testear esto hace falta: Daniel (wko), un pasajero joven, y un pasajero no joven. Pero todavía no tenemos definiciones completas de "pasajero" como concepto general del negocio.

### 4. Objetos anónimos

En vez de modelar de entrada todo el comportamiento de "pasajero" en general, Wollok permite crear **objetos anónimos**: objetos sin nombre propio, asignados directamente a una referencia local, útiles cuando no nos interesa reutilizarlos fuera de ese contexto (típicamente, en los tests):

```wollok
import choferes.*

describe "Tests de Daniel" {

  test "Daniel no lleva a un pasajero joven" {
    const pasajeroJoven = object {
      method esJoven() = true
    }
    assert.notThat(daniel.llevaA(pasajeroJoven))
  }

  test "Daniel lleva a un pasajero que no es joven" {
    const pasajeroViejo = object {
      method esJoven() = false
    }
    assert.that(daniel.llevaA(pasajeroViejo))
  }
}
```

La única interfaz necesaria para el pasajero de prueba es tener el método `esJoven()`, sin parámetros, devolviendo un booleano — ni más ni menos de lo que Daniel realmente usa.

### 5. Evitando repetir ideas: `method` dentro de un `describe`

Cuando la creación del objeto anónimo se repite (o los pasajeros tienen más atributos), conviene abstraer esa construcción en un **método definido dentro del propio `describe`**:

```wollok
import choferes.*

describe "Tests de Daniel" {

  method crearPasajero(joven) = object {
    method esJoven() = joven
  }

  test "Daniel no lleva a un pasajero joven" {
    const pasajeroJoven = self.crearPasajero(true)
    assert.notThat(daniel.llevaA(pasajeroJoven))
  }

  test "Daniel lleva a un pasajero que no es joven" {
    const pasajeroViejo = self.crearPasajero(false)
    assert.that(daniel.llevaA(pasajeroViejo))
  }
}
```

Equivalente, escrito con `return` explícito:

```wollok
method crearPasajero(joven) {
  return object {
    method esJoven() = joven
  }
}
```

### 6. Convivencia de objetos anónimos y wko

No siempre hace falta un objeto anónimo. Si ya existe un objeto de negocio "real" (un wko), como `magali` (19 años, considera joven hasta los 30):

```wollok
object magali {
  var edad = 19

  method cumplirAnios() {
    edad = edad + 1
  }

  method esJoven() = edad < 30
}
```

Se puede testear directamente contra ese objeto:

```wollok
test "Daniel no lleva a alguien que piensa que es joven" {
  assert.notThat(daniel.llevaA(magali))
}
```

Lo interesante: `magali` y los objetos anónimos usados antes son **polimórficos entre sí** en el contexto de Daniel — todos entienden `esJoven()`, sin importar si son objetos con nombre o anónimos.

### Resumen del módulo 6

Los **objetos anónimos** permiten crear abstracciones sencillas y descartables, ideales cuando no hace falta reutilizarlas fuera de un contexto puntual (como el testeo unitario), evitando así definir de más en el modelo de negocio real. Tienen atributos y comportamiento igual que los wko, y son plenamente **polimórficos** con ellos.

---

## Módulo 10: Manejo de errores

### 1. Errores y excepciones

Recordando la definición de sistema (conjunto de objetos que colaboran), cuando un objeto envía un mensaje, **espera** que el receptor pueda realizar la tarea. Una **excepción** es un evento que altera ese flujo normal de mensajes entre objetos — desde errores graves de hardware hasta errores simples de programación (acceder fuera de los límites de un vector, dividir por cero, enviar un mensaje a una referencia nula).

### 2. Ejemplo motivador: un monedero

Se quiere modelar un monedero con `poner(cantidad)` y `sacar(cantidad)`, arrancando con $500, y con dos reglas de negocio:

- El monto para poner o sacar debe ser positivo (> 0).
- No se puede sacar más plata de la que hay.

### 3. y 4. Objetivo y primera solución (sin validar)

```wollok
object monedero {
  var plata = 500

  method plata() = plata
  method poner(cantidad) { plata = plata + cantidad }
  method sacar(cantidad) { plata = plata - cantidad }
}
```

Falta contemplar qué hacer si `cantidad` no cumple las reglas del negocio.

### 5. Alternativas para modelar el error

#### 5.1 Mirar a un costado (la peor opción)

```wollok
method poner(cantidad) {
  if (cantidad > 0) {
    plata += cantidad
  }
}
```

```
> monedero.poner(-2)
✓
```

No da error, pero tampoco cambia nada — resultado **desconcertante** para quien envía el mensaje: no hay forma de saber que algo salió mal. Esta opción queda descartada.

#### 5.2 Códigos de error

```wollok
method poner(cantidad) {
  if (cantidad > 0) {
    plata = plata + cantidad
    return 0
  } else {
    return -1
  }
}
```

Desventajas:

- Obliga a quien llama a chequear el valor de retorno antes de seguir.
- Con varios errores posibles, los `if` de chequeo se encadenan y complican la lectura.
- Los números "mágicos" (`-1`, `-2`, ...) no son autoexplicativos, ni siquiera reemplazándolos por strings.
- Cuando hay que **propagar** el código de error entre varios métodos (como en un ejemplo de inscripción a materias con validación de cupo y correlativas), la lógica se llena de variables intermedias y `if` anidados sólo para transportar el código de error hacia arriba.

#### 5.3 Excepciones (la solución elegida)

Se lanza una excepción por cada condición que sale del flujo normal.

### 6. Cómo generar una excepción

Dos formas:

```wollok
// Enviando un mensaje que entiende todo objeto
self.error("Pasó tal cosa")

// Instanciando una excepción (usando el concepto de clase)
throw new UserException(message = "Pasó tal cosa")
```

Aplicado al monedero:

```wollok
class UserException inherits Exception { }

method poner(cantidad) {
  if (cantidad < 0) {
    throw new UserException(message = "La cantidad debe ser positiva")
  }
  plata += cantidad
}

method sacar(cantidad) {
  if (cantidad < 0) {
    throw new UserException(message = "La cantidad a retirar debe ser positiva")
  }
  if (cantidad > plata) {
    throw new UserException(message = "Debe retirar menos de " + plata)
  }
  plata = plata - cantidad
}
```

Como la validación de monto positivo se repite, se puede extraer a un método propio:

```wollok
method poner(cantidad) {
  self.validarMontoPositivo(cantidad)
  plata = plata + cantidad
}

method sacar(cantidad) {
  self.validarMontoPositivo(cantidad)
  if (cantidad > plata) {
    throw new UserException(message = "Debe retirar menos de " + plata)
  }
  plata = plata - cantidad
}

method validarMontoPositivo(cantidad) {
  if (cantidad < 0) {
    throw new UserException(message = "La cantidad debe ser positiva")
  }
}
```

### 7. ¿Dónde se atrapa una excepción?

Los **objetos de dominio** (cliente, alumno, héroe, ave, monedero) generalmente **no atrapan** errores — sólo los lanzan o dejan que ocurran, porque rara vez hay algo sensato que el objeto de dominio pueda hacer al respecto.

Ejemplo con `try/catch` disponible pero **sin saber qué poner adentro**:

```wollok
method inscribirse(alumno, materia) {
  try {
    materia.validarCupo()
    alumno.validarCorrelativas(materia)
    ... inscripción propiamente dicha ...
  } catch e : UserException {
    ¿¿¿¿¿ Qué hacer ?????
  }
}
```

La mejor solución: **dejar que la excepción siga su curso**, sin atraparla en el dominio:

```wollok
method inscribirse(alumno, materia) {
  materia.validarCupo()
  alumno.validarCorrelativas(materia)
  ... inscripción propiamente dicha ...
}
```

¿Quién sí puede atrapar realmente una excepción?

- Un **test unitario** (para verificar que se produjo el error esperado).
- La **interfaz de usuario** (para mostrarle al usuario un mensaje entendible antes de que la aplicación se caiga).

### 8. Excepciones de usuario y de programa

| Tipo                     | Quién la entiende       | Ejemplos                                                                               | Quién actúa                                             |
| ------------------------ | ----------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| **De usuario / dominio** | El usuario final        | "no hay saldo", "no hay stock", "no hay precio cargado"                                | El usuario corrige el dato que ingresó                  |
| **De programa**          | Un especialista técnico | "falló el acceso a la base", "división por cero", "el objeto no entiende este mensaje" | Un desarrollador corrige el código o la infraestructura |

En la interfaz de usuario, conviene diferenciar si la excepción es de negocio o no, más que atrapar cada tipo específico por separado.

### 9. Tareas propuestas para el lector

Agregar al monedero las validaciones:

- No permitir más de 3 depósitos el mismo día.
- No permitir extraer más de $1.000 por día.

### 10. Buenas y malas prácticas

#### Principios generales

- **Sólo las atrapa quien las sabe tratar** — en una materia introductoria de objetos, rara vez hace falta usar `try/catch`.
- Las excepciones deberían tratarse en **pocos lugares**; abusar del control produce código difícil de leer y mantener.
- Usar **mensajes claros**, tanto para usuarios como para desarrolladores.
- Representar como excepción sólo lo que realmente sale del **flujo normal** del negocio.
- **Nunca** atrapar una excepción y no hacer nada con ella (ni siquiera loguearla y seguir) — eso oculta el problema y hace que aparezca, disfrazado, más adelante en otro lugar del sistema.

Ejemplo de mal uso — usar `try/catch` sólo para simular un valor de retorno numérico:

```wollok
method compraDirecta() {
  try {
    const cliente = new Cliente()
    cliente.comprar(25)
    ...
    return 0
  } catch e : NoTengoUnMangoException {
    return -1
  }
}
```

Mejor, dejando que la excepción se propague sin controlarla artificialmente:

```wollok
method compraDirecta() {
  const cliente = new Cliente()
  ...
  cliente.comprar(25)  // sabemos que puede fallar, y está bien que así sea
}
```

#### 10.1 To throw or not to throw

Una validación de negocio real (presupuesto insuficiente para contratar) sí amerita una excepción:

```wollok
method contratarA(contratista) {
  if (contratista.costoArreglo(casa) > self.presupuesto())
    throw new UserException(message = "El presupuesto es insuficiente")
  ...
}
```

Pero usar una excepción para una situación que **no** es excepcional (por ejemplo, que la lista de servicios contratados esté vacía) es un error de diseño:

```wollok
// Mal: forzar al llamador a preguntar antes, para "evitar" la excepción
method contratistasContratados() {
  if (serviciosContratados.isEmpty()) {
    throw new UserException(message = "No contrató servicios aún")
  }
  return serviciosContratados.map({ ... })
}
```

Empezar sin servicios contratados **es un estado normal**, no un error — y además `map` sobre una colección vacía ya funciona perfectamente bien (`[].map(...)` devuelve `[]`), así que ni siquiera hace falta el `if`.

#### 10.2 Consistencia en el valor de retorno

Si un método responde una **pregunta** ("¿puede tomar el viaje?"), su respuesta debe ser siempre del mismo tipo — no `true` en un caso y una excepción en el otro:

```wollok
// Mal: mezcla booleano con excepción
method puedeTomar(viaje) {
  if (viaje.destino() == "Korn") return true
  throw new Exception(message = "No puede tomar el viaje")
}

// Bien: siempre devuelve un booleano
method puedeTomar(viaje) = viaje.destino() == "Korn"
```

En cambio, para una **acción** que no devuelve nada (como agregar un viaje), sí es correcto que o bien no haga nada visible (éxito) o bien lance una excepción (fallo):

```wollok
method tomarViaje(viaje) {
  if (!self.puedeTomar(viaje)) {
    throw new Exception(message = "No puede tomar el viaje")
  }
  viajes.add(viaje)
}
```

#### 10.3 Mejorando los mensajes de error

Cuando hay **varias condiciones combinadas**, un mensaje de error genérico no ayuda a saber cuál falló. Conviene separar cada condición en su propio método consultable, y en la acción usar mensajes de error específicos por condición:

```wollok
method destinoComodo(viaje) = viaje.destino() == "Korn"
method edadPasajeroCorrecta(viaje) =
  viaje.pasajero().edad() > EDAD_MINIMA_PASAJERO

method puedeTomar(viaje) = self.destinoComodo(viaje) &&
  self.edadPasajeroCorrecta(viaje)

method agregarViaje(viaje) {
  if (!self.destinoComodo(viaje)) {
    throw new Exception(message = "No puede tomar el viaje porque el destino "
      + viaje.destino() + " no es cómodo")
  }
  if (!self.edadPasajeroCorrecta(viaje)) {
    throw new Exception(message = "El pasajero debe tener "
      + EDAD_MINIMA_PASAJERO + " años o más")
  }
  viajes.add(viaje)
}
```

Esta técnica de separar pregunta (`puedeXXX`, sin efecto, devuelve booleano) de acción (`hacerXXX`, con efecto o excepción) sólo hace falta cuando realmente se necesitan **ambos** métodos.

### Resumen del módulo 10

Las excepciones modelan condiciones que se salen del flujo normal del negocio. Frente a errores de **programa**, los objetos de dominio no pueden (ni deben) hacer nada — se dejan propagar. Frente a errores de **usuario**, hay que detectarlos y comunicarlos lo antes posible con un mensaje claro y accionable. Atrapar excepciones y devolver el sistema a un estado consistente es una posibilidad que existe (`try/catch`), pero en el dominio es poco frecuente que sea la decisión correcta — eso queda mayormente para los tests y la interfaz de usuario.

---

## Ideas que conectan los cuatro módulos

- El **polimorfismo** del Módulo 3 reaparece constantemente: en las colecciones (`filter`, `map`, `sum` del Módulo 4 dependen de que los elementos entiendan cierto mensaje en común), en el testeo (Módulo 6: objetos anónimos y wko son polimórficos entre sí, siempre que compartan la interfaz que el test necesita), y en el manejo de errores (Módulo 10: distintos objetos de dominio lanzan las mismas clases de excepción de manera uniforme).
- El **testeo unitario** (Módulo 5) y los **objetos anónimos** (Módulo 6) están directamente relacionados: los objetos anónimos son, en la práctica, la herramienta más común para armar rápidamente los "dobles de prueba" que necesita un test, sin tener que modelar de más en el dominio real.
- El **manejo de errores** (Módulo 10) y el testeo (Módulo 5) se retroalimentan: `assert.throwsException` y variantes existen justamente porque los tests son uno de los pocos lugares legítimos donde sí tiene sentido atrapar una excepción, para verificar que efectivamente se lanzó cuando correspondía.
- La idea de **abstracción** (separar el qué del cómo, vista desde el Módulo 1) es la misma razón por la que conviene delegar validaciones a métodos propios (`validarMontoPositivo`, `destinoComodo`), separar preguntas de acciones, y confiar en el polimorfismo de las colecciones sin conocer el detalle de cada elemento.
