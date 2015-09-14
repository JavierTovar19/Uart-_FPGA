![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T21-baud-tx)

## Introducción

Empezaremos el diseño de nuestra UART por el **transmisor**. En este capítulo nos centraremos en la **transmisión de bits**, para entenderla perfectamente. Como ejemplo haremos varios ejemplos de **circuitos que envíen caracteres al PC**, a la velocidad de **115200 baudios**.

## Transmitiendo bits
Los bits se envían al PC de **uno en uno** a través del **pin Tx**. Los datos no se envían aislados, sino que están metido en **una trama**. El **estándar de transmisión** serie define diferentes tramas. Nosotros usaremos la típica, conocida como **8N1** (8 bits de datos, Ninguno de paridad y 1 bit de stop) que tiene el siguiente **formato**:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/serial-frame-format-2.png)

La trama comienza con un **bit a 0**, que se llama **bit de start**. A continuación están los **8 bits del dato a transmitir**, pero **comenzando por el bit 0** (la transmisión se hace comenzando por el bit de menor peso, 0, hasta el mayor, 7). La trama finaliza con un **bit a 1**, llamado **bit de stop**.

Así, para transmitir un dato, la línea (tx) tomará lo siguientes valores. Inicialmente estará en reposo (tx = 1). Se transmite primero el bit de start, por lo que tx = 0. A continuación el bit de menor peso: tx = bit0, luego el siguiente, tx = bit1, y el siguiente tx = bit2 hasta llegar al de mayor peso tx = bit7. Por último se envía el bit de stop, poniendo tx = 1. tx Permanece a 1 hasta que se envíe la siguiente trama.

Como ejemplo veremos cómo **transmitir el caracter ASCII "K"**. Su valor en hexadecimal es 0x4B y en binario: 01001011

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/k-car.png)

La línea de transmisión a lo largo del tiempo tendrá esta forma:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/serial-frame-format.png)

**Todos los bits tienen la misma duración**, que denominaremos **periodo de bit** (Tb)

## Velocidad de transmisión: Baudios

La **velocidad de transmisión** se mide en **baudios**. Como estamos usando una transmisión binaria, en la que sólo hay dos valores (0 y 1), **un baudio equivale a un bit por segundo** (bps)

Para que diferentes circuitos se puedan comunicar entre ellos, **las velocidades están normalizadas**. Pueden tener los siguientes valores: 115200, 56700, 38400, 19200, 9600, 4800, 2400, 1200, 600 y 300 baudios. Nosotros la fijaremos a la máxima: **115200 baudios**

Para transmitir a una velocidad de **X baudios**, necesitamos generar una **señal cuadrada cuya frecuencia sea igual a X**. Cada flanco de subida de esta señal indica cuándo enviar el siguiente bit:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/serial-frame-3.png)

## Generador de señal de reloj para la transmisión

Lo primero que necesitamos para transmitir datos es **generar la señal de reloj** con la frecuencia adecuada. Esto ya lo sabemos hacer: usaremos un **divisor de frecuencias**.

Cuando trabajamos con la placa iCEstick, **el divisor para conseguir una velocidad de B baudios** viene dado por la ecuación: M = 12000000 / B

Por tanto, para transmitir a **115200 baudios** necesitamos un divisor de:  M = 12000000 / 115200  = 104.16 -> **M = 104**

Los **valores de los divisores** para **transmitir a las velocidades estándares** están calculados con la fórmula anterior y se encuentran en el fichero **baudgen.vh**:

```verilog
//-- Fichero baudgen.vh
`define B115200 104
`define B57600  208
`define B38400  313

`define B19200  625
`define B9600   1250
`define B4800   2500
`define B2400   5000
`define B1200   10000
`define B600    20000
`define B300    40000
```
Para generar la **señal de reloj para  transmitir a una velocidad** (por ejemplo 115200 baudios) es tan sencillo como **instanciar el divisor** que ya conocemos usando las constantes anteriores:

```verilog
divider #(`B115200)
  BAUD0 (
    .clk_in(clk),
    .clk_out(clk_baud)
  );
```

## Ejemplo 1: Transmitiendo el caracter "K"

Este primer ejemplo envía el carácter "K" desde la FPGA al ordenador cada vez que la señal dtr pasa de 0 a 1, a la velocidad de 115200 baudios

### Funcionamiento

Para realizar una transmisión del dato usaremos un **registro de desplazamiento**, con **carga paralela**. La **salida serie** se conecta directamente a la linea de transmisión **tx**, a través de un multiplexor. Cuando la señal de load está a 0, el registro se carga con un valor de 10 bits: el dato "K", seguido de los bits 01 (bit de start y bit de reposo). 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx-1.png)

Cuando la carga está a 1 se realiza el desplazamiento hacia la derecha y por la izquierda se inserta un 1. De esta forma, al transmitirse la trama completa, el bit menos significativo será un 1, para que la línea tx permanezca a 1 (en reposo)

**El multiplexor de salida** pone en reposo (a 1) la linea de transmisión cuando se está cargando el registro. Se usa para que **no se envíen caracteres "basura"** por la línea tx en el momento del arranque. El registro de desplazamiento inicialmente tiene el valor 0 en todos sus bits, por lo tx se pondría a 0 y receptor lo interpretaría como que es un bit de start, recibiendo un carácter "basura" que no quería ser enviado.

Para hacer las pruebas, **load está conectada a la señal dtr**, por lo que la podremos **controlar manualmente desde el pc**.  Al ponerla a 1 se empieza la transmisión. Cuando se haya enviado el carácter K, el registro de desplazamiento tendrá todos sus bits a 1 y lo que sale por tx será siepre un 1. Es decir, que no habrá transmisión. Al poner load a 0, se carga con el nuevo valor, y cuando pase a 1 se enviará el nuevo dato. 

### baudtx.v: Descripción del hardware

La descripción en lenguaje Verilog del circuito anterior es la siguiente:

``` verilog
//-- Fichero: baudtx.v
`default_nettype none

`include "baudgen.vh"

//--- Modulo que envia un caracter cunado load esta a 1
module baudtx(input wire clk,       //-- Reloj del sistema (12MHz en ICEstick)
              input wire load,      //-- Señal de cargar / desplazamiento
              output wire tx        //-- Salida de datos serie (hacia el PC)
             );

//-- Parametro: velocidad de transmision
parameter BAUD =  `B115200;

//-- Registro de 10 bits para almacenar la trama a enviar:
//-- 1 bit start + 8 bits datos + 1 bit stop
reg [9:0] shifter;

//-- Reloj para la transmision
wire clk_baud;

//-- Registro de desplazamiento, con carga paralela
//-- Cuando DTR es 0, se carga la trama
//-- Cuando DTR es 1 se desplaza hacia la derecha, y se 
//-- introducen '1's por la izquierda
always @(posedge clk_baud)
  if (load == 0)
    shifter <= {"K",2'b01};
  else
    shifter <= {1'b1, shifter[9:1]};

//-- Sacar por tx el bit menos significativo del registros de desplazamiento
//-- Cuando estamos en modo carga (dtr == 0), se saca siempre un 1 para 
//-- que la linea este siempre a un estado de reposo. De esta forma en el 
//-- inicio tx esta en reposo, aunque el valor del registro de desplazamiento
//-- sea desconocido
assign tx = (load) ? shifter[0] : 1;

//-- Divisor para obtener el reloj de transmision
divider #(BAUD)
  BAUD0 (
    .clk_in(clk),
    .clk_out(clk_baud)
  );

endmodule
```
La primera línea de código es nueva:

``` verilog
`default_nettype none
```
Por defecto en Verilog, si aparecen **etiquetas no declaras** se asumen que son cables (tipo wire). Esto podría parecer muy útil pero es una fuente de problemas en la depuración. Cuando el diseño es complejo y se tienen muchos cables, puede ocurrir que uno de ellos se escriba mal. El compilador, en vez de dar un error, supondrá que se trata de un cable nuevo. Este comportamiento se puede cambiar con la instrucción anterior. Al definir el tipo de cable a **none**, cada vez que se detecte un identificador no declarado, saltará un mensaje de error

En el componente se instancia el divisor para generar la señal de reloj para transmitir a 115200 baudios. Esta señal se usa como reloj para el registro de desplazamiento

### Simulación

El banco de pruebas instancia el componente baudtx, se establece un proceso para generar el relog y otro que genera 3 pulsos en dtr para que se envíen 3 characteres

Para simular ejecutamos el comando:

    $ make sim

El resultado es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx-1-sim.png)

La primera señal el reloj para la transmisión de los bits a 115200 baudios. Cuando dtr se pone a cero se carga el registro.  Al ponerse a 1 se empieza a enviar el dato en serie. En el pantallazo se observan 2 pulsos en dtr y cómo despues de ellos se comienza a enviar el dato en serie

### Síntesis y pruebas

Para sintetizar ejecutamos el comando:

    $ make sint

Los recursos ocupados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 6 / 96
|PLBs      | 8 / 160
|BRAMs     | 0 / 16

Lo cargamos en la FPGA con:

    $ sudo iceprog baudtx.bin

Para probarlo arrancamos el **gtkterm** y con configuramos para que el puerto sea el **/dev/ttyUSB1** a la velocidad de **115200 baudios**. Con la **tecla F7** cambiamos el estado de la **señal DTR**. La primera vez que pulsamos aparecerá una "K" en la pantalla. La siguiente vez DTR cambia de estado, pero no ocurre nada. Si volvemos a pulsar aparecerá otra "K". Cada dos pulsaciones de F7 obtendremos una "K". Si dejamos apretada F7, aparecerán multiples "K"s

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx-1-gtkterm.png)

## Ejemplo 2: Transmisión continua
Con este ejemplo comprobamos si el transmisor funciona correctamente a la **máxima velocidad**, **transmitiendo un carácter inmediatamete a continuacion del otro**. Cada vez que la señal DTR se ponga a 1, se transmite el carácter K constantemente

### Funcionamiento

El cronograma de transmisión continua para el caracter K se muestra en esta figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/serial-frame-4.png)

Las líneas verticales punteadas muestran el comienzo y fin de los caracteres enviados. Sin embargo, si no tenemos esas referencias visuales, es **imposible distinguir donde empieza y acaba la trama**. Esto mismo le pasa al receptor serie del PC. Si hacemos que la FPGA envíe constantemente caracteres y arrancamos el programa gtkterm, la lectura comenzará en cualquier punto de la trama. Si pasa esto, **se interpretará como bit de start otro diferente del correcto**, recibiéndose una ráfaga de caracteres incorrectos. En el dibujo se ha marcado en rojo otro punto, llamado A, que podría ser leido como bit de arranque. Si esto ocurre, tanto receptor como transmisor quedarán desincronizados hasta que la línea alcance el estado de reposo.

Es una de las razones por las que **se usa la señal DTR para arrancar la transmisión de datos** en ejemplo. Así garantizamos que siempre haya sincronización.

Para lograr la transmisión continúa sólo hay que hacer un pequeño cambio en el ejemplo anterior: Ahora en vez de introducir "1"s por la izquierda del registro de desplazamiento, **haremos que se inserten los bits enviados**, **conectando la _ser_out_ con _ser_in_**, de manera que construimos un anillo en el que todos los bits de la trama se están constantemente enviando

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx2-1.png)

### baudtx2.v: Descripción del hardware

La nueva descripción en Verilog queda como la siguiente:

```verilog
`default_nettype none

`include "baudgen.vh"

//--- Modulo que envia un caracter cunado load esta a 1
module baudtx2(input wire clk,       //-- Reloj del sistema (12MHz en ICEstick)
               input wire load,      //-- Señal de cargar / desplazamiento
               output wire tx        //-- Salida de datos serie (hacia el PC)
              );

//-- Parametro: velocidad de transmision
parameter BAUD =  `B115200;

//-- Registro de 10 bits para almacenar la trama a enviar:
//-- 1 bit start + 8 bits datos + 1 bit stop
reg [9:0] shifter;

//-- Reloj para la transmision
wire clk_baud;

//-- Registro de desplazamiento, con carga paralela
//-- Cuando DTR es 0, se carga la trama
//-- Cuando DTR es 1 se desplaza hacia la derecha, y se 
//-- introducen '1's por la izquierda
always @(posedge clk_baud)
  if (load == 0)
    shifter <= {"K",2'b01};
  else
    shifter <= {shifter[0], shifter[9:1]};

//-- Sacar por tx el bit menos significativo del registros de desplazamiento
//-- Cuando estamos en modo carga (dtr == 0), se saca siempre un 1 para 
//-- que la linea este siempre a un estado de reposo. De esta forma en el 
//-- inicio tx esta en reposo, aunque el valor del registro de desplazamiento
//-- sea desconocido
assign tx = (load) ? shifter[0] : 1;

//-- Divisor para obtener el reloj de transmision
divider #(BAUD)
  BAUD0 (
    .clk_in(clk),
    .clk_out(clk_baud)
  );

endmodule
```
La diferencia con respecto al ejemplo anterior está en esta línea:

```verilog
shifter <= {shifter[0], shifter[9:1]};
```
Ahora por la izquierda se inserta shifter[0] en vez de un bit a 1

### Simulación

El banco de pruebas es muy parecido al del ejemplo anterior. Se ha cambiado la temporización de la señal dtr

``` verilog
`include "baudgen.vh"

module baudtx2_tb();

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Linea de tranmision
wire tx;

//-- Simulacion de la señal dtr
reg dtr = 0;

//-- Instanciar el componente para que funcione a 115200 baudios
baudtx2 #(`B115200)
  dut(
    .clk(clk),
    .load(dtr),
    .tx(tx)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("baudtx2_tb.vcd");
  $dumpvars(0, baudtx2_tb);

  //-- Primer envio: cargar y enviar
  #10 dtr <= 0;
  #300 dtr <= 1;
 
  //-- Segundo envio
  #10000 dtr <=0;
  #2000 dtr <=1;

  //-- Tercer envio
  #10000 dtr <=0;
  #2000 dtr <=1;

  #5000 $display("FIN de la simulacion");
  $finish;
end

endmodule
```
Para hacer la simulación ejecutamos:

    $ make sim2

Y el resultado de la simulación es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx-2-sim.png)

Cuando dtr se pone a 1, se observa la transmisión continua. Cuando se pone a 0 se deja de transmitir y la línea permanece en reposo. Nuevamente se pone a 1 y comienza otra ráfaga de transmisión

### Síntesis y pruebas

Para sintetizar ejecutamos el comando:

    $ make sint2

Los recursos ocupados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 6 / 96
|PLBs      | 7 / 160
|BRAMs     | 0 / 16

Lo cargamos en la FPGA con:

    $ sudo iceprog baudtx2.bin

Abrimos el gtkterm y pulsamos F7 para cambiar el DTR. Empezarán a aparecer Ks llenando rápidamente la pantalla completa del terminal:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx-2-gtkterm.png)

Al pulsar F7 nuevamente la ráfaga se para

## Ejemplo 3: Transmisión periódica 

En este ejemplo enviaremos el carácter K cada 250ms, sin tener que usar el DTR

### Funcionamiento

Este circuito **no tiene entrada exterior para la carga del registro**, sino que se ha añadido un **divisor** para generar una **señal de 250ms** y que se cargue periódicamente. De esta forma, cada 250ms, se envía el carácter K:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx3-1.png)

La señal de carga no se conecta directamente desde el divisor, sino que se pasa por **un registro con entrada de reloj clk_baud**. Se ha añadido para que **la señal de load esté sincronizada con la de clk_baud**, y que sus distancias relativas sean siempre las mismas. Si no se pone, pueden ocurrir situaciones en tengan los flancos de subida muy cercanos y se capturen datos incorrectos

### baudtx3.v: Descripción del hardware

La descripción del circuito en verilog es:

```verilog
//-- baudtx3.v
`default_nettype none

`include "baudgen.vh"
`include "divider.vh"

//--- Modulo que envia un caracter cunado load esta a 1
module baudtx3(input wire clk,       //-- Reloj del sistema (12MHz en ICEstick)
               output wire tx        //-- Salida de datos serie (hacia el PC)
              );

//-- Parametro: velocidad de transmision
parameter BAUD =  `B115200;
parameter DELAY = `T_250ms;

//-- Registro de 10 bits para almacenar la trama a enviar:
//-- 1 bit start + 8 bits datos + 1 bit stop
reg [9:0] shifter;

//-- Reloj para la transmision
wire clk_baud;

//-- Señal de carga periodica
wire load;

//-- Señal de load sincronizada con clk_baud
reg load2;

//-- Registro de desplazamiento, con carga paralela
//-- Cuando DTR es 0, se carga la trama
//-- Cuando DTR es 1 se desplaza hacia la derecha, y se 
//-- introducen '1's por la izquierda
always @(posedge clk_baud)
  if (load2 == 0)
    shifter <= {"K",2'b01};
  else
    shifter <= {1'b1, shifter[9:1]};

//-- Sacar por tx el bit menos significativo del registros de desplazamiento
//-- Cuando estamos en modo carga (dtr == 0), se saca siempre un 1 para 
//-- que la linea este siempre a un estado de reposo. De esta forma en el 
//-- inicio tx esta en reposo, aunque el valor del registro de desplazamiento
//-- sea desconocido
assign tx = (load2) ? shifter[0] : 1;

//-- Sincronizar la señal load con clk_baud
//-- Si no se hace, 1 de cada X caracteres enviados tendrá algún bit
//-- corrupto (en las pruebas con la icEstick salian mal 1 de cada 13 aprox
always @(posedge clk_baud)
  load2 <= load;

//-- Divisor para obtener el reloj de transmision
divider #(BAUD)
  BAUD0 (
    .clk_in(clk),
    .clk_out(clk_baud)
  );

//-- Divisor para generar la señal de carga periodicamente
divider #(DELAY)
  DIV0 (
    .clk_in(clk),
    .clk_out(load)
  );

endmodule
```
La señal de carga del registro es _load2_, en vez de _load_, que está sincronizada con _clk_baud_

### Simulación

El banco de pruebas es como el de los otros ejemplos, salvo que al instanciar el componente baudtx3 se añade el **parámetro DELAY**. En la simulación se pone otro más corto que 250ms, para que no se haga la simulación eterna :-)

```verilog
//-- Fichero baudtx3_tb.v
`include "baudgen.vh"

module baudtx3_tb();

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Linea de tranmision
wire tx;


//-- Instanciar el componente para que funcione a 115200 baudios
baudtx3 #(.BAUD(`B115200), .DELAY(4000))
  dut(
    .clk(clk),
    .tx(tx)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("baudtx3_tb.vcd");
  $dumpvars(0, baudtx3_tb);

  #40000 $display("FIN de la simulacion");
  $finish;
end

endmodule
```
Realizamos la simulación con el comando:

    $ make sim3

Los resultados son:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx3-sim.png)

Se observa cómo con cada pulso de load se envía un dato nuevo. También se puede observar como las señales _load_ y _load2_ son ligeramente diferente. La distancia entre ellas va variando debido a que **_load_ es asíncrona**, pero _load2_ ya está sincronizada. 

### Síntesis y pruebas

Para sintetizar ejecutamos el comando:

    $ make sint3

Los recursos ocupados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 6 / 96
|PLBs      | 19 / 160
|BRAMs     | 0 / 16

Lo cargamos en la FPGA con:

    $ sudo iceprog baudtx3.bin

Al arrancar el gtkterm empezarán a aparecer Ks en la pantalla, a la velocidad de 4 por segundo.

Si la señal de _load_ **NO está sincronizada**, se pueden obtener resultado como estos:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx-3-gtkterm.png) 

Se observa que periódicamente un carácter se recibe incorrectamente.

Cuando está todo sincronizado bien, el resultado es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T21-baud-tx/images/baudtx-3-gtkterm-ok.png)

## Ejercicios
* Probar el último ejemplo pero cambiando las velocidades de transmisión a 19600 y 9600 baudios

## CONCLUSIONES
TODO

## CRÉDITOS

* Aunque este transmisor se ha escrito desde cero, me he inspirado en la UART del [proyecto swapforth](https://github.com/jamesbowman/swapforth), de **James Bowman**. ¡Muchas gracias!


