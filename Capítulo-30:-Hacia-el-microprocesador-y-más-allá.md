dibujo)

# Introducción

Uno de los aspectos que más me apasionan de las FPGAs es la posibilidad de **crear tu propio microprocesador** y hacerlo funcionar en la **FPGA**. En este último capítulo diseñaremos un microprocesador extremadamente simple: **Microbio**. Sólo tiene 4 instrucciones y apenas nos permite hacer nada, sin embargo, es un punto de partida para poder ampliarlo con nuevas instrucciones y empezar a adentrarse en este apasionante mundo. **¿Te animas a hacer tu propio microprocesador?**

# Microprocesador Microbio

Microbio es un **procesador "hola mundo"**. Es extremadamente simple, pero es perfecto para comprender su funcionamiento: **leer instruciones de la memoria principal** y **ejecutarlas**

En realidad, Microbio es un **microcontrolador**: es un microprocesador que incluye la memoria y dos periféricos simples: un **puerto de 4 bits** para mostrar información por los leds y un **temporizador de 100ms**

## Características

* Memoria de 64 bytes (6 bits para las direcciones)
* Instrucciones de 8 bits
* 4 instrucciones: HALT, LEDS, WAIT y JP
* No puede realizar cálculos: no tiene unidad aritmético lógica
* No tiene registros de propósito general para almacenar información o hacer operaciones
* Tiene 2 **periféricos**:
    * Puerto de salida de 4 bits: conectado a los leds
    * Temporizador de 100ms
* Indicador de programa terminado: El led verde se activa cuando se ejecuta la instrucción HALT que detiene el microprocesador

## Pines

## Memoria

## Formato de instrucciones

## Instrucciones

# Implementación de Microbio

# Programación de microbio

# Referencias

# Conclusiones
TODO