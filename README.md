## Estrategia del DAaas
Comparador de vuelos y recomendador de estancias Airbnb para dicha fecha seleccionada.

__Definición de las dudas anteriores:__

__1.- Formato de los datos de entrada:__

	- El usuario introducirá salida, destino, mes y duración del viaje.

__2.- Interacción del usuario en el proceso:__

	- El usuario una vez introducidos los datos de entrada, la web le devolvera los vuelos disponibles para ese rango de fechas introducido. Luego, lanzara el segundo job para obtener la recomendacion de airbnb.

__3.- Disponibilidad de apartamentos en Airbnb:__

	- Para asegurar que tenemos siempre el dataset actualizado en HDFS, realizaremos crawler del listing de airbnb con un cron, de esta manera evitamos tener apartamentos en el listado que ya no estan en la plataforma.


## Arquitectura

__Arquitectura en Cloud basada en scrapy, HDFS, Google Cloud Storage, Hive, MongoDB y Dataproc.__

Utilizaremos una instancia fuera de Hadoop donde alojaremos un servidor wen, donde ira alojada la web con la los usuarios interactuaran. En ella deberán introducir ciudad de origen, destino, mes en el que desean viajar y la duración del mismo.

Crawler con scrapy mediante un cron que descarga los datos actualizados del listing de airbnb, para asegurar que el dataset que vayamos a usar tenga siempre los apartamentos disponibles actualizados, lo guardaremos en un csv. Guardamos dataset en HDFS.

Crawler con scrapy mediante un cron que descarga los datos actualizados de Google Flights, obteniendo los datos de todos los vuelos y sus precios, lo guardaremos en un csv. Guardamos dataset en HDFS.

Insertamos dataset de flights en HIVE para realizar una query y mostrar al usuario los vuelos disponibles en ese rango de fechas, para que seleccione el vuelo deseado. El resultado de esa query se guardara en Google segment y se mostrara en la web para que el usuario seleccione el vuelo deseado.

Una vez conocemos la fecha exacta del usuario, desde HDFS cogeré los datos de Airbnb y realizaremos un query para que seleccione las mejores opciones para la fecha seleccionada. Realizando primero una limpieza por ciudad de destino, availability…

De la query obtendremos el listado de los 20 mejores apartamentos de Airbnb, según los criterios establecidos.

Ambos resultados de las query se guardaran en Google Segment, pero para evitar estar repitiendo querys complejas utilizaremos una capa de cache, para guardar durante algun tiempo (expireAfterSeconds en MongoDB) los resultados de querys complejas por si tenemos la opción de no repetirlas. Lo hacemos con una nueva instancia con MongoDB encargada de alojar los resultados de las querys pesadas.

Insertamos el resultado de la query en Google Storage y lo mostramos en la web.


## Operating model

El cluster estará siempre activo, ya que será una aplicación que recibas muchos request por parte de los usuarios.


__1.- Asegurarnos que los datos en HDFS son actualizados.__

	Para evitar que haya apartamentos en HDFS que ya no estén disponibles para alquilar en Airbnb, cada dia y mediante un cron descargaremos el fichero nuevo y lo guardaremos en HDFS, mediante un crawler con scrapy.
	Mediante otro crawler, descargaremos los datos de todos los vuelos disponibles con sus respectivos precios. Esto supone un pequeño sacrificio, ya que si la búsqueda se realiza x horas después de haber descargado los datos, puede ser que los tiempos hayan cambiado. Es decir, estamos sacrificando algo de calidad del dato, respecto al timelines. 
	Lo primordial de esta herramienta es que el usuario obtenga el dato lo más rápido posible, por lo que las primeras querys a lo mejor no tendrán el valor real del precio, pero en un paso final se actualizarán dichos valores.
	Las columnas que nos interesarían descargar de dicho crawler:

	- Origen
	- Destino
	- Fecha ida
	- Fecha vuelta
	- Duración del viaje
	- Compañía
	- Precio ida y vuelta
	- Directo/escalas
	
	Es importante una vez el usuario eliga el vuelo, realizar una actualización de dichos dataset para asegurarnos de que el usuario reciba el precio correcto.

__2.- ¿Quién y cómo lanzarás los jobs?__

	Este servicio funcionara por peticiones a través de una pagina web, cada vez que un usuario quiera realizar una búsqueda introducirá los parámetros anteriormente descritos, ciudad de origen, ciudad de destino, mes del viaje y la duración del mismo. Dando al botón buscar, se activará un Python request con Google Function y se lanzara el job.
	Tendremos dos tipos de Jobs. Cuando el usuario entre en la web e introduzca los parámetros de su vuelo, activara un Python request en el cual se le devolverá un listado con los vuelos disponibles para dicho rango de fecha seleccionado. Una vez seleccione la fecha concreta, lanzara un nuevo job para la recomendación de los Airbnb.
	
		__1.- Primer job, mostrar vuelos para el rango de fecha seleccionado.__

			Para tener un mayor acceso a los datos, por si el usuario desea cambiar de vuelo, guardaremos el resultado de esa query en Google segments: 
		
				- Crear tabla flights
				- load data inpath de hdfs://XXXX:dataset_Google flights/ into table Flights 
				- Query de HIVE para obtener solamente los viajeF para el destino y el rango seleccionado.
				- Este fichero se guardará en un segmento con el nombre Flights. Select into directotry 'gs://output/Flights.
				- A parte de mostrar el resultado en la web, se guardara en mongoDB (explicación abajo)
	
			En este momento se deberían volver a actualizar los datos del vuelo, para mostrar el precio real al cliente. No se realmente como hacerlo.
	
			Mostrar dichos resultados en la web. 

		__2.- Segundo job, query de recomendación Airbnb.__
	
			Una vez mostrados los resultados de viajes en la web, el usuario deberá elegir una fecha final sobre la que quiere que se le recomiende una lista de apartamentos.
		
			- Crear tabla airbnb
			- load data inpath de hdfs://XXXX:dataset_airbnb/ into table Airbnb.
			- Query de HIVE para la recomendación de los apartamentos, para ello debemos calcular el precio total de la estancia, precio de una noche*días duración del viaje. La Query seleccionara una recomendación de los TOP20 apartamentos para la fecha y destino seleccionados, teniendo en cuenta los siguientes parámetros:
				
				Aquellos apartamentos con una disponibilidad mayor o igual que la de número de días que se quiera viajar.
				Aquellos apartamentos que cumplan los requisitos de noches mínimas.
				Aquellos apartamentos where location review > 6.
				Aquellos apartamentos where score rating > 6.
				Aquellos donde se alquile el apartamento completo.
	
			Este fichero se guardará en un segmento con el nombre update_airbnb. Select into directotry 'gs://output/update_airbnb. 


	Como vamos a realizar querys bastante pesada y para evitar que el usuario espere por los resultados, utilizaremos una capa de cache en la que el servidor web ira primero para ver si los resultados de la petición estan en sus tablas y ahorrarnos asi el realizar de nuevo todas las querys tan pesados. Las tablas estaran en MongoDB solamente un dia, luego se borraran.

##Diagrama

https://docs.google.com/drawings/d/1RCBEE4bhRuqp7Sc7ZK-8CXGDumQhtWfho1qgBD5Ep5I/edit