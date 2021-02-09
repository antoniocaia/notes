- [14.09.20](#140920)
	- [Boolean retrieval model](#boolean-retrieval-model)
	- [Matrix document-term](#matrix-document-term)
	- [Inverted list](#inverted-list)
- [15.09.20](#150920)
	- [Skip pointers](#skip-pointers)
	- [Zone index](#zone-index)
	- [Recursive Merge](#recursive-merge)
- [22.09.2020](#22092020)
	- [Crawling](#crawling)
		- [Life-cycle Crawler](#life-cycle-crawler)
	- [Mercator](#mercator)
	- [Bloom Filter](#bloom-filter)
- [28.09.2020](#28092020)
	- [Spectral Bloom Filter](#spectral-bloom-filter)
		- [Problemi in SBF](#problemi-in-sbf)
	- [Consistent Hashing](#consistent-hashing)
- [29.09.2020](#29092020)
	- [Locality-sensitive hashing (LSH)](#locality-sensitive-hashing-lsh)
		- [Funzionamento LSH](#funzionamento-lsh)
		- [Utilizzo LSH](#utilizzo-lsh)
	- [Exact-duplicate documents](#exact-duplicate-documents)
		- [Karp-Rabin's rolling hash](#karp-rabins-rolling-hash)
	- [Near-duplicate documents](#near-duplicate-documents)
		- [Shingling](#shingling)
		- [Jaccard similarity](#jaccard-similarity)
			- [min-hashing](#min-hashing)
		- [Cosine similarity](#cosine-similarity)
	- [BSBI (Blocked sort-based indexing)](#bsbi-blocked-sort-based-indexing)
		- [BSBI sorting (multi-way merge sort)](#bsbi-sorting-multi-way-merge-sort)
	- [SPIMI (Single-pass in-memory indexing)](#spimi-single-pass-in-memory-indexing)
	- [Distributed indexing](#distributed-indexing)
		- [DOCUMENT BASED](#document-based)
		- [TERM BASED: (MapReduce)](#term-based-mapreduce)
	- [Dinamic indexing](#dinamic-indexing)
	- [LZ77](#lz77)
	- [Z-delta](#z-delta)
	- [Rsync](#rsync)
	- [Zsync](#zsync)
	- [Parsing](#parsing)
		- [tokenization](#tokenization)
		- [normalization](#normalization)
		- [lemmatization](#lemmatization)
		- [stemming](#stemming)
		- [thesauri](#thesauri)
	- [Statistical properties](#statistical-properties)
		- [Zipf Law](#zipf-law)
		- [Heap Law](#heap-law)
	- [Keyword extraction](#keyword-extraction)
		- [Statistical](#statistical)
		- [Pearson's che-square (bigrams)](#pearsons-che-square-bigrams)
		- [Rapid Automatic Keyword Extraction](#rapid-automatic-keyword-extraction)
		- [Spell correction](#spell-correction)
	- [Edit distance](#edit-distance)
	- [n-gram](#n-gram)
		- [Wildcard query](#wildcard-query)
		- [Soundex](#soundex)
- [23.11.2020](#23112020)
	- [high idf](#high-idf)
	- [champion lists](#champion-lists)
	- [many query-terms](#many-query-terms)
	- [fancy hits](#fancy-hits)
	- [clustering](#clustering)
	- [WAND](#wand)
	- [blocked-WAND.](#blocked-wand)
# 14.09.20
## Boolean retrieval model
Una query booleana è una query in cui i termini sono messi in relazione tramite keywords booleane (**AND, OR, NOT ...**).  

## Matrix document-term
**Funzione:** struttura su cui processare query booleane.

Un primo approccio per processare query booleane fa uso di una tabella booleane in cui le righe rappresentano i termini e le colonne i documenti in cui ricercare.

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


## Inverted list
**Funzione:** struttura su cui processare query booleane.

Le inverted list rappresentano un'evoluzione delle matrici doc-term. Le matrici booleane sono sparse. Memorizzando solo i valori diversi da 0, è possibile occupare solo 1-3% della memoria necessaria alla matrice. 

Vengono utilizzate al posto della matrice delle liste (una per ciascun termine di ricerca) contenenti gli identificativi dei documenti. Ad ogni documento viene assegnato un identificativo univoco.
Due termini appartengono allo stesso documento solo se in entrambe le rispettive liste è presente lo stesso identificatore di documento.  

Riprendendo l'esempio della query `Antony AND Caesar`, quanti confronti sono necessari per verificare che due parole siano entrambe presenti nello stesso documento? Siano *m* e *n* le dimensioni delle due liste, si hanno due casi:
- liste non ordinate: **n\*m**
- liste ordinate: **n+m**

E' fondamentale le liste siano ordinate in modo da utilizzare un algoritmo che confronti sequenzialmente gli id delle liste, e possa incrementare il minore dei due ad ogni iterazione. L'intersezione delle posting list è infatti l'aspetto più critico:
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


Nel caso vengano utilizzati più termini di ricerca,vengono effettuati i confronti sempre a coppie, partendo dalle due liste con un numero minore di elementi e utilizzando il risultato come lista temporanea. In questo modo il risultato di ogni iterazione avrà come risultato una lista lunga al più quanto la più piccola delle due liste utilizzate.

# 15.09.20
## Skip pointers
**Funzione:** ottimizzare lo scorrimento parallelo delle posting list  

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

## Zone index
**Funzione:** ottimizzare l'utilizzo di query associando alle diverse parti di un documento dei metadati.

Attraverso un zone index è possibile specificare in quale parte di un documento (titolo, corpo, autore) si voglia andare a ricercare i termini.

## Recursive Merge
**Funzione:** ottimizzare lo scorrimento parallelo delle posting list, alternativa agli skip pointers

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

# 22.09.2020
## Crawling

Il web crawling è il processo attraverso il quale è possibile raccogliere e indicizzare pagine in web in modo da supportare il lavoro di un motore di ricerca.

Le caratteristiche che un crawler deve possedere sono molteplici
- Robusto: essere in grado di riconoscere ed evitare "trappole"
- Analisi di qualità: il crawler non deve procedere casualmente, ma visitare le pagine migliori per prime
- Etiquette: gli host in genere specificano all'interno di un file *Robots.txt* cosa è permesso fare al crawler e cosa no. In genere si vuole evitare un sovraccarico del web server, e intervallare l'invio di query. In alcuni casi non si vuole indicizzare determinate pagine.
- Efficienza: evitare duplication e near-duplication

Quando si fa crawling, le considerazioni da fare sono:
- Copertura: quanto è grosso il web e quanto ne vogliamo coprire
- Copertura relativa: la competizione (altre aziende) quanto web indicizzano di già?

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

## Mercator
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

## Bloom Filter
**Fuzione:** il Bloom filter è una tecnica utilizzata per determinare se una pagina è già stata visitata (da un crwaler) in modo da evitare *page duplication*.  

Un Bloom Filter prevede l'uso di:
- array binario *B* (dimensione *m*) inizializzato con soli 0
- *k* funzioni hash che, dato un URL, ritornano un intero compreso tra *0* e *m-1* (una posizione dell'array *B*)

Per controllare se un URL *u* è già stato visitato:
- si computano le *k* funzioni di hash con input *u*
- si controlla i valori nel vettore binario in corrispondenza degli indici *i1, i2, ... , ik* restituiti dall'hashing: se TUTTE le posizioni hanno valore 1, allora l'URL è già stato visitato.
  - Nel caso non tutti i bit siano 1, visitiamo la pagina e poniamo 1 nel vettore binario in corrispondenza degli indici *i1, i2, ... , ik*.

Questo approccio introduce possibili errori (falsi positivi):
- la probabilità che un bit sia 0 dopo l'inserimento di tutti gli URL effettivamente visitati è  
`p = (1 - 1/m)^(kn) =  e^(-kn/m)` dove `n = numero URL`
- la possibilità di avere un falso positivo (ERRORE) è quindi: `(1 - p)^k`, ovvero la probabilità di avere un 1 per ognuna delle funzioni di hashing piuttosto che 0.
- Come minimizzare l'errore:
  - il numero ideale di funzioni di hash si ha da `k = (m/n)*ln(2)`, sostituendo si ottiene  circa `p = 0.62^(m/n)`. Questo vuol dire che aumentando il rapporto tra *m* ed *n* l'errore diminuisce.  
  Ad esempio, con un rapporto di 10 (ovvero usare 10 bit per URL) si ottiene 0.0084 (0.84% di probabilità) che risulta essere un valore accettabile.
- Attraverso il bloom filter c'è un risparmio considerevole di memoria: con l'hashing memorizzare la stringa di un URL richiede in media 8 * 1000 bit, mentre con il bloom filter è possibile memorizzare anche solo 10 bit per URL.
- Quante funzioni di hash sono necessarie? poichè `ln(2)` è trascurabile, scegliendo `m = 10n` k diventa circa 10.

# 28.09.2020	
## Spectral Bloom Filter

Spectral Bloom Filter rappresenta una evoluzione del BF e le sue applicazioni vanno oltre la *page duplication* (Aggregate query)  
Vantaggi SBF:
- miglior performance a scapito di un uso di memoria leggermente maggiore
- inserimento e eliminazione possibili

SBF utilizza:
- *f(x)*: funzione che calcola la frequenza di *x* in un dato set *S*
- vettore *C* di dimensione *m* contenente le frequenze mappate dalle funzioni di hash (*C* sostituisce il vettore binario in BF)
- k funzioni di hash (*hi*)

Come nel BF, le posizioni in C non sono univoche e possono presentarsi collisioni.

Procedimento:
- f(x) viene valutata, restituento una quantità *q*
- vengono valutate le k funzioni hash, che restituiscono gli indici *i1, i2, ... , ik*
- I valori in *i1, i2, ... , ik* vengono incrementati di q.

Per l'**inserimento**, basta valutare le funzioni h(x) e incrementare di uno i contatori corrispondenti.  
Per la **rimozione**, come per l'inserimento ma decrementando.  
Per la **ricerca**, prendo il valore minore tra quelli puntati dalle funzioni di hash.


### Problemi in SBF 
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
- Se è presente un *recurring minimum* mx (il valore minimo appare più volte in corrispondenza degli indici di hash, per cui posso supporee che dovrei avere almeno due posizioni non alterate) allora ritorno mx.
- Se non ho RM allora esiste il rischio che tutti i valori siano stati alterati (almeno k-1 lo sono) e che lo sia anche il minimo.
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

## Consistent Hashing

# 29.09.2020	
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
- **LSH**: introdurre fingerprint per far si che elemnti di U simili diventino uguali: facili da confrontare, costo ridotto.  
  Appunti prof: https://www.dropbox.com/s/ovavpl1s0yu71fo/Archivio29.09.2020.zip?dl=0

### Funzionamento LSH

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

*Aumentando K riduciamo i falsi positivi (K = d comporta nessun falso positivo), <del>ma aumentiamo i falsi negativi<del>.*

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
P(g(p) = g(q)) = 1 - [1 - S^K]^L
```

Come varia la probabilità al variare di S?  
Banalmente, al ridursi della similirità, diminuisce la probabilità.  
La funzione tuttavia non raggiunge mai lo zero e neanche l'uno.

### Utilizzo LSH

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


## Exact-duplicate documents

### Karp-Rabin's rolling hash

Usato per verificare se due "finestre" di bit sono uguali.   
Sequenza *a* di *m* bit. Poichè vogliamo associare a tale sequenza una valore intero, aggiungiamo un bit aggiuntivo a sx della sequenza e lo poniamo a 1.  
Si prende un valore primo *p* in *U* tc 2*p occupi pochi byte. (per cui in genere U = 2^64).  
La funzione di fingerprint è f(a) = a mod p.  

Se `a = 1 a1 a2 ... am`  
e `b = 1 a2 a3 ... am+1`  
dove quindi |a| = |b| e *b* è equivalente allo shift a sx di *a*  
è possibile valutare l'hash di b senza bisogno di ricomputare la funzione ma con semplici operazioni aritmetiche.

> Now, suppose we are given hq(T[i . . .(i + p − 1)]) and want to compute hq(T[(i + 1). . .(i + p)]).
What do we do?  
Easy. Take hq(T[i . . .(i + p − 1)]), subtract T[i] · 2^(p−1) mod q from it (note that T[i] · 2^(p−1) mod q is either 0 or 2p−1 mod q, and we can precompute and store 2^(p−1) mod q), then multiply by 2, and add in T[j + 1]. (All modulo q, of course.) A constant number of arithmetic operations modulo q, as advertised.  
[source](http://www.cs.cmu.edu/afs/cs/academic/class/15451-f14/www/lectures/lec6/karp-rabin-09-15-14.pdf)


## Near-duplicate documents

### Shingling

Separare documenti in *q-grams* (shingles), ovvero set di q termini contigui.  
Due documenti sono simili se i rispettivi insiemi di shingles si intersecano suffcientemente.  

Maggiore è *q*, maggiore sarà la precisione con cui appurare la somiglianza tra due documenti. Tuttavia, all'aumentare di q aumentiamo esponenzialmente l'impatto di ciascun termine sull'insime di shingles. Una parola cambiata può influire su un numbero altissimo di q-gram (4 < q < 8).  
Come valutare l'intersezione tra due insiemi di shingles Sa Sb ? 
- intersezione non è sufficiente, in quanto influenzata dalla lunghezza dei documenti.
- Jaccard similarity è una stima molto più precisa

### Jaccard similarity

Jaccard similarity `(sim) = Sa inter Sb / Sa union Sb`  

#### min-hashing

Come ottenere Jaccard-sim? Uso Min-hashing  

Set di shingles. Trasformiamo le stringhe in valori numerici, modulo 2^64.  
Calcolare l'intersezione richiede iterazioni, ed è troppo costoso! Soluzione:  
- permuto con funzione `p(x) = ax + b mod 2^64` con *a* e *b* co-primi
- prendo i minimi di ciascun set
- `P( min(p(Sa)) min(p(Sb)) )  =  jac-sim(Sa, Sb)  =  Sa inter Sb / Sa union Sb`  

for the dummies like me:  
- per ogni set di shingles si valutano *m* funzioni di permutazione *p* (da cui otteniamo lo sketch di dimensione *m* di ciascun set)
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

### Cosine similarity

La cosine similarity fa uso di una rappresentazione vettoriale dei documenti, e risulta particolarmente vantaggiosa quando ad essere confrontati sono documenti di lunghezze molto diverse.  

Reminder:
- Prodotto scalare *X*: `sommatoria (pi * qi)` dove *p* e *q* sono due vettori
- `cos(a) = p X q / norm2(p) * norm2(q)` dove `norm2(p) = ( sommatoria pi^2 )^1/2` e `norm2(p) -> |p|`
- `-1 <= cos(a) <= 1`

```
cos_sim(d1, d2) = [ V(d1) X V(d2) ] / [ |V(d1)| |V(d2)| ]
```
Valutando l'angolo tra V(d1) e V(d2) è possibile ignorare la dimensione della pagina.

 <!-- Todo normalizzazione dei vettori -->

## BSBI (Blocked sort-based indexing)

BSBI è una tecnica per la creazione di inverted list.

Per collezioni che è possibile gestire direttamente in memoria, è possibile seguire i seguenti passi:  
Ogni documento è scansionato, ed ad ogni singolo termine (quindi sono compresi i duplicati) viene assegnato l'id del documento in cui è presente (*docId*).  
I termini vengono prima ordinati alfabeticamente, poi raggruppati per termine e in fine raggruppati per *docId*.  
In questo modo è possibile costruire per ogni termine la lista di *docId* in cui compare (posting list); oltre ad altre info come la frequenza.

Per collezioni di dati che non è possibile tenere interamente in memoria vengono applicate alcune modifiche:  
Non vengono usati i valori effettivi dei termini ma piuttosto un valore univoco intero *termId*.  
Per quanto riguarda il sorting dei termini, è necessario un algoritmo che utilizzi il disco poichè la memoria risulta insufficiente.

### BSBI sorting (multi-way merge sort)
- i documenti sono divisi in blocchi facilemnte gestibili nella memoria.
- sequenzialmente i blocchi vengono prelevati dal disco e caricati in memoria.
- quando un blocco è caricato in memoria viene creata la corrispondente posting list.
- la posting list è memorizzata sul disco, e il successivo blocco viene caricato ed elaborato.
- Come passo finale, viene fatto il merge di tutte le posting list. Dal disco vengono progressivamente caricati per ogni list i termini con termId minore, e di questi viene fatto il merge, l'algoritmo termina quando tutte le list sono vuote.

**BSBI deve mappare i termId con i termini effettivi, e per collezzioni troppo grandi questo risulta un problema.**

## SPIMI (Single-pass in-memory indexing)

<!-- Todo -->

SPIMI rappresenta una evoluzione di BSBI.   
Piuttosto che lavorare su blocchi di dimensione statica, SPIMI lavora co posting list dinamiche.  
- Finchè memoria è disponibile, viene genrato un token docId termId.
- Il token viene aggiunto alla posting list
  - Se la posting list è piena (la memoria del blocco allocato temporaneamente è eusarita) allora raddoppio la dimensione del blocco)
- Una volta terminata la memoria, la posting list viene ordinata e scaricata sul disco.
- Come ultimo passo, tutte le liste vengono unite



## Distributed indexing
Sistema distribuito per la creazione di IL con Big Data.

### DOCUMENT BASED

- Ogni set è assegnato ad un *parser*: il parser genera la tabella idTerm idDoc
- Ad ogni parser è associato un *inverter*, il parser invia la tabella e l'*inverter* crea l'inverted list.
- Al termine, avremmo tante inverted list

Quando facciamo una query, essa deve essere processata su diverse Inverted List!

### TERM BASED: (MapReduce)

- Ogni set è assegnato ad un *parser*, e il parser genera la tabella (le coppie idTerm idDoc) e poi *parziona il risultato secondo un criterio arbitrario (es, 3 gruppi basandosi sulla prima letterea del termine)*
- Ogni diversa partizione è inviata ad uno specifico inverter. (es, un inverter per ciascun gruppo; inverter per il primo gruppo, inverter per il secondo, inverter per il terzo)
- Gli inverter creano le IL

Un buon criterio di partizionamento deve essere effettuabile da ogni parser, per esempio hashing.

## Dinamic indexing

<!-- Todo -->
PAG 115

## LZ77 
<!-- Todo -->

## Z-delta
<!-- Todo -->

## Rsync
<!-- Todo -->

## Zsync
<!-- Todo -->

## Parsing 

### tokenization

La trasformazione dei termini in token non deve cambiare il significato di certe parole.
Es: nomi e cognomi (specialmente di personaggi famosi)   
Es: nomi composti (San Francisco)  
Es: Numeri  


### normalization

Normalizzazione prevede dove opportuno di eliminare caratteri speciali che alterano la forma di un termine senza cambiarne il significato.  
Es: gli acronimi, come U.S.A e la versione "semplice" USA.  
Es: '-' usato per separare termini di parole composte  

### lemmatization

Ricondurre variazioni/declinazioni alla forma base di una parola.  
Es: tutti i verbi coniugati vengono riportati alla loro forma all'infinito.  
Es: forme plurali o abbreviazioni vengono riportate alla loro forma base singolare.

### stemming

Troncamento dei termini in modo da ottenere la radice di una parola. Attraverso la radice è possibile risalire a termini con significato uguale o simile. 

### thesauri

Gestione di sinonimi e omonimi.

## Statistical properties
### Zipf Law

Zipf law prevede che la distribuzione dei token/termini in un testo sia uniforme:
- pochi token sono molto frequenti
- alcuni token hanno una frequenza media
- molti token sono poco frequenti

I primi 100 token costituiscono in genere il 50% del testo, e molti di questi token sono stop words.

Le stop words sono quei termini di uso molto frequente ma quasi privi di significato, come articoli, preposizioni, etc.  
Utilizzando buone tecniche di compressione e di ottimizzazione le stop words hanno un impatto molto ridotto.

Formalmente la Zipf Law prevede che il *k-th* token abbia frequenza di *1/k*.  
Equivalentemente, il prodotto tra la frequenza e il rank *k* è costante, per cui:  
```
k * f(k) = c
f(k) = c / k

Da cui si ha la legge generale:
f(k) = c / k^s		s = 1.5-2.0
```
Zipf Law è una *power law* e il suo grafico log-log è quindi una retta

### Heap Law

La Heap Law definisci la crescita del numero di token distinti.
Se *n* è il totale numero di token, allora si hanno *n^b* distinti token, con *b* circa 0.5.  

La heap Law suggerisci due importanti considerazioni:  
- la dimensione del dizionario continua a crescere all'aumentare dei documenti nella collezione (invece di arrestarsi ad un certo punto)
- La dimensione del dizionario risulta essere considerevole per grandi collezioni di documenti.

<!-- Todo LUHN -->

## Keyword extraction 

Collocation: due o più parole che, quando affiancate, assumono un significato particolare.  
I termini di questi particolari costrutti non possono essere sostituiti con sinonimi o modificati, e spesso non è possibile comprenderne il significato dall'analisi dei suoi termini.

### Statistical

Un approccio statistico usa frequenza e Part-of-Speach tagging. 
Utilizzare solo la frequenza di coppie di termini è poco utile a causa dell'alta frequenza di stop words.  
Introducendo PoS è possibile definire un ranking utilizzando solo alcune tuple (Es, ignoriamo le coppie articolo-preposizione e consideriamo invece le coppie nome-nome, nome-aggettivo, etc)

Utilizzare frequenza e PoS tuttavia non tiene conto della flessibilità della lingua: due parole "connesse" possono trovarsi a una distanza (numero di termini che separano le due parole) variabile.  
Viene quindi computata la media e la viarianza delle distanze all'interno di una finestra. Vengono scartate le coppie con varianza troppa alta, e prese quelle con media > 0 e varianza piccola.

### Pearson's che-square (bigrams)

Da qui:
https://github.com/rmassidda/ir-2019

### Rapid Automatic Keyword Extraction

Da qui:
https://github.com/rmassidda/ir-2019

### Spell correction

## Edit distance

## n-gram


### Wildcard query

Caso generale:
- utilizzo un B-tree per le query in cui *\** compare alla fine 
- utilizzo un reverse B-tree per le query in cui *\** compare all'inizio
- per le query in cui *\** compare in una posizione intermedia (sc*ool) viene utilizzato il B-tree per la sottoquery sc\*, il reverse B-tree per la sottoquery \*ool, e infine viene fatta l'intersezione dei risultati.

L'intersezione è troppo costa, viene quindi utilizzato un permutem index.  
Aggiungendo il carattere *$* alla fine della query e ruotanto i caratteri è possibile ricondursi ad il caso con *\** al termine della query.


### Soundex

Alterare le query in modo che spelling diversi ma con suoni simili diano gli stessi risultati.  
Si creano set di lettere da sostituire con valori generici. 




# 23.11.2020
## high idf

## champion lists

## many query-terms

## fancy hits

## clustering

## WAND 

## blocked-WAND.



- Cos'è LSH and why we introduce it?
  - How LSH works?
  - Assume the vector is binary (length D) and the vectors have Q diff, calc S
  - What if we have a vector of CHAR, we need to change something? 
  
- PForDelta
  - What happen with a sorted array?

- Dice e Jacardi distance?
  - Perchè è necessario introdurre altri sistemi invece di Jacardi?
  - Due vettori di interi, come trovare i vettori simili? Contare le posizioni differenti -> Hamming
  - Modifica dinamica degli indici, tecnica esponenziale -> Logaritmic merge
  
- Web Graph -> compressione del grafo, quali proprietà sono richieste?
- Copy Block, alcuni nodi sono extra, cosa si può dire? (Compressione? delle liste)
  - Fingerprint Karph Robin
  - dato un numero a, massimo numero di divisori? Ristosta log(a)

- Test X square

- Indicizzazione (two leve indexing)
  - trie
  - Come memorizzare?
  
- Due leve indexing -> come funzioana la ricerca in memoria interna sul trie?

- Z delta compression	
- SPIMI e co
- soft end, che vuol dire	

FILE aabbaa (new)    aab (old)

- Consisten HASHING

- Zone INDEX
- COME CAMBIARE LA POSTING LIST PER SUPPORTARE LA QUERY?
- Heap Laws, Ziph, Luhn

Tecnica per trovare approssimata i TOP-K documenti 

Relazione tra la Overlap distance e la Edit distance

LSH -> online (clust) e offline per un dizionario	

Variable byte = t-nibble

Applicazione del bloom filter, individuazione dei virus

One error dictionary ? 






WHAT 

cosine sim (11)
SPIMI
Pearson’s chi-squared test (16)
2 lvl ind, come lavoro nei blocchi?
A cosa servono le inv ind in Overlap Dist?
Wildcard