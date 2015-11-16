![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T30-microbio)

# Introducción

Uno de los aspectos que más me apasionan de las FPGAs es la posibilidad de **crear tu propio microprocesador** y hacerlo funcionar en la **FPGA**. En este último capítulo diseñaremos un microprocesador extremadamente simple: **Microbio**. Sólo tiene 4 instrucciones y apenas nos permite hacer nada, sin embargo, es un punto de partida para poder ampliarlo con nuevas instrucciones y empezar a adentrarse en este apasionante mundo. **¿Te animas a hacer tu propio microprocesador?**

# Microprocesador Microbio

Microbio es un **procesador "hola mundo"**. Es extremadamente simple, pero es perfecto para comprender su funcionamiento: **leer instruciones de la memoria principal** y **ejecutarlas**

En realidad, Microbio es un **microcontrolador**: es un microprocesador que incluye la memoria y dos periféricos simples: un **puerto de 4 bits** para mostrar información por los leds y un **temporizador de 100ms**

## Características

* Memoria ROM de 64 bytes (6 bits para las direcciones)
* Instrucciones de 8 bits
* 4 instrucciones: HALT, LEDS, WAIT y JP
* No puede realizar cálculos: no tiene unidad aritmético lógica
* No tiene registros de propósito general para almacenar información o hacer operaciones
* Funcuencia de funcionamiento: Reloj de 12MHz
* Tiene 2 **periféricos**:
    * Puerto de salida de 4 bits: conectado a los leds
    * Temporizador de 100ms
* Indicador de programa terminado: El led verde se activa cuando se ejecuta la instrucción HALT que detiene el microprocesador

## Interfaz con el exterior

Microbio se comunica con el exterior mediante los siguientes pines:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-1.png)

* **clk**: Entrada de reloj. Se conecta el reloj de 12 MHz de la FPGA
* **rstn**: Entrada de reset. Está conectada a la señal DTR para poder hacer reset desde el PC. Al hacer un reset, microbio empieza a ejecutar el programa que tiene almacenado a partir de la dirección de memoria 0
* **stop**: Señal de salida, conectada al led verde de la placa Icestick. Se pone a 1 cuando Microbio ha ejecutado la instrucción HALT y el programa ha terminado
* **leds**: Puerto de salida de 4 pines. Está conectado a los 4 leds rojos de la icestick

## Instrucciones

Microbio puede ejecutar las 4 instrucciones siguientes:

| Instrucción  | Operando  | Descripción
|----------|-----------|--------------
| **HALT**     |     | Detener la ejecución. Se activa la señal stop que se muestra por el led verde
| **LEDS**     | _val_    | Escribir el dato _val_ en el puerto de salida de 4 bits, para visualizarlo en los leds
| **WAIT**     |    | Realizar una pausa de 100ms
| **JP**       | _dir_    | Saltar a la dirección de memoria indicada por el operador _dir_

## Memoria

Microbio dispone de una **memoria ROM** de **64 posiciones** donde se almacena el programa a ejecutar. Cada instrucción tiene un ancho de **8 bits**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-2.png)

## Formato de instrucciones

Todas las instrucciones de microbio tienen el mismo formato, que se muestra a continuación:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-3.png)

Constan de dos campos:
* **Código de operación** (CO): Campo de **2 bits** que indica el tipo de operación (HALT, LEDS, WAIT o JP)
* **Campo de datos** (DAT): Campo de **6 bits** que contiene el dato necesario para las instrucciones LEDS y JP

La tabla con los **códigos de operación** es:

| Instrucción  | Código de operación (binario)
|----------|-----------
| WAIT  | 00
| HALT  | 01
| LEDS  | 10
| JP    | 11

# Programas de ejemplo

Se muestran **tres programas de ejemplo** muy sencillo para probar el procesador microbio y aprender su programación: **M0.asm**, **M1.asm** y **M2.asm**

## Ensamblador de Microbio

Las instructiones que entiendo microbio están en **código máquina**: son números almacenados en su memoria rom, que el procesador va leyendo secuencialmente y ejecutando. Podemos programar microbio directamente en código máquina, escribiendo los **números hexadecimales** de sus instrucciones en el fichero **prog.list**

Sin embargo, es mucho más sencillo escribir programas en el **lenguaje ensamblador de Microbio** y utilizar un **programa ensamblador** para **traducirlos a código máquina**. Este proceso se denomina **ensamblado**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-4.png)

El ensamblador de microbio se llama **masm.py**, y ha sido programado en **python 3.5** (Aunque funciona igual en 2.7.9) . Para ensamblar el **programa M0.asm**,por ejemplo, hay que ejecutar el siguiente comando:

```
$ python3 masm.py M0.asm
Assembler for the MICROBIO microprocessor
Released under the GPL license

File prog.list successfully generated
```

Esto genera el fichero **prog.list** con el código máquina, que será cargado en la memoria rom de microbio al realizar la **síntesis / simulación**

Si se ejecuta con la opción _-verbose_ se obtiene más información:

```
$ python3 masm.py M0.asm -verbose
Assembler for the MICROBIO microprocessor
Released under the GPL license

File prog.list successfully generated

Symbol table:

Microbio assembly program:

[00] LEDS 0xF
[01] HALT

Machine code:

8F   //-- [00] LEDS 0xF
40   //-- [01] HALT
```
Nos devuelve la **tabla de símbolos** (en este caso no hay porque no se han definido etiquetas), el **programa en ensamblador**, ya procesado y el programa generado en **código máquina**

**Las fuentes del programa masm.py están disponibles** [en el repositorio de este capítulo](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T30-microbio), de forma que se pueda estudiar cómo funciona y sobre todo mejorarlo y ampliarlo. Por ejemplo, si se amplían las instrucciones de Microbio, será necesario que el ensamblador las soporte

## M0.asm: Hola mundo

El primer programa que haremos en Microbio es el **M0.asm**: Enciende los 4 leds rojos con la instrucción LEDS y se para:

```
//-- M0.asm:  Ejemplo hola mundo para el procesador MICROBIO
//-- Encender todos los leds y terminar

LEDS 0xF  //-- Encender todos los leds
HALT      //-- Terminar
```

El fichero **prog.list**, con el **código máquina** esamblado a través de masm.py es:

```
8F   //-- [00] LEDS 0xF
40   //-- [01] HALT
```

Sólo ocupa 2 bytes, uno por cada instrucción

En el apartado de simulación y pruebas se muestran más detalles, pero para hacer una primera prueba en la placa icestick, el proceso a seguier es:

* Ensamblar el programa****

    $ python3 masm.py M0.asm

* **Sintetizar**:

    $ make sint

* **Cargar microbio con su programa en la FPGA**:

    $ sudo iceprog microbio.bin
* **Desactivar el DTR para hacer RESET**: Ejecutar el programa gtkterm y pulsar la tecla F7 para desactivar el DTR y que se haga un reset del micro. El programa comenzará a ejecutarse

El resultado de la ejecución será que **se encienden tanto los 4 leds rojos como el led verde** (por ejecutarse la instrucción HALT)

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/M0-asm-picture.png)

## M1.asm: Secuencia en los leds

Este ejemplo hace **una secuencia de 4 valores en los leds** y termina. En la secuencia se enciende un único led cada vez, que rota en sentido horario. Al llegar la posición inicial se termina

Se utiliza la instrucción **WAIT** que genera un retraso de **200ms**

```
//-- M1.asm:  Ejemplo de una secuencia simple, sin repeticion

LEDS 0x1  //-- Encender primer led
WAIT      //-- Esperar
LEDS 0x2  //-- Segundo led
WAIT
LEDS 0x4  //-- Tercer led
WAIT
LEDS 0x8  //-- Cuarto led
WAIT
LEDS 0x1  //-- Primer led
HALT      //-- Terminar
```
El fichero **prog.list**, con el **código máquina** esamblado a través de masm.py es:

```
81   //-- [00] LEDS 0x1
00   //-- [01] WAIT
82   //-- [02] LEDS 0x2
00   //-- [03] WAIT
84   //-- [04] LEDS 0x4
00   //-- [05] WAIT
88   //-- [06] LEDS 0x8
00   //-- [07] WAIT
81   //-- [08] LEDS 0x1
40   //-- [09] HALT
```

Para probarlo hay que seguir los mismos pasos que para el programa M0.asm

## M2.asm: Secuencia con repetición infinita

Como tercer ejemplo, se ejecuta una **secuencia en los leds infinita**. Tiene dos partes: la primera es una rotación en sentido horario, similar a la del ejemplo anterior (M1.asm). Cuando termina se inicia otra, que tarda el **doble de tiempo**. Para ello se ejecutan seguidas **dos instrucciones WAIT**, lográndose una pausa de **400ms**

Al llegar al final, se comienza desde el principio mediante la **instrucción JP** (jump).

Se utilizan etiquetas para facilitar la escritura del programa en ensamblador, pero se podría directamente usar la instrucción **JP 0** que salta a la dirección 0, en vez de **JP ini**

```
//-- M2.asm:  Ejemplo de una secuencia infinita

ini:
      LEDS 0x01
      WAIT
      LEDS 0x2
      WAIT
      LEDS 0x4
      WAIT
      LEDS 0x8
      WAIT
      LEDS 0x1
      WAIT
      WAIT
      LEDS 0x3
      WAIT
      WAIT
      LEDS 0x6
      WAIT
      WAIT
      LEDS 0xC
      WAIT
      WAIT
      LEDS 0x9
      WAIT
      WAIT

      JP ini      //-- Saltar al comienzo
```

El código máquina generado por el masm.py es:

```
81   //-- [00] LEDS 0x1
00   //-- [01] WAIT
82   //-- [02] LEDS 0x2
00   //-- [03] WAIT
84   //-- [04] LEDS 0x4
00   //-- [05] WAIT
88   //-- [06] LEDS 0x8
00   //-- [07] WAIT
81   //-- [08] LEDS 0x1
00   //-- [09] WAIT
00   //-- [0A] WAIT
83   //-- [0B] LEDS 0x3
00   //-- [0C] WAIT
00   //-- [0D] WAIT
86   //-- [0E] LEDS 0x6
00   //-- [0F] WAIT
00   //-- [10] WAIT
8C   //-- [11] LEDS 0xC
00   //-- [12] WAIT
00   //-- [13] WAIT
89   //-- [14] LEDS 0x9
00   //-- [15] WAIT
00   //-- [16] WAIT
C0   //-- [17] JP 0x0
```

Al ensamblar en el modo "verbose", podemos ver cómo la **tabla de símbolos** contiene la **etiqueta INI**, asignada a la dirección 0:
```
Symbol table:

INI = 0x00
```

# Implementación de Microbio

El procesador está formado por su **ruta de datos** y su **unidad de control** que genera las **microórdenes** de gobierno de la ruta de datos

## Diagrama de bloques

El diagrama de bloques se muestra en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-5.png)

Las señales en rojo son las microórdenes, generadas por la unidad de control.

## Ruta de datos

La ruta de datos incluye los siguientes elementos:

* **Memoria ROM**:  Memoria de anchura 8 bits y 64 posiciones (Bus de direcciones de 6 bits). Almacena el programa en código máquina que MICROBIO debe ejecutar. Se lee desde el fichero **prog.list**

* **Contador de programa (CP)**: Registro de **6 bits** que almacena la **dirección de la siguiente instrucción** a ejecutar. Cuando está activada la microórden **cp_inc**, se incrementa en una unidad. Si está activada **cp_load** se carga con un valor nuevo procedente del campo DAT de la instrucción almacenada en el registro de instrucción

* **Registro de instrucción (RI)**: Registro de **8 bits** que almacena **la instrucción leída de memoria**. Cuando está activada la microórden ri_load se carga con el dato proveniente de la memoria

* **Registro LEDS**: Registro de **4 bits**, cuya salida está conectada a los 4 leds rojos de la icestick. Se carga el valor proveniente del campo de datos DAT cuando se activa la microórden leds_load

* **Divisor de 200ms**: Registro que hace de divisor de reloj para generar a su salida una señal que emite un pulso cada 200ms. Este pulso es leído por la unidad de control para generar un retardo de 200ms cuando se está ejecutando la instrucción WAIT

* **Unidad de control**: Máquina de estados que genera las microórdenes

* **Registro de halt**: Registro de 1 bit para encender el led verde al ejecutarse la instrucción HALT.

* **Registro de reset**: Registro de 1 bit para sincronizar la señal de reset (y hacerla síncrona)

## Controlador

Las **microórdenes** generadas son por la unidad de control son:

* **cp_inc**: Incrementar contador de programa
* **cp_load**: Cargar contador de programa
* **ri_load**: Cargar registro de instrucción
* **leds_load**: Cargar registro de leds
* **halt**: Instrucción halt ejecutada

El autómata tiene 3 estados, mostrados en el siguiente diagrama:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-6.png)

Los estados son:

* **INIT**: Estado de reposo. Todas las señales están a 0
* **FETCH**: Ciclo de carga de la instrucción. Pasa de la memoria rom al registro de instrucción RI
* **EXEC**: Ciclo de ejecución. Se ejecuta la instrucción correspondiente. Dependiendo de la instrucción que sea, se activan unas microordenes u otras. En el caso de la instrucción JP, se pasa al estado de INIT antes de volver a cargar la siguiente instrucción

Todos los estados duran **1 ciclo**, salvo el de exec al ejecutar la instrucción WAIT (que dura 200ms)
Al ejecutar la instrucción HALT se activa la microorden halt a 1 y se permanece en el mismo estado hasta que se haga un reset

## Microbio.v: Descripción en verilog

Esta es la **descripción completa** del microprocesador en verilog:

```verilog
//-- Fichero microbio.v
`default_nettype none
`include "divider.vh"

//-- Procesador microbio
module microbio (input wire clk,          //-- Reloj del sistema
                 input wire rstn_ini,     //-- Reset
                 output wire [3:0] leds,  //-- leds
                 output wire stop);       //-- Indicador de stop

//-- Parametro: Tiempo de espera para la instruccion WAIT
parameter WAIT_DELAY = `T_200ms;

//-- Parametro: fichero con el programa a cargar en la rom
parameter ROMFILE = "prog.list";

//-- Tamaño de la memoria ROM a instanciar
localparam AW = 6;     //-- Anchura del bus de direcciones
localparam DW = 8;     //-- Anchura del bus de datos

//-- Codigo de operacion de las instrucciones
localparam WAIT = 2'b00;
localparam HALT = 2'b01;
localparam LEDS = 2'b10;
localparam JP = 2'b11;

//-- Instanciar la memoria ROM
wire [DW-1: 0] data;
wire [AW-1: 0] addr;
genrom
  #( .ROMFILE(ROMFILE),
     .AW(AW),
     .DW(DW))
  ROM (
        .clk(clk),
        .addr(addr),
        .data(data)
      );

//-- Registrar la señal de reset
reg rstn = 0;

always @(posedge clk)
  rstn <= rstn_ini;

//-- Declaracion de las microordenes
reg cp_inc = 0;  //-- Incrementar contador de programa
reg cp_load = 0; //-- Cargar el contador de programa
reg ri_load = 0; //-- Cargar una instruccion en el registro de instruccion
reg halt = 0;    //-- Instruccion halt ejecutada
reg leds_load = 0;  //-- Mostrar un valor por los leds

//-- Contador de programa
reg [AW-1: 0] cp;

always @(posedge clk)
  if (!rstn)
    cp <= 0;
  else if (cp_load)
    cp <= DAT;
  else if (cp_inc)
    cp <= cp + 1;

assign addr = cp;

//-- Registro de instruccion
reg [DW-1: 0] ri;

//-- Descomponer la instruccion en los campos CO y DAT
wire [1:0] CO = ri[7:6];    //-- Codigo de operacion
wire [5:0] DAT = ri[5:0];   //-- Campo de datos

always @(posedge clk)
  if (!rstn)
    ri <= 0;
  else if (ri_load)
    ri <= data;

//-- Registro de stop
//-- Se pone a 1 cuando se ha ejecutado una instruccion de HALT
reg reg_stop;

always @(posedge clk)
  if (!rstn)
    reg_stop <= 0;
  else if (halt)
    reg_stop <= 1;

//-- Registro de leds
reg [3:0] leds_r;
always @(posedge clk)
  if (!rstn)
    leds_r <= 0;
  else if (leds_load)
    leds_r <= DAT[3:0];

assign leds = leds_r;

//-- Mostrar la señal de stop por el led verde
assign stop = reg_stop;

//-------- Debug
//-- Sacar codigo de operacion por leds
//assign leds = CO;

/*-- Hacer parpadear el led de stop
reg cont=0;
always @(posedge clk)
  if (clk_tic)
    cont <= cont + 1;
assign stop = cont;
*/

//----------- UNIDAD DE CONTROL
localparam INIT = 0;
localparam FETCH = 1;
localparam EXEC = 2;

//-- Estado del automata
reg [1:0] state;
reg [1:0] next_state;

//-- Transiciones de estados
wire clk_tic;
always @(posedge clk)
  if (!rstn)
    state <= INIT;
  else
    state <= next_state;

//-- Generacion de microordenes
//-- y siguientes estados
always @(*) begin

  //-- Valores por defecto
  next_state = state;      //-- Por defecto permanecer en el mismo estado
  cp_inc = 0;
  cp_load = 0;
  ri_load = 0;
  halt = 0;
  leds_load = 0;


  case (state)
    //-- Estado inicial y de reposo
    INIT:
      next_state = FETCH;

    //-- Ciclo de captura: obtener la siguiente instruccion
    //-- de la memoria
    FETCH: begin
      cp_inc = 1;  //-- Incrementar CP (en el siguiente estado)
      ri_load = 1; //-- Cargar la instruccion (en el siguiente estado)
      next_state = EXEC;
    end

    //-- Ciclo de ejecucion
    EXEC: begin
      next_state = FETCH;

      //-- Ejecutar la instruccion
      case (CO)

        //-- Instruccion HALT
        HALT: begin
          halt = 1;           //-- Activar microorden de halt
          next_state = EXEC;  //-- Permanecer en este estado indefinidamente
        end

        //-- Instruccion WAIT
        WAIT: begin
          //-- Mientras no se active clk_tic, se sigue en el mismo
          //-- estado de ejecucion
          if (clk_tic) next_state = FETCH;
          else next_state = EXEC;
        end

        //-- Instruccion LEDs
        LEDS:
          leds_load = 1;  //-- Microorden de carga en el registro de leds

        //-- Instruccion de Salto
        JP: begin
          cp_load = 1;    //-- Microorden de carga del CP
          next_state = INIT;  //-- Realizar un ciclo de reposo para
                              //-- que se cargue CP antes del estado FETCH
        end

      endcase

    end
  endcase

end

//-- Divisor para marcar la duracion de cada estado del automata
dividerp1 #(WAIT_DELAY)
  TIMER0 (
    .clk(clk),
    .clk_out(clk_tic)
  );

endmodule
```

# Simulación
El banco de pruebas de microbio simplemente activa la señal de reset y genera la señal de reloj. El fichero verilog es el siguiente:

```verilog
module microbio_tb();

//-- Para la simulacion se usa un WAIT_DELAY de 3 ciclos de reloj
parameter WAIT_DELAY = 3;
parameter ROMFILE = "prog.list";

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Datos de salida del componente
wire [3:0] leds;
wire stop;
reg rstn = 0;

//-- Instanciar el componente
microbio #(.WAIT_DELAY(WAIT_DELAY), .ROMFILE(ROMFILE))
  dut(
    .clk(clk),
    .rstn_ini(rstn),
    .leds(leds),
    .stop(stop)
  );

//-- Generador de reloj. Periodo 2 unidades
always #1 clk = ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("microbio_tb.vcd");
  $dumpvars(0, microbio_tb);

  #2 rstn <= 1;

  # 160 $display("FIN de la simulacion");
  $finish;
end

endmodule
```
## Simulación de M0.asm

Para simular el ejemplo hola mundo (**M0.asm**) ensamblamos el programa, con lo que se genera el fichero prog.list con el código máquina a probar:

    $ python3 masm.py M0.asm

y a continuación **lanzamos la simulación** con:

    $ make sim

Se nos abrirá el gtkwave con lo siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/M0-asm-sim1.png)

Para entender la ejecución del programa (que sólo tiene 2 instrucciones: (8F, 40)) nos fijamos en la **señal state**, que nos indica **el estado de la unidad de control**. Sus valores en el tiempo se corresponden con 0, 1, 2, 1, 2 que se corresponden con los estados INIT, FETCH, EXEC, FETCH y EXEC.  Es decir, que efectivamente se ejecutan 2 instrucciones (hay dos ciclos de captura y dos ciclos de ejecución)

En la parte inferior observamos **las microórdenes**. Vemos que en la última ejecución se ha activado **halt a 1** y a partir de ahí la actividad del micro ha cesado

**El contador de programa**, CP, toma los valores 0, 1 y 2, para acceder a esas direcciones (A la 2 no accede, pero apunta a ella porque contiene la dirección de la siguiente instrucción a ejecutar, que estaría en esa dirección)

El **registro de instrucción** inicialmente está a 0 y luego toma los valores 8F y 40, que se corresponden con las instrucciones del programa

# Síntesis y pruebas

# Referencias

# Conclusiones
TODO