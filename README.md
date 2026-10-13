# RedSens — Trabajo Práctico Integral

**Taller de Algoritmos y Estructura de Datos II**  
**Universidad Siglo 21 · Modalidad PH**

> **Estado:** en desarrollo. Este README documenta los requerimientos del trabajo práctico; no implica que las funcionalidades ya estén implementadas.

## 1. Descripción del proyecto

**RedSens** es un sistema de consola para gestionar y mantener una red provincial de estaciones de medición de **DATA Investigaciones**, distribuidas en localidades de Córdoba.

Las estaciones registran **temperatura, humedad, precipitación y nivel de batería**, y transmiten sus mediciones a una central. El proyecto busca resolver tres problemas:

1. **Registro y consulta:** encontrar rápidamente estaciones y conocer su estado.
2. **Mantenimiento:** planificar recorridos eficientes para las cuadrillas que visitan estaciones críticas.
3. **Transmisión:** reducir, sin pérdida de información, los archivos de mediciones enviados por enlaces de baja velocidad.

Antes de su puesta en producción, el sistema también debe permitir **generar datos simulados reproducibles y evaluar el rendimiento** de las estructuras y algoritmos implementados.

## 2. Datos generales

| Aspecto | Requisito |
| --- | --- |
| Modalidad | Grupos de **4 o 5 integrantes**, sin excepciones |
| Lenguaje | **Java (JDK 17+)** o **Python (3.10+)** |
| Interfaz | Aplicación de consola con menú |
| Exposición | Una exposición oral grupal en alguna de las tres clases entre el primer y segundo parcial |
| Entrega | Antes del final de la materia, según la fecha establecida en Canvas |
| Contenidos | Árboles y recursión, ABB, AVL, hashing y colisiones, grafos (BFS, DFS, Dijkstra, Floyd), Huffman, simulación y Josephus |

**Criterio de evaluación:** no alcanza con que el programa funcione; todos los integrantes deben poder explicar las estructuras elegidas, sus fundamentos y las alternativas posibles.

## 3. Modelo de datos

Cada estación debe contener, como mínimo, los siguientes atributos:

| Campo | Tipo | Ejemplo | Regla |
| --- | --- | --- | --- |
| `codigo` | `String` | `CBA-0347` | Clave única, formato `CBA-NNNN` |
| `numero` | `int` | `347` | Parte numérica del código; clave del índice ordenado |
| `nombre` | `String` | `Estación Embalse Norte` | Texto libre |
| `localidad` | `String` | `Río Tercero` | Debe existir como vértice del grafo |
| `tipo` | `String` | `PLUVIOMETRICA` | `PLUVIOMETRICA`, `METEOROLOGICA` o `HIDROMETRICA` |
| `bateria` | `int` | `18` | Porcentaje de 0 a 100; **menor a 20** = crítica |
| `diasSinVisita` | `int` | `42` | Días desde la última visita técnica |

El equipo puede agregar otros atributos si los justifica.

**Ejemplo de línea del archivo de mediciones:**

```text
CBA-0347;2026-10-13;08:00;T=18.4;H=62;P=0.0;BAT=18
```

## 4. Módulos y requerimientos

### M1 — Registro de estaciones (tabla hash)

**Objetivo:** localizar, insertar y eliminar estaciones por `codigo` en tiempo constante promedio.

- [ ] Implementar una clase `TablaHash` propia mediante **direccionamiento abierto** (hash cerrado).
- [ ] Diseñar y justificar una función hash para claves `CBA-NNNN`.
- [ ] Implementar **exploración lineal** y **exploración cuadrática**, configurables al crear la tabla.
- [ ] Implementar `insertar`, `buscar` y `eliminar`.
- [ ] Utilizar **eliminación perezosa**: marcar posiciones como borradas.
- [ ] Controlar el factor de carga y hacer **rehashing** al superar el umbral: duplicar la capacidad y llevarla al siguiente número primo.
- [ ] Justificar el umbral elegido para cada estrategia.
- [ ] Registrar colisiones y sondeos por operación.

**Experimento obligatorio:** insertar **10.000 estaciones simuladas** con cada estrategia y comparar colisiones, sondeos promedio en búsquedas exitosas y fallidas, y cantidad de rehashings. Presentar una tabla de resultados y analizar el **agrupamiento primario**.

**Opcional (suma puntos):** implementar una tercera variante con **encadenamiento** (hashing abierto).

### M2 — Índice ordenado (ABB y AVL)

**Objetivo:** consultar estaciones ordenadas por `numero`, filtrar por rangos e identificar estaciones críticas.

- [ ] Implementar un **árbol binario de búsqueda (ABB)**: inserción, búsqueda, eliminación en sus tres casos y recorridos inorden, preorden y postorden.
- [ ] Implementar un **árbol AVL** con las mismas operaciones, factor de equilibrio y rotaciones simples y dobles.
- [ ] Implementar recursivamente altura, cantidad de nodos y cantidad de hojas, identificando los casos base.
- [ ] Listar las estaciones cuyo número esté entre `A` y `B`, en orden.
- [ ] Generar el reporte de estaciones **con batería < 20**, ordenadas por número; este resultado alimenta al M3.

**Experimento obligatorio:** insertar **1.000, 5.000 y 10.000** estaciones tanto en ABB como en AVL, primero en **orden ascendente** y luego en **orden aleatorio**. Comparar alturas y tiempos de búsqueda y explicar las diferencias.

> **Si se usa Python:** contemplar el límite de recursión (aproximadamente 1.000 llamadas) cuando un ABB degenera al insertar miles de claves ordenadas; resolverlo y explicarlo (por ejemplo, con inserción iterativa).

### M3 — Red vial y planificación de recorridos (grafos)

**Objetivo:** representar rutas entre localidades cordobesas y proponer recorridos de mantenimiento.

El grafo de referencia es **no dirigido y ponderado**, con **12 localidades y 18 tramos**. Los pesos representan kilómetros aproximados válidos para el trabajo.

| Origen | Destino | Distancia (km) |
| --- | --- | ---: |
| Córdoba | Villa Carlos Paz | 36 |
| Córdoba | Alta Gracia | 39 |
| Córdoba | Jesús María | 51 |
| Córdoba | Río Tercero | 102 |
| Córdoba | Villa María | 146 |
| Córdoba | San Francisco | 205 |
| Villa Carlos Paz | Cosquín | 24 |
| Cosquín | La Falda | 20 |
| Villa Carlos Paz | Alta Gracia | 35 |
| Alta Gracia | Río Tercero | 85 |
| Río Tercero | Villa María | 70 |
| Villa María | Río Cuarto | 130 |
| Río Tercero | Río Cuarto | 110 |
| Jesús María | Deán Funes | 70 |
| La Falda | Deán Funes | 95 |
| Villa Carlos Paz | Villa Dolores | 140 |
| Villa Dolores | Río Cuarto | 210 |
| Villa María | San Francisco | 150 |

**El grafo debe cargarse desde un archivo de texto; no debe quedar codificado directamente en el programa.**

- [ ] Representar el grafo como **matriz** y como **lista de adyacencia**, justificando cuál conviene más.
- [ ] Ejecutar **BFS** desde la base asignada, mostrando localidades alcanzables en 1, 2 y 3 tramos.
- [ ] Ejecutar **DFS** para determinar si la red es conexa; simular la eliminación de una arista y detectar posibles localidades aisladas.
- [ ] Ejecutar **Dijkstra** desde la base, mostrando distancias mínimas **y caminos completos**.
- [ ] Ejecutar **Floyd** para calcular una matriz de distancias mínimas entre todos los pares.
- [ ] Elaborar un plan de visitas que pase por las localidades con estaciones críticas (M2), salga de la base y regrese a ella, utilizando Floyd y el criterio del **vecino más cercano**.
- [ ] Informar los kilómetros totales y explicar por qué el vecino más cercano no garantiza el recorrido óptimo.

**Verificación:** Córdoba puede usarse como base de control para probar Dijkstra antes de ejecutar con la base asignada al grupo.

### M4 — Transmisión comprimida (Huffman)

**Objetivo:** comprimir archivos diarios de mediciones sin pérdida de información.

- [ ] Generar, mediante M6, un archivo de **al menos 5.000 líneas** con el formato de mediciones definido.
- [ ] Implementar `Nodo`, `construirArbolHuffman`, `codificar` y `decodificar`, siguiendo la lectura de la materia.
- [ ] Mostrar las frecuencias y los códigos de cada símbolo; comprobar la propiedad **libre de prefijos**.
- [ ] Calcular tamaño original (**8 bits por carácter**), tamaño comprimido y tasa de compresión.
- [ ] Descomprimir y verificar automáticamente que el texto recuperado sea **idéntico** al original.

**Caso de verificación obligatorio:**

```text
Cadena: WWXWYZWX
Tamaño original: 64 bits
Tamaño esperado con Huffman: 14 bits
```

**Pregunta para el informe:** explicar por qué un archivo de mediciones se comprime bastante, pero menos que un texto con muchos caracteres repetidos, a partir de las frecuencias.

### M5 — Guardias rotativas (Josephus)

**Objetivo:** asignar guardias de fin de semana mediante una regla circular reproducible. Se ubican `N` técnicos en ronda y, contando `M` pasadas, el técnico seleccionado sale de la ronda y recibe el siguiente turno de guardia. El último en permanecer es el suplente de emergencias.

- [ ] Implementar una versión simple con **lista enlazada e iterador**, de complejidad `O(N·M)`.
- [ ] Implementar una versión eficiente con **árbol binario de búsqueda que permita acceder al k-ésimo elemento**, de complejidad `O(N log N)`.
- [ ] Mostrar el calendario para **12 técnicos**, usando el valor de `M` asignado al grupo.

**Caso de verificación obligatorio:**

```text
N = 5, M = 1
Orden de eliminación: 2, 4, 1, 5
Suplente final: 3
```

**Experimento obligatorio:** comparar tiempos de ambas implementaciones con `N = 1.000`, `10.000` y `100.000`, para **al menos dos valores de M**. Presentar tabla y explicar cuándo las diferencias importan.

### M6 — Simulación y pruebas

**Objetivo:** generar datos reproducibles y medir el rendimiento del sistema.

- [ ] Generar estaciones y mediciones con `Random` (Java) o `random.Random` (Python) usando la **semilla del grupo**.
- [ ] Demostrar que dos generadores con la misma semilla producen la misma secuencia y con semillas diferentes no.
- [ ] Verificar la uniformidad de los números generados (por ejemplo, baterías de 0 a 100) contando frecuencias y comparándolas con el valor esperado.
- [ ] Medir tiempos con `System.nanoTime()` o `time.perf_counter_ns()`.
- [ ] Reunir los experimentos de M1, M2 y M5 en un informe de rendimiento.

## 5. Integración: menú principal

Los módulos deben funcionar como **un único sistema**. Como mínimo, el menú de consola permitirá:

- [ ] **Cargar la red:** generar `N` estaciones e insertarlas tanto en la tabla hash (M1) como en el AVL (M2).
- [ ] **Buscar una estación por código** (M1).
- [ ] **Consultar estaciones por rango numérico** (M2).
- [ ] **Listar estaciones críticas** (M2) y **planificar visitas** para la cuadrilla (M3).
- [ ] **Generar, comprimir, descomprimir y verificar** mediciones diarias (M4).
- [ ] **Generar el calendario de guardias** (M5).
- [ ] **Ejecutar experimentos de rendimiento** (M6).

## 6. Restricciones y recursos permitidos

Las estructuras principales deben ser **implementaciones propias**; no está permitido reemplazarlas por colecciones que resuelvan directamente el problema.

| Uso | Java permitido | Python permitido |
| --- | --- | --- |
| Josephus, versión simple | `LinkedList` + `Iterator` | `list` o `collections.deque` |
| Huffman, cola de prioridad | `PriorityQueue` | `heapq` |
| Huffman, tabla de códigos | `HashMap` | `dict` (solo para esta tabla) |
| Aleatoriedad con semilla | `java.util.Random` | `random.Random(semilla)` |
| Medición de tiempos | `System.nanoTime()` | `time.perf_counter_ns()` |

**Prohibiciones para M1, M2 y M3:**

- **Java:** `HashMap`, `TreeMap` y bibliotecas de grafos como reemplazo de las estructuras pedidas.
- **Python:** `dict`, `set` y bibliotecas externas como `networkx` o `sortedcontainers` para reemplazar las estructuras pedidas.

En caso de elegir Python, el grupo deberá adaptar el código de las lecturas (que está en Java) y ser capaz de explicar esa adaptación.

## 7. Parámetros asignados por grupo

Cada grupo deberá utilizar sus parámetros en los experimentos, resultados del informe y exposición.

| Grupo | Semilla Random | Tamaño inicial hash | Base de cuadrilla (M3) | M de Josephus (M5) |
| ---: | ---: | ---: | --- | ---: |
| 1 | 1026 | 11 | Río Cuarto | 1 |
| 2 | 2026 | 13 | Villa María | 2 |
| 3 | 3026 | 17 | San Francisco | 3 |
| 4 | 4026 | 19 | Villa Dolores | 4 |
| 5 | 5026 | 23 | Deán Funes | 1 |
| 6 | 6026 | 29 | Río Cuarto | 2 |
| 7 | 7026 | 31 | Villa María | 3 |
| 8 | 8026 | 37 | San Francisco | 4 |
| 9 | 9026 | 41 | Villa Dolores | 1 |
| 10 | 10026 | 43 | Deán Funes | 2 |
| 11 | 11026 | 47 | Río Cuarto | 3 |
| 12 | 12026 | 53 | Villa María | 4 |
| 13 | 13026 | 59 | San Francisco | 1 |
| 14 | 14026 | 61 | Villa Dolores | 2 |
| 15 | 15026 | 67 | Deán Funes | 3 |

**Grupo asignado:** `Pendiente de completar`  
**Semilla:** `Pendiente` · **Tamaño inicial de hash:** `Pendiente` · **Base:** `Pendiente` · **M de Josephus:** `Pendiente`

## 8. Cronograma y exposición oral

| Instancia | Avance esperado |
| --- | --- |
| Inicio | Confirmar integrantes y número de grupo en Canvas. |
| Antes del 1.er parcial | Avance libre y consultas en clase. |
| Exposición 1 | Modelo de datos, M6 con semilla, M1 con ambas exploraciones, ABB de M2 y diseño del resto. |
| Exposición 2 | Lo anterior más M2 completo y su experimento; M3 con representación, BFS, DFS y Dijkstra; M4 y M5 en desarrollo. |
| Exposición 3 | Seis módulos funcionando e integrados; resultados preliminares de experimentos. |
| Fin de la materia | Entrega final completa, según indicaciones de Canvas. |

El orden de exposición se sortea o acuerda en clase. La exigencia se ajusta a la semana asignada.

**Dinámica de exposición:**

- **15 minutos de exposición + 10 minutos de preguntas**.
- Deben asistir todos los integrantes; la ausencia sin justificación implica no tener nota de exposición.
- Se exige una **demostración en vivo del programa** (no capturas).
- El docente puede preguntar por **cualquier módulo a cualquier integrante**.
- Se puede pedir una traza manual en el pizarrón o un cambio pequeño en el código durante la exposición.

## 9. Entrega final

Preparar un archivo **`TPI_Grupo_NN.zip`** con:

- [ ] **Código fuente** ordenado en paquetes o módulos, compilable y ejecutable sin errores. Se recomienda un repositorio Git con commits de todos los integrantes e incluir su enlace.
- [ ] **README** con instrucciones de compilación y ejecución, integrantes y rol principal.
- [ ] **Informe técnico PDF de máximo 15 páginas**: descripción de la solución, decisiones de diseño justificadas, complejidades, experimentos (tablas o gráficos), casos de verificación y conclusiones.
- [ ] **Bitácora de uso de IA** (ver apartado siguiente).
- [ ] **Archivos de datos:** grafo de rutas y muestras del archivo de mediciones comprimido y descomprimido.

El enunciado indica subir el ZIP a un **Drive**, mientras que el cronograma menciona entrega final por **Canvas**; confirmar en clase o en Canvas el procedimiento exacto de entrega.

## 10. Uso de inteligencia artificial

El uso de herramientas de IA **está permitido**, pero el equipo debe comprender, controlar y defender todo lo entregado.

Se solicita una **bitácora de 1 a 2 páginas** que incluya:

1. Herramientas de IA utilizadas y para qué tareas.
2. Al menos **un error o sugerencia incorrecta detectada** y cómo se corrigió.
3. Partes del proyecto escritas o modificadas sustancialmente por los integrantes.

Los casos de Huffman y Josephus, así como los parámetros propios del grupo, deben utilizarse para verificar el programa.

## 11. Equipo de trabajo

| Integrante | Rol principal |
| --- | --- |
| Por completar | Por definir |
| Por completar | Por definir |
| Por completar | Por definir |
| Por completar | Por definir |
| Por completar (si corresponde) | Por definir |

## 12. Instalación y ejecución

**Lenguaje elegido:** `Pendiente de definir`  
**Versión requerida:** Java JDK 17+ o Python 3.10+.

> Esta sección debe completarse cuando se defina la estructura real del repositorio. No se incluyen comandos de ejecución inventados.

**Pasos pendientes de documentar:**

1. Requisitos e instalación de dependencias.
2. Compilación (si se utiliza Java).
3. Comando para iniciar el menú principal.
4. Ubicación y formato de los archivos de datos.
5. Ejecución de pruebas y experimentos.

## 13. Recomendaciones de desarrollo

1. Confirmar el grupo, registrar sus parámetros y distribuir módulos.
2. Comenzar por **M6 y M1**: generar los datos y poder encontrarlos facilita el resto.
3. Verificar primero los **casos pequeños de las lecturas** antes de escalar a miles de elementos.
4. Comprender y probar manualmente el código sugerido por IA.
5. Ensayar la exposición haciendo que todos los integrantes expliquen distintos módulos.

---

**Fuente:** *Trabajo Práctico Integral — Proyecto RedSens*, Taller de Algoritmos y Estructura de Datos II, Universidad Siglo 21 (enunciado de 8 páginas).
