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





