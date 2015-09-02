![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/8bf3e940aebcf444a33c0345be1fcec0de6c5f32/tutorial/T19-secnotas/images/secnotas-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T19-secnotas)

## Introducción
Hasta ahora hemos generado sonidos en canales diferentes, de manera independiente.  Implementaremos un **secuenciador** que nos permita tocar **4 notas** que sonarán **consecutivamente** en el altavoz. Reproduciremos la secuencia: DO, RE, MI, (silencio) y se volverá al comienzo. Esto nos abre la puerta a la creación de melodías sencillas :-)

## secnotas.v: Descripción del hardware
Una manera de lograr la secuenciación de las notas es utilizar un **multiplexor**, con el que se seleccionará en cada momento el canal a escuchar

En este ejemplo, tocaremos las notas **DO**, **RE**, **MI**, que se introducirán por las entradas 0, 1 y 2 respectivamente del multiplexor. Por la cuarta pondremos un "0" para que haya **silencio**.

La entrada de selección la conectamos a un **contador de 2 bits**, de forma que seleccione alternativamente los canales 0, 1, 2 y 3. Este contador tendrá una entrada de reloj que determinará **la duración de la nota**, que establecemos en **250ms**.

El esquema del circuito se muestra en esta figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T19-secnotas/images/secnotas-2.png)

La descripción en verilog asociada a este esquema es:

``` verilog 
//-- Fichero: secnotas.v

//-- Incluir las constantes del modulo del divisor
`include "divider.vh"

//-- Parameteros:
//-- clk: Reloj de entrada de la placa iCEstick
//-- ch_out: Canal de salida
module secnotas(input wire clk, output reg ch_out);

//-- Parametros: notas a tocar
//-- Se define como parametro para poder modificarlas desde el testbench
//-- para hacer pruebas
parameter N0 = `DO;
parameter N1 = `RE;
parameter N2 = `MI;
parameter DUR = `T_250ms;

//-- Cables de salida de los canales
wire ch0, ch1, ch2;

//-- Selección del canal del multiplexor
reg [1:0] sel = 0;

//-- Reloj con la duracion de la nota
wire clk_dur;

//-- Canal 0
divider #(N0)
  CH0 (
    .clk_in(clk),
    .clk_out(ch0)
  );

//-- Canal 1
divider #(N1)
  CH1 (
    .clk_in(clk),
    .clk_out(ch1)
  );

//-- canal 2
divider #(N2)
  CH2 (
    .clk_in(clk),
    .clk_out(ch2)
  );

//-- Multiplexor de seleccion del canal de salida
always @*
  case (sel)
     0 : ch_out <= ch0;
     1 : ch_out <= ch1;
     2 : ch_out <= ch2;
     3 : ch_out <= 0;
     default : ch_out <= 0;
  endcase

//-- Contador para seleccion de nota
always @(posedge clk_dur)
  sel <= sel + 1;

//-- Divisor para marcar la duración de cada nota
divider #(DUR)
  TIMER0 (
    .clk_in(clk),
    .clk_out(clk_dur)
  );

endmodule
```
En el fichero **divider.vh** se han añadido constantes para indicar la **duración de las notas**. Por defecto se han puesto un tiempo de **250ms**:

```verilog
`define T_1s     12000000
`define T_500ms  6000000
`define T_250ms  3000000
```
