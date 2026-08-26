# TP Lógico — Roster de MMA (para el coloquio)

> Apunte de estudio del TP grupal. Por cada parte: **qué pide la consigna**, **cómo lo resolvimos**, y sobre todo **por qué** — que es lo que se pregunta en el coloquio.

---

## Índice

| Punto | Predicado | Quién |
|---|---|---|
| [1](#punto-1--base-de-conocimientos) | base de conocimientos | todo el grupo |
| [2](#punto-2--esfielasudisciplina-integrante-1) | `esFielASuDisciplina` | Integrante 1 |
| [3](#punto-3--espicante-integrante-2) | `esPicante` | Integrante 2 |
| [4](#punto-4--esnavajasuiza-mi-punto-individual) | `esNavajaSuiza` | **mío** |
| [5](#punto-5--efectividad-y-escontundente) | efectividad + `esContundente` | todo el grupo |
| [6](#punto-6--tieneunpuntoflojo-integrante-1) | `tieneUnPuntoFlojo` | Integrante 1 |
| [7](#punto-7--escertero-integrante-2) | `esCertero` | Integrante 2 |
| [8](#punto-8--estemible-mi-segundo-punto-individual) | `esTemible` | **mío** |
| [9](#punto-9--comboposible-todo-el-grupo) | `comboPosible` | todo el grupo |
| [Cierre](#cierre--conceptos-transversales-para-repasar-el-día-antes) | conceptos transversales | — |

Los puntos marcados como **míos** están explicados paso a paso; el resto, más resumido.

---
---

# Punto 1 — Base de conocimientos

**Consigna**: modelar los luchadores, las disciplinas que entrenan, las técnicas que saben y los cinturones obtenidos. Las técnicas de piso se identifican por la **posición** (guardia, montada, espalda); las de parado, los golpes por la **zona** (cabeza, cuerpo, pierna) y los lances por la **categoría** (tackle, proyección, barrida). Un cinturón violeta de Jiu Jitsu equivale a un negro de Judo. Y lo que **no** agreguemos a la base hay que justificarlo con conceptos de la materia.

## 1. Los luchadores

```prolog
luchador(mora).
luchador(tomi).
luchador(vale).
luchador(mati).
luchador(facu).
luchador(fede).
```

**Por qué existe este predicado.** A primera vista parece redundante: si Mora entrena Jiu Jitsu, ya "se sabe" que Mora es luchadora, y podríamos deducirlo con una regla:

```prolog
% lo que NO hicimos
luchador(Luchador) :- entrena(Luchador, _).
```

El problema es **Fede**: no entrena ninguna disciplina y no sabe ninguna técnica. Si `luchador/1` se dedujera de `entrena/2`, Fede **no existiría como individuo del universo**: no aparecería en ninguna consulta inversible (`luchador(Quien)` no lo traería), y no podríamos preguntar nada sobre él. Y el enunciado dice explícitamente que forma parte del roster.

Ese conocimiento — "Fede pertenece al roster" — **no se deduce de ningún otro hecho**, y por lo tanto tiene que estar escrito en la base. Por eso `luchador/1` va como hechos para los seis.

**El trade-off, dicho en voz alta**: sí, hay una redundancia mínima con `entrena/2`. La aceptamos a propósito, porque la alternativa (que un luchador sin disciplinas no exista) es peor. Además, así `luchador/1` queda definido **todo con hechos**, sin mezclar hechos y reglas para el mismo predicado.

> 🎓 **Concepto**: *individuos*. En el paradigma lógico, un individuo existe en el universo sólo si aparece en algún hecho de la base. No hay "objetos vacíos".

## 2. Las disciplinas y su tipo

```prolog
disciplina(jiuJitsu, piso).
disciplina(sambo, piso).
disciplina(judo, piso).
disciplina(muayThai, parado).
disciplina(boxeo, parado).
disciplina(kickboxing, parado).

otorgaCinturon(Disciplina) :-
	disciplina(Disciplina, piso).
```

**Por qué el tipo va acá y no en `entrena/2`.** Podríamos haber escrito `entrena(mora, jiuJitsu, piso)`, pero eso repetiría "el Jiu Jitsu es de piso" en cada luchador que lo entrene. Es **información redundante**: si mañana cambia, hay que corregirla en N lugares, y peor, la base puede quedar **inconsistente** (que en un hecho diga piso y en otro parado). Poniéndola una sola vez en `disciplina/2`, esa contradicción es imposible.

**Por qué `otorgaCinturon/1` es regla y no hechos.** "Las disciplinas de piso otorgan cinturones" es conocimiento que **se deduce** del tipo de disciplina. Todo lo que se puede deducir va como regla; los hechos se reservan para el conocimiento que no se deduce de nada.

> 🎓 **Concepto**: *hechos vs. reglas*. Regla de oro — si lo puedo inferir, es una regla. Escribirlo como hecho es redundancia.

## 3. Quién entrena qué

```prolog
entrena(mora, jiuJitsu).
entrena(mora, muayThai).
entrena(tomi, muayThai).
...
```

Un hecho por par luchador–disciplina, en vez de una lista (`entrena(mora, [jiuJitsu, muayThai])`).

**Por qué predicados individuales y no listas.** Acá el **orden no importa**: no hay nada en el dominio que dependa de qué disciplina empezó a entrenar primero. Y cuando dos alternativas no tienen una ventaja clara una sobre la otra, **elegimos la más simple**. Los hechos sueltos son más simples de consultar (`entrena(mora, Disciplina)` recorre las dos por backtracking sin tener que meterse en una lista) y más simples de combinar con otros predicados.

**Fede no tiene ningún hecho de `entrena/2`.** No hace falta declarar que no entrena nada: por el **universo cerrado**, todo lo que no se puede deducir de la base es falso. `entrena(fede, _)` ya falla solo. Escribir algo como `entrena(fede, ninguna)` sería inventar un individuo (`ninguna`) que no existe en el dominio y que rompería consultas como "¿quiénes entrenan disciplinas de piso?".

> 🎓 **Concepto**: *universo cerrado*. Lo que no está en la base y no se deduce de ella, es falso. Por eso la ausencia de información **es** información.

## 4. Los cinturones y la equivalencia

```prolog
cinturonObtenido(mora, jiuJitsu, violeta).
cinturonObtenido(vale, judo, marron).
cinturonObtenido(mati, sambo, negro).

cinturon(Luchador, Disciplina, Color) :-
	cinturonObtenido(Luchador, Disciplina, Color).

% El violeta de Jiu Jitsu equivale al negro de Judo.
cinturon(Luchador, judo, negro) :-
	cinturonObtenido(Luchador, jiuJitsu, violeta).
```

**Ésta es la parte más interesante del punto 1.** Hay dos conocimientos distintos y los separamos en dos predicados:

- `cinturonObtenido/3` → lo que la disciplina **realmente otorgó**. Hechos puros.
- `cinturon/3` → lo que se le **reconoce** al luchador, incluyendo equivalencias. Reglas.

**Por qué la equivalencia no es un hecho.** Si hubiéramos escrito `cinturonObtenido(mora, judo, negro).` estaríamos guardando conocimiento que **se deduce** de que Mora tiene el violeta de Jiu Jitsu. Y si mañana Mora asciende a marrón en JJ, habría que acordarse de borrar el hecho del judo a mano. Con la regla, la base **se mantiene sola**: cambiás el cinturón de JJ y la equivalencia se recalcula.

**Por qué dos predicados y no dos cláusulas de `cinturonObtenido/3`.** Para no mezclar hechos y reglas en el mismo predicado, y porque son dos conceptos distintos del dominio: "lo que te dieron" y "lo que te vale". Si más adelante un punto pregunta por cinturones realmente otorgados, tenemos el predicado; si pregunta por equivalencias, también.

**Sentido de la equivalencia.** La modelamos sólo de Jiu Jitsu → Judo, que es como está enunciada ("un violeta en JJ debe considerarse equivalente a negro en Judo") y el único sentido con datos en la base. Modelarla también al revés sería agregar conocimiento que el enunciado no afirma.

Resultado: `cinturon(mora, jiuJitsu, violeta)` y `cinturon(mora, judo, negro)` son ambas verdaderas, aunque Mora no entrene Judo.

## 5. Las técnicas — el uso de functores

```prolog
% piso(Nombre, Posicion)    Posicion: guardia | montada | espalda
% golpe(Nombre, Zona)       Zona: cabeza | cuerpo | pierna
% lance(Nombre, Categoria)  Categoria: tackle | proyeccion | barrida

sabe(mora, piso(triangulo, guardia)).
sabe(mora, golpe(rodillazo, cuerpo)).
sabe(vale, lance(oSotoGari, proyeccion)).
sabe(mati, piso(llaveDeBrazo, espalda)).
sabe(mati, piso(llaveDeBrazo, montada)).
```

> ⚠️ **Ojo**: en el **punto 5** estos tres functores pasan a **aridad 3**, porque se les agrega el atributo de efectividad (`piso(triangulo, guardia, 4)`). Todo lo que se explica acá vale igual; sólo cambia la cantidad de argumentos.

### 5.1 ¿Qué es un functor?

Un **término compuesto** es un individuo formado por un nombre y una lista de argumentos:

```
        piso( triangulo , guardia )
        └┬─┘  └────┬───────────┘
     functor      argumentos
```

El **functor** es el nombre que encabeza el término. Se lo identifica junto con su **aridad** (cantidad de argumentos): acá el functor es `piso/2`. En nuestra base los functores son tres: `piso/2`, `golpe/2` y `lance/2`.

Lo importante es qué **no** es un functor:

- **No es un predicado.** `piso(triangulo, guardia)` no se consulta, no se demuestra, no es verdadero ni falso. Es un **dato**. Prolog nunca va a intentar probarlo, porque está en posición de argumento, no de objetivo. La sintaxis es idéntica a la de un predicado (`nombre(args)`), y lo único que los distingue es **la posición donde aparecen**: lo de afuera se demuestra, lo de adentro es información.
- **No es una función.** No calcula ni devuelve nada. Sólo **agrupa** información que va junta.

Un functor es, en criollo, **la forma de armar un individuo compuesto**: en vez de un individuo simple como `mora` o `guardia`, armás uno que tiene partes.

### 5.2 Dónde están los functores en nuestro código

```prolog
sabe( mora , piso(triangulo, guardia) ).
└─┬─┘         └──────────┬───────────┘
predicado          término compuesto
 sabe/2           (functor piso/2)
```

- `sabe/2` es el **predicado**: es lo que se consulta, y es verdadero o falso.
- `piso/2`, `golpe/2` y `lance/2` son **functores**: arman el individuo "técnica" que ocupa el segundo argumento.
- `mora`, `triangulo`, `guardia`, `cabeza` son **átomos**: individuos simples, sin partes.

**Ojo con un detalle que puede confundir**: la palabra `piso` aparece en dos roles distintos en la base.

```prolog
disciplina(jiuJitsu, piso).           % acá `piso` es un ÁTOMO (individuo simple)
sabe(mora, piso(triangulo, guardia)). % acá `piso` es un FUNCTOR de aridad 2
```

Para Prolog no hay ninguna ambigüedad, porque los distingue por la aridad (`piso/0` vs. `piso/2`) y nunca los va a confundir. Son dos cosas distintas que casualmente se escriben igual.

### 5.3 Para qué sirve el functor acá: la unificación

Dos términos compuestos **unifican** si tienen el mismo functor, la misma aridad, y sus argumentos unifican uno a uno. Eso es lo que hace que estas consultas funcionen sin escribir una sola regla:

```prolog
?- sabe(Quien, golpe(_, cabeza)).       % quién pega a la cabeza
?- sabe(mati, piso(llaveDeBrazo, Donde)). % desde dónde hace la llave de brazo
?- sabe(Quien, lance(_, _)).            % quién sabe algún lance
?- sabe(mora, Tecnica).                 % todas las técnicas de Mora, del tipo que sean
```

`golpe(_, cabeza)` no unifica con `piso(triangulo, guardia)` porque el functor es distinto; sí unifica con `golpe(patada, cabeza)`. Ese filtrado es **gratis**: lo hace el motor de Prolog al unificar, no lo programamos nosotros.

Si en cambio hubiéramos escrito el nombre y el criterio pegados en un solo átomo (`trianguloDesdeLaGuardia`), la posición quedaría **escondida adentro del átomo** y ninguna de esas consultas sería posible: un átomo no tiene partes que unificar.

### 5.4 Por qué no una base de técnicas aparte

La primera alternativa que se podría intentar es declarar las técnicas por su cuenta y que `sabe/2` guarde sólo el nombre:

```prolog
% lo que NO hicimos
golpe(patada, cabeza).
piso(triangulo, guardia).

sabe(mora, patada).
sabe(vale, patada).
```

**Esto rompe con un dato que ya está en el TP: la patada.**

```prolog
sabe(mora, golpe(patada, cabeza)).   % Mora patea a la cabeza
sabe(vale, golpe(patada, pierna)).   % Vale patea a la pierna
```

Con una base aparte, `sabe(vale, patada)` + `golpe(patada, Zona)` devolvería **cabeza y pierna** las dos, y es falso: Vale patea a la pierna. Lo mismo con la llave de brazo de Mati, que la sabe desde la espalda **y** desde la montada.

El concepto de fondo: **el nombre no identifica a la técnica**. La identidad de la técnica es nombre + criterio. Si los separás en dos predicados, perdés justamente la información que la consigna pide modelar.

Y hay un segundo motivo, de diseño: una base aparte tendría sentido si de una técnica supiéramos algo **independiente de quién la sabe** (dificultad, puntaje, años de aprendizaje). No sabemos nada de eso, así que declararla aparte sería pura **redundancia**.

### 5.5 Por qué no un predicado por cada tipo

La otra alternativa era usar tres predicados distintos:

```prolog
% lo que TAMPOCO hicimos
piso(mora, triangulo, guardia).
golpe(mora, rodillazo, cuerpo).
lance(vale, oSotoGari, proyeccion).
```

Ésta **no pierde datos** — es correcta. Lo que pierde es **polimorfismo**. Mirá qué pasa con una consulta general como "todas las técnicas que sabe Mora":

```prolog
% con functores: una sola consulta
?- sabe(mora, Tecnica).

% con predicados separados: tres findall y unir listas
?- findall(t(N,P), piso(mora,N,P), Ps),
   findall(t(N,Z), golpe(mora,N,Z), Gs),
   findall(t(N,C), lance(mora,N,C), Ls),
   append([Ps,Gs,Ls], Todas).
```

Y esa consulta general no es hipotética: apenas un punto pida "cuántas técnicas sabe", "qué técnicas comparten dos luchadores" o "es versátil el que sabe de los tres tipos", la vas a necesitar. Con functores, `sabe/2` funciona como **predicado paraguas**: unifica con cualquier tipo de técnica, y el pattern matching distingue el tipo sólo cuando hace falta. Con predicados separados hay **más indirección**: cada consulta general obliga a preguntar por los tres y combinar.

> 🎓 **Concepto**: *functores / términos compuestos*. Agrupan información que va junta y describe a un mismo individuo, permitiendo un predicado "paraguas" que se consulta polimórficamente y se filtra por unificación.

**Frase para el coloquio**: *el functor mantiene junta la información que identifica a la técnica —porque el nombre solo no la identifica, como se ve con la patada— y a la vez deja un único `sabe/2` consultable de forma polimórfica sobre los tres tipos.*

---

## 6. Todo lo que NO agregamos (y la justificación)

Ésta es la parte que la consigna pide explícitamente justificar. En orden de importancia:

| Qué no agregamos | Por qué |
|---|---|
| Hechos de `entrena/2` y `sabe/2` para Fede | **Universo cerrado**: lo que no se deduce es falso, `entrena(fede, _)` ya falla. Declarar la ausencia sería redundante. |
| `luchador/1` como regla derivada | Fede dejaría de existir como individuo. Ver punto 1. |
| Relación entre técnicas y disciplinas | El enunciado aclara que las técnicas **no** están necesariamente relacionadas con la disciplina del luchador. Modelar esa relación sería inventar conocimiento que **no existe en el dominio**. |
| Una base de técnicas (`tecnica/1`) | No hay nada que sepamos de una técnica con independencia de que alguien la sepa. La info (nombre + criterio) ya está en los términos de `sabe/2`; una base aparte sería redundante. |
| Bases de posiciones, zonas y categorías | Ídem: ya están dentro de los functores. Sólo tendría sentido si algún punto necesitara enumerarlas o darles propiedades propias. |
| El orden jerárquico de los cinturones (blanco < azul < violeta < marrón < negro) | El punto 1 no pide compararlos. La única relación entre cinturones que pide el enunciado es la equivalencia violeta-JJ / negro-Judo, y ésa la resolvimos con una regla. Si un punto posterior pide comparar, ahí se agrega. |
| El tipo piso/parado repetido en `entrena/2` | Está una sola vez en `disciplina/2`. Repetirlo abre la puerta a inconsistencias. |

---

## 7. Preguntas probables del coloquio (y cómo contestarlas)

**"¿Por qué Fede está en la base si no hace nada?"**
Porque pertenece al roster, y eso no se deduce de ningún otro hecho. Si no estuviera declarado en `luchador/1`, no existiría como individuo y sería invisible a cualquier consulta inversible. Lo que **no** declaramos son sus disciplinas y técnicas, porque el universo cerrado ya las da por falsas.

**"¿No es redundante `luchador/1` con `entrena/2`?"**
Parcialmente sí, y es una decisión consciente. La redundancia peligrosa es la que puede volverse **inconsistente**; ésta no lo es (un luchador de más en `entrena/2` sin estar en `luchador/1` sería un error de carga, no una contradicción de datos). A cambio, ganamos poder representar luchadores sin disciplinas.

**"¿Por qué no usaste listas para las disciplinas o las técnicas?"**
Porque el orden no importa y los hechos sueltos son más simples de consultar y de combinar. Las listas convienen cuando el orden es parte del conocimiento.

**"¿Por qué `cinturon/3` tiene dos cláusulas?"**
Porque un cinturón puede estar en la base por dos motivos distintos: haberlo obtenido, o que se lo reconozca por equivalencia. Son dos formas de llegar al mismo hecho, y en lógico eso se expresa con dos cláusulas (**o** lógico).

**"¿Qué pasa si en vez de la regla ponés el hecho `cinturon(mora, judo, negro)`?"**
Funciona igual **hoy**, pero queda conocimiento duplicado: si cambia el cinturón de Jiu Jitsu de Mora, el hecho del Judo queda desactualizado y la base se vuelve inconsistente. La regla mantiene la coherencia sola.

**"¿Cómo consultarías X?"** — algunas que salen directo del modelo:

```prolog
?- entrena(Quien, Disciplina), disciplina(Disciplina, piso).  % quiénes entrenan algo de piso
?- sabe(Quien, golpe(_, cabeza)).                             % quiénes pegan a la cabeza
?- cinturon(Quien, judo, negro).                              % negros de judo (incluye a Mora)
?- sabe(mati, piso(_, montada)).                              % qué hace Mati desde la montada
```

---

## 8. Tests

El archivo tiene 11 tests en la unidad `roster`. Se corren con:

```prolog
?- [solucion].
?- run_tests.
```

Los que vale la pena mirar porque documentan las decisiones:

```prolog
test(fede_no_entrena_ninguna_disciplina, [fail]) :-
	entrena(fede, _).                       % universo cerrado

test(mora_equivale_a_cinturon_negro_en_judo) :-
	cinturon(mora, judo, negro).            % la regla de equivalencia

test(mati_sabe_llave_de_brazo_desde_dos_posiciones) :-
	findall(Posicion, sabe(mati, piso(llaveDeBrazo, Posicion)), Posiciones),
	Posiciones = [espalda, montada].        % el nombre solo no identifica la técnica

test(facu_solo_sabe_golpes_a_la_cabeza) :-
	forall(sabe(facu, Tecnica), Tecnica = golpe(_, cabeza)).
```

Notar el uso de `[fail]` para testear que algo **no** se cumple (el equivalente lógico de "esperar que falle"), y de `forall/2` para verificar una condición sobre todos los individuos que cumplen otra.

---

## 9. Detalle a consensuar con el grupo

La categoría del ippon-seoi-nage la escribimos `tackle`, siguiendo la enumeración de la consigna ("tackle, proyección, barrida"), aunque en el dato de Vale aparece escrito "tacle". Es una decisión de **normalización**: el mismo concepto tiene que estar escrito siempre igual en la base, o después no se puede agrupar ni comparar.

---
---

# Punto 2 — `esFielASuDisciplina` (Integrante 1)

> ⚠️ La consigna original de este punto no la tengo textual: lo que sigue está reconstruido a partir del código. Confirmá el enunciado con quien lo hizo.

**Qué hace**: un luchador es fiel a su disciplina si **todas** las técnicas que sabe corresponden al tipo (piso o parado) de **alguna** disciplina que entrena.

```prolog
categoriaDeTecnica(piso(_, _, _), piso).
categoriaDeTecnica(golpe(_, _, _), parado).
categoriaDeTecnica(lance(_, _, _), parado).

esFielASuDisciplina(Persona) :-
	luchador(Persona),
	forall(
		(sabe(Persona, Tecnica), categoriaDeTecnica(Tecnica, Tipo)),
		(entrena(Persona, Disciplina), disciplina(Disciplina, Tipo))
	).
```

**Resultado**: `[mora, vale, mati, facu, fede]`. El único que no es fiel es **Tomi**, porque sabe el mataleón (técnica de piso) pero sólo entrena Muay Thai y Boxeo, las dos de parado.

## Lo interesante de este punto

**`categoriaDeTecnica/2` es un traductor entre dos vocabularios.** Las técnicas se agrupan en tres functores (`piso`, `golpe`, `lance`) pero las disciplinas sólo en dos tipos (`piso`, `parado`). Este predicado mapea uno al otro: los golpes **y** los lances son de parado. Sin él no habría forma de comparar una técnica con una disciplina.

Fijate que es **polimorfismo por pattern matching** otra vez: tres cláusulas, una por functor, sin preguntar por el tipo.

**El `forall/2` tiene condición y consecuente compuestos**, los dos entre paréntesis:

```prolog
forall( (sabe(P, T), categoriaDeTecnica(T, Tipo)),      % para toda técnica suya y su tipo…
        (entrena(P, D), disciplina(D, Tipo)) )          % …entrena alguna disciplina de ese tipo
```

El consecuente es **existencial**: no pide que *todas* sus disciplinas sean de ese tipo, sino que **exista alguna**. Eso sale gratis, porque un objetivo común dentro del `forall` se satisface con encontrar un solo camino.

**Fede es fiel por verdad vacua**: no sabe ninguna técnica, así que no hay ninguna que incumpla. Es el mismo criterio que después se usó en el punto 8 (`esTemible`), y por eso los dos son coherentes entre sí.

---
---

# Punto 3 — `esPicante` (Integrante 2)

> ⚠️ Consigna reconstruida del código.

**Qué hace**: entrena **alguna** disciplina de parado, **ninguna** de piso, y **todas** sus técnicas son golpes a la cabeza.

```prolog
esPicante(Luchador) :-
	luchador(Luchador),
	entrena(Luchador, Disciplina),
	disciplina(Disciplina, parado),
	not((entrena(Luchador, OtraDisciplina),
	     disciplina(OtraDisciplina, piso))),
	forall(sabe(Luchador, Tecnica), Tecnica = golpe(_, cabeza, _)).
```

**Resultado**: sólo **Facu**.

## Las tres formas de cuantificar, en un solo predicado

Este punto es un buen resumen de todo:

| Línea | Cuantificador | Herramienta |
|---|---|---|
| `entrena(L, D), disciplina(D, parado)` | **existe alguna** de parado | objetivo común |
| `not((entrena(L, O), disciplina(O, piso)))` | **ninguna** de piso | `not/1` sobre la existencia |
| `forall(sabe(L, T), T = golpe(_, cabeza, _))` | **todas** las técnicas | `forall/2` |

Y notá el `forall` del final: la condición se verifica **por unificación pura**, sin llamar a ningún predicado. `T = golpe(_, cabeza, _)` es verdadero sólo si la técnica es un golpe a la cabeza.

## ⚠️ Este predicado devuelve soluciones repetidas

```prolog
?- findall(L, esPicante(L), Ls).
Ls = [facu, facu, facu].
```

**Por qué**: `entrena(Luchador, Disciplina), disciplina(Disciplina, parado)` se satisface **una vez por cada disciplina de parado** que entrene. Facu entrena Muay Thai, Boxeo y Kickboxing — las tres de parado — así que hay tres caminos distintos que demuestran lo mismo.

Es exactamente el problema del **Paso 5 del punto 4**, con otra cara. El test del grupo lo deja documentado (`Luchadores = [facu, facu, facu]`), pero conviene saber que **es un defecto**, no una característica: un predicado inversible no debería repetir individuos.

**Cómo se arregla**, con lo mismo que se usó en el punto 4:

```prolog
esPicante(Luchador) :-
	luchador(Luchador),
	entrenaAlgunaDisciplinaDeParado(Luchador),   % ya existe, del punto 4
	not(entrenaAlgunaDisciplinaDePiso(Luchador)),
	forall(sabe(Luchador, Tecnica), Tecnica = golpe(_, cabeza, _)).
```

…pero `entrenaAlgunaDisciplinaDeParado/1` también se satisface tres veces para Facu, así que habría que envolverla en un `distinct` como en el Paso 6. **Si les preguntan por esto en el coloquio, la respuesta honesta es reconocerlo y explicar de dónde sale la multiplicidad.**

> 🚩 **Decisión: NO lo corregimos.** Es código de otro integrante y su test pasa tal como está. Lo dejo anotado sólo para tenerlo identificado, no para cambiarlo. Si sale en el coloquio, alcanza con reconocer de dónde viene la multiplicidad y decir cómo se resolvería.

---
---

# Punto 4 — `esNavajaSuiza` (MI PUNTO INDIVIDUAL)

**Consigna**: se satisface para un luchador **graduado con cinturón negro**, que **sabe un mismo sometimiento desde distintas posiciones**, que **no sabe ningún golpe** y que **no entrena ninguna disciplina de parado**. Debe ser **inversible**.

> Esta sección está escrita como una **construcción paso a paso**: empieza de cero, muestra los intentos que fallan y por qué se corrigen. Si querés sólo el código terminado, está en el **Paso 9**.

---

## Paso 0: partir la consigna en pedazos

Lo primero, antes de escribir una línea, es cortar el enunciado en condiciones independientes. La consigna tiene una `y` entre cada una, así que son **cuatro condiciones que se tienen que cumplir todas**:

| # | Texto de la consigna | Qué necesito de la base |
|---|---|---|
| 1 | "graduado con cinturón negro" | los cinturones |
| 2 | "sabe un mismo sometimiento desde distintas posiciones" | las técnicas de piso |
| 3 | "no sabe ningún golpe" | las técnicas de golpe |
| 4 | "no entrena ninguna disciplina de parado" | las disciplinas y su tipo |

En lógico, "**y**" se escribe con **coma**. Así que el esqueleto ya está definido: un predicado con cuatro objetivos separados por comas.

---

## Paso 1: el esqueleto — un predicado por condición

Podríamos escribir todo junto, en una sola regla gigante:

```prolog
% lo que NO conviene hacer
esNavajaSuiza(Luchador) :-
	cinturon(Luchador, _, negro),
	sabe(Luchador, piso(Sometimiento, UnaPosicion)),
	sabe(Luchador, piso(Sometimiento, OtraPosicion)),
	UnaPosicion \= OtraPosicion,
	not(sabe(Luchador, golpe(_, _))),
	not((entrena(Luchador, Disciplina), disciplina(Disciplina, parado))).
```

Funciona, pero es **ilegible**: hay que leer las 6 líneas y reconstruir mentalmente cuál corresponde a cada condición del enunciado. En vez de eso, le damos **un nombre a cada condición**:

```prolog
esNavajaSuiza(Luchador) :-
	esCinturonNegro(Luchador),
	sabeUnSometimientoDesdeDistintasPosiciones(Luchador),
	not(sabeAlgunGolpe(Luchador)),
	not(entrenaAlgunaDisciplinaDeParado(Luchador)).
```

Ahora el predicado **se lee igual que la consigna**. Eso es **delegación**: cada predicado auxiliar tiene una única responsabilidad, y `esNavajaSuiza/1` sólo coordina. Ventajas concretas:

- Se entiende sin leer las definiciones de abajo.
- Si una condición está mal, sabés exactamente qué predicado testear.
- Los auxiliares son **reutilizables** en otros puntos del TP (`esCinturonNegro/1` seguro sirve para algo más).

Ahora hay que definir los cuatro auxiliares, uno por uno.

---

## Paso 2: condición 1 — "graduado con cinturón negro"

**Primer intento:**

```prolog
esCinturonNegro(Luchador) :-
	cinturonObtenido(Luchador, _, negro).
```

**¿Por qué está mal?** Porque `cinturonObtenido/3` son los hechos, los cinturones **literalmente otorgados**:

```prolog
cinturonObtenido(mora, jiuJitsu, violeta).
cinturonObtenido(vale, judo, marron).
cinturonObtenido(mati, sambo, negro).
```

Con ese predicado, Mora no cuenta como cinturón negro. Pero la consigna dice explícitamente: *"Mora, por la equivalencia de cinturones, cuenta como cinturón negro en Judo"*. **Ese ejemplo está puesto justamente para verificar esto.** Hay que usar el predicado derivado, el que incluye la regla de equivalencia que escribimos en el punto 1:

```prolog
cinturon(Luchador, Disciplina, Color) :-
	cinturonObtenido(Luchador, Disciplina, Color).

cinturon(Luchador, judo, negro) :-
	cinturonObtenido(Luchador, jiuJitsu, violeta).
```

**Segundo intento:**

```prolog
esCinturonNegro(Luchador) :-
	cinturon(Luchador, _, negro).
```

Ahora sí. `esCinturonNegro(mora)` da verdadero (por la equivalencia) y `esCinturonNegro(mati)` también (por el sambo).

> ⚠️ **Ojo con el falso positivo de "me da bien"**: Mora **no** es navaja suiza, pero falla por las **otras tres** condiciones, no por ésta. Si hubieras usado `cinturonObtenido/3`, el resultado final igual habría dado `mati` — pero el predicado estaría mal. Llegar al resultado correcto por el camino equivocado no cuenta.

El guión bajo `_` es la **variable anónima**: significa "no me importa el valor, sólo que exista alguno". Acá: "en alguna disciplina, cualquiera, tiene negro".

*(A esta condición todavía le falta un ajuste, que aparece en el Paso 7.)*

---

## Paso 3: condición 2 — "un mismo sometimiento desde distintas posiciones"

Primero hay que traducir la palabra **sometimiento** a nuestro modelo. En el punto 1 dijimos que las técnicas de piso son las que se identifican **por la posición desde la que se ejecutan**:

```prolog
piso(Nombre, Posicion)
```

Y los sometimientos son exactamente ésos: `triangulo`, `armlock`, `mataleon`, `llaveDeBrazo`, etc. Así que el patrón que necesito es `piso(Sometimiento, Posicion)`.

Ahora, ¿cómo digo "**el mismo** sometimiento desde **distintas** posiciones"? Necesito **dos** técnicas del luchador que cumplan dos cosas a la vez:

```prolog
sometimientoDesdeDistintasPosiciones(Luchador) :-
	sabe(Luchador, piso(Sometimiento, UnaPosicion, _)),
	sabe(Luchador, piso(Sometimiento, OtraPosicion, _)),
	UnaPosicion \= OtraPosicion.
```

Leelo despacio, porque acá está toda la lógica del punto:

- **La variable `Sometimiento` se repite** en los dos objetivos `sabe`. En lógico, **repetir una variable obliga a que sea el mismo valor**. Eso es lo que expresa "el **mismo** sometimiento".
- **Las variables `UnaPosicion` y `OtraPosicion` son distintas entre sí**, así que pueden tomar valores diferentes... pero por sí solo eso no alcanza, porque *también* podrían tomar el mismo valor. Por eso hace falta la tercera línea.
- **`UnaPosicion \= OtraPosicion`** es lo que fuerza que sean **distintas**. `\=` significa "no unifica con".

**Ojo con `Sometimiento`**: es una variable **interna** de la regla, no un argumento. No sale hacia afuera — sólo sirve para obligar a que las dos líneas `sabe` hablen de la misma técnica. El predicado tiene **un solo argumento**, el luchador, y responde sí o no.

Probémoslo con Mati:

```prolog
sabe(mati, piso(llaveDeBrazo, espalda)).
sabe(mati, piso(llaveDeBrazo, montada)).
```

- `Sometimiento = llaveDeBrazo`, `UnaPosicion = espalda`, `OtraPosicion = montada` → `espalda \= montada` ✅

Y con Mora:

```prolog
sabe(mora, piso(triangulo, guardia)).
sabe(mora, piso(armlock, montada)).
```

- `Sometimiento = triangulo`: la única posición es `guardia`, así que `UnaPosicion` y `OtraPosicion` valen las dos `guardia` → `guardia \= guardia` ❌
- `Sometimiento = armlock`: idem con `montada` ❌

Mora falla, que es lo que la consigna dice. 

> 🔗 **Conexión con el punto 1**: esta condición **sólo se puede escribir** porque en la base el nombre y la posición son dos argumentos separados dentro del functor `piso/2`. Si hubiéramos modelado `llaveDeBrazoDesdeLaEspalda` como un átomo pegado, no habría ninguna parte que comparar y el punto sería imposible. Es la mejor justificación de aquella decisión de modelado.

---

## Paso 4: condiciones 3 y 4 — las negativas

La consigna dice "**no** sabe **ningún** golpe". En lógico **no se puede escribir "ninguno" directamente**: la negación siempre se aplica sobre algo que se pueda demostrar. La forma de decir "ninguno" es **negar la existencia de alguno**:

> "no sabe ningún golpe" = **no es cierto que** "sabe algún golpe"

Entonces se define el predicado **en positivo** y se lo niega desde afuera:

```prolog
sabeAlgunGolpe(Luchador) :-
	sabe(Luchador, golpe(_, _)).
```

Y en `esNavajaSuiza`: `not(sabeAlgunGolpe(Luchador))`.

Fijate que `golpe(_, _)` usa el **pattern matching sobre el functor**: unifica sólo con los golpes y descarta solo `piso/2` y `lance/2`, sin que tengamos que escribir ninguna condición extra. Es gratis, lo hace el motor al unificar.

La cuarta condición es igual, pero necesita dos objetivos porque el tipo de disciplina está en otro predicado:

```prolog
entrenaAlgunaDisciplinaDeParado(Luchador) :-
	entrena(Luchador, Disciplina),
	disciplina(Disciplina, parado).
```

Leído: "existe una disciplina que el luchador entrena **y** que es de parado". Negado desde afuera: "no entrena ninguna de parado".

**Caso curioso — Fede**: como no entrena nada y no sabe nada, las dos condiciones negativas le dan **verdadero**. Negar algo que no existe es verdadero (**universo cerrado**). Fede falla, pero por las condiciones positivas.

---

## Paso 5: lo probamos... y aparece el primer problema

Con las cuatro condiciones armadas, la primera versión completa sería:

```prolog
% PRIMERA VERSIÓN (tiene un bug)
esNavajaSuiza(Luchador) :-
	esCinturonNegro(Luchador),
	sometimientoDesdeDistintasPosiciones(Luchador),
	not(sabeAlgunGolpe(Luchador)),
	not(entrenaAlgunaDisciplinaDeParado(Luchador)).
```

Probamos si funciona en modo chequeo:

```prolog
?- esNavajaSuiza(mati).
true.        % ✅ bien
```

Pero probamos en **modo inversible**, que es lo que la consigna exige:

```prolog
?- findall(L, esNavajaSuiza(L), Ls).
Ls = [mati, mati].    % ❌ ¡Mati aparece DOS VECES!
```

**¿Por qué?** Porque estos dos hechos:

```prolog
sabe(mati, piso(llaveDeBrazo, espalda)).
sabe(mati, piso(llaveDeBrazo, montada)).
```

hacen que `sometimientoDesdeDistintasPosiciones(mati)` se satisfaga de **dos formas distintas**:

| | `UnaPosicion` | `OtraPosicion` | `\=` |
|---|---|---|---|
| 1ª solución | `espalda` | `montada` | ✅ |
| 2ª solución | `montada` | `espalda` | ✅ |

Son el **mismo par**, mirado en los dos órdenes. Prolog no tiene forma de saber que a nosotros nos da igual: para él son dos soluciones legítimas, y por lo tanto devuelve a Mati dos veces.

> 🎓 **Regla**: un predicado inversible **no debe devolver soluciones repetidas**. Si al hacerle `findall` aparece un individuo más de una vez, el predicado está mal, aunque en modo chequeo dé bien.

---

## Paso 6: `distinct/2` para colapsar el no determinismo

### La duda razonable: "pero si yo no necesito la lista"

Es la primera reacción al ver este problema, y es una buena objeción:

> *"A mí no me importa **cuáles** son los sometimientos repetidos. Sólo necesito saber si **existe alguno**. ¿No alcanza con consultar `sometimientoDesdeDistintasPosiciones(Luchador)` y ya?"*

**Conceptualmente tenés razón.** El problema es otro, y es la idea más importante de este paso:

> 🎓 En Prolog un objetivo **no devuelve `true`**. Un objetivo **se satisface tantas veces como formas haya de demostrarlo**. No hay "verdadero o falso": hay "cuántos caminos distintos llevan a que esto sea cierto".

`sometimientoDesdeDistintasPosiciones(mati)` es cierto por **dos caminos** (el par de posiciones en los dos órdenes). No importa que a nosotros nos alcance con uno: Prolog los recorre los dos, y cada uno hace que `esNavajaSuiza(mati)` tenga éxito. Por eso salen dos Mati.

Entonces lo que necesitamos no es *obtener* nada, sino **colapsar esos varios caminos en uno solo**.

### La solución: `distinct/2`

```prolog
sabeUnSometimientoDesdeDistintasPosiciones(Luchador) :-
	distinct(Luchador, sometimientoDesdeDistintasPosiciones(Luchador)).
```

**Cómo se lee `distinct/2`**:

```prolog
distinct( Testigo , Objetivo )
```

Deja pasar el objetivo **una sola vez por cada valor distinto del testigo**. Se acuerda de los testigos que ya dejó pasar y descarta las soluciones que repiten uno.

Acá el testigo es `Luchador`, que **ya viene ligado** a un luchador concreto (lo ligó `esCinturonNegro/1`, el objetivo anterior). Entonces:

1. Primer camino (espalda / montada) → testigo `mati` → **pasa**, y queda registrado.
2. Segundo camino (montada / espalda) → testigo `mati` otra vez → **se descarta**.

Resultado: un solo éxito.

```prolog
?- findall(L, esNavajaSuiza(L), Ls).
Ls = [mati].    % ✅
```

Y si el luchador no tiene ningún sometimiento repetido —como Mora—, el objetivo de adentro **falla** y `distinct` no tiene nada que dejar pasar, así que el predicado falla. Que es lo que queremos.

### ⚠️ El testigo tiene que ser el luchador

Es el único detalle delicado. Para usar el sometimiento como testigo habría que devolverle el segundo argumento al predicado auxiliar:

```prolog
% ❌ frágil
sometimientoDesdeDistintasPosiciones(Luchador, Sometimiento) :- ...

distinct(Sometimiento, sometimientoDesdeDistintasPosiciones(Luchador, Sometimiento))
```

Andaría hoy —Mati repite un solo sometimiento, la llave de brazo— pero **volvería a duplicar** si alguien repitiera **dos** sometimientos distintos: `distinct` dejaría pasar uno por cada sometimiento distinto, o sea dos veces.

Comprobado agregando un segundo sometimiento repetido a Mati: con testigo `Sometimiento` da 2 éxitos; con testigo `Luchador`, 1.

> 🎓 **Criterio**: el testigo de `distinct` tiene que ser **aquello que no querés que se repita en la respuesta**. Como `esNavajaSuiza/1` devuelve luchadores, el testigo es el luchador.

### Las otras formas de resolverlo (todas probadas)

| | Cómo se escribe | Resultado | |
|---|---|---|---|
| **A** | `distinct(Luchador, sometimiento...(Luchador))` | `[mati]` | ✅ **la que usamos** |
| **B** | `sometimientoDesdeDistintasPosiciones(Luchador)` (directo) | `[mati, mati]` | ❌ el bug |
| **C** | `findall(S, sometimiento...(L, S), Ss), Ss \= []` | `[mati]` | ✅ estándar ISO |
| **D** | `setof(S, sometimiento...(L, S), _)` | `[mati]` | ✅ estándar ISO |
| **E** | cambiar `\=` por `@<` entre las posiciones | `[mati]` | ⚠️ frágil |

**C — `findall/3`** era la versión anterior de este resumen. Junta en una lista todos los sometimientos repetidos y pregunta si la lista quedó vacía. Funciona porque **`findall` siempre se satisface exactamente una vez**, pase lo que pase adentro. La lista se descarta: sólo se mira si está vacía. Su ventaja es que `findall/3` es **estándar ISO** y ya se usa en el resto del TP; su desventaja, que hacen falta cuatro líneas y una variable (`Sometimiento`) como argumento del predicado auxiliar, sólo para que el `findall` tenga qué juntar.

**D — `setof/3`** es la más corta de las estándar: como `setof` **falla** cuando no hay soluciones (a diferencia de `findall`, que devuelve `[]`), se ahorra la línea del `\= []`.

**E — `UnaPosicion @< OtraPosicion`** en lugar de `\=`. Del par `espalda`/`montada` acepta sólo el orden alfabético, así que lo cuenta una vez. Con estos datos anda, pero es **frágil**: si un luchador supiera el mismo sometimiento desde **tres** posiciones, saldrían 3 pares y volverían los duplicados (comprobado: `[mati, mati, mati]`). Sólo arregla el caso simétrico.

### ⚠️ Lo que hay que saber defender de `distinct`

`distinct/2` vive en `library(solution_sequences)`. SWI la carga sola, así que no hace falta importar nada — pero:

- **No es estándar ISO**: no existe en otros Prolog.
- **Es extra-lógica**: por dentro usa una tabla mutable para acordarse de los testigos que ya devolvió. No es una relación, es un filtro con memoria.

Si en el coloquio te preguntan por qué usaste algo no declarativo, la respuesta es que resuelve exactamente el problema (evitar soluciones repetidas) en una línea, y que la alternativa declarativa equivalente es el `findall` de la opción C, que hace lo mismo en cuatro líneas y obligando a que el auxiliar tenga un argumento de salida que después no se usa.

---

## Paso 7: ¿hace falta colapsar también el cinturón?

Ésta es una pregunta que **hay que hacerse cada vez**, y la respuesta acá es **no**. Vale la pena entender por qué, porque saber cuándo *no* agregar maquinaria es tan importante como saber cuándo agregarla.

La duda aparece porque al correr los tests SWI avisa:

```
Warning: test esNavajaSuiza:mati_es_navaja_suiza: Test succeeded with choicepoint
```

"Sucedió con choicepoint" quiere decir: *el predicado dio verdadero, pero quedó un camino sin explorar*. Ese camino es `cinturon/3`, que tiene **dos cláusulas** (el cinturón obtenido y el equivalente), así que Prolog deja abierta la posibilidad de que la segunda también sirva.

**Pero un choicepoint no es lo mismo que una solución repetida.** Que quede un camino abierto no significa que ese camino vaya a dar verdadero. Con los datos del TP, la consulta inversible da:

```prolog
?- findall(L, esNavajaSuiza(L), Ls).
Ls = [mati].     % ✅ una sola vez
```

Comparalo con el Paso 5, donde el problema era **real y comprobable**: ahí sí salía `[mati, mati]`.

**¿Cuándo se rompería?** Sólo si un luchador tuviera **dos cinturones negros a la vez** — por ejemplo negro en Sambo *y* violeta en Jiu Jitsu (que vale como negro en Judo). En ese caso `cinturon(L, _, negro)` daría dos soluciones y el luchador saldría repetido. Comprobado agregando ese dato a la base: da `[mati, mati]`.

**La decisión**: la base del TP está dada y fija, y **ningún luchador tiene dos cinturones negros**. Envolver la condición en un `distinct` para cubrir un caso que no existe agrega ruido y hace el predicado menos legible sin ganar nada. Así que se deja simple:

```prolog
esCinturonNegro(Luchador) :-
	cinturon(Luchador, _, negro).
```

> 🎓 **Criterio**: colapsá el no determinismo **cuando existe de verdad con los datos que tenés**, no por las dudas. En el sometimiento existe (Paso 5, comprobado). Acá no.

**Diferencia entre los dos casos, en una línea:**

| | ¿Duplica con los datos del TP? | ¿Necesita `distinct`? |
|---|---|---|
| Sometimiento desde distintas posiciones | **Sí**, `[mati, mati]` | ✅ sí |
| Cinturón negro | No, `[mati]` | ❌ no |

---

## Paso 8: el orden de los objetivos y la inversibilidad

**Inversible** significa que tiene que andar en los dos modos:

```prolog
?- esNavajaSuiza(mati).    % modo chequeo:    ¿Mati cumple?
?- esNavajaSuiza(Quien).   % modo generación: ¿quiénes cumplen?
```

Para que ande el segundo modo hace falta un **generador**: algún objetivo que **proponga candidatos** recorriendo una base de hechos. Los demás objetivos sólo **filtran**.

En nuestro predicado el generador es `cinturon/3`, adentro de `esCinturonNegro/1`, que es el primer objetivo. Es inversible porque recorre los hechos de `cinturonObtenido/3`, así que con la variable libre propone luchadores:

```prolog
?- findall(L, esCinturonNegro(L), Ls).
Ls = [mati, mora].    % mati por el sambo, mora por la equivalencia
```

Sólo propone dos candidatos, y está bien: cualquiera que no tenga cinturón negro iba a fallar en esa condición de todas formas. El resto de los objetivos filtra a Mora y queda Mati.

> 🎓 **Patrón general**: **generador + condiciones**. Primero un objetivo que genera individuos, después los que filtran. El generador tiene que ser un objetivo **positivo e inversible**.

**Y por eso el orden importa.** Los `not/1` tienen que ir **después** del generador, nunca antes:

```prolog
% ❌ INCORRECTO
esNavajaSuiza(Luchador) :-
	not(sabeAlgunGolpe(Luchador)),      % Luchador viene SIN ligar
	esCinturonNegro(Luchador),
	...
```

Si `Luchador` está sin ligar, `not(sabeAlgunGolpe(Luchador))` no pregunta "¿Mati sabe algún golpe?" sino "**¿existe algún luchador que sepa algún golpe?**". Como sí existe (Mora, Tomi, Vale, Facu), el objetivo interno da verdadero, el `not` da **falso**, y el predicado entero **falla siempre, para todos**.

> 🎓 **Regla**: `not/1` es un **filtro**, no un generador. Las variables tienen que llegar ligadas. Los objetivos negativos van al final.

---

## Paso 9: la versión final

```prolog
esNavajaSuiza(Luchador) :-
	esCinturonNegro(Luchador),
	sabeUnSometimientoDesdeDistintasPosiciones(Luchador),
	not(sabeAlgunGolpe(Luchador)),
	not(entrenaAlgunaDisciplinaDeParado(Luchador)).

% Usa cinturon/3 (no cinturonObtenido/3) para contemplar la equivalencia.
% Es además el generador que da la inversibilidad.
esCinturonNegro(Luchador) :-
	cinturon(Luchador, _, negro).

% Los sometimientos son las técnicas de piso: la variable Sometimiento
% repetida obliga a que sea el mismo, y el \= a que las posiciones difieran.
sometimientoDesdeDistintasPosiciones(Luchador) :-
	sabe(Luchador, piso(Sometimiento, UnaPosicion)),
	sabe(Luchador, piso(Sometimiento, OtraPosicion)),
	UnaPosicion \= OtraPosicion.

% distinct/2 deja pasar el objetivo una sola vez por cada valor distinto del
% testigo. Como el testigo es el luchador, no se repiten soluciones.
sabeUnSometimientoDesdeDistintasPosiciones(Luchador) :-
	distinct(Luchador, sometimientoDesdeDistintasPosiciones(Luchador)).

sabeAlgunGolpe(Luchador) :-
	sabe(Luchador, golpe(_, _)).

entrenaAlgunaDisciplinaDeParado(Luchador) :-
	entrena(Luchador, Disciplina),
	disciplina(Disciplina, parado).
```

---

## Paso 10: traza de `?- esNavajaSuiza(Quien)`

Seguir esto una vez a mano es lo que termina de hacer clic.

El **generador** es `esCinturonNegro/1`, o sea `cinturon(Quien, _, negro)`. Como `cinturon/3` tiene dos cláusulas, propone candidatos en este orden: primero los de la 1ª cláusula (cinturones obtenidos), después los de la 2ª (equivalencias). O sea, **sólo dos candidatos**: `mati` y `mora`. Tomi, Vale, Facu y Fede ni siquiera se proponen, porque ninguno tiene un cinturón negro que ofrecer.

**1) `Quien = mati`** — por la 1ª cláusula: `cinturonObtenido(mati, sambo, negro)` ✅

- `sabeUnSometimiento...(mati)`:
  el objetivo de adentro tiene éxito dos veces (los dos órdenes del par de posiciones), pero `distinct` deja pasar sólo el primero, porque el testigo `mati` ya quedó registrado ✅
- `not(sabeAlgunGolpe(mati))`:
  Mati no tiene ningún término `golpe/2`, así que el objetivo interno **falla** → el `not` **da verdadero** ✅
- `not(entrenaAlgunaDisciplinaDeParado(mati))`:
  entrena Sambo, y `disciplina(sambo, piso)` — no es de parado, el interno **falla** → el `not` **da verdadero** ✅
- **✅ Primera solución: `Quien = mati`**

**2) `Quien = mora`** — por la 2ª cláusula: `cinturonObtenido(mora, jiuJitsu, violeta)`, que le da el negro de Judo por equivalencia ✅

- `sabeUnSometimiento...(mora)`:
  se intenta con `triangulo` (sólo desde `guardia`) y con `armlock` (sólo desde `montada`); en los dos casos `UnaPosicion` y `OtraPosicion` toman el mismo valor y el `\=` falla, así que `distinct` no tiene nada que dejar pasar ❌
- **Falla acá.** Ni siquiera se evalúan las condiciones negativas (que Mora también incumpliría: sabe golpes y entrena Muay Thai).

**Resultado**: `Quien = mati`, una sola vez.

> Notá que Mora **sí pasó** la condición del cinturón negro. Ése es el punto exacto donde se ve que usar `cinturon/3` y no `cinturonObtenido/3` cambia el recorrido: con `cinturonObtenido/3`, Mora ni siquiera habría sido propuesta como candidata.

---

## Quién cumple qué

| Luchador | ¿Negro? | ¿Sometimiento repetido? | ¿Sin golpes? | ¿Sin parado? | |
|---|---|---|---|---|---|
| **Mati** | ✅ sambo | ✅ llaveDeBrazo (espalda + montada) | ✅ | ✅ sólo Sambo | **✅ navaja suiza** |
| Mora | ✅ judo, **por equivalencia** | ❌ triángulo y armlock son distintos | ❌ rodillazo, patada | ❌ Muay Thai | ❌ |
| Vale | ❌ marrón | ❌ | ❌ | ❌ | ❌ |
| Tomi | ❌ sin cinturón | ❌ | ❌ | ❌ | ❌ |
| Facu | ❌ sin cinturón | ❌ | ❌ | ❌ | ❌ |
| Fede | ❌ sin cinturón | ❌ | ✅ universo cerrado | ✅ universo cerrado | ❌ |

---

## Preguntas probables del coloquio

**"¿Por qué usaste `cinturon/3` y no `cinturonObtenido/3`?"**
Porque la consigna establece que el violeta de Jiu Jitsu equivale al negro de Judo, y `cinturon/3` es el predicado que contempla esa equivalencia. `cinturonObtenido/3` tiene sólo los cinturones literalmente otorgados. El ejemplo de Mora en el enunciado está puesto justamente para verificar esto.

**"¿Por qué el `distinct` si igual te daba bien el resultado?"**
En modo chequeo daba bien, pero en modo inversible `findall(L, esNavajaSuiza(L), Ls)` devolvía `[mati, mati]`: la condición del sometimiento se satisface dos veces, una por cada orden del par de posiciones. Un predicado inversible no debe devolver soluciones repetidas. `distinct/2` deja pasar una sola solución por cada valor distinto del testigo, y así colapsa ese no determinismo.

**"Si no necesitás la lista, ¿por qué no consultás el predicado directamente?"**
Porque en Prolog un objetivo no devuelve `true`: se satisface **tantas veces como formas haya de demostrarlo**. `sometimientoDesdeDistintasPosiciones(mati)` es cierto por dos caminos (el par de posiciones en los dos órdenes), y cada camino hace que `esNavajaSuiza(mati)` tenga éxito. `distinct/2` deja pasar el primero y descarta el segundo, porque los dos tienen el mismo testigo.

**"¿Por qué el testigo del `distinct` es el luchador y no el sometimiento?"**
Porque el testigo tiene que ser aquello que no quiero que se repita en la respuesta, y `esNavajaSuiza/1` devuelve luchadores. Con el sometimiento como testigo andaría hoy, pero volvería a duplicar si alguien repitiera dos sometimientos distintos.

**"¿Y por qué no usaste `distinct` también en `esCinturonNegro/1`?"**
Porque ahí el no determinismo **no existe con estos datos**: ningún luchador tiene dos cinturones negros, así que `cinturon(L, _, negro)` da una sola solución por luchador y la consulta inversible devuelve `[mati]`. Sólo se rompería si alguien tuviera, por ejemplo, negro en Sambo *y* violeta en Jiu Jitsu. Agregar la maquinaria por las dudas haría el predicado menos legible sin arreglar nada. El criterio es: colapsar cuando el problema es real y comprobable, no preventivo. (SWI igual avisa "succeeded with choicepoint", pero un choicepoint **no es** una solución repetida.)

**"¿Qué pasa si movés los `not` al principio?"**
El predicado falla siempre, para todos. Con la variable sin ligar, `not(sabeAlgunGolpe(Luchador))` pregunta si **nadie** sabe golpes; como alguien sabe, da falso. `not/1` necesita las variables ligadas: es filtro, no generador.

**"¿Cómo garantizás la inversibilidad?"**
Con el patrón generador + condiciones. El primer objetivo, `esCinturonNegro/1`, es inversible porque `cinturon/3` recorre los hechos de `cinturonObtenido/3`: con la variable libre propone `mati` y `mora`. El resto de los objetivos sólo filtra.

**"¿Qué es un sometimiento en tu modelo?"**
Una técnica de piso, o sea un término `piso(Nombre, Posicion)`: las que se identifican por la posición desde la que se ejecutan, que es justamente lo que la condición necesita comparar.

**"¿`distinct` no es estándar, no?"**
No, vive en `library(solution_sequences)` de SWI y es extra-lógica: por dentro usa una tabla mutable para acordarse de los testigos que ya dejó pasar. La alternativa declarativa y estándar es `findall(S, sometimiento...(L, S), Ss), Ss \= []`, que hace lo mismo en cuatro líneas y obliga a que el auxiliar tenga un argumento de salida que después no se usa.

**"¿Podrías haberlo escrito con `forall/2`?"**
Sí, las condiciones negativas son equivalentes: `not(sabeAlgunGolpe(L))` es lo mismo que `forall(sabe(L, T), not(T = golpe(_, _)))`. Elegimos `not/1` sobre un auxiliar en positivo porque es más directo y porque el auxiliar queda reutilizable.

---

## Tests

```prolog
test(mati_es_navaja_suiza) :-
	esNavajaSuiza(mati).

test(mora_no_es_navaja_suiza, [fail]) :-   % tiene el negro por equivalencia,
	esNavajaSuiza(mora).                   % pero falla las otras tres

% Verifica inversibilidad Y ausencia de repetidos en una sola línea
test(es_inversible_y_sin_repetidos) :-
	findall(Luchador, esNavajaSuiza(Luchador), Luchadores),
	Luchadores = [mati].
```

El último es **el test más valioso del punto**: si sacaras el `distinct` de `sabeUnSometimientoDesdeDistintasPosiciones/1`, ese test falla (daría `[mati, mati]`) mientras que `test(mati_es_navaja_suiza)` **seguiría pasando**. Testear en modo inversible es lo único que detecta las soluciones repetidas.

---
---

# Punto 5 — Efectividad y `esContundente`

**Consigna**: extender cada técnica con un atributo nuevo — los sometimientos con los **segundos** que tardan, los golpes con su **potencia** (1 a 10), los lances con los **puntos** que otorgan. Y definir `esContundente`: un sometimiento lo es si tarda 5 segundos o menos; un golpe, si es a la cabeza y tiene potencia 8 o más; un lance, si otorga 8 puntos o más. **No necesita ser inversible.**

## 1. La decisión de modelado: ¿dónde vive la efectividad?

Hay dos formas de agregar el dato, y antes de elegir hay que mirar **los datos del enunciado**:

```prolog
% El mataleón de Tomi tarda 8 segundos.
% El mataleón de Mati tarda 5 segundos.

% La patada de Mora: potencia 9.
% La patada de Facu: potencia 10.
% La patada de Vale: potencia 7.

% La llave de brazo de Mati: 2 segundos desde la espalda, 4 desde la montada.
```

**Ése es el dato que decide todo.** La efectividad **no es una propiedad del nombre de la técnica**: el mismo mataleón tarda distinto según quién lo aplique, y la misma llave de brazo tarda distinto según desde dónde. La efectividad es una propiedad de **la técnica de ese luchador desde esa posición**.

Eso descarta de entrada la opción más tentadora:

```prolog
% ❌ INCORRECTO: una tabla de efectividad por técnica
efectividad(mataleon, 8).
efectividad(mataleon, 5).   % ¿de quién es cuál?
```

Esto daría el mismo valor para el mataleón de todos, o peor, dos valores sin forma de saber a quién corresponde cada uno. **Es información mal ubicada.**

La opción correcta es **extender el functor**, que es además lo que la consigna pide literalmente ("extender cada técnica con un nuevo atributo"):

```prolog
piso(Nombre, Posicion, Segundos)     % ← antes piso/2, ahora piso/3
golpe(Nombre, Zona, Potencia)
lance(Nombre, Categoria, Puntos)
```

```prolog
sabe(tomi, piso(mataleon, espalda, 8)).
sabe(mati, piso(mataleon, espalda, 5)).   % mismo nombre, distinta efectividad ✅
```

**¿Y una tercera opción, `efectividad(Luchador, Tecnica, Valor)`?** Funcionaría, pero obliga a **repetir el término de la técnica en dos predicados distintos** (`sabe/2` y `efectividad/3`). Si uno se modifica y el otro no, la base queda **inconsistente** y no hay nada que lo impida. Con el atributo adentro del functor, el dato viaja pegado a la técnica y esa inconsistencia es imposible.

> 🎓 **Concepto**: un atributo va donde está el individuo al que pertenece. Si el valor depende de la combinación luchador + técnica + posición, tiene que vivir dentro del término que representa esa combinación.

## 2. El costo de esta decisión: se rompe la aridad

Cambiar `piso/2` por `piso/3` **rompe todo lo que hacía pattern matching sobre esos functores**. Hubo que tocar:

```prolog
% en el punto 4
sabe(Luchador, piso(Sometimiento, UnaPosicion))   →  piso(Sometimiento, UnaPosicion, _)
sabe(Luchador, golpe(_, _))                       →  golpe(_, _, _)

% en los tests del punto 1
sabe(mati, piso(llaveDeBrazo, Posicion))          →  piso(llaveDeBrazo, Posicion, _)
Tecnica = golpe(_, cabeza)                        →  golpe(_, cabeza, _)
```

**Lo peligroso es cómo fallan**: no dan error. `golpe(_, _)` simplemente **no unifica** con `golpe(patada, cabeza, 9)`, así que el predicado devuelve `false` en silencio. Sin tests, este cambio rompe el punto 4 sin que nadie se entere.

> 🎓 **Concepto**: *acoplamiento*. La forma de un functor es una decisión de diseño que **todos sus usuarios comparten**. Cambiarla impacta en cadena. Es el argumento honesto a favor de la tabla separada — pero pierde contra el problema del mataleón, que es un error de modelado y no sólo una molestia.

**Moraleja práctica**: después de cambiar la aridad de un functor, correr los tests. Los 30 tests fueron lo que confirmó que el punto 4 seguía dando `[mati]`.

## 3. `esContundente`: una cláusula por tipo de técnica

```prolog
esContundente(piso(_, _, Segundos)) :-
	Segundos =< 5.

esContundente(golpe(_, cabeza, Potencia)) :-
	Potencia >= 8.

esContundente(lance(_, _, Puntos)) :-
	Puntos >= 8.
```

**Éste es el momento en que se cobra la decisión del punto 1.** No hay que preguntar de qué tipo es la técnica, ni hacer un `if`, ni tener tres predicados con nombres distintos: **la unificación elige sola la cláusula que corresponde**. Si le pasás un `lance/3`, las dos primeras cláusulas ni se intentan porque el functor no unifica.

> 🎓 **Concepto**: *polimorfismo por pattern matching*. Varias cláusulas del mismo predicado, cada una unificando con un functor distinto. El predicado se comporta distinto según el tipo del individuo, sin preguntar por el tipo.

**Detalle fino que conviene saber defender**: en la cláusula del golpe, la condición "a la cabeza" está **en la cabeza de la regla**, no en el cuerpo. La alternativa sería:

```prolog
% equivalente, pero peor
esContundente(golpe(_, Zona, Potencia)) :-
	Zona = cabeza,
	Potencia >= 8.
```

Las dos dan el mismo resultado, pero la primera es mejor: la condición se resuelve **al unificar**, antes de entrar al cuerpo, y se lee de un vistazo que esa cláusula habla sólo de golpes a la cabeza. Poner en la cabeza todo lo que se pueda expresar con unificación es buen estilo en lógico.

**Las tres cláusulas son un "o" lógico**: una técnica es contundente si cumple la primera **o** la segunda **o** la tercera. Como los functores son distintos, nunca se pisan entre sí.

## 4. Por qué este punto no necesita ser inversible

La consigna lo aclara, y hay una razón concreta:

```prolog
?- esContundente(piso(triangulo, guardia, 4)).
true.

?- esContundente(Tecnica).
ERROR: Arguments are not sufficiently instantiated
```

Los operadores aritméticos de comparación (`=<`, `>=`) **exigen que sus dos lados estén ligados**: no pueden "inventar" un número que cumpla la condición. Con la técnica sin ligar, `Segundos` no tiene valor y Prolog no puede evaluar la comparación.

Por eso `esContundente/1` se usa siempre como **condición**, nunca como generador:

```prolog
?- sabe(mora, Tecnica), esContundente(Tecnica).   % ✅ sabe/2 genera, esContundente filtra
Tecnica = piso(triangulo, guardia, 4) ;
Tecnica = golpe(patada, cabeza, 9).
```

Es el mismo patrón **generador + condiciones** del punto 4: alguien tiene que ligar las variables antes.

## 5. ⚠️ Dos problemas con los datos de la consigna

Hay que llevarlos consensuados al grupo:

**a) El rodillazo de Mora aparece con dos potencias distintas.**

- En la lista de datos: *"El rodillazo de Mora (cuerpo, **potencia 6**)"*
- En los ejemplos: *"el rodillazo (cuerpo, **potencia 9**) no [es contundente], por no ser a la cabeza"*

Cargamos **6**, porque la lista de datos es la sección que define la base; el ejemplo parece estar ilustrando que *aunque tuviera 9*, seguiría sin ser contundente por la zona. **Con cualquiera de los dos valores el resultado es el mismo** (no es contundente), así que no cambia nada — pero conviene decirlo.

**b) La potencia de la patada de Mora no está en la lista de datos.** Sólo aparece en los ejemplos: *"La patada de Mora (cabeza, **potencia 9**)"*. La tomamos de ahí, porque es la única fuente.

## 6. Preguntas probables del coloquio

**"¿Por qué extendiste el functor en vez de hacer un predicado `efectividad` aparte?"**
Porque la efectividad no depende del nombre de la técnica sino del luchador y la posición: el mataleón de Tomi tarda 8 segundos y el de Mati 5, y la llave de brazo de Mati tarda 2 desde la espalda y 4 desde la montada. Una tabla por nombre de técnica sería directamente incorrecta. Una tabla `efectividad(Luchador, Tecnica, Valor)` funcionaría, pero repetiría el término de la técnica en dos predicados, con riesgo de inconsistencia.

**"¿Qué se rompió al cambiar la aridad?"**
Todo lo que unificaba con `piso/2` o `golpe/2`: dos predicados del punto 4 y dos tests del punto 1. Lo peligroso es que no dan error — simplemente dejan de unificar y devuelven `false` en silencio. Los tests fueron lo que lo detectó.

**"¿Por qué `esContundente` tiene tres cláusulas y no un solo predicado con condiciones?"**
Porque cada tipo de técnica tiene una regla distinta y se identifica por un functor distinto. Con tres cláusulas, la unificación elige sola cuál aplica: es polimorfismo por pattern matching, sin preguntar por el tipo.

**"¿Por qué `cabeza` va en la cabeza de la regla y no en el cuerpo?"**
Porque es una condición que se puede expresar por unificación. Resolverla al unificar es más eficiente (descarta antes de entrar al cuerpo) y más legible: se ve de un vistazo que esa cláusula habla sólo de golpes a la cabeza.

**"¿Por qué no necesita ser inversible?"**
Porque usa comparaciones aritméticas (`=<`, `>=`), que exigen sus dos lados ligados. Con la técnica sin ligar tira error de instanciación. Se usa siempre como condición después de un generador, típicamente `sabe/2`.

## 7. Tests

Los tests cubren los tres tipos, los **bordes de cada escala** (5 segundos, potencia 8, 8 puntos — que según la consigna **sí** son contundentes) y los dos motivos distintos por los que un golpe puede no serlo:

```prolog
% no es contundente por la ZONA, aunque la potencia alcanzara
test(rodillazo_de_mora_no_es_contundente, [fail]) :-
	esContundente(golpe(rodillazo, cuerpo, 9)).

% no es contundente por la POTENCIA, aunque sea a la cabeza
test(gancho_de_tomi_no_es_contundente, [fail]) :-
	esContundente(golpe(gancho, cabeza, 6)).

% el mataleón de Tomi tarda 8 y el de Mati 5: la efectividad depende
% del luchador, no sólo del nombre de la técnica
test(el_mataleon_de_tomi_no_es_contundente, [fail]) :-
	sabe(tomi, piso(mataleon, Posicion, Segundos)),
	esContundente(piso(mataleon, Posicion, Segundos)).

test(todas_las_tecnicas_de_mati_son_contundentes) :-
	forall(sabe(mati, Tecnica), esContundente(Tecnica)).
```

El anteúltimo es el que **documenta la decisión de modelado**: si la efectividad estuviera en una tabla por nombre de técnica, ese test no podría ni escribirse.

---
---

# Punto 6 — `tieneUnPuntoFlojo` (Integrante 1)

> ⚠️ Consigna reconstruida del código.

**Qué hace**: el luchador sabe **algún** sometimiento que **no** es contundente.

```prolog
tieneUnPuntoFlojo(Luchador) :-
	sabe(Luchador, piso(Nombre, Posicion, TiempoDeEjecucion)),
	not(esContundente(piso(Nombre, Posicion, TiempoDeEjecucion))).
```

**Resultado**: `[mora, tomi]` — Mora por el armlock (7 segundos) y Tomi por el mataleón (8 segundos).

## Lo interesante

**Es existencial, no universal.** "Tiene *un* punto flojo" = "existe al menos un sometimiento suyo que no es contundente". Por eso alcanza con un objetivo común y **no** hace falta `forall/2`. Es el contrapunto exacto de `esTemible/1`:

| | `esTemible` | `tieneUnPuntoFlojo` |
|---|---|---|
| Cuantificador | **todas** son contundentes | **existe una** que no lo es |
| Alcance | todas las técnicas | sólo los sometimientos (`piso/3`) |
| Herramienta | `forall/2` | objetivo común + `not/1` |

**No usa `luchador/1` como generador**, y acá está bien: `sabe/2` alcanza, porque a alguien que no sabe ningún sometimiento no le puede faltar ninguno. Fede queda afuera, que es lo correcto.

**El filtro por tipo va en el patrón**: al escribir `sabe(Luchador, piso(...))` en vez de `sabe(Luchador, Tecnica)`, sólo unifica con sometimientos. Los golpes y lances se descartan por unificación, sin condición extra.

## ⚠️ Riesgo latente de soluciones repetidas

Con los datos actuales no pasa, porque Mora y Tomi tienen **un solo** sometimiento flojo cada uno. Pero si alguien tuviera **dos**, aparecería dos veces: `sabe/2` se satisface una vez por cada sometimiento que cumpla.

Es la misma situación que discutimos en `esCinturonNegro/1` (Paso 7 del punto 4): el no determinismo existe en la estructura pero no se manifiesta con estos datos. Si el grupo quiere blindarlo, es el `distinct` del Paso 6.

> 🚩 **Decisión: NO lo corregimos.** Es código de otro integrante y con los datos del TP no falla. Además, siguiendo el criterio del Paso 7 del punto 4, no corresponde agregar maquinaria por un no determinismo que hoy no se manifiesta.

---
---

# Punto 7 — `esCertero` (Integrante 2)

> ⚠️ Consigna reconstruida del código.

**Qué hace**: **todas** las técnicas **contundentes** que sabe el luchador son de parado.

```prolog
esCertero(Luchador) :-
	luchador(Luchador),
	forall((sabe(Luchador, Tecnica), esContundente(Tecnica)),
	       categoriaDeTecnica(Tecnica, parado)).
```

**Resultado**: `[tomi, facu, fede]`.

| Luchador | Sus técnicas contundentes | ¿Todas de parado? |
|---|---|---|
| **Tomi** | codazo (cabeza, 9) | ✅ |
| **Facu** | directo, gancho y patada, los tres a la cabeza | ✅ |
| **Fede** | ninguna | ✅ verdad vacua |
| Mora | triángulo (piso) y patada | ❌ el triángulo es de piso |
| Vale | estrangulación (piso), cruzado, o-soto-gari | ❌ |
| Mati | sus seis sometimientos | ❌ todos de piso |

## Lo interesante: un `forall` con la condición filtrada

Compará las dos condiciones:

```prolog
% punto 8 — esTemible
forall( sabe(L, T) ,                        Cumple )   % TODAS las técnicas

% punto 7 — esCertero
forall( (sabe(L, T), esContundente(T)) ,    Cumple )   % sólo las CONTUNDENTES
```

La condición del `forall` no tiene por qué ser un objetivo suelto: puede ser una **conjunción que filtre** el universo sobre el que cuantificás. Acá se lee "para toda técnica que sea suya **y además** contundente…".

> 🎓 **Concepto**: en `forall(Condicion, Cumple)`, `Condicion` define **sobre qué individuos** cuantificás y `Cumple` es **lo que se les exige**. Filtrar dentro de la condición es la forma de decir "todos los X que además son Y".

**Ojo con la verdad vacua acá**: alcanza con no tener **ninguna** técnica contundente para ser certero. Tomi es certero teniendo un solo golpe contundente, y Fede lo es sin tener ninguno. No es un error, es lo que dice el enunciado — pero es el tipo de caso que conviene tener pensado antes de que lo pregunten.

---
---

# Punto 8 — `esTemible` (MI SEGUNDO PUNTO INDIVIDUAL)

**Consigna**: se satisface para un luchador tal que **todas** las técnicas que sabe son contundentes. Debe ser **inversible**.

## 1. La solución

```prolog
esTemible(Luchador) :-
	luchador(Luchador),
	forall(sabe(Luchador, Tecnica), esContundente(Tecnica)).
```

Tres líneas. Todo el trabajo pesado ya estaba hecho: `esContundente/1` viene del punto 5, y `luchador/1` del punto 1. Éste es el punto donde se cobra haber modelado bien antes.

## 2. El concepto central: "todas" se dice con `forall/2`

Lo primero que hay que entender es por qué **no alcanza** con lo obvio:

```prolog
% ❌ INCORRECTO
esTemible(Luchador) :-
	luchador(Luchador),
	sabe(Luchador, Tecnica),
	esContundente(Tecnica).
```

Eso **no** dice "todas sus técnicas son contundentes", dice "**sabe alguna** técnica contundente". Son cosas distintas, y con estos datos se nota enseguida:

```prolog
?- findall(L, esTemibleIncorrecto(L), Ls).
Ls = [mora, mora, tomi, vale, vale, vale, mati, mati, mati, mati, mati, mati, facu, facu, facu].
```

**Mora aparece**, porque su triángulo (4 segundos) es contundente — aunque su armlock no lo sea. Y cada luchador aparece una vez por cada técnica contundente que sabe.

El motivo de fondo es el mismo del punto 4: **un objetivo se satisface una vez por cada forma de demostrarlo**. `sabe(L, T)` recorre las técnicas una por una; nunca "mira todas juntas". Para eso hace falta `forall/2`:

```prolog
forall( Condicion , Cumple )
```

Se lee: *"para todos los casos en que se cumple **Condición**, también se cumple **Cumple**"*. Acá: "para todas las técnicas que sabe el luchador, esa técnica es contundente".

> 🎓 **Concepto**: `forall/2` es la herramienta para cuantificar universalmente. Internamente equivale a `not((Condicion, not(Cumple)))` — "no existe ningún caso que cumpla la condición y falle" — pero se lee infinitamente mejor.

## 3. Por qué el generador es `luchador/1` y no `sabe/2`

Ésta es la decisión importante del punto. Podría parecer que basta con `sabe/2` para proponer candidatos, pero se rompe de **dos** formas distintas:

```prolog
% ❌ INCORRECTO
esTemible(Luchador) :-
	sabe(Luchador, _),
	forall(sabe(Luchador, Tecnica), esContundente(Tecnica)).
```

```prolog
?- findall(L, esTemibleConSabe(L), Ls).
Ls = [mati, mati, mati, mati, mati, mati, facu, facu, facu].
```

1. **Soluciones repetidas**: Mati sale 6 veces (una por técnica que sabe) y Facu 3 veces. El `sabe(Luchador, _)` del principio no es un generador de luchadores, es un generador de **pares luchador-técnica**.
2. **Se pierde a Fede**: como no sabe ninguna técnica, `sabe(fede, _)` falla y Fede nunca llega a evaluarse.

Con `luchador/1` como generador, cada luchador se propone **exactamente una vez** y todos llegan a evaluarse:

```prolog
?- findall(L, esTemible(L), Ls).
Ls = [mati, facu, fede].
```

> 🎓 **Acá sí se ve para qué sirvió declarar `luchador/1` en el punto 1.** (En el punto 4 el generador había terminado siendo `cinturon/3`; acá `luchador/1` es imprescindible, porque es el único predicado que menciona a Fede.)

## 4. ⚠️ El caso borde: Fede es temible por verdad vacua

**`forall/2` con la condición vacía da verdadero.** Como Fede no sabe ninguna técnica, no tiene **ninguna** que incumpla, así que "todas sus técnicas son contundentes" es **verdadero** — de forma vacía, pero verdadero.

```prolog
?- esTemible(fede).
true.
```

Verificado en la base: Fede tiene **cero** técnicas no contundentes, igual que Mati y Facu.

**¿Es lo que corresponde?** La consigna dice que Mati y Facu son temibles y que Mora, Tomi y Vale no — y **no menciona a Fede**, así que hay que decidirlo. Nos quedamos con que **sí es temible**, por dos razones:

1. Es la **traducción literal** del enunciado: "todas las técnicas que sabe son contundentes" es cierto para quien no sabe ninguna.
2. **Consistencia con el punto 2**: `esFielASuDisciplina/1` usa el mismo `forall/2` y su test espera `[mora, vale, mati, facu, fede]`, o sea que el grupo ya aceptó la verdad vacua para Fede en el mismo TP. Sería incoherente resolverlo distinto acá.

**Si el grupo o la cátedra prefiere la otra lectura** (que alguien que no sabe nada no puede ser temible), el cambio es de una línea:

```prolog
esTemible(Luchador) :-
	luchador(Luchador),
	sabe(Luchador, _),                     % ← exige saber al menos una técnica
	forall(sabe(Luchador, Tecnica), esContundente(Tecnica)).
```

⚠️ Pero ojo: agregar ese `sabe(Luchador, _)` **reintroduce las soluciones repetidas** (Mati 6 veces). Habría que envolverlo con `distinct`, igual que en el punto 4. **La versión con verdad vacua es más simple justamente porque no necesita nada de eso.**

## 5. Por qué acá no hizo falta colapsar nada (a diferencia del punto 4)

Es la comparación más útil entre los dos puntos individuales:

| | Punto 4 — `esNavajaSuiza` | Punto 8 — `esTemible` |
|---|---|---|
| Condición central | "existe **un** sometimiento repetido" | "**todas** las técnicas son contundentes" |
| Cuantificador | existencial | universal |
| Herramienta | `distinct/2` | `forall/2` |
| ¿Cuántas veces se satisface? | el objetivo crudo, **muchas** → hubo que colapsarlo | `forall/2` ya se satisface **a lo sumo una vez** |

> 🎓 **La clave**: `forall/2` **ya es determinístico**. No propaga un camino por cada caso que revisa: los revisa todos internamente y se satisface una sola vez (o falla). Por eso no necesita ningún envoltorio. El `distinct` del punto 4 hizo falta porque ahí la condición era **existencial** y estaba escrita como un objetivo crudo, que sí se satisface una vez por cada demostración.

**Regla práctica**: si la condición es "todos", usás `forall/2` y no tenés problema de duplicados. Si es "existe alguno", el objetivo crudo puede duplicar y hay que fijarse.

## 6. Quién es temible y por qué

| Luchador | Técnicas no contundentes | ¿Temible? |
|---|---|---|
| **Mati** | ninguna (sus 6 sometimientos tardan ≤ 5 s) | ✅ |
| **Facu** | ninguna (sus 3 golpes son a la cabeza con potencia ≥ 8) | ✅ |
| **Fede** | ninguna — **porque no sabe ninguna técnica** | ✅ verdad vacua |
| Mora | armlock (7 s) y rodillazo (al cuerpo) | ❌ |
| Tomi | gancho (potencia 6), directo (al cuerpo), mataleón (8 s) | ❌ |
| Vale | patada (a la pierna) e ippon-seoi-nage (6 puntos) | ❌ |

Notá que los tres que fallan lo hacen por motivos distintos —tiempo, zona y puntos—, o sea que el test ejercita las tres cláusulas de `esContundente/1`.

## 7. Preguntas probables del coloquio

**"¿Por qué `forall/2` y no consultar `sabe` y `esContundente` juntos?"**
Porque `sabe(L, T), esContundente(T)` dice "sabe **alguna** técnica contundente", no "todas lo son". Con estos datos, esa versión incluye a Mora, cuyo triángulo es contundente pero cuyo armlock no. Un objetivo se satisface una vez por cada forma de demostrarlo; para cuantificar sobre **todos** los casos hace falta `forall/2`.

**"¿Por qué el generador es `luchador/1` y no `sabe/2`?"**
Por dos motivos. Primero, `sabe/2` genera pares luchador-técnica, así que Mati saldría 6 veces y Facu 3. Segundo, se perdería a Fede, que no aparece en ningún hecho de `sabe/2`. `luchador/1` propone cada luchador exactamente una vez y a todos.

**"¿Fede es temible? ¿No es raro?"**
Sí es temible, por **verdad vacua**: `forall/2` con la condición vacía da verdadero, y Fede no tiene ninguna técnica que incumpla. Es la lectura literal del enunciado, y además es consistente con el punto 2, donde el mismo `forall/2` hace que Fede sea fiel a su disciplina. Si se quisiera excluirlo habría que exigir explícitamente que sepa al menos una técnica — pero eso reintroduce soluciones repetidas y obliga a envolverlo en un `distinct`.

**"¿Por qué acá no usaste `distinct` como en el punto 4?"**
Porque `forall/2` ya es determinístico: revisa todos los casos internamente y se satisface a lo sumo una vez. El `distinct` del punto 4 hizo falta porque ahí la condición era existencial y estaba escrita como un objetivo crudo, que se satisface una vez por cada demostración.

**"¿Qué pasa si `esContundente/1` recibe una técnica sin ligar?"**
No pasa, porque `forall/2` la liga: la condición `sabe(Luchador, Tecnica)` se evalúa primero y le da un valor concreto a `Tecnica` antes de llamar a `esContundente/1`. Es el mismo patrón generador + condición de siempre, pero adentro del `forall`.

## 8. Tests

```prolog
test(mati_es_temible)  :- esTemible(mati).   % 6 sometimientos ≤ 5 s
test(facu_es_temible)  :- esTemible(facu).   % 3 golpes a la cabeza, potencia ≥ 8

test(mora_no_es_temible, [fail]) :- esTemible(mora).   % armlock 7 s
test(tomi_no_es_temible, [fail]) :- esTemible(tomi).   % gancho potencia 6
test(vale_no_es_temible, [fail]) :- esTemible(vale).   % patada a la pierna

% Verdad vacua: no sabe ninguna técnica que incumpla
test(fede_es_temible) :- esTemible(fede).

% Inversibilidad Y ausencia de repetidos
test(es_inversible_y_sin_repetidos) :-
	findall(Luchador, esTemible(Luchador), Luchadores),
	Luchadores = [mati, facu, fede].
```

El último test es el que detecta las dos versiones incorrectas de una sola vez: con `sabe/2` como generador daría `[mati, mati, mati, mati, mati, mati, facu, facu, facu]`, y sin `forall/2` daría una lista con Mora, Tomi y Vale adentro.

---
---

# Punto 9 — `comboPosible` (todo el grupo)

**Consigna**: relacionar un luchador, una cantidad máxima de técnicas y un combo que pueda realizar. Un combo está formado **únicamente por técnicas que sabe el luchador** y puede contener **entre cero y la cantidad máxima**. Debe ser **inversible para el luchador y el combo**.

## 0. Antes que nada: ¿qué es un combo?

**Un combo es una lista de técnicas.** Nada más que eso.

```prolog
Combo = [golpe(directo, cabeza, 9), golpe(patada, cabeza, 10)]
```

Ése es un combo de Facu con dos técnicas: su directo y su patada.

> ⚠️ **La confusión más fácil de tener**: en `comboPosible(facu, 2, Combo)`, el **2 no es la cantidad de técnicas que sabe Facu** (sabe 3). Es el **máximo de técnicas que puede tener cada combo**.

Facu sabe directo, gancho y patada. Con máximo 2, sus combos posibles son **7**:

| | Combo |
|---|---|
| con 0 técnicas | `[]` |
| con 1 técnica | `[directo]`, `[gancho]`, `[patada]` |
| con 2 técnicas | `[directo, gancho]`, `[directo, patada]`, `[gancho, patada]` |

1 + 3 + 3 = 7. No aparece `[directo, gancho, patada]` porque tiene 3 y el máximo era 2. Por eso con máximo 3 son 8.

*(De acá en adelante escribo `directo` en lugar de `golpe(directo, cabeza, 9)` para que las trazas entren en pantalla. Es lo mismo: `subconjunto/2` mueve los términos enteros y no le importa qué tienen adentro.)*

## 1. Deducir si un combo es un conjunto o una secuencia

El enunciado no lo dice con palabras, pero **lo dice con los números**:

| | Enunciado | Si fueran subconjuntos | Si fueran permutaciones |
|---|---|---|---|
| Facu (3 técnicas), hasta 1 | **4** | 4 ✅ | 4 |
| Facu, hasta 2 | **7** | 7 ✅ | 10 ❌ |
| Facu, hasta 3 | **8** | 8 ✅ | 16 ❌ |
| Mati (6 técnicas), hasta 2 | **22** | 22 ✅ | 37 ❌ |

**Un combo es un subconjunto**: el orden no importa y no se repiten técnicas. Si el orden importara, `[directo, gancho]` y `[gancho, directo]` serían dos combos distintos y Facu tendría 10, no 7.

> 🎓 Cuando el enunciado da ejemplos numéricos, **son la especificación**. Contrastarlos con las dos interpretaciones posibles resuelve la ambigüedad sin adivinar.

## 2. La solución

```prolog
comboPosible(Luchador, CantidadMaxima, Combo) :-
	luchador(Luchador),
	findall(Tecnica, sabe(Luchador, Tecnica), Tecnicas),
	subconjunto(Tecnicas, Combo),
	length(Combo, Cantidad),
	Cantidad =< CantidadMaxima.

subconjunto([], []).

subconjunto([Tecnica | RestoTecnicas], [Tecnica | Combo]) :-
	subconjunto(RestoTecnicas, Combo).

subconjunto([_ | RestoTecnicas], Combo) :-
	subconjunto(RestoTecnicas, Combo).
```

## 3. Las tres cláusulas de `subconjunto/2`

Primero, la sintaxis de listas, que es de donde sale todo:

```prolog
[directo, gancho, patada]  =  [Cabeza | Cola]

     Cabeza = directo        Cola = [gancho, patada]
```

La barra `|` **parte una lista en su primer elemento y el resto**.

**Cláusula 1 — el caso base**
```prolog
subconjunto([], []).
```
El único subconjunto de la lista vacía es la lista vacía. Es donde frena la recursión, y **no tiene cuerpo**.

**Cláusulas 2 y 3 — la técnica entra o no entra**

Puestas una debajo de la otra se ve que **la única diferencia está en el segundo argumento**:

```prolog
subconjunto([Tecnica | RestoTecnicas], [Tecnica | Combo]) :- subconjunto(RestoTecnicas, Combo).
subconjunto([_       | RestoTecnicas],            Combo ) :- subconjunto(RestoTecnicas, Combo).
                                       ↑↑↑↑↑↑↑↑↑
                                  la única diferencia
```

En **las dos** la primera lista se achica igual (pierde su primer elemento). Lo que cambia es el combo:

- **Cláusula 2**: el combo **crece** — le pega `Tecnica` adelante. → la técnica **entra**
- **Cláusula 3**: el combo **pasa igual** — no le agrega nada. → la técnica **no entra**

Como cada técnica tiene 2 opciones, salen **2ⁿ** subconjuntos. Con las 3 de Facu, 8. Con las 6 de Mati, 64.

## 4. Paso a paso de la primera propuesta

Ésta es la parte que hay que poder contar en el pizarrón. Consulta:

```prolog
?- subconjunto([directo, gancho, patada], TuCombo).
```

- La **primera** lista viene **llena** (la armó el `findall`).
- `TuCombo` viene **vacía**: es una variable libre. **Eso es lo que hay que construir.**

Algo importante: **cada vez que se usa una cláusula, sus variables son nuevas**. Por eso las numero `Combo1`, `Combo2`, `Combo3`.

### Nivel 1 — `subconjunto([directo, gancho, patada], TuCombo)`

Prolog prueba la cláusula 1 (`subconjunto([], [])`): no sirve, la lista no está vacía. Pasa a la cláusula 2.

```
primer argumento:  [directo, gancho, patada]  con  [Tecnica | RestoTecnicas]
                   Tecnica = directo     RestoTecnicas = [gancho, patada]

segundo argumento: TuCombo  con  [Tecnica | Combo1]
                   TuCombo = [directo | Combo1]
```
👉 Ya se sabe que el combo **empieza con directo**. `Combo1` queda pendiente.

El cuerpo pide: `subconjunto([gancho, patada], Combo1)`

### Nivel 2 — `subconjunto([gancho, patada], Combo1)`

```
Tecnica = gancho     RestoTecnicas = [patada]
Combo1  = [gancho | Combo2]
```
El cuerpo pide: `subconjunto([patada], Combo2)`

### Nivel 3 — `subconjunto([patada], Combo2)`

```
Tecnica = patada     RestoTecnicas = []
Combo2  = [patada | Combo3]
```
El cuerpo pide: `subconjunto([], Combo3)`

### Nivel 4 — `subconjunto([], Combo3)`

Acá la cláusula 2 **ya no unifica**: `[]` no se puede partir en cabeza y cola. Entra la cláusula 1:

```
Combo3 = []
```
Como la cláusula 1 no tiene cuerpo, **la recursión termina**.

### Y ahora se arma la respuesta, de abajo hacia arriba

Cada nivel tenía un pendiente. Ahora se completan:

```
Combo3  = []
Combo2  = [patada | Combo3]  = [patada]
Combo1  = [gancho | Combo2]  = [gancho, patada]
TuCombo = [directo | Combo1] = [directo, gancho, patada]
```

> 🎓 **La idea**: al **bajar**, la primera lista se vacía y el combo se va armando dejando un hueco en cada nivel. Al llegar a `[]`, el último hueco se llena. Al **volver**, los huecos se completan hacia arriba y aparece la lista entera.

## 5. Cómo vuelve atrás: las marcas

`[directo, gancho, patada]` mide 3, así que falla el `Cantidad =< 2`. ¿Cómo sabe Prolog qué deshacer?

**No lo decide: va anotando marcas mientras avanza.** Cada vez que aplica una cláusula **habiendo otras sin probar**, deja una:

| Nivel | Lista que recibió | Cláusula usada | ¿Le quedan otras? | |
|---|---|---|---|---|
| 1 | `[directo, gancho, patada]` | 2 | sí, la 3 | 📌 marca |
| 2 | `[gancho, patada]` | 2 | sí, la 3 | 📌 marca |
| 3 | `[patada]` | 2 | sí, la 3 | 📌 marca |
| 4 | `[]` | 1 | **no** | sin marca |

El nivel 4 **no deja marca**: para `[]` las cláusulas 2 y 3 ni siquiera unifican, las dos piden una lista con al menos un elemento.

**Al fallar, Prolog va a la marca más reciente** — la del nivel 3 — y hace tres cosas:

1. **Borra todo lo que se hizo después de esa marca**:
   ```
   Combo3 = []                  ← se borra
   Combo2 = [patada | Combo3]   ← se borra
   ```
   Lo de **antes** queda intacto: `TuCombo = [directo | Combo1]` y `Combo1 = [gancho | Combo2]`.

2. **Reintenta ese nivel con la cláusula 3**:
   ```
   [patada] con [_ | RestoTecnicas]  →  RestoTecnicas = []
   Combo2 pasa igual, sin agregar nada
   cuerpo: subconjunto([], Combo2)  →  cláusula 1  →  Combo2 = []
   ```

3. **Rearma hacia arriba**:
   ```
   Combo2  = []
   Combo1  = [gancho]
   TuCombo = [directo, gancho]        ← segunda propuesta, mide 2, pasa ✅
   ```

> 🎓 Las marcas funcionan como una **pila**: se apilan al bajar y al fallar se saca **la de más arriba**, la más reciente. Por eso siempre cambia primero la **última** técnica.

## 6. Las 8 propuestas, una por una

Cláusula 2 = entra, cláusula 3 = no entra. Prolog **siempre prueba primero la 2**.

**1ª → `[directo, gancho, patada]`** ❌ mide 3, falla el `=< 2`
```
directo → cláusula 2 (entra)
gancho  → cláusula 2 (entra)
patada  → cláusula 2 (entra)
[]      → cláusula 1 (fin)
```

**2ª → `[directo, gancho]`** ✅ *1ª respuesta* — se deshace la última marca
```
directo → 2
gancho  → 2
patada  → 3 (no entra)     ← lo único que cambió
```

**3ª → `[directo, patada]`** ✅ — `patada` agotó sus dos cláusulas, sube a `gancho`; `patada` reinicia
```
directo → 2
gancho  → 3                ← cambió
patada  → 2                ← reiniciada
```

**4ª → `[directo]`** ✅
```
directo → 2
gancho  → 3
patada  → 3                ← cambió
```

**5ª → `[gancho, patada]`** ✅ — `patada` y `gancho` agotadas, sube hasta `directo`
```
directo → 3                ← cambió
gancho  → 2                ← reiniciada
patada  → 2                ← reiniciada
```

**6ª → `[gancho]`** ✅
```
directo → 3
gancho  → 2
patada  → 3                ← cambió
```

**7ª → `[patada]`** ✅
```
directo → 3
gancho  → 3                ← cambió
patada  → 2                ← reiniciada
```

**8ª → `[]`** ✅
```
directo → 3
gancho  → 3
patada  → 3                ← cambió
```

**Se acabó**: no queda ninguna marca sin usar. `subconjunto` falla y `comboPosible` no tiene más para dar. **7 respuestas.**

> 🎓 **La regla que se repite en los 8 pasos**: se deshace la **marca más reciente** que todavía tenga la cláusula 3 sin probar, y todo lo que venga **después** de esa técnica **reinicia por la cláusula 2**.
>
> Por eso `patada` cambia en casi todos los pasos (es la última) y `directo` cambia una sola vez (es la primera). Es un cuentakilómetros: la rueda de la derecha gira todo el tiempo.

## 7. Generar y probar: dónde entra `CantidadMaxima`

Fijate el orden de los objetivos:

```prolog
luchador(facu),                 % 1 sola solución  → nunca se vuelve acá
findall(..., Tecnicas),         % 1 sola solución  → nunca se vuelve acá
subconjunto(Tecnicas, Combo),   % 8 soluciones     ← el único con alternativas
length(Combo, Cantidad),        % depende de Combo → se rehace
Cantidad =< CantidadMaxima.     % depende de Cantidad → se rehace
```

Al fallar, Prolog vuelve **al objetivo más reciente que todavía tenga alternativas**. Los dos primeros ya dieron su única respuesta, así que el único al que puede volver es `subconjunto`. Y cuando éste entrega un combo nuevo, todo lo que viene después hay que rehacerlo con el valor nuevo.

**`CantidadMaxima` no frena la generación: filtra después.** `subconjunto/2` genera **todos** los subconjuntos, sin enterarse del límite. Con Mati:

| | |
|---|---|
| `subconjunto/2` genera | **64** combos (2⁶) |
| el `=< 2` deja pasar | **22** |
| el `=< 2` descarta | **42** |

> 🎓 Es el patrón **generar y probar**: uno propone, otro filtra. El mismo generador + condiciones de todos los puntos.

Es ineficiente a propósito: con 6 técnicas son 64 caminos, nada. Con 30 técnicas serían mil millones y habría que meter el límite adentro de la generación. Para este TP no hace falta.

## 8. ⚠️ Volver atrás no pasa sólo cuando algo falla

De las 8 propuestas, **una sola falla**. Las otras 7 pasan. Entonces, ¿por qué sigue buscando después de una que pasó?

Porque hay **dos motivos** para volver atrás:

1. Algo **falló** (el combo de 3 técnicas).
2. **Alguien pidió otra respuesta.**

Ese "alguien" es quien haga la consulta. Si preguntás con `findall`, pide soluciones hasta que no queden más — por eso ves las 7. Si pedís una sola, **frena en la primera que pasa**:

```prolog
?- once(comboPosible(facu, 2, C)).
C = [golpe(directo,cabeza,9), golpe(gancho,cabeza,8)].
   % se detuvo acá, los otros 6 ni se generaron
```

> 🎓 `comboPosible` **no devuelve 7 combos**. Devuelve **un combo por vez**, y da 7 en total si le seguís pidiendo. En la consola de SWI, cada `;` que apretás le pide el siguiente, hasta que dice `false`.

## 9. Por qué el `findall` acá sí es para obtener la lista

Ojo con esto, porque es distinto de los puntos 4 y 8:

```prolog
findall(Tecnica, sabe(Luchador, Tecnica), Tecnicas),
```

Acá **sí necesitamos la lista de verdad**, no la usamos para colapsar nada. `subconjunto/2` trabaja sobre una **lista**, y `sabe/2` entrega las técnicas de a una por backtracking. El `findall` es el puente entre las dos formas de tener la información.

| Punto | Herramienta | Para qué |
|---|---|---|
| 4 — `esNavajaSuiza` | `distinct/2` | **colapsar** muchas soluciones en una; no se obtiene ningún dato |
| 9 — `comboPosible` | `findall/3` | **obtener** la lista, que es el dato que necesita la recursión |

## 10. Inversibilidad

La consigna pide que sea inversible **para el luchador y para el combo** — notá que **no** pide que lo sea para la cantidad máxima, y hay una razón: `Cantidad =< CantidadMaxima` es una comparación aritmética y exige que los dos lados estén ligados.

Los modos que funcionan:

```prolog
?- comboPosible(facu, 2, Combo).        % ✅ genera los 7 combos
?- comboPosible(Quien, 0, []).          % ✅ genera los 6 luchadores
?- comboPosible(Quien, 1, [golpe(patada, cabeza, 10)]).
Quien = facu.                            % ✅ deduce de quién es el combo
?- comboPosible(facu, 2, [golpe(directo, cabeza, 9), golpe(patada, cabeza, 10)]).
true.                                    % ✅ modo chequeo
```

**Quién genera qué**: `luchador/1` propone los luchadores y `subconjunto/2` propone los combos. Las dos son inversibles, y por eso el predicado anda con cualquiera de los dos argumentos libre — o con los dos a la vez.

Con el combo ligado, `subconjunto/2` funciona como **verificador** en vez de generador: recorre el árbol buscando el camino que produce ese combo exacto. Deja un *choicepoint* (queda árbol sin explorar), pero da **una sola solución** — por eso el test en modo chequeo va marcado `[nondet]`.

## 11. El caso borde: el combo vacío

**El combo vacío siempre es posible**, para cualquier luchador y cualquier máximo (incluso 0), porque el caso base `subconjunto([], [])` se alcanza recorriendo toda la lista por el camino de "no entra".

Eso hace que:
- `comboPosible(mati, 0, Combo)` tenga una única solución: `Combo = []` ✅ (lo dice la consigna)
- **Fede**, que no sabe ninguna técnica, tenga exactamente un combo posible: el vacío. No es un caso especial que haya que programar aparte — sale solo del caso base.

## 12. Preguntas probables del coloquio

**"¿Cómo supiste que un combo es un subconjunto y no una secuencia ordenada?"**
Por los números del enunciado. Facu sabe 3 técnicas y tiene 7 combos de hasta 2. Con subconjuntos da 7; con permutaciones daría 10. Lo mismo con los 22 de Mati, que como permutaciones serían 37.

**"¿Qué es un combo?"**
Una lista de técnicas del luchador. Puede tener desde cero técnicas (la lista vacía) hasta la cantidad máxima que se pida. Ojo que el segundo argumento de `comboPosible/3` es ese máximo, no la cantidad de técnicas que sabe el luchador.

**"¿Cómo sabe Prolog qué deshacer cuando el `=<` falla?"**
No lo decide: va dejando una marca cada vez que aplica una cláusula habiendo otras sin probar. Al fallar vuelve a la marca **más reciente**, borra las ligaduras hechas después de ella y prueba ahí la cláusula siguiente. Funciona como una pila.

**"Si sólo uno de los 8 combos falla, ¿por qué sigue buscando después de los que pasan?"**
Porque volver atrás no pasa sólo por fallar: también pasa cuando alguien pide otra respuesta. `findall` pide hasta que no queden más, por eso salen las 7. Con `once/1` frenaría en la primera.

**"¿Dónde se controla la cantidad máxima?"**
No se controla durante la generación. `subconjunto/2` genera los 2ⁿ subconjuntos igual, y recién después `length/2` mide y `=<` descarta. Con Mati se generan 64 y sobreviven 22. Es el patrón generar y probar.

**"Explicá `subconjunto/2`."**
Por cada elemento hay dos caminos: entra al combo o no entra. Son las dos cláusulas recursivas. El caso base es lista vacía → combo vacío. Como los dos caminos son excluyentes y se respeta el orden original, cada subconjunto se genera exactamente una vez, y en total salen 2ⁿ.

**"¿Por qué el `findall` acá, si en el punto 4 usaste `distinct`?"**
Porque resuelven problemas distintos. En el punto 4 no necesitaba ningún dato: sólo evitar que un luchador saliera repetido, y para eso `distinct/2` es directo. Acá la lista **es** lo que necesito, porque `subconjunto/2` recorre una lista y `sabe/2` entrega las técnicas de a una por backtracking.

**"¿Por qué no es inversible para la cantidad máxima?"**
Porque `Cantidad =< CantidadMaxima` es aritmética y exige los dos lados ligados. La consigna sólo pide inversibilidad para el luchador y el combo, así que no hace falta.

**"¿Qué pasa con Fede?"**
Tiene exactamente un combo posible, el vacío, para cualquier cantidad máxima. Sale solo del caso base de la recursión, sin programar nada especial.

**"¿No genera combos repetidos?"**
No. Hay un test que compara la cantidad de combos generados contra la cantidad de combos únicos (con `sort/2`, que además elimina duplicados) y da igual.

## 13. Tests

Los cinco ejemplos de la consigna están como tests, más los bordes:

```prolog
test(facu_tiene_siete_combos_de_hasta_dos_tecnicas) :-
	findall(Combo, comboPosible(facu, 2, Combo), Combos),
	length(Combos, 7).

test(mati_tiene_veintidos_combos_de_hasta_dos_tecnicas) :-
	findall(Combo, comboPosible(mati, 2, Combo), Combos),
	length(Combos, 22).

test(con_maximo_cero_el_unico_combo_es_el_vacio) :-
	findall(Combo, comboPosible(mati, 0, Combo), Combos),
	Combos = [[]].

% No se puede armar un combo con una técnica que el luchador no sabe
test(facu_no_puede_usar_una_tecnica_ajena, [fail]) :-
	comboPosible(facu, 2, [piso(triangulo, guardia, 4)]).

% Inversible para el luchador: con el combo ligado, deduce de quién es
test(es_inversible_para_el_luchador) :-
	findall(Luchador, comboPosible(Luchador, 1, [golpe(patada, cabeza, 10)]), Luchadores),
	Luchadores = [facu].

% No genera repetidos
test(no_genera_combos_repetidos) :-
	findall(Combo, comboPosible(mati, 6, Combo), Combos),
	sort(Combos, Unicos),
	length(Combos, Cantidad),
	length(Unicos, Cantidad).
```

Los tests de conteo son los más valiosos: cualquier error en `subconjunto/2` (un caso base de más, cláusulas que se pisan) se manifiesta como un número distinto de 7, 8 o 22.

---
---

# Cierre — Conceptos transversales (para repasar el día antes)

Todo el TP se apoya en un puñado de ideas que se repiten. Si entendés estas seis, podés defender cualquier punto.

## 1. Elegir el cuantificador correcto

Es **la** decisión que se repite en casi todos los puntos. Traducir mal el "todos"/"alguno"/"ninguno" del enunciado es el error más común.

| El enunciado dice… | Se escribe… | Ejemplo del TP |
|---|---|---|
| "**todas** las X cumplen" | `forall(Condicion, Cumple)` | `esTemible`, `esCertero`, `esFielASuDisciplina` |
| "**existe** alguna X" | objetivo común | `tieneUnPuntoFlojo`, `esCinturonNegro` |
| "**ninguna** X" | `not(existeAlguna)` | `esNavajaSuiza`, `esPicante` |
| "existe alguna" *y no quiero repetidos* | `distinct(Testigo, Objetivo)` | `sabeUnSometimientoDesdeDistintasPosiciones` |

**Regla de oro**: "ninguno" **no se escribe directo**. Se define el predicado en positivo ("existe alguno") y se lo niega desde afuera.

## 2. Soluciones repetidas: el error silencioso

Un objetivo **no devuelve `true`**: se satisface **una vez por cada forma de demostrarlo**. Si dos caminos distintos llevan al mismo individuo, el predicado lo devuelve dos veces.

| Predicado | ¿Repite? | Por qué |
|---|---|---|
| `esNavajaSuiza` | no (arreglado) | el par de posiciones unificaba en los dos órdenes → `distinct` |
| `esPicante` | **sí** ⚠️ | Facu entrena 3 disciplinas de parado → 3 caminos |
| `tieneUnPuntoFlojo` | no, por suerte | nadie tiene 2 sometimientos flojos… todavía |

**Cómo se detecta**: nunca con un test en modo chequeo. **Siempre** con uno en modo inversible:

```prolog
test(sin_repetidos) :-
	findall(L, elPredicado(L), Ls),
	Ls = [loEsperado].
```

**Cómo se arregla**: `distinct(Testigo, Objetivo)` deja pasar el objetivo una sola vez por cada valor distinto del testigo. La alternativa estándar es `findall/3`, que colapsa muchas soluciones en una porque **siempre se satisface exactamente una vez** (la lista se descarta, se usa sólo para preguntar si quedó vacía).

**Cuándo NO arreglarlo**: si el no determinismo no existe con los datos reales (caso `esCinturonNegro`), agregar el envoltorio es ruido. Un *choicepoint* **no es** una solución repetida.

## 3. Inversibilidad = generador + condiciones

Para que un predicado ande con la variable libre hace falta un objetivo que **proponga candidatos** recorriendo hechos. El resto sólo filtra.

```prolog
predicado(Individuo) :-
	generador(Individuo),      % ← recorre una base de hechos
	condicionPositiva(Individuo),
	not(condicionNegativa(Individuo)).   % ← los not SIEMPRE al final
```

**Los `not/1` van al final, sin excepción.** Con la variable sin ligar, `not(sabeAlgunGolpe(L))` no pregunta "¿este luchador sabe golpes?" sino "¿**alguien** sabe golpes?" — y como alguien sabe, el predicado falla siempre, para todos.

Lo mismo vale para las **comparaciones aritméticas** (`=<`, `>=`, `is`): exigen las variables ligadas. Por eso `esContundente/1` y `comboPosible/3` (en su tercer argumento) no son inversibles, y las consignas lo aclaran.

**Quién fue el generador en cada punto**: `luchador/1` en los puntos 2, 3, 7, 8 y 9; `cinturon/3` en el 4; `sabe/2` en el 6.

## 4. Universo cerrado y verdad vacua

**Universo cerrado**: lo que no está en la base y no se deduce de ella, es **falso**. Por eso no hace falta declarar que Fede no entrena nada — `entrena(fede, _)` ya falla solo.

**Verdad vacua**: `forall/2` sobre un conjunto vacío da **verdadero**. Fede no sabe ninguna técnica, así que "todas sus técnicas son contundentes" es cierto.

Consecuencia en este TP: **Fede aparece en `esFielASuDisciplina`, `esCertero` y `esTemible`**, y tiene un combo posible (el vacío). No es un bug, es la lógica funcionando — pero es lo primero que un docente va a picar, así que hay que tenerlo pensado.

## 5. Functores: modelar y despachar

Un **functor** encabeza un término compuesto y arma un individuo con partes. No es un predicado (no se demuestra) ni una función (no calcula).

Sirvieron para dos cosas distintas:

- **Modelar**: `piso(llaveDeBrazo, espalda, 2)` mantiene junta la información que **identifica** a la técnica. El nombre solo no alcanza — la patada de Mora va a la cabeza y la de Vale a la pierna.
- **Despachar**: tres cláusulas de `esContundente/1`, una por functor, y la unificación elige sola. Eso es **polimorfismo por pattern matching**, sin preguntar por el tipo.

**El costo**: cambiar la aridad (punto 5) rompió todo lo que unificaba con `piso/2` y `golpe/2`, **sin dar error**. Sólo dejaron de unificar. Los tests fueron lo único que lo detectó.

## 6. Hechos vs. reglas

- **Hecho**: conocimiento que no se deduce de nada (`cinturonObtenido/3`, `luchador/1`).
- **Regla**: todo lo que se puede inferir (`otorgaCinturon/1`, la equivalencia de cinturones, `cinturon/3`).

Guardar como hecho algo deducible es **redundancia**, y la redundancia habilita **inconsistencias**: si Mora asciende de cinturón y el hecho del Judo queda escrito a mano, la base se contradice. Con la regla, se recalcula sola.

---

## 7. 🚩 Observaciones sobre código de otros integrantes

Dos cosas que detecté al documentar el TP y que **decidí no corregir**, porque son de mis compañeros y sus tests pasan:

| Predicado | Qué observé | Estado |
|---|---|---|
| `esPicante` (punto 3) | devuelve `[facu, facu, facu]` — Facu entrena 3 disciplinas de parado, o sea 3 caminos que demuestran lo mismo | defecto real, el test lo da por esperado |
| `tieneUnPuntoFlojo` (punto 6) | podría repetir si alguien tuviera 2 sometimientos flojos; hoy no pasa | riesgo latente, sin impacto |

**Por qué las anoto igual**: en el coloquio pueden preguntar por cualquier punto del TP, no sólo por los míos. Si sale `esPicante`, la respuesta es explicar de dónde viene la multiplicidad —`entrena/2` se satisface una vez por disciplina— y cómo se resolvería con `findall`, sin echarle la culpa a nadie: es el mismo error que yo tuve en el punto 4 y que detecté sólo por testear en modo inversible.

---
---

# Tabla final: quién cumple qué

| Predicado | Resultado inversible | Punto |
|---|---|---|
| `esFielASuDisciplina` | `[mora, vale, mati, facu, fede]` | 2 |
| `esPicante` | `[facu]` ⚠️ *tres veces* | 3 |
| `esNavajaSuiza` | `[mati]` | 4 |
| `tieneUnPuntoFlojo` | `[mora, tomi]` | 6 |
| `esCertero` | `[tomi, facu, fede]` | 7 |
| `esTemible` | `[mati, facu, fede]` | 8 |
| `comboPosible(facu, 2, C)` | 7 combos | 9 |

**Estado de los tests**: 71, en 9 unidades (una por predicado), todos en verde.

```prolog
?- [solucion].
?- run_tests.                  % los 71
?- run_tests(comboPosible).    % una sola unidad
```
