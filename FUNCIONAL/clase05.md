# Módulo 4 — Paradigma Funcional: Recursividad y Evaluación Diferida

## Recursividad

Una función puede llamarse a sí misma. Es el mecanismo que reemplaza a las estructuras de iteración (`while`, `for`) del paradigma imperativo.

Todo algoritmo recursivo necesita:

- **Caso base:** corta la recursividad
- **Caso recursivo:** la función se llama a sí misma con un valor más cercano al caso base

---

## Ejemplo 1: Factorial

```haskell
-- Con guardas
factorial n
  | n == 0 = 1
  | n > 0  = n * factorial (n - 1)

-- Con pattern matching (preferible)
factorial 0 = 1
factorial n = n * factorial (n - 1)
```

```haskell
factorial :: Int -> Int
```

El orden de los patrones importa: Haskell los evalúa de arriba hacia abajo y aplica el primero que encaje.

---

## Recursividad e inducción

La recursividad está ligada al concepto matemático de inducción:

- **Caso base** → P(0): se verifica directamente
- **Caso recursivo** → P(N+1) definido en base a P(N)

**Metáfora del dominó:** si el primero cae (caso base) y cada ficha derriba a la siguiente (caso recursivo), caen todas.

---

## Ejemplo 2: Fibonacci

```haskell
fibonacci 0 = 1
fibonacci 1 = 1
fibonacci n = fibonacci (n - 1) + fibonacci (n - 2)
```

Dos casos base (para `0` y `1`) y un caso recursivo que combina los dos anteriores.

---

## Ejemplo 3: número primo

```haskell
primo :: Int -> Bool
primo 1 = False
primo 2 = True
primo n = noHayDivisores 2 (n - 1) n

noHayDivisores :: Int -> Int -> Int -> Bool
noHayDivisores minimo maximo n
    | minimo `esDivisorDe` n = False
    | minimo == maximo       = True
    | otherwise              = noHayDivisores (minimo + 1) maximo n

esDivisorDe :: Int -> Int -> Bool
esDivisorDe unNumero otroNumero = mod otroNumero unNumero == 0
```

> **Nota:** los parámetros `minimo` y `maximo` sirven para almacenar el **estado** de la ejecución. En el paradigma funcional, el estado se pasa como argumento en lugar de guardarse en variables mutables.

Reducción de ejemplo:

```
primo 5 = noHayDivisores 2 4 5
        = noHayDivisores 3 4 5
        = noHayDivisores 4 4 5
        = True   -- minimo == maximo
```

---

## Recursividad con listas

La lista tiene estructura recursiva:

- Caso base: lista vacía `[]`
- Caso recursivo: cabeza + cola `(x:xs)`

### `length`

```haskell
length :: [a] -> Int
length []     = 0
length (_:xs) = 1 + length xs
```

### `sum`

```haskell
sum :: Num a => [a] -> a
sum []     = 0
sum (x:xs) = x + sum xs
```

### `last`

```haskell
last :: [a] -> a
last [x]    = x          -- lista de un solo elemento
last (_:xs) = last xs    -- lista de dos o más
```

### `take`

```haskell
take :: Int -> [a] -> [a]
take n _      | n <= 0 = []
take _ []              = []
take n (x:xs)          = x : take (n - 1) xs
```

### `drop`

```haskell
drop :: Int -> [a] -> [a]
drop 0 xs     = xs
drop n (_:xs) = drop (n - 1) xs
```

### `(!!)` — elemento en posición n

```haskell
(!!) :: [a] -> Int -> a
(!!) (x:_)  0 = x
(!!) (_:xs) n = xs !! (n - 1)
```

### `elem`

```haskell
elem :: Eq a => a -> [a] -> Bool
elem _ []     = False
elem e (x:xs) = e == x || elem e xs
```

### `(++)` — concatenación

```haskell
(++) :: [a] -> [a] -> [a]
(++) []     ys = ys
(++) (x:xs) ys = x : (xs ++ ys)
```

### `reverse`

```haskell
reverse :: [a] -> [a]
reverse []     = []
reverse (x:xs) = reverse xs ++ [x]
```

### `maximum`

```haskell
maximum :: Ord a => [a] -> a
maximum [x]      = x
maximum (x:y:ys)
    | x > y     = maximum (x:ys)
    | otherwise = maximum (y:ys)
```

---

## Evaluación diferida (lazy evaluation)

### El problema

```haskell
muchosDe n = n : (muchosDe n)  -- lista infinita de n
```

Con evaluación **ansiosa** (como C o Java), esto entraría en loop infinito al evaluarse. Con evaluación **diferida**, Haskell solo evalúa lo que realmente necesita:

```haskell
(head . muchosDe) 5          -- 5        ✓ solo evalúa el primer elemento
(sum . take 10 . muchosDe) 5 -- 50       ✓ solo evalúa 10 elementos
(sum . muchosDe) 5           -- diverge  ✗ necesita sumar infinitos elementos
```

### Definición

La **evaluación diferida** (o perezosa) consiste en evaluar los argumentos solo cuando son necesarios, no antes.

> **Metáfora:** es como pedirle al ferretero 10 cm de cable. La evaluación ansiosa desenrolla todo el rollo de 100 m para luego cortar. La evaluación diferida corta directamente los 10 cm que necesitás.

### Ventajas

- Solo se evalúa lo que realmente se necesita
- Permite trabajar con estructuras **potencialmente infinitas** siempre que el algoritmo converja
- Es posible gracias a la **transparencia referencial**: como no hay efecto colateral, se puede cambiar el orden de evaluación sin afectar el resultado

> **¿Por qué no existe en C?** Por el efecto colateral. Si cambiar el orden de evaluación puede modificar el estado del programa, no podés diferirlo libremente.

---

## Resumen

| Concepto              | Descripción                                                             |
| --------------------- | ----------------------------------------------------------------------- |
| Recursividad          | Función que se llama a sí misma; reemplaza la iteración imperativa      |
| Caso base             | Corta la recursividad                                                   |
| Caso recursivo        | Reduce el problema hacia el caso base                                   |
| Estado como argumento | En funcional, el estado se pasa como parámetro (sin variables mutables) |
| Evaluación diferida   | Se evalúa solo lo necesario; permite listas infinitas                   |

## Conceptos clave

`recursividad` · `caso base` · `caso recursivo` · `inducción` · `pattern matching` · `evaluación diferida` · `lazy evaluation` · `listas infinitas` · `transparencia referencial`

```

```
