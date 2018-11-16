
<!-- .slide: class="titulo" -->


# Robots móviles <!-- .element: class="column half" -->

## Tema 6. Navegación y planificación de trayectorias <!-- .element: class="column half" -->

---

<!-- .slide: class="titulo" -->


# Navegación  <!-- .element: class="column half" -->

## Introducción: ¿qué es navegar? <!-- .element: class="column half" -->

---

## Navegación

Conjunto de técnicas y algoritmos necesarios para que un robot móvil pueda llegar hasta su destino por el **camino más corto** posible y **sin chocar con los obstáculos**

![](imag/path_planning.png)

---

## Global vs local

- **Navegación global**: encontrar el camino óptimo (más corto o más adecuado) Necesitaremos:
    +  Un **mapa**
    +  Un algoritmo para el **cálculo del camino** óptimo según el mapa
- **Navegación local**: no chocar con obstáculos no reflejados en el mapa. Necesitaremos:
    + Información de los **sensores de rango**
    + Un algoritmo de **evitación de obstáculos**, que calcule la mejor dirección de movimiento para evitarlos (sin alejarnos demasiado del camino óptimo)

---

<!-- .element: class="caption" --> 
Tomada del curso ["Introduction to Mobile Robotics"](http://ais.informatik.uni-freiburg.de/teaching/ss17/robotics/) de Wolfram Burgard 

![](imag/2_layer_arch.png) <!-- .element class="stretch" -->



---

## ROS: navegación global (izq.) y local (der.)

Se muestra el *costmap* global y el cálculo del camino,  y el *costmap* local

![](imag/global_nav.png) <!-- .element: class="column half" -->
![](imag/local_nav.png) <!-- .element: class="column half" -->

Notas:

Como veremos, en ROS se discretiza el espacio en una rejilla y se crea un *costmap* local y uno global, al estilo de las rejillas de ocupación. En cada celda de este "mapa" se almacena el "coste" que tiene para el robot pasar por ella. Costes altos indican que el robot no debería pasar por ahí. 

En el *costmap* global los costes altos están en las zonas ocupadas del mapa y alrededor de ellas, ya que el robot no solo debe evitar chocar contra la pared, además no debe acercarse demasiado, ya que al haber una incertidumbre en la localización podría chocar. 

El *costmap* local se actualiza con la información de los sensores de rango. En este último los colores cálidos indican una zona apropiada para moverse (coste bajo) y los fríos una zona a evitar (coste alto).

---

## El *stack* de navegación en ROS


![](imag/ros_navigation_stack.png) <!-- .element: class="stretch" -->


---

<!-- .slide: class="titulo" -->


# Planificación global  <!-- .element: class="column half" -->

## Cálculo del camino más "corto" <!-- .element: class="column half" -->


---


La gran mayoría de algoritmos de cálculo de rutas trabajan en el **espacio de configuraciones** o CSpace: el espacio formado por las posibles poses del robot

- Tantas dimensiones como grados de libertad tenga el robot
- Buscamos una ruta en este espacio que solo pase por espacio libre

Es un concepto habitual en planificación de movimiento de brazos robot ([Demo](https://www.cs.unc.edu/~jeffi/c-space/robot.xhtml))

---

## CSpace para robots móviles

Típicamente la *pose* se define con $(x,y,\theta)$, lo que daría un CSpace 3D. 

No obstante, vamos a hacer simplificaciones:

- **Ignoraremos la ~$\theta$~**  (el CSpace se queda en 2D). Esto podemos hacerlo si el robot es *holonómico*, es decir puede seguir cualquier trayectoria en el CSpace.
- Podemos suponer que **el robot es un punto**, si **"dilatamos" los obstáculos** al menos en un tamaño igual al radio del robot

---

La operación de dilatación que necesitamos ha sido formalizada en diferentes campos de las matemáticas:
- *Suma de Minkowski* de la forma del robot y los obstáculos 
- En *morfología matemática* la operación se denomina también [*dilatación*](https://es.wikipedia.org/wiki/Morfolog%C3%ADa_matemática#Dilatación)

![](imag/suma_minkowski.png) <!-- .element: class="stretch" -->


---

En ROS se aplica la misma idea, generando un *costmap* llamado *inflation costmap*

![](imag/costmapspec.png) <!-- .element: class="stretch" -->


---

![](imag/inflation.png) <!-- .element: class="stretch" -->

---


## Búsqueda del camino más corto

Hay infinitos caminos posibles entre un origen y un destino, pero el espacio de búsqueda no puede ser infinito, hay que *restringirlo*. Normalmente:

1. Transformar el CSpace en un grafo que contenga todos los caminos a considerar.
2. Aplicar algún algoritmo de búsqueda de camino más corto en grafos.

---

## Conversión del CSpace en un grafo

Dependerá de la representación del mapa:

- *Mapas poligonales*: varias formas:
    + Grafo de visibilidad
    + Grafo de voronoi
- *Rejillas de ocupación*: cada celda será un nodo, conectado con sus 8 vecinos más inmediatos. 

![](imag/rejilla_8_conectada.png) <!-- .element: class="stretch" -->

---

## Grafo de visibilidad

Grafo cuyos nodos son los vértices de los polígonos, y los arcos las conexiones entre ellos que no intersectan ningún obstáculo.

<p class="caption">Tomado de <a href="https://www.slideshare.net/GauravGupta527/visibility-graphs">https://www.slideshare.net/GauravGupta527/visibility-graphs</a></p>

![](imag/grafo_visibilidad.png) <!-- .element: class="stretch" -->


---

## Algoritmos de búsqueda de camino más corto

- Algoritmos clásicos de búsqueda en grafos el más típico es **Dijkstra**
- Guiados por heurísticas: el más usado es A*

---

## A*

- Función de evaluación para cada nodo: $f(n) = g(n) + h(n)$
    + $g(n)$: coste del camino ya recorrido
    + $h(n)$ una heurística admisible (una estimación "optimista") del camino que queda 


---

## Otros algoritmos: D* y D* lite

- Similares a A*, aunque parten del destino en lugar del origen
- Pueden *replanifica* trayectorias: pueden "reparar" la trayectoria de modo incremental si hay cambios en el grafo 


---

## Búsqueda del camino más corto en ROS

Paquete `global_planner`, implementa Dijkstra y A*, seleccionables cambiando el parámetro `use_dijkstra`

---

[http://qiao.github.io/PathFinding.js/visual/](http://qiao.github.io/PathFinding.js/visual/)