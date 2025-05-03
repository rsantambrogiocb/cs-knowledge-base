# Running on a cluster
To run a spark application on a cluster you need to use the command spark-submit. 
This script needs some argument:
- `- class <class.object.of.the.MainFunction>`
- `- jars <paths for additional dependencie>`
- `- files <files to put alongside the application and they should be small like a little look up file>`

Then it’s up to spark itself to figure out what cluster manager you’re running on top of (spark built-it manager, Hadoop YARN, Mesos) and work with it to distribute the processing. 
Basically, the spark **driver script is running on the master node and comunicate with the cluster manager to distribute out the work to different executors** (which are on worker machines).
The **cluster manager is also responsible for dealing with single node failures and to collect the result** back together to the driver script when it’s done. 

**Other spark-submit parameters:**
- master: it just to be set to the right thing for what kind of cluster we have; 
	- if we’re running with YARN, this parameter will be set to “yarn”
	- For connecting to a master on a standalone spark cluster you need to specify “hostname:port” as values
	- If Mesos is in use, the value needs to be set “mesos://masternode:port”
- num-executors: specify how many executor nodes you want to use and by default it’s set to 2
- executor-memory: manages how much memory is available to each executor (make sure that it does not exceed the available phisical memory in each individual executor node)
- total-executor-cores: if you have multi-cores on your virtual nodes then you might want to tweak that to put an upper limit to how many cores your script can consume

If we have a configuration in our script, it will ignore what’s on the command. The hierarchy is: what’s in your script, what is specified in the command line and as less important whatever is in the configuration files for spark. 
So.. never hard code a master in your script.

Keep in mind that local[*] use all the CPU available in the machine to parallelise the processing.

Defining the number of executors, their cores and memory
https://stackoverflow.com/questions/24622108/apache-spark-the-number-of-cores-vs-the-number-of-executors
https://stackoverflow.com/a/37709689

# Deploy mode
### Spark Client Mode
Adatto per applicazioni interattive, debugging e sviluppo. 
Il **driver Spark viene eseguito sulla macchina da cui viene avviata l'applicazione (client)**.
Solo gli executor vengono eseguiti sui nodi del cluster.
Il **client deve rimanere attivo per tutta la durata del job Spark**.
L'*output* dell'applicazione viene *visualizzato direttamente sul terminale del client*
### Spark Cluster Mode
**Sia il driver Spark che gli executor vengono eseguiti sui nodi worker del cluster.**
Il **client può disconnettersi dopo aver sottomesso l'applicazione**
*I log dell'applicazione vengono archiviati nel cluster*, non sul client
Più adatto per ambienti di produzione e per job di lunga durata.
### Differenze principali

#### Posizione del driver:
Client Mode: il driver è sulla macchina client
Cluster Mode: il driver è su un nodo del cluster
#### Tolleranza ai guasti:
Client Mode: se il client si interrompe, l'applicazione fallisce
Cluster Mode: l'applicazione continua anche se il client si disconnette
#### Risorse:
Client Mode: le risorse del driver utilizzano la macchina client
Cluster Mode: tutte le risorse sono allocate all'interno del cluster
#### Casi d'uso:
Client Mode: sviluppo, debug, notebook interattivi (Jupyter, Zeppelin)
Cluster Mode: job di produzione, elaborazioni batch, job pianificati

Questa distinzione è fondamentale per ottimizzare le prestazioni e la resilienza delle tue applicazioni Spark in base al loro scopo.

## Considerazioni specifiche per EMR:
In EMR, il nodo master è generalmente utilizzato per eseguire il driver in cluster mode, mentre i nodi core e task vengono utilizzati per gli executor.
Il client mode in EMR esegue il driver sul nodo da cui sottometti il job (spesso il nodo master stesso, se ti connetti tramite SSH).

**Best practice su EMR**:
Cluster mode è generalmente consigliato per job di produzione e batch
Client mode è utile per sviluppo interattivo, debugging o quando usi EMR Notebooks