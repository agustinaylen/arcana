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
- Lista de operaciones con nombres estandarizados (por ejemplo: push/pop/peek, insert/delete/find, append/concat, union/intersect).
- Para cada operación: breve descripción de lo que hace.

### Complejidad
- Por operación: tiempo (peor/ promedio/ amortizado) y complejidad espacial adicional.
- Notas sobre costos ocultos (reallocs, rehash, recorridos, copias).

### Detalles operativos 
- Casos especiales: operaciones en estructura vacía/llena, duplicados, orden, límites de tamaño.
- Comportamiento en concurrencia o fallos (si aplica).

Debe responder a: "¿qué puedo hacer y cuánto cuesta?"

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
