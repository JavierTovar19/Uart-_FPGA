![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-ex1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T29-tristate)

# Introducción
Las **puertas triestado** (También conocidas como [Buffers triestado](https://es.wikipedia.org/wiki/Buffer_triestado)) nos permiten **desconetar las salidas de nuestros circuitos**, impidiendo que vuelquen su tensión (0 ó 1) en el cable al que se conectan.

Las describiremos en Verilog, las simularemos y documentaremos las **LIMITACIONES QUE PRESENTAN** al ser **sintetizadas** con las **herramientas libres** del proyecto icestorm. El soporte de Yosys, Arachne y Icestorm para puertas triestado es muy limitado todavía al día de hoy (Nov - 2015)

# Puertas triestado

Describiremos cómo funcionan y cómo se modelan en verilog

## Funcionamiento
Las puertas triestado se comportan como **interruptores** que conectan la entrada con la salida cuando su señal de habilitación está activada. Su símbolo es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-1.png)

Tiene 3 puertos de 1 bit:
* **entrada**:  Señal de entrada
* **salida**: Señal de salida
* **enable**: Señal de habilitación

Cuando la señal de habilitacion está a **1**, la **entrada está conectada directamente a la salida**. Sin embargo, cuando está a **0**, **la salida queda desconectada**. Se considera que está en un tercer estado, denominado **alta impedancia**, y se representa con la letra **Z**.  Así, la salida de una puerta triestado puede encontrarse en los **estados 0**, **1** ó **Z**

## Descripción en Verilog

Una puerta triestado se modela de la siguiente forma:

```verilog
assign salida = (enable) ? entrada : 1'bz;
```

El sintetizador reconoce que es una puerta triestado porque se **le asigna el valor z** cuando enable es 0

# Ejemplo 1: conexión de un led

El primer ejemplo es un hola mundo: conexión de un led a través de una puerta triestado

## Diagrama de bloques

Se coloca un **biestable** con un 1 a su entrada, para que se cargue en el **primer flanco de subida**. Su salida se conecta al led a través de una **puerta triestado**. La señal de enable se conecta a un **divisor de frecuencia** que genera un **pulso cuadrado de 1 segundo**, de manera que el led deberá estar medio segundo encendido y medio segundo apagado

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-ex1.png)

## Descripción en verilog

El código verilog es el siguiente:

```verilog
`default_nettype none

`include "divider.vh"

module tristate1  (
         input wire clk,      //-- Entrada de reloj
         output wire led0);   //-- Led a controlar

//-- Parametro: periodo de parpadeo
parameter DELAY = `T_1s;

//-- Cable con la señal de tiempo
wire clk_delay;

//-- Biestable que está siempre a 1
reg reg1;
always @(posedge clk)
  reg1 <= 1'b1;

//-- Conectar el biestable al led a traves de la puerta tri-estado
assign led0 = (clk_delay) ? reg1 : 1'bz;

//-- Divisor para temporizacion
divider #(DELAY)
  CH0 (
    .clk_in(clk),
    .clk_out(clk_delay)
  );

endmodule
```

## Simulación

El banco de pruebas sólo instancia el componente tristate1 y genera el reloj. La pausa se pone en 4 ciclos para poderla apreciar mejor en la simulación. El código del banco de pruebas es:

```verilog
module tristate1_tb();

//-- Pausa pequeña: divisor
localparam DELAY = 4;

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Led a controlar
wire led0;

//-- Instanciar el componente
tristate1 #(.DELAY(DELAY))
  dut(
    .clk(clk),
    .led0(led0)
  );

//-- Generador de reloj. Periodo 2 unidades
always #1 clk = ~clk;

//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("tristate1_tb.vcd");
  $dumpvars(0, tristate1_tb);

  # 100 $display("FIN de la simulacion");
  $finish;
end

endmodule
```
La simulación se realiza con el siguiente comando:

    $ make sim

Y la simulación en gtkwave es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-ex1-sim.png)

Se puede ver cómo la señal del led permanece a 1 durante 2 ciclos de reloj, y luego a alta impedancia (colo amarillo) durante los otros 2 ciclos

## Síntesis y pruebas

# Ejemplo 2: Conexión de un led a un bus de 1 bit

# Limitaciones en síntesis

# Alternativas a las puertas triestado

# Ejercicios

# Conclusiones
TODO