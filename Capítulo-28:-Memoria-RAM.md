![]()

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T28-ram)

# Introducción
Las **memorias RAM** nos permiten almacenar datos y recuperarlos durante el funcionamiento del circuito. Son memorias donde podemos leer y escribir

Crearemos una **memoria RAM genérica**, síncrona, con una entrada de datos para escritura y una salida para lectura. Como ejemplo diseñaremos un **buffer de almacenamiento** de 16 bytes que cargue a través de datos provenientes del puerto serie, y una vez llenado, se vuelca su contenido también por el puerto serie.

# Memoria RAM genérica

La memoria RAM genérica la denominaremos genram

## Puertos y parámetros

Los puertos y parámetros se muestran esta figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/genram-1.png)

Los **parámetros** son los mismos que en la memoria ROM:
* **DW** (Data width): Anchura de los datos (en bits)
* **AW** (Address width): Anchura de las direcciones (en bits)
* **ROMFILE**: Fichero con el contenido inicial precargado de la ram

Los **puertos** son:
* **data_in**: Entrada de datos, para escritura
* **data_out**: Salida de datos, en la lectura
* **addr**: Dirección de acceso, tanto para escritura como para lectura
* **rw**: Modo de acceso: lectura (rw = 1) o escritura (rw = 0)

## Cronograma

## Descripción en verilog

# Ejemplo: Buffer

## Diagrama de bloques

## Descripción en verilog

## Simulación

## Síntesis y pruebas

# Ejercicios propuestos

# Conclusiones
TODO
