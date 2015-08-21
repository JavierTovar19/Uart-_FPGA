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
Utilizaremos un **multiplexor de 4 a 1** para hacer un **secuenciador de 4 estados**, generando una secuencia que salga por los leds. Por las 4 entradas cableamos los valores de la secuencia. Por defecto serán: 0000, 1010, 1111 y 0101. La entrada de sección se conecta a un **contador de 2 bits** para que se vayan seleccionando secuencialmente las 4 entradas.  El reloj del contador está conectado a un **prescaler** para que vaya más lento

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-4.png)

El código Verilog es:

    //-- mux4.v
    module mux4(input wire clk, output reg [3:0] data);
    
    //-- Parametros del secuenciador:
    parameter NP = 23;         //-- Bits del prescaler
    parameter VAL0 = 4'b0000;  //-- Valor secuencia 0
    parameter VAL1 = 4'b1010;  //-- Valor secuencia 1
    parameter VAL2 = 4'b1111;  //-- Valor secuencia 2
    parameter VAL3 = 4'b0101;  //-- Valor secuencia 3
    
    //-- Cables para las 5 entradas del multiplexor
    wire [3:0] val0;
    wire [3:0] val1;
    wire [3:0] val2;
    wire [3:0] val3;
    wire [1:0] sel;  //-- Dos bits de selección
    
    //-- Contador de 2 bits
    reg [1:0] count = 0;
    wire clk_pres; //-- Reloj una vez pasado por prescaler
    
    //-- Por las entradas del mux cableamos los datos de entrada
    assign val0 = VAL0;
    assign val1 = VAL1;
    assign val2 = VAL2;
    assign val3 = VAL3;
    
    //-- Implementación del multiplexor de 4 a 1
    always @*
      case (sel)
         0 : data <= val0;
         1 : data <= val1;
         2 : data <= val2;
         3 : data <= val3;
         default : data <= 0;
      endcase
    
     //-- Contador de 2 bits para realizar la seleccion de la fuente de datos
     always @(posedge(clk_pres))
      count <= count + 1;
     
    //-- El contador esta conectado a la entrada sel del mux
    assign sel = count;
    
    //-- Presaler que controla el incremento del contador para la selección
    prescaler #(.N(NP))
      PRES (
        .clk_in(clk),
        .clk_out(clk_pres)
      );
    
    endmodule

## Síntesis en la FPGA

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T12-mux-4-1/images/mux4-1.png)

## Simulación

## Ejercicios propuestos

## Conclusiones
TODO