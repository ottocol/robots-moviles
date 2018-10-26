
# Robots móviles
# SLAM


---

## SLAM
## Introducción al problema del SLAM

---


**Mapeado** y **localización** dependen el uno del otro

- Para localizarse hace falta tener un mapa
- Para construir un mapa necesitamos saber dónde estamos en cada momento 

---

## SLAM (Simultaneous Localization And Mapping)

Resolver los dos problemas **simultáneamente**

---


![](imag_intro_slam/slam.png)

Notas:

- (a) Al comienzo, la incertidumbre en la posición del robot es 0. Desde aquí el robot observa un *landmark* cuya posición tendrá una incertidumbre asociada al modelo de error del sensor
- (b) Conforme el robot se va moviendo, la incertidumbre en su posición crece según el modelo de error de la odometría.
- (c) Cuando el robot observa nuevos *landmarks* la incertidumbre de su posición es mayor que en (a) ya que es una combinación de la incertidumbre en la localización más la del sensor. Es decir, *la incertidumbre del mapa se correlaciona con la del robot*. 
- (e) Cuando el robot observa *landmarks* con una posición más precisa (por ejemplo porque los ha observado al comienzo), la incertidumbre en su propia localización *decrece*. Además el mapa entero se actualiza y la incertidumbre de todos los *landmarks* vistos previamente también decrece.  Esto se conoce en SLAM como **cerrar el ciclo** (*closing the loop*)

---

![inline](imag_intro_slam/SLAM_graph_model.png)

- $x_1,...x_t$ son las posiciones del robot
- $m$ es el mapa, típicamente formado por *landmarks*
- $z_1,...z_t$ son las detecciones de *landmarks* hechas por los sensores
- $u_1,...u_t$ son los *movimientos* hechos por el robot

---

Dos variantes del problema:

- *Online SLAM*: solo nos interesa la posición actual del robot (también llamado *filtering*)

$$
p(x_t,m | z_{1:t},u_{1:t})
$$

- *Full SLAM*: nos interesa toda la trayectoria (también llamado *smoothing*)

$$
p(x_{1:t},m|z_{1:t},u_{1:t})
$$


---

## Algunos algoritmos

- *Filtering*
    + EKF SLAM
    + Filtros de partículas "Rao-Blackwellizados" 

- *Smoothing*
    + Graph SLAM
    

---

## SLAM
## EKF SLAM

---

El mismo algoritmo EKF que usábamos para localización, pero ahora en el estado tendremos, además de la **posición** del robot, también la de los **landmarks**, es decir tiene “dos partes” $\mu_t = (x_t, m)$

![](estado_EKFSLAM.png)

**Dimensionalidad**: En el caso de un mapa bidimensional con N landmarks puntuales, tendremos una dimensión de 3 + 2N (x,y,*theta*) robot + (x,y) para cada landmark

La matriz de covarianza se divide en 4 partes:
- la parte amarilla representa la covarianza en las posiciones del robot, sin tener en cuenta el mapa
- la Azul la de los landmarks
- las partes verdes representan la correlación entre la posición del robot y los landmarks

---

Recordad el EKF para localización

**Extended\_Kalman\_Filter$(\mu_{t-1},\Sigma_{t-1},u_t,z_t)$:**

1. **Predicción** 
2. $\bar\mu_t=g(u_t,\mu_{t-1})$
3. $\bar\Sigma_t=G_t\Sigma_{t-1}G^T_t+Q_t$
4. **Corrección**
5. $K_t= \frac{\bar\Sigma_t H_t^T}{(H_t \bar\Sigma_tH^T_t)}$
6. $\mu_t= \bar\mu_t + K_t(Z_t-h(\bar\mu_t))$
7. $\Sigma_t=(I-K_tH_t)\bar\Sigma_t$

Donde H y G son los *jacobianos*

$$H_t=\frac{\partial h(\bar\mu_t)}{\partial x_t} \space \space G_t=\frac{\partial g(u_t,\mu_{t-1})}{\partial x_{t-1}}$$


Las fórmulas van a ser prácticamente iguales, lo que cambia es que **tenemos  además la parte de los *landmarks***

---

## Inicialización

El origen de coordenadas es de donde parte el robot. Por el momento no tenemos *landmarks*

$$\bar \mu_1=\begin{bmatrix}x\\\ y\\\  \theta\end{bmatrix}= \begin{bmatrix}0\\\ 0\\\ 0\end{bmatrix} \space \space \space \bar \Sigma_1=\begin{bmatrix}0 & 0 & 0\\\ 0 & 0 & 0\\\ 0 & 0 & 0\end{bmatrix} $$

---

## Predicción

Solo se actualiza la parte en la que intervienen las coordenadas del robot (ya que se supone el mapa estático). Por esto la actualización se puede implementar con coste *lineal* con el número de *landmarks*.

 ![](correccion_ekf_slam.png)

---

## Corrección

- La ganancia K implica **toda la matriz de covarianza**, y eso hace que cambie la correlación entre la posición del robot y la de los landmarks.

---

## Nuevo landmark

Cada vez que detectamos un nuevo landmark añadimos una fila al vector de estado y una fila y una columna a la matriz de covarianzas

---

## SLAM 
## SLAM con filtros de partículas

---

## Recordando filtros de partículas para localización

- Cada partícula es una hipótesis de pose $(x,y,\theta)$ del robot
- **Sampling**: del modelo de movimiento (*predicción*)
- **Importance sampling** (*corrección*)
    + Asignar peso con el modelo del sensor 
    + Remuestrear basándose en ese peso

¿Podríamos aplicar la misma idea a SLAM?

---

Por desgracia ¡la dimensionalidad del problema del SLAM es mucho mayor!. **De 3 dimensiones pasamos fácilmente a cientos** (más exactamente a 2N+3, con N=número de *landmarks*)

Necesitaríamos un número enorme de partículas para cubrir adecuadamente un espacio tan grande

---


Vamos a intentar aplicar algún truco para "reducir" la dimensionalidad. La clave va a estar en separar el problema en varias partes (cada una con mucho menos dimensiones)

---

## Factorización del SLAM

Por ciertas propiedades básicas de la probabilidad condicional, podemos hacer

$$P(x_{1:t},l_{1:m} | z_{1:t}, u_{0,t-1}) = $$
$$ P(x_{1:t}|z_{1:t}, u_{0,t-1}) P(l_{1:m}|x_{1:t},z_{1:t})$$

Sobre esto podemos aplicar un filtro de partículas "Rao-Blackwellizado": aplicar partículas sobre una parte del problema y un método analítico (por ejemplo un EKF) sobre la otra 

---


![](imag_intro_slam/grafo_fastslam.png)

Suponiendo conocidas las poses del robot, $x_{1:t}$, las posiciones de los *landmarks* son independientes entre sí (p.ej. tener más información sobre la posición de $m_1$ no nos da más datos sobre $m_2$). Formalmente esto se conoce como *d-separación* en redes bayesianas.

---

## FastSLAM

$$P(x_{1:t},l_{1:m} | z_{1:t}, u_{0,t-1}) = $$ 
$$ P(x_{1:t}|z_{1:t}, u_{0,t-1}) P(l_{1:m}|x_{1:t},z_{1:t}) = $$
$$ P(x_{1:t}|z_{1:t}, u_{0,t-1}) \prod_{i=1}^M P(l_i|x_{1:t},z_{1:t})$$

---

![](imag_intro_slam/descomposicion_fastslam.png)

---

- Cada partícula representa una posible trayectoria del robot (explícitamente solo guardamos la última posición)
- Para cada partícula mantenemos M EKFs de tamaño 2x2 que se actualizan por separado 

![](imag_intro_slam/estructura_fastslam.png)

---

## Algoritmo FastSLAM

![](imag_intro_slam/algoritmo_fastslam.png)

---

## FastSLAM y "cerrar el ciclo"

- Ya vimos que el EKF mantiene **explícitamente** las correlaciones entre las poses de los *landmarks* y la del robot. Eso permite reducir la incertidumbre en todas ellas cuando se *cierra el ciclo*.

- En FastSLAM las correlaciones se mantienen a través de las partículas, que son distintas hipótesis sobre la trayectoria. Cuando se cierra el ciclo se reduce la incertidumbre descartando las que no "cuadran" con las medidas de los sensores.


---

<iframe width="569" height="427" src="https://www.youtube.com/embed/jBPZIU6AIS0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Problema: como solo podemos mantener un número limitado, se "perderán" muchas trayectorias posibles, si hay pocas partículas al final acaban teniendo todas una "historia común" en algún punto (no podemos cerrar ciclos muy largos).

---

## FastSLAM para rejillas de ocupación

La extensión del algoritmo es bastante directa:

1. Cada **partícula** tiene un **mapa asociado** (una rejilla)

![](imag_intro_slam/particula_y_su_mapa.png) <!-- .element: class="stretch" -->

---

## FastSLAM para rejillas de ocupación (II)

2. Como una **partícula** supone una **posición** del robot, para actualizar el mapa podemos aplicar el algoritmo de **mapeado con posición conocida** que ya  visteis en el tema anterior


---

Cosas que faltan por ver (o quizá se pueden obviar)

- El problema de la asociación de datos
- FastSLAM 2.0 vs 1.0
- Inicializar landmarks en los distintos algoritmos