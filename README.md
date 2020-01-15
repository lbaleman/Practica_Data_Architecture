#Practica Data architecture

## Idea general
Herramienta para usuarios que deseen encontrar vuelos y estancias en Airbnb económicas. Scraping de skyscanner para encontrar los vuelos más económicos en las fechas introducidas por el usuario y dar una lista de los apartamentos en Airbnb mas barato para la zona. (¿Ubicación importante?)

## Nombre del prodcuto
Buscador de vuelo+estancia económicas

## Estrategia del DAaas
Establecer una lista con los 25 apartamentos de airbnb más económicos en un rango de ubicación establecido por el usuario, para la fecha donde los vuelos sean más baratos.


__Primera duda:__

1.- Formato de la fecha:
	- El usuario debe introducir la fecha concreta, tipo desde 14/01/2020 hasta 17/01/2020.
	- Podría hacerse de una forma más general en la que el usuario solamente introduzca el mes del viaje. Aunque esto creo que es raro, ya que una vez hecho obtenido el dataset con los precios de todos los días, el usuario debería volver a interactuar para decidir la fecha final y que ahí se haga la query con el dataset de Airbnb.

2.- Scraping de skycanner:
	- Skyscanner una vez realizas la búsqueda, en la cabecera de la misma aparecen 3 opciones de vuelo: la mejor, la más barata y la más rápida.
	- ¿En el scraping descargamos solo la opción más barata o sería conveniente descargar más?.

3.- Interacción del usuario en el proceso(ligada a la duda 1):
	- El usuario (en este caso yo) solamente tendrá que introducir la ciudad de origen, ciudad de destino y fecha. Una vez introducidas, no tendrá ninguna interacción más con el programa.
	- En el caso de que pueda introducir una fecha no concreta, una solución posible sería, en los resultados dejarle varias opciones de vuelos y que cada opción de vuelo tenga sus recomendaciones de apartamentos en Airbnb.
	- Mi duda es que se que sería mucho más completo que el usuario pudiera introducir una fecha menos concreta. Ya que si el usuario introduce una fecha exacta el dataset de skyscanner sería muy pobre, a no ser que pudiera comparar varias ciudades para un mismo rango de fechas.
4.- Disponibilidad de apartamentos en Airbnb
	- Una vez tengamos el dataset de Airbnb y el de skyscanner, se hará una query en HIVE que determine los apartamentos más baratos en la ciudad. ¿Para determinar si esos apartamentos están disponibles habría que volver a crawlear esas direcciones de los apartamentos de airbnb?

## Arquitectura

Arquitectura en Cloud basada en scrapy, Google Cloud Storage, Hive y Hadoop (Dataproc).

#1.- ¿Cómo el usuario introducirá los datos?.
	Pequeño servidor con una API donde el usuario pueda introducir los parámetros de entrada. 
	__No se me ocurre otra manera de hacerlo.__

#2.- Obtener dataset de Google flight.
	Crawler con scrapy de Google flights con los siguientes parámetros de entrada: origen, destino, mes y duración del viaje. ¿Con que se inicia?	
	Se podría descargar un fichero.csv con los siguientes parámetros: ciudad origen, ciudad destino, fecha inicio, fecha fin(fecha inicio+días seleccionados), precio ida+vuelta, compañía, hora de salida vuelo de ida, hora de salida vuelo de vuelta, enlace al vuelo. 

#3.- Almacenamiento y limpieza dataset de Airbnb.
	Insertamos el dataset de Airbnb completo en Google Storage. 
	Realizamos la limpieza del dataset.
	__Dudas con la limpieza:__
	La limpieza de datos esta clara, extraer solo aquellos datos disponibles para la fecha seleccionada, que se encuentren en la ciudad de destino y que cumplan las condiciones de noches mínimas.
	¿Cómo se podría ver la disponibilidad de los airbnb?: Se que esto con un crawler se puede hacer, pero creo que con una query de HIVE no se puede.

#4.- Almacenar dataset en Google Storage.
	Almacenar dataset actualizado en Google Storage. 
	Dudas sobre que hacer una vez terminada la consulta:
	Deberíamos borrar ese dataset, porque no servirá para futuras consultas porque a lo mejor esos apartamentos ya han sido ocupados. Pero, ¿Cuánto tiempo después de realizar la consulta?.

#5.- Inserción de datasets en HIVE.
	Con los dos dataset (precios de vuelos) y Airbnb, creación de dos tablas en HIVE y se realizara una query que seleccione aquellos Airbnb.
	La query deberá mostrar una lista de los 20 apartamentos más baratos para le fecha seleccionada. También es importante que tengan unos Reviw scores location y Review rates buenos, porque si solamente seleccionamos los baratos nos dará apartamentos muy lejos del centro, tampoco queremos eso.

#6.- Resultados en fichero en Google storage .
	Almacenar fichero en Google Storage.

#7.- Mostrar resultados.
	En el caso de elegir una API como interacción con el usuario, mostrar los resultados.

