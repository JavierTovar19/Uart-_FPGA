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

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 14 / 96
|PLBs      | 6 / 160
|BRAMs     | 0 / 16

Para probarlo lo cargamos en la FPGA como siempre:

    $ sudo iceprog counter.bin

En este vídeo de youtube podemos ver el contador en funcionamiento:

[![Click to see the youtube video](http://img.youtube.com/vi/x9_OwUAtts4/0.jpg)](https://www.youtube.com/watch?v=x9_OwUAtts4)

## Prescaler

Una de las aplicaciones de los contadores es usarlos como **prescalers**. Sirven para **dividir la frecuencia** de una señal de reloj, obteniendo otra de menor frecuencia. Como ejemplo veremos un prescaler de 2 bits (que es un contador de 2 bits). 

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-5.png)

Se incrementa con cada flanco de subida de la señal de reloj clk, que tiene un periodo T. Si observamos las señales de salida de sus dos bits:

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-4.png)

vemos que el periodo de la señal d0 es 2 veces T, y la de la señal d1 es 4 veces T. En general, si tenemos un **prescaler de N bits**, el bit de mayor peso tendrá un periodo de **TN = 2^N * T**. Si tomamos los inversos para pasarlo a frecuancias, tenemos que **fN = f / 2^N**. **La frecuencia del bit de mayor peso del prescaler es la frecuencia de su relog dividida entre 2 elevado al número de bits del prescaler**.

## Periodo del contador

El contador de 26 bits del tutorial lo podemos ver como un contador de 4 bits cuyos bits se sacan por los leds y cuya señal de relog proviene de un prescaler de 22 bits. La señal de reloj del prescaler es de 12Mhz

![Imagen 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-6.png)

Aplicando la fórmula anterior, la **frecuencia de reloj del contador de 4 bits es **f22 = 12Mhz / 2^22 = **2.86 Hz**. Se incrementa cada 1 / 2.86 =  **0.35 segundos**.



## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO



