(dibujo)

Enlace carpeta github

## Introducción

Empezaremos el diseño de nuestra UART por el **transmisor**. En este capítulo nos centraremos en la **transmisión de bits**, para entenderla perfectamente. Como ejemplo haremos un **circuito que envíe el un carácter al PC** cada segundo, a la velocidad de **115200 baudios**.

## Transmitiendo bits
Los bits se envían al PC de **uno en uno** a través del **pin Tx**. Los datos no se envían aislados, sino que están metido en **una trama**. El **estándar de transmisión** serie define diferentes tramas. Nosotros usaremos la típica, conocida como **8N1** (8 bits de datos, Ninguno de paridad y 1 bit de stop) que tiene el siguiente **formato**:

(dibujo)

La trama comienza con un **bit a 0**, que se llama **bit de start**. A continuación están los **8 bits del dato a transmitir**, pero **comenzando por el bit 0** (la transmisión se hace comenzando por el bit de menor peso, 0, hasta el mayor, 7). La trama finaliza con un **bit a 1**, llamado **bit de stop**.

Así, para transmitir un dato, la línea (tx) tomará lo siguientes valores. Inicialmente estará en reposo (tx = 1). Se transmite primero el bit de start, por lo que tx = 0. A continuación el bit de menor peso: tx = bit0, luego el siguiente, tx = bit1, y el siguiente tx = bit2 hasta llegar al de mayor peso tx = bit7. Por último se envía el bit de stop, poniendo tx = 1. tx Permanece a 1 hasta que se envíe la siguiente trama.

Como ejemplo veremos cómo transmitir el caracter ASCII "K". Su valor en hexadecimal es 0x4B y en binario: 01001011. La línea de transmisión a lo largo del tiempo tendrá esta forma:

(dibujo)

**Todos los bits tienen la misma duración**, que denominaremos **periodo de bit** (Tb)

