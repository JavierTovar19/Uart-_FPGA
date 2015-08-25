## Introducción

Con los **prescalers** conseguimos obtener señales de reloj de una frecuencia menor. Sin embargo, con ellos **no podemos conseguir la frecuencia exacta** que queramos. Si queremos por ejemplo implementar un contador que se incremente cada segundo, ¿cómo lo podríamos hacer?. Necesitamos los **divisores de frecuencia**.

Con estos divisores conseguimos **generar señales de frecuencias exactas**. Son muy importantes para aplicaciones de comunicaciones (uarts, spi, bus i2c...), refresco de pantallas gráficas, generación de señales PWM, cronómetros, temporizadores, generación de tonos audibles, etc.

En este capítulo implementaremos un divisor de frecuencia genérico y lo usaremos para hacer parpadear un led a la frecuencia de 1 Hz (una vez por segundo)

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

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/div3_sim.png)

Vemos cómo el periodo de la señal de salida clk_out es 3 veces el de la señal de entrada

### div3: Síntesis y pruebas

Para sintetizar conectamos el reloj de 12Mhz de la iCEstick a clk_in y mandamos clk_out a uno de los leds

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/div3-sintesis.png)

Como **la frecuencia de salida es de 4Mhz, NO veremos el led parpadear**, simplemente veremos que se enciende. Tendremos que conectar un osciloscopio para poder comprobar que efectivamente la señal es de 4Mhz

Sintetizamos con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 2 / 96
|PLBs      | 1 / 160
|BRAMs     | 0 / 16

Cargamos en la FPGA con el comando:

    $ sudo iceprog div3.bin

## Divisor entre M

Para conseguir más frecuencias de salida necesitamos usar un **divisor genérico**, que pueda dividir la frecuencia de entrada entre M. Es similar al divisor entre 3, con las siguientes consideraciones:

* Usa un **contador módulo M**
* Tiene un  **registro de N bits**, donde N está dimensionado para poder almacenar el número genérico M
* La señal clk_out será el bit de mayor peso del registro: el **bit genérico N - 1**

### divM.v: Descripción del hardware

El código verilog es el siguiente:

    //- divM.v
    module divM(input wire clk_in, output wire clk_out);
    
    //-- Valor por defecto del divisor
    //-- Como en la iCEstick el reloj es de 12MHz, ponermos un valor de 12M
    //-- para obtener una frecuencia de salida de 1Hz
    parameter M = 12_000_000;
    
    //-- Numero de bits para almacenar el divisor
    //-- Se calculan con la funcion de verilog $clog2, que nos devuelve el 
    //-- numero de bits necesarios para representar el numero M
    //-- Es un parametro local, que no se puede modificar al instanciar
    localparam N = $clog2(M);
    
    //-- Registro para implementar el contador modulo M
    reg [N-1:0] divcounter = 0;
    
    //-- Contador módulo M
    always @(posedge clk_in)
      if (divcounter == M - 1) 
        divcounter <= 0;
      else 
        divcounter <= divcounter + 1;
    
    //-- Sacar el bit mas significativo por clk_out
    assign clk_out = divcounter[N-1];
    
    endmodule

El contador módulo M es igual al módulo 3, pero ahora se usa la **constante M**. Para la conexión del bit más significativo del contador a clk_out se usa la **constante N**

**M es un parámetro del divisor**, que se puede establecer al instanciarlo desde otro componente. Por defecto se ha establecido a un valor de 12000000 (12M) para conseguir una frecuencia de salida de 1Hz:

fout = fin / M  =  12Mhz  / 12M = 1Hz

Para mejorar la legibilidad del valor, y no confundirse, **los dígitos** de los números en verilog se pueden **separar usando el guión bajo** (**_**).  De esta forma, el valor de 12 millones se puede escribir como 12_000_000

**La constante N es local**, y NO puede ser modificada fuera del módulo divM. Esto se consigue declarándola del tipo  **localparam**

**N define el número de bits necesarios para almacenar el valor M**. Por ejemplo, si M es 3, necesitamos 2 bits. Si M es 200, necesitamos 8 bits.  Esta cálculo se realiza usando la **función de Verilog $clog2(M)**. Matemáticamente se consigue haciendo el logaritmo en base 2 de M, y obteniendo el valor entero inmediatamente superior (operación ceiling). 

Este mismo **cálculo** se puede hacer en **python** de la siguiente forma:

    import math as m
    
    M = 12000000
    N = int( m.ceil( m.log(M,2) ) )
    print (N)

nos devuelve el resultado de **N = 24**. Es decir, que para **generar una señal de 1Hz necesitamos un contador de 24 bits**.

### divM.v: Síntesis

El esquema es el mismo que para el divisor entre 3: la entrada de reloj de 12Mhz se usa para conectar clk_in, y la señal de salida de 1Hz se saca por el led

Realizamos la síntesis ejecutando el comando:

    $ make sint2

(el 2 hace referencia al proyecto 2 el capítulo)

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 8 / 160
|BRAMs     | 0 / 16

Para cargar en la FPGA ejecutamos el comando:

    $ sudo iceprog divM.bin

El led empezará a parpadear con un frecuencia de exactamente 1 Hz: una vez por segundo

(VIDEO)

### divM.v: Simulación

El banco de pruebas es similar al del divisor entre 3, pero ahora al instanciar el divisor genérico establecemos su parámetro M. Para que la simulación no lleve mucho tiempo, usaremos **valores bajos de M**. Comenzamos por un valor de **M = 5**, para dividir la frecuencia de la señal entre 5

    //-- divM.v
    module divM_tb();
    
    //-- Registro para generar la señal de reloj
    reg clk = 0;
    wire clk_out;
    
    //-- Instanciar el componente y establecer el valor del divisor
    divM #(5)
      dut(
        .clk_in(clk),
        .clk_out(clk_out)
      );
    
    //-- Generador de reloj. Periodo 2 unidades
    always #1 clk = ~clk;
    
    //-- Proceso al inicio
    initial begin
    
      //-- Fichero donde almacenar los resultados
      $dumpfile("divM_tb.vcd");
      $dumpvars(0, divM_tb);
    
      # 30 $display("FIN de la simulacion");
      $finish;
    end
    endmodule

Para realizar la simulación, ejecutamos:

    $ make sim2

La salida de gtkwave es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divM_sim_M5.png)

Vamos a comprobar otro valor del divisor: M = 7.  En el banco de pruebas hay que modificar:

    divM #(7)

Ahora la salida de gtkwave es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T15-divisor/images/divM_sim_M7.png)

Efectivamente, la señal de salida tiene un periodo igual a 7 ciclos de la señal de entrada

## Ejercicios propuestos
* **Ejercicio 1**: Modificar el valor del divisor para que el led parpadee a una frecuencia de 0.5Hz
* **Ejercicio 2**: Pensar cómo se puede hacer que el led parpadee a una frecuencia de 1Hz pero con una señal con ciclo de trabajo del 50%, es decir, que el led esté encendido durante 0.5seg y apagado otros 0.5seg

## Conclusiones
TODO