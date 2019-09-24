# Dead End_ para Amstrad CPC

Dead End_ está basado en Dead End, el juego del mismo nombre (pero sin el guión bajo) creado para Macintosh por Wolfgang Thaller entre los años 1993 y 1998. Esta versión para Amstrad CPC se ha portado a partir de la de [ZX Spectrum 48K](https://compiler.speccy.org/spectrum-dead-end_.html), que se presentó al Concurso de BASIC 2020 de Bytemaniacos.

El objetivo del juego es alcanzar la salida de cada mapa. Se aplican las siguientes reglas:

* Los muros no se pueden atravesar.
* Para empujar una caja, tiene que haber espacio tras el personaje (para poder coger carrerilla y empujar) y tras la caja, para que se pueda desplazar.
* No hay límite de movimientos.
* No se ha implementado la funcionalidad de deshacer un movimiento; si nos quedamos atascados solo podemos reiniciar el nivel desde el principio.

## Enfoque

La idea era hacer una traducción directa del listado en BASIC para ZX Spectrum. Como BASIC no es un estándar, cada ordenador de la época implementaba su propio dialecto, también influenciado por sus características técnicas.

Por lo tanto, en ningún caso se ha buscado optimizar el listado original ni modificar su arquitectura; simplemente se han sustituido las instrucciones no disponibles en el Locomotive BASIC del Amstrad CPC por instrucciones o conjuntos de instrucciones equivalentes.

## Herramientas

* El juego se ha desarrollado usando el emulador [Retro Virtual Machine](https://www.retrovirtualmachine.org/), inicialmente sobre un CPC 6128 Español y, posteriormente, se ha probado también en un CPC 464 Español.
* Retro Virtual Machine permite acceder al contenido de un fichero de disco (`.dsk`) tanto para extraer como para añadir ficheros. De esa forma, y guardando el listado en formato ASCII &mdash;`SAVE "FICHERO.BAS",A`, es posible editarlo desde el ordenador de forma cómoda. El fichero de cinta `.cdt` también se ha generado desde el emulador.

## Arquitectura del código

* El estado del mapa del juego se guarda en la variable `BUFFER`.
* Cada vez que se va a hacer un movimiento en una de las cuatro direcciones, se hacen las comprobaciones necesarias. Para ello, se leen siempre las dos casillas siguientes en el sentido del movimiento y la anterior (en sentido contrario):
  * Si la casilla siguiente está vacía, se hace el movimiento sin más.
  * Si la casilla siguiente está ocupada por un muro, no se puede mover.
  * Si la casilla siguiente está ocupada por una caja, y las otras dos que se miran están vacías, se empuja la caja.
* Hay puntos de entrada, marcados con líneas `REM` de comentarios, para lo que sería cada "función" del código. De esa forma el código queda más estructurado. 
* Se usa Modo 0 en todo el juego, menos en la pantalla de información, que se cambia a Modo 1 para que quepa más texto.

## Cambios respecto a la versión de ZX Spectrum

Sólo se reflejan los más relevantes.

### Diferencias de sintaxis

* En Locomotive BASIC la palabra clave `LET` es opcional. Sin embargo, se ha decidido mantenerla por minimizar el número de cambios.
* En Spectrum sólo hay un modo de vídeo. Para lo que queríamos hacer, en Amstrad nos encaja usar el `MODE 0`, en el que tenemos más colores a costa de perder resolución.
* En Spectrum, el color de tinta se selecciona de entre los disponibles con la palabra clave `INK`. En Amstrad se usa `PEN`, ya que `INK` sirve para seleccionar un color de la paleta de 16 simultáneos (en Modo 0) entre los 27 disponibles.
* En Spectrum los GDU (Gráficos Definidos por el Usuario) se introducen poniendo el cursor en un modo especial, `[G]`, y usando una de las letras del abecedario. En Amstrad se puede redefinir todo el juego de caracteres mediante el uso de la palabra clave `SYMBOL`.
* En Spectrum se usa la construcción `PRINT AT y,x` para situar el cursor en las coordenadas (x,y). El origen de coordenadas es el punto (0,0) que está en la esquina superior izquierda. En Amstrad se usa la palabra clave `LOCATE x,y`, o bien una secuencia de caracteres de control. El origen está en el mismo lugar pero es el punto (1,1).
* Amstrad carece de la palabra clave `INVERSE`, para pintar un carácter con los atributos invertidos. Hay que simularla con caracteres de control.
* Amstrad carece de la palabra clave `BRIGHT`, para modificar el brillo del color empleado. Por el contrario, en Modo 0 disponemos de 16 colores a elegir entre 27.
* En Spectrum, `GO TO` y `GO SUB` se escriben separado. Sus correspondientes en Amstrad son `GOTO` y `GOSUB`.
* En Spectrum, la palabra clave `DIM` define un vector de n elementos (con `DIM(n)`). Sin embargo, en Amstrad, sería un vector de n+1 elementos.
* En Amstrad, la palabra clave `RESTORE` no admite una variable como parámetro.


### Cambios en la implementación

* La pantalla del Spectrum es de 32x24 caracteres (256x192), mientras que la del Amstrad CPC en Modo 0 (el más colorido) es de 20x25 caracteres (160x200), con píxeles rectangulares de 2x1. Por tanto, ha habido que adaptar tanto los gráficos como la distribución de marcadores y textos a la resolución disponible.
* Otro cambio respecto a la resolución es que, en la versión de Spectrum, los objetos son de 2x2 caracteres, y el movimiento se hace carácter a carácter; por tanto, mover cada objeto a una posición adyacente implica 2 movimientos de un carácter. En Amstrad los objetos son de 1x2 caracteres. Para evitar que el movimiento vertical fuese la mitad de rápido que en horizontal, se ha hecho que en vertical se recorran dos caracteres en cada paso.
* En el listado de Spectrum se hace un `CLEAR 59999`, con lo que reservamos la memoria por encima de ese punto para que no la use el BASIC. Se define una variable llamada `BUFFER` como un puntero a la posición de memoria 60000, donde se almacenan los datos de estado del mapa de juego. En Amstrad hemos decidido usar un vector definido en una variable con el mismo nombre. El problema aparece con el algoritmo que usa el listado de Spectrum para calcular si es posible mover el muñeco y/o empujar una caja. En los bordes superior e inferior, es posible que intentemos leer fuera de la zona de datos. En la versión Spectrum no pasa nada porque, por cómo están construidos los mapas, el dato que se va fuera de rango no se usa, por lo que su valor da igual. En Amstrad, si hacemos lo mismo, obtendremos dos errores diferentes: "5 Improper argument" si usamos un índice negativo, o "9 Subscript out of range" si el índice se sale del rango de definición del vector. Para solventarlo, se ha implementado una subrutina en la línea 7000, que devuelve 0 si el puntero está fuera de rango.
* Para cargar los datos del nivel en el que estamos, en Spectrum lo hacemos pasando una variable a la palabra clave `RESTORE`. Como en Amstrad no se puede, se ha implementado una subrutina en la línea 8000.
* En Spectrum, los GDU se definen _pokeando_ la memoria. Los datos están definidos en la línea 9500 y siguientes. En Amstrad se usa la palabra clave `SYMBOL`. El código se ha ubicado a partir de la misma línea.
* En Spectrum los colores son fijos. En Amstrad, en Modo 0, podemos construir una paleta de 16 a partir de 27 disponibles. Se han usado como base los 8 primeros más parecidos a los de Spectrum, más otros 4 adicionales. El código está en la línea 9900 y siguientes.
* En Amstrad se han ubicado un par de líneas a partir de la 9990 para restaurar de forma fácil el Modo 2, con fondo azul y color amarillo, que es el más cómodo para editar el listado. Hay que tener en cuenta que, a diferencia del BASIC del Spectrum, basado en tokens, en Amstrad podríamos introducir una línea con errores de sintaxis, que se lanzaría en tiempo de ejecución.
* Parece ser que en el Locomotive BASIC 1.0 hay un bug a la hora de pintar una cadena si no lo hacemos ubicando el cursor en la columna 1. Se tienen en cuenta, cuando no debería, los caracteres de control a la hora de calcular si la cadena cabe en el espacio restante de línea. Y, si no cabe, la cadena se imprime en la línea siguiente. Eso no ocurre en el BASIC 1.1 del Amstrad CPC 6128, por lo que el error apareció al probar el juego en un CPC 464. La solución elegida ha sido pintar todas las cadenas a partir de la columna 1 y reposicionar el cursor con caracteres de control `CHR$(31)+CHR$(x)*CHR$(y)`.  Gracias a [Fran Gallego](https://github.com/lronaldo), nos puso tras la pista del error. 
