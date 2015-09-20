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

Las reglas son:

### Regla 1: Un único reloj para gobernarlos a todos

**TODAS las entradas de reloj se deben conectar directamente a un único reloj**, común a todos. Esta regla se ha violado en casi todos los ejemplos de este tutorial hasta ahora, para que fuesen sencillos. A partir de ahora todos los ejemplos respetarán esta regla

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-2-unico-reloj.png)

Esta regla nos **prohibe conectar a la entrada del reloj cualquier otra cosa que NO sea el reloj del sistema**. De forma que no podemos conectar ningún circuito combinacional (por ej. la puerta xor de la explicación de los pulsos espúreos) ni tampoco secuencial (por ejemplo la salida de un contador para usarlo como divisor). En las siguientes secciones veremos qué añadir a los circuitos para que cumplan esta regla.

### Regla 2: Sensibilidad al mismo flanco: ¡Todos a una, fuenteovejuna!
Todos los elementos que lleven reloj, serán sensibles al mismo flanco. Bien al de subida o al de bajada, es indiferente, pero **todos sensibles al mismo**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-1-mismo-flanco.png)

### Regla 3: Antes de entrar a un circuito combinacional, pase por registro, por favor

**Todas las entradas de los circuitos combacionales** deben estar **conectadas a salidas de circuitos secuenciales**, sincronizadas con el reloj del sistema, o bien a **otros circuitos combinacionales que cumplan esta regla**. Es decir, que **cualquier entrada** de un circuito combinacional **tiene que ser capturada** antes por un registro. Las entradas que cumplen esta regla se denominan **entradas sincronizadas**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/c5baf5771fea985001de3c0976583c5f0f3b1e9f/tutorial/T22-syncrules/images/regla-3-entradas-sincronizadas.png)

### Regla 4: Antes de entrar a un circuito secuencial, pase por registro por favor
**Todas las entradas de los circuitos secuenciales** deben provenir de **las salidas de otros circuitos secuenciales** o bien de **combinacionales que cumplan la regla 3**. Es decir, que incluso para entrar en los circuitos secuenciales, es necesario que **las señales estén sincronizadas**.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-4-entradas-sec-sincronizadas.png)

Combinando la regla 3 y la 4, llegamos a la conclusión de que **cualquier entrada a nuestro circuito síncrono tiene que estar registrada**.

### Regla 5: Salidas de un combinacional: Sólo a entradas de otro combinacional, entradas síncronas o salidas del circuito síncrono

Esta regla nos indica **dónde ponemos conectar la salida de un circuito combinacional**. Bien a la **entrada de otro combinacional**, a **la entrada síncrona de un secuencial** o bien como **salida directa de nuestro circuito**. Queda terminantemente prohibido conectarlas a las entradas del propio combinacional (nada de realimentaciones directas) o a la entrada de la señal de reloj

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-5-salidas-combinacionales.png)

## Mejorando el transmisor serie

Rediseñaremos los ejemplos del transmisor serie para cumplir las reglas

### Ejemplo 1: txtest.v

Partimos del ejemplo baudtx, donde se envía el carácter "K" con el flanco de subida de la señal DTR

### Aplicando las reglas de diseño síncrono

El esquema del circuito es el siguiente:

(Dibujo)

Observamos varias violaciones de las reglas del diseño síncrono:

1. No hay un reloj global único (no se respeta la regla 1). La señal de reloj no entra directamente en el registro de desplazamiento. La señal proveniente del divisor NO puede entrar directamente por la entrada de reloj sino que debe haber una entrada nueva sincronizada. Para que el divisor funcione correctamente, el ancho del pulso de salida debe ser de 1 periodo de longitud.

2. La entrada load NO está registrada (se viola la regla 3)

3. La salida TX proviene de un circuito combinacional y se puede sacar directamente sin violar ninguna regla. Sin embargo al estar conectada a un bus asíncrono, cualquier pulso espúreo enviado será interpretado como una dato y provocará un error en las comunicaciones. Por ello es más seguro registrar esta salida.

### Baudgen.v: Modificación del divisor

### txtest.v: Modificación del transmisor

## Créditos
* [Corazón](https://commons.wikimedia.org/wiki/File:Coraz%C3%B3n.svg#/media/File:Coraz%C3%B3n.svg). Licensed under CC BY-SA 3.0 via Wikimedia Commons 
* [Puerta XOR](https://commons.wikimedia.org/wiki/File:Puerta_XOR.svg#/media/File:Puerta_XOR.svg) by Dnu72 - Own work. Licensed under CC BY-SA 3.0 via Wikimedia Commons
* [Unico Anello](https://commons.wikimedia.org/wiki/File:Unico_Anello.png#/media/File:Unico_Anello.png) by Xander - own work, (not derivative from the movies). Licensed under Public Domain via Commons

## Referencias
[1]: "Diseño síncrono de Circuitos Digitales". Miguel Angel Freire Rubio. ETSIST. UPM. Septiembre 2008 ([PDF](http://www.etsist.upm.es/uploaded/464/DS_Sep_2008.pdf))

