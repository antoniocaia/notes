# Advanced Programming (DRAFT) <!-- omit in toc --> 

- [Lez. 1-3: Abstract machine e PL](#lez-1-3-abstract-machine-e-pl)
	- [Da sapere](#da-sapere)
	- [Abstract Machine](#abstract-machine)
	- [Interpreter](#interpreter)
	- [Implementazione di un AB](#implementazione-di-un-ab)
	- [Implementazione puramente interpretata e compilata](#implementazione-puramente-interpretata-e-compilata)
- [Lez. 4-?:](#lez-4-)
	- [Runtime system and JVM](#runtime-system-and-jvm)

# Lez. 1-3: Abstract machine e PL

## Da sapere

- Reading: Ch. 1 of Programming Languages: Principles and Paradigms by M. Gabbrielli and S. Martini
- Syntax, Semantics and Pragmatics of PLs
- Programming languages and Abstract Machines
- Interpretation vs. Compilation vs. Mixed
- Examples of Virtual Machines
- Examples of Compilation Schemes

## Abstract Machine

Una abstract machine (AB) è intuitivamente una astrazione di un computer fisico. Si tratta di un sistema che eseguendo opportuni algoritmi per sfruttare istruzione e costrutti definiti da un determinato linguaggio (L).

Una AB è composta da:
- Memory
  - Data
  - Program
- Interpreter
  - Sequence control
  - Data control
  - Memory management
- Operation

Una *AB per L (Ml)* è l'insieme di data structures e algoritmi responsabili di memorizzare ed eseguire dei programmi scritti in L.


## Interpreter

Indipendentemente dal linguaggio associato all'interprete, le funioni di un interprete possono essere divise in:
- processing primitive data: primitive data hanno la caratteristica di essere rappresentabili direttamente rappresentati dall'AB
- controlling execution sequence: il flow di un programma non è necessariamente lineare, è necessario usare operazioni e strutture per memorizzare e operare con indirizzi (salvare l'indirizzo dell'istruzione successiva).
- controlling data transfer: accesso alla memoria per il recupero di dati
- memory management: il programma deve essere memorizzato in memoria

Il ciclo di esecuzione di un generico interpreter è il seguente:
- fetch next instruction from memory
- decode to determinate which operation it is. (and to check if arguments are needed)
- [fetch operands]
- Execute operations. Eventually halt.
- Store the result

Data una Ml, il linguaggio che il suo interpreter è in grado di comprendere prende il nome di *linguaggio macchina di Ml*

**Implementazione di un linguaggio**  

Ml è  un dispositivo che consente l'esecuzione di programmi scritti nel linguaggio L. A una Ml corrisponde un solo L, viceversa a un linguaggio L corrispondono infinite Ml (implementazioni diverse dell'interpreter).

## Implementazione di un AB

Il miglior approccio quando si implementa una AB è utilizzare un sistema gerarchico, favorendo l'astrazione (ogni livello dovrebbe poter vedere solo quelllo immediatamente sottostante) e la riusabilità, in mododo da non dover reimplementare i livelli più bassi.  
Una tipica gerarchia:
- E-business machine (applicazione web)
- Wb service machine (linguaggio per web servers)
- Web machine (browser)
- High level machine (java)
- Intermediate machine (java byte code)
- OS machine
- Firmware
- hardware machine

Possibili implementazioni:
- Implementazione mediante Hardware:  
E' sempre possibile implementare tramite hardware una macchina fisica il cui linguaggio macchina coincida con L. SI tratta di realizzare algoritmi e data structure tramite componenti fisiche.
Avendo una corrsipondenza diretta tra linguaggio e hardware, l'esecuzione dei programmi risulta estremamente veloce. Tuttavia, maggiore è la complessità di L (quanto è ad "alto livello") tanto più diventa complessa e rigida l'implementazione di Ml, precludendo insoltre la possibilità di aggiornare Ml nel caso di modifiche ad L.
E' possibile usare questo approccio per linguaggi low level o per sistemi time critical.
- Simulazione tramite Software:  
Per implementare una Ml è possibile usare un linguaggio già esistente L' il cui scopo è interpretare le istruzioni in L. Questo approccio è più flessibile, ma meno veloce: il linguaggio L' corisponde a una Ml' 
- Emulazione (Firmware):
  Come software, ma il linguaggio intermediario sfrutta la microprogrammazione: attraverso l'uso di istruzioni a bassissimo livello eseguibili dall'hardware, memorizzate in una memoria veloce read only speciale, è possibile garantire una velocità maggiore rispeotto all'approccio software e aggiungere flessibilità per eventuali modifiche, cosa non possibile per l'approccio hd.



## Implementazione puramente interpretata e compilata

Sia Mo la host machine e Lo il linguaggio ad essa asociato. Implementare il linguaggi L affinchè sia eseguibile su Mo richiede che Ml sia scritto usando Lo.

- puramente interpretato:
	- programma in L
	- interprete per L, scritto in Lo
	Un approccio puramente interpretato richiede semplicemenete un interprete scritto in Lo in grado di eseguire il programma sulla Mo.
- puramente compilato:
  - programma in L
  - compilatore da L in LO (esecuzione sulla abstract machine???)
  - Programma compilato in LO
  - Per il puramente compilato il programma viene prima trasformato da L in Lo, e successivamente eseguito sulla Mo

**PRO e CONTRO**  
Puramente interpretato: lento, ma facilemnte analizzabile a run time (debuggin tools)
Puramente compilato: veloce, ma difficilmente analizzabile a runtime

Un approccio puramente interpretato o puramente compilato non è ottimale. Per esempio: alcune parti di codice, come la gestione IO, non sono mai compilate a causa di problemi di espansione del codice. Il codiece "problematiche" viene quindi sostituito con procedure (non compilate) e sarà l'interprete a gestirle ed eseguirle.
La soluzione è usare un approccio ibrido in cui il codice viene compilato in un linguaggio intermedio e successivamente interpretato.  
I vantaggi di un linguaggio ibrido (come Java) sono:
- portabilità
- interoperabilità


You can define a "common intermediate language" (CIL), so you can compile different hight level language and all this languages will be executed by the same infrastructure (Common language infrastructure CLI).  
Plus, if you create a new language, you don't need a new brand compiler, but you can use the CIL so you compiler will be way easier.  
In CLI the code CIL is then compiled again and only then executed by an interpreter.

# Lez. 4-?: 

## Runtime system and JVM





