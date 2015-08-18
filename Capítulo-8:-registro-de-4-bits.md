<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/T08-blink4-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T08-register)

## Introducción
Los **registros** son **elementos esenciales** en los circuitos digitales. Nos pemiten almacenar información, desde 1 hasta N bits. Se utilizan para implementar procesadores, realizar segmentación, almacenamiento de resultados intermedios, etc.

El registro básico captura los datos de la entrada, en flanco de subida o bajada del reloj, los almacena y los saca por la salida. Su esquema es:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/blink4-2.png)

En este capitulo utilizaremos un registro de 4 bits para hacer que parpadeen los 4 leds de la placa iCEstick

## blink4: Encendiendo y apagando los 4 leds

El esquema que utilizaremos es el siguiente:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/blink4-3.png)

El registro de 4 bits inicialmente está inicializado a 0. Por la salida dout salen 4 bits a 0, que se convierten en 4 bits a 1 al pasar por el inversor. En el siguiente flanco de subida del reloj, se captura este nuevo valor 4'b1111 en el registro. Al salir por dout, se vuelve a invertir obteniéndose 4 bits a 0, como al principio.  El resultado es que se obtiene la siguiente secuencia  0000, 1111, 0000, 1111 ... que cambia con cada flanco de subida del reloj.  Si ahora la salida dout la conectamos a los leds, estos empezarán a encenderse y apagarse con la frecuencia del reloj.

## Descripción del hardware

El diseño completo a implementar en la FPGA se muestra en esta figura:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/blink4-1.png)

Para poder apreciar el parpadeo, se incluye un prescaler. La descripción del hardware en Verilog es:

    //-- blink4.v
    module blink4(input wire clk,           //--Reloj
                  output wire [3:0] data    //-- Salida del registro);
    
    //-- Bits para el prescaler
    parameter N = 22;
    
    //-- Reloj principal (prescalado)
    wire clk_base;
    
    //-- Datos del registro
    reg [3:0] dout = 0;
    
    //-- Cable de entrada al registro
    wire [3:0] din;
    
    //-- Instanciar el prescaler
    prescaler #(.N(N))
      PRES (
        .clk_in(clk),
        .clk_out(clk_base)
      );
    
    //-- Registro
    always @(posedge(clk_base))
      dout <= din;
    
    //-- Puerta NOT entra la salida y la entrada
    assign din = ~dout;
    
    //-- Sacar datos del registro por la salida
    assign data = dout;
    
    endmodule

Obsevamos que la **implementación de un registro** es extremadamente sencilla. Basta con este código:

    always @(posedge(clk_base))
      dout <= din;

Esta es la razón por la que normalmente no usaremos un diseño jerárquico: no es necesario incluir el código en un fichero a parte por lo sencillo que es.

Hemos usado una **definición compacta** de los parámetros del módulo blink4: todo está definido en la propia declaración del módulo:

    module blink4(input wire clk, output wire [3:0] data);

Esto se podría haber hecho también así:

    module blink4(input clk, output [3:0] data);
    wire clk;
    wire data;
    
o así:

    module blink4(clk, data);
    input clk;
    output data;
    wire clk;
    wire [3:0] data;
    
Esta última forma se utiliza para definir componentes paramétricos (lo veremos más adelante)

## Síntesis en la FPGA
Para sintetizar el diseño en la FPGA hacemos:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 6 / 160
|BRAMs     | 0 / 16

Para descargar en la FPGA hacemos:

    $ sudo iceprog blink4.bin

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/T08-blink4-iCEstick-1.png" width="400" align="center">

En este vídeo se puede ver el resultado:

[![Click to see the youtube video](http://img.youtube.com/vi/YFdEBQlX604/0.jpg)](https://www.youtube.com/watch?v=YFdEBQlX604)

## Simulación

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO



