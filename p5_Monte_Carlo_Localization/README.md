# P5 Montecarlo localization

## Introducción

La práctica consiste en la creación de un algoritmo de localización basado en el uso de partículas y un sensor láser en el robot. Se empleará el uso de probabilidades usando la técnica de Montecarlo

La aproximación a este problema se realizará mediante 3 partes:

1. Modelo probabilístico de observación: Esta fase se encarga de inicializar las partículas de manera aleatoria por el espacio de trabajo. Con esta distribución simulamos de manera hipotética la posición del robot	

2. Modelo de movimiento: Este es el encargado de desplazar las partículas acompasadas por el movimiento del robot. Es importante destacar que el movimiento nunca es perfecto y por ello se añade ruido al desplazamiento de las partículas

3. Regeneración de partículas (Resampling): Tras un desplazamiento, recogemos las medidas de los láseres reales y simulamos las mediciones por cada partícula creada. Estas medidas darán como resultado distancias, las cuales emplearemos para hallar las diferencias y asignar pesos a las partículas en función de las similitudes de estas medidas.

![Ejemplo_inicialización](https://ars.els-cdn.com/content/image/1-s2.0-S0378475414000548-gr1.jpg)

## Experiencias y resultados

### Modelo probabilístico

Para conseguir que las partículas quedaran distribuidas de manera uniforme sobre la zona de trabajo, recordé que puedo dividir el espacio en cuadrantes y ubicar una partícula aleatoriamente dentro de estos.

En función del numero de partículas, se generarían mas o menos cuadrantes distribuidos uniformemente por el área de trabajo. 

**Formula empleada**

$square_aprox = \sqrt{N_PARTICLES}$

$square_height = \frac{Space_height}{square_aprox}$
$square_width = \frac{Space_width}{square_aprox}$

### Modelo de movimiento

El calculo se realiza sobre la distancia teórica del robot, recogida desde getOdom(). Se debe calcular la diferencia de posiciones inicial del robot y la posterior al movimiento, añadiendo consigo un ruido a la nueva coordenada calculada.

El resultado permite desplazar las partículas de manera similar al movimiento real del robot.

**Formula empleada**

$ Desplazamiento = Distancia_inicial(x,y,\theta) – Distancia_final(x,y,\theta)$

### Regeneración de partículas

Tras el desplazamiento de las partículas, se procesaran individualmente cada una de ellas con los datos recibidos por el láser. En esta ocasión, para poder procesar todas las partículas se ha empleado multiprocessing. La formula empleada es la diferencia entre las distintas medidas del laser, frente a las estimaciones de la partícula a procesa.

**Formula empleada**

$ P(X,Y,\theta) = e^{-distancia(obs_laser - obs_particula)}

### Observaciones



### Video del funcionamiento
