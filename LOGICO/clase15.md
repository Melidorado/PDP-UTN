# Paradigma Lógico — Predicados de Orden Superior y Testing con PlUnit

> Fuentes: Parte 1 → Módulo 4 de la cátedra, "Predicados de orden superior" (Dodino, Leiva, Lombardi, Matos, Passerini, Solmirano) + transcripción de la clase en vivo.
> Parte 2 → apunte "Unit Testing usando PlUnit" (Palumbo, Dodino, Contardo, Rau) + transcripción de la clase en vivo.
>
> Nota: esta clase tuvo dos partes con temas distintos entre sí (orden superior, y luego testing), por eso quedan como dos grandes bloques dentro de este mismo archivo.

## 1. ¿Qué es un predicado de orden superior?

Hasta ahora, todos los predicados que definimos trabajaban con **individuos simples o compuestos** como argumentos:

```prolog
?- humano(socrates).
?- par(2).
?- length([1, 2, 3], 3).
?- nacio(melina, fecha(19, 8, 2001)).
```

Un **predicado de orden superior** es aquel que recibe, entre sus argumentos, **una consulta o predicado** (no solo individuos). En este módulo vemos los tres más importantes: `not/1`, `forall/2` y `findall/3` (este último en la Parte 2).

## 2. El predicado `not/1`

`not/1` verifica que una consulta **no se satisfaga** — es la negación lógica.

### 2.1 Uso básico

Base de conocimiento de apostadores de ruleta:

```prolog
juega(julia, 3).
juega(beto, 6).
juega(dodain, 5).
juega(juana, 15).
juega(sergio, 3).
```

Un número es **yeta** si nadie apuesta por él: `∄x / juega(x)` se cumple.

```prolog
yeta(Numero) :- not(juega(_, Numero)).
```

`not` tiene **aridad 1** — pero el argumento que recibe es en sí mismo una consulta (por eso es de orden superior).

```prolog
?- yeta(3).
false.

?- yeta(25).
true.
```

Otro ejemplo visto en clase, con una base de personas y lenguajes de programación:

```prolog
programa(nahuel, haskell).
programa(juan, ruby).
programa(caro, javascript).
programa(caro, scala).
% (nadie programa en Cobol)
```

```prolog
outsider(Persona) :- not(programa(Persona, javascript)).
```

```prolog
?- outsider(juan).
true.    % juan no programa en javascript

?- outsider(caro).
false.   % caro sí programa en javascript
```

### 2.2 Inversibilidad de `not/1`

¿Qué pasa si hacemos una consulta existencial?

```prolog
?- yeta(Numero).
false.

?- outsider(Persona).
false.   % (o falla directamente, según la versión)
```

**No funciona.** ¿Por qué? Porque Prolog solo sabe encontrar los individuos que **sí** satisfacen una relación — nunca definimos cuál es el universo completo de números de ruleta o de personas, así que no hay forma de enumerar "todo lo que no cumple" la condición negada.

**Solución**: usar un **predicado generador** que acote el universo de la variable _antes_ de negarla.

```prolog
yeta(Numero) :-
    numeroRuleta(Numero),
    not(juega(_, Numero)).

numeroRuleta(Numero) :- between(1, 36, Numero).
```

`between(1, 36, Numero)` relaciona un número con el intervalo cerrado entre `1` y `36`, y es totalmente inversible. Así, `numeroRuleta/1` funciona como generador, y ahora sí:

```prolog
?- yeta(Cual).
Cual = 1 ;
Cual = 2 ;
Cual = 4 ;
...
```

De la misma manera, en el ejemplo de `outsider`:

```prolog
outsider(Persona) :-
    programa(Persona, _),
    not(programa(Persona, javascript)).
```

Acá `programa(Persona, _)` actúa como generador del universo de personas, y recién dentro del `not` la variable `Persona` llega **ligada** — por eso ahora sí podemos preguntar `outsider(Persona)` y obtener todas las respuestas.

### 2.3 Repaso: perros famosos

Si tengo el universo de perros famosos (`perroFamoso/1`), no puedo determinar directamente cuáles son los perros **no** famosos sin conocer el universo total de perros:

```prolog
perroComun(Perro) :- not(perroFamoso(Perro)).
```

Puedo preguntar `perroComun(lassie)` (consulta individual, funciona), pero **no** `perroComun(Perro)` (consulta existencial, falla) — no es inversible.

**Solución 1** — generador:

```prolog
perroComun(Perro) :- perro(Perro), not(perroFamoso(Perro)).
```

**Solución 2** — definir el complemento directamente como hechos (transformar un predicado negativo en uno asertivo):

```prolog
perroComun(bachi).
perroComun(sultan).
perroComun(copito).
```

En ambos casos, la dificultad de fondo es la misma: **no siempre conocemos el conjunto universal**.

### 2.4 `not/1` sigue siendo de aridad 1

Si queremos negar la ocurrencia conjunta de dos condiciones (`¬(p ∧ q)`), hay que **agrupar todo entre paréntesis** para que siga siendo una sola consulta (y por lo tanto respete la aridad 1 de `not`):

```prolog
?- not(10 > 3, juega(dodain, 5)).
ERROR: Undefined procedure: not/2

?- not((10 > 3, juega(dodain, 5))).
false.
```

#### Ejemplo: persona imprescindible

_"Una persona es imprescindible si programa en un lenguaje en el que ninguna otra persona programa."_

```prolog
personaImprescindible(Persona) :-
    programa(Persona, Lenguaje),
    not((programa(OtraPersona, Lenguaje), Persona \= OtraPersona)).
```

Puntos clave de este ejemplo (que en la clase costó ir armando paso a paso):

- `programa(Persona, Lenguaje)`, **afuera** del `not`, funciona como generador: liga tanto `Persona` como `Lenguaje` antes de negar nada.
- Dentro del `not` hay **dos condiciones** (`p ∧ q`): que exista otra persona que programe en ese lenguaje, **y** que sea distinta de la persona ya encontrada — por eso van agrupadas con un paréntesis extra y separadas por coma.
- **Cuidado con el orden y con qué queda adentro/afuera del `not`**: si la comparación `Persona \= OtraPersona` queda **afuera** del `not` (comparando contra una variable todavía libre), la comparación **siempre falla** (no se puede probar que algo sea distinto de una incógnita — de nuevo, Principio de Universo Cerrado), y entonces el `not` que la rodea **siempre se satisface**, dando como resultado que _todas_ las personas parecen "imprescindibles" — un resultado incorrecto y contraintuitivo. La comparación tiene que ir **dentro** del `not`, junto con la consulta que estamos negando.

#### Ejemplo: lenguaje más difícil (y por qué la lógica no garantiza unicidad)

Con una base de conocimiento de tiempos de aprendizaje por lenguaje, se puede definir:

```prolog
lenguajeMasDificil(Lenguaje) :-
    tiempoAprendizaje(Lenguaje, Tiempo),
    not((tiempoAprendizaje(OtroLenguaje, OtroTiempo),
         OtroLenguaje \= Lenguaje,
         OtroTiempo > Tiempo)).
```

Un detalle interesante que se vio en clase: si en la base de conocimiento hay **dos lenguajes empatados** en el mayor tiempo de aprendizaje, esta consulta va a devolver **ambos** como soluciones — algo que en Haskell, pensando en términos de una función, no podría pasar (a menos que devolviéramos explícitamente una lista). Esto refuerza una idea de fondo: **en lógico trabajamos con relaciones, no con funciones**, y una relación puede tener más de una solución simultáneamente válida.

## 3. El predicado `forall/2`

`forall/2` verifica que una condición se cumpla **para todos** los individuos posibles — es la **cuantificación universal** (`∀x / p(x) ⇒ q(x)`).

```prolog
forall(Condicion, Consecuencia).
```

- El **primer argumento** (`p`) genera el universo de individuos a considerar.
- El **segundo argumento** (`q`) es la condición que debe cumplirse para _todos_ esos individuos.

### 3.1 Uso básico

```prolog
materia(algoritmos, 1).
materia(analisisI, 1).
materia(pdp, 2).
materia(proba, 2).
materia(sintaxis, 2).

nota(nicolas, pdp, 10).
nota(nicolas, proba, 7).
nota(nicolas, sintaxis, 8).
nota(malena, pdp, 6).
nota(malena, proba, 2).
nota(raul, pdp, 9).
```

_"Un alumno terminó un año si aprobó todas las materias de ese año."_

```prolog
aprobo(Alumno, Materia) :- nota(Alumno, Materia, Nota), Nota >= 6.

terminoAnio(Alumno, Anio) :-
    forall(materia(Materia, Anio), aprobo(Alumno, Materia)).
```

```prolog
?- terminoAnio(nicolas, 2).
true.

?- terminoAnio(malena, 2).
false.
```

**Error común**: confundir cuál va como universo (`p`) y cuál como condición (`q`) — invertirlos cambia completamente el significado. Por ejemplo, escribir _"todas las materias que aprobó son de ese año"_ en vez de _"aprobó todas las materias de ese año"_ termina pidiendo algo distinto (que el alumno **solo** haya aprobado materias de ese año en particular, no que las haya aprobado todas). Conviene siempre releer con cuidado cuál es el universo y cuál la condición.

### 3.2 `forall/2` e inversibilidad

```prolog
?- terminoAnio(Alumno, 2).
```

Esto **no es inversible tal cual está**: como `Alumno` no está ligado, `forall` termina preguntando si _para toda materia de 2° año, existe algún alumno_ que la haya aprobado — algo completamente distinto a lo que queremos.

**Solución**: agregar un generador para `Alumno` (definido por comprensión a partir de `nota/3`, sin necesidad de enumerar alumnos a mano):

```prolog
alumno(Alumno) :- nota(Alumno, _, _).

terminoAnio(Alumno, Anio) :-
    alumno(Alumno),
    forall(materia(Materia, Anio), aprobo(Alumno, Materia)).
```

```prolog
?- terminoAnio(Alumno, 2).
Alumno = nicolas ;
...
```

(Puede repetir soluciones — no nos preocupamos por eso en esta cursada.)

#### El caso del `Anio` sin ligar: verdad vacía

Si dejamos `Anio` como incógnita sin generador:

```prolog
?- terminoAnio(nicolas, 5).
true.
```

¿Cómo puede dar `true` si no existe un 5° año con materias? Repasemos la tabla de verdad de `p ⇒ q`:

| p   | q   | p ⇒ q |
| --- | --- | ----- |
| V   | V   | V     |
| V   | F   | F     |
| F   | V   | V     |
| F   | F   | **V** |

Cuando `p` (en este caso, `materia(Materia, 5)`) es **falso** —no existe ninguna materia de 5° año—, la implicación se satisface **igual**, sin importar `q`. Es una "verdad vacía" (_vacuous truth_): no hay nada que revisar, así que se da por cumplido.

**Solución**: agregar también un generador para `Anio`:

```prolog
terminoAnio(Alumno, Anio) :-
    alumno(Alumno),
    anio(Anio),
    forall(materia(Materia, Anio), aprobo(Alumno, Materia)).
```

```prolog
?- terminoAnio(nicolas, 5).
false.
```

Ahora falla directamente porque `anio(5)` no se satisface — ni siquiera se llega a evaluar el `forall`.

> **Importante**: dentro del `forall`, la variable `Materia` **sí debe quedar libre** (es la que cuantifica universalmente) — no hay que generarla también. `Alumno` y `Anio`, en cambio, deben llegar **ya ligados** antes del `forall`, porque son "parámetros" del requerimiento, no parte de lo que se cuantifica.

### 3.3 Cuidado con "pasarse de rosca" generando de más

Un detalle importante que se remarcó en clase: si además generamos `Materia` **antes** del `forall` (por ejemplo `materia(Materia, Anio), forall(materia(Materia, Anio), aprobo(...))`), el `forall` se queda **sin incógnitas libres** — el universo pasa a ser una sola materia concreta, no todas las materias del año. Esto equivale, en la práctica, a **no tener `forall`** en absoluto (sería lo mismo que preguntar si el alumno aprobó _alguna_ materia de ese año, no _todas_). Hay que dejar **exactamente** la variable que corresponde cuantificar universalmente libre dentro del `forall`, ni una más.

### 3.4 Segundo ejemplo: autos y personas

```prolog
vieneCon(p206, abs).
vieneCon(p206, levantavidrios).
vieneCon(p206, direccionAsistida).
vieneCon(kadisco, abs).
vieneCon(kadisco, mp3).
vieneCon(kadisco, tacometro).

quiere(carlos, abs).
quiere(carlos, mp3).
quiere(roque, abs).
quiere(roque, direccionAsistida).
```

_"Un auto le viene perfecto a una persona si tiene todas las características que esa persona quiere."_

```prolog
auto(p206).
auto(kadisco).
persona(carlos).
persona(roque).

leVienePerfecto(Auto, Persona) :-
    auto(Auto),
    persona(Persona),
    forall(quiere(Persona, Caracteristica), vieneCon(Auto, Caracteristica)).
```

Acá **ambos** argumentos (`Auto` y `Persona`) necesitan su propio generador, porque el `forall` requiere que los dos lleguen ligados.

### 3.5 Ejercicios combinados: lógica de conjuntos

**Incluido**: un conjunto `A` está incluido en `B` si todos los elementos de `A` están en `B`.

```prolog
incluido(A, B) :- forall(member(X, A), member(X, B)).
```

**Disjuntos**: `A` es disjunto de `B` si ningún elemento de `A` está en `B`.

```prolog
disjuntos(A, B) :- forall(member(X, A), not(member(X, B))).
```

## 4. Del `not` al `forall`: equivalencias lógicas

Estas dos afirmaciones son equivalentes:

```
∀x / p(x) ⇒ q(x)      ≡      ∄x / p(x) ∧ ¬q(x)
```

("para todo x que cumple p, cumple q" es lo mismo que "no existe ningún x que cumpla p y no cumpla q")

Ejemplo:

```prolog
materia(am1). materia(sysop). materia(pdp).
alumno(clara). alumno(matias). alumno(valeria). alumno(adelmar).
estudio(clara, am1). estudio(clara, sysop). estudio(clara, pdp).
estudio(matias, pdp). estudio(matias, am1).
estudio(valeria, pdp).
```

_"Un alumno estudioso es aquel que estudia para todas las materias"_, con `forall`:

```prolog
estudioso(Alumno) :-
    alumno(Alumno),
    forall(materia(Materia), estudio(Alumno, Materia)).
```

La misma idea con `not` anidado (equivalente, pero mucho menos legible):

```prolog
estudiosoNot(Alumno) :-
    alumno(Alumno),
    not((materia(Materia), not(estudio(Alumno, Materia)))).
```

Ambas definiciones son **lógicamente equivalentes**, pero la segunda es más difícil de leer (tiene un `not` anidado dentro de otro, y hay que prestar mucha atención a qué alcanza cada paréntesis). **Cuando el requerimiento se presta a una afirmación positiva y directa, conviene usar `forall`.**

En cambio, cuando el requerimiento ya es naturalmente una negación, `not` puede ser más directo:

```prolog
dificil(Alumno) :- alumno(Alumno), not(estudio(Alumno, _)).
```

("un alumno difícil es el que no estudia para ninguna materia")

### 4.1 Comparando `forall` vs. `not` en la práctica

En clase se compararon ambas variantes con varios ejemplos, para ir desarrollando criterio sobre cuál conviene en cada caso:

**Menor elemento de una lista** — ambas variantes quedan igual de legibles:

```prolog
% con forall
menor(Lista, Menor) :-
    member(Menor, Lista),
    forall(member(Elemento, Lista), Menor =< Elemento).

% con not (dando vuelta la comparación)
menorNot(Lista, Menor) :-
    member(Menor, Lista),
    not((member(Elemento, Lista), Elemento < Menor)).
```

**Lenguaje "shipiado"** (todos los que lo programan, les gusta) — acá `forall` gana en claridad:

```prolog
% con forall (más directo)
lenguajeShipiado(Lenguaje) :-
    lenguaje(Lenguaje),
    forall(programa(Persona, Lenguaje), gusta(Persona, Lenguaje)).

% con not (funciona, pero es menos directo de leer)
lenguajeShipiadoNot(Lenguaje) :-
    lenguaje(Lenguaje),
    not((programa(Persona, Lenguaje), not(gusta(Persona, Lenguaje)))).
```

**Lenguaje "tolerado"** (a nadie que lo programa le gusta) — acá la negación directa con `not` es más natural que negar el consecuente dentro de un `forall`:

```prolog
% con forall negando la condición
lenguajeTolerado(Lenguaje) :-
    lenguaje(Lenguaje),
    forall(programa(Persona, Lenguaje), not(gusta(Persona, Lenguaje))).

% con not (queda igual de simple, y usa un solo predicado de orden superior)
lenguajeToleradoNot(Lenguaje) :-
    lenguaje(Lenguaje),
    not((programa(Persona, Lenguaje), gusta(Persona, Lenguaje))).
```

**Lenguaje "ecléctico"** (no _todos_ los que lo programan les gusta) — acá `forall` negado es mucho más legible que su equivalente con `not` anidado:

```prolog
% con forall (clara)
lenguajeEcleptico(Lenguaje) :-
    lenguaje(Lenguaje),
    not(forall(programa(Persona, Lenguaje), gusta(Persona, Lenguaje))).

% con not anidado (funciona, pero se vuelve difícil de leer y mantener)
lenguajeEclepticoNot(Lenguaje) :-
    lenguaje(Lenguaje),
    not(not((programa(Persona, Lenguaje), not(gusta(Persona, Lenguaje))))).
```

**Conclusión de la clase**: ambos predicados (`not` y `forall`) permiten resolver básicamente los mismos requerimientos — son intercambiables en poder expresivo —, pero conviene elegir en cada caso la variante que sea **más simple de leer, explicar y mantener**. Como regla general, un requerimiento que se piensa naturalmente como _"para todos... se cumple"_ suele quedar más claro con `forall`; uno que se piensa como una negación directa (_"no existe ninguno que..."_) suele quedar más claro con `not`. Anidar varios `not` casi siempre empeora la legibilidad.

## 5. Resumen de la Parte 1

Los predicados de orden superior como `not/1` y `forall/2` trabajan con **consultas como argumentos**, permitiendo subir el nivel de abstracción respecto a los predicados de primer orden. `forall/2` expresa la cuantificación universal ("para todos se cumple"), mientras que `not/1` expresa la cuantificación negativa individual ("no existe x tal que cumpla"); son intercambiables en muchos casos, aunque suele ser más cómodo trabajar con afirmaciones asertivas que negando predicados. En ambos casos, **para lograr inversibilidad hace falta ligar previamente las variables involucradas mediante predicados generadores** — porque Prolog no puede determinar los individuos que _no_ satisfacen algo sin conocer primero el universo completo (de nuevo, el Principio de Universo Cerrado en acción).

---

## 📌 Glosario rápido para repasar

| Término                            | Definición                                                                                                                                                             |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Predicado de orden superior**    | Predicado que recibe una consulta (u otro predicado) como argumento, no solo individuos.                                                                               |
| **`not/1`**                        | Verifica que una consulta no se satisfaga (negación lógica). Aridad fija = 1: para negar varias condiciones a la vez hay que agruparlas entre paréntesis.              |
| **`forall/2`**                     | Verifica que una condición (2do argumento) se cumpla para todos los individuos que satisfacen una consulta (1er argumento). Cuantificación universal.                  |
| **Verdad vacía (_vacuous truth_)** | Cuando el antecedente de un `forall` no tiene ningún individuo que lo satisfaga, la implicación se da por cumplida automáticamente (p ⇒ q es verdadero si p es falso). |
| **Predicado generador**            | Un hecho u otro predicado inversible que se usa para ligar de antemano una variable, acotando su universo posible antes de usarla dentro de un `not` o un `forall`.    |
| **"Pasarse de rosca" generando**   | Error de generar también la variable que debería quedar libre dentro de un `forall`, lo que anula su cuantificación universal.                                         |
| **Equivalencia `forall`/`not`**    | `∀x (p(x) ⇒ q(x))` es lógicamente equivalente a `∄x (p(x) ∧ ¬q(x))`.                                                                                                   |

---

---

# Parte 2: Testing Unitario con PlUnit

## 1. ¿Qué es y para qué sirve?

Testear en Prolog significa tomar parte de nuestras definiciones y someterlas a pruebas para verificar que cumplen con lo que tienen que cumplir. SWI Prolog trae incluido un framework de testing llamado **PlUnit**.

Ejemplo base para toda esta sección:

```prolog
sueldo(electra, 80000).
sueldo(loki, 120000).
sueldo(thor, 60000).
sueldo(odin, 50000).

horasTrabajo(electra, 6).
horasTrabajo(loki, 8).
horasTrabajo(thor, 5).
horasTrabajo(odin, 8).

ganaBuenSueldo(Persona) :- sueldo(Persona, Sueldo), Sueldo > 75000.
tieneTiempoLibre(Persona) :- horasTrabajo(Persona, Horas), Horas < 7.

tieneBuenTrabajo(Persona) :- ganaBuenSueldo(Persona).
tieneBuenTrabajo(Persona) :- tieneTiempoLibre(Persona).
```

## 2. Test individual: `test/1`

Para definir un test alcanza con escribir una regla para el predicado `test/1`, que recibe como parámetro el nombre del test como átomo (o string, si necesitamos espacios):

```prolog
:- begin_tests(buenTrabajo).

test("persona que gana un buen sueldo") :-
    ganaBuenSueldo(electra).

:- end_tests(buenTrabajo).
```

- `begin_tests/1` y `end_tests/1` agrupan y nombran el conjunto de tests (conviene escribirlos en un archivo aparte, separando la implementación de las pruebas).
- Se corren todos con `run_tests/0` (aridad 0).
- Un test verifica que una **consulta existencial se satisfaga** para algún individuo concreto.

### 2.1 Estrategia bottom-up vs. top-down

Si un predicado depende de otros más simples (como `tieneBuenTrabajo` depende de `ganaBuenSueldo` y `tieneTiempoLibre`), conviene elegir una estrategia:

- **Bottom-up**: testear primero los predicados más simples (`ganaBuenSueldo`, `tieneTiempoLibre`) y recién después el que los combina.
- **Top-down**: empezar directamente probando el predicado más general.

Ambas tienen ventajas y desventajas; en los ejemplos de clase se usó bottom-up para mantener las cosas simples al principio.

### 2.2 Pensar en clases de equivalencia, no en el individuo puntual

Al elegir qué probar, conviene identificar la **clase de equivalencia** involucrada (por ejemplo: _personas que ganan un buen sueldo_ vs. _personas que no_), y elegir **cualquier representante** de esa clase — no centrar el nombre ni el foco del test en ese individuo puntual, porque lo que se está probando es la regla en general, no ese caso particular. Por eso se prefiere nombrar el test como `"persona que gana un buen sueldo"` y no `"electra_gana_un_buen_sueldo"`: cualquier persona de esa clase serviría igual.

## 3. Predicados no determinísticos y _choicepoints_

Al correr los tests puede aparecer esta advertencia:

```
Warning: ... Test succeeded with choicepoint
```

Esto pasa cuando el predicado testeado es **no determinístico**: hay más de un camino posible para llegar a una solución (por ejemplo, `tieneBuenTrabajo/1` se satisface tanto por `ganaBuenSueldo` como por `tieneTiempoLibre` — es un OR, con dos ramas posibles). PLUnit avisa: _"por este camino se cumple, pero podría haber otros caminos donde no"_.

Otro ejemplo, con una regla que además de hechos también tiene una cláusula recursiva/transitiva:

```prolog
cursa(diego, pdp, dodain).
cursa(mary, pdp, gaston).
cursa(elias, pdp, juan).
cursa(celeste, pdp, juan).
cursa(mora, pdp, alf).

cursa(Alguien, pdp, dodain) :- cursa(Alguien, pdp, juan).
cursa(Alguien, operativos, adriano) :- cursa(Alguien, pdp, gaston).
```

`cursa(mary, operativos, adriano)` se satisface (porque mary cursó `pdp` con `gaston`), pero como hay ramas alternativas del predicado que no aplican para este caso, Prolog también avisa con un choicepoint.

**Si sabemos de antemano que el predicado es no determinístico**, usamos `test/2` con la opción `nondet`, para indicarle a PLUnit que no nos importa esa advertencia:

```prolog
test("persona que curso con profe por transitividad", nondet) :-
    cursa(mary, operativos, adriano).
```

## 4. Test individual: negación (`fail`)

Si queremos testear que una consulta **no** se cumple, la forma recomendada es la opción `fail` (con `test/2`):

```prolog
test("persona que no gana un buen sueldo", fail) :-
    ganaBuenSueldo(odin).
```

**Cuidado** si se quiere probar que **varias** condiciones no se cumplen a la vez (`p ∧ q`) usando `fail`: eso en realidad testea `¬(p ∧ q)` — es decir, alcanza con que **una sola** de las dos falle para que el test pase, aunque la otra se cumpla. Si lo que se necesita es que **cada una** de varias condiciones falle por separado, conviene armar la consulta explícitamente con `not/1` en vez de apoyarse en `fail`, para no arriesgarse a un falso positivo.

```prolog
% ojo: esto pasa si SOLO UNA de las dos falla, no necesariamente ambas
test("thor y odin no ganan buen sueldo", fail) :-
    ganaBuenSueldo(thor), ganaBuenSueldo(odin).

% más seguro si necesitamos que cada condición falle individualmente
test("thor y odin no ganan buen sueldo") :-
    not(ganaBuenSueldo(thor)), not(ganaBuenSueldo(odin)).
```

## 5. Test de inversibilidad (`set`)

Cuando queremos testear que un predicado es **inversible**, la opción recomendada es `set`, indicando el conjunto completo de individuos que debería satisfacer la consulta:

```prolog
test("personas que ganan un buen sueldo", set(Persona == [electra, loki])) :-
    ganaBuenSueldo(Persona).
```

- Solo se puede usar con **una** variable.
- Al ser un **conjunto**, no importa el orden (`[electra, loki]` es lo mismo que `[loki, electra]`) ni los duplicados.
- Es un test **exhaustivo**: recorre _toda_ la clase de equivalencia. Esto es específico del paradigma lógico — no se acostumbra en otros paradigmas —, porque acá la base de conocimiento _es_ el universo completo y conocido.
- Contracara: son tests **frágiles** — cualquier agregado a la base de conocimiento que sume una persona más a esa clase de equivalencia va a romper el test, aunque el predicado siga estando bien definido. Se considera un costo aceptable en esta cursada porque efectivamente modelamos todo el universo.

### 5.1 Opciones desaconsejadas: `all` y `true`

- **`all(Persona == [...])`**: trabaja con una **lista**, no con un conjunto — por lo tanto **sí importa el orden**, y un cambio en el orden de las soluciones (aunque sean las mismas personas) puede romper el test sin que haya ningún error real.
- **`true(Persona == individuo)`**: unifica un único individuo posible. Tiene varios problemas:
  - el predicado testeado debe ser **determinístico** (una única solución, sin choicepoints);
  - debe respetarse el **orden exacto** en que aparece ese individuo entre las soluciones;
  - solo se puede testear **un** valor del universo, no todos.

**Por eso la recomendación general es siempre usar `set` para testear inversibilidad**, evitando `all` y `true`.

## 6. Consejos para definir los casos de prueba

- **Enfocar el nombre del test en la clase de equivalencia**, no en el individuo concreto: `"personas_con_buena_cursada"` es mejor que `"mora_tuvo_buena_cursada_con_adriano"`.
- **No tiene sentido testear hechos**: por definición ya son ciertos (están en la base de conocimiento).
- **Tampoco tiene sentido testear lo que no existe**: por Principio de Universo Cerrado, ya sabemos que va a ser falso.
- Cuando el predicado es **no determinístico**, pensar en "casos borde" individuales puede ser confuso (puede satisfacerse por un camino y fallar por otro). En esos casos suele convenir ir directo a un test de **inversibilidad con `set`**, aceptando que va a ser un test más frágil.
- Los tests **individuales** (con o sin `fail`) dan la ventaja de identificar rápido **qué clase de equivalencia específica falló**. Los tests de **inversibilidad (`set`)** dan cobertura exhaustiva de todo el universo, pero si fallan hay que investigar un poco más para saber qué elemento puntual está mal. La elección entre uno u otro estilo, cuando ambos aplican, es en buena medida una cuestión de preferencia.

---

## 📌 Glosario — Parte 2 (Testing con PlUnit)

| Término                             | Definición                                                                                                                                                                     |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **PlUnit**                          | Framework de testing unitario incluido en SWI Prolog.                                                                                                                          |
| **`begin_tests/1` / `end_tests/1`** | Directivas que agrupan y nombran un conjunto de tests.                                                                                                                         |
| **`run_tests/0`**                   | Corre todos los tests definidos.                                                                                                                                               |
| **`test/1`**                        | Define un test simple: se espera que la consulta del cuerpo se satisfaga (`true`).                                                                                             |
| **`test/2`**                        | Define un test con una opción adicional: `fail` (se espera que falle), `nondet` (se acepta que sea no determinístico), o `set(Var == [...])` (test de inversibilidad).         |
| **Choicepoint**                     | Punto de la ejecución donde existen caminos alternativos para seguir buscando soluciones — indica que el predicado es no determinístico.                                       |
| **Predicado no determinístico**     | Predicado que puede satisfacerse por más de un camino distinto (típicamente por una disyunción/OR).                                                                            |
| **Clase de equivalencia**           | Conjunto de individuos que, a los fines del test, se comportan igual frente al predicado que se está probando. Un test debería poder usar cualquier representante de la clase. |
| **Test de inversibilidad (`set`)**  | Test exhaustivo que verifica el conjunto completo de individuos que satisfacen una consulta existencial. No importa orden ni duplicados.                                       |
| **`all`**                           | Variante desaconsejada de `set`: usa lista en vez de conjunto, por lo que el orden de las soluciones sí importa.                                                               |
| **`true`**                          | Variante desaconsejada: solo testea un único valor, requiere que el predicado sea determinístico y respeta el orden de aparición.                                              |
