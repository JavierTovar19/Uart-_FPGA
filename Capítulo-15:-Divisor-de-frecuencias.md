## Introducción

Con los **prescalers** conseguimos obtener señales de reloj de una frecuencia menor. Sin embargo, con ellos **no podemos conseguir la frecuencia exacta** que queramos. Si queremos por ejemplo implementar un contador que se incremente cada segundo, ¿cómo lo podríamos hacer?. Necesitamos los **divisores de frecuencia**.

Con estos divisores conseguimos **generar señales de frecuencias exactas**. Son muy importantes para aplicaciones de comunicaciones (uarts, spi, bus i2c...), refresco de pantallas gráficas, generación de señales PWM, cronómetros, temporizadores, generación de tonos audibles, etc.

## "Hola mundo": Divisor entre 3

Dividir la frecuencia de una señal entre 2 es muy sencillo: colocamos un prescaler de 1 bit. En general, **para divir entre cualquier potencia de 2** (2, 4, 8, 16...2^N) nos basta con un **prescaler de N bits**. Para el resto de frecuencias necesitamos el divisor de frecuencias. Comenzaremos por el más pequeño posible, el **"hola mundo"**, un **divisor entre 3**. Obendremos una señal con una frecuencia 3 veces menor. En el caso de probarlo en la placa iCEstick con el reloj de 12Mhz, obtendríamos una señal de 12/3 = 4Mhz. 

