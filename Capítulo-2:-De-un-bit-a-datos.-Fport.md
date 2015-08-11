<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T02-Fport/images/Fport-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T02-Fport)

## Introducción

Ahora en vez de 1 bit sacaremos 4, y los mostraremos por los leds. Se trata de un valor fijo, que está "cableado por hardware". Si queremos visualizar otro número por los leds, habrá que sintentizar otro circuito.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T02-Fport/images/Fport-1.png)

Este componente lo denominaremos Fport (Fixed port). Tiene un bus de salida de 4 bits, etiquetado como data, que está cableado al valor binario 1010

## Fport.v: Descripción del hardware

