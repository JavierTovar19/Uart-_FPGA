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

Se puede ver cómo la señal del led permanece a 1 durante 2 ciclos de reloj, y luego a alta impedancia (color amarillo) durante los otros 2 ciclos

## Síntesis y pruebas

La síntesis se realiza con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 10 / 160
|BRAMs     | 0 / 16

El diseño se carga con:

    $ sudo iceprog tristate1.bin

Al ejecutarse se verá cómo parpadea el led a la frecuencia de 1 Hz

# Ejemplo 2: Conexión de un led a un bus de 1 bit

En este ejemplo conectaremos la salida de **tres registros de 1 bit** a un **bus de 1 bit**, mediante puertas triestado

## Diagrama de bloques

El **bus de un bit** está conectado a un led de salida para poder visualizar lo que ocurre. Los tres biestables están conectados al bus a través de **puertas triestado**. Se inicializan con los valores iniciales 1, 0 y 1 respectivamente.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-ex2.png)

Las puertas triestado están habilitadas por los bits 0, 1 y 2 de un **registro de desplazamiento**, que inicialmente tiene el valor binario 0001, y va tomando los valores 0010, 0100 y 1000 por cada pulso de la señal clk_delay (de 1 segundo). De esta forma, primero se vuelca al bus el registro 0, luego el 1, luego el 2 y después ninguno. Al cabo de 4 ciclos de **clk_delay** se vuelve a comenzar

## Descripción en verilog

```verilog
`default_nettype none

`include "divider.vh"

module tristate2  (
         input wire clk,      //-- Entrada de reloj
         output wire led0);   //-- Led a controlar

//-- Parametro: periodo de parpadeo
parameter DELAY = `T_1s;

//-- Cable con la señal de tiempo
wire clk_delay;

//-- Bus de 1 bit
wire bus;

//-- Señal de reset
reg rstn = 0;

//-- Registro de desplazamiento
reg [3:0] sreg;

//-- Definir los 3 bistables a conectar al bus, cada uno con un
//-- valor inicial cualquiera
reg reg0;
always @(posedge clk)
  reg0 <= 1'b1;

reg reg1;
always @(posedge clk)
  reg1 <= 1'b0;

reg reg2;
always @(posedge clk)
  reg2 <= 1'b1;

//-- Conectar los biestables al bus, a traves de puertas triestado
assign bus = (sreg[0]) ? reg0 : 1'bz;
assign bus = (sreg[1]) ? reg1 : 1'bz;
assign bus = (sreg[2]) ? reg2 : 1'bz;

//-- Conectar el bus al led
assign led0 = bus;

//-- Registros de desplazamiento
always @(posedge clk)
  if (!rstn)
    sreg <= 4'b0001;
  else if (clk_delay)
    sreg <= {sreg[2:0],sreg[3]};

//-- Inicializador
always @(posedge clk)
  rstn <= 1'b1;

//-- Divisor para temporizacion
dividerp1 #(DELAY)
  CH0 (
    .clk(clk),
    .clk_out(clk_delay)
  );

endmodule
``` 

Lo importante del ejemplo es la conexión al mismo bus de los tres registros mediante las asignaciones:

```verilog
assign bus = (sreg[0]) ? reg0 : 1'bz;
assign bus = (sreg[1]) ? reg1 : 1'bz;
assign bus = (sreg[2]) ? reg2 : 1'bz;
```

## Simulación

El banco de pruebas es similar al del ejemplo anterior.

Para simular ejecutamos el comando:

    $ make sim2

En gtkwave obtenemos lo siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-ex2-sim.png)

Observamos cómo primero se vuelca el registro 0, que saca un 1 por el led. Luego el registro 1, que vuelca un 0, después el 2, que vuelve a volcar un 1 y finalmente todos desconectados, visualizándose la línea amarilla que indica **alta impedancia**.  El ciclo se repite indefinidamente

## Síntesis y pruebas

La síntesis se realiza con el comando:

    $ make sint2

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 13 / 160
|BRAMs     | 0 / 16

El diseño se carga con:

    $ sudo iceprog tristate2.bin

Al ejecutarse se verá cómo parpadea el led a la frecuencia de 1 Hz

# Limitaciones en síntesis

La versión actual del **sintetizador** usado en el proyecto icestorm (Yosys) tiene un **soporte para puertas triestado MUY LIMITADO**.  Por ello, de momento **conviene NO USARLAS EN NUESTROS DISEÑOS**

Mostraremos una **serie de ejemplos** que sí **funcionan en simulación**, pero que al sintetizarse con las herramientas libres **obtenemos errores**. Se incluyen aquí como documentación, para poder comprobar en las versiones futuras y si ya está solucionado

## Versión de las herramientas
```
$ yosys -V
Yosys 0.5+385 (git sha1 ddf3e2d, clang 3.6.0-2ubuntu1 -fPIC -Os)

$ arachne-pnr -v
arachne-pnr 0.1+134+0 (git sha1 788cf79, g++ 4.9.2-10ubuntu13 -O2)
```

## Error1.v: Conexión de un registro de 2 bits

En este ejemplo conectaremos un registro de 2 bits a dos leds mediante una puerta triestado de 2 bits. El diagrama es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/error1-1.png)

La descripción en verilog:

```verilog
`default_nettype none


module error1  (
         input wire clk,            //-- Entrada de reloj
         output wire [1:0] leds);   //-- Leds a controlar

wire ena = 1'b1;

//-- Registro de 2 bits
reg [1:0] reg1;

//-- Cargar el registro con valor inicial
always @(posedge clk)
  reg1 <= 2'b11;

//-- Conectar el registro a 2 leds
assign leds = (ena) ? reg1 : 2'bzz;

endmodule
```

El código verilog es correcto, por lo que se simula sin problema, mostrándose el número binario 11 por los leds:

    $ make sim3

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/error1-sim.png)

Sin embargo, al sintetizar obtenemos el siguiente mensaje de error:

   $ make sint3

```
...
2.12.2. Executing Verilog-2005 frontend.
Parsing Verilog input from `/usr/local/bin/../share/yosys/ice40/arith_map.v' to AST representation.
Generating RTLIL representation for module `\_80_ice40_alu'.
Successfully finished Verilog frontend.
Mapping error1.$ternary$error1.v:42$2 ($tribuf) with simplemap.
terminate called after throwing an instance of 'std::out_of_range'
  what():  vector::_M_range_check: __n (which is 1) >= this->size() (which is 1)
Aborted (core dumped)
Makefile:143: recipe for target 'error1.bin' failed
make: *** [error1.bin] Error 134
```

## Error2.v: Dos puertas con la misma señal de habilitación

En este ejemplo se conectan dos registros de un bit a un bus de 2 bits mediante dos puertas triestado que usan la misma señal de habilitación: 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/error2-1.png)

La descripción en verilog es:

```verilog
`default_nettype none

module error2  (
         input wire clk,            //-- Entrada de reloj
         output wire [1:0] leds);   //-- Leds a controlar

//-- Senal de habilitacion para las puertas
wire ena = 1'b1;

//-- Registro de 1 bit
reg reg0;
always @(posedge clk)
  reg0 <= 1'b1;

//-- Registro de 1 bit
reg reg1;
always @(posedge clk)
  reg1 <= 1'b1;

//-- Conectar los registros al cable de 2 bits
//-- Se controlan con la misma señal de habilitacion
assign leds[0] = (ena) ? reg0 : 1'bz;
assign leds[1] = (ena) ? reg1 : 1'bz;

endmodule
```
Se simula con el comando:

    $ make sim4

y se obtiene la misma simulación que en el ejemplo error1

Sintetizamos con:

    $ make sint4

y se obtiene el siguiente mensaje de error

```
arachne-pnr -d 1k -p error2.pcf error2.blif -o error2.txt
seed: 1
device: 1k
read_chipdb +/share/arachne-pnr/chipdb-1k.bin...
  supported packages: tq144, vq100
read_blif error2.blif...
prune...
read_pcf error2.pcf...
instantiate_io...
fatal error: $_TBUF_ gate must drive top-level output or inout port
Makefile:176: recipe for target 'error2.bin' failed
make: *** [error2.bin] Error 1
```
En este caso es el arachne el que provoca el fallo

# Alternativas a las puertas triestado

Las puertas triestado se usan normalmente para interconectar elementos a través de un bus****. La **alternativa** es sustituirlas por **multiplexores**

Imaginemos un bus en el que hay 3 registros que pueden escribir en él, y dos que pueden leer. El acceso en escritura de los registro se controla mediante las 3 señales de habilitación de las puertas triestado

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-ex3.png)

La alternativa es sustituir las puertas triestado por **un multiplexor de 4 a 1**, con una señal de selección de 2 bits:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-ex4.png)

# Ejercicios

# Conclusiones
TODO