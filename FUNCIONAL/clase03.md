# Módulo 2 — Paradigma Funcional: Composición y Aplicación Parcial

## Composición

La composición permite construir **nuevas funciones** a partir de la aplicación sucesiva de funciones existentes. Matemáticamente:

```
g(f(x)) = (g ∘ f) x
```

En Haskell se usa el operador `.`:

```haskell
g (f x) = (g . f) x
```

> **Importante:** primero se aplica `f` y luego `g`, pero al escribirlo en Haskell el orden se invierte: `g . f`.

**Restricción:** la imagen de `f` debe coincidir con el dominio de `g`.

---

### Ejemplo 1: cuádruple

```haskell
-- 4 * x = 2 * (2 * x), es decir, aplicamos doble dos veces
cuadruple = doble . doble
```

Se puede simplificar porque `doble . doble` es una expresión entre paréntesis que genera una nueva función, sin necesidad de nombrar el parámetro.

---

### Ejemplo 2: longitud de nombre par

"Saber si la longitud del nombre de una persona es una cantidad par"

```haskell
nombrePar :: String -> Bool
nombrePar = even . length
```

- `length :: String -> Int` — longitud del String
- `even :: Int -> Bool` — si el número es par
- La composición da `String -> Bool`, que es exactamente lo que necesitamos

---

### Ejemplo 3: mayor de edad

```haskell
type Persona = (String, Integer)

edad :: Persona -> Integer
edad = snd   -- snd devuelve el segundo elemento de una tupla

mayorDeEdad :: Integer -> Bool
mayorDeEdad e = e > 18

esMayorEdad :: Persona -> Bool
esMayorEdad = mayorDeEdad . edad
```

```haskell
laura = ("Laura", 41)
-- ghci> esMayorEdad laura
-- True
```

---

### Composición de más de dos funciones

```haskell
esMenorEdad :: Persona -> Bool
esMenorEdad = not . mayorDeEdad . edad

-- ghci> esMenorEdad laura
-- False
```

La única restricción es que la imagen de cada función coincida con el dominio de la siguiente.

---

## Acoplamiento

El **acoplamiento** es el grado en que dos componentes se conocen entre sí.

- `esMayorEdad` y `edad` están acopladas (necesariamente, para resolver el requerimiento)
- Pero ese acoplamiento es **bajo**: si cambia la implementación interna de `edad` (por ejemplo, la tupla `Persona` agrega un campo), `esMayorEdad` no necesita cambiar
- Lo que importa es la **interfaz** (qué recibe y qué devuelve), no la implementación interna

```haskell
-- Si Persona pasa a tener domicilio:
type Persona = (String, Integer, String)
laura = ("Laura", 41, "Medrano 951 CABA")

edad :: Persona -> Integer
edad (_, e, _) = e   -- pattern matching sobre la tupla

-- esMayorEdad NO cambia:
esMayorEdad :: Persona -> Bool
esMayorEdad = mayorDeEdad . edad
```

---

## Aplicación parcial

Consiste en **no pasarle todos los parámetros** a una función, obteniendo como resultado una nueva función.

```haskell
mod :: Int -> Int -> Int   -- función completa: resto de la división

mod 6 :: Int -> Int        -- aplicación parcial: dado n, devuelve resto de 6/n

mod 6 4 :: Int             -- aplicación total: devuelve 2
```

---

### Ejemplo: siguiente

```haskell
-- Definición explícita
siguiente :: Integer -> Integer
siguiente n = n + 1

-- Con aplicación parcial del operador (+)
siguiente = (1 +)
```

Más ejemplos:

```haskell
doble    = (2 *)
cuadrado = (^ 2)
cubo     = (^ 3)
```

> **Nota:** para operadores conmutativos como `+` y `*`, el orden no importa: `(2+)` y `(+2)` son equivalentes. Para `^` sí importa: `(2^)` y `(^2)` son distintas.

---

## Composición + Aplicación parcial

Combinar ambas herramientas permite resolver requerimientos reutilizando funciones existentes sin necesidad de definir funciones nuevas.

---

### Ejemplo 1: comienza con 'p'

"Saber si una palabra comienza con p"

```haskell
-- head toma el primer elemento de una lista (o String)
-- ('p' ==) es (==) aplicado parcialmente: dado un Char, dice si es 'p'

comenzaConP :: String -> Bool
comenzaConP = ('p' ==) . head

-- ghci> comenzaConP "palabras"
-- True
```

---

### Ejemplo 2: costo de estacionamiento

"El costo es de 50 pesos la hora, con un mínimo de 2 horas"

```haskell
-- max 2: aplicación parcial, devuelve el mayor entre 2 y el argumento
-- (* 50): aplicación parcial, multiplica por 50

costoEstacionamiento :: Integer -> Integer
costoEstacionamiento = (* 50) . max 2

-- ghci> costoEstacionamiento 1  →  100  (mínimo 2 hs * 50)
-- ghci> costoEstacionamiento 5  →  250  (5 hs * 50)
```

> **Error común:** escribir `(* 50) . max 2 horas` es incorrecto porque `max 2 horas` se reduce a un valor entero, no a una función, y no se puede componer.

---

## Resumen

| Concepto           | Qué hace                                                           | Ejemplo                                 |
| ------------------ | ------------------------------------------------------------------ | --------------------------------------- |
| Composición (`.`)  | Construye una nueva función aplicando funciones en cadena          | `esMayorEdad = mayorDeEdad . edad`      |
| Aplicación parcial | Genera una nueva función pasando menos argumentos de los esperados | `siguiente = (1 +)`                     |
| Combinación        | Reutiliza funciones existentes sin definir nuevas                  | `costoEstacionamiento = (* 50) . max 2` |

## Conceptos clave

`composición` · `aplicación parcial` · `acoplamiento` · `tupla` · `pattern matching` · `type alias` · `notación punto`

```

```
