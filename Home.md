# Diseño Digital para FPGAs, con herramientas libres

## Descripción

Tutorial para aprender a diseñar **sistemas digitales sintetizables** en FPGAs usando **SOLO** herramientas libres. Descripción del hardware en lenguaje **Verilog**
* **Placa de desarrollo**: [iCEStick](http://www.latticesemi.com/icestick). Con la FPGA iCE40HX-1k de Lattice
* **Simulador de Verilog**: [ícarus Verilog](http://iverilog.icarus.com/) 
* **Visualizador de señales**: [Gtkwave](http://gtkwave.sourceforge.net/)
* **Sintetizador**: [Yosys](http://www.clifford.at/yosys/) ([Repo en github](https://github.com/cliffordwolf/yosys))
* **Place & route**: [Arachne-prn](https://github.com/cseed/arachne-pnr) (en github)
* **Utilidades y descarga en FPGA**: [Proyecto icestorm](http://www.clifford.at/icestorm/)

## TODO  
Why libre?  
Por donde íbamos? --> Retomar artículos sobre el tema  
--> convirtiendo hw en sw  
--> Hispalinux  
--> JCRA barcelona  

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

## FAQs
* **¿Por qué Verilog y no VHDL?**
Verilog es el lenguaje que soporta el [proyecto icestorm](http://www.clifford.at/icestorm/) de momento. Pero como es un proyecto libre, la comunidad lo irá ampliando y el VHDL se soportará tarde o temprano. Al día de hoy (sep-2015), para trabajar con herramientas libres y FPGAs, hay que usar Verilog
