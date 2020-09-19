# Advanced Programming (DRAFT) <!-- omit in toc --> 

- [Abstract machine](#abstract-machine)
	- [Interpreter](#interpreter)
	- [Implementazione di un linguaggio](#implementazione-di-un-linguaggio)
		- [Implementazione di un AB](#implementazione-di-un-ab)
		- [Implementazione puramente interpretata e compilata](#implementazione-puramente-interpretata-e-compilata)

# Abstract machine

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

## Implementazione di un linguaggio

Ml è  un dispositivo che consente l'esecuzione di programmi scritti nel linguaggio L. A una Ml corrisponde un solo L, viceversa a un linguaggio L corrispondono infinite Ml (implementazioni diverse dell'interpreter).

### Implementazione di un AB

- Implementazione mediante Hardware:  
E' sempre possibile implementare tramite hardware una macchina fisica il cui linguaggio macchina coincida con L. SI tratta di realizzare algoritmi e data structure tramite componenti fisiche.
Avendo una corrsipondenza diretta tra linguaggio e hardware, l'esecuzione dei programmi risulta estremamente veloce. Tuttavia, maggiore è la complessità di L (quanto è ad "alto livello") tanto più diventa complessa e rigida l'implementazione di Ml, precludendo insoltre la possibilità di aggiornare Ml nel caso di modifiche ad L.
E' possibile usare questo approccio per linguaggi low level o per sistemi time critical.
- Simulazione tramite Software:  
Per implementare una Ml è possibile usare un linguaggio già esistente L' il cui scopo è interpretare le istruzioni in L. Questo approccio è più flessibile, ma meno veloce: il linguaggio L' corisponde a una Ml' 
- Emulazione (Firmware):
  Come software, ma il linguaggio intermediario sfrutta la microprogrammazione: attraverso l'uso di istruzioni a bassissimo livello eseguibili dall'hardware, memorizzate in una memoria veloce read only speciale, è possibile garantire una velocità maggiore rispeotto all'approccio software e aggiungere flessibilità per eventuali modifiche, cosa non possibile per l'approccio hd.



### Implementazione puramente interpretata e compilata

Sia Mo la host machine e Lo il linguaggio ad essa asociato. Implementare il linguaggi L affinchè sia eseguibile su Mo richiede che Ml sia scritto usando Lo.

- puramente interpretato:
	- programma in L
	- interprete per L, scritto in Lo
	Un approccio puramente interpretato richiede semplicemenete un interprete scritto in Lo in grado di eseguire il programma sulla Mo.
- puramente interpretato:
  - programma in L
  - compilatore da L in LO (esecuzione sulla abstract machine???)
  - Programma compilato in LO
  - Per il puramente compilato il programma viene prima trasformato da L in Lo, e successivamente eseguito sulla Mo

Puramente interpretato: lento, ma facilemnte analizzabile a run time (debuggin tools)
Puramente compilato: veloce, ma difficilmente analizzabile a runtime

# TODO <!-- omit in toc -->

Frameworks: reusable abstraction of code wrapped in a well defined API

Inversion of controll: the program flows is dictate by framework.

Design pattern are a "precooked" solution to a problem. Design pattern are generic solution to a ricorrent problem.

Genericamente, un design pattern può essere usato sia per progettare un'intera applicazione, oppure per risolvere un problema generale in un contesto preciso, oppure consistere in un design riusabile come le hashtable, linked list. Ergo qualsiasi livello di astrazione.

Design pattern are more abstracted than frameworks. Design patterns are "smaller": framework contains DP, reverse is not true.

## Programming Language <!-- omit in toc -->

Defined by syntax, semantics, and pragmatics.  
- Syntax: describe the grammar how to write correctly the program.   
- semantics: generally in natural language. You can use a formal approch, but it's hard and long, so you normally you don't explain all the PL. K framework gives you the semantic of a language  
- pragmatic: includes coding convention, guidilens for elegant code. Paradigms is a style of programming, charactrized by particular concept and abstraction.
List of paradigm: imperative, OO, concurrent, functional, logic ... Of course more paradigm can coexist at the same time. And don't categorize lp by paradigm.

ES if cinvention: 
java/codeconvention.pdf   
google/styleguide/javaguide.html


## Abstract Machine <!-- omit in toc -->
Every Language as a Abstract Machine (AM).  
Memory: include the programs and the data  

Interprenter: set of operation and data structere for:
- primitive data processing
- Sequence control
- Data transfer (parameter passing and returning value from fuction)
- Memory managment (allocation of programs and dat in memory)

General struct of Interpreter (equals for every language more or less):

- fetch instruction
- decode (and itcheck if arguments are needed)
- [fetch operands]
- Execute operations
- Store the result

Best way to build AM is use hierarchy, giving more abstraction (we hope taht each level can see only the first lower level), so you don't have to reimplement the lower level everytime:
- hardware
- firmware
- OS
- Intermediate (Java Bytecode)
- High level (java)
- web machine (browser)
- web service machine (language for web service)


 
## Implementig a Programmin Languafge <!-- omit in toc -->

Pro vs Con
- PRO comp: 
  - anticipait some thing, like type checking at compile time 
  - static allocation
  - static linking
  - code optimization
  - Better performance: you waste time at compilation time, but executing is way faster (using hd accelaration)
- PRO int:
  - easier debug and test, better diagnostic
  - procedure can be invoke from command line (WOW)
  - var can be inspected and modified by a user

[todo what is repl interpreter?]

Using only comp or interp is not good, using both is better.
IO are never compiled, because they can became HUGE and problematic. So they became procedure, and the interpreter will be the one taking care.
Compilation is needed for 

Best solution is using an AM intermediate (AMI). So you basically compile the code in an intermediate language (NOT the machine code! ) and an interpreter run it.

You can define a "common intermediate language" (CIL), so you can compile different hight level language and all this languages will be executed by the same infrastructure (Common language infrastructure CLI).  
Plus, if you create a new language, you don't need a new brand compiler, but you can use the CIL so you compiler will be way easier.  
In CLI the code CIL is then compiled again and only then executed by an interpreter.
