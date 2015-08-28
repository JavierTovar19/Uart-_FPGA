<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T17-tones/images/T17-tones-icestick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T17-tones)

## Introducción

Utilizaremos el **divisor de frecuencia** para **generar tonos** de 1, 2, 3 y 4Khz que se pueden escuchar conectando un altavoz o un zumbador a su salida

## Generación de un tono
En nuestro sistema digital tenemos una señal de reloj de entrada f_in, que en la iCEstick es de 12MHz. Para generar un tono de frecuencia f_t tenemos que calcular **el valor del divisor** usando la **fórmula**:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T17-tones/images/T17-formula-divisor.png)

Así, para generar un **tono de 1Khz** tenemos que aplicar un divisor de valor:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T17-tones/images/T17-calculo-divisor-1Khz.png)

En la siguiente tabla se muestran los valores de los divisores para generar tonos de 1, 2, 3 y 4KHz:

| Frecuencia del Tono   |  Valor del divisor
|-----------------------|---------------------
|  1KHz                 |  12000
|  2KHz                 |  6000
|  3KHz                 |  4000
|  4KHz                 |  3000

**Los divisores tienen que ser números enteros**. De manera que si al dividir f_in entre f_t **se obtiene un valor decimal, tendremos que redondearlo**

## tones.v: Descripción del hardware

## Síntesis y pruebas

[![Click to see the youtube video](http://img.youtube.com/vi/uMFJ4ET1wcg/0.jpg)](https://www.youtube.com/watch?v=uMFJ4ET1wcg)

## Ejercicios propuestos

## Conclusiones

## Referencias


