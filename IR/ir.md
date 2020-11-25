# IR (DRAFT) <!-- omit in toc -->

- [Lezione 1](#lezione-1)
	- [Intro](#intro)
	- [Boolean retrieval model](#boolean-retrieval-model)
	- [Inverted index](#inverted-index)
- [Lezione 2](#lezione-2)
	- [Skips](#skips)
	- [Recursive Merge](#recursive-merge)
	- [Zones](#zones)
	- [Web and web-browsers](#web-and-web-browsers)
		- [Spam](#spam)
- [Lezione 3](#lezione-3)
	- [Crawler](#crawler)
		- [Life-cycle Crawler](#life-cycle-crawler)
	- [Marcator](#marcator)
	- [Check for page duplicate](#check-for-page-duplicate)
		- [URL match](#url-match)
		- [Bloom filter](#bloom-filter)
- [Lezione 4](#lezione-4)
	- [Spectral Bloom Filter](#spectral-bloom-filter)
		- [Errore in SBF](#errore-in-sbf)
	- [Parallel Crawlers and how to avoid duplication](#parallel-crawlers-and-how-to-avoid-duplication)
	- [Static assignment](#static-assignment)
- [Lezione 5](#lezione-5)
	- [Compressed storage of WG](#compressed-storage-of-wg)
	- [Locality-sensitive hashing (LSH)](#locality-sensitive-hashing-lsh)
- [Lezione 6](#lezione-6)
	- [(cont)](#cont)
	- [LSH vs K-m](#lsh-vs-k-m)
	- [Duplication](#duplication)
		- [Shingling](#shingling)
- [Lezione 8](#lezione-8)
	- [cosine distance / smilarity](#cosine-distance--smilarity)
- [Lezione 9](#lezione-9)
	- [SPIMI](#spimi)
		- [Big data sorting](#big-data-sorting)
	- [Distributed indexing](#distributed-indexing)
- [Lezione 10](#lezione-10)
	- [Dinamic Indexing](#dinamic-indexing)
	- [Compression of documents](#compression-of-documents)
		- [LZ 777](#lz-777)
		- [Compression and networking](#compression-and-networking)
	- [Lezione ?.1](#lezione-1-1)
		- [Top-k (approssimazione)](#top-k-approssimazione)
	- [Lezione ?.2](#lezione-2-1)

# Lezione 1

## Intro

> Information retrieval (IR) è la scienza che si occupa di trovare materiale (in genere documenti) di natura non-strutturata (in genere testo) al fine di  soddisfare un bisogno di reperire informazioni da una grande quantità di dati (in genere memorizzati su un computer).

Evoluzione dei search engine (Google) riguardo l'indicizzazione:
- Generazione 0: uso esclusivo di metadati aggiunti dagli utenti stessi.
- Generazione 1: uso del contenuto testuale della pagina e di valori statistici come la frequenza delle parole.
- Generazione 2: uso dell'analisi dei collegamenti (link) e uso di anchor-text (il testo assegnato ad un link per riferirsi ad una pagina).
- Generazione 3: uso di più risorse come notizie, immagini. Maggiore attenzione riguardo i bisogni dell'utente rispetto alla natura della query durante l'analisi della query.
- Generazioen 4: struttura "a grafo" per favore risultati trasversali. Vengono fornite più informazioni rispetto a quella che è la richiesta dell'utente (Cercando il termine "Galileo" appaiono sia la sua biografia sia testi da lui pubblicati, nonostante non sia stato specificato "Galileo vita" o "Galileo bibliografia"). Altre caratteristiche sono:
  -  Risoluzione efficace di ambiguità: *the paparazzi photographed the star* e *the astronomer photographed the star* contengono entrambe la parola *star* ma con due significati diversi.
  -  Parole diverse, stesso concetto: *Internt Explorer* e *Microsoft browser* devono produrre gli stessi risultati.
  -  Modi di interagire diversi: una ricerca testuale (poche parole, concetti chiave) risulta differente da una ricerca vocale (formulazione di una domanda: "hey Siri, come si chiama l'autore della Monna Lisa?")
  -  Engine non solo per il web: motori di ricerca per calciatori si focalizzazione sulle prestazioni degli atleti e sulle loro statistiche.

## Boolean retrieval model

Una query booleana è una query che connette diversi termini di ricerca con le keywords **AND, OR, NOT**.  
Per processare query booleane esistono diversi approcci:  
una prima soluzione fa uso di una tabella booleane dove le righe rappresentano i termini e le colonne i documenti in cui ricercare.

|            | Hamlet | Othello | Macbeth |
| ---------- | :----: | :-----: | :-----: |
| **Antony** |   0    |    0    |    1    |
| **Brutus** |   1    |    0    |    0    |
| **Caesar** |   1    |    1    |    1    |

Esempio:  
con la query `Antony AND Caesar` il risultato è:  
`001 AND 111 = 001`  
ovvero la terza colonna, *Macbeth*.

Una matrice di queste dimensioni non è utilizzabile quando si hanno milioni di documenti e di termini a causa dell'enorme memoria necessaria alla sua memorizzazione.


## Inverted index
Un altro approccio per processare query boolean è quello dell'inverted inex.  
Le matrici booleane sono sparse. Memorizzando solo i valori diversi da 0, è possibile occupare solo 1-3% della memoria necessaria alla matrice. 

Vengono utilizzate al posto della matrice delle liste (una per ciascun termine di ricerca) contenenti gli identificativi dei documenti. Ad ogni documento viene assegnato un identificativo univoco.
Due termini appartengono allo stesso documento solo se in entrambe le rispettive liste è presente lo stesso identificatore di documento.  

Riprendendo l'esempio della query `Antony AND Caesar`, quanti confronti sono necessari per verificare che due parole siano entrambe presenti nello stesso documento? Siano *m* e *n* le dimensioni delle due liste, si hanno due casi:
- liste non ordinate: **n\*m**
- liste ordinate: **n+m**

E' fondamentale le liste siano ordinate in modo da utilizzare un algoritmo che confronti sequenzialmente gli id delle liste, e possa incrementare il minore dei due ad ogni iterazione.
```
intersect(p1, p2) {
	answer = empty_list;
	while p1 != NULL and p2 != NULL
		if p1.docID == p2.docID then 
			answer.add(p1.docID)
			p1 = p1.next
			p2 = p2.next
		else if p1.docID < p2.docID then
			p1 = p1.next
		else
			p2 = p2.next
	return answer
}

Costo: O(n + m)
```

Utilizzando liste ordinate è possibile inoltre ottimizzare ulteriormente l'uso della memoria: è possibile memorizzare il *gap* tra i vari valori piuttosto che i valori stessi. (*gap* è la differenza tra un valore e il suo successivo).

Avendo una lista contenente:  
`| 12345780 | 12345781 | 12345784 |`  

E' possibile risparmiare memoria memorizzando:  
`| 12345780 | 1 | 3 |`  


Nel caso vengano utilizzati più termini di ricerca, una possibile modifica è quella di effettuare i confronti sempre a coppie, partendo dalle due liste con un numero minore di elementi e utilizzando il risultato come lista temporanea. In questo modo infatti il risultato di ogni iterazione avrà come risultato una lista lunga al più quanto la più piccola delle due liste utilizzate.

# Lezione 2

## Skips

Scorrere parallelamente due liste, specialmente se di dismensioni molto diverse, comporta uno spreco di tempo.  

Una possibile soluzione è quella di utilizzare degli *skips*:  
La lista viene divisa in blocchi logici di dimensione **n**. Per ogni blocco, all'elemento di testa viene associato un puntatore al valore iniziale del blocco successivo (*look ahead*).  
In questo modo, durante l'applicazione dell'algoritmo, quando uno degli elementi di testa viene scartato perchè minore rispetto all'elemento dell'altra lista, è possibile andare a vedere il suo look-ahead: se anche questo è più piccolo del valore di confronto, allora è possibile scartare tutto il blocco, e osservare il successivo look-ahead.

Qual'è la dimensione ideale dei blocchi, affinchè sia possibile risparmiare tempo e memoria? Blocchi piccoli richiedono un numero maggiore di puntatori (consumo di memoria) e offrono una maggior probabilità di effettuare salti ma con una minor efficienza; contrariamente blocchi grandi permettono di avere meno puntatori memorizzati e una probabilità minore di saltare ma con una efficienza manggiore.  

Sia **n** la lunghezza di una lista:  
La dimensione ideale di ogni blocco è uguale a **n^(1/2)**.  
Il numero ideale di blocchi è uguale a **n^(1/2)**.
Perchè?  

Sia **L** la dimensione di un blocco:
Il caso peggiore prevede **n/L - 1** confronti per arrivare all'ultimo blocco, e L confronti per arrivare all'ultimo elemento del blocco. Semplificando, si ha che il numero totale di confronti è **n/L + L**. Affichè si abbia il minor numero di confronti, la dimensione ideale dei blocchi è **L = n^(1/2)**.

Un differente approccio di ottimizzazione prevede di usare blocchi di dimensione variabile. Ad ogni ID è associata la rispettiva frequenza con cui sono il risultato di uan query. La lista viene segmentata in modo che gli ID con una frequenza maggiore siano elementi di testa dei blocchi. In questo modo, quando viene effettuato un salto, saranno i documenti meno richiesti ad essere saltati, e sarà più probabile che l'elemento in testa del successivo blocco sia quello ricercato.


## Recursive Merge

Recursive merge è un algoritmo alternativo agli skips.  
Dalla lista (e sottoliste) con meno elementi viene selezionato l'elemento centrale. Attraverso una ricerca binaria viene ricercato (o "collocato" nel caso non sia presente) nella lista con più elementi. Ricorsivamente viene riapplicato l'algoritmo, utilizzando però le sottoliste "nate" dalla ricerca binaria.

L'elemento mediano viene preso dalla lista minore perchè si vuole applicare la ricerca binaria nella lista con un maggior numero di elementi in modo da ridurre il numero di confronti.
 
Sia **m** la dimenisone della lista B contenente meno elementi.
Sia **n** la dimensione della lista A contenente più elementi.

Selezionare l'elemento mediano in B e applicare la ricerca binaria ha costo log n. Generalizzando, poniamo che l'elemento vorrispondente in A sia sempre a metà della lista. Applicando ricorsivamente l'algoritmo alle sottoliste (Asx e Adx, entrambe di dimensione n/2, e Bsx e Bdx, entrambe di dimensione m/2) otteniamo la seguente relazione:

**T(A,B) = O(log n) + T(Asx,Bsx) + T(Adx,Bdx)**  
**T(n,m) = O(log n) + T(n/2, m/2) + T(n/2, m/2) = O(log n) + 2T(n/2, m/2) = n log(n/m)**

Nel caso in cui n e m siano vicini, allora il costo è assimilabile ad **n+m** (simile al merge), nel caso in cui m << n, allora il costo è circa **m*log n** (per ogni elemento di B, ricerca binaria in A).


Caso particolare: se l'elemento cercato non è mai prensente nella lista maggiore, allora il costo **O(log m + log n)** 


Fin ora è stato valutato il costo del merge con l'operando AND.   
Con AND NOT la complessività è sempre **n+m**, ma sono richiesti controlli addizionali.

OR NOT invece crea problemi: `Antony OR NOT Caesar` ritorna miliardi di pagine poichè NOT Caesar ritorna tutte i documenti che non contengono Caesar, e OR fa si che vengano presi tutti i risultati.

## Zones

Per rendere le query binarie un po' più precise, è possibile introdurre le "zones" o campi. Un campo può essere:
- Title
- Author
- Abstract
- References
- ...

E' possibile specificare un campo durante la ricerca con ad esempio :  
`((author = Ullman ) AND (text contains automata))`

Per realizzare ciò è necessario apportare delle modifiche:  
- per ogni termine, creare una lista indipendente per ciascuno dei suo campi.
- mantenere una unica lista, ma specificare insieme all'ID del documento in quali campi appare il termine

## Web and web-browsers

Il web può essere visto come un orientato i cui nodi sono pagine html e gli archi i link che collegano due pagine. Gli archi "entranti" in nodo rappresentano i suoi *in-links* (link che conducono ad una pagina) mentre gli archi uscenti sono definiti *out-links* e rappresentano tutti i collegamente tra un nodo e il resto del web.  
Il link non rispettano una distribuzione di Poisson, piuttosto danno vita ad un grafo dalla forma di "farfallino", dove distinguiamo 3 macrogruppi:
- IN: pagine prive di in-links
- OUT: pagine prive di out-link
- SCC (strongly connected components): partendo da qualsiasi elemento in ssc è possibile raggiunge qualsiasi degli altri nodi di ssc (esiste sempre un cammino che collega due elementi i ssc).

### Spam

Con *spam* ci si riferisce alla pratica di "ingannare" i web search engine in modo da incrementare il rank di una pagina web.  
Alcuni engine che sfruttavano la frequenza di determinati termini per rankare le pagine erano facilemnte ingannabili aggiungendo apposite sezioni nella pagina web (il cui testo era dello stesso colore del background, in modo da non disturbare l'utente) in cui aggiungere e ripetere tutti i termini che si volevano usare per "indicizzare" la propria pagina.  
L'evolversi dei motori di ricerca ha portato alla nascita di nuove tecniche di spam: alcune pagine sono in grado di determinare se una richuesta http proviene da un crawler, e in quel caso forniscono una pagina "fake" da usare per l'indicizzazione. Quando è un utente ad eseguire la richiesta, viene fornito una pagina con contenuti diversi da quelli per l'indicizzazione.

# Lezione 3

## Crawler

Il web crawling è il processo attraverso il quale è possibile raccogliere e indicizzare pagine in web in modo da supportare il lavoro di un mototre di ricerca.

Le caratteristiche che un crawler deve possedere sono molteplici
- Robusto: essere in grado di riconoscere ed evitare "trappole"
- Analisi di qualità: il crawler non deve procedere casualmente, ma visitare le pagine migliori per prime
- Etiquette: gli host in genere specificano all'interno di un file *Robots.txt* cosa è permesso fare al crawler e cosa no. In genere si vuole evitare un sovraccarico del web server, e intervallare l'invio di query. In alcuni casi non si vuole indicizzare determinate pagine.
- Efficienza: evitare duplication e near-duplication

Quando si fa crawling, le considerazioni da fare sono:
- Copertura: quanto è grosso il web e quanto ne vogliamo coprire
- Copertura relativa: la competizioni quanto web indicizza

Infine, quanto spesso fare crawling?
- freschezza: quanto è cambiata una pagina dall'ultima visita 
- frequenza: ogni quanto visitare le pagine


### Life-cycle Crawler

**Glossario:**  
- Seed Page (*SP*): pagina contenente gli URLs utilizzabili da un crawler come punto di partenza.  
- Priority Queue (*PQ*): URLs da valutare  
- Assigned Repository (*AR*): sinonimo per Frontier  
- Page Repository (PR): repository per le pagine una volta che sono state scaricate

Un "crawler manager" preleva URL dalla *PQ*, se la pagina non è stata mai visitata o è stata modificata dall'ultima visita viene inserita in *AR*.  
Ogni pagina ha un "downloader": preleva da *AR* l'URL e scarica e compressa la pagina per salvare il risultato in *PR*.  
Il "link extractor" preleva una pagina da *PR* e cerca eventuali link, per inserirli in *PQ*.

Con che criterio vengono scelte le pagine dall'*AR*? Dipende dalla policy scelta: BFS, DFS, Random, Popularity driven, Page Rank, topic driven, combined.  

## Marcator
Marcator è un crawler che segue alcune semplici regole:
- una sola richiesta alla volta per host, in modo da non sovraccaricare l'host
- aspettare alcuni secondi tra una query e l'altra 
- preferire pagine con un alta priorità per il crawling

**Overview:**  
Un modulo preleva gli URLs dalla Frontier per poi distribuirli in *K* queue; ogni queue ha un diverso livello di priorità.  
Un successivo modulo estrae dalle queue gli URLs preferendo quelli con priorità maggiore.  
Gli URLs sono distribuiti in *B* queue (ogni queue rappresenta un host diverso); le pagine provenenti dallo stesso host vanno nella stessa queue. Ciò permette di regolare gli accessi agli host, evitanto multipli accessi contemporaneamente. Se una queue si "svuota", può essere riassegnata ad un nuovo host.  
URL selector preleva un URL da ciascuna delle *B* queue (una pagian per host) e gli assegna ad un min-heap. Il min-Heap si basa su dei time-stamp.  
Il time-stamp è calcolata sommando il tempo di esecuzione della pagina precedente (dello stesso host) + un tempo di attesa extra.  
L'heap restituisce l'URL con il tempo minore (p1), il crawler visita la pagina. L'heap deve rimpiazzare p1 con una pagina proveniente dallo stesso host (p2), e per fare ciò assegna come time-stamp a p2 un tempo pari a t(p1) + delta. Questo è applicabile nel caso p2 sia presente. Se per quell'host la queue è vuota, allora andiamo a prendere una pagina tra le front queue (stando attenti che non sia una pagina il cui host ha già una queue assegnata).


## Check for page duplicate

Come determinare se una pagina è già stata visitata? Devo tenere traccia degli URLs già visitati. 

- URL match
- Duplicate document match
- Near-duplicate document match

### URL match  
Un possibile approccio è quello di usare un hash-table per memorizzare gli URLs. La memoria necessaria è tuttavia eccessiva, ed anche il tempo di accesso è infattibile. Questo approccio è stato usato da Altavista ma solo finchè il numero di pagine lo ha consentito. 

### Bloom filter

Bloom filter usa:
- array binario (size *m*) inizializzato con soli 0
- *k* funzioni hash che, dato un URL, ritornano un intero compreso tra *0* e *m-1*

Per controllare se un URL *u* è già stato visitato:
- si computano le *k* funzioni di hash con input *u*
- si controlla i valori nel vettore binario in corrispondenza degli indici *i1, i2, ... , ik* restituiti dall'hashing: se TUTTE le posizioni hanno valore 1, allora l'URL è già stato visitato.
  - Nel caso non tutti i bit siano 1, visitiamo la pagina e poniamo 1 nel vettore binario in corrispondenza degli indici *i1, i2, ... , ik*.

Questo approccio introduce possibili errori (falsi positivi):
- la probabilità che un bit sia 0 dopo l'inserimento di tutti gli URL effettivamente visitati  
`p = (1 - 1/m)^(kn) =  e^(-kn/m)` dove `n = numero URL`
- la possibilità di avere un falso positivo è quindi: `(1 - p)^k`, ovvero la probabilità di avere un 1 per ognuna delle funzioni di hashing piuttosto che 0.
- Come minimizzare l'errore:
  - il numero ideale di funzioni si ha da `k = (m/n)*ln(2)`, sostituendo si ottiene  circa `p = 0.62^(m/n)`. Questo vuol dire che aumentando il rapporto tra *m* ed *n* l'errore diminuisce.  
  Con un rapporto di 10 (che corrsiponde ad usare 10 bit per URL, o un BF di n*10 bits) si ottiene 0.0084 (0.84% di probabilità) che risulta essere un valore accettabile.
- Attraverso il bloom filter c'è un risparmio considerevole di memoria: con l'hashing memorizzare la stringa di un URL richiede in media 8 * 1000 bit, mentre con il bloom filter è possibile memorizzare anche solo 10 bit per URL.
- Quante funzioni di hash sono necessarie? poichè `ln(2)` è trascurabile, scegliendo `m = 10n` k diventa circa 10.

# Lezione 4

## Spectral Bloom Filter

Evoluzione del tradizionale Bloom Filter:
- miglior performance a scapito di un uso di memoria leggermente maggiore
- inserimento e eliminazione possibili

SBF utilizza:
- *f(x)*: funzione che calcola la frequenza di *x* in un dato set *S*
- vettore *C* di dimensione *m* contenente le frequenze mappate dalle funzioni di hash
- k funzioni di hash (*hi*)

Come nel BF, le posizioni in C non sono univoche e possono presentarsi collisioni.

Procedimento:
- f(x) viene valutata, restituento una quantità *q*
- vengono valutate le k funzioni hash, che restituiscono gli indici *i1, i2, ... , ik*
- I valori in *i1, i2, ... , ik* vengono incrementati di q.

Per l'**inserimento**, basta valutare le funzioni h(x) e incrementare di uno i contatori corrispondenti.  
Per la **rimozione**, come per l'inserimento ma decrementando.  
Per la **ricerca**, prendo il valore minore tra quelli puntati dalle funzioni di hash.

### Errore in SBF 
``` 
Sia mx il più piccolo valore ottenuto tramita la ricerca di x.
Allora mx >= fx (dove mx > fx implica una collisione). 
Segue che fx != mx con probabilità  e = (1 - p)^k 
Identico all'errore del classico BF
```

Problemi implementazione SBF:  
Tenere basso l'errore per inserimento ed eliminazione.  
Utilizzo di un SBF secondario, più piccolo in dimensioni (SBF2) perchè conterrà solo gli elementi senza RM.  

L'algoritmo prevede due possibili strade quando effettuo la ricerca:  
- Se è presente un *recurring minimum* mx (il valore minimo appare più volte agli indici di hash, aka "dovrei avere almeno due posizioni non alterate") allora ritorno   mx.
- Se non ho RM allora esiste il richio che tutti i valori siano stati alterati (almeno k-1 lo sono) e che lo sia anche il minimo.
  - Per questo caso viene introdotta una seconda SBF (SBF2), ma molto più piccola.
  - Quando non ho un RM, cerco in SBF2:
    - Se mx2 in SBF2 > 0 allora ritorno mx2
    - Se mx2 in SBF2 = 0, allora ritorno il minimo mx in SBF

Per l'inserimento:
- vado a incrementare tutte i contatori di x in SBF
  - se in SBF1 non ho RM allora vado ad aggiornare anche SBF2
    - Incremento tutti i contatori se diversi da 0
    - altrimenti le inizializzo con il valore minimo presente in SBF

Per l'eliminazione è come l'inserimento, solo che decremento.

we use a heap to keep the top querys


## Parallel Crawlers and how to avoid duplication

Un problema nel far lavorare contemporaneamente più crawler è che esiste la possibilità che la stessa pagina venga visitata più volte e duplicata.

Soluzioni:  
- Dynamic assignment: controllo centrale che assigna ai singoli crawlers gli URL da visitare. Necessaria comuninicazione tra crawlers e controllo si crea collo di bottiglia.
- Static assignment: web diviso staticamente, una parte per crawler. La divisione deve essere scalabile e bilanciata. Se un crawler "muore", una intera parte di web rimane scoperta. E' possibile bilanciare/riallocare gli URL nel caso di rimozione/aggiunta di crawler, ma è complesso.


## Static assignment

- Basato su URL/host: ad ogni crawler si assegna un dominipo. Problemi: carico non bilanciato, modificare la distribuzione è complicato.
- HASH:
  - Assegnare ad ogni crawler un indice {0 ... d}. 
  - Definire una funzione di hash per URL con codominio {0 ... d}
    - la funzione assegna ad ogni URL un unico crawler: ogni crawler computa tutti gli URL, ma visita solo quelli la cui funzine di hash corrisponde al proprio indice.


**Consistent hashing**: 
- mapping dei crawler and url con la stessa funzione di hash. (Una buona funzione è `h(x) = a*x mod p` dove a casuale e p primo e p > |I|
  - crawler ids (cid): c1, c10, c16
  - URL ids (uid): u2, u14, u16, 19  
- un url viene assegnato al primo crwaler il cui cid > uid
- facile inserimetno nuovo crawler (guardo il crawler con l'indice "successivo" e ricomputo i suoi URL), e facile eliminazione (ricomputo gli URL del crawler rimosso)

crawler open source (download):
- NUTCH 
- SCRAPY


# TODO <!-- omit in toc -->

# Lezione 5

Il Web Graph può essere visto come una coppia di insiemi V e E, dove V sono i vertici (pagine) ed E i rami (link).

Il WG gode di tre importanti proprietà:
- Skewed distribuition:
  La probabilità che un nodo abbia X collegamenti è `1/(k^a)` con `a = 2.1`.  
  È stato osservato che il numero di link in entrata (così come quelli in uscita, e altre proprietà) rispecchiano una power low distribuition.
	<!--
	```
	y = 1/x^a
	log2(y) = log2(1/x^a) = -log2(x^a) = -a*log2(x)
	y' = -a*x'
	```
	-->

- Locality:  
  sia H l'host di U, allora l'80% dei link di U andrà verso URLs presenti sempre in H
- Similarity:  
  siano U e V URLs nel solito host, allora molti dei link presenti in uno saranno presenti anche nell'altro.
  

URL sorting:
usiamo la forma invertita degli url per memorizzarli, in modo che domini uguali o simili siano contigui quando memorizzati.   
`www.di.unipi.it/inf/...`  diventa `it.unipi.di.www/inf/...`


## Compressed storage of WG

Per memorizzare il WG è necessario ridurre il più possibile lo spazio utilizzato.  
Sfruttando le proprietà di locality e similarity è possibile effettuare varie ottimizzazioni. 

Due URL nello stesso host sono lessicalmente vicini e quindi vicini nella tabella, inoltre hanno in comune l'80% dei link. Ciò permette di evitare di memorizzare più volte gli stessi link: ogni URL controlla le liste di link degli URLs precedenti interni a una window definita e sceglie quella con più elementi in comune. Attraverso un riferimento (mioId - refId) e una lista binaria indica i link in comune.  

Per migliorare ulteriormente la compressione è possibile cambiare il tipo di memorizzazione dei riferimenti, da cui si ottiene la struttura finale:
- id nodo (url)
- \# link in uscita
- riferiemnto per l'URL di cui vogliamo la lista di link
- \# di blocchi nella "lista binaria" per la lettura della lista di riferiemnto
- valore del primo blocco e dimensione dei blocchi (-1)
  - Il numero di blocchi effettivamente presenti è sempre inferiore di uno rispetto a quello dichiarato: per ottenere questo ultimo blocco basta vedere quanto è lunga la lista dell'url di riferimento e sottrarre la dimensione dei vari blocchi
- link extra 


| Node  | Outd  |  Ref  | # blocks | copy blocks | extra nodes |
| :---: | :---: | :---: | :------: | :---------: | :---------: |
|  15   |   3   |   0   |          |             | 10, 11, 12  |
|  16   |   4   |   1   |    3     |    1,1,0    |   13, 14    |


## Locality-sensitive hashing (LSH)

Dati:
- *U* users
- set od *d* features
Trovare il più grande gruppo di utenti simili.

Possibili soluzioni:
- brute force, provare le possibili combinazioni:
  - costo: `2^U * U^2`  
  - parallel comp o cpu migliori non comportano miglioramenti  
  - anche limitando la dimensione dei gruppi ad un massimo L, sia ha comunque un   costo di `U^L * L^2` 
- algoritmi di clustering (K-means)
  - K è il numero di gruppi 
  - Ogni iterazione dell'algoritmo costa: K * U
  - confrontare gli attribuiti dei vari punti ha costo O(d)
  - provare i vari K (senza saperlo a priori) ha costo `U^3`
    - vista la relazione `U = tempo^(1/3)`, incrementare la potenza computazionale di un fattore `s` di fatto migliora i tempi di solo `s^(1/3)`
- LSH: introdurre fingerprint per far si che elemnti di U simili diventino uguali: facili da confrontare, costo ridotto

Locality-sensitive hashing appartiene a quest'ultima categoria.  
Appunti prof: https://www.dropbox.com/s/ovavpl1s0yu71fo/Archivio29.09.2020.zip?dl=0

**HOW THIS WORKS:**

- Siano *p* e *q* due vettori binari e *d* la dimensione dei due vettori.  
- `D(p,q)` è la funzione di similirità e restituisce il numero di bit differenti in p e q (hamming distance).   
- Similirità `S = (d - D(p,q)) / d` con `0 <= S <= 1`  
  - Probabilità che presa una posizione x casuale in p e q i bit siano uguali: `P(x) = S`

Inoltre:

- sia *hI(p)* la funzione di hash di restrizione su I.
  - hI si basa su un set I = { i1, i2, ... ik} con K = |I| contenente le posizioni su cui effettuare le restrizioni, aka quali bit prendere mentre il resto viene escluso.
  - I è determinato casualmente!
- Invece di valutare D su p e q, valuto `D(hI(p), hI(q))`
  - Probabilità che presa una posizione x casuale in p e q i bit siano uguali: `P(x) = S^K`

*Aumentando K riduciamo i falsi positivi (K = d comporta nessun falso positivo), ma aumentiamo i falsi negativi.*

Soluzione: Definite L set I (e rispettive funzioni di hash) con K costante.  
Iterare quindi tutte le L funzioni, e se anche una sola restituisce un match tra hIi(p) = hIi(q) allora definiamo p e q "uguali"  

*Aumentando L riduciamo i falsi negativi ma aumentiamo i falsi positivi.*

Necessario trovare un bilanciamento tra L e K e per avere un compromesso tra prestazioni, falsi positivi e falsi negativi.

```
Definiamo g come la funzione di "concatenzione" delle funzioni di hash:  
g(p) = < hI1(p), hI2(p), ..., hIl(p) >   
Allora p è simile a q sse esiste un j tc hIj(p) = hIj(q)
```
Sappiamo che la probabilità che due fingerprint siano uguali è `S^K`.  
Quindi la probabilità che `g(p) = g(q)` è `P(g(p) = g(q)) = 1 - P(tutte hIj per p e q siano diverse)` da cui:  
```
P(g(p) = g(q)) = 1 - [1 - S^K]
```

Come varia la probabilità al variare di S?  
Banalmente, al ridursi della similirità, diminuisce la probabilità.  
La funzione tuttavia non raggiunge mai lo zero e neanche l'uno.

# Lezione 6

[Parte 1](https://web.microsoftstream.com/video/f93bddf6-7e68-4167-88f2-1f7a2befc018)  
[Parte 2](https://web.microsoftstream.com/video/b05f1333-28bb-4512-91dd-ae5d34c8468d)  

## (cont)

Come utilizzare tutto ciò, aka come capire quali sono le pagine simili?  

**Approccio offline: (browsers)**  
- calcolare g(pi) per ogni i
- ordinare  g() per hI1() (aka raggruppare basendoci sul primo fingerprint).
- ordinare  g() per hI12()
- ...
- ordinare  g() per hIl()

|   p   |  gI1  |  gI2  |
| :---: | :---: | :---: |
|  p1   |   0   |   3   |
|  p2   |   1   |   0   |
|  p3   |   1   |   2   |
|  p4   |   3   |   0   |
|  p5   |   3   |   1   |

Costruisco trasitivamente le similità tra le varie pagine:  
p2 e p3 sono simili attraverso gI1; p2 e p4 sono simili per gI2; p3 e p4 sono simili.  
(Possono esserci restrizioni maggiori irl)

**Approccio online: (db)**  

"Searching approximation"  

Piuttosto che ordinare e raggruppare, creo L hash table, una per ogni funzione hIj, dove inserire i risultati.

Ad un certo punto il db riceve una query q, che viene computata da tutte le funzioni di hash. In ogni tabella viene prelevato il contenuto dello slot puntato dalla rispettiva funzione (NB: se hI1 mi ritorna un slot contenete p3, hI2 potrebbe ritornare un slot vuoto o contenete p7, non necessariamente p3).  
In sostanza, vado a prelevare e confrontare pagine che so essere rimili, e poi tramite opportune operazioni decido cosa ritornare.

## LSH vs K-m

Slide 

## Duplication

I motori di ricerca evitano di indicizzare pagine il cui contenuto è identico/quasi identico.  

**Duplicazione esatta:**  
In base al contesto, è possibile adottare tecniche più lente (MD5, checksum) oppure tecniche più veloci (Karp-Rabin).  

**K-B fingerprint:** 
Usato per verificare se due "finestre" di bit sono uguali.   
Sequenza a di m bit. Poichè vogliamo associare a tale sequenza una valore intero, aggiungiamo un bit aggiuntivo a sx della sequenza e lo poniamo a 1.  
Si prende un valore primo p in U tc 2*p sia occupi pochi byte. (U) = 2^64).  
La funzione di fingerprint è f(a) = a mod p.  

### Shingling

Separare documenti in q-grams (shingles), ovvero set di q termini contigui.  
Due documenti sono simili se i rispettivi insiemi di shingles si intersecano suffcientemente.  

Maggiore è q, maggiore sarà la somiglianza tra due documenti. Tuttavia, all'aumentare di q aumentiamo esponenzialmente l'impatto di ciascun termine sull'insime di shingles. Una parola cambiata può influire su un numbero altissimo di q-gram (4 < q < 8).  
Come valutare l'intersezione tra due insiemi di shingles Sa Sb ? 
- intersezione non è sufficiente, in quanto influenzata dalla lunghezza dei documenti.
- JACCARDI similarity `(sim) = Sa inter Sb / Sa union Sb`

**Come ottenere Jaccard-sim?**

Set di shingles. Trasformiamo le stringhe in valori numerici, modulo 2^64.  
Calcolare l'intersezione richiede iterazioni, ed è troppo costoso! SOluzione:  
- permuto con funzione `p(x) = ax + b mod 2^64` con a e b co-primi
- prendo i minimi di ciascun set
- `P( min(p(Sa)) min(p(Sb)) )  =  jac-sim(Sa, Sb)  =  Sa inter Sb / Sa union Sb`  

for the dummies:   
- per ogni set di shingles Si valutano m funzioni di permutazione p (da cui otteniamo lo sketch di ciascun S)
- in media, avrò che:  
```
# valori uguali tra gli sketch di A e B = # di funzioni di permutazione (m) * P( min(p(Sa)) min(p(Sb)) )

Divido da entrambi i lati per m

ottengo che :
# valori uguali tra gli sketch di A e B / m = 
P( min(p(Sa)) min(p(Sb)) ) = 
Sa inter Sb / Sa union Sb

Per cui ho una approssimazione di Jac-sim
```

# Lezione 8

## cosine distance / smilarity

- Prodotto scalare X: `sommatoria (pi * qi)` dove p e q sono due vettori
- `cos(a) = p X q / norm2(p) * norm2(q)` dove `norm2(p) = ( sommatoria pi^2 )^1/2`
  - cos(a) range tra -1 e 1
  - Il vantaggio è che l'angolo è indipendente dalla lunghezza del vettore (aka la dimensione della pagina)

Come funziona:
Vengono generati r1 ... rk vettori che computiamo per p e q:
- sign(p X ri) = alfa -> { 1 se 0 < alfa < 90, 0 se 90 < alfa < 180}
  - per p e q avremmo una sequenza lunga k di 0 e 1 
- P(hr(p) = hr(1)) = 1 - alfa / PI


# Lezione 9

Nota sulla gerarchia di memoria:
- registri cpu (reg)
- cache (ca)
- ram (mem or M)
- hd
- net

Criticità:  
- spatial locality (i dati vengono prelevati dalla memoria in blocchi, è importante che in uno stesso blocco abbia sia il dato necessario in quel momento che i dati che mi serviranno per gli step successivi dell'algoritmo)
- Temporal locality usare i dati finche sono disponibili in memoria!
  

**Problema: sorting degli elementi (STRINGHE), risorse temporali/spaziali.**
Quando si fa il sorting di A, A contiene i puntatori a stringhe, quindi non solo devrò confrontare stringhe ma anche tenere conto di eventuali miss nella memoria. (Gli elementi di A sono adiacenti, gli elementi puntati da A no!)

Approssimativamente, con un normale algoritmo di sorting, rischio di incappare in O(n logn) IO operation in random memory

## SPIMI
Algoritmo per la generazione di inverted list.  

Si abbia un set di documenti d1, d2 ... dk che prend il nome di *blocco*.  
Processo ogni documento, e per ogni termine creo una sola inverted list contenente gli id dei documenti in cui il termine compare (se "abaco" compare in d1 e d3, in memoria avrò  "abaco -> 1, 3")  

Processato un blocco, lo flushiamo nel disco (liberando la memoria interna) e procediamo a processare il blocco successivo (con una nuova IL).

Approccio alternativo: una sola tabella per costruire l'IL.
- Creare una unica tabella 
- Per ogni termine di ogni documento viene inserita una nuova entry nella tabella < termine, id doc >
- sort stabile (serve per la frequenza dei termini)
- parole uguali saranno affiancate, scansionando la tabella è facile costruire l'IL e tenere traccia della frequenza dei termini
Create a unique table, then you store for each documente tutte le parole ssociando l'id del cocumento (ci saranno duplicati)

sort (stabile! importante mantenere l'ordine dei documenti) alfabeticamente

Parole uguali saranno affianco, per trovare una parola semplicemente la ricerco, e partendo dalla prima ottengo tutte le parole che cerco (e posso calcolare la frequenza!)

Problema: sorting di stringhe di lunghezza variabile crea complicazioni per la memorizzazione. Soluzione:  
Quando faccio lo scan del documento, inserisco le parole in un hash table (associo ad ogni parola un identificativo, e non ho ripetizioni, quindi la memoria occupata dall'hash table è poca è posso tenerla in memoria tutto il tempo (NB questa è una assunzione))

- scanning dei documenti per inserire nell'hashtable (dictionary)
- sort dell'hash table (veloce perchè memoria interna) e associo ad ogni toke un id "lessografico" -> parola 1 < parola 2 allora id(parola 1) < id(parola 2)
- scan dei documenti (di nuovo) e inserisco nell'unique table, con coppia id_documento (come prima) e id(pa rola) (l'id assegnato nell'hash table) così da avere due indici interi
- sort della unique table (come prima)
- decoding per la ricerca usando l'hash table

### Big data sorting 

Riguardo all'algoritmo di sorting, esso deve essere realizzato ad hoc, perchè dobbiamo ordinare miliardi di valori.

Assumiamo set di interi (termId).  
M = dimensione memoria interna.  
B = dimensione pagina sul disco (quanti byte sono conivolti per ogni read write)  
#blocchi  = n/M dove n presumo sia la memoria totale dei dati da ordinare  

1) Dividiamo la tabella in blocchi di dimensione M  
   Su ogni blocco si fa fetch (M/B), sort (in ram, quindi veloce), write (M/B)  
   Quanto costa il totale? 2 M/B * n/M = n/B

2) Dopo viene fatto un multiway-merge: sul disco abbiamo già dei blocchi ordinati (risultato di iteerazioni precedenti) quindi dal disco viene preso un blocchetto all'inizio di ognuno di essi, finchè ho spazio in memoria. Uso il multi-merge, e quando output è pieno allora scrivo sul disco  

## Distributed indexing

Big data richiede sistemi distribuiti per creare le IL, quindi viene usato un cluster.  
Poichè le macchine possono rompersi, è necessaria una gestione particolare.

- master: gestisce le altre macchine
- idle machine: si occupano del lavoro, possono ricoprire diversi ruoli

**DOCUMENT BASED:** 
- Ogni macchina si occuperà di una serie di blocchi (sets)
- Ogni set è assegnato ad un *parser*: il parser genera la tabella idTerm idDoc
- Ad ogni parser è associato un inverter, il parser invia le coppie ad una macchina *inverter* il cui compito è creare l'inverted list.
- Al termine, avremmo tante inverted list

Quando facciamo una query, essa deve essere processata su diverse Inverted List!

**TERM BASED:**

- Ogni macchina si occuperà di una serie di blocchi (sets)
- Ogni set è assegnato ad un *parser*, e il parser genera la tabella (le coppie idTerm idDoc) e poi *parziona il risultato secondo un criterio arbitrario (es, 3 gruppi basandosi sulla prima letterea del termine)*
- Ogni diversa partizione è inviata ad uno specifico inverter. (es, inverter per il primo gruppo, inverter per il secondo, inverter per il terzo)
- Gli inverter creano le IL

Un buon criterio di partizionamento deve essere effettuabile da ogni parser, per esempio hashing.


___
___

# Lezione 10

## Dinamic Indexing

**Soluzione 1:**  

N = dimensione di tutta la collezione

teniamo due inverted list i1 (in mem M) e i2 (sul disk) (dictionary)  
quando i1 è grande quanto d, allora i1 è meged con i2  
Per fare il merge, se x è in i1 e non in i2, inserisco e basta. se x è anche in i2, appendo le due index list. (linear cost)  
In totale faremo O(N/M) chiamate di funzioni merge  
Il costo totale dei merge merge è:  
- M 0 -> M
- M M -> 2M
- M 2M -> 3M
- ...

O(M * sommatoria) = M* (N/M)^2 = O(N^2 / M)  
se M è circa N allora O(N^2 / M) -> O(N) = O(M)  

**Soluzione 2:**  

Logaritmic merge: 

Internal memory M e disk( Im I2m I3m ... I 2^i m) con i <= log2N/M
Riempiamo la memoria con i documenti. Quando la memoria è piena, allora facciamo una sequenza di merge:  
riempio Im, se Im è vuoto, inserisco con un merge, se è pieno allora guardo l'indice successivo e così via.   Quando passo a una nuova Iim, butto tutto dentro al nuovo I svuotando gli I precedenti
- Im con 0 -> Im
- Im con Im -> I2M
- Im con Im, I2m -> I4m

.  # merges is log2 N/M
1 query trigger O(logN/M) query


## Compression of documents

Per i motori di ricerca, ritornare durane una ricerca parte del contesto è importante. È necessario quindi memorizzare i documenti (compressi).  

Due gli aspetti da tenere in considerazione: compressione veloce, e decompressione ancora più veloce!

### LZ 777

cerca la più lunga stringa che hai già incontrato, sostituisci con una tripla
<x, y, z>:

- x: quante poszioni indietro è la stringa già incontrata
- y: lunghezza della stringa attuale
- z: il carattere successivo (è necessario per inizio stringa, e quando non puoi copiare)

Quando scansioni, vai avanti finchè puoi, anche con overlapping.

In genere per trovare un match non si controlla tutta la stringa già analizzata, ma solo quella che rientra in una "finestra".  
**Larger the window, slower is the decompression and faster is the compression (read less triples)**  

Decompressione è fatta sequenzialmente, perchè se no non potremmo risolvere l'overlapping.  
Ma che succede se abbiamo len > dist? No prob se decompressione fatta sequenzialmente.


### Compression and networking

Se i dati devono essere inviati sulla rete, va tenuto conto di due fattori: 
- tempo di invio, che quindi porterebbe a prediligere tecniche di compressioni piu efficaci in modo da ridurre la quantità di dati da inviare
- tempo di decompressione

È necessario trovare un compromesso: comprimere "troppo" per minimizzare il tempo di invio è inutile se la decompressione diventa troppo lenta come conseguenza.



## Lezione ?.1 

### Top-k (approssimazione)

We use cosine similarity
Obbiettivo: trovare un sub set *k* 	di papabili elementi (Set A, |A| << N)

1. Cerco tutti i documenti in cui compaiono più query term possibili. Se la query contiene 4 termini (|q| = 4) in genere ne cerco almeno q - 1. Uso AND tra le liste. Abbastanza costoso.
2. High-IDF Usiamo liste in cui compaiono solo termini "rari" (liste più corte e più precise, perchè escludiamo i termini comuni e poco significativi con un basso rank, tipo articoli, "the", preposizioni)
3. Assegnare ad ogni termine i suoi *m* migliori documenti. Per ogni termine della query, prendiamo le liste di best doc e facciamo il merge. Computiamo cosine sim tra la query e i documenti ottenuti dal merge, e scegliamo i top *k* documenti. (NB deve valere *m* > *k*). C'è il rischio che il migliore documento per la query non rientri tra nessuno dei top *m* documenti di ciascun termine, quindi non è infallibile come sistema.
   t1 -> A, B, G  
   t2 -> C, D, G  
   t3 -> E, F, G  
   *m* = 2; *k* = 1
   G è il migliore ma non rientra tra i candidati dopo il merge!
4. Fancy hits: 
   Preprocess:
   - per ogni documento ho list decreasing order by PR -> incremento doc-id con il decrementare del PR
   - Prendo i top m documenti by tf-idf 
   - ...
5. Clustering: definisco per ogni cluster of doc un leader, la query viene comparata con i leader e sceglie quello pià vicino

Problema: questi approcci sono tutti approssimativi, vogliamo una soluzione esatta e efficiente.  
Ottimizzazione: non fare lo score di quelli che sappiamo non rientrare nei top k

WAND e WAND ottimizzato

## Lezione ?.2

precision = quanto accurato è il risultato di una query
recall = quante informazioni rispetto al totale sono state restituite da una query
NDCG = normalized discount cumulative gain


