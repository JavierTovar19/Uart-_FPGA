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

Conectamos dos cables a los pines 3.3v y GND. Serán los que usemos para introducir "1"s y "0" por el pin 44:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/T03-inv-iCEstick-2.png" width="400" align="center">

Al conectar el cable de GND, el led se encenderá. Lo sacamos. Conectamos el cable de 3.3v. El led estará apagado.  Cuando no hay nada conectado, el resultado es aleatorio: puede estar encendido, apagado u oscilando entre ambos estados.  Esta es una de las reglas básicas de la electrónica digital: "No dejar los pines de entrada al aire. Siempre a '0' ó '1'.


## Simulación
En el banco de pruebas se instancia la puerta inversora y se le conecta el **cable dout** por la salida B.

Por la entrada introduciremos diferentes valores y comprobaremos qué se obtiene a la salida. Por ello, conectamos el **registro** denotado como **din**.

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/inv-3.png)

 A diferencia del cable, el **registro** funciona como una variable, a la que podemos asignar diferentes valores. Primero se introduce un '0' y se comprueba que la salida tenga un '1' (que es el negado). Y luego al revés: se introduce un '1' y se comprueba que se obtiene un '0'.

 En el banco de pruebas se instancia el inversor, conectando su entrada A al registro din y su salida B al cable dout. Desde el bucle principal se asignan los valores a din y se comprueba el valor de dout. 

    //-- inv_tb.v
    module inv_tb();
    
    //-- Registro de 1 bit conectado a la entrada del inversor
    reg din;
    
    //-- Cable conectado a la salida del inversor
    wire dout;
    
    //-- Instaciar el inversor, conectado din a la entrada A, y dout a la salida B
    inv NOT1 (
     .A (din),
     .B (dout)
    );
    
    //-- Comenzamos las pruebas
    initial begin
    
      //-- Fichero donde almacenar los resultados
      $dumpfile("inv_tb.vcd");
      $dumpvars(0, inv_tb);
    
      //-- Ponemos la entrada del inversor a 0
      #5 din = 0;
    
      //-- Tras 5 unidades de tiempo comprobamos la salida
      # 5 if (dout != 1)
            $display("---->¡ERROR! Esperado: 1. Leido: %d", dout);
    
      //-- Tras otras 5 unidades ponemos un 1 en la entrada
      # 5 din = 1;
     
      //-- Tras 5 unidades comprobamos si hay un 0 en la entrada
      # 5 if (dout != 0)
            $display("---> ¡ERROR! Esperado: 0. Leido: %d", dout);
    
      # 5 $display("FIN de la simulacion");
      # 10 $finish;
    end
    endmodule

Realizamos la simulación, ejecutando el comando make sim

    $ make sim

En la simulación vemos como dout es siempre el negado de din

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO




