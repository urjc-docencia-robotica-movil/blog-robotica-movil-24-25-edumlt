# P2 Follow line

## Introducción

Esta práctica consiste en la creación de un controlador PID, capaz de hacer que un coche de carreras siga la línea roja de un circuito de carreras.

Primero tendremos que ser capaces de detectar la línea roja que se encuentra sobre el circuito. A continuación realizamos los cálculos que componen la parte proporcional, integral y derivativa del controlador. Y por último, implementaremos una escala para regular la velocidad de giro máximo en ambos sentidos.

## Filtro de color

Para implementar esta parte, he preferido emplear un filtro de color HSV[^1]. Con él, puedo aislar de manera más precisa el color rojo de la carretera y conseguir una imagen limpia.

![Modelo HSV](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6c/Cono_de_la_coloraci%C3%B3n_HSV.png/250px-Cono_de_la_coloraci%C3%B3n_HSV.png)

Los valores para el color rojo se encontraban entre: 

  - Filtro bajo -> **H:120 S:100 V:100**
  - Filtro alto -> **H:150 S:255 V:255**

[^1]: https://es.wikipedia.org/wiki/Modelo_de_color_HSV.

## Calculo del error

Para estos cálculos, tome como referencia la imagen y el número de pixeles que la componen. Estas medidas las empleé para mover mi línea de cálculo sobre 15 pixeles por debajo de la mitad de la imagen. Con esta línea comprobé cuantos pixeles había entre el centro y el primer pixel de color rojo a cada lado del coche. Finalmente, invierto el valor del dato para que coincida con el sentido de giro que se realizará más adelante. 

**Formula empleada**
```
img[center_line + 15][center_point + i][2] > 0
```
El último campo pertenece al color en la imagen, y los dos anteriores a los pixeles en altura y anchura respectivamente.

El índice del _bucle for_ es el valor que empleo como error desde el centro de la imagen.

## PID

Parte más importante del ejercicio. Su implementación es simple, pero necesita de valores ajustables llamados **_KD KI KP_**.
Los cálculos que se deben realizar para el PID son los siguentes[^2]:

![Controlador PID](https://upload.wikimedia.org/wikipedia/en/1/11/PID-feedback-loop-v1.png)

Como era de esperar, no se pueden realizar estos cálculos sin librerías matemáticas de alto nivel. Por ello se ha implementado de la siguiente forma:

```
P = Kp*error
integral = integral + Ki*error
D = Kd*(error-error_prev)

W = P + integral + D
```
_Los valores KP KI KD se modifican dependiendo del modelo de coche empleado_
De esta manera conseguimos el control del error en pixeles; aún queda convertirlo a velocidad angular.

[^2]: https://es.wikipedia.org/wiki/Controlador_PID.

## Escalado de velocidades

Como el error está basado en distancia entre pixeles, la velocidad de giro tendrá que estar acotada entre unos valores máximos y mínimos, que permitan que el coche gire sin brusquedad.

La fórmula empleada coge como valor **máximo original** la mitad del ancho de la pantalla hacia la derecha y el **mínimo** hacia la izquierda. Los valores de velocidad varían en función del modelo empleado.
```
scaled_data = (MIN_VEL_W + ((vel - min_original) / (max_original - min_original)) * (MAX_VEL_W - MIN_VEL_W))
```

## Modelo Simple

Modelo en el que las velocidades angulares afectan al eje central del coche, haciendo que gire sobre si mismo.
La velocidad lineal del coche es absoluta y no guarda inercia cuando se decelera el coche.

### Primeras experiencias y resultados

Las primeras pruebas no fueron sencillas. Seguí las indicaciones **Trial and Error** de la documentación del ejercicio con los siguientes resultados:

  - El circuito se podía completar tan solo empleando el controlador *Proporcional*, pero con muchas oscilaciones.
  - Cuando se añadía la parte *Integral* el sistema no era capaz de soportar valores por encima del 0.01 debido a que el error está basado en pixeles.
  - La parte _Derivativa_ mejoró la respuesta y suavizó la salida del controlador.

Al ir cambiando de mapas, me di cuenta que había curbas casi de 90º en las que la línea se perdía en la imagen. Por este motivo implementó el sistema de recuperación, que se encuentra en uso en ambos modelos (video[^3]).

### Video del funcionamiento

Circuito Simple: [^4]

Circuito Montreal: [^5]

[^3]: https://urjc-my.sharepoint.com/:v:/g/personal/e_martint_2022_alumnos_urjc_es/EaO5f-uYlzRJporu-dv_NOUBqvAH38CiXoG3eaDXG7PjTw?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=8Y1PGv

[^4]: https://urjc-my.sharepoint.com/:v:/g/personal/e_martint_2022_alumnos_urjc_es/EYOJHrdtZ9lLtfOoqai7q2IBzfw1r3AZ0RBbvwvc1yM9Fw?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=FjjUVY

[^5]: https://urjc-my.sharepoint.com/:v:/g/personal/e_martint_2022_alumnos_urjc_es/EZkseKaT7w9OtWQl8UjW3WcB4_DpEJgtNPwKug1sX39ZjA?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=coTtsu

## Modelo Simple Ackerman 

Modelo en el que las velocidades angulares afectan al eje delantero del coche, haciendo que gire únicamente la parte delantera de este, dejando las ruedas traseras rígidas al movimiento. Si el coche dispone de suficiente velocidad lineal, este se moverá en función del ángulo de las ruedas delanteras.
La velocidad lineal del coche es relativa y guarda inercia. 

**IMPORTANTE:** *Para activar el modelo Ackerman, hay que cambiar la variable global **ACKERMAN_ON** por True; por defecto se encuentra en False.*

### Primeras experiencias y resultados

El modelo es complicado de manejar. La primera parte a tener en cuenta es la facilidad con la que las ruedas traseras derrapan y pierden el control. Si tenemos en cuenta esto, el control al error tiene que ser muy ligero.

Durante la prueba del circuito simple, he podido comprobar que el coche no puede soportar velocidades altas. Cuando se pone una velocidad de 50, el coche puede volcar sin apenas actuación del controlador.

En el resto de circuitos, hay tramos en los que el coche pierde velocidad sin ser cambiado en el código.

Por lo demás, el sistema de recuperación facilita que en ciertos tramos el coche pueda volver a encontrar la línea cuando gira lentamente en curvas cerradas.

### Video del funcionamiento

Circuito Ackerman: [^6]

[^6]: https://urjc-my.sharepoint.com/:v:/g/personal/e_martint_2022_alumnos_urjc_es/EchC776sz0RIu_404AEnlwoBDtn4ML0FLBw7xrd4ARXCaw

## Conclusión

El PID es una herramienta esencial en el control de procesos donde se requiera precisión en las respuestas. Aunque, presenta dificultades a la hora de encontrar los valores óptimos de K.
