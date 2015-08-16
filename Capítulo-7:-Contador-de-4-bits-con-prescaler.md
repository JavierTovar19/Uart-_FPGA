![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T07-contador-prescaler/images/counter4-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T07-contador-prescaler)

## Introducción
**Contador de 4 bits** conectado a los leds. Para que cuente más lentamente, la señal de reloj se pasa por un **prescaler de 22 bits**.  Se trata del mismo contador del capítulo 4, pero con un diseño mejorado. Para cambiar la frecuencia de cuenta sólo hay que cambiar los bits del prescaler (y no hace falta modificar los bits del contador ni reasignarlos en el fichero .pcf)



## Descripción del hardware

El contador tiene una **entrada de reloj clk** y una **salida de datos data** de 4 bits. También tiene un paráemtro N para indicar el número de bits del prescaler y establecer su frecuencia de funcionamiento. El código verilog es el siguiente:

    //-- counter4.v
    module counter4(input clk, output [3:0] data);
    wire clk;
    reg [3:0] data = 0;
    
    //-- Parametro para el prescaler
    parameter N = 22;
    
    //-- Reloj de salida del prescaler
    wire clk_pres;
    
    //-- Instanciar el prescaler de N bits
    prescaler #(.N(N))
      pres1 (
        .clk_in(clk),
        .clk_out(clk_pres)
      );
    
    //-- Incrementar el contador en cada flanco de subida
    always @(posedge(clk_pres)) begin
      data <= data + 1;
    end
    
    endmodule

Se podría haber definido un contador genérico en un fichero aparte e instanciarlo (igual que se ha hecho con el prescaler). Sin embargo se ha hecho así para mostrar un ejemplo de diseño jerárquico mezclado con un componente definido en un proceso.  Además, el contador es tan sencillo, que no merece la pena almacernarlo en un fichero separado.


## Síntesis en la FPGA

![Imagen 2]()


<img src="" width="400" align="center">

## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO





Borrador...

El contador de 26 bits del tutorial lo podemos ver como un contador de 4 bits cuyos bits se sacan por los leds y cuya señal de reloj proviene de un prescaler de 22 bits, con un reloj de entrada de 12Mhz

![Imagen 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-6.png)

Aplicando la fórmula anterior, la **frecuencia de reloj del contador de 4 bits es **fp = 12Mhz / 2^22 = **2.86 Hz**. Se incrementa cada Tp = 1 / fp = 1 / 2.86 =  **0.35 segundos**.