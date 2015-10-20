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

![https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T27-rom-param/images/genrom-sim-1.png]()

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

# Ejemplo 2: Tocando musica

## Diagrama de bloques

## Descripción en verilog

## Fichero rom1.list

## Simulación

## Síntesis y pruebas

# Ejercicios

# Conclusiones
