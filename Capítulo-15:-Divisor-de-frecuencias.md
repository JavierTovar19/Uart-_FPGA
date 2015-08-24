## Introducción

Con los **prescalers** conseguimos obtener señales de reloj de una frecuencia menor. Sin embargo, con ellos **no podemos conseguir la frecuencia exacta** que queramos. Si queremos por ejemplo implementar un contador que se incremente cada segundo, ¿cómo lo podríamos hacer?. Necesitamos los **divisores de frecuencia**.

Con estos divisores conseguimos **generar señales de frecuencias exactas**. Son muy importantes para aplicaciones de comunicaciones (uarts, spi, bus i2c...), refresco de pantallas gráficas, generación de señales PWM, cronómetros, temporizadores, generación de tonos audibles, etc.

En este capítulo ....

## El divisor de frecuencias

Dividir la frecuencia de una señal entre 2 es muy sencillo: colocamos un prescaler de 1 bit. En general, **para divir entre cualquier potencia de 2** (2, 4, 8, 16...2^N) nos basta con un **prescaler de N bits**. Para el resto de frecuencias necesitamos el divisor de frecuencias

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divisor-1.png)

Es un componente que tiene una señal de entrada (clk_in), con frecuencia fin y periodo Tin. Como salida tiene otra señal (clk_out) cuya frecuencia es la de la **entrada dividia entre M**. O si lo vemos con el **periodo**, el de la señal de salida es **M veces mayor que el de la entrada**

## "Hola mundo": Divisor entre 3

 Comenzaremos por el divisor más pequeño posible, el **"hola mundo"**, un **divisor entre 3**. Obendremos una señal con una frecuencia 3 veces menor. En el caso de probarlo en la placa iCEstick con el reloj de 12Mhz, obtendríamos una señal de 12/3 = 4Mhz. 

Las señales de entrada y salida son las siguientes:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divisor-2.png)

Vemos que clk_out tiene un periodo Tout 3 veces mayor que Tin (La tercera parte de su frecuencia). Aunque **el ciclo de trabajo es diferente**. Clk_in está el mismo tiempo a nivel alto que bajo, mientras que clk_out está dos tercios del periodo a nivel bajo y uno a nivel alto. Para temas de temporización, **el ciclo de trabajo es indiferente**. Lo importante es la frecuencia.

Un **divisor de frecuencia entre M** se implementa usando un **contador módulo M**, y tomando como **salida** el **bit de mayor peso**.

Para implementar este divisor hola mundo, hay que utilizar por tanto un **contador módulo 3**, como se muestra en el siguiente diagrama

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divisor-3.png)

## Contador módulo 3

Este componente es un contador que al cabo de 3 pulsos vuelve al comienzo, repitiendo la cuenta 0, 1, 2, 0, 1, 2, 0, 1, 2...

En general,  un **contador módulo M** se inicializa al cabo de M pulsos. Empieza en 0 y va contando 1, 2, 3, 4... hasta M-1. En el siguiente pulso pasa de nuevo a 0 y vuelve a empezar



