<h1 align="center">Welcome to kc-practica-bigdata-arquitecture 👋</h1>
<p>
  <img alt="Version" src="https://img.shields.io/badge/version-1.0-blue.svg?cacheSeconds=2592000" />
  <a href="https://twitter.com/javiermiguelg">
    <img alt="Twitter: javiermiguelg" src="https://img.shields.io/twitter/follow/javiermiguelg.svg?style=social" target="_blank" />
  </a>
</p>

> Práctica de Arquitectura de BigData - Keepcoding 2019

## Estrategia

Servicio de información de pisos y posibilidad de enfermar en ellos.

### Definición resumen

- Descarga de Airbnb 
- Scrapeo www.sareb.es, idealista, fotocasa y otros con scripts en python/node.js corriendo en raspberries. Saco direcciones y precios.
- Recupero datasets de enfermedades del corazón que contengan localización (https://catalog.data.gov/dataset?tags=heart-disease, https://www.physionet.org/data/?topic=heart+disease&orderby=relevance-desc&types=0)
- Diseño un algoritmo que da un indicador de infarto en función de la zona geográfica
- Aplico el algoritmo a todas las viviendas descargadas
- Monto HDFS/Yarn
- Cargo en Hive los datasets de pisos, enfermedades y localizaciones.
- Hago un API que le pasas un sitio, hace geocodificación con un API de terceros (de Google por ejemplo) y te da un indicador de posible infarto, precio de alquiler y precio de venta de vivienda de la zona, para que te decidas si te conviene vivir ahí :)
- Con rate-limiting controlar que las cuentas de prueba solo hacen unas pocas consultas y las de pago se limiten a la suscripción comprada.
- Los logs los enchufo a un logstash que guarda la información transaccional en ElasticSearch, con Kibana monto unos dashboards con KPI’s del estatus de las raspberries, proceso etl, peticiones al API, errores, compras de suscripciones y peticiones de soporte.
- En el dashboard incluyo indicadores de sentimiento (con [sentiment](https://github.com/thisandagain/sentiment)) con respecto a mi marca/servicio en twitter/fb y posts en mis foros de soporte.
- Monto un proceso de actualización que está continuamente scrapeando enviando con kafka hacia storm que me alimenta el hive.



### Servicios

El catalogo de servicios proporcionado por el datalake se basa en información de longevidad o posibilidad de enfermar en distintas partes del planeta en contraposición de los precios de alquiler o venta en la zona.

### Datasets

Los datasets iniciales a incorporar son:
* Dataset de [Airbnb](https://public.opendatasoft.com/explore/dataset/airbnb-listings/export)
  * Se le aplicará un proceso de limpieza de datos inservibles
* Datasets de enfermedades del corazón que contengan localización
  * Estos datsets serán descargados de fuentes públicas de información
  * Se comenzará con enfermedades del corazón con datasets como:
    * [Datasets de data.gov](https://catalog.data.gov/dataset?tags=heart-disease)
    * [Datasets de Physionet](https://www.physionet.org/data/?topic=heart+disease&orderby=relevance-desc&types=0)
  * Posteriormente se irán incluyendo más datasets de enfermedades del corazón como de otras enfermedades de las que tengamos localización
* Datasets de portales de venta de viviendas como Idealista, Fotocasa, etc
  * Estos datasets serán scrapeados



### Diagrama de Staging

Aquí se muestran los [procesos](https://docs.google.com/drawings/d/1Ivb9oEZ5ztXWgYvPKWyj8zl0z1WF0_otQK8-3oKW6Uk/edit?usp=sharing) de staging.

<img alt="Staging" src="http://javiermiguel.com/images/misc/Practica%20JMG%20-%20Staging.jpg" />



## Arquitectura

A continuación se muestra un [diagrama](https://docs.google.com/drawings/d/1Yd_pnRy8ug83_h3uz-jg_c4py9ycoTn8pw20beO7Kqs/edit?usp=sharing) de la arquitectura de la solución.

<img alt="Staging" src="http://javiermiguel.com/images/misc/Practica%20JMG%20-%20DataLake.jpg" />

El modelo se desplegará en la nube de Google Cloud.

### Componentes

Los componentes del sistema son:

* Hadoop HDFS
* YARN
* Hive
* Kafka
* Storm
* Logstash
* Elasticsearch
* Kibana
* API a consumidores



### Procesos

Los procesos definidos son:

* Descarga de ficheros.
  * Viviendas en alquiler - Airbnb
  * Enfermedades de corazón por ubicación - data.gov, physionet, etc
* Web scraping de viviendas en venta - Idealista, Fotocasa, Sareb, etc
* Data quality. Cada dataset de entrada se beneficiará de un proceso de calidad de datos:
  * Limpieza de registros que no cumplen el mínimo definido de información
  * Normalización de datos según la enciclopedia de tipos de datos
  * Deduplicación, se chequea la existencia de los nuevos datos que se pretenden incorporar para evitar tenerlos duplicados
  * Asignación de ubicación a nivel de coordenadas lat/lon. Para la información relativa a areas extensas, se usará el centroide del área
  * Proceso que calcula probabilidad de infarto en cada ubicación
* Carga en cluster por streaming
  * Los datos se emiten usando Kafka
  * Kafka se conecta a Storm para pasar los datos a HDFS
* Carga en Hive



## Modelo operativo

Primera Fase:

- La descarga de ficheros de airbnb, durante la 1ª fase se hace a mano. En segunda fase de hará de forma automatizada con scripts en python/node.js.
- El proceso de web scraping de viviendas se programa para que esté de forma contínua crwleando varios portales y scrapeando datos.
- Los procesos de data quality son invocados de forma automática ante cada fichero descargado.



Segunda Fase:

* Se van añadiendo otros datasets tanto de infartos como de otras enfermedades.

* Se automatiza la descarga de los ficheros de forma periódica. Un script se encargará de verificar cada día si existen nuevas versiones de los datasets y en caso afirmativo lanzará el script de descarga.




## Author

👤 **Javier Miguel**

* Twitter: [@javiermiguelg](https://twitter.com/javiermiguelg)
* Github: [@jamg44](https://github.com/jamg44)

## Show your support

Give a ⭐️ if this project helped you!
