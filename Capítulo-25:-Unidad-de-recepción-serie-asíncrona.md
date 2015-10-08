![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T25-uart-rx)

# Introducción
Diseñaremos una **unidad de recepción serie asíncrona**, que nos permita recibir datos tanto del PC como de otro dispositivo de comunicación serie. Obtendremos el componente final, listo para usar en nuestros diseños

Para probarla haremos dos ejemplos: uno que **saca los 4 bits menos segnificativos** del dato recibido por **los leds** de la ICEStick. Y otro que **hace eco de todos los caracteres recibidos**. Los caracteres recibidos se envían al transmisor serie para que lleguen de nuevo al PC. Esto nos permitirá comprobar que efectivamente hemos recibido el dato correctamente.

# Módulo UART-RX

La unidad de transmisión serie la encapsularemos dentro del **módulo uart-rx**

## Descripción de la interfaz

La unidad tiene 3 entradas y 2 salidas:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-1.png)

* **Entradas**:
    * **clk**: Reloj del sistema (12MHz en la ICEstick)
    * **rstn**: Reset negado. Cuando rstn es 0, se hace un reset síncrono de la unidad de transmisión
    * **rx**: Linea de recepción serie. Por donde llegan los datos en serie

* **Salidas**:
    * **rcv**: Notificación de carácter recibido. Es un pulso de 1 ciclo de ancho
    * **data**: Dato de recibido (8 bits)

## Cronograma

Inicialmente, cuando **la línea está en reposo** y no se ha recibido nada, la señal **rcv está a 0**. Al recibirse el **bit de start** por rx, el receptor comienza a funcionar, leyendo los siguientes bits y almacenándolos internametne en su registro de desplazamiento.  En el **instante t1**, cuando se ha recibido el **bit de stop**, **el dato se captura** y se saca por la **salida data**. En el siguiente ciclo de reloj, **instante t2** (en el cronograma el tiempo se ha exagerado para que se aprecie), aparece un pulso de un ciclo de reloj de anchura (exagerado también en el dibujo) que permita **capturar el dato en un registro externo**.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-3.png)

**Esta interfaz es muy cómoda**. Para usar uart-rx en nuestros diseños, sólo hay que conectar la salida _data_ a la entrada de datos de un **registro** y la señal _rcv_ usarla como **habilitación**. El dato recibido se capturará automáticamente. Esta señal rcv también la podemos usar en los controladores para saber cuándo se ha **recibido un dato nuevo**.

## Diagrama de bloques

El diagrama de bloques completo del receptor se muestra en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-2.png)

### Ruta de datos

La **señal rx se registra**, para cumplir con las normas de diseño síncrono, y se introduce por el bit más significativo de un **registro de desplazamiento** de 10 bits. El desplazamiento se realiza cuando llega un pulso por la señal **clk_baud**, proveniente del **generador de baudios**. Este generador sólo funciona cuando la miroorden **bauden** está activada.

Un **contador de 4 bits** realiza la cuenta de los bits recibidos (cuenta cada pulso de clk_baud). Se pone a 0 con la microórden **clear**

Por último tenemos el **controlador**, que genera las microórdenes **baudgen**, **load**, **clear** y la señal de interfaz **rcv**. La señal load se activa para que el dato recibido se almacene en el **registro de datos de 8 bits**, de manera que se mantenga estable durante la recepción del siguiente carácter

### baudgen_rx: Generador de baudios para recepción

El receptor tiene su **propio generador de baudios** que es **diferente al del transmisor**. En el transmisor, al activar su generador con la microorden bauden, emite inmediatamente un pulso. Sin embargo, en el receptor, se emite **en la mitad del periodo**. De esta forma se garantiza que el dato se lee en la mitad del periodo, donde es **mucho más estable** (y la probabilidad de error es mejor)

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-4.png)

En el cronograma se puede ver cómo al recebir el **flanco de bajada** del bit de start **bauden se activa**, para que comience a funcionar el reloj del receptor. Sin embargo, hasta que **no ha alcanzado la mitad del periodo de bit no se pone a 1**. A partir de entonces, los pulsos coinciden con la mitad de los periodos de los bits recibidos 

### Controlador

El controlador está modelado como una máquina de **4 estados**:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-5.png)

Los estados son:

* **IDLE**: Estado de reposo. Esperando a recibir el bit de start por rx.  En cuanto se recibe se pasa al siguiente estado
* **RCV**: Recibiendo datos. Se activa **el temporizador de bits** mediante la microorden **baudgen** y se van recibiendo todos los bits, que se almacenan en **el registro de desplazamiento**. Cuando se han recibido 10 bits (1 de start + 8 de datos + 1 de stop) la salida del **contador** (**bitc**) estará a 10 y se pasa al siguiente estado
* **LOAD**: Almacenamiento del dato recibido. Se activa la microorden **load** para guardar el dato recibido (8 bits) en el **registro de datos**
* **DAV**: (Data AVailable). Señalización de que existe un **dato disponible**. Se pone a uno la señal **rcv** para que los circuitos externos puedan capturar el dato

## Descripción en verilog

La **unidad de recepción** se compone de dos ficheros: el generador de baudios de recepción (_baudgen_rx.v_) y el propio receptor (_uart_rx.v_)

### baudgen_rx.v

Este componente es similar al generador de baudios del transmisor, pero una vez habilitado el pulso se envía transcurrida **la mitad del periodo**

El código verilog es el siguiente:

```verilog
//-- Fichero: baudgen_rx.v
`include "baudgen.vh"

//-- ENTRADAS:
//--     -clk: Senal de reloj del sistema (12 MHZ en la iceStick)
//--     -clk_ena: Habilitacion. 
//--            1. funcionamiento normal. Emitiendo pulsos
//--            0: Inicializado y parado. No se emiten pulsos
//
//-- SALIDAS:
//--     - clk_out. Señal de salida para lograr la velocidad en baudios indicada
//--                Anchura de 1 periodo de clk. SALIDA NO REGISTRADA
module baudgen_rx(input wire clk,
               input wire clk_ena, 
               output wire clk_out);

//-- Valor por defecto de la velocidad en baudios
parameter M = `B115200;

//-- Numero de bits para almacenar el divisor de baudios
localparam N = $clog2(M);

//-- Valor para llegar a la mitad del periodo
localparam M2 = (M >> 1);

//-- Registro para implementar el contador modulo M
reg [N-1:0] divcounter = 0;

//-- Contador módulo M
always @(posedge clk)

  if (clk_ena)
    //-- Funcionamiento normal
    divcounter <= (divcounter == M - 1) ? 0 : divcounter + 1;
  else
    //-- Contador "congelado" al valor maximo
    divcounter <= M - 1;

//-- Sacar un pulso de anchura 1 ciclo de reloj si el generador
//-- esta habilitado (clk_ena == 1)
//-- en caso contrario se saca 0
assign clk_out = (divcounter == M2) ? clk_ena : 0;

endmodule
```

### uart_rx.v

Descripción en verilog de la unidad de recepción:

```verilog
//-- Fichero: uart_rx.v
`default_nettype none

`include "baudgen.vh"

module uart_rx (input wire clk,         //-- Reloj del sistema
                input wire rstn,        //-- Reset
                input wire rx,          //-- Linea de recepcion serie
                output reg rcv,         //-- Indicar Dato disponible
                output reg [7:0] data); //-- Dato recibo

//-- Parametro: velocidad de recepcion
parameter BAUD = `B115200;

//-- Reloj para la recepcion
wire clk_baud;

//-- Linea de recepcion registrada
//-- Para cumplir reglas de diseño sincrono
reg rx_r;

//-- Microordenes
reg bauden;  //-- Activar señal de reloj de datos
reg clear;   //-- Poner a cero contador de bits
reg load;    //-- Cargar dato recibido


//-------------------------------------------------------------------
//--     RUTA DE DATOS
//-------------------------------------------------------------------

//-- Registrar la señal de recepcion de datos
//-- Para cumplir con las normas de diseño sincrono
always @(posedge clk)
  rx_r <= rx;

//-- Divisor para la generacion del reloj de llegada de datos 
baudgen_rx #(BAUD)
  baudgen0 ( 
    .clk(clk),
    .clk_ena(bauden), 
    .clk_out(clk_baud)
  );

//-- Contador de bits
reg [3:0] bitc;

always @(posedge clk)
  if (clear)
    bitc <= 4'd0;
  else if (clear == 0 && clk_baud == 1)
    bitc <= bitc + 1;


//-- Registro de desplazamiento para almacenar los bits recibidos
reg [9:0] raw_data;

always @(posedge clk)
  if (clk_baud == 1) begin
    raw_data = {rx_r, raw_data[9:1]};
  end

//-- Registro de datos. Almacenar el dato recibido
always @(posedge clk)
  if (rstn == 0)
    data <= 0;
  else if (load)
    data <= raw_data[8:1];

//-------------------------------------
//-- CONTROLADOR
//-------------------------------------
localparam IDLE = 2'd0;  //-- Estado de reposo
localparam RECV = 2'd1;  //-- Recibiendo datos
localparam LOAD = 2'd2;  //-- Almacenamiento del dato recibido
localparam DAV = 2'd3;   //-- Señalizar dato disponible 

reg [1:0] state;

//-- Transiciones entre estados
always @(posedge clk)

  if (rstn == 0)
    state <= IDLE;

  else
    case (state)

      //-- Resposo
      IDLE : 
        //-- Al llegar el bit de start se pasa al estado siguiente
        if (rx_r == 0)  
          state <= RECV;
        else
          state <= IDLE;

      //--- Recibiendo datos      
      RECV:
        //-- Vamos por el ultimo bit: pasar al siguiente estado
        if (bitc == 4'd10)
          state <= LOAD;
        else
          state <= RECV;

      //-- Almacenamiento del dato
      LOAD:
        state <= DAV;

      //-- Señalizar dato disponible
      DAV:
        state <= IDLE;

    default:
      state <= IDLE;
    endcase


//-- Salidas de microordenes
always @* begin
  bauden <= (state == RECV) ? 1 : 0;
  clear <= (state == IDLE) ? 1 : 0;
  load <= (state == LOAD) ? 1 : 0;
  rcv <= (state == DAV) ? 1 : 0;
end


endmodule
```

# Ejemplo 1: Visualizando en los leds

Como primer ejemplo de uso de la **unidad de recepción**, haremos un circuito que **captura en un registro el dato recibido** por el puerto serie, a la **velocidad de 115200 baudios** y sus 4 bits menos significativos los saca por los **leds rojos** de la placa ICEstick

La **línea rx** se envía directamente al **led verde** para ver la actividad (se envía negada para que normalmente esté el led apagado, y cuando se reciban caracteres parpadee)

## rxleds.v: Descripción 

El Diagrama de bloques del circuito se muestra a continuación:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/rxleds-1.png)

Simplemente **se instancia la unidad de recepción** y se colocan un **inicializador** para hacer el reset y **un registro** para capturar el dato recibido. Este registro tiene un **enable** para capturar cuando se reciba el dato (indicándose por la señal rcv)

Descripción del código en verilog:

```verilog
`default_nettype none

`include "baudgen.vh"

//-- Top design
module rxleds(input wire clk,         //-- Reloj del sistema
              input wire rx,          //-- Linea de recepcion serie
              output reg [3:0] leds,  //-- 4 leds rojos
              output wire act);       //-- Led de actividad (verde)

//-- Parametro: Velocidad de transmision
localparam BAUD = `B115200;

//-- Señal de dato recibido
wire rcv;

//-- Datos recibidos
wire [7:0] data;

//-- Señal de reset
reg rstn = 0;

//-- Inicializador
always @(posedge clk)
  rstn <= 1;

//-- Instanciar la unidad de recepcion
uart_rx #(BAUD)
  RX0 (.clk(clk),      //-- Reloj del sistema
       .rstn(rstn),    //-- Señal de reset
       .rx(rx),        //-- Linea de recepción de datos serie
       .rcv(rcv),      //-- Señal de dato recibido
       .data(data)     //-- Datos recibidos
      );

//-- Sacar los 4 bits menos significativos del dato recibido por los leds
//-- El dato se registra
always @(posedge clk)
    //-- Capturar el dato cuando se reciba
    if (rcv == 1'b1)
      leds <= data[3:0]; 

//-- Led de actividad
//-- La linea de recepcion negada se saca por el led verde
assign act = ~rx;

endmodule
```

## Simulación
El banco de pruebas genera el reloj del sistema y envía dos caracteres en serie.  Para facilitar el envío y poder realizar bancos de pruebas más elaborados, que puedan enviar mayor cantidad de datos, se ha creado la **tarea _send_car_**.  En Verilog, las tareas son como las funciones en C (pero sin devolver ningún valor). Permiten reutilizar el código y hacer que los bancos de prueba sean más sencillos y legibles

El código de **send_car** es:

```verilog
task send_car;
    input [7:0] car;
  begin
    rx <= 0;                 //-- Bit start 
    #BITRATE rx <= car[0];   //-- Bit 0
    #BITRATE rx <= car[1];   //-- Bit 1
    #BITRATE rx <= car[2];   //-- Bit 2
    #BITRATE rx <= car[3];   //-- Bit 3
    #BITRATE rx <= car[4];   //-- Bit 4
    #BITRATE rx <= car[5];   //-- Bit 5
    #BITRATE rx <= car[6];   //-- Bit 6
    #BITRATE rx <= car[7];   //-- Bit 7
    #BITRATE rx <= 1;        //-- Bit stop
    #BITRATE rx <= 1;        //-- Esperar a que se envie bit de stop
  end
  endtask
```

Recibe como **argumento** un **carácter de 8 bits** y saca por la señal rx la trama serie, comenzando por el bit de start, luego los de datos y finalmente el bit de stop. Se hace a la velocidad indicada por el parametro BITRATE que se define en función de la velocidad en baudios.

Para enviar un caracter sólo hay que **invocar a la tarea** y pasar este caracter como parámetro. Por ejemplo, para enviar una "A" en ascii sólo habría que ejecutar:

senc_car("A");

El código completo del banco de pruebas es el siguiente:

```verilog
//-- Fichero rxleds_tb.v
`include "baudgen.vh"

module rxleds_tb();

localparam BAUD = `B115200;

//-- Tics de reloj para envio de datos a esa velocidad
//-- Se multiplica por 2 porque el periodo del reloj es de 2 unidades
localparam BITRATE = (BAUD << 1);

//-- Tics necesarios para enviar una trama serie completa, mas un bit adicional
localparam FRAME = (BITRATE * 10);

//-- Tiempo entre dos bits enviados
localparam FRAME_WAIT = (BITRATE * 4);

//----------------------------------------
//-- Tarea para enviar caracteres serie  
//----------------------------------------
  task send_car;
    input [7:0] car;
  begin
    rx <= 0;                 //-- Bit start 
    #BITRATE rx <= car[0];   //-- Bit 0
    #BITRATE rx <= car[1];   //-- Bit 1
    #BITRATE rx <= car[2];   //-- Bit 2
    #BITRATE rx <= car[3];   //-- Bit 3
    #BITRATE rx <= car[4];   //-- Bit 4
    #BITRATE rx <= car[5];   //-- Bit 5
    #BITRATE rx <= car[6];   //-- Bit 6
    #BITRATE rx <= car[7];   //-- Bit 7
    #BITRATE rx <= 1;        //-- Bit stop
    #BITRATE rx <= 1;        //-- Esperar a que se envie bit de stop
  end
  endtask

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Cables para las pruebas
reg rx = 1;
wire act;
wire [3:0] leds;

//-- Instanciar el modulo rxleds
rxleds #(BAUD)
  dut(
    .clk(clk),
    .rx(rx),
    .act(act),
    .leds(leds)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("rxleds_tb.vcd");
  $dumpvars(0, rxleds_tb);

  //-- Enviar datos de prueba
  #BITRATE    send_car(8'h55);
  #FRAME_WAIT send_car("K");

  #(FRAME_WAIT*4) $display("FIN de la simulacion");
  $finish;
end
```
Se envían dos caracteres de prueba. Primero el 0x55, que en binario es 01010101 (se alternan los ceros y los unos). El segundo es el carácter K


Para realizar la simulación ejecutamos el comando:

    $ make sim

Y el resultado en gtkwave es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/rxleds-gtkwave.png)

La simulación comienza con la **línea rx en reposo** (a 1) y a continuación se envía el **valor 0x55** en serie al receptor. Cuando se ha terminado se ve en la simulación cómo aparece un **pulso en la señal rcv** y cómo el **valor 0x55** se guarda en el **registro de datos** (_data_). Además, por los leds se sacan los 4 bits menos significativos (que tienen el valor 5 en hexadecimal). 

A continuación, pasado un tiempo, se envía el carácter (K), y comprobamos que se recibe correctamente

## Síntesis y pruebas

Hacemos la síntesis con el siguiente comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 7 / 96
|PLBs      | 25 / 160
|BRAMs     | 0 / 16

y lo cargamos en la FPGA con:

    $ sudo iceprog rxleds.bin

Abrimos el **gtkterm** y lo configuramos a **115200 baudios**. Si pulsamos teclas, veremos cómo cambian los leds de la ICEstick. Por ejemplo, si pulsamos el 7, se envía el número 0x37, cuyos 4 bits menos significativos coinciden con el número 7 (el código ASCII se diseño adrede para cumplir con esta propiedad). En binario es 0111.  Veremos cómo se encienden los leds 0, 1 y 3.   

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/rxleds-test.png)

Si ahora pulsamos la tecla 0, todos los leds estarán apagados

En este **vídeo de youtube** se puede ver el ejemplo en acción:

[![Click to see the youtube video](http://img.youtube.com/vi//0.jpg)](https://www.youtube.com/watch?v=)



# Ejemplo 2: eco
## Descripción
## Simulación
## Síntesis y pruebas

# Ejercicios propuestos
# Conclusiones
TODO
