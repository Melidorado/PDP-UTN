# Paradigmas de Programación — Introducción

## ¿Qué es un paradigma?

Un conjunto de ideas (herramientas conceptuales) que configura una respuesta a "¿qué es un programa?". Cada paradigma tiene una **abstracción principal** que define criterios de programación radicalmente diferentes. El objetivo de la materia es poder **elegir las herramientas de programación más convenientes** para cada problema, en lugar de usar siempre el mismo enfoque.

---

## Programación imperativa (punto de partida)

- Secuencia de pasos e instrucciones
- Variables como posiciones de memoria (el valor nuevo pisa al anterior)
- Tipos de datos simples o compuestos
- Estructuras de control: `secuencia`, `if/else`, `while`

---

## Declarativo vs. Procedimental

|                    | Procedimental                    | Declarativo                                 |
| ------------------ | -------------------------------- | ------------------------------------------- |
| Orden de ejecución | Hay un orden definido            | La secuencia pierde relevancia              |
| Foco               | Dice _cómo_ resolver el problema | Expresa _características_ del problema      |
| Control            | Mayor control sobre el algoritmo | Menor control, menos cosas a pensar         |
| Verificación       | Más puntos a verificar           | Necesita un mecanismo externo para resolver |

**Ejemplo — filtrar pares:**

```haskell
-- Pseudocódigo: dos índices, riesgo de errores en asignaciones
-- Haskell: foco en el qué, no el cómo
pares = filter even
```

````

---

## Declaratividad ≠ Expresividad

- **Expresividad:** resuelve un problema de forma más clara y menos compleja.
- **Declaratividad:** menor grado de información sobre cómo se resuelve; la secuencia no importa.
- No siempre una solución declarativa es más clara que una no declarativa. Son conceptos independientes.

---

## Gap semántico

Es la distancia entre el lenguaje del programador/usuario y el lenguaje que ejecuta la máquina. A lo largo del tiempo los lenguajes incorporaron abstracciones de mayor nivel para **reducir ese gap**, acercándose más al dominio del problema.

---

## Conceptos clave

`paradigma` · `declarativo` · `procedimental` · `expresividad` · `gap semántico` · `abstracción` · `imperativo`

```

```
````
