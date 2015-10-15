(dibujo)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T26-rom)

# Introducción
Las **memorias** nos permiten **almacenar información** para usarlas en nuestros circuitos: datos, instrucciones, configuraciones, etc. Son los **componentes esenciales** para crear circuitos más complejos, como por ejemplo **microprocesadores**.

Mostraremos cómo se modelan las **memorias ROM** (de sólo lectura) en verilog y haremos tres ejemplos muy sencillos: un hola mundo y dos ejemplos de **generación de secuencias en los leds**, para cargar en la placa ICEstick

# Memoria ROM 32x4

Comenzaremos por una **rom muy sencilla**, que puede almacenar 32 valores de 4 bits. Las direcciones de memoria van desde la 0 hasta la 31. Se necesitan **5 bits** para representarlas

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T26-rom/images/rom32x4-1.png)



# Ejemplo 1: ¡Hello world ROM!

# Ejemplo 2: Secuencia de luces

# Cargando la ROM desde un fichero

# Ejemplo 3: Otra secuencia