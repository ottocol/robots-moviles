


En esta práctica vamos a programar un robot móvil para que realice una tarea compleja que implique navegar por el entorno realizando una serie de subtareas. La tarea puede ser la que queráis, por ejemplo:

- Patrullar por un edificio pasando por diversos puntos y detectando posibles "intrusos"
- Navegar por una habitación detectando objetos tirados por el suelo y recogiéndolos si el robot tiene un brazo capaz de ello
- Navegar por un entorno en que el robot va detectando señales (por ejemplo flechas) que le van indicando a dónde ir
- Intentar localizar una pelota en el suelo e irla empujando para marcar gol en una portería

> NOTA: según sea la tarea que diseñéis es posible que no la podáis probar más que en simulación. Se valorarán las pruebas en los Turtlebot reales (y además son más divertidas que las simulaciones :)) pero también podéis usar otros modelos de robots distintos y simplemente simularlos en ROS.

La tarea puede ser muy diversa pero en general vais a necesitar tres tipos de elementos para implementarla:

- Un formalismo para especificar **cómo se coordinan las subtareas**: por ejemplo habrá subtareas que se deberán realizar en una secuencia ("primero ve al *waypoint* 1 y luego al 2"), otras serán condicionales ("navega aleatoriamente hasta que te encuentres una pelota"), otras tareas serán en paralelo... En robótica para coordinar este tipo de subtareas se pueden usar varios mecanismos, como las máquinas de estados finitos y los *behavior trees*.
- Algunas subtareas pueden requerir ***navegar* a puntos concretos del mapa** (por ejemplo una tarea de vigilancia). Para eso podéis usar el *stack* de navegación de ROS que se explica en la sección siguiente.   
- Otras subtareas serán de **detección de condiciones** (por ejemplo, "si estoy en un pasillo navego hacia adelante hasta que se acabe", o "buscar una pelota de color rojo"). Para estas tendréis que hacer uso de los sensores del robot.


## El *stack* de navegación de ROS

El *stack* de navegación de ROS es un conjunto de nodos, de distintos paquetes, que sirven para gestionar las tareas de navegación en robots móviles. Básicamente el *stack* nos permite mover el robot a un destino siguiendo un camino "razonablemente" corto (puede no ser el óptimo) y a la vez evitando los obstáculos. Estos pueden ser de dos tipos: los que están reflejados en el mapa (paredes, vallas, ...) y los obstáculos "temporales" que se va encontrando por el camino (sillas, personas, ...). Por tanto el *stack* de navegación usará tanto información del mapa (que habéis construido en la práctica anterior) como datos de los sensores (laseres, sonares, cámara,...).

La siguiente figura, tomada de [esta presentación](https://www.dis.uniroma1.it/~nardi/Didattica/CAI/matdid/robot-programming-ROS-introduction-to-navigation.pdf) muestra la estructura del *stack*. Como puede verse es bastante compleja.

![](imag/navigation_stack.png)

Fijáos en que el *stack* necesita localizarse y por tanto necesita un mapa, por lo que **tendréis que disponer del mapa** en formato YAML para el entorno que estéis probando.

### Probar el *stack* de navegación con el Turtlebot simulado

Como se puede apreciar en la figura anterior el *stack* está formado por muchos nodos y ponerlos todos en marcha simultáneamente puede resultar complicado. Afortunadamente en los paquetes de turtlebot que hay instalados tenemos un par de *demos* que nos pueden servir:

Para probar con el simulador Gazebo (en 3D) y el mundo por defecto (`playground.world`)

```bash
#simulador
roslaunch turtlebot_gazebo turtlebot_world.launch
#stack de navegación
roslaunch turtlebot_gazebo amcl_demo.launch
#rviz para visualizar el resultado y poder fijar destinos
roslaunch turtlebot_rviz_launchers view_navigation.launch
```

Una vez puesto todo en marcha, puedes probar en `rviz` a fijar un destino para el robot. Clica en el botón que aparece en la barra superior llamado `2D Nav Goal`. Luego clica en el punto del mapa al que quieres que se mueva el robot y opcionalmente arrastra para indicar la orientación final del robot. Si hay un camino factible el robot debería seguirlo y moverse hasta el destino.

> NOTA: ya veremos en clase de teoría con más detalle cómo funciona el *stack* de navegación. De momento podéis observar en los nodos de rviz (panel izquierdo) que hay un `Local Planning` (evitar obstáculos basándose en lo que detectan los sensores) y un `Global Planning` (planificar la mejor trayectoria según el mapa, sin chocar con los obstáculos reflejados en él)

Si quieres usar un mundo propio tendrás que hacerlo de este modo:

```bash
#simulador
roslaunch turtlebot_gazebo turtlebot_world.launch world_file:=<trayectoria al fichero con el mundo>
#stack de navegación
roslaunch turtlebot_gazebo amcl_demo.launch map_file:=<trayectoria al .yaml del mapa>
#rviz se lanza igual
roslaunch turtlebot_rviz_launchers view_navigation.launch
```

También podéis probar con el simulador *stage* (un simulador ligero en "2D y 1/2", apropiado para máquinas que no sean demasiado potentes)

```bash
#Esto lanza el simulador, el stack de navegación y rviz todo en uno
roslaunch turtlebot_stage turtlebot_in_stage.launch
```

## Especificación de tareas con Behavior Trees

Aunque podríamos especificar cómo se van coordinando las subtareas simplemente con las construcciones habituales de nuestro lenguaje de programación (`if else`, `while`,...) en general esta forma no se suele recomendar por generar código demasiado *ad hoc*, "embrollado" y poco reutilizable. En su lugar en robótica se usan otros formalismos que nos permiten encapsular las tareas de forma más modular y 

## Documentación a entregar

La documentación de la práctica es una parte muy importante en la puntuación final. Se debe entregar una documentación (cualquier formato: PDF, HTML, etc.) con los siguientes puntos:

1. Indice del contenido de la práctica 
2. Contenido de la práctica: 
    - Explicación de la tarea que se quiere abordar
    - *Behavior tree* resultante y descripción general de su comportamiento
    - Descripción de cada uno de los nodos, detallando en su caso los algoritmos/métodos que haya usado cada uno de ellos: por ejemplo si un nodo busca objetos de color azul decir cómo lo habéis hecho.
    - Experimentos realizados en simulación y si es posible con el robot real
3. Conclusiones/discusión.

## Baremo

- Como es lógico la nota estará relacionada con la dificultad de la tarea pero también con la documentación de la práctica. Debéis no solo documentar la implementación que habéis hecho sino también **todos** los experimentos en la simulación y con el robot real (¡aun los que no funcionen!, en estos podéis analizar qué es lo que no ha funcionado y cómo lo pretendéis resolver)
- Para sacar hasta un 6 podéis implementar la tarea en ROS simplemente usando C++/Python sin necesidad de usar ningún formalismo adicional como los árboles.
- Para una nota hasta el 7.5, usad *behavior trees* para modelar las subtareas 
- Para el 8 en adelante, alguna de estas ideas (u otras que podéis proponer a los profesores) 
    - Implementar *ADEMÁS* del *behavior tree* una máquina de estados finitos, por ejemplo con SMACH o FlexBE. Comparar ambos enfoques indicando cuál os parece más sencillo/intuitivo y si creéis que vuestras conclusiones se pueden extender a cualquier tipo de tarea o según los casos sería mejor un enfoque u otro.
    - Implementar tareas que hagan uso de visión: colores, formas, reconocimiento de objetos. Podéis usar cualquier paquete ROS/librería de terceros que encontréis. La nota dependerá de la dificultad de uso y también de la experimentación realizada. 

## Normas de entrega de la práctica:

- La práctica se entregará antes de las 24 horas del domingo 23 de diciembre de 2018.
- La entrega se realizará a través del Campus Virtual de la Universidad de Alicante. Se habilitará una entrega específica en el CV para subir el código fuente y la memoria asociada a la práctica.
      

