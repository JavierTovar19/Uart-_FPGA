
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

Dibujo


