# Laboratorio_2_Edge_Computing

# Parte A: Observabilidad del Sistema y Cross-Compiling

### Actividad 1: Enlazado estático vs. dinámico (Ejecución Nativa - Casa)

Contexto: El sistema operativo necesita cargar las instrucciones a la memoria. Las dependencias (como iostream) pueden empaquetarse dentro del mismo archivo (estático) o invocarse en tiempo de ejecución (dinámico).

### 1. Usando el compilador estándar de su máquina (g++), compile un programa "Hola Mundo" dos veces: una con enlazado estático (-static) y otra con enlazado dinámico.

***Creación de carpeta - Instancia inicial***

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/f655e6e9-9bdf-45b8-aedb-91258922752f" />

<br>

***Compilación "Hola Mundo"***

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/4801cbed-ec96-4255-8b49-704e5a43d68e" />

<br>
### 2. Guía de Análisis:

***Ejecute ls -lh sobre ambos binarios. ¿Cuál es la proporción exacta de diferencia en tamaño?***

<img width="876" height="170" alt="image" src="https://github.com/user-attachments/assets/88daf905-ec34-4740-98a7-9a1497fb191a" />

<br>

<img width="952" height="312" alt="image" src="https://github.com/user-attachments/assets/07e7647a-2181-423d-b383-83ae2328fddf" />

<br>

El tamaño del binario dinámico es de 16496 bytes, mientras que el binario estático es de 2263216 bytes.
Esto significa que el binario estático es aproximadamente 137.2 veces más grande que el dinámico.

***Ejecute ldd <binario> en ambos. ¿Qué librerías exige el sistema operativo para que la versión dinámica funcione?***

<img width="1556" height="708" alt="image" src="https://github.com/user-attachments/assets/16ac979c-0384-4987-9767-105d97de1c7f" />

El binario dinámico depende de las siguientes librerías:
libstdc++.so.6
libc.so.6
libm.so.6
libgcc_s.so.1
ld-linux-x86-64.so.2
El binario estático mostró:
not a dynamic executable

Análisis:

Esto demuestra que la versión dinámica requiere que el sistema operativo cargue en tiempo de ejecución varias librerías compartidas necesarias para su ejecución. Estas librerías contienen funcionalidades estándar como manejo de entrada/salida, matemáticas y soporte del compilador.

En contraste, la versión estática no depende de librerías externas, ya que todas las dependencias han sido incorporadas dentro del propio ejecutable durante la compilación.

***Trazabilidad de Syscalls: Ejecute strace redirigiendo la salida a un archivo de texto para ambos casos:***

strace -o traza_dinamica.txt ./binario_dinamico

strace -o traza_estatica.txt ./binario_estatico

<img width="597" height="165" alt="image" src="https://github.com/user-attachments/assets/835a4bfb-bacc-4ff1-85a4-511bc9bad998" />

<br>

***Compare ambos archivos usando el comando diff traza_estatica.txt traza_dinamica.txt (o wc -l para contar las líneas). ¿Qué grupo de llamadas al sistema (ej. openat, mmap, mprot ect) tiene que ejecutar el kernel exclusivamente en la versión dinámica antes de poder imprimir el texto en pantalla?*** 

Se compararon con wc -l y diff.

<img width="1142" height="1293" alt="image" src="https://github.com/user-attachments/assets/b3e068aa-9428-46fc-93f6-9401132977a9" />

<br>

<img width="1159" height="662" alt="image" src="https://github.com/user-attachments/assets/2a88ed79-3060-49f9-8b67-f922382fc7f4" />
<br>
<img width="1032" height="549" alt="image" src="https://github.com/user-attachments/assets/65001090-0be6-494d-8db0-428c453ea768" />


Antes de imprimir el mensaje en pantalla, la versión dinámica ejecuta un conjunto adicional de llamadas al sistema relacionadas con la carga de bibliotecas compartidas. Las más representativas son openat, mmap y mprotect.

Estas llamadas permiten al kernel localizar las librerías dinámicas en disco, abrirlas, mapearlas en memoria y configurar sus permisos de acceso. Este proceso ocurre exclusivamente en la versión dinámica porque sus dependencias no están incluidas dentro del ejecutable.

En cambio, la versión estática no necesita cargar bibliotecas externas al inicio, por lo que realiza menos llamadas de inicialización antes de ejecutar la impresión del texto.

## Actividad 2: Flujo de Cross-Compiling (Ejecución Cruzada - Clase)

Contexto: Desarrollar en el hardware objetivo (SBC) suele ser lento. El cross-compiling permite generar código para otra arquitectura (aarch64) desde su PC.

1. Usando aarch64-linux-gnu-g++, realice las compilaciones de la Actividad 1 en su máquina host.

2. Guía de Análisis:

Utilice file <binario> para comprobar la arquitectura del ejecutable.

Ejecútelo simulando la arquitectura localmente: qemu-aarch64 -L /usr/aarch64-linux-gnu/ ./<binario>.

<img width="1213" height="349" alt="image" src="https://github.com/user-attachments/assets/5020e194-d2c3-44d6-ac43-bc5795d73679" />


Transfiera el binario a la Raspberry Pi 5 usando scp y ejecútelo nativamente para confirmar el flujo de trabajo.


# Parte B: Interacción con Hardware (Fase Presencial)

Actividad 3: GPIO moderno con libgpiod (Ejecución Cruzada - Clase)

Contexto: En Linux, el acceso directo a los registros físicos está prohibido. El espacio de usuario interactúa con los pines a través del subsistema GPIO y "character devices" expuestos en /dev/.

1. Escriba un programa en C++ para "parpadear" un pin usando la librería libgpiod, buscando el chip mediante una etiqueta (Label) y no por un número estático. Debe usar la opción -lgpiod en el comando de compilación

Guía de Análisis:

Compile cruzado su código y transfiéralo a la Raspberry Pi 5.

Ejecútelo en el hardware real. Mientras corre, abra otra terminal vía SSH y use gpioget o gpioinfo para verificar que la línea solicitada está reservada por su proceso.

# Parte C: Tiempo, Memoria y Concurrencia (Ejecución Nativa - Casa)

Actividad 4: Abstracción de Tiempo y Syscalls

Contexto: En microcontroladores sencillos, un delay() suele ser un bucle infinito ("espera activa") que bloquea el procesador. En un sistema operativo multitarea, esto es inaceptable. El proceso debe pedirle al kernel que lo "duerma" y lo despierte en el futuro, liberando la CPU.

1. Implemente una función delay(ms) usando llamadas al sistema de POSIX (nanosleep).

2. Guía de Análisis:

Escriba un programa que llame a delay(2000). Ejecútelo anteponiendo el comando time (ej. time ./su_programa).

Observe la salida (real, user, sys). ¿Por qué el tiempo user (tiempo de CPU consumido) es casi 0.000s, aunque el programa duró 2 segundos reales? ¿Quién tenía el control del procesador?


## Actividad 5: Consumo de Heap y Stack

Contexto: El espacio de direcciones tiene límites. El Heap maneja memoria dinámica (crece hacia arriba) y el Stack maneja variables locales/llamadas a funciones (crece hacia abajo).

1. Escriba un código con dos tareas:

Heap: Un ciclo infinito que reserve memoria dinámicamente y la inicialice (ej. con memset).

Stack: Una función recursiva infinita con un arreglo local muy grande (ej. 1MB).

2. Guía de Análisis:

Heap: Ejecute observando htop (columnas VIRT y RES). Tras ser cancelado, ejecute sudo dmesg | tail -n 20 e identifique el mensaje donde el OOM Killer asesina su proceso. (No ejecutar en una máquina ejecutando procesos sensibles pues puede volverse inestable)

Stack: Anote a qué profundidad exacta (MB) se detiene. Identifique el error (Segmentation fault). ¿Por qué el OS detiene este proceso mucho antes de quedarse sin RAM, a diferencia de la prueba del Heap?

## Actividad 6: Procesamiento Paralelo: Procesos vs. Hilos

Contexto: Para concurrencia, podemos crear Procesos aislados (requieren pipes para comunicarse) o Hilos (comparten memoria, pero requieren mutexes).

1. Implemente un programa que cuente la frecuencia de palabras del archivo https://www.gutenberg.org/ebooks/2000.txt.utf-8. Al final de cada programa incluya un llamado a sleep o entrada de datos de usuario de modo que le dé tiempo de realizar el análisis antes de que el programa termine.

Versión Procesos: El padre lee y envía palabras por un pipe al hijo, quien las cuenta.

Versión Hilos: Un hilo encola las palabras; otro hilo las desencola y cuenta usando pthread_mutex.

2. Guía de Análisis:

Ejecute la versión de Procesos. En top (o htop pero las instrucciones pueden cambiar), observe que existen dos filas con PID distintos. ¿Se duplicó la memoria RES consumida?

Ejecute la versión de Hilos. En top, presione f, active TGID (Thread Group ID) y vuelva con q. Presione H para ver los hilos individualmente.

Explique qué significa que el PID (Thread ID) sea diferente para cada hilo, pero compartan el mismo TGID y la misma cantidad exacta de memoria RES
