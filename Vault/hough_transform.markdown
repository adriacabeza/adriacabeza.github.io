Hoy quería retomar cosas sobre la Visión por Computador tradicional (no hay ninguna razón particular para ello). Así que este post va a ser sobre una de las técnicas de extracción de características más utilizadas: **Transformación de Hough**

La técnica de Hough es particularmente útil para calcular una descripción global de las características y encontrar instancias de un objeto basadas en su forma.

Veamos el caso más sencillo de la transformada de Hough: la detección de líneas rectas.

<div align="center">
<img class="ui image" src="https://docs.opencv.org/3.4/houghlines3.jpg">
</div>
<br>

Una línea $y=mx+n$ puede representarse como un punto $(n,m)$ en el espacio de parámetros. Sin embargo, las líneas verticales plantearían un problema, darían lugar a valores no limitados del parámetro de la pendiente *m*. Por eso expresamos la recta en una [forma normal de Hesse](http://www.nabla.hr/CG-LinPlanIn3Dan2DA3.htm):

<div align="center">
$p = x \cdot cos(\theta)+ y\cdot sin(\theta)$
<br>
<img class="ui image" src="https://www.math-only-math.com/images/straight-line-in-normal-form.png">
Fuente: Math Only Math
</div>
<br>

donde $p$ es la distancia del origen al punto más cercano de la recta (que se acerca perpendicularmente a la misma) y $\theta$ es el ángulo entre el eje x y la recta que une el origen con ese punto más cercano. Así, la ecuación anterior se satisface en todos los puntos $x,y$ que pertenecen a la recta $(p, \theta)$.

## El secreto: Mapeo del espacio de la imagen al espacio de Hough

Por lo tanto, es posible asociar un par $(p, \theta)$ con cada línea de la imagen. El plano $(p,\theta)$ se denomina a veces espacio de Hough para el conjunto de rectas en dos dimensiones.

Dado un único punto en el plano ($x,y$), el conjunto de todas las rectas que pasan por ese punto corresponden a una curva sinusoidal en el plano $(p, \theta)$.

<div align="center">
<img class="ui image" src="https://miro.medium.com/max/700/0*JT-hhPkp-Tx4ywtu.jpg">
Fuente: Medium
</div>
<br>

Así, podemos decir que un punto en el espacio ($x,y$) puede resultar en una sinusoide en el espacio de Hough. Tomando un punto y deslizando por todos los valores posibles de $\theta$ obtenemos $p$ valores que forman una sinusoide (recordemos la forma normal de Hesse $p = x \cdot cos(\theta)+ y\cdot sin(\theta)$). 

La parte mágica viene cuando comprobamos las sinusoides formadas por puntos de una recta. En el espacio de Hough todas estas sinusoides se cruzarán exactamente en un punto.
<div align="center">
<img class="ui image" src="https://miro.medium.com/max/700/0*VPVsLApWiEayRGdQ.jpg">
</div>
<br>


Esto significa que el problema de detectar puntos colineales puede convertirse en el problema de encontrar curvas concurrentes. Ahora sólo tenemos que encontrar intersecciones en el espacio de Hough. ¿No es genial?
<br>
<div align="center">
<img class="ui image" src="https://miro.medium.com/max/700/0*KoAmeuoTu9Uz0eNW.png">
</div>



### Fuentes

- [Edinburgh University](http://homepages.inf.ed.ac.uk/rbf/HIPR2/hough.htm)
- [Standford University](https://stacks.stanford.edu/file/druid:rz261ds9725/Ho_Papusha_RealTimeHoughTransform.pdf)
- [Medium](https://medium.com/@tomasz.kacmajor/hough-lines-transform-explained-645feda072ab)
- [Wikipedia](https://es.wikipedia.org/wiki/Transformada_de_Hough)