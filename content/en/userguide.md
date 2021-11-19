---
title: "User Guide"
---
Once the user is able to log into purpleJeans (Connect to Resources), upload and access their files on the cluster (Data Storage and Transfer), and upload software tools using the (Software) module system, you are ready to scheduling access to the compute cluster to perform calculations.

# Introduction
The purpleJeans compute cluster is a shared resource used by the entire RCF community. Sharing computational resources creates unique challenges:

* Jobs must be managed (job scheduler) equally for all users.
* Resource consumption must be logged.
* Access to resources must be controlled.

The purpleJeans compute cluster uses a scheduler to handle requests for access to compute resources. These requests are called jobs. In particular, Slurm's resource manager is used to schedule jobs, as well as interactive access to compute nodes.

Slurm is an open source, fault tolerant, and highly scalable system for cluster management and job sheduling for large and small Linux clusters. Slurm does not require kernel modifications for its operation and is relatively self-contained. As a cluster workload manager, Slurm has three key functions. First, it gives users exclusive and / or non-exclusive access to resources (compute nodes) for a certain period of time, so that they can run jobs. Second, it provides a framework for starting, running, and monitoring jobs (usually jobs to run in parallel) on the set of allocated nodes. Finally, it arbitrates contention for resources by managing a queue (partitions) of pending jobs. Optional plug-ins can be used for resource usage accounting, advanced booking, group scheduling (sharing time for parallel jobs), backfill scheduling, topology-optimized resource selection, boundaries of resources per user or account and sophisticated multi-factor job prioritization algorithms.

Below is the essential information you need to know to get started on purpleJeans. For more detailed information on running specialized compute jobs, see Running Jobs on purpleJeans (link).

# Computing nodes types
The purpleJeans compute cluster consists of compute nodes with different architectures and configurations. A partition is a collection of compute nodes that all have the same or similar architecture and configuration. Currently, purpleJeans has the following partitions:

| Resources           | Partitions | Nodes | CPU/Node                                             | Core/Node | Memory/Node   | Notes                                                                       |
|-------------------|------------|------|------------------------------------------------------|-----------|----------------|-------------------------------------------------------------------------------------------------------------------------|
| purpleJeans (2019)| xhicpu     |    4 | 2 CPU Intel(R) Xeon(R) Xeon 16-Core 5218 2,3Ghz 22MB |        32 |         192 GB | Cluster dedicato alla ricerca nell’ambito del machine learning e big data. Mellanox CX4 VPI SinglePort FDR IB 56Gb/s x16. |
|                   | xgpu       |    4 | 2 CPU Intel(R) Xeon(R) Xeon 16-Core 5218 2,3Ghz 22MB |        32 |         192 GB | 4 GPU  (NVLINK) NVIDIA Tesla V100 32GB SXM2. Infiniband Mellanox CX4 VPI SinglePort FDR IB 56Gb/s x16. |

It is possible to have a summary of the partitions on purpleJeans using the sinfo command:

```sh
$ sinfo shared
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
xhicpu*      up   12:00:00      4  alloc wnode[01-04]
xgpu         up    6:00:00      1    mix gnode02
xgpu         up    6:00:00      1  alloc gnode03
xgpu         up    6:00:00      2   idle gnode[01,04]
```

In the summary obtained with sinfo shared, the “NODES” column gives the total number of nodes in each partition. This summary also lists the partitions reserved for use by certain laboratories.

# Interactive jobs
After submitting an interactive job on purpleJeans, Slurm will connect you to a compute node and load an interactive shell environment for use on that compute node. This interactive session will persist until disconnected from the compute node or until the maximum time required is reached. The default required time is 2 hours.

## srun
An alternative to sinteractive is the srun command:

```sh
$ srun --pty bash
srun: job 2808 queued and waiting for resources
```

Unlike sinteractive, this command does not set X11 forward, which means that it is not possible to display graphics using srun. Both the srun and sinteractive commands have the same options. During the execution of an interactive task, you have access to your home and consequently to all installed libraries. An example of using srun with various options is as follows:

```sh
$ srun --ntasks=1 \
> --nodes=1 \
> --partition=xhicpu \
> --cpus-per-task=8 \
> --mem=4000 \
> --pty \
> bash
```

| Opzione            | Descrizione                                                 |
|--------------------|-------------------------------------------------------------|
| --ntasks=1         | Un solo task da eseguire                                    |
| --nodes=1          | Richiede un unico nodo                                      |
| --partition=xhicpu | Richiede i nodi di calcolo presenti sulla partizione xhicpu |
| --cpus-per-task=8  | Richiede di allocare 8 CPU per ogni task                    |
| --mem=4000         | Richiede di utilizzare un totale di 4000 MB (4 GB)          | 
| --pty              | Esegue il task in modalità terminale                        |
| bash               |Eseguibile da lanciare, in questo caso il comando bash       |

In this example, only the wnode01 node will be used, requiring a limited number of resources (44GB of memory and 8 CPUs for 1 task), this allows the parallel execution of multiple jobs (without distinction between srun and sbatch)

## Job di tipo Batch
Il comando sbatch è il comando più comunemente usato dagli utenti RCF per richiedere risorse di calcolo sul cluster purpleJeans. Anziché specificare tutte le opzioni nella riga di comando, gli utenti in genere scrivono uno “script sbatch” che contiene tutti i comandi e i parametri necessari per eseguire il programma sul cluster.

In uno script sbatch, tutti i parametri di Slurm sono dichiarati con #SBATCH, seguito da ulteriori definizioni.

Ecco un esempio di uno script sbatch:

```sh
#!/bin/bash
#SBATCH --job-name=example
#SBATCH --output=example.out
#SBATCH --error=example.err
#SBATCH --time=00:10:00
#SBATCH --partition=xhicpu
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=16
#SBATCH --mem-per-cpu=2000
 
# Set stack size (to avoid warning message)
ulimit -s 10240

# Load OpenMPI module
module load ompi-4.1.0-gcc-8.3.1

# Run
mpirun ./hello-mpi
```

Di seguito i dettagli dei comandi:

| Opzione                     | Descrizione                                                                           |
|-----------------------------|---------------------------------------------------------------------------------------|
|#SBATCH --job-name=example   |Assegna il nome example al job                                                         |
|#SBATCH --output=example.out |Scrive l’output della console nel file example.out                                     |
|#SBATCH --error=example.err  |Scrive il messaggio di errore nel file example.err                                     |
|#SBATCH --time=00:10:00      |Riserva le risorse richiesta per 10 minuti (o anche meno se il processo termina prima) |
|#SBATCH --partition=xhicpu   |Richiede i nodi di calcolo presenti sulla partizione xhicpu                            |
|#SBATCH --nodes=4            |Richiede 4 nodi di calcolo                                                             |
|#SBATCH --ntasks-per-node=16 |Richiede 16 CPU per ogni singolo nodo                                                  |
|#SBATCH --mem-per-cpu=2000   |Richiede 2000 MB (2 GB) di memoria RAM per singola CPU                                 |

In questo esempio, abbiamo richiesto 4 nodi di calcolo con 16 CPU ciascuno. Pertanto, abbiamo richiesto un totale di 64 CPU per l'esecuzione del nostro programma. Le ultime due righe dello script caricano il modulo OpenMPI e avviano l'eseguibile basato su MPI che abbiamo chiamato hello-mpi (vedi job MPI).

Continuando l'esempio sopra, supponiamo che questo script sia salvato nella directory corrente in un file chiamato example.sbatch. Questo script viene inviato al cluster usando il comando seguente:

```sh
$ sbatch ./example.sbatch
```

Molte altre opzioni sono disponibili per l'invio di lavori utilizzando il comando sbatch. Per esigenze computazionali più specializzate, consultare Esecuzione di job su purpleJeans (link). Inoltre, per un elenco completo delle opzioni disponibili, consultare la documentazione ufficiale SBATCH (link: https://slurm.schedmd.com/sbatch.html ).

**Nota:**
Impostare il size dello stack è buona norma per evitare messaggi di warning in runtime.
```ulimit -s 10240```

# Storage temporaneo
Molte applicazioni generano file temporanei o intermedi scritti in /tmp. (Queste applicazioni possono scrivere file su /tmp anche senza che tu sappia che ciò sta accadendo.) Questa cartella si trova in genere su un'unità locale o sul disco RAM virtualizzato nella memoria di sistema.

- I contenuti in /tmp lasciati dal lavoro di un utente non verranno automaticamente eliminati prima di riavviare il nodo corrispondente, e pertanto potrebbero influire su altri lavori eseguiti in seguito sullo stesso nodo. Pertanto, RCF applica una politica di eliminazione dei dati per i file scritti in / tmp su nodi di calcolo:

- Per ciascun job in esecuzione, viene creata una cartella /tmp/jobs/${SLURM_JOB_ID}, leggibile e scrivibile solo dal job, su ciascun nodo assegnato. Il suo contenuto viene eliminato in modo sicuro solo al termine del job (quando viene completato, annullato o ucciso con successo).

- Per tutti i job in esecuzione, le variabili di ambiente SLURM_TMPDIR e TMPDIR sono impostate su /tmp/jobs/${SLURM_JOB_ID}. Quando possibile, gli utenti dovrebbero scrivere nei percorsi specificati da queste variabili d'ambiente anziché usare esplicitamente /tmp. (La maggior parte delle applicazioni dovrebbe già utilizzare queste variabili di ambiente per impostazione predefinita, quindi in molti casi ciò non richiede alcuna modifica al codice.)

- Oltre a utilizzare $TMPDIR, gli utenti dovrebbero anche verificare che non vengano scritti altri file su /tmp.

- Notare che al termine di un job, eventuali cartelle o file direttamente in /tmp che appartengono al mittente di questo lavoro verranno eliminati.

- I contenuti di /tmp non persistono al termine dei lavori. RCF non è responsabile per il recupero o il recupero dei dati memorizzati lì. Per gli output critici, salvarli nei sistemi di archiviazione dei file persistenti; vedere Archiviazione e trasferimento dati.

**Importante:**
Le cartelle o i file creati dagli utenti in /tmp all'esterno di $TMPDIR sono accessibili da tutti i job sottomessi dallo stesso utente ed in esecuzione sul nodo. Ad esempio, considerare il caso in cui l'utente ha due job in esecuzione (A e B) sullo stesso nodo e il lavoro B sta scrivendo direttamente i file in /tmp. Se il lavoro A termina prima del lavoro B, anche il contenuto di /tmp verrà eliminato e, in alcuni casi, il lavoro B potrebbe non riuscire. Per evitare errori, gli utenti devono quindi scrivere qualsiasi dato temporaneo nella cartella protetta da lavoro, SLURM_TMPDIR o TMPDIR.

# Gestire i Job
Lo Slurm offre diversi strumenti da riga di comando per controllare lo stato dei lavori e gestirli. Per un elenco completo dei comandi di Slurm, consultare le pagine man di Slurm (link: https://slurm.schedmd.com/man_index.html ). Ecco alcuni comandi che potresti trovare particolarmente utili:

- squeue: scopre lo stato dei job inviati da te e da altri utenti.
- sacct: recupera la cronologia dei job e le statistiche sui lavori passati.
- scancel: annulla i job che hai inviato.

## Controllare lo stato di un Job
Utilizzare il comando squeue per verificare lo stato dei job e altri job in esecuzione su purpleJeans. La chiamata più semplice elenca tutti i job che sono attualmente in esecuzione o in attesa nella coda dei job ("PENDING"), insieme ai dettagli su ciascun job come l'id e il numero di nodi richiesti:

```sh
$ squeue
```

Qualsiasi job con 0:00 nella colonna “TIME” è ancora in attesa nella coda.

Per visualizzare solo i job inviati, utilizzare il flag --user

```sh
$ squeue --user=$USER
```

Questo comando ha molte altre opzioni utili per interrogare lo stato della coda e ottenere informazioni sui singoli lavori. Ad esempio, per ottenere informazioni su tutti i lavori in attesa di esecuzione sulla partizione xhicpu, immettere:

```sh
$ squeue --state=PENDING --partition=xhicpu
```

In alternativa, per ottenere informazioni su tutti i lavori in esecuzione sulla partizione xhicpu, digitare:

```sh
$ squeue --state=RUNNING --partition=xhicpu --user=$USER
```

L'ultima colonna dell'output ci dice quali nodi sono allocati per ciascun job. Ad esempio, se 
mostra wnode01 per uno dei job con il tuo nome, puoi digitare 

```sh
$ ssh wnode01 
```

per accedere a quel nodo di calcolo e controllarne l'avanzamento in locale.

Per ulteriori informazioni, consultare la guida della riga di comando digitando squeue --help o visitare la documentazione online ufficiale.

## Cancellare i Job
Per annullare un job inviato, utilizzare il comando scancel. Ciò richiede di specificare l'id del job che si desidera annullare. Ad esempio, per annullare un job con ID 1008, procedere come segue:

```sh
$ scancel 1008
```

Se non si è sicuri di quale sia l'id del lavoro che si desidera annullare, vedere la colonna “JOBID” dall'esecuzione di squeue --user=$USER.

Per annullare più di un job separare gli id con una virgola:

```sh
$ scancel 1008,1009,1012
```

Per annullare tutti i job inviati in esecuzione o in attesa in coda:

```sh
$ scancel --user=$USER
```

## Job steps paralleli
I job che coinvolgono un numero molto elevato di calcoli indipendenti dovrebbero essere combinati in qualche modo per ridurre il numero di lavori inviati a Slurm. Qui illustriamo una strategia per farlo usando srun. Il programma parallelo esegue le attività contemporaneamente fino a quando tutte le attività, che vengono chiamate job steps, non sono state completate. 

```sh
#!/bin/bash
#SBATCH --job-name=parallel-tasks
#SBATCH --output=parallel-tasks.out
#SBATCH --error=parallel-tasks.err
#SBATCH --partition=xhicpu
#SBATCH --nodes=1
#SBATCH --ntasks=3
#SBATCH --mem-per-cpu=2000
 
srun -n1 --exclusive ./program1.sh &
srun -n1 --exclusive ./program2.sh &
srun -n1 --exclusive ./program3.sh &
 
wait
```

| Opzione | Descrizione |
|------------------------------------|-------------|
|#SBATCH --job-name=parallel-tasks   |Assegna il nome parallel-tasks al job |
|#SBATCH --output=parallel-tasks.out |Scrive l’output della console nel file parallel-tasks.out |
|#SBATCH --error=parallel-tasks.err  |Scrive il messaggio di errore nel file parallel-tasks.err |
|#SBATCH --partition=xhicpu          |Richiede i nodi di calcolo presenti sulla partizione xhicpu |
|#SBATCH --nodes=1                   |Richiede 1 nodo di calcolo |
|#SBATCH --ntasks=3 |Richiede l’esecuzione di massimo 3 tasks paralleli |
|#SBATCH --mem-per-cpu=2000 |Richiede 2000 MB (2 GB) di memoria RAM per singola CPU |

L’esempio vuole eseguire un unico job in cui vengono eseguiti tre programmi in modo parallelo che non sono obbligatoriamente collegati tra di loro. Per poter eseguire più di un singolo task bisogna specificare il flag --ntasks che di default è posto a 1. Una volta specificato possiamo utilizzare il comando srun all’interno del file sbatch ed utilizzare al termine di ogni riga il simbolo &. Ogni srun può avere dei propri argomenti, che devono essere coerenti con le risorse massime richieste dallo script. Nell’esempio ogni srun richiede esattamente un task con l’opzione -n1 (una singola CPU), il parametro --exclusive, che non è obbligatorio, assicura che venga allocata una CPU distinta per ogni attività. Il comando wait attende che tutti i task terminino, falliscano o vengano uccisi (killed).

## GPU Job
La partizione xgpu è dedicata al software che utilizza GPU (Graphical Processing Unit). 

CUDA e OpenACC sono due strumenti ampiamente utilizzati per lo sviluppo di software basato su GPU. Per informazioni sulla compilazione di codice con CUDA e OpenACC, consultare le rispettive documentazioni online.

### Eseguire codice GPU

Per inoltrare un job a uno dei nodi GPU, è necessario includere le seguenti righe di comando nello script sbatch (equivalentemente per srun):

```sh
#SBATCH --partition=xgpu
#SBATCH --gres=gpu:tesla:N
```

Dove --gres richiede esplicitamente la risorsa GPU, gpu:tesla è la nomenclatura specifica da utilizzare su purpleJeans, N è il numero di GPU richieste. Impostazioni consentite per l'intervallo N da 1 a 4. Se l'applicazione è interamente basata su GPU, non è necessario esplicitamente richiedere core poiché un core CPU verrà assegnato per impostazione predefinita come master per avviare il calcolo basato su GPU. Se tuttavia la tua applicazione è CPU-GPU mista, dovrai richiedere il numero di core con –-ntasks come richiesto dal tuo job. 
Slurm imposta automaticamente la variabile d'ambiente CUDA_VISIBLE_DEVICES con la risorsa GPU libera al momento della richiesta. Non tentare di impostare manualmente la variabile CUDA_VISIBLE_DEVICES.



Ecco un esempio di script sbatch che alloca le risorse della GPU e carica i compilatori CUDA

```sh
#!/bin/bash
#SBATCH --job-name=gpu  
#SBATCH --output=gpu.out 
#SBATCH --error=gpu.err  
#SBATCH --time=01:00:00  
#SBATCH --nodes=1        
#SBATCH --partition=xgpu
#SBATCH --ntasks=1       
#SBATCH --gres=gpu:tesla:1     

# Set stack size (to avoid warning message)
ulimit -s 10240

module load cuda/10.1
 
# Aggiungere i comandi necessari ad eseguire l'elaborazione GPU
```

| Opzione | Descrizione |
|---------------------------|-----------------------------------------------------------------------------|
|#SBATCH --job-name=gpu     | Assegna il nome gpu al job                                                  |
|#SBATCH --output=gpu.out   | Scrive l’output della console nel file gpu.out                              |
|#SBATCH --error=gpu.err    |Scrive il messaggio di errore nel file gpu.err                               |
|#SBATCH --time=01:00:00    |Riserva le risorse richiesta per 1 ora (o meno se il processo termina prima) |
|#SBATCH --nodes=1          |Richiede 1 nodo di calcolo                                                   |
|#SBATCH --partition=xgpu   |Richiede i nodi di calcolo presenti sulla partizione xgpu                    |
|#SBATCH --ntasks=1         |Un solo task da eseguire                                                     |
|#SBATCH --gres=gpu:tesla:1 |Richiede un’unica GPU                                                        |

Importante. Le scorciatoie “gres” per l’utilizzo delle GPU, riportate sulla documentazione ufficiale, non sono configurate. 

### Job multipli sui nodi
L’esecuzione di un job su di un nodo di calcolo non assegna quel nodo in modo esclusivo all’utente che ha eseguito il job. Questo è possibile con una richiesta responsabile delle risorse di calcolo disponibili. E’ possibile eseguire più job su singolo nodo anche se il job in esecuzione è di un altro utente, senza interferire con esso. Per fare questo è necessario specificare i parametri negli “script sbatch” o srun che permettono di riservare le sole risorse di cui si ha bisogno.

Vediamo il seguente esempio:

```sh
#!/bin/bash
#SBATCH --job-name=limited-resources
#SBATCH --partition=xhicpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --mem=16000
#SBATCH --cpus-per-tasks=8

# Set stack size (to avoid warning message)
ulimit -s 10240

./program1.sh
```

Nell’esempio viene richiesto di eseguire un job allocando 16GB di memoria e 8 CPU Core totali sul nodo che verrà assegnato dallo scheduler. Considerando che tale nodo ha 192GB di RAM e 32 CPU Core, il nodo ha ancora a disposizione abbastanza risorse (176GB e 24 CPU Core) per permettere ad altri job di poter essere eseguiti su di esso.

Conoscere le risorse libere su ogni singolo nodo
Per conoscere le risorse libere su ogni singolo nodo di ogni partizione, in modo da poter allocare le risorse necessarie per il job che si sta cercando di lanciare è possibile utilizzare il comando sinfo con le seguenti opzioni:

```sh
$ sinfo -N -o "%N %.5a %.11T %.10l %.10m %.10e %.15C" 
 
NODELIST   AVAIL     STATE  TIMELIMIT  MEMORY  FREE_MEM  CPUS(A/I/O/T)
gnode01     up       idle    6:00:00   178000    29353     0/32/0/32
gnode02     up      mixed    6:00:00   178000    14831    16/16/0/32
gnode03     up  allocated    6:00:00   178000   179125     32/0/0/32
gnode04     up       idle    6:00:00   178000    55842     0/32/0/32
wnode01     up       idle   12:00:00   178000   188149     0/32/0/32
wnode02     up  allocated   12:00:00   178000   181558     32/0/0/32
wnode03     up  allocated   12:00:00   178000   182190     32/0/0/32
wnode04     up  allocated   12:00:00   178000   180030     32/0/0/32
```

| Opzione | Descrizione                                                                |
|---------|----------------------------------------------------------------------------|
| -N      | Stampa le informazioni per ogni singolo nodo                               |
| -o      | Specifica quali informazioni visualizzare                                  |
| %N      | Lista dei nodi                                                             | 
| %.5a    | Disponibilità del nodo                                                     |
| %.11T   | Stato del nodo                                                             |
| %.10l   | Limite di tempo per cui è possibile eseguire un singolo job                |
| %.10m   | Memoria totale del nodo                                                    |
| %.10e   | Memoria libera richiedibile del nodo                                       |
| %.15C   | Quantità di CPU in ogni possibile stato (Allocated / Idle / Other / Total) |

Per verificare anche lo stato dell’allocazione delle risorse definite generiche (es. GPU) è possibile usare ad esempio il seguente comando:

```sh
$ sinfo -N -O "nodelist:12,freemem:10,cpusstate:15,gresused"
 
NODELIST    FREE_MEM  CPUS(A/I/O/T)  GRES_USED           
gnode01     29356     0/32/0/32      gpu:tesla:0(IDX:N/A)
gnode02     23472     24/8/0/32      gpu:tesla:1(IDX:0)  
gnode03     175952    32/0/0/32      gpu:tesla:0(IDX:N/A)
gnode04     55845     0/32/0/32      gpu:tesla:0(IDX:N/A)
wnode01     188152    0/32/0/32      gpu:0               
wnode02     180899    32/0/0/32      gpu:0               
wnode03     179464    32/0/0/32      gpu:0               
wnode04     179768    32/0/0/32      gpu:0  
```

N.B. L’equivalente di gresused non è disponibile utilizzando l’opzione -o. Per conoscere tutte le possibili informazioni visualizzabili rifarsi alla documentazione ufficiale di sinfo. 

### Limiti ai Job
Per distribuire le risorse di calcolo in modo equo a tutti gli utenti di purpleJeans, RCF stabilisce limiti sulla quantità di risorse di elaborazione che possono essere richieste da un singolo utente in qualsiasi momento.

Il tempo di esecuzione massimo per un singolo job (batch o interattivo) è di 12 ore per la partizione xhicpu e di 6 ore per la partizione xgpu.

Ulteriori informazioni sui limiti, come il numero massimo di CPU che possono essere richieste da un utente in qualsiasi momento o il numero di lavori che possono essere inviati contemporaneamente su una determinata partizione, sono disponibili inserendo il comando qos su qualsiasi nodo login o calcolo di purpleJeans. Si noti che questi limiti sono spesso diversi a seconda della partizione.

I limiti di utilizzo possono variare, il comando qos fornisce sempre le informazioni più aggiornate.

Se il lavoro di ricerca dell’utente richiede un'eccezione temporanea a un limite particolare, è possibile richiedere un'assegnazione speciale contattando RCF (rcf@uniparthenope.it). Le allocazioni speciali sono valutate su base individuale e possono essere concesse o meno.

# Software
purpleJeans utilizza i moduli di ambiente per la gestione del software. Il sistema dei moduli consente di impostare l'ambiente shell per facilitare l'esecuzione e la compilazione del software. Consente inoltre di rendere disponibili numerosi pacchetti software e librerie che altrimenti sarebbero in conflitto tra loro.

Quando accedi per la prima volta a purpleJeans, verrai inserito in un ambiente utente molto basilare con un software minimo disponibile. Il sistema del modulo è un sistema basato su script utilizzato per gestire l'ambiente utente e per "attivare" i pacchetti software. Per accedere al software installato su purpleJeans, è innanzitutto necessario caricare il modulo software corrispondente.

Comandi base module:

| Comando              | Descrizione                                       |
|----------------------|---------------------------------------------------|
| module avail         | elenca tutti i moduli software disponibili        |
| module disp [name]   | elenca i moduli corrispondenti a [name]           |
| module load [name]   | carica il modulo denominato                       |
| module unload [name] | scarica il modulo indicato                        |
| module list          | elenca i moduli attualmente caricati per l'utente |

Elenco completo dei moduli software:

```
anaconda/3                                     netcdf/4.8.1-gcc-4.8.5
cmake/3.19.2                                   netcdf/4.8.1-gcc-8.3.1
cmake/3.21.4                                   null
cuda/10.0                                      nvhpc/20.11
cuda/10.1                                      nvhpc-byo-compiler/20.11
cuda/11.0                                      nvhpc-nompi/20.11
cuda/11.2                                      ompi-4.0.1-gcc-4.8.5
cuda/11.3                                      ompi-4.0.1-gcc-8.3.1
cuda/11.4.2                                    ompi-4.1.0-gcc-8.3.1
cuda/11.5                                      opengrads/2.2.1
dot                                            openmpi/3.1.3/2019
gcc-8.3.1                                      paraview/5.8.1
intel_MKL-2019u5                               tau/tau-2.30.2-ompi-4.0.1-cuda-11.2-gcc-4.8.5
jasper/2.0.14-gcc-8.3.1                        tau/tau-2.30.2-ompi-4.1.0-cuda-11.2-gcc-8.3.1
module-git                                     use.own
module-info                                    vapor/3.2.0
modules                                        vapor/3.3.0
mvapich2-2.3.5-gcc-8.3.1 
```
