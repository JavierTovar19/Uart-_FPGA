<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/T08-blink4-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T08-register)

## Introducción
Los **registros** son **elementos esenciales** en los circuitos digitales. Nos pemiten almacenar información, desde 1 hasta N bits. Se utilizan para implementar procesadores, realizar segmentación, almacenamiento de resultados intermedios, etc.

El registro básico captura los datos de la entrada, en flanco de subida o bajada del reloj, los almacena y los saca por la salida. Su esquema es:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/blink4-2.png)

En este capitulo utilizaremos un registro de 4 bits para hacer que parpadeen los 4 leds de la placa iCEstick

## blink4: Encendiendo y apagando los 4 leds

El esquema que utilizaremos es el siguiente:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/blink4-3.png)

El registro de 4 bits inicialmente está inicializado a 0. Por la salida dout salen 4 bits a 0, que se convierten en 4 bits a 1 al pasar por el inversor. En el siguiente flanco de subida del reloj, se captura este nuevo valor 4'b1111 en el registro. Al salir por dout, se vuelve a invertir obteniéndose 4 bits a 0, como al principio.  El resultado es que se obtiene la siguiente secuencia  0000, 1111, 0000, 1111 ... que cambia con cada flanco de subida del reloj.  Si ahora la salida dout la conectamos a los leds, estos empezarán a encenderse y apagarse con la frecuencia del reloj.

## Descripción del hardware

El diseño completo a implementar en la FPGA se muestra en esta figura:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T08-register/images/blink4-1.png)

Para poder apreciar el parpadeo, se incluye un prescaler. La descripción del hardware en Verilog es:



## Síntesis en la FPGA




## Simulación

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO



