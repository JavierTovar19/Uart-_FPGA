![Image header](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T05-prescaler/images/prescaler-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T05-prescaler)

## Introducción

Los _prescalers_ sirven para **ralentizar las señales de reloj**. Por la entrada entra una señal de reloj de frecuencia f y por la salida se obtiene una de frecuencia menor.  En este tutorial haremos un **prescaler de N bits** para hacer parpadear un led a diferentes frecuencias

Para un prescaler de N bits, las **fórmulas** que relacionan las frecuencias y periodos de entrada con los de salida son:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T05-prescaler/images/prescaler-2.png)

## Entendiendo el prescaler de 2 bits

Antes de implementar un prescaler de N bits, vamos a entender cómo funciona uno de 2 bits

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T05-prescaler/images/prescaler-3.png)

Internamente está constituido por un **contador de 2 bits**, cuyas salidas son d0 y d1. La de mayor peso es la que se saca como señal de salida. Este contador se incrementa en cada flanco de subida de clk, que tiene un periodo T. Si observamos las señales de salida de sus dos bits (d0 y d1):

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T05-prescaler/images/prescaler-4.png)

vemos que el periodo de la señal d0 es 2 veces T, y la de la señal d1 es 4 veces T. Es decir, que cada nuevo bit duplica el periodo de la señal anterior. Siguiendo la fórmula general, el periodo de este prescaler de 2 bits es: Tout = 2^2 * T = 4 * T (y gráficamente comprobamos que es así).

## Frecuencias y periodos del prescaler

La **frecuencia de entrada** al prescaler en la placa iCEStick es de **12Mhz**. Aplicando la fórmula anterior, obtenemos esta **tabla con periodos y frecuencias** para prescalers con diferente número de bits (N). Esto nos da una idea de qué valor de N elegir para hacer parpadear el led

| Bits (N)  | Frecuencia  |  Periodo | Visible
|-------|-------------|---------| No
|  1    |  6 MHz      |  0.167 usec | No
|  2    |  3 MHz      |  0.333 usec | No
|  3    |  1.5 Mhz    |  0.666 usec | No
|  4    |  750 Khz    |  1.333 usec | No
|  5    |  375 Khz    |  2.666 usec | No
|  6    |  187.5 Khz  |  5.333 usec | No
|  7    |  93.75 KHz  |  10.666 usec | No
|  8    |  46.875 Khz |  21.333 usec | No
|  9    |  23437.5 Hz |  42.666 usec | No
| 10    |  11718.7 Hz |  85.333 usec | No
| 11    |  5859.37 Hz |  170.66 usec | No
| 12    |  2929.68 Hz |  341.33 usec | No
| 13    |  1464.84 Hz |  682.66 usec | No
| 14    |  732.42 Hz  |  1.365 ms    | No
| 15    |  366.21 Hz  |  2.73 ms | No
| 16    |  183.1 Hz   |  5.46 ms | No
| 17    |  92.552 Hz  |  10.92 ms | No
| 18    |  45.776 Hz  |  21.84 ms | No
| 19    |  22.888 Hz  |  43.69 ms | Si
| 20    |  11.444 Hz  |  87.38 ms | Si
| 21    |  5.722 Hz   |  174.76 ms | Si
| 22    |  2.861 Hz   |  349.52 ms | Si

El ojo humano tiene una frecuencia de refresco de unos 25Hz. Esto significa que frecuencias superiores no se aprecian. Si hacemos parpadear el led con una frecuencia superior, lo apreciaremos como si siempre estuviese encendido (no lo veremos parpadear)

Al usar el prescarler con el led, **a partir de 19 bits es cuando se puede apreciar el parpadeo**. Cuanto más bits, más lento parpadeará el led.

