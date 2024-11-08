PR0302: Ejercicios comando case
Realiza las siguientes tareas:

Ejercicio 1: Menú de operaciones matemáticas
Crea un script que presente un menú con las operaciones básicas (suma, resta, multiplicación, división) y solicite al usuario que elija una operación. El script debe ejecutar la operación seleccionada utilizando case.
```bash
echo "Elija la entre las siguiente opciones"
echo "1) Suma"
echo "2) Resta"
echo "3) División"
echo "4) Multiplicación"
read opcion

echo "Introduzca el primer número"
read n1

echo "Introduzca el segundo número"
read n2

case $opcion in
        1)
                r=$(($n1 + $n2)) ;;
        2)
                r=$(($n1 - $n2)) ;;
        3)
                r=$(($n1 / $n2)) ;;
        4)
                r=$(($n1 * $n2)) ;;
esac

echo "El resultado de la operación es $r"
```

Ejercicio 2: Identificación de extensión de archivo
Haz un script que solicite al usuario un nombre de archivo y, usando case, determine si es un archivo de texto (.txt), un archivo comprimido (.zip o .tar.gz), o cualquier otro tipo de archivo.
```bash
echo "Inserte la extensión (.pdf, .txt, .jpg)"
read a

case $a in
        .txt)
                echo "Archivo de texto." ;;
        .jpg|.jpeg|.png|.gif)
                echo "Archivo de imagen." ;;
        .pdf)
                echo "Documento PDF." ;;
        .doc|.docx)
                echo "Documento de Word." ;;
        .xls|.xlsx)
                echo "Hoja de cálculo de Excel." ;;
        .ppt|.pptx)
                echo "Presentación de PowerPoint." ;;
        .mp3|.wav)
                echo "Archivo de audio." ;;
        .mp4|.avi|.mkv)
                echo "Archivo de video." ;;
        *)
                 echo "Tipo de archivo desconocido." ;;
esac
```

Ejercicio 3: Conversor de unidades
Crea un script que ofrezca un menú para convertir entre diferentes unidades de longitud (metros a kilómetros, pies a metros, etc.) utilizando case para gestionar las conversiones.
```bash
echo "Inserte el tipo de cambio"
echo "1) M a KM"
echo "2) KM a M"
echo "3) FT a M"
echo "4) M a FT"

read op
read -p "Ingrese el valor: " valor

case $op in
        1)
                echo "$valor metros = $(echo "$valor / 1000" | bc -l) kilómetros" ;;
        2)
                echo "$valor kilómetros = $(echo "$valor * 1000" | bc -l) metros" ;;
        3)
                echo "$valor pies = $(echo "$valor * 0.3048" | bc -l) metros" ;;
        4)
                echo "$valor metros = $(echo "$valor / 0.3048" | bc -l) pies" ;;
esac
```

Ejercicio 4: Menú de configuración del sistema
Realiza un script que presente un menú con varias opciones de configuración del sistema: por ejemplo, apagar, reiniciar o cerrar sesión. Usa case para gestionar las opciones seleccionadas.
```bash
echo "Elija la opción que desee"
echo "1) Apagar"
echo "2) Reiniciar"
echo "3) Cerrar sesión"
read op

case $op in
        1)
                sudo shutdown now ;;
        2)
                sudo reboot ;;
        3)
                gnome-session-quit --logout --no-prompt ;;
esac
```

Ejercicio 5: Día de la semana
Haz un script que solicite un número entre 1 y 7 al usuario y muestre el nombre del día correspondiente (1 para lunes, 2 para martes, etc.) utilizando case.
```bash
echo "Inserte un número entre el 1 y el 7 y te devolveré el día"
read num

case $num in
        1)
                echo "El $num es Lunes" ;;
        2)
                echo "El $num es Martes" ;;
        3)
                echo "El $num es Miércoles" ;;
        4)
                echo "El $num es Jueves" ;;
        5)
                echo "El $num es Viernes" ;;
        6)
                echo "El $num es Sábado" ;;
        7)
                echo "El $num es Domingo" ;;
esac
```

Ejercicio 6: Clasificación de notas
Realiza un script que solicite una calificación numérica y, usando case, la clasifique como "Sobresaliente", "Notable", "Aprobado", o "Suspenso" según el rango de valores.
```bash
echo "Inserte una nota entera del 1 al 10"
read nota

case $nota in
        0|1|2|3|4)
                echo "El $nota es un suspenso" ;;
        5)
                echo "El $nota es un suficente" ;;
        6)
                echo "El $nota es un bien" ;;
        7|8)
                echo "El $nota es un notable" ;;
        9|10)
                echo "El $nota es un sobresaliente" ;;
esac
```

Ejercicio 7: Clasificación de números
Haz un script que solicite un número y lo clasifique como "Positivo", "Negativo" o "Cero".
```bash
read -p "Introduzca un número entero" num

case $num in
        0)
                echo "El número es 0" ;;
        -*)
                echo "El número $num es negativo" ;;
        *)
                echo "El número $num es positivo" ;;
esac
```

Ejercicio 8: Control de servicios en Linux
Crea un script que pregunte por el nombre de un servicio y luego presente un menú para iniciar, detener o reiniciar un servicio en Linux (como apache2 o nginx). Usa case para gestionar las opciones y los comandos correspondientes (systemctl start, stop, restart).
```bash

```

Después de realizar la operación solicitada comprueba su código de estado de finalización (recuerda que puedes obtener el estado de finalización de un comando con la variable $? tras ejecutarlo) y muestra un mensaje indicando si la operación se ha realizado correctamente o no.
```bash

```