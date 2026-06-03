# TP Entrega 2 — Videojuegos: Análisis para defensa oral

## Índice

- [TP Entrega 2 — Videojuegos: Análisis para defensa oral](#tp-entrega-2--videojuegos-análisis-para-defensa-oral)
  - [Índice](#índice)
  - [Punto 1.1 — Base compartida (todos)](#punto-11--base-compartida-todos)
    - [Tipos definidos](#tipos-definidos)
    - [`aplicarEventos`](#aplicareventos)
    - [`aplicarTemporada`](#aplicartemporada)
    - [`mejoroSegun`](#mejorosegun)
  - [La abstracción central: `aplicarEventosQue`](#la-abstracción-central-aplicareventosque)
  - [Punto 1.4 — Mi parte (Integrante 3)](#punto-14--mi-parte-integrante-3)
  - [Punto 2.3 — Mi parte (Integrante 3)](#punto-23--mi-parte-integrante-3)
  - [Punto 3.3 — Mi parte (Integrante 3)](#punto-33--mi-parte-integrante-3)
  - [Parte Integrante 1](#parte-integrante-1)
    - [Punto 1.2 — `soloEventosQueMejoranPrecio`](#punto-12--soloeventosquemejoranprecio)
    - [Punto 2.1 — `preciosAscendentes`](#punto-21--preciosascendentes)
    - [Punto 3.1 — `temporada2024`](#punto-31--temporada2024)
  - [Parte Integrante 2](#parte-integrante-2)
    - [Punto 1.3 — `soloEventosQueAchicanCatalogo`](#punto-13--soloeventosqueachicancatalogo)
    - [Punto 2.2 — `catalogoOrdenadoPor`](#punto-22--catalogoordenadopor)
    - [Punto 3.2 — `juegosInfinitos`](#punto-32--juegosinfinitos)
  - [Preguntas probables para la defensa](#preguntas-probables-para-la-defensa)
    - [Sobre la abstracción general](#sobre-la-abstracción-general)
    - [Sobre mi parte (Integrante 3)](#sobre-mi-parte-integrante-3)
    - [Sobre conceptos generales](#sobre-conceptos-generales)
  - [Eta reduction](#eta-reduction)
    - [Dónde SÍ se aplica](#dónde-sí-se-aplica)
    - [Dónde NO se aplica y por qué](#dónde-no-se-aplica-y-por-qué)
    - [Regla general](#regla-general)
  - [Aplicación parcial en el punto 1.4](#aplicación-parcial-en-el-punto-14)

---

## Punto 1.1 — Base compartida (todos)

### Tipos definidos

```haskell
type Evento   = Videojuego -> Videojuego
type Criterio = Videojuego -> Number

data Temporada = Temporada {
  numero         :: Number,
  listaDeEventos :: [Evento]
}
```

Un `Evento` es una función que transforma un `Videojuego`. Un `Criterio` es una función que extrae un número de un `Videojuego`. Esto permite pasar eventos y criterios como parámetros — base del **orden superior**.

---

### `aplicarEventos`

```haskell
aplicarEventos :: [Evento] -> Videojuego -> Videojuego
aplicarEventos eventos juego =
  foldl (\juegoAcumulado evento -> evento juegoAcumulado) juego eventos
```

**Qué hace:** aplica una lista de eventos en orden sobre un juego. Cada evento recibe el resultado del anterior (acumulación en cadena).

**Conceptos:**

- `foldl` es **orden superior** — recibe una función lambda como primer argumento
- La lambda `(\juegoAcumulado evento -> evento juegoAcumulado)` aplica cada evento al estado acumulado
- El juego inicial actúa como acumulador
- Reemplaza un `for` mutable: en Haskell no hay estado mutable, todo se acumula en el parámetro recursivo de `foldl`
- La consigna pedía **sin recursividad** en el punto 1 — `foldl` es la alternativa idiomática

---

### `aplicarTemporada`

```haskell
aplicarTemporada :: Temporada -> Videojuego -> Videojuego
aplicarTemporada temporada juego = aplicarEventos (listaDeEventos temporada) juego
```

Delega en `aplicarEventos`. Existe para dar nombre semántico a la operación: "aplicar una temporada a un juego". Función de una sola línea.

---

### `mejoroSegun`

```haskell
mejoroSegun :: Criterio -> Evento -> Videojuego -> Bool
mejoroSegun criterio evento juego = criterio (evento juego) > criterio juego
```

**Qué hace:** dado un criterio, un evento y un juego, dice si el criterio mejoró (aumentó) tras aplicar el evento.

**Conceptos:**

- `criterio` es un parámetro que es una función → **orden superior**
- `criterio (evento juego)`: primero aplica el evento al juego, luego aplica el criterio al resultado → **composición de funciones** aplicada manualmente

---

## La abstracción central: `aplicarEventosQue`

```haskell
aplicarEventosQue :: (Number -> Number -> Bool) -> Criterio -> Temporada -> Videojuego -> Videojuego
aplicarEventosQue comparacion criterio temporada juego =
  aplicarEventos (filter cumple (listaDeEventos temporada)) juego
  where cumple evento = comparacion (criterio (evento juego)) (criterio juego)
```

Esta es la pieza más importante del diseño. La consigna pedía "reutilizar ideas comunes sin copy-paste" entre 1.2, 1.3 y 1.4.

**Qué hace:** filtra los eventos de la temporada según una condición y aplica solo los que la cumplen. La condición siempre se evalúa contra el juego **original** (no el acumulado).

**Conceptos concentrados:**

| Concepto           | Dónde aparece                                                                                                               |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| Orden superior     | recibe `comparacion` (función) y `criterio` (función) como parámetros                                                       |
| `filter`           | orden superior — recibe `cumple` como función                                                                               |
| Clausura (`where`) | `cumple` captura `comparacion`, `criterio` y `juego` del contexto exterior sin recibirlos como parámetros                   |
| Aplicación parcial | los tres usos fijan solo los primeros 2 argumentos y devuelven una función que espera `Temporada → Videojuego → Videojuego` |

**Diferencia clave con `aplicarTemporada`:**

- `aplicarTemporada`: aplica **todos** los eventos en cadena (el segundo parte del resultado del primero)
- `aplicarEventosQue`: filtra primero evaluando contra el original, luego aplica en cadena solo los seleccionados

**Los tres usos — aplicación parcial pura:**

```haskell
soloEventosQueMejoranPrecio   = aplicarEventosQue (>) precioBase      -- Integrante 1
soloEventosQueAchicanCatalogo = aplicarEventosQue (<) cantExpansiones  -- Integrante 2
soloEventosQueMejoranImpacto  = aplicarEventosQue (>) impacto          -- Integrante 3
```

Ninguna tiene cuerpo propio ni parámetros explícitos. Son pura **aplicación parcial**: fijan comparador y criterio, el resto queda pendiente.

---

## Punto 1.4 — Mi parte (Integrante 3)

```haskell
soloEventosQueMejoranImpacto :: Temporada -> Videojuego -> Videojuego
soloEventosQueMejoranImpacto = aplicarEventosQue (>) impacto
```

- Comparador `(>)`: solo pasan eventos que **suben** el impacto
- Criterio `impacto`: la función que calcula el impacto del juego
- Sin parámetros explícitos → **aplicación parcial** de `aplicarEventosQue`

**Caso de prueba clave:** temporada 2022 `[parchear 8, remasterizar, relanzar]` sobre Zelda (impacto inicial 12000):

- `parchear 8` baja el impacto a 3900 → descartado
- `remasterizar` sube el impacto a 19600 → aplica
- `relanzar` sube el impacto a 30000 (sobre el original) → aplica
- Se aplican remasterizar y relanzar **en ese orden** sobre el estado original

---

## Punto 2.3 — Mi parte (Integrante 3)

```haskell
impactosAscendentes :: [Temporada] -> Videojuego -> Bool
impactosAscendentes [t] juego    = impacto (aplicarTemporada t juego) > impacto juego
impactosAscendentes (t:ts) juego =
  impacto siguiente > impacto juego && impactosAscendentes ts siguiente
  where siguiente = aplicarTemporada t juego
impactosAscendentes [] _         = False
```

**Qué hace:** verifica que al aplicar cada temporada en cadena, el impacto suba estrictamente en cada paso.

**Conceptos:**

- **Recursividad**: con pattern matching. Casos: lista vacía (`[]` → False), un solo elemento (`[t]`), y el caso general `(t:ts)`
- **Acumulación de estado**: `siguiente = aplicarTemporada t juego` es el juego transformado que se pasa en la llamada recursiva. La temporada siguiente parte del resultado de la anterior (distinto al punto 1.4 donde se evaluaba contra el original)
- **Cortocircuito con `&&`**: si `impacto siguiente > impacto juego` es `False`, Haskell no evalúa el lado derecho. Clave para el punto 3.3
- La consigna pedía **únicamente recursividad** en el punto 2 — sin `filter`, sin `foldl`

**Diferencia entre 1.4 y 2.3:**

|             | Punto 1.4                        | Punto 2.3                                                |
| ----------- | -------------------------------- | -------------------------------------------------------- |
| Evaluación  | siempre contra el juego original | acumulativa, cada temporada parte del resultado anterior |
| Herramienta | `filter` + `aplicarEventos`      | recursividad pura                                        |
| Restricción | sin recursividad                 | solo recursividad                                        |

---

## Punto 3.3 — Mi parte (Integrante 3)

```haskell
temporadasInfinitas :: [Temporada]
temporadasInfinitas = temporada2022 : soloVacias
  where soloVacias = temporadaVacia : soloVacias
```

**¿Puede `impactosAscendentes` dar resultado con lista infinita?**

**Sí, pero el único resultado posible es `False`.**

**Por qué `False` es posible:**

- La lazy evaluation evalúa la lista elemento por elemento, solo cuando se necesita
- El `&&` es **no estricto** en su segundo argumento: `False && _ = False` sin evaluar el resto
- Si en algún momento el impacto no sube (ej. temporada vacía no cambia nada), la recursión se detiene y devuelve `False`

**Por qué `True` es imposible:**

- `True` requeriría verificar **todos** los elementos de la lista
- Una lista infinita nunca termina → la función **diverge** (loop infinito)
- No hay forma de garantizar que todos los pasos son ascendentes sin recorrer la lista entera

**Definición recursiva de lista infinita:**

- `soloVacias = temporadaVacia : soloVacias` se define en términos de sí misma
- En Haskell esto es válido gracias a lazy evaluation — no se construye la lista al definirla, solo cuando se pide el siguiente elemento

---

## Parte Integrante 1

### Punto 1.2 — `soloEventosQueMejoranPrecio`

```haskell
soloEventosQueMejoranPrecio :: Temporada -> Videojuego -> Videojuego
soloEventosQueMejoranPrecio = aplicarEventosQue (>) precioBase
```

`precioBase = precio` es un alias de la función `precio`. Comparador `(>)`: solo pasan eventos que suben el precio. Aplicación parcial igual que el integrante 3.

### Punto 2.1 — `preciosAscendentes`

```haskell
preciosAscendentes [] juego = True
preciosAscendentes (evento:eventos) juego
  | mejoroSegun precio evento juego = preciosAscendentes eventos juego
  | otherwise = False
```

Recursividad con pattern matching y guardas. Verifica que cada evento suba el precio respecto al juego que se va pasando.

> **Nota:** la función tal como está siempre compara contra el `juego` original (no acumula el resultado de cada evento antes de pasarlo). Para el comportamiento acumulativo que pide la consigna, el caso recursivo debería pasar `evento juego` en lugar de `juego`.

### Punto 3.1 — `temporada2024`

```haskell
temporada2024 = Temporada { numero = 2024, listaDeEventos = [lanzarExpansion "DLC0", relanzar] ++ map parchear [1..] }
```

`map parchear [1..]` genera una lista infinita de eventos. Gracias a lazy evaluation se puede definir. El resultado posible es `False`: en algún momento un `parchear x` supera la longitud de las expansiones y el precio no sube, cortocircuitando con `otherwise`.

---

## Parte Integrante 2

### Punto 1.3 — `soloEventosQueAchicanCatalogo`

```haskell
soloEventosQueAchicanCatalogo :: Temporada -> Videojuego -> Videojuego
soloEventosQueAchicanCatalogo = aplicarEventosQue (<) cantExpansiones
```

Diferencia clave: usa `(<)` en lugar de `(>)` porque acá se quiere que el criterio **disminuya** (menos expansiones). `cantExpansiones` cuenta cuántas expansiones tiene el juego.

### Punto 2.2 — `catalogoOrdenadoPor`

```haskell
catalogoOrdenadoPor _ [] = True
catalogoOrdenadoPor _ [juego] = True
catalogoOrdenadoPor funcionCriterio (juego:juegos)
  | funcionCriterio juego > (funcionCriterio . head) juegos = False
  | otherwise = True && catalogoOrdenadoPor funcionCriterio juegos
```

Recursividad con pattern matching. `(funcionCriterio . head) juegos` es **composición de funciones**: primero `head` obtiene el primer elemento de `juegos`, luego `funcionCriterio` le aplica el criterio. El `True &&` en el `otherwise` es redundante pero no incorrecto.

### Punto 3.2 — `juegosInfinitos`

```haskell
juegosInfinitos = catalogoInfinito []
  where
    catalogoInfinito [] = [zelda, hollowKnight] ++ catalogoInfinito [witcher3, cyberpunk]
    catalogoInfinito _  = [witcher3, cyberpunk]  ++ catalogoInfinito [witcher3, cyberpunk]
```

Lista infinita que empieza con zelda y hollow y luego cicla entre witcher3 y cyberpunk. Resultado posible: `False`, porque zelda tiene precio mayor que hollow → `catalogoOrdenadoPor` devuelve `False` sin evaluar el resto gracias a lazy evaluation.

---

## Preguntas probables para la defensa

### Sobre la abstracción general

**¿Por qué crearon `aplicarEventosQue` en lugar de copiar el código?**
Para evitar repetición (no copy-paste). La lógica de "filtrar eventos según un criterio y aplicarlos" es la misma en 1.2, 1.3 y 1.4 — lo único que cambia es el comparador y el criterio. La abstracción parametriza esas diferencias.

**¿Por qué las tres funciones no tienen parámetros explícitos?**
Aplicación parcial. `aplicarEventosQue (>) impacto` ya devuelve una función que espera `Temporada` y `Videojuego`. No hace falta escribirlos.

**¿Qué es `where cumple` dentro de `aplicarEventosQue`?**
Una función local que captura variables del contexto exterior (`comparacion`, `criterio`, `juego`) sin recibirlas como parámetros. Eso es una clausura.

### Sobre mi parte (Integrante 3)

**¿Por qué el filtro en 1.4 evalúa siempre contra el juego original?**
Porque en `cumple evento = comparacion (criterio (evento juego)) (criterio juego)`, el `juego` es siempre el parámetro original de `aplicarEventosQue`, no el acumulado. Todos los eventos se evalúan contra el mismo punto de partida.

**¿Qué diferencia hay entre `aplicarTemporada` y `soloEventosQueMejoranImpacto`?**
`aplicarTemporada` aplica todos los eventos en cadena. `soloEventosQueMejoranImpacto` primero filtra cuáles suben el impacto evaluando contra el original, y luego aplica en cadena solo los seleccionados.

**¿Por qué en 2.3 el estado es acumulativo y en 1.4 no?**
Porque la consigna lo pide así en cada caso. En 1.4 la aclaración dice "la evaluación se hace siempre contra el estado original". En 2.3 dice "la temporada siguiente se aplica sobre el juego que dejó la anterior".

**¿Por qué `True` es imposible en 3.3 con lista infinita?**
Porque `True` requeriría verificar todos los elementos de la lista. Una lista infinita no termina → la función diverge. `False` sí es posible porque el `&&` cortocircuita en cuanto encuentra un paso que no sube.

### Sobre conceptos generales

**¿Qué es orden superior?**
Una función que recibe o devuelve otras funciones como parámetros. Ejemplos en el TP: `aplicarEventosQue`, `foldl`, `filter`, `mejoroSegun`.

**¿Qué es aplicación parcial?**
Aplicar una función a menos argumentos de los que espera, obteniendo una nueva función que espera el resto. En Haskell todas las funciones están currificadas, lo que hace esto natural.

**¿Qué es lazy evaluation?**
Haskell no evalúa expresiones hasta que su valor es necesario. Permite definir listas infinitas y operar sobre ellas siempre que no se necesite recorrerlas enteras. El `&&` y el `||` son no estrictos en su segundo argumento, permitiendo cortocircuito incluso sobre estructuras infinitas.

**¿Qué es una lista infinita en Haskell?**
Una estructura definida recursivamente en términos de sí misma. Haskell la construye elemento a elemento solo cuando se necesita, gracias a lazy evaluation. Ejemplo: `soloVacias = temporadaVacia : soloVacias`.

## Eta reduction

Eta reduction es cuando eliminás el último parámetro de una función porque aparece aplicado directamente en ambos lados:

```haskell
f x = g x  -- con eta reduction queda: f = g
```

### Dónde SÍ se aplica

**Las tres funciones del punto 1:**

```haskell
soloEventosQueMejoranPrecio   = aplicarEventosQue (>) precioBase
soloEventosQueAchicanCatalogo = aplicarEventosQue (<) cantExpansiones
soloEventosQueMejoranImpacto  = aplicarEventosQue (>) impacto
```

Sin eta reduction serían:

```haskell
soloEventosQueMejoranImpacto temporada juego = aplicarEventosQue (>) impacto temporada juego
```

Se eliminaron `temporada` y `juego` porque aparecen al final en ambos lados. Funciona porque Haskell está currificado — aplicar parcialmente los primeros dos argumentos devuelve una función que espera los dos restantes.

**`aplicarTemporada`** — se _podría_ aplicar eta reduction sobre `juego`:

```haskell
-- como está
aplicarTemporada temporada juego = aplicarEventos (listaDeEventos temporada) juego
-- con eta reduction
aplicarTemporada temporada = aplicarEventos (listaDeEventos temporada)
```

No se hizo, probablemente por claridad.

### Dónde NO se aplica y por qué

**`aplicarEventos`:** `juego` no aparece al final de la expresión derecha sino en el medio como segundo argumento de `foldl`. Eta reduction solo aplica cuando el parámetro es el último argumento en ambos lados.

```haskell
aplicarEventos eventos juego =
  foldl (\juegoAcumulado evento -> evento juegoAcumulado) juego eventos
```

**`mejoroSegun`:** `juego` aparece dos veces en el lado derecho (`evento juego` y `criterio juego`). Eta reduction solo funciona cuando el parámetro aparece exactamente una vez, al final.

```haskell
mejoroSegun criterio evento juego = criterio (evento juego) > criterio juego
```

**`impactosAscendentes`:** tiene múltiples casos con pattern matching — necesita el parámetro visible para hacer el match, y `juego` aparece múltiples veces en el cuerpo.

### Regla general

| Condición                                                       | ¿Eta reduction? |
| --------------------------------------------------------------- | --------------- |
| El parámetro es el último en ambos lados y aparece una sola vez | ✓ Sí            |
| El parámetro aparece más de una vez en el lado derecho          | ✗ No            |
| El parámetro aparece en el medio de la expresión derecha        | ✗ No            |
| Hay pattern matching sobre ese parámetro                        | ✗ No            |

## Aplicación parcial en el punto 1.4

La firma completa de `aplicarEventosQue` espera 4 argumentos en orden:

```haskell
aplicarEventosQue :: (Number -> Number -> Bool) -> Criterio -> Temporada -> Videojuego -> Videojuego
```

1. un comparador `(Number -> Number -> Bool)`
2. un criterio `Criterio`
3. una temporada `Temporada`
4. un juego `Videojuego`

Cuando escribimos:

```haskell
soloEventosQueMejoranImpacto = aplicarEventosQue (>) impacto
```

Se le pasan solo los primeros dos — `(>)` como comparador e `impacto` como criterio. Como todas las funciones en Haskell están currificadas, aplicar menos argumentos de los que espera devuelve una nueva función que espera el resto.

`soloEventosQueMejoranImpacto` no es el resultado de llamar a `aplicarEventosQue` — es una **nueva función** con firma:

```haskell
soloEventosQueMejoranImpacto :: Temporada -> Videojuego -> Videojuego
```

Los dos primeros argumentos quedaron fijos (`(>)` e `impacto`), y los dos últimos (`Temporada` y `Videojuego`) se reciben recién cuando alguien llama a `soloEventosQueMejoranImpacto` en los tests.

Se puede pensar así:

```
aplicarEventosQue → recibe (>)  → recibe impacto  → queda esperando Temporada y Videojuego
     4 args           3 restantes    2 restantes                    ↓
                                                      soloEventosQueMejoranImpacto
```

Esto es lo que permite la eta reduction: como los últimos dos argumentos todavía no fueron aplicados, no hace falta escribirlos explícitamente en la definición.

```

```
