![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/buffer-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T28-ram)

# Introducción
Las **memorias RAM** nos permiten almacenar datos y recuperarlos durante el funcionamiento del circuito. Son memorias donde podemos leer y escribir

Crearemos una **memoria RAM genérica**, síncrona, con una entrada de datos para escritura y una salida para lectura. Como ejemplo diseñaremos un **buffer de almacenamiento** de 16 bytes que cargue a través de datos provenientes del puerto serie, y una vez llenado, se vuelca su contenido también por el puerto serie.

# Memoria RAM genérica

La memoria RAM genérica la denominaremos genram

## Puertos y parámetros

Los puertos y parámetros se muestran esta figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/genram-1.png)

Los **parámetros** son los mismos que en la memoria ROM:
* **DW** (Data width): Anchura de los datos (en bits)
* **AW** (Address width): Anchura de las direcciones (en bits)
* **ROMFILE**: Fichero con el contenido inicial precargado de la ram

Los **puertos** son:
* **data_in**: Entrada de datos, para escritura
* **data_out**: Salida de datos, en la lectura
* **addr**: Dirección de acceso, tanto para escritura como para lectura
* **rw**: Modo de acceso: lectura (rw = 1) o escritura (rw = 0)

## Cronograma

En este cronograma se muestran **dos ciclos de lectura** y **uno de escritura** intercalado:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/genram-2.png)

**Ciclo de lectura**:
* Se coloca la dirección de lectura en **addr**
* Se pone la **señal rw a 1** para indicar que se quiere leer
* En el siguiente **flanco de subida** se **devolverá el dato** por el puerto **data_out**

**Ciclo de escritura**:
* Se coloca la **dirección** de escritura en** addr**
* Se coloca el **dato a escribir** en el puerto **data_in**
* Se pone la **señal rw a 0** para indicar **escritura**
* En el siguiente **flanco de subida** del reloj se escribirá el **dato en la memoria**

## Descripción en verilog

El código es similar al de la memoria rom pero añadiendo un proceso nuevo que se encarga de **la escritura en cada flanco de subida del reloj**, usando la señal rw como habilitación

```verilog
module genram #(             //-- Parametros
         parameter AW = 5,   //-- Bits de las direcciones (Adress width)
         parameter DW = 4)   //-- Bits de los datos (Data witdh)

       (        //-- Puertos
         input clk,                      //-- Señal de reloj global
         input wire [AW-1: 0] addr,      //-- Direcciones
         input wire rw,                  //-- Modo lectura (1) o escritura (0)
         input wire [DW-1: 0] data_in,   //-- Dato de entrada
         output reg [DW-1: 0] data_out); //-- Dato a escribir

//-- Parametro: Nombre del fichero con el contenido de la RAM
parameter ROMFILE = "bufferini.list";

//-- Calcular el numero de posiciones totales de memoria
localparam NPOS = 2 ** AW;

  //-- Memoria
  reg [DW-1: 0] ram [0: NPOS-1];

  //-- Lectura de la memoria
  always @(posedge clk) begin
    if (rw == 1)
    data_out <= ram[addr];
  end

  //-- Escritura en la memoria
  always @(posedge clk) begin
    if (rw == 0)
     ram[addr] <= data_in;
  end

//-- Cargar en la memoria el fichero ROMFILE
//-- Los valores deben estan dados en hexadecimal
initial begin
  $readmemh(ROMFILE, ram);
end

endmodule
```

# Ejemplo: Buffer

Para probar la memoria ram, implementaremos un **buffer de 16 bytes**. Se comienza volcando el buffer a través del puerto serie (tiene unos datos iniciales que establecemos en el fichero bufferini.list). Los siguientes caracteres recibidos por el puerto serie, se van almacenando en la memoria, comenzando por la dirección 0. Una vez llenada la memoria, se procede a su volcado

## Diagrama de bloques

La arquitectura del circuito es la clásica: una ruta de datos y un controlador:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/buffer-1.png)

### Ruta de datos
En la ruta de datos se instancian la unidad de transmisión de serie (**uart_tx.v**), la de recepción (**uart_rx.v**) y la memoria ram genérica (**genram.v**). El resto de componentes son: un **contador** para direccionar la memoria, un **comparador** para detectar cuando hay overflow en el **contador** (paso de 0xF a 0) y un inicializador

El **contador es de 4 bits** (para direccionar la memoria de 16 posiciones). Tiene una entrada de enable controlada por la microórden **cena** para habilitar su cuenta. Cuando está a 1 se incrementa. Su salida se compara con el valor 0xF para activar la **señal de overflow ov**, que indica que se ha llegado a la última posición de la memoria.

La **salida de datos del receptor serie** está conectada directamente a la **entrada de datos de la memoria**, para guardar los datos recibidos en la dirección indicada por el contador. La **salida de datos de la memoria** está conectada a la **entrada de datos del transmisor serie**, para su volcado

### Controlador

Las **microórdenes** que genera el controlador son:

* **cena**:  Habilitación del contador (counter enable). Cuando está a 1 se incrementa en una unidad el contador
* **rw**:  Selección de lectura (1) o escritura (0) en la memoria
* **transmit**: Realizar la transmisión serie de un carácter

El diagrama de estados del autómata es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/buffer-2.png)

Comienza en el estado **WAIT_TX**. Se queda esperando hasta que el transmisor esté listo para transmitir (cuando ready está a 1). Se pasa al estado **READ_TX** donde se lee el dato de la memoria y se inicia la transmisión por el puerto serie. Se habilita el contador para que se incremente la dirección. Se vuelve al estado WAIT_TX hasta que se pueda enviar el siguiente.

Permanece en ese bucle hasta que se alcanza la **dirección 0xF**, activándose la **señal de overflow ov**. En estos primeros dos estados se vuelca el contenido completo de la memoria. En los siguientes estados se almacenan los datos recibidos desde el puerto serie.

El funcionamiento es similar, se comienza con **WAIT_RX**, esperando a recibir un dato. Una vez recibido se pasa a **WRITE_RX** donde se escribe en la ram y se incrementa el contador de direcciones. Se repite el bucle hasta que se han almacenado los 16 caracteres. Y se procede al estado inicial, para comenzar su volcado


## Descripción en verilog

El código del buffer en Verilog es el siguiente:

```verilog
//--Fichero: buffer.v
`default_nettype none

`include "baudgen.vh"

module buffer (input wire clk,
               input wire rx,
               output wire tx,
               output wire [3:0] leds,
               output reg debug);

//-- Velocidad de transmision
parameter BAUD = `B115200;

//-- Fichero con la rom
parameter ROMFILE = "bufferini.list";

//-- Numero de bits de la direccion de memoria
parameter AW = 4;

//-- Numero de bits de los datos almacenados en memoria
parameter DW = 8;

//-- Señal de reset
reg rstn = 0;

//-- Inicializador
always @(posedge clk)
  rstn <= 1;

//--------------------- Memoria RAM
reg [AW-1: 0] addr = 0;
wire [DW-1: 0] data_in;
wire [DW-1: 0] data_out;
reg rw;

genram
  #( .ROMFILE(ROMFILE),
     .AW(AW),
     .DW(DW))
  RAM (
        .clk(clk),
        .addr(addr),
        .data_in(data_in),
        .data_out(data_out),
        .rw(rw)
      );

//------ Contador
//-- counter enable: Se pone a 1 cuando haya que acceder a la siguiente
//-- posicion de memoria
reg cena;

always @(posedge clk)
  if (!rstn)
    addr <= 0;
  else if (cena)
    addr <= addr + 1;

//-- Overflow del contador: se pone a uno cuando todos sus bits
//-- esten a 1
wire ov = & addr;

//-------- TRANSMISOR SERIE
wire ready;
reg transmit;

//-- Instanciar la Unidad de transmision
uart_tx #(.BAUD(BAUD))
  TX0 (
    .clk(clk),
    .rstn(rstn),
    .data(data_out),
    .start(transmit),
    .ready(ready),
    .tx(tx)
  );

//-------- RECEPTOR SERIE
wire rcv;

uart_rx #(BAUD)
  RX0 (.clk(clk),         //-- Reloj del sistema
       .rstn(rstn),     //-- Señal de reset
       .rx(rx),           //-- Linea de recepción de datos serie
       .rcv(rcv),         //-- Señal de dato recibido
       .data(data_in)     //-- Datos recibidos
      );

assign leds = data_in[3:0];

//------------------- CONTROLADOR

//-- Estado del automata
reg [1:0] state;
reg [1:0] next_state;

localparam TX_WAIT = 0;
localparam TX_READ = 1;
localparam RX_WAIT = 2;
localparam RX_WRITE = 3;

//-- Transiones de estados
always @(posedge clk) 
  if (!rstn)
    state <= TX_WAIT;
  else
    state <= next_state;

//-- Generacion de microordenes
//-- y siguientes estados
always @(*) begin

  //-- Valores por defecto
  next_state = state;
  rw = 1;
  cena = 0;
  transmit = 0;
  debug = 0;

  case (state)

    //-- Esperar a que transmisor este listo para enviar
    TX_WAIT: begin
      if (ready)
        next_state = TX_READ;
      else
        next_state = TX_WAIT;
    end

    //-- Lectura del dato en memoria y transmision serie
    TX_READ: begin

      transmit = 1;
      cena = 1;

      //-- Si es el ultimo caracter pasar a recepcion
      if (ov) 
        next_state = RX_WAIT;
      else
        next_state = TX_WAIT;
    end

    //-- Esperar a que lleguen caracteres por el puerto serie
    RX_WAIT: begin

      debug = 1;

      if (rcv)
        next_state = RX_WRITE;
      else
        next_state = RX_WAIT;
    end

    //-- Escritura de dato en memoria
    RX_WRITE: begin
      rw = 0;
      cena = 1;

      //-- Si es el ultimo caracter, ir al estado inicial
      if (ov)
        next_state = TX_WAIT;
      else
        next_state = RX_WAIT;
    end
  endcase

end

endmodule
```

Observamos que **la máquina de estados** se ha hecho de una **forma diferente** a como se había hecho en los otros ejemplos. Es otra modalidad para hacer autómatas. En esta se definen el **registro de estado** (**state**) y su **siguiente valor** (**next_state**):

```verilog
//-- Estado del automata
reg [1:0] state;
reg [1:0] next_state;
```

Se define un proceso que simplemente actualiza el estado actual con el siguiente:

```verilog
//-- Transiones de estados
always @(posedge clk) 
  if (!rstn)
    state <= TX_WAIT;
  else
    state <= next_state;
```

En un segundo **proceso combinacional** se definen los **valores de las microórdenes** y las **transiciones** entre estados

Esta forma de escribir autómatas es más compacta que la usada anteriormente

## Simulación

## Síntesis y pruebas

# Ejercicios propuestos

# Conclusiones
TODO
