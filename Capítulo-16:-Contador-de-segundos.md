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

### Síntesis

Para sintetizarlo conectamos a la entrada de reloj la señal de 12Mhz de la IceStick y la salida data a los leds

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T16-countsec/images/countsec-1.png)

ejecutamos el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 10 / 160
|BRAMs     | 0 / 16

Lo cargamos en la FPGA con el comando:

    $ sudo iceprog countsec.bin

Los leds empezarán a encenderse. Al cabo de 16 segundos se volverá al comienzo:

[![Click to see the youtube video](http://img.youtube.com/vi/btzHsdh0Pys/0.jpg)](https://www.youtube.com/watch?v=btzHsdh0Pys)

## Simulación

El banco de pruebas es el mismo que en capítulos anteriores. No se simula la obtención de la frecuencia de 1Hz porque consume mucho tiempo y espacio. En vez de ello, se usa un divisor entre 10 para verificar que el contador está funcionando:

    //-- countsec_tb.v
    module countsec_tb();
    
    //-- Registro para generar la señal de reloj
    reg clk = 0;
    wire [3:0] data;
    
    //-- Instanciar el componente y establecer el valor del divisor
    //-- Se pone un valor bajo para simular (sino tardaria mucho)
    countsec #(10)
      dut(
        .clk(clk),
        .data(data)
      );
    
    //-- Generador de reloj. Periodo 2 unidades
    always 
      # 1 clk <= ~clk;
    
    //-- Proceso al inicio
    initial begin
    
      //-- Fichero donde almacenar los resultados
      $dumpfile("countsec_tb.vcd");
      $dumpvars(0, countsec_tb);
    
      # 100 $display("FIN de la simulacion");
      $finish;
    end
    endmodule

Para simular ejecutamos:

    $ make sim

En el simulador veremos cómo el contador "cuenta":

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T16-countsec/images/T16-countsec-sim-1.png)

## Ejercicios propuestos
* **Ejercicio 1**: Cambiar la frecuencia del contador a 2 Hz
* **Ejercicio 2**: Cambiar la frecuencia de forma que el contador cuente décimas de segundo, en vez de segundos

## Conclusiones
TODO


