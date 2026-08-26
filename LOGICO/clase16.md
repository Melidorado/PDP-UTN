# Paradigma Lógico — Recursividad (Módulo 5)

> Fuentes: Módulo 5 de la cátedra (Dodino, Lombardi, Passerini, Solmirano) + transcripción de la clase teórico-práctica en vivo.

## 1. ¿Qué es la recursividad en lógico?

Recursividad es que un elemento de software se defina **en función de sí mismo**. En el paradigma lógico, esto significa **definir un predicado en términos de sí mismo**.

Repasando: en lógico trabajamos con predicados/cláusulas, y un programa lógico es un conjunto de cláusulas que definen relaciones entre individuos. Entonces, lo recursivo va a ser **la definición de un predicado**.

Como en cualquier definición recursiva (la recursividad en lógico se apoya en la idea de **inducción**), necesitamos:

- al menos un **caso base** (el `P₀`),
- y al menos un **caso recursivo**, donde dado que se cumple `Pₙ`, podemos verificar que se cumple `Pₙ₊₁`.

## 2. Primer ejemplo: ancestros

Pensándolo declarativamente: _"¿Quiénes son mis ancestros? Mis padres, y los ancestros de mis padres."_ — noten que estamos definiendo `ancestro` en términos de `ancestro` (y recordemos que ese "y" en castellano es en realidad un **or** lógico: dos cláusulas separadas).

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

% caso base: mi ancestro es mi progenitor directo
ancestro(Ancestro, Persona) :- progenitor(Ancestro, Persona).

% caso recursivo: mi ancestro es el progenitor de alguien que ya es mi ancestro...
% o, dicho como en la cátedra: existe alguien que es hijo directo de Ancestro,
% y ese alguien es (a su vez) ancestro de Persona
ancestro(Ancestro, Persona) :-
    progenitor(Padre, Persona),
    ancestro(Ancestro, Padre).
```

```prolog
?- ancestro(Quien, bart).
Quien = homero ;   % caso base: progenitor directo
Quien = marge  ;   % caso base: progenitor directo
Quien = abe    ;   % caso recursivo: progenitor de homero
Quien = mona   ;   % caso recursivo: progenitor de homero
Quien = jacqueline ; % caso recursivo: progenitor de marge
false.
```

La solución recursiva encuentra relaciones a **n niveles de profundidad**, sin que nosotros tengamos que escribir explícitamente cada nivel de parentesco (abuelo, bisabuelo, etc.).

### ⚠️ El orden dentro de la cláusula recursiva importa (y mucho)

Uno podría pensar: _"¿y si escribo el caso recursivo dando vuelta el orden, primero la llamada recursiva a `ancestro` y después `progenitor`?"_ — total, `p ∧ q` debería ser lo mismo que `q ∧ p`, ¿no?

```prolog
% ⚠️ Versión problemática — NO HACER ESTO
ancestro(Ancestro, Persona) :- progenitor(Ancestro, Persona).
ancestro(Ancestro, Persona) :-
    ancestro(Ancestro, OtraPersona),
    progenitor(OtraPersona, Persona).
```

Con esta versión, las primeras soluciones aparecen bien (los progenitores directos y luego los abuelos), pero en algún momento la búsqueda **entra en un loop infinito** que termina rompiendo el motor (se llena el _stack_). ¿Por qué, si lógicamente `p ∧ q ≡ q ∧ p`?

El problema es **operacional**, no lógico: cuando Prolog va resolviendo de izquierda a derecha, en esta versión lo **primero** que intenta hacer para satisfacer el caso recursivo es llamar de nuevo a `ancestro/2` — pero en ese punto todavía no ligamos ninguna variable con algo concreto (`OtraPersona` es una incógnita nueva). Como no hay ninguna cláusula previa que "corte" o acote esa búsqueda, cada llamada recursiva genera **otra** llamada recursiva, sin nunca llegar a un punto donde el motor pueda fallar y frenar.

En cambio, en la versión original (`progenitor(Padre, Persona), ancestro(Ancestro, Padre)`), lo primero que se intenta es `progenitor/2` — que es un **hecho**, con una cantidad **finita** de cláusulas. Tarde o temprano, esa búsqueda de progenitores se agota (no hay más antepasados cargados en la base), la consulta `progenitor(...)` falla, y como falla, **nunca se llega a hacer la llamada recursiva** en ese camino — así el algoritmo converge.

> **Regla general clave**: al definir un predicado recursivo, asegurate de que **antes** de la llamada recursiva haya alguna cláusula (típicamente un hecho, o un predicado ya acotado) que eventualmente pueda **fallar** y así cortar la recursión. El orden de las condiciones, aunque lógicamente equivalente, **sí importa operacionalmente** en Prolog.

### Inversibilidad

Como `ancestro/2` está definido en base a `progenitor/2` (que es totalmente inversible, al ser hechos), `ancestro/2` también termina siendo inversible en su primer argumento — e incluso se puede consultar con **ambos** argumentos libres:

```prolog
?- ancestro(Ancestro, Persona).
```

Esto devuelve **todas** las relaciones de ancestro posibles entre las personas de la base de conocimiento.

## 3. Segundo ejemplo: distancias y rutas

### 3.1 Versión simple: solo la distancia total

```prolog
distancia(buenosAires, puertoMadryn, 1300).
distancia(puertoMadryn, puertoDeseado, 732).
distancia(puertoDeseado, rioGallegos, 736).
distancia(puertoDeseado, calafate, 979).
distancia(rioGallegos, calafate, 304).
distancia(calafate, chalten, 213).
```

Podemos calcular los kilómetros entre dos puntos de dos maneras: porque conocemos la distancia directa entre ellos, o porque conocemos la distancia hasta un punto intermedio, desde el cual ya sabemos llegar al destino. De nuevo, una definición recursiva:

```prolog
kilometrosViaje(Origen, Destino, Kms) :-
    distancia(Origen, Destino, Kms).

kilometrosViaje(Origen, Destino, KmsTotales) :-
    distancia(Origen, PuntoIntermedio, KmsIntermedios),
    kilometrosViaje(PuntoIntermedio, Destino, KmsFinales),
    KmsTotales is KmsIntermedios + KmsFinales.
```

Como el mapa **no es lineal** (hay bifurcaciones: desde Puerto Deseado se puede ir a Río Gallegos o directamente a Calafate), Prolog puede encontrar **más de una ruta posible** entre dos puntos, cada una con su propia distancia total — el mismo mecanismo de backtracking que venimos viendo, aplicado ahora sobre una definición recursiva.

Como las distancias solo están cargadas "en un sentido", para poder viajar también "de vuelta" agregamos un predicado que prueba ambos sentidos:

```prolog
totalViaje(Origen, Destino, Kms) :- kilometrosViaje(Origen, Destino, Kms).
totalViaje(Origen, Destino, Kms) :- kilometrosViaje(Destino, Origen, Kms).
```

```prolog
?- totalViaje(calafate, buenosAires, Kms).
Kms = 3285 ;
...
```

### 3.2 Versión enriquecida: `ruta/4`, agregando el recorrido

En la clase se amplió el ejemplo para además poder ver **por qué ciudades se pasa**, agregando un cuarto argumento con la lista de ciudades recorridas (sin incluir el origen, ya que "ahí ya estamos"):

```prolog
% caso base: hay distancia directa; el recorrido es solo el destino
ruta(Origen, Destino, Distancia, [Destino]) :-
    distancia(Origen, Destino, Distancia).

% caso recursivo: paso por una ciudad intermedia
ruta(Origen, Destino, DistanciaTotal, [CiudadIntermedia|RestoCiudades]) :-
    distancia(Origen, CiudadIntermedia, DistanciaParcial),
    ruta(CiudadIntermedia, Destino, DistanciaResto, RestoCiudades),
    DistanciaTotal is DistanciaParcial + DistanciaResto.
```

```prolog
?- ruta(buenosAires, chalten, Distancia, Ciudades).
Distancia = 3285, Ciudades = [puertoMadryn, puertoDeseado, rioGallegos, calafate, chalten] ;
Distancia = 3224, Ciudades = [puertoMadryn, puertoDeseado, calafate, chalten] ;
...
```

Este predicado resultó **totalmente inversible** (no tiene ninguna restricción, ya que la suma con `is/2` se calcula recién cuando las incógnitas ya están resueltas), lo que permite consultas muy potentes, por ejemplo:

```prolog
% ¿A dónde puedo ir desde Puerto Madryn haciendo entre 1000 y 2000 km?
?- ruta(puertoMadryn, Destino, Distancia, _),
   Distancia > 1000, Distancia < 2000.

% ¿A dónde puedo llegar desde Buenos Aires pasando por exactamente 3 ciudades?
?- ruta(buenosAires, Destino, Distancia, Ciudades),
   length(Ciudades, 3).

% ¿Y pasando por menos de 3 ciudades?
?- ruta(buenosAires, Destino, Distancia, Ciudades),
   length(Ciudades, Cantidad), Cantidad < 3.
```

## 4. Predicado `member/2`

_"Un elemento está en una lista si está en la cabeza, o si está en la cola."_ Como la cola es a su vez una lista, la definición termina siendo recursiva (en la clase se lo llamó `elemento/2` para no chocar con el `member/2` que ya trae Prolog):

```prolog
elemento(X, [X|_]).
elemento(X, [_|Cola]) :- elemento(X, Cola).
```

```prolog
?- elemento(5, [5, 8]).
true.     % matchea la primera cláusula

?- elemento(5, [4, 6]).
false.

?- elemento(5, [4, 3, 5, 6]).
true.     % 5 está en la cola de la cola...
```

### El caso base "implícito": la lista vacía

¿Qué pasa si la lista está vacía? **No hace falta escribir ninguna cláusula para ese caso.** Una lista vacía no se puede descomponer en cabeza y cola, así que **ninguna** de las dos cláusulas matchea con `[]` — y por Principio de Universo Cerrado, si no hay ninguna cláusula que se satisfaga, el predicado simplemente **falla**. Ese fallo automático funciona como un caso base implícito.

### ¿Por qué esto no entra en loop infinito (a diferencia de la versión mala de `ancestro`)?

Buena pregunta, porque a primera vista `elemento/2` tiene el mismo "problema" que la versión problemática de `ancestro/2`: no hay ningún hecho ni predicado acotado _antes_ de la llamada recursiva, es una llamada directa a sí mismo. Sin embargo, acá **sí termina**, y la razón es distinta a la de `ancestro`:

- En `ancestro/2`, lo que evitaba el loop infinito era que **antes** de la recursión había un hecho finito (`progenitor/2`) que eventualmente fallaba.
- En `elemento/2`, en cambio, lo que garantiza el corte es que **cada llamada recursiva recibe una lista estrictamente más chica** (la cola, sin la cabeza). Como las listas son finitas, tarde o temprano llegamos a la lista vacía `[]` — que no matchea ninguna cláusula — y ahí la búsqueda falla y se corta.

> **Dos formas distintas de garantizar que una recursión termine**: 1) que haya una cláusula previa finita que eventualmente falle (como en `ancestro`), o 2) que cada llamada recursiva reciba una versión **estructuralmente más chica** del dato (como la cola de una lista, en `elemento`/`member`), hasta llegar a un caso que no matchea nada.

### Inversibilidad de `member/2`

```prolog
?- elemento(X, [5, 2, 7]).
X = 5 ;
X = 2 ;
X = 7 ;
false.
```

`elemento/2` (`member/2`) es **inversible para el primer argumento**: dejando la lista instanciada, se pueden encontrar todos los elementos que contiene. **No** tiene mucho sentido, en cambio, dejar el segundo argumento (la lista) libre — ¿cuál sería el universo de todas las listas posibles que contienen, por ejemplo, un `6`? No hay forma de acotar eso.

### No determinismo

```prolog
?- elemento(2, [5, 2, 7]).
true ;
false.
```

El predicado tiene una bifurcación (caso base / caso recursivo), así que es **no determinístico**: encuentra que `2` sí está (por la rama del caso base, una vez que baja hasta la cabeza correcta), pero al seguir buscando por las demás ramas, esas fallan — de ahí que aparezca tanto `true` como, si seguimos pidiendo soluciones, un eventual `false`.

## 5. Predicado `nth1/3` (`enesimo/3`)

Relaciona la posición que ocupa un elemento dentro de una lista (contando desde 1). _(En Prolog ya existe con este nombre; también existe `nth0/3`, que arranca contando desde 0.)_

```prolog
?- nth1(2, [1, 2, 3, 5, 8, 13], Elemento).
Elemento = 2.

?- nth1(Posicion, [1, 2, 3, 5, 8, 13], 5).
Posicion = 4.
```

- **Caso base**: el elemento está en la cabeza → ocupa la posición 1.
- **Caso recursivo**: si no está en la cabeza, hay que buscarlo en la cola — y su posición va a ser 1 + la posición que ocupa dentro de esa cola.

```prolog
enesimo(1, [Elemento|_], Elemento).
enesimo(Posicion, [_|Resto], Elemento) :-
    enesimo(PosicionCola, Resto, Elemento),
    Posicion is PosicionCola + 1.
```

### ⚠️ Un detalle sutil: el orden importa también acá, por culpa de `is/2`

Es tentador escribir el caso recursivo "yendo para abajo" (calculando primero la nueva posición y recién después recursando):

```prolog
% ⚠️ Versión que rompe la inversibilidad del 2do argumento (Posición)
enesimo(1, [Elemento|_], Elemento).
enesimo(Posicion, [_|Resto], Elemento) :-
    NuevaPosicion is Posicion - 1,
    enesimo(NuevaPosicion, Resto, Elemento).
```

El problema: `is/2` necesita que la expresión de la derecha esté **totalmente instanciada**. Si dejamos `Posicion` como incógnita (por ejemplo, para preguntar "¿en qué posición está el 5?"), `Posicion - 1` no se puede evaluar, y la consulta se rompe con `ERROR: is/2: Arguments are not sufficiently instantiated`.

**La solución** es la que ya vimos en la versión "buena" más arriba: calcular `Posicion` **después** de la llamada recursiva (usando el valor que ya devolvió `PosicionCola`), en vez de necesitarlo _antes_. Así, dejar `Posicion` libre en la consulta deja de ser un problema, porque nunca hace falta evaluarlo como entrada — se termina **derivando** como salida.

Con la versión correcta:

```prolog
?- enesimo(Posicion, [7, 3, 9], 3).
Posicion = 2.
```

Y si el elemento aparece más de una vez en la lista, `enesimo/3` (siendo no determinístico) puede devolver **todas** las posiciones posibles al seguir pidiendo soluciones.

### Paso a paso: cómo lo resuelve el motor de inferencia (backtracking + recursividad)

Veamos en detalle qué hace Prolog para resolver `enesimo(3, [5, 8, 9, 3], Elemento)` (¿qué elemento ocupa la posición 3?), usando la versión correcta de la sección anterior:

```
Nivel 0: enesimo(3, [5,8,9,3], Elemento)
  · Caso base: ¿1 = 3?  No → falla.
  · Caso recursivo: descarto la cabeza (5), Resto = [8,9,3].
      Necesito resolver primero: enesimo(PosCola₀, [8,9,3], Elemento)
      y recién después verificar: 3 is PosCola₀ + 1

Nivel 1: enesimo(PosCola₀, [8,9,3], Elemento)   (ambas variables libres acá)
  · Caso base: PosCola₀ = 1, Elemento = 8. ✔ (matchea directo, sin condiciones extra)
  · Subo a Nivel 0 y verifico: 3 is 1 + 1 → 3 = 2 → FALSO.
    Backtrack: descarto esta solución y pruebo la otra rama en el Nivel 1.
  · Caso recursivo (Nivel 1): descarto la cabeza (8), Resto = [9,3].
      Necesito: enesimo(PosCola₁, [9,3], Elemento), y luego PosCola₀ is PosCola₁ + 1

Nivel 2: enesimo(PosCola₁, [9,3], Elemento)
  · Caso base: PosCola₁ = 1, Elemento = 9. ✔
  · Subo a Nivel 1: PosCola₀ is 1 + 1 = 2. ✔
  · Subo a Nivel 0: 3 is 2 + 1 = 3. ✔✔✔ ¡Se satisface!

Resultado: Elemento = 9
```

(Efectivamente, en `[5, 8, 9, 3]`, la posición 3 es `9` — ¡chequea!)

Después de encontrar esta solución, si seguimos pidiendo más (`;`), Prolog retoma desde la última marca de backtracking y sigue probando ramas alternativas más profundas (con `[3]` y luego `[]`), hasta que — igual que con `member/2` — llega a la lista vacía, que no matchea ninguna cláusula, falla, y ahí se termina la búsqueda. **Por eso `enesimo/3` tampoco entra en loop infinito**: cada llamada recursiva recibe una lista más chica (misma razón estructural que vimos con `member/2`).

## 6. Resumen

Prolog permite definir predicados recursivos, que deben tener **un caso base** (o corte de la recursividad) y **al menos un caso recursivo**, donde el predicado se define en términos de sí mismo. El motor de inferencia puede encontrar múltiples soluciones también para predicados recursivos, siempre que no haya restricciones de inversibilidad de por medio (como las que vimos con `is/2`).

Un punto crucial — que no suele estar tan explícito en otros paradigmas — es que **el orden en el que se escriben las condiciones dentro de una cláusula recursiva importa operacionalmente**, aunque lógicamente sean equivalentes. Para que una recursión termine (y no entre en loop infinito), hace falta al menos uno de estos dos mecanismos:

1. Una cláusula previa a la llamada recursiva que sea **finita** y eventualmente **falle** (como `progenitor/2` en el ejemplo de `ancestro`), o
2. Que cada llamada recursiva reciba una versión **estructuralmente más chica** del dato con el que trabaja (como la cola de una lista en `member/2` o `nth1/3`), hasta llegar a un caso que no matchea nada.

---

## 📌 Glosario rápido para repasar

| Término                                        | Definición                                                                                                                                                                                              |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Recursividad (en lógico)**                   | Definir un predicado en términos de sí mismo, apoyándose en la idea de inducción: un caso base y al menos un caso recursivo.                                                                            |
| **Caso base**                                  | Cláusula(s) que resuelven directamente la relación, sin necesidad de una llamada recursiva.                                                                                                             |
| **Caso recursivo**                             | Cláusula que define el predicado en función de una llamada a sí mismo (usualmente sobre una versión "más chica" del problema).                                                                          |
| **Orden operacional en la cláusula recursiva** | Aunque `p ∧ q` es lógicamente igual a `q ∧ p`, el **orden en que Prolog evalúa** las condiciones sí puede determinar si una recursión termina o entra en loop infinito.                                 |
| **Corte por cláusula previa finita**           | Mecanismo de terminación: antes de la llamada recursiva hay un predicado acotado (ej. un hecho) que eventualmente falla, cortando la recursión.                                                         |
| **Corte por reducción estructural**            | Mecanismo de terminación: cada llamada recursiva recibe una versión más chica de la estructura (ej. la cola de una lista), hasta llegar a un caso que no matchea ninguna cláusula (ej. la lista vacía). |
| **Caso base implícito**                        | Un caso que no hace falta escribir explícitamente porque, por Principio de Universo Cerrado, simplemente falla al no matchear ninguna cláusula (ej. `elemento(X, [])`).                                 |
| **`nth1/3` / `nth0/3`**                        | Relacionan la posición de un elemento en una lista, contando desde 1 o desde 0 respectivamente.                                                                                                         |

# Predicado `findall/3` (cierre del Módulo 4: Predicados de Orden Superior)

## 1. El problema que resuelve `findall`

```prolog
padre(homero, bart).
padre(homero, maggie).
padre(homero, lisa).
```

Con lo que ya sabemos podemos hacer muchas consultas:

```prolog
?- padre(homero, bart).
true.

?- padre(homero, _).
true.

?- padre(Padre, bart).
Padre = homero.

?- padre(homero, Hijo).
Hijo = bart ;
Hijo = maggie ;
Hijo = lisa.

?- padre(Padre, Hijo).
Padre = homero, Hijo = bart ;
...
```

Pero hay preguntas que se vuelven complicadas o directamente imposibles de responder así. Por ejemplo: **¿cuántos hijos tiene Homero?** Con lo que tenemos, tendríamos que hacer `padre(homero, Hijo)` y **contar a mano** cuántas respuestas nos da — impracticable si hay muchas, y además no responde la pregunta de forma directa.

Queremos un predicado `cantidadDeHijos/2` que relacione directamente una persona con la cantidad de hijos que tiene:

```prolog
?- cantidadDeHijos(homero, Cantidad).
Cantidad = 3.
```

Para definirlo por comprensión (que sirva para cualquier persona, no solo Homero), nos falta una pieza: una forma de juntar **todas** las respuestas de una consulta existencial en una sola lista. Eso es exactamente lo que hace `findall/3`:

```prolog
findall(UnIndividuoOVariable, Consulta, Lista)
```

`findall/3` relaciona un individuo o variable, con una consulta, y con la **lista** de todos los individuos que satisfacen esa consulta.

```prolog
cantidadDeHijos(Padre, Cantidad) :-
    findall(Hijo, padre(Padre, Hijo), Hijos),
    length(Hijos, Cantidad).
```

```prolog
?- cantidadDeHijos(homero, Cantidad).
Cantidad = 3.
```

La variable `Padre` llega **ligada** desde la cláusula que define `cantidadDeHijos/2`, así que la consulta interna del `findall` (`padre(Padre, Hijo)`) efectivamente tiene 3 respuestas para ese `Padre` en particular.

## 2. Inversibilidad de `findall`

Agreguemos más hechos a la base:

```prolog
padre(homero, bart).
padre(homero, maggie).
padre(homero, lisa).
padre(juan, fede).
padre(nico, julieta).
```

Y probemos dejar `Padre` libre:

```prolog
?- cantidadDeHijos(Padre, Cantidad).
Cantidad = 5.
```

**No funciona como esperaríamos** (nos gustaría ver los distintos padres con su cantidad correspondiente, no un solo número). ¿Por qué? Porque `Padre` no llega ligado a `findall`, así que la consulta interna termina siendo `padre(Padre, Hijo)` con **ambas** variables libres — y eso tiene, en total, **5 soluciones** en toda la base (todos los pares padre-hijo que existen). `findall/3` no tira error ni falla por tener variables libres de más (a diferencia de `is/2` o las comparaciones): simplemente junta **todas** las soluciones que encuentre, sin importar que eso mezcle resultados de personas distintas.

**Solución**: como siempre, ligar la variable con un **predicado generador** antes de llegar al `findall`:

```prolog
cantidadDeHijos(Padre, Cantidad) :-
    persona(Padre),   % generación: Padre llega ligado al findall
    findall(Hijo, padre(Padre, Hijo), Hijos),
    length(Hijos, Cantidad).
```

**Otra forma de generar el universo**, considerando no a "todas las personas" sino solo a los que efectivamente son padres:

```prolog
cantidadDeHijos(Padre, Cantidad) :-
    padre(Padre, _),   % generación: solo entre quienes ya son padres
    findall(Hijo, padre(Padre, Hijo), Hijos),
    length(Hijos, Cantidad).
```

### La forma de generar cambia el resultado — no es un detalle menor

```prolog
?- cantidadDeHijos(bart, Cantidad).
```

- Con `persona(Padre)` como generador (si `bart` está considerado una persona): `Cantidad = 0` — bart es una persona válida, simplemente no tiene hijos.
- Con `padre(Padre, _)` como generador: `false` — bart directamente no es un padre, así que ni siquiera entra en el universo que estamos generando.
  En este caso se considera "más saludable" un `0` que un `false`, pero lo importante es entender que **la forma de generar el universo es una decisión de diseño**, y distintas decisiones pueden dar resultados distintos ante el mismo caso límite.

Si no tenemos ya un predicado `persona/1`, hay que agregarlo, con las dos formas que ya conocemos:

```prolog
% por extensión
persona(bart).
persona(lisa).
% ... etc.

% por comprensión
persona(Papa) :- padre(Papa, _).
persona(Hijo) :- padre(_, Hijo).
```

## 3. Consultas más complejas dentro de `findall`

El segundo parámetro de `findall` puede ser **cualquier consulta**, no solo un predicado suelto — se pueden combinar varias condiciones, encerrándolas entre paréntesis (para no romper la aridad 3 de `findall`):

```prolog
findall(X, (p(X), q(X), r(X)), Xs)
```

Ejemplo: cuántos hijos **varones** tiene una persona:

```prolog
cuantosPibes(Persona, Cantidad) :-
    persona(Persona),
    findall(Hijo, (padre(Persona, Hijo), varon(Hijo)), Varones),
    length(Varones, Cantidad).
```

## 4. Intersección de conjuntos

```prolog
interseccion(Xs, Ys, Zs) :-
    findall(E, (member(E, Xs), member(E, Ys)), Zs).
```

(Este es justamente el mismo patrón que vimos antes al hablar de `member(X,A), member(X,B)` dentro de `forall` para `incluido/2` — acá, en vez de verificar que se cumpla para todos, juntamos en una lista los que sí cumplen ambas condiciones.)

## 5. Ejemplo: niños y juegos

```prolog
% jugoCon/3: nene, juego, minutos
jugoCon(tobias, pelota, 15).
jugoCon(tobias, bloques, 20).
jugoCon(tobias, rasti, 15).
jugoCon(tobias, dakis, 5).
jugoCon(tobias, casita, 10).
jugoCon(cata, muniecas, 30).
jugoCon(cata, rasti, 20).
jugoCon(luna, muniecas, 10).
```

```prolog
juegosQueJugo(Nene, CantidadJuegos) :-
    findall(Juego, jugoCon(Nene, Juego, _), Juegos),
    length(Juegos, CantidadJuegos).

?- juegosQueJugo(tobias, CuantosJuegos).
CuantosJuegos = 5.

minutosQueJugo(Nene, CuantosMinutos) :-
    findall(Minutos, jugoCon(Nene, _, Minutos), ListaMinutos),
    sumlist(ListaMinutos, CuantosMinutos).

?- minutosQueJugo(cata, Minutos).
Minutos = 50.
```

De nuevo, sin generador, `minutosQueJugo(Nene, Minutos)` con `Nene` libre no es inversible correctamente: suma los minutos de **todos** los chicos juntos (`Minutos = 125`), porque `Nene` queda libre dentro del `findall`. La solución es la misma de siempre — generar `Nene` antes:

```prolog
minutosQueJugo(Nene, CuantosMinutos) :-
    nene(Nene),
    findall(Minutos, jugoCon(Nene, _, Minutos), ListaMinutos),
    sumlist(ListaMinutos, CuantosMinutos).
```

## 6. Individuos compuestos en el primer parámetro de `findall`

A veces queremos una lista de individuos que **no existen tal cual** en el programa — hace falta combinarlos usando un **functor**.

_Ejemplo_: con un predicado `puntaje(Equipo, CantidadPuntos)`, queremos armar la "tabla de posiciones": una lista donde cada elemento sea un equipo **junto con** su cantidad de puntos.

```prolog
?- findall(puntos(Equipo, CantidadPuntos),
           puntaje(Equipo, CantidadPuntos),
           Tabla).
```

- `puntos/2` es un **functor** (agrupa equipo y puntos en un solo individuo compuesto), **no** un predicado.
- `puntaje/2` es el **predicado**, no un functor.
- `Tabla` termina siendo una lista de functores `puntos(...)`, uno por cada solución de la consulta.

## 7. Ejercicio integrador: coleccionista

```prolog
tiene(juan, foto([juan, hugo, pedro, lorena, laura], 1988)).
tiene(juan, foto([juan], 1977)).
tiene(juan, libro(saramago, "Ensayo sobre la ceguera")).
tiene(juan, bebida(whisky)).
tiene(valeria, libro(borges, "Ficciones")).
tiene(lucas, bebida(cusenier)).
tiene(pedro, foto([juan, hugo, pedro, lorena, laura], 1988)).
tiene(pedro, foto([pedro], 2010)).
tiene(pedro, libro(octavioPaz, "Salamandra")).

premioNobel(octavioPaz).
premioNobel(saramago).
```

_"Alguien es coleccionista si todos los elementos que tiene son valiosos: un libro de un premio Nobel es valioso; una foto con más de 3 integrantes es valiosa; una foto anterior a 1990 es valiosa; el whisky es valioso."_

```prolog
coleccionista(Alguien) :-
    forall(tiene(Alguien, Cosa), vale(Cosa)).

vale(foto(Gente, _)) :- length(Gente, Cantidad), Cantidad > 3.
vale(foto(_, Anio)) :- Anio < 1990.
vale(libro(Escritor, _)) :- premioNobel(Escritor).
vale(bebida(whisky)).
```

```prolog
?- coleccionista(pedro).
false.

?- coleccionista(juan).
true.
```

Como ya sabemos por el módulo de `forall`, no es válido preguntar `coleccionista(Quien)` tal cual está — hace falta un predicado generador para `Alguien`:

```prolog
coleccionista(Alguien) :-
    tiene(Alguien, _),
    forall(tiene(Alguien, Cosa), vale(Cosa)).
```

(Este ejercicio en particular no necesita `findall` — se resuelve directamente con `forall/2` —, pero se incluye acá porque es el cierre integrador del módulo completo de predicados de orden superior.)

## 8. Resumen del Módulo 4 completo

Mientras que los predicados de **primer orden** definen características o relacionan individuos entre sí, los predicados de **orden superior** trabajan con consultas como argumentos. Conocimos tres: `not/1`, `forall/2` y `findall/3`, que permiten subir el grado de abstracción de nuestras soluciones.

- `forall/2` trabaja en base a la **cuantificación universal**: "para todos se cumple".
- `not/1` trabaja en base a la **cuantificación individual negativa**: "no existe x tal que cumpla...".
- `findall/3` permite juntar **todas** las soluciones de una consulta en una lista, algo que el motor de inferencia no nos da "de regalo" con una consulta existencial común.
  `forall/2` y `not/1` pueden usarse de forma indistinta en muchos casos (son lógicamente equivalentes), aunque trabajar con afirmaciones asertivas (`forall`) suele ser más cómodo de leer que negar predicados (`not`).

Por último, para que estos tres predicados sean **inversibles**, siempre hace falta ligar de antemano las variables que participan, mediante **predicados generadores**:

- en la negación lógica (`not/1`), porque Prolog no puede determinar los individuos que **no** satisfacen un predicado sin conocer primero todo el universo;
- en `findall/3` y `forall/2`, porque hay que estar atentos a qué variables llegan libres y cuáles ligadas antes de usarlos — de lo contrario, `findall` puede devolver resultados "mezclados" entre distintos casos, y `forall` puede terminar preguntando algo distinto de lo que queríamos.

---

## 📌 Glosario — Findall

| Término                                                | Definición                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`findall/3`**                                        | `findall(Individuo, Consulta, Lista)`: relaciona una plantilla de individuo, una consulta, y la lista de todos los valores que satisfacen esa consulta.                                                                                                                                 |
| **Inversibilidad de `findall`**                        | Si la variable de "afuera" (ej. la persona sobre la que se pregunta) no llega ligada antes del `findall`, la consulta interna corre con esa variable libre y termina mezclando resultados de todos los individuos posibles — sin dar error, a diferencia de `is/2` o las comparaciones. |
| **Predicado generador (en findall)**                   | Un predicado (hecho, o definido por extensión/comprensión) que se usa para ligar de antemano la variable "externa" antes de usarla dentro de un `findall`, evitando que se mezclen resultados.                                                                                          |
| **Individuo compuesto en el 1er parámetro de findall** | Usar un functor en el primer argumento de `findall` para agrupar en un solo individuo información que en la consulta original aparece como variables sueltas (ej. armar una tabla de posiciones).                                                                                       |
| **`sumlist/2`**                                        | Relaciona una lista de números con la suma total de sus elementos (equivalente a `sum_list/2`).                                                                                                                                                                                         |
