# Paradigma Lógico — Elementos de Diseño: Delegación, Acoplamiento y Code Smells (Módulo 6)

> Fuentes: Módulo 6 de la cátedra (Dodino, Bulgarelli, Matos, Passerini, Prieto, Sanzo) + transcripción de la clase teórico-práctica en vivo.

## 1. ¿Qué es diseñar?

Diseñar es definir **componentes**, qué **responsabilidades** tiene cada uno, y qué **relaciones** hay entre ellos. Dentro del paradigma lógico, los componentes se implementan mediante **predicados e individuos**. Por ejemplo: `member/2` tiene la responsabilidad de relacionar un elemento con la lista a la que pertenece; `forall/2` tiene la responsabilidad de determinar si todos los individuos que cumplen un predicado satisfacen también otra condición.

## 2. Decisiones de diseño al modelar desde cero

### 2.1 Predicados individuales vs. "formato de arista" (con listas)

Si queremos registrar que ciertas personas jugaron con ciertos juegos, hay (al menos) dos formas:

```prolog
% opción A: predicados individuales
jugoCon(pablo, rasti).
jugoCon(pablo, bloques).

% opción B: "arista" única, con una lista
jugoCon(pablo, [rasti, bloques]).
```

**¿Cuándo conviene cada una?**

- Si el **orden** entre los elementos importa (por ejemplo, si necesitamos saber que Pablo jugó _primero_ con rasti y _después_ con bloques), conviene el formato de **lista**.
- Si el orden **no** importa, o no sabemos si va a importar, los **predicados individuales** son más simples de trabajar dentro del paradigma.

**Heurística general de diseño**: si tenemos dos alternativas y ninguna saca una ventaja clara sobre la otra, elegimos la variante **más simple**.

### 2.2 Cantidad de individuos en una relación: functores vs. predicados separados

Otra decisión de diseño es cuántos individuos hacemos participar de una misma relación, y si conviene usar **functores** o **predicados separados** con distinto nombre.

```prolog
% opción A: functores dentro de una única relación "tiene"
tiene(teresa, foto([juan, hugo, pedro], 1988)).
tiene(teresa, figurita(messi)).

% opción B: predicados separados
foto(teresa, [juan, hugo, pedro], 1988).
figurita(teresa, messi).
```

- Con **functores** (opción A), el predicado `tiene/2` "lleva la delantera": es el que se usa para disparar las consultas, y permite trabajar de forma **polimórfica** con pattern matching (por ejemplo, para saber qué "cosas valiosas" tiene alguien, sin importar si son fotos o figuritas — como vimos en el ejemplo de deportistas y en el ejercicio integrador de `findall`).
- Con **predicados separados** (opción B), cada uno dispara sus propias consultas por separado, y hay más **indirección** a la hora de combinarlos (por ejemplo, para juntar "todo lo valioso" habría que consultar cada predicado por separado y combinar resultados).

### 2.3 Inversibilidad como decisión de diseño

Qué argumentos de una relación vamos a admitir como inversibles también es parte del diseño. Además, diseñar implica elegir qué primitivas de Prolog usamos "de fábrica" (como `member/2`, `between/3`, `forall/2`) y cuáles construimos nosotros mismos — ambas son decisiones válidas según el contexto.

### 2.4 Normalización de la información

Si escribimos algo así:

```prolog
vacaciones(mayu, "Villa Pehuenia").
vacaciones(alf, "Villa Pehueña").
vacaciones(nico, "villa pehuenia").
```

...la misma ciudad quedó escrita de **tres formas distintas**, lo cual va a hacer muy difícil agrupar correctamente esa información más adelante (por ejemplo, para contar cuánta gente fue al mismo lugar).

**Solución: normalización.** Asignamos un identificador unívoco (por ejemplo, un código) para cada lugar, y usamos siempre ese identificador dentro de la relación. Esto evita la redundancia y las inconsistencias, aunque agrega el costo de tener que mantener esa tabla de identificadores.

## 3. Modelando una solución desde cero: ejemplo de remisería

_Relevamiento_: una empresa de remises tiene una flota de 20 autos y registra todos los viajes. Cada chofer puede manejar un auto distinto cada día. A cada cliente se le da una tarjeta de fidelización (nombre completo + número correlativo). Necesitamos saber: qué choferes hicieron viajes a determinada localidad, quiénes manejaron determinado auto en determinada fecha, y qué viajes hizo un cliente.

**Entidades a modelar**: Clientes, Choferes, Autos, Viajes — ¿y quizás una entidad `Asignación` (qué auto le tocó a qué chofer en qué fecha)? Podríamos modelarla aparte, o resolver esa relación directamente a través del viaje. **Es una decisión de diseño**, no hay una única respuesta correcta.

Una posible solución:

```prolog
viaje(cliente(3, sonia), fecha(11, 2, 2017),
      auto(gol, "ZFE447"), matias,
      "Dorrego y Roque Perez", "Hudson", "Cadorna 283", "Wilde").
```

Notemos que la forma de representar cada entidad **varía** dentro de la misma solución:

- el cliente, el auto y la fecha se representan como **functores**;
- el chofer es un **individuo simple**;
- los domicilios son **strings**;
- `viaje` termina siendo un **predicado de aridad 8**.

**¿Podría `viaje` ser un functor en vez de un predicado?** Sí, pero entonces necesitaríamos igual un predicado que lo relacione con algo más (por ejemplo, la sucursal que gestionó el viaje):

```prolog
viajeHecho(viaje(cliente(3, sonia), fecha(11, 2, 2017),
                 auto(gol, "ZFE447"), matias,
                 "Dorrego y Roque Perez", "Hudson", "Cadorna 283", "Wilde"),
           sucursal3).
```

**Un detalle importante sobre `cliente/2`**: al ser un **functor** (no un predicado), el universo de "clientes" queda definido implícitamente por quienes ya hicieron algún viaje — no por clientes potenciales que todavía no viajaron. Si necesitamos un predicado generador para clientes, hay que construirlo a partir de `viaje/8`:

```prolog
clienteReal(Cliente) :- viaje(Cliente, _, _, _, _, _, _, _).
```

(Lo mismo aplicaría para generar el universo de autos o de choferes reales.)

## 4. Acoplamiento

**Acoplamiento** es el grado en que los componentes **se conocen entre sí**. Retomemos un ejemplo ya conocido:

```prolog
terminoAnio(Alumno, Anio) :-
    forall(materia(Materia, Anio), aprobo(Alumno, Materia)).

aprobo(Alumno, Materia) :- nota(Alumno, Materia, Nota), Nota >= 6.
```

Hay acoplamiento entre `terminoAnio/2` y los predicados `materia/2` y `aprobo/2` — y ese acoplamiento es **buscado y necesario**: sin él, no podríamos resolver el requerimiento.

### ¿Cuándo un acoplamiento es "adecuado"?

Supongamos que cambia la forma de registrar las notas, para poder incluir los aplazos:

```prolog
% antes
nota(raul, pdp, 9).
nota(herminio, pdp, 6).

% después: ahora es una lista de notas
nota(raul, pdp, [9]).
nota(herminio, pdp, [2, 2, 6]).
```

¿Qué predicados hay que tocar?

- **`aprobo/2` sí**, porque tiene que considerar solamente la **última** nota (asumiendo que la nota de aprobación queda al final de la lista):

```prolog
aprobo(Alumno, Materia) :-
    nota(Alumno, Materia, Notas),
    last(Notas, Nota),
    Nota >= 6.
```

- **`terminoAnio/2` NO hace falta tocarlo.** Aunque está acoplado con `aprobo/2`, ese acoplamiento queda "contenido": `terminoAnio/2` sigue trabajando exactamente con los mismos individuos (`Alumno`, `Materia`) sin que le importe **cómo** se calculan las notas por dentro.

Esto es justamente lo que hace que el acoplamiento entre `terminoAnio/2` y `aprobo/2` sea **adecuado**: el cambio se propagó solo hasta donde tenía sentido, y no "contaminó" el resto del diseño. Medir qué tan lejos se propaga un cambio es una forma concreta de evaluar si el acoplamiento de una solución está bien pensado.

## 5. Code smells

Los **code smells** son señales de que una solución — aunque funcione — debería mejorarse: agregando abstracciones que faltan, usando conceptos más adecuados del paradigma, o haciendo el código más general o expresivo. A este proceso de mejora (sin cambiar el comportamiento) se lo llama **refactorización**.

### 5.1 `findall` + `length > 0` en vez de verificar existencia directamente

```prolog
% ⚠️ mal: arma una lista entera solo para ver si no está vacía
prepara(juana, solero).
prepara(lisandro, solero).
prepara(juana, margarita).

algunPrepara(Trago) :-
    trago(Trago),
    findall(Bartender, prepara(Bartender, Trago), Bartenders),
    length(Bartenders, Cantidad),
    Cantidad > 0.
```

Esto **funciona**, pero no respeta la idea del paradigma: si lo que queremos es saber si **existe** alguien que prepara un trago, alcanza con la consulta existencial directa, sin construir nada:

```prolog
% ✔️ bien: la propia consulta ya verifica existencia
algunPrepara(Trago) :- prepara(_, Trago).
```

(Si `Bartender` termina siendo una variable que no se usa más que una vez, la convertimos en variable anónima `_`.)

### 5.2 `findall` + `length == 0` en vez de `not/1`

Del mismo modo, para verificar que **nadie** prepara un trago:

```prolog
% ⚠️ mal
nadiePrepara(Trago) :-
    trago(Trago),
    findall(Bartender, prepara(Bartender, Trago), Bartenders),
    length(Bartenders, 0).

% ✔️ bien: negamos directamente la consulta
nadiePrepara(Trago) :-
    trago(Trago),
    not(prepara(_, Trago)).
```

Recordemos que `Trago` tiene que llegar **ligado** antes del `not` (necesitamos el predicado generador `trago/1`), porque Prolog no puede determinar el universo de tragos que nadie prepara sin conocer antes el universo completo de tragos.

### 5.3 `findall` + `member` (son operaciones inversas)

```prolog
% ⚠️ mal: primero armo la lista, después la "desarmo" con member
quienPrepara(Trago, Bartender) :-
    findall(B, prepara(B, Trago), Bartenders),
    member(Bartender, Bartenders).

% ✔️ bien: la consulta existencial ya me da cada individuo, uno por uno
quienPrepara(Trago, Bartender) :- prepara(Bartender, Trago).
```

`findall/3` arma listas; `member/2` las desarma — son operaciones inversas, así que encadenarlas sin necesidad es casi siempre una señal de que se podría resolver con una consulta existencial simple y directa.

### 5.4 `findall` + `forall`/`member` en vez de `forall` directo

_"Un trago es copado si todos los bartenders que lo preparan ganan bien."_

```prolog
% ⚠️ mal: armo dos listas y comparo con forall+member
tragoCopado(Trago) :-
    trago(Trago),
    findall(B, prepara(B, Trago), Preparan),
    findall(B, ganaBien(B), GananBien),
    forall(member(B, Preparan), member(B, GananBien)).

% ✔️ bien: la relación entre las dos condiciones ya es justo lo que hace forall
tragoCopado(Trago) :-
    trago(Trago),
    forall(prepara(Bartender, Trago), ganaBien(Bartender)).
```

**Aclaración importante**: esto **no** significa que `findall` + `member` (dentro de un `forall`) esté siempre mal. Por ejemplo, la definición de `incluido/2` que vimos en el módulo de predicados de orden superior es perfectamente correcta:

```prolog
incluido(A, B) :- forall(member(X, A), member(X, B)).
```

Acá el uso de `member/2` es legítimo porque el **dominio ya es una lista** de por sí (los conjuntos `A` y `B` están representados como listas) — no se construyó ninguna lista de más para llegar a esta relación.

### 5.5 Excepción: cuando `findall` + `member` sí es necesario

_Ejemplo: armar todos los itinerarios posibles (permutaciones) de una lista dinámica de lugares a visitar._

```prolog
itinerario([], []).
itinerario(Lugares, [Lugar|RestoItinerario]) :-
    member(Lugar, Lugares),
    sacar(Lugar, Lugares, RestoLugares),
    itinerario(RestoLugares, RestoItinerario).

% sacar(Elemento, Lista, Resto): relaciona una lista con el resultado
% de sacarle un elemento (asumiendo que no hay elementos repetidos)
sacar(_, [], []).
sacar(Lugar, [Lugar|Resto], Resto).
sacar(Lugar, [Otro|Resto], [Otro|RestoSinLugar]) :-
    Lugar \= Otro,
    sacar(Lugar, Resto, RestoSinLugar).
```

```prolog
?- itinerario([villaPehuenia, sanMartinDeLosAndes, caviahue], Orden).
Orden = [villaPehuenia, sanMartinDeLosAndes, caviahue] ;
Orden = [villaPehuenia, caviahue, sanMartinDeLosAndes] ;
Orden = [sanMartinDeLosAndes, villaPehuenia, caviahue] ;
...
```

Acá `member/2` **no** aparece pegado inmediatamente después de un `findall` (de hecho no hay ningún `findall`) — aparece dentro de un predicado **recursivo** que va armando las permutaciones paso a paso. En los hechos, esto termina siendo una implementación casera del predicado `permutation/2`, que ya viene incluido en Prolog. **Moraleja**: el patrón "findall seguido de member" suele ser un code smell, pero no es una regla absoluta — siempre hay que mirar el contexto.

### 5.6 Negación vs. aserción (doble negación)

```prolog
todosSiguenA(Rey) :-
    personaje(Rey),
    not((personaje(Personaje), not(sigueA(Personaje, Rey)))).
```

Este predicado es inversible y funciona bien, pero es **difícil de leer**: "todos siguen a un personaje si no existe otro personaje que no lo siga" tiene una doble negación que cuesta interpretar. Usando la equivalencia lógica que ya conocemos (`∄x (p(x) ∧ ¬q(x)) ≡ ∀x (p(x) ⇒ q(x))`):

```prolog
todosSiguenA(Rey) :-
    personaje(Rey),
    forall(personaje(Personaje), sigueA(Personaje, Rey)).
```

Mismo comportamiento, mucho más fácil de leer: _"todos siguen a un personaje si... todos los personajes siguen a ese personaje"_.

Otro ejemplo visto en clase: _"una golosina es deseada si no existe una persona que no la quiera comprar"_ → mejor como _"una golosina es deseada si toda persona la quiere comprar"_:

```prolog
% ⚠️ mal: doble negación
golosinaDeseada(Golosina) :-
    golosina(Golosina),
    not((persona(Persona), not(quiereComprar(Persona, Golosina)))).

% ✔️ bien: forall directo, con menos paréntesis y más legible
golosinaDeseada(Golosina) :-
    golosina(Golosina),
    forall(persona(Persona), quiereComprar(Persona, Golosina)).
```

### 5.7 Evitar duplicidades

_"Una ciudad es interesante si es antigua y tiene más de 10 puntos de interés copados. Un bar es copado si tiene más de 4 variedades de cerveza. Un museo de ciencias naturales siempre es copado."_

```prolog
% ⚠️ mal: se repite el patrón findall + length para cada tipo de lugar
baresCopados(Ciudad, Bares) :-
    findall(bar(CantVarCer), (puntoDeInteres(bar(CantVarCer), Ciudad), CantVarCer > 4), Bares).

museosCopados(Ciudad, Museos) :-
    findall(museo(cienciasNaturales), puntoDeInteres(museo(cienciasNaturales), Ciudad), Museos).

ciudadInteresante(Ciudad) :-
    antigua(Ciudad),
    baresCopados(Ciudad, Bares),
    museosCopados(Ciudad, Museos),
    length(Bares, CantidadBares),
    length(Museos, CantidadMuseos),
    CantidadLugaresCopados is CantidadBares + CantidadMuseos,
    CantidadLugaresCopados > 10.
```

El problema: si mañana aparecen restaurantes copados, avenidas copadas, etc., hay que agregar un `findall` + `length` más, y sumarlo a mano cada vez. Además, `CantVarCer` es un nombre poco expresivo.

```prolog
% ✔️ bien: trabajamos de forma polimórfica con "puntoDeInteres copado"
ciudadInteresante(Ciudad) :-
    antigua(Ciudad),
    cosasCopadas(Ciudad, Cosas),
    length(Cosas, CantidadCosas),
    CantidadCosas > 10.

cosasCopadas(Ciudad, Cosas) :-
    findall(Cosa, (puntoDeInteres(Cosa, Ciudad), copada(Cosa)), Cosas).

copada(museo(cienciasNaturales)).
copada(bar(VariedadesCerveza)) :- VariedadesCerveza > 4.
```

Ahora, si aparecen nuevos tipos de puntos de interés copados, **no hace falta tocar ni `ciudadInteresante/1` ni `cosasCopadas/2`** — alcanza con agregar una nueva cláusula a `copada/1`. Esta es una métrica real y concreta de bajo acoplamiento: el cambio queda completamente aislado.

### 5.8 Keep it simple (evitar "trucos algorítmicos" en vez de reglas lógicas)

_"`inFraganti/2` relaciona un delito y un delincuente si el delincuente cometió el delito y hay al menos un testigo."_

```prolog
% ⚠️ mal
inFraganti(Delito, Delincuente) :-
    cometio(Delito, Delincuente),
    findall(Testigo, testigo(Delito, Testigo), Testigos),
    length(Testigos, Cantidad),
    Cantidad > 0.

% ✔️ bien: no necesito saber CUÁNTOS testigos hay, solo que exista uno
inFraganti(Delito, Delincuente) :-
    cometio(Delito, Delincuente),
    testigo(Delito, _).
```

_"Una persona no tiene hijos"_:

```prolog
% ⚠️ mal
noTieneHijos(Persona) :-
    persona(Persona),
    findall(Hijo, padre(Persona, Hijo), Hijos),
    length(Hijos, 0).

% ✔️ bien: p ∧ ¬q, directamente
noTieneHijos(Persona) :-
    persona(Persona),
    not(padre(Persona, _)).
```

_"Hijo único"_ — mejor pensado como _"no tiene hermanos"_, en vez de _"la cantidad de hijos de mi padre es exactamente 1"_:

```prolog
% ⚠️ mal (truco algorítmico: contar)
hijoUnico(Persona) :-
    padre(Padre, Persona),
    findall(Hijo, padre(Padre, Hijo), Hijos),
    length(Hijos, 1).

% ✔️ bien (regla lógica: no existe otro hijo distinto del mismo padre)
hijoUnico(Persona) :-
    padre(Padre, Persona),
    not((padre(Padre, Persona2), Persona \= Persona2)).
```

_"Una persona está complicada si tiene algún hijo menor de 18 años problemático"_:

```prolog
% ⚠️ mal: findall seguido de member, otra vez operaciones inversas
estaComplicado(Persona) :-
    findall(Hijo, (padre(Persona, Hijo), edad(Hijo, Edad), Edad < 18), Hijos),
    member(Hijo, Hijos),
    problematico(Hijo).

% ✔️ bien: consulta individual directa
estaComplicado(Persona) :-
    padre(Persona, Hijo),
    edad(Hijo, Edad),
    Edad < 18,
    problematico(Hijo).
```

**El error más común** para quien recién empieza en el paradigma es armar listas cuando no hacen falta para resolver el problema — se nota justamente por la combinación `findall` seguido de `member` sobre esa misma lista.

### 5.9 Lazy predicate

```prolog
estaComplicado(Persona) :- personaComplicada(Persona).

personaComplicada(Persona) :- ...
```

Delegar responsabilidades en otro predicado está bien — pero acá `estaComplicado/1` **no agrega ningún valor**: es exactamente la misma responsabilidad que `personaComplicada/1`, con otro nombre. Es una duplicación de ideas conceptual. El refactor es simplemente **eliminar uno de los dos** y quedarse con una sola abstracción.

### 5.10 Incógnitas innecesarias

```prolog
% el número de la suerte de una persona es su día de nacimiento,
% salvo para Joaquín, que además es el 8

numeroDeLaSuerte(Persona, Numero) :- diaDelNacimiento(Persona, Numero).

% ⚠️ mal: una incógnita que no aporta nada (equivalente a una "variable local" de más)
numeroDeLaSuerte(joaquin, Numero) :- Numero is 8.

% ✔️ bien: directamente pattern matching con el valor
numeroDeLaSuerte(joaquin, 8).
```

### 5.11 Expresividad

```prolog
obraMaestra(Compositor, Obra) :-
    compositor(Compositor, Obra),
    forall(movimiento(Obra, Movimiento), cumpleCond(Movimiento)).
```

Renombrar `cumpleCond/1` a `cumpleCondiciones/1` **no resuelve nada de fondo**: seguimos sin saber, con solo leer la regla, **qué condiciones concretas** debe cumplir un movimiento para que una obra sea considerada maestra. El problema real de expresividad es que la regla de negocio queda escondida dentro de la implementación de `cumpleCond/1` — hay que ir a leer ese predicado para entender la regla. La solución pasa por usar nombres y conceptos que reflejen el dominio del negocio, no solo por renombrar variables.

### 5.12 Parametrizar lo que está fijo (hardcoding)

```prolog
% ⚠️ mal: los valores están "hardcodeados" en el NOMBRE del predicado
puedeComer(analia, Comida) :-
    ingrediente(Comida, _),
    forall(ingrediente(Comida, Ingrediente),
           (not(contieneCarne(Ingrediente)),
            not(contieneHuevo(Ingrediente)),
            not(contieneLeche(Ingrediente)))).
```

`carne`, `huevo` y `leche` no son individuos que participan de ninguna relación — están escondidos en el **nombre** de tres predicados distintos (`contieneCarne/1`, `contieneHuevo/1`, `contieneLeche/1`), lo cual hace más difícil extender la solución (agregar un nuevo elemento indeseado implica escribir un predicado nuevo).

**Primer paso del refactor**: parametrizar, convirtiendo esos elementos en individuos de una relación:

```prolog
contiene(Ingrediente, Elemento).
```

**Pero el refactor más importante todavía falta**: lo que Analía puede comer es cualquier comida **apta para veganos** — ese concepto (veganismo) es una abstracción de negocio que faltaba nombrar explícitamente:

```prolog
% ✔️ bien
puedeComer(analia, Comida) :-
    comida(Comida),
    forall(ingrediente(Comida, Ingrediente), aptoParaVegano(Ingrediente)).

aptoParaVegano(Ingrediente) :-
    not((elementoIndeseadoParaVegano(Elemento), contiene(Ingrediente, Elemento))).

elementoIndeseadoParaVegano(carne).
elementoIndeseadoParaVegano(huevo).
elementoIndeseadoParaVegano(leche).
```

Interesante: `aptoParaVegano/1` en sí mismo también podría parametrizarse más (¿qué pasa con dietas para hipertensos, diabéticos, celíacos?) — pero **cuidado con "pasarse de rosca"**: al refactorizar conviene ir por metas cumplibles y no generalizar de más sin necesidad concreta.

## 6. Resumen

Diseñar en el paradigma lógico implica tomar decisiones propias del paradigma: cómo modelar la información (predicados individuales vs. listas, functores vs. predicados separados, normalización) y cómo relacionar predicados e individuos entre sí. Algunas soluciones son objetivamente mejores que otras según criterios concretos: el grado de **acoplamiento** entre componentes, la **simplicidad** de las reglas, evitar **código duplicado**, preferir **afirmaciones por sobre negaciones**, y otras características que se estudian a través de antipatrones llamados **code smells**. El objetivo de identificarlos y corregirlos (**refactorizar**) es mantener el código igual de funcional pero más fácil de entender, mantener y extender.

---

## 📌 Glosario rápido para repasar

| Término                       | Definición                                                                                                                                                                           |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Diseñar**                   | Definir componentes, sus responsabilidades, y las relaciones entre ellos.                                                                                                            |
| **Acoplamiento**              | Grado en que los componentes de una solución se conocen entre sí. Puede ser necesario (buscado) o excesivo (una señal de mal diseño).                                                |
| **Acoplamiento "adecuado"**   | Cuando un cambio en un componente se propaga solo hasta donde tiene sentido, sin afectar componentes que no deberían verse impactados.                                               |
| **Normalización**             | Asignar identificadores unívocos a los datos para evitar inconsistencias y redundancia (ej. distintas formas de escribir el mismo lugar).                                            |
| **Code smell**                | Señal de que una solución, aunque funcione, debería mejorarse: le faltan abstracciones, usa conceptos poco adecuados, o es poco general/expresiva.                                   |
| **Refactorización**           | Proceso de mejorar el diseño de una solución sin cambiar su comportamiento externo.                                                                                                  |
| **`findall` + `length > 0`**  | Code smell: verificar existencia armando una lista completa, en vez de usar la consulta existencial directamente.                                                                    |
| **`findall` + `length == 0`** | Code smell: verificar no-existencia armando una lista, en vez de usar `not/1` directamente.                                                                                          |
| **`findall` + `member`**      | Code smell: armar una lista para luego "desarmarla" con `member`, cuando son operaciones inversas — casi siempre alcanza con la consulta existencial directa.                        |
| **Doble negación**            | Code smell: definir un predicado con dos `not` anidados, cuando existe una versión equivalente y más legible usando `forall` directamente.                                           |
| **Lazy predicate**            | Code smell: un predicado que solo delega en otro con la misma responsabilidad, sin agregar valor — duplicación conceptual.                                                           |
| **Incógnita innecesaria**     | Code smell: usar una variable donde alcanzaría con pattern matching directo sobre un valor conocido.                                                                                 |
| **Hardcoding**                | Fijar un valor específico dentro del nombre de un predicado (ej. `contieneCarne/1`) en vez de parametrizarlo como un individuo de una relación (ej. `contiene(Ingrediente, carne)`). |
| **Abstracción faltante**      | Cuando el código resuelve un requerimiento pero no nombra explícitamente un concepto del negocio que le daría más claridad (ej. "veganismo").                                        |
