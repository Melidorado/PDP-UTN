# Módulo 6 — Paradigma Funcional: Expresiones Lambda y Currificación

## Cálculo lambda — origen

Haskell está basado en el **cálculo lambda**, un sistema de reglas de transformación diseñado por Alonzo Church en 1937. El motor de reducción de expresiones de Haskell está basado en él.

En cálculo lambda, la función identidad se escribe:

```
λx.x
```

- `λ` denota la función anónima
- `x` es el argumento
- `.x` es el cuerpo (lo que devuelve)

Al aplicarle el valor `2`, la regla de sustitución reemplaza `x` por `2` y devuelve `2`.

> **Detalle importante:** en cálculo lambda **todas las funciones son de un solo parámetro**. La suma de dos números no se puede escribir como `λx y. x + y` sino como `λx.(λy. x + y)`: una función que dado `x` devuelve una función que dado `y` devuelve `x + y`. Este es el origen de la currificación y la aplicación parcial.

---

## Expresiones lambda en Haskell

Haskell implementa el cálculo lambda con esta sintaxis:

```haskell
\x -> x * x          -- función cuadrado anónima
\x y -> x + y        -- función suma anónima (Haskell admite más de un parámetro)
```

```haskell
-- Evaluación:
(\x -> x * x) 2        -- 4
(\x y -> x + y) 2 3    -- 5
(\x y -> x + y) 2      -- devuelve una función (aplicación parcial)
```

La contrabarra `\` representa la letra griega λ. Luego de los parámetros separados por espacios, `->` separa los argumentos del cuerpo.

```haskell
-- También se puede nombrar:
siguiente = \n -> n + 1
doble     = \n -> n * 2

map siguiente [1..5]   -- [2,3,4,5,6]
map doble [1..5]       -- [2,4,6,8,10]
```

---

## Definición local con `where`

Para definir expresiones auxiliares dentro de una función sin exponerlas globalmente:

```haskell
numeroDeRaices a b c
    | discriminante > 0  = 2
    | discriminante == 0 = 1
    | discriminante < 0  = 0
    where discriminante = b*b - 4*a*c
```

Ventajas: tiene nombre explícito, se entiende mejor, y no contamina el scope global.

---

## Cuándo usar lambdas y cuándo no

### Preferir composición y aplicación parcial

En la cursada se prefiere composición y aplicación parcial antes que lambdas, porque demuestran mayor entendimiento del paradigma:

```haskell
-- Con lambda (evitar si se puede)
filter (\cliente -> edad cliente > 40) clientes

-- Con composición y aplicación parcial (preferible)
filter ((> 40) . edad) clientes

-- Lambda innecesaria: no aporta nada sobre llamar directo a even
filter (\x -> even x) [1..10]

-- Correcto: usar la función directamente
filter even [1..10]
```

### Cuándo sí usar lambdas

**1. Cuando la condición no se puede expresar limpiamente con composición:**

```haskell
-- Condición con OR: no se puede expresar con composición simple
filter ((\unaEdad -> unaEdad < 20 || unaEdad > 60) . edad) clientes
```

**2. Como adaptador cuando no se puede cambiar el orden de parámetros:**

```haskell
-- deudaSupera espera primero el cliente, luego el monto
-- No podemos conectarlo directamente con filter
deudaSupera :: Cliente -> Float -> Bool

-- Con flip:
filter (flip deudaSupera 100000) clientes

-- Con lambda (alternativa equivalente):
filter (\cliente -> deudaSupera cliente 100000) clientes
```

**3. Para operar sobre elementos de tuplas:**

```haskell
map (\(a, b) -> a + b) [(1,2), (3,5), (6,3)]   -- [3,8,9]
```

**4. Con `foldl` cuando la función binaria no se puede expresar con composición:**

```haskell
-- Total de cuerdas de una lista de guitarras
foldl (\totalCuerdas guitarra -> totalCuerdas + cuerdas guitarra) 0 guitarras

-- Guitarra más vieja
foldl (\guitarraMasVieja guitarra ->
    if anio guitarra < anio guitarraMasVieja
    then guitarra
    else guitarraMasVieja) cabeza cola
```

> **Regla general:** si podés nombrar bien la lógica, nombrala como función. Si no hay un nombre claro asociado a ese pedazo de lógica, o si es una función ad-hoc que no vas a reutilizar, usá lambda.

---

## Lambdas en otros lenguajes

Las lambdas no son exclusivas de Haskell. Se encuentran en Scala, Kotlin, Java, Ruby, JavaScript, entre otros.

### JavaScript — sin currificar

```javascript
// Sin currificar: no admite aplicación parcial
const plus = (a, b) => a + b;

plus(3, 5); // 8
plus(3); // NaN — no hay aplicación parcial
```

### JavaScript — currificada

```javascript
// Currificada: función que devuelve una función
const plusCurrificada = (a) => (b) => a + b;

plusCurrificada(1); // devuelve una función
plusCurrificada(1)(2) // 3
  [(1, 2, 3, 4, 5)].map(plusCurrificada(1)); // [2,3,4,5,6]
```

---

## Currificación

### Definición

**Currificar** una función es transformarla de recibir una tupla de n elementos a recibir n argumentos por separado (uno a la vez).

```haskell
-- Sin currificar (como en C/Pascal): recibe una tupla, no admite aplicación parcial
suma :: (Int, Int) -> Int
suma (a, b) = a + b

suma (4, 7)    -- 8 ✓
suma 4         -- Error ✗ espera una tupla completa
```

```haskell
-- Currificada: recibe los argumentos uno a uno
suma :: Int -> Int -> Int
suma a b = a + b

suma 4 7       -- 11 ✓
suma 4         -- devuelve una función (Int -> Int) ✓
map (suma 1) [1..5]  -- [2,3,4,5,6] ✓
```

|                    | Sin currificar     | Currificada        |
| ------------------ | ------------------ | ------------------ |
| Firma              | `f :: (a, b) -> c` | `f :: a -> b -> c` |
| Aplicación parcial | ✗ No se puede      | ✓ Se puede         |
| Similar a          | C, Pascal          | Haskell nativo     |

> El nombre **currificación** viene en honor al lógico **Haskell Curry** (de quien también toma el nombre el lenguaje), y fue desarrollado ampliamente por el lógico ucraniano **Moses Schönfinkel**.

### Por qué permite la aplicación parcial

En Haskell, `->` **asocia a derecha**:

```haskell
suma :: Int -> Int -> Int
-- es equivalente a:
suma :: Int -> (Int -> Int)
-- "recibe un Int y devuelve una función que recibe otro Int y devuelve un Int"
```

Esto significa que `suma` no recibe dos números: **recibe un número y devuelve una función** que espera el segundo. Por eso `(suma 1)` funciona: es `suma` aplicada parcialmente.

```haskell
(+) :: Num a => a -> (a -> a)

(2 +) :: Num a => a -> a    -- función que suma 2
(2 +) 3                     -- 5
map (2 +) [1..5]            -- [3,4,5,6,7]
```

### Cómo la currificación construye funciones paso a paso

En cálculo lambda la suma se escribe como:

```
λx.(λy. x + y)
```

En Haskell es equivalente a:

```haskell
\x -> (\y -> x + y)

-- Aplicando x = 1:
(\x -> (\y -> x + y)) 1  =  (\y -> 1 + y)  =  siguiente
```

### Ejemplo: `empiezaCon`

```haskell
empiezaCon :: Char -> String -> Bool
empiezaCon letra palabra = ((== letra) . head) palabra

-- En forma lambda currificada:
(\letra -> (\palabra -> ((== letra) . head) palabra))

-- Aplicando solo la letra 'a':
empiezaCon 'a'         -- devuelve una función String -> Bool

-- Aplicando los dos parámetros:
empiezaCon 'r' "rosa"  -- True
```

### `curry` y `uncurry`

```haskell
-- curry: transforma una función de tupla en currificada
suma :: (Int, Int) -> Int
suma (a, b) = a + b

map (curry suma 1) [1..5]   -- [2,3,4,5,6]
-- curry suma toma la función de tupla y la currifica,
-- permitiendo aplicarla parcialmente con 1
```

`uncurry` hace el proceso inverso: transforma una función currificada en una que recibe una tupla.

---

## Resumen

| Concepto           | Descripción                                            | Ejemplo                    |
| ------------------ | ------------------------------------------------------ | -------------------------- |
| Lambda             | Función anónima definida en el lugar de uso            | `\x -> x * 2`              |
| `where`            | Definición local con nombre dentro de una función      | `where disc = b*b - 4*a*c` |
| Currificación      | Función de n args en lugar de una tupla de n elementos | `f :: a -> b -> c`         |
| Sin currificar     | Función que recibe una tupla                           | `g :: (a, b) -> c`         |
| Aplicación parcial | Posible gracias a la currificación                     | `(2 *)`, `filter even`     |
| `curry`            | Transforma función de tupla en currificada             | `curry suma 1`             |
| `uncurry`          | Transforma función currificada en una de tupla         | `uncurry (+) (3, 4)`       |

> **Regla clave:** toda función currificada `a -> b -> c` se asocia a derecha como `a -> (b -> c)`: recibe `a` y devuelve una función `b -> c`. Esto es lo que habilita la aplicación parcial.

## Conceptos clave

`lambda` · `cálculo lambda` · `función anónima` · `where` · `currificación` · `uncurry` · `curry` · `aplicación parcial` · `asociatividad a derecha` · `Alonzo Church` · `Haskell Curry` · `Moses Schönfinkel`

```

```
