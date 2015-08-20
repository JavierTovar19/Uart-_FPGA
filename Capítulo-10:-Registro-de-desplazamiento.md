![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T10-shif-register/images/shift4-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T10-shif-register)

## Introducción
Los registros de desplazamiento **almacenan un valor** y nos pemiten **desplazarlo**. Son extremadamente útiles. Se utilizan para convertir la información de paralelo a serie (y vice-versa) para usar en las **comunicaciones síncronas**. Las comunicaciones a través de SPI, I2C, etc, se implementan con estos registros. También nos permiten realizar las operaciones de multiplicar por 2 y dividir entre 2 para número enteros.

En este capítulo los utilizaremos para generar una secuencia de 4 estados en los leds de la placa iCEstick, moviendo las luces en sentido horario

## Descripción del registro
El registro de desplazamiento que usaremos es como el siguiente:

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T10-shif-register/images/shift4-2.png)

La **salida** del registro es **de N bits** (en nuestro ejemplo usaremos un registro de 4 bits). Tiene una **entrada en paralelo** de N bits, que nos permite cargar en el registro con un valor nuevo. La **señal load_shift** nos permite determinar el **modo de funcionamiento**: cuando está a 0 se realiza la **carga** de un valor nuevo al llegar un flanco de subida de reloj. Cuando está a 1 se realiza un **desplazamiento hacia la izquierda** en el flanco de subida del reloj.

En este desplazamiento **el bit más significativo se pierde**, y el nuevo se lee de la **entrada serin** (serial input). Si tenemos almacenado el valor inicial 1001, la señal load_shift está a 1, serin está a 0 y llega un flanco de subida de reloj, obtendremos el valor:  0010.  En el siguiente flanco (si serin sigue a 0) obtendremos 0100, luego 1000 y luego 0000.

## shift4: Rotación de bits

Como ejemplo usaremos un **registro de desplazamiento de 4 bits** para rotar una secuencia de bits y mostrarlos por los 4 leds de la placa iCEStick. La secuencia obtenida por los leds dependerá del valor inicial cargado en el registro.

El diagrama de bloques del componente **shift4** es:

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T10-shif-register/images/shift4-1.png)

El componente principal es un **registro de desplazamiento de 4 bits**. Por su entrada de reloj se conecta el reloj de la placa iCEstick a través de un **prescaler**, para disminuir su frecuencia y poder ver la rotación de los bits en los leds.

El bit más significativo del registro (**data[3]**) se conecta directamente a la entrada **serin**, de forma que se consiga la **rotación de bits** (el más significativo pasa a ser el de menor peso)

Por la entrada paralelo introducimos el **valor inicial**, que por defecto será el **0001**. Al rotar aparecerá la secuencia 0010, 0100, 1000 y 0001.

La carga inicial la realizamos usando un **inicializador**, como el mostrado en el capítulo 9. Está conectado a la entrada load_shift, de manera que inicialmente está a 0 y al llegar el primer flanco de reloj pasa a '1', cargándose el valor inicial y pasando a modo desplazamiento. En el resto de flancos se realizará el desplazamiento (y no la carga).  Este es un ejemplo de cómo funciona el inicializador.


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



