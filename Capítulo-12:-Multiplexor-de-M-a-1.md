![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T12-mux-4-1)

## Introducción
Los multiplexores nos permiten **seleccionar entre M entradas** (siendo M una potencia de 2). Por la salida se saca sólo la fuente que está seleccionada. En el capítulo anterior vimos cómo implementar un multiplexor sencillo, de 2 a 1. En este lo generalizaremos a uno de M a 1, haciendo un ejemplo de 4 a 1. Lo usaremos para crear un secuenciador de 4 estados que saque una secuencia por los leds.

## Multiplexor de M a 1

**El multiplexor tiene M entradas de datos**, siendo M una potencia de 2  (2, 4, 8, 16...). Todas las entradas y la salida son de N bits. La entrada de selección tiene b bits, cumpliéndose que M = 2 ^ b  (2 elevado a b)

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-2.png)

## Multiplexor de 4 a 1

Como ejemplo vamos a implementar un multiplexor de 4 entradas (M = 4) de 4 bits (N = 4). La entrada de selección tiene 2 bits:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-3.png)

El código Verilog es muy intuitivo. Usaremos la instrucción case:

    always @*
      case (sel)
         0 : out <= fuente0;
         1 : out <= fuente1;
         2 : out <= fuente2;
         3 : out <= fuente3;
         default : data <= 0;
      endcase

Observamos que **están cubiertos todos los casos**. Pero aún así, se ha añadido el caso **default** (que no se cumplirá nunca). Esto es así para asegurarse que todos los casos se cubren (por si en el código por error se quita un caso, o se comenta). Esto nos garantiza que se implementa un **circuito combinacional**. Si no se cubren todos los casos, el sintetizador puede inferior algún registro.

En la lista de sensilidad habría que colocar las 5 entradas. Por ello se ha preferido usar @*

## mux4: secuenciador de 4 estados

## Síntesis en la FPGA

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-1.png)

## Simulación

## Ejercicios propuestos

## Conclusiones
TODO