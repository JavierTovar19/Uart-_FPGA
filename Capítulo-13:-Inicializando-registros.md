![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T13-reg-init)

## Introducción
Ya podemos responder a la pregunta planteada en el capítulo 8 sobre **cómo realizar la inicialización de los registros**. Los registros sintetizados están a 0. Normalmente necesitamos cargar en ellos un valor inicial, y que luego funcionen para lo que hayan sido diseñados. En este capítulo mostraremos cómo hacerlo, usando las herramientas que ya conocemos: el **multiplexor** y el **inicializador**

## Inicializando registros

Partimos de un **registro genérico de N bits**, que ya conocemos, con una entrada din, una salida dout y una señal de reloj

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-2.png)

Queremos que se cargue con un valor inicial al principio y que luego funcione normalmente. Para hacerlo colocamos un **multiplexor de 2 a 1** en su entrada (para dividir la entrada en 2). Por una entrada del multiplexor ponemos el **valor inicial** y por la otra la entrada genérica del registro din2.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-3.png)

**Es muy importante que el valor inicial se introduzca por la fuente 0 del multiplexor**.

Ahora ya simplemente conectamos un inicializador a la entrada de selección del multiplexor.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-4.png)

 De esta forma, al arrancar, el inicializador emitirá un 0 y por la entrada din del registro llegará el valor inicial. En el siguiente franco de subida este valor inicial se captura y el inicializador pasa a 1, por lo que ahora se seleccionará la fuente 1, que será por donde vengan los datos del registro en el régimen normal de funcionamiento

## reginit.v: Secuenciador de 2 estados con registro

Vamos a reacer el circuito blink4 del capítulo 8. Este circuito hacía parpadear los 4 leds a la vez, produciendo la secuencia: 0000, 1111, 0000 ...

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/blink4-3.png)

Ahora lo vamos a mejorar haciendo que se pueda poner cualquier valor inicial en el registro, lográndose la secuencia INI, ~INI, INI ... (valor inicial y su negado alternativamente):

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-5.png)

La descripción de este circuito en Verilog es:

    //-- reginit.v
    module reginit(input wire clk, output wire [3:0] data);
    
    //-- Parametros del secuenciador:
    parameter NP = 23;        //-- Bits del prescaler
    parameter INI = 4'b1100;  //-- Valor inicial a cargar en registro
    
    //-- Reloj a la salida del presacaler
    wire clk_pres;
    
    //-- Salida del regitro
    reg [3:0] dout;
    
    //-- Entrada del registro
    wire [3:0] din;
    
    //-- Señal de seleccion del multiplexor
    reg sel = 0;
    
    //-- Registro
    always @(posedge(clk_pres))
      dout <= din;
    
    //-- Conectar el registro con la salida
    assign data = dout;
    
    //-- Multiplexor de inicializacion
    assign din = (sel == 0) ? INI : ~dout;
    
    //-- Inicializador
    always @(posedge(clk_pres))
      sel <= 1;
    
    //-- Prescaler
    prescaler #(.N(NP))
      PRES (
        .clk_in(clk),
        .clk_out(clk_pres)
      );
    
    endmodule

El multiplexor de 2 a 1 ha implementado usando el **operador ? :** (similar al del lenguaje C). Es un if-else abreviado:

    assign din = (sel == 0) ? INI : ~dout;

Es equivalente a:

    always @*
      if (sel == 0)
        din <= INI;
      else
        din <= ~dout;
    
pero la primera notación es más compacta

## Síntesis en la FPGA

Para sintetizarlo en la fpga conectaremos las salidas data a los leds, y la entrada de reloj a la de la placa iCEstick

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-1.png)

Sintetizamos con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 8 / 160
|BRAMs     | 0 / 16

Para cargar en la FPGA ejecutamos:

    $ sudo iceprog reginit4.bin

En este **vídeo de Youtube** se puede ver la salida de los leds:

[![Click to see the youtube video](http://img.youtube.com/vi/dYikGANv1t4/0.jpg)](https://www.youtube.com/watch?v=dYikGANv1t4)

## Simulación
El banco de pruebas es uno básico, que instancia el componente reginit, con 1 bit para el prescaler (para que la simulación tarde menos). Tiene un proceso para la señal de reloj y uno para la inicialización de la simulación

![Imagen 3]()

La simulación se realiza con:

    $ make sim

El resultado en gtkwave es:

![Imagen 4]()

TODO

## Ejercicios propuestos
* Ejercicio 1: Modificar el valor inicial para obtener una secuencia diferente

## Conclusiones
TODO


