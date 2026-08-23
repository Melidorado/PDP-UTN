# Módulo 5 — Paradigma Funcional: Orden Superior

## El problema: baja cohesión

Cuando una función hace más de una cosa a la vez, tiene **baja cohesión**. Estas tres funciones mezclan dos responsabilidades: recorrer la lista Y aplicar el criterio de filtrado:

```haskell
clientesPalindromos [] = []
clientesPalindromos (cliente:clientes)
    | (palindromo . nombre) cliente = cliente : clientesPalindromos clientes
    | otherwise                     = clientesPalindromos clientes

clientesQueDeben plata [] = []
clientesQueDeben plata (cliente:clientes)
    | ((> plata) . deuda) cliente = cliente : clientesQueDeben plata clientes
    | otherwise                   = clientesQueDeben plata clientes

clientesConFacturaDe plata [] = []
clientesConFacturaDe plata (cliente:clientes)
    | (elem plata . facturas) cliente = cliente : clientesConFacturaDe plata clientes
    | otherwise                       = clientesConFacturaDe plata clientes
```

Los tres problemas tienen el mismo esquema: **filtrar una lista según un criterio**. La solución es separar el algoritmo de recorrido del criterio de selección, pasando la función de criterio como parámetro.

---

## Funciones de orden superior

Una función de **orden superior** es aquella que recibe o devuelve otras funciones como parámetros. Permite construir abstracciones más generales delegando comportamiento.

> Las funciones son valores como cualquier otro: igual que pasamos un `Int` o un `Bool`, podemos pasar una función como parámetro. Esto significa que podemos pasar un bloque de código para que sea ejecutado en el momento en que se necesite.

---

## `filter` — seleccionar elementos

```haskell
filter :: (a -> Bool) -> [a] -> [a]
filter f []     = []
filter f (x:xs)
    | f x       = x : filter f xs
    | otherwise = filter f xs
```

> **Importante:** los paréntesis en `(a -> Bool)` son obligatorios. Sin ellos, Haskell interpreta que `filter` recibe tres parámetros separados (`a`, `Bool` y `[a]`) en lugar de una función y una lista.

```haskell
filter ((> 10000) . deuda)        clientes  -- deben más de 10.000
filter (palindromo . nombre)      clientes  -- nombre palíndromo
filter (elem 500000 . facturas)   clientes  -- tienen factura de 500.000
```

`filter` no sabe nada del criterio: solo sabe que recibe una función `a -> Bool` y la aplica.

### Construcción de funciones auxiliares para usar con `filter`

```haskell
deudaMayorA :: Float -> Cliente -> Bool
deudaMayorA monto cliente = ((> monto) . deuda) cliente
-- o simplificado:
deudaMayorA monto = (> monto) . deuda

-- Uso:
filter (deudaMayorA 300) clientes
```

### `filter` y declaratividad

Comparando tres soluciones para filtrar palíndromos:

```
pseudocódigo  →  recursividad  →  filter
  (más procedimental)          (más declarativo)
```

- **Pseudocódigo:** maneja índices, memoria dinámica, longitud fija del array, dos variables de iteración
- **Recursividad:** desaparece el índice y la memoria, pero sigue habiendo separación cabeza/cola y casos base/recursivo; sabemos que procesamos en orden
- **`filter`:** no separamos en cabeza y cola, no conocemos el orden en que se filtran los elementos; solo especificamos el criterio → **máxima declaratividad**

---

## Composición vs. orden superior

```haskell
-- Composición: construye una nueva función de primer orden
(palindromo . nombre)        -- función que dado un cliente dice si su nombre es palíndromo

-- Orden superior: pasa una función como parámetro
filter even [1..10]          -- filter es orden superior, even es primer orden
```

|                            | Composición (`.`)                                    | Orden superior                    |
| -------------------------- | ---------------------------------------------------- | --------------------------------- |
| Qué hace                   | Construye una nueva función                          | Recibe una función como parámetro |
| Las funciones involucradas | Ambas de primer orden                                | `filter` es orden superior        |
| La función `.`             | Es en realidad orden superior (recibe dos funciones) | —                                 |

---

## Cohesión y acoplamiento con orden superior

Antes teníamos **una función con dos objetivos**. Ahora tenemos **dos funciones, cada una con un solo objetivo**:

- `filter`: recorre la lista y filtra, delegando el criterio
- `(deudaMayorA 300)`: determina si un cliente cumple el criterio

Esto facilita el **testing unitario**: puedo testear el criterio con un solo cliente, sin necesitar una lista entera.

El acoplamiento entre `filter` y el criterio existe (es necesario para resolver el requerimiento), pero es **bajo**: `filter` solo conoce el tipo `(a -> Bool)`, no la implementación del criterio. Si hay un error en `palindromo`, no hay que tocar `filter`.

Además, permite **dividir el trabajo**: una persona puede desarrollar `filter` y otras pueden desarrollar los criterios, coordinándose solo en la interfaz `(a -> Bool)`.

---

## `map` — transformar elementos

Aplica una función a todos los elementos de una lista, devolviendo una nueva lista.

```haskell
map :: (a -> b) -> [a] -> [b]
map f []     = []
map f (x:xs) = f x : map f xs
```

```haskell
map (* 2) [1..5]                        -- [2,4,6,8,10]
map toUpper "hola"                      -- "HOLA" (requiere import Data.Char)
map nombre clientes                     -- lista de nombres de clientes
map deuda clientes                      -- lista de deudas de clientes
map facturas clientes                   -- lista de listas de facturas
```

> `map` puede ser **heterogéneo**: la lista de entrada y la de salida pueden ser de distinto tipo (`a` puede ser distinto de `b`).

### Ejemplo: sumar longitudes de palabras

```haskell
sumarPalabras :: [String] -> Int
sumarPalabras = sum . map length

sumarPalabras ["paradigmas", "rules", "the", "world"]  -- 18
```

### Caso avanzado: `map` con strings

Para filtrar palíndromos ignorando mayúsculas/minúsculas:

```haskell
import Data.Char

tieneNombrePalindromo :: Cliente -> Bool
tieneNombrePalindromo = palindromo . map toLower . nombre
-- convierte el nombre a minúsculas antes de comparar
```

### `map` devuelve una lista de funciones

```haskell
map (+) [1, 2, 3]
-- devuelve: [(1+), (2+), (3+)]
-- es decir, una lista de funciones parcialmente aplicadas
```

---

## `all` y `any`

```haskell
all :: (a -> Bool) -> [a] -> Bool
all f = and . map f     -- True si TODOS cumplen la condición

any :: (a -> Bool) -> [a] -> Bool
any f = or . map f      -- True si ALGUNO cumple la condición
```

```haskell
all even [2, 4, 6]                    -- True
all (> 0) [1..9]                      -- True
any (elem 3) [[6..9], [2..4]]         -- True
any ((> 1000) . deuda) clientes       -- ¿algún cliente debe más de 1000?
```

La implementación se basa en:

- `map f` transforma la lista en una lista de booleanos
- `and` / `or` la reduce a un único booleano

### `sumaF` — patrón común con `map`

```haskell
-- Sumar un campo específico de una lista de estructuras
sumaF :: (a -> Float) -> [a] -> Float
sumaF f = sum . map f

sumaF deuda clientes      -- total de deudas
sumaF sueldo empleados    -- total de sueldos
```

---

## `fold` — reducción

Combina todos los elementos de una lista usando una **operación binaria**, "intercalándola" entre los elementos.

### Idea visual

```
[4, 5, 8]  con (+)    →  4 + 5 + 8        →  17
[4, 5, 8]  con (*)    →  4 * 5 * 8        →  160
[4, 5, 8]  con max    →  4 `max` 5 `max` 8 →  8
[[1,2],[], [3,4]] con (++) →  [1,2,3,4]
```

### `foldl1` y `foldr1` — sin valor inicial

```haskell
foldl1 :: (a -> a -> a) -> [a] -> a
```

```haskell
sum     = foldl1 (+)
product = foldl1 (*)
maximum = foldl1 max
concat  = foldl1 (++)
```

- `foldl1`: asocia de **izquierda** a derecha: `((4 + 5) + 8)`
- `foldr1`: asocia de **derecha** a izquierda: `(4 + (5 + 8))`

Con operaciones **no conmutativas** (como `/`) el resultado es diferente:

```haskell
foldl1 (/) [8, 2, 2]   -- (8 / 2) / 2 = 2.0
foldr1 (/) [8, 2, 2]   -- 8 / (2 / 2) = 8.0
```

⚠️ `foldl1` y `foldr1` fallan con lista vacía.

### `foldl` y `foldr` — con valor inicial (semilla)

```haskell
foldl :: (a -> b -> a) -> a -> [b] -> a
foldr :: (a -> b -> b) -> b -> [a] -> b
```

La semilla se coloca a izquierda (`foldl`) o a derecha (`foldr`) de la lista:

```haskell
foldl (*) 1 [4, 5, 8]   -- (((1 * 4) * 5) * 8) = 160
foldr (*) 1 [4, 5, 8]   -- (4 * (5 * (8 * 1))) = 160
```

Con lista vacía, devuelve el valor inicial. Por eso:

```haskell
sum     = foldl (+) 0
product = foldl (*) 1
concat  = foldl (++) []
```

El valor inicial suele ser el **elemento neutro** de la operación.

> **Diferencia de tipos entre `foldl` y `foldr`:** en `foldr` la función binaria espera primero el elemento de la lista y luego la semilla `(a -> b -> b)`. En `foldl` es al revés: primero la semilla y luego el elemento `(a -> b -> a)`. Esto es importante cuando se usa `flip`.

### Usando `fold` con lambdas

```haskell
-- Suma total de sueldos con lambda
foldl (\total empleado -> total + sueldo empleado) 0.0 empleados

-- Equivalente con composición (más conciso)
foldr ((+) . sueldo) 0 empleados

-- Empleado que más gana
foldl1 (\e1 e2 -> if sueldo e1 > sueldo e2 then e1 else e2) empleados
```

### `map` implementado con `foldr`

```haskell
map f = foldr ((:) . f) []

-- Ejemplo:
foldr ((:) . (2 *)) [] [4, 3, 7]   -- [8, 6, 14]
```

### `length` implementado con `foldr`

```haskell
length = foldr incr 0
         where incr _ a = a + 1
```

### `foldl` vs. `foldr`

```haskell
foldr f i []     = i
foldr f i (x:xs) = f x (foldr f i xs)    -- no es recursivo a la cola

foldl f i []     = i
foldl f i (x:xs) = foldl f (f i x) xs    -- recursivo a la cola
```

|                     | `foldl`                                        | `foldr`                                      |
| ------------------- | ---------------------------------------------- | -------------------------------------------- |
| Asociatividad       | Izquierda                                      | Derecha                                      |
| Recursivo a la cola | ✓ Sí — no necesita stack                       | ✗ No — necesita guardar estado               |
| Listas muy grandes  | Puede terminar sin Stack Overflow              | Puede dar Stack Overflow                     |
| Listas infinitas    | ✗ Nunca termina (necesita llegar al caso base) | ✓ Puede terminar (gracias a lazy evaluation) |

```haskell
foldr (&&) False (repeat False)   -- False ✓ termina: False && _ = False, no evalúa el resto
foldl (&&) False (repeat False)   -- nunca termina ✗ necesita llegar a la lista vacía
```

> **¿Por qué `foldr` puede terminar con listas infinitas?** Porque Haskell usa evaluación diferida: al aplicar `f x (foldr f i xs)`, si `f x` ya determina el resultado sin necesitar el segundo argumento, no se evalúa el resto. Con `foldl`, en cambio, siempre necesita llegar a la lista vacía para devolver algo.

### `foldr` aplicado a funciones con `$`

```haskell
-- Aplicar sucesivamente una lista de funciones a un valor
foldr ($) 0 [(^ 6), (2 *), (1 +)]
-- (^6) $ (2*) $ (1+) $ 0
-- (^6) $ (2*) 1
-- (^6) 2
-- 64
```

### `foldr` con funciones de `Persona -> Persona`

```haskell
data Persona = Persona { edad :: Int, felicidad :: Int, amigos :: [String] }

-- Operaciones cerradas: Persona -> Persona
cumplirAnios :: Persona -> Persona
caminar :: Int -> Persona -> Persona
hacerseAmigoDe :: String -> Persona -> Persona

-- Aplicar una lista de transformaciones a una persona
foldr ($) persona [cumplirAnios, caminar 10, hacerseAmigoDe "Toby"]
-- aplica de derecha a izquierda: primero hacerseAmigoDe, luego caminar, luego cumplirAnios
```

---

## `flip` — invertir parámetros

Recibe una función y la aplica con los parámetros en orden inverso.

```haskell
flip :: (a -> b -> c) -> b -> a -> c
flip f y x = f x y
```

### Caso de uso: adapter para `filter`

```haskell
data Ciudad = Ciudad { nombre :: String, tempPromedio :: Float }

haceMasDe :: Ciudad -> Float -> Bool
haceMasDe ciudad temp = tempPromedio ciudad > temp

-- No compila: filter espera (Ciudad -> Bool), pero haceMasDe recibe Ciudad primero
filter (haceMasDe ???? 25) ciudades

-- Con flip: invierte el orden, ahora primero viene el número y luego la ciudad
filter (flip haceMasDe 25) ciudades
```

> `flip` actúa como **adaptador**: permite encajar una función en un contexto sin modificar su definición original. Útil cuando la función pertenece a una biblioteca externa o se usa en varios lugares.

---

## `$` — función aplicación

Permite aplicar una función a un valor evitando paréntesis.

```haskell
($) :: (a -> b) -> a -> b
f $ x = f x
```

```haskell
-- Sin $
show (head [1, 2])
(take 1 . filter even) [1..10]

-- Con $
show $ head [1, 2]
take 1 . filter even $ [1..10]
```

### `foldr` + `$` para aplicar funciones en cadena

```haskell
foldr ($) persona [cumplirAnios, caminar 10, hacerseAmigoDe "Toby"]
-- aplica cada función de la lista sobre la persona, de derecha a izquierda
```

---

## Resumen

| Función           | Tipo                         | Qué hace                                                                   |
| ----------------- | ---------------------------- | -------------------------------------------------------------------------- |
| `filter`          | `(a -> Bool) -> [a] -> [a]`  | Selecciona elementos que cumplen un criterio                               |
| `map`             | `(a -> b) -> [a] -> [b]`     | Transforma cada elemento                                                   |
| `all`             | `(a -> Bool) -> [a] -> Bool` | True si todos cumplen la condición                                         |
| `any`             | `(a -> Bool) -> [a] -> Bool` | True si alguno cumple la condición                                         |
| `foldl1`/`foldr1` | `(a->a->a) -> [a] -> a`      | Reduce lista sin valor inicial                                             |
| `foldl`           | `(a->b->a) -> a -> [b] -> a` | Reduce de izquierda a derecha con semilla; recursivo a la cola             |
| `foldr`           | `(a->b->b) -> b -> [a] -> b` | Reduce de derecha a izquierda con semilla; compatible con listas infinitas |
| `flip`            | `(a->b->c) -> b -> a -> c`   | Invierte el orden de los parámetros                                        |
| `$`               | `(a->b) -> a -> b`           | Aplica una función, evita paréntesis                                       |

## Conceptos clave

`orden superior` · `filter` · `map` · `fold` · `foldl` · `foldr` · `semilla` · `flip` · `$` · `cohesión` · `acoplamiento` · `recursivo a la cola` · `tail recursion` · `listas infinitas` · `lazy evaluation` · `operación binaria` · `elemento neutro`

```

```

## `foldl` vs `foldr` — Por qué `foldl` no necesita stack

### `foldl` — recursivo a la cola

El resultado de cada paso **se convierte en la nueva semilla** de la siguiente llamada. No queda nada pendiente.

```haskell
foldl (+) 0 [1, 2, 3]
foldl (+) 1 [2, 3]      -- calculó (0+1)=1, lo pasa como nueva semilla
foldl (+) 3 [3]         -- calculó (1+2)=3, lo pasa como nueva semilla
foldl (+) 6 []          -- calculó (3+3)=6, lo pasa como nueva semilla
6                       -- lista vacía, devuelve la semilla
```

Cada llamada recursiva ya tiene todo calculado antes de seguir. No hay operaciones pendientes → no necesita apilar estado → no hay Stack Overflow.

### `foldr` — no es recursivo a la cola

Cada llamada **queda esperando** el resultado de la siguiente antes de poder operar.

```haskell
foldr (+) 0 [1, 2, 3]
1 + (foldr (+) 0 [2, 3])        -- espera el resultado de la llamada recursiva
1 + (2 + (foldr (+) 0 [3]))     -- sigue esperando...
1 + (2 + (3 + (foldr (+) 0 [])))
1 + (2 + (3 + 0))
1 + (2 + 3)
1 + 5
6
```

Con una lista de 10.000 elementos hay 10.000 operaciones pendientes apiladas en memoria → puede dar Stack Overflow en listas muy grandes.

### Resumen

|                          | `foldl`                    | `foldr`                            |
| ------------------------ | -------------------------- | ---------------------------------- |
| Estado entre llamadas    | Lo pasa como nueva semilla | Lo guarda en el stack              |
| Operaciones pendientes   | Ninguna                    | Una por elemento                   |
| Riesgo de Stack Overflow | No                         | Sí, en listas grandes              |
| Listas infinitas         | ✗ Nunca termina            | ✓ Puede terminar (lazy evaluation) |

> `foldr` compensa su desventaja con el stack gracias a la **evaluación diferida**: si la función binaria puede determinar el resultado sin evaluar el segundo argumento (como `False && _`), no necesita seguir recorriendo la lista y termina antes.


### ¿Qué es la recursividad a la cola (tail recursion)?

Una función es **recursiva a la cola** cuando la llamada recursiva es **lo último que hace**, sin ninguna operación pendiente después de ella.

```haskell
-- Recursivo a la cola: la llamada recursiva es lo último, no hay nada pendiente
foldl f i (x:xs) = foldl f (f i x) xs
--                 ^^^^^^^^^^^^^^^^^^^^ esto es todo, no hay nada esperando afuera

-- NO es recursivo a la cola: hay una operación (+) pendiente afuera de la llamada
foldr f i (x:xs) = f x (foldr f i xs)
--                 ^^^ esta suma está esperando que termine la llamada recursiva
```

La ventaja es que si la llamada recursiva es lo último que hay que hacer, el programa no necesita recordar en qué estado estaba antes de llamarse: puede descartar el frame actual y reemplazarlo por el nuevo. Esto hace que la memoria usada sea constante sin importar el tamaño de la lista.


### ¿Por qué `foldl` necesita llegar a la lista vacía?

Mirando la definición:

```haskell
foldl f i []     = i                  -- único lugar donde devuelve algo
foldl f i (x:xs) = foldl f (f i x) xs -- solo se llama a sí mismo, no devuelve nada
```

El único lugar donde `foldl` **devuelve un valor** es en el caso base, cuando la lista está vacía. En el caso recursivo solo se llama a sí mismo. Entonces el flujo es siempre:

```
llamada → llamada → llamada → ... → lista vacía → recién ahí devuelve
```

No hay ningún punto intermedio donde pueda decir "ya sé el resultado, paro acá".

`foldr` en cambio puede devolver en el caso recursivo:

```haskell
foldr f i (x:xs) = f x (foldr f i xs)
```

`f x` se evalúa **antes** de la llamada recursiva. Si `f x` ya puede determinar el resultado sin necesitar el segundo argumento, Haskell nunca evalúa `(foldr f i xs)` gracias a la lazy evaluation:

```haskell
foldr (&&) False (repeat False)
-- evalúa: (&&) False (foldr ...)
-- False && _ = False sin importar lo que siga
-- → termina de inmediato, sin recorrer la lista infinita ✓

foldl (&&) False (repeat False)
-- necesita llegar a la lista vacía para devolver algo
-- la lista es infinita → nunca llega → loop infinito ✗
```

> **En una frase: `foldl` solo puede devolver algo cuando la lista se terminó. `foldr` puede devolver algo en cualquier paso.**

### Regla práctica

| Situación | Usar |
|---|---|
| Lista finita, operación no puede cortocircuitar (sumas, productos) | `foldl` |
| Lista potencialmente infinita | `foldr` |
| Operación que puede terminar antes de recorrer toda la lista | `foldr` |



### `foldr ($)` y `foldl (flip $)` — aplicar una lista de funciones en cadena

#### Tipos de `foldr` y `foldl`

```haskell
foldr :: (a -> b -> b) -> b -> [a] -> b
--        elemento        semilla  lista
--        primero

foldl :: (b -> a -> b) -> b -> [a] -> b
--        semilla         semilla  lista
--        primero
```

La diferencia clave está en el orden de la función binaria:
- `foldr`: recibe primero el **elemento de la lista**, luego la **semilla**
- `foldl`: recibe primero la **semilla**, luego el **elemento de la lista**

#### Con `foldr ($)`

`($)` espera primero la función y luego el valor, lo cual coincide con el orden de `foldr` (elemento primero). Funciona naturalmente:

```haskell
foldr ($) 0 [(^6), (2*), (1+)]
-- intercala $ de derecha a izquierda:
(^6) $ (2*) $ (1+) $ 0
(^6) $ (2*) $ 1        -- (1+) 0 = 1
(^6) $ 2               -- (2*) 1 = 2
64                     -- (^6) 2 = 64
```

#### Con `foldl ($)`

`foldl` pasa primero la semilla y luego el elemento. Con `($)` eso significa `semilla $ función`, es decir, intenta aplicar el **valor como si fuera una función**. No tiene sentido:

```haskell
foldl ($) 0 [(1+), (2*), (^6)]   -- ERROR: intenta hacer 0 $ (1+)
```

La solución es `flip ($)`, que invierte el orden: primero el valor, luego la función:

```haskell
foldl (flip ($)) 0 [(1+), (2*), (^6)]
-- va de izquierda a derecha:
(1+) aplicada a 0 = 1
(2*) aplicada a 1 = 2
(^6) aplicada a 2 = 64
```

#### Comparación

```haskell
-- foldr: funciones en orden inverso al de aplicación
foldr ($)        0 [(^6), (2*), (1+)]   -- aplica: (1+) → (2*) → (^6)

-- foldl: funciones en orden de aplicación, pero necesita flip ($)
foldl (flip ($)) 0 [(1+), (2*), (^6)]   -- aplica: (1+) → (2*) → (^6)
```

Ambos producen el mismo resultado (`64`), pero el orden de las funciones en la lista es inverso.
````
