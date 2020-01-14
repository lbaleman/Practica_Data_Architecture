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

Al ser una herramienta que quiero usar yo a modo personal lo haría en Docker, pero entiendo que es mucho mas interesante hacerla con cloud. Por lo tanto, sería una arquitectura en Cloud basada en scrapy, Google Cloud Storage, Hive y Hadoop (Dataproc).
Crawler con scrapy de skyscanner para obtener los vuelos de una fecha (ya sea concreta o no) establecida por el cliente. Sería interesante ejecutarla desde un cloud function. Mi duda es si el cliente tiene que meter los datos de ciudad y fecha, para hacerlo de manera elegante se podría hacer a través de una API con Python request???.

Insertamos el dataset de Airbnb en Hive (esto solo lo hacemos una vez).

Guardamos ambos dataset en un segmento de Google Cloud.

Realizar una query que selecciones los vuelos más baratos con los alojamientos disponibles más baratos. Duda: Para asegurarnos de que los alojamientos estén disponibles, una vez hecha la transformación hay que crawlear las ubicaciones seleccionadas para ver su disponibilidad???

Resultados en fichero en Google storage ¿y enviados por email?
