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
- **Idea central:** Es una estructura de datos de tipo arbol pero que busca optimizar el problema de hacer operaciones sobre los elementos de un array, para asi bajar la complejidad de estas operaciones. Basicamente es un arbol con los resultados precargados de realizar operaciones sobre un array (tipicamente sumas por Ejemplo). 
- **Problema que resuelve:** Una situación común en la cual nos podemos encontrar es en querer operar sobre los elementos de un array, por ejemplo hacer una sumatoria sobre todos los elementos de un array de números. Lo cual es costoso en términos de rendimiento porque para cada vez que queramos calcularlo habría una complejidad algorítmica de O(n). Esta estructura de datos lo que propone es precargar los resultados de esa sumatoria en un árbol para poder acceder a los resultados de las sumatorias del array o de rangos en particular del mismo de manera optima, la complejidad con esta estructura crece en torno a O(log n).

### Definición / propiedades
- Definición formal: Para cargar el árbol se trabaja con la representación en binario del numero de los índices de los elementos del array y consta de los siguientes pasos:

> [!NOTE] Buscar los padres de cada nodo utilizando la funcion "Parent"
> **Lowbit(i) = i & (−i)**
> **Parent(i) = i - Lowbit(i)**

Donde la operación "&" es un "and" lógico entre los números en binario, la letra "i" es el índice del elemento del vector que se esta analizando y "-i" es hacer el complemento a la base mas uno.


**Ejemplo para índice 6**
Considerando: 6<sub>10</sub> = 110<sub>2</sub>     

Parent(110<sub>2</sub>) =   110<sub>2</sub> - Lowbit(110<sub>2</sub>)        
Parent(110<sub>2</sub>) =   110<sub>2</sub> - 110<sub>2</sub> & (−110<sub>2</sub>)
Parent(110<sub>2</sub>) =   110<sub>2</sub> - 110<sub>2</sub> & 010<sub>2</sub>
Parent(110<sub>2</sub>) =   110<sub>2</sub> - 010<sub>2</sub>
Parent(110<sub>2</sub>) =   100<sub>2</sub>

Y si lo vemos en forma decimal quedaría: Parent(6<sub>10</sub>) =   4<sub>10</sub> lo que indica que el nodo de índice 4 es el padre del nodo de índice 6 en el arbol. Si se replica esto con todos los índices del array se construye la estructura y te indica cuales nodos son hijos de cuales otros dando como resultado la estructura final del arbol.

- Propiedades clave: orden, acotamiento, restricciones sobre elementos, estabilidad, etc.

### Representación
- Descripción de la organización interna (arrays, nodos enlazados, árboles, tablas, etc.).
- Ilustración sugerida: incluye aquí un diagrama ASCII o referencia a una imagen en `attachments/`.

Debe responder a: "¿qué estoy mirando?"

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
