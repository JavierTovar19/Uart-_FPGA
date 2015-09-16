(dibujo)

Ejemplos de este capítulo en github

## Introducción
En este capítulo se explican algunas **reglas de diseño síncrono** [1] para la **creación de circuitos digitales complejos**. Su aplicación nos evitará problemas según crece la complejidad de nuestros circuitos. Las aplicaremos para **mejorar el transmisor serie** que se esbozó en el capítulo anterior

## Sistemas síncronos
Los **sistemas digitales complejos son síncronos**: existe **una señal de reloj** (clk) que marca los tiempos en los que el resto de señales cambian. Es un reloj "corazón", al ritmo de cuyos pulsos se transportan y modifican los bits. En la placa iCEstick, nuestro reloj es de 12Mhz

## Mejoras en el transmisor serie
* Reloj del sistema (clk) en vez de clk_baud (reglas de diseño síncrono)





## Referencias
[1]: "Diseño síncrono de Circuitos Digitales". Miguel Angel Freire Rubio. ETSIST. UPM. Septiembre 2008 ([PDF](http://www.etsist.upm.es/uploaded/464/DS_Sep_2008.pdf))