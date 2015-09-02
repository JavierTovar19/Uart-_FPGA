![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/8bf3e940aebcf444a33c0345be1fcec0de6c5f32/tutorial/T19-secnotas/images/secnotas-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T19-secnotas)

## Introducción
Hasta ahora hemos generado sonidos en canales diferentes, de manera independiente.  Implementaremos un **secuenciador** que nos permita tocar **4 notas** que sonarán **consecutivamente** en el altavoz. Reproduciremos la secuencia: DO, RE, MI, (silencio) y se volverá al comienzo. Esto nos abre la puerta a la creación de melodías sencillas :-)

## secnotas.v: Descripción del hardware
Una manera de lograr la secuenciación de las notas es utilizar un **multiplexor**, con el que se seleccionará en cada momento el canal a escuchar

En este ejemplo, tocaremos las notas **DO**, **RE**, **MI**, que se introducirán por las entradas 0, 1 y 2 respectivamente del multiplexor. Por la cuarta pondremos un "0" para que haya **silencio**.

La entrada de selección la conectamos a un **contador de 2 bits**, de forma que seleccione alternativamente los canales 0, 1, 2 y 3. Este contador tendrá una entrada de reloj que determinará **la duración de la nota**, que establecemos en **250ms**.

El esquema del circuito se muestra en esta figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T19-secnotas/images/secnotas-2.png)


