# sgp
Simple Game Protocol

Protocolo para el intercambio de datos entre aplicaciones distribuidas en redes heterogeneas. Esta siendo utilizado en: http://juegos.jovenclub.cu/.

Elementos del protocolo.

El protocolo se centra en manejar sesiones de soluciones (videojuegos) y enviar los paquetes de datos a las soluciones que lo requieran en cada actualización del estado de las sesiones.

Está diseñado con una arquitectura cliente-servidor-cliente, de manera que los clientes intercambian paquetes a través de uno o varios servidores conectados en red.

A través del uso de unos pocos comandos se consigue implementar el protocolo. Estos comandos son: INIT, PLAY, START, STOP, RESET, SEND, RECV, GET, SET, NOTE, DOWN y HELLO.

INIT: lo envían los clientes al servidor para obtener los datos de las sesiones disponibles.
PLAY: lo envían los clientes al servidor para entrar o crear una sesión.
START: lo envían los clientes al servidor para arrancar el cronometro que indica el tiempo de juego.
STOP: lo envían los clientes al servidor para detener el cronómetro.
RESET: lo envían los clientes al servidor para reiniciar el cronómetro.
SEND: lo envían los clientes al servidor para que este envíe mensajes a otros clientes de la sesión.
RECV: lo envía el servidor a los clientes para entregar mensajes de otro cliente de la sesión.
GET: lo envían los clientes al servidor para obtener datos de la sesión a la que están conectados.
SET: lo envían los clientes al servidor para actualizar datos en la sesión en la que están conectados.
NOTE: lo envía el servidor a los clientes para notificarles sobre el cambio de estado de la sesión.
DOWN: lo envían los clientes al servidor para solicitar su salida de la sesión.
HELLO: lo envían los clientes al servidor para actualizar su permanencia en la sesión.

Comandos.

La línea de comandos está formada por el comando que envía el cliente o servidor, un identificador del cliente y los parámetros que describen la acción del comando. 

Los parámetros pueden aparecer en cualquier orden en el comando. Todos los comandos terminan en “\r\nEOM\r\n”.  Cada comando tiene parámetros que comparte con los demás y parámetros propios, como se describe a continuación.
INIT.

El comando INIT es enviado por los clientes a los servidores a los que se conectan. Es el primer comando que debe enviar un cliente al servidor al que se va a. En todos los comandos lo primero que se pasa es la profundidad de búsqueda en los servidores padres del servidor al que se conecta la aplicación cliente. El comando INIT va acompañado del identificador de la aplicación. Por ejemplo, el siguiente comando le solicita al servidor iniciar una sesión con una profundidad de búsqueda de dos, en un grupo de aplicaciones con identificador 153, la versión 1.0, la secuencia de mensajes número uno, la cantidad máxima de cuatro miembros de la sesión y un tiempo de inactividad en la sesión de 30 segundos:

“2\r\nINIT 153\r\nVersion:1.0\r\nSeq:1\r\nCount:4\r\nTime:30\r\nEOM\r\n”.

El número 2 es la profundidad de búsqueda en los servidores. Este es el nivel del juego; o sea si el juego se va a jugar a nivel de Centro (0), Provincial (1) o Nacional (2).

El número 153 es el identificador de la aplicación (videojuego), este es un número único que se le ha asignado a la aplicación a la hora de aprobar su creación. Los identificadores de aplicaciones son enteros sin signo (unsigned int).
La secuencia tiene que ser mayor que cero.

El número 4 es la cantidad máxima de clientes que se pueden conectar a la sesión. Por ejemplo, esto significa que pueden existir como máximo 4 jugadores en una sesión para el juego con ID 153.

El número 30 es el tiempo máximo en segundos que puede estar un cliente de la sesión sin actividad, después de este tiempo el cliente sale automáticamente de la sesión. El servidor le cierra la sesión al cliente después de este tiempo sin actividad.

A estos comandos el servidor responde con un mensaje de aceptación o de error. En caso que se acepte el mensaje el servidor responde OK y le envía al cliente las sesiones a las que puede ingresar, los datos de cada sesión disponible y los clientes conectados a esas sesiones. Si el servidor no envía ninguna sesión la aplicación cliente tiene que crear una sesión nueva en el comando PLAY. Algunos ejemplos de respuestas:

“200\r\nSeq:1\r\nEOM\r\n”

“200\r\nSeq:1\r\nSession:Domino prescolar\r\nData:5x5\r\nName:juan\r\nEOM\r\n”

En el primer caso el cliente tiene que crear una sesión nueva cuando ejecute el comando PLAY. Los nombres de la sesión pueden tener hasta 50 caracteres de longitud. En el segundo caso el cliente puede entrar solo a la sesión “Domino prescolar” que está disponible. En el campo Data el cliente recibe los datos que ha puesto la aplicación que ha creado la sesión. En el campo Name el cliente recibe el listado de los usuarios conectados a la sesión.

Si el servidor encuentra un error, lo devuelve. Por ejemplo, si no soporta la versión, devuelve:

“405\r\nSeq:1\r\nEOM\r\n”.

PLAY.

El comando PLAY lo envía el cliente para comenzar una sesión en el servidor. En el comando PLAY el cliente envía al servidor el ID de la aplicación, el nombre del cliente, la contraseña del cliente, el tiempo en que expira la sesión, la cantidad máxima de miembros de la sesión, el nombre de la sesión en que va a participar, los datos de la sesión, qué otros jugadores se puedan conectar a la sesión y la secuencia de mensajes. Por ejemplo, la aplicación 153 le envía el segundo mensaje, un cliente con nombre “Acuario”, la contraseña “algo*01”, un tiempo sin actividad en la sesión de 30 segundos y el nombre de sesión “Damas 2”:

“0\r\nPLAY 153\r\nSeq:2\r\nName:Acuario\r\nPass:qwertyuiopasdfghjklzxcvbnm12345\r\nTime:30\r\nSession:Damas 2\r\nData:8x8\r\nOthers:Juan\r\nCount:4\r\nEOM\r\n”

El número de la secuencia de mensajes tiene que ser mayor que cero.

Los nombres de cliente, contraseña y de sesión no pueden estar vacíos. La contraseña es en MD5.

El tiempo que demora la sesión sin actividad tiene que ser mayor que cero.

El campo Data de la sesión puede estar vacío. Pero lo pueden utilizar las aplicaciones que crean la sesión para dar más información acerca del estado de la sesión a otras aplicaciones que se van a unir a la sesión.

El campo Others contiene un listado separado por ',' de los jugadores que se pueden conectar a la sesión. Este campo puede estar vacío, en ese caso cualquier otro jugador puede conectarse a la sesión.

El servidor responde OK y el identificador de cliente que ha asignado el servidor al cliente en caso que el cliente haya iniciado sesión correctamente o el mensaje de error correspondiente. Por ejemplo, un mensaje OK con ID de cliente 342:

“200\r\nSeq:2\r\nID:342\r\nEOM\r\n”

A partir de este momento el cliente está en la sesión con un identificador 342. Este identificador es el que tiene que utilizar para identificarse en el servidor.

Un mensaje de error en que el servidor no encontró la sesión con espacio para que el cliente comience en esa sesión y por tanto no pudo crear un espacio para el cliente en esa sesión, es el siguiente:

“305\r\nSeq:2\r\nEOM\r\n”

START.

El comando START lo envía el cliente para comenzar un juego en la sesión a la que se ha conectado en el servidor. En el comando START el cliente envía al servidor el ID del cliente con que está jugando y la secuencia de los mensajes.

El mensaje START tiene el siguiente formato:

“0\r\nSTART 12\r\nSeq:123\r\nEOM\r\n”

El número 12 es el ID del cliente con que está jugando.

Si el mensaje fue resuelto con éxito en el servidor el cliente recibe una respuesta satisfactoria:

“200\r\nSeq:123\r\nEOM\r\n”

Si el mensaje no fue resuelto con éxito en el servidor el cliente recibe el error correspondiente.

STOP.

El comando STOP lo envía el cliente para terminar un juego en la sesión a la que se ha conectado en el servidor. En el comando STOP el cliente envía al servidor el ID del cliente con que está jugando y la secuencia de los mensajes.

El mensaje STOP tiene el siguiente formato:

“0\r\nSTOP 12\r\nSeq:123\r\nEOM\r\n”

El número 12 es el ID del cliente con que está jugando.

Si el mensaje fue resuelto con éxito en el servidor el cliente recibe una respuesta satisfactoria:

“200\r\nSeq:123\r\nEOM\r\n”

Si el mensaje no fue resuelto con éxito en el servidor el cliente recibe el error correspondiente.

RESET.

El comando RESET lo envía el cliente al servidor para reiniciar la sesión en la que está jugando. Este comando solo lo puede enviar el primer cliente que se encuentra conectado a la sesión; este se toma como el servidor del videojuego. En este caso se registra en la base de datos lo que se ha jugado hasta el momento. En el comando RESET el cliente envía al servidor el ID del cliente con que está jugando y la secuencia de los mensajes.

El mensaje RESET tiene el siguiente formato:

“0\r\nRESET 12\r\nSeq:123\r\nEOM\r\n”

El número 12 es el ID del cliente con que está jugando.

Si el mensaje fue resuelto con éxito en el servidor el cliente recibe una respuesta satisfactoria:

“200\r\nSeq:123\r\nEOM\r\n”

Si el mensaje no fue resuelto con éxito en el servidor el cliente recibe el error correspondiente.

SEND.

El comando SEND lo utilizan los clientes para enviar notificaciones a los demás clientes de la sesión. El comando le informa al servidor a qué clientes de una sesión enviar la información del comando. Los clientes a los que enviar la información pueden ser: uno (unicast), varios (multicast) o a todos (broadcast). Por ejemplo, el cliente 342, puede enviar un mensaje a todos los demás clientes con los que comparte sesión:

“0\r\nSEND 342\r\nSeq:3\r\nTo:\r\nInfo:23,18=C\r\nEOM\r\n”

O puede enviar el mensaje a los clientes 432 y 498 de  la misma sesión:

“0\r\nSEND 342\r\nSeq:3\r\nTo:432,498\r\nInfo:23,18=C\r\nEOM\r\n”

El servidor responde OK o el error correspondiente. Por ejemplo, OK:

“200\r\nSeq:3\r\nEOM\r\n”

El servidor no encontró el cliente 498:

“352\r\nSeq:3\r\nInfo:498\r\nEOM\r\n”

El cliente que no se encuentre conectado cuando el servidor le debe enviar un mensaje, lo pierde. Es necesario que los clientes se encuentren conectados para recibir todas las notificaciones del servidor y los mensajes de los demás clientes de la sesión.

RECV.

El comando RECV lo utilizan los servidores para enviar los mensajes de un cliente a los demás clientes que lo necesiten. Se utilizan en función del comando SEND. Cuando un cliente envía un comando SEND al servidor, en el parámetro To especifica a qué otros clientes de la sesión el servidor tiene que enviar la información contenida en el parámetro Info. El servidor envía a todos los clientes en To, a través del comando RECV, la información contenida en Info. Este comando lo genera el servidor y tiene un formato como el siguiente:

“0\r\nRECV 432\r\n23,18=C\r\nEOM\r\n”

Cuando este comando lo recibe un cliente, lo lee de la siguiente manera: el cliente 432 me envió el mensaje “23,18=C”.
A este comando el cliente no le tiene que responder al servidor, lo recibe y hace con él lo que corresponda.

GET.

Este comando lo utilizan los clientes para obtener información del estado de la sesión en que participan. Los clientes lo utilizan de la siguiente forma: en cada línea solicitan la información que necesitan del cliente. Por ejemplo, para pedir el nombre de la sesión en que se encuentran:

“0\r\nGET 342\r\nSeq:4\r\nSession:\r\nEOM\r\n”

El servidor llena los datos pedidos por el cliente en el mensaje de respuesta si todo está OK:

“200\r\nSeq:4\r\nSession:Damas 2\r\nEOM\r\n”

El cliente puede hacer varias peticiones en un solo GET, por ejemplo puede pedir el nombre de la sesión, el ID de todos los participantes en la sesión y sus nombres y el tiempo que dura la sesión sin actividad:

“0\r\nGET 342\r\nSeq:5\r\nSession:\r\nID:\r\nName:\r\nTime:\r\nEOM\r\n”

El servidor responde llenando todos los datos pedidos, si no hay error:

“200\r\nSeq:5\r\nSession:Damas 2\r\nID:342,438\r\nName:Pepe,Yaque\r\nTime:30\r\nEOM\r\n”

El servidor puede responder con error a un comando:

“0\r\nGET 342\r\nSeq:4\r\nSession:\r\nEOM\r\n”

“352\r\nSeq:4\r\n EOM\r\n”

Si el servidor no entiende algún dato pedido no lo devuelve, como en el siguiente ejemplo con Sesion:

“0\r\nGET 342\r\nSeq:4\r\nSesion:\r\nTime:\r\nEOM\r\n”

“200\r\nSeq:4\r\nTime:30\r\nEOM\r\n”

Los datos que se puede pedir en un comando GET son los siguientes:

•	Session: nombre de la sesión en que se encuentra el cliente.
•	Count: cantidad máxima de miembros de la sesión.
•	Time: tiempo en que expira la sesión sin actividad.
•	ID: ID de los clientes de la sesión.
•	Name: nombre de los clientes de la sesión.
•	Data: datos almacenados en la sesión.
•	Closed: saber si la sesión está cerrada.
•	Others: listado de los otros usuarios que pueden iniciar sesión.

Cuando se piden los ID y nombres de los miembros de la sesión se devuelven en el mismo orden: al primer ID corresponde el primer nombre y así sucesivamente.

SET.

Este comando lo utilizan los clientes para cambiar sus datos en la sesión. El cliente que puede cambiar el nombre de la sesión, los campos Others y Closed es el que creó la sesión. El campo Data de la sesión lo puede cambiar cualquier miembro de la sesión.  Los demás clientes solo pueden cambiar su nombre. Un cliente puede cambiar su nombre en la sesión de la siguiente forma:

“0\r\nSET 342\r\nSeq:5\r\nName:Pepito\r\nEOM\r\n”

Para cambiar el nombre de la sesión:

“0\r\nSET 342\r\nSeq:6\r\nSession:Damas 3\r\nEOM\r\n”

El servidor responde OK o el mensaje de error correspondiente:

“200\r\nSeq:5\r\nEOM\r\n”

“301\r\nSeq:6\r\nEOM\r\n”

Cuando el cliente que crea la sesión sale de ella, puede cambiar el nombre de la sesión el cliente que le sigue con más tiempo en la sesión.

Los datos que se pueden cambiar con el comando SET son:
•	Name: nombre del cliente en la sesión.
•	Session: nombre de la sesión en que participa el cliente.
•	Data: datos de la sesión.
•	Others: otros miembros de la sesión.
•	Closed: almacena el estado de la sesión: abierta o cerrada.

NOTE.

El servidor utiliza este comando para enviar notificaciones de cambio de estado de la sesión a sus clientes. El servidor no espera respuesta del cliente, el cliente recibe la notificación y hace lo que debe o la desecha. Las notificaciones las envía el servidor cuando un cliente nuevo inicia sesión, cuando un cliente cierra sesión, cuando se ha cerrado la sesión al cliente por inactividad o cuando se cierra toda la sesión por inactividad.

Por ejemplo, cuando un cliente nuevo “Juan” ha iniciado sesión, los demás clientes de la sesión reciben el comando:

“NOTE 101\r\nID:760\r\nName:Juan\r\nEOM\r\n”

Cuando un cliente recibe este mensaje debe actualizar la lista de clientes que tiene en la sesión.

Cuando un cliente (760) ha cerrado la sesión los demás clientes reciben la notificación:

“NOTE 102\r\nID:760\r\nName:Juan\r\nEOM\r\n”

Un cliente tiene que cerrar su sesión cuando recibe la notificación 103 o 104, las dos por inactividad.

“NOTE 103\r\nEOM\r\n”

DOWN.

Los clientes cierran su sesión a través del comando DOWN, de la siguiente manera:

“0\r\nDOWN 342\r\nSeq:6\r\nEOM\r\n”

Reciben OK o el error correspondiente:

“200\r\nSeq:6\r\nEOM\r\n”

“352\r\nSeq:6\r\nEOM\r\n”

HELLO.

Los clientes pueden utilizar el comando HELLO para actualizar su estado en la sesión. Este comando hace que el servidor actualice la fecha de última actividad del cliente en la sesión. Se utiliza de la siguiente forma:

“0\r\nHELLO 342\r\nSeq:123\r\nEOM\r\n”

El servidor responde OK o el error correspondiente:

“200\r\nSeq:123\r\nEOM\r\n”

“406\r\nSeq:123\r\nEOM\r\n”

Mensajes de error.

Los mensajes de error que el servidor envía son los siguientes:

300: solicitud incorrecta.
301: no autorizado a ejecutar este comando.
302: pago requerido.
303: prohibido.
304: no encontrado.
305: sesión llena, no hay espacio para otro miembro.
351: parámetro no entendido.
352: ID de cliente no encontrado.
353: ID de cliente incorrecto.
354: cantidad de miembros de la sesión incorrecta.
355: secuencia incorrecta.
356: profundidad incorrecta.
357: incorrecto ID de aplicación.
358: parámetro con varios valores en el comando.
359: cantidad de miembros de la sesión incorrecta.
360: el nick del cliente esta vacío.
361: el nombre de la sesión esta vacío.
362: tiempo de la sesión incorrecto.
363: el nick del cliente ya existe en la sesión.
364: el nombre de la sesión ya existe.
365: la información a enviar esta vacía.
366: ID de la sesión de un cliente no encontrado.
367: El nuevo usuario de la sesión no se encuentra entre los que pueden iniciar la sesión.
368: la contraseña del cliente está vacía.
369: el id del servidor no es correcto.
370: el lugar que ocupa el jugador en la sesión no es correcto.

