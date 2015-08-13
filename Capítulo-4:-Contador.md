<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/T04-counter-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T04-counter)

## Introducción
Modelaremos nuestro **primer circuito secuencial**: un contador conectado a los LEDs. Los circuitos secuenciales, a diferencia de los combinacionales, **almacenan información**. El contador almacena un número que se incrementa con cada tic del reloj.

Esta es la pinta de nuestro componente. Se actualiza en cada flanco de subida del reloj, y su **salida data** es de **26 bits**.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-1.png)

**La señal de reloj** de la placa iCEstick es de **12Mhz**. Si hacemos un contador de sólo 4 bits y le conectamos a su entrada clk esta señal de 12Mhz, el resultado será que se incrementará tan rápido que siempre veremos los leds encendidos. Por ello utilizaremos un contador de 26 bits y usaremos los 4 más significativos para mostrarlos en los leds. 

## Descripción del hardware

El contador tiene **una entrada clk**, que es un cable, y una **salida data de 26  bits** que nos devuelve el valor del contador. Esta salida es un **registro de 26 bits**, que almacena el valor de la cuenta.

    //-----------------------------------
    //-- Entrada: señal de reloj
    //-- Salida: contador de 26 bits
    //-----------------------------------
    module counter(input clk, output [25:0] data);
    wire clk;
    
    //-- La salida es un registro de 26 bits, inicializado a 0
    reg [25:0] data = 0;
    
    //-- Sensible al flanco de subida
    always @(posedge clk) begin
      //-- Incrementar el registro
      data <= data + 1;
    end
    endmodule

El funcionamiento del componente se describe dentro del bloque **always @(posedge clk)**, que indica que todo lo que está dentro de este bloque sólo se evaluará cada vez que llegue un **flanco de subida** en la señal clk. Cada vez que llega uno, se incrementa en una unidad el registro data (y saldrá por la salida del componente).

Cualquier contador, con independencia de su número de bits, se construye de esta manera. Si queremos que tenga 20 bits, sólo tenemos que cambiar el número 25 por 19 en las definiciones del registro data.

## Síntesis en la FPGA

Los 4 bits de mayor peso (data[25], data[24], data[23] y data[22]) los conectaremos a los pines de la fpga donde están los leds. El reloj de la iCEstick entra por el pin 21, que lo conectaremos a la señal de reloj clk de nuestro contador

![Image 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-2.png)

La asociación entre pines de nuestro componente y pines de la fpga se establece en el fichero **counter.pcf**:

    set_io data[25] 96
    set_io data[24] 97
    set_io data[23] 98
    set_io data[22] 99
    set_io clk 21

Realizamos la síntesis como siempre:

    $ make sint

[![Click to see the youtube video](http://img.youtube.com/vi/x9_OwUAtts4/0.jpg)](https://www.youtube.com/watch?v=x9_OwUAtts4)

## División de frecuencias

Borrador
El bit de mayor peso (25) parpadea con un frecuencia de f = 1 / T   T0 = 2 * 2^0 T,  T1 = 2 * 2 * T, Tb = 2 * 2^b * T = 2^(b+1) * T.    T25 = 2^26 * T = 5.6 seg (aprox)

## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO



