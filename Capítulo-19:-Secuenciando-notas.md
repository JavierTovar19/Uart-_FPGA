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
parameter N0 = `DO_4;
parameter N1 = `RE_4;
parameter N2 = `MI_4;
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
En el fichero **divider.vh** se han añadido constantes para indicar la **duración de las notas**

```verilog
`define T_1s     12000000
`define T_500ms  6000000
`define T_250ms  3000000
```
En el ejemplo se usa una duración de **250ms** 

## Síntesis

Para sintetizarlo conectamos a la entrada de reloj la señal de 12Mhz de la IceStick y la de salida al pin 44, donde colocaremos el zumbador al hacer las pruebas

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/8bf3e940aebcf444a33c0345be1fcec0de6c5f32/tutorial/T19-secnotas/images/secnotas-1.png)

Para sintetizar ejecutamos el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 6 / 96
|PLBs      | 26 / 160
|BRAMs     | 0 / 16

## Pruebas

Lo cargamos en la FPGA con el comando:

    $ sudo iceprog secnotas.bin

Alimentamos el zumbador desde los pines de 3.3v y gnd de la IceStick. El esquema es el siguiente: 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T19-secnotas/images/secnotas-3.png)

En esta foto se muestra el zumbador conectado a la iCEstick, listo para hacer las pruebas:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T19-secnotas/images/secnotas-4.png)

En este **vídeo de youtube** se muestra el ejemplo en acción, tocando las tres notas:

[![Click to see the youtube video](http://img.youtube.com/vi/iSH8HCGW-l0/0.jpg)](https://www.youtube.com/watch?v=iSH8HCGW-l0)

## Simulación
No se simula la generación de las notas reales porque llevaría muchísimo tiempo. En vez de eso se comprueba que por el canal de salida salen alternativamente las señales de entrada al multiplexor

```verilog
//-- Fichero: secnotas_tb.v
module secnotas_tb();

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Salidas de los canales
wire ch_out;

//-- Instanciar el componente y establecer el valor del divisor
//-- Se pone un valor bajo para simular (de lo contrario tardaria mucho)
secnotas #(.N0(2), .N1(3), .N2(4), .DUR(10))
  dut(
    .clk(clk),
    .ch_out(ch_out)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;

//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("secnotas_tb.vcd");
  $dumpvars(0, secnotas_tb);

  # 200 $display("FIN de la simulacion");
  $finish;
end

endmodule
```
Para realizar la simulación se ejecuta:

    $ make sim

El resultado es:

![]()


## Ejercicios propuestos

## Conclusiones
TODO
