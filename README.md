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

<img width="1245" height="101" alt="image" src="https://github.com/user-attachments/assets/cf77c251-cab0-4a20-900b-5ac90804311e" />

Ejecútelo simulando la arquitectura localmente: qemu-aarch64 -L /usr/aarch64-linux-gnu/ ./<binario>.

<img width="1213" height="349" alt="image" src="https://github.com/user-attachments/assets/5020e194-d2c3-44d6-ac43-bc5795d73679" />

Transfiera el binario a la Raspberry Pi 5 usando scp y ejecútelo nativamente para confirmar el flujo de trabajo.

<img width="1600" height="229" alt="image" src="https://github.com/user-attachments/assets/2bb71a82-abed-4d90-b958-fd3dba948100" />

Se realizó la compilación cruzada del programa utilizando aarch64-linux-gnu-g++, generando ejecutables para la arquitectura ARM64 desde una máquina host distinta. Mediante el comando file se verificó que los binarios correspondían a ejecutables ELF para aarch64, confirmando que fueron correctamente construidos para la arquitectura objetivo. Posteriormente, se ejecutaron en la máquina local usando qemu-aarch64, lo que permitió simular su funcionamiento y validar que el programa operaba correctamente al imprimir “Hola Mundo”. Finalmente, los binarios fueron transferidos a la Raspberry Pi 5 mediante scp y ejecutados de forma nativa, obteniendo el mismo resultado. Esto demuestra que el flujo de cross-compiling fue exitoso y que los ejecutables son portables entre diferentes arquitecturas, validando tanto la emulación como la ejecución real en el hardware destino.

# Parte B: Interacción con Hardware (Fase Presencial)

Actividad 3: GPIO moderno con libgpiod (Ejecución Cruzada - Clase)

Contexto: En Linux, el acceso directo a los registros físicos está prohibido. El espacio de usuario interactúa con los pines a través del subsistema GPIO y "character devices" expuestos en /dev/.

1. Escriba un programa en C++ para "parpadear" un pin usando la librería libgpiod, buscando el chip mediante una etiqueta (Label) y no por un número estático. Debe usar la opción -lgpiod en el comando de compilación

Guía de Análisis:

Compile cruzado su código y transfiéralo a la Raspberry Pi 5.

Ejecútelo en el hardware real. Mientras corre, abra otra terminal vía SSH y use gpioget o gpioinfo para verificar que la línea solicitada está reservada por su proceso.







## Solución
Para esta actividad se desarrolló un programa en C++ que permite controlar un pin GPIO utilizando la librería libgpiod, la cual es la interfaz moderna en Linux para interactuar con los pines mediante dispositivos de tipo character device ubicados en /dev/.

El programa no accede al chip por un número fijo, sino que busca dinámicamente el chip GPIO utilizando su etiqueta (label), en este caso "pinctrl-rp1", lo cual hace el código más portable y adaptable al hardware.

<img width="1600" height="900" alt="WhatsApp Image 2026-04-21 at 2 46 14 PM" src="https://github.com/user-attachments/assets/1cc26a29-0b77-4c46-910f-baf7e26eea23" />

### Compilación

El programa fue compilado utilizando el comando:
*g++ blink_gpiod.cpp -o blink_gpiod -lgpiod*

### Ejecución en Raspberry Pi 5
El ejecutable fue transferido a la Raspberry Pi 5 y ejecutado directamente en el hardware. Al correr el programa, se logró identificar correctamente el chip GPIO mediante su etiqueta, y se solicitó la línea correspondiente al GPIO 17 como salida.

El programa realizó el parpadeo del pin, alternando entre estados HIGH (ON) y LOW (OFF) cada 500 ms durante 10 ciclos, lo cual fue verificado visualmente en el hardware.

### Análisis
El uso de libgpiod demuestra el modelo moderno de acceso a hardware en Linux, donde el acceso directo a registros está prohibido y en su lugar se utilizan interfaces controladas por el kernel. Esto garantiza seguridad, control de acceso y correcta administración de los recursos hardware entre múltiples procesos.

https://github.com/user-attachments/assets/8aa647e8-342d-4bab-bdaa-58c64bc26abf

# Parte C: Tiempo, Memoria y Concurrencia (Ejecución Nativa - Casa)

## Actividad 4: Abstracción de Tiempo y Syscalls

Contexto: En microcontroladores sencillos, un delay() suele ser un bucle infinito ("espera activa") que bloquea el procesador. En un sistema operativo multitarea, esto es inaceptable. El proceso debe pedirle al kernel que lo "duerma" y lo despierte en el futuro, liberando la CPU.

<img width="530" height="77" alt="image" src="https://github.com/user-attachments/assets/205738c5-b994-4258-aed7-a1808cfb26ba" />
Figura 1. Verificación de la distribución Linux instalada en WSL. Se observa Ubuntu ejecutándose sobre WSL2, lo que permite utilizar llamadas al sistema POSIX como nanosleep().

<img width="495" height="60" alt="image" src="https://github.com/user-attachments/assets/eb056e97-53e5-4258-bbac-d04c5a247a4a" />
Figura 2. Entorno Linux (Ubuntu en WSL) listo para compilar y ejecutar programas en C utilizando el kernel Linux.

<img width="907" height="118" alt="image" src="https://github.com/user-attachments/assets/b424bae1-80d9-4de9-8d27-b6dd95c9a223" />
Figura 3. Verificación del compilador GCC instalado. GCC permite compilar programas en C que utilizan llamadas al sistema POSIX.

<img width="554" height="602" alt="image" src="https://github.com/user-attachments/assets/c2b65280-819e-43a3-babb-328fc9d9d72e" />
Figura 4. Implementación de la función delay(ms) utilizando la llamada al sistema nanosleep(). La función convierte milisegundos a segundos y nanosegundos para que el kernel gestione la suspensión del proceso.

<img width="739" height="24" alt="image" src="https://github.com/user-attachments/assets/b823cf45-4ca4-412f-babc-f53dfab26636" />
Figura 5. Compilación exitosa del programa. El archivo ejecutable delay es generado sin errores.

<img width="654" height="155" alt="image" src="https://github.com/user-attachments/assets/cae0fb8e-ebba-4c1b-b9c3-59285da2df94" />
Figura 6. Medición de tiempos con time. El valor real refleja el tiempo total transcurrido (≈2 s), mientras que user es cercano a 0 debido a que el proceso fue suspendido por el kernel mediante nanosleep(), liberando la CPU para otros procesos.


## Actividad 5: Consumo de Heap y Stack

Contexto: El espacio de direcciones tiene límites. El Heap maneja memoria dinámica (crece hacia arriba) y el Stack maneja variables locales/llamadas a funciones (crece hacia abajo).

1. Escriba un código con dos tareas:

<img width="470" height="502" alt="WhatsApp Image 2026-04-21 at 9 07 35 PM" src="https://github.com/user-attachments/assets/8e69a7f6-ecb1-4c3b-82c2-cffa6c6ab11d" />



Heap: Un ciclo infinito que reserve memoria dinámicamente y la inicialice (ej. con memset).

Stack: Una función recursiva infinita con un arreglo local muy grande (ej. 1MB).

2. Guía de Análisis:

Heap: Ejecute observando htop (columnas VIRT y RES). Tras ser cancelado, ejecute sudo dmesg | tail -n 20 e identifique el mensaje donde el OOM Killer asesina su proceso. (No ejecutar en una máquina ejecutando procesos sensibles pues puede volverse inestable)

<img width="777" height="488" alt="image" src="https://github.com/user-attachments/assets/4658de0b-f01c-49e1-b77a-241925ed6ca2" />

<img width="1228" height="757" alt="image" src="https://github.com/user-attachments/assets/434eb538-5845-4eba-971b-1f803662c262" />

<img width="535" height="369" alt="image" src="https://github.com/user-attachments/assets/a3ab517c-3415-42f0-8da2-d4f819996f03" />

<img width="1230" height="420" alt="image" src="https://github.com/user-attachments/assets/7614f3a6-d3bf-4295-afcb-6f602ebda186" />


Stack: Anote a qué profundidad exacta (MB) se detiene. Identifique el error (Segmentation fault). ¿Por qué el OS detiene este proceso mucho antes de quedarse sin RAM, a diferencia de la prueba del Heap?

<img width="713" height="470" alt="image" src="https://github.com/user-attachments/assets/7b927609-f022-4221-ac55-449302dd6dd5" />

<img width="785" height="380" alt="image" src="https://github.com/user-attachments/assets/76bd069e-a780-4225-8ee6-ebfb11e37d4e" />

En la prueba de Heap se ejecutó un programa que reservaba memoria dinámicamente en bloques de 1 MB dentro de un ciclo infinito, inicializando cada bloque con memset. En htop se observó el proceso heap_test con un crecimiento progresivo de memoria, alcanzando aproximadamente 1379 MB de memoria virtual (VIRT) y 1143 MB de memoria residente (RES). Esto demuestra que la memoria no solo fue reservada, sino también usada físicamente en RAM.

La salida del programa mostró que se llegó a reservar cerca de 1961 MB en el Heap, momento en el cual el sistema terminó el proceso. Esto se confirmó con el comando sudo dmesg | tail -n 20, donde apareció el mensaje “Out of memory: Killed process 8740 (heap_test)”, indicando que el OOM Killer del sistema operativo intervino al agotarse la memoria disponible.

El OOM Killer mata el proceso porque el Heap puede seguir creciendo dinámicamente mientras haya recursos disponibles en el sistema. Cuando la memoria global se vuelve insuficiente, el kernel selecciona y finaliza procesos para proteger la estabilidad del sistema.

## Actividad 6: Procesamiento Paralelo: Procesos vs. Hilos

Contexto: Para concurrencia, podemos crear Procesos aislados (requieren pipes para comunicarse) o Hilos (comparten memoria, pero requieren mutexes).

1. Implemente un programa que cuente la frecuencia de palabras del archivo https://www.gutenberg.org/ebooks/2000.txt.utf-8. Al final de cada programa incluya un llamado a sleep o entrada de datos de usuario de modo que le dé tiempo de realizar el análisis antes de que el programa termine.

<img width="705" height="294" alt="image" src="https://github.com/user-attachments/assets/db05e04f-868d-4416-8b64-c683e36dccc2" />

<img width="752" height="647" alt="image" src="https://github.com/user-attachments/assets/956713b8-1967-496c-8c81-1f68299bcdff" />

<img width="782" height="661" alt="image" src="https://github.com/user-attachments/assets/ae984c69-af08-4684-a9ae-f1e2d66cf4ce" />


Versión Procesos: El padre lee y envía palabras por un pipe al hijo, quien las cuenta.

Versión Hilos: Un hilo encola las palabras; otro hilo las desencola y cuenta usando pthread_mutex.

2. Guía de Análisis:

Ejecute la versión de Procesos. En top (o htop pero las instrucciones pueden cambiar), observe que existen dos filas con PID distintos. ¿Se duplicó la memoria RES consumida?

<img width="571" height="158" alt="image" src="https://github.com/user-attachments/assets/783056f0-0374-4a9b-8aab-268190193818" />

<img width="1150" height="686" alt="image" src="https://github.com/user-attachments/assets/58733628-2932-4218-9c00-0d01a4661c3b" />

<br>

En la versión con procesos, al ejecutar el programa y observar en top, se evidencian dos filas con PID distintos correspondientes al proceso padre y al proceso hijo. Sin embargo, la memoria RES no se duplica completamente, ya que el sistema operativo utiliza el mecanismo de copy-on-write, mediante el cual ambos procesos comparten inicialmente las mismas páginas de memoria y solo se duplican cuando alguno de ellos realiza modificaciones.


Ejecute la versión de Hilos. En top, presione f, active TGID (Thread Group ID) y vuelva con q. Presione H para ver los hilos individualmente.

<img width="432" height="251" alt="image" src="https://github.com/user-attachments/assets/c52b954a-1e6b-48ad-82f6-573817657fb5" />

<img width="1228" height="708" alt="image" src="https://github.com/user-attachments/assets/7df2a999-80f7-4797-a836-888f5c3a7ca5" />

<img width="1231" height="740" alt="image" src="https://github.com/user-attachments/assets/332c7757-05d3-4205-8032-4f691c47828c" />

<img width="1235" height="709" alt="image" src="https://github.com/user-attachments/assets/580cd435-7a02-444d-af21-ac935db2ba3c" />

En la versión con hilos, al activar la visualización de hilos con la tecla H y habilitar la columna TGID en top, se pueden observar dos filas correspondientes a los diferentes hilos del programa. Cada hilo aparece individualmente en la lista, permitiendo analizar su comportamiento de manera independiente dentro del mismo proceso.

Explique qué significa que el PID (Thread ID) sea diferente para cada hilo, pero compartan el mismo TGID y la misma cantidad exacta de memoria RES

El hecho de que cada hilo tenga un PID (Thread ID) diferente indica que el kernel los gestiona de forma individual. Sin embargo, todos comparten el mismo TGID, lo que significa que pertenecen al mismo proceso. Además, presentan el mismo valor de memoria RES, lo cual confirma que los hilos comparten el mismo espacio de memoria. Esto permite una comunicación más eficiente entre ellos, pero requiere mecanismos de sincronización como mutexes para evitar condiciones de carrera.

