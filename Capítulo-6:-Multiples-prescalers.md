![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T06-multiples-prescalers/images/mpres-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T06-multiples-prescalers)

## Introducción
Mediante el **diseño jerárquico** podemos reutilizar componentes ya probados para construir otros más complejos. En este capítulo haremos parpadear 4 leds de manera independiente, cada uno con su propio prescaler.

  En el capítulo anterior hemos creado un **prescaler paramétrico**, que tiene el número de bits como parámetro. Instanciaremos 5 prescalers, con diferente número de bits, para controlar de manera independiente los leds, y que cada uno pueda parpear a una frecuencia diferente. Según las frecuencias usadas, podremos crear diferentes patrones en los leds.

  La arquitectura es la mostrada en el diagrama del comienzo del capítulo. Hay **4 prescalers independientes** que controlan cada led. Su señales de entrada, en vez de provenir directamente de la señal de 12Mhz de la placa, vienen a su vez de otro prescaler. Esto nos permite establecer **la frecuencia base de los leds**. Luego, cada prescaler divide esta señal para obtener la suya propia. Esto nos permite ahorrar bits y hacer que los prescalers tengan menos bits.

## Descripción del hardware

El componente se ha denominado **mpres.v** (multiple prescalers). Tiene 5 parámetros, N0, N1, N2, N3 y N4 que se corresponden con el tamaño en bits de cada uno de los 5 prescalers. Tiene una señal de reloj de entrada y 4 salidas para controlar el parpadeo de los 4 leds.

La descripción del hardware se obtiene directamente de transcribir la imagen 1 a código Verilog:

    //-- mpres.v
    module mpres(input clk_in, output D1, output D2, output D3, output D4);
    wire clk_in;
    wire D1;
    wire D2;
    wire D3;
    wire D4;
    
    //-- Parametros del componente
    //-- Bits para los diferentes prescalers
    //-- Cambiar estos valores segun la secuencia a sacar por los leds
    parameter N0 = 21;  //-- Prescaler base
    parameter N1 = 1;
    parameter N2 = 2;
    parameter N3 = 1;
    parameter N4 = 2;
    
    //-- Cable con señal de reloj base: la salida del prescaler 0
    wire clk_base;
    
    //-- Prescaler base. Conectado a la señal de reloj de entrada
    //-- Su salida es por clk_base
    //-- Tiene N0 bits de tamaño
    prescaler #(.N(N0))  
      Pres0(
       .clk_in(clk_in),
       .clk_out(clk_base)
      );
    
    //-- Canal 1: Prescaner de N1 bits, conectado a led 1
    prescaler #(.N(N1))
      Pres1(
        .clk_in(clk_base),
        .clk_out(D1)
      );
    
    //-- Canal 2: Prescaler de N2 bits, conectado a led 2
    prescaler #(.N(N2))
      Pres2(
        .clk_in(clk_base),
        .clk_out(D2)
      );
    
    //-- Canal 3: Prescaler de N3 bits, conectado a led 3
    prescaler #(.N(N3))
      Pres3(
        .clk_in(clk_base),
        .clk_out(D3)
      );
    
    //-- Canal 4: Prescaler de N4 bits, conectado a led 4
    prescaler #(.N(N4))
      Pres4(
        .clk_in(clk_base),
        .clk_out(D4)
      );
    
    endmodule






Borrador...

El contador de 26 bits del tutorial lo podemos ver como un contador de 4 bits cuyos bits se sacan por los leds y cuya señal de reloj proviene de un prescaler de 22 bits, con un reloj de entrada de 12Mhz

![Imagen 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-6.png)

Aplicando la fórmula anterior, la **frecuencia de reloj del contador de 4 bits es **fp = 12Mhz / 2^22 = **2.86 Hz**. Se incrementa cada Tp = 1 / fp = 1 / 2.86 =  **0.35 segundos**.