# Procesos y otras yerbas
- qué es POSIX
  <details>
    <summary>Respuesta</summary>
    The Portable Operating System Interface is a family of standards specified by the IEEE Computer Society for maintaining compatibility between operating systems
  </details>
- explicar los estados
  <details>
    <summary>Respuesta</summary>

    - CREATED: El proceso acaba de ser creado. Puede transicionar de aca a READY.
    - RUNNING: El proceso esta ejecutandose en el CPU. Transiciona de aca a BLOCKED si hace algo que lo bloquee o a READY si se le acaba el quantum o a DONE si termina con su trabajo.
    - BLOCKED: El proceso se encuentra bloqueado por algun motivo (I/O, semaforo, etc). Transiciona de aca a READY.
    - READY: El proceso esta esperando a ser ejecutado en la CPU.
    - DONE: El proceso termino con lo que tenia que hacer.
  </details>
- que subsistema utiliza los estados de los procesos?
  <details>
    <summary>Respuesta</summary>
    scheduling
  </details>
- donde se guarda la información de los procesos
  <details>
    <summary>Respuesta</summary>
    PCB, tabla de procesos
  </details>
- diferencia entre threads y procesos, que cosas comparte un thread y que tiene exclusivo.
  <details>
    <summary>Respuesta</summary>
    Proceso: Es un programa puesto a ejecutar. Puedo tener multiples procesos ejecutando simultaneamente para un mismo programa. Es la unidad de trabajo en un sistema.

    Thread: Es un unidad basica de utilizacion del CPU, el cual contiene un thread id, un program counter, un set de registros y un stack. Comparte a su vez con otros threads que pertenecen al mismo proceso la seccion de codigo, de datos y otros recursos del OS como los files abiertos.

    Como se ve, un proceso no comparte su espacio de memoria con otros (excepto quizas bibliotecas compartidas x ejemplo, pero no stack, heap, codigo) a diferencia del thread que si comparte heap, codigo *pero no stack tampoco*.

  </details>
- Qué es necesario modificar para que la PCB soporte threads
  <details>
    <summary>Respuesta</summary>
    Se agregan las TCB. Como un thread es un hijo del proceso y comparten memoria, no es necesario tener un PCB por cada thread donde se guardan las paginas de memoria que tiene, los archivos abiertos, etc. Lo que si es necesario es lo minimo indispensable para correr ese thread, que se guarda en la TCB:
    - Program Counter
    - Stack Pointer
    - Thread ID
    - Estado de los registros
    - Pointer al PCB del padre
    - Estado en scheduling (BLOCKED, RUNNING, etc) 

  </details>
- Qué pasa si los threads comparten stack
  <details>
    <summary>Respuesta</summary>
    Si los threads compartieran stack, entonces se interferirian entre ellos al hacer llamados a funciones y tambien al pushear variables al stack.
  </details>
- La clásica de agregar memoria y un procesador a un sistema.
  <details>
    <summary>Respuesta</summary>
    Caso Procesador: Agregar un procesador mas tiene sentido si los procesos se encuentran normalmente en los estados READY y RUNNING. Esto implica que nunca llegan a bloquearse porque probablemente no les da el quantum y son desalojados antes de esto. Al agregar un procesador, se pueden tener mas procesos corriendo a la vez, osea se mejora el throughput y potencialmente pueden llegar a pasar a BLOCKED mas frecuentemente. Habria que chequear esto ultimo analizando la *carga del sistema*, que seria la cantidad de procesos en READY.

    Caso Memoria: Agregar memoria tiene sentido si los procesos estan mayormente BLOCKED porque se esta produciendo *trashing*, es decir, la memoria esta ocupada por todos los procesos. Esto provoca que cuando un proceso quiere acceder a una porcion de memoria que no esta cargada en memoria, se produzca un *page fault*, que lleva a un *context switch* (es caro), y el SO tiene que encargarse de swappear una pagina de la memoria con la del disco que necesita el proceso. Esto ultimo es lento y provoca que el proceso pase a BLOCKED.
  </details>
- Afinidad. Que informacion extra tenemos que tener en la PCB para mantener afinidad en un sistema multicore?
  <details>
    <summary>Respuesta</summary>
    No era exactamente asi, pero la respuesta era que necesitamos saber cual fue el ultimo core en donde se ejecuto el proceso, para poder mantener la afinidad (no andarlo cambiando de core si es posible). Esto vino a cuento de que los procesos se mantienen en UNA sola lista en memoria, y no que cada core tiene la suya propia.
  </details>
- PCB
  <details>
    <summary>Respuesta</summary>
    
    Contiene informacion de un proceso
    - Estado de los registros
    - Estado del PC
    - Estado del SP
    - File descriptors abiertos
    - PID
    - Informacion de la memoria del proceso, en cuanto a page tables por ejemplo
    - Estado del proceso (blocked, ready, running, etc)
  </details>
- en que momento se ejecuta un thread?
  <details>
    <summary>Respuesta</summary>
    En el quantum del proceso

    Si hablamos de los user-level threads: Since the kernel is not aware of the existence of threads, it operates as it always does, picking a process, say, A, and giving A control for its quantum. The thread scheduler inside A decides which thread to run, say A1. Since there are no clock interrupts to multiprogram threads, this thread may continue running as long as it wants to. If it uses up the process’ entire quantum, the kernel will select another process to run.
  </details>
- relacion entre thread y funciones reentrantes.
  <details>
    <summary>Respuesta</summary>
    La idea es que una funcion reentrante es una funcion que puede ser ejecutada varias veces simultaneamente sin que se "rompa nada". Por ejemplo, las funciones recursivas tienen que ser reentrantes, porque se ejecutan varias veces el mismo codigo recursivamente y no tiene que romperse. Por otro lado, la relacion con los threads es que estos pueden correr la funcion reentrante simultaneamente y va a funcionar como deberia, ya que eso nos garantiza la funcion, sin necesidad de usar un lock por ejemplo para que solo un thread la ejecute a la vez.
  </details>
- Una página es compartida por dos procesos. Puede suceder que para un proceso sea de solo lectura mientras que el otro tenga permitida la escritura? 
  <details>
    <summary>Respuesta</summary>
    Puede pasar si estamos por ejemplo en el caso del problema del productor - consumidor, donde uno escribe y el otro lee. En este caso el productor tiene la pagina con permisos de escritura y el consumidor con solo lectura.
  </details>
# IPC
- qué es un file descriptor? Nombre 3 syscalls que los afecten. Por qué el fd se crea cuando uno hace open y no sé vuelve a chequear permisos del archivo cada vez que uno hace write/read,etc.. (Era por una cuestión de performance)
  <details>
  <summary>Respuesta</summary>
  Es un identificador unico representado por un numero entero que identifica un file abierto por el proceso. Cada FD se almacena en la FD table del proceso particular. Cada FD apunta a una entry en la global file table que guarda varios datos sobre el file abierto. Entre estos esta el modo en el cual se abrio el archivo (lectura, escritura, etc), el inodo del file, etc.

  Hay 3 FDs que se crean por default al iniciar un proceso, y estos son STDIN, STDOUT y STDERR.

  Syscalls que afecten a los FD:
  - Create: crea un file descriptor dado un nombre de file y un modo
  - Open: abrimos un archivo y nos retorna el FD asociado para operar sobre el
  - Close: cerramos el archivo mediante apuntado por el FD
  - Read
  - Write
  </details>
- Que es una syscall, para que están, como funcionan. Para que sirve el fork y que problemas trae.
  <details>
  <summary>Respuesta</summary>
    SYSCALL

    Son parte de la API del kernel, funciones que nos provee para hacer cosas que solo el kernel puede hacer. Nos permiten pedirle al kernel distintos servicios, como acceder al disco, crear procesos, etc.
    La idea es que cuando un proceso hace utiliza una syscall, se genera una excepcion,  se genera un cambio de privilegio de user mode a kernel mode, y se le transfiere el control al handler de la excepcion particular. El codigo de este handler es parte del codigo de kernel, y por lo tanto se genera un cambio de privilegios para poder correr eso.

    FORK

    fork se utiliza para crear un proceso hijo de otro proceso, el cual va a compartir el mismo codigo, mismo estado al momento de hacer el fork.

    PROBLEMAS
    
    el fork() utiliza la estrategia de *copy on write* para cuando se crea un proceso hijo que tiene su propia memoria, lo cual es eficiente porque hasta que se modifique el hijo se crea super rapido y no hay que copiar todo.
    El problema de esto es que si bien no se copia toda la memoria, a medida que el proceso padre tiene mas y mas memoria asignada, esto conlleva a copiar mas y mas estructuras de la memoria virtual del proceso padre al hijo, para tener esta misma informacion.
  </details>

# Scheduling
- qué objetivo prioriza SJF y por qué no se usa en la práctica?
- problema de inversión de prioridades
  <details>
  <summary>Respuesta</summary>
    Es un problema que se da en scheduling cuando una tarea de mas prioridad es indirectamente reemplazada por una de menor prioridad, invirtiendo asi las prioridades. Se da comunmente cuando hay un recurso que ambas quieren y hay un tercer proceso con la prioridad entre estos 2. La idea es que el proceso de baja prioridad toma un recurso, luego le toca al de mayor prioridad y se bloquea esperandolo. Luego cuando le toca al de menor prioridad devuelta este va a ser desalojado por el de prioridad media, sin que el recurso sea liberado y asi impidiendole a la tarea de mas prioridad cuando corra hacer algo.
  </details>
- Teniendo un algoritmo de scheduling al que los procesos le pueden pedir que se le agregue un quantum en determinado momento, y el scheduler te lo da sólo si eso ayuda al rendimiento total del sistema. ¿En que se tendría que fijar el scheduler para decidir si le corresponde quantum o no?
  <details>
    <summary>Respuesta</summary>
      Si hay más procesos en ready, si es probable que el proceso termine con ese quantum extra (si hay algún otro esperando que este termine para empezar), si durante ese quantum va a empezar a hacer entrada/salida…
  </details>
- ¿Qué desafíos enfrentan los algoritmos de scheduling para sistemas con varios procesadores?
  <details>
    <summary>Respuesta</summary>
    
    Tenemos por un lado el tema de la memoria cache en el procesador que viene corriendo un proceso. Al querer cambiar al proceso a otro procesador para que lo ejecute, perdemos esta memoria cache con los datos del proceso, potencialmente haciendo mas lenta la ejecucion de este. Para esto surge el concepto de afinidad de procesador, donde el scheduler mantiene informacion sobre la afinidad para intentar que siempre se mantenga ejecutando en el mismo proceador el proceso, aunque le tome mas tiempo ejecutarse. Obviamente, si en algun caso la espera es grande, puede cambiarse eventualmente a otro procesador sacrificando la cache por sobre la velocidad del sistema.

    El otro aspecto tiene que ver con balancear la carga de los procesadores, mediante 2 conceptos que se llaman push y pull migration. En push migration, un procesador se encuentra sobrecargado en su cola de procesos y le "pushea" a otro procesador alguno de esos procesos esperando. Tambien puede pasar lo opuesto,que un CPU vea que tiene espacio libre en su cola y transfiere todos los datos relevantes del procesador a este otro,
  </details>
- Que se busca optimizar en Scheduling? 
  <details>
  <summary>Respuesta</summary>

  - Ecuanimidad: cada proceso recibe un quantum justo de CPU
  - Eficiencia: que la CPU este ocupada todo el tiempo
  - Carga del sistema: minimizar cant de procesos en estado ready (hacer que la mayor parte se encuentre bloqueada)
  - Tiempo de respuesta: favorecer a los procesos que tienen que ver con cosas interactivas, para que el usuario perciba el minimo tiempo de respuesta posible
  - Latencia: minimizar tiempo requerido pára que un proceso aporte valor, de algun resultado
  - Tiempo de ejecucion: minimizar el tiempo total que le toma a un prcoeso ejecutar completamente
  - Rendimiento (throughput): a diferencia del anterior, es maximizar el numero de procesos terminados en una unidad de tiempo, a diferencia de 1 como en el anterior
  - Liberacion de recursos: priorizar que los procesos que tienen recursos para ellos terminen lo antes posible para que los liberen
  </details>
- Que scheduler usarias para un celular? 
  <details>
    <summary>Respuesta</summary>
    Usaria uno de interactivo porque el usuario tiene que percibir que la interfaz del celular funciona y no esta tildada, y que de alguna manera las cosas progresan mientras va tocando los botones de la interfaz.
  </details>
- carga del cpu
  <details>
    <summary>Respuesta</summary>
      Es la cantidad de procesos en Ready. Mientras mas baja la carga mejor.
  </details>
- scheduling con múltiples colas y después me pregunto si había algún problema con que las prioridades fuesen fijas 
  <details>
    <summary>Respuesta</summary>
    Este tipo de scheduling tiene varias colas de procesos, donde cada cola se corresponde con una prioridad distinta. La idea es que siempre van a ejecutarse los procesos con mayor prioridad hasta que no haya mas, y luego los de segunda mayor prioridad, y asi. 
    Cada cola puede manejar una forma de scheduling distinta internamente.

    El mayor problema de este algoritmo es que las prioridades no cambian y los procesos con menor prioridad pueden quedarse esperando sin ejecutarse nunca si sigue llegando procesos con mayor prioridad antes. Esto produciria starvation.
  </details>
- el scheduler guarda informacion en la TLB?
  <details>
    <summary>Respuesta</summary>
    No, la TLB se utiliza para cachear las traducciones de las direcciones de memoria de virtual a fisica, para reducir el tiempo de traduccion en proximas veces. Esto esta adentro de la MMU, el scheduler no puede tocar eso.

     memory cache that stores recent translations of virtual memory to physical addresses for faster retrieval.
  </details>

- (creo) por que puede pasar que tenes muchos procesos en waiting, y como podria tratar de arreglarlo si no pudiese agregar memoria
  <details>
    <summary>Respuesta</summary>
    Podemos tener muchos procesos en waiting porque no tenemos CPUs suficiente para estar corriendo a los procesos y que terminen

    Podriamos agregar entonces un par de CPUs para que aumente el throughput y asi tengamos mas procesos que finalicen y menos procesos en waiting
  </details>
- Tenemos un SO Real Time. Y queremos a pesar de penalizar a otras tareas o que las cosas sean más lentas, asegurar predictibilidad en el tiempo que nos toma poder atender una tarea. Que podemos hacer desde el enfoque de la memoria. 
  <details>
    <summary>Respuesta</summary>
    COMPLETAR QUIZAS NO
  </details>
# Sincronizacion entre procesos
- como podrías sincronizar dos procesos usando IPC?
  <details>
    <summary>Respuesta</summary>
    Pueden hablarse por el pipe mandandole al otro que quiere entrar a la seccion critica por ejemplo, junto con un timestamp. El otro proceso recibe esto y puede aceptar o no dependiendo de si el envio un mensaje antes con un timestamp mas viejo, en cuyo caso el proceso original va a dejarlo que entre primero y dps entra el.
    Podria verse como el token ring mejorado este.
  </details>
- las primitivas en general y semáforos haciendo hincapié tanto en monoprocesadores como en multiprocesadores
- problema de lectores y escritores y como evitar inanicion del escritor.
  <details>
  <summary>Respuesta</summary>
  readSwitch keeps track of how many readers are in the room; 
  
  it locks
roomEmpty when the first reader enters and unlocks it when the last reader
exits.

  turnstile is a turnstile for readers and a mutex for writers.

  Codigo de escritor:

  1 turnstile.wait()

  2 roomEmpty.wait()

  3 # critical section for writers

  4 turnstile.signal()

  5

  6 roomEmpty.signal()

  Posee un turnstile que cuando lockea no permite que mas readers entren hasta que el writer entre y haga sus cosas. 
  Con esto el writer solo tiene que esperar que el room este Empty pero de los readers que estaban dentro hasta el momento, no mas que eso.

  Codigo de reader:

  1 turnstile.wait()

  2 turnstile.signal()

  3

  4 readSwitch.lock(roomEmpty)

  5 # critical section for readers

  6 readSwitch.unlock(roomEmpty)

  </details>

- Para qué necesitamos sincronización? Qué soluciones nos ofrece el HW?
  <details>
  <summary>Respuesta</summary>
    La sincronizacion la necesitamos para que varios procesos puedan cooperar estorbandose lo menos posible y logren un objetivo, sin romper nada en el medio basicamente. Esto seria que no afecten una posicion de memoria a la vez por ejemplo, es decir, queremos evitar race conditions.

    El HW nos ofrece lo que se llaman primitivas de sincronizacion (reg atomicos, TAS, CAS (Compare and Swap), semaforos), las cuales son mecanismos de bajo nivel  que nos permiten tener un control sobre que hace un conjunto de procesos, para que coordine, etc. 
  </details>
- Cuándo conviene spinlock y cuándo semáforos? Me pidió que le hable sobre TTAS.
  <details>
  <summary>Respuesta</summary>
    SPINLOCK

    basicamente cuando el busy waiting va a ser corto, pero si va a ser largo no porque se va a perder mucha CPU al pedo ahi

    SEMAFORO

    si la sección crítica es "corta"/poco compleja no vale la pena porque el tiempo que pasan otros procesos haciendo busy waiting resulta menor comparativamente, pero si es larga si

    TTAS (Test and Test-and-Set)

    La idea es que cuando quiero hacer un lock, primero chequeo en un loop si la variable esta en false, en lugar de usar la llamada atomica TAS que es mas cara

    Si este es el caso, entonces ahi si uso TAS para intentar lockear el recurso.

    Con esto se logra mas eficiencia para solo usar el TAS cuando el lock esta libre.
  </details>
- Luego me dio tres escenarios para sincronizacion entre procesos.
    - Acceso a disco con SpinLock()
      <details>
        <summary>Respuesta</summary>
         No esta mal esto, va a funcionar bien, porque se va a evitar que 2 procesos accedan al mismo recurso efectivamente. El problema de esto es que no es eficiente, ya que estamos en un escenario donde la *seccion critica* seria el acceso al disco, y el disco es un E/S *lento*. Esto implica que puede tardar un tiempo entre que un proceso accede a este y lo libera, por lo que otros procesos van a estar un tiempo largo esperando su turno. Debido a esto, el Spinlock no es buena opcion, ya que se la pasa haciendo *busy waiting* para adquirir el disco, y esto gasta CPU innecesario. Una mejor opcion seria usar un mutex que utiliza un semaforo por abajo, que para escenarios donde la espera es larga esto encaja bien, ya que el proceso se va a dormir y se despierta recien cuando el disco se libero, dejandole el CPU a otros procesos en el medio sin desperdicir CPU.
      </details>
    - Acceso a una estructura que permite hasta 3 accesos simultáneos con semáforos.
      <details>
        <summary>Respuesta</summary>
        Es correcto porque se puede configurar al semaforo con un numero N que indica cuantos pueden hacer wait sobre el semaforo y adquirir el semaforo al instante. Entonces, poniendo N = 3, podemos tener hasta 3 que accedan al semaforo a la vez, y cada vez que uno termina de acceder y hacer sus cosas, puede hacer signal y liberar uno de esos 3 espacios para otro. De esta manera, siempre se tienen a lo sumo 3 accediendo a la estructura.
      </details>
    - Acceso a memoria con semaforo binario
      <details>
        <summary>Respuesta</summary>
        No esta mal, porque se esta logrando el acceso exclusivo a la memoria para modificarla o leerla uno a la vez. El problema es que no es ideal, ya que probablemente el proceso de acceder a la memoria, leerla o modificarla sea breve, y no justifica utilizar un semaforo que implica un *context switch* para que el proceso se vaya a dormir y se despierte cuando este se libere, que seguro sea rapido. En estos escenarios es mejor un Spinlock, ya que si bien se queda haciendo *busy waiting*, probablemente no lo haga por mucho tiempo dada la duracion dentro de la *seccion critica* en este caso.
      </details>
- condiciones de coffman y para que sirven
   <details>
      <summary>Respuesta</summary>
      Empezamos por las condiciones: ¿
      
      1 Un recurso puede ser usado por a lo sumo 1 proceso a la vez (Exclusion Mutua)
      
      2 Un proceso puede tener mas de 1 recurso a la vez (Hold and Wait)
      
      3 El SO no puede sacarle un recurso a un proceso (No preemption)
      
      4 Se produce un ciclo de 2 o mas procesos pidiendo un recurso que el otro tiene (Espera circular)

      Las condiciones buscan decirnos en que casos puede producirse un deadlock, aunque son incorrectas porque en algunos casos no es suficiente y en otros no son necesarias todas para que ocurra.
    </details>
- como te afecta si el scheduler es preemptive o non-preemptive en la implementación de un semaforo?
   <details>
      <summary>Respuesta</summary>
      No tiene sentido tener un semaforo en un scheduler non-preemptive porque no te van a sacar de la CPU en medio de la seccion critica, sino que vos decidis cuando hacerlo. No existe la idea de lockear un recurso si mientras estas ejecutandote vos podes usarlo todo lo que quieras hasta terminar de usarlo y asi dejar a la siguiente tarea.
    </details>
# Administracion de memoria
- por qué podía ser que todos los procesos de un sistema compartieran una página, y por qué 2 procesos en particular podían compartir una página 
  <details>
    <summary>Respuesta</summary>
    - hubo un fork y el copy on write deja las mismas paginas hasta que se produzcan cambios
    - bibliotecas
    - syscalls
    - shared memory
  </details>
- qué pasa cuando hay un page fault D:
   <details>
    <summary>Respuesta</summary>
    
    1. Se produce una interrupcion, se hace un context switch y el kernel pasa a tomar control
    2. Se fija que ocurrio un page fault y se pone a ver cual es la pagina virtual que se pidio. 
    3. Una vez obtenido esto, se chequea que el proceso pueda acceder a esta pagina. osea tenga permisos (en caso que no se tira segfault y se lo mata)
    4. Si este chequeo vale, se fija el OS si hay un page frame libre, y si no lo hay aplica el algoritmo para reemplazar un page frame.
    5. En este paso puede pasar que haya que escribir en disco el page frame que se esta tirando.
    6. Luego se carga de disco el page frame en memoria
    7. Finalmente se vuelve a hacer el context switch para que el proceso continue.
    </details>
- tiene relación el tamaño de una página con la cantidad de bloques que ocupa (que si no ocupa una cantidad entera hay fragmentación interna)
- paginacion, me pregunto que era y que atributos podia encontrar en una pagina.
  <details>
    <summary>Respuesta</summary>
    
    PAGINACION

    Es una forma de permitirle a un proceso que su memoria fisica no sea contigua. Lo que se hace es darle al proceso direcciones logicas, que pueden ser mas grandes que toda la capacidad que se tiene de memoria fisica, y luego se hace un mapeo a las fisica. 
    Un proceso tiene lo que se llaman pages en su mapa de memoria, y por atras estas mapean a frames que son la memoria fisica.

    Hay una page table por proceso

    PAGE TABLE ENTRY (atributos de pagina)
    - caching disabled
    - referenced (la leyeron)
    - modified (la escribieron)
    - protection
    - present/absent -> si esta cargada o no en RAM (podria estar en disco)
    - numero de frame (nos dice en que frame de la memoria fisica esta)
  </details>
- memoria virtual
   <details>
      <summary>Respuesta</summary>
      Es una tecnica para manejar la memoria que hace el OS permitiendole a un proceso tener mas memoria que la que hay de memoria fisica (RAM), utilizando por ejemplo el disco.
    </details>
- segfaults
   <details>
      <summary>Respuesta</summary>
      Ocurre cuando un proceso intenta acceder a una parte de la memoria que esta restringida, lo cual generalmente termina matando al proceso en cuestion
    </details>
- trashing y como evitarlo sin agregar memoria
   <details>
      <summary>Respuesta</summary>
      Es una situacion en la cual el SO ocupa mas tiempo de CPU swappeando paginas que ejecutando procesos propiamente.
      Se podria intentar evitarlo cambiando la estrategia de como ase administran los page frames para los procesos.
      Una opcion seria limitar a cada proceso a tener un conjunto fijo de page frames, para que no le robe los page frames a otro proceso y le produzca el swapping constante y este BLOCKED. El tema es que no queremos limitar a un proceso a una cantidad X de page frames, entonces se podria mejorar esto analizando estadisticamente cuantos page frames activos tiene por vez cada proceso (Working-Set en el silver)
    </details>
- para que sirve el paginado contra tener todo en memoria contigua.
   <details>
      <summary>Respuesta</summary>
      Sirve para poder asignar memoria a un proceso que no necesariamente esta siempre toda junta, lo cual evita el problema de que no haya mas espacio contiguo en memoria para asignarle a un proceso porque otro tiene ocupada la parte que le sigue por ejemplo.
    </details>
- Qué es la fragmentación?
  <details>
    <summary>Respuesta</summary>
      Ocurre cuando nuestro espacio de memoria libre se encuentra partido / fragmentado en distintas porciones. Esto produce que tengamos la cantidad suficiente de memoria para alojar a un proceso, pero al no ser contigua no podemos. Se pueden dar 2 tipos de fragmentacion:
    
    - Interna:

      Ocurre cuando se le asigna un bloque de memoria a un proceso pero este es mas grande que el que necesita. Terminamos con una porcion de memoria sin uso interna en el bloque, ya que no puede ser utilizado por otro proceso. Los fragmentos en este caso son esas porciones libres del bloque.

    - Externa:

      Hay suficiente memoria libre para satisfacer un pedido, pero no es contigua, entonces no puede usarse directo. Los fragmentos en este caso son esas porciones libres no contiguas de los bloques. Esto se resuelve con paginacion, ya que al dividir en paginas la memoria virtual y a su vez en frames la fisica, los posibles agujeros que vamos a tener son multiplos de estas pages/blocks, entonces siempre podemos rellenar esos agujeros con asignaciones de paginas/frames.

  </details>
- ¿Cómo es la protección de la memoria para los procesos, usando paginación?
   <details>
    <summary>Respuesta</summary>
    
    Some operating systems set up a different address space for each process, which provides hard memory protection boundaries.[2] It is impossible for an unprivileged[c] application to access a page that has not been explicitly allocated to it, because every memory address either points to a page allocated to that application, or generates an interrupt called a page fault. Unallocated pages, and pages allocated to any other application, do not have any addresses from the application point of view.

    Paging Memory protection in paging is achieved by associating protection bits with each page. These bits are associated with each page table entry and specify protection on the corresponding page.
    </details>
- Describa de qué manera el Sistema Operativo protege el espacio de memoria de un proceso. (2017)
   <details>
      <summary>Respuesta</summary>
      Depende de la abstracción de memoria que utilice el SO. Los SO actuales utilizan memoria virtual como método para que cada programa tenga su propio espacio de direcciones (dividido en páginas). Cuando un programa hace referencia a una dirección que no tiene asociada, la MMU (Memory Managment Unit) que se encarga de traducir las direcciones virtuales a direcciones físicas hace que la CPU haga un trap al SO (pagefault) y la rutina del kernel correspondiente termina el proceso (violación de segmento).
    </details>

# Administracion de E/S

- Drivers (qué son, qué permiten (abstraccion del hardware))
  <details>
    <summary>Respuesta</summary>
    Los drivers son una interfaz entre el hardware y el SO, que nos permite hablar con distintos dispositivos para lograr algun objetivo.
    Cada driver es especifico para un dispositivo, habla con el controlador del mismo que esta en el, y corre con maximo privilegio.
</details>

- Si hay un bug en un driver, qué puede pasar?
   <details>
      <summary>Respuesta</summary>
      Se puede colgar todo el sistema porque el driver corre a nivel kernel. Tambien podria explotarse esta vulne para ejecutar cualquier codigo a nivel kernel
    </details>
- me preguntó por RAID y que más o menos le vaya diciendo ventajas y desventajas de los distintos tipos.
  <details>
    <summary>Respuesta</summary>
    RAID

    Es una forma de guardar la misma informacion en distintos lugares en multiples discos para protegerla en caso de que se produzca una falla.

    Se suele usar mirroring (copiar la misma data en multiples discos) y stripping (partir la data y guardarla en distintos discos cada una)

    RAID 0 (Stripping)

    Reparto la info de un file en varios discos. Pro es que puedo paralelizar escritura, Cons es que se rompe un disco y cague xq no tengo redundancia

    RAID 1 (Mirroring)

    Tengo el doble de discos por informacion que guardo. Entoncces al momento de escribir en un disco escribo en otro tambien para replicar los datos. La Pro es que cuando hago lecturas puedo paralelizar y tengo redundancia. Cons es que gasto el doble de discos.

    RAID 4 (Block Level Stripping)
    La idea es guardar en un disco la paridad con respecto a los datos de los otros discos.
    Pros de esto es que puedo leer y escribir en paralelo de varios discos y ante una posible falla se arregla esa disco junto con la info de los otros mas el bit de paridad asociados. Cons de esto es que si llega a romperse el disco de bit de paridad cague y estoy escribiendo muy seguido en el mismo disco para actualizar paridades

    RAID 5
    Es igual a RAID 4 solo que la paridad por bloque no se guarda toda en un mismo disco sino que separte en cada uno de los discos un bloque de paridad

    RAID 0+1
    Se hace un combo entre RAID 0 y 1 (Stripping + Mirroring). La idea es ganar redundancia que falta en RAID 0.
    
    RAID 1+0
    Es lo mismo que 0+1 pero se hace mirroring primero y luego stripping. La idea es que los datos primero estan separados en 2 partes (stripping) y estos luego estan duplicados cada uno (mirroring). Estamos ganando mas redundancia aca porque tenemos duplicados los datos al fin del dia, y ganamos velocidad porque como la data esta strippeada tambien podemos escribir y leer pedazos de datos de distintos discos a la vez.
    
    <details>

- problema de lectura en HDD
  <details>
    <summary>Respuesta</summary>
    HDD
    Son lentos, porque tienen que mover un brazo y girar platos para leer datos
  </details>

- problemas de escritura en SSD
  <details>
    <summary>Respuesta</summary>
    Problema de escritura (write amplification) https://www.ontrack.com/en-us/blog/what-is-write-amplification-wa-and-how-does-it-effect-ssds MEJORAR ESTO

    Los SSD tienen un problema llamado write amplification, que puede acortar la vida util del SSD ya que este tiene una cantidad limitada de writes a sus celdas.
    Write amplificaton es un problema que consiste en que para una escritura logica al disco, se realizan multiples fisicas. Esto se debe a como es el SSD por dentro:
    Blocks are made out of several pages and one page is made out of several storage chips. The main challenge is that the Flash cells can only be deleted block-wise and written on page-wise. To write new data on a page, it must be physically totally empty. If it is not, then the content of the page has to be deleted. However, it is not possible to erase a single page, but only all pages that are part of one block. Because the block sizes of an SSD are fixed – for example, 512kb, 1024kb up to 4MB. – a block that only contains a page with only 4k of data, will take the full storage space of 512kb anyway.

    Sumado a como es el SSD por dentro, el otro problema es el algoritmo que pasa atras cuando uno hace un write:

    And this is not all when any data in the SSD is changed, the corresponding block must first be marked for deletion in preparation of writing the new data. Then the read/modify/write algorithm in the SSD controller will determine the block to be written to, retrieve any data already in it, mark the block for deletion, redistribute the old data, then lay down the new data in the old block.

    Retrieving and redistributing the new data means that the old data will be copied to a new location and other complex metadata coping and calculations will also add to the total amount of data.

    The result is that by simply deleting data from an SSD, more data is being created than being destroyed. Since Flash NAND chips are only good for a certain amount of read-/write cycles, Write Amplification (WA) results into lower life expectancy, endurance, and speed.

    El Garbage Collector esta en esto tambien:
    Garbage collection starts when a flash block is full of data, usually a mix of valid (good) and invalid (older, replaced) data. The invalid data must be tossed out to make room for new data, so the flash controller copies the valid data of a flash block to a previously erased block, and skips copying the invalid data of that block. The final step is to erase the original whole block, preparing it for new data to be written.

    Before and during garbage collection, some data — valid data copied during garbage collection and the (typically) multiple copies of the invalid data — is in two or more locations at once, a phenomenon known as write amplification.
  </details>

- que pasa cuando se borra un archivo en ssd y hdd.
  <details>
    <summary>Respuesta</summary>
    HDD:

    No tiene problemas como el SSD para escribir sobre datos que ya no sirven. Por eso lo unico que hace es marcar los datos como que ya no sirven y luego los sobreescribe sin problema.
    
    SSD:

    El SO utiliza un comando llamado TRIM para indicarle al SSD que bloques de datos no son mas necesarios y pueden ser borrados.
  </details>
- Discos, diferencia entre ssd y hdd.
  <details>
    <summary>Respuesta</summary>
    HDD

    Un HDD es un disco magnetico que contiene partes mecanicas en movimiento, como un brazo, platos, etc. Sus tiempos de lectura y escritura son lentos debido a estas partes mecanicas que deben moverse para leer y escribir. La transferencia de datos es secuencial, osea que escribe datos contiguos.

    SSD

    No contiene partes mecanicas, solo partes electronicas. Su tiempo de escritura y lectura es rapido, y la transferencia de datos es random access, osea que puede escribir datos en cualquier direccion arbitraria.
  </details>
- Como hace un disco ssd para saber si un bloque está libre/puede ser borrado.
  <details>
    <summary>Respuesta</summary>
    Hay un comando llamado Trim, que le dice al SSD que especificas areas contienen data que ya no esta en uso y se pueden borrar

  </details>
- en un disco no confiable cómo duplicaría todo si no puedo confiar en el disco (acá full guitarra tbh)
  <details>
    <summary>Respuesta</summary>
    WTF? ni idea esto, no vale la pena
  </details>
- diferencia entre RAID4 y RAID5, cuál es mejor y por qué
  <details>
    <summary>Respuesta</summary>
    en raid4 tenemos todos los bloques de paridad en un disco, en 5 están distribuídos en los discos => por más que en ambos se usen dos discos por escritura, en raid4 el disco de paridad es un cuello de botella para todas las escrituras mientras que en raid5 se distribuye la carga de actualizar paridad entre todos los discos)
  </details>

- que es un backup incremental/diferencial 
  <details>
    <summary>Respuesta</summary>
    Backup incremental

    Hacemos un backup el primer dia, y los subsiguientes dias vamos guardando las diferencias de los files con respecto al dia anterior.
    
    Para recuperar los datos necesito empezar desde el backup del primer dia e ir aplicando los cambios en cada dia hasta el dia que quiero recuperar los datos

    Backup diferencial

    Hacemos un backup el primer dia, y todos los subsiguientes dias vamos guardando las diferencias de los files con respecto al primer dia

    Para recuperar los datos necesita el backup del primer dia y el del dia hasta el cual se quieren recuperar los datos

  </details>
- cuando tiene sentido tener los archivos no en lugares aleatorios, sino de manera secuencial
  <details>
    <summary>Respuesta</summary>
    CD/DVD que tenes una sola escritura
  </details>
- que beneficio tengo por escribir los datos secuencialmente en CD/DVD por ejemplo?
  <details>
    <summary>Respuesta</summary>
    performance, algoritmo sencillo, menos metadatos porque puedo decir donde empieza y termina un file
  </details>
- por qué los algoritmos de scheduling se miden en cilindros (seek time). ¿Está bien? ¿por qué?
  <details>
    <summary>Respuesta</summary>
    La idea de esto es que los algoritmos de scheduling buscan mejorar lo mas posible los tiempos de acceso al disco para los distintos pedidos que existen. Por lo tanto, si uno compara 2 algoritmos para un conjunto de pedidos se va a quedar con el que gaste menos tiempo en total para todos esos pedidos. El tiempo que se gasta en cada pedido se relaciona con el *seek time*, que es el tiempo que le lleva al controlador de disco posicionar el brazo en el lugar correcto para leer/escribir data. Entonces tiene sentido que se mida con este tiempo la eficiencia de un algoritmo de scheduling.

    The performance of a disk scheduling algorithm is measured in terms of it seek time, rotational
    latency and bandwidth. Seek time is the amount of time taken by the read/write head of a disk to
    move to a particular track where the information is present. Rotational latency is the amount of
    time taken by the read/write head of a hard disk to get to a specified sector in a track, by rotation
    of platter. It depends on the rotating speed of the disk. Data Transfer time is the summation of
    time required for selection of a read/write head and the time required to transfer the data. and
    number of bytes to be transferred. Bandwidth is the rate at which data is read from or written on
    hard disk. The main objective of scheduling algorithm is to process all memory request with
    lesser seek time and rotational latency. 
  </details>
- SAN
  <details>
    <summary>Respuesta</summary>
    Es una red privada que conecta multiples servidores y unidades de almacenamiento. Permite tener multiples servidores y almacenamientos conectados a la vez, y el almacenamiento para un server se puede reservar dinamicamente. Por ejemplo, si un server esta con poco almacenamiento de disco, SAN puede reservarle mas a el. SAN le permite a clusters de servers compartir el mismo almacenamiento.

    Simply stated, a SAN is a network of disks that is accessed by a network of servers. There are several popular uses for SANs in enterprise computing. A SAN is typically employed to consolidate storage. For example, it's common for a computer system, such as a server, to include one or more local storage devices. But consider a data center with hundreds of servers, each running virtual machines that can be deployed and migrated between servers as desired. If the data for one workload is stored on that local storage, the data might also need to be moved if the workload is migrated to another server or restored if the server fails. Rather than attempt to organize, track and use the physical disks located in individual servers throughout the data center, a business might choose to move storage to a dedicated storage subsystem, such as a storage array, where the storage can be collectively provisioned, managed and protected.
  </details>
- como afectan los dispositivos de memoria de estado solido a los algoritmos de scheduler de E/S
  <details>
    <summary>Respuesta</summary>
    En SSD no es necesario hacer un algoritmo eficiente, por lo que dice en la respuesta de abajo.
  </details>
- diferencias de scheduling de pedidos a disco en HDD y SSD
  <details>
    <summary>Respuesta</summary>
    En SSD no es necesario hacer un analisis entre distintos algoritmos basado en los cilindros que utilizan a diferencia de HDD porque SSD permite acceder a cualquier posicion aleatoriamente, por lo que el *seek time* y *rotational latency* no existen.
    Debido a esto las opciones de scheduling en disco que existen en HDD en este caso dan igual, y la opcion de FCFS que podria no ser la mejor en HDD aca va bien porque no hay delay. 
  </details>
- Algoritmos de scheduling de disco
  <details>
    <summary>Respuesta</summary>
    https://www.geeksforgeeks.org/disk-scheduling-algorithms/

    Estos algoritmos se encargan de la planificacion de disco, que trata de como manejar la cola de pedidos de E/S eficientemente para lograr el mejor rendimiento posible. La planificacion no es responsabilidad del controlador de disco, porque no sabe los pedidos que voy a hacerle.

    Tenemos varias opciones:

    FCFS: Atiendo el primer pedido que encontre. Lo bueno es que es sencillo y no produce starvation. Lo malo es que es super ineficiente porque estoy moviendo la cabeza de un lado al otro sin tener en cuenta todos los pedidos.

    SSTF: Atender el pedido que esta mas cerca de la posicion actual del cabezal. Lo bueno de esto es que la cabeza se mueve lo menos posible y lo hace eficiente. Lo malo es que puede generar starvation si tengo un pedido que esta lejos y vivo atendiendo pedidos cercanos.

    Ascensor: voy atendiendo los primeros pedidos que encuentro en el camino mientras voy bajando / subiendo en los platos. Lo bueno de esto es que no tiene starvation como SSTF. Lo malo es que si recibo un pedido que esta justo en una parte que acabo de ver entonces hay que esperar hasta ir y volver a este.
  </details>
# Sistemas de archivos

- cómo garantizar transparencia para acceder a directorios remotos
  <details>
    <summary>Respuesta</summary>

    VFS

  <details>
- VFS
  <details>
    <summary>Respuesta</summary>
    Es una capa de abstraccion arriba de un FS concreto. La idea del VFS es permitirles a aplicaciones de cliente acceder a diferentes tipos de FS concretos en una forma uniforme.
    Por ejemplo puede usarse para acceder a almacenamientos locales o remotos transparentemente sin que sepa el cliente.
    Provee un conjunto de abstracciones del FS y las operaciones que son permitidas en estas abstracciones
  <details>
- Cuál es la diferencia, ventaja y desventaja de hard links y soft/symb links? Y por qué usaría un soft link? (se usa cuando el link es entre archivos de distinto FS).
  <details>
    <summary>Respuesta</summary>
    
    Hard link vs Symbolic link: https://www.linkedin.com/pulse/hard-link-vs-symbolic-mateo-garcia/
    
    Hard link

    Se tienen varios directory entry (varios file names, porque cada directory entry tiene metadata como el nombre del file y le apunta al inodo con los datos posta)
    La data del file real no se borra hasta que se borra el ultimo hard link. La desventaja que tiene este approach es que no puede apuntarle a files de otro FS. La ventaja que tiene es que si se actualiza el file va a seguir funcionando este link.

    A file in the file system is basically a link to an inode.
    A hard link, then, just creates another file with a link to the same underlying inode.

    When you delete a file, it removes one link to the underlying inode. The inode is only deleted (or deletable/over-writable) when all links to the inode have been deleted.
    Deleting, renaming, or moving the original file will not affect the hard link as it links to the underlying inode. Any changes to the data on the inode is reflected in all files that refer to that inode.

    Soft/Symbolic link

    En este caso en lugar de apuntar con 2 nombres a la misma data, se crea un primer file que tiene en su path el nombre al file de verdad. En este caso si se borran los soft links no se borra el file original. La ventaja de este approach es que estos links pueden cruzar los limites de un disco e ir a otro disco (disk mount) o hasta hablar de files remotos en otra computadora. Lo que tiene de malo es que es lento porque hay que hacer una lectura mas a disco para obtener la posicion del file posta, y tambien que si se renombra el file por ejemplo se rompe el link ya que usa el path name y no se actualiza cuando el file apuntado se actualiza.

    Symbolic links can span file systems as they are simply the name of another file.
  </details>

- Por que no puedo hacer referencia a los inodos de otro FS?
  <details>
    <summary>Respuesta</summary>https://unix.stackexchange.com/questions/290525/why-are-hard-links-only-valid-within-the-same-filesystem

    El inodo es una estructura de datos que se utiliza para representar a un objeto del FS, no puedo apuntarle a el porque es interno del FS. Lo que si puedo hacer es apuntarle a un directory entry que va a terminar apuntandole al inodo del FS, a traves de symbolic links. Esto es posible porque el directory entry es la *interfaz del FS, no el objeto interno en si*
  </details>

- Qué estructuras se ven afectadas y como cuando yo hago rm de un archivo en Linux (No olvidarse que puede tener hard links, hay que hablar hasta de cómo se modifica el block group)
  <details>
    <summary>Respuesta</summary>

    - en EXT2 se marca como unused al inode, pero se dejan intactos los pointers
    - en EXT3 se zeroean los pointers por el SO
    - se elimina el inodo de la lista del directory
    - se decrementa en 1 el link count de hard links
    - si el count llego a 0 entonces se marca al inode como eliminado
    - se marcan en el bitmap del block group a los blocks de este inodo como libres
  </details>

- ext2 y ext3. Cuáles son sus estructuras y para que está cada una
  <details>
    <summary>Respuesta</summary>
    Ext2
  
    the disk partition is divided into groups of blocks, irrespective of where the disk cylinder boundaries fall.

    The first block is the superblock. It contains information about the layout of
    the file system, including the number of i-nodes, the number of disk blocks, and
    the start of the list of free disk blocks (typically a few hundred entries). Next
    comes the group descriptor, which contains information about the location of the
    bitmaps, the number of free blocks and i-nodes in the group, and the number of directories in the group. This information is important since ext2 attempts to spread
    directories evenly over the disk.

    Two bitmaps are used to keep track of the free blocks and free i-nodes, respectiv ely, a choice inherited from the MINIX 1 file system (and in contrast to most UNIX file systems, which use a free list).

    Following the superblock are the i-nodes themselves. They are numbered from
    1 up to some maximum. Each i-node is 128 bytes long and describes exactly one
    file. An i-node contains accounting information (including all the information returned by stat, which simply takes it from the i-node), as well as enough information to locate all the disk blocks that hold the file’s data.
    Following the i-nodes are the data blocks. All the files and directories data are stored here. If a file or directory consists of more than one block, the blocks need not
    be contiguous on the disk. In fact, the blocks of a large file are likely to be spread
    all over the disk.
    I-nodes corresponding to directories are dispersed throughout the disk block
    groups.

    Ext3

    La diferencia principal que tiene con Ext2 es que se le agrega journaling

  </details>

- Ext2: Qué info hay en el superbloque.
  <details>
    <summary>Respuesta</summary>
    Es un bloque que se encuentra replicado en cada *block group* del FS y contiene informacion sobre la disposicion del FS:
  
    - numero de inodos
    - numero de bloques
    - numero de bloques libres
    - numero de inodos libres
    - numero de bloques por grupo
    - numero de inodos por grupo
    - comienzo de la lista de bloques libres
    - tamaño de bloque
    - tamaño de inodo
  </details>
  
- Ext2: Para qué la partición en grupos? / por qué se separa en block groups?
  <details>
    <summary>Respuesta</summary>
    Los block groups contienen informacion del bloque y varios inodos. La ventaja que aportan es que los files guardados en el mismo block group pueden ser accedidos con menor seek time en promedio de disco.  

    Block groups reduce file fragmentation, since the kernel tries to keep the data blocks belonging to a file in the same block group, if possible.
    The advantage of using block groups is that it adds to reliability and increased performance, because the control structures are replicated in each block group and hte distance betweent he i-node table and the data block is reduced.
  </details>
- Ext2: Cómo se da cuenta el SO de que puede haber inconsistencias.
  <details>
    <summary>Respuesta</summary>
    El superbloque tiene un dato que es *Mount Count and Maximum Mount Count*, los cuales indican si el FS tendria que ser chequeado, mediante FSCK.
   
    a file system can record its state within the file-system metadata.
    At the start of any metadata change, a status bit is set to indicate that the
    metadata is in flux. If all updates to the metadata complete successfully, the file
    system can clear that bit. If, however, the status bit remains set, a consistency
    checker is run.

    When fsck runs, it checks to see if the ext2
    file system was cleanly unmounted by reading
    the state field in the file system superblock. If
    the state is set as VALID, the file system is already consistent and does not need recovery;
    fsck exits without further ado. If the state is
    INVALID, fsck does a full check of the file system integrity, repairing any inconsistencies it
    finds. In order to check the correctness of allocation bitmaps, file nlinks, directory entries,
    etc., fsck reads every inode in the system, every indirect block referenced by an inode, and
    every directory entry. Using this information, it
    builds up a new set of inode and block allocation bitmaps, calculates the correct number of
    links of every inode, and removes directory entries to unreferenced inodes. It does many other
    things as well, such as sanity check inode fields,
    but these three activities fundamentally require
    reading every inode in the file system

    
    Aca estan todos los chequeos que se hacen: https://docs.oracle.com/cd/E19455-01/805-7228/6j6q7uf0e/index.html

    The superblock is checked for inconsistencies in:
    - File system size
    - Number of inodes
    - Free-block count
    - Free-inode count

    FS and Inode list size checks:
    - FS size tiene que ser mas grande que la cantidad de bloques del super bloque y la lista de inodos
    - La cant de inodos tiene que ser menor al maximo permitido

    Free Block Checks

    Free Inode Checks

    Los Inodes se chequean por inconsistencias en estas cosas:
    - Format and type
    - Link count
    - Duplicate block
    - Bad block numbers
    - Inode size

    Link Count Checks

    Inode Size Checks

    Each inode contains a count of the number of data blocks that it references. The number of actual data blocks is the sum of the allocated data blocks and the indirect blocks.
  </details>
- ¿Cuál es la diferencia a nivel de operaciones, de cuando se hace ls y ls -l?
  <details>
    <summary>Respuesta</summary>
    ls solo debe acceder al inodo del directorio para listar los nombres de los archivos. ls -l, en cambio, accede al inodo de cada archivo para listar sus metadatos.
  </details>
- como implementaría un UNDO de un paso en FAT y en Inodos
  <details>
    <summary>Respuesta</summary>
    El UNDO que te plantea chapa es de un paso, sería hacer ctrl+Z solo una vez. En inodos lo que haces es que cuando vas a sobreescribir el bloque D en realidad haces un copy on write escribiendo en un nuevo bloque E y te guardas el puntero a D. Entonces si queres volver atras esta escritura simplemente restaurás el puntero viejo que te guardaste
    en FAT es parecido pero con los numeros de las entradas de la tabla
    después te pregunta cual FS usarías para hacer el UNDO, no se si hay una respuesta correcta yo me la jugué por inodos
    Copias el bloque antes de modificarlo y cuando reemplazas el puntero en la tabla FAT te guardas el viejo en algún lado
  </details>
- en ext3 recuperaba los datos si habia una caida.
  <details>
    <summary>Respuesta</summary>

    Sacado de aca: https://piazza.com/class_profile/get_resource/il71xfllx3l16f/inz4wsb2m0w2oz

    While recovering after a system failure, the e2fsck program distinguishes the following two cases:
    - The system failure occurred before a commit to the journal. Either the copies of the blocks
    relative to the high-level change are missing from the journal or they are incomplete; in both
    cases, e2fsck ignores them. 
    - The system failure occurred after a commit to the journal. The copies of the blocks are valid,
and e2fsck writes them into the filesystem.

    In the first case, the high-level change to the filesystem is lost, but the filesystem state is still
    consistent. In the second case, e2fsck applies the whole high-level change, thus fixing every
    inconsistency due to unfinished I/O data transfers into the filesystem.
    It is worth noting that journaling ensures consistency only at the system call level. For instance, a
    system failure that occurs while you are copying a large file by issuing several write() system

  </details>

- estructura de un inodo en general
  <details>
    <summary>Respuesta</summary>
    INODO
    Tiene:

    - nro de inodo
    - cant de hard links al file
    - ultima vez que fue modificado/accedido
    - tamaño del file
    - la info del owner
    - el modo, que puede ser file, directorio
    - cant de bloques que usa el file

    Tiene los data blocks, que pueden ser de distintos tipos:
    - Punteros a los bloques de datos directo
    - Puntero con una indireccion (apunta a una table de punteros los cuales si apuntan a bloques de datos)
    - Puntero con 2 indirecciones (idem anterior pero un paso mas de tabla)
    - Puntero con 3 indirecciones (idem anterior pero un paso mas de tabla)
  </details>
  
- Snapshots me dio rienda suelta. Hable de como se implementaban, para que se usaban.
  <details>
    <summary>Respuesta</summary>
    https://unix.stackexchange.com/questions/108131/how-are-filesystem-snapshots-different-from-simply-making-a-copy-of-the-files

    La idea es hacer *copy on write*, para solo copiar los datos del snapshot en el momento que se tengan que modificar.

    Explicacion en silber 590-591 para el caso de WAFL
  </details>
- en EXT2, si ocurre un apagón o lo que sea (y por ende se desmonta el filesystem en medio de una operación) cómo detectarlo y cómo recuperar el filesystem, considerando que en EXT2 todavía no tenemos journalling
  <details>
    <summary>Respuesta</summary>
    For instance, the Ext2 filesystem status is stored in the mount state field of the superblock on disk.
    The e2fsck utility program is invoked by the boot script to check the value stored in this field; if the
    filesystem was not properly unmounted, the e2fsck starts checking all disk data structures of the
    filesystem

    Se pueden hacer 2 tipos de chequeos de consistencia: blocks y files

    Block consistency

    Se arman 2 tablas, una contiene por cada block cuantas veces esta presente en los files, y la otra cuantas veces esta presente en la lista de bloques libres (o bitmap de bloques libres)

    Se recorren todos los inodos viendo sus blocks numbers usados y se incrementa en la primer tabla estos blocks en 1.

    Despues se examina la lista / bitmap de los bloques libres y se va incrementando en la segunda tabla para cada block encontrado en 1 tambien.

    Luego de esto, para chequear si el FS esta consistente, cada bloque deberia tener un 1 en una de las 2 tablas unicamente, osea 0 y 1 / 1 y 0. Si esto no pasa, hay que ver cada caso particular para manejarlo:
    - Los 2 marcan 0 y 0: Esto seria un missing block, que no se usa por el FS, y la solucion es ponerlo como free block.
    - Un block figura 2 veces como libre: La solucion es directa, dejarlo como 1 vez free en la lista (en el bitmap solo puede tener un 0/1 asi que no puede figurar twice)
    - Un block figura 2 veces utilizado: el peor caso, aca habria que crear un bloque nuevo y copiar el contenido del block a ese nuevo, y hacer que uno de los files que esta usando el bloque pase a usar a este nuevo bloque.

    Tambien se chequea la consistencia de directorios. Similarmente, una tabla por inodo que mantiene cuantos files (directory entry) le apuntan. Estos pueden ser 2 o mas debido a los hard links. Una vez hecho esto, se revisa cada inodo y se compara el dato del inodo del hard link count con el que dio en la tabla. Si estos coinciden entonces es consistente el FS, sino no.
  </details>
- EXT2: en que parte se guardan los bloques libres de memoria
  <details>
    <summary>Respuesta</summary>
    Los block groups contienen un bitmap que indica los blocks libres dentro del grupo
  </details>
- Para un backup total, que FS elegirias?
  <details>
    <summary>Respuesta</summary>
    Elegiria a FAT, porque con cargar una vez la tabla de FAT en memoria ya puedo ir mandando a la controladora de disco los pedidos de los distintos bloques de la tabla para hacer el backup. En cambio, si tuviera que hacerlo con inodos, tendria que ir pidiendo recursivamente para cada inodo los inodos de las indirecciones y tomaria mas tiempo ir leyendo esas tablas intermedias, pareceria mas pesado en estructura inodos.
  </details>
- para un incremental, que FS elegirias?
  <details>
    <summary>Respuesta</summary>
    Podemos ir devuelta por FAT, ya que una vez cargada la tabla de FAT, podemos inspeccionar los metadatos de las entradas de directorios y ver cual fue la ultima fecha de modificacion para saber si copiarlo o no.
  </details>
- para un snapshot consistente, con el sistema corriendo, que FS elegirias?
  <details>
    <summary>Respuesta</summary>
    Yo le dije inodos porque tenían una estructura arborea, y le dije cómo funcionaba en un file system que hacía eso, creo que se llama WAFL (está en el silber) y nada, me dijo que estaba bien

    Pensa en los locks

    Y fijate que pasa si en vez de backup total te piden un snapshot en caliente (que el sistema siga corriendo)

    Creo que el de snapshot en cliente de relaciona con el fs con undo

    O sea para esto, fat te bloquea todo el fs. 
    
    Pero si el backup es total esta bien. Si es incremental no, entonces ahí combine inodos.

    El tema es que onda son el snapshot

    Claramente fat no puede ser por lo anterior, bloqueas todo el fs

    Pero que pasa si usas inodos y vas bloqueando por partes?

    Bueno eso funca si impedís escritura en el medio

    Si permitís escritura es más complicado

    No podes grabar X mientras se esta backuoesndo, pero cuando se libera y todavía está haciendo el backup, podes hacer una modificación que afecte a algo que ya se grabó y que todavía no se grabó y ahí metiste una inconsistencia

    Por ahí tendrías que encolar toda modificación a algo que ya se grabó para que pise el snapshot antes de que termine

    Pero entonces podes hacer que el snapshot no termine nunca
  </details>
- que fs para sistema embebido
  <details>
    <summary>Respuesta</summary>
    Parece que FAT

    https://www.hitex.com/fileadmin/documents/sw_components/AN101_choosing_a_file_system_hitex.pdf
  <details>
- Journaling en ext3 logico/fisico. Qué problema resuelve? donde estan los datos commiteados (el log)?
  <details>
    <summary>Respuesta</summary>
    Journaling resuelve el problema de perder los cambios en el FS ante un crash
    
    The basic idea behind a journaling file system is to maintain a journal, which
    describes all file-system operations in sequential order. By sequentially writing out
    changes to the file-system data or metadata (i-nodes, superblock, etc.), the operations do not suffer from the overheads of disk-head movement during random disk
    accesses. Eventually, the changes will be written out, committed, to the appropriate
    disk location, and the corresponding journal entries can be discarded. If a system
    crash or power failure occurs before the changes are committed, during restart the
    system will detect that the file system was not unmounted properly, traverse the
    journal, and apply the file-system changes described in the journal log.

    Physical journals

    A physical journal logs an advance copy of every block that will later be written to the main file system. If there is a crash when the main file system is being written to, the write can simply be replayed to completion when the file system is next mounted.
    Physical journals impose a significant performance penalty because every changed block must be committed twice to storage, but may be acceptable when absolute fault protection is required.

    Logical journals

    A logical journal stores only changes to file metadata in the journal, and trades fault tolerance for substantially better write performance. A file system with a logical journal still recovers quickly after a crash, but may allow unjournaled file data and journaled metadata to fall out of sync with each other, causing data corruption.
  <details>
- cómo hacen ext2 y ext3 para aprovechar las características de un HDD
  <details>
    <summary>Respuesta</summary>
    mantienen los inodos y los bloques de datos de un mismo archivo en un mismo grupo de bloques
  <details>
- ¿Las estructuras a las que apuntan los inodos con indireccion tambien son inodos?
  <details>
    <summary>Respuesta</summary>
    las tablas de single, double y triple indirect y el bloque de datos tambien son inodos pero de otro tipo.
  <details>
- proponer un file system para un dispositivo que se escribe una única vez (y sabes todos los datos que queres escribir) y lecturas múltiples (estilo DVD). Detallar como se puede minimizar el espacio utilizado en el dispositivo.
  <details>
    <summary>Respuesta</summary>
    Me imagino un FS que escribe todo contiguo y ya, no parece mucha ciencia
  <details>

# Seguridad

- Qué problemas de seguridad hay asociados a los file descriptors y cómo se resuelve eso en SELinux (Ahí me mato jajajajajaja) la respuesta era que si uno le quita permisos a un proceso de acceder a cierto archivo mientras este lo tiene abierto, puede seguir accediendo.
- Si yo tengo binarios en una computadora, cómo puedo saber que no fueron modificados?
  <details>
    <summary>Respuesta</summary>
    Puedo tener un hash en MD5 por ejemplo del binario (que no puede ser modificado) y con eso chequear el hash del binario que fue posiblemente modificado para ver si da el mismo hash o no
  <details>
- Ahora si el atacante me modifica el binario y el hash, como puedo solucionar esto? Cómo se resuelve en RedHat?
  <details>
    <summary>Respuesta</summary>
    En este caso tendria que usar algo en el proceso de hasheo para certificar que el hash es el original del binario. Esto podria hacerlo a traves de un approach como en firma digital, teniendo un hash encriptado con la clave privada del owner del binario, lo cual solo desencripto con la publica. De esta manera si el atacante quisiera suplantar el hash deberia tener la clave privada para que luego use yo la publica y me de el mismo resultado que el hash del binario, sino va a fallar la desencriptacion.
  <details>
  
- como se usa la funcion de hash en el login de un SO. q problemas tiene usar un hash para autenticar usuarios?. como haría para mejorar la seguridad?. que otros usos tiene la funcion de hash? 
  <details>
    <summary>Respuesta</summary>
    problema de usar hash -> se puede bruteforcear, habia q hablar de salt y de aplicar varias veces la función de hash

    mejorar seguridad -> aumentar la cantidad de veces q se aplica el hash

    otros usos de la funcion hash -> integridad

    OS linux/unix: salted (8 random alphanumeric character) hash that's key-strengthened about 5000 times using a simple cryptographic hash like sha256/sha512. 
  <details>
- como se encriptan las contraseñas, de qué exploit te protege SALT y me pidió un detalle más (acá patiné) que es que en linux hay una técnica adicional que es rehashear la contraseña varias veces para dificultar que un atacante pueda hacer un ataque de fuerza bruta.
  <details>
    <summary>Respuesta</summary>
    Las contraseñas se encriptan utilizando una funcion de hash, de manera que sea dificil determinar la contraseña original, ya que un hash es one way (SHA-1, MD5)
    
    SALT te protege de un ataque de tabla de hash, porque este tiene una tabla con contraseñas comunes y sus hashes, pero al intentar comparar alguno de estos con las contraseñas de la base de datos no le va a servir, porque todas las contraseñas tienen un string agregado (SALT), entonces tienen que descifrar tambien el SALT o probar distintos para ver si le pegan a la correcta combinacion para que el hash final llegue a donde quieren. Para que esto sea efectivo el SALT deberia ser diferente para cada password, si es la misma no tiene mucha gracia

    por otro lado tenemos la tecnica de key stretching que consiste en construir una strengthened key, la cual se genera aplicando multiples veces el hash sobre una contraseña, dificultandole al que quiera romperla el trabajo, porque va a tener que tomar por ejemplo en el caso del hash table attack cada contraseña comun y rehashearla muchas veces
  <details>
- DAC Y MAC
  <details>
    <summary>Respuesta</summary>
    COMPLETAR
  </details>
- RSA
  <details>
    <summary>Respuesta</summary>
    Es un algoritmo de encriptacion asimetrico, donde la idea es que se tienen una clave publica y privada. 
    La publica la conocen todos y la privada solo el dueño. 
    Con esto, si el de la clave privada quiere pedirle algo a otro con su publica, le manda un mensaje encriptado con su privada.
    Con esto el que lo recibe sabe que el dueño del mensaje es el de la privada porque lo desencripta con la publica, con lo que se conserva integridad y autoridad?
    Luego este le responde encriptando con la publica, y solo el dueño va a poder desencriptar eso con la privada, entonces el mensaje es confidencial.
  </details>
- HTTP y HTTPS, más que nada las diferencias principales y como HTTPS nos garantiza mayor seguridad.
- como te puede complicar un atacante si usas http para el tema de las actualizaciones, suponiendo que las mismas estas firmadas digitalmente

  <details>
    <summary>Respuesta</summary>
    Se mete en el medio y te dice que no hay actualizaciones, entonces no actualizas mas el SO y quedas vulnerable
  </details>
- cómo evitar buffer overflow (acá me pidió alguna forma más además de canary que no me acordaba, me dijo que marcar páginas como no ejecutables).
  <details>
    <summary>Respuesta</summary>
    Se pueden utilizar los stack canaries. La idea de estos es que luego de pushear la direccion de retorno al stack se pushea un valor generado al azar llamado canary. El proposito de esto es que si llega a haber un buffer overflow y se intenta pisar la direccion de retorno se va a pisar a su vez el canary. Entonces al momento de retornar de la funcion, antes de utilizar la direccion de retorno se chequea que el canary no haya sido modificado, y si lo fue, se aborta la ejecucion del programa.

    te sirve principalmente cuando te quieren escribir el ret de una funcion, y que se valida cada vez que estas volviendo de la funcion. 
    
    Hablando que la parte del canary es un agregado del compilador
  </details>
- certificados, firma digital, autoridades certificantes.
  <details>
    <summary>Respuesta</summary>
    Firma digital: La idea es que queremos enviar un mensaje grande, tipo documento, y que la otra persona tenga certeza de quien es el autor del mensaje.
    Podemos hacer esto con RSA, pero es muy costoso el algoritmo sobre un documento entero. Entonces la idea es que en lugar de encriptar todo el documento, se encripta un digest del mismo, que seria un hash. Luego se envia el conjunto (hash encriptado, documento original, clave publica)
    El que lo reciba puede desencriptar el hash con la publica, hashear el documento y comparar que estos 2 son iguales. Si esto ocurre, entonces el documento es el original del que lo envio, sino no.

    Certificado digital: es una forma de compartir una clave publica certificada que identifica una entidad que es dueña de la misma.

    Autoridad certificante: son los que emiten los certificados digitales y son una entidad confiable.

    Digital certificates are a proof of an endpoint’s authenticity, like a server or a user. For example, if a browser requests a website, how do we know that the page that’s returned to us is the genuine one? Digital certificates provide the stamp of genuineness by binding the public key with the entity (server or client) that owns it, provided the entity possesses the corresponding private key. Digital certificates are issued by a Certificate Authority (CA).

    A digital certificate contains the name of the certificate holder, a serial number, expiration dates, a copy of the certificate holder’s public key (used for encrypting messages and digital signatures) and the digital signature of the certificate-issuing authority (CA) so that a recipient can verify that the certificate is real
  </details>
- Como el so valida los certificados de autenticacion para los updates.
- cómo implementar distribución de actualizaciones para mi propia distribución de ubuntu
  <details>
    <summary>Respuesta</summary>
    dije que usando sistema de clave publica/privada + hashing para garantizar autoridad e integridad de los paquetes de actualizaciones, porque no quiero que alguien distribuya paquetes de actualizacion que no son mios y quiero garantizar que le llegue lo que le mandé a los usuarios)
    Me preguntó a quién le pediría esa clave privada/pública (dije a alguna autoridad certificante) y cómo distribuiría la pública (pasandoselas cuando se bajen mi distro o poniendola en alguna especie de programa actualizador de mi distro, es pública así que no importa mucho)
  </details>
- encriptacion simetrica/asimetrica, en asimetrica explicar clave pública/clave privada
  <details>
    <summary>Respuesta</summary>
    Simetrica: los mensajes se encriptan y desencriptan con una misma clave

    Asimetrica: los mensajes se encriptan y desencriptan con distintas claves
  </details>
- SQL injection
  <details>
    <summary>Respuesta</summary>
    Es una forma de mandarle queries SQL a la base de datos que esta detras de una aplicacion, mediante el input que nos permite ingresar la app
    Uno termina recibiendo un input que es una query SQL y termina ejecutando sin saberlo esa query en la DB, potencialmente borrandola toda
  </details>
- que es una función de hash?
  <details>
    <summary>Respuesta</summary>
    Es una funcion que transforma un input en un valor tal que no es posible obtener a partir de este el input original. La idea de la funcion de hash es que no se pueda obtener la preimagen (el input original) y tampoco otro input tal que produzca el mismo valor (libre de colisiones)
  </details>
- para que sirve el bit executed de la tabla de paginas y que puede hacer el atacante que no pudo ejecutar su codigo porque el bit estaba activado para agitarla igual
  <details>
    <summary>Respuesta</summary>
    Con el  bit NX (No eXecute) podemos evitar que se ejecute shell code luego de que se hace un buffer overflow inyectandolo

    El atacante igualmente puede pisar el return para volver a la libc y, por ejemplo, hacer la llamada para obtener una terminal
  </details>
- Cómo aseguras que un driver sea confiable?
  <details>
    <summary>Respuesta</summary>
    Lo podes firmar digitalmente.

    por ejemplo, en Windows, Microsoft puede firmarlo
  </details>
- Qué es una firma? Cómo se hace? Qué hace el receptor cuando quiere comprobar la firma?
  <details>
    <summary>Respuesta</summary>
    Una firma asegura la integridad de lo que se esta firmando, osea que nadie puede modificarlo sin que lo sepamos. Con esto se logra autenticidad y autorizacion. Podemos saber DE QUIEN ES y SI FUE MODIFICADO.
  </details>
- Se puede considerar Deadlock un problema de seguridad?
  <details>
    <summary>Respuesta</summary>
    Si, porque si esto ocurre el sistema se trabaria y estaria ocurriendo una denegacion de servicio (DoS) ya que los usuarios no podrian acceder al sistema por ejemplo.
  </details>
- Como podria un cliente A conectarse a un servidor B (podia usar publica y privada A)
- diferencia entre el comando su y sudo
  <details>
    <summary>Respuesta</summary>
    The main difference between Su and Sudo is that the Su command can interchange between superuser and root user if executed without prior additional options while the Sudo command provides single root privileges. Su demands the password of the root account while Sudo demands the password of the current user account.
  </details>
- por qué Windows requiere que los drivers tengan una firma digital y de qué otra manera se podría lograr el mismo objetivo
- SETUID
  <details>
    <summary>Respuesta</summary>
    Es un bit que se encuentra en los binarios que si se encuentra en 1 indica que al correr ese binario, este va a tener los permisos del dueño del mismo.
    Esto sirve por ejemplo para cambiar las contraseñas en linux, ya que uno como usuario no tiene los permisos suficientes para hacerlo, pero puede correr el programa de cambio de contraseñas que tiene el SETUID prendido y superuser como owner, el cual va a permitir cambiar la contraseña ya que tiene los permisos de superuser al ser ejecutado.
  </details>
- ataque que incluya a setUID junto con buffer overflow
  <details>
    <summary>Respuesta</summary>
      Podria aprovechar que un binario con setuid tiene maximo privilegio, entonces si puedo encontrar una forma de hacerle un buffer overflow a eso, y por ejemplo pisarle el return de la funcion para que pase a ejecutar un shellcode que me levante una terminal, obtendria asi control total, ya que la terminal tendria privilegio de nivel 0 por haber sido lanzada desde este.
      Tambien puedo hacer un return to libc y usar llamadas del sistema con maximo privilegio
  </details>
- buffer overflow
  <details>
    <summary>Respuesta</summary>
      Buffer overflow se da cuando un proceso intenta escribir posiciones de memoria que estan por afuera de la capacidad de un buffer, pisando asi potencialmente variables del stack, la direccion de retorno de la funcion, etc.
  </details>

# Sistemas Distribuidos

- 2pc, 3pc, problema bizantino
  <details>
    <summary>Respuesta</summary>
    https://iberasync.es/2pc-two-phase-commit-3pc-three-phase-commit-protocol/

    2PC

    Es un protocolo utilizado para realizar una transaccion atomica en un algoritmo distribuido entre todos los procesos que participan de la transaccion atomica distribuida.
    Tiene 2 fases (https://docs.particular.net/nservicebus/azure/two-phase-commit.png):
    - Fase de votacion: el coordinador de la transaccion le envia a todos los participantes un pedido para que voten hacer el commit o abortar
    - Fase de Commit: Basado en el resultado de la votacion, se hace el commit si todos los participantes dijeron si o se aborta si 1 o mas dijeron no


    3PC

    El objetivo es el mismo que 2PC, lograr un consenso en transacciones de sistemas distribuidos, pero no ser *blocking*.
    Lo que se hace es modificar la commit phase en 2 phases (https://miro.medium.com/max/640/0*Zbi_zo67uAq1J7pt.png):
    - Prepare to commit: el coordinator le pregunta a los participantes si estan preparados para hacer commit. En esta parte los participantes adquieren los locks necesarios pero no commitean
    - Si el coordinator recibe yes de parte de todos los procesos que se prepararon para commitear, entonces les dice que commiteen.

    Esto permite que sea mas facil recuperarse de problemas si el coordinator muere y actua un nodo de coordinator. Si el nuevo coordinator tiene que saber como quedo todo, puede preguntar y ver si quedaron en la commit phase, osea que el coordinator los habia preparado a todos para commitear en la prepare to commit phase. Por otro lado, si algun nodo dice que no recibio el mensaje de prepare to commit, podemos asumir que ningun participante commiteo los cambios todavia porque recien se estaban preparando, y podemos abortar la transaccion sin dejar inconsistencias.

    Problema bizantino

    El problema dice que todos los procesos que pueden tener un valor 0 o 1 inicialmente, tienen que ponerse de acuerdo en un numero finito de pasos (WAIT-FREEDOM) en un valor comun. Dependiendo de las posibles fallas que tenga un nodo, este problema puede o no tener solucion.

  </details>

- token ring, los problemas de token ring y como mejorarlos
  <details>
    <summary>Respuesta</summary>
    
    Token ring es una forma de tener exclusion mutua en un conjunto de n nodos organizado en manera de ring
    La idea es que se esta pasando constantemente un token por el ring, y el que recibe el token tiene 2 opciones:
    - lo pasa al siguiente
    - lo retiene para hacer algo que requiere exclusion mutua y luego lo vuelve a pasar

    Los problemas que tiene esto es que el token esta constantemente circulando por la red, agregando un overhead innecesario a los nodos que tienen que son interrumpis por esto. Otro problema que tiene es que si falla el nodo con el token perdi el token.

    Mejora del anterior, donde:
    - cuando un proceso quiere entrar a la seccion critica le envia al resto (incluyendose) una solicitud (Pi, ts), donde Pi es su pid y ts es el timestamp actual.
    - cada proceso cuando recibe la solicitud responde o encola la respuesta.
    - el proceso que solicito solo entra si recibe todas las respuestas
    - si logro entrar cuando salgo le digo a respondo a todas las solicitues que pueden entrar (porque tengo que esperar mi turno para entrar devuelta).
    - cuando se recibe un mensaje, contesto bajo estas condiciones:
      - si no me interesa entrar
      - quiero entrar, no entre todavia y el ts de la solicitud que recibi es menor a la que yo hice, entonces tiene prioridad.
    - para que este algoritmo funcione necesita conocer a todos los demas para enviarle solicitudes, y tambien requiere conocer el orden entre los eventos, es decir, sincronizar los timestamps en la red.
      
    Esta version tiene una pro con respecto a la v1, que es que no circulan mensajes de procesos que no quieren entrar a la seccion critica.

    Para que todo esto funcione se necesita:
    - No perder mensajes
    - Ningun proceso falla
  </details>
- consenso y sincronización también.
- clusters y grid y como están implementadas
  <details>
    <summary>Respuesta</summary>
    Clusters: Son un conjunto de computadoras que trabajan juntas como un solo sistema. Cada nodo (computadora) hace la misma tarea y esta controlado y scheduleado por el mismo software. Los nodos estan conectados en una red rapida, con cada nodo con su OS y mismo hardware.

    Grids: Es un sistema distribuido donde cada nodo hace su propio tarea. No suelen estar todas conectadas como en un cluster, sino que estan dispersas geograficamente
  </details>
- que es un sistema distribuido, que beneficios trae. Qué problemas nos puede traer?
  <details>
    <summary>Respuesta</summary>
    SISTEMA DISTRIBUIDO

    Definicion:
    - Varias maquinas conectadas entre si en una red
    - Tambien se denomina sistema distribuido a un procesador con varias memorias
    - O varios procesadores y varias memorias

    Lo importante de esto es que los sistemas distribuidos se caracterizan por 2 aspectos primordiales:
    - Los procesadores no comparten clock, entonces no existe la idea de ponerse de acuerdo para hacer algo a cierta hora
    - Ningun procesador tiene toda la informacion, sino que cada procesador tiene info parcial

    tiene 2 beneficios: velocidad de computo por el "paralelismo" y descentralizacion del computo, si se cae un nodo el servicio sigue en pie, el uso la palabra REDUNDANCIA

    tiene varios problemas: envío de mensaje es lento, no es fácil mantener consistencia y coordinar a las partes, dificultad de acceder a un recurso compartido
  </details>
- en NFS, cómo saben los nodos a que servidor pedirle los datos.
- NFS es distribuido? cómo implementaría distribución de verdad y consistente.
  <details>
    <summary>Respuesta</summary>
    NFS le permite a un numero arbitrario de clientes y servidores compartir un FS en comun
    Los clientes y servidores se piensan en maquinas distintas, pero se pueden tener maquinas que son al mismo tiempo client y server

    Los servers exportan 1 o mas directorios internos para qeu lo accedan clients remotos
    - Ademas de exponer el directorio se exponen todos los subdirectorios de este, entonces se expone como una UNIT todo
    Para que un client acceda a uno de los directorios exportados, tiene que *montarlo* en su FS, y asi este forma parte de su arbol de directorios
    IMPORTANTE: el server NO SE ENTERA donde esta montado por los clients

    No es un distributed file system como hoy en dia porque toda la data de un file system se encuentra en un solo server. Lo que hace es permitirle a varios clients aceder a los mismos files que se encuentran en un mismo server, lo cual se denominaria shared file system.
    Uno verdadero DFS es Ceph por ejemplo, que tiene sus datos distribuidos en multiples servidores, manejando redundancia a traves de la replicacion de files entre otras cosas.
    Con esto se evita el problema de que se si cae un server se cae todo el FS, lo cual en NFS puede pasar y no hay server de backup por tener todo en un solo lugar

    para que conste: toda la información está en un servidor central y esa información es la que se distribuye a los demás nodos, entonces podemos decir que "no esta verdaderamente distribuído". Si se cae el servidor central, nos quedamos sin servicio

    expliqué 2/3pc, ya que nos permite realizar operaciones de forma consistente en todos los nodos
  </details>
- NFS: Supongamos que lo expandimos del lado del servidor y queremos generar redundancia. Ahora tenemos N servidores que ofrecen el NFS y los tengo que mantener sincronizados, como hago?
  <details>
    <summary>Respuesta</summary>
    Esta ya la habia visto, le propuse que por cada transaccion hecha podiamos hacer un 2PC para que todos se enteren
  <details>
- en qué caso particular falla 2PC
  <details>
    <summary>Respuesta</summary>
    CHEQUEAR BIEN LOS CASOS DE FALLAS, EXCEPTO EL CLASICO BLOCKING, ESE SEGURO ESTA BIEN

    Puede ocurrir que se este en la Commit Phase y no se tenga respuesta de uno de los nodos para saber si hizo el commit o aborto, entonces quedamos con una posible inconsistencia.

    https://www.geeksforgeeks.org/recovery-from-failures-in-two-phase-commit-protocol-distributed-transaction/

    Puede fallar tanto un nodo participante como el coordinador

    Nodo participante

    - Este puede fallar antes de responder con yes/no, en cuyo caso el coordinador asumiria que el mensaje es no
    
    - Puede fallar luego de decir que si, entonces el coordinador va a ignorar su falla y va a seguir ejecutando el protocolo 2pc. Cuando el nodo se recupere de su falla va a tener que revisar su log para saber que hacer (se contesta en la 3ra pregunta abajo de esta)

    Coordinador

    Los nodos tienen que ver los logs de los demas nodos para saber que hacer:
    - Si todos los nodos tienen el mensaje de commit, entonces se commiteo la transaccion
    - Si todos los nodos tienen el mensaje de abort, entonces se aborto la transaccion
    - Si algunos nodos no tienen el mensaje de yes en su log, entonces el coordinador cuando se recupere no va a saber si commitear o abortar porque hubo nodos que no le respondieron con si/no, entonces es mejor abortar la transaccion en lugar de bloquearse
    - En el caso de que todos los logs digan si, entonces hay que esperar al coordinador que se recupere para ver si aborta o commitea, en cuyo caso todos los nodos se encuentran en *BLOCKING*
  <details>
- weak/strong termination en 2PC/3PC.
  <details>
    <summary>Respuesta</summary>
   The weak termination condition says
    that if there are no failures then all processes eventually decide. The strong
    termination condition (also known as the non-blocking condition) says that
    all nonfaulty processes eventually decide. 

    Commit algorithms that satisfy the strong termination condition are sometimes
    called non-blocking commit algorithms, while commit algorithms that satisfy the
    weak termination condition but not the strong one are sometimes called blocking
    commit algorithms. 

    2PC satisface weak termination

    3pc satisface strong termination
    
  <details>
- en token ring, que harias si el token se pierde?
  <details>
    <summary>Respuesta</summary>
    termine usando elección de lider y parece q era esa la respuesta
  <details>
- en 2PC, como se sincroniza un nodo participante que se cayo despues de aceptar la transación pero antes de recibir el commit/abort.
  <details>
    <summary>Respuesta</summary>
    CHEQUEAR BIEN EL TEMA DEL LOG

    Para sincronizarse tiene que chequear su registro de logs que dice lo que paso despues:
    - Si ve que le llego un mensaje de commit, entonces commitea los cambios
    - Si llego un mensaje de abort entonces no aplica los cambios
    - Finalmente si tiene un mensaje de ready tiene que consultarle al coordinator para saber en que quedo el estado de la transaccion
  <details>
- Cómo convertir un sistema no-distribuido a distribuido
  <details>
    <summary>Respuesta</summary>
    
    Para pasarlo de no distibuido a distribuido podemos pasar a poner los distintos componentes del sistema en varios computadoras y que estan se sincronicen entre ellas
  <details>
- En NFS,  Si un nodo se cae, como hacemos para que se entere despues de las transacciones que no tiene?.
  <details>
    <summary>Respuesta</summary>
    o sea si queda desactualizado, le dije que con un journal y que cuando conecte pregunte los ultimos cambios o algo asi
  <details>
- Como podria hacer para poder tener 2 discos distribuidos, y que los dos contengan la misma informacion, para usarlo de redundancia. Ahi hablamos de como se podia manejar esto para que no tenga fallas, si se caia la conexion en el medio y demas
  <details>
    <summary>Respuesta</summary>
    Esto suena a usar 2PC para poder commitear los mismos cambios en ambos discos y mantenerlo sync
    En caso de problemas en la conexion podemos ver como se manejan los casos de 2PC
  <details>
- explicar diferencias entre sistema paralelo y distribuidos
  <details>
    <summary>Respuesta</summary>
    La principal diferencia es que un sistema paralelo tiene una computadora con multiples procesadores comunicandose entre ellos a traves de memoria compartidao o bus de datos, mientras que un sistema distribuido tiene varias computadores con sus procesadores comunicandose mediante la red, intercambiando mensajes, sin compartir memoria ni clock.
  <details>
- En 2PC ¿Que fallas son bloqueantes?
  <details>
    <summary>Respuesta</summary>
    Los participantes dijeron todos si y estan esperando una respuesta sobre si hacer el commit o no se del coordinador
    
    Se quedan bloqueados hasta que llega tal respuesta mientras que mantienen los recursos asociados al consenso, y si el coordinador murio nunca les va a llegar y van a tildarse. 
  <details>
- Problema sobre un algoritmo de commit distribuido:
un algoritmo de commit distribuido funciona de la siguiente manera: opera sobre una red donde los paquetes pueden perderse o demorarse, y cuando un nodo quiere escribir sobre cierto archivo que está replicado en todos los nodos, envía un pedido de escritura. Si recibe confirmación de todos los nodos, escribe y le notifica a los demás que pueden hacer lo propio. Alguien nota que este algoritmo puede fallar si los nodos se caen entre la escritura y la notificación y propone para solucionarlo el envío de mensajes de confirmación de parte de los nodos. ¿Este algoritmo resuelve el problema? Justifique. 
    <details>
      <summary>Respuesta</summary>
      No lo resuelve del todo porque existe el problema del acuerdo bizantino. No importa cuantos intercambios de mensajes haya entre los nodos, nunca va a poder saberse si les llego el mensaje de confirmacion a todos.
    </details>
# Virtualizacion
- virtualizacion y containers (que son y como se implementan) 
  <details>
    <summary>Respuesta</summary>
    
    Virtualizacion

    Permite corre diferentes sistemas operativos dentro de la misma maquina.

    Containers

    Permiten correr varias aplicaciones que utilicen el mismo sistema operativo en una maquina con otro sistema operativo.
  </details>