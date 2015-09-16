![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/sync-corazon.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T22-syncrules)

## Introducción
En este capítulo se explican algunas **reglas de diseño síncrono** [1] para la **creación de circuitos digitales complejos**. Su aplicación nos evitará problemas según crece la complejidad de nuestros circuitos. Las aplicaremos para **mejorar el transmisor serie** que se esbozó en el capítulo anterior

## Sistemas síncronos
Los **sistemas digitales complejos son síncronos**: existe **una señal de reloj** (clk) que marca los tiempos en los que el resto de señales cambian. Es un **reloj "corazón"**, al ritmo de cuyos pulsos se transportan y modifican los bits. En la placa iCEstick, nuestro reloj es de **12Mhz**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/sync-corazon.png)

A la hora de diseñar circuitos digitales síncronos conviene tener **una serie de reglas en la cabeza**, que nos ahorrarán muchos problemas, sobre todo cuando se hacen diseños más complejos. Algunas de estas reglas **se han violado** en los ejemplos presentados hasta ahora. Se trata de ejemplos sencillos, que nos han permitido introducir conceptos y hacer pruebas rápidamente en nuestra FPGA.  Es el momento de aprender **reglas de diseño más potentes**, y aplicarlas

Pero antes vamos a conocer un poco más el **origen de los problemas**

## La causa de los males: el retardo

Los problemas en los diseños están causados por el **retardo en las señales**, debido a la propagación y al tiempo de procesado de las puertas lógicas. En el mundo digital diseñamos la funcionalidad asumiendo que las señales digitales son perfectas, que cambian instantáneamente y todos los bits a la vez.

La realidad es diferente. Así por ejemplo, si estamos usando una **puerta NOT** y la simulamos, vemos el caso ideal, en el que la salida B es la negada de la A en todo momento.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/retardo-not.png)

En la realidad la salida B está retrasada con respecto a la entrada (además de que el cambio de 0 a 1 no es instantáneo).


## Reglas de diseño síncrono
### Regla 1: Un único reloj para gobernarlos a todos

## Mejoras en el transmisor serie
* Reloj del sistema (clk) en vez de clk_baud (reglas de diseño síncrono)

## Créditos
* [Corazón](https://commons.wikimedia.org/wiki/File:Coraz%C3%B3n.svg#/media/File:Coraz%C3%B3n.svg). Licensed under CC BY-SA 3.0 via Wikimedia Commons 

## Referencias
[1]: "Diseño síncrono de Circuitos Digitales". Miguel Angel Freire Rubio. ETSIST. UPM. Septiembre 2008 ([PDF](http://www.etsist.upm.es/uploaded/464/DS_Sep_2008.pdf))

