## Punto 1: consultas con expresiones for

### Código

```scala
def fuertes(r: Registro, umbral: Double): List[String] =
  for {
    s <- r
    if s.magnitud > umbral
  } yield s.region

def enRegion(r: Registro, region: String): List[Double] =
  for {
    s <- r
    if s.region == region
  } yield s.magnitud

def paresCercanos(r: Registro, dif: Double): List[(String, String)] = {
  val regiones = (for (s <- r) yield s.region).distinct
  val indexadas = regiones.zipWithIndex

  for {
    (a, i) <- indexadas
    (b, j) <- indexadas
    if i < j
    cercanos = for {
      s1 <- r if s1.region == a
      s2 <- r if s2.region == b
      if Math.abs(s1.magnitud - s2.magnitud) <= dif
    } yield (s1, s2)
    if cercanos.nonEmpty
  } yield (a, b)
}
```

Las tres consultas son expresiones `for` con `yield`. No hay `var`, `while` ni `return`, y las funciones auxiliares (`regiones`, `indexadas`, `cercanos`) viven dentro de `paresCercanos`.

### Qué hace cada función

* `fuertes` recorre el registro, descarta los sismos cuya magnitud no supera el umbral y devuelve la región de los que quedan. No se quitan repetidos, así que una región con dos sismos fuertes aparece dos veces, en el orden del registro.
* `enRegion` recorre el registro, se queda con los sismos de la región pedida y devuelve sus magnitudes. Si la región no está, ningún sismo pasa la guardia y el resultado es `List()`.
* `paresCercanos` hace cuatro cosas:
    * Calcula las regiones sin repetir, en orden de primera aparición (`regiones`).
    * Les pone un índice con `zipWithIndex`.
    * Forma todas las combinaciones de dos regiones y se queda solo con las que cumplen `i < j`. Así una región nunca se empareja consigo misma y cada pareja sale una sola vez.
    * Deja pasar la pareja solo si existe algún par de sismos, uno de cada región, con magnitudes que difieren a lo sumo `dif`. Eso lo decide el `for` interno (`cercanos`).

### Expresiones `for`: en qué se traducen

El compilador traduce todo `for` a `map`, `flatMap` y `withFilter` con estas reglas:

* Un generador con `yield`: `for (x <- c) yield e` se traduce a `c.map(x => e)`.
* Un generador con guardia: `for (x <- c if p) yield e` se traduce a `c.withFilter(x => p).map(x => e)`.
* Varios generadores: todos menos el último se traducen a `flatMap`, y el último a `map`.

**`fuertes`**

```scala
r.withFilter(s => s.magnitud > umbral).map(s => s.region)
```

**`enRegion`**

```scala
r.withFilter(s => s.region == region).map(s => s.magnitud)
```

**`paresCercanos`, regiones sin repetir**

```scala
r.map(s => s.region).distinct
```

**`paresCercanos`, el `for` interno (`cercanos`)**

```scala
r.withFilter(s1 => s1.region == a).flatMap(s1 =>
  r.withFilter(s2 => s2.region == b)
   .withFilter(s2 => Math.abs(s1.magnitud - s2.magnitud) <= dif)
   .map(s2 => (s1, s2)))
```

**`paresCercanos`, el `for` externo**

Tiene dos generadores, una guardia, una definición (`cercanos = ...`) y otra guardia. Es equivalente a:

```scala
indexadas.flatMap { case (a, i) =>
  indexadas
    .withFilter { case (b, j) => i < j }
    .map { case (b, j) =>
      val cercanos = /* el for interno */
      (b, cercanos)
    }
    .withFilter { case (b, cercanos) => cercanos.nonEmpty }
    .map { case (b, _) => (a, b) }
}
```

La definición `cercanos = ...` se convierte en un `val` dentro de un `map`, y la guardia siguiente filtra sobre ese valor. El patrón de tupla `(a, i) <- indexadas` puede añadir un `withFilter` que deja pasar todo, porque toda tupla cumple el patrón.

### Ejecución paso a paso: `fuertes(registro, 5.0)`

| Posición | Sismo | Magnitud | ¿Mayor que 5.0? | Aporta |
|:---:|---|:---:|:---:|---|
| 1 | Valle | 4.2 | No | — |
| 2 | Santander | 5.1 | Sí | "Santander" |
| 3 | Valle | 3.8 | No | — |
| 4 | Choco | 6.0 | Sí | "Choco" |
| 5 | Santander | 4.7 | No | — |
| 6 | Valle | 5.3 | Sí | "Valle" |
| 7 | Choco | 3.1 | No | — |

Resultado: `List("Santander", "Choco", "Valle")`.

Con umbral `6.0` el sismo de Choco (magnitud exacta 6.0) no entra, porque la comparación es estricta. Con umbral `9.0` ningún sismo pasa y el resultado es `List()`.

### Ejecución paso a paso: `enRegion(registro, "Valle")`

| Posición | Región | ¿Es "Valle"? | Aporta |
|:---:|---|:---:|:---:|
| 1 | Valle | Sí | 4.2 |
| 2 | Santander | No | — |
| 3 | Valle | Sí | 3.8 |
| 4 | Choco | No | — |
| 5 | Santander | No | — |
| 6 | Valle | Sí | 5.3 |
| 7 | Choco | No | — |

Resultado: `List(4.2, 3.8, 5.3)`.

Caso borde, `enRegion(registro, "Amazonas")`: ningún sismo cumple la guardia y el resultado es `List()`. Con el registro vacío tampoco hay elementos que recorrer, y el resultado es `List()`.

### Ejecución paso a paso: `paresCercanos(registro, 0.6)`

1. `regiones` es `List("Valle", "Santander", "Choco")`, en el orden en que aparece cada una por primera vez.
2. `indexadas` es `List(("Valle",0), ("Santander",1), ("Choco",2))`.
3. Los dos generadores producen las 9 combinaciones $(a, b)$. La guardia `i < j` deja solo tres:

   | Pareja | Magnitudes de $a$ | Magnitudes de $b$ | Menor diferencia | ¿Es $\le 0.6$? | ¿Sale? |
         |---|---|---|:---:|:---:|:---:|
   | (Valle, Santander) | 4.2, 3.8, 5.3 | 5.1, 4.7 | 0.2 (5.3 y 5.1) | Sí | Sí |
   | (Valle, Choco) | 4.2, 3.8, 5.3 | 6.0, 3.1 | 0.7 (5.3 y 6.0, o 3.8 y 3.1) | No | No |
   | (Santander, Choco) | 5.1, 4.7 | 6.0, 3.1 | 0.9 (5.1 y 6.0) | No | No |

Resultado: `List(("Valle", "Santander"))`.

Con `dif = 1.0` las tres parejas cumplen la condición y salen en el orden `(Valle, Santander)`, `(Valle, Choco)`, `(Santander, Choco)`. Con `dif = 0.1` ninguna cumple y el resultado es `List()`.

Caso borde: con una sola región, `indexadas` tiene un solo elemento, no existe ningún par con `i < j` y el resultado es `List()`.

### Diagramas

Orden en que `fuertes(registro, 5.0)` pide y descarta los sismos:

```mermaid
sequenceDiagram
    participant F as for con yield
    participant W as withFilter
    participant M as map
    F->>W: sismo 1, magnitud 4.2
    W-->>F: no pasa, 4.2 no es mayor que 5.0
    F->>W: sismo 2, magnitud 5.1
    W->>M: pasa
    M-->>F: Santander
    F->>W: sismo 3, magnitud 3.8
    W-->>F: no pasa
    F->>W: sismo 4, magnitud 6.0
    W->>M: pasa
    M-->>F: Choco
    F->>W: sismos 5 y 7 no pasan
    F->>W: sismo 6, magnitud 5.3
    W->>M: pasa
    M-->>F: Valle
```

Flujo de `paresCercanos`:

```mermaid
flowchart TD
    A["r: Registro"] --> B["regiones: regiones sin repetir en orden de aparicion"]
    B --> C["indexadas: cada region con su indice"]
    C --> D["generar (a,i) y (b,j)"]
    D --> E{"i menor que j ?"}
    E -- no --> X["descartar"]
    E -- si --> F["cercanos: pares (s1,s2) de regiones a y b con diferencia menor o igual que dif"]
    F --> G{"cercanos no vacio ?"}
    G -- no --> X
    G -- si --> H["yield (a, b)"]
```
---
## Punto 2: agrupar con mapas

### Código

```scala
def porRegion(r: Registro): Map[String, List[Sismo]] =
  r.foldLeft(Map[String, List[Sismo]]()) { (mapa, sismo) =>
    mapa + (sismo.region -> (mapa.getOrElse(sismo.region, List()) :+ sismo))
  }

def conteoPorRegion(r: Registro): Map[String, Int] = {
  val grupos = porRegion(r)
  grupos.map { case (region, sismos) =>
    (region, sismos.length)
  }
}

def magnitudMaxima(r: Registro): Map[String, Double] = {
  val grupos = porRegion(r)
  grupos.map { case (region, sismos) =>
    val magnitudes = sismos.map(sismo => sismo.magnitud)
    (region, magnitudes.max)
  }
}
```

### Qué hace cada operación

* Un `Map[String, List[Sismo]]` es una tabla: cada **llave** (la región) apunta a
  un **valor** (la lista de sismos de esa región).
* `r.foldLeft(inicio) { (mapa, sismo) => ... }` recorre la lista `r` de
  izquierda a derecha. Empieza con el valor `inicio` y, por cada sismo, aplica la
  función `(mapa, sismo) => ...`, que recibe lo armado hasta ahora (`mapa`) y el
  sismo que toca, y devuelve el mapa nuevo. El mapa que devuelve el último sismo
  es el resultado.
* `Map[String, List[Sismo]]()` es el `inicio`: un mapa vacío.
* `mapa.getOrElse(sismo.region, List())` pide la lista de la región del sismo.
  Si la región todavía no está en el mapa, devuelve una lista vacía.
* `:+ sismo` agrega el sismo al final de esa lista y devuelve una lista nueva.
* `mapa + (region -> lista)` devuelve un mapa nuevo con ese par. Si la región ya
  estaba, reemplaza su lista; si no, la agrega. El mapa anterior no se modifica.
* `grupos.map { case (region, sismos) => ... }` visita cada par del mapa. El
  `case (region, sismos)` le pone nombre a la llave y al valor. La función
  devuelve un par nuevo, y esos pares forman un `Map` nuevo.
* `sismos.length` es la cantidad de sismos de la lista.
* `sismos.map(sismo => sismo.magnitud)` convierte la lista de sismos en la lista
  de sus magnitudes, y `.max` toma la mayor.
* `val grupos = porRegion(r)` le pone nombre al resultado para usarlo en la
  línea siguiente. Un `val` se fija una vez y no cambia, así que no es una
  variable.

`foldLeft` reemplaza al ciclo: no hay `var`, `while` ni `return`. Lo que va
armado viaja de un paso al siguiente como el parámetro `mapa`.

### Expresiones `for`

En el punto 2 **no se usó ninguna expresión `for`**: los agrupamientos se hacen
con mapas (`foldLeft` y `map`). Si `conteoPorRegion` se escribiera con `for`,
sería:

```scala
for ((region, sismos) <- porRegion(r)) yield (region, sismos.length)
```

y el compilador lo traduce a `porRegion(r).map { case (region, sismos) => (region, sismos.length) }`,
que es la versión entregada.

### Ejecución paso a paso: `porRegion(registro)`

Se empieza con `Map()`. En cada paso se procesa un sismo y se crea un mapa nuevo
(para abreviar, las listas muestran solo las magnitudes).

| Paso | Sismo que se procesa | ¿La región ya estaba? | Mapa después del paso |
|:---:|---|:---:|---|
| 0 | (inicio) | | `Map()` |
| 1 | Valle, 4.2 | No | Valle → (4.2) |
| 2 | Santander, 5.1 | No | Valle → (4.2), Santander → (5.1) |
| 3 | Valle, 3.8 | Sí | Valle → (4.2, 3.8), Santander → (5.1) |
| 4 | Choco, 6.0 | No | Valle → (4.2, 3.8), Santander → (5.1), Choco → (6.0) |
| 5 | Santander, 4.7 | Sí | Valle → (4.2, 3.8), Santander → (5.1, 4.7), Choco → (6.0) |
| 6 | Valle, 5.3 | Sí | Valle → (4.2, 3.8, 5.3), Santander → (5.1, 4.7), Choco → (6.0) |
| 7 | Choco, 3.1 | Sí | Valle → (4.2, 3.8, 5.3), Santander → (5.1, 4.7), Choco → (6.0, 3.1) |

Al terminar la lista, el resultado es el mapa del paso 7. Cada región conserva
sus sismos en el orden en que aparecen en el registro.

### Ejecución paso a paso: `conteoPorRegion(registro)`

1. `porRegion(registro)` da el mapa del paso 7 y se guarda en `grupos`.
2. `map` visita cada par y cambia la lista por su tamaño:

   | Par que entra | `sismos.length` | Par que sale |
   |---|:---:|---|
   | Valle → (4.2, 3.8, 5.3) | 3 | (Valle, 3) |
   | Santander → (5.1, 4.7) | 2 | (Santander, 2) |
   | Choco → (6.0, 3.1) | 2 | (Choco, 2) |

3. Los pares se reúnen en un mapa:
   `Map("Valle" -> 3, "Santander" -> 2, "Choco" -> 2)`.

### Ejecución paso a paso: `magnitudMaxima(registro)`

1. `porRegion(registro)` da el mapa del paso 7 y se guarda en `grupos`.
2. `map` visita cada par. Primero saca las magnitudes y luego toma la mayor:

   | Región | `magnitudes` | `magnitudes.max` | Par que sale |
   |---|---|:---:|---|
   | Valle | List(4.2, 3.8, 5.3) | 5.3 | (Valle, 5.3) |
   | Santander | List(5.1, 4.7) | 5.1 | (Santander, 5.1) |
   | Choco | List(6.0, 3.1) | 6.0 | (Choco, 6.0) |

3. Los pares se reúnen en un mapa:
   `Map("Valle" -> 5.3, "Santander" -> 5.1, "Choco" -> 6.0)`.

### Casos borde

* **Registro vacío.** `foldLeft` no tiene sismos que procesar y devuelve el
  `inicio`, `Map()`. Entonces `grupos` es `Map()`, el `map` no tiene pares que
  visitar y `conteoPorRegion` y `magnitudMaxima` también dan `Map()`.
* **Región que no está en el registro.** Una región solo entra al mapa cuando se
  procesa un sismo suyo, así que una región ausente no aparece.
* **Un solo sismo.** El mapa queda con una sola región. Su lista tiene un sismo,
  su conteo es 1 y su máxima es la magnitud de ese sismo.
* **`max` sobre una lista vacía.** Nunca ocurre: una región solo existe si tiene
  al menos un sismo, así que cada lista tiene por lo menos un elemento.
* **Orden.** Si el registro se recorre al revés, el conteo y la magnitud máxima
  son los mismos. Solo cambia el orden de los sismos dentro de cada lista de
  `porRegion` (ver `correccion.md`, sección 2.6). Además, el orden en que se
  muestran las regiones dentro del `Map` no es parte del resultado.

### Diagramas

Cómo se relacionan las tres funciones:

```mermaid
flowchart LR
    R["registro: List de Sismo"] --> F["porRegion: foldLeft arma el Map"]
    F --> G["Map: region a lista de sismos"]
    G --> C["conteoPorRegion: map con length"]
    G --> M["magnitudMaxima: map con max de las magnitudes"]
    C --> RC["Map: region a conteo"]
    M --> RM["Map: region a magnitud maxima"]
```

Los primeros pasos de `foldLeft` en `porRegion(registro)`:

```mermaid
sequenceDiagram
    participant F as foldLeft
    participant M as mapa
    F->>M: inicio, Map vacio
    F->>M: Valle 4.2, no estaba, agrega Valle con lista de 4.2
    M-->>F: Valle con 4.2
    F->>M: Santander 5.1, no estaba, agrega Santander con lista de 5.1
    M-->>F: Valle con 4.2 y Santander con 5.1
    F->>M: Valle 3.8, ya estaba, agrega 3.8 al final de la lista de Valle
    M-->>F: Valle con 4.2 y 3.8, Santander con 5.1
    F->>M: sigue con los 4 sismos que faltan
```
---
## Punto 3: consultas seguras

Un mapa normal tiene un defecto incómodo: si se le pide una llave que no tiene, se cae. Por ejemplo, `conteoPorRegion(registro)("Amazonas")` lanza `NoSuchElementException: key not found: Amazonas`. Este punto resuelve ese problema de tres maneras distintas, según lo que tenga sentido responder cuando no hay datos:

* En `conteoSeguro`, la respuesta natural es cero: una región sin sismos tiene cero sismos.
* En `maximaDe`, no hay una respuesta natural: la mayor magnitud de una región sin sismos no existe, y inventar un número sería mentir. Por eso la respuesta es `None`.
* En `regionMasActiva`, si el registro está vacío no hay ninguna región que mencionar, y la respuesta también es `None`.
  Para todas las trazas de este punto usamos el registro de siete sismos del enunciado. Con él, `conteoPorRegion(registro)` es `Map(Valle -> 3, Santander -> 2, Choco -> 2)` y `magnitudMaxima(registro)` es `Map(Valle -> 5.3, Santander -> 5.1, Choco -> 6.0)`.

### conteoSeguro

```scala
def conteoSeguro(r: Registro): Map[String, Int] =
  conteoPorRegion(r).withDefaultValue(0)
```

La función no calcula nada nuevo: toma el mapa de conteos del punto 2 y le agrega una regla para las llaves que faltan. Por dentro, `withDefaultValue(0)` devuelve el mismo mapa envuelto en una cápsula que sabe responder cero. Las entradas que tenía el mapa siguen exactamente igual.

Lo que hay que entender bien es cuándo actúa ese cero. Solo actúa cuando se consulta con `c(k)`, es decir, con `apply`. Las otras operaciones del mapa no se enteran de que existe. Esto es lo que se obtuvo al probarlas sobre `c = conteoSeguro(registro)`:

| Operación | Resultado | Comentario |
|-----------|-----------|------------|
| `c("Valle")` | 3 | la llave está, se devuelve su valor |
| `c("Amazonas")` | 0 | la llave no está, se devuelve el valor por defecto |
| `c.get("Amazonas")` | `None` | `get` no usa el valor por defecto |
| `c.contains("Amazonas")` | `false` | la llave sigue sin existir |
| `c.size` | 3 | el mapa no creció |
| `c.keySet` | `Set(Valle, Santander, Choco)` | las llaves son las de siempre |

La fila de `size` es la importante: responder cero no es lo mismo que guardar un cero. El valor por defecto se devuelve y se olvida, y el mapa queda como estaba. Por eso las llaves de `conteoSeguro(r)` siguen siendo exactamente las regiones del registro.

Traza de `conteoSeguro(registro)("Valle")`:

| Paso | Qué pasa | Valor |
|------|----------|-------|
| 1 | se calcula `conteoPorRegion(registro)` | `Map(Valle -> 3, Santander -> 2, Choco -> 2)` |
| 2 | `withDefaultValue(0)` envuelve el mapa | los mismos conteos, con cero como respuesta de reserva |
| 3 | se consulta la llave `"Valle"` | la llave está en el mapa |
| 4 | se devuelve su valor | `3` |

Traza de `conteoSeguro(registro)("Amazonas")`. Los pasos 1 y 2 son los mismos, y cambia lo que sigue:

| Paso | Qué pasa | Valor |
|------|----------|-------|
| 3 | se consulta la llave `"Amazonas"` | la llave no está en el mapa |
| 4 | se usa el valor por defecto | `0` |
| 5 | el mapa queda como estaba | `size` sigue valiendo 3 |

Con el registro vacío, `conteoPorRegion(List())` es el mapa vacío, y cualquier consulta cae en el caso de la llave ausente: `conteoSeguro(List())("Valle")` da 0, y `conteoSeguro(List()).size` sigue siendo 0.

Se pueden hacer varias consultas seguidas sobre el mismo mapa sin que ninguna lo cambie:

| Consulta sobre el mismo `c` | ¿La llave está? | Respuesta | Tamaño del mapa después |
|-----------------------------|-----------------|-----------|-------------------------|
| `c("Valle")` | sí | 3 | 3 |
| `c("Amazonas")` | no | 0 | 3 |
| `c("Cauca")` | no | 0 | 3 |
| `c("Choco")` | sí | 2 | 3 |

```mermaid
flowchart TD
    A["consulta c(k)"] --> B{"la llave k<br/>está en el mapa?"}
    B -->|"sí"| C["se devuelve el conteo guardado"]
    B -->|"no"| D["se devuelve 0"]
    D --> E["el mapa no cambia,<br/>no se guarda la llave"]
```

### maximaDe

```scala
def maximaDe(r: Registro, region: String): Option[Double] =
  magnitudMaxima(r).get(region)
```

Aquí se hace lo contrario de lo anterior. En lugar de inventar una respuesta de reserva, se le deja a la función decir con claridad que no hay respuesta. Para eso sirve `Option`: un valor de tipo `Option[Double]` es una de dos cosas, `Some(m)`, que trae la magnitud, o `None`, que no trae nada.

`get` es la versión del mapa que nunca falla: si la llave está, devuelve `Some` con su valor, y si no está, devuelve `None`. La función entonces tiene dos pasos: calcula el mapa de máximas y le pregunta por la región con `get`.

Traza de `maximaDe(registro, "Choco")`:

| Paso | Qué pasa | Valor |
|------|----------|-------|
| 1 | se calcula `magnitudMaxima(registro)` | `Map(Valle -> 5.3, Santander -> 5.1, Choco -> 6.0)` |
| 2 | se llama `.get("Choco")` | la llave está |
| 3 | se devuelve `Some` con su valor | `Some(6.0)` |

Y varias consultas para ver los dos caminos:

| Consulta | ¿La región está? | Respuesta |
|----------|------------------|-----------|
| `maximaDe(registro, "Valle")` | sí | `Some(5.3)` |
| `maximaDe(registro, "Santander")` | sí | `Some(5.1)` |
| `maximaDe(registro, "Choco")` | sí | `Some(6.0)` |
| `maximaDe(registro, "Amazonas")` | no | `None` |
| `maximaDe(List(), "Valle")` | no, el registro está vacío | `None` |

La ventaja de `Option` es que quien llama a la función no puede olvidarse del caso vacío, porque el tipo se lo recuerda. Se puede usar de dos maneras:

```scala
maximaDe(registro, "Choco") match {
  case Some(m) => s"la máxima es $m"   // "la máxima es 6.0"
  case None => "sin datos"
}
maximaDe(registro, "Amazonas").getOrElse(0.0)   // 0.0
```

Con `Choco` se obtiene `la máxima es 6.0`, con `Amazonas` el `match` da `sin datos`, y el `getOrElse` devuelve `0.0`. La diferencia con `conteoSeguro` es que aquí es quien consulta, y no la función, el que decide qué hacer cuando no hay dato.

```mermaid
flowchart LR
    R["registro"] --> M["magnitudMaxima<br/>Map región -> máxima"]
    M --> G{"get(región)"}
    G -->|"la llave está"| S["Some(máxima)"]
    G -->|"la llave no está"| N["None"]
```

### regionMasActiva

```scala
def regionMasActiva(r: Registro): Option[String] = {
  val conteo = conteoPorRegion(r)
  if (conteo.isEmpty) None
  else {
    val ordenadas = conteo.keys.toList.sorted
    Some(ordenadas.maxBy(region => conteo(region)))
  }
}
```

La función avanza en cuatro pasos:

1. Calcula el conteo de sismos por región.
2. Si el conteo está vacío, no hay regiones y responde `None`.
3. Si no, ordena los nombres de las regiones alfabéticamente.
4. Recorre esa lista con `maxBy`, que busca la región con el mayor conteo, y responde `Some` con ella.
   El truco para el desempate está en el orden de los pasos 3 y 4. `maxBy` se queda con el primer elemento de la lista que alcanza el valor más grande. Para eso recorre la lista de izquierda a derecha guardando el mejor hasta el momento, y solo lo reemplaza cuando aparece uno estrictamente mayor. Un empate no lo reemplaza. Como la lista ya está en orden alfabético, el primero en llegar a ese valor es el que va antes en el alfabeto.

Traza con el registro del enunciado:

| Paso | Valor |
|------|-------|
| `conteo` | `Map(Valle -> 3, Santander -> 2, Choco -> 2)` |
| ¿está vacío? | no |
| `ordenadas` | `List(Choco, Santander, Valle)` |
| `maxBy`, ver tabla siguiente | `Valle` |
| resultado | `Some("Valle")` |

El recorrido de `maxBy`, región por región:

| Región | Conteo | Mejor hasta ahora | ¿Supera estrictamente al mejor? |
|--------|--------|-------------------|---------------------------------|
| Choco | 2 | Choco, es el primero | no hay con qué comparar |
| Santander | 2 | Choco | no, 2 no es mayor que 2 |
| Valle | 3 | Valle | sí, 3 es mayor que 2 |

Traza de un empate. Con los dos primeros sismos del registro, `registro.take(2)`, que son uno de Valle y uno de Santander:

| Paso | Valor |
|------|-------|
| `conteo` | `Map(Valle -> 1, Santander -> 1)` |
| `ordenadas` | `List(Santander, Valle)` |

| Región | Conteo | Mejor hasta ahora | ¿Supera estrictamente al mejor? |
|--------|--------|-------------------|---------------------------------|
| Santander | 1 | Santander, es el primero | no hay con qué comparar |
| Valle | 1 | Santander | no, 1 no es mayor que 1 |

El resultado es `Some("Santander")`. Vale la pena fijarse en que Valle aparece primero en el registro, pero gana Santander, porque el desempate es alfabético y no depende del orden del registro. Otro empate parecido, con un sismo de Zulia y otro de Arauca, da `Some("Arauca")`.

Traza con el registro vacío:

| Paso | Valor |
|------|-------|
| `conteo` | `Map()` |
| ¿está vacío? | sí |
| resultado | `None` |

La comprobación de que el conteo está vacío no es un adorno. Llamar `maxBy` sobre una lista vacía lanza `UnsupportedOperationException: empty.maxBy`, así que si no se revisara antes, `regionMasActiva(List())` se caería en lugar de devolver `None`.

Por último, como el resultado sale de una lista que se ordena primero, no depende de cómo el mapa entregue sus llaves. Con el registro al revés, `registro.reverse`, el resultado sigue siendo `Some("Valle")`.

```mermaid
flowchart TD
    A["conteoPorRegion(registro)"] --> B{"el conteo<br/>está vacío?"}
    B -->|"sí"| C["None"]
    B -->|"no"| D["ordenar las regiones<br/>alfabéticamente"]
    D --> E["maxBy: recorrer en orden<br/>y quedarse con la primera<br/>de mayor conteo"]
    E --> F["Some(región)"]
```
---
## Punto 4: réplicas perezosas
### Código

```scala
def replicas(magnitud: Double): LazyList[Double] = {
  def redondear(x: Double): Double = Math.round(x * 100.0) / 100.0
  LazyList.iterate(magnitud)(_ * 0.8).map(redondear)
}

def replicasSensibles(magnitud: Double, minimo: Double): List[Double] = {
  replicas(magnitud).takeWhile(_ > minimo).toList
}
```

### Qué hace cada operación

* `LazyList.iterate(magnitud)(_ * 0.8)` produce la secuencia infinita
  `magnitud, magnitud * 0.8, magnitud * 0.8 * 0.8, ...`. Es **perezosa**: guarda
  el primer valor y una "receta" para calcular el resto, pero no lo calcula.
* `.map(redondear)` redondea a dos decimales. En un `LazyList`, `map`
  también es perezoso: no recorre nada, solo anota que cada elemento, cuando
  se pida, debe pasar por `redondear`.
* `.takeWhile(_ > minimo)` entrega los elementos desde el principio y se
  detiene en el primero que **no** supera el mínimo. No mira ninguno después.
* `.toList` es lo que obliga a calcular. Recorre lo que `takeWhile` deja pasar
  y lo guarda en una lista normal, que ya es finita.
* `.take(4)` (en las pruebas) entrega los primeros cuatro elementos.

El redondeo se hace **después** de `iterate`, no dentro. Así cada valor sale de
multiplicar la magnitud original por $0.8^k$ y se redondea una sola vez; si se
redondeara en cada paso, el error se acumularía de réplica en réplica.

### Expresiones `for`

En los puntos 4 y 5 **no se usó ninguna expresión `for`**: la secuencia se
construye con `iterate`, `map` y `takeWhile`, que ya describen el _qué_ sin
recorrer nada a mano. No hay ciclos, `var`, `while` ni `return`.

### Ejecución paso a paso: `replicas(5.0).take(4).toList`

Los valores "crudos" son los que produce `iterate`; los "redondeados" son los
que salen de `map`.

| Posición $k$ | Valor crudo | Valor redondeado | ¿Se calcula? |
|:---:|:---:|:---:|:---|
| 0 | 5.0 | 5.0 | Sí |
| 1 | 4.0 | 4.0 | Sí |
| 2 | 3.2 | 3.2 | Sí |
| 3 | 2.5600000000000005 | 2.56 | Sí |
| 4 | 2.048... | 2.05 | **No**, `take(4)` ya no lo pide |
| 5, 6, ... | ... | ... | **No** |

Resultado: `List(5.0, 4.0, 3.2, 2.56)`. Se calculan exactamente cuatro
elementos, aunque la secuencia no termine nunca.

### Ejecución paso a paso: `replicasSensibles(5.0, 3.0)`

1. `replicas(5.0)` crea la secuencia, sin calcular ningún elemento.
2. `takeWhile(_ > 3.0)` se arma, todavía sin calcular nada.
3. `toList` pide el primer elemento: se calcula `5.0`. Como $5.0 > 3.0$, pasa.
4. Pide el segundo: `4.0`. Como $4.0 > 3.0$, pasa.
5. Pide el tercero: `3.2`. Como $3.2 > 3.0$, pasa.
6. Pide el cuarto: `2.56`. Como $2.56 > 3.0$ es falso, `takeWhile` **se detiene**.
   El quinto elemento nunca se calcula.
7. `toList` entrega `List(5.0, 4.0, 3.2)`.

Caso borde, `replicasSensibles(2.0, 3.0)`: el primer elemento es `2.0`, no
supera `3.0`, `takeWhile` se detiene de inmediato y el resultado es `List()`.
Solo se calculó un elemento.

### Por qué la secuencia infinita no agota la memoria

Una secuencia infinita **no cabe** en memoria si se construye completa; esta
nunca se construye completa. Hay tres razones:

1. **Cada elemento se calcula solo cuando alguien lo pide.** Un `LazyList` es
   una celda con la cabeza ya calculada y la cola como un cálculo pendiente.
   Mientras nadie pida la cola, ocupa una cantidad fija de memoria.
2. **`takeWhile` deja de pedir elementos apenas falla la condición.** Si hay
   $n$ réplicas sensibles, se calculan $n + 1$ elementos: los $n$ que pasan y
   el primero que falla. El resto de la secuencia, infinito, nunca existe. Con
   `take(4)` se calculan exactamente 4.
3. **`replicasSensibles` no guarda la secuencia.** `replicas(magnitud)` es una
   expresión intermedia que nadie conserva en una `val`; cuando `toList`
   termina, el resultado es una `List` finita de $n$ elementos y las celdas
   intermedias quedan libres para el recolector de basura. La memoria usada es
   proporcional a $n$, no al largo de la secuencia.

Una `List` normal no podría hacer esto: pedir "la lista infinita" calcularía
elementos para siempre y se agotaría la memoria. Por la misma razón, **nunca**
se debe llamar a `toList`, `size` o `foreach` sobre `replicas(m)` sin antes
limitarla con `take` o `takeWhile`.

### Diagramas

Cadena de transformaciones (nada se calcula hasta `toList`):

```mermaid
flowchart LR
    A["iterate: 5.0, 4.0, 3.2, 2.56, ... sin calcular"] --> B["map con redondear"]
    B --> C["takeWhile: x mayor que minimo"]
    C --> D["toList: aqui se calcula"]
    D --> E["List finita"]
```

Orden en que se piden los elementos en `replicasSensibles(5.0, 3.0)`:

```mermaid
sequenceDiagram
    participant T as toList
    participant W as takeWhile
    participant M as map
    participant I as iterate
    T->>W: siguiente
    W->>M: elemento 0
    M->>I: elemento 0
    I-->>M: 5.0
    M-->>W: 5.0
    W-->>T: pasa, 5.0 mayor que 3.0
    T->>W: siguiente
    W->>M: elemento 1
    M->>I: elemento 1
    I-->>M: 4.0
    M-->>W: 4.0
    W-->>T: pasa
    T->>W: siguiente
    W->>M: elemento 2
    M->>I: elemento 2
    I-->>M: 3.2
    M-->>W: 3.2
    W-->>T: pasa
    T->>W: siguiente
    W->>M: elemento 3
    M->>I: elemento 3
    I-->>M: 2.56
    M-->>W: 2.56
    W-->>T: falla, fin de la secuencia
```

---

## Punto 5: informe combinado
### Código

```scala
def informe(r: Registro, minimo: Double): Map[String, Resumen] = {
  porRegion(r).map { case (region, sismos) =>
    val maxima = sismos.map(_.magnitud).max
    region -> Resumen(sismos.length, maxima, replicasSensibles(maxima, minimo).length)
  }
}
```

`informe` no calcula nada por su cuenta: combina `porRegion` (punto 2) para
agrupar y `replicasSensibles` (punto 4) para contar las réplicas.

### Qué hace cada parte

* `porRegion(r)` es `r.groupBy(sismo => sismo.region)`: un
  `Map[String, List[Sismo]]` con una entrada por cada región que aparece en el
  registro.
* `.map { case (region, sismos) => ... }` visita cada entrada del mapa. El
  `case (region, sismos)` separa la llave del valor.
* `sismos.length` es el conteo de sismos de la región.
* `sismos.map(_.magnitud).max` es la magnitud máxima. Nunca falla por lista
  vacía, porque `groupBy` solo crea una región si tiene al menos un sismo.
* `replicasSensibles(maxima, minimo).length` cuenta las réplicas sensibles del
  sismo más fuerte de la región.
* `region -> Resumen(...)` crea el par `(region, Resumen(...))`. Un `map` sobre
  un `Map` que devuelve pares produce otro `Map`.

### Expresiones `for`

No se usó `for`. Si se escribiera con `for`, sería:

```scala
for ((region, sismos) <- porRegion(r)) yield {
  val maxima = sismos.map(_.magnitud).max
  region -> Resumen(sismos.length, maxima, replicasSensibles(maxima, minimo).length)
}
```

y el compilador lo traduce a `porRegion(r).map { case (region, sismos) => ... }`,
que es justo la versión que se entregó (con un generador y `yield` solo
aparece `map`; el patrón de tupla puede añadir un `withFilter` que deja pasar
todo).

### Ejecución paso a paso: `informe(registro, 3.0)`

1. `porRegion(registro)` agrupa los siete sismos:

   | Región | Sismos (magnitudes) |
      |---|---|
   | Valle | 4.2, 3.8, 5.3 |
   | Santander | 5.1, 4.7 |
   | Choco | 6.0, 3.1 |

2. Para cada región se calcula el resumen:

   | Región | Conteo | Máxima | Réplicas de la máxima (mayores que 3.0) | Cantidad |
      |---|:---:|:---:|---|:---:|
   | Valle | 3 | 5.3 | 5.3, 4.24, 3.39 (la siguiente, 2.71, no pasa) | 3 |
   | Santander | 2 | 5.1 | 5.1, 4.08, 3.26 (la siguiente, 2.61, no pasa) | 3 |
   | Choco | 2 | 6.0 | 6.0, 4.8, 3.84, 3.07 (la siguiente, 2.46, no pasa) | 4 |

3. Se arman los pares y el `map` los reúne en un solo mapa:
   `Map("Valle" -> Resumen(3, 5.3, 3), "Santander" -> Resumen(2, 5.1, 3), "Choco" -> Resumen(2, 6.0, 4))`.

Caso borde, `informe(List(), 3.0)`: `groupBy` sobre una lista vacía da un mapa
vacío, el `map` no tiene entradas que visitar, y el resultado es `Map()`.

Observación: el orden en que se muestran las regiones dentro del `Map` no es
parte del resultado. Dos mapas con las mismas llaves y los mismos valores son
iguales sin importar cómo se impriman.

### Diagrama

```mermaid
flowchart TD
    R["registro"] --> P["porRegion: groupBy region"]
    P --> G["Map: region a lista de sismos"]
    G --> E["por cada region"]
    E --> C["conteo: length"]
    E --> X["maxima: map magnitud y max"]
    X --> S["replicasSensibles de la maxima"]
    S --> N["cantidad de replicas: length"]
    C --> U["Resumen"]
    X --> U
    N --> U
    U --> I["Map: region a Resumen"]
```

