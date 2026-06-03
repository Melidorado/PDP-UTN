# Módulo 3 — Paradigma Funcional: Modelado de información

## Listas

Una lista es una serie de elementos **del mismo tipo**. Puede estar vacía.

```haskell
[1, 2, 3]
["Hola", "mundo"]
"Hola"  -- equivale a ['H','o','l','a']
```

### Formas de generar listas

```haskell
[1..10]        -- [1,2,3,4,5,6,7,8,9,10]
[1, 3..16]     -- [1,3,5,7,9,11,13,15]
[8, 7.. -2]    -- [8,7,6,5,4,3,2,1,0,-1,-2]
[1..]          -- lista infinita
```

Todos los elementos deben ser del mismo tipo, de lo contrario Haskell lanza un error.

### Pattern matching sobre listas

| Pattern    | Denota                                                                                |
| ---------- | ------------------------------------------------------------------------------------- |
| `[]`       | Lista vacía                                                                           |
| `(x:xs)`   | Lista con al menos un elemento: `x` es la cabeza, `xs` es la cola (siempre una lista) |
| `(x:y:ys)` | Lista con al menos dos elementos                                                      |
| `[x]`      | Lista de exactamente un elemento                                                      |
| `[x, y]`   | Lista de exactamente dos elementos                                                    |

**Importante:** la cola siempre es una lista. Para `[1, 2, 3]`: cabeza = `1`, cola = `[2, 3]`.

Una lista `[1, 2, 3]` puede escribirse como:

```haskell
(1:(2:(3:[])))
```

### Funciones sobre listas

```haskell
head (x:_)  = x   -- cabeza: head :: [a] -> a
tail (_:xs) = xs  -- cola:   tail :: [a] -> [a]
```

| Función   | Qué hace                         | Ejemplo                           |
| --------- | -------------------------------- | --------------------------------- |
| `length`  | Longitud de la lista             | `length [2..4]` → `3`             |
| `sum`     | Suma los elementos               | `sum [4, 9, 1]` → `14`            |
| `(++)`    | Concatena dos listas             | `[3..5] ++ [4,6]` → `[3,4,5,4,6]` |
| `take n`  | Primeros n elementos             | `take 3 [4..11]` → `[4,5,6]`      |
| `drop n`  | Lista sin los primeros n         | `drop 2 "CIENCIA"` → `"ENCIA"`    |
| `(!!)`    | Elemento en posición n (desde 0) | `[1..10] !! 3` → `4`              |
| `reverse` | Invierte la lista                | `reverse "ANITA"` → `"ATINA"`     |

La variable anónima `_` se usa cuando no nos interesa un valor en el pattern matching.

---

## Tuplas

Una tupla es un tipo de dato compuesto con **elementos de distinto tipo** y **cantidad fija** de elementos.

```haskell
(23, 02, 1973)       -- fecha: (Int, Int, Int)
("Juan", 23)         -- persona: (String, Int)
((1,2),(3,4))        -- tupla de tuplas
```

### Pattern matching sobre tuplas

```haskell
(x, y)    -- tupla de dos elementos
(x, y, z) -- tupla de tres elementos
```

### Comparación listas vs. tuplas

|                       | Lista                        | Tupla                               |
| --------------------- | ---------------------------- | ----------------------------------- |
| Tipos de elementos    | Homogéneos (todos iguales)   | Heterogéneos (pueden ser distintos) |
| Cantidad de elementos | Variable, puede ser infinita | Fija                                |
| Estructura            | Recursiva                    | No recursiva                        |

### Funciones sobre tuplas

```haskell
fst :: (a, b) -> a    -- primer elemento
snd :: (a, b) -> b    -- segundo elemento
```

Conviene definir funciones con nombres de dominio en lugar de usar `fst`/`snd` directamente:

```haskell
notas = snd   -- más expresivo que usar snd directamente
```

### Sinónimos de tipo

```haskell
type Complejo = (Float, Float)

sumarComplejos :: Complejo -> Complejo -> Complejo
sumarComplejos (real1, im1) (real2, im2) = (real1 + real2, im1 + im2)
```

---

## Tipos propios (`data`)

Permiten definir nuevos tipos de datos con constructores propios.

```haskell
data Persona = Persona String Int
```

Se lee: "el tipo `Persona` usa un constructor `Persona` que recibe un `String` y un `Int`".

### Pattern matching sobre `data`

```haskell
nombre (Persona n _) = n
edad   (Persona _ e) = e
```

### `data` vs. tupla

|                            | Tupla                           | `data`                           |
| -------------------------- | ------------------------------- | -------------------------------- |
| Expresividad               | Baja (estructura genérica)      | Alta (nombre propio del dominio) |
| Tipo esperado en funciones | `(a, b)` — acepta cualquier par | `Persona` — solo acepta personas |
| Claridad                   | Requiere conocer la estructura  | El nombre documenta la intención |

---

## Record syntax

Cuando el tipo tiene muchos campos, se usa **record syntax** para mayor expresividad:

```haskell
data Persona = Persona {
    nombre          :: String,
    edad            :: Int,
    domicilio       :: String,
    telefono        :: String,
    fechaNacimiento :: (Int, Int, Int),
    buenaPersona    :: Bool,
    plata           :: Float
} deriving (Show)
```

### Ventajas del record syntax

- Las funciones `nombre`, `edad`, `domicilio`, etc. se generan automáticamente
- Al crear un valor se puede especificar en cualquier orden
- Más legible que una lista de parámetros posicionales

```haskell
juan = Persona {
    nombre          = "Juan",
    edad            = 29,
    domicilio       = "Ayacucho 554",
    telefono        = "45232598",
    fechaNacimiento = (17, 7, 1988),
    buenaPersona    = True,
    plata           = 30.0
}
```

### `deriving (Show)`

Sin esto, Haskell no sabe cómo mostrar el tipo en consola y lanza un error. Se agrega al final de la definición `data`.

---

## Múltiples constructores

Un tipo puede tener más de un constructor, útil para enumeraciones:

```haskell
data ColorPrimario = Rojo | Amarillo | Azul

recargoPorColor :: ColorPrimario -> Int
recargoPorColor Rojo = 50
recargoPorColor _    = 20   -- por descarte: Amarillo o Azul
```

---

## Ejercicio integrador: Alumnos

### Modelado del dominio

```haskell
-- Parcial: materia y cantidad de preguntas
data Parcial = Parcial String Int deriving (Show)

materia           :: Parcial -> String
materia           (Parcial mat _) = mat

cantidadPreguntas :: Parcial -> Int
cantidadPreguntas (Parcial _ cant) = cant

-- CriterioEstudio: función que dado un parcial dice si el alumno va a estudiar
type CriterioEstudio = Parcial -> Bool

-- Alumno
data Alumno = Alumno {
    nombre          :: String,
    fechaNacimiento :: (Int, Int, Int),
    legajo          :: Int,
    materiasQueCursa :: [String],
    criterioEstudio :: CriterioEstudio
} deriving (Show)
```

### Criterios de estudio

```haskell
-- Estudioso: siempre estudia
estudioso :: CriterioEstudio
estudioso _ = True

-- Hijo del rigor: estudia si el parcial tiene más de n preguntas
hijoDelRigor :: Int -> CriterioEstudio
hijoDelRigor n (Parcial _ preguntas) = preguntas > n

-- Cabulero: estudia si la materia tiene cantidad impar de letras
cabulero :: CriterioEstudio
cabulero (Parcial materia _) = (odd . length) materia
```

### Modelar a Nico y cambiar su criterio

```haskell
nico = Alumno {
    nombre           = "Nico",
    fechaNacimiento  = (10, 3, 1993),
    legajo           = 124124,
    materiasQueCursa = ["sysop", "proyecto"],
    criterioEstudio  = estudioso
}

-- Cambiar criterio sin modificar el original (sin efecto colateral)
cambiarCriterioEstudio :: CriterioEstudio -> Alumno -> Alumno
cambiarCriterioEstudio nuevoCriterio alumno = alumno {
    criterioEstudio = nuevoCriterio
}
```

### Saber si un alumno va a estudiar

```haskell
estudia :: Parcial -> Alumno -> Bool
estudia parcial alumno = (criterioEstudio alumno) parcial

parcialPDP = Parcial "PDP" 3

-- ghci> (estudia parcialPDP . cambiarCriterioEstudio (hijoDelRigor 5)) nico
-- False  (3 preguntas no supera el mínimo de 5)

-- ghci> (estudia parcialPDP . cambiarCriterioEstudio (hijoDelRigor 2)) nico
-- True   (3 preguntas supera el mínimo de 2)
```

> **Sin efecto colateral:** en el paradigma funcional no se modifica el valor original. `cambiarCriterioEstudio` recibe un alumno y devuelve uno **nuevo** con el criterio cambiado.

---

## Resumen

| Tipo           | Características                      | Cuándo usarlo                               |
| -------------- | ------------------------------------ | ------------------------------------------- |
| Lista `[a]`    | Homogénea, longitud variable         | Colecciones del mismo tipo                  |
| Tupla `(a, b)` | Heterogénea, longitud fija           | Agrupar datos relacionados de distinto tipo |
| `data`         | Tipo propio con constructor nombrado | Entidades del dominio con semántica clara   |
| Record syntax  | `data` con campos nombrados          | Tipos con muchos campos                     |

## Conceptos clave

`lista` · `tupla` · `data` · `record syntax` · `pattern matching` · `sinónimo de tipo` · `múltiples constructores` · `deriving Show` · `efecto colateral` · `abstracción`

```

```
