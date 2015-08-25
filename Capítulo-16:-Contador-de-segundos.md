![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T16-countsec/images/countsec-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T16-countsec)

## Introducción

Para practicar con los divisores de frecuencia, vamos a implementar **un contador que se incremente cada segundo**. La cuenta la mostraremos por los leds de la iCEstick

## divisor.v: Descripción del hardware

Usaremos el divisor de frecuencia del capítulo pasado, pero con algunas mejoras:

    //-- divider.v
    //-- Incluir constantes definidas para este componente
    `include "divider.vh"
    
    //-- Entrada: clk_in. Señal original
    //-- Salida: clk_out. Señal de frecuencia 1/M de la original
    module divider(input wire clk_in, output wire clk_out);
    
    //-- Valor por defecto del divisor
    //-- Lo ponemos a 1 Hz (Constantes definidas en divider.vh)
    parameter M = `F_1Hz;
    
    //-- Numero de bits para almacenar el divisor
    //-- Se calculan con la funcion de verilog $clog2, que nos devuelve el 
    //-- numero de bits necesarios para representar el numero M
    //-- Es un parametro local, que no se puede modificar al instanciar
    localparam N = $clog2(M);
    
    //-- Registro para implementar el contador modulo M
    reg [N-1:0] divcounter = 0;
    
    //-- Contador módulo M
    always @(posedge clk_in)
      divcounter <= (divcounter == M - 1) ? 0 : divcounter + 1;
    
    //-- Sacar el bit mas significativo por clk_out
    assign clk_out = divcounter[N-1];
    
    endmodule

La primera mejora es la inclusión del fichero **divisor.vh** donde se han **definido varias constantes** para facilitar la generación de señales de determinadas frecuencias. 

    `include "divider.vh"

En vez de utilizar un número para los divisores, es más claro asociarlos a sus respectivas frecuencias mediante definiciones. Así por ejemplo, para obtener una señal de 1 Hz, basta con usar la constante F_1Hz en vez del número 12_000_000

     parameter M = `F_1Hz;

Otra majera es la de utilizar el **operador ? :**  en vez de **if-else** para la descripción del contador módulo M. De esta forma el código es más compacto. También le da un aspecto más parecido a C, y eso mola :-)

    //-- Contador módulo M
    always @(posedge clk_in)
      divcounter <= (divcounter == M - 1) ? 0 : divcounter + 1;

## divisor.vh: Constantes

La **extensión .vh** es la que se usa en verilog para la declaración de constantes y macros. La definición de constantes es similar a cómo se hace en C, usando la palabra clave **`define** . El archivo tiene definidas las siguientes constantes:

    //-- Megaherzios  MHz
    `define F_4MHz 3
    `define F_1MHz 12
    
    //-- Hertzios (Hz)
    `define F_2Hz   6_000_000
    `define F_1Hz   12_000_000

En los siguientes capítulos se irá ampliando, a medida que usemos más frecuencias e los ejemplos

## countsec.v: Contador de segundos

Implementaremos un contador de segundos de 4 bits, que saca por los leds la cuenta

### Descripción del hardware

El diseño está formado por un **contador de 4 bits** a cuya entrada de reloj se le ha acoplado un **divisor de frecuencia** para generar una señal de 1Hz

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T16-countsec/images/countsec-2.png)

El código Verilog es el siguiente:

    //-- countsec.v
    //-- Incluir las constantes del modulo del divisor
    `include "divider.vh"
    
    //-- Parameteros:
    //-- clk: Reloj de entrada de la placa iCEstick
    //-- data: Valor del contador de segundos, a sacar por los leds de la iCEstick
    module countsec(input wire clk, output wire [3:0] data);
    
    //-- Parametro del divisor. Fijarlo a 1Hz
    //-- Se define como parametro para poder modificarlo desde el testbench
    //-- para hacer pruebas
    parameter M = `F_1Hz;
    
    //-- Señal de reloj de 1Hz. Salida del divisor
    wire clk_1HZ;
    
    //-- Contador de segundos
    reg [3:0] counter = 0;
    
    //-- Instanciar el divisor
    divider #(M)
      DIV (
        .clk_in(clk),
        .clk_out(clk_1HZ)
      );
    
    //-- Incrementar el contador en cada flanco de subida de la señal de 1Hz
    always @(posedge clk_1HZ)
      counter <= counter + 1;
    
    //-- Sacar los datos del contador hacia los leds
    assign data = counter;
    
    endmodule

