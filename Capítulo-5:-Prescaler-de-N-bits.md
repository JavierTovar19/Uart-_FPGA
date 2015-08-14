![Image header](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T05-prescaler/images/prescaler-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T05-prescaler)

## Introducción

Los _prescalers_ sirven para **ralentizar las señales de reloj**. Por la entrada entra una señal de reloj de frecuencia f y por la salida se obtiene una de frecuencia menor.  En este tutorial haremos un **prescaler de N bits** para hacer parpadear un led a diferentes frecuencias

Para un prescaler de N bits, las **fórmulas** que relacionan las frecuencias y periodos de entrada con los de salida son:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T05-prescaler/images/prescaler-2.png)

## Entendiendo el prescaler de 2 bits

Antes de implementar un prescaler de N bits, vamos a entender cómo funciona uno de 2 bits

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T05-prescaler/images/prescaler-3.png)

Internamente está constituido por un contador de 2 bits, cuyas salidas son d0 y d1. La de mayor peso es la que se saca como señal de salida. Este contador se incrementa en cada flanco de subida de clk, que tiene un periodo T. Si observamos las señales de salida de sus dos bits (d0 y d1):

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-4.png)

vemos que el periodo de la señal d0 es 2 veces T, y la de la señal d1 es 4 veces T. En general, si tenemos un **prescaler de N bits**, el bit de mayor peso tendrá un periodo de **Tp = 2^N * T**. Si tomamos los inversos para pasarlo a frecuancias, tenemos que **fp = f / 2^N**. **La frecuencia del bit de mayor peso del prescaler es la frecuencia de su reloj dividida entre 2 elevado al número de bits del prescaler**.

## Periodo del contador

El contador de 26 bits del tutorial lo podemos ver como un contador de 4 bits cuyos bits se sacan por los leds y cuya señal de reloj proviene de un prescaler de 22 bits, con un reloj de entrada de 12Mhz

![Imagen 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-6.png)

Aplicando la fórmula anterior, la **frecuencia de reloj del contador de 4 bits es **fp = 12Mhz / 2^22 = **2.86 Hz**. Se incrementa cada Tp = 1 / fp = 1 / 2.86 =  **0.35 segundos**.

La frecuencia del contador la podemos variar cambiando el prescaler. En la siguiente tabla se han recopilado los datos de las frecuencias y periodos de prescalers según su número de bits:

| Bits  | Frecuencia  |  Periodo 
|-------|-------------|---------
|  1    |  6 MHz      |  0.167 usec
|  2    |  3 MHz      |  0.333 usec
|  3    |  1.5 Mhz    |  0.666 usec
|  4    |  750 Khz    |  1.333 usec
|  5    |  375 Khz    |  2.666 usec
|  6    |  187.5 Khz  |  5.333 usec
|  7    |  93.75 KHz  |  10.666 usec
|  8    |  46.875 Khz |  21.333 usec
|  9    |  23437.5 Hz |  42.666 usec
| 10    |  11718.7 Hz |  85.333 usec
| 11    |  5859.37 Hz |  170.66 usec
| 12    |  2929.68 Hz |  341.33 usec
| 13    |  1464.84 Hz |  682.66 usec
| 14    |  732.42 Hz  |  1.365 ms
| 15    |  366.21 Hz  |  2.73 ms
| 16    |  183.1 Hz   |  5.46 ms
| 17    |  92.552 Hz  |  10.92 ms
| 18    |  45.776 Hz  |  21.84 ms
| 19    |  22.888 Hz  |  43.69 ms
| 20    |  11.444 Hz  |  87.38 ms
| 21    |  5.722 Hz   |  174.76 ms
| 22    |  2.861 Hz   |  349.52 ms