[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T11-mux-2-1)

## Introducción
Los **multiplexores** son circuitos combinacionales que nos permiten **seleccionar** entre varias fuentes de datos. En este capítulo utilizaremos un multiplexor de 2 a 1 para mostrar por los leds una secuencia de dos valores, que se mostrarán alternativamente

## Descripción del multiplexor 2 a 1

Un multiplexor 2 a 1 selecciona entre 2 fuentes de datos según el valor de su **entrada de selección** sel. Si sel es 0, se saca por la salida la fuente 0, si es 1 se saca la fuente 1

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/e9142aee8f70b7342c69b990159529ce68601487/tutorial/T11-mux-2-1/images/mux2-2.png)

Los describimos en Verilog usando la intrucción if ... else:

    always @(fuente0 or fuente1 or sel)
      if (sel == 0)
        dout <= fuente0;
      else
        dout <= fuente1;

Es muy importante que exista el **else**. Al tratarse de un circuito combinacional, **todos los casos deben estar cubiertos**. Si no es así, se pueden inferir registros.  También es muy importante colocar en la **lista de sensibilidad** TODAS las señales de entrada: fuente0, fuente1 y sel.

La lista de sensibilidad se puede escribir de forma abreviada con el argumento @*

    always @*

Esta lista incluye automáticamente todas las señales de entrada

## mux2.v: Secuenciador de 2 estados

Como ejemplo de uso, haremos un **secuenciador de 2 estados**: Un circuito que envía alternativamente dos datos de 4 bits a los leds. El esquema del circuito es el siguiente:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T11-mux-2-1/images/mux2-3.png)

Se utilizan 2 fuentes de datos fijas (están cableadas a valores fijos) que determinan el estado de los leds en cada momento. El multiplexor selecciona alternativamemente entre una y otra a través de una señal de reloj que pasa por un prescaler (para reducir la frecuencia y que podamos apreciar el movimiento de los leds).

El código verilog es el siguiente:

    //-- mux2.v
    module mux2(input wire clk, output reg [3:0] data);
    
    //-- Parametros del secuenciador:
    parameter NP = 22;         //-- Bits del prescaler
    parameter VAL0 = 4'b1010;  //-- Valor secuencia 0
    parameter VAL1 = 4'b0101;  //-- Valor secuencia 1
    
    //-- Cables de las 3 entradas del multiplexor
    wire [3:0] val0;
    wire [3:0] val1;
    wire sel;
    
    //-- Por las entradas del mux cableamos los datos de entrada
    assign val0 = VAL0;
    assign val1 = VAL1;
    
    //-- Implementación del multiplexor
    always @(sel or val0 or val1)
      if (sel==0)
        data <= val0;
      else
        data <= val1;
    
    //-- Presaler que controla la señal de selección del multiplexor
    prescaler #(.N(NP))
      PRES (
        .clk_in(clk),
        .clk_out(sel)
      );
    
    endmodule

La implementación del multiplexor es sencilla por lo que se incluye directamente en un proceso, en vez de definirla en un fichero separado y luego instanciarlo (diseño jerárquico).

## Síntesis en la FPGA

Para sintetizarlo en la fpga conectaremos las salidas data a los leds, y la entrada de reloj a la de la placa iCEstick

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T11-mux-2-1/images/mux2-1.png)

Sintetizamos con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 5 / 160
|BRAMs     | 0 / 16

Para cargar en la FPGA ejecutamos:

    $ sudo iceprog mux2.bin

En este **vídeo de Youtube** se puede ver la salida de los leds:

[![Click to see the youtube video](http://img.youtube.com/vi/4GnH5lqlTOU/0.jpg)](https://www.youtube.com/watch?v=4GnH5lqlTOU)

## Simulación
El banco de pruebas es uno básico, que instancia el componente mux2, con 1 bit para el prescaler (para que la simulación tarde menos). Tiene un proceso para la señal de reloj y uno para la inicialización de la simulación

![Imagen 3]()

La simulación se realiza con:

    $ make sim

El resultado en gtkwave es:

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T10-shif-register/images/T10-shift4-sim-1.png)

Vemos cómo inicialmente el registro tiene un valor indefinido (está en rojo, aunque en la síntesis tendrá un 0) y al llegar el **primer flanco de subida** se inicializa con el valor **0001**  (Gracias al circuito inicializador). En los siguientes flancos vemos cómo efectivamente el bit se desplaza hacia la izquierda: 0010, 0100, 1000  y por último vuelve al valor inicial: 0001, repitiéndose la secuencia hasta el final de la simulación.


## Ejercicios propuestos

## Conclusiones
TODO