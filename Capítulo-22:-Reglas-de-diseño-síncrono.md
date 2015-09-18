![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/sync-corazon.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T22-syncrules)

## Introducción
En este capítulo se explican algunas **reglas de diseño síncrono** [1] para la **creación de circuitos digitales complejos**. Su aplicación nos evitará problemas según crece la complejidad de nuestros circuitos. Las aplicaremos para **mejorar el transmisor serie** que se esbozó en el capítulo anterior

## Sistemas síncronos
Los **sistemas digitales complejos son síncronos**: existe **una señal de reloj** (clk) que marca los tiempos en los que el resto de señales cambian. Es un **reloj "corazón"**, al ritmo de cuyos pulsos se transportan y modifican los bits. En la placa iCEstick, nuestro reloj es de **12Mhz**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/sync-corazon.png)

A la hora de diseñar circuitos digitales síncronos conviene tener **una serie de reglas en la cabeza**, que nos ahorrarán muchos problemas, sobre todo cuando se hacen diseños más complejos. Algunas de estas reglas **se han violado** en los ejemplos presentados hasta ahora. Se trata de ejemplos sencillos, que nos han permitido introducir conceptos y hacer pruebas rápidamente en nuestra FPGA.  Es el momento de aprender **reglas de diseño más potentes**, y aplicarlas

Pero antes vamos a conocer un poco más el **origen de los problemas**

## Problemas en los circuitos digitales

### La causa de los males: el retardo

Los problemas en los diseños están causados por el **retardo en las señales**, debido a la propagación y al tiempo de procesado de las puertas lógicas. En el mundo digital diseñamos la funcionalidad asumiendo que las señales digitales son perfectas, que cambian instantáneamente y todos los bits a la vez.

La realidad es diferente. Así por ejemplo, si estamos usando una **puerta NOT** y la simulamos, vemos el caso ideal, en el que la salida B es la negada de la A en todo momento.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/retardo-not.png)

En la realidad **la salida B está retrasada con respecto a la entrada** (además de que el cambio de 0 a 1 no es instantáneo). ¿Y por qué es un problema? Porque mirando el cronograma, vemos que hay un instante de tiempo donde **NO SE CUMPLE que B sea el negado de A**. Sucede que A=1 y B=1, lo que significa que **NO SE CUMPLEN LAS ECUACIONES BOOLEANAS de la lógica digital** en ese intervalo de tiempo (esto hay que tenerlo en la cabeza). Además, estos retrasos originan **pulsos espúreos transitorios** (glitches)

### Pulsos espúreos (Glitches)

**Cada señal** de un circuito digital **tiene un retraso diferente** que dependerá de la longitud de sus cables y de las puertas que atraviese. Así, dos señales de entrada llegarán a la misma puerta en tiempos diferentes. Esto provoca la **aparición de pulsos espúreos**, que desaparecen en el régimen permanente.

Vamos a ver su efecto en un circuito combinacional sencillo: una **puerta XOR** de dos entradas.  Las dos entradas están a 0 (y por tanto la salida permanece a 0). En un instante las señales de entrada A y B cambian a 1. Idealmente, lo que se obtiene por la salida es también 0, ya que 1 xor 1 es igual a 0.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/glitches-xor.png)

Sin embargo, en este ejemplo, debido a los retardos, la señal B ha llegado un poco después que la A. Esto hace que haya un momento en el que A = 1 y B = 0, por lo que la salida es C = 1.  Finalmente, cuando B = 1 la salida se pone a su valor estable: C = 0.

El resultado es que **ha aparecido un pulso** en una señal que **se suponía que debería estar siempre a 0**

Imaginemos lo que puede pasar si tenemos este circuito conectado a **la entrada de reloj de un contador**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/xor-contador.png)

Debido al pulso espúreo, **¡el contador hará una cuenta falsa!** ¡Contará cuando NO tiene que contar!. El diseñador, por más que mire y remire el diseño digital, verá que está todo correcto. Y al simularlo idealmente todo funcionará bien... pero al cargarlo en la FPGA: ¡zas! Contará mal. **Estos errores son muy difíciles de detectar**

Para evitar estos problemas es por lo que usamos **las reglas de diseño síncrono**

## Reglas de diseño síncrono
### Regla 1: Sensibilidad al mismo flanco: ¡Todos a una, fuenteovejuna!
Todos los elementos que lleven reloj, serán sensibles al mismo flanco. Bien al de subida o al de bajada, es indiferente, pero **todos sensibles al mismo**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-1-mismo-flanco.png)

### Regla 1: Un único reloj para gobernarlos a todos


## Mejoras en el transmisor serie
* Reloj del sistema (clk) en vez de clk_baud (reglas de diseño síncrono)

## Créditos
* [Corazón](https://commons.wikimedia.org/wiki/File:Coraz%C3%B3n.svg#/media/File:Coraz%C3%B3n.svg). Licensed under CC BY-SA 3.0 via Wikimedia Commons 
* [Puerta XOR](https://commons.wikimedia.org/wiki/File:Puerta_XOR.svg#/media/File:Puerta_XOR.svg) by Dnu72 - Own work. Licensed under CC BY-SA 3.0 via Wikimedia Commons

## Referencias
[1]: "Diseño síncrono de Circuitos Digitales". Miguel Angel Freire Rubio. ETSIST. UPM. Septiembre 2008 ([PDF](http://www.etsist.upm.es/uploaded/464/DS_Sep_2008.pdf))

