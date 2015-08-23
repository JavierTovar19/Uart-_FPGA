
![Image 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/regreset-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T14-regreset)

## Introducción
Los registros son elementos muy usados para hacer circuitos digitales. Aunque su descripción en verilog es muy sencilla, crearemos un **registro genérico de N bits** que podremos instanciar en nuestros diseños. Esto nos permitirá **colocar muchos registros fácilmente**. Además, añadiremos una **entrada de reset síncrona** que cargará el registro con un valor inicial (Por defecto será 0 pero podremos indicar otro valor al instanciar el registro). Como ejemplo haremos un **secuenciador con 2 registros**, sacándose por los leds los datos almacenados en ellos, alternativamente.

## register.v: Registro de N bits, con reset síncrono

El esquema del registro es el mostrado en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/regreset-2.png)

Tiene una **entrada de datos din de N bits**, y su **salida dout** correspondiente. Los datos se capturan en **flanco de subida** de la señal de reloj. Dispone de una **entrada de reset síncrona activa a nivel bajo**: Si se recibe un 0 por esta señal y llega un flanco de reloj, el registro se carga con su valor inicial (INI), que se define al instanciarlo

Esta manera de inicilizar el registro ya la conocemos del capítulo anterior, pero ahora se ha integrado dentro del propio registro. Su descripción en Verilog es la siguiente:

    //-- register.v
    module register (rst, clk, din, dout);
    
    //-- Parametros:
    parameter N = 4;     //-- Número de bits del registro
    parameter INI = 0;   //-- Valor inicial
    
    //-- Declaración de los puertos
    input wire rst;
    input wire clk;
    input wire [N-1:0] din;
    output reg [N-1:0] dout;
    
    //-- Registro
    always @(posedge(clk))
      if (rst == 0)
        dout <= INI; //-- Inicializacion
      else
        dout <= din; //-- Funcionamiento normal
    
    endmodule

Esto es equivalente a la implementación en el capítulo anterior, donde usábamos dos procesos: uno para el multiplexor y otro para el registro. Aquí están los dos componentes dentro del mismo proceso. Es **la forma típica de implementar un registro con inicialización**

## regreset.v: Secuenciador con dos registros de 4 bits

Como ejemplo de prueba vamos a implementar un **secuenciador de 2 estados**, usando **2 registros**. Cada uno almacena inicialmente el valor a mostrar en los leds en cada estado. **Los registros están encadenados**, de manera que la salida de uno se conecta a la entrada del otro. De esta forma, cada vez que llega un flanco de  subida de reloj, los registros **intercambian sus valores**. La salida de uno de ellos está conectada los leds.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/regreset-3.png)

La salida del registro 0 sale al exterior (puerto data) pero también se envía a la entrada del registro 1, cuya salida está conectada a la del registro 0

En las entradas de reset de los registros se ha colocado **un inicializar** que general la señal escalón para realizar la **carga inicial** en el primer flanco de subida del reloj. Los cables de reset han dibujado en verde en la figura. En el resto de ciclos funcionan como registros normales, cargando lo que les llega por su entradas din

El reloj (cables rojos) se pasa a través de un prescaler y se introduce tanto en los registros como en el inicializador

El código Verilog es el siguiente:

    //-- regreset.v
    module regreset(input wire clk, output wire [3:0] data);
    
    //-- Parametros del secuenciador:
    parameter NP = 23;        //-- Bits del prescaler
    parameter INI0 = 4'b1001; //-- Valor inicial para el registro 0
    parameter INI1 = 4'b0111; //-- Valor inicial para el registro 1
    
    //-- Reloj a la salida del presacaler
    wire clk_pres;
    
    //-- Salida de los regitros
    wire [3:0] dout0;
    wire [3:0] dout1;
    
    //-- Señal de inicializacion del reset
    reg rst = 0;
    
    //-- Inicializador
    always @(posedge(clk_pres))
      rst <= 1;
    
    //-- Registro 0
    register #(.INI(INI0))
      REG0 (
        .clk(clk_pres),
        .rst(rst),
        .din(dout1),
        .dout(dout0)
      );
    
    //-- Registro 1
    register #(.INI(INI1))
      REG1 (
        .clk(clk_pres),
        .rst(rst),
        .din(dout0),
        .dout(dout1)
      );
    
    //-- Sacar la salida del registro 0 por la del componente
    assign data = dout0;
    
    //-- Prescaler
    prescaler #(.N(NP))
      PRES (
        .clk_in(clk),
        .clk_out(clk_pres)
      );
    
    endmodule

## Síntesis en la FPGA

Para sintetizarlo en la fpga conectaremos las salidas data a los leds, y la entrada de reloj a la de la placa iCEstick

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/regreset-1.png)

Sintetizamos con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 5 / 96
|PLBs      | 12 / 160
|BRAMs     | 0 / 16

Para cargar en la FPGA ejecutamos:

    $ sudo iceprog regreset.bin

Estos son los valores que se visualizan en los leds alternativamente:

| Valor 1  |  Valor 2
|-----------|----------
| [[https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/T14-regreset-leds-seq1.png|width=200px]] | [[https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/T14-regreset-leds-seq2.png|width=200px]]


En este **vídeo de Youtube** se puede ver la salida de los leds:

[![Click to see the youtube video](http://img.youtube.com/vi/TgnjJZ7BZEg/0.jpg)](https://www.youtube.com/watch?v=TgnjJZ7BZEg)

## Simulación
El banco de pruebas es uno básico, que instancia el componente regreset, con 1 bit para el prescaler (para que la simulación tarde menos). Tiene un proceso para la señal de reloj y uno para la inicialización de la simulación

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/regreset-4.png)

El código verilog es:

TODO

La simulación se realiza con:

    $ make sim

El resultado en gtkwave es:

![Imagen 4]()



## Ejercicios propuestos
* **Ejercicio 1**: 

## Conclusiones
TODO



