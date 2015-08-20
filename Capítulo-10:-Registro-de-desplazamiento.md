![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T10-shif-register/images/shift4-2.png)
[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T10-shif-register)

## Introducción
Los registros de desplazamiento **almacenan un valor** y nos pemiten **desplazarlo**. Son extremadamente útiles. Se utilizan para convertir la información de paralelo a serie (y vice-versa) para usar en las **comunicaciones síncronas**. Las comunicaciones a través de SPI, I2C, etc, se implementan con estos registros. También nos permiten realizar las operaciones de multiplicar por 2 y dividir entre 2 para número enteros.

En este capítulo los utilizaremos para generar una secuencia de 4 estados en los leds de la placa iCEstick, moviendo las luces en sentido horario

## Descripción del registro
El registro de desplazamiento que usaremos es como el siguiente:

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T10-shif-register/images/shift4-2.png)

La **salida** del registro es **de N bits** (en nuestro ejemplo usaremos un registro de 4 bits). Tiene una **entrada en paralelo** de N bits, que nos permite cargar en el registro con un valor nuevo. La **señal load_shift** nos permite determinar el **modo de funcionamiento**: cuando está a 0 se realiza la **carga** de un valor nuevo al llegar un flanco de subida de reloj. Cuando está a 1 se realiza un **desplazamiento hacia la izquierda** en el flanco de subida del reloj.



## Descripción del hardware
![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T10-shif-register/images/shift4-1.png)

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



