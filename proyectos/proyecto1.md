# Proyecto de la 1ª Evaluación
## Análisis de redes
## Descripción del proyecto
Este proyecto consistirá en crear un script en Bash que permita analizar una red local para detectar equipos conectados, identificar puertos abiertos y tratar de obtener el sistema operativo de cada equipo. El script guardará la información recopilada en un archivo de texto.

<<<<<<< HEAD
## Funciones del Script 
A partir del puerto vamos a Obtener el servicio:
- La funcion get_service utilizara un archivo CSV (tcp.csv) que contiene la información de los puertos y los servicios asociados a cada puerto. 
- Para llevar esto a cabo utilizaremos awk patra extraer el nombre del servicio y filtrar el archivo.
``` bash 
function get_service(){

  
} 
=======
## FUNCIONES DEL SCRIPT:
A partir del puerto vamos a Obtener el servicio:
- La funcion "get_service" utilizara un archivo CSV (tcp.csv) que contiene la información de los puertos y los servicios asociados a cada puerto. 
- Para llevar esto a cabo utilizaremos "awk" patra extraer el nombre del servicio y filtrar el archivo.
``` bash 
function get_service(){
    local port=$1
    awk -F, -v port ="$port" 'NR > 1 && $2 == port {gsub(/"/,"",$3); print $3}' ./puertos/tcp.csv 
} 
```
### OBTENCION DE LA DIRECCION MAC:

- "get_mac" nos servira para obtener al direccion MAC asociada  auna dirección IP específica utilizando "arp".

- Para ello ejecutaremos el comando "arp -n" que nos servira para listar las direcciones IP junto con sus direcciones MAC. Utilizando awk filtraremos la salida y nos mostrara la direccion MAC de la IP que hemos solicitado.
```bash
function get_mac(){
    local ip=$1
    arp -n "$ip" | awk 'NR > 1 {print $3}'
}
```
### GENERAR SALIDA EN FORMATO CSV:
- "output_csv" nos ayudara a generar un archivo de salida en formato CSV, que contiene direcciones IP, direcciones MAC, puertos abiertos y el sistema operativo de cada equipo.
- Crearemos un archivo CSV donde pondremos las irecciones IP, direcciones MAC, puertos abiertos y el sistema operativo de cada equipo.
```bash 
function output_csv(){
    local output_file=$1
    echo"DIRECCIONIP,DIRECCIONMAC,SISTEMAOPERATIVO,PUERTOSABIERTOS" > "$output_file"
    for entry in "${results[@]}"; do 
        echo "$entry" >> "$output_file"
    done
}
```
### DIRECCION IP EN FORMATO CIDR
- El usuario tendra que poner unan direccion IP  en formato CIDR (192.168.1.0/24).
- Utilizaremos el comando "read" para que el usuario introduzca la direccion IP y nos ayudaremos de "IFS" para que la direccion de red y la mascara se separen.
```bash
read -p "INTRODUCE LA DIRECCION IP EN FORMATO CIDR(192.168.1.0/24):" cidr
IFS="/" read -r network mask <<< "$cidr"
```
### CALCULO DEL RANGO DE IPS SUSTENTADO POR LA MASCARA:
- Utilizaremos el comando "case" el cual nos ayuadara a determinar el rango basado en la mascara.
```bash
case $mask in 
    1) range=256 ;;
    2) range=65536 ;;
    3) range=166777216 ;;
    *) echo "LA MASCARA DEBE SER /8, /16 O /24." ; exit 1 ;;
esac 
```
### INICIALIZAR EL ARCHIVO DE SALIDA:
- Inicializaremos el archivo de salida donde se almacenara la informacion recopilada durante el analisis de red. Nos ayudaremos el comando "echo" para escribir el encabezado del analisis de red para la direccion CIDR que nos inndique el usuario.
```bash
echo "ANALISIS DE RED PARA $cidr" > "$ouput_file"
```
### MARCA DE TIEMPO DE INICIO 
- Registraremos la marca de tiempo de inicio del analisis para poder medir la duracion del proceso posteriormente.Nos ayudaremos del comando "date" con "%s" para que nos de el tiempo actual en segundos. Esto se almacena en "start_time".
```bash
start_time=$(date +%s)
```
### ESCANEADO DE LA RED 
- Escanearemos las direcciones IP en el rango que se especifica, ademas enviaremos pings para identificar cuales son los equipos activos.Nos ayudaremos de un bucle "for" y un ping para comprobar si estan disponibles las IPS.
```bash 
echo "ESCANENANDO LA RED..."
for i in $(seq 1 $((range-2))); do 
    ip="${network%.*}.$i"
    if ping -c 1 -W 1 "$ip" &> /dev/null; then 
        echo "HAS ENCONTRADO UN EQUIPO: $ip"
        echo "HAS ENCONTRADO UN EQUIPO: $ip" >> "$output_file"
    fi
done
```
### OBTENER LA DIRECCION MAC
- Obtendremos la direccion MAC que esta asociada a una IP en concreto.Nos ayudaremos d la funcion "get_mac" que usara la direccion IP y almacenara el resultado en la variable "mac". Luego con "echo" mostraremos la informacion.
```bash
mac=$(get_mac) "$ip")
echo "LA MAC DEL EQUIPO ES: $mac"
echo "LA MAC DEL EQUIPO ES: $mac" >> "$output_file"
```
### BASANDONOS EN TTL DETECCION DEL SISTEMA OPERATIVO
- Determinaremos el sistema operativo de cada equipo activo con ayuda del valor TTL de la respuesta del ping. Extraeremos el TTL.
```bash
ttl=$(ping -c 1 "$ip" | grep "ttl=" | awk -F'ttl=' '{print $2}' | awk '{print $1}')
if [[ $ttl -eq 63 ]]; then
    os="LINUX"
elif [[ $ttl -eq 127 ]]; then
    os="WINDOWS"
else
    os="NO CONOCIDO"
fi
echo "EL SISTEMA OPERATIVO ES: $os"
echo "EL SISTEMA OPERATIVO ES: $os" >> "$output_file"
```
### ESCANEAR LOS PUERTOS ABIERTOS
- Escanearemos los puertos(1-1024) para saber cuales estan abiertos y que servicios estan corriendo en cada equipo activo. Nos ayudaremos del comando "nc" para escanear los puertos y usaremos la funcion "get_service"  para obtener el servicio asociado a cada puerto.
```bash
echo "ESCANEANDO PUERTOS ABIERTOS EN $ip"
for port in {1..1024}; do
    if nc -z -w 1 "$ip" "$port" 2>/dev/null; then
        service=$(get_service "$port")
        service=${service:-"Desconocido"}
        echo "PUERTO ABIERTO: $port (SERVICIO: $service)"
        echo "PUERTO ABIERTO: $port (SERVICIO: $service)" >> "$output_file"
    fi
done
```
### REGISTRO DEL TIEMPO DE FINALIZACION 
- Registaremos el tiempo de finalizacion del analisis para calcular el tiempo total que marca el proceso. Nos ayudaremos del comando "date" para obtener la fecha y hora actual y la guardaremos en la variable "end_time". 
```bash
end_time=$(date +%s)
elapsed_time=$((end_time - start_time))
```
### GUARDAR EL RESULTADO EN UN ARCHIVO
- Guardaremos el resultado del analisis en un archivo que incluira la direccion IP, el sistema operativo y los puertos abiertos
```bash
echo "ANALISIS COMPLETADO, LA INFORMACION SE GUARDA EN $output_file."
echo "EL TIEMPO TOTAL DE EJECUCION ES DE $elapsed_time SEGUNDOS."
echo "EL TIEMPO TOTAL DE EJECUCION ES DE $elapsed_time SEGUNDOS." >> "$output_file"
```
### SCRIPT COMPLETO 
```bash
#!/bin/bash

# OBTENCION DEL SERVICIO A TRAVES DEL PUERTO
function get_service() {
    local port=$1
    awk -F, -v port="$port" 'NR > 1 && $2 == port {gsub(/"/, "", $3); print $3}' ./puertos/tcp.csv
}

# OBTENCION DE LA DIRECCION MAC:
function get_mac() {
    local ip=$1
    arp -n "$ip" | awk 'NR >1 {print $3}'
}

# GENERAR SALIDA EN FORMATO CSV:
function output_csv() {
    local output_file=$1
    echo"DIRECCIONIP,DIRECCIONMAC,SISTEMAOPERATIVO,PUERTOSABIERTOS" > "$output_file"
    for entry in "${results[@]}"; do
        echo "$entry" >> "$output_file"
    done
}

# DIRECCION IP EN FORMATO CIDR
read -p "INTRODUCE LA DIRECCION IP EN FORMATO CIDR(192.168.1.0/24):" cidr
IFS='/' read -r network mask <<< "$cidr"

# CALCULO DEL RANGO DE IPS SUSTENTADO POR LA MASCARA
case $mask in 
    1) range=256 ;;
    2) range=65536 ;;
    3) range=166777216 ;;
    *) echo "LA MASCARA DEBE SER /8, /16 O /24." ; exit 1 ;;
esac 

# ARCHIVO DE SALIDA
read -p "INTRODUCE EL NOMBRE DEL ARCHIVO PATRA GUARDAR LA INFORMACION: " output_file

# INICIALIZAR EL ARCHIVO DE SALIDA:
echo "ANALISIS DE RED PARA $cidr" > "$ouput_file"
echo "===================================" >> "$output_file"

# MARCA DE TIEMPO DE INICIO 
start_time=$(date +%s)

# ESCANEADO DE LA RED 
echo "ESCANENANDO LA RED..."
for i in $(seq 1 $((range-2))); do 
    ip="${network%.*}.$i"
    if ping -c 1 -W 1 "$ip" &> /dev/null; then 
        echo "HAS ENCONTRADO UN EQUIPO: $ip"
        echo "HAS ENCONTRADO UN EQUIPO: $ip" >> "$output_file"
# OBTENER LA DIRECCION MAC
mac=$(get_mac "$ip")
echo "LA MAC DEL EQUIPO ES: $mac"
echo "LA MAC DEL EQUIPO ES: $mac" >> "$output_file"

# BASANDONOS EN TTL DETECCION DEL SISTEMA OPERATIVO
ttl=$(ping -c 1 "$ip" | grep "ttl=" | awk -F'ttl=' '{print $2}' | awk '{print $1}')
if [[ $ttl -eq 63 ]]; then
    os="LINUX"
elif [[ $ttl -eq 127 ]]; then
    os="WINDOWS"
else
    os="NO CONOCIDO"
fi
echo "EL SISTEMA OPERATIVO ES: $os"
echo "EL SISTEMA OPERATIVO ES: $os" >> "$output_file"

        # ESCANEAR LOS PUERTOS ABIERTOS
        echo "ESCANEANDO PUERTOS ABIERTOS EN $ip"
        for port in {1..1024}; do
            if nc -z -w 1 "$ip" "$port" 2>/dev/null; then
                service=$(get_service "$port")
                service=${service:-"Desconocido"}
                echo "PUERTO ABIERTO: $port (SERVICIO: $service)"
                echo "PUERTO ABIERTO: $port (SERVICIO: $service)" >> "$output_file"
            fi
        done
        echo "-----------------------------------" >> "$output_file"
    fi
done

# REGISTRO DEL TIEMPO DE FINALIZACION 
end_time=$(date +%s)
elapsed_time=$((end_time - start_time))

# GUARDAR EL RESULTADO EN UN ARCHIVO
echo "ANALISIS COMPLETADO, LA INFORMACION SE GUARDA EN $output_file."
echo "EL TIEMPO TOTAL DE EJECUCION ES DE $elapsed_time SEGUNDOS."
echo "EL TIEMPO TOTAL DE EJECUCION ES DE $elapsed_time SEGUNDOS." >> "$output_file"
>>>>>>> eb57c8bab9fe43abb774bd65a3f8818c7b9c723f
```