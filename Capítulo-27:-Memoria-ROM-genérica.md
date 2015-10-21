![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T27-rom-param/images/romnotes-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T27-rom-param)

# Introducción
Las **memorias** son **elementos muy comunes**, que usaremos mucho en nuestros diseños. En vez de estar haciendo memorias con un tamaño determinado, es más versátil crear una **memoria genérica** cuyos parámetros de longitud de datos y de direcciones se establezcan al instanciarlas.

Crearemos una memoria rom genérica y la utilizaremos en dos ejemplo: uno para reproducir una secuencia de luces en los leds y otro para tocar una melodía: **la marcha imperial**

# Memoria rom paramétrica

La memoria rom genérica la denominaremos **genrom**

## Parámetros
Tiene 3 parámetros que se asignan al instanciarse la rom:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T27-rom-param/images/genrom-1.png)

* **DW** (Data width): Anchura de los datos (en bits)
* **AW** (Address width): Anchura de las direcciones (en bits)
* **ROMFILE**: Fichero con el contenido de la rom

## Puertos

Los **puertos de la rom** son los clásicos, pero ahora su tamaño no está especificado:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T27-rom-param/images/genrom-2.png)

* **Addr**: Bus de direcciones
* **data**: Bus de datos
* **clk**: Reloj del sistema

## genrom.v: Descripción en Verilog

Puesto que los puertos (addr y data) son genéricos, y por tanto se tienen que definir al declarar el módulo, es necesario primero definir los parámetros AW y DW. Luego, en función de ellos se definen los puertos. Esto se codifica en verilog con esta estructura:

_module nombre #(definición de parametros)_
               _(definicion de puertos);_

El código verilog de la memoria rom genérica es:

```verilog
//-- Fichero: genrom.v
module genrom #(             //-- Parametros
         parameter AW = 5,   //-- Bits de las direcciones (Adress width)
         parameter DW = 4)   //-- Bits de los datos (Data witdh)

       (                              //-- Puertos
         input clk,                   //-- Señal de reloj global
         input wire [AW-1: 0] addr,   //-- Direcciones
         output reg [DW-1: 0] data);  //-- Dato de salida

//-- Parametro: Nombre del fichero con el contenido de la ROM
parameter ROMFILE = "rom1.list";

//-- Calcular el numero de posiciones totales de memoria
localparam NPOS = 2 ** AW;

  //-- Memoria
  reg [DW-1: 0] rom [0: NPOS-1];

  //-- Lectura de la memoria
  always @(posedge clk) begin
    data <= rom[addr];
  end

//-- Cargar en la memoria el fichero ROMFILE
//-- Los valores deben estan dados en hexadecimal
initial begin
  $readmemh(ROMFILE, rom);
end

endmodule
```
El tercer parámetro, ROMFILE, se define de la manera habitual, usando la palabra clave **parameter** dentro del módulo. Dentro de #() sólo se definen los parámetros que afectan a los puertos

## Instanciando la rom en nuestros diseños

La mejor manera de utilizar la rom genérica en nuestros diseños es volviendo a definir los parámetros AW y DW (aunque se pueden usar otros nombres) usando como valores los que necesitemos en nuestro diseño:

```verilog
parameter AW = 5;
parameter DW = 5;
```
Ahora declaramos **los cables** que van conectados a la rom genérica en función de estos parámetros:

```verilog
reg [AW-1: 0] addr;  //-- Bus de direcciones
reg [DW-1: 0] data;  //-- Bus de datos
```
y finalmente instanciamos la rom:

```verilog
genrom 
  #( .ROMFILE(ROMFILE),  //-- Asignacion de parametros
     .AW(AW),
     .DW(DW))
  ROM (                  //-- coneion de cables
        .clk(clk),
        .addr(addr),
        .data(data)
      );
```

# Ejemplo 1: secuencia en los leds

En este ejemplo reproduciremos una **secuencia en los leds**, igual que en los ejemplos del capítulo anterior, pero usando una rom genérica

## Diagrama de bloques

Utilizaremos **5 leds para la secuencia**, por lo que la **anchura de los datos** será de 5 bits (**DW = 5**), y una **anchura de direcciones** también de 5 bits (**AW = 5**, para tener 32 posiciones). La secuencia es un contador. En cada posición de la memoria se almacena su número de dirección

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T27-rom-param/images/genrom-3.png)

## genromleds.v: Descripción en verilog

La descripción en verilog del ejemplo es:

```verilog
//-- Fichero: genromleds.v
`default_nettype none

`include "divider.vh"

module genromleds (input wire clk,
                   output wire [4:0] leds);

//- Tiempo de envio
parameter DELAY = `T_500ms;

//-- Fichero con la rom
parameter ROMFILE = "rom1.list";

//-- Numero de bits de la direccione
parameter AW = 5;
parameter DW = 5;

//-- Cable para direccionar la memoria
reg [AW-1: 0] addr;

reg rstn = 0;
wire clk_delay;

//-- Instanciar la memoria rom
genrom 
  #( .ROMFILE(ROMFILE),
     .AW(AW),
     .DW(DW))
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

## Fichero rom1.list

El fichero que se graba en la rom, con la secuencia de ejemplo (contador) es el siguiente:

```verilog
//-- Fichero rom1.list
//-- Cada linea se corresponde con una posicion de memoria
//-- Se pueden poner comentarios
//-- ROM1: contiene los numeros del 0 al 31 (en hexadecimal)
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
F 
10
11
12
13
14
15
16
17
18
19
1A
1B
1C
1D
1E
1F
```

## Simulación

El banco de pruebas es el mismo que en el capítulo anterior. Para simular ejecutamos:

    $ make sim

y en el gtkwave vemos lo siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T27-rom-param/images/genrom-sim-1.png)

Por los leds aparece la secuencia de cuenta, desde 0 hasta 31 (en hexadecimal)

## Síntesis y pruebas

Para sintetizar hay que ejecutar el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 7 / 96
|PLBs      | 16 / 160
|BRAMs     | 1 / 16

El diseño se carga con:

    $ sudo iceprog genromleds.bin

Se podrá ver por los leds la secuencia de cuenta

# Ejemplo 2: Tocando la marcha imperial

En el segundo ejemplo tocaremos un fragmento de [la marcha imperial](https://es.wikipedia.org/wiki/Marcha_Imperial) de star wars. Esto es un clásico en el grupo de [Clone wars](http://www.reprap.org/wiki/Proyecto_Clone_Wars), tocándose con los motores paso a paso para comprobar su funcionamiento. Pues bien, **una FPGA no la tendrás dominada hasta que no toques con ella la marcha imperial** :-)

## Diagrama de bloques

El diseño consta de una **memoria ROM de 64 posiciones de 16 bits**. En cada una de ellas se almacena el valor del divisor para generar una nota musical. Los **5 bits menos significativos** de cada valor de la nota **se envían a los leds**, para ver la actividad

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T27-rom-param/images/romnotes-1.png)

El generador de notas (**notegen**) es similar al que se hizo en el [capítulo 17](https://github.com/Obijuan/open-fpga-verilog-tutorial/wiki/Cap%C3%ADtulo-17%3A-Generando-tonos-audibles) pero adaptándolo a las reglas del diseño síncrono y haciendo que **módulo del contador** se pasa como una entrada más, en vez de ser un valor fijo. Además, se ha modificado para que la señal cuadrada generada tenga siempre un **ciclo de trabajo del 50%** (y que todas las notas suenen con la misma intensidad)

La **duración de cada nota** se ha establecido en **200ms**. Colocando la misma nota en dos posiciones consecutivas de la memoria rom, su duración será el doble (400ms). De esta forma se controla de forma sencilla la duración de todas las notas y los silencios

## Descripción en verilog

Los dos ficheros principales son el **notegen.v**, que contiene el componente de generación de las notas musicales y **romnotes.v** que tiene el reproductor completo

### notegen.v:

```verilog
//-- Fichero: notegen.v
module notegen(input wire clk,          //-- Senal de reloj global
               input wire rstn,         //-- Reset
               input wire [15:0] note,  //-- Divisor
               output reg clk_out);     //-- Señal de salida

wire clk_tmp;

//-- Registro para implementar el contador modulo note
reg [15:0] divcounter = 0;

//-- Contador módulo note
always @(posedge clk)

  //-- Reset
  if (rstn == 0)
    divcounter <= 0;      

  //-- Si la nota es 0 no se incrementa contador
  else if (note == 0)
    divcounter <= 0;

  //-- Si se alcanza el tope, poner a 0
  else if (divcounter == note - 1)
    divcounter <= 0;

  //-- Incrementar contador
  else
    divcounter <= divcounter + 1;

//-- Sacar un pulso de anchura 1 ciclo de reloj si el generador
assign clk_tmp = (divcounter == 0) ? 1 : 0;

//-- Divisor de frecuencia entre 2, para obtener como salida una señal
//-- con un ciclo de trabajo del 50%
always @(posedge clk)
  if (rstn == 0)
    clk_out <= 0;

  else if (note == 0)
    clk_out <= 0;

  else if (clk_tmp == 1)
    clk_out <= ~clk_out;
 
endmodule
```
### romnotes.v

La descripción en verilog del circuito reproductor se muestra a continuación:

```verilog
//-- Fichero romnotes.v
//-- Incluir las constantes del modulo del divisor
`include "divider.vh"

//-- Parametros:
//-- clk: Reloj de entrada de la placa iCEstick
//-- ch_out: Canal de salida
module romnotes(input wire clk, 
                output wire [4:0] leds,
                output wire ch_out);

//-- Parametros
//-- Duracion de las notas
parameter DUR = `T_200ms;

//-- Fichero con las notas para cargar en la rom
parameter ROMFILE = "imperial.list";

//-- Tamaño del bus de direcciones de la rom
parameter AW = 6;

//-- Tamaño de las notas
parameter DW = 16;

//-- Cables de salida de los canales
wire ch0, ch1, ch2;

//-- Selección del canal del multiplexor
reg [AW-1: 0] addr = 0;

//-- Reloj con la duracion de la nota
wire clk_dur;
reg rstn = 0;

wire [DW-1: 0] note;

//-- Instanciar la memoria rom
genrom 
  #( .ROMFILE(ROMFILE),
     .AW(AW),
     .DW(DW))
  ROM (
        .clk(clk),
        .addr(addr),
        .data(note)
      );

//-- Generador de notas
notegen
  CH0 (
    .clk(clk),
    .rstn(rstn),
    .note(note),
    .clk_out(ch_out)
  );

//-- Sacar los 5 bits menos significativos de la nota por los leds
assign leds = note[4:0];

//-- Inicializador
always @(posedge clk)
  rstn <= 1;


//-- Contador para seleccion de nota
always @(posedge clk)
  if (clk_dur)
    addr <= addr + 1;

//-- Divisor para marcar la duración de cada nota
dividerp1 #(DUR)
  TIMER0 (
    .clk(clk),
    .clk_out(clk_dur)
  );

endmodule
```

El **parámetro DUR** determina la **duración mínima de una nota** (o un silencio), que se ha establecido en **200ms**. Para reproducir una nota del doble de duración, simplemente se toca dos veces. Tocándola N veces durará N * 200ms. 

## Contenido de la ROM: imperial.list

La melodía de la marcha imperial está almacenada en este fichero de texto:

```
//-- Marcha imperial
0  //-- Un 0 es un SILENCIO
0
0
0
0
0
0
0
0
471A //-- MI_4
471A
0
471A //-- MI_4
471A
0
471A //-- MI_4
471A
0
5996 //-- DO_4
5996
3BCA //-- SOL_4
471A //-- MI_4
471A
0
5996 //-- DO_4
5996
3BCA //-- SOL_4
471A //-- MI_4
471A
//----------- Segundo trozo
0
0
0
2F75  //-- SI_4
2F75
0
2F75  //-- SI_4
2F75
0
2F75  //-- SI_4
2F75
0
2CCB  //-- DO_5
2CCB
3BCA //-- SOL_4
471A //-- MI_4
471A
0
5996 //-- DO_4
5996
3BCA //-- SOL_4
471A //-- MI_4
471A
```

Lo que se almacenan son **los valores de los divisores** (en hexadecimal) para generar las notas. Los valores de las diferentes notas se encuentran en el archivo **notegen.vh**.

Para que una nota dure más tiempo, se reproduce 2 ó más veces, colocándose en memoria las copias de la misma nota.

La **nota 0** equivale a **un silencio**

## Simulación

El banco de pruebas es el clásico: se instancia el componente romnotes y se genera el reloj para que funcione:

```verilog
//-- Fichero: romnotes_tb.v
module romnotes_tb();

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Salidas de los canales
wire ch_out;


//-- Instanciar el componente y establecer el valor del divisor
//-- Se pone un valor bajo para simular (de lo contrario tardaria mucho)
romnotes #(.DUR(2))
  dut(
    .clk(clk),
    .ch_out(ch_out)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("romnotes_tb.vcd");
  $dumpvars(0, romnotes_tb);

  # 200 $display("FIN de la simulacion");
  $finish;
end

endmodule
```

Para simulare ejecutamos:

    $ make sim2

Y en la simulación podremos ver cómo se van enviando los 5 bits menos significativos de la nota a los leds. En la simulación la duración de la nota está puesta a 2 unidades

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T27-rom-param/images/romnotes-sim-1.png)

## Síntesis y pruebas

# Ejercicios
* Completar la marcha imperial, aumentando la memoria y añadiendo el resto de notas

# Conclusiones
TODO
