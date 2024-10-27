# P3 Obstacle-avoidance

## Introducción

Esta práctica consiste en atravesar un circuito esquibando obstaculos en el camino. El coche solo dispone de laser, sus coordenadas y la referencia del camino a seguir desglosado en objetivos.

Un aproximación a la solución, es basar el comportamiento en un modelo VFF , que tome los obtaculos detectados por el laser.

![imagen](https://github.com/user-attachments/assets/250a862d-4c5b-4272-b8d2-dc0d8dcf5f69)

## Modelo VFF

El modelo consiste en recrear una serie de fuerzas que actuan sobre el coche, teniendo en cuenta el entorno que le rodea y el objetivo al que queremos llegar.
Este modelo basa su comportamiento en un escenario local y cercano, esto permite reactividad en su comportamiento frente a obstaculos.

Es necesario emplear un sistema de sensores para poder reconocer el medio, de esta forma podremos transformar estas detecciones/obstaculos, en fuerzas de repulsión sobre nuestro modelo. Ademas, es necesario tomar un objetivo con el cual fijar una fuerza de atracción que impulse al modelo hacia este.

![Modelo VFF](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRExno3pU1tAoEtRTgtIqoEAv3tPE9YhA_gZQ&s)

_Rojo: Vector repulsivo; Azul: Vector atractivo; Verde: vector_resultado_

### Vector atractivo

Para conseguir el vetor atractivo, se ha empleado una serie de objetivos desplegados por el circuito. Estos puntos estan basados en coordenadas globales y tras pasarlos a coordenadas relativas al robot, conseguiremos el vector guía para generar el vector atractivo.

**Formula empleada**

Una vez tengo el objetivo, calculo el angulo que hay entre la posición del coche y este, usando la arcotangente. Con esto conseguiremos tener una orientación.
Despues calculo el modulo del vector, así tengo la dirección en la que el coche deberá moverse.
Por último, dependiendo del modulo, calculo los ejes del vector atractivo multiplicando el modulo por el seno o coseno dependiendo de que eje quiera obtener. Y en caso de tener un modulo muy alto, hago que no pueda subir su valor a mas de 2.

### Vector repulsivo

Este es el vector que más cuesta extraer.
Para calcular este vector, necesitaremos los datos tomados por el laser. El laser devuelve un array de 180 posiciones, las cuales equivalen a los grados en los que hay un haz de laser y que contiene las distancias hasta el obstaculo encontrado.
Para trabajar mejor con los datos, se pasa de grados a radianes.
Como la distancia no deja de ser el modulo del vector, tengo que coonseguir que sea inversamente proporcional para trasladarlo a mi vector repulsivo. Por ello empleo esta formula.

**Formula empleada**

$$\left( 3 - e^{x-3}\right)$$

![imagen](https://github.com/user-attachments/assets/472b39d6-e232-4541-93f5-45771befe189)

Esta imagen representa el resultado de aplicar la formula sobre el modulo del vector objetivo.

Mas adelante se emplea la misma tecnica que con el vector atractivo; el modulo es multiplicado por el coseno del angulo para obtener X y el seno para obtener Y.
Por último, se hará una media de todas las coordenadas X e Y, por separado, para determinar cual es el vector repulsivo resultante y dominante.

### Vector resultado

Una vez calculado los vectores atractivo y repulsivo, sumaremos sus componentes X e y por separado. No sin antes acentuar los valores, multiplicando los distintos vectores por las constantes ælpha y ßeta. Estas serviran para decidir que fuerza influirá más en el resultado del vector.

Como hemos visto en imagenes anteriores, el vector resultado sera el que determine en que dirección a de moverse el coche para esquivar el obstaculo y seguir el objetivo al mismo tiempo. 

**_Para la velocidad lineal y angular, se ha empleado el eje X y el eje Y respectivamente_**

## Primeras experiencias y resultados

Las primeras pruebas que realice, consistieron en determinar que expresión matematica era la mas indicada para obtener una curva de evolución del modulo del vector atractivo. Hice uso de la herramienta GeoGebra[^1].

[^1]: https://www.geogebra.org/classic?lang=es

Al principio opte por una función logaritmica, pero su evolución despues de los calculos restantes, en el proceso de creación del vector, tenía un comportamiento exponencial creciente. Este no era de utilidad.

Más tarde, probé a transformar una expresion exponencial de **e** en una curba descendiente. El resultado fue el esperado y es el que actulamente se encuentra en la solución.

Despues de implentar el resto de vctores y tras un gran numero de pruebas, decidi implementar un filtro de paso bajo para suabizar las velocidades lineales y angulares. El resultado fue muy positivo al principio, pero tras cambiar el metodo de obtención del vector atractivo, dejo de funcionar como antes.

Luego, empece a modificar los valores de ælpha y ẞeta, pero no llegaba a encontrar una solución estable. El vector atractivo era muy cambiante y producia comportamientos inesperados.

Finalmente conseguí que el coche se moviera por el circuito esquibando los primero obstaculos. Pero actualmente, tras una de las ultimas curvas, el coche se detiene debido a un minimo tras un cambio brusco en la posicíon del target.

### Video del funcionamiento[^2]

[^2]: https://urjc-my.sharepoint.com/:v:/g/personal/e_martint_2022_alumnos_urjc_es/ERsUYvq6aqtDi-2qGU6JUqUBz2WiHx-MeGGQmXsDIgcPug?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=bYhh0U

## Conclusión

El metodo de control por Virtual Force Field (VFF), es muy util para generár un sistema reactivo a eventos locales.
La implementación no es complicada, pero requiere de una parametrización y configuración extensa.



