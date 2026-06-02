# Módulo 1 — Paradigma Funcional: Introducción

## El paradigma funcional — idea central

Un programa es una **función**: una transformación de un input en un output. El output depende exclusivamente del input. La metáfora es la **calculadora**: entrás un valor, salís con otro.

---

## Unicidad

Para que algo sea una función matemática, **para cada valor del dominio debe haber una sola imagen**. Si para un mismo input obtengo dos resultados distintos, no es una función.

---

## Transparencia referencial

Una solución tiene **transparencia referencial** si podemos reemplazar en cualquier punto del programa toda expresión `E` por su valor `V` sin alterar el resultado, independientemente del contexto o el orden de evaluación.

- ✓ Si `f(1)` siempre da `1`, puedo reemplazar `f(1)` por `1` en cualquier lugar del programa.
- ✗ Si `f(1)` puede dar `1` ó `2` según el estado de una variable global, **no hay transparencia referencial**.

**Ventajas:**

- Más fácil demostrar la corrección del programa (simplifica el testing)
- Podemos calcular la expresión una vez y cachearla (mejor performance)
- No importa en qué momento se evalúa la expresión

---

## Variable: imperativo vs. funcional

|                   | Imperativo                      | Funcional                                  |
| ----------------- | ------------------------------- | ------------------------------------------ |
| Qué es            | Posición de memoria             | Incógnita matemática                       |
| Para qué          | Guarda valores intermedios      | Valor desconocido o no calculado aún       |
| Reasignación      | El valor nuevo pisa al anterior | No tiene sentido (`x = x + 1` es inválido) |
| `x = 2` y `x = 3` | Dos asignaciones sucesivas      | Dos puntos distintos del dominio           |

---

## Tipos básicos en Haskell

| Tipo                | Descripción          | Ejemplo         |
| ------------------- | -------------------- | --------------- |
| `Int`               | Entero sin decimales | `42`            |
| `Float`             | Número con decimales | `3.14`          |
| `Bool`              | Verdadero o falso    | `True`, `False` |
| `Char`              | Carácter individual  | `'a'`           |
| `String` / `[Char]` | Cadena de texto      | `"hola"`        |

---

## Definición de funciones

Toda función tiene dos partes: la **firma de tipo** (opcional pero recomendada) y el **cuerpo**.

```haskell
-- Firma: nombre :: TipoEntrada -> TipoSalida
aproboAlumno :: Int -> Bool

-- Cuerpo: nombre parametro = expresion
aproboAlumno nota = nota >= 6
```

El `=` es una igualdad matemática, no una asignación. La variable `nota` es una incógnita: no tiene valor hasta que se aplica la función.

Más ejemplos:

```haskell
doble :: Int -> Int
doble x = 2 * x

pesosADolares :: Float -> Float
pesosADolares pesos = pesos / 206.50

millasAKilometros :: Float -> Float
millasAKilometros millas = millas * 1.609344
```

---

## Aplicación de funciones

En Haskell las funciones usan **notación prefija**: primero el nombre, luego los parámetros (sin paréntesis ni comas). Al pasar parámetros, la expresión se _reduce_ hasta un valor irreductible.

```haskell
aproboAlumno 8    -- True
aproboAlumno 3    -- False
doble 4           -- 8  (doble 4 => 2 * 4 => 8)
```

---

## Variables asociadas a valores (constantes)

Una variable sin parámetros no es una función, es una **asociación a un valor**. Sirve para nombrar valores y hacer el código más legible.

```haskell
edadSeniorBurns = 120
segundoNombreHomero = "Jay"
```

---

## Chequeo de tipos — inferencia estática

Haskell tiene **chequeo estático de tipos**: el compilador verifica los tipos en tiempo de compilación, no en ejecución. No es obligatorio declarar el tipo — Haskell lo _infiere_ — pero es buena práctica hacerlo.

```haskell
:t aproboAlumno        -- aproboAlumno :: Int -> Bool
:t aproboAlumno 8      -- Bool
:t True                -- Bool
```

El compilador detecta tres tipos de errores:

1. Valor de tipo incorrecto en una expresión (`"pepe" * x`)
2. Firma declarada que no coincide con el cuerpo
3. Parámetro de tipo incorrecto al aplicar (`doble "2"`)

> **Nota:** el tipado estático/dinámico es una característica del _lenguaje_, no del paradigma. Existen lenguajes funcionales con tipado dinámico (Joy, Factor).

---

## Guardas

Las guardas permiten definir una función **por casos** (como una función matemática definida "por trozos"). Se usan cuando el resultado depende de una condición.

```haskell
max x y | x > y     = x
        | otherwise = y
```

- `otherwise` siempre se cumple → debe ir al final
- La indentación importa: la segunda línea debe tener al menos un espacio

### Uso inadecuado de guardas

Si la condición ya es una expresión booleana, **no hace falta usar guardas**:

```haskell
-- MAL: usa guardas innecesariamente
puedoAvanzar color | color == "verde" = True
                   | otherwise        = False

-- BIEN: la expresión booleana es el resultado directamente
puedoAvanzar color = color == "verde"
```

---

## Conceptos clave

`función` · `transparencia referencial` · `unicidad` · `variable como incógnita` · `tipos` · `aplicación` · `guardas` · `inferencia de tipos` · `tipado estático`
