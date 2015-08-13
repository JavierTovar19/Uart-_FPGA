<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/T04-counter-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T04-counter)

## Introducción
Modelaremos nuestro **primer circuito secuencial**: un contador conectado a los LEDs. Los circuitos secuenciales, a diferencia de los combinacionales, **almacenan información**. El contador almacena un número que se incrementa con cada tic del reloj.

Esta es la pinta de nuestro componente. Se actualiza en cada flanco de subida del reloj, y su **salida data** es de **26 bits**.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-1.png)

**La señal de reloj** de la placa iCEstick es de **12Mhz**. Si hacemos un contador de sólo 4 bits y le conectamos a su entrada clk esta señal de 12Mhz, el resultado será que se incrementará tan rápido que siempre veremos los leds encendidos. Por ello utilizaremos un contador de 26 bits y usaremos los 4 más significativos para mostrarlos en los leds. 

## Descripción del hardware




Borrador
El bit de mayor peso (25) parpadea con un frecuencia de f = 1 / T   T0 = 2 * 2^0 T,  T1 = 2 * 2 * T, Tb = 2 * 2^b * T = 2^(b+1) * T.    T25 = 2^26 * T

## Síntesis en la FPGA


[![Click to see the youtube video](http://img.youtube.com/vi/x9_OwUAtts4/0.jpg)](https://www.youtube.com/watch?v=x9_OwUAtts4)

## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO



