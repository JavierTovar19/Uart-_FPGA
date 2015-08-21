![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T12-mux-4-1)

## Introducción
Los multiplexores nos permiten **seleccionar entre M entradas** (siendo M una potencia de 2). Por la salida se saca sólo la fuente que está seleccionada. En el capítulo anterior vimos cómo implementar un multiplexor sencillo, de 2 a 1. En este lo generalizaremos a uno de M a 1, haciendo un ejemplo de 4 a 1. Lo usaremos para crear un secuenciador de 4 estados que saque una secuencia por los leds.

## Multiplexor de M a 1

**El multiplexor tiene M entradas de datos**, siendo M una potencia de 2  (2, 4, 8, 16...). Todas las entradas y la salida son de N bits. La entrada de selección tiene b bits, cumpliéndose que M = 2 ^ b  (2 elevado a b)

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-2.png)

## Multiplexor de 4 a 1

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-4.png)

## mux4: secuenciador de 4 estados

## Síntesis en la FPGA

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-1.png)

## Simulación

## Ejercicios propuestos

## Conclusiones
TODO