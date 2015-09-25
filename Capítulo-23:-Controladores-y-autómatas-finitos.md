![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T23-fsmtx)

# Introducción
Los **controladores** son la parte de los circuitos digitales que se **encargan de generar todas las señales** (microórdenes) **que gobiernan  el resto de componentes**. Se implementan mediante **autómatas finitos** (también denominadas **máquinas de estados**). En este capítulo diseñaremos un controlador para nuestro **transmisor serie**

# Arquitectura genérica

La **estructura** de los circuitos digitales la podemos descomponer en dos partes:  la **ruta de datos** y el **controlador**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-3.png)

* **Ruta de datos**: Es la parte que **procesa los datos**. Comprende todas las unidades funciones junto con sus conexiones para realizar ese procesamiento: registros de desplazamiento, unidades aritmético-lógicas, contadores, etc.

* **Controlador**: Es la **parte de gobierno**, que genera las señales necesarias, denominadas **microórdenes**, para que la ruta de datos se comporte adecuadamente. Envía microórdenes para poner en marcha un contador, cargar un dato en un registro, realizar tal operación oritmética, etc. Todo ello secuenciado en el tiempo.

El controlador **toma decisiones** y genera unas microórdenes u otras en función de los resultados parciales de la ruta de datos. Por ejemplo, si un contador ha alcanzado un determinado valor, entonces se deberá general tal señal para hacer tal cosa.  Todo eso lo hace el controlador.

Los controladores se implementan mediante [autómatas finitos](https://es.wikipedia.org/wiki/Aut%C3%B3mata_finito) (también denominados máquinas de estados). Es necesario primero definir **los estados** del circuito y **las transiciones** entre estos estados según las condiciones del circuito. Luego, en función de estos estados se generan las **microórdenes**. 

# Máquinas de estado en verilog

Partimos de este diagrama de 4 estados, para explicar cómo describirlo en Verilog

https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-4.png

**Inicialmente** el circuito se encuentra en el **estado 0** y el controlador generará las microórdenes necesarias (no mostradas en el dibujo). Mientras que la **señal a** esté a 0, se mantendrá siempre en ese estado. En cuanto se ponga a 1 se pasará al **estado 1**, donde se generarán otras microórdenes. En este estado, según el valor de la **señal b**, bien se volverá al estado inicial o se avanzará al **estado 2**. El estado 2 **no tiene condiciones**, por lo que **en el siguiente ciclo de reloj** se pasa al **estado 4**. Cuando la señal c valga 0, se vuelve al estado inicial.

# Arquitectura del transmisor serie
## Ruta de datos

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-1.png)

## Controlador


# Test

