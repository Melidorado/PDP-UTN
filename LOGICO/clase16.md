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
