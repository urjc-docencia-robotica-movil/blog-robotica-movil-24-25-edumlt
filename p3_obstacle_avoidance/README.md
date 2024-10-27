# P3 Obstacle-avoidance

## Introducción

Esta práctica consiste en atravesar un circuito esquivando obstáculos en el camino. El coche solo dispone de láser, sus coordenadas y la referencia del camino a seguir desglosado en objetivos.

Una aproximación a la solución, es basar el comportamiento en un modelo VFF , que tome los objetos detectados por el láser.

![imagen](https://github.com/user-attachments/assets/250a862d-4c5b-4272-b8d2-dc0d8dcf5f69)

## Modelo VFF

El modelo consiste en recrear una serie de fuerzas que actuan sobre el coche, teniendo en cuenta el entorno que le rodea y el objetivo al que queremos llegar.
Este modelo basa su comportamiento en un escenario local y cercano. Esto permite reactividad en su comportamiento frente a obstáculos.

Es necesario emplear un sistema de sensores para poder reconocer el medio, de esta forma podremos transformar estas detecciones/obstaculos, en fuerzas de repulsión sobre nuestro modelo. Además, es necesario tomar un objetivo con el cual fijar una fuerza de atracción que impulse al modelo hacia este.

![Modelo VFF](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRExno3pU1tAoEtRTgtIqoEAv3tPE9YhA_gZQ&s)

_Rojo: Vector repulsivo; Azul: Vector atractivo; Verde: vector_resultado_

### Vector atractivo

Para conseguir el vetor atractivo, se ha empleado una serie de objetivos desplegados por el circuito. Estos puntos están basados en coordenadas globales y tras pasarlos a coordenadas relativas al robot, conseguiremos el vector guía para generar el vector atractivo.

**Formula empleada**

Una vez tengo el objetivo, calculo el ángulo que hay entre la posición del coche y este, usando la arcotangente. Con esto conseguiremos tener una orientación.
Después calculo el módulo del vector, así tengo la dirección en la que el coche deberá moverse.
Por último, dependiendo del módulo, calculo los ejes del vector atractivo multiplicando el módulo por el seno o coseno dependiendo de que eje quiera obtener. Y en caso de tener un módulo muy alto, hago que no pueda subir su valor a más de 2.

### Vector repulsivo

Este es el vector que más cuesta extraer.
Para calcular este vector, necesitaremos los datos tomados por el láser. El laser devuelve un array de 180 posiciones, las cuales equivalen a los grados en los que hay un haz de laser y que contiene las distancias hasta el obstáculo encontrado.
Para trabajar mejor con los datos, se pasa de grados a radianes.
Como la distancia no deja de ser el módulo del vector, tengo que conseguir que sea inversamente proporcional para trasladarlo a mi vector repulsivo. Por ello empleo esta fórmula.

**Formula empleada**

$$\left( 3 - e^{x-3}\right)$$

![imagen](https://github.com/user-attachments/assets/472b39d6-e232-4541-93f5-45771befe189)

Esta imagen representa el resultado de aplicar la fórmula sobre el módulo del vector objetivo.

Más adelante se emplea la misma técnica que con el vector atractivo; el módulo es multiplicado por el coseno del ángulo para obtener X y el seno para obtener Y.
Por último, se hará una media de todas las coordenadas X e Y, por separado, para determinar cual es el vector repulsivo resultante y dominante.

### Vector resultado

Una vez calculados los vectores atractivo y repulsivo, sumaremos sus componentes X e Y por separado. No sin antes acentuar los valores, multiplicando los distintos vectores por las constantes ælpha y ßeta. Estas servirán para decidir qué fuerza influirá más en el resultado del vector.

Como hemos visto en imágenes anteriores, el vector resultado será el que determine en qué dirección ha de moverse el coche para esquivar el obstáculo y seguir el objetivo al mismo tiempo. 

**_Para la velocidad lineal y angular, se ha empleado el eje X y el eje Y respectivamente._**

## Primeras experiencias y resultados

Las primeras pruebas que realicé consistieron en determinar qué expresión matemática era la más indicada para obtener una curva de evolución del módulo del vector atractivo. Hice uso de la herramienta GeoGebra[^1].

[^1]: https://www.geogebra.org/classic?lang=es

Al principio opté por una función logaritmica, pero su evolución después de los cálculos restantes, en el proceso de creación del vector, tenía un comportamiento exponencial creciente. Este no era de utilidad.

Más tarde, probé a transformar una expresión exponencial de **e** en una curba descendente. El resultado fue el esperado y es el que actualmente se encuentra en la solución.

Después de implementar el resto de vectores y tras un gran número de pruebas, decidí implementar un filtro de paso bajo para suavizar las velocidades lineales y angulares. El resultado fue muy positivo al principio, pero tras cambiar el método de obtención del vector atractivo, dejó de funcionar como antes.

Luego, empecé a modificar los valores de ælpha y ẞeta, pero no llegaba a encontrar una solución estable. El vector atractivo era muy cambiante y producía comportamientos inesperados.

Finalmente conseguí que el coche se moviera por el circuito, esquivando los primeros obstáculos. Pero actualmente, tras una de las últimas curvas, el coche se detiene debido a un mínimo tras un cambio brusco en la posición del target.

### Video del funcionamiento[^2]

[^2]: https://urjc-my.sharepoint.com/:v:/g/personal/e_martint_2022_alumnos_urjc_es/ERsUYvq6aqtDi-2qGU6JUqUBz2WiHx-MeGGQmXsDIgcPug?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=bYhh0U

## Conclusión

El método de control por Virtual Force Field (VFF), es muy útil para generar un sistema reactivo a eventos locales.
La implementación no es complicada, pero requiere de una parametrización y configuración extensa.
