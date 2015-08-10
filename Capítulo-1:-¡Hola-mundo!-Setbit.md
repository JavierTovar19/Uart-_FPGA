## Introducción

El circuito digital más sencillo es simplemente un cable que está conectado a un nivel lógico conocido, por ejemplo 1. De esta forma al conectar un led se iluminará (1) o se apagará (0)

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-1.png)

La salida de este circuito la hemos denominado A.

Para sintetizar este circuito en la fpga, primero lo tenemos que describir usando un lenguaje de descripción hardware (HDL). En este tutorial utilizaremos Verilog, ya que tenemos todas las herramientas libres para su simulación / síntesis.

Verilog es un lenguaje que sirve para describir hardware... pero ¡Cuidado! ¡NO ES UN LENGUAJE DE PROGRAMACIÓN! ¡Es un lenguaje de descripción! Nos permite describir las conexiones y los elementos de un sistema digital.

El código verilog que implementa este circuito "hola mundo" se encuentra en el fichero [setbit.v](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/T01-setbit/setbit.v). Tiene esta pinta:

    module setbit(output A);
    wire A;
    
      assign A = 1;
    
    endmodule


