![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-3.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T24-uart-tx)

# Introducción
**Completaremos** la unidad de transmisión serie asíncrona y haremos dos **ejemplos de uso**, que envían la cadena "Hola!...". Uno de manera contínua al activarse la señal de DTR y otro cada segundo

Se trata de la **última versión de la unidad de transmisión**, que quedará lista para usar en nuestros proyectos

# Módulo UART-TX

La unidad de transmisión serie la encapsularemos dentro del **módulo uart-tx**

## Descripción de la interfaz

La unidad de transmisión tiene 4 entradas y 2 salidas:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-2.png)

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

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-4.png)

## Diagrama de bloques

El diagrama de bloques del transmisor se muestra en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-1.png)

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

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-3.png)

### Controlador

El controlador tiene las siguientes **entradas**:
* **car_count**: Contador de caracteres. Es la salida del contador de 3 bits. Indica el carácter que se está enviando
* **ready**: Señal proveniente de la unidad de transmisión para saber cuándo está lista para enviar el siguiente caracter
* **transmit**: Señal proveniente del exterior del circuito para indicar que comience la transmisión de la cadena. Está conectada a la señal DTR que controla el usuario

Las **microórdenes** que genera son:
* **start**: Para comenzar la transmisión de un carácter
* **cena**: Counter enable. Activar el contador para acceder al siguiente caracter a enviar

El comportamiento del controlador queda definido por esta máquina de estados:

(Diagrama máquina estados)

## scicad1.v: Descripción en verilog
## Simulación
## Síntesis y pruebas

# Ejemplo 2: Enviando una cadena cada segundo
## scicad2.v: Descripción en verilog
## Simulación
## Síntesis y pruebas

# Ejercicios propuestos
* Modificar cualquiera de los dos ejemplos para enviar la cadena "Hola como estas "

# Conclusiones
TODO