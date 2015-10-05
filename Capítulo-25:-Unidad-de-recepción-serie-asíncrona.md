![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T25-uart-rx)

# Introducción
Diseñaremos una **unidad de recepción serie asíncrona**, que nos permita recibir datos tanto del PC como de otro dispositivo de comunicación serie. Obtendremos el componente final, listo para usar en nuestros diseños

Para probarla haremos dos ejemplos: uno que **saca los 4 bits menos segnificativos** del dato recibido por **los leds** de la ICEStick. Y otro que **hace eco de todos los caracteres recibidos**. Los caracteres recibidos se envían al transmisor serie para que lleguen de nuevo al PC. Esto nos permitirá comprobar que efectivamente hemos recibido el dato correctamente.

# Módulo UART-RX
## Descripción de la interfaz

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-1.png)

## Cronograma
## Diagrama de bloques

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-2.png)

## Descripción en verilog

# Ejemplo 1: Visualizando en los leds
## Descripción 
## Simulación
## Síntesis y pruebas

# Ejemplo 2: eco
## Descripción
## Simulación
## Síntesis y pruebas

# Ejercicios propuestos
# Conclusiones
TODO
