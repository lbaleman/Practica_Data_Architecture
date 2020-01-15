#Practica Data architecture

## Idea general
Herramienta para usuarios que deseen encontrar vuelos y estancias en Airbnb económicas. Scraping de skyscanner para encontrar los vuelos más económicos en las fechas introducidas por el usuario y dar una lista de los apartamentos en Airbnb mas barato para la zona. (¿Ubicación importante?)

## Nombre del prodcuto
Buscador de vuelo+estancia económicas

## Estrategia del DAaas
Establecer una lista con los 25 apartamentos de airbnb más económicos en un rango de ubicación establecido por el usuario, para la fecha donde los vuelos sean más baratos.

__Definición de las dudas anteriores:__

__1.- Formato de los datos de entrada:__
	- El usuario introducirá salida, destino, mes y duración del viaje.
__2.- Interacción del usuario en el proceso:___
	Esta duda se deberá resolver en función de la estrategia del DAaaS
	- Con interacción
		Por ejemplo, si el usuario desea viajar 1 semana en Marzo. Podrá ver el listado de todos los precios ida y vuelta para una semana en Marzo y ahí, seleccionar el que desea. De esta manera ya sabemos cual es la fecha exacta.Solamente, insertaremos en HIVE listado de vuelos y los apartamentos para esas fechas concretas.
	- Sin interacción:
		Por ejemplo, si el usuario desea viajar 1 semana en Marzo. El usuario tendrá al final los datos de precios de ida y vuelta más alojamiento para todos los días del mes. Es decir, tendrás un precio de billete de ida y vuelta si deseas salir el día 1(volver el día 1 + días de vacaciones introducidos) y lo que cuesta una noche en el apartamento de airbnbn*los días introducidos (Así para todos los días del mes). __Para esta opción tendríamos muchos mas datos, pero también sería mas interesante.__

__4.- Disponibilidad de apartamentos en Airbnb.__
	- Esa duda la expongo en el apartado de arquitectura, no se si es posible realizar la limpieza de los apartamentos ocupados en esas fechas con una query o hay que hacerlo con un crawler.


## Arquitectura

Arquitectura en Cloud basada en scrapy, Google Cloud Storage, Hive y Hadoop (Dataproc).

__1.- ¿Cómo el usuario introducirá los datos?.__
	Pequeño servidor con una API donde el usuario pueda introducir los parámetros de entrada. 
	__No se me ocurre otra manera de hacerlo.__

__2.- Obtener dataset de Google flight.__
	Crawler con scrapy de Google flights con los siguientes parámetros de entrada: origen, destino, mes y duración del viaje. ¿Con que se inicia?	
	Se podría descargar un fichero.csv con los siguientes parámetros: ciudad origen, ciudad destino, fecha inicio, fecha fin(fecha inicio+días seleccionados), precio ida+vuelta, compañía, hora de salida vuelo de ida, hora de salida vuelo de vuelta, enlace al vuelo. 

__3.- Almacenamiento y limpieza dataset de Airbnb.__
	Insertamos el dataset de Airbnb completo en Google Storage. 
	Realizamos la limpieza del dataset.
	__Dudas con la limpieza:__
	La limpieza de datos esta clara, extraer solo aquellos datos disponibles para la fecha seleccionada, que se encuentren en la ciudad de destino y que cumplan las condiciones de noches mínimas.
	¿Cómo se podría ver la disponibilidad de los airbnb?: Se que esto con un crawler se puede hacer, pero creo que con una query de HIVE no se puede.

__4.- Almacenar dataset en Google Storage.__
	Almacenar dataset actualizado en Google Storage. 
	Dudas sobre que hacer una vez terminada la consulta:
	Deberíamos borrar ese dataset, porque no servirá para futuras consultas porque a lo mejor esos apartamentos ya han sido ocupados. Pero, ¿Cuánto tiempo después de realizar la consulta?.

__5.- Inserción de datasets en HIVE.__
	Con los dos dataset (precios de vuelos) y Airbnb, creación de dos tablas en HIVE y se realizara una query que seleccione aquellos Airbnb.
	La query deberá mostrar una lista de los 20 apartamentos más baratos para le fecha seleccionada. También es importante que tengan unos Reviw scores location y Review rates buenos, porque si solamente seleccionamos los baratos nos dará apartamentos muy lejos del centro, tampoco queremos eso.

__6.- Resultados en fichero en Google storage.__
	Almacenar fichero en Google Storage.

__7.- Mostrar resultados.__
	En el caso de elegir una API como interacción con el usuario, mostrar los resultados.

