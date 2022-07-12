# 1 - Introduccion
## SO
- Una forma de dividir a los sistemas informaticos. Es lo que esta en el medio entre el hardware y el software
- Es una pieza de software que hace de intermediario entre el HW y los programas de usuario
- Tiene que manejar:
  - contencion (conflicto en el acceso a recurso compartido), mediante mecanismos de bajo nivel como semaforos, mutex, queues, etc
  - concurrencia
  
  De manera tal de lograr:
    - hacerlo con buen rendimiento
    - hacerlo correctamente 

Tiene 2 responsabilidades:
- El software especifico no se tenga que preocupar con detalles bajo de nivel de HW
- El usuario use correctamente el HW

Multiprogramacion: permite que varios programas funcionen a la vez, lo cual implica que:
- Throughput: puede ser que un trabajo tome mas tiempo que antes, pero ahora el conjunto de programas ejecutandose tardan menos
- Contencion: ahora hay multiples programas que pueden querer acceder al mismo recurso

Timesharing: es una variacion de la multiprogramacion, que va cambiando entre los programas que se estan ejecutando para darle tiempo de procesamiento a cada uno

UNIX sistema operativo historico
Linux insipirado en UNIX

### Abstraccion del HW
El SO nos abstrae un monton de informacion especifica del hardware que no afecta al programa que estamos usando / desarrollando. Por ejemplo, no necesitamos saber como funciona un disquette y su construccion para utilizarlo, el SO nos provee una primitiva open y close por ejemplo y las usamos transparentemente.

### Utilidades
Los SO traen utilidades, programas que vienen con el SO que van a ser utiles para el usuario. 
Hay distintos niveles de utilidades, por ejemplo la calculadora es algo util pero no necesario, en cambio la ventana de procesos y la de impresion son mas importantes para operar en el SO.

### Que es y que no es un SO
Tenemos el siguiente stack en una computadora:

```
Progamas de usuario
Shell
Nucleo  -------------------/ SO
Drivers ------------------/
Lo pateable (HW)
```

Los drivers son del SO y hablan con el hardware

El nucleo / core / kernel es del SO, y los programas de usuario hablan con el nucleo.

A veces se usa SO para referirse a mas cosas que estas 2, como el Shell, pero este igualmente es un programa mas. La terminal es un Shell y un poco mas. Shell intermedia entre los programas de usuario y el kernel para que se comuniquen, y tambien para lanzar procesos.


# 2 - Procesos y otras yerbas
Un programa puesto a ejecutar es un proceso. Como escribo un programa una vez lo puedo ejecutar multiples veces y tener asi varios procesos con distintos caminos de ejecucion.
## Process ID
El SO le asigna a cada proceso un process id, abreviado como pid.

## Composicion de un proceso
- El area de texto, donde esta el codigo de maquina del programa
- El area de datos, donde se almacena el heap
- El stack del proceso
- Las variables locales van al stack
- Las variables dinamicas van al heap

## Que puedo hacer con un proceso
- Terminar
  - El proceso le indica al SO que puede liberar los recursos que tiene el mismo, como la memoria, mediante exit()
  - Cuando termina indica el status en el que termino, representado por un numero, y este es reportado al proceso padre
- Lanzar un proceso hijo mediante una primitiva del SO (system(), fork(), exec())
- Ejecutarse en la CPU
  - hace operaciones entre registros y direcciones de memoria
- Ejecutar un system call (llamar al kernel)
  - Las system calls son funciones provistas por el kernel
  - Ejemplos de estas son fork(), exec(), imprimir en pantalla mediante write(), etc.
  - Son costosas, ya que hay que estamos invocando al kernel, y tiene que haber un *cambio de contexto* para cambiar el privilegio a nivel 0.
- Hablar con dispositivos de E/S
    - Es muy lenta
    - Es dificil de predecir
    - Un proceso se queda bloqueando esperando la respuesta de la E/S, y genera lo que se llama *busy waiting*, desperdicir ciclos de CPU al pedo.
    - Hay alternativas para evitar perder ciclos de CPU cuando se bloquea un proceso, y estos son:
      - Polling
      - Interrupciones: Se manda a dormir al proceso hasta que su E/S este lista, no otorgandole mas quantums, y cuando la E/S termine la interrupcion asociada a esta sera atendida por el SO y lo despertara al proceso devuelta.
## Arbol de procesos
Los procesos de un SO estan organizados como un arbol, donde la raiz es lanzado por el SO al comenzar, y se lo suele llamar init o systemd.
Un proceso hijo luego es lanzado mediante fork(), que es un calco al padre que lo lanzo. Lo unico distinto entre estos 2 es el PID, el cual es retornado por fork() al padre, y al hijo el fork le retorna 0 para que el hijo sepa que es el hijo.
El proceso padre puede suspenderse hasta que su hijo termine llamando a wait(), y al terminar el hijo el padre obtiene su status de exit.

### Proceso detras de lanzar un programa en Shell
El shell hace fork(), entonces ahora hay un proceso hijo de este, y luego hace wait hasta que termina el hijo. Esto lo hace para cederle el control al proceso hijo ya que no tiene nada que hacer.
Luego el hijo hace un exec con el path del programa que recibio el shell originalmente, para que se ejecute el codigo del programa asociado al path. Eventualmente este proceso hijo termina y el shell vuelve a tener el control.
```
        parent                      resumes
        |------------------>wait()---------->
        |                    ^
fork()--|                    |
        |---->exec()------->exit()
        child
```

## Cuanto tiempo deja un SO que se ejecute un proceso en CPU?
Hay varias opciones:
- Hasta que este termine: Esto es bueno desde la perspectiva del proceso, pero no para el sistema en conjunto. Ademas podria no terminar este.
- Un tiempito, lo cual se denomina quantum. Esto implica decidir cuanto tiempo y a quien le toca siguiente entre todos los procesos. 
  - Para hacer esto los SO hacen preemption, que es luego de un quantum suspender al proceso actual, que le toque al siguiente, y eventualmente volver a darle un tiempo a este
    - Para lograr esto se configura la interrupcion de clock, para que luego de X interrupciones (el quantum que se le da al proceso) se le de el control al SO, y este le pregunte al scheduler cual es el siguiente proceso que deberia ejecutarse
    - Para cambiar el proceso que se va a ejecutar ahora hay que hacer un *cambio de contexto*:
      - Guardar registros que corresponden al proceso actual
      - Guardar el IP que corresponden al proceso actual
      - Si es la primera vez que aparece el proceso siguiente, cargarlo en memoria
      - Cargar registros del proceso siguiente
      - Poner el valor del IP del proceso siguiente
    - Todos los datos utilizados para este *cambio de contexto* se guardan en una estructura llamada *PCB* (Process Control Block). Es literalmente una tabla que tiene una relacion 1:1 con el PID.
    - El *cambio de contexto* es tiempo muerto ya que ningun proceso esta ejecutandose durante este tiempo, lo cual implica 2 cosas:
      - Impacto en la arquitectura del HW.
      - Es fundamental determinar un quantum optimo para minimizar los cambios de contexto y asi minimizar el tiempo muerto. No demasiado para no pagar mucho tiempo muerto, pero lo suficiente para que se ejecuten los procesos que conviene que se ejecuten.


## Definiciones con Multi
- Multiprocesador: un equipo con mas de un procesador
- Multiprogramacion: la capacidad del SO de tener varios procesos en ejecucion
- Multoprocesamiento: cuando se tiene multiprogramacion en un multiprocesador
- Multitarea: un caso de multiprogramacion, donde el switcheo entre procesos es tan rapido que parece que estan corriendo todos los procesos en paralelo
- Multithreaded: son procesos que tienen "mini procesos" corriendo en paralelo

## Estado de un proceso
Un proceso puede estar en 4 estados, desde los cuales tiene distintas transiciones posibles:
- Terminado: F
- Corriendo: ejecutandose en la CPU
- Listo: listo para correr, esperando a su turno
- Bloqueado: esperando E/S por ejemplo

Un diagrama de esto mismo con las transiciones es este:

![alt text](2/proceso_estados.png)

## Carga del sistema
Hace referencia a cuantos procesos listos tiene.


Cosas importantes
- Concepto de proceso en detalle
- Estado del proceso
- System call
  - Caros
  - Cambios de contexto
- Quantum
  - Los procesos se ejecutan durante un *quantum* y luego el SO elige otro (esto ultimo para la clase 3).


# 3 - IPC
Comunicacion entre procesos (Inter Process Communication)
Un proceso tiene varias formas de comunicarse con otro:
- Memoria compartida: Hay un cacho de memoria compartida, en el cual para comunicarse los procesos leen y escriben para hablarse.
- Algun otro recurso compartido como un archivo
- Pasaje de mensajes <- ESTE ES EL INTERESANTE
  - Si el proceso A quiere hablarle al B le envia un mensaje al kernel, y este se encarga de hacerle llegar el mensaje al otro proceso.

![alt text](2/memoria_compartida_vs_pasaje_mensajes.png)

## Pipes
Es un canal de comunicacion con 2 extremos, donde cada proceso esta en un extremo escribiendo y leyendo mensajes.
Un pipe es como un pseudo archivo, que tiene las primitivas read y write.

## IPC sincronica / asincronica

### Sincronica
Un write sincronico no devuelve el control al proceso hasta que el mensaje llego al otro lado, osea, hasta que fue LEIDO por el otro proceso. El proceso queda bloqueado mientras esto pasa.
- PROS: Es sencillo de programar
- CONS: Es bloqueante
### Asincronica
Un write asincronico le devuelve el control al toque al proceso y este puede continuar con su vida. 
- PROS: No es bloqueante, libera al emisor para que haga otras tareas.
- CONS: Hay que implementar algun mecanismo extra para saber que el mensaje llego.


# 4 - Scheduling
Scheduling se encarga de decidir de todos los procesos que tengo en ready cual pongo en run
Tiene 2 decisiones clave:
- A quien pongo a correr
- Por cuanto tiempo lo dejo corriendo
  
Una principal caracteristica de un SO es su politica de scheduling, lo cual tiene un gran impacto sobre la performance
Es tan importante que un SO puede proveer multiples politicas de scheduling dependiendo del uso que se le esta dando.

## Opimizacion de la politica de scheduling
- Ecuanimidad: cada proceso recibe un quantum justo de CPU
- Eficiencia: que la CPU este ocupada todo el tiempo
- Carga del sistema: minimizar cant de procesos en estado ready (hacer que la mayor parte se encuentre bloqueada)
- Tiempo de respuesta: favorecer a los procesos que tienen que ver con cosas interactivas, para que el usuario perciba el minimo tiempo de respuesta posible
- Latencia: minimizar tiempo requerido pára que un proceso aporte valor, de algun resultado
- Tiempo de ejecucion: minimizar el tiempo total que le toma a un prcoeso ejecutar completamente
- Rendimiento (throughput): a diferencia del anterior, es maximizar el numero de procesos terminados en una unidad de tiempo, a diferencia de 1 como en el anterior
- Liberacion de recursos: priorizar que los procesos que tienen recursos para ellos terminen lo antes posible para que los liberen

Estos objetivos no son posibles de cumplir a la vez, porque puede haber usuarios con distintos intereses.

La politica de scheduling a implementar va a buscar maximizar una combinacion de los objetivos impactando lo menos posible en los demas objetivos.

## Cuando actua el scheduler?
Hay 2 tipos de scheduling:
- Cooperativo: Los procesos le avisan al scheduler mediante una syscall que lo desaloje. No queda mucho de estos.
- Desalojo: El scheduler usa la interrupcion del clock para desalojar o no al proceso actual y si lo desaloja poner al siguiente.

## A quien pongo de los procesos en ready?
Hay multiples enfoques:
- FIFO: El primero que entra es el primero que sale. Es ingenuo porque los trata a todos los procesos por igual.
- FIFO con prioridades: Si llega un proceso mas importante que el que iba siguiente, mando al proceso nuevo mas importante.
  - Esto puede traer un problema llamado *inanicion* (starvation). Los procesos de menor prioridad son demorados por los de mayor prioridad.
  - Una solucion para esto es ir aumentando  la prioridad de los procesos que se van quedando por varias iteraciones en la cola.
- Round Robin: agarro los procesos, le doy un quantum al 1, le doy al 2, etc.
  - La pregunta es cuanto dura el quantum.
    - Si es muy largo y el SO es interactivo el sistema va a parecer tildado.
    - Si es muy corto el tiempo de mantenimiento del *scheduler* + *context switch* se vuelve proporcionalmente muy grande y hay mucho menos trabajo de procesos posta.
  - Se le puede ir dando menos quantum a un proceso cada vez que le toca devuelta, para que los procesos que son mas *CPU hungry* sean mas penalizados y tengan mas tiempo los que no la usan.
  - Se le puede dar mas quantum a un proceso que hace E/S porque ocupa menos el CPU ya que esta en estado *wait* esperando a la E/S.
- Multiples queues: Colas que tengan cada una un quantum distinto, por ejemplo 1, 2, 4 y 8 c/u.
  - Siempre se elige al de la cola con menos quantum, pensando que van a terminar rapido.
  - Si un proceso no termina con el quantum de CPU que se le dio, se lo pasa a la cola con mas quantum para que si llegue a terminar, pero va a tener menos prioridad para tener el CPU a futuro.
  - Los procesos con mas prioridad son los interactivos para que el usuario perciba poco tiempo de rta.
  - Se puede poner a un proceso en maxima prioridad cuando hace E/S.
  - Con este esquema se minimiza el tiempo de respuesta para procesos interactivos, asumiendo que los computos largos no son afectados por las demoras de tiempo.
- SJF (Shortest Job First): Se usa en sistemas para trabajos batch, y maximiza el throughput.
  - La idea es ir analizando estadisticamente a los procesos, recuperando informacion para estimar cuanto tarda el proceso.

## Scheduling para RT
En los sistemas de tiempo real no hay que pasarse del tiempo que se le da a los procesos.
Se suele hacer un sistema de deadline mas cercano, donde se fija lo mas urgente que tiene y se le da a ese proceso prioridad.

## Scheduling para SMP (Multiple symmetric processors)
### Desde la perspectiva del proceso
Tiene un problema principalmente para la cache, particularmente cuando se libera un CPU y se tiene un proceso que estaba en otro CPU ejecutandose. Aca hay 2 opciones:
- Se puede migrar al proceso a ese otro CPU, pero puede ser que tenia datos en la cache y eso lo va a hacer mas lento en este.
- Se puede intentar dejar a este proceso para que corra en el mismo CPU que viene corriendo. A esto se lo conoce como *afinidad al proceador*.

### Desde la perspectiva del CPU
Se puede buscar distribuir la carga entre todos los CPUs, en lugar de dejar a uno sin uso porque los procesos que estan en ready tienen afinidad con los otros CPUs. Esto tiene 2 conceptos:
- Push migration: Un CPU esta  muy cargado y toma los procesos de su cola y los *pushea* a la cola de otro CPU para balancear la carga.
- Pull migration: Un CPU esta bastante libre y ve que los demas CPU tiene mas carga y se *pullea* procesos de las colas de los otros CPUs para distribuir mejor la carga.

## Arquitectura NUMA
Los CPU tienen su porcion de memoria que acceden rapido, pero eventualmente pueden acceder a la memoria del otro CPU, lo cual es mas lento. La memoria del sistema es la suma de la memoria de cada CPU, se ve como una sola
## Arquitectura UMA
A diferencia de NUMA, acceder a cualquier porcion de memoria del sistema toma lo mismo, pero es un promedio entre lo mas rapido y mas lento de NUMA.

## IMPORTANTE
Schapa dijo leer libros y darle importancia a los ejemplos numericos para entender los algoritmos


# 6 - Sincronizacion entre procesos P1
Como hacer que varios procesos cooperen estorbandose lo menos posible.
Cuando se tienen 2 procesos corriendo simultaneamente, se dice que la ejecucion tiene que dar un resultado equivalente a ulguna ejecucion secuencial de los mismos procesos. Esto seria equivalente a decir que la ejecucion intercalada entre 2 procesos A y B deberia ser lo mismo que la ejecucion secuencial A -> B o B -> A.

## Race condition
Lo que sucede cuando el resultado final varia dependiendo del momento exacto en el que se ejecutan las instrucciones

Un programa esta libre de una *race condition* si no se puede dar un entrelazado entre las ejecuciones tal que de mal el resultado

## Solucion a race condition
Una forma de arreglarlo es garantizando *exclusion mutua* mediante *secciones criticas*.

### Seccion critica
Es un cacho de codigo tal que:
- Solo un proceso que esta ejecutando a un programa se encuentra a la vez en la *seccion critica*
- Todo proceso esperando a entrar a una *seccion critica* eventualmente entra.
- Ningun proceso fuera de la *seccion critica* puede bloquearle el acceso a la *seccion critica* a otro proceso.

Se puede pensar que entrar a la *seccion critica* es como poner un cartel que diga no molestar en la puerta.

#### Implementacion de seccion critica

##### TAS
El HW provee una instruccion que permite establecer atomicamente el valor de una variable entera en 1.
Esta instruccion se la denomina TestAndSet, la cual pone en 1 un registro y devuelve el valor anterior, de manera atomica, por lo cual no puede ser interrumpido mientras lo hace. TAS (TestAndSet) es una instruccion de asm.

Se puede usar a TAS para entrar y salir de una seccion critica asi:

while(TAS(lock)){} <---> El proceso se queda intentando cambiar el lock a true hasta que vea que el fue el que lo cambio de false a true, porque eso significaria que nadie estaba en la seccion critica ya que el lock valia false, y el fue el primero en ponerlo en true, por lo que puede entrar
// entre a seccion critica


lock = false;
// sali de seccion critica

Esta forma de quedarse esperando para entrar preguntando todo el tiempo se lo denomina *busy waiting* y tiene las siguientes caracteristicas:
- El proceso se queda todo el tiempo de su quantum intentando conseguir un lock nomas
- Consume CPU al pedo


### Productor-Consumidor
Tengo un buffer de tamaño acotado, donde tengo 2 tipos de agentes:
- Productor: pone cosas en el buffer
- Consumidor: saca cosas del buffer

Me gustaria decirle al productor que deje de poner cosas si no hay espacio
Tambien al consumidor me gustaria decirle que no saque cosas si no hay

El tema de esto es que hay un problema de concurrencia con las variables ed posiciones vacias y ocupadas del buffer.
El desafio es resolver este problema sin usar *busy waiting*, para lo cual surgen los semaforos.

#### Semaforo
Variable entera con las siguientes caracteristicas:
- Se puede iniciar con cualquier valor
- Se puede manipular mediante 2 ops:
  - wait()
  - signal()
- wait(s): while(s<=0) dormir(); s--;
- signal(s): s++; if (alguien espera por s) wakeup alguno.

IMPORTANTE: esto ultimo es pseudo-codigo, estas 2 operaciones tienen que ser atomicas.
Existe un tipo especial de semaforo que tiene valores binarios y se llama mutex (mutual exclusion).

#### Overhead
Al hacer un wait el proceso es pasado a la cola de procesos bloqueados, y esto necesariamente pasa a nivel kernel. Para que pase esto se tiene que hacer mediante una syscall, e implica un *cambio de contexto* lo cual lo hace caro.

Los semaforos entonces pasan a ser eficientes cuando la seccion critica que tienen que cubrir es lo suficientemente pesada en cuanto a codigo para que el proceso se quede un rato en ella. Si esto no pasa, esp referible usar otra alternativa como un mutex que es mucho mas rapido y sin *context switch*
### Deadlock
El sistema se traba porque un proceso A esta esperando a B y a su vez B esta esperando a A, entonces ninguno esta liberado y se traba todo.


### Herramientas de lenguajes de alto nivel para concurrencia
Los lenguajes de alto nivel proveen alternativas para implementar seciones criticas:
- bool atomicos
- ints atomicos
- colas atomicas


### TAS con objetos
Existe un objeto / registro atomico basico, que provee el testAndTest anterior y una funcion mas general, llamada getAndSet().

### Spin lock
Se puede implementar un lock basado en TAS, denominado comunmente *spin lock*.
Este tiene la siguiente estructura:
atomic<bool> reg;
```
void create() { reg.set(false); }
void lock() { while (reg.testAndSet()) {} } <----> no es atomico esto
void unlock()  { reg.set(false); }
```
### Local spinning TTASLock
Es una mejora del Spin lock, que para lockear en lugar de estar haciendo todo el tiempo testAndSet que es caro, primero se queda haciendo get del valor del mutex para chequear cuando se hace false, porque hasta ese momento no tiene sentido intentar nada.

### Registros Read-Modify-Write atomicos
Son registros de tipo entero que permite hacen operaciones como getAndInc, getAndAdd y luego se tiene una operacion llamada compareAndSwap
Esta funcion es importante, y lo que hace es tomar 2 parametros, u y v, y lo que hace es fijarse si el valor del registro es igual a u, y en caso de serlo le pone el valor v al registro. Esto se puede pensar como que yo quiero actualizar el valor solo en el caso de que el valor que estoy viendo es su valor actual, sino no.

### Deadlock: condiciones de Coffman
Estas condiciones estan mal, pero no tan mal. Son condiciones necesarias para un deadlock:
- Exclusion mutua: un recurso no puede estar asignado a mas de un proceso
- Hold and wait: los procesos que ya tienen algun recurso pueden solicitar otro
- No preemption: no hay mecanismo compulsivo para quitarle los recursos a un proceso
- Espera circular: tiene que haber un ciclo de N >= 2 procesos.


## Problemas de sincronizacion
Problemas 
- Race condition
- Deadlock
- Starvation


# 6 - Sincronizacion entre procesos P2
Problemas en sistemas paralelos


## Problema de turnos
Tenemos una serie de procesos 0… N-1, y queremos que vayan ejecutando una accion secuencialmente
Para solucionar esto se pueden usar semáforos:
Poner foto de código solucion

NOTA: no sirve la herramienta para probar correctitud de un programa que conocemos en el mundo paralelo

Hay nuevas definiciones para esto

## Correctitud en mundo paralelo
- Safety: no pasan cosas malas como deadlock
Los contraejemplos de esto son finitos, es mostrar una corrida en una cant finita de pasos hasta llegar a un deadlock x ej
- Liveness: en algun momento algo bueno hace el programa 
Los contraejemplos no son finitos, porque puedo dar un contraejemplo de que en X pasos algo bueno no paso, pero en realidad puede ser que pase en el pasoX+1, entonces no se puede en finitos.
- Fairness: Los procesos reciben su turno con infinita turno, osea, reciben su turno y lo recibiran luego mas adelante seguro. La idea de esta propiedad es que el scheduler no es un wacho y siempre le da regularmente a los procesos un quantum.

IMPORTANTE: para saber si una propiedad es de safety o liveness esta bueno analizar como seria el contraejemplo, si finito o infinito, y con eso ya puedo decir cual es

## Demostrando el problema de turnos
Primero vamos a querer garantizar una propiedad que llamaremos TURNOS:
los procesos pi ejecutan en orden: p0, p1, ...., p(n-1)
esta es una propiedad de *safety*, ya que su contraejemplo es finito. Puedo dar una secuencia de pasos tal que ejecutan en desorden los procesos.

sale demostrando por absurdo esta propiedad por el absurdo:
supones que el p(i+1) vino antes que el pi, pero para que eso pase el pi tuvo que hacerle signal a p(i+1) antes, absurdo.


## Problema de barrera de sincronizacion (Rendezvous)
Cada proceso Pi tiene que ejecutar el codigo a(i);b(i), satisfaciendo la propiedad BARRERA, que es:
b(j) se ejecuta despues de todos los a(i)
Esto significa que primero tienen que ejecutarse todos los a(i), y recien ahi los b(i). No hay que imponer ningun orden sobre como se eejcutan por adentro los a(i) o b(i)!

## Modelo de proceso
Sirve para razonar sobre un programa paralelo. 

Estados
- CRIT: la seccion critica
- EXIT: cuando sale de CRIT
- TRY: antes de entrar a CRIT, intentando entrar, mediante busy waiting, wait, etc.
- REM: hace cosas que no interesan, y eventualmente va a TRY

## Propiedades utilizando este modelo de proceso
### Propiedad de WAIT-FREEDOM
Es parecido a liveness, nos dice que estamos libre de procesos que esperan (para siempre), y dice:
todo proceso cuando intenta acceder a la seccion critica, en algun momento tiene exito
formalmenee se puede pensar que si: 
un proceso esta en un paso en el estado TRY, entonces en un paso posterior va a estar en el estado CRIT

puede ser muy dificil demostrar esto, entonces es mejor quedarse con algo mas debil

### Propiedad de EXCL (Exclusion mutua)
Para toda ejecucion pi y estado pí_k, no puede haber mas de un proceso i tal que pi_k(i) = CRIT.
Esto se puede pensar asi formalmente:
EXCL = en todos los estados de ejecucion, la cantidad de procesos que estan en CRIT es <= 1.

### Propiedad de LOCK-FREEDOM (Progreso del sistema)
Siempre sucede que si hay al menos un proceso en TRY y ninguno en CRIT, entonces en algun momento la seccion critica se va a ocupar, osea, va a haber al menos un proceso en CRIT.
Esto es mas debil que WAIT-FREEDOM, porque dice que si hay un proceso intentando, entonces eventualmente alguno entra despues, pero no necesariamente es el mismo.
En WAIT-FREEDOM es el mismo que esta intentando entrar que logra entrar.
Esta propiedad nos esta diciendo que si la seccion critica esta vacia y si los procesos estan intentando entrar, eventualmente la seccion critica se va a ocupar. Debid a esto es progreso del SISTEMA, porque habla de todos los procesos a la vez.

### Propiedad de DEADLOCK/LOCKOUT/STARVATION-FREEDOM (Progreso global dependiente)
Si todo proceso se porta bien, osea que si esta en la seccion critica en algun momento sale, entonces vale que todo proceso la va a pasar bien, osea que si esta intentando entrar a la seccion critica, entonces eventualmente va a entrar.
Esta propiedad NO GARANTIZA que los procesos salen de la seccion critica, porque es un implica, los procesos pueden colgarse y cagaste.

### Propiedad de WAIT-FREEDOM (Progreso global absoluto)
Todo proceso que intenta entrar a la seccion critica lo logra. 
Es la condicion mas fuerte.

### Dificultad en probar alguna propiedad
Puede ser que no pueda garantizar alguna propiedad de estas en mi programa por la propia naturaleza de mi algoritmo, que puede hacer que se de que falle la propiedad.

## Bibliografia
The Art of Multiprocessor Programming
Distributed Algorithms, N. Lynch.

# 7 - Sincronizacion entre procesos P3

## Livelock
Pasa cuando los procesos se estimulan mutuamente y hacen que el sistema este cambiando continuamente sin llegar llegar a un estado estable.
Ejemplo: noqueda poco espacio en disco. Proceso A detecta la situacion y notifica a proceso B (bitacora del sistema). B registra el evento en disco, disminuyendo el espacio libre, lo que hace que A detecte la situacion y...

## Problema de seccion critica de a M
La idea es que en lugar de haber a lo sumo 1 solo proceso en la seccion critica a la vez, ahora pueden haber M.
La solucion para esto es sencilla, porque basta con usar un semaforo inicializado en el valor entero M, en lugar de 1, y hacer wait y signal sobre este.
sem = semaphore(M);
P(i) {
  sem.wait()

  s(i)

  sem.signal()
}

IMPORTANTE: si por ejemplo quisiera probar que en este caso no hay starvation, tendria que decir que estamos considerando un scheduler con FAIRNESS, porque sino podria pasar que:
- hay 1 lugar disponible y 2 procesos intentando entrar
- uno entra y el otro espera su turno
- llega otro proceso mas para esperar entrar
- se libera un lugar y entra el proceso que llego ultimo, osea que sigue esperando el original, y asi al infinito

esta situacion produciria STARVATION, pero como el scheduler tiene FAIRNESS esta todo bien.

## Problama de lectores / escritores
Se da en base de datos
Hay una variable compartida
Los escritores necesitan acceso exclusivo
Los lectores pueden leer simultaneamente
Propiedad SWMR (Single-Writer/Multiple-Readers)

Hay una posible solucion a este problema en las diapos, pero genera inanicion, lo que viola la propiedad STARVATION-FREEDOM (progreso global dependiente)
Lo que pasa es que mas alla de que el scheduler sea FAIRNESS, puede pasar que no paren de aparecer lectores y se produzca inanicion de escritores, porque nunca se libera el semaforo para ellos.

## Problema de filosofos que cenan
Se tienen 5 filosofos en una mesa circulas, donde cada uno tiene 1 plato y 1 tenedor por plato
Necesitan comer con 2 tenedores, y el desafio es lograr estas propiedades:
- EXCL-FORK: Los tenedores son de uso exclusivo
- WAIT-FREEDOM: No haya deadlock
- STARVATION-FREEDOM: No haya inanicion
- EAT: Mas de un filosofo comiendo a la vez

PROPIEDAD INTERESANTE: No existe ninguna solucion en la que todos los filosofos hagan lo mismo (no existe solucion simetrica)

## Se puede lograr EXCL (Exclusion Mutua) sin TAS (TestAndSet) ?

### Registros RW
Caracteristicas:
- Si read() y write() NO se solapan
  - read() devuelve el ultimo valor escrito
- Si read() y write() se solapan hay varias opciones
  - "Safe": read() devuelve cualquier valor, como si se hubiera roto todo
  - Regular: read() devuelve algun valor escrito en la lista de valores que se escribieron historicamente sobre el registro
  - Atomic: read() devuelve un valor consistente con una secuenciacion de las acciones de read y write.


## Problema del consenso
Hay N procesos que eligen un valor inicial booleano cada uno.
La idea es que todos los procesos decidan un valor booleano final, y todos coincidan en su decision.
Esto tiene que hacerse en un nro finito de transiciones (WAIT-FREEDOM)

### Teorema
No se puede garantizar consenso para un n arbitrario (pero si para alguno particular) con registros RW atomicos.

### Jerarquia de mecanismos de sincronizacion
Se habla de la cantidad de procesos para el cual un mecanismo puede resolver el problema del consenso.
Este numero se lo denomina numero de consenso

### Numero de consenso
Una lista de los numeros de consenso para distintos mecanismos:
- Registros RW atomicos: 1
- Colas, pilas: 2
- (TAS) getAndSet(): 2
- (CAS) compareAndSwap(): infinito

## Resumen
Buscar de problema de los filosofos
Propiedades adicionales vistas

# 8 - Administracion de memoria
Fundamental para la multiprogramacion
Las responsabilidades del modulo de memoria son:
- manejar espacio libre/ocupado
- asignar y liberar memoria
- controlar el swapping


## Mecanismos para poner memoria del proceso a ejecutar
La pregunta es que hacer con la memoria cuando se cambia de proceso a ejecutar. Hay que poner en memoria la del proceso nuevo y guardar la del viejo.
Hay varios mecanismos:
- Swapping: pasar datos de la memoria al disco y viceversa
  - El problema de esto es que es lento.
- Dejarlo en memoria hasta que no tenga mas espacio. Luego lo bajo a disco.
  - El problema es que cuando vuelvo a traer la memoria, no se donde ponerla para que no se rompa todo, porque tiene que estar en la posicion de memoria correcta.
- Tener un registro que haga de base, y todas las direcciones del programa son relativas a este
  - El problema es que un proceso puede terminar leyendo la memoria del otro, si uno tenia base 100 y el otro 200, el de 100 puede eventualmente leer la otra.

Hay 3 problemas principales para esto:
- Proteccion: Como asegurarnos que un proceso no lea los datos del otro? (memoria privada de los procesos)
- Manejo del espacio libre: Como sabemos que pedazos de memoria tenemos libres? (evitando la fragmentacion)
- Reubicacion: En donde conviene ubicar un programa? (cambio de contexto, swapping)

### Fragmentacion / Manejo del espacio libre
Tengo suficiente memoria para atender un pedido, pero no es toda continua (necesito que sea continua).

Una solucion seria tomar todos los datos de memoria y *compactarlos*, pero esto es muy costoso en tiempo, entonces se evita la fragmentacion directamente.

#### Organizacion de la memoria para evitar la fragmentacion
Para evitar la fragmentacion, se toman varias medidas:
- Se hace que el stack crezca en una direccion y el heap en el otro, para que lo que uno no ocupa lo ocupa el otro.
- Divido en bloques a la memoria a traves de:
  - Bitmap
    - Cada bloque tiene x ej 4kb.
    - Se le pone 1 en el bitmap si el bloque esta ocupado, y 0 si esta libre.
    - Hay un problema ya que si bien para asignar y liberar es O(1), pero encontrar bloques consecutivos requiere una barrida lineal, osea O(N)
    - Otro problema es que si la pagina es muy grande, al achicarlo el bitmap se hace gigante, entonces hay un problema entre granularidad y tamaño de bitmap.
  - Lista enlazada
    - Cada nodo representa:
      - Un proceso con la memoria que tiene asignada
      - Un bloque libre que esta en un lugar y tiene X tamaño
    - Liberar memoria es facil, en O(1) se saca el nodo y se unen las 2 puntas, chequeando si hay que juntar ambos extremos si eran espacios libres o si hay que extender alguno si era libre con la memoria que se acaba de liberar
    - Al momento de tener que liberar memoria puedo tener un diccionario que me mapea la direccion de memoria al nodo correspondiente en la lista.
    - Al momento de asignar memoria puedo tener una estructura de datos donde puedo buscar todos los bloques libres de memoria para encontrar alguno que sirva.

##### Que bloque asignar para cierto pedido de tamaño?
Tengo que decidir que bloque asignar para cierto pedido, las cuales pueden ser:
- First fit: En el primer bloque que encuentro que entra.
  - Es rapido
  - Tiende a fragmentar la memoria, partiendo bloques grandes
- Best fit: Me fijo donde entra mas justo
- Quick fit: mantengo una lista de bloques libres de los tamaños mas frecuentemente solicidatos

Estos esquemas fallan porque son muy ingenuos, y producen alguna de estos tipos de fragmentacion:
- Fragmentacion externa: bloques libres pero pequeños y dispersos
- Fragmentacion interna: espacio desperdiciado dentro de los propios bloques

### Reubicacion / Memoria virtual

Si tengo N bytes de memoria y un programa con tamaño M > N, pero que no necesita mas de K < N bytes a la vez, me gustaria correrlo. Como hago?

Para solucionar esto se combina swapping con virtualizacion del espacio de direcciones, y esto se denomina *memoria virtual*.

Se utiliza una pieza de HW llamada MMU. 

#### MMU
La memoria virtual esta dividida en bloques de tamaño fijo que se llaman *paginas*

La memoria fisica esta dividida en bloques del mismo tamaño llamados *page frames*

Esta se encarga de mapear la direccion virtual o logica que le da la CPU a la direccion fisica o real en memoria.

Cuenta con una tabla donde cada entrada dice para esta direccion virtual le corresponde esta real, y tambien dice si esta cargada o no en memoria ese pedazo.

Esto ultimo nos dice que un programa trabaja con tamaño virtual de memoria, que es la suma del tamaño de memoria fisica mas la del disco que configure como memoria swap.

La MMU traduce paginas a frames, interpretando a las direcciones como pagina + offset, donde los n bits mas significativos son la pagina y el resto el offset dentro de esta. El problema de esto es que el tamaño de la tabla puede ser gigante, y lento para consultar. La solucion a esto es dividir la busqueda en 2 niveles, es decir, se parten los n bits en 2 partes, y cada uno es una tabla distinta. Voy con los primeros bits a consultar la tabla principal, y esta me dice a que tabla consultar con los demas bits de la pagina (la joda es que la segunda tabla es del tamaño de una pagina y puede swapearse facil).

![alt text](8/mmu_entrada_tabla.png)

IMPORTANTE: repasar funcionamiento de MMU de orga2

IMPORTANTE: en el caso de que haya que usar un pedazo de memoria que no esta cargado hay que cargarlo del disco, en cuyo caso el proceso se bloquea hasta que el disco responda con los datos. Notar que esto es un ejemplo mas de cuando un proceso puede bloquearse (usar E/S, hacer wait con semaforos, acceder a memoria no cargada)

##### Page faults
Cuando un proceso intenta acceder a una pagina que no tiene, se produce un page fault.
El kernel va a intervenir, y se va a fijar si esa direccion de memoria que intento acceder es autorizada para el, y en caso de no serlo se produce una *segmentation violation*, y el kernel mata al proceso.

##### TLB: Solucion a lentitud de MMU
Acceder a memoria es un problema porque estoy haciendo un acceso a memoria mas por cada acceso que quiero hacer, ya que el MMU hace 1 acceso a la tabla para encontrar la pagina correcta.

La solucion para esto es agregar HW, un cache, llamado TLB (Translation Lookaside Buffer), el cual es pequeño pero tiene las siguientes caracteristicas:
- Implementado con registros muy rapidos
- Se cachean las tablas de paginas. Cuando tengo que mapear dentro del mismo proceso este no pasa por memoria ya que tiene todos los datos ahi.


#### Problema con swapear paginas

Hay un detalle al momento de traer un pedazo de memoria del disco, en el caso de que este no entre en los espacios libres actuales de la memoria. En este caso, hay que bajar a disco a alguna porcion de memoria, y la pregunta es a quien.

Existen multiples algoritmos para esto:
- FIFO
- FIFO con segunda oportunidad
- NRU (Not Recently Used)
- LRU Least Recently Used) <--- El mas usado


#### Trashing
Un fenomeno que ocurre cuando no alcanza la memoria y hay muchos procesos utilizandola. 
El SO esta mas tiempo swapeando paginas en lugar de ejecutando.
Requiere poner mas memoria para que no haya que paginar tanto.


### Proteccion
Cada proceso tiene su propia tabla de paginas, entonces no hay forma que un proceso acceda la pagina del otro.
Esto es asi exceptuando el caso de paginas compartidas entre procesos. Las razones para tener paginas compartidas son 3:
- Memoria compartida
- Bibliotecas
- Paginas de kernel



## fork() y la memoria
Cuando se hace fork() y se crea un proceso nuevo, no se copian todas las paginas a memoria del proceso original que hizo fork, sino que apunta a las originales.
Lo que se hace es una estrategia llamada copy-on-write, para lecturas se lee la misma memoria del proceso padre, pero en el momento que ocurre un write, ahi si se duplica la pagina para el hijo.

## Resumen
- Como compartir memoria
- Swapping, paginacion, segmentacion
- Como manejar espacio libre
- Como decidir que paginas sacar de memoria
- MMU
- Page faults
- Thrashing
- Copy-on-write

# 9 - Administracion de E/S
Foco puntual en los dispositivos de almacenamiento de E/S.

Hay varios de estos que el SO tiene que preocuparse:
- Discos rigidos
- Unidades de cinta
- Discos removibles

Sumado a estos, esta el disco virtual, que no esta en la misma pc, sino en otro punto de la red. Se los denomina NAS (Network Attached Storage)

Partes de un dispositivo E/S
- El dispositivo fisico
- El controlador del dispositivo: interactua el SO con el, mediante algun buffer, registro para decirle donde escribir/leer.


Diagrama de como se ve la comunicacion con los dispositivos con los procesos:

![alt text](9/diagrama_entrada_salida.png)

En este vemos a los procesos corriendo, que hablan con la API del SO mediante syscalls
El kernel implementa estas syscall con alguno de los subsistemas de E/S.
El subsistema de E/S pasa por el driver del tipo de dispositivo apropiado, el cual es software especifico para el SO y dispositivo.
Luego el driver habla finalmente con el controlador del dispositivo, que es propio del dispositivo ya.

IMPORTANTE: un driver corre con maximo privilegio, a nivel kernel, por lo que si se cuelga, el sistema entero se cuelga, porque es el kernel.

## Interaccion con los dispositivos
- polling
  - el driver verifica periodicamente si el dispositivo le hablo
  - pros: es sencillo y da mas control
  - cons: consume CPU
- interrupciones
  - el dispositivo avisa mediante una interrupcion una novedad (como se toco una tecla, termino algo)
  - pros: eventos asincronicos que pasan poco, resuelve el consumo de CPU
  - cons: los cambios de contexto son impredecibles
- DMA (Direct Memory Access)
  - Sirve para transferir choclos de datos, sin taponar al CPU asi puede seguir con su vida.
  - Se necesita un componente de HW
  - Recien cuando el DMA termina le avisa al CPU mediante una interrupcion.


No necesariamente interrupciones es mejor que polling. Por ejemplo, para una placa de red no esta bueno interrumpir todo el tiempo porque hay cambios de contexto todo el dia, es mejor hacer un polling mas inteligente.


## Subsistema de E/S
Provee al programador una API sencilla con las ops:
- open() / close()
- read() / write()
- seek()

Con esta API se oculta toda la complejidad por atras, y el SO se encarga de hacerlo de manera correcta y eficiente.

Los dispositivos se dividen en 2 grupos, dependiendo de cuanto informacion se transfiere por vez:
- Char device: se transmite byte a byte, como en un mouse, teclado, etc.
- Block device: se transmite en bloque la info, como un disco rigido, flash memory, etc.


## Disco
Tiene relacion con el swapping con el disco para el sistema de memoria. El swapping anda mal si el disco no anda rapido.

El disco tiene como desafio minimizar los movimientos de la cabeza que tiene.

La estructura del disco es la siguiente:

![alt text](9/estructura_disco.png)

Tiene:
- Cabeza que entra y sale.
- Platos
- Cada plato tiene anillos, como las capas de una cebolla
- Cada anillo se divide en sectores, que son los que despues lee la cabeza

## Planificacion de disco
Para leer el disco, es necesario esperar el tiempo suficiente para que el plato gire, mover en profundiad la cabeza para que quede en el sector apropiada y luego hacer escritura o lectura.

Esto ultimo hace necesario llevar a cabo la planificacion de disco, que trata de como manejar la cola de pedidos de E/S eficientemente para lograr el mejor rendimiento posible. La planificacion no es responsabilidad del controlador de disco, porque no sabe los pedidos que voy a hacerle.

La *planficiacion* la tiene que hacer el SO para reordenar los pedidos y hacer lo mas eficiente posible comunicacion con el disco.

Las opciones pueden ser:
- Reordenar los pedidos para atender los pedidos de sectores en orden.
- Atender el mas cercano a donde esta la cabeza en el momento (SSTF (Shortest Seek Time First)). Mejora tiempos de respuesta pero puede generar inanicion.
- Algoritmo de ascensor: voy atendiendo los primeros pedidos que encuentro en el camino mientras voy bajando / subiendo en los platos.


## Problema de discos SSD
Se pueden reeescribir una cantidad acotada de veces. Cada celda se va desgastando, y el SSD tiene cierta inteligencia para no estar escribiendo la misma celda todo el tiempo

## Gestion de disco

### Formateo
Se agrega informacion redundante para que la controladora de disco pueda hacer deteccion y correccion de errores.
Con esto, al momento de leer un dato por ejemplo, se chequean los codigos que se agregaron en el formateo para corroborar si hubo un error de lectura o no.

### Booteo
Hay un programa en la computadora que se encarga de leer la parte inicial del disco, la carga en memoria, y la ejecuta.
Luego, este programa cargado es el que se encarga de cargar el SO, se podria decir que este es el cargador del SO.

### Bloques dañados
Se suele manejar por software. Algunos discos lo hacian solos, se daban cuenta que estaban defectuosos y usaban otra parte del disco

## Spooling
Se suele usar en la impresion, al momento de imprimir varios archivos, permitiendo asi que multiples programas puedan usar la impresora a la vez, ya que hablan con el *spooler*.
Cuando se mandan a imprimir varios documentos, se le envia al *spooler*. Este se encarga de encolar todos los pedidos, y libera al software que lo llamo diciendole que ya termino.
Con esto se evita bloquear al software hasta que la impresora que es algo lento termina en imprimir.
El *spooler* se encarga de mandarle al driver de impresora como haga falta los docs a imprimir, mediante por ejemplo DMA, polling, interrupcion, etc.

IMPORTANTE: el *spooler* no es kernel, es codigo de usuario.

## Uso de E/S: locking
El standard POSIX permite abrir un file con flags O_CREAT | O_EXCL, lo cual es una forma atomica de abrir un file.
Es la exclusion mutua para pobres, para tener entre varios procesos un "Mutex" en forma de file.

## Proteccion de informacion
Quiero hacer un esfuerzo para protegerla equivalente al valor que le doy, utilizando una politica de resguardo.

Hay distintas politicas.

### Copias de seguridad

Hacer un backup, que consiste en copiar la informacion en otro lugar.

#### En donde hago el backup
Se pueden copiar en distintos lugares, como un disco SSD si es algo personal, pero si es empresarial por ejemplo en una cinta.
Hay robotitos que se encargan de hacer automaticamente los backups en cintas, tomando una cinta, poniendola para que se haga el backup, y la saca y pone otra.

#### Donde dejo el backup
Guardo las cintas en un lugar que no es inflamable y no le afecta el calor, porque el calor funde las cintas.

#### Estrategias de backup
Se puede armar una rotacion de cintas, donde se usa una en cada dia de la semana x ejemplo, y luego se vuelve a empezar

El problema de esto igual es que copiar todos los datos puede ser muy costoso, entonces hay otras formas:
- Total: La que se menciono antes clasica
- Incremental
  - Cada dia guardo los cambios con respecto a la ultima copia incremental
- Diferencial 
  - El lunes mando todos los datos a la cinta
  - El martes me fijo los archivos que cambiaron con respecto al lunes, y pongo esos nomas
  - El miercoles lo mismo que el martes.
  - En caso de backup, necesito la cinta del lunes y luego la del dia en cuestion para recuperar el delta.
  - Lo malo de esto es que es medio lento.

#### Redundancia
Una copia de seguridad puede no alcanzar, porque restaurarla puede tomar mucho tiempo, que es super grave si yo necesito hacerlo en tiempo real.

La idea para arreglar esto es utilizar mecanismos de redundancia, llamados RAID (Redundant Array of Inexpensive Disks).
Para el usuario el disco parece uno, pero en realidad es un disco logico, no fisico, que esta organizado con varios discos fisicos atras con cierta logica.
Las posibles formas son:
- RAID 0 (stripping):
  - Tengo varios discos, y voy distribuyendo los bloques de un mismo archivo en cada uno.
    - Pros: Mejoro la performance, porque puedo escribir en paralelo.
    - Cons: No aporta redundancia.
- RAID 1 (mirroring): Utilizar 2 discos, donde en cada escritura se hace en los 2. 
  - Pros: Si uno se rompe, tengo el otro. Lo bueno es que en lecturas puedo paralelizar las lecturas, yendo a ambos discos.
  - Cons: Es costoso, tengo que tener el doble de discos para la informacion que guardo.
- RAID 0+1: no le intereso a schapa
- RAID 2: no le intereso a schapa
- RAID 3: no le intereso a schapa
- RAID 4: no le intereso a schapa
- RAID 5: Poniendo un ejemplo con 4 discos, tenemos a cada disco partido en 4 partes.
  - Cada fila tiene 3 partes que se escriben en cada disco, y la 4ta se utiliza para paridad entre los otros 3.
  - Si se rompe en una fila la parte de paridad, no hay problema porque la info esta en las demas partes de los otros 3 discos.
  - En cambio, si se rompe de la fila la parte de un disco con info importante, puedo usar las otras 2 partes junto con la paridad del 4to para reconstruir la info.
    - En este caso entra en modo degradado el disco, y se tiene que reconstruir, asi que va a tardar mas, y hace esto hasta reconstruir toda la info y salir de modo degradado.
  - La paridad se va distribuyendo en cada una de las filas alternando entre cada disco, para cargar tanto al mismo disco de escrituras para actualizar las paridades.
  - Protege de la rotura de un disco completo.
  - El costo aca es por 3 discos de datos pagar 1 mas, no es el doble como en *mirroring*.
  - IMPORTANTE: La controladora del RAID recibe data a escribir, y la guarda en un cache, y libera al proceso que la esta usando.

IMPORTANTE: El RAID NO REEMPLAZA al backup, solo me salva de problemas estando online. Si hay un error de usuario, si borro info peligrosa de eso me salva el backup.

IMPORTANTE: Que pasa si se corta la luz con las escrituras que se tenian que hacer en disco y se corta la luz? Las controladoras tienen una bateria interna para encargarse de completar la operacion hasta cuando se corta la luz.
La cagada es que si la bateria esta dañada esto falla, entonces hay que estar revisando la bateria siempre. Esto hace que todo sea un infierno.

# 10 - Sistemas de archivos

## Que es un archivo?
Una secuencia de bytes sin estructura.
Se lo identifica con un nombre.
A veces tiene una extension para distinguir el contenido del mismo

## Modulo que maneja archivos
El modulo en el kernel que maneja los archivos se llama file system.
Los SO hoy en dia vienen con file system que soportan cualquier sistema de archivo.
Ejemplos de file systems: FAT, UFS, FFS, ext2, ext3, ext4, XFS, RaiserFS, ZFS, ISO-9660.
Los file system originales son FAT y UFS, los demas son variantes de estos.
Tambien existen file systems distribuidos, donde los datos estan distribuidos en varias maquinas en la red.

## Punto de montaje
Sirve para montar un disco en un path del arbol de directorios, y con esto se lo puede utilizar para guardar datos ahi.
Esto permite cambiar un disco por otro o montar un disco en un directorio que esta muy lleno para aumentar su capacidad.

## Responsabilidades del FS
Su principal responsabilidad es ver como organizar desde el punto de vista logico los archivos. Se divide en:
- Organizacion interna: es como se organiza el archivo internamente, que por lo general es una secuencia de bytes, sin nada especial (pueden haber metadatos).
- Organizacion externa: es como se ordenan los archivos

### Arbol de directorios: 
Tenemos organizado todo en directorios (carpetas) y archivos. En un directorio se pueden poner archivos o referencias a archivos que estan en otro lugar.

### Nombre de archivo
El FS determina como se nombran los archivos, donde chequea entre algunas cosas:
- caracteres de separacion de directorio
- si tienen o no extension
- restricciones a la longitud y caracteres permitidos
- distincion o no entre mayusculas y minusculas

El FS determina cuanto del nombre de un archivo es una cuestion portable o de donde se ubico el disco al cual se esta refiriendo.

### Como se guarda un archivo?
Tengo dividido todo en bloques como la memoria, y tengo que saber cosas muy parecidas al manejo de memoria:
- En que bloques queda guardado un archivo y como me entero de eso
- Como se que bloques estan libres

Metadatos: son los datos que no tienen q ver con el contenido del archivo, como permisos, atributos, etc

Los objetivos de esto es guardar los datos sin errores y rapido.

#### Representando archivos

##### Naive
La forma mas sencilla es poner un archivo en un contiguo de bloques.
- Lecturas super rapidas
- Si el archivo crece y no tengo espacio adelante cague
- Hay fragmentacion

Este approach sirve para un FS *readonly* unicamente, ya que cuando lo escribi no puedo cambiarlo y anda super rapido leerlo.

##### Linked-List
Para un FS de lectura-escritura se podria usar una lista enlazada de bloques, solucionando los 2 problemas anteriores.
Mientras mas grande el disco, mas ocupan los punteros al siguiente bloque.
Tengo que pedirle a la controladora el primer bloque, esperar a la controladora, luego ver el siguiente bloque y pedirselo devuelta a la controladora, etc.. ESTO ES MUY LENTO, ROMPE TODA LA IDEA DE LO QUE VIMOS ANTES

##### Linked-List V2 (FAT)
Generar una tabla que se guarda en un bloque y me condensa todos los datos de la lista enlazada, donde me dice por cada bloque cual es el siguiente en la lista.
Con esto puedo pedirle multiples bloques a la controladora a la vez (algoritmo del ascensor, reordering, lectura-prefetching)

Esto es lo que hace FAT, carga inicialmente la tabla esta en memoria y trabaja sobre ella. 

El problema que tiene es que hay que estar guardando cada tanto la tabla en el disco, porque sino puede cortarse la electricidad y termino con todos los bloques con los datos en memoria bien, pero la tabla incompleta que no me dice como construir esos bloques en la lista enlazada.

Otro problema que tiene es que ocupa demasiada memoria la tabla esta a medida que el disco es mas grande.

Ademas hay otro problema, que es la contencion ilegitima con una unica tabla. Si tengo varios procesos escribiendo a la vez con una sola tabla tengo que lockear la tabla, y traba todo eso. FAT no fue pensando para esto porque los CPUs eran de un solo nucleo.

##### Inodos
Cada archivo tiene un inodo.
El inodo tiene una estructura que ocupa un bloque de disco que tiene:
- Metadata: size, block count, etc
- Direct blocks: es una lista de tamaño fijo con los punteros a los bloques que conforman al archivo (puedo tener un tamaño maximo con esto solo de archivo)
- Single indirect: es un puntero a una tabla de Direct blocks
- Double indirect: es un puntero igual que en Single indirect pero la tabla en este caso tiene punteros Single indirect
- Triple indirect: idem a Double indirect pero la tabla tiene punteros a Double indirect

IMPORTANTE: las tablas de single, double y triple indirect y el bloque de datos tambien son inodos pero de otro tipo.

Cuando quiero acceder a un archivo pago el costo de traer el inodo, y luego le mando todos los punteros en Direct blocks al modulo de E/S para que los traiga con todas sus tecnicas reorder, prefetch, etc. Si estos bloques no son suficientes, entonces sigo por Single indirect, pidiendo el bloque con la tabla al que apunta este a la controladora y por cada puntero de esa tabla consigo mas bloques. Esto se puede repetir para double y triple indirect.
Esto asi solo arregla el problema de contencion ilegitima, porque ahora si se tocan archivos distintos se puede, porque cada archivo tiene sus propios datos. Obvio tengo contencion legitima cuando se toca un mismo archivo, pero eso es normal.


###### Booting en Inodos
Los inodos tienen mayor capacidad de recuperacion
Se fija ciertas cosas en el arranque el FS:
- Se fija que el tamaño de inodo sea coherente con la cant de bloques que dice tener
- Cada bloque de datos esta referenciado desde un unico inodo
- Todos los bloques que no figuran como libres estan referenciados desde algun lugar


###### Implementacion de arbol de directorios
Hay inodos de tipo directorio
Hay un inodo directorio que es el *root directory*.
Al buscar el inodo de un path hace lo siguiente:
- Empieza por el inodo de root directory, y se fija en las entradas que tiene en sus bloques donde esta el primer directorio del path junto al puntero a su inodo
- Una vez hecho esto, se fija dentro de este directorio donde esta el siguiente directorio igual que antes
- Esto se repite hasta que llega al ultimo del path y se fija que es un archivo y termina


###### Link
Cuando se crea un link, se tiene un nombre de archivo que apunta a otro archivo
A nivel estructura, en el mismo inodo directorio se tienen 2 archivos que apuntan al mismo inodo, uno es el que se creo originalmente y el otro es el link.

###### Link simbolico
Es un inodo especial que es un link, a diferencia del anterior
Dentro del inodo apunta al archivo que fue linkeado, pero si este se borra va a seguir apuntando ahi
El uso es que sirve para switchear a que file se apunta, por ejemplo switchear entre 2 configs para algo.

##### Manejo de espacio libre

Investigar

##### Cache

Investigar (min 1:40:00 aprox)

##### Consistencia
Cuando un proceso hace un write, puede implicar modificar el inodo (crear mas bloques)
La pregunta es si ir a disco por cada escritura o no:
- Si voy por cada write se degrada la performance
- Si no lo hago como me recupero ante un crash?

El SO tiene un cache para ir guardando los write que se quieren hacer, y espera para escribir en disco. 
Si se desea pasar la cache a disco hay una syscall que se llama fsync para forzar esto.
Termina siendo responsabilidad de la aplicacion hacer esto.

##### Journaling

Reservo un cacho de espacio fijo donde voy grabando todos los cambios que tengo pendientes. Se graba a disco pero es una estructura rapida. Cuando hay algun crash se graba lo de journaling a las partes del disco correctas.

## Caracteristicas avanzadas

- Cuotas de disco
  - Un usuario puede escribir lo que quiera pero hasta ciertos gigas.
  - PREGUNTA IMPORTANTE DE FINAL: como implementarlo
- Encripcion
  - Encriptar el FS
- Snapshot
  - Congelar una parte de un archivo (copia en O(1) que no consume nada)
  - La idea es acordarse que esta snapshoteado y se va haciendo copy on write, teniendo una copia especial como diferencial
- Manejo de RAID por software
- Compresion

## Performance
Hay muchos factores que afectan el rendimiento:
- Tecnologia de disco
- Politica de scheduling de E/S
- Configurar bien tamaños de bloques
- Caches del SO tienen que ser consitentes con los tamaños de bloques
- Caches de controladoras tienen que llevarse bien con todas las otras cosas
- Manejo de locking en el kernel
- FS bueno

## NFS
IMPORTANTE

Es un protocolo de file system distribuido que le permite a u usuario en una computadora acceder a los files de una red.

Es cliente servidor.


## Resumen
- Responsabilidades del FS
- Punto de montaje
- Representacion de archivos
- Manejo del espacio libre
- FAT, inodos
- Atributos
- Directorios
- Cache

# 11 - Seguridad

Hay una diferencia entre Proteccion y Seguridad:
- Proteccion: Mecanismos para asegurarse que ningun usuario / proceso accede a cosas que le pertenecen a otro
- Seguridad: Asegurarse que un usuario es quien dice ser. Evitar destruccion o adulteracion de datos.

La seguridad es todo lo que hay que hacer para preservar:
- Confidencialidad
- Integridad
- Disponibilidad

Seguridad de la informacion != Seguridad informatica

## Sistema de seguridad
Se tienen sujetos (personas o procesos o usuarios) que interactuan sobre objetos (cosas) y sobre ellos ejercen acciones.
Los sujetos y objetos pueden ser lo mismo (procesos).

El objetivo principal de un sistema de seguridad es ver quien puede hacer que cosa sobre que entidad.

### Usuarios

Un usuario es una abstraccion en un SO para pensar en alguien que es dueño de cosas, que puede ejecutar acciones.
Los usuarios se agrupan comunmente en grupos (colleciones) de usuarios, donde al gfrupo se le asigna permisos particulares. Entonces con esto se asigna au nusuario a un grupo y pasa a tener todos los permisos esos.

- Autenticacion: el proceso por el cual verificamos que un usuario es quien dice ser 
- Autorizacion: que permisos le damos al usuario
- Auditoria: registrar las acciones que viene haciendo la persona para que quede un registro

### Criptografia
Se trata de ocultar informacion para que solo la desoculte el que uno quiera
Se dividen en 2 categorias los algoritmos:
- Simetrico: son aquellos que utilizan misma clave para encriptar / desencriptar.
  - Ejemplos: Caesar, DES, Blowfish, AES.
  - Son rapidos de computar.
- Asimetrico: utilizan claves distintas para encriptar / desencriptar
  - Son mas robustos que los simetricos.
  - Son mas costosos de computar.
- Funciones de hash one-way: no sirven para encriptacion porque no se pueden desencriptar.

#### Funciones de hash
Son utiles para almacenar contraseñas. Se suelen combinar con un SALT, para no hashear solo la contraseña, sino la contraseña + SALT
Se suele pedir a una funcion de hash lo siguiente:
- Evitar colisiones
  - Resistencia a la preimagen: dado H deberia ser dificil encontrar otro valor M tal que H = hash(M)
  - Resistencia a la segunda preimagen: dado M1 deberia ser dificil encontrar un M2 tal que hash(M1) = hash(M2)


Ataques de diccionario: tengo un diccionario con palabras, las voy hasheando y me fijo si algun hash sobre estas palabras coincide con la de la clave, es brute force

#### RSA
https://www.signix.com/blog/bid/93731/How-Does-it-Work-Digital-Signature-Technology-for-Dummies
La idea es que se tienen a 2 personas, Bob y Alice.

Queremos que Alice le envie un mensaje a Bob pero que solo Bob pueda leer.

Para esto Bob comparte a todos una public key, y a su vez tiene la private key asociada a esta secreta solo para el.

Cualquiera que le quiere enviar un mensaje a Bob va a utilizar su public key sobre el mensaje (ponerle el candado que le dio Bob).

Luego Bob recibe este mensaje encriptado con la public key, y es el unico que puede desencriptarlo con su private key (la llave del candado).

#### Firma digital con RSA
https://www.signix.com/blog/bid/93731/How-Does-it-Work-Digital-Signature-Technology-for-Dummies

Se utiliza para saber que el mensaje no fue adulterado y vino de la persona que uno quiere.

Bob toma el mensaje, genera un hash del mismo y lo encripta con su private key

Luego envia como signed documento al conjunto {hash encriptado, documento, public key}

Alice recibe esto, y lo que hace es hashear el documento como Bob, y luego comparar esto con el hash de Bob desencriptado con su public key
Si estos 2 coinciden el mensaje no fue adulterado

#### REPLAY-ATTACK
Es un tipo de ataque sobre data enviada por la red.
Una persona no autorizada (el malo) captura el trafico de la persona and manda esos datos al destino original, haciendose pasar por la persona.
Con esto el que lo recibe va a pensar que es la persona original, porque los datos estan bien, pero no es asi.


#### Representando permisos
Los permisos se organizan con el siguiente formato
-rwxrwxrwx
1. El primer rwx representa los permisos del owner del file
2. El segundo rwx representa los permisos del grupo del file
3. El tercer rwx representa los permisos de todos los otros usuarios  

IMPORTANTE REPASARLO

#### DAC vs MAC
DAC (Discrtionary Access Control):
- Se le pone a un archivo permisos donde se indica para cada usuario los permisos sobre el archivo

MAC (Mandatory Access Control):
- Cada usuario tiene un nivel de seguridad
- Todo lo que genera ese usuario queda con una restriccion que impide el acceso para accesos superiores
- Ningun archivo puede ser bajado de nivel

#### Permisos en DAC en UNIX
Los archivos tienen asociado el usuario dueño, el grupo dueño, y una seria de atributos que dicen:
- tipo: si es un file, un directorio, un pipe
- r: si el dueño puede leer
- w: si el dueño puede escribir
- x: si el dueño puede ejecutar

estos atributos estan presentes tanto para el dueño, el grupo y los usuarios.

*setuid* y *setgid* son permisos especiales que permiten a los usuarios correr procesos con permisos mas elevados.
Esto sirve por ejemplo para correr la aplicacion de cambio de contraseña, que necesita modificar un archivo con permisos elevados.
La idea es que al utilizar estas funciones un proceso puede correr con los permisos del dueño del la aplicacion, y no el del que lo puso a correr que puede tener menos permisos.

IMPORTANTE: investigar el bit SETUID (SUID)
#### SUID & SGID
- SUID: Set User ID, es un bit que cuando esta habilitado en un programa al ejecutarlo uno hereda los permisos del file owner del mismo
- SGID: Set Group ID, similar al SUID, pero en lugar de heredar los permisos del file owner, son los del group owner.

Estos 2 bits se ubican en el lugar del bit x de execution en el bloque de permisos de user y group respectivamente

IMPORTANTE: investigar el *sticky bit*
#### Sticky bit
Es un bit utilizado en los directorios, e indica que solo se puede remover un archivo si el usuario es alguno de estos:
- owner del directorio
- owner del file
- super user, osea root

#### sudo (superuser do)
Se puede configurar para decir que usuarios pueden hacer que cosas privilegiadas.
Por ejemplo permite que un usuario cualquiera pueda correr cualquier proceso como root si esta configurado para eso sudo.


### Seguridad en el software  
Hay que pensar en la arquitectura, la implementacion y la ejecucion de una aplicacion para tener seguridad.
Si alguna de estas falla fuimos.


Veamos errores de implementacion

#### Buffer overflow
Es cuando se escribe mas alla de los bounds de un array
Se puede terminar pisando el IP y asi cambiar la direccion de retorno en el stack que se utilizara al salir de la funcion
Esto podria ser controlado por el usuario si es input de usuario lo que termina pisando la direccion de retorno.
Se puede agregar shellcode para hacer cosas malignas

Hay varias soluciones para esto:
- Permisos de paginas: Declarar paginas de memoria como no ejecutables, para que no se pueda ejecutar shellcode
- Address randomization: hacer random la ubicacion de los buffers en memoria en ejecuciones distintas
- Canary: Antes de llamar a una funcion, elcompilador mete un valor aleatorio entre el IP y las variables del stack. Luego antes de salir de la funcion chequea si el vlaor escrito fue modificado, y en caso de serlo se aborta la ejecucion porque se considera que se modifico el stack.

#### Control de parametros
Hay 2 soluciones posibles:
- Darle el minimo privilegio necesario a un programa
  - Si un buffer overflow se da sobre un programa que corre como root cagamos
  - Si en cambio se da sobre uno que corre como usuario el riesgo esta contenido
- Validar los parametros que me pasan, chequeando por ejemplo en el caso del string que tenga el largo correcto y que no tenga caracteres raros.

Lo malo de esto es que hay que estar conscientes como programadores siempre.

*tainted data*: la idea es marcar cosas que vienen por ejemplo del usuario o de un file como manchadas, y todo lo que involucre usar datos manchados tambien termina manchado
eventualmente si se intenta ejecutar algo que involucra datos manchados se aborta.
solo se elimina el estado de manchado cuando se sanetizan los datos manchados, aunque se los sanetize mal

#### SQL injection
Se trata de no sanitizar la entrada que luego se utiliza en la query SQL, permitiendo que se inyecten comandos SQL en el mismo input y modificando asi las queries que se ejecutan finalmente.

#### Malware
Software malicioso, diseñado para llevar a cabo acciones sin el consentimiento explicito del usuario
Ejemplos de donde sale el malware son:adjuntos por mail, phishing, softwares vulnerables, descargas no chequeadas.
Es importante actualizar el software, restringir nivel de permiso.

#### Sandboxes
Tiene que ver con virtualizacion, corro una VM para que en el caso de que hackeen la VM, no pueden escapar de ahi y el resto del sistema esta a salvo.

#### Otros ataques
- DOS (Denial of Service): dejar sin servicio a alguien, se suele combinar con botnets.

# 12 - Sistemas Distribuidos
Definicion:
- Varias maquinas conectadas entre si en una red
- Tambien se denomina sistema distribuido a un procesador con varias memorias
- O varios procesadores y varias memorias

Lo importante de esto es que los sistemas distribuidos se caracterizan por 2 aspectos primordiales:
- Los procesadores no comparten clock, entonces no existe la idea de ponerse de acuerdo para hacer algo a cierta hora
- Ningun procesador tiene toda la informacion, sino que cada procesador tiene info parcial

## Sistemas distribuidos de memoria compartida
Son los casos de UMA y NUMA

## Sistemas distribuidos SIN memoria compartida
La pregunta es como hacer para colaborar cuando no hay memoria compartida
Para ver de que manera se lidia con esto hay que diseñar una arquitectura de software:
- Una manera en la que disponemos ciertas piezas de softwware y las conexiones entre si para que colaboren y lograr un proposito

Hay distintos tipos de arquitectura

### Telnet
Conectarse a otra computadora como en SSH.

### RPC (Remote Procedure Calls)
La idea es correr una funcion de manera remota. Por ejemplo, correr un computo matematico se hace llamando a una funcion remota en un server mas polenta para hacer el calculo.
La idea es que al usar las funciones remotas en el codigo, esto se compila y reemplaza las llamadas a funcion a un modulo. Con esto, en el momento que se invoca a la funcion en ejecucion, se invoca a una funcion falsa que envia los parametros por la red a otra computadora que esta escuchando por un socket, esta recibe los parametros y hace el computo y lo devuelve.
Ya esta viejo hoy en dia, pero las mas nuevas se inspiraron en esta.

### RPC asincronico
Es como RPC, pero permite especificar una funcion de callback para llamar con el resultado cuando este vuelva del server, asi no se queda el proceso trabado con la llamada remota.

### Cliente - Servidor
Es una arquitectura donde un programa le hace un pedido a otro (cliente a servidor). El servidor deberia estar constantemenete esperando pedidos y manejar multiples a la vez. Es importante que no siempre los programas cumplen el mismo rol, pueden intercambiarse.

## Pasaje de mensajes (MPI)
El proceso que quiere recibir abre un socket y se queda bloqueando esperando sobre un puerto que le llegue el mensaje.

El proceso que envia cuando hace un send abre un socket y le pide que lo conecte contra cierto IP de otra computadora y su port.

- Serializar: Tomar los datos a enviar y transformarlo en una tira de bytes
- Hidratar/Deserializar: Tomar los datos recibidos y reconstruir el mensaje

Problemas posibles que pueden surgir
- Los mensajes pueden perderse a menos que se use TCP que es confiable.
- Tambien los nodos (compus) se pueden morir o revivir (con una foto vieja de lo que venia pasando)
- Los nodos pueden quedar en grupos incomunicados y siguen por su lado. Puedo tener todos los nodos divididos en 2 grupos separados y cada uno piensa que el otro grupo murio y siguen por su lado, haciendo que tengan informacion distinta los 2 grupos.

Pasa a ser relevante cuantos intercambios de mensajes hago
Se hacen modelos de costo donde la complejidad se mide en #mensajes enviados

IMPORTANTE
## Conjetura de BREWER (CAP Theorem)
Dice que cualquier base de datos distribuida puede proveer 2 de las 3 siguientes garantias:
- Consistencia: Cada lector recibe la escritura mas reciente o un error
- Disponibilidad: Cada pedido una respuesta que no es un error, sin la garantia de que contenga la escritura mas reciente
- Tolerancia a particiones: El sistema se banca seguir andando aunque se pierdan mensajes dentro de la red entre distintos nodos


## Locks enentornos distribuidos
No tenemos TestAndSet atomico en este entorno
Es dificil saber si varios piden el lock decidir quien lo dijo primero.
- Uno podria mirar el timestamp, pero todos no necesariamente tienen el mismo clock, porque no lo comparten.
  
Soluciones posibles:
- Tener un nodo orquestrador (central) al cual le pido el lock
  - Es un cuello de botella esto porque todos los pedidos van a este
  - Unico punto de falla


## Como saber en un entorno sin clocking compartido cuando sucede un evento antes que otro?
Lo primero que es clave es que NO ES NECESARIO sincronizar relojes para saber si un evento sucede antes que otro.
Lo importante es saber si un evento paso antes o despues que otro, pero no exactamente CUANDO paso.

La idea es definir un orden parcial no reflexivo entre los eventos. Orden parcial porque no siempre voy a poder decir para cualquier par de eventos cual vino primero, por la regla 4:
- Si dentro de un proceso, A sucede antes que B, tonces A -> B
- Si E es el envio ed un mensaje y R su recepcion, E -> R.
- Si A -> B y B -> C, entonces vale A -> C (transitividad).
- Si no vale ni A -> B, ni B -> A, entonces A y B son concurrentes. Basicamente si no puedo asegurar que 2 eventos se dieron en cierto orden, entonces pasaron a la vez.

Para implementarlo se hace asi:
- Cada procesador tiene un reloj real o un entero que va creciendo cada vez que lo leo.
- Cada mensaje que intercambio lleva adosado la lectura del reloj.
- Como la recepcion es siempre posterior al envio, cuando recibo un mensaje que dice que fue enviado en el momento T pero T es un valor mayor al de mi reloj actual, actualizo mi reloj a t+1. Basicamente si recibo un mensaje que es del futuro segun mi reloj esto no puede pasar y actualizo mi reloj para arreglar esto.

Con esta metodologia se da un orden parcial. Se puede definir un orden total desempatando por process id por ejemplo.

Hay varios algoritmos que se montan sobre esta idea.

## Acuerdo bizantino
El problema dice que todos los procesos que pueden tener un valor 0 o 1 inicialmente, tienen que ponerse de acuerdo en un numero finito de pasos (WAIT-FREEDOM) en un valor comun.

Hay distintos resultados dependiendo de la falla presente entre nodos.

### Falla de comunicacion
Es un problema que no tiene solucion.

### Nodos pueden morir
Se puede decir que si fallan a lo sumo k < n procesos entonces se puede resolver el consenso con O((k+1)n^2) mensajes.

### Nodos no confiables
Se puede resolver para n procesos y k que mienten sii n > 3k y la conectividad es mayor a 2k.

## Clusters
Son muchas computadoras juntas, que estan conectadas en una red de alta velocidad con un scheduler en comun.
Definicion mas comun es computadoras que trabajan de manera cooperativa

## Grids
Tengo un conjunto de *clusters*. Cada cluster colabora con los demas para lograr algo.

## Clouds
Clusters donde uno puede alquilar una capacidad fija o bajo demanda.

## Scheduling en sistemas distribuidos
Compartir vs balancear: IMPORTANTE INVESTIGAR POR CUENTA PROPIA PERO NO ES RELEVANTE

## Algoritmos clasicos en sistemas distribuidos

### Modelo de fallas
En cualquier algoritmo distribuido algo importante a considerar es cual es el modelo de fallas que soporta el algoritmo, es decir, que se banca o no:
- Nadie falla
- Que un nodo no funcione
- Que la red se particione
- Me banco zombies pero en ciertos momentos
- Fallas bizantinas, donde los procesos pueden comportarse de manera impredecible

Los clasicos van a suponer lo primero, que ningun nodo falla, pero aun asi son un quilombo.

### Metricas de complejidad
Es la complejidad en cuanto a mensajes enviados. Tambien suele ser importante cuanta informacion requiere un algoritmo para funcionar.
No necesariamente por requerir mas informacion para funcionar mejor va a ser mejor, puede ser que no tenga esa opcion en mi situacion y me tenga que conformar con poca info.

### Algoritmo Exclusion mutua distribuida
En cada momento un solo nodo en la red tiene un token. Esto se lo denomina *token passing*.
Los nodos se configuran de manera circular, se enumeran y se van pasando en ronda el token.
Si yo quiero entrar a la seccion critica en este caso simplemente espero a recibir el token.

Considerando que un nodo puede fallar, si el nodo que tiene token se cae cague.

La performance es mala porque estoy todo el tiempo siendo interrumpido para recibir el token y luego tengo que encargarme de enviarlo. Puede no ser necesario estar haciendo circular el token.

Hay varios protocolos que usan esta idea:
- FDDI
- TDMA
- TTA

### Algoritmo Exclusion mutua distribuida V2
Mejora del anterior, donde:
- cuando un proceso quiere entrar a la seccion critica le envia al resto (incluyendose) una solicitud (Pi, ts), donde Pi es su pid y ts es el timestamp actual.
- cada proceso cuando recibe la solicitud responde o encola la respuesta.
- el proceso que solicito solo entra si recibe todas las respuestas
- si logro entrar cuando salgo le digo a respondo a todas las solicitues que pueden entrar (porque tengo que esperar mi turno para entrar devuelta).
- cuando se recibe un mensaje, contesto bajo estas condiciones:
  - si no me interesa entrar
  - quiero entrar, no entre todavia y el ts de la solicitud que recibi es menor a la que yo hice, entonces tiene prioridad.
- para que este algoritmo funcione necesita conocer a todos los demas para enviarle solicitudes, y tambien requiere conocer el orden entre los eventos, es decir, sincronizar los timestamps en la red.
  
Esta version tiene una pro con respecto a la v1, que es que no circulan mensajes de procesosque no quieren entrar a la seccion critica.

Para que todo esto funcione se necesita:
- No perder mensajes
- Ningun proceso falla

### Locks distribuidos
Usa el protocolo de mayoria, el cual tiene la siguiente idea:
- Queremos lockear un objeto del cual hay N copias *sincronizadas*
- Para obtener un lock, tengo que conseguir el lock de la mitad mas 1 de los nodos.
- Cada nodo me responde si me puede dar su lock o no.
- Cada copia del objeto  tiene un nro de version. 
- Si logro escribir un objeto, tomo entre todas las copias la de nro de version mas alta, la incremento en 1 y actualizo a todos con este nuevo nro.

Este protocolo no permite que 2 nodos tengan el lock a la vez, porque cada uno decidio que tenia la mitad mas 1, lo cual es imposible.

Tampoco puede pasar que se lea una copia desactualizada, porque si yo tengo la mitad + 1, seguro hay una copia ahi que es la mas nueva, ya que el ultimo que hizo una modificacion tambien escribio sobre la mitad + 1, entonces yo necesariamente tengo alguna de las copias que escribio el anterior.

### Eleccion de lider
Tengo un grupo de procesos que quiere elegir a uno como el lider, porque mi protocolo necesita que haya un proceso que haga algo distinto al resto.
Suponemos que la red no tiene fallas.
- Cada nodo mantiene un status que dice si es o no el lider, inicialmente que no es el lider
- Los procesos se organizan en anillo y se pone a circular el ID
- Al recibir un mensaje, el receptor compara el ID recibido con el suyo, y hace circular el mayor
- Cuando este mensaje dio toda la vuelta se sabe quien es el lider
- Se gira devuelta un mensaje de notifiacion para que todos sepan quien es el lider

### Instantanea global consistente
Se tiene una red de nodos que intercambia info.
Se puede pensar como un banco (red) con distintas sucursales (nodos)
La pregunta es como conseguir un snapshot del sistema consistente.

Cuando un proceso quiere recibir un snapshot del sistema se envia a si mismo un mensaje de marca.
Cuando un proceso recibe por primera vez un mensaje de marca guarda una copia del mensaje y envia otro mensaje de marca a todos los demas procesos de la red. 
Desde el momento que envia a los demas procesos mensajes, el proceso emisor empieza a registrar todos los mensajes que recibe de cada vecino hasta que recibe la marca de todos.
De esta manera el emisor construye una secuencia de todos los mensajes que recibio de cada proceso antes de que se tome la instantanea (llegue la marca).
Finalmente cuando se termina esto queda constituido el estado global por las copias de las marcas que recibieron todos los procesos y los recibidos de cada proceso, que se pueden enviar al que pidio el snapshot por ejemplo para constituir la imagen completa.

### 2PC (2 Phase Commit)
Se lo denomina acuerdo en 2 etapas.
Se trata de grabar informacion en una base de datos de la cual hay varias copias asegurandonos que o se escribio en todas o todas las copias estan de acuerdo que no se pudo escribir.

Basicamente la idea se trata de tener 2 fases:
- La primera es preguntar a todos si estan de acuerdo en que se haga la transaccion.
- La segunda es que si se recibio si de todos se hace el commit

Si se recibe un no en la primera fase se aborta, o en caso de que haya pasado un tiempo maximo sin recibir todos los si tambien.
Anotamos cada si que recibimos
Si se logro conseguir el si de todos entonces entramos a la segunda fase donde avisamos a todos que quedo confirmado el commit

## Variaciones de algoritmos de consenso
- K-Agreement: me conformo con que al menos K esten de acuerdo
- Aproximado: ninguna esta mas de x unidades de tiempo desactualizado
- Probabilidad: acoto la proba de que esten actualizados los nodos
  
## Resumen
- Modelos de fallas y metricas de complejidad
- Exclusion mutua y locks distribuidos
- Eleccion de lider
- Instantanea global consistente
- 2PC

LIBRO RECOMENDADO: Nancy Lynch, Distributed Algorithms, Morgan Kaufmann

# 13 - Virtualizacion IMPORTANTE: NO ES TAN RELEVANTE PARA SO ESTO

## Microkernels
Era una idea para combatir las desventajas del kernel monolitico, con el objetivo de:
- Tener menos codigo privilegiado, porque en el monolitico mucho era privilegiado y podia ser muy vulnerable
- Actualizar mas facil, porque en ese momento habia que recompilar todo el kernel y rebootear ante una actualizacion
- Mayor flexbilidad y extensibilidad
- Evitar que si crashea un servicio se muera el sistema
- Poder cambiar facil a distintos modos de servicios, porque antes habia que compilar un kernel especificamente para un cierto modo de uso, por ejemplo workstation

El microkernel buscaba tener:
- Manejo basico de memoria
- Comunicacion entre procesos (IPC) liviana
- Manejo basico de E/S

El resto seria provisto por servicios. Por ejemplo, el manejo de networking seria un proceso que yo me comunico con el a traves de IPC para conseguir lo que necesito.

No funciono porque era mucho mas lento que un monolitico
Si se utilizaron ideas de esto, para solucionar los mismos problemas que solucionaba el micro kernel pero con el monolitico

## Virtualizacion
Tenemos por un lado los n recursos fisicos y los quiero utilizar como m recursos logicos.
Dicho facil es cuando hablamos de una computadora que esta simulando ser otra.

Hay varios objetivos de esto:
- Portabilidad (JVM)
- Simulacion: un ejemplo es cuando se hace codigo para celu y se lo simula en un emulador para testing
- Aislamiento: pongo varias maquinas virtuales en una maquina real, una para cada cliente para aislarlos y que no afecten al resto
- Pariticionamiento de HW
- Agrupamiento de funciones 

Hubo varios intentos de virtualizacion

### Simulacion
Se tiene un proceso que simula ser una maquina, con memoria, registros, etc.
No escala porque es muy lento y ademas se pierde simular cosas como DMA, interrupciones, etc.

### Emulacion de HW
La idea es emulacion mediante HW. Para emular otro sistema le pedimos ayuda al HW, donde tenemos un chip que ayuda a virtualizar.
Problemas solucionados por hardware:
- Ring aliasing: Si mi emulador de VMs es un proceso mas, no corre con maximo privilegio. Si lo corro con maximo privilegio, no voy a poder ejecutar un programa dentro de el
- Address-space compression: Como hacer que funcione la segmentacion, la paginacion con todo su sistema que aisla entre procesos corriendo.
- Non-faulting access to privileged state: Que hacer con el trap generado por instrucciones privilegiadas.
- Interrupt virtualization: Como hacer que haya interrupciones en el SO virtualizado.
- Access to hidden state: como puede hacer el simulador para simular los registros que son parte del estado del procesador y no consultables por software
- Cache: esta para el SO principal, que pasa con el virtual? 

La solucion mediante HW de parte de intel es agregar extensiones VT-x al procesador, que proveen 2 modos:
- VMX root: Es el modo normal con el que se venia trabajando, con alguna cosa nueva.
- VMX non-root: Es el modo nuevo, al cual se puede switchear mediante una instruccion desde VMX root. Lo importante es que el hardware sigue corriendo el codigo de este proceso, pero con restricciones.


#### Virtual Machine Control Structure 
Estructura nueva que es una especie de registro pero en memoria.
- Campos de control: indica cuales de las interrupciones tiene que recibir cada uno. Un ejemplo seria setear en esta estructura que el input del teclado lo reciba la VM, o por ejemplo con puertos E/S.
- Estado completo de la VM
- Estado completo de la maquina real para switchear.
  
Cada vez que la computadora esta en VMX non-root y se realiza una accion no permitida entonces se retorna al modo normal y la maquina real toma el control.
Si esto pasa el controlador de la VM recibe el control y puede emular por ejemplo el disco si se quizo acceder al disco que estaria virtualizado, o terminar a la VM, etc.


### Desafios / Problemas
Las preguntas son:
- Que pasa con todo lo que tenia  el kernel para acceder rapido al disco (reorder, etc)
- Que pasa cuando tengo picos de carga en mas de una CPU si 2 VMs me exigen mucho a la vez
- Unico punto de falla: Si falla el HW real, se caen las VMs

### Exitos de virtualizacion
- Capacidad de correr sistemas viejos
- Aprovechar el equipamiento para meter varias VMs en una
- Sirve para desarrollo/testing/debugging, porque puedo correr software dentro de un entorno donde quizas se modifica el disco pero al ser virtualizado puedo cerrar la VM y volver a abrirla como si nada hubiera sucedido.
  

### Esquema de no VM vs VM

![alt text](13/non_vm_vs_vm.png)

## Contenedores (Docker x ej) IMPORTANTE INVESTIGAR NO SE ENTENDIO
La  situacion es la siguiente:
Tengo varias aplicaciones que usan distintas bibliotecas
Lo hago es tomar mi aplicacion y generar una imagen Docker.
Agarrro la estructura de directorios y lo guardo en una imagen de disco

### Dockerizar infraestructura
Supongamos que tengo 3 apps que usan el mismo web server y db.
Genero una imagen docker que tiene el webserver configurado de cierta manera. 
Genero una imagen docker que tiene la db configurada de cierta otra manera.
Estas 2 imagenes son archivos binarios con un nombre.

Luego armo mi app con todas las configuraciones, y luego para que la app funcione registro los nombres de las iamgenes del web server y la db que serian las dependencias a correr para que funcione la app.
Esto tiene una gran ventaja que es que el codigo que uno arma es el mismo en todos los lugares que corra.


## Orquestacion de aplicaciones contenerizadas: Kubernetes
Uno tiene una app dockerizada, con todas las dependencias en el file de config donde uno dice remapeame tal puerto, levantame tal volumen, dependo de tal base de datos o server.

En Kubernetes uno tiene un cluster de computadoras que pueden ser reales o virtuales, seria una serie de nodos, distribuidos en tales maquinas y le dice que vaya levantando imagenes docker de acuerdo a parametros que tienen que ver con el nivel de carga. 
Esto seria el escalamiento automatico. Uno puede decirle basicamente a Kubernetes como ir levantando y bajando instancias dependiendo la demanda. Tambien puede hacer un deploy de una nueva imagen docker de manera "suave", diciendole de a cuantas instancias ir bajando y reemplazando por las nuevas versiones.

## Cloud
El Cloud es la pc de otra corporacion, que tiene un datacenter en algun pais, con otra legislacion a la de uno, asi que no estas protegido por las leyes de tu pais.
Aparece la idea de combinar las ideas anteriores. 
No hace falta adminisitrar la infra con los centros de computos con la dificultad que implica. 
Es una forma de bajar el costo de brindar apps en la red, alquilando por hora unos que ya estan montados y te provee con herramientas para adminisitrar todo.

Hay que tener CUIDADO porque las nubes no son hetereas


# NOTAS DE FINAL
Plantear escenarios de distintas cosas, y ver que hariamos ahi. Ejemplo:
Funcionalidad loca de un FS y en cual FS estaria bueno implementarlo y por que

