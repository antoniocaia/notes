# IR (DRAFT) <!-- omit in toc -->

- [Lezione 1](#lezione-1)
	- [Intro](#intro)
	- [Boolean retrieval model](#boolean-retrieval-model)
	- [Inverted index](#inverted-index)
- [Lezione 2 (cont)](#lezione-2-cont)
	- [Skips](#skips)
	- [Recursive Merge](#recursive-merge)
	- [Zones](#zones)
	- [Crwaler](#crwaler)

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
- por ogni termine, creare una lista indipendente per ciascuno dei suo campi.
- mantenere una unica lista, ma specificare insieme all'ID del documento in quali campi appare il termine


# TODO <!-- omit in toc -->

## Crwaler

scc strongly connected components: 
due elementi a,b, si dicono ssc se a è collegato ad a e b è collegato ad a. Gli elementi di questo gruppo appartengono tutti allo stesso gruppo  

SSC centrale: agglomerato 

IN: contiene ssc, ma alcuni rami portano al SSC centrale senza poter tornare indietro

OUT: una volta entarto in un ssc in out, non puoi uscirne

tubes: collegano in e out ma skippano il core centrale



Da dove far partire il crwaler (i crawler usano i link per passare da una pagina all'altra)

Si fa partire i crwaler da IN (in modo da non ignorare nulla) e dal core, perchè contiene informazioni importanti


crawler usa tipo snapshot: fa il crowling, salva ciò che trova (snapshot), il crowling continua e aggiorna le modifiche, quando ha finito aggiorna un nuovo snaphot. (semplificato) 