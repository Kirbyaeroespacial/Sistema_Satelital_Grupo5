
VERSIÓN 1
🛰️ Satélite (Arduino) 
El satelite responde a comandos remotos (iniciar, pausar y reanudar transmisión) enviados desde la PC, y transmite periódicamente los datos de temperatura y humedad al satélite. Cuando la transmisión está pausada, envía un heartbeat (g) para indicar que el sistema sigue operativo, y reporta errores en caso de fallos del sensor.

🌍 Ground Station (Arduino) 
Su función principal es enviar comandos provenientes del software en Python hacia el satélite y retornar la telemetría recibida de vuelta al Python. Además, implementa un sistema básico de detección de fallos, utilizando un timeout de comunicación y un LED de error para indicar pérdida de enlace o fallos en la transmisión.


💻 Ground Station (Python + GUI) 
El software en Python proporciona una interfaz gráfica de control y monitoreo en tiempo real. Proyecta gráficas dinámicas de temperatura y humedad y la posibilidad al usuario mediante botones de parar, reanudar e iniciar la transmisión de datos.












VERSIÓN 2
🛰️ Satélite (Arduino)  
Respecto a la versión 2 el satélite  implementa un protocolo de aplicación permitiendo el envío simultáneo de múltiples tipos de datos: telemetría ambiental, distancia, ángulo del servo y estados de error. Además se añade el cálculo de una temperatura media sobre los últimos 10 valores de temperatura recibidos, junto con un sistema de alerta crítica que detecta sobretemperatura sostenida. Finalmente también hemos integrado  un servo motorizado y un sensor ultrasónico de distancia que envía un ángulo (hasta 180º) y distancia. 

🌍 Ground Station (Arduino) 
La estación de tierra, dado al protocolo del satélite realiza una traducción de protocolo, encargándose de clasificar, reenviar y etiquetar los datos del satélite mediante identificadores como letras. Esto permite separar claramente sensores, estados y errores, facilitando la escalabilidad del sistema y su interpretación en el código. Por otro lado, hemos introducido un control local mediante potenciómetro para el ángulo del servo, enviando comandos periódicos al satélite.
💻 Ground Station (Python + GUI) 
En el Python hemos incorporado una visualización tipo radar en coordenadas polares,  para poder mostrar en tiempo real las distancia y ángulo que encuentra el sensor ultrasónico, además de la incorporación de la media de las temperaturas en el gráfico de temperatura y humedad. De botones se han añadido dos para determinar si el servo se mueve de manera automática o de manera manual mediante un potenciómetro en la estación de tierra y un apartado para modificar la frecuencia con la que se envía la telemetría.








VERSIÓN 3
🛰️ Satélite (Arduino)  
En esta versión 3 hemos incorporado un sistema con checksum para el envío de mensajes de manera que antes de enviar el mensaje lo pasa por una función checksum que transforma el valor del paquete y lo envía, además si el checksum enviado con corresponde con el que debería el mensaje se descarta ya que es considerado un mensaje corrupto. Por otro lado también hemos implementado una función que calcula a tiempo real una hipotética órbita satelital con unas funciones y valores ya asumidos. Otro gran avance es que la comunicacion ahora es mediante LoRa, a distancia, y no mediante cables, lo que nos ha llevado a tener que crear un sistema mediante de funciones que según quien tenga el token puede enviar o no información para no saturar el LoRa.

🌍 Ground Station (Arduino)
El ground station, al igual que el satélite también tiene una función checksum, de tal manera que calcula el checksum del mensaje que recibe y mira si coincide con el valor que debería tener para descartar mensajes corruptos.
💻 Ground Station (Python + GUI) 
En el Python esta versión también se encuentra implementado la función de checksum y la la órbita también, ya que también se ha agregado un gráfico visual en 2D para representar dicha posición simulada. Por último también hemos añadido una pequeña ventana donde el usuario puede añadir cualquier nota/observación y esta es guardada en un archivo con otros eventos, estos se pueden filtrar por dia o tipo de evento (todos/comando/alarma/observación)












VERSIÓN 4
🛰️ Satélite (Arduino)  
En el satélite para la versión 4 hay grandes innovaciones. Lo primero y mas destacable ha sido incorporar una placa solar que se despliega y repliega mediante un complejo sistema de engranajes según la cantidad de luz que recibe el sensor de luz. Este proceso es no-bloqueante y si hay mucha luz recibida se despliega más que si hubiese poca. Además las funciones se han ordenado y el código es más legible y entendible.


🌍 Ground Station (Arduino)
La estación de tierra también tiene mejoras. El LED rojo de error funciona correctamente y se ha estructurado de nuevo la telemtria que recibe y que envia, pasando del formato ASCII a binario reduciendo el tamaño de cada paquete de 120 bytes a solo 29 bytes, un 75% menos, la temperatura que antes ocupaba 16 bytes en texto, ahora son 2 bytes.

💻 Ground Station (Python + GUI) 

En el Python hay grandes mejoras visibles, lo primero de todo es que la interfaz es mucho más amigable e intuitiva con iconografías y demás mejoras. Por otro lado la proyección 2D sobre la órbita satelital ahora es en 3D y mucho más visual, además esta órbita ha sido modificada y ahora es una órbita kepleriana. Por otro lado ahora el modo manual del servomotor del radar no se controla mediante un potenciómetro si no que desde la central del Python puedes definir a qué ángulo exacto quieres que se dirija.
