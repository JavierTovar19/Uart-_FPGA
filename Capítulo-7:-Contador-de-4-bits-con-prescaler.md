<img src="" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T07-contador-prescaler)

## Introducción


![Imagen 1]()

## Descripción del hardware


## Síntesis en la FPGA

![Imagen 2]()


<img src="" width="400" align="center">

## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO





Borrador...

El contador de 26 bits del tutorial lo podemos ver como un contador de 4 bits cuyos bits se sacan por los leds y cuya señal de reloj proviene de un prescaler de 22 bits, con un reloj de entrada de 12Mhz

![Imagen 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T04-counter/images/counter-6.png)

Aplicando la fórmula anterior, la **frecuencia de reloj del contador de 4 bits es **fp = 12Mhz / 2^22 = **2.86 Hz**. Se incrementa cada Tp = 1 / fp = 1 / 2.86 =  **0.35 segundos**.