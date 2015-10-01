(Dibujo diagrama ejemplo 1)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T24-uart-tx)

# Introducción
**Completaremos** la unidad de transmisión serie asíncrona y haremos dos **ejemplos de uso**, que envían la cadena "Hola!...". Uno de manera contínua al activarse la señal de DTR y otro cada segundo

Se trata de la **última versión de la unidad de transmisión**, que quedará lista para usar en nuestros proyectos

# Módulo UART-TX

La unidad de transmisión serie la encapsularemos dentro del **módulo uart-tx**

## Descripción de la interfaz

La unidad de transmisión tiene 4 entradas y 2 salidas:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-2.png)

* **Entradas**:
    * **clk**: Reloj del sistema (12MHz en la ICEstick)
    * **rstn**: Reset negado. Cuando rstn es 0, se hace un reset síncrono de la unidad de transmisión
    * **start**: Comienzo de la transmisión. Cuando está a 1, se captura el carácter que entra por data y se empieza a enviar
    * **data**:  Dato de 8 bits a enviar

## Cronograma

(Dibujo del cronograma de uso)

## Diagrama de bloques

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-1.png)

## Descripción del módulo en Verilog

# Ejemplo 1: Enviando cadenas continuamente

## Diagrama de bloques
## scicad1.v: Descripción en verilog
## Simulación
## Síntesis y pruebas

# Ejemplo 2: Enviando una cadena cada segundo
## scicad2.v: Descripción en verilog
## Simulación
## Síntesis y pruebas

# Ejercicios propuestos
* Modificar cualquiera de los dos ejemplos para enviar la cadena "Hola como estas "

# Conclusiones
TODO