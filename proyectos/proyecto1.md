Proyecto de la 1ª Evaluación
Análisis de redes
Descripción del proyecto
Este proyecto consistirá en crear un script en Bash que permita analizar una red local para detectar equipos conectados, identificar puertos abiertos y tratar de obtener el sistema operativo de cada equipo. El script guardará la información recopilada en un archivo de texto.

Funcionalidades del proyecto
El script comenzará solicitando un dirección IP de red en formato CIDR (p.e. 192.168.1.0/24) y, a continuación, realizará los siguientes pasos:

Escaneo de red: enviará una señal de ping a cada una de las direcciones IP de la red para saber si hay algún equipo con esa IP o no. Por simplicidad, asumiremos que no habrá subredes, y, por tanto, las máscaras solo podrán ser /8, /16 y /24.
Detección de puertos abiertos: una vez identificado cada equipo, realiza un escaneo de puertos para identificar aquellos que están abiertos, guardando el número de puerto y el servicio asociado. Para saber si hay un puerto abierto o no puedes utilizar el comando nc, que se explica un poco más adelatne. Para conocer el servicio asociado a cada puerto tienes que recurrir al fichero tcp.csv que contiene una relación de puertos y el servicio correspondiente.
Detección del sistema operativo: nuestro script también va a intentar identificar el sistema operativo de cada uno de los equipos. Para ello, utilizarás el valor TTL (Time to Live) de la respuesta a los mensajes ping, ya que, por norma general, los sistemas Linux tienen un TTL de 64 y los Windows de 128
Almacenamiento de la información: toda la información obtenida (IP de equipos detectados, puertos abiertos y sistema operativo) se irá mostrando por pantalla y, además, se almacenará en un archivo de texto cuyo nombre indique el usuario. Se valorará la claridad en la presentación de la información.
El comando nc (netcat)
El comando nc es una herramienta muy versátil que permite realizar diveras operaciones de red como escanear puertoss, transferir archivos y configurar conexiones TCP y UDP. En nuestro caso concreto, la utilizaremos para comprobar si un puerto específico está abierto en una máquina. Para ello, necesitaremos los modificadores -z para indicar que realice el escaneo sin enviar datos y -v para habilitar la salida en modo detallado (verboso), que mostrará más información sobre el estado del escaneo.

victor@SERVER:~$ nc -zv 192.168.1.1 80
Connection to 192.168.1.1 80 port [tcp/http] succeeded!
Funcionalidades opcionales
Este es un proyecto bastante abierto en el que se valorará la inclusión de funcionalidades opcionales. Algunas sugerencias son:

Datos en parámetros: en lugar de preguntar al usuario por la IP de la red, el rango de puertos y el nombre del fichero de salida, se pueden recoger como parámetros en el script.
Informe en JSON o CSV: generar la salida en formato CSV o JSON para facilitar la lectura y análisis de los datos.
Registro de la marca de tiempo: para mostrar al final del escaneo el tiempo que ha llevado realizarlo.
Direcciones MAC: junto con la dirección IP de cada equipo detectado se puede almacenar la dirección MAC del mismo.
Recursos
Extrayendo subcadenas en Bash
Aplicar colores en Bash
Comando netcat
```ruby
# Solicitar la red en formato CIDR
read -p "Introduce la red en formato CIDR (ej. 192.168.1.0/24): " NETWORK

# Solicitar el nombre del archivo de salida
read -p "Introduce el nombre del archivo de salida (sin extensión): " OUTPUT_FILE
OUTPUT_FILE="${OUTPUT_FILE}.txt"

echo "Iniciando análisis de red..."
echo "Red analizada: $NETWORK" > "$OUTPUT_FILE"
echo "----------------------------------------" >> "$OUTPUT_FILE"

# Extraer la IP base y el rango
IP_BASE=$(echo "$NETWORK" | cut -d'/' -f1)
MASK=$(echo "$NETWORK" | cut -d'/' -f2)

# Convertir máscara en número de hosts
if [[ "$MASK" -eq 24 ]]; then
  HOSTS=256
elif [[ "$MASK" -eq 16 ]]; then
  HOSTS=65536
elif [[ "$MASK" -eq 8 ]]; then
  HOSTS=16777216
else
  echo "Máscara no válida. Solo se permiten /8, /16, o /24."
  exit 1
fi

# Escaneo de red
for i in $(seq 1 $((HOSTS-1))); do
  IP=$(echo "$IP_BASE" | awk -F '.' '{printf "%d.%d.%d.%d\n", $1,$2,$3,ENVIRON["i"]}')
  
  # Ping para ver si está activo
  ping -c 1 -W 1 "$IP" > /dev/null 2>&1
  if [[ $? -eq 0 ]]; then
    echo "Dispositivo detectado: $IP" | tee -a "$OUTPUT_FILE"
    
    # Detectar sistema operativo (TTL)
    TTL=$(ping -c 1 "$IP" | grep -oP "ttl=\K\d+")
    if [[ "$TTL" -eq 64 ]]; then
      OS="Linux"
    elif [[ "$TTL" -eq 128 ]]; then
      OS="Windows"
    else
      OS="Desconocido"
    fi
    echo "Sistema operativo: $OS" | tee -a "$OUTPUT_FILE"

    # Escanear puertos abiertos
    echo "Puertos abiertos:" >> "$OUTPUT_FILE"
    for PORT in $(seq 1 1024); do
      nc -zv -w 1 "$IP" "$PORT" 2>&1 | grep -q "succeeded"
      if [[ $? -eq 0 ]]; then
        echo "  - Puerto $PORT abierto" >> "$OUTPUT_FILE"
      fi
    done
    echo "----------------------------------------" >> "$OUTPUT_FILE"
  fi
done

echo "Análisis completado. Información guardada en $OUTPUT_FILE."

```