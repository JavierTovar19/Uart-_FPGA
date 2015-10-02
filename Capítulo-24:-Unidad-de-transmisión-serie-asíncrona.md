![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-3.png)

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

* **Salidas**:
    * **tx**: Salida serie del dato. Conectar a la línea de transmisión
    * **ready**: Estado de la transmisión. Cuando ready es 1, la unidad está lista para transmitir. Y empezará en cuanto start se ponga a 1. Si ready es 0 la unidad está ocupada con el envío de un carácter

## Cronograma

Para transmitir primero se poner el **carácter de 8 bits en la entrada _data_** y **se activa la señal _start_**. Se comienza a transmitir por _tx_. La **señal _ready_** se pone a **0** para indicar que **la unidad está ocupada**. Cuando el carácter se ha terminado de enviar, **_ready_ se pone de nuevo a 1**.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-4.png)

## Diagrama de bloques

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-1.png)

## Descripción del módulo en Verilog

# Ejemplo 1: Enviando cadenas continuamente

## Diagrama de bloques

El diagrama de bloques del transmisor se muestra en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T24-uart-tx/images/scicad-3.png)

El diseño, como hicimos en el capítulo pasado, está dividido en su **ruta de datos** y su **controlador**. Hay **dos microórdenes** que genera el controlador: _bauden_ y _load_, con las que activa el temporizador de bits y la carga del registro de desplazamiento respectivamente. _Load_ también se usa para poner a cero el contador de bits

El dato a transmitir se recibe por _data_, y se registra para cumplir con las **normas del diseño síncrono**. El controlador genera también la **señal ready** para indicar cuándo se ha terminado de transmitir

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