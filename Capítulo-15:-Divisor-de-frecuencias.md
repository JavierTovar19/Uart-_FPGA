## Introducción

Con los **prescalers** conseguimos obtener señales de reloj de una frecuencia menor. Sin embargo, con ellos **no podemos conseguir la frecuencia exacta** que queramos. Si queremos por ejemplo implementar un contador que se incremente cada segundo, ¿cómo lo podríamos hacer?. Necesitamos los **divisores de frecuencia**.

Con estos divisores conseguimos **generar señales de frecuencias exactas**. Son muy importantes para aplicaciones de comunicaciones (uarts, spi, bus i2c...), refresco de pantallas gráficas, generación de señales PWM, cronómetros, temporizadores, generación de tonos audibles, etc.

En este capítulo ....

## El divisor de frecuencias

Dividir la frecuencia de una señal entre 2 es muy sencillo: colocamos un prescaler de 1 bit. En general, **para divir entre cualquier potencia de 2** (2, 4, 8, 16...2^N) nos basta con un **prescaler de N bits**. Para el resto de frecuencias necesitamos el divisor de frecuencias

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divisor-1.png)

Es un componente que tiene una señal de entrada (clk_in), con frecuencia fin y periodo Tin. Como salida tiene otra señal (clk_out) cuya frecuencia es la de la **entrada dividida entre M**. O si lo vemos con el **periodo**, el de la señal de salida es **M veces mayor que el de la entrada**

## "Hola mundo": Divisor entre 3

 Comenzaremos por el divisor más pequeño posible, el **"hola mundo"**, un **divisor entre 3**. Obtendremos una señal con una frecuencia 3 veces menor. En el caso de probarlo en la placa iCEstick con el reloj de 12Mhz, obtendríamos una señal de 12/3 = 4Mhz. 

Las señales de entrada y salida son las siguientes:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divisor-2.png)

Vemos que clk_out tiene un periodo Tout 3 veces mayor que Tin (La tercera parte de su frecuencia). Aunque **el ciclo de trabajo es diferente**. Clk_in está el mismo tiempo a nivel alto que bajo, mientras que clk_out está dos tercios del periodo a nivel bajo y uno a nivel alto. Para temas de temporización, **el ciclo de trabajo es indiferente**. Lo importante es la frecuencia.

Un **divisor de frecuencia entre M** se implementa usando un **contador módulo M**, y tomando como **salida** el **bit de mayor peso**.

Para implementar este divisor hola mundo, hay que utilizar por tanto un **contador módulo 3**, como se muestra en el siguiente diagrama

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divisor-3.png)

### Contador módulo 3

En general,  un **contador módulo M** se inicializa al cabo de M pulsos. Empieza en 0 y va contando 1, 2, 3, 4... hasta M-1. En el siguiente pulso pasa de nuevo a 0 y vuelve a empezar

El contador módulo 3 repite la cuenta: 0, 1, 2, 0, 1, 2, 0, 1, 2...

El esquema del hardware para implementarlo se muestra a continuación:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divisor-4.png)

Un **registro de 2 bits** almacena la cuenta actual, que sale por data. A través del **comparador** se comprueba si el valor es igual a 2. Si es así, se activa la entrada de selección 1 por lo que la cuenta se inicializa a 0. Si no, se activa la otra entrada por la que entra la **salida del registro más 1**, y el registro se incrementa.

Este **contador módulo 3** se puede describir en **Verilog** de una forma muy sencilla:

    reg [3:0] data = 0;
    
    always @(posedge clk)
      if (data == 2) 
        data <= 0;
      else 
        data <= data + 1;

**NOTA**: estamos usando always @(posedge clk) en vez de always @(posedge(clk)). Son equivalentes

### div3.v: Descripción del hardware del divisor entre 3

Ya tenemos todos los elementos para implementar el divisor entre 3. El código completo es el siguiente:

    //-- div3.v
    module div3(input wire clk_in, output wire clk_out);
    
    reg [1:0] divcounter = 0;
    
    //-- Contador módulo 3
    always @(posedge clk_in)
      if (divcounter == 2) 
        divcounter <= 0;
      else 
        divcounter <= divcounter + 1;
    
    //-- Sacar el bit mas significativo por clk_out
    assign clk_out = divcounter[1];
    
    endmodule

### div3.v: Simulación

Para simularlo cremos un **banco de pruebas muy sencillo**, que simplemente instancie el componente, genere una señal de reloj e inicialice la simulación

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/div3_tb.png)

El código Verilog es:

    //-- div3_tb.v
    module div3_tb();
    
    //-- Registro para generar la señal de reloj
    reg clk = 0;
    wire clk_3;
    
    //-- Instanciar el divisor
    div3
      dut(
        .clk_in(clk),
        .clk_out(clk_3)
      );
    
    //-- Generador de reloj. Periodo 2 unidades
    always #1 clk = ~clk;
    
    //-- Proceso al inicio
    initial begin
    
      //-- Fichero donde almacenar los resultados
      $dumpfile("div3_tb.vcd");
      $dumpvars(0, div3_tb);
    
      # 30 $display("FIN de la simulacion");
      $finish;
    end
    endmodule

Ejecutamos el comando:

    $ make sim

El resultado de la simulación es:

Figure

## Divisor entre M


