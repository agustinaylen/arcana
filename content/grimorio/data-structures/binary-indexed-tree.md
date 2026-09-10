---
title: Arbol Indexado
tags:
  - data-structures
  - template
alias:
  - nombre
  - otro
---
## 1. Qué es y cómo funciona

### Intuición
- **Idea central:** Un Árbol Indexado Binario (Fenwick Tree o Binary Indexed Tree) es una estructura de datos basada en un arreglo que permite realizar de manera eficiente operaciones sobre rangos de elementos, especialmente sumas prefijas, mientras se mantienen actualizaciones puntuales.

La idea consiste en no almacenar solamente los valores originales del arreglo, sino también sumas parciales correspondientes a determinados intervalos. Estos intervalos se organizan de acuerdo con la representación binaria de los índices.

De esta manera, una consulta que normalmente requeriría recorrer muchos elementos puede resolverse combinando unas pocas sumas parciales.

- **Problema que resuelve:** Supongamos que tenemos un arreglo de números y queremos consultar repetidamente la suma de sus elementos hasta una determinada posición.

Con un arreglo tradicional, obtener una suma prefija como:

```text
A[1] + A[2] + ... + A[i]
```

requiere recorrer los elementos desde `1` hasta `i`, por lo que una consulta puede tener un costo de `O(n)` en el peor caso.

Si además se realizan modificaciones sobre los elementos, utilizar un arreglo auxiliar de sumas prefijas tampoco resulta conveniente: modificar un elemento puede obligar a actualizar todas las sumas posteriores.

El Fenwick Tree permite realizar tanto **actualizaciones puntuales** como **consultas de suma prefija** en `O(log n)`.

### Definición / propiedades
- **Definición formal:** Un Fenwick Tree es una estructura de datos que utiliza un arreglo de tamaño `n + 1`, donde cada posición `i` almacena la suma de un intervalo específico del arreglo original. La estructura utiliza índices comenzando en `1` y determina el tamaño de cada intervalo mediante el bit menos significativo encendido del índice.

Por lo tanto, no se trata de un árbol compuesto por nodos enlazados. La estructura de árbol es **implícita**: las relaciones entre sus posiciones se determinan mediante operaciones sobre los índices.

- **Propiedades clave:** Para determinar qué intervalo representa una posición se utiliza la operación:

```text
Lowbit(i) = i & (-i)
```

`Lowbit(i)` obtiene el valor correspondiente al bit menos significativo que está encendido en la representación binaria de `i`.

Por ejemplo:

```text
6₁₀ = 110₂

Lowbit(6) = 6 & (-6) = 2
```

Por lo tanto, la posición `6` representa un intervalo de **2 elementos** que termina en dicha posición:

```text
BIT[6] → A[5] + A[6]
```

En general, `BIT[i]` almacena la suma de los `Lowbit(i)` elementos que terminan en la posición `i`.

Algunos ejemplos:

```text
BIT[1] → A[1]
BIT[2] → A[1] + A[2]
BIT[3] → A[3]
BIT[4] → A[1] + A[2] + A[3] + A[4]
BIT[5] → A[5]
BIT[6] → A[5] + A[6]
BIT[8] → A[1] + ... + A[8]
```

Esto permite representar diferentes tamaños de intervalos utilizando únicamente el índice.

#### Relación entre las posiciones

Las posiciones del Fenwick Tree presentan una relación jerárquica implícita. Para obtener una posición anterior durante el cálculo de una suma prefija se utiliza:

```text
Parent(i) = i - Lowbit(i)
```

Por ejemplo, para `i = 6`:

```text
6₁₀ = 110₂

Lowbit(6) = 2

Parent(6) = 6 - 2
          = 4
```

Por lo tanto, durante el recorrido de una consulta se pasa de la posición `6` a la posición `4`, y luego a la posición `0`, obteniendo los bloques necesarios para construir la suma prefija.

Es importante destacar que `Parent(i)` no representa un puntero almacenado en memoria. La relación se calcula dinámicamente a partir del índice.

---

### Representación

Consideremos el siguiente arreglo:

```text
A = [5, 4, 1, -1, 0, 8]
```

Utilizando posiciones desde `1`:

```text
Índice:   1   2   3   4   5   6
A:        5   4   1  -1   0   8
```

El Fenwick Tree almacena sumas parciales:

```text
Índice:   1   2   3   4   5   6
BIT:      5   9   1   9   0   8
```

Cada valor representa un intervalo diferente:

```text
BIT[1] = A[1]
BIT[2] = A[1] + A[2]
BIT[3] = A[3]
BIT[4] = A[1] + A[2] + A[3] + A[4]
BIT[5] = A[5]
BIT[6] = A[5] + A[6]
```

Visualmente:

```text
A:

Índice    1    2    3    4    5    6
          │    │    │    │    │    │
          5    4    1   -1    0    8


BIT:

Índice    1    2    3    4    5    6
          │    │    │    │    │    │
          5    9    1    9    0    8
          │    │    │    │    │    │
         [1] [1-2] [3] [1---4] [5] [5-6]
```

La cantidad de elementos representados por cada posición está determinada por `Lowbit(i)`.

Por ejemplo:

```text
i = 4

4 = 100₂
Lowbit(4) = 4
```

Por eso `BIT[4]` representa cuatro elementos:

```text
A[1] + A[2] + A[3] + A[4]
```

Mientras que:

```text
i = 6

6 = 110₂
Lowbit(6) = 2
```

por lo que `BIT[6]` representa:

```text
A[5] + A[6]
```

Esta organización permite combinar diferentes bloques para obtener una suma prefija sin recorrer todos los elementos individualmente.

## 2. Operaciones y complejidad

### Operaciones principales

Las operaciones principales de un Fenwick Tree están relacionadas con la consulta y modificación de sumas.

- **`add(i, valor)` / actualización puntual:** incrementa el valor almacenado en la posición `i` en una determinada cantidad. Para mantener actualizadas las sumas parciales, se modifican las posiciones del Fenwick Tree que contienen a `i` dentro de su intervalo.

- **`prefixSum(i)` / suma prefija:** calcula la suma de todos los elementos desde la posición `1` hasta la posición `i`. Para hacerlo, combina los intervalos almacenados en diferentes posiciones del Fenwick Tree.

- **`rangeSum(l, r)` / suma de rango:** obtiene la suma de los elementos comprendidos entre las posiciones `l` y `r`. Se calcula utilizando dos sumas prefijas:

```text id="k3v1s8"
rangeSum(l, r) = prefixSum(r) - prefixSum(l - 1)
```

Por ejemplo, si se quiere obtener la suma entre las posiciones `3` y `6`:

```text id="jq8nqz"
A[3] + A[4] + A[5] + A[6]

= prefixSum(6) - prefixSum(2)
```

- **`get(i)` / obtener valor individual:** si se utiliza la estructura únicamente con actualizaciones incrementales, el valor de una posición puede obtenerse mediante una diferencia de sumas prefijas:

```text id="2j84y5"
get(i) = prefixSum(i) - prefixSum(i - 1)
```

### Complejidad

| Operación | Tiempo | Espacio adicional |
| :--- | :---: | :---: |
| `add(i, valor)` | `O(log n)` | `O(1)` |
| `prefixSum(i)` | `O(log n)` | `O(1)` |
| `rangeSum(l, r)` | `O(log n)` | `O(1)` |
| `get(i)` | `O(log n)` | `O(1)` |
| Almacenamiento de la estructura | — | `O(n)` |

Las operaciones `add` y `prefixSum` tienen una complejidad de `O(log n)` porque en cada paso se modifica el índice utilizando el valor obtenido mediante `Lowbit(i)`.

Para calcular una suma prefija, el índice disminuye:

```text id="h0w7i2"
i = i - Lowbit(i)
```

mientras que para realizar una actualización puntual aumenta:

```text id="s7p1a4"
i = i + Lowbit(i)
```

En ambos casos, la cantidad de posiciones recorridas está acotada por `O(log n)`.

La operación `rangeSum` realiza dos consultas `prefixSum`, por lo que:

```text id="w0g8ra"
O(log n) + O(log n) = O(log n)
```

manteniendo una complejidad total de `O(log n)`.

### Detalles operativos

El Fenwick Tree utiliza normalmente **índices desde 1**. La posición `0` se reserva como condición de finalización de los recorridos y no representa un elemento del arreglo.

Por ejemplo, al calcular una suma prefija:

```text id="g0q8qj"
mientras i > 0:
    utilizar BIT[i]
    i = i - Lowbit(i)
```

el recorrido finaliza cuando `i` llega a `0`.

Una actualización funciona de manera inversa:

```text id="3qj4q8"
mientras i <= n:
    actualizar BIT[i]
    i = i + Lowbit(i)
```

y finaliza cuando el índice supera el tamaño `n`.

El Fenwick Tree almacena **O(n)** valores adicionales, uno por cada posición utilizada por la estructura. No necesita crear nodos ni utilizar referencias o punteros, lo que permite una representación compacta en memoria.

Estas complejidades suponen que la operación utilizada es una suma y que las actualizaciones son **puntuales**. Si el problema requiere otro tipo de operación o actualizaciones sobre rangos, pueden ser necesarias variantes del Fenwick Tree o una estructura diferente.

# 3. Implementación

### Idea de implementación

El Fenwick Tree se implementa utilizando un arreglo auxiliar, generalmente llamado `BIT` o `tree`, de tamaño `n + 1`. Se utiliza una posición adicional porque la estructura trabaja con índices desde `1`.

La operación fundamental para determinar qué posiciones deben recorrerse es:

```java
i & -i
```

que obtiene el `Lowbit(i)`.

A partir de esta operación se implementan los dos algoritmos principales:

**Actualización puntual (`add`)**

Cuando se modifica un elemento de la posición `i`, no es necesario actualizar todas las posiciones posteriores. Solo se actualizan aquellas posiciones del Fenwick Tree cuyos intervalos contienen a `i`.

Para avanzar entre esas posiciones se utiliza:

```text
i = i + Lowbit(i)
```

El proceso continúa hasta superar el tamaño del arreglo.

**Suma prefija (`prefixSum`)**

Para calcular la suma desde la posición `1` hasta `i`, se toman los bloques almacenados en diferentes posiciones del Fenwick Tree.

Después de utilizar el bloque correspondiente a `BIT[i]`, se continúa hacia el bloque anterior mediante:

```text
i = i - Lowbit(i)
```

El proceso termina cuando `i` llega a `0`.

De esta forma, ambas operaciones recorren una cantidad de posiciones proporcional a la cantidad de bits del índice, obteniendo una complejidad de `O(log n)`.

### Invariantes

Para que el Fenwick Tree funcione correctamente deben cumplirse las siguientes condiciones:

- La posición `0` se utiliza como condición de finalización y no representa un elemento del arreglo.
- Para cada posición `i`, `BIT[i]` debe contener la suma de los `Lowbit(i)` elementos que terminan en `i`.
- Las actualizaciones deben modificar todas las posiciones del BIT cuyo intervalo contiene al elemento actualizado.
- Las consultas deben combinar bloques que no se superpongan, de modo que cada elemento sea contabilizado exactamente una vez.
- El tamaño del arreglo utilizado por el Fenwick Tree debe ser suficiente para representar todas las posiciones válidas.

### Ejemplo de código

Una implementación mínima en Java puede ser:

```java
class FenwickTree {
    private final int[] tree;

    public FenwickTree(int n) {
        tree = new int[n + 1];
    }

    public void add(int i, int delta) {
        while (i < tree.length) {
            tree[i] += delta;
            i += i & -i;
        }
    }

    public int prefixSum(int i) {
        int result = 0;

        while (i > 0) {
            result += tree[i];
            i -= i & -i;
        }

        return result;
    }

    public int rangeSum(int left, int right) {
        return prefixSum(right) - prefixSum(left - 1);
    }
}
```

Por ejemplo, para construir la estructura a partir de los valores:

```java
FenwickTree bit = new FenwickTree(6);

int[] values = {5, 4, 1, -1, 0, 8};

for (int i = 0; i < values.length; i++) {
    bit.add(i + 1, values[i]);
}

System.out.println(bit.prefixSum(4));  // 9
System.out.println(bit.rangeSum(2, 5)); // 4
```

La primera consulta calcula:

```text
5 + 4 + 1 - 1 = 9
```

mientras que la segunda calcula:

```text
4 + 1 - 1 + 0 = 4
```

Si posteriormente se modifica la posición `3` agregando `2`:

```java
bit.add(3, 2);
```

la estructura actualiza únicamente las posiciones necesarias del BIT. A partir de ese momento, las consultas reflejarán automáticamente el nuevo valor.

## 4. Uso y criterio

### ¿Cuándo conviene usar un Árbol Indexado?

El Árbol Indexado es especialmente útil cuando se necesita trabajar con un arreglo que cambia frecuentemente y, al mismo tiempo, realizar consultas sobre sumas acumuladas o rangos.

Por ejemplo, puede utilizarse para:

- calcular rápidamente la suma de los elementos desde una posición inicial hasta una determinada posición;
- consultar la suma de un rango de elementos;
- modificar el valor de un elemento y mantener las consultas actualizadas;
- resolver problemas donde se realizan muchas operaciones de actualización y consulta sobre el mismo arreglo.

La principal ventaja aparece cuando estas operaciones se realizan muchas veces. Un arreglo común permite modificar un elemento en `O(1)`, pero calcular una suma de rango puede costar `O(n)`. En cambio, el Árbol Indexado permite realizar tanto la actualización puntual como las consultas de suma en `O(log n)`.

### ¿Cuándo no conviene usarlo?

No es la mejor opción cuando:

- el arreglo casi no cambia y solamente se realizan consultas de suma. En ese caso, un arreglo de sumas prefijas puede ser más simple y permitir consultas en `O(1)`;
- se necesitan operaciones más complejas que una suma, como consultar mínimos o máximos de rangos, dependiendo de la operación y de la variante utilizada;
- se necesitan actualizaciones sobre rangos completos y consultas más complejas. Para estos casos puede ser conveniente utilizar un Segment Tree u otra estructura especializada;
- se requiere acceder directamente al valor de cada posición sin realizar cálculos adicionales. En ese caso, el arreglo original es más sencillo.

### Comparación con otras estructuras

| Estructura | Consulta de rango | Actualización puntual | Memoria | Característica principal |
|---|---:|---:|---:|---|
| Arreglo común | `O(n)` | `O(1)` | `O(n)` | Simple y acceso directo |
| Sumas prefijas | `O(1)` | `O(n)` | `O(n)` | Excelente si los datos casi no cambian |
| Árbol Indexado | `O(log n)` | `O(log n)` | `O(n)` | Equilibrio entre consultas y actualizaciones |
| Segment Tree | `O(log n)` | `O(log n)` | `O(n)`* | Mayor flexibilidad para distintos tipos de consultas |

\* Un Segment Tree suele requerir aproximadamente `4n` posiciones en una implementación basada en arreglos, aunque existen otras representaciones.

La diferencia más importante con un **Segment Tree** es la flexibilidad. El Árbol Indexado tiene una implementación más simple, utiliza menos memoria y resulta muy eficiente para sumas y actualizaciones puntuales. El Segment Tree, en cambio, permite representar una mayor variedad de operaciones y variantes, aunque a costa de una implementación más compleja.

### ¿Cómo reconocer que puede ser una buena opción?

Una situación es candidata a utilizar un Árbol Indexado cuando aparecen simultáneamente estas características:

> **“Tengo un arreglo, los valores cambian y necesito consultar repetidamente sumas de posiciones o rangos.”**

En ese escenario, el Árbol Indexado ofrece un buen equilibrio entre eficiencia, memoria y simplicidad de implementación.

## 5. Relaciones y extensiones

### Variantes

Aunque su uso más común consiste en realizar sumas prefijas con actualizaciones puntuales, el Árbol Indexado puede extenderse para resolver otros tipos de problemas.

Una variante frecuente permite realizar **actualizaciones sobre rangos**. Mediante una técnica basada en diferencias, es posible modificar todos los elementos de un intervalo y mantener consultas eficientes utilizando un Fenwick Tree.

También es posible combinar **dos Fenwick Trees** para realizar determinadas actualizaciones y consultas sobre rangos en `O(log n)`.

Otra extensión es el **Fenwick Tree bidimensional**, que aplica la misma idea sobre una matriz. En este caso, la estructura permite realizar consultas y actualizaciones sobre regiones bidimensionales, utilizando índices para filas y columnas.

### Relación con otras estructuras

El Fenwick Tree se relaciona directamente con el **Array**, ya que su implementación se basa en un arreglo y aprovecha el acceso directo a sus posiciones. Sin embargo, a diferencia de un arreglo común, cada posición del BIT puede almacenar información sobre un intervalo de elementos.

También puede entenderse como una alternativa dinámica a las **sumas prefijas**. Un arreglo de sumas prefijas permite obtener consultas de rango en `O(1)`, pero una modificación puede requerir actualizar muchas posiciones. El Fenwick Tree busca un equilibrio entre ambas operaciones, permitiendo consultas y actualizaciones en `O(log n)`.

Otra estructura estrechamente relacionada es el **Segment Tree**. Ambas permiten realizar consultas y actualizaciones eficientes sobre un arreglo, pero el Segment Tree representa explícitamente intervalos mediante una estructura jerárquica más general. El Fenwick Tree, en cambio, utiliza una representación implícita basada en las propiedades binarias de los índices.

### Notas avanzadas

La idea fundamental del Fenwick Tree no está limitada exclusivamente a sumas. Puede utilizarse con otras operaciones siempre que sus propiedades permitan combinar correctamente los valores almacenados.

Sin embargo, la posibilidad de reconstruir una consulta de rango mediante:

```text id="p9se5u"
consulta(l, r) = prefijo(r) - prefijo(l - 1)
```

depende de que la operación utilizada permita realizar una operación inversa. Por este motivo, las sumas son uno de los casos más naturales para esta estructura.

En implementaciones más avanzadas, pueden utilizarse técnicas de **compresión de coordenadas** junto con un Fenwick Tree. Esto permite trabajar con valores o posiciones muy grandes utilizando un arreglo de tamaño reducido.

Estas variantes muestran que el Árbol Indexado forma parte de una familia de estructuras orientadas a optimizar consultas sobre colecciones dinámicas. Su principal característica es lograr esta eficiencia mediante una representación compacta y operaciones simples sobre los bits de los índices.

## 6. Referencias y recursos

- Peter M. Fenwick. **“A New Data Structure for Cumulative Frequency Tables”**. *Software: Practice and Experience*, 1994. Trabajo original en el que se presenta la estructura conocida actualmente como Fenwick Tree.

- [CP-Algorithms — Fenwick Tree](https://cp-algorithms.com/data_structures/fenwick.html). Explicación de la estructura, sus operaciones y diferentes variantes para consultas y actualizaciones.

- [VisuAlgo — Fenwick Tree](https://visualgo.net/en/fenwicktree). Visualización interactiva que permite observar cómo se organizan las posiciones y cómo se realizan las consultas y actualizaciones.

- Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest y Clifford Stein. **Introduction to Algorithms**. MIT Press. Libro de referencia general sobre algoritmos y estructuras de datos.

- Mark Allen Weiss. **Data Structures and Algorithm Analysis**. Pearson. Referencia general para el análisis de estructuras de datos y complejidad algorítmica.
