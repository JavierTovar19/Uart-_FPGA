![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad-3.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T24-uart-tx)

# Introducción
**Completaremos** la unidad de transmisión serie asíncrona y haremos dos **ejemplos de uso**, que envían la cadena "Hola!...". Uno de manera contínua al activarse la señal de DTR y otro cada segundo

Se trata de la **última versión de la unidad de transmisión**, que quedará lista para usar en nuestros proyectos

# Módulo UART-TX

La unidad de transmisión serie la encapsularemos dentro del **módulo uart-tx**

## Descripción de la interfaz

La unidad de transmisión tiene 4 entradas y 2 salidas:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad-2.png)

* **Entradas**:
    * **clk**: Reloj del sistema (12MHz en la ICEstick)
    * **rstn**: Reset negado. Cuando rstn es 0, se hace un reset síncrono de la unidad de transmisión
    * **start**: Comienzo de la transmisión. Cuando está a 1, se captura el carácter que entra por data y se empieza a enviar
    * **data**:  Dato de 8 bits a enviar

* **Salidas**:
    * **tx**: Salida serie del dato. Conectar a la línea de transmisión
    * **ready**: Estado de la transmisión. Cuando ready es 1, la unidad está lista para transmitir. Y empezará en cuanto start se ponga a 1. Si ready es 0 la unidad está ocupada con el envío de un carácter

## Cronograma

Para transmitir primero se poner el **carácter de 8 bits en la entrada _data_** y **se activa la señal _start_**. Se comienza a transmitir por _tx_. La **señal _ready_** se pone a **0** para indicar que **la unidad está ocupada**. Cuando el carácter se ha terminado de enviar, **_ready_ se pone de nuevo a 1**.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad-4.png)

## Diagrama de bloques

El diagrama de bloques del transmisor se muestra en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad-1.png)

El diseño, como hicimos en el capítulo pasado, está dividido en su **ruta de datos** y su **controlador**. Hay **dos microórdenes** que genera el controlador: _bauden_ y _load_, con las que activa el temporizador de bits y la carga del registro de desplazamiento respectivamente. _Load_ también se usa para poner a cero el contador de bits

El dato a transmitir se recibe por _data_, y se registra para cumplir con las **normas del diseño síncrono**. El controlador genera también la **señal ready** para indicar cuándo se ha terminado de transmitir

## Descripción del módulo en Verilog

El código de la unidad de transmisión en verilog es el siguiente:

```verilog
//-- Fichero: uart_tx.v
`default_nettype none

`include "baudgen.vh"

//--- Modulo de transmision serie
//--- La salida tx ESTA REGISTRADA
module uart_tx (
         input wire clk,        //-- Reloj del sistema (12MHz en ICEstick)
         input wire rstn,       //-- Reset global (activo nivel bajo)
         input wire start,      //-- Activar a 1 para transmitir
         input wire [7:0] data, //-- Byte a transmitir
         output reg tx,         //-- Salida de datos serie (hacia el PC)
         output wire ready      //-- Transmisor listo / ocupado
       );

//-- Parametro: velocidad de transmision
parameter BAUD =  `B115200;

//-- Señal de start registrada
reg start_r; 

//-- Reloj para la transmision
wire clk_baud;

//-- Bitcounter
reg [3:0] bitc;

//-- Datos registrados
reg [7:0] data_r;

//--------- Microordenes
wire load;    //-- Carga del registro de desplazamiento. Puesta a 0 del
              //-- contador de bits
wire baud_en; //-- Habilitar el generador de baudios para la transmision

//-------------------------------------
//-- RUTA DE DATOS
//-------------------------------------

//-- Registrar la entrada start
//-- (para cumplir con las reglas de diseño sincrono)
always @(posedge clk)
  start_r <= start;

always @(posedge clk)
  if (start == 1 && state == IDLE)
    data_r <= data;

//-- Registro de 10 bits para almacenar la trama a enviar:
//-- 1 bit start + 8 bits datos + 1 bit stop
reg [9:0] shifter;

//-- Cuando la microorden load es 1 se carga la trama
//-- con load 0 se desplaza a la derecha y se envia un bit, al
//-- activarse la señal de clk_baud que marca el tiempo de bit
//-- Se introducen 1s por la izquierda
always @(posedge clk)
  //-- Reset
  if (rstn == 0)
    shifter <= 10'b11_1111_1111;

  //-- Modo carga
  else if (load == 1)
    shifter <= {data_r,2'b01};

  //-- Modo desplazamiento
  else if (load == 0 && clk_baud == 1)
    shifter <= {1'b1, shifter[9:1]};

//-- Contador de bits enviados
//-- Con la microorden load (=1) se hace un reset del contador
//-- con load = 0 se realiza la cuenta de los bits, al activarse
//-- clk_baud, que indica el tiempo de bit
always @(posedge clk)
  if (load == 1)
    bitc <= 0;
  else if (load == 0 && clk_baud == 1)
    bitc <= bitc + 1;

//-- Sacar por tx el bit menos significativo del registros de desplazamiento
//-- ES UNA SALIDA REGISTRADA, puesto que tx se conecta a un bus sincrono
//-- y hay que evitar que salgan pulsos espureos (glitches)
always @(posedge clk)
  tx <= shifter[0];

//-- Divisor para obtener el reloj de transmision
baudgen #(BAUD)
  BAUD0 (
    .clk(clk),
    .clk_ena(baud_en),
    .clk_out(clk_baud)
  );

//------------------------------
//-- CONTROLADOR
//------------------------------

//-- Estados del automata finito del controlador
localparam IDLE  = 0;  //-- Estado de reposo
localparam START = 1;  //-- Comienzo de transmision
localparam TRANS = 2;  //-- Estado: transmitiendo dato

//-- Estados del autómata del controlador
reg [1:0] state;

//-- Transiciones entre los estados
always @(posedge clk)

  //-- Reset del automata. Al estado inicial
  if (rstn == 0)
    state <= IDLE;

  else
    //-- Transiciones a los siguientes estados
    case (state)

      //-- Estado de reposo. Se sale cuando la señal
      //-- de start se pone a 1
      IDLE: 
        if (start_r == 1) 
          state <= START;
        else 
          state <= IDLE;

      //-- Estado de comienzo. Prepararse para empezar
      //-- a transmitir. Duracion: 1 ciclo de reloj
      START:
        state <= TRANS;

      //-- Transmitiendo. Se esta en este estado hasta
      //-- que se hayan transmitido todos los bits pendientes
      TRANS:
        if (bitc == 11)
          state <= IDLE;
        else
          state <= TRANS;

      //-- Por defecto. NO USADO. Puesto para
      //-- cubrir todos los casos y que no se generen latches
      default:
        state <= IDLE;

    endcase

//-- Generacion de las microordenes
assign load = (state == START) ? 1 : 0;
assign baud_en = (state == IDLE) ? 0 : 1;

//-- Señal de salida. Esta a 1 cuando estamos en reposo (listos
//-- para transmitir). En caso contrario esta a 0
assign ready = (state == IDLE) ? 1 : 0;

endmodule
```

# Ejemplo 1: Enviando cadenas continuamente

En este primer ejemplo de utilización del módulo de transmisión **enviaremos la cadena "Hola!..."** de forma continua, mientras que la **señal de dtr esté activada**

## Diagrama de bloques

El circuito sigue el mismo esquema de ruta de datos / controlador

### Ruta de datos

Los **8 caracteres** de la cadena están cableados a un **multiplexor de 8 a 1**, cuya entrada de selección está controlada por un **contador de 3 bits**, de forma que se vayan sacando los caracteres uno a uno. El caracter a transmitir se registra y se introduce por la entrada data del transmisor.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad-3.png)

### Controlador

El controlador tiene las siguientes **entradas**:
* **car_count**: Contador de caracteres. Es la salida del contador de 3 bits. Indica el carácter que se está enviando
* **ready**: Señal proveniente de la unidad de transmisión para saber cuándo está lista para enviar el siguiente caracter
* **transmit**: Señal proveniente del exterior del circuito para indicar que comience la transmisión de la cadena. Está conectada a la señal DTR que controla el usuario

Las **microórdenes** que genera son:
* **start**: Para comenzar la transmisión de un carácter
* **cena**: Counter enable. Activar el contador para acceder al siguiente caracter a enviar

Este es el diagrama de estados del autómata:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad-5.png)

Tiene 4 estados:
* **IDLE**: Estado de reposo. Permanece en este estado indefinidamente, hasta que se activa la orden transmit para empezar a enviar la cadena
* **TXCAR**: Transmisión de un caracter. Permanece en este estado mientras se envía un carácter. Termina cuando la señal ready de transmisor se pone a 1 para indicar que está lista para enviar otro carácter
* **NEXT**: Transmisión del siguiente carácter. Sólo dura 1 ciclo de reloj. Se incrementa el contador de caracteres para pasar al siguiente carácter. Si no es el último carácter se pasa al estado TXCAR para enviarlo. Si es el último se pasa al estado END
* **END**: Envío del último carácter y se espera a que su transmisión se complete (ready = 1). Al ocurrir esto se pasa al estado de reposo y el ciclo vuelve a comenzar

## scicad1.v: Descripción en verilog

El código en verilog es:

```verilog
//-- Fichero: scicad1.v
`include "baudgen.vh"

//-- Modulo para envio de una cadena por el puerto serie
module scicad1 (input wire clk,  //-- Reloj del sistema
                input wire dtr,  //-- Señal de DTR
                output wire tx   //-- Salida de datos serie
               );

//-- Velocidad a la que hacer las pruebas
parameter BAUD = `B115200;

//-- Reset
reg rstn = 0;

//-- Señal de listo del transmisor serie
wire ready;

//-- Dato a transmitir (normal y registrado)
reg [7:0] data;
reg [7:0] data_r;

//-- Señal para indicar al controlador el comienzo de la transmision
//-- de la cadena. Es la de DTR registrada
reg transmit;

//-- Microordenes
reg cena;      //-- Counter enable (cuando cena = 1)
reg start;  //-- Transmitir cadena (cuando transmit = 1)

//------------------------------------------------
//-- 	RUTA DE DATOS
//------------------------------------------------

//-- Inicializador
always @(posedge clk)
  rstn <= 1;

//-- Instanciar la Unidad de transmision
uart_tx #(.BAUD(BAUD))
  TX0 (
    .clk(clk),
    .rstn(rstn),
    .data(data_r),
    .start(start),
    .ready(ready),
    .tx(tx)
  );

//-- Multiplexor con los caracteres de la cadena a transmitir
//-- se seleccionan mediante la señal car_count
always @*
  case (car_count)
    8'd0: data <= "H";
    8'd1: data <= "o";
    8'd2: data <= "l";
    8'd3: data <= "a";
    8'd4: data <= "!";
    8'd5: data <= ".";
    8'd6: data <= ".";
    8'd7: data <= ".";
    default: data <= ".";
  endcase

//-- Registrar los datos de salida del multiplexor
always @*
  data_r <= data;

//-- Contador de caracteres
//-- Cuando la microorden cena esta activada, se incrementa
reg [2:0] car_count;

always @(posedge clk)
  if (rstn == 0)
    car_count = 0;
  else if (cena)
    car_count = car_count + 1;

//-- Registrar señal dtr para cumplir con normas diseño sincrono
always @(posedge clk)
  transmit <= dtr;

//----------------------------------------------------
//-- CONTROLADOR
//----------------------------------------------------
localparam IDLE = 0;   //-- Reposo
localparam TXCAR = 2'd1;  //-- Transmitiendo caracter
localparam NEXT = 2'd2;   //-- Preparar transmision del sig caracter
localparam END = 3;    //-- Terminar

//-- Registro de estado del automata
reg [1:0] state;

//-- Gestionar el cambio de estado
always @(posedge clk)

  if (rstn == 0)
    //-- Ir al estado inicial
    state <= IDLE;

  else
    case (state)
      //-- Estado inicial. Se sale de este estado al recibirse la
      //-- señal de transmit, conectada al DTR
      IDLE: 
        if (transmit == 1) state <= TXCAR;
        else state <= IDLE;

      //-- Estado de transmision de un caracter. Esperar a que el 
      //-- transmisor serie este disponible. Cuando lo esta se pasa al
      //-- siguiente estado
      TXCAR: 
        if (ready == 1) state <= NEXT;
        else state <= TXCAR;

      //-- Envio del siguiente caracter. Es un estado transitorio
      //-- Cuando se llega al ultimo caracter se pasa para finalizar
      //-- la transmision 
      NEXT:	
        if (car_count == 7) state <= END;
        else state <= TXCAR;

      //-- Ultimo estado:finalizacion de la transmision. Se espera hasta
      //-- que se haya enviado el ultimo caracter. Cuando ocurre se vuelve
      //-- al estado de reposo inicial
      END: 
        //--Esperar a que se termine ultimo caracter
        if (ready == 1) state <= IDLE;
        else state <= END;

      //-- Necesario para evitar latches
      default:
         state <= IDLE;

    endcase

//-- Generacion de las microordenes
always @*
  case (state)
    IDLE: begin
      start <= 0;
      cena <= 0;
    end

    TXCAR: begin
      start <= 1;
      cena <= 0;
    end

    NEXT: begin
      start <= 0;
      cena <= 1;
    end

    END: begin
      start <= 0;
      cena <= 0;
    end

    default: begin
      start <= 0;
      cena <= 0;
    end
  endcase

endmodule
```

## Simulación

El banco de pruebas **envía dos pulsos en la señal dtr**, para que se envíe la cadena 2 veces. El código verilog es el siguiente:

```verilog
//-- Fichero: scicad1_tb.v
`include "baudgen.vh"

module scicad1_tb();

//-- Baudios con los que realizar la simulacion
localparam BAUD = `B115200;

//-- Tics de reloj para envio de datos a esa velocidad
//-- Se multiplica por 2 porque el periodo del reloj es de 2 unidades
localparam BITRATE = (BAUD << 1);

//-- Tics necesarios para enviar una trama serie completa, mas un bit adicional
localparam FRAME = (BITRATE * 11);

//-- Tiempo entre dos bits enviados
localparam FRAME_WAIT = (BITRATE * 4);

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Linea de tranmision
wire tx;

//-- Simulacion de la señal dtr
reg dtr = 0;

//-- Instanciar el componente
scicad1 #(.BAUD(BAUD))
  dut(
    .clk(clk),
    .dtr(dtr),
    .tx(tx)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;

//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("scicad1_tb.vcd");
  $dumpvars(0, scicad1_tb);

  #1 dtr <= 0;

  //-- Comenzar primer envio
  #FRAME_WAIT dtr <= 1;
  #(BITRATE * 2) dtr <=0;

  //-- Segundo envio (2 caracteres mas)
  #(FRAME * 11) dtr <=1;
  #(BITRATE * 1) dtr <=0;

  #(FRAME * 11) $display("FIN de la simulacion");
  $finish;
end

endmodule
```
Para simular ejecutamos el comando:

    $ make sim

En este pantallazo del gtkwave sólo se muestra la transmisión de la **primera cadena**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad1-sim.png)

Se observa cómo en cuanto **transmit** se pone a **1**, se **empieza la transmisión** (tx se pone a 0 para tranmitir el bit de start). La señal de **ready** se pone a 1 cada vez que se ha enviado un carácter. Al enviarse el octavo, permanece a 1 (porque transmit está a 0, que indica que no se quiere transmitir más)

## Síntesis y pruebas
Hacemos la síntesis con el siguiente comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 8 / 96
|PLBs      | 25 / 160
|BRAMs     | 0 / 16

y lo cargamos en la FPGA con:

    $ sudo iceprog scicad1.bin

Abrimos el **gtkterm** y lo configuramos a **115200 baudios**. Al apretar la tecla **F7** para **activar el DTR**, se envía la cadena "**Hola!...**" continuamente. Al volver a apretar F7 se para. Observamos que ahora **en ningún momento aparecen caracteres extraños**. Da igual si dejamos pulsado el F7 y la señal de dtr se pone a oscilar

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad1-gtkterm.png)

En este **vídeo de youtube** se puede ver el ejemplo en acción:

[![Click to see the youtube video](http://img.youtube.com/vi/hWNw9ULRMvw/0.jpg)](https://www.youtube.com/watch?v=hWNw9ULRMvw)

# Ejemplo 2: Enviando una cadena cada segundo

Este circuito de ejemplo es similar al anterior, pero ahora **el envío de la cadena se hace cada segundo**, en vez de tener que activar el DTR

## Diagrama de bloques

El diagrama de bloques incluye el nuevo **divisorP1** que genera a su salida un pulso de 1 ciclo de anchura cuando transcurre 1 segundo. Se conecta directamente a la señal de transmit, que antes llegaba por el dtr.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad-6.png)

## scicad2.v: Descripción en verilog

La descripción en verilog es:

```verilog
//-- Fichero scicad2.v
`include "baudgen.vh"
`include "divider.vh"

//-- Modulo para envio de una cadena por el puerto serie
module scicad2 (input wire clk,  //-- Reloj del sistema
                output wire tx   //-- Salida de datos serie
               );

//-- Velocidad a la que hacer las pruebas
parameter BAUD = `B115200;

//- Tiempo de envio
parameter DELAY = `T_1s;

//-- Reset
reg rstn = 0;

//-- Señal de listo del transmisor serie
wire ready;

//-- Dato a transmitir (normal y registrado)
reg [7:0] data;
reg [7:0] data_r;

//-- Señal para indicar al controlador el comienzo de la transmision
//-- de la cadena
reg transmit_r;
wire transmit;

//-- Microordenes
reg cena;      //-- Counter enable (cuando cena = 1)
reg start;  //-- Transmitir cadena (cuando transmit = 1)


//------------------------------------------------
//-- 	RUTA DE DATOS
//------------------------------------------------

//-- Inicializador
always @(posedge clk)
  rstn <= 1;

//-- Instanciar la Unidad de transmision
uart_tx #(.BAUD(BAUD))
  TX0 (
    .clk(clk),
    .rstn(rstn),
    .data(data_r),
    .start(start),
    .ready(ready),
    .tx(tx)
  );

//-- Multiplexor con los caracteres de la cadena a transmitir
//-- se seleccionan mediante la señal car_count
always @*
  case (car_count)
    8'd0: data <= "H";
    8'd1: data <= "o";
    8'd2: data <= "l";
    8'd3: data <= "a";
    8'd4: data <= "!";
    8'd5: data <= ".";
    8'd6: data <= ".";
    8'd7: data <= ".";
    default: data <= ".";
  endcase

//-- Registrar los datos de salida del multiplexor
always @*
  data_r <= data;

//-- Contador de caracteres
//-- Cuando la microorden cena esta activada, se incrementa
reg [2:0] car_count;

always @(posedge clk)
  if (rstn == 0)
    car_count = 0;
  else if (cena)
    car_count = car_count + 1;

//-- Registrar señal dtr para cumplir con normas diseño sincrono
always @(posedge clk)
  transmit_r <= transmit;

//---------------------------
//--  Temporizador
//---------------------------
dividerp1 #(.M(DELAY))
  DIV0 ( .clk(clk),
         .clk_out(transmit)
       );

//----------------------------------------------------
//-- CONTROLADOR
//----------------------------------------------------
localparam IDLE = 0;   //-- Reposo
localparam TXCAR = 2'd1;  //-- Transmitiendo caracter
localparam NEXT = 2'd2;   //-- Preparar transmision del sig caracter
localparam END = 3;    //-- Terminar

//-- Registro de estado del automata
reg [1:0] state;

//-- Gestionar el cambio de estado
always @(posedge clk)

  if (rstn == 0)
    //-- Ir al estado inicial
    state <= IDLE;

  else
    case (state)
      //-- Estado inicial. Se sale de este estado al recibirse la
      //-- señal de transmit_r, conectada al temporizador de 1s
      IDLE: 
        if (transmit_r == 1) state <= TXCAR;
        else state <= IDLE;

      //-- Estado de transmision de un caracter. Esperar a que el 
      //-- transmisor serie este disponible. Cuando lo esta se pasa al
      //-- siguiente estado
      TXCAR: 
        if (ready == 1) state <= NEXT;
        else state <= TXCAR;

      //-- Envio del siguiente caracter. Es un estado transitorio
      //-- Cuando se llega al ultimo caracter se pasa para finalizar
      //-- la transmision 
      NEXT:	
        if (car_count == 7) state <= END;
        else state <= TXCAR;

      //-- Ultimo estado:finalizacion de la transmision. Se espera hasta
      //-- que se haya enviado el ultimo caracter. Cuando ocurre se vuelve
      //-- al estado de reposo inicial
      END: 
        //--Esperar a que se termine ultimo caracter
        if (ready == 1) state <= IDLE;
        else state <= END;

      //-- Necesario para evitar latches
      default:
         state <= IDLE;

    endcase

//-- Generacion de las microordenes
always @*
  case (state)
    IDLE: begin
      start <= 0;
      cena <= 0;
    end

    TXCAR: begin
      start <= 1;
      cena <= 0;
    end

    NEXT: begin
      start <= 0;
      cena <= 1;
    end

    END: begin
      start <= 0;
      cena <= 0;
    end

    default: begin
      start <= 0;
      cena <= 0;
    end
  endcase

endmodule
```

## Simulación

El banco de pruebas sólo tiene que generar el reloj del sistema y esperar un cierto tiempo para completar la simulación

```verilog
//-- Fichero: scicad2_tb.v
`include "baudgen.vh"


module scicad2_tb();

//-- Baudios con los que realizar la simulacion
localparam BAUD = `B115200;
localparam DELAY = 10000;

//-- Tics de reloj para envio de datos a esa velocidad
//-- Se multiplica por 2 porque el periodo del reloj es de 2 unidades
localparam BITRATE = (BAUD << 1);

//-- Tics necesarios para enviar una trama serie completa, mas un bit adicional
localparam FRAME = (BITRATE * 11);

//-- Tiempo entre dos bits enviados
localparam FRAME_WAIT = (BITRATE * 4);

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Linea de tranmision
wire tx;

//-- Instanciar el componente
scicad2 #(.BAUD(BAUD), .DELAY(DELAY))
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
  $dumpfile("scicad2_tb.vcd");
  $dumpvars(0, scicad2_tb);

  #(FRAME * 20) $display("FIN de la simulacion");
  $finish;
end

endmodule
```
Para simular ejecutamos el comando:

    $ make sim2

En este pantallazo del gtkwave se muestra la transmisión periódica de la cadena:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad2-gtkterm.png)

Se observa cómo la señal transmit se pone a 1 periódicamente, transmitiéndose la cadena

## Síntesis y pruebas
Hacemos la síntesis con el siguiente comando:

    $ make sint2

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 8 / 96
|PLBs      | 37 / 160
|BRAMs     | 0 / 16

y lo cargamos en la FPGA con:

    $ sudo iceprog scicad2.bin

Abrimos el **gtkterm** y lo configuramos a **115200 baudios**. Aparecerá la cadena "**Hola!...**" cada segundo

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T24-uart-tx/images/scicad2-sim.png)

En este **vídeo de youtube** se puede ver el ejemplo en acción:

[![Click to see the youtube video](http://img.youtube.com/vi/gKz9vYfRkdI/0.jpg)](https://www.youtube.com/watch?v=gKz9vYfRkdI)

# Ejercicios propuestos
* Modificar cualquiera de los dos ejemplos para enviar la cadena "Hola como estas "

# Conclusiones
TODO