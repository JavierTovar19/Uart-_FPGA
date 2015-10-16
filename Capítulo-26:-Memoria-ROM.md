(dibujo)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T26-rom)

# Introducción
Las **memorias** nos permiten **almacenar información** para usarlas en nuestros circuitos: datos, instrucciones, configuraciones, etc. Son los **componentes esenciales** para crear circuitos más complejos, como por ejemplo **microprocesadores**.

Mostraremos cómo se modelan las **memorias ROM** (de sólo lectura) en verilog y haremos tres ejemplos muy sencillos: un hola mundo y dos ejemplos de **generación de secuencias en los leds**, para cargar en la placa ICEstick

# Memoria ROM 32x4

Comenzaremos por una **rom muy sencilla**, que puede almacenar 32 valores de 4 bits. Las direcciones de memoria van desde la 0 hasta la 31. Se necesitan **5 bits** para representarlas

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T26-rom/images/rom32x4-1.png)

## Pines de acceso

Los puertos de acceso a la memoria son:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T26-rom/images/rom32x4-2.png)

* **clk**: Señal de reloj del sistemas.  Es un circuito síncrono, que nos devuelve los datos en el flanco activo del reloj
* **addr**: Entrada: Dirección a leer (5 bits)
* **data**: Salida: Dato almacenado en esa posición

## Cronograma

Para acceder a la memoria rom se deposita la dirección el puerto addres. En el siguiente flanco de bajada del reloj, se obtendrá el dato

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T26-rom/images/rom32x4-4.png)

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

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T26-rom/images/romhw-1.png)

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

## Síntesis y pruebas

# Ejemplo 2: Secuencia de luces

# Cargando la ROM desde un fichero

# Ejemplo 3: Otra secuencia