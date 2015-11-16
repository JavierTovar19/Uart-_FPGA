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
| **HALT**     |     | Detener la ejecución. Se activa la señal stop que se muestra por el led verde
| **LEDS**     | _val_    | Escribir el dato _val_ en el puerto de salida de 4 bits, para visualizarlo en los leds
| **WAIT**     |    | Realizar una pausa de 100ms
| **JP**       | _dir_    | Saltar a la dirección de memoria indicada por el operador _dir_

## Memoria

Microbio dispone de una **memoria ROM** de **64 posiciones** donde se almacena el programa a ejecutar. Cada instrucción tiene un ancho de **8 bits**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-2.png)

## Formato de instrucciones

Todas las instrucciones de microbio tienen el mismo formato, que se muestra a continuación:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-3.png)

Constan de dos campos:
* **Código de operación** (CO): Campo de **2 bits** que indica el tipo de operación (HALT, LEDS, WAIT o JP)
* **Campo de datos** (DAT): Campo de **6 bits** que contiene el dato necesario para las instrucciones LEDS y JP

La tabla con los **códigos de operación** es:

| Instrucción  | Código de operación (binario)
|----------|-----------
| WAIT  | 00
| HALT  | 01
| LEDS  | 10
| JP    | 11

# Programas de ejemplo

Se muestran **tres programas de ejemplo** muy sencillo para probar el procesador microbio y aprender su programación: **M0.asm**, **M1.asm** y **M2.asm**

## Ensamblador de Microbio

Las instructiones que entiendo microbio están en **código máquina**: son números almacenados en su memoria rom, que el procesador va leyendo secuencialmente y ejecutando. Podemos programar microbio directamente en código máquina, escribiendo los **números hexadecimales** de sus instrucciones en el fichero **prog.list**

Sin embargo, es mucho más sencillo escribir programas en el **lenguaje ensamblador de Microbio** y utilizar un **programa ensamblador** para **traducirlos a código máquina**. Este proceso se denomina **ensamblado**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T30-microbio/images/microbio-4.png)

El ensamblador de microbio se llama masm.py, y ha sido programado en python 3.5. Para ensamblar el programa M0.asm,por ejemplo, hay que ejecutar el siguiente comando:

```
$ python3 masm.py M0.asm
Assembler for the MICROBIO microprocessor
Released under the GPL license

File prog.list successfully generated
```

Esto genera el fichero **prog.list** con el código máquina, que será cargado en la memoria rom de microbio al realizar la **síntesis / simulación**

Si se ejecuta con la opción _-verbose_ se obtiene más información:

```
$ python3 masm.py M0.asm -verbose
Assembler for the MICROBIO microprocessor
Released under the GPL license

File prog.list successfully generated

Symbol table:

Microbio assembly program:

[00] LEDS 0xF
[01] HALT

Machine code:

8F   //-- [00] LEDS 0xF
40   //-- [01] HALT
```
Nos devuelve la **tabla de símbolos** (en este caso no hay porque no se han definido etiquetas), el **programa en ensamblador**, ya procesado y el programa generado en **código máquina**

**Las fuentes del programa masm.py están disponibles** [en el repositorio de este capítulo](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T30-microbio), de forma que se pueda estudiar cómo funciona y sobre todo mejorarlo y ampliarlo. Por ejemplo, si se amplían las instrucciones de Microbio, será necesario que el ensamblador las soporte

## M0.asm: Hola mundo
## M1.asm: Secuencia en los leds
## M2.asm: Secuencia con repetición infinita

# Implementación de Microbio

# Programación de microbio

# Referencias

# Conclusiones
TODO