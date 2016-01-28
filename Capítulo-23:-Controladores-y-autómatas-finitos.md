![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T23-fsmtx)

# Introducción
Los **controladores** son la parte de los circuitos digitales que se **encargan de generar todas las señales** (microórdenes) **que gobiernan  el resto de componentes**. Se implementan mediante **autómatas finitos** (también denominadas **máquinas de estados**). En este capítulo diseñaremos un controlador para nuestro **transmisor serie**

# Arquitectura genérica

La **estructura** de los circuitos digitales la podemos descomponer en dos partes:  la **ruta de datos** y el **controlador**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx-3.png)

* **Ruta de datos**: Es la parte que **procesa los datos**. Comprende todas las unidades funciones junto con sus conexiones para realizar ese procesamiento: registros de desplazamiento, unidades aritmético-lógicas, contadores, etc.

* **Controlador**: Es la **parte de gobierno**, que genera las señales necesarias, denominadas **microórdenes**, para que la ruta de datos se comporte adecuadamente. Envía microórdenes para poner en marcha un contador, cargar un dato en un registro, realizar tal operación aritmética, etc. Todo ello secuenciado en el tiempo.

El controlador **toma decisiones** y genera unas microórdenes u otras en función de los resultados parciales de la ruta de datos. Por ejemplo, si un contador ha alcanzado un determinado valor, entonces se deberá general tal señal para hacer tal cosa.  Todo eso lo hace el controlador.

Los controladores se implementan mediante [autómatas finitos](https://es.wikipedia.org/wiki/Aut%C3%B3mata_finito) (también denominados máquinas de estados). Es necesario primero definir **los estados** del circuito y **las transiciones** entre estos estados según las condiciones del circuito. Luego, en función de estos estados se generan las **microórdenes**. 

# Máquinas de estado en verilog

Partimos de este **diagrama de 4 estados**, para explicar cómo describirlo en Verilog

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx-4.png)

**Inicialmente** el circuito se encuentra en el **estado 0** y el controlador generará las microórdenes necesarias (no mostradas en el dibujo). Mientras que la **señal a** esté a 0, se mantendrá siempre en ese estado. En cuanto se ponga a 1 se pasará al **estado 1**, donde se generarán otras microórdenes. En este estado, según el valor de la **señal b**, bien se volverá al estado inicial o se avanzará al **estado 2**. El estado 2 **no tiene condiciones**, por lo que **en el siguiente ciclo de reloj** se pasa al **estado 3**. Cuando la **señal c** valga 0, se vuelve al estado inicial.

Primero definimos las **etiquetas para los estados** mediante **parámetros locales**, y un **registro** para **almacenar el estado**. Como tenemos 4 estados, este registro será de **2 bits**:

```verilog
localparam ESTADO0 = 0;
localparam ESTADO1 = 1;
localparam ESTADO2 = 2;
localparam ESTADO3 = 3;

reg [1:0] state;
```
Las **transiciones** entre estados las modelamos mediante un **proceso que depende del reloj** y que al recibir el **reset** se inicializa en el **estado inicial**. Usando un _**case**_ definimos las transiciones para cada estado:

```verilog
always @(posedge clk)
  if (rst)
    //-- Establecer el estado inicial al arrancar
    state <= ESTADO0;
  else 
    case (state)
      ESTADO0:
        //-- Cambio de estado condicional
        if (a) state <= ESTADO1;
        else state <= ESTADO0;

      ESTADO1:
        //-- Cabmio de estado condidiconal
        if (b) state <= ESTADO2;
        else state <= ESTADO1;

      ESTADO2:
        //-- Cambiar de estado en el próximo ciclo
        state <= ESTADO3;

      ESTADO3:
        //-- Cabmio de estado condidiconal
        if (c) state <= ESTADO0;
        else state <= ESTADO3;

      default:
        //-- Siempre hay que ponerlo, para evitar
        //-- latches en la síntesis
        state <= ESTADO0;

    endcase
```
En todos los estados salvo el ESTADO2 las **transiciones son condicionales**: según el estado de las señales a,b y c se salta a un estado o a otro.  **El ESTADO2 sólo dura 1 ciclo**. En cuanto se entra en él, se salta al ESTADO3 **incondicionalmente**, en el siguiente ciclo de reloj.

Ahora falta la parte de **generación de las microórdenes**. Supondremos que este circuito tiene 2 microórdenes: **m1** y **m2**. Dependiendo del estado en el que se encuentre, se asignarán diferentes valores. Lo implementamos como un **proceso combinacional** con un **case**:

```verilog
always @*
  case (state)
    ESTADO0: begin
      m1 <= 0;
      m2 <= 0;
    end

    ESTADO1: begin
      m1 <= 1;
      m2 <= 0;
    end

    ESTADO2: begin
      m1 <= 0;
      m2 <= 0;
    end

    ESTADO3: begin
      m1 <= 1;
      m2 <= 1;
    end

    default: begin
      m1 <= 0;
      m2 <= 0;
    end
  endcase
```
Usar el **case** es la manera directa, pero hay que **definir el valor de todas las microórdenes para cada estado**. Mediante **assignaciones directas** se puede reducir el código sustancialmente. En el ejemplo anterior, vemos que la microórden **m2** sólo se activa en el ESTADO3 y en el resto está desactivada. Para **m1** vemos que se activa en los estados ESTADO1 y ESTADO3. Esto lo podemos describir así:

```verilog
assign m1 = (state == ESTADO1 || state == ESTADO3) ? 1 : 0;
assign m2 = (state == ESTADO3) ? 1 : 0;
```
El código queda mucho más compacto.

# Transmisor serie mejorado

Aplicaremos esta arquitectura al transmisor serie, dividiéndolo en su ruta de datos y el controlador 

## fsmtx.v: Transmisión continua
Haremos un transmisor que envíe el **carácter A** cuando se ponga a 1 la **señal de entrada _start_**, que la controlaremos a través de la **señal dtr** (como en los ejemplos del capítulo anterior). Sin embargo, esta señal no controla directamente la transmisión, sino que **sólo la inicia**. Será el controlador el que controle esta transmisión. De esta manera, **nunca se quedará un carácter a medio transmitir** como pasaba en el capítulo anterior.

 El esquema es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx-1.png)

### Ruta de datos
En la **ruta de datos** se encuentra el **registro de desplazamiento** que hace la serialización de los datos, el **generador de baudios**, el inicializador, un **contador de 4 bits** para llevar la cuenta de bits enviados y los **flip-flops** para registrar señales y cumplir con las normas de diseño síncrono

El contador lleva la cuenta de **cuántos bits se han enviado** y servirá para que el controlador sepa cuándo ha finalizado la transmisión. La señal de "load" sirve para **ponerlo a cero**, y es la misma que se usa para realizar la carga del registro de desplazamiento

### Controlador

El **diagrama de estados** del controlador es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx-2.png)

El transmisor puede estar en **3 estados**:

* **IDLE**: Estado de reposo. No se está transmitiendo nada. Se espera a que se active la señal de start para comenzar con la transmisión del carácter "A"

* **START**: Comienzo de transmisión. Se carga el dato en el registro de desplazamiento. Se arranca el reloj para la temporización de los bits a la velocidad configurada. Se pone el contador de bits a cero. Es un estado transitorio que dura 1 ciclo de reloj

* **TRANS**: Estado de transmisión. Permanece en este estado durante la trasmisión del carácter. En cuanto el contador de bits alcanza el valor 11 (10 bits ya transmitidos) se pasa al estado inicial de reposo

Para controlar la ruta de datos se necesitan **2 microórdenes**, _load_ y _baudgen_. La se usa para cargar el dato en el registro de desplazamiento y poner a cero el contador de bits. La segunda es la habilitación del temporizador de bits (generador de baudios) 

### fsmtx.v: Descripción del hardware

La descripción completa del transmisor mejorado, en Verilog, es:

```verilog
//-- Fichero fsmtx.v
`default_nettype none

`include "baudgen.vh"

//--- Modulo que envia un caracter cuando load esta a 1
//--- La salida tx ESTA REGISTRADA
module fsmtx (input wire clk,       //-- Reloj del sistema (12MHz en ICEstick)
              input wire start,     //-- Activar a 1 para transmitir
              output reg tx         //-- Salida de datos serie (hacia el PC)
             );

//-- Parametro: velocidad de transmision
//-- Pruebas del caso peor: a 300 baudios
parameter BAUD =  `B300;

//-- Caracter a enviar
parameter CAR = "A";

//-- Registro de 10 bits para almacenar la trama a enviar:
//-- 1 bit start + 8 bits datos + 1 bit stop
reg [9:0] shifter;

//-- Señal de start registrada
reg start_r; 

//-- Reloj para la transmision
wire clk_baud;

//-- Reset
reg rstn = 0;

//-- Bitcounter
reg [3:0] bitc;

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

//-- Registro de desplazamiento, con carga paralela
//-- Cuando load_r es 0, se carga la trama
//-- Cuando load_r es 1 y el reloj de baudios esta a 1 se desplaza hacia
//-- la derecha, enviando el siguiente bit 
//-- Se introducen '1's por la izquierda
always @(posedge clk)
  //-- Reset
  if (rstn == 0)
    shifter <= 10'b11_1111_1111;

  //-- Modo carga
  else if (load == 1)
    shifter <= {CAR,2'b01};

  //-- Modo desplazamiento
  else if (load == 0 && clk_baud == 1)
    shifter <= {1'b1, shifter[9:1]};

always @(posedge clk)
  if (load == 1)
    bitc <= 0;
  else if (load == 0 && clk_baud == 1)
    bitc <= bitc + 1;

//-- Sacar por tx el bit menos significativo del registros de desplazamiento
//-- Cuando estamos en modo carga (load_r == 0), se saca siempre un 1 para 
//-- que la linea este siempre a un estado de reposo. De esta forma en el 
//-- inicio tx esta en reposo, aunque el valor del registro de desplazamiento
//-- sea desconocido
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
localparam IDLE = 0;
localparam START = 1;
localparam TRANS = 2;

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


//-- Inicializador
always @(posedge clk)
  rstn <= 1;

endmodule
```
### Simulación

El banco de pruebas es este:

```verilog
//-- Fichero: fsmtx.v
`include "baudgen.vh"

module fsmtx_tb();

//-- Baudios con los que realizar la simulacion
//-- A 300 baudios, la simulacion tarda mas en realizarse porque los
//-- tiempos son mas largos. A 115200 baudios la simulacion es mucho
//-- mas rapida
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

//-- Simulacion de la señal start
reg start = 0;

//-- Instanciar el componente
fsmtx #(.BAUD(BAUD))
  dut(
    .clk(clk),
    .start(start),
    .tx(tx)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("fsmtx_tb.vcd");
  $dumpvars(0, fsmtx_tb);

  #1 start <= 0;

  //-- Enviar primer caracter
  #FRAME_WAIT start <= 1;
  #(BITRATE * 2) start <=0;

  //-- Segundo envio (2 caracteres mas)
  #(FRAME_WAIT * 2) start <=1;
  #(FRAME * 1) start <=0;

  #(FRAME_WAIT * 4) $display("FIN de la simulacion");
  $finish;
end

endmodule
```

La simulación se realiza con:

    $ make sim

y el resultado en gtkwave es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx-sim-1.png)

La línea superior es la de **start**. Observamos cómo esta señal se pone a 0 antes de que termine el primer carácter, pero se  sigue enviando. Al terminar, se vuelve a poner a 1 para enviar el siguiente. Esta vez se deja subido más tiempo, de forma que se envían 2 caracteres más (En total se envían 30 bits, por lo que hay 30 pulsos de la señal _clk_baud_).

En las línea inferiores se ve el contador de bits y los estados por los que pasa el controlador

### Síntesis y pruebas

Hacemos la síntesis con el siguiente comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 6 / 96
|PLBs      | 21 / 160
|BRAMs     | 0 / 16

y lo cargamos en la FPGA con:

    $ sudo iceprog fsmtx.bin

Desde el **gtkterm**, cada vez que le damos al **F7** para modificar la **señal DTR** se empezarán a enviar los **caracteres A** (por defecto a la velocidad de 300 baudios). Si ahora dejamos pulsada F7 o la apretamos aleatoriamente, veremos que **no aparecen caracteres basura**, porque el controlador nos garantiza que siempre se envíe el caracter

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx-gtkterm-1.png)

## fsmtx2.v: Transmisión temporizada
Este circuito **transmite periódicamente el carácter "A" cada 100ms**. El circuito es similar al del ejemplo anterior pero la señal de start se toma de un divisor de 100ms en vez de la señal externa DTR

Para que solo se envíe 1 caracter cada vez, el **divisor** está modificado para generar **un pulso de 1 ciclo de anchura**.

El esquema del circuito es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx2-1.png)

y la descripción en Verilog:

```verilog
//-- Fichero: fsmtx2.v
`default_nettype none

`include "baudgen.vh"
`include "divider.vh"

//--- Modulo que envia un caracter cuando load esta a 1
//--- La salida tx ESTA REGISTRADA
module fsmtx2 (input wire clk,       //-- Reloj del sistema (12MHz en ICEstick)
              output reg tx         //-- Salida de datos serie (hacia el PC)
             );

//-- Parametro: velocidad de transmision
parameter BAUD =  `B115200;

//-- Caracter a enviar
parameter CAR = "A";

//- Tiempo de envio
parameter DELAY = `T_100ms;

//-- Registro de 10 bits para almacenar la trama a enviar:
//-- 1 bit start + 8 bits datos + 1 bit stop
reg [9:0] shifter;

wire start;

//-- Señal de start registrada
reg start_r; 

//-- Reloj para la transmision
wire clk_baud;

//-- Reset
reg rstn = 0;

//-- Bitcounter
reg [3:0] bitc;

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

//-- Registro de desplazamiento, con carga paralela
//-- Cuando load_r es 0, se carga la trama
//-- Cuando load_r es 1 y el reloj de baudios esta a 1 se desplaza hacia
//-- la derecha, enviando el siguiente bit 
//-- Se introducen '1's por la izquierda
always @(posedge clk)
  //-- Reset
  if (rstn == 0)
    shifter <= 10'b11_1111_1111;

  //-- Modo carga
  else if (load == 1)
    shifter <= {CAR,2'b01};

  //-- Modo desplazamiento
  else if (load == 0 && clk_baud == 1)
    shifter <= {1'b1, shifter[9:1]};

always @(posedge clk)
  if (load == 1)
    bitc <= 0;
  else if (load == 0 && clk_baud == 1)
    bitc <= bitc + 1;

//-- Sacar por tx el bit menos significativo del registros de desplazamiento
//-- Cuando estamos en modo carga (load_r == 0), se saca siempre un 1 para 
//-- que la linea este siempre a un estado de reposo. De esta forma en el 
//-- inicio tx esta en reposo, aunque el valor del registro de desplazamiento
//-- sea desconocido
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

//---------------------------
//--  Temporizador
//---------------------------
dividerp1 #(.M(DELAY))
  DIV0 ( .clk(clk),
         .clk_out(start)
       );

//------------------------------
//-- CONTROLADOR
//------------------------------

//-- Estados del automata finito del controlador
localparam IDLE = 0;
localparam START = 1;
localparam TRANS = 2;

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

//-- Inicializador
always @(posedge clk)
  rstn <= 1;

endmodule
```
### dividerp1.v: Divisor de pulsos de anchura de 1 ciclo
El divisor que hemos usado en otros ejemplos está modificado para que la anchura sea de 1 ciclo de reloj:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx2-2.png)

Su descripción en Verilog es:

```verilog
//-- Fichero: dividerp1.v
`include "divider.vh"

//-- ENTRADAS:
//--     -clk: Senal de reloj del sistema (12 MHZ en la iceStick)
//
//-- SALIDAS:
//--     - clk_out. Señal de salida para lograr la velocidad en baudios indicada
//--                Anchura de 1 periodo de clk. SALIDA NO REGISTRADA
module dividerp1(input wire clk,
                 output wire clk_out);

//-- Valor por defecto de la velocidad en baudios
parameter M = `T_100ms;

//-- Numero de bits para almacenar el divisor de baudios
localparam N = $clog2(M);

//-- Registro para implementar el contador modulo M
reg [N-1:0] divcounter = 0;

//-- Contador módulo M
always @(posedge clk)
    divcounter <= (divcounter == M - 1) ? 0 : divcounter + 1;

//-- Sacar un pulso de anchura 1 ciclo de reloj si el generador
assign clk_out = (divcounter == 0) ? 1 : 0;

endmodule
```

### Simulación
El banco de pruebas sólo genera los pulsos de reloj para comprobar el funcionamiento. No se simulan los 100ms, para que vaya más rápido. La señal de start se cambia a 8Khz

El código verilog es:

```verilog
//-- Fichero fsmtx2_tb.v
`include "baudgen.vh"

module fsmtx2_tb();

//-- Baudios con los que realizar la simulacion
//-- A 300 baudios, la simulacion tarda mas en realizarse porque los
//-- tiempos son mas largos. A 115200 baudios la simulacion es mucho
//-- mas rapida
localparam BAUD = `B115200;

//-- Tiempo entre caracteres
localparam DELAY = `F_8KHz;

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
fsmtx2 #(.BAUD(BAUD), .DELAY(DELAY))
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
  $dumpfile("fsmtx2_tb.vcd");
  $dumpvars(0, fsmtx2_tb);

  #(FRAME_WAIT * 10) $display("FIN de la simulacion");
  $finish;
end

endmodule
```
Lo simulamos ejecutando el comando:

    $ make sim2

y el resultado en gtkwave es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx2-sim.png)

La señal superior se corresponde con tx y la que está debajo a _clk_baud_, que marca el tiempo de envío de los bits. En la simulación se ve cómo se envían 2 caracteres completos y el comienzo del tercero.

La señal periódica _start_ es la inferior. Se pueden ver los tres pulsos y cómo por cada uno de ellos se manda un carácter

### Síntesis y pruebas
Hacemos la síntesis con el siguiente comando:

    $ make sint2

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 6 / 96
|PLBs      | 24 / 160
|BRAMs     | 0 / 16

y lo cargamos en la FPGA con:

    $ sudo iceprog fsmtx2.bin

Al abrir el gtkterm a la 115200 baudios veremos cómo van apareciendo los caracteres A periódicamente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T23-fsmtx/images/fsmtx2-gtkterm.png)

# Ejercicios propuestos
* Probar los ejemplos a diferentes baudios y cambiando de retardo de envío de caracteres en el segundo
* Modificar el controlador para que se genere una señal de salida **ready**, que se pogna a 1 cuando el transmisor esté listo para enviar el siguiente carácter y pemanezca a 0 cuando esté ocupado

# Conclusiones
TODO