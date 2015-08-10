## Introducción

El circuito digital más sencillo es simplemente un cable que está conectado a un nivel lógico conocido, por ejemplo 1. De esta forma al conectar un led se iluminará (1) o se apagará (0)

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-1.png)

La salida de este circuito la hemos denominado A.

## setbit.v: Descripción del hardware

Para sintetizar este circuito en la fpga, primero lo tenemos que describir usando un lenguaje de descripción hardware (HDL). En este tutorial utilizaremos Verilog, ya que tenemos todas las herramientas libres para su simulación / síntesis.

Verilog es un lenguaje que sirve para describir hardware... pero ¡Cuidado! ¡NO ES UN LENGUAJE DE PROGRAMACIÓN! ¡Es un lenguaje de descripción! Nos permite describir las conexiones y los elementos de un sistema digital.

El código verilog que implementa este circuito "hola mundo" se encuentra en el fichero [setbit.v](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/T01-setbit/setbit.v). Tiene esta pinta:

    module setbit(output A);
    wire A;
    
      assign A = 1;
    
    endmodule

## Síntesis en la FPGA

Además del fichero en verilog del componente, necesitamos indicar a qué pin de la FPGA queremos conectar la salida A de nuestro componente. Este mapeo se realiza en el fichero setbit.pcf (pcf = Physical Constraint file). Lo sacaremos por el pin 99 que se corresponde con el led D1 de la placa ICEStick. Pero podría ser cualquier otro :

    set_io A 99

Sólo consta de una línea, en la que se asocia la etiqueta A del componente al pin 99 de la FPGA

En la figura 2 se muestra gráficamente esta idea.  Como lo que estamos describiendo es hardware, siempre es interesante hacerse esquemas y dibujos para comprenderlo mejor:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-2.png)

Para hacer la síntesis completa nos vamos al directorio _tutorial/T01-setbit_ y ejecutamos este comando desde la consola:

    $ make sint

Se nos genera el fichero setbit.bin que contiene la conguración de la FPGA para que se nos implemente nuestro circuito digital.

Lo descargamos en la fpga mediante el comando:

    $ sudo iceprog setbit.bin

El led D1 de la ICEStick se enciende

(Foto)

## Simulación


