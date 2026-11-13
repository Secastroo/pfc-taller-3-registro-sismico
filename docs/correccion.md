## Punto 1: consultas con expresiones for

### 1.1 Especificaciones

**`fuertes`.** Sea $u$ el umbral y sean $i_1 < i_2 < \cdots < i_k$ exactamente
los índices $i$ con $\mu(s_i) > u$. Entonces:

```math
f_{\text{fuertes}}(r, u) = \text{List}\big(\rho(s_{i_1}), \ldots, \rho(s_{i_k})\big)
```

Es decir: las regiones de los sismos que superan el umbral, en el orden del
registro y **sin quitar repetidos**.

**`enRegion`.** Para una región $g$:

```math
f_{\text{enRegion}}(r, g) = \text{List}\big(\mu(s_{i_1}), \ldots, \mu(s_{i_k})\big), \quad i_1 < \cdots < i_k, \quad \{i_1, \ldots, i_k\} = \{\, i \mid \rho(s_i) = g \,\}
```

**`paresCercanos`.** Sea $d \geq 0$ la diferencia permitida. Sea
$G(r) = (g_1, \ldots, g_m)$ la lista de las regiones de $R(r)$ sin repetir,
ordenadas por su primera aparición en $r$, es decir,
$\text{prim}(g_1) < \cdots < \text{prim}(g_m)$ con
$\text{prim}(g) = \min\{\, i \mid \rho(s_i) = g \,\}$. Para dos regiones $a, b$:

```math
C(a, b) \iff \exists\, s, t \in r : \rho(s) = a \ \wedge\ \rho(t) = b \ \wedge\ |\mu(s) - \mu(t)| \leq d
```

Entonces:

```math
f_{\text{pares}}(r, d) = \text{List}\big( (g_i, g_j) \;\mid\; 1 \leq i < j \leq m,\ C(g_i, g_j) \big)
```

ordenada lexicográficamente por $(i, j)$.

### 1.2 Corrección de `fuertes`

El programa es:

```scala
for { s <- r; if s.magnitud > umbral } yield s.region
```

que, por la traducción del compilador, es $P_{\text{fuertes}}(r, u) = \text{map}_{\rho}(\text{filtrar}_{p}(r))$
con $p(s) \equiv \mu(s) > u$.

Vamos a demostrar que $\forall r : P_{\text{fuertes}}(r, u) == f_{\text{fuertes}}(r, u)$.
Se usa inducción estructural sobre $r$.

**Caso base:** $r = \text{List}()$

```math
P_{\text{fuertes}}(\text{List}(), u) \rightarrow \text{map}_{\rho}(\text{filtrar}_{p}(\text{List}())) \rightarrow \text{map}_{\rho}(\text{List}()) \rightarrow \text{List}()
```

Por otro lado, no hay índices, así que $f_{\text{fuertes}}(\text{List}(), u) = \text{List}()$.
Entonces $P_{\text{fuertes}}(\text{List}(), u) == f_{\text{fuertes}}(\text{List}(), u)$.

**Caso de inducción:** $r = s :: r'$. La hipótesis de inducción (HI) es
$P_{\text{fuertes}}(r', u) == f_{\text{fuertes}}(r', u)$. Hay dos posibilidades:

- Si $\mu(s) > u$:

```math
P_{\text{fuertes}}(s :: r', u) \rightarrow \text{map}_{\rho}(s :: \text{filtrar}_p(r')) \rightarrow \rho(s) :: \text{map}_{\rho}(\text{filtrar}_p(r')) = \rho(s) :: P_{\text{fuertes}}(r', u) \overset{\text{HI}}{=} \rho(s) :: f_{\text{fuertes}}(r', u)
```

Por otro lado, el índice 1 cumple la condición y es el menor de todos; los
demás índices seleccionados son los de $r'$, desplazados en uno. Entonces
$f_{\text{fuertes}}(s :: r', u) = \rho(s) :: f_{\text{fuertes}}(r', u)$.

- Si $\mu(s) \leq u$:

```math
P_{\text{fuertes}}(s :: r', u) \rightarrow \text{map}_{\rho}(\text{filtrar}_p(r')) = P_{\text{fuertes}}(r', u) \overset{\text{HI}}{=} f_{\text{fuertes}}(r', u)
```

Y el índice 1 no se selecciona, de modo que $f_{\text{fuertes}}(s :: r', u) = f_{\text{fuertes}}(r', u)$.

En ambos casos $P_{\text{fuertes}}(r, u) == f_{\text{fuertes}}(r, u)$.

Concluimos por inducción que $\forall r : P_{\text{fuertes}}(r, u) == f_{\text{fuertes}}(r, u)$.  

**Dos consecuencias que pide el enunciado.**

- **Repetidos:** el programa no aplica ninguna eliminación de duplicados. Si
  $\rho(s_i) = \rho(s_j)$ con $i \neq j$ y ambos superan el umbral, la región
  aparece dos veces, porque la longitud del resultado es
  $\text{len}(f_{\text{fuertes}}(r, u)) = |\{\, i \mid \mu(s_i) > u \,\}|$.
- **Umbral estricto:** se usa $>$, así que un sismo con $\mu(s) = u$ no entra.

### 1.3 Corrección de `enRegion`

El programa es:

```scala
for { s <- r; if s.region == region } yield s.magnitud
```

es decir, $P_{\text{enRegion}}(r, g) = \text{map}_{\mu}(\text{filtrar}_{q}(r)) = \text{map}_{\mu}(S_g(r))$
con $q(s) \equiv \rho(s) = g$.

Vamos a demostrar que $\forall r : P_{\text{enRegion}}(r, g) == f_{\text{enRegion}}(r, g)$.

**Caso base:** $r = \text{List}()$

```math
P_{\text{enRegion}}(\text{List}(), g) \rightarrow \text{map}_{\mu}(S_g(\text{List}())) \rightarrow \text{map}_{\mu}(\text{List}()) \rightarrow \text{List}()
```

Y $f_{\text{enRegion}}(\text{List}(), g) = \text{List}()$, porque no hay índices.

**Caso de inducción:** $r = s :: r'$. La HI es
$P_{\text{enRegion}}(r', g) == f_{\text{enRegion}}(r', g)$.

- Si $\rho(s) = g$: $S_g(s :: r') = s :: S_g(r')$, luego

```math
P_{\text{enRegion}}(s :: r', g) \rightarrow \text{map}_{\mu}(s :: S_g(r')) \rightarrow \mu(s) :: P_{\text{enRegion}}(r', g) \overset{\text{HI}}{=} \mu(s) :: f_{\text{enRegion}}(r', g)
```

que es $f_{\text{enRegion}}(s :: r', g)$, porque el índice 1 se selecciona y es el menor.

- Si $\rho(s) \neq g$: $S_g(s :: r') = S_g(r')$, luego
  $P_{\text{enRegion}}(s :: r', g) = P_{\text{enRegion}}(r', g) \overset{\text{HI}}{=} f_{\text{enRegion}}(r', g) = f_{\text{enRegion}}(s :: r', g)$.

Concluimos por inducción que $\forall r : P_{\text{enRegion}}(r, g) == f_{\text{enRegion}}(r, g)$.  

**Región ausente.** Si $g \notin R(r)$, entonces $S_g(r) = \text{List}()$ y
$\text{enRegion}(r, g) = \text{List}()$, que es lo que pide el enunciado.

### 1.4 Corrección de `paresCercanos`

Se demuestra en tres lemas y un teorema.

**Lema 1 (regiones sin repetir).** En el programa,
`regiones` $= G(r)$ y `indexadas` $= \text{List}((g_1, 0), \ldots, (g_m, m-1))$.

_Demostración._ `(for (s <- r) yield s.region)` es $\text{map}_{\rho}(r)$,
la lista $(\rho(s_1), \ldots, \rho(s_n))$. Por definición, `distinct` conserva la
**primera aparición** de cada elemento y su orden relativo, así que entrega
$(g_1, \ldots, g_m)$ con $\text{prim}(g_1) < \cdots < \text{prim}(g_m)$, que es $G(r)$.
`zipWithIndex` agrega el índice contando desde 0. Si llamamos $i$ al índice de
la matemática (desde 1) y $i'$ al del código, $i' = i - 1$, y la condición
$i' < j'$ equivale a $i < j$.  

**Lema 2 (el `for` interno).** Sea
$\text{cercanos}(a, b)$ la lista que produce el `for` interno. Entonces

```math
\text{cercanos}(a, b) \neq \text{List}() \iff C(a, b)
```

_Demostración._ El `for` interno se traduce a

```math
\text{cercanos}(a, b) = \text{flatMap}_{\,h}\big(S_a(r)\big), \qquad h(s_1) = \text{map}_{(s_1, \cdot)}\big(\text{filtrar}_{\,|\mu(s_1) - \mu(\cdot)| \leq d}(S_b(r))\big)
```

Por la ecuación de $\text{flatMap}$, el resultado es la concatenación de los
bloques $h(s_1)$ para cada $s_1 \in S_a(r)$. Una concatenación es no vacía si y
solo si algún bloque es no vacío. A su vez, $h(s_1)$ es no vacío si y solo si
existe $s_2 \in S_b(r)$ con $|\mu(s_1) - \mu(s_2)| \leq d$. Por lo tanto,
$\text{cercanos}(a, b) \neq \text{List}()$ si y solo si existen
$s_1 \in S_a(r)$ y $s_2 \in S_b(r)$ con esa propiedad, que es $C(a, b)$.  

**Lema 3 (los generadores y la guardia).** Los generadores
`(a, i) <- indexadas` y `(b, j) <- indexadas` recorren los pares
$(g_i, g_j)$ de $G(r) \times G(r)$ en orden lexicográfico de $(i, j)$, y la
guardia $i < j$ deja exactamente los pares con $i < j$.

_Demostración._ El `for` externo se traduce a un $\text{flatMap}$ sobre los
elementos de `indexadas`, en orden creciente de $i$. Para cada $i$, el segundo
generador recorre `indexadas` en orden creciente de $j$. Por la ecuación de
$\text{flatMap}$, el resultado es la concatenación, en orden creciente de $i$, de
los bloques ordenados por $j$: es decir, orden lexicográfico de $(i, j)$. La
guardia $i < j$ descarta los demás.  

**Teorema.** $\forall r : P_{\text{pares}}(r, d) == f_{\text{pares}}(r, d)$.

_Demostración._ Por el Lema 1, los pares sobre los que se itera son los de
$G(r)$ con sus índices. Por el Lema 3, se consideran exactamente los pares
$(g_i, g_j)$ con $i < j$, en orden lexicográfico de $(i, j)$. Por el Lema 2,
la guardia `cercanos.nonEmpty` deja pasar el par si y solo si $C(g_i, g_j)$.
Por último, `yield (a, b)` devuelve $(g_i, g_j)$ sin cambiar el orden. El
resultado es entonces la lista de los $(g_i, g_j)$ con $i < j$ y $C(g_i, g_j)$,
ordenados lexicográficamente por $(i, j)$, que es $f_{\text{pares}}(r, d)$.  

**Propiedades que pide el enunciado.**

- **Regiones distintas:** los elementos de $G(r)$ son distintos entre sí, así
  que $i < j$ implica $g_i \neq g_j$. Nunca se empareja una región consigo misma.
- **Cada pareja una sola vez:** cada pareja no ordenada $\{g_i, g_j\}$ con
  $i \neq j$ tiene exactamente una representación con $i < j$. Por eso, si sale
  $(\text{Valle}, \text{Choco})$, no sale $(\text{Choco}, \text{Valle})$.
- **Orden de aparición:** $i < j$ significa $\text{prim}(g_i) < \text{prim}(g_j)$, es
  decir, la primera región del par es la que aparece antes en el registro.
- **Simetría de $C$:** como $|x - y| = |y - x|$, se cumple $C(a, b) \iff C(b, a)$,
  así que evaluar cada pareja en un solo orden no pierde ni inventa resultados.
- **Casos vacíos:** si ninguna pareja de sismos de regiones distintas cumple
  $|\mu(s) - \mu(t)| \leq d$, entonces $C$ es falso para todo par y el resultado
  es $\text{List}()$. Con una sola región, $m = 1$ y no existen pares con $i < j$.
  Si $d < 0$, la condición $|\cdot| \leq d$ es imposible y el resultado también
  es $\text{List}()$.

### 1.5 Verificación con los ejemplos del enunciado

Con el registro del enunciado, $G(r) = (\text{Valle}, \text{Santander}, \text{Choco})$.

| $d$ | Parejas $(g_i, g_j)$ con $C$ | Resultado |
|:---:|---|---|
| 0.1 | ninguna | `List()` |
| 0.6 | (Valle, Santander), pues $\lvert 5.3 - 5.1 \rvert = 0.2 \leq 0.6$ | `List(("Valle", "Santander"))` |
| 1.0 | las tres: $0.2$, $0.7$ y $0.9$ son $\leq 1.0$ | `List(("Valle", "Santander"), ("Valle", "Choco"), ("Santander", "Choco"))` |

Y para `fuertes` y `enRegion`:

```math
f_{\text{fuertes}}(r, 5.0): \ \mu > 5.0 \text{ en } s_2, s_4, s_6 \Rightarrow \text{List}(\text{Santander}, \text{Choco}, \text{Valle})
```

```math
f_{\text{enRegion}}(r, \text{Valle}): \ \rho = \text{Valle} \text{ en } s_1, s_3, s_6 \Rightarrow \text{List}(4.2, 3.8, 5.3)
```

### 1.6 Independencia del orden de recorrido

**Teorema.** Sea $r'$ una permutación de $r$ (los mismos sismos, en otro orden).

1. $\text{fuertes}(r', u)$ es una permutación de $\text{fuertes}(r, u)$. Lo
   mismo vale para $\text{enRegion}(r', g)$ y $\text{enRegion}(r, g)$.
2. El conjunto de parejas **no ordenadas** de $\text{paresCercanos}(r', d)$ es
   igual al de $\text{paresCercanos}(r, d)$.
3. Si además la permutación conserva el orden de primera aparición de las
   regiones, entonces $\text{paresCercanos}(r', d) = \text{paresCercanos}(r, d)$
   como listas.

_Demostración._

1. La decisión de incluir un sismo $s$ depende solo de $s$ (de $\mu(s) > u$ o de
   $\rho(s) = g$), nunca de los demás sismos ni del momento en que se evalúa.
   Entonces el multiconjunto de sismos seleccionados es
   $\{\!\{\, s \in r \mid p(s) \,\}\!\}$, y una permutación de $r$ no cambia los
   elementos, solo su posición. Aplicar $\rho$ o $\mu$ a cada uno da el mismo
   multiconjunto de resultados. Lo único que depende del orden es la posición
   en la lista, y por especificación es la del registro.
2. $R(r') = R(r)$, y para cada región $S_a(r')$ es una permutación de $S_a(r)$.
   La condición
   $C(a, b) \iff \exists s \in S_a, \exists t \in S_b : |\mu(s) - \mu(t)| \leq d$
   es una proposición existencial: la disyunción de proposiciones es
   conmutativa y asociativa, así que su valor no cambia al reordenar los sismos
   que se examinan, ni al intercambiar el orden de los dos generadores internos.
   Entonces $C$ es la misma para $r$ y $r'$, y también el conjunto de parejas
   no ordenadas $\{a, b\}$ que la cumplen.
3. $f_{\text{pares}}$ depende de $r$ solo a través de $G(r)$ y de $C$. $C$ no
   cambia por el punto 2, y $G(r') = G(r)$ cuando se conserva el orden de primera
   aparición de las regiones. Luego las dos listas son iguales.  

Esto garantiza que la manera en que el compilador evalúe el `for` (qué sismo
mira primero, en qué orden recorre los generadores internos) no altera lo que
se busca.

### 1.7 Precisión numérica

Las diferencias $|\mu(s) - \mu(t)|$ se calculan en `Double`, así que pueden
traer errores del orden de $10^{-16}$. La demostración es exacta sobre los
reales. En `Double` solo podría cambiar un resultado si una diferencia cae
prácticamente sobre el valor de `dif`; en los ejemplos del enunciado las
diferencias relevantes (0.2, 0.7 y 0.9) están lejos de los valores de `dif`
usados (0.1, 0.6 y 1.0).
## Punto 2: agrupar con mapas

Este punto usa la notación común. Resume el registro por región con tres
funciones: `porRegion`, `conteoPorRegion` y `magnitudMaxima`.

### 2.1 Especificación

Para un registro $r$, el enunciado pide:

```math
\text{porRegion}(r) = \{\, g \mapsto S_g(r) \mid g \in R(r) \,\}
```

```math
\text{conteoPorRegion}(r) = \{\, g \mapsto \text{len}(S_g(r)) \mid g \in R(r) \,\}
```

```math
\text{magnitudMaxima}(r) = \{\, g \mapsto \max\{\mu(s) \mid s \in S_g(r)\} \mid g \in R(r) \,\}
```

Las llaves de cada mapa son exactamente las regiones que aparecen en $r$.
Una región que no está en el registro no aparece, y para $r = \text{List}()$
los tres mapas son $\text{Map}()$. En `porRegion`, los sismos de cada lista
quedan en el mismo orden en que aparecen en $r$.

### 2.2 El programa

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

### 2.3 Corrección de `porRegion`

`foldLeft` es un proceso iterativo: va sismo por sismo y guarda lo que lleva
armado en un mapa. Se argumenta como un programa iterativo, con estado,
invariante y transformación. Sea $r = \text{List}(s_1, \ldots, s_n)$ y
$p_i = \text{List}(s_1, \ldots, s_{i-1})$ los sismos que ya se procesaron.

**Hecho útil.** Sea $p$ una lista y $s$ un sismo con región $g_0 = \rho(s)$.
Al agregar $s$ al final de $p$:

```math
S_g(p \mathbin{:+} s) = \begin{cases} S_g(p) \mathbin{:+} s & \text{si } g = g_0 \\ S_g(p) & \text{si } g \neq g_0 \end{cases}
\qquad R(p \mathbin{:+} s) = R(p) \cup \{g_0\}
```

Esto pasa porque filtrar por la región $g$ solo deja pasar a $s$ cuando
$\rho(s) = g$, y $s$ queda de último.

**El proceso iterativo.**

- Un estado $e = (m, l)$: $m$ es el mapa armado hasta ahora y
  $l = \text{List}(s_i, \ldots, s_n)$ son los sismos que faltan.
- El estado inicial es $e_0 = (\text{Map}(), r)$.
- El estado es final si $l$ es vacía.
- La invariante es:

```math
\text{Inv}(m, l) \equiv l = \text{List}(s_i, \ldots, s_n) \land \text{llaves}(m) = R(p_i) \land \forall g \in R(p_i) : m(g) = S_g(p_i)
```

- $\text{transformar}((m, l)) = (m', l.\text{tail})$, donde, con
  $g_0 = \rho(s_i)$ y $s_i = l.\text{head}$:

```math
m' = m + \bigl(g_0 \mapsto m.\text{getOrElse}(g_0, \text{List}()) \mathbin{:+} s_i\bigr)
```

**Demostración.**

**1.** $\text{Inv}(e_0)$. Aquí $i = 1$ y $p_1 = \text{List}()$, así que
$R(p_1) = \emptyset$. El mapa $\text{Map}()$ no tiene llaves y no hay nada
más que cumplir. Además $l = r$.

**2.** La invariante se mantiene. Se supone $\text{Inv}(m, l)$ y
$l \neq \text{List}()$. Sea $g_0 = \rho(s_i)$ y $p_{i+1} = p_i \mathbin{:+} s_i$.

- Si $g \neq g_0$: $m'(g) = m(g)$, porque $m'$ solo cambia la llave $g_0$.
  Por la invariante $m(g) = S_g(p_i)$, y por el hecho útil
  $S_g(p_i) = S_g(p_{i+1})$.
- Si $g_0 \in R(p_i)$: por la invariante $m(g_0) = S_{g_0}(p_i)$, y
  $\text{getOrElse}$ devuelve esa lista. Entonces
  $m'(g_0) = S_{g_0}(p_i) \mathbin{:+} s_i = S_{g_0}(p_{i+1})$.
- Si $g_0 \notin R(p_i)$: no hay llave $g_0$, $\text{getOrElse}$ devuelve
  $\text{List}()$ y $m'(g_0) = \text{List}(s_i)$. Como $S_{g_0}(p_i)$ es vacía,
  $S_{g_0}(p_{i+1}) = \text{List}(s_i)$.
- Las llaves de $m'$ son las de $m$ más $g_0$, es decir
  $R(p_i) \cup \{g_0\} = R(p_{i+1})$. Además
  $l.\text{tail} = \text{List}(s_{i+1}, \ldots, s_n)$.

Por lo tanto $\text{Inv}(m', l.\text{tail})$.

**3.** $\text{Inv}(e_f) \rightarrow \text{respuesta}(e_f) == f(r)$. En el estado
final $l$ es vacía, así que $i = n + 1$ y $p_{n+1} = r$. La invariante dice
que las llaves de $m$ son $R(r)$ y que $m(g) = S_g(r)$. Eso es
$\text{porRegion}(r)$ según la especificación.

**4.** En cada paso $l$ pierde un elemento (`l.tail`). Después de $n$ pasos
queda vacía, así que siempre se llega al estado final.

**Teorema 2.1.** Para todo registro $r$,
$\text{porRegion}(r) = \{\, g \mapsto S_g(r) \mid g \in R(r) \,\}$.
 

Con $r = \text{List}()$ el resultado es $\text{Map}()$. Una región ausente no
tiene llave, porque solo se agrega la región del sismo que se está procesando.

### 2.4 Corrección de `conteoPorRegion`

**Teorema 2.2.** Para todo registro $r$,
$\text{conteoPorRegion}(r) = \{\, g \mapsto \text{len}(S_g(r)) \mid g \in R(r) \,\}$.

_Demostración._

1. Por el Teorema 2.1, `grupos` tiene un par $(g, S_g(r))$ por cada
   $g \in R(r)$, y las llaves son distintas.
2. `map` sobre un mapa aplica la función a cada par. Aquí devuelve
   $(g, \text{len}(S_g(r)))$, que conserva la llave. Como las llaves de entrada
   eran distintas, las de salida también, y ninguna entrada pisa a otra.
3. `sismos.length` es $\text{len}(S_g(r))$, el valor pedido.
4. Para $r = \text{List}()$, `grupos` es $\text{Map}()$. `map` no visita ninguna
   entrada y devuelve $\text{Map}()$.  

### 2.5 Corrección de `magnitudMaxima`

**Teorema 2.3.** Para todo registro $r$,
$\text{magnitudMaxima}(r) = \{\, g \mapsto \max\{\mu(s) \mid s \in S_g(r)\} \mid g \in R(r) \,\}$.

_Demostración._

1. Por el Teorema 2.1, `grupos` tiene un par $(g, S_g(r))$ por cada
   $g \in R(r)$.
2. Si $g \in R(r)$, existe un sismo $s \in r$ con $\rho(s) = g$, así que
   $s \in S_g(r)$ y la lista no es vacía. Por eso `max` está definido y no
   lanza excepción.
3. `sismos.map(sismo => sismo.magnitud)` aplica $\mu$ a cada sismo y da la
   lista $\text{List}(\mu(s) \mid s \in S_g(r))$.
4. `max` de una lista no vacía devuelve su mayor elemento. Es el mismo
   argumento del ejemplo "el máximo de una lista" de la sección 1. Entonces el
   valor es $M_g = \max\{\mu(s) \mid s \in S_g(r)\}$.
5. La función del `map` devuelve $(g, M_g)$, que conserva la llave, igual que
   en el Teorema 2.2. Para $r = \text{List}()$ el resultado es $\text{Map}()$.
    

Comprobación con `registro`:

| Región | $S_g$ (magnitudes) | $\text{len}(S_g)$ | $M_g$ |
|---|---|:---:|:---:|
| Valle | 4.2, 3.8, 5.3 | 3 | 5.3 |
| Santander | 5.1, 4.7 | 2 | 5.1 |
| Choco | 6.0, 3.1 | 2 | 6.0 |

que coincide con los resultados del enunciado.

### 2.6 El resultado no depende del orden de recorrido

**Teorema 2.4.** Si $r'$ es una permutación de $r$ (los mismos sismos, en otro
orden), entonces:

```math
\text{conteoPorRegion}(r') = \text{conteoPorRegion}(r)
\qquad
\text{magnitudMaxima}(r') = \text{magnitudMaxima}(r)
```

_Demostración._

1. $R(r') = R(r)$: ambos conjuntos contienen las regiones de los mismos
   sismos. Las llaves de los mapas coinciden.
2. Para cada región $g$, $S_g(r')$ es una permutación de $S_g(r)$, porque
   filtrar por región selecciona los mismos sismos. Una permutación no cambia
   la longitud, así que $\text{len}(S_g(r')) = \text{len}(S_g(r))$.
3. El máximo no depende del orden. La operación $\max$ es conmutativa
   ($\max(a,b) = \max(b,a)$) y asociativa
   ($\max(\max(a,b),c) = \max(a,\max(b,c))$), así que cualquier forma de
   combinar las magnitudes da el mismo valor: el único que es mayor o igual
   que todos los demás. Entonces $M_g(r') = M_g(r)$.
4. Dos mapas son iguales cuando tienen las mismas llaves y el mismo valor para
   cada llave. El orden interno de las entradas no cuenta. Por 1, 2 y 3, todas
   las entradas coinciden.  

**`porRegion`.** Tiene las mismas llaves y cada región recibe los mismos
sismos. Lo único que cambia es el orden dentro de cada lista, porque por el
Teorema 2.1 ese orden es el que tienen los sismos en el registro. Esa es
exactamente la especificación, y por eso el conteo y el máximo, que no miran
ese orden, no cambian. La prueba `propia punto 2: el resultado no cambia si el
registro se recorre al revés` comprueba esto con `reverse`.

---
## 3. Punto 3: consultas seguras

En este punto usamos la notación de la sección 0 ($\text{Reg}(R)$, $R_k$, $\text{cuenta}(k, R)$ y $\text{maxima}(k, R)$) y los resultados del punto 2: las llaves de $\text{conteoPorRegion}(R)$ y de $\text{magnitudMaxima}(R)$ son exactamente $\text{Reg}(R)$, y para cada una de ellas sus valores son $\text{cuenta}(k, R)$ y $\text{maxima}(k, R)$.

De los mapas damos por sabido cómo se comporta un mapa $M$ con valor por defecto $d$. Escribimos $M_d$ para ese mapa y $\text{llaves}(M)$ para el conjunto de sus llaves:

```math
M_d(k) =
\begin{cases}
M(k) & \text{si } k \in \text{llaves}(M) \\
d & \text{si no}
\end{cases}
\qquad
\text{llaves}(M_d) = \text{llaves}(M)
```

Y $M.\text{get}(k)$ es $\text{Some}(M(k))$ si $k \in \text{llaves}(M)$, y $\text{None}$ si no.

### 3.0 Un hecho auxiliar: una región ausente tiene un grupo vacío

Las tres funciones se apoyan en que una región que no aparece en el registro es lo mismo que una región con la sublista $R_k$ vacía. Lo demostramos:

```math
\forall R,\ \forall k : \quad k \notin \text{Reg}(R) \iff R_k = [\,]
```

Por inducción estructural sobre $R$.

Caso base: $R = [\,]$. Por definición $\text{Reg}([\,]) = \emptyset$, así que $k \notin \text{Reg}([\,])$ es verdadero, y $[\,]_k = [\,]$ también. Las dos partes de la equivalencia son verdaderas.

Caso de inducción: $R = s :: R'$, con la hipótesis de inducción (HI) $k \notin \text{Reg}(R') \iff R'_k = [\,]$. Sabemos que $\text{Reg}(s :: R') = \{\text{reg}(s)\} \cup \text{Reg}(R')$. Hay dos posibilidades:

* Si $\text{reg}(s) = k$: entonces $k \in \text{Reg}(R)$, y la parte izquierda es falsa. Por otro lado $R_k = s :: R'_k$, que no es vacía, y la parte derecha también es falsa. La equivalencia se cumple entre dos falsedades.
* Si $\text{reg}(s) \neq k$: entonces $k \in \text{Reg}(R) \iff k \in \text{Reg}(R')$, y además $R_k = R'_k$. La equivalencia que se quiere es entonces la misma de la HI.

Concluimos por inducción que $k \notin \text{Reg}(R) \iff R_k = [\,]$, para todo registro y toda región. En particular, para una región ausente, $\text{cuenta}(k, R) = |R_k| = |[\,]| = 0$.

### 3.1 conteoSeguro

Lo que se pide es que el mapa responda, para cualquier cadena $k$, incluso para una región que no está en el registro, la cantidad de sismos de esa región:

```math
\forall R,\ \forall k : \quad \text{conteoSeguro}(R)(k) = \text{cuenta}(k, R)
```

Con el modelo de sustitución, el programa `conteoPorRegion(r).withDefaultValue(0)` es el mapa $\big(\text{conteoPorRegion}(R)\big)_0$:

```math
\text{conteoSeguro}(R)(k) \rightarrow \big(\text{conteoPorRegion}(R)\big)_0(k)
```

Demostramos la igualdad por casos sobre $k$.

Caso $k \in \text{Reg}(R)$: la llave está en el mapa, porque las llaves de $\text{conteoPorRegion}(R)$ son $\text{Reg}(R)$ por el punto 2. Entonces el valor por defecto no se usa:

```math
\big(\text{conteoPorRegion}(R)\big)_0(k) \rightarrow \text{conteoPorRegion}(R)(k) = |R_k| = \text{cuenta}(k, R)
```

Caso $k \notin \text{Reg}(R)$: la llave no está en el mapa, así que se responde el valor por defecto:

```math
\big(\text{conteoPorRegion}(R)\big)_0(k) \rightarrow 0
```

Y por el hecho auxiliar 3.0, $R_k = [\,]$, de modo que $\text{cuenta}(k, R) = 0$. Los dos lados coinciden.

Concluimos que $\text{conteoSeguro}(R)(k) = \text{cuenta}(k, R)$ para todo $R$ y toda cadena $k$.  

Esta demostración deja tres consecuencias que el enunciado y las pruebas usan:

La primera es que el mapa no agrega llaves. Como $\text{llaves}(M_d) = \text{llaves}(M)$, las llaves de $\text{conteoSeguro}(R)$ siguen siendo $\text{Reg}(R)$. Consultar por una región ausente devuelve un valor, pero no construye un mapa nuevo ni guarda nada. Tampoco cambia lo que responden $\text{get}$ y $\text{contains}$, que siguen diciendo que la llave no está.

La segunda es el caso del registro vacío. Si $R = [\,]$, entonces $\text{Reg}(R) = \emptyset$, y toda región cae en el segundo caso. Por eso $\text{conteoSeguro}([\,])(k) = 0$ para toda $k$.

La tercera es que los conteos suman el tamaño del registro:

```math
\sum_{k \in \text{Reg}(R)} \text{conteoSeguro}(R)(k) = |R|
```

Por el primer caso de la demostración, esa suma es $S(R) = \sum_{k \in \text{Reg}(R)} \text{cuenta}(k, R)$. Antes de demostrarla, observemos que, por el hecho 3.0, $\text{cuenta}(k, R) = 0$ si $k \notin \text{Reg}(R)$, así que para cualquier conjunto finito $K \supseteq \text{Reg}(R)$ se tiene $S(R) = \sum_{k \in K} \text{cuenta}(k, R)$: las regiones de más suman cero. Demostramos $S(R) = |R|$ por inducción sobre $R$.

Caso base: $R = [\,]$. No hay regiones, la suma es vacía y $S([\,]) = 0 = |[\,]|$.

Caso de inducción: $R = s :: R'$, con la HI $S(R') = |R'|$. Tomamos $K = \text{Reg}(R)$, que contiene a $\text{Reg}(R')$. Por la definición de $R_k$, un sismo agregado al frente sube en 1 la cuenta de su región y no toca las demás:

```math
\text{cuenta}(k, s :: R') = \text{cuenta}(k, R') + \mathbb{1}[k = \text{reg}(s)]
```

Entonces:

```math
S(R) = \sum_{k \in K} \Big(\text{cuenta}(k, R') + \mathbb{1}[k = \text{reg}(s)]\Big) = \sum_{k \in K} \text{cuenta}(k, R') + \sum_{k \in K} \mathbb{1}[k = \text{reg}(s)]
```

La primera suma es $S(R')$ por la observación de las regiones de más, y la segunda vale 1, porque $\text{reg}(s)$ está en $K$ una sola vez:

```math
S(R) = S(R') + 1 \overset{\text{HI}}{=} |R'| + 1 = |R|
```

Concluimos por inducción que los conteos de todas las regiones suman exactamente el número de sismos del registro. Es lo que revisa la prueba `conteoPorRegion y magnitudMaxima: valores sueltos del registro completo`, con $7$ sismos.

### 3.2 maximaDe

La especificación es:

```math
\text{maximaDe}(R, k) =
\begin{cases}
\text{Some}\big(\text{maxima}(k, R)\big) & \text{si } R_k \neq [\,] \\
\text{None} & \text{si } R_k = [\,]
\end{cases}
```

El programa `magnitudMaxima(r).get(region)` se reduce así:

```math
\text{maximaDe}(R, k) \rightarrow \text{magnitudMaxima}(R).\text{get}(k)
```

Demostramos que cumple la especificación por casos sobre $k$.

Caso $k \in \text{Reg}(R)$: la llave está en $\text{magnitudMaxima}(R)$ por el punto 2, así que $\text{get}$ devuelve $\text{Some}$ con su valor, que por el punto 2 es $\text{maxima}(k, R)$:

```math
\text{magnitudMaxima}(R).\text{get}(k) \rightarrow \text{Some}\big(\text{magnitudMaxima}(R)(k)\big) = \text{Some}\big(\text{maxima}(k, R)\big)
```

Por el hecho 3.0, en este caso $R_k \neq [\,]$, de modo que el máximo está bien definido y esta es la primera línea de la especificación.

Caso $k \notin \text{Reg}(R)$: la llave no está en el mapa, así que $\text{get}$ devuelve $\text{None}$. Por el hecho 3.0, $R_k = [\,]$, y es la segunda línea de la especificación. En este caso el máximo de un conjunto vacío no existe, y por eso $\text{None}$ es la única respuesta honesta.

Concluimos que $\text{maximaDe}$ cumple la especificación para todo $R$ y todo $k$.  

Una consecuencia que conecta este punto con el punto 1: cuando la respuesta es $\text{Some}(v)$, ese $v$ es una magnitud que realmente está en la región, y ninguna otra la supera. Es decir, cumple las dos cosas que definen un máximo:

```math
\exists\, s \in R_k : \text{mag}(s) = v
\qquad \text{y} \qquad
\forall\, s \in R_k : \text{mag}(s) \leq v
```

Como por el punto 1 $\text{enRegion}(R, k)$ es la lista de las magnitudes de los sismos de $R_k$, esto significa que $\text{maximaDe}(R, k)$ es $\text{Some}$ del mayor elemento de $\text{enRegion}(R, k)$ cuando esa lista no es vacía, y $\text{None}$ cuando lo es. Las dos funciones se corresponden en todos los casos.

### 3.3 regionMasActiva

Sea $f(k) = \text{cuenta}(k, R)$ para las regiones $k \in \text{Reg}(R)$. Si $R \neq [\,]$, sean $M = \max_{k \in \text{Reg}(R)} f(k)$ el mayor conteo y $A$ el conjunto de regiones que lo alcanzan:

```math
A = \{\, k \in \text{Reg}(R) : f(k) = M \,\}
```

La especificación dice que, si $R = [\,]$, la respuesta es $\text{None}$, y si no, es $\text{Some}(a)$, con $a$ la región de $A$ que va primera en el orden alfabético.

El programa se reduce a:

```math
\text{regionMasActiva}(R) \rightarrow \text{if } (\text{conteo.isEmpty})\ \text{None else Some}(\text{ordenadas.maxBy}(f))
```

Demostramos que cumple la especificación por casos sobre $R$.

Caso $R = [\,]$: por el punto 2, $\text{conteoPorRegion}([\,])$ es el mapa vacío, así que se cumple `conteo.isEmpty` y la respuesta es $\text{None}$, que es la especificación.

Caso $R \neq [\,]$: $R$ tiene al menos un sismo $s$, y $\text{reg}(s)$ es una llave del conteo, así que el mapa no está vacío. Sea $L = [k_1, k_2, \ldots, k_m]$ la lista `ordenadas`. Contiene las llaves del mapa, que son $\text{Reg}(R)$, sin repetir, porque las llaves de un mapa son distintas, y ordenadas de menor a mayor en el orden alfabético, es decir $k_1 < k_2 < \cdots < k_m$.

Falta ver qué devuelve $L.\text{maxBy}(f)$. En lugar de darlo por sabido, lo demostramos, porque de ahí sale el desempate. `maxBy` es un programa iterativo: recorre la lista de izquierda a derecha, guardando el mejor elemento hasta el momento, y solo lo reemplaza si el siguiente es estrictamente mejor. Para cada $i$, sea $j^*(i)$ el primer índice $j \leq i$ en que $f$ alcanza su máximo sobre $k_1, \ldots, k_i$:

```math
j^*(i) = \min\Big\{\, j \leq i : f(k_j) = \max_{l \leq i} f(k_l) \,\Big\}
```

* Un estado es $(i, b)$, donde $i$ es cuántos elementos de $L$ se han leído y $b$ es el mejor elemento guardado.
* El estado inicial es $(1, k_1)$.
* Un estado es final si $i = m$.
* La invariante es $\text{Inv}(i, b) \equiv 1 \leq i \leq m \ \land\ b = k_{j^*(i)}$.
* La transformación lee $k_{i+1}$ y produce $(i + 1, b')$, con $b' = k_{i+1}$ si $f(k_{i+1}) > f(b)$, y $b' = b$ si no.

Demostramos los puntos:

1. $\text{Inv}(1, k_1)$: con un solo elemento, $j^*(1) = 1$ y $b = k_1$.
2. La invariante se mantiene. Por la invariante, $f(b) = \max_{l \leq i} f(k_l)$. Hay dos casos al leer $k_{i+1}$:
    * Si $f(k_{i+1}) > f(b)$: el nuevo máximo es $f(k_{i+1})$ y solo lo alcanza el índice $i + 1$, porque todos los anteriores valen menos. Entonces $j^*(i + 1) = i + 1$ y el nuevo $b' = k_{i+1}$ coincide con $k_{j^*(i+1)}$.
    * Si $f(k_{i+1}) \leq f(b)$: el máximo no cambia. El primer índice que lo alcanza sigue siendo $j^*(i)$, porque el elemento nuevo, si empata, viene después. Entonces $b' = b = k_{j^*(i)} = k_{j^*(i+1)}$.
3. Con $i = m$, la invariante dice $b = k_{j^*}$, donde $j^* = \min\{\, j : f(k_j) = M \,\}$ es el primer índice que alcanza el máximo global.
4. En cada paso $i$ aumenta en 1, y se detiene al llegar a $m$.

Entonces $L.\text{maxBy}(f) = k_{j^*}$. Veamos que ese elemento es $a$. Primero, $f(k_{j^*}) = M$, así que $k_{j^*} \in A$. Segundo, si $k_j \in A$, entonces $f(k_j) = M$ y, por la minimalidad de $j^*$, $j \geq j^*$; como la lista es creciente, $k_{j^*} \leq k_j$. Es decir, $k_{j^*}$ es el menor de $A$ en orden alfabético, y es $a$.

Concluimos que $\text{regionMasActiva}(R) = \text{Some}(a)$ cuando $R \neq [\,]$ y $\text{None}$ cuando $R = [\,]$.  

Con esto quedan claras varias cosas:

* La comparación estricta del recorrido es lo que da el desempate. Si `maxBy` reemplazara al mejor también en un empate, ganaría la última región de la lista y no la primera.
* La respuesta siempre es una región del registro, porque $a \in A \subseteq \text{Reg}(R)$.
* No depende del orden del registro. Si $R'$ es una permutación de $R$, el punto 2 dice que sus conteos son los mismos, y entonces las llaves, la función $f$ y la lista ordenada $L$ también lo son. El recorrido de $L$ es entonces idéntico, y da la misma respuesta. En particular $\text{regionMasActiva}(R) = \text{regionMasActiva}(R^{\text{inv}})$ con $R^{\text{inv}}$ el registro al revés, que es lo que revisa la prueba `regionMasActiva: con los últimos cuatro sismos gana el Choco`.

Dos ejemplos con el registro del enunciado. Con $R$ completo, $f(\text{Choco}) = 2$, $f(\text{Santander}) = 2$ y $f(\text{Valle}) = 3$, de modo que $M = 3$, $A = \{\text{Valle}\}$ y la respuesta es $\text{Some}(\text{Valle})$. Con los dos primeros sismos, $f(\text{Valle}) = f(\text{Santander}) = 1$, $M = 1$, $A = \{\text{Santander}, \text{Valle}\}$ y $L = [\text{Santander}, \text{Valle}]$, de modo que $j^* = 1$ y la respuesta es $\text{Some}(\text{Santander})$.

### 3.4 Las tres funciones son totales

Una función es total si da una respuesta para toda entrada válida, sin lanzar excepciones. Las tres lo son:

* $\text{conteoSeguro}$: consultar un mapa con valor por defecto nunca falla, porque si la llave no está se responde el valor por defecto. Esto es justo lo que la distingue de consultar $\text{conteoPorRegion}$ directamente, que sí lanza `NoSuchElementException` con una llave ausente.
* $\text{maximaDe}$: $\text{get}$ nunca falla, porque devuelve $\text{None}$ en lugar de lanzar una excepción.
* $\text{regionMasActiva}$: `maxBy` solo falla con una lista vacía, y el programa lo llama únicamente en el caso $R \neq [\,]$, donde $L$ tiene al menos un elemento, como vimos arriba.

---

## Punto 4: réplicas perezosas

Este punto no recorre un registro: construye una secuencia infinita. El
argumento de inducción es distinto al de los demás puntos: cada elemento se
calcula cuando se pide, así que las propiedades se enuncian sobre **la
posición $k$** y sobre **los primeros $n$ elementos**, no sobre toda la
secuencia a la vez.

### 4.1 Especificación

Sea $m$ la magnitud inicial y $\mu_{min}$ el mínimo. Se supone $m > 0$ y
$\mu_{min} \geq 0$ (las magnitudes del registro son positivas; ver 4.6 para el
caso en que $\mu_{min}$ es negativo).

El redondeo a dos decimales que usa el programa, `Math.round(x * 100.0) / 100.0`,
es:

```math
\text{red}(x) = \frac{\lfloor 100\,x + \tfrac{1}{2} \rfloor}{100}
```

La secuencia ideal de magnitudes sin redondear es $a_0 = m$ y
$a_{k+1} = 0.8 \cdot a_k$, es decir $a_k = m \cdot 0.8^k$. La secuencia de
réplicas es:

```math
r_k = \text{red}(a_k) \qquad k \in \mathbb{N}
```

Lo que el enunciado pide es:

- $\text{replicas}(m) = (r_0, r_1, r_2, \ldots)$.
- $\text{replicasSensibles}(m, \mu_{min}) = \text{List}(r_0, \ldots, r_{n-1})$,
  donde $n = \min\{\,k \in \mathbb{N} \mid r_k \leq \mu_{min}\,\}$. Es decir, las
  réplicas mientras **superen** el mínimo.

### 4.2 Elemento $k$ de `replicas`

**Lema 1 (`iterate`).** Sea $\text{xs} = \text{LazyList.iterate}(s)(f)$. Para
todo $k \in \mathbb{N}$, $\text{xs}(k) = f^k(s)$, donde $f^0(s) = s$ y
$f^{k+1}(s) = f(f^k(s))$.

_Demostración._ Por la definición de `iterate`,
$\text{iterate}(s)(f) = s \;\#::\; \text{iterate}(f(s))(f)$. Inducción sobre
$k$ (para todo valor de $s$):

**Caso base:** $k = 0$. $\text{xs}(0) = \text{head}(\text{xs}) = s = f^0(s)$.

**Caso de inducción:** $k + 1$. Hipótesis de inducción (HI): para todo $s'$,
$\text{iterate}(s')(f)(k) = f^k(s')$.

```math
\text{xs}(k+1) = \text{tail}(\text{xs})(k) = \text{iterate}(f(s))(f)(k) \overset{\text{HI}}{=} f^k(f(s)) = f^{k+1}(s)
```

Por lo tanto $\text{xs}(k) = f^k(s)$ para todo $k$.  

**Lema 2 (`map`).** Para toda secuencia $\text{xs}$ y función $g$,
$\text{xs.map}(g)(k) = g(\text{xs}(k))$. Es la definición de `map`: aplica $g$
a cada elemento conservando la posición.

**Teorema A.** Para todo $k \in \mathbb{N}$, $\text{replicas}(m)(k) = r_k$.

_Demostración._ Con $f(x) = 0.8\,x$, el Lema 1 da
$\text{iterate}(m)(f)(k) = f^k(m) = m \cdot 0.8^k = a_k$. Por el Lema 2,
$\text{replicas}(m)(k) = \text{red}(a_k) = r_k$.  

Comprobación con $m = 5.0$: $r_0 = 5.0$, $r_1 = 4.0$, $r_2 = 3.2$ y
$r_3 = \text{red}(2.56) = 2.56$, que es lo que da `replicas(5.0).take(4)`.

### 4.3 La secuencia es decreciente

**Lema 3.** Si $m > 0$, entonces $r_{k+1} \leq r_k$ para todo $k$.

_Demostración._ Como $a_k = m \cdot 0.8^k > 0$, se tiene
$a_{k+1} = 0.8\,a_k < a_k$. La función $\text{red}$ es monótona no decreciente:
si $x \leq y$ entonces $100x + \tfrac12 \leq 100y + \tfrac12$, y como
$\lfloor \cdot \rfloor$ es monótona, $\text{red}(x) \leq \text{red}(y)$. Entonces:

```math
r_{k+1} = \text{red}(a_{k+1}) \leq \text{red}(a_k) = r_k \qquad \blacksquare
```

(Es no estricta: dos réplicas seguidas pueden redondear al mismo valor cuando
son muy pequeñas.)

### 4.4 `replicasSensibles` devuelve exactamente las réplicas sensibles

**Lema 4 (terminación).** Si $m > 0$, existe $k$ con $r_k \leq \mu_{min}$.

_Demostración._ $a_k < 0.005 \iff 0.8^k < \dfrac{0.005}{m} \iff k > \dfrac{\ln(200\,m)}{\ln(5/4)}$.
Para todo $k$ mayor que esa cota, $a_k < 0.005$, luego
$100\,a_k + \tfrac12 < 1$ y $r_k = \text{red}(a_k) = 0 \leq \mu_{min}$, porque
$\mu_{min} \geq 0$.  

Por el Lema 4 el conjunto $\{k \mid r_k \leq \mu_{min}\}$ no es vacío, así que
$n = \min\{k \mid r_k \leq \mu_{min}\}$ existe.

**Teorema B.** $\text{replicasSensibles}(m, \mu_{min}) = \text{List}(r_0, \ldots, r_{n-1})$,
y además $\{\,k \mid r_k > \mu_{min}\,\} = \{0, \ldots, n-1\}$: no hay ninguna réplica
sensible fuera de ese prefijo.

_Demostración._

1. Por el Teorema A, el elemento $k$ de `replicas(m)` es $r_k$.
2. `takeWhile(p)` entrega el prefijo más largo de elementos que cumplen $p$:
   avanza mientras $p$ sea verdadero y se detiene en el primer elemento que
   falla. Aquí $p(x) \equiv x > \mu_{min}$.
3. Por la definición de $n$ como mínimo, para todo $k < n$ se cumple
   $r_k > \mu_{min}$, y $r_n \leq \mu_{min}$. Entonces `takeWhile` deja pasar
   exactamente $r_0, \ldots, r_{n-1}$ y se detiene en $r_n$.
4. `toList` convierte ese prefijo finito en `List`, así que el resultado es
   $\text{List}(r_0, \ldots, r_{n-1})$.
5. Para la segunda afirmación: si $k \geq n$, por el Lema 3
   $r_k \leq r_n \leq \mu_{min}$, así que ninguna réplica posterior supera el
   mínimo.  

Comprobación:

```math
m = 5.0,\ \mu_{min} = 3.0:\quad r = (5.0,\ 4.0,\ 3.2,\ 2.56,\ \ldots) \Rightarrow n = 3 \Rightarrow \text{List}(5.0, 4.0, 3.2)
```

```math
m = 2.0,\ \mu_{min} = 3.0:\quad r_0 = 2.0 \leq 3.0 \Rightarrow n = 0 \Rightarrow \text{List}()
```

El mínimo es estricto: si $r_k = \mu_{min}$ la réplica no entra, porque
$r_k > \mu_{min}$ es falso.

### 4.5 No se calculan más elementos de los pedidos

**Proposición.** Con memoización de cada celda del `LazyList`:

- `replicas(m).take(j).toList` calcula exactamente $j$ elementos.
- $\text{replicasSensibles}(m, \mu_{min})$ calcula exactamente $n + 1$ elementos:
  $r_0, \ldots, r_n$.

_Argumento._ Un elemento solo se calcula cuando un consumidor lo pide.
`toList` pide elementos de uno en uno, de la posición $0$ en adelante, y solo
pide el siguiente si el anterior fue aceptado. Con `take(j)` se aceptan las
posiciones $0, \ldots, j-1$ y el consumo termina. Con `takeWhile`, por el
Teorema B se aceptan $0, \ldots, n-1$ y la posición $n$ se evalúa solo para
comprobar que falla. Ninguna posición mayor se pide. La memoria usada es
proporcional a $n$, no al largo infinito de la secuencia.

### 4.6 Condiciones sobre los datos y precisión

- **Si $\mu_{min} < 0$ la función no termina.** Entonces $r_k \geq 0 > \mu_{min}$
  para todo $k$, nunca se cumple $r_k \leq \mu_{min}$ (el Lema 4 necesita
  $\mu_{min} \geq 0$), `takeWhile` no se detiene y `toList` se cuelga. Por eso se
  exige $\mu_{min} \geq 0$. Con $m \leq 0$ tampoco vale el Lema 3.
- **Punto flotante.** El programa calcula $a_{k+1}$ multiplicando en `Double`,
  no evalúa $m \cdot 0.8^k$ en los reales, así que los valores crudos tienen
  errores del orden de $10^{-16}$ (por ejemplo, $3.2 \cdot 0.8$ da
  `2.5600000000000005`). El argumento es exacto sobre los reales; en `Double`
  solo podría cambiar un resultado si $100\,a_k + \tfrac12$ cae a $10^{-14}$
  de un entero, y el redondeo a dos decimales absorbe ese error en los casos
  del taller.

### 4.7 Independencia del orden de recorrido

El punto 4 no recorre un registro. La secuencia $(r_k)$ está determinada solo
por $(m, \mu_{min})$ y su orden es parte de la definición (la posición $k$ es
el número de réplica), así que la función es pura: las mismas entradas dan
siempre el mismo resultado. Cuando el punto 5 la usa sobre la magnitud máxima
de una región, el resultado depende solo de ese valor máximo, y por tanto no
depende del orden de los sismos del registro (ver Teorema D).

---

## Punto 5: informe combinado
### 5.1 Especificación

Para un registro $r$ y un mínimo $\mu_{min}$, para cada región
$g \in R(r)$ se definen:

```math
c_g = \text{len}(S_g(r)), \qquad M_g = \max\{\mu(s) \mid s \in S_g(r)\}, \qquad n_g = \text{len}\bigl(\text{replicasSensibles}(M_g, \mu_{min})\bigr)
```

El enunciado pide:

```math
\text{informe}(r, \mu_{min}) = \{\, g \mapsto \text{Resumen}(c_g,\ M_g,\ n_g) \mid g \in R(r) \,\}
```

Es decir, un mapa cuyas llaves son exactamente las regiones del registro y sin
entradas para regiones ausentes. Para $r = \text{List}()$ debe ser $\text{Map}()$.

### 5.2 Corrección

**Lema 5 (`porRegion`, del punto 2).** `porRegion(r)` tiene como llaves
exactamente $R(r)$ y `porRegion(r)(g)` $= S_g(r)$ para toda $g \in R(r)$. Es la
especificación de `groupBy`, que conserva el orden relativo dentro de cada grupo.

**Teorema C.** Para todo registro $r$ y $\mu_{min} \geq 0$, el programa
`informe` cumple la especificación 5.1.

_Demostración._ Por inducción estructural sobre $r$.

**Caso base:** $r = \text{List}()$. $R(r) = \emptyset$, y por el Lema 5
`porRegion(List())` es `Map()`. Un `map` sobre un mapa vacío no visita ninguna
entrada y devuelve `Map()`. La especificación también da el mapa vacío.

**Caso de inducción:** $r = s :: r'$, con $g_0 = \rho(s)$. La hipótesis de
inducción (HI) dice que `informe(r')` cumple la especificación. Comparando
$r$ con $r'$ (el sismo $s$ queda al frente de $S_{g_0}$):

- Si $g \neq g_0$: $S_g(r) = S_g(r')$, así que $c_g$, $M_g$ y $n_g$ no cambian
  y la entrada coincide con la de $r'$, correcta por HI.
- Si $g = g_0$: $S_{g_0}(r) = s :: S_{g_0}(r')$, luego $c_{g_0}(r) = c_{g_0}(r') + 1$
  (o $c_{g_0} = 1$ si $g_0$ era una región nueva), y
  $M_{g_0}(r) = \max(\mu(s),\ M_{g_0}(r'))$ (o $\mu(s)$ si era nueva).
  $n_{g_0}$ se recalcula sobre ese nuevo máximo.

Falta ver que el programa calcula bien cada entrada, para $g \in R(r)$:

1. Por el Lema 5, $\text{porRegion}(r)$ contiene el par $(g, S_g(r))$ y las
   llaves son distintas.
2. Como $g \in R(r)$, $S_g(r)$ es no vacía, así que
   `sismos.map(_.magnitud).max` está definido (no lanza excepción) y es $M_g$.
3. `sismos.length` es $c_g$.
4. Por el Teorema B, `replicasSensibles(maxima, minimo)` es la lista
   $\text{List}(r_0, \ldots, r_{n-1})$ de la magnitud $M_g$, y su `length` es
   $n_g$.
5. La función del `map` devuelve el par $(g, \text{Resumen}(c_g, M_g, n_g))$,
   que conserva la llave. Como las llaves de entrada eran distintas, las de
   salida también, y ninguna entrada pisa a otra.  

Comprobación con `informe(registro, 3.0)`:

| Región | $S_g$ (magnitudes) | $c_g$ | $M_g$ | $r_0, r_1, \ldots$ | $n_g$ |
|---|---|:---:|:---:|---|:---:|
| Valle | 4.2, 3.8, 5.3 | 3 | 5.3 | 5.3, 4.24, 3.39, 2.71 | 3 |
| Santander | 5.1, 4.7 | 2 | 5.1 | 5.1, 4.08, 3.26, 2.61 | 3 |
| Choco | 6.0, 3.1 | 2 | 6.0 | 6.0, 4.8, 3.84, 3.07, 2.46 | 4 |

que coincide con el resultado del enunciado.

### 5.3 El resultado no depende del orden de recorrido

**Teorema D.** Si $r'$ es una permutación de $r$, entonces
$\text{informe}(r', \mu_{min}) = \text{informe}(r, \mu_{min})$.

_Demostración._ Sea $r'$ una permutación de $r$ (los mismos sismos, en otro
orden).

1. $R(r') = R(r)$: ambos conjuntos contienen las regiones de los mismos
   sismos. Las llaves del informe coinciden.
2. Para cada $g$, $S_g(r')$ es una permutación de $S_g(r)$, porque filtrar por
   región selecciona los mismos sismos. Una permutación no cambia la longitud,
   así que $c_g(r') = c_g(r)$.
3. El máximo no depende del orden. La operación $\max$ es conmutativa
   ($\max(a,b) = \max(b,a)$) y asociativa
   ($\max(\max(a,b),c) = \max(a,\max(b,c))$), por lo que cualquier forma de
   combinar los elementos da el mismo valor: el único elemento del conjunto
   de magnitudes que es mayor o igual que todos los demás. Entonces
   $M_g(r') = M_g(r)$.
4. $n_g$ es una función de $(M_g, \mu_{min})$ únicamente (Teorema B), luego
   $n_g(r') = n_g(r)$.
5. Dos mapas son iguales cuando tienen las mismas llaves y el mismo valor para
   cada llave; el orden interno de las entradas no cuenta. Por 1 a 4, todas
   las entradas coinciden.  

Esto es lo que garantiza que `groupBy` y los pliegues internos (`length`,
`max`) puedan recorrer el registro en cualquier orden sin cambiar el informe.

## Notación común

Un sismo es $s = (f, \rho, \mu, p)$ con región $\rho(s)$ y magnitud $\mu(s)$.
Un registro es una lista $r = \text{List}(s_1, \ldots, s_n)$, y se define
recursivamente: el registro vacío $\text{List}()$ es el caso base, y
$s :: r'$ es el registro que añade el sismo $s$ al frente de $r'$.

Para una región $g$, se define el sub-registro de esa región:

```math
S_g(r) = \text{List}(s \in r \mid \rho(s) = g)
```

y el conjunto de regiones presentes $R(r) = \{\rho(s) \mid s \in r\}$.

Las operaciones de lista que usa el compilador al traducir un `for` cumplen
estas ecuaciones (son su definición):

```math
\text{filtrar}_p(\text{List}()) = \text{List}(), \qquad
\text{filtrar}_p(x :: L) =
\begin{cases}
x :: \text{filtrar}_p(L) & \text{si } p(x) \\
\text{filtrar}_p(L) & \text{si no}
\end{cases}
```

```math
\text{map}_g(\text{List}()) = \text{List}(), \qquad \text{map}_g(x :: L) = g(x) :: \text{map}_g(L)
```

```math
\text{flatMap}_g(\text{List}()) = \text{List}(), \qquad \text{flatMap}_g(x :: L) = g(x) \mathbin{+\!\!+} \text{flatMap}_g(L)
```

Con esta notación, $S_g(r) = \text{filtrar}_{\rho = g}(r)$, y la traducción de cada
`for` del taller es una composición de estas tres operaciones.
