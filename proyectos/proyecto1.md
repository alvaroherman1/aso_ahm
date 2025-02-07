# Proyecto de la 1ª Evaluación
## Análisis de redes
## Descripción del proyecto
Este proyecto consistirá en crear un script en Bash que permita analizar una red local para detectar equipos conectados, identificar puertos abiertos y tratar de obtener el sistema operativo de cada equipo. El script guardará la información recopilada en un archivo de texto.

## Funciones del Script 
A partir del puerto vamos a Obtener el servicio:
- La funcion get_service utilizara un archivo CSV (tcp.csv) que contiene la información de los puertos y los servicios asociados a cada puerto. 
- Para llevar esto a cabo utilizaremos awk patra extraer el nombre del servicio y filtrar el archivo.
``` bash 
function get_service(){

  
} 
```