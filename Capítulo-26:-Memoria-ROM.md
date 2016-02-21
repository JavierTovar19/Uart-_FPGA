![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/romleds-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T26-rom)

# Introducción
Las **memorias** nos permiten **almacenar información** para usarlas en nuestros circuitos: datos, instrucciones, configuraciones, etc. Son los **componentes esenciales** para crear circuitos más complejos, como por ejemplo **microprocesadores**.

Mostraremos cómo se modelan las **memorias ROM** (de sólo lectura) en verilog y haremos tres ejemplos muy sencillos: un hola mundo y dos ejemplos de **generación de secuencias en los leds**, para cargar en la placa ICEstick

# Memoria ROM 32x4

Comenzaremos por una **rom muy sencilla**, que puede almacenar 32 valores de 4 bits. Las direcciones de memoria van desde la 0 hasta la 31. Se necesitan **5 bits** para representarlas

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/rom32x4-1.png)

## Pines de acceso

Los puertos de acceso a la memoria son:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/rom32x4-2.png)

* **clk**: Señal de reloj del sistema.  Es un circuito síncrono, que nos devuelve los datos en el flanco activo del reloj
* **addr**: Entrada: Dirección a leer (5 bits)
* **data**: Salida: Dato almacenado en esa posición

## Cronograma

Para acceder a la memoria rom se deposita la dirección el puerto addres. En el siguiente flanco de bajada del reloj, se obtendrá el dato

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/rom32x4-4.png)

## Descripción en verilog

El código verilog de la memoria rom de 32x4 es el siguiente:

```verilog
//-- Fichero rom32x4.v
`default_nettype none

module rom32x4 (input clk,
                input wire [4:0] addr,
                output reg [3:0] data);

  //-- Memoria
  reg [3:0] rom [0:31];

  //-- Proceso de acceso a la memoria. 
  //-- Se ha elegido flanco de bajada en este ejemplo, pero
  //-- funciona igual si es de subida
  always @(negedge clk) begin
    data <= rom[addr];
  end

//-- Inicializacion de la memoria. 
//-- Solo se dan valores a las 8 primeras posiciones
//-- El resto permanecera a 0
  initial begin
    rom[0] = 4'h0; 
    rom[1] = 4'h1;
    rom[2] = 4'h2;
    rom[3] = 4'h3;
    rom[4] = 4'h4; 
    rom[5] = 4'h5;
    rom[6] = 4'h6;
    rom[7] = 4'h7;
   end
endmodule
```
Los valores almacenados pueden ser cualesquiera. Esta memoria se ha inicializado sólo con 8 valores, iguales a su número de dirección:  En la dirección 0 hay un 0,en la 1 un 1, etc. El resto de posiciones permanece a 0

# Ejemplo 1: ¡Hello world ROM!

Como primer ejemplo, instanciaremos la memoria rom anterior (rom32x4) y mostraremos el contenido de una dirección por los leds

## Diagrama de bloques

El esquema es muy sencillo:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/romhw-1.png)

La **salida de datos** de la rom se conecta directamente a los **4 leds rojos de la placa ICEStick**, para visualizar el dato que sale.  Colocamos una **dirección fija** por la entrada **addr**. El contenido de esa dirección se mostrará por los leds. Dependiendo de la dirección, por los leds se mostrará un dato u otro

## romhw.v: Descripción en Verilog

El código verilog es el siguiente:

```verilog
//-- Fichero romhw.v
`default_nettype none

module romhw (input wire clk,
              output wire [3:0] leds);

localparam ADDR = 5'h5;  //-- Direccion 5 por defecto

//-- Instanciar la memoria rom
rom32x4 
  ROM (
        .clk(clk),
        .addr(ADDR),
        .data(leds)
      );

endmodule
```
El **parámetro ADDR** se usa para establecer la dirección constante que queremos visualizar en los leds. Por defecto se ha elegido la dirección 5

## Simulación

El banco de pruebas es muy sencillo. Sólo instancia el circuito romhw y genera la señal de reloj para que funcione

La simulación se realiza con:

    $ make sim

y los resultados obtenidos en gtkwave son:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/romhw-sim.png)

El valor que se obtiene es 5 (que es lo que se ha almacenado en la posición de memoria 5 de la rom)

## Síntesis y pruebas

La síntesis se realiza con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 2 / 160
|BRAMs     | 1 / 16

Observamos que ahora se está usando **1 bloque de memoria** (BRAM, Block RAM) de los 16 que tiene la FPGA. El sintetizador ha detectado que hay una memoria en nuestro diseño y la ha sintetizado mediante un bloque de memoria.

El diseño se carga con:

    $ sudo iceprog romhw.bin

Por los leds se visualizará esto:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/romhw-test.png)

(led superior e inferior encendidos)

# Ejemplo 2: Secuencia de luces

Como segundo ejemplo vamos a generar una **secuencia de luces en los leds**. Los valores están almacenados en una memoria rom de 16x4

## Diagrama de bloques

La memoria se direcciona mediante un **contador de 4 bits**, de forma que se recorre la memoria desde la dirección 0 hasta la 15. El valor de cada posición se envía directamente a los leds. El contador se incrementa cada **medio segundo**, mediante un temporizador

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/romleds-1.png)

Simplemente cambiando los valores almacenados en la memoria, se consigue una secuencia diferente

## Descripción en Verilog
### Memoria ROM de 16x4
La memoria rom es igual que la de 32x4, pero inicializada con los valores de la secuencia:

```verilog
//-- Fichero: rom16x4.v
module rom16x4 (input clk,
                input wire [3:0] addr,
                output reg [3:0] data);

  //-- Memoria
  reg [3:0] rom [0:31];

  always @(negedge clk) begin
    data <= rom[addr];
  end


//-- ROM2: Secuencia
initial begin
    rom[0] = 4'h1; 
    rom[1] = 4'h2;
    rom[2] = 4'h4;
    rom[3] = 4'h8;
    rom[4] = 4'h1; 
    rom[5] = 4'h8;
    rom[6] = 4'h4;
    rom[7] = 4'h2;
    rom[8] = 4'h1; 
    rom[9] = 4'hF;
    rom[10] = 4'h0;
    rom[11] = 4'hF;
    rom[12] = 4'hC; 
    rom[13] = 4'h3;
    rom[14] = 4'hC;
    rom[15] = 4'h3;
   end

endmodule
```
### romleds.v: Secuenciador

La descripción del ejemplo completo es:

```verilog
//-- Fichero: romleds.v
`default_nettype none

`include "divider.vh"

module romleds (input wire clk,
                 output wire [3:0] leds);

//- Tiempo de envio
parameter DELAY = `T_500ms; //`T_1s;

reg [3:0] addr;
reg rstn = 0;
wire clk_delay;

//-- Instanciar la memoria rom
rom16x4
  ROM (
        .clk(clk),
        .addr(addr),
        .data(leds)
      );

//-- Contador
always @(negedge clk)
  if (rstn == 0)
    addr <= 0;
  else if (clk_delay)
    addr <= addr + 1;

//---------------------------
//--  Temporizador
//---------------------------
dividerp1 #(.M(DELAY))
  DIV0 ( .clk(clk),
         .clk_out(clk_delay)
       );

//-- Inicializador
always @(negedge clk)
  rstn <= 1;

endmodule
```

## Simulación

El banco de pruebas es muy sencillo. Sólo instancia el circuito romleds y genera la señal de reloj para que funcione

```verilog
//-- Fichero: romleds_tb.v
module romleds_tb();

//-- Para la simulacion se usa un retraso de 2 ciclos de reloj
parameter DELAY = 2;

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Datos de salida del componente
wire [3:0] leds;

//-- Instanciar el componente
romleds #(DELAY)
  dut(
    .clk(clk),
    .leds(leds)
  );

//-- Generador de reloj. Periodo 2 unidades
always #1 clk = ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("romleds_tb.vcd");
  $dumpvars(0, romleds_tb);

  # 100 $display("FIN de la simulacion");
  $finish;
end

endmodule
```

La simulación se realiza con:

    $ make sim2

y los resultados obtenidos en gtkwave son:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/romleds-sim.png)


## Síntesis y pruebas
La síntesis se realiza con el comando:

    $ make sint2

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 13 / 160
|BRAMs     | 1 / 16

El diseño se carga con:

    $ sudo iceprog romleds.bin

La secuencia que aparece en los leds se puede ver en este vídeo de youtube:

[![Click to see the youtube video](http://img.youtube.com/vi/HKHM_NONhKg/0.jpg)](https://www.youtube.com/watch?v=HKHM_NONhKg)

# Cargando la ROM desde un fichero
La **carga de las roms** se puede hacer tamién desde **un fichero**. Esto es especialmente útil si lo que se está cargando es el código máquina de un programa. El ensamblador genera este código máquina en un fichero de texto que posteriormente se carga en nuestra rom

La lectura desde fichero se realiza con las funciones **$readmemh** y **$readmemb**. El primero es para leer un fichero con datos en hexadecimal, y el segundo en binario.

## romfile16x4.v: Descripción en Verilog

Esta es una memoria de 16x4 cuyo contenido se carga desde el fichero **rom1.list**. El código verilog es:

```verilog
//-- Fichero romfile16x4
module romfile16x4 (input clk,
                    input wire [3:0] addr,
                    output reg [3:0] data);

//-- Parametro: Nombre del fichero con el contenido de la ROM
parameter ROMFILE = "rom1.list";

  //-- Memoria
  reg [3:0] rom [0:31];

  //-- Lectura de la memoria
  always @(negedge clk) begin
    data <= rom[addr];
  end

//-- Cargar en la memoria el fichero ROMFILE
//-- Los valores deben estan dados en hexadecimal
initial begin
  $readmemh(ROMFILE, rom);
end

endmodule
```
Al instanciar la memoria, el fichero pasado en el parámetro **ROMFILE** se usa para inicializar la memoria. Por defecto se usa el fichero **rom1.list**, cuyo contenido es:

```
//-- Fichero rom1.list
//-- Cada linea se corresponde con una posicion de memoria
//-- Se pueden poner comentarios
//-- ROM1: contiene los numeros del 0 al 15 (en hexadecimal)
0   //-- Posicion 0
1   //-- Posicion 1
2
3
4
5
6
7
8
9
A
B
C
D
E
F  //-- Posicion 15
```

# Ejemplo 3: Secuencia cargada desde fichero
Este es el ejemplo 2 modificado para utilizar el componente **romfile**, que se carga desde un fichero. De esta manera, podemos especificar las secuencias que queremos en los leds simplemente **cambiando el nombre del fichero**. Definimos dos secuencias, en los ficheros **rom1.list** y **rom2.list**.

## romleds2.v: Descripción en verilog

La descripción en verilog es la siguiente:

```verilog
//-- Fichero romleds2.v
`default_nettype none

`include "divider.vh"

module romleds2 (input wire clk,
                 output wire [3:0] leds);

//-- Parametros:
//- Tiempo de envio
parameter DELAY = `T_500ms; //`T_1s;

//-- Fichero para cargar la rom
parameter ROMFILE = "rom1.list";  //--  rom2.list

reg [3:0] addr;
reg rstn = 0;
wire clk_delay;

//-- Instanciar la memoria rom
romfile16x4 #(ROMFILE)
  ROM (
        .clk(clk),
        .addr(addr),
        .data(leds)
      );

//-- Contador
always @(negedge clk)
  if (rstn == 0)
    addr <= 0;
  else if (clk_delay)
    addr <= addr + 1;

//---------------------------
//--  Temporizador
//---------------------------
dividerp1 #(.M(DELAY))
  DIV0 ( .clk(clk),
         .clk_out(clk_delay)
       );

//-- Inicializador
always @(negedge clk)
  rstn <= 1;

endmodule
```

## Simulación

El banco de pruebas es muy sencillo. Sólo instancia el circuito romleds y genera la señal de reloj para que funcione

```verilog
//-- Fichero: romleds_tb.v
module romleds2_tb();

//-- Para la simulacion se usa un retraso de 2 ciclos de reloj
parameter DELAY = 2;
parameter ROMFILE = "rom1.list";

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Datos de salida del componente
wire [3:0] leds;

//-- Instanciar el componente
romleds2 #(.DELAY(DELAY), .ROMFILE(ROMFILE))
  dut(
    .clk(clk),
    .leds(leds)
  );

//-- Generador de reloj. Periodo 2 unidades
always #1 clk = ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("romleds2_tb.vcd");
  $dumpvars(0, romleds2_tb);

  # 100 $display("FIN de la simulacion");
  $finish;
end

endmodule
```

La simulación se realiza con:

    $ make sim3

y los resultados obtenidos en gtkwave son:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/romleds2-sim1.png)

Se está usando la rom1.list, que tiene en cada posición un valor igual a su dirección, como se puede comprobar en la simulación

Para simular la secuencia con la otra rom, hay que cambiar el fichero rom1.list por rom2.list en esta línea del banco de pruebas:
```verilog
parameter ROMFILE = "rom2.list";
```
Ahora al simular vemos una secuencia diferente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T26-rom/images/romleds2-sim2.png)

## Síntesis y pruebas

Primero **seleccionamos el fichero con el contenido de la rom** y lo ponemos en esta línea del fichero **romleds2.v**:

```verilog
parameter ROMFILE = "rom1.list"; //--  rom2.list
```

La síntesis se realiza con el comando:

    $ make sint3

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 12 / 160
|BRAMs     | 1 / 16

El diseño se carga con:

    $ sudo iceprog romleds2.bin

Igual que en el ejemplo 2, veremos cómo leds siguen una secuencia, dada por los valores que hayamos colocado en la rom

# Ejercicios propuestos
* Crear una secuencia en los leds con 32 valores, que se carguen desde un fichero

# CONCLUSIONES
TODO