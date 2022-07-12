# Preguntas de Telegram (A partir de 20/12/2019)
PREGUNTAS Y RESPUESTAS: https://docs.google.com/document/d/1QOFw99da5EBd8kaa5tli7xId3tFPLQNIvbWytqu8m7k/edit#heading=h.r3og0pdzdv7r

## 20-12-2019
Procesos:
   - X Una página es compartida por dos procesos. Puede suceder que para un proceso sea de solo lectura mientras que el otro tenga permitida la escritura? 

Sincronización:
   - X como podrías sincronizar dos procesos usando IPC?
   - X Cuál es la diferencia entre Spin locks y semáforos? Cuándo utilizaría cada uno? 

Seguridad:
   - X describir setUID y buffer overflow y un ataque que incluya ambos

E/S:
   - X como afectan los dispositivos de memoria de estado solido a los algoritmos de scheduler de E/S
   - X definir Storage Area Network (SAN)

File System:
   - X proponer un file system para un dispositivo que se escribe una única vez (y sabes todos los datos que queres escribir) y lecturas múltiples (estilo DVD). Detallar como se puede minimizar el espacio utilizado en el dispositivo.

Distribuidos:
   - X Describir los algoritmos de 2-phase commit y 3-phase commit


## 10-2-2020
- X Estructuras de reserva de memoria
- X RAID + SAN
- 2PC
  - X ¿Que fallas son bloqueantes?
- X Escrituras en SSD
- Journaling: Logico/fisico
  - X ¿Donde estan los datos commiteados (el log)?
- Estructuas en EXT2/inodos. 
  - X ¿Las estructuras a las que apuntan los inodos con indireccion tambien son inodos?


## 11-2-2020
- X como afectan los dispositivos de memoria de estado solido a los algoritmos de scheduler de E/S

## 21-2-2020
X Diferencia entre virtualizadores y contenedores, estructura del FS ext2 y diferencias con ext3, explicar journaling, qué es una función de hash, diferencia entre el comando su y sudo, consideraciones a tener en cuenta cuando se pasa de disco duro a solido, y qué es POSIX

X 2 phase commit; buffer overflow, relacionarlo con permisos y formas de prevenirlo; pasar de disco duro a sólido; diferencias entre spin locks y semáforos; si tenés un HW muy rudimentario cómo hacés para implementar semáforos; cómo garantizar transparencia para acceder a directorios remotos; por qué Windows requiere que los drivers tengan una firma digital y de qué otra manera se podría lograr el mismo objetivo

## 25-2-2020
a) Diseñar un protocolo para que un cliente A pueda hacer RPC con un servidor B a través de un canal seguro. A conoce la IP de B y su clave pública, pero B no conoce ninguno de estos datos de A (y tampoco puede recibir la clave pública a través de un mensaje, porque es trampa :) ).
Me dejó asumir que B tiene una lista de usuarios y passwords, y A es uno de ellos.

## 27-2-2020
- X Que se busca optimizar en Scheduling? 
- X Que scheduler usarias para un celular? 
- X Scheduling en e/s hdd vs ssd
- X Dac en fat, ext y ntfs
- X Que alternativa hay a Dac?
- X Usaste Windows alguna vez? (También puede ser chistoso el profe)
- X Diff semáforo y spin lock
- X Que es deadlock? 
- X Condiciones de Coffman
- X Monitores
- X VM vs CT
- X Nfs
- X Distribuido basado en cluster

me preguntó con qué tema quería empezar
- X le dije seguridad
- X me preguntó para qué se usaba una función de hash en sistemas operativos
- X le hablé de qué era una función de hash y qué propiedades tenía
- X y que guardás las contraseñas hasheadas
- X me preguntó si tenían algún otro uso y le dije lo de firmar código
- X después me preguntó lo de las diferencias de scheduling de pedidos a disco en HDD y SSD
- X cómo hacen ext2 y ext3 para aprovechar las características de un HDD (mantienen los inodos y los bloques de datos de un mismo archivo en un mismo grupo de bloques)
- X y después me preguntó por qué NFS no es realmente distribuido
- X y cómo podría hacer para que sí lo sea

## 5-3-2020

X Diferencia entre proceso y thread y relacion entre thread y funciones reentrantes.
X Que es el arbol de procesos y su importancia. 
X Que es un deadlock. 
X Journaling. 
X Storage Area Network. 
X DAC vs MAC. 
X Que es una funcion de hash y dar dos usos en seguridad. 
X Por que se miden en cilindros los algoritmos de scheduling de disco.
X Que problemas trae el token passing y soluciones. Diferencia entre sistemas paralelos y distribuidos

Procesos
- X explicar diferencia entre proceso y thread y relación de este último con funciones reentrantes
- X explicar qué es un árbol de procesos y cuál es su importancia

Sincronización
- X explicar deadlock con un dibujo y en pocas palabras

Administración de E/S
- X por qué los algoritmos de scheduling se miden en cilindros. ¿Está bien? ¿por qué?
- X explicar SAN

Seguridad
- X explicar qué es una función de hash segura y ejemplificar dos usos en un sistema operativo
- X comparar DAC y MAC

Sistemas de archivos
- X explicar journaling en ext3. Qué problema resuelve?


Sistemas distribuidos
- X explicar diferencias entre sistema paralelo y distribuidos
- X explicar los problemas del mecanismo de token passing y sus resoluciones

## 30-8-2020
1) X Cuál es la diferencia entre threads y proceso?

2) X Tenemos un SO Real Time. Y queremos a pesar de penalizar a otras tareas o que las cosas sean más lentas, asegurar predictibilidad en el tiempo que nos toma poder atender una tarea. Que podemos hacer desde el enfoque de la memoria. 

3.A) X Hay relación entre las páginas y los bloques? Cuál es? Cómo aprovecha esto ext2?

3.B) X Qué es la fragmentación? Cuál nos impactaba?

4.A) X Tenemos una actualización del SO. Cómo nos podemos asegurar que quedó todo bien y era de quién decía?

4.B) X Qué cosas conoces se usan y le agregarías en este caso a la firma a parte del hash?

5) X Tenemos unos sensores que sensan y escriben en memoria compartida. También tenemos unos procesos que leen los valores, hacen un cómputo, y escriben en memoria compartida. Que primitivas brindarías para implementar esto y por qué?

## 9-2-2021
- X que fs para sistema embebido
- X como funciona canary
- X (creo) por que puede pasar que tenes muchos procesos en waiting, y como podria tratar de arreglarlo si no pudiese agregar memoria
NOTA: En canary le interesaba no solo la idea general sino especificamente que sucedia, es decir hubiera servido hablar de que te sirve principalmente cuando te quieren escribir el ret de una funcion, y que se valida cada vez que estas volviendo de la funcion. Hablando que la parte del canary es un agregado del compilador

X Me tomó Chapa, preguntó lo mismo de procesos que pregunta siempre: si un sistema está lento por qué funcionaría agregar otro procesador, y lo mismo pero agregando memoria.
X Lo mismo con sincronización: qué primitivas de sincronización usar para distintos escenarios.
X De FS me preguntó qué FS me conviene para implementar un undo del último cambio hecho (acá hice aguas). Y diferencias entre FAT e inodos.
X De seguridad me preguntó también, la típica de buffer overflow, mecanismos para prevenirlo, y en qué se basa MAC.
X De memoria me mostró un árbol de procesos y cada proceso tenía páginas asignadas y cada proceso tenía repetida la página C y me preguntó por qué podía pasar eso.

X VFS
X RAID
X Inversión de prioridades
X 2PC
X Algoritmos de scheduling de disco
X Buffer overflow

- X Hablar de file systems en si (FAT/inodes) hable un poco las pros y las contras de uno. Hablamos tambien de cuando tiene sentido tener los archivos no en lugares aleatorios, sino de manera secuencial (CD/DVD que tenes una sola escritura). Para que sirven los block groups.
- X File systems distribuidos, hablamos de la interfaz VFS aca. Sus limitaciones. Como podria hacer para poder tener 2 discos distribuidos, y que los dos contengan la misma informacion, para usarlo de redundancia. Ahi hablamos de como se podia manejar esto para que no tenga fallas, si se caia la conexion en el medio y demas
- X Seguridad: Como poder brindar de manera segura las actualizaciones de un software. Como poder un cliente A conectarse a un servidor B (podia usar publica y privada A)

## 18-2-2021

Que es un proceso, que es un thread y en que se diferencian?
  -> X Que estructura debo mantener en memoria para poder tener procesos? (PCB)
  -> X Que deberia agregar a la PCB para manejar los threads?
  -> X Algo asi de como, en que momento se ejecuta un thread? (En el quantum del proceso)


X Afinidad. Que informacion extra tenemos que tener en la PCB para mantener afinidad en un sistema multicore?
  -> No era exactamente asi, pero la respuesta era que necesitamos saber cual fue el ultimo core en donde se ejecuto el proceso, para poder mantener la afinidad (no andarlo cambiando de core si es posible). Esto vino a cuento de que los procesos se mantienen en UNA sola lista en memoria, y no que cada core tiene la suya propia.


Paginado.
  -> X Para que sirve, que ventajas tiene vs tener todo en memoria contigua. (pregunto algunas cosas mas de paginado que no me acuerdo)

File System.
  -> X Que es, para que sirve.
  -> X Me hizo una pregunta que no entendi, pero que queria que le explique como se compone (inodos)
  -> X Que mantiene un inodo, como es la estructura de los directorios.
  -> X Concepto de BlockGroup (o clouster como esta en el sylber).

Sistemas Distribuidos.
  -> X Que son
  -> X Para que sirven/Ventajas (tiene 2 beneficios, velocidad de computo por el "paralelismo" y descentralizacion del computo, si se cae un nodo el servicio sigue en pie, el uso la palabra REDUNDANCIA)

NFS.
  -> X Se puede considerar que es distribuido? (Rta: no, tiene un modelo cliente-servidor)
  -> X Supongamos que lo expandimos del lado del servidor y queremos generar redundancia. Ahora tenemos N servidores que ofrecen el NFS y los tengo que mantener sincronizados, como hago? (Esta ya la habia visto, le propuse que por cada transaccion hecha podiamos hacer un 2PC para que todos se enteren)
  -> X Si un nodo se cae, como hacemos para que se entere despues de las transacciones que no tiene? (o sea si queda desactualizado, le dije que con un journal y que cuando conecte pregunte los ultimos cambios o algo asi).

Seguridad.
  -> X Ejemplo de un error de seguridad ocacionado por race condition (aca mori, segun me dijo fue el ultimo ejercicio de el taller de seguridad).
  -> X Se puede considerar Deadlock un problema de seguridad? (rta: si)


## 25-2-2021
1. X Explicar paginación, para que sirve. Ahí charlamos de los page faults, del tamaño de las paginas y como se relaciona con el tamaño del bloque. 
2. X para que sirve el bit executed de la tabla de paginas (evitar buffer ovverflow) y que puede hacer el atacante que no pudo ejecutar su codigo porque el bit estaba activado para agitarla igual. esto último no lo sabía pero parece que se llama "return to libc"
3. X qué es un sistema distribuído; nfs es distribuido? como implementaría nfs distribuido? ahí mencione 2pc y me hizo explicarlo y explicar como se sincroniza un nodo participante que se cayo despues de aceptar la transación pero antes de recibir el commit/abort.
4. X describir ext2 (le hable de la estructura en gral). explicar que pasa cuando apago abruptamente la compu y vuelvo a prenderla. diferencia symlinks vs hardlinks

- X Sistemas Distribuidos (para qué, por qué, ventajas, desventajas, etc)
- X Algoritmos de commits, problema bizantino
- X NFS: qué es
- X Si Samba era distribuido o no.
- X Cómo convertir un sistema no-distribuido a distribuido
- X Drivers (qué son, qué permiten (abstraccion del hardware))
- X Si hay un bug en un driver, qué puede pasar? (desde ejecución arbitraria de código nivel kernel a trabar el kernel)
- X Cómo aseguras que un driver sea confiable? (por ejemplo, en Windows, qué puede hacer Microsoft para mejorar eso (firmarlo))
- X Qué es una firma? Cómo se hace? Qué hace el receptor cuando quiere comprobar la firma?

- pregunta - procesador

  - X tengo un sistema que mejora con agregarle procesador, cual sería la razón y como podría verla

  - X y que debería ver para que la solución sea agregar memoria?

  - X contame algo de los estados por los que atravieza un proceso

- X pregunta - arbol de procesos

  me mostró un arbol de procesos, con alguna páginas compartidas, me pidió que explique las razones

- X pregunta - overflow

  contame que es buffer overflow

- X pregunta - sincronización 1

  - X usarías un TASLock, para acceder con mutex al disco? 

  - X para acceder a una estructura de memoria, accesible por a lo sumo 3 procesos. Que te parece un contador atómico y un if que comprueba el contador y luego lo incrementás?

  - X usarías un semáforo para incrementar un contador compartido?

- X pregunta - FS

  Para un backup total, que FS elegirias?

  - para un incremental?
  - para un snapshot consistente, con el sistema corriendo?

Erdosain, [25/2/2021 21:18]
al final, todas las discusiones de FS, lo estabamos encarando mal

Erdosain, [25/2/2021 21:19]
la palabra clave que el chabón quiere que le cuentes es LOCKING de las estructuras

Erdosain, [25/2/2021 21:19]
encará la pregunta por ahí y te va a ir mejor que a mi

Erdosain, [25/2/2021 21:19]
(igual no me fué mal eh, por más que no respondí algunas cosas me puso una buena nota)


## 22-4-2021
Randy 📵, [22/4/2021 17:58]
[In reply to Randy 📵]
X 1) si un sistema mejora su performance por agregarle un cpu más, a qué se puede deber/qué debería ver para concluir que necesita un cpu más
X 2) qué filesystem elegir para implementar undo del último cambio y cómo hacerlo
X 3) muestra un árbol de procesos en lo que algunos comparten páginas y pregunta a qué se puede deber
X 4) buffer overflow: qué es, cómo mitigarlo
X 5) evaluar las siguientes implementaciones para sincronización:
i) usar tas para acceder a disco
ii) usar semáforos para incrementar un contador
iii) usar spinlock para un recurso que tienen que poder acceder 3 procesos a la vez

X 1) Que es un page fault y como sa handleea?
Aca hable un poco de todo de paginacion y termine mencionando sobre el bit de ejecucion para mitigar BO 
X 2) Que es buffer overflow y como se mitiga?
X 3) Que es un VFS? y derivo en que es NFS? es distribuido? como lo harias distribuido? Aca medio que hable de hacerlo consistente y enganche con 2pc
X 4) Que es el PCB? Aca hable de procesos en general y me pregunto que contiene un PCB, y como lo extenderia para threads
X 5) No me acuerdo bien la pregunta pero era algo tipo: tenes ext2 y no tiene journaling: como hace el sitema para darse cuenta de que hubo un problema y como lo arregla?

X 1) que es un proceso? el scheduler guarda informacion en la TLB? cual? que pasa con los threads? que pasaria si compartieran el stack? que es la afinidad? como te afecta si el scheduler es preemptive o non-preemptive en la implementación de un semaforo?
X 2) que es una función de hash? como se usa en el login de un SO. q problemas tiene usar un hash para autenticar usuarios? (se puede bruteforcear, habia q hablar de salt y de aplicar varias veces la función de hash). como haría para mejorar la seguridad? (aumentar la cantidad de veces q se aplica el hash). que otros usos tiene la funcion de hash? (integridad)
X 3) explicar ext2. que es un fs distribuido? que es nfs? como harías un file system que sea realmente distribuido (q se guarden los bloques en servers distintos)? q algoritmo usas para actualizar una copia del archivo que esta en varios nodos? (quería q diga 2pc, pero lo agarré x el lado de token ring y exclusión mutua en red)
X 4) que problemas tiene token ring? que harias si el token se pierde? (termine usando elección de lider y parece q era esa la respuesta)

## 14-5-2021

X Pregunta si hay algún tema que te guste más. Yo le dije que fs y scheduler. Me dijo que le explique fat y ext2. Me preguntó si fat tiene algo de seguridad. Me preguntó cuál es la motivación para agrupar las cosas en grupos con ext2. me preguntó si en un escenario en el que tenés ext2 sin ningún tipo de journaling hay alguna manera de saber si cuando se apaga repentinamente la PC tenés inconsistencias. La idea era responder que tenés que chequear si es consistente la meta data de los inodos y los bitmap de bloques
X Me preguntó cómo funcionaba el scheduling con múltiples colas y después me pregunto si había algún problema con que las prioridades fuesen fijas.
X Después me hizo preguntas de seguridad con respecto a encriptación. Básicamente explicar lo de clave pública/clave privada y usar hash.
X Después me preguntó sobre NFS, me preguntó si era realmente distribuido y después me preguntó cómo haría para hacerlo distribuido de manera consistente. O sea, explicar 2pc.

## 11-6-2021

Tobi Carreira Munich, [11/6/2021 17:25]
[Forwarded from Tobi Carreira Munich]
X Qué tendria que ver en un sistema para que decida que va a andar mejor
- Con un procesador mas
- Con mas memoria

X Que sistema de archivos usaria para implementar un undo

X Estaria bien usar spinlock para acceso a disco?
X Estaria bien usar un semaforo binario para el acceso a un contador?
Mostro un codigo del estilo
if (x<3) {
    x++
    Seccion crit
    x--
}
X Para controlar a lo sumo 3 procesos en la sección critica, hay algun problema?

Tobi Carreira Munich, [11/6/2021 17:25]
[Forwarded from Tobi Carreira Munich]
X Explicar buffer overflow

## 27-7-2021
Me tomo Baader el miercoles pasado, duro 36 mins 
X Me pregunto si preferia algun tema, le dije que distribuidos
Y me pidio que explicara token ring, 2pc, y hacia preguntas que pasa si falla un nodo, si se cae la conexion, etc. Luego me pregunto sobre VFS y si era realmente distribuido, que necesitaria para ser distribuido, luego hablamos un poco de sguridad, stack overflow, canarios y otros tipos de ataques como ddos. Luego hablamos de filesystems, me pidio que explicara FAT vs ext2, y me hizo varias preguntas especificas sobre la estructura de ext2, como por ejemplo en que parte se guardan los bloques libres de memoria (hay un bitmap), etc. Hablamos de memoria virtual, paginacion, segfaults y thrashing. Finalmente hablamos de raid, me pregunto la diferencia entre raid 4 y 5 y que era la paridad  o que informacion se guardaba en los discos o secciones de paridad

## 11-8-2021
X Como hacer snapshot, buffer overflow, que es un buckup incremental/diferencial, por qué dos procesos compartirían páginas
Y tas (test and set), carga del cpu, trashing. Varias preguntas son similares a los últimos finales que aparecen en cubawiki

X Rodolfo me tomo un poco de todo, File system ext2 (mas que nada)/fat, raid 4 vs raid 5. monitor vs semaforo (condiciones de coffman). encripcion simetrica/asimetrica y sincronizacion

X yo elegi empezar por threading y me pregunto dif proceso-thread, primitivas de sync (reg atomicos, TAS, CAS, semaforos)
algunas cosas de scheduling
X me pregunto RAID 5 y me re descoloco
X NFS y SAN
X 2/3PC
X RSA

X Empezo por procesos, la pregunta de siempre que esta pasando si agregar una memoria ayuda y si agregar un procesador ayuda. (Aca no le gusto mucho mi forma de expresarlo o explicarlo, como que no fue muy limpia)
X Despues memoria. Me mostro un arbol de 4 procesos y que paginas tenian mapeadas. Me pregunto porque un proceso padre y su hijo tenian las mismas paginas mapeadas. Despues porque todos los procesos tenian la pagina C, que podia ser esa pagina (ahi le dije kernel pero tambien estaba esperando biblioteca compartida).
X File system: Snapshot (queria como se implemente, esa no la tenia). 
X Que file system usarias para poder hacer un undo del ultimo cambio (le dije ext2 + journaling, le gusto).
X Seguridad: Buffer overflow, exlicar y metodos para prevenir. 
X SQL injection
X Sincronizacion: la pregunta de siempre. 3 situaciones errones. Acceso a disco con spin lock. Seccion critica para incrementar un contador con semaforos. y una "seccion critica" a al que quiero que solo entren 3 procesos pero mal implementada

## 13-9-2021
Me tomó baader, varias de las preguntas son las mismas que tomaron en las ultimas fechas. Así fue:
X me preguntó que tema me gustaba, le dije sistemas distribuidos. Me preguntó qué es un sistema distribuido y que hablara un poco de eso (lo que yo tuviera ganas), yo lo definí y empecé a hablar de los problemas de un sistema distribuido (en particular dificultad de acceder a un recurso compartido). 
- X Sistemas distribuídos: me hizo la pregunta de si NFS es un sistema distribuído (para que conste: toda la información está en un servidor central y esa información es la que se distribuye a los demás nodos, entonces podemos decir que "no esta verdaderamente distribuído". Si se cae el servidor central, nos quedamos sin servicio) y cómo implementaría distribución de verdad y consistente (expliqué 2/3pc, ya que nos permite realizar operaciones de forma consistente en todos los nodos).
- X Filesystems: me pregunto cuales son las estructuras importantes de EXT2 (superbloque, inodos, grupos de bloques, hablar un poco de ellos) y después me preguntó si ocurre un apagón o lo que sea (y por ende se desmonta el filesystem en medio de una operación) cómo detectarlo y cómo recuperar el filesystem, considerando que en EXT2 todavía no tenemos journalling (yo ya había mencionado el dirty bit, mencioné eso de nuevo y agregué que podemos correr fsck para comparar la info del superbloque contra la de los inodos/block groups, hablamos un poquitito mas en detalle de qué se compara)
- X sincronización: diferencia entre spin lock y semáforos, en qué contexto preferiríamos uno sobre el otro y por qué (sobre lo segundo: semáforo induce un cambio de contexto, lo cuál es un overhead significativo del uso de semáforos, si la sección crítica es "corta"/poco compleja no vale la pena porque el tiempo que pasan otros procesos haciendo busy waiting resulta menor comparativamente)
- X filesystems/sincronización: qué es una PCB y qué se guarda en ella? (me lo preguntó cuando hablé de cambio de contexto en la pregunta anterior)
- X raid: diferencia entre RAID4 y RAID5, cuál es mejor y por qué (en raid4 tenemos todos los bloques de paridad en un disco, en 5 están distribuídos en los discos => por más que en ambos se usen dos discos por escritura, en raid4 el disco de paridad es un cuello de botella para todas las escrituras mientras que en raid5 se distribuye la carga de actualizar paridad entre todos los discos)
- X seguridad: cómo implementar distribución de actualizaciones para mi propia distribución de ubuntu (dije que usando sistema de clave publica/privada + hashing para garantizar autoridad e integridad de los paquetes de actualizaciones, porque no quiero que alguien distribuya paquetes de actualizacion que no son mios y quiero garantizar que le llegue lo que le mandé a los usuarios). Me preguntó a quién le pediría esa clave privada/pública (dije a alguna autoridad certificante) y cómo distribuiría la pública (pasandoselas cuando se bajen mi distro o poniendola en alguna especie de programa actualizador de mi distro, es pública así que no importa mucho)

## 18-10-2021
Juan Gonzalez, [18/10/2021 19:01]
Bueno, me tomo Schapa. Acá van los temas:
- X Arrancamos hablando de Snapshots me dio rienda suelta. Hable de como se implementaban, para que se usaban.
- X Que file system usarias para implementarlo? me dijo que cualquiera le servía siempre y cuando pueda justificarlo. Le dije que Ext2 porque tenia una estructura arborea.

- X Seguridad: Me dio un codigo para mirar. Era algo asi:
void f (char[16] usuario): 
  char[18] usuarioCopia;   
  int puedeAcceder = puedeAcceder(usuario);   
  strcpy(usuario, usuarioCopia);
  if (puedeAcceder){  
   //codigo super seguro  
}

Juan Gonzalez, [18/10/2021 19:05]
- X Me pregunto que problemas podia llegar a tener el codigo de arriba.
- X Me pregunto que habiamos visto para mitigar buffer overflow, hablamos de stack canaries y me pregunto si servia en este caso

-  Luego seguimos con paginacion, me pregunto que era y que atributos podia encontrar en una pagina. Me pidio los mas simples que se me ocurrieran y que las explique.
- X Me preguntó en que casos dos procesos compartirian memoria con paginacion. Aca hable un poco de fork pero me di cuenta que queria que hable de Bibliotecas compartidas.
- X Me pregunto un motivo mas por el cual querria compartir memoria. Hable un poco de evitar redundancias y metí tema de Bibliotecas compartidas.

- X Me pregunto que tenia una page Table.
 Aca no estaba seguro de para donde encarar y me dijo que piense en PCBs. 
 Basicamente estuve hablando de qué eran y para que servian.

- X Hablamos de system calls, para que sirven, que pasos trae aparejados.

- X Luego me dio tres escenarios para sincronizacion entre procesos.
 1- acceso a disco con SpinLock()
 2- me dijo que queria acceder a una estructura pero con a lo sumo 3 procesos a la vez y me dio un  codigo tipo:
  int atomic =3;
 if (atomic.get() <=3) {
  atomic.inc();
  //CRIT
  atomic.dec(); 
}
3-Acceso a memoria con semaforo binario

## 18-11-2021
Me tomó Baader. Me preguntó por cuál tema empezamos, sincro fue mi respuesta.
X Para qué necesitamos sincronización? Qué soluciones nos ofrece el HW (primitivas)?
X Cuándo conviene spinlock y cuándo semáforos? Me pidió que le hable sobre TTAS.
X Scheduling: qué datos sobre los procesos maneja el scheduler? Qué cambia cuando hay threads?
X De seguridad me pidió que hable de cómo se protegen las claves. Él tenía muchas ganas de que le hable de SALT me di cuenta. Patiné una banda cuando me preguntó de qué exploit te protege SALT.
Seguimos hablando de FS.
X EXT2: Estructuras del FS. Qué info hay en el superbloque. Para qué la partición en grupos.
X Cómo se da cuenta el SO de que puede haber inconsistencias (bit del superbloque).
X Quiso que hable sobre hardlink vs softlink y me cabió. Quería que explique un toque ambas. Problemas del softlink. Por qué entonces no usamos siempre hardlink, qué problema tiene.
X Para qué queremos sistemas distribuidos? (paralelismo + robustez ante fallas) Qué problemas nos puede traer?
Le dije que los componentes precisan comunicarse vía envío de mensajes, lo cual es lento.
Y que no es fácil mantener consistencia y coordinar a las partes.
X Hablamos de 2PC, weak/strong termination.


## 14-12-2021
Al principio me preguntó por qué tema quería arrancar así que hablamos de distribuidos. 
X Me preguntó si NFS es distribuido, 
X en qué caso particular falla 2PC y cuál es la diferencia con 3PC. 
X Después sobre file systems preguntó la estructura de un inodo en general y por qué se separa en block groups. 
X Sobre I/O me preguntó la diferencia entre un disco magnético y un SSD. 
X Después preguntó la diferencia entre un thread y un proceso, qué info guarda el scheduler de un thread
X me pidió que le hable del problema de inversión de prioridades. 
X Para cerrar, me preguntó qué pasa cuando hay un page fault D:
Agustin, [14/12/2021 18:15]
X en una me mostró 4 procesos y sus páginas mapeadas (compartian varias y otras no) y me pidió que especule qué podrían ser las compartidas y por qué
Esa fue la más 'rara', ya que no la vi en preguntas viejas
Y fue en la que patiné un cachito tambien, porque no se me ocurrió en qué caso esas páginas compartidas serían del kernel

Agustin, [14/12/2021 18:17]
X Dsps me tiró situaciones de  sincronizacion con posibles soluciones y tenia que juzgar si estaban bien

Agustin, [14/12/2021 18:18]
X Tambien como implementaría un UNDO de un paso en FAT y en Inodos

## 21-12-2021
X q tiene q pasar para q un sistema mejore agregando cpu, q tiene q pasar para q mejore agregando memoria

X memoria: el arbol de procesos compartiendo memoria y te pregunta xq pueden estar compartiendo (en lugar de un motivo me pidió varios), terminé hablando de syscalls un toque 

X seguridad: buffer overflow, me preguntó formas del ataque o algo así y hablé de return to libc, shellcodes y despues preguntó como solucionarlo (de ahi a problemas del canary)

X filesystems: la mas rara, algo así como en un disco no confiable cómo duplicaría todo si no puedo confiar en el disco (acá full guitarra tbh)

X sincronización: los 3 casos de siempre (spinlock a disco, el del contador y semaforo para contador)

X Procesos: tabla de procesos, PCB. Afinidad de procesador.
X E/S: sdd write amplification, borrado en sdd y en disco magnetico.
X De FS me pregunto por journaling y como recuperaba de una caida sin journaling. Si en ext3 recuperaba los datos si habia una caida. (Solo consistencia)
X En sync vimos el problema de lectores y escritores y como evitar inanicion del escritor.
X Algo de scheduler, prioridades, inanicion si hay prioridades fijas.
X En seguridad buffer overflow, setuid y como si se me ocurria una forma de usar ambos mecanismos juntos.
PENDING Como el so valida los certificados de autenticacion para los updates.
Seguro me estoy olvidando de algo porque me paseo bastante por todos lados, pero me ayudaba y me tiraba pistas si desbarrancaba un poco.

## 17-02-2022

X La clásica de agregar memoria y un procesador a un sistema.
X Después de memoria preguntó por qué podía ser que todos los procesos de un sistema compartieran una página (podía ser para syscalls o biblioteca compartida), y por qué 2 procesos en particular podían compartir una página (ahí hablé de fork y copy on write).
X De seguridad me preguntó por buffer overflow, qué era y cómo evitarlo (acá me pidió alguna forma más además de canary que no me acordaba, me dijo que marcar páginas como no ejecutables).
X De FS me preguntó qué filesystem elegiría para implementar un UNDO de un paso (tanto FAT como inodos sirven).
X Último me tomó los tres casos de siempre de sincro.

Suerte para las próximas fechas!

juan pablo miceli:
X Me tomó baader, el esquema del final fue el mismo de siempre. Arrancamos por decir que es un sistema distribuido, que beneficios trae. Después pasamos a seguridad, hablamos de certificados, RSA, firma digital, autoridades certificantes.
X Volvimos a sistemas distribuidos y me preguntó la diferencia entre grid y cluster, hablamos de 2PC y 3PC. 
X Que es una syscall, para que están, como funcionan. Para que sirve el fork y que problemas trae.
X Discos, diferencia entre ssd y hdd, que problemas tiene el escribir en ssd. Como hace un disco para saber si un bloque está libre/puede ser borrado (ahí me cabió, dijo que existe algo llamado "trim", pero ni se que es).
X Vimos un poco también de virtualizacion y containers (que son y como se implementan) 
Es un copado bárbaro, te re ayuda a pensar las cosas que no te salen de una

## 24-02-2022

Buenas, dejo por acá lo que me tomó Rodolfo. 
X Empecé hablando de procesos, me hizo hincapié en explicar los estados, que subsistema utiliza eso (scheduling) y donde se guarda la información (PCB, tabla de procesos). Me pidió que le diga que información extra puede utilizar el scheduling para los procesos (prioridad, afinidad, etc). Después me preguntó la diferencia entre threads y procesos, que cosas comparte un thread y que tiene exclusivo.
X Después pasamos a FS, me preguntó diferencia entre un hard link y soft link y por qué usaría un soft link (se usa cuando el link es entre archivos de distinto FS).
X Después pasamos a disco, me preguntó problema y algoritmos de lectura en HDD, después diferencia con SDD y problemas de escritura (write amplification) y que pasa cuando se borra un archivo en sdd.
X Por último seguridad .. como se encriptan las contraseñas, uso de SALT y me pidió un detalle más (acá patiné) que es que en linux hay una técnica adicional que es rehashear la contraseña varias veces para dificultar que un atacante pueda hacer un ataque de fuerza bruta.

Me tomó muy similar. 
Agrego: 
X También me hizo hablar de los estados de los procesos.
X En la parte de compartir páginas muestra un árbol de procesos y los page frames asignados a cada uno (compartir toda la memoria vs compartir algunos frames: posibles causas). Por qué todos los procesos de un sistema compartirían un page frame (área de kernel).
X Para sincronización muestra distintos códigos muy simples de contador. Spinlock sobre disco: es eficiente? Alternativa?

Arrancamos por sincro, me preguntó bastante por las primitivas en general y semáforos haciendo hincapié tanto en monoprocesadores como en multiprocesadores.
X De E/S hablamos de SSD y HDD. También me preguntó por RAID y que más o menos le vaya diciendo ventajas y desventajas de los distintos tipos.
X De FS hablamos bastante se ext2 y ext3. Cuáles son sus estructuras y para que está cada una. Me preguntó también como se crean y borran archivos con estas estructuras y hablamos un poco de como manejar los espacios de memoria libre. Hablamos también de links, tanto softlink como hardlink.
X En distribuidos hablamos de 2pc, 3pc, token ring, los problemas de token ring y como mejorarlos, hablamos un poco de consenso y sincronización también.
X Terminamos hablando de clusters y grid y como están implementadas.
X Terminamos con seguridad. Hablamos mucho sobre RSA, SETUID, DAC y MAC. Y terminamos hablando un poco sobre HTTP y HTTPS, más que nada las diferencias principales y como HTTPS nos garantiza mayor seguridad.

## 05-03-2022

- X Hablar sobre procesos, qué es, qué los diferencia de un thread, explicar la estructura de la PCB
- X Qué es necesario modificar para que la PCB soporte threads
- X Qué pasa si los threads comparten stack
- X qué es un file descriptor? Nombre 3 syscalls que los afecten. Por qué el fd se crea cuando uno hace open y no sé vuelve a chequear permisos del archivo cada vez que uno hace write/read,etc.. (Era por una cuestión de performance). 
- PENDING Qué problemas de seguridad hay asociados a los file descriptors y cómo se resuelve eso en SELinux (Ahí me mato jajajajajaja) la respuesta era que si uno le quita permisos a un proceso de acceder a cierto archivo mientras este lo tiene abierto, puede seguir accediendo.
- X Cuál es la diferencia, ventaja y desventaja de hard links y soft links?
- X Por que no puedo hacer referencia a los inodos de otro FS?
- PENDING Qué estructuras se ven afectadas y como cuando yo hago rm de un archivo en Linux (No olvidarse que puede tener hard links, hay que hablar hasta de cómo se modifica el block group)
- X Si yo tengo binarios en una computadora, cómo puedo saber que no fueron modificados? (Hay que tenerlos hasheados)
- X Ahora si el atacante me modifica el binario y el hash, como puedo solucionar esto? Cómo se resuelve en RedHat? (Hay que pensar en hashearlos con algo que el atacante no tenga acceso, por ejemplo, la clave privada del owner del binario, es decir que los binarios vengan firmados)