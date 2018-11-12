


En esta práctica vamos a programar un robot móvil para que realice una tarea compleja que implique navegar por el entorno realizando una serie de subtareas. La tarea puede ser la que queráis: por ejemplo 

//TO-DO: ejemplos de tareas y condiciones que deben cumplir para ser evaluables

En robótica se usan distintos formalismos para especificar tareas compuestas de secuencias de subtareas, subtareas que se ejecutan solo si 

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

- Documentar **todos** los experimentos, aun los que no funcionen
- Para el 8 en adelante, alguna de estas ideas (u otras que podéis proponer a los profesores) 
    - implementar también máquinas de estados finitos, por ejemplo con SMACH o FlexBE. Comparar ambos enfoques
    - implementar tareas que hagan uso de visión: colores, formas, reconocimiento de objetos 

## Normas de entrega de la práctica:

- La práctica se entregará antes de las 24 horas del domingo 23 de diciembre de 2018.
- La entrega se realizará a través del Campus Virtual de la Universidad de Alicante. Se habilitará una entrega específica en el CV para subir el código fuente y la memoria asociada a la práctica.
      

