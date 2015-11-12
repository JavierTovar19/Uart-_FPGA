![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T30-microbio)

# Introducción

Uno de los aspectos que más me apasionan de las FPGAs es la posibilidad de **crear tu propio microprocesador** y hacerlo funcionar en la **FPGA**. En este último capítulo diseñaremos un microprocesador extremadamente simple: **Microbio**. Sólo tiene 4 instrucciones y apenas nos permite hacer nada, sin embargo, es un punto de partida para poder ampliarlo con nuevas instrucciones y empezar a adentrarse en este apasionante mundo. **¿Te animas a hacer tu propio microprocesador?**

# Microprocesador Microbio

Microbio es un **procesador "hola mundo"**. Es extremadamente simple, pero es perfecto para comprender su funcionamiento: **leer instruciones de la memoria principal** y **ejecutarlas**

En realidad, Microbio es un **microcontrolador**: es un microprocesador que incluye la memoria y dos periféricos simples: un **puerto de 4 bits** para mostrar información por los leds y un **temporizador de 100ms**

## Características

* Memoria ROM de 64 bytes (6 bits para las direcciones)
* Instrucciones de 8 bits
* 4 instrucciones: HALT, LEDS, WAIT y JP
* No puede realizar cálculos: no tiene unidad aritmético lógica
* No tiene registros de propósito general para almacenar información o hacer operaciones
* Funcuencia de funcionamiento: Reloj de 12MHz
* Tiene 2 **periféricos**:
    * Puerto de salida de 4 bits: conectado a los leds
    * Temporizador de 100ms
* Indicador de programa terminado: El led verde se activa cuando se ejecuta la instrucción HALT que detiene el microprocesador

## Interfaz con el exterior

Microbio se comunica con el exterior mediante los siguientes pines:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-1.png)

* **clk**: Entrada de reloj. Se conecta el reloj de 12 MHz de la FPGA
* **rstn**: Entrada de reset. Está conectada a la señal DTR para poder hacer reset desde el PC. Al hacer un reset, microbio empieza a ejecutar el programa que tiene almacenado a partir de la dirección de memoria 0
* **stop**: Señal de salida, conectada al led verde de la placa Icestick. Se pone a 1 cuando Microbio ha ejecutado la instrucción HALT y el programa ha terminado
* **leds**: Puerto de salida de 4 pines. Está conectado a los 4 leds rojos de la icestick

## Instrucciones

Microbio puede ejecutar las 4 instrucciones siguientes:

| Instrucción  | Operando  | Descripción
|----------|-----------|--------------
| HALT     | no    | Detener la ejecución. Se activa la señal stop que se muestra por el led verde
| LEDS     | _val_    | Escribir el dato _val_ en el puerto de salida de 4 bits, para visualizarlo en los leds
| WAIT     | no   | Realizar una pausa de 100ms
| JP       | _dir_    | Saltar a la dirección de memoria indicada por el operador _dir_

## Memoria

Microbio dispone de una **memoria ROM** de **64 posiciones** donde se almacena el programa a ejecutar. Cada instrucción tiene un ancho de **8 bits**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-2.png)

## Formato de instrucciones

# Implementación de Microbio

# Programación de microbio

# Referencias

# Conclusiones
TODO