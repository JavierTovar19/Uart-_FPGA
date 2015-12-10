![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/tutorial-fpga-logo.png)

# Diseño Digital para FPGAs, con herramientas libres

## Descripción

Tutorial para aprender a diseñar **sistemas digitales sintetizables** en FPGAs usando **SOLO** herramientas libres. Descripción del hardware en lenguaje **Verilog**
* **Placa de desarrollo**: [iCEStick](http://www.latticesemi.com/icestick). Con la FPGA iCE40HX-1k de Lattice
* **Simulador de Verilog**: [ícarus Verilog](http://iverilog.icarus.com/) 
* **Visualizador de señales**: [Gtkwave](http://gtkwave.sourceforge.net/)
* **Sintetizador**: [Yosys](http://www.clifford.at/yosys/) ([Repo en github](https://github.com/cliffordwolf/yosys))
* **Place & route**: [Arachne-prn](https://github.com/cseed/arachne-pnr) (en github)
* **Utilidades y descarga en FPGA**: [Proyecto icestorm](http://www.clifford.at/icestorm/)

## Comunidad: FPGA-WARS
Únete al grupo [FPGA-WARS: explorando el lado libre](https://groups.google.com/forum/#!forum/fpga-wars-explorando-el-lado-libre) para conocer las últimas noticias, preguntar, compartir y colaborar en el desarrollo de hardware libre reconfigurable

## Repo

[Repositorio con el código verilog](https://github.com/Obijuan/open-fpga-verilog-tutorial)

## Autor
[Juan González Gómez](http://obijuan.github.io/) (Obijuan)

## Licencia
<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T00-Intro/images/bq-logo-cc-sa-small-150px.png">

Licensed under a  [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/)

## Créditos
* [Proyecto icestorm](http://www.clifford.at/icestorm/ ), por Clifford Wolf and Mathias Lasser
* [Arachne-pnr](https://github.com/cseed/arachne-pnr), por Cotton Seed 

## Agradecimientos
* Este tutorial no habría sido posible sin las increibles herramientas libres del proyecto Icestorm y Arachne-pnr, por **Clifford Wolf**, **Mathias Lasser** y **Cotton Seed**. Representan un avance espectacular para el patrimonio tecnólogico de la humanidad ¡Muchísimas gracias! 

* A [BQ](http://bq.com), por financiar el material usado en el tutorial. Soy un verdadero privilegiado por trabajar en una empresa donde fomentan el espíritu maker y apoyan las iniciativas de exploración y experimentación con tecnologías Libres ¡Muchas gracias!

* A **Jesús Arroyo**, por hacer la compra de las placas iCEStick y empezar con el empaquetado para Debian/Ubuntu de las herramientas Arachne-pnr y las del proyecto icestorm. ¡Muchas Gracias!

* A **David Sánchez Falero**, por enseñarme cómo resaltar código Verilog en los markdown de github :-) ¡Esto ha hecho que el código sea muchísimo más legible! ¡Muchas Gracias! 

* A **Carlos García Saura***, por las instrucciones de instalación de las herramientas icestorm en Fedora. ¡Muchas gracias!

## FAQs
* **¿Qué diferencia hay entre una FPGA y Arduino?**  
 Arduino es una placa que lleva un microcontrolador, que se programa.  Las FPGAs NO SE PROGRAMAN, se configuran para implementar en ellas HARDWARE. Nos sirven para diseñar Circuitos

* **¿Qué hace una FPGA que no pueda hacer Arduino?**  
Muchas aplicaciones se pueden hacer tanto por software (programando) o por hardware. Por ejemplo un contador. La diferencia está en que al hacerlo por hardware conseguimos velocidades muchísimo mayores. Además, el hardware se puede "clonar" directamente y funciona en paralelo. Tener en software varias cosas funcionando en paralelo no es inmediato.

* **¿Por qué Verilog y no VHDL?**  
Verilog es el lenguaje que soporta el [proyecto icestorm](http://www.clifford.at/icestorm/) de momento. Pero como es un proyecto libre, la comunidad lo irá ampliando y el VHDL se soportará tarde o temprano. Al día de hoy (sep-2015), para trabajar con herramientas libres y FPGAs, hay que usar Verilog

* **¿Dónde puedo comprar la tarjeta IceStick**
-[Icestick en Mouser España ](http://www.mouser.es/new/Lattice-Semiconductor/lattice-icestick-kit/)
-[Icestick en LatticeUSA](http://www.latticesemi.com/icestick)

## Historia
* **2015-05-29**: La noticia sale en un [post en hack-a-day](http://hackaday.com/2015/05/29/an-open-source-toolchain-for-ice40-fpgas/). Me entero de su existencia a través de este post y empiezo a evaluar las herramientas (obijuan)
* **2015-05-27**: El [proyecto icestorm](http://www.clifford.at/icestorm/) alcanza el primer gran hito: ya se tiene un flujo de trabajo que funciona completamente usando sólo herramientas libres