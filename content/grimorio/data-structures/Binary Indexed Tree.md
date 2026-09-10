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

## 3. Implementación

### Idea de implementación
- Descripción de la(s) estrategia(s) típica(s) para implementar la estructura.
- Algoritmos clave y pasos principales.

### Invariantes
- Lista de comprobaciones e invariantes que el código debe garantizar siempre (por ejemplo: punteros no nulos, tamaño consistente, heap property, ordenamiento mantenido).

### Ejemplo de código
- Proporciona 1-2 snippets claros y mínimos (en Python).
- Ejemplo de uso típico con entrada y salida esperada.

Debe responder a: "¿cómo lo programo sin romperlo?"

## 4. Uso y criterio

### Casos de uso
- Situaciones y problemas donde la estructura encaja naturalmente.

### Cuándo NO usarlo
- Escenarios donde su uso es contraproducente o subóptimo.

### Comparaciones
- Alternativas comunes y cuándo elegir cada una (lista comparativa breve).

### Ventajas / desventajas
- Trade-offs prácticos en rendimiento, memoria, simplicidad, y facilidad de implementación.

### Señales de reconocimiento
- Pistas en el enunciado de un problema que indican que esta estructura es adecuada.

Debe responder a: "¿cuándo conviene usarlo?"

## 5. Relaciones y extensiones

### Variantes
- Variantes y mejoras (por ejemplo: versiones balanceadas, persistentes, acotadas, indexadas, con hashing, etc.).

### Relación con otras estructuras
- Dependencias conceptuales y cómo se combina con otras estructuras.

### Notas avanzadas
- Temas avanzados como persistencia, concurrencia, paralelismo, ordenamientos aleatorios, caching, tuning de parámetros.

Debe responder a: "¿cómo encaja en el mapa general de estructuras de datos?"

## 6. Referencias y recursos
- Enlaces y libros de referencia, artículos científicos.
- Visualizaciones y demostraciones.
