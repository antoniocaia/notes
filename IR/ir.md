# IR (DRAFT) <!-- omit in toc -->

- [Lezione 1](#lezione-1)
	- [Intro](#intro)
	- [Boolean retrieval model](#boolean-retrieval-model)
	- [Inverted index](#inverted-index)
- [Lezione 2 (cont)](#lezione-2-cont)
	- [Skips](#skips)
	- [Recursive Merge](#recursive-merge)
	- [Zones](#zones)
	- [Web and web-browsers](#web-and-web-browsers)
		- [Spam](#spam)
- [Lezione 3](#lezione-3)
	- [Crawler](#crawler)
	- [Life-cycle](#life-cycle)
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
	- [Locality-sensitive hashing](#locality-sensitive-hashing)

## Info
Paolo Ferragina 

For meets with the teacher, connect to the Microsoft Team room, or write in the chat if the call isn't open.

Exam: 
- written pre-test and oral exam
- No mid-term exam
- You can use italian during exam

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


`TODO rimpiazzare termini con dizionario`

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

# Lezione 2 (cont)

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


## Life-cycle


Seed Page (SP): pagina contenente gli URLs utilizzabili da un crawler come punto di partenza.
Frontiers (F): lista di URLs che è necessario visitare.
Priority Queue (PQ): URLs da valutare
Assigned Repository (AR): Frontier
Page Repository (PR):

Un "crawler manager" preleva URL dalla PQ, se la pagina non è stata mai visitata o è stata modificata dall'ultima visita viene inserita in AR.  
Ogni pagina ha un "downloader": preleva da AR l'URL e scarica e compressa la pagina per salvare il risultato in PR.  
Il "link extractor" preleva da PR e cerca eventuali link, per inserirli in PQ.

Con che criterio vengono scelte le pagine dall'AR? Dipende dalla policy scelta: BFS, DFS, Random, Popularity driven, Page Rank, topic driven, combined.  

## Marcator
Marcator è un crawler che segue alcune semplici regole:
- una sola richiesta alla volta per host, in modo da non sovraccaricare l'host
- aspettare alcuni secondi tra una query e l'altra 
- preferire pagine con un alta priorità per il crawling

**Overview:**  
Un modulo preleva gli URLs nella F per poi distribuirli in K queue, una per ciascun livello di priorità.  
Un successivo modulo estrare dalle queue gli URLs preferendo quelli con priorità maggiore. Gli URLs sono distribuiti in B queue, dove ogni queue rappresenta un host, le pagine provenenti dallo stesso host vanno nella stessa queue. Ciò permette di regolare gli accessi agli host, evitanto multipli accessi contemporaneamente. Se una queue si "svuota", può essere riassegnata ad un nuovo host.  
URL selector preleva un URL da ciascuna delle B queue (una pagian per host) e gli assegna ad un min-heap. Il min-Heap si basa su dei time-stamp. Questo time-stamp è calcolata sommando il tempo di esecuzione della pagina precedente (dello stesso host) + un tempo di attesa extra.  
L'heap restituisce l'url con il tempo minore (p1), il crawler visita la pagina. L'heap deve rimpiazzare p1 con una pagina proveniente dallo stesso host (p2), e per fare ciò assegna come time-stamp a p2 un tempo pari a t(p1) + delta. Questo è applicabile nel caso p2 sia presente. Se per quell'host la queue è vuota, allora andiamo a prendere una pagina tra le front queue (stando attenti che non sia una pagina il cui host ha già una queue assegnata!).


## Check for page duplicate
- URL match
- Duplicate document match
- Near-duplicate document match

### URL match  
Un possibile approccio è quello di usare un hash-table per memorizzare gli URLs. La memoria necessaria è tuttavia eccessiva, ed anche il tempo di accesso è infattibile. Questo approccio è stato usato da Altavista ma solo finchè il numero di pagine lo ha consentito. 

### Bloom filter

Bloom filter usa:
- array binario (size m) inizializzato con soli 0
- k funzioni hash che, data un URL, ritorna un intero compreso tra 0 e m-1

Per controllare se un URL *u* è già stato visitato:
- si computano le k funzioni di hash con input *u*
- si controlla i valori nel vettore binario in corrispondenza degli indici i1, i2, ..., ik restituiti dall'hashing: se TUTTE le posizioni hanno valore 1, allora l'URL è già stato visitato.
  - Nel caso non tutti i bit siano 1, visitiamo la pagina e poniamo 1 nel vettore binario in corrispondenza degli indici i1, i2, ..., ik.

Questo approccio introduce possibili errori (falsi positivi):
- la possibilità che un bit sia 0 dopo l'inserimento di tutti gli URL effettivamente visitati  
`p = (1 - 1/m)^(kn) =  e^(-kn/m)` dove `n = numero URL`
- la possibilità di avere un falso positivo è quindi: `(1 - p)^k` la probabilità di avere un 1 per ognuna delle funzioni di hashing piuttosto che 0.
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
- utile per alcuni tipi di query (db, iceberg query) on uso esclusivo per search engine!

SBF utilizza:
- f(x): funzione che calcola la frequenza di x in un dato set S
- vettore (C) di dimensione m contenente le frequenze mappate dalle funzioni di hash
- h1...hk(x): k funzioni di hash

Come nel BF, le posizioni in C non sono univoche e possono presentarsi collisioni.

Procedimento:
- f(x) viene valutata, restituento una quantità q
- vengono calcolate le k funzioni h, ottenendo gli indici i1...ik
- I valori in i1...ik vengono incrementati di q.

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
- mapping dei crawler and url con la stessa funzione di hash. (Una buona funzione è `h(x) = a*x mod p` dove a casuale e p primo e p > max{I, S})
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


| Node | Outd  | Ref | # blocks| copy blocks | extra nodes |
| :--: | :--: | :--: | :--: | :--: | :--: |
| 15 | 3 | 0 |   |       | 10, 11, 12 |
| 16 | 4 | 1 | 3 | 1,1,0 | 13, 14     |


## Locality-sensitive hashing

Contesto vasto.   
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
- Introdurre fingerprint per far si che elemnti di U simili diventino uguali: facili da confrontare, costo ridotto

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