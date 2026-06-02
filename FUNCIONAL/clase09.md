````markdown
# Módulo 7 — Paradigma Funcional: Sistema de Tipos

## Introducción

Hasta el momento conocemos:

- **Tipos simples:** `Int`, `Bool`, `Float`, `Char`, funciones
- **Tipos compuestos:** `String`, listas, tuplas, `data`
- **Sinónimos de tipo:** `type Nombre = String` — no hay diferencia técnica entre `Nombre` y `String`, pero acerca el código al dominio del problema

En este módulo explicaremos el sistema de tipos de Haskell, que permite **inferir** el tipo de cada función o expresión automáticamente.

---

## Tipos genéricos

### La función `id`

```haskell
id x = x
```

```haskell
λ id True
True :: Bool

λ id "hola"
"hola" :: [Char]

λ id head
<function>
```

```haskell
id :: a -> a
```

`a` representa una variable de tipo sin restricciones: cualquier tipo puede participar.

> **Distinción importante:** hay dos tipos de variables con el mismo nombre:
>
> - **Variable de tipo** (`a` en `id :: a -> a`): usada por el compilador para verificar corrección de tipos
> - **Variable de valor** (`a` en `id a = a`): incógnita que se resuelve al aplicar la función

---

### Funciones genéricas con listas

```haskell
head :: [a] -> a
tail :: [a] -> [a]
last :: [a] -> a
```

`length` no usa el valor de cada elemento, solo cuenta:

```haskell
length [] = 0
length (_:xs) = 1 + length xs

length :: [a] -> Int
```

```haskell
λ length [head, last]
2

λ length [(4, True), (8, False), (5, False)]
3
```

Otras funciones genéricas con listas: `map`, `all`, `any`, `filter`, familia `fold`.

---

### Funciones genéricas con tuplas

```haskell
fst :: (a, b) -> a
snd :: (a, b) -> b
```

`a` y `b` son variables distintas: la tupla puede tener elementos de diferente tipo.

```haskell
(2, 5)    :: (Int, Int)    -- a es Int, b es Int
(2, True) :: (Int, Bool)   -- a es Int, b es Bool
```

---

### Polimorfismo paramétrico

Las funciones que aceptan valores de cualquier tipo **sin restricciones** tienen **polimorfismo paramétrico**: se definen una sola vez y funcionan para todos los tipos.

---

## Typeclasses

### El problema

`(+)` funciona para `Int`, `Float`, `Double`... pero no para todos los tipos:

```haskell
λ True + False   -- Error
λ last + head    -- Error
```

No podemos escribir `(+) :: a -> a -> a` porque no aplica a todos los tipos. Necesitamos una **restricción**.

---

### Definición de typeclass

Un typeclass:

- **Agrupa tipos** con características comunes
- **Define una interfaz**: operaciones que todos los tipos pertenecientes deben implementar
- Permite definir **comportamiento por defecto** para ciertas operaciones

```haskell
-- La restricción se escribe antes del =>
(+) :: Num a => a -> a -> a
(>) :: Ord a => a -> a -> Bool
(==) :: Eq a => a -> a -> Bool
```

---

### `Num` — tipos numéricos

Agrupa: `Int`, `Float`, `Double`, `Integer`, entre otros.

```haskell
(+), (-), (*) :: Num a => a -> a -> a
negate, abs, signum :: Num a => a -> a
```

Ejemplo de inferencia:

```haskell
funcionLoca (x, y) = abs x + y
-- x necesita abs → Num
-- y se suma con el resultado de abs x → Num
funcionLoca :: Num a => (a, a) -> a
```

---

### `Ord` — tipos ordenables

Agrupa: `Int`, `Float`, `Double`, `Char`, `Bool`, `String`, tuplas, entre otros.

```haskell
(<), (>), (<=), (>=) :: Ord a => a -> a -> Bool
max, min :: Ord a => a -> a -> a
```

```haskell
λ "hola" > "chau"    -- True
λ (3, 2) > (1, 0)    -- True
λ max 4 7            -- 7
λ last > head        -- Error: las funciones no se pueden ordenar
```

La función `max`:

```haskell
max a b | a > b     = a
        | otherwise = b

max :: Ord a => a -> a -> a
```

---

### `Eq` — tipos comparables por igualdad

Agrupa: `Int`, `Float`, `Double`, `Char`, `Bool`, `String`, listas, entre otros.

```haskell
(==) :: Eq a => a -> a -> Bool
(/=) :: Eq a => a -> a -> Bool
```

```haskell
λ 1 == 3              -- False
λ [1, 3, 2] == [1, 2, 3]  -- False
λ (3, 2) == (2+1, 2)  -- True
λ last == head        -- Error: las funciones no se pueden comparar
```

Ejemplo: el tipo inferido de `elem` depende de `Eq`:

```haskell
elem _     []     = False
elem valor (x:xs) = valor == x || elem valor xs

elem :: Eq a => a -> [a] -> Bool
```

- `valor` y `x` deben ser del mismo tipo
- Deben poder compararse con `(==)` → restricción `Eq`

---

### Un tipo pertenece a muchas clases

- Una clase agrupa varios tipos (interfaz común)
- Un tipo puede pertenecer a **varios typeclasses** a la vez
- Cada typeclass que implementa obliga al tipo a definir sus funciones

Por ejemplo, `Int` es instancia de `Num`, `Ord` y `Eq` simultáneamente.

---

### Relaciones entre typeclasses

Todo `Ord` es también `Eq`: para poder ordenar dos valores, primero deben poder compararse por igualdad. `Ord` **especializa** `Eq`.

```
Eq  ←  Ord
```

---

### Polimorfismo ad-hoc

Las funciones restringidas por un typeclass (no aplican a todos los tipos, pero sí a varios) tienen **polimorfismo ad-hoc**:

```haskell
(==) :: Eq a  => a -> a -> Bool
(>)  :: Ord a => a -> a -> Bool
(+)  :: Num a => a -> a -> a
```

|             | Polimorfismo paramétrico | Polimorfismo ad-hoc                  |
| ----------- | ------------------------ | ------------------------------------ |
| Restricción | Ninguna (`a`)            | Typeclass (`Num a`, `Ord a`, `Eq a`) |
| Ejemplo     | `id :: a -> a`           | `(+) :: Num a => a -> a -> a`        |
| Aplica a    | Cualquier tipo           | Solo tipos instancia del typeclass   |

---

## Ejercicios de inferencia de tipos

Al calcular el tipo de una función, hay que mirar las expresiones usadas en su definición e identificar qué restricciones imponen sobre cada parámetro.

### Ejemplo 1

```haskell
f x _ [] = x
f x y (z:zs)
    | y z > 0   = z + f x y zs
    | otherwise = f x y zs
```

Análisis:

- Tercer parámetro es una lista → `[b]`
- Por el primer patrón, `f` devuelve el mismo tipo que `x` → tipo `a`
- `y` se aplica sobre `z` y el resultado se compara con `> 0` → `y :: b -> c` donde `c` es `Ord` y `Num`
- `z` se suma con el resultado de `f` → `z` debe ser `Num`; el tipo `a = b`

```haskell
f :: (Num a, Ord c, Num c) => a -> (a -> c) -> [a] -> a
```

---

### Ejemplo 2 (parcial)

```haskell
f a b c d e
    | (a . d e) (1, True) = 0
    | otherwise           = length (b:c) + e
```

Análisis:

- `f` devuelve `Int` (por `0` y por `length`)
- `b` es cualquier tipo, `c` es lista del mismo tipo
- `e :: Int` (se suma con `length`)
- `d` recibe `e` (Int) y una tupla `(Int, Bool)` → `d :: Int -> (Int, Bool) -> b`
- `a` recibe la imagen de `d` y devuelve `Bool` (para la guarda)

```haskell
f :: (b -> Bool) -> a -> [a] -> (Int -> (Int, Bool) -> b) -> Int -> Int
```

---

### Ejemplo 3 (final)

```haskell
f x y = (> 10) . head . filter (x y) . map y
```

Análisis:

- `y` es una función, `map y` transforma `[a]` en `[b]`
- `(x y) :: b -> Bool` → `x :: (a -> b) -> b -> Bool`
- `head` toma el primer `b`
- `(> 10)` requiere que `b` sea `Ord` y `Num`
- `f` tiene un tercer argumento implícito: la lista `[a]`

```haskell
f :: (Ord b, Num b) => ((a -> b) -> b -> Bool) -> (a -> b) -> [a] -> Bool
```

---

## BONUS: Profundizando typeclasses

### Mecanismo de derivación (`deriving`)

```haskell
data Nota = Insuficiente | Regular | Aprobado | Sobresaliente
    deriving (Eq, Ord, Show, Enum)
```

Sin `deriving`:

```haskell
λ Sobresaliente < Regular   -- Error: No instance for (Ord Nota)
λ show Regular              -- Error: No instance for (Show Nota)
λ Sobresaliente == Regular  -- Error: No instance for (Eq Nota)
```

Con `deriving`:

```haskell
λ Sobresaliente < Regular         -- False
λ Sobresaliente == Regular        -- False
λ show Regular                    -- "Regular"
λ Sobresaliente > Regular         -- True
λ enumFromTo Insuficiente Aprobado -- [Insuficiente, Regular, Aprobado]
```

Haskell genera las implementaciones automáticamente según el **orden de declaración** de los constructores. Para enumeraciones, el orden importa: `Insuficiente` es el "menor" y `Sobresaliente` el "mayor".

> **Regla práctica:** siempre agregar al menos `deriving (Show)` para poder ver el tipo en consola. Agregar `deriving (Eq)` para poder comparar valores. Si el `data` contiene funciones, `deriving (Show)` y `deriving (Eq)` no funcionan automáticamente.

---

### Definición de `Eq`

```haskell
class Eq a where
    (==), (/=) :: a -> a -> Bool
    x /= y = not (x == y)   -- método default
    x == y = not (x /= y)   -- método default
```

- `a` es una variable de tipo: cualquier tipo que sea instancia de `Eq` encaja aquí
- La **definición mínima completa** es implementar solo `(==)` o solo `(/=)`: el otro surge del método default
- Los **métodos default** son implementaciones heredadas por todos los tipos instancia del typeclass

---

### Definición de `Ord` en base a `Eq`

```haskell
class (Eq a) => Ord a where
    compare              :: a -> a -> Ordering
    (<), (<=), (>), (>=) :: a -> a -> Bool
    max, min             :: a -> a -> a
    max x y = if x <= y then y else x   -- método default
    min x y = if x <= y then x else y   -- método default
```

- `compare` no tiene implementación por defecto: hay que definirla para cada tipo
- `max` y `min` tienen métodos default que delegan en `(<=)`
- Todo tipo instancia de `Ord` **debe ser también instancia de `Eq`**

---

### Adhesión manual con `instance`

Cuando el comportamiento de `deriving` no es el que necesitamos, definimos la instancia manualmente.

**Ejemplo:** el `deriving (Ord)` compara personas por nombre (primer campo). Si queremos comparar por edad:

```haskell
data Persona = Persona {
    nombre  :: String,
    edad    :: Int,
    hobbies :: [String]
} deriving (Eq, Show)

instance Ord Persona where
    (>) unaPersona otraPersona = edad unaPersona > edad otraPersona
```

Sin definir `Eq` primero, Haskell lanza error porque todo `Ord` es también `Eq`.

```haskell
λ dodain > alf    -- True  (44 > 29)
λ dodain > nico   -- True  (44 > 30)
λ alf < nico      -- True  (29 < 30)
```

**Ejemplo con libros:** comparar por cantidad de autores en lugar del nombre:

```haskell
instance Ord Libro where
    (<=) unLibro otroLibro = (length . autores) unLibro <= (length . autores) otroLibro
```

**Ejemplo con `Eq` personalizado:** dos libros son iguales si tienen el mismo nombre:

```haskell
instance Eq Libro where
    (==) unLibro otroLibro = nombre unLibro == nombre otroLibro
```

> **Importante:** si definís `Eq` e `Ord` con criterios distintos (por ejemplo, igualdad por nombre y orden por autores), puede haber comportamientos inconsistentes. Es recomendable que ambos criterios sean coherentes.

Para ver de qué typeclasses es instancia un tipo:

```haskell
λ :i Persona
instance Eq Persona
instance Ord Persona
instance Show Persona
```

---

### Mostrar funciones en consola

Las funciones no son instancia de `Show` por defecto:

```haskell
λ map (+ 1)
Error -- No instance for (Show ([b0] -> [b0]))
```

Solución:

```haskell
import Text.Show.Functions

-- Equivale a definir:
instance Show (a -> b) where
    show _ = "<function>"
```

```haskell
λ map (+ 1)   -- <function>
λ map         -- <function>
```

> Esto también explica por qué cuando un `data` tiene una función como campo, `deriving (Show)` y `deriving (Eq)` no funcionan: las funciones no tienen implementación de `Show` ni `Eq` por defecto. Hay que importar `Text.Show.Functions` o definir las instancias manualmente.

---

### Creando typeclasses propios

Los typeclasses son extensibles. Se pueden crear los propios:

```haskell
class Calificacion a where
    aprobo     :: a -> Bool
    promociono :: a -> Bool
```

> **Un typeclass no es un tipo**: es un mecanismo para que uno o más tipos suscriban un contrato (una interfaz). No se puede usar `Calificacion` como tipo de un campo en un `data`.

**Instancia para notas numéricas:**

```haskell
instance Calificacion Integer where
    aprobo nota     = nota >= 6
    promociono nota = nota >= 8
```

**Nota de concepto:**

```haskell
data NotaConceptual = Insuficiente | Regular | Aprobado | Sobresaliente
    deriving (Eq, Ord, Show, Enum)

instance Calificacion NotaConceptual where
    aprobo Insuficiente = False
    aprobo _            = True

    promociono Sobresaliente = True
    promociono _             = False
```

```haskell
λ aprobo 10              -- True
λ aprobo 4               -- False
λ aprobo Regular         -- True
λ promociono 7           -- False
λ promociono 10          -- True
λ promociono Sobresaliente -- True

λ :t aprobo
aprobo :: Calificacion a => a -> Bool

λ :i NotaConceptual
-- instance Eq NotaConceptual
-- instance Ord NotaConceptual
-- instance Show NotaConceptual
-- instance Calificacion NotaConceptual
```

Un tipo puede pertenecer a **múltiples typeclasses** simultáneamente. `NotaConceptual` es instancia de `Eq`, `Ord`, `Show` y `Calificacion`.

**Definición mínima completa:** como `aprobo` y `promociono` no se pueden derivar una de la otra, ambas son obligatorias. No hay métodos default en este caso.

**Instancia de `Show` personalizada:**

```haskell
instance Show NotaConceptual where
    show Insuficiente = "estás al horno"
    show _            = "es una nota de concepto"
```

---

## Resumen

| Concepto                 | Descripción                                                         |
| ------------------------ | ------------------------------------------------------------------- |
| Variable de tipo (`a`)   | Representa cualquier tipo, sin restricciones                        |
| Polimorfismo paramétrico | Función válida para todos los tipos (`id`, `head`)                  |
| Typeclass                | Agrupa tipos que comparten una interfaz                             |
| Polimorfismo ad-hoc      | Función válida para tipos de un typeclass (`Num`, `Ord`, `Eq`)      |
| Método default           | Operación implementada en el typeclass, heredada por sus instancias |
| Definición mínima        | Operaciones mínimas obligatorias para ser instancia de un typeclass |
| `deriving`               | Adhesión automática a un typeclass                                  |
| `instance`               | Adhesión manual con implementación propia                           |

## Conceptos clave

`typeclass` · `Num` · `Ord` · `Eq` · `Show` · `Enum` · `polimorfismo paramétrico` · `polimorfismo ad-hoc` · `variable de tipo` · `inferencia de tipos` · `deriving` · `instance` · `definición mínima` · `método default` · `instancia` · `interfaz`
````
