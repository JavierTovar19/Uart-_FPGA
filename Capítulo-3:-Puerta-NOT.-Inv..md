<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/T03-inv-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T03-inv)

## Introducción
Modelaremos nuestro primer **circuito combinacional**: una **puerta inversora** (puerta NOT). 

Los circuitos combinacionales realizan operaciones con los bits de entrada y los sacan por las salidas. **No almacenan bits, sólo los transforman**. El más básico de todos es una puerta NOT, que tiene un bit de entrada y otro de salida. Por la salida siempre saca el inverso del bit de la entrada: El "1" lo transforma a "0"  y el "0" a "1".

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/inv-1.png)

Nuestro componente lo llamaremos INV. Su entrada es A y su salida B

## inv.v: Descripción del hardware

La descripción es muy parecida al componente hola mundo setbit.v, pero ahora tenemos una entrada y una salida:

    //-- inv.v
    //-- El componente tiene una entrada (A) y una salida (B)
    module inv(input A, output B);
    
    //-- Tanto la entrada como la salida son "cables"
    wire A;
    wire B;
    
      //-- Asignar a la salida la entrada negada
      assign B = ~A;
    
    endmodule

A la salida B le asignamos la entrada A negada. Usamos el **operador ~** delante de A para negar (mismo operador que en el lenguaje C).

Como se trata de un circuito combinacional, que NO almacena información, **tanto A como B se define como cables**: la información se transmite por ellos, pero no se almacena.

## Síntesis en la FPGA

La salida de la puerta la conectamos al pin donde está el LED D1 de la iCEstick. La entrada al pin 44, que está disponible en el puerto de expansión de la iCEstick

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/inv-2.png)

Al pin 44 le introduciremos un "0" o un "1" desde el exterior para encender o apagar el led. Esto lo hacemos conectando el pin a la alimentación (3.3v) o a tierra (0v, GND) mediante un cable externo

Realizamos la síntesis como siempre, ejecutando el comando make sint:

    $ make sint

Los recursos que esta puerta consume son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 2 / 96
|PLBs      | 1 / 160
|BRAMs     | 0 / 16

Ahora lo cargamos en la FPGA con:

    $ sudo iceprog inv.bin

## Probando la puerta inversora
La placa iCEstick tiene pines para su expansión, que dan acceso a algunos de los pines de la fpga. Es muy útil soldar una tira de pines hembra en estos pines para acceder fácilmente a ellos. En la imagen se muestra la tira soldada en los pines inferiores

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/inv-4.png)

El pin inferior izquierdo se corresponde con el pin 44 de la FPGA y es el que usaremos como entrada de la entrada inversora.  El pin inferior derecho es la alimentación de 3.3 y lo usaremos para introducir un "1" al inversor y el que está a su izquierda es GND, y nos servirá para introducir un "0"


## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO




