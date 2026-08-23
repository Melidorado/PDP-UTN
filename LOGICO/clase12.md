# Paradigma Lógico — Clase 1: Predicados, Variables, Consultas, Reglas y Universo Cerrado

> Fuentes: Módulo 1 y Módulo 2 de la cátedra (Dodino, Lombardi, Passerini, Solmirano — rev. Freyre) + transcripción de la clase teórico-práctica en vivo (Partes 1 y 2).

## 1. Introducción al paradigma

Un paradigma nos da un **marco conceptual** para resolver problemas: define cómo modelamos datos y algoritmos. Hasta ahora vimos dos visiones bien distintas:

- **Imperativo / procedimental** (arquitectura Von Neumann): defino una secuencia de pasos, y voy guardando estados intermedios en posiciones de memoria (variables mutables). Hay una separación marcada entre _datos_ y _algoritmo/estado_, y los valores se pueden pisar (mutar) en cada paso.
- **Funcional** (McCarthy, LISP): modela funciones como expresiones, como abstracciones de primer nivel. El estado existe pero es **inmutable**. Comparte con el imperativo la separación entre datos y procesos, pero cambia radicalmente qué significa "variable" y "estado".

El **paradigma lógico** propone una tercera forma, todavía más declarativa: **no hay algoritmo explícito**, solo _definiciones_ (el "qué"). Hay un motor de inferencia —una "magia" que, si es transparente, nos permite concentrarnos en lo esencial del problema— que se encarga de resolver las consultas. Ese es el beneficio de la abstracción: declaro conocimiento y delego en el motor la construcción del algoritmo.

## 2. Contexto histórico

- **Alain Colmerauer** (matemático francés) creó, junto con **Philippe Roussel**, el primer lenguaje de programación lógica: **Prolog**, en la Universidad de Marsella (1972).
- A partir de 1975, en la Universidad de Edimburgo, **Robert Kowalski** fue quien le dio formato a las cláusulas y a los predicados, sentando las bases formales del paradigma. Es un caso curioso: primero apareció el lenguaje (Prolog) y recién después se formalizó el paradigma que lo sustenta.
- **Verónica Dahl**, investigadora argentino-canadiense, es una referente reconocida en programación lógica y procesamiento del lenguaje natural.

## 3. Base de conocimiento: hechos, predicados e individuos

Todo programa lógico arranca definiendo una **base de conocimiento** en un archivo `.pl`, que es lo que delimita nuestro **universo de cosas conocidas**.

```prolog
% pastas.pl
pastas(ravioles).
pastas(fideos).
pastas(sorrentinos).
pastas(fetuchinis).
```

- **Predicado**: `pastas` es el nombre de la relación. Como en este caso participa un solo individuo, decimos que tiene **aridad 1** y se escribe `pastas/1`.
- **Predicados monádicos**: aridad 1. Expresan una **característica o atributo** de un individuo (ej: `pastas(ravioles)` dice que ravioles _es_ una pasta).
- **Individuos**: los elementos que participan de las relaciones (`ravioles`, `fideos`). Es natural pensarlos como el equivalente lógico a un _dato_ en otros paradigmas. También se los llama **átomos**.
- **Símbolos vs. strings**: los individuos se pueden escribir como símbolos (minúscula, sin comillas, ej. `ravioles`) o como strings (entre comillas dobles, ej. `"ravioles"`). La diferencia es de implementación; en clase se usaron strings para algunos ejemplos, pero los símbolos son más frecuentes dentro del paradigma. Lo importante es **ser consistente**: si un predicado se definió con individuos en cierto orden y tipo, hay que respetarlo siempre.

## 4. Consultas: individuales y existenciales

Ejecutar un programa lógico abre una consola donde se hacen **consultas** (_queries_) contra la base de conocimiento. El motor de inferencia recorre la base y determina si la relación se verifica.

### 4.1 Consulta individual

Todos los argumentos son individuos concretos (están **instanciados**). Responde `true`/`false`.

```prolog
?- pastas(ravioles).
true.

?- pastas(bohio).
false.
```

### 4.2 Consulta existencial

Al menos un argumento es una **variable** (está **libre**), es decir, una incógnita que arranca con **mayúscula**. Se puede leer como _"¿cuáles son los individuos que satisfacen esta relación?"_.

```prolog
?- pastas(Pasta).
Pasta = ravioles ;
Pasta = fideos ;
Pasta = sorrentinos ;
Pasta = fetuchinis ;
false.
```

- El motor de inferencia va **unificando** la variable con un individuo a la vez, y se detiene mostrando el primero.
- Para pedirle más soluciones, se presiona `;` (o `n`, `r`, `<espacio>`, `<tab>` — todas hacen lo mismo). Para dejar de buscar, `.` o `<Enter>`.
- Al agotar las soluciones posibles, devuelve `false` (no se puede probar que exista otro individuo más).
- Este mecanismo de ir buscando alternativas y "marcando por dónde se pasó" para poder seguir buscando se llama **backtracking**. Es análogo a resolver un laberinto: se deja una marca en la última bifurcación explorada y, si hace falta, se retoma desde ahí para encontrar otras salidas (soluciones) posibles.

### 4.3 Variables anónimas

Cuando no importa **qué** individuo satisface la relación sino solo si **existe alguno**, se usa el guión bajo `_`, llamado **variable anónima**:

```prolog
?- pastas(_).
true.
```

### 4.4 Instanciado vs. libre

En una consulta, cada argumento puede estar:

- **instanciado**: es un individuo concreto (ej. `ravioles`).
- **libre**: es una variable (ej. `Pasta`), que el motor intentará unificar.

Ejemplo con un predicado poliádico:

```prolog
come(juan, ravioles).
come(melina, ravioles).
come(brenda, fideos).
come(juan, fideos).
```

```prolog
?- come(Persona, ravioles).
Persona = juan ;
Persona = melina ;
false.

?- come(juan, Comida).
Comida = ravioles ;
Comida = fideos ;
false.
```

## 5. Inversibilidad

Decimos que un predicado es **inversible** cuando admite consultas dejando **variables libres** en cualquiera de sus argumentos (no solo en uno en particular). Mientras trabajamos únicamente con **hechos**, no hay restricciones: `pastas/1` y `come/2` son **totalmente inversibles**.

Ejemplo con un predicado poliádico de aridad 2, `plato(Pasta, Salsa)`, que relaciona un tipo de pasta con su salsa/acompañamiento:

```prolog
?- plato(ravioles, fileto).      % consulta individual: ¿existe ese plato exacto?
true.

?- plato(ravioles, Salsa).       % ¿con qué salsas se puede acompañar ravioles?
Salsa = fileto ;
...

?- plato(Pasta, fileto).         % ¿qué pastas se acompañan con fileto?
Pasta = ravioles ;
Pasta = fetuchinis ;
...

?- plato(Pasta, Salsa).          % todas las combinaciones posibles
Pasta = ravioles, Salsa = fileto ;
...
```

Poder consultar el mismo predicado dejando libre **cualquiera** de sus argumentos (o varios a la vez) es la **inversibilidad**. Acá está una de las grandes ventajas comparativas del paradigma lógico frente al funcional: en Haskell necesitaríamos **una función distinta para cada una de estas consultas**; en Prolog alcanza con una sola relación.

> Más adelante vamos a ver que esto no siempre se cumple para todos los argumentos: hay predicados (sobre todo definidos con reglas) que son inversibles solo para algunos de sus argumentos.

## 6. Principio de Universo Cerrado

Si consultamos algo que no está en la base:

```prolog
?- pastas(pechitoCerdo).
false.

?- pastas(ñoquis).
false.
```

Hay dos formas de interpretar esto:

1. **Principio de Universo Abierto**: existen tres estados posibles — verdadero, falso y **desconocido**. No probar algo no significa que sea falso, sino que no se sabe.
2. **Principio de Universo Cerrado**: todo lo que no se puede probar se considera directamente **falso**. **Prolog (y la mayoría de los lenguajes lógicos) trabaja así.**

Esto tiene sentido porque, al escribir la base de conocimiento, estamos definiendo explícitamente nuestro alcance/universo. Por eso no tiene sentido escribir hechos negativos ("el asado no es una pasta"): simplemente no se escriben, y quedan fuera del universo (=false). Es más fácil de implementar algoritmos bajo esta premisa que bajo el principio de universo abierto (que requeriría manejar un tercer estado "desconocido").

- **Hecho**: una afirmación codificada en la base de conocimiento, asumida como verdadera por el solo hecho de estar ahí (es **axiomático**). Cada hecho es una **cláusula**.

## 7. Predicados poliádicos

Los predicados con **aridad mayor a 1** se llaman **poliádicos**, y expresan **relaciones entre individuos** (a diferencia de los monádicos, que expresan una característica de un único individuo).

Reglas importantes:

- Las consultas deben respetar la **cantidad de argumentos** (aridad) con la que se definió el predicado. Si no, da error (`Undefined procedure: come/3` si el predicado se definió con aridad 2).
- El **orden de los argumentos importa**: `come(brenda, fideos)` no es lo mismo que `come(fideos, brenda)`. El sentido de cada posición lo definimos nosotros al modelar.
- **Tip de estilo**: conviene escribir todas las cláusulas de un mismo predicado de forma contigua (sin intercalar otros predicados en el medio), porque si no Prolog tira un warning de "clauses not together" (se puede silenciar con `:- discontiguous`, pero mejor evitarlo siendo ordenados).

## 8. Hechos y reglas

Hasta acá solo escribimos **hechos**: predicados sin condiciones, ciertos por el solo hecho de estar en la base de conocimiento (son **axiomas**). Prolog también permite definir **reglas**: predicados que se satisfacen si se cumplen ciertas condiciones.

### El silogismo de Sócrates

> Todo humano es mortal. (cuantificación universal)
> Sócrates es humano. (cuantificación particular)
> Ergo, Sócrates es mortal. (cuantificación particular)

En lógica de primer orden:

```
∀(x) (humano(x) => mortal(x))
humano(socrates)
∴ mortal(socrates)
```

Esto es el clásico **modus ponens**: dado _p_ y _p ⇒ q_, se concluye _q_.

### Cláusula de Horn

En lógica escribimos la implicación como _p ⇒ q_ ("si p entonces q"). En Prolog se escribe **al revés**, consecuente primero:

```prolog
q :- p.
% consecuente :- antecedente(s).
```

Esta forma (consecuente `:-` antecedentes) se llama **cláusula de Horn**. Aplicada al ejemplo:

```prolog
mortal(Persona) :- humano(Persona).
humano(socrates).
```

- Una regla tiene **uno o más antecedentes** (acá, `humano/1`) y **un consecuente** (`mortal/1`).
- Si se cumplen todos los antecedentes, se satisface el consecuente.
- Es clave que la **variable sea la misma** en el antecedente y el consecuente (`Persona` en ambos lados) — así es como Prolog conecta/relaciona los mismos individuos.
- **No tiene sentido escribir `mortal(socrates)` como hecho**: esa información ya se puede _inferir_ a partir de la base de conocimiento. Ahí empieza a verse una de las grandes ventajas del paradigma: evitar redundancia, dejando que el motor derive lo que ya se puede deducir.

### Consultas sobre una regla

```prolog
?- mortal(socrates).
true.

?- mortal(deadpool).
false.
```

`mortal(deadpool)` da `false` porque para satisfacerse necesita que se cumpla `humano(deadpool)`, y como no está en la base de conocimiento, por **Principio de Universo Cerrado** se asume falso.

```prolog
?- mortal(Persona).
Persona = socrates.

?- mortal(_).
true.
```

`mortal/1` **es inversible**, porque admite tanto consultas por individuo concreto como por variable.

Un detalle conceptual importante: mientras los **hechos** definen un conjunto **por extensión** (enumerando), las **reglas** definen un conjunto **por comprensión** (mediante una condición) — este concepto se retoma formalmente en la sección 12.

## 9. Declaratividad

Cuando trabajamos de forma declarativa:

- el concepto de **secuencia** pierde importancia,
- delegamos el algoritmo a un componente externo — en Prolog, el **motor de inferencia**.

No hay estructuras de control (`if`, `while`, etc.) en la base de conocimiento, solo hechos y reglas. Cómo funciona exactamente el motor de inferencia queda fuera del alcance de la materia (para el lector curioso, se puede investigar el **Principio de Resolución de Robinson**); como usuarios, nos alcanza con saber que agregar más hechos o reglas puede hacer que aparezcan más soluciones a una consulta ya existente, sin tener que tocar nada más.

## 10. Unificación

Cuando hacemos `mortal(Persona)` y obtenemos `Persona = socrates`, **no hay una asignación** (como en imperativo). Lo que ocurre es **unificación**: la variable (incógnita) se resuelve con uno o más valores que satisfacen la relación.

Punto clave para no confundirse: **las consultas no "devuelven" nada**. Un predicado no es una expresión que produce un valor (como en funcional); solo se puede satisfacer o no, y a lo sumo permite encontrar los individuos que lo satisfacen. Por eso, en lógico conviene evitar frases como "la función devuelve X" — no hay funciones que retornan, hay relaciones que se verifican.

Consecuencia interesante: como no existe el concepto de asignación, algo como

```prolog
X = X + 1.
```

no es una asignación (no "incrementa" X), sino una **condición** — y en particular, una que **nunca se puede cumplir**, porque es contradictoria como igualdad lógica (si X es 2, no puede ser 3 al mismo tiempo).

## 11. Reglas compuestas

### 11.1 Conjunciones (AND)

Base de conocimiento:

```prolog
viveEn(tefi, lanus).
viveEn(gise, lanus).
viveEn(alf, lanus).
viveEn(dodain, liniers).

docente(alf).
docente(tefi).
docente(gise).
docente(dodain).
```

Requerimiento: _"Cualquier docente que vive en Lanús es un afortunado"_ → `p ∧ q ⇒ r`.

```prolog
afortunado(Persona) :- docente(Persona), viveEn(Persona, lanus).
```

- El conector lógico **AND** se representa con una **coma** en Prolog.
- `Persona` se unifica primero con cada individuo que satisface `docente/1`; para cada uno de esos valores, se intenta satisfacer `viveEn(Persona, lanus)` (ya no hay incógnita en ese punto, `Persona` es un valor concreto).
- Ejemplo: `docente(alf)` se cumple y `viveEn(alf, lanus)` también → `afortunado(alf)` es verdadero. En cambio `docente(dodain)` se cumple pero `viveEn(dodain, lanus)` no (vive en Liniers) → `afortunado(dodain)` falla.
- `afortunado/1` también es inversible.

### 11.2 Disyunciones (OR)

Requerimiento: _"Si una persona es docente **o** vive en Lanús, es afortunada"_ → `p ∨ q ⇒ r`.

En Prolog, el OR se traduce como **dos reglas separadas** para el mismo predicado (no hay un operador de "or" dentro de una sola cláusula para este caso):

```prolog
afortunado(Persona) :- docente(Persona).
afortunado(Persona) :- viveEn(Persona, lanus).
```

- Alcanza con que se cumpla **cualquiera** de las dos ramas.
- Con OR pueden aparecer **soluciones repetidas**: por ejemplo, `alf` puede satisfacer `afortunado/1` tanto por ser docente como por vivir en Lanús, así que al pedir todas las soluciones existenciales puede aparecer más de una vez. Esto es normal y esperable; si molesta, más adelante se ve un predicado (`distinct/2`) para evitar duplicados.
- **Acoplamiento**: `afortunado/1` depende de las definiciones de `docente/1` y `viveEn/2` — cualquier cambio en esas dos puede impactar en `afortunado/1`.

## 12. Definición por extensión y por comprensión

- **Por extensión**: un conjunto de **hechos** para el mismo predicado. Hay que enumerar uno por uno todos los individuos.

```prolog
animal(tigre).
animal(oso).
animal(elefante).
```

Equivale a _Animales = { tigre, oso, elefante }_. Desventaja: hay que enumerar todo (incómodo o inviable para conjuntos grandes/infinitos), y no aporta más información que la que el observador puso ahí.

- **Por comprensión**: una **regla** que define el conjunto mediante una condición, sin enumerar. Ejemplo matemático:

```
A = { a ∈ ℕ / 2 < a < 6 }
B = { p ∈ ℕ / p < 10, siendo p par }
```

En Prolog:

```prolog
a(X) :- X > 2, X < 6.
b(X) :- X < 10, par(X).
```

Un ejemplo ya conocido: `mortal(Persona) :- humano(Persona).` define el conjunto de los mortales **por comprensión** (todos los individuos que sean humanos), en contraste con haberlos enumerado directamente (`mortal(socrates). mortal(platon).`), que sería por extensión.

## 13. Comparación con el paradigma funcional: el caso Leo DiCaprio 🎬

Antes de programar en lógico, en clase se resolvió el mismo problema en **funcional**, para poder comparar bien las diferencias entre paradigmas. Los requerimientos eran:

1. Saber si Leo DiCaprio actuó en determinada película.
2. Saber en qué películas actuó un actor.
3. Saber quiénes actuaron en una película.
4. Saber si un actor ganó algún Oscar.

**En funcional** se modeló una estructura `Pelicula { titulo, actores }` (lista de actores anidada dentro de la película), y para cada requerimiento hubo que **definir una función distinta** (`actuoEn`, `buscarPelicula`, `actoresDe`, etc.), a veces reutilizando funciones auxiliares y componiendo (`head . filter`). Esto trajo complicaciones: por ejemplo, si se preguntaba por una película que no estaba en la base, se rompía o devolvía una lista vacía poco intuitiva; y hubo que lidiar con aplicación parcial, funciones de orden superior y composición.

**En lógico**, en cambio, alcanzó con definir una única relación:

```prolog
actuo(Pelicula, Actor).
```

Y los cuatro requerimientos se resolvieron con **consultas directas** sobre esa misma relación (variando qué argumento es individuo concreto y cuál es variable), sin tener que escribir ni una función. Es el mismo fenómeno que vimos con `plato(Pasta, Salsa)` en la sección de inversibilidad: **una sola relación inversible reemplaza a varias funciones**.

### ¿Por qué el paradigma lógico resulta más simple acá?

- **Bidireccionalidad / inversibilidad**: una función en el paradigma funcional tiene un dominio fijo y una imagen fija (va "en una sola dirección"). Una **relación**, en cambio, sirve tanto para "¿en qué películas actuó X?" como para "¿quién actuó en Y?", simplemente cambiando qué argumento dejamos como variable.
- **No hace falta pensar en una función por requerimiento**: en lógico, "relaciono los individuos correctamente" y las distintas consultas salen solas.
- **No hay jerarquía entre los individuos**: en el modelo funcional, los actores quedaban _anidados dentro_ de la estructura película (una jerarquía). En el modelo lógico, `pelicula` y `actor` aparecen como **individuos "hermanos"** dentro de la relación `actuo`, sin que uno esté contenido en el otro.

> Importante: esto no significa que el paradigma funcional esté "mal", sino que cada paradigma tiene reglas de juego distintas, y hay problemas que calzan naturalmente mejor en uno que en otro.

## 14. ¿Dónde se aplica la lógica en computación?

- **Bases de datos**: lenguajes de restricciones y de consulta (ej. SQL).
- **Inteligencia artificial**: representación del conocimiento, deducción.
- **Especificación de sistemas** (lenguaje Z), usado en industria aeronáutica, ferroviaria, de defensa, etc.
- **Criptografía**: verificación de protocolos criptográficos.
- **Procesamiento del lenguaje natural**.
- En general, cualquier modelo que defina **reglas para la toma de decisiones**.

## 15. Práctica resuelta en clase

### 15.1 Ejemplo: Gustos

Consigna en lenguaje natural (ojo: el castellano es ambiguo — hay que interpretar bien si un "y" es en realidad un OR lógico):

**"Juan gusta de María"** → hecho simple, predicado poliádico (relaciona dos individuos):

```prolog
gusta(juan, maria).
```

**"Pedro gusta de Ana y de Nora"** → en realidad son dos hechos independientes (equivale a un OR: "a Pedro le gusta Ana, y también le gusta Nora"):

```prolog
gusta(pedro, ana).
gusta(pedro, nora).
```

**"Todos los que gustan de Nora gustan de Zulema"** → es una implicación (_p ⇒ q_, "si a alguien le gusta Nora, entonces le gusta Zulema"):

```prolog
gusta(Alguien, zulema) :- gusta(Alguien, nora).
```

**"Julián gusta de las morochas y de las chicas con onda"** → conviene leerlo dado vuelta: "a Julián le gusta una persona si es morocha, o si tiene onda" (acá el "y" del enunciado es en realidad un OR → dos reglas separadas):

```prolog
gusta(julian, Persona) :- morocha(Persona).
gusta(julian, Persona) :- tieneOnda(Persona).
```

**"Mario gusta de las morochas con onda y de Luisa"** → acá sí hay un AND real (tiene que ser morocha **y** tener onda a la vez) más un hecho aparte para Luisa:

```prolog
gusta(mario, Chica) :- chica(Chica), morocha(Chica), tieneOnda(Chica).
gusta(mario, luisa).
```

**"Todos los que gustan de Ana y de Luisa, gustan de Laura"** — con "y" (AND, deben cumplirse ambas condiciones):

```prolog
gusta(Alguien, laura) :- gusta(Alguien, ana), gusta(Alguien, luisa).
```

...y la misma consigna cambiando el "y" por un "o" (OR, dos reglas separadas):

```prolog
gusta(Alguien, laura) :- gusta(Alguien, ana).
gusta(Alguien, laura) :- gusta(Alguien, luisa).
```

### 15.2 Ejemplo: Familia (Los Simpson)

Base de conocimiento:

```prolog
progenitor(homero, bart).
progenitor(homero, lisa).
progenitor(homero, maggie).
progenitor(abe, homero).
progenitor(abe, jose).
progenitor(jose, pepe).
progenitor(mona, homero).
progenitor(jacqueline, marge).
progenitor(marge, bart).
progenitor(marge, lisa).
progenitor(marge, maggie).
```

**Hermano**: dos personas son hermanas si comparten un progenitor y son individuos distintos.

```prolog
hermano(Hermano1, Hermano2) :-
    progenitor(Padre, Hermano1),
    progenitor(Padre, Hermano2),
    Hermano1 \= Hermano2.
```

El operador `\=` compara dos individuos: se satisface si son **distintos**, falla si son iguales (necesario para que `hermano(bart, bart)` no dé `true`).

**Tío**: se puede definir en dos direcciones distintas, según el orden de los argumentos que se elija:

```prolog
% tio(Sobrino, Tio)
tio(Sobrino, Tio) :- progenitor(Padre, Sobrino), hermano(Padre, Tio).

% tio(Tio, Sobrino)
tio(Tio, Sobrino) :- hermano(Tio, Padre), progenitor(Padre, Sobrino).
```

**Primo**:

```prolog
primo(Primo1, Primo2) :-
    progenitor(Padre, Primo1),
    progenitor(Tio, Primo2),
    hermano(Padre, Tio).
```

O reutilizando la definición de `tio/2` ya escrita:

```prolog
primo(Primo1, Primo2) :-
    tio(Primo1, Tio),
    progenitor(Tio, Primo2).
```

**Abuelo**:

```prolog
abuelo(Abuelo, Nieto) :-
    progenitor(Abuelo, Padre),
    progenitor(Padre, Nieto).
```

Consultas de ejemplo:

```prolog
?- abuelo(Quien, bart).
Quien = abe.

?- progenitor(homero, Quien).
Quien = bart ;
Quien = lisa ;
Quien = maggie ;
false.
```

> Para repasar: pensar cuáles de estos predicados (`hermano`, `tio`, `primo`, `abuelo`) son totalmente inversibles y cuáles no.

## 16. Adelanto: motor de inferencia y backtracking

Se mostró una demo de un juego de laberinto resuelto en Prolog, donde dos "fantasmitas" competían por encontrar la salida con dos estrategias de búsqueda distintas (uno con DFS le ganaba al otro). Es la misma idea que el backtracking de las consultas: el motor va dejando una marca en el último punto donde encontró una solución, y si hace falta más, retoma la búsqueda desde ahí — como ir explorando un laberinto y volver sobre los propios pasos cuando una rama no lleva a ninguna salida.

---

## 📌 Glosario rápido para repasar

| Término                           | Definición                                                                                                             |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Base de conocimiento**          | Archivo `.pl` donde se declaran los hechos y reglas; define el universo conocido.                                      |
| **Individuo / átomo**             | Elemento concreto que participa de una relación (equivalente a un "dato"). Se escribe en minúscula.                    |
| **Predicado**                     | Nombre de una relación o característica. Tiene una aridad fija.                                                        |
| **Aridad**                        | Cantidad de argumentos de un predicado.                                                                                |
| **Predicado monádico**            | Aridad 1. Expresa un atributo/característica de un individuo.                                                          |
| **Predicado poliádico**           | Aridad > 1. Expresa una relación entre varios individuos.                                                              |
| **Hecho**                         | Cláusula axiomática: verdadera por estar en la base de conocimiento; sin antecedentes.                                 |
| **Regla**                         | Cláusula con uno o más antecedentes; se satisface si se cumplen todos ellos.                                           |
| **Cláusula**                      | Hecho o regla; unidad de la base de conocimiento.                                                                      |
| **Cláusula de Horn**              | Forma `consecuente :- antecedente(s).` usada por Prolog para escribir reglas (equivalente a _p ⇒ q_ escrito al revés). |
| **Antecedente / consecuente**     | Condición(es) de una regla / lo que se concluye si se cumplen.                                                         |
| **Variable**                      | Incógnita dentro de una consulta o regla. Se escribe en mayúscula.                                                     |
| **Variable anónima (`_`)**        | Variable cuyo valor no importa; solo se usa para verificar existencia.                                                 |
| **Instanciado / libre**           | Un argumento está instanciado si es un individuo concreto, y libre si es una variable.                                 |
| **Consulta individual**           | Consulta sin variables; responde `true`/`false`.                                                                       |
| **Consulta existencial**          | Consulta con al menos una variable; el motor busca los valores que la satisfacen.                                      |
| **Unificación**                   | Proceso por el cual una variable se resuelve con uno o más valores que satisfacen la relación (no es una asignación).  |
| **Inversibilidad**                | Capacidad de un predicado de admitir consultas existenciales sobre cualquiera de sus argumentos.                       |
| **Backtracking**                  | Mecanismo del motor de inferencia para probar y descartar alternativas hasta encontrar (todas) las soluciones.         |
| **Principio de Universo Cerrado** | Todo lo que no se puede demostrar se asume falso (el usado por Prolog).                                                |
| **Principio de Universo Abierto** | Existen tres estados posibles: verdadero, falso y desconocido.                                                         |
| **Definición por extensión**      | Definir un predicado enumerando uno por uno todos los hechos que lo satisfacen.                                        |
| **Definición por comprensión**    | Definir un predicado mediante una regla/condición, sin enumerar los casos.                                             |
| **Conjunción (AND)**              | Se escribe con una coma `,` entre condiciones dentro de una misma regla.                                               |
| **Disyunción (OR)**               | Se escribe como dos (o más) reglas distintas para el mismo predicado.                                                  |
