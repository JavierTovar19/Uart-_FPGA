<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/T05-prescaler-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T05-prescaler)

## Introducción

Los _prescalers_ sirven para **ralentizar las señales de reloj**. Por la entrada entra una señal de reloj de frecuencia f y por la salida se obtiene una de frecuencia menor.  En este tutorial haremos un **prescaler de N bits** para hacer parpadear un led a diferentes frecuencias

![Image header](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-1.png)

Para un prescaler de N bits, las **fórmulas** que relacionan las frecuencias y periodos de entrada con los de salida son:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-2.png)

## Entendiendo el prescaler de 2 bits

Antes de implementar un prescaler de N bits, vamos a entender cómo funciona uno de 2 bits

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-3.png)

Internamente está constituido por un **contador de 2 bits**, cuyas salidas son d0 y d1. La de mayor peso es la que se saca como señal de salida. Este contador se incrementa en cada flanco de subida de clk, que tiene un periodo T. Si observamos las señales de salida de sus dos bits (d0 y d1):

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-4.png)

vemos que el periodo de la señal d0 es 2 veces T, y la de la señal d1 es 4 veces T. Es decir, que cada nuevo bit duplica el periodo de la señal anterior. Siguiendo la fórmula general, el periodo de este prescaler de 2 bits es: Tout = 2^2 * T = 4 * T (y gráficamente comprobamos que es así).

## Frecuencias y periodos del prescaler

La **frecuencia de entrada** al prescaler en la placa iCEStick es de **12Mhz**. Aplicando la fórmula anterior, obtenemos esta **tabla con periodos y frecuencias** para prescalers con diferente número de bits (N). Esto nos da una idea de qué valor de N elegir para hacer parpadear el led

| Bits (N)  | Frecuencia  |  Periodo | Visible
|-------|-------------|---------|-----------
|  1    |  6 MHz      |  0.167 usec | No
|  2    |  3 MHz      |  0.333 usec | No
|  3    |  1.5 Mhz    |  0.666 usec | No
|  4    |  750 Khz    |  1.333 usec | No
|  5    |  375 Khz    |  2.666 usec | No
|  6    |  187.5 Khz  |  5.333 usec | No
|  7    |  93.75 KHz  |  10.666 usec | No
|  8    |  46.875 Khz |  21.333 usec | No
|  9    |  23437.5 Hz |  42.666 usec | No
| 10    |  11718.7 Hz |  85.333 usec | No
| 11    |  5859.37 Hz |  170.66 usec | No
| 12    |  2929.68 Hz |  341.33 usec | No
| 13    |  1464.84 Hz |  682.66 usec | No
| 14    |  732.42 Hz  |  1.365 ms    | No
| 15    |  366.21 Hz  |  2.73 ms | No
| 16    |  183.1 Hz   |  5.46 ms | No
| 17    |  92.552 Hz  |  10.92 ms | No
| 18    |  45.776 Hz  |  21.84 ms | No
| 19    |  22.888 Hz  |  43.69 ms | Si
| 20    |  11.444 Hz  |  87.38 ms | Si
| 21    |  5.722 Hz   |  174.76 ms | Si
| 22    |  2.861 Hz   |  349.52 ms | Si

El ojo humano tiene una frecuencia de refresco de unos 25Hz. Esto significa que frecuencias superiores no se aprecian. Si hacemos parpadear el led con una frecuencia superior, lo apreciaremos como si siempre estuviese encendido (no lo veremos parpadear)

Al usar el prescarler con el led, **a partir de 19 bits es cuando se puede apreciar el parpadeo**. Cuanto más bits, más lento parpadeará el led.

## Descripción del hardware

El código es prácticamente igual al de un contador, sin embargo introducimos la novedad de que **el prescaler es paramétrico**, de forma que el número de bits está determinado por el **parámetro N**. Sólo cambiando este parámetro podemos sintetizar prescalers de diferentes tamaños

    //-- prescaler.v
    //-- clk_in: señal de reloj de entrada
    //-- clk_out: Señal de reloj de salida, con menor frecuencia
    module prescaler(input clk_in, output clk_out);
    wire clk_in;
    wire clk_out;
    
    //-- Numero de bits del prescaler (por defecto)
    parameter N = 22;
    
    //-- Registro para implementar contador de N bits
    reg [N-1:0] count = 0;
    
    //-- El bit más significativo se saca por la salida
    assign clk_out = count[N-1];
    
    //-- Contador: se incrementa en flanco de subida
    always @(posedge(clk_in)) begin
      count <= count + 1;
    end
    
    endmodule

Definimos un registro de N bits, que se incrementa en cada flanco de subida de la señal de reloj de entrada. Su bit más significativo se conecta directamente a la salida clk_out.

**Por defecto el prescaler es de 22 bits**, por lo que la **frecuencia de clk_out** es de **2.9Hz** aprox. Sólo hay que dar otro valor a N para cambiar la frecuencia de salida

## Síntesis en la FPGA

La señal de la placa iCEstick de 12 MHZ se introduce a clk_in a través del pin 21 de la fpga. La salida clk_out se envía directamente al led D1 (pin 99), para que parpadee a su misma frecuencia

![Imagen 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-5.png)

Para sintentizar ejecutamos el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 2 / 96
|PLBs      | 5 / 160
|BRAMs     | 0 / 16

Lo descargamos en la FPGA mediante:

    $ sudo iceprog prescaler.bin

El led D1 empezará a parpadear

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/T05-prescaler-iCEstick-1.png" width="400" align="center">

En este **vídeo de youtube** se puede ver el led parpadeando:

[![Click to see the youtube video](http://img.youtube.com/vi/HSqNa5iC2Qc/0.jpg)](https://www.youtube.com/watch?v=HSqNa5iC2Qc)

## Simulación

En el banco de pruebas colocamos el **prescaler de N bits** (por defecto con N = 2), un **generador de reloj** y un **bloque de comprobación** que se ejecuta con cada flanco de bajada del reloj. Este bloque tiene un registro interno que se incrementa y su bit más significativo se comprueba con clk_out, para asegurarse que está funcionando correctamente.  Hay un cuarto bloque que inicializa todo y espera a que se termine la simulación

![Imagen 6](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-6.png)

El código del banco de pruebas es el siguiente:

    //-- prescaler_tb.v
    module prescaler_tb();
    
    //-- Numero de bits del prescaler a comprobar
    parameter N = 2;
    
    //-- Registro para generar la señal de reloj
    reg clk = 0;
    
    //-- Salida del prescaler
    wire clk_out;
    
    //-- Registro para comprobar si el prescaler funciona
    reg [N-1:0] counter_check = 0;
    
    //-- Instanciar el prescaler de N bits
    prescaler #(.N(N))  //-- Parámetro N
      Pres1(
        .clk_in(clk),
        .clk_out(clk_out)
      );
    
    //-- Generador de reloj. Periodo 2 unidades
    always #1 clk = ~clk;

    //-- Comprobacion del valor del contador
    //-- En cada flanco de bajada se comprueba la salida del contador
    //-- y se incrementa el valor esperado
    always @(negedge clk) begin
    
      //-- Incrementar variable del contador de prueba
      counter_check = counter_check + 1;
    
      //-- El bit de mayor peso debe coincidir con clk_out
      if (counter_check[N-1] != clk_out) begin
        $display("--->ERROR! Prescaler no funciona correctamente");
        $display("Clk out: %d, counter_check[2]: %d", 
                  clk_out, counter_check[N-1]);
      end
    
    end
    
    //-- Proceso al inicio
    initial begin
    
      //-- Fichero donde almacenar los resultados
      $dumpfile("prescaler_tb.vcd");
      $dumpvars(0, prescaler_tb);
    
      # 99 $display("FIN de la simulacion");
      # 100 $finish;
    end
    endmodule

Para simular ejecutamos:

    $ make sim

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/T05-prescaler-sim-N-2.png)

En esta simulación del prescaler de 2 bits vemos cómo efectivamente la señal de salida tiene un periodo de 4 veces el de la señal de entrada.

Repetimos la simulación pero ahora estableciendo un prescaler de 3 bits, cambiando esta línea:

    parameter N = 2;

El resultado es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/T05-prescaler-sim-N-3.png)

Ahora la señal de salida tiene  un periodo 8 veces mayor que el de la entrada

## Ejercicios propuestos
* Modificar el prescaler para valores de N = 18, 19, 20 y 21. Sintetizarlos y descargarlos en la iCEstick
* Realizar la simulación para N = 4

## Conclusiones
TODO



