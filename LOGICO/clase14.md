# Paradigma Lógico — Estructuras de Datos: Individuos Simples y Compuestos (Módulo 3)

> Fuentes: Módulo 3 de la cátedra (Dodino, Lombardi, Passerini, Solmirano — rev. Freyre) + transcripción de la clase teórico-práctica en vivo.

## 1. Modelando información

Más allá del paradigma en el que estemos parados, siempre hay que poder modelar la información del dominio. Ejemplo: modelar un viaje en micro de larga distancia:

- la cantidad de pasajes: un número entero
- el precio: un número que admite decimales
- el día del viaje: una fecha
- si es ida y vuelta: un booleano
- la persona que viaja: hasta ahora la veníamos modelando como individuo simple (`juan`, `macarena`) — otra opción es un string (`"Tobías Sandler"`)
- el viaje en sí: podría ser una estructura compuesta de varios átomos
- varios viajes: ahí aparece la noción de conjunto/lista

Este módulo se dedica a las estructuras de datos disponibles en Prolog para resolver este tipo de modelado.

## 2. Individuos simples

Ya veníamos trabajando con individuos (`ravioles`, `juan`, `fideos`): son **atómicos**, no se pueden descomponer en otros elementos. Se pueden comparar:

```prolog
?- juan = pepe.
false.

?- juan = juan.
true.

?- tobi \= ravioles.
true.
```

### 2.1 Números

Los números se usan como literales:

```prolog
ingrediente(1, pollo).
nota(salvatelli, 8).
```

Además de comparar por igualdad/desigualdad, definen:

- **operadores aritméticos**: `+`, `-`, `*`, `/`, `abs(-2)`, etc.
- **operadores de orden**: `<`, `=<`, `>`, `>=`

#### ⚠️ Igualdad vs. reducción: un punto clave (y confuso al principio)

En Prolog, `=` compara **expresiones**, no las evalúa/reduce primero. Esto genera resultados que a primera vista parecen "raros":

```prolog
?- 5 = 2 + 3.
false.   % 5 no es la MISMA expresión que 2+3, aunque valgan lo mismo

?- 2 + 3 = 3 + 2.
false.   % son expresiones distintas (orden de términos distinto)

?- 2 + 3 = 2 + 3.
true.    % son exactamente la misma expresión
```

La idea es que `2 + 3` es una **expresión sin reducir**: Prolog no la "resuelve" a `5` a menos que se lo pidamos explícitamente. Es como comparar dos recetas en vez de comparar los platos que resultan de cocinarlas — aunque den el mismo resultado, como _expresiones_ son distintas.

#### El operador `is/2`

Para forzar la evaluación de una expresión aritmética y relacionarla con un número, se usa `is/2`:

```prolog
Numero is Operacion.
```

```prolog
?- 4 is 2 * 2.
true.

?- 4 is 24 - 6.
false.   % 24 - 6 = 18, no 4

?- Z is 2 * 2.
Z = 4.
```

Con esto podemos definir, por ejemplo, el predicado `siguiente/2`:

```prolog
siguiente(Valor, Siguiente) :- Siguiente is Valor + 1.
```

```prolog
?- siguiente(2, 3).
true.

?- siguiente(4, Numero).
Numero = 5.
```

**Restricción importante**: no se puede dejar una variable **libre** (sin instanciar) del lado derecho del `is/2`:

```prolog
?- siguiente(Numero, 4).
ERROR: is/2: Arguments are not sufficiently instantiated
```

Esto pasa porque Prolog necesita poder evaluar la expresión `Numero + 1`, y no puede hacerlo si `Numero` es una incógnita — no tiene forma de "adivinar" o enumerar qué números podrían servir.

Igual pasa con las comparaciones: se puede comparar un número contra un literal (`6 > 3.2` → `true`), pero no se puede dejar una variable libre en una comparación:

```prolog
?- Numero > 3.2.
ERROR: >/2: Arguments are not sufficiently instantiated
```

Tiene sentido: Prolog no puede determinar el universo de todos los números mayores a 3.2 (¡sería infinito!), así que no sabe con qué valores unificar `Numero`.

**En resumen: hay restricciones de inversibilidad en**:

- comparaciones (`>`, `<`, `>=`, `=<`, `=`)
- operaciones aritméticas (`+`, `-`, `*`, `/`)
- el operador `is/2`

En todos estos casos, Prolog necesita que los valores involucrados ya estén instanciados (conocidos) — no puede "adivinar" un universo infinito de números.

### 2.2 Strings

Sirven sobre todo para modelar individuos que necesitan espacios (nombres compuestos, por ejemplo):

```prolog
escritor("Jorge Luis Borges").
escritor("Julio Cortázar").
escritor("Elsa Bornemann").
```

Un átomo y un string son cosas distintas aunque "se vean parecido" (`fede \= "fede"` sería `true`). Más allá de permitir espacios, los strings no aportan mucho más dentro del paradigma, así que no vale la pena invertir demasiado tiempo en ellos.

## 3. Pattern matching: lógico vs. funcional

En Haskell, el pattern matching respeta la **unicidad** de la función matemática: el motor busca el **primer patrón que coincide** y ahí se detiene.

```haskell
valor 0 = 1
valor numero = numero
```

```
Main> valor 0
1
```

(Y no se puede hacer la consulta inversa `valor x` para saber qué entradas dan cierto resultado — no tiene sentido para una función.)

En Prolog, el mismo ejemplo:

```prolog
valor(0, 1).
valor(Incognita, Incognita).
```

```prolog
?- valor(0, Cual).
Cual = 1 ;
Cual = 0.
```

Acá `0` matchea con **ambas** definiciones: con el `0` literal de la primera cláusula, y también con `Incognita` de la segunda (que matchea cualquier cosa). Esto pasa porque una **relación** es más abarcativa que una función: Prolog prueba unificar con **todos** los patrones existentes, no se queda con el primero. En el fondo, esto no es más que una **disyunción (OR)**: son dos cláusulas separadas para el mismo predicado, ni más ni menos que lo que vimos con `afortunado/1` en la clase anterior.

Además, Prolog **no hace chequeo de tipos**: cualquier individuo (simple o compuesto) matchea con una variable libre:

```prolog
?- valor("cero", Cual).
Cual = "cero".
```

## 4. Individuos compuestos: listas

Las listas representan una serie ordenada de elementos, que pueden repetirse:

```prolog
[]                  % lista vacía
[borges]            % lista con un elemento
[1, 4, 9, 10]        % lista con 4 elementos
[8, hermanos, 6.0, "jeje"]   % al no haber chequeo de tipos, se pueden mezclar tipos (aunque haya que pensar bien qué hacer con eso)
```

Una lista es una **estructura recursiva**:

- **caso base**: la lista vacía `[]`
- **caso recursivo**: una lista con al menos un elemento, dividida en **cabeza** (el primer elemento) y **cola** (el resto, que es a su vez una lista). Se denota `[Cabeza|Cola]`.

Ejemplo: `[1, 4, 9, 10]` tiene cabeza `1` y cola `[4, 9, 10]`.

## 5. Operaciones básicas sobre listas

No hace falta entender cómo están implementadas — alcanza con saber usarlas.

### 5.1 `length/2`

Relaciona una lista con su longitud.

```prolog
?- length([picasso, vanGogh, dali], 3).
true.

?- length([picasso, vanGogh, dali], Cantidad).
Cantidad = 3.
```

¿Y si dejamos la lista libre? `length(Lista, 3)` es un caso límite: en teoría Prolog podría generar listas de longitud 3, pero como **no tiene delimitado el universo de listas posibles** (podría ser cualquier combinación de infinitos individuos), en la práctica esto **no funciona bien** — en versiones viejas de Prolog directamente tira error de argumentos no instanciados; en versiones nuevas puede quedarse "inventando" listas sin sentido en un bucle. En criollo: **`length/2` no es un predicado bien inversible para el primer argumento** — funciona bien yendo de lista → longitud, pero no al revés.

### 5.2 `member/2`

Verifica si un elemento pertenece a una lista.

```prolog
?- member(5, [5, 8]).
true.

?- member(Valor, [5, 8]).
Valor = 5 ;
Valor = 8.

?- member(4, [5, 8]).
false.
```

Si la **lista** es la incógnita (`member(4, Lista)`), pasa lo mismo que con `length`: no hay forma de acotar el universo de listas posibles que contengan un 4. Por eso, **`member/2` es inversible solo para el primer argumento** (el elemento), no para el segundo (la lista).

### 5.3 `append/3`

Relaciona dos listas con su concatenación. Este predicado sí es bastante inversible:

```prolog
?- append([1, 3], [2], Resto).
Resto = [1, 2, 3].    % (nota: cuidado, el orden real de concatenación es [1,3] ++ [2])

?- append([1, 3], [2], [1, 2, 3]).
false.

?- append([1, 3], SegundaLista, [1, 3, borges]).
SegundaLista = [borges].

?- append(PrimeraLista, SegundaLista, [1, 3, borges]).
PrimeraLista = [], SegundaLista = [1, 3, borges] ;
PrimeraLista = [1], SegundaLista = [3, borges] ;
PrimeraLista = [1, 3], SegundaLista = [borges] ;
PrimeraLista = [1, 3, borges], SegundaLista = [] ;
false.
```

Es inversible dejando como incógnita el primer argumento, el segundo, el tercero, o incluso el primero y el segundo a la vez — porque acá Prolog sí puede acotar el universo de soluciones posibles a partir de la lista concatenada resultante (el tercer argumento).

### 5.4 `nth0/3` y `nth1/3`

Relacionan un elemento con su posición. `nth0` indexa desde 0, `nth1` desde 1.

```prolog
?- nth0(2, [borges, cortazar, bioy], Elemento).
Elemento = bioy.

?- nth0(Posicion, [borges, cortazar, bioy], cortazar).
Posicion = 1.

?- nth1(2, [borges, cortazar, bioy], Elemento).
Elemento = cortazar.

?- nth1(Posicion, [borges, cortazar, bioy, borges, marechal], borges).
Posicion = 1 ;
Posicion = 4.
```

### 5.5 `last/2`

Relaciona una lista con su último elemento.

```prolog
?- last([1, 2, 3, 4, 5], Elemento).
Elemento = 5.

?- last([1, 2, 3, 4, 5], 4).
false.
```

### 5.6 `reverse/2`

Se verifica si los elementos de una lista están invertidos en la segunda.

```prolog
?- reverse(X, [1, 2, 3]).
X = [3, 2, 1].

?- reverse([1, 2, 3], [1, 2, 3]).
false.

?- reverse([1, 2, 3], X).
X = [3, 2, 1].
```

### 5.7 Otros predicados útiles

```prolog
?- sum_list([7, 2, 6], Total).
Total = 15.

?- list_to_set([2, 3, 2, 2, 3], Conjunto).
Conjunto = [2, 3].

?- max_member(Mayor, [borges, auster, tolstoi]).
Mayor = tolstoi.
% también existe min_member/2

?- subset([1, 2], [1, 2, 3]).
true.
```

También existen operaciones de conjuntos: `intersection/3`, `union/3`, `subtract/3` (vale la pena leer la documentación por su cuenta).

> **Nota de clase**: `max_member`/`min_member` no distingue tipos: compara todo como si fueran strings/átomos según un orden estándar de términos. En ese orden, los números "pesan menos" que las letras — por eso en una lista como `[3, "hola", "amigos"]`, el máximo terminaría siendo un string, no el número.

## 6. Pattern matching con listas

Además del comportamiento general (una variable libre matchea cualquier cosa, incluidas listas), las listas tienen patrones propios:

| Patrón              | Explicación                                                                                                                                           |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `[]`                | Lista vacía. Solo coincide con `[]`.                                                                                                                  |
| `[Elemento]`        | Lista de **un solo** elemento. Para `[8]`, `Elemento = 8`. No coincide con `[]` ni con listas de 2+ elementos.                                        |
| `[Numero\|Numeros]` | Lista con **al menos un** elemento: cabeza y cola. Para `[1,2,3]`, cabeza=`1`, cola=`[2,3]`. Para `[1]`, cabeza=`1`, cola=`[]`. No coincide con `[]`. |

Con esto podemos definir nuestro propio `head/2`:

```prolog
head([Cabeza|_], Cabeza).
```

## 7. Functores

Los **functores** agrupan información relacionada, dándole un nombre a la agrupación (el equivalente lógico al `data` de funcional, pero con "prefijo" o nombre propio que aporta más expresividad):

```prolog
nacio(karla, fecha(22, 08, 1979)).

compro(cliente(231024, "Nelson Pedernera"),
       producto(pirulo, 239, 1)).
```

- `nacio/2` es un **predicado** que relaciona dos átomos: `karla` (individuo simple) y `fecha(22,08,1979)` (un **functor** de aridad 3).
- Un functor **no es un predicado**: no tiene valor de verdad. No se puede hacer `?- fecha(22, 08, 1979).` como consulta independiente — el functor solo tiene sentido _dentro_ de un predicado que lo contenga.
- Tiene **nombre y aridad** (cantidad de individuos que agrupa) → denota un **individuo compuesto**.
- Dos functores con el mismo nombre pero **distinta aridad representan abstracciones distintas** (ej: `fecha/3` no es lo mismo que `fecha/1` con un timestamp único, aunque compartan nombre).
- Es equivalente a una tupla en funcional, pero con nombre propio que aclara de qué se trata.
- Muy útil para agrupar elementos **heterogéneos** que comparten algo en común pero difieren en su información puntual.
- Puede contener átomos, otros functores, o listas.

## 8. Pattern matching con functores

Como una variable libre matchea cualquier individuo (simple o compuesto), esto también aplica a los functores:

```prolog
?- valor(fecha(20, 2, 1988), Valor).
Valor = fecha(20, 2, 1988).
```

Y se puede hacer pattern matching **sobre los átomos que componen un functor**:

```prolog
nacio(karla, fecha(22, 08, 1979)).
nacio(sergio, fecha(14, 10, 1986)).
```

```prolog
?- nacio(Quien, fecha(_, _, 1986)).
Quien = sergio.

?- nacio(_, fecha(_, _, Anio)).
Anio = 1979 ;
Anio = 1986.
```

**Restricción**: no se puede usar una variable como **nombre** del functor — el nombre tiene que ser fijo:

```prolog
?- nacio(_, Functor(_, _, Anio)).   % esto NO es válido en Prolog
```

## 9. El "predicado generador": cómo sortear las limitaciones de inversibilidad

Vimos que operaciones como `is/2`, las comparaciones, `length/2` (segundo argumento libre) o `member/2` (segunda variable libre) **no son inversibles**, porque Prolog no puede determinar el universo completo de valores posibles (números infinitos, listas infinitas, etc.).

**El truco**: si antes de usar una de estas operaciones "no inversibles" acotamos la variable usando un **hecho** (que sí es totalmente inversible, porque su universo de individuos ya está definido explícitamente en la base de conocimiento), entonces sí podemos hacer consultas existenciales.

A este hecho (o predicado ya inversible) que se usa para "marcarle la cancha" a Prolog y acotar el universo de una variable se lo llama **predicado generador**. En el ejemplo integrador de la próxima sección, el predicado `practica/2` cumple ese rol: al ser un conjunto de hechos, permite acotar qué personas y qué deportes existen, y **a partir de ahí** se pueden aplicar con seguridad predicados que de otra forma no serían inversibles (como `is/2` dentro de `esBueno/1`).

> Esta idea se retoma con más fuerza más adelante, cuando se vean recursividad y predicados como `forall`.

## 10. Ejemplo integrador: deportistas 🏅

Base de conocimiento:

```prolog
% natacion: estilos (lista), metros nadados, medallas
practica(ana, natacion([pecho, crawl], 1200, 10)).
practica(luis, natacion([perrito], 200, 0)).
practica(vicky, natacion([crawl, mariposa, pecho, espalda], 800, 0)).

% futbol: medallas, goles marcados, veces que fue expulsado
practica(deby, futbol(2, 15, 5)).
practica(mati, futbol(1, 11, 7)).

% rugby: posición que ocupa, medallas
practica(zaffa, rugby(pilar, 0)).
```

Notar que `natacion/3`, `futbol/3` y `rugby/2` son **functores** (no predicados): agrupan la información específica de cada deporte, con distinta aridad y distinto orden de campos porque cada deporte "le interesa al negocio" de forma diferente.

### 10.1 Quiénes son nadadores

Un nadador es alguien que practica el deporte natación — no importa qué estilos, ni cuántos metros, ni cuántas medallas, por eso usamos variables anónimas:

```prolog
nadador(Quien) :- practica(Quien, natacion(_, _, _)).
```

Por Principio de Universo Cerrado, no hace falta escribir nada sobre futbolistas o rugbiers: simplemente no van a satisfacer este patrón, así que no son nadadores.

### 10.2 Medallas obtenidas

```prolog
medallas(Alguien, Medallas) :-
    practica(Alguien, Deporte),
    cuantasMedallas(Deporte, Medallas).

cuantasMedallas(natacion(_, _, Medallas), Medallas).
cuantasMedallas(futbol(Medallas, _, _), Medallas).
cuantasMedallas(rugby(_, Medallas), Medallas).
```

Acá `practica/2` funciona como **predicado generador**: al ser un hecho, es totalmente inversible, así que unifica `Alguien` con cada persona posible y `Deporte` con el functor correspondiente. Una vez que `Deporte` queda instanciado (ligado a un valor concreto), recién ahí se puede aplicar pattern matching con `cuantasMedallas/2` para extraer las medallas — que en principio no sería invertible por sí solo, pero acá funciona porque ya no hay variables libres sin acotar.

```prolog
?- cuantasMedallas(rugby(hooker, 10), Medallas).
Medallas = 10.
```

El predicado `medallas/2` termina siendo inversible en sus dos argumentos gracias a este mecanismo.

### 10.3 Buen deportista

Reglas del negocio:

- natación: bueno si recorre más de 1000 metros diarios, **o** nada más de 3 estilos
- fútbol: bueno si (goles − expulsiones) > 5
- rugby: bueno si juega de wing o de pilar

```prolog
buenDeportista(Alguien) :-
    practica(Alguien, Deporte),
    esBueno(Deporte).

esBueno(natacion(Estilos, _, _)) :-
    length(Estilos, CantidadEstilos),
    CantidadEstilos > 3.
esBueno(natacion(_, Metros, _)) :- Metros > 1000.

esBueno(futbol(_, Goles, Expulsiones)) :-
    Valor is Goles - Expulsiones,
    Valor > 5.

esBueno(rugby(pilar, _)).
esBueno(rugby(wing, _)).
```

Las últimas dos cláusulas de rugby también se pueden escribir con `member/2`:

```prolog
esBueno(rugby(Puesto, _)) :- member(Puesto, [pilar, wing]).
```

De nuevo, `practica/2` actúa como predicado generador, permitiendo:

```prolog
?- buenDeportista(Alguien).
Alguien = ana ;
Alguien = deby ;
Alguien = zaffa ;
Alguien = vicky ;
false.
```

**¿Por qué usar functores acá y no predicados separados por deporte?** Sin functores, necesitaríamos un `esBueno/1`, `esBueno/2`, `esBueno/3`... con distintas aridades según el deporte, lo cual sería incómodo. Con functores, todos los deportes son "individuos intercambiables" (polimórficos) dentro de un mismo predicado `practica/2`, y el matching se encarga de diferenciarlos.

**Extensibilidad**: si agregamos un deporte nuevo...

```prolog
% nos interesa el handicap del polista; no tiene medallas porque no importa
practica(gise, polo(8)).

esBueno(polo(Handicap)) :- Handicap > 6.
```

...`buenDeportista/1` **no necesita ningún cambio**: sigue funcionando tal cual estaba definido, porque delega en `esBueno/1` la lógica específica de cada deporte. Esta es una ventaja importante de modelar con functores + pattern matching.

## 11. Resumen del módulo

Dentro del paradigma lógico podemos modelar información con **individuos simples** (números, strings, átomos) o **individuos compuestos** (listas y functores). El pattern matching en lógico se diferencia del funcional en que **no busca un único resultado**: prueba unificar con **todos** los patrones definidos, porque en el fondo es equivalente a una disyunción. Además, ciertas operaciones (aritmética, comparaciones, y predicados como `length/2` o `member/2` en algunos de sus argumentos) tienen **restricciones de inversibilidad**, que en la práctica se pueden sortear usando un **predicado generador** que acote de antemano el universo de valores posibles.

---

## 📌 Glosario rápido para repasar

| Término                           | Definición                                                                                                                                                                                         |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Individuo simple**              | Dato atómico: número, string o átomo. No se puede descomponer.                                                                                                                                     |
| **Individuo compuesto**           | Dato formado por varios individuos agrupados: lista o functor.                                                                                                                                     |
| **Expresión**                     | Una operación sin evaluar (ej. `3 + 2`). `=` compara expresiones tal cual están escritas, no las reduce.                                                                                           |
| **`is/2`**                        | Operador que evalúa la expresión de su derecha y la unifica con el valor de su izquierda. Requiere que la expresión de la derecha esté totalmente instanciada.                                     |
| **Restricción de inversibilidad** | Situación en la que Prolog no puede resolver una consulta existencial porque no puede acotar el universo de valores posibles (típico en aritmética, comparaciones, o listas/elementos libres).     |
| **Predicado generador**           | Un hecho (u otro predicado ya inversible) que se usa para acotar de antemano el universo de una variable, permitiendo luego aplicar sobre ella operaciones que de otro modo no serían inversibles. |
| **Pattern matching (lógico)**     | A diferencia de funcional, Prolog intenta unificar con _todos_ los patrones/cláusulas definidos, no solo el primero que coincide — equivale a una disyunción.                                      |
| **Lista**                         | Estructura recursiva de elementos ordenados (pueden repetirse). Caso base: `[]`. Caso recursivo: `[Cabeza\|Cola]`.                                                                                 |
| **Cabeza / Cola**                 | Primer elemento de una lista / resto de la lista (que es a su vez una lista).                                                                                                                      |
| **Functor**                       | Agrupación de individuos bajo un nombre y una aridad. No es un predicado (no tiene valor de verdad), pero sí se puede usar con pattern matching dentro de un predicado.                            |
| **Aridad de un functor**          | Cantidad de individuos que agrupa. Dos functores con igual nombre y distinta aridad son abstracciones distintas.                                                                                   |
