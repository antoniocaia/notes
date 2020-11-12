# LCI (Draft) <!-- omit in toc -->

- [Lezione 1](#lezione-1)
	- [Compilatore](#compilatore)
	- [Front end](#front-end)
	- [Ottimizzatore](#ottimizzatore)
	- [Back end](#back-end)
- [Lezione 2](#lezione-2)
	- [Linguaggi formali](#linguaggi-formali)
	- [Linguaggi regolari](#linguaggi-regolari)
- [Lezione 3 (cont)](#lezione-3-cont)
	- [Pumping Lemma](#pumping-lemma)
	- [Linguagggi Context Free](#linguagggi-context-free)
- [Lezione 4](#lezione-4)
	- [Analisi lessicalse](#analisi-lessicalse)
- [Lezione 5](#lezione-5)
	- [Tecniche di parsing](#tecniche-di-parsing)
	- [Top-down parser LL(1)](#top-down-parser-ll1)
- [Lezione 6](#lezione-6)
	- [LL Parsing](#ll-parsing)
- [Lezione 7](#lezione-7)
	- [Bottom up parser](#bottom-up-parser)
- [Lezione 8](#lezione-8)
	- [Costruzione tabelle ACTION e GOTO](#costruzione-tabelle-action-e-goto)
		- [Computing Closures](#computing-closures)
		- [Computing Gotos](#computing-gotos)
		- [Filling Action and Goto tables](#filling-action-and-goto-tables)
- [Lezione 9](#lezione-9)
- [Lezione 10](#lezione-10)



# Lezione 1

## Compilatore
- front end: dipende principalmente dal codice sorgente (es linguaggio utilizzato)
- back end: dipende dalla macchina su cui viene compilato il codice (es architettura processore)

Tra il front end e il back end viene utilizzato un linguaggio intermedio IR 

## Front end

Composto da:
- scanner: lo scnaner analizza il codice sorgente carattere per carattere, suddividendo in tokens (elementi che hanno un "significato" indipendente all'interno del linguaggio)
e assogna loro un "ruolo" creando così delle coppie o pairs. Questa funnzione è l'**lessical analysis**. Non si preoccupa che la sequenza abbia significato.
- parser: esegue **syntax analysis**, seguendo una grammatica libera (context free), solo successivamente verrà verificato se quello che è stato scritto a senso (quando dichiaro una variabile devo usare < tipo > < nome >. Costruisce un abstract syntax tree (AST) che può essere visto come una forma di IR. 


## Ottimizzatore

Si passa da IR in un altro IR.
Funzioni:
- cerca e propaga costanti: se ho una costante in un registro, e trovo una nuova costante con lo stesso valore, è inutile usare un nuovo registro quando il valore è già stato memorizzato una volta.
- Rimuove ridondanza
- Rimuove codice irraggiungibile

## Back end

Trasforma il IR code in machine code. Prende un IR (per esempio l'AST). Deve essere fatta in maniera ottimale, cercando di ottimizzare. Una altra forma intermedia è iloc ("ailoc") sembraassembler
composto da:
- Instruction selection: cerca di trasformare l'IR in codice macchina
- Register allocation: quali e quanti registri?
- instruction scheduling: cambiando l'ordine delle istruzioni possiamo ottimizzare i tempi?

Spesso bisogna scendere a compromessi tra memoria e tempo.

# Lezione 2

## Linguaggi formali

**Alfabeto (E)**: è un set finito di simboli.  
**Stringa:** si dice string su l'alfabeto E una sequenza finita di simboli di appartenenti a E  
**Epsilon** è la stringa vuota.	Lunghezza di string è il numero di simboli che compongono la stringa |x|. Concatenazione di stringhe, termine neutro è epsilon. Sottostringa: parte di una string, se x = ysz, y = prefisso.

**Potenze di un alfabeto E**: E^n sono tutte le stringhe di lunghezza n ottenibile combinando gli elementi dell'albafeto E. E^+ è l'unione di tutte le stringhe di qualsiasi l'unghezza eccetto la string vuota, E^* = E^+ U epsilon

**Linguaggio (L)**: sottonsieme di stringhe di un alfabeto.  
Su linguaggi si possono fare le seguenti operazioni: Unione Intersezione Differenza Complementare Concatenazione.  
**Chiusura di Kleene:** L* = UL^i

**Grammatica**: partendo da una grammatica (e le sue regole) è possibile generare tutte le stringhe di un linguaggio. E' possibile controllare se una stringa appartiene alla grammatica.

**G = (E, N, S, P)**
- E: alfabeto
- N: simboli non terminali
- S: simboli iniziale
- P: insieme di produzioni (es: A -> B)

**Classificazione grammatiche:**
- regular 3
- context free 2
- context sensitive 1
- Phrase structure 0

**La stringa w appartiene a L?**

| Grammatica |    Costo    |
| :--------: | :---------: |
|     3      |      P      |
|     2      |      P      |
|     1      | PSPACE (NP) |
|     0      | undecidable |



## Linguaggi regolari

Possibili rappresentazioni:
- Grammatica regolare (RG)  
- Determinist finite automata (DFA)  
- Non-determinist finite automata (NFA)  
- Non-determinist finite automata epsilon (epsilon-NFA)  
- Regular Expression (RE)  


**Equivalenze**  
````
RE <- DFA <-> NFA <-> RG  
RE -> epsilon-NFA <-> NFA
````

**Grammatica regolare:**  
G = (E, N, S, P)  
ES:  
L = { a^n b^n | n,m > 0 }  
S->aS|B  
B->bB|b  
Per ogni regola, un solo simbolo terminale, sempre o solo a dx o solo a sx, e un solo simbolo non terminale.


**Automa finito deterministico:**  
DFA (Q, E, o, qo, F)
- Q: insieme di stati
- E: alfabeto
- o: funzione di transazione Q x E -> Q 
- qo: stato iniziale
- F: insieme degli stati finali

Chiusura transitiva: funzione di transizione definita per una stringa ricorsivamente.

Data una GRAMMATICA REGOLARE posso costruire un NFA equivalente e viceversa.

**RG -> NFA**  
Creo uno stato per ogni simbolo non terminale + uno stato F (finale)
Per ogni regola della grammatica costruisco un collegamento tra due stati in base al simbolo non-terminale. Se non c'è simbolo terminale, vado in F.


**RG <- NFA**  
Per ogni arco del grafo, deinisco una regola. Per gli stati non finali ogni stato diventa un simbolo non terminale; per gli stati finali allora aggiungo una regola in cui non campaiono simboli non finali.

**NFA -> DFA**  
Definiamo nuovi stati in base a tutte le possibili combinazioni di stati già esistenti. (con q0 e q1 otterrei q0, q1, q0q1, vuoto). 
Osservando l'automa iniziale, definisco le nuove transazioni, e se con uno stesso valore una transazione andava in più stati diversi, adesso va nello stato descritto dalla combinazione di quegli stati (q0,a -> q0 e q1,a -> q1 diventa q0,a -> q0q1).  
Per quanto riguarda gli stati generati dalle combinazioni di stati, è necessario unire i "risultati delle transazioni":  
Se q0,a -> q0 e q1,a -> q1 allora lo stato q0q1 con q0q1,a -> q0q1. Perchè? La sua "parte q0" con a lo mandava in q0, la "parte q1" lo mandava in q1, mettendo insisme i risultati si ha q0q1. Procedere per tutti gli stati.

**NFA <- DFA**  
Ez

**Epsilon chiusura e e-NFA**
L'epsilon chiusura di uno stato è l'insieme di stati raggiungibili con epsilon transizioni U all'epsilon chiusura di tali stati. In soldoni, è l'insime di stati che posso raggiungere utilizzando solo epsilon transazioni.

**e-NFA -> NFA**  
Basta considerare gli stati connessi da e-transizioni come "lo stesso stato". 
Quindi guardo come si comportano tutti gli stati appartenenti alla mia e-chiusura.

# Lezione 3 (cont)

**DFA -> RE**
Eliminare gli stati, rimpiazzando gli archi con espressioni regolari includendo il comportamento dello stato eliminato. Idealmente vogliamo ottenere un solo stato finale, un solo stato iniziale. Nel caso di più stati finali, analizzarne uno alla volta singolarmente e poin + i risultati.

**RE -> e-NFA**
Guardare le slide per le regole meccaniche.

## Pumping Lemma

Il pumping lemma è utilizzato per determinare se un linguaggio non è regolare. 

```
Sia L regolare, allora esite un k tc per ogni z appartenente a L 
|z| >= k 
z = uvw con |uv| <= k, |v| > 0

tc per ogni i in N 
u(v^i)w appartiene a L
```

Il pumping lemma definisce una condizione di esistenza, trovando quindi un solo caso che non rispetta il PL dimostriamo che il linguaggio non è regolare.

## Linguagggi Context Free

// TODO

- automi a pila
- pumping lemma per CF
- proprietà

# Lezione 4

A noi interessano solo grammatiche regolari e grammatiche context free, le altre sono troppo complesse (e quindi costose).

Grammatiche regolari -> scanner
Grammatiche contex free -> parser

## Analisi lessicalse

Verificare che il codice sorgente sia composto da termini che appartengano al linguaggio e classificare i token in base alla loro funzione.

Un lexical analyser può essere realizzato "a mano" o automaticamente.

Scrivere il codice per uno scanner: 
```
Char ← next character
State ← s0

while (Char ≠ EOF)
	Next ← δ(State,Char)
 	Act ← α(State,Char)
 	perform action Act
 	State ← Next
 	Char ← next character 

if (State is a final state)
	then report success
	else report failure
 ```

δ è codificata mediante una tabella, permettendo l'accesso con costo O(1). 
α, strettamente legata a δ, restituisce informazioni aggiuntive, come il tipo di token che stiamo costruendo.

To convert a specification into code:
1. Write down the RE for the input language
2. Build a ε-NFA collecting all the NFA for the RE
3. Build a NFA corresponding to the ε-NFA
4. Build the DFA that simulates the NFA
5. Systematically minimise the DFA
6. **Turn it into code**

Tre possibili approcci per il passaggio a codice:
- Tabella delle transizioni
- Direct code: corrispondenza tra stati e codice (un po' come se creassi un oggetto o funzione per simulare il comportamento di ciascun stato, ovvero le sue transizioni)
- hand coded

Ciò che cambia è la "tabella delle transizioni".  
In ogni caso devono tutti e tre gli approcci scorrere carattere per carattere, riconoscere una parola (nel caso lo stato non abbia archi e sia finale) oppure effettuare il roll back nel caso di bisogno.

Scanner "tokenizza" il source code abbinando ad ogni parola un ruolo, il parser costruisce l'albero di derivazione.

# Lezione 5

## Tecniche di parsing

Parser usa linguaggi CF. Tuttavia non è possibile descrivere tutte le regole di un linguaggio di progragrammazione con un CF, specialmente per quanto riguarda espressioni vincolate ad altre espressioni:  
`foo = fucn(17)` potrebbe risultare corretta sintatticamente (variabile, assegnazione, funzione, parentesi, parametro, parentesi), ma come possiamo essere sicuri che `foo` sia effettivamente una variabile?  
Sono necessarie operazioni addizionali.

Ricevuta un istruzioni in input, il parse rgenera una derivazione nel tentativo di matchare l'input.  

Due tipi di derivazioni:
- sinistra ("sostituisco/derivo" il termine non finale più a sinistra)
- destra

**Problema**: derivazioni sx e dx di una stessa espressione possono dare alberi di parsing diversi, dando un ordine di priorità diverso alle istruzioni.  
**Soluzione**: pr rimuovere l'ambiguità, si affina la grammatica dando un grado di priorità alle struzioni.

**Problema**: è possibile ottenere derivazioni sx diverse che portano allo stesso risultato, e ciò crea problemi al parser che vorrebbe essere il più "deterministico" possibile.  
Una grammatica ambigua può portare problemi (es:  *if* exp *then* stm [*else* stm])


## Top-down parser LL(1)

- Partenza dalla radice
- Backtrack per correggere scelte errate

Top-down parsing algorithm:
Construct the root node of the parse tree.  
Repeat until lower fringe of the parse tree matches the input string.  
1. At a node labeled A, select a production with A on its lhs and, for each symbol on its rhs, construct the appropriate child.
2. When a terminal symbol is added to the border and it doesn’t match the border, backtrack.
3. Find the next node to be expanded. 

**Problema**: essendo l'algoritmo automatico, potrebbero presentirsi loop.  
Se viene usata derivazione sinistra (dx), NON può esserci ricorsione sinistra (dx). 
**Soluzione**: usare algoritmo per l'eliminazione diretta e indiretta della ricorsione.

Per il problema legato al backtracking, è possibile unsare una sorta di look-ahead per avere una sorta di contesto, ed evitare di imboccare strade errate. Maggiore è il numero K di caratteri che si scansionano, maggiore sarà la precisione, minore sarà l'efficienza. L'approccio migliore si ha con k = 1 da cui l'algoritmo LL(1)


# Lezione 6

## LL Parsing  

Quando scelgo una derivazione voglio avere informazioni necessarie per poter fare la scelta giusta. Se ho A -> a | b , voglio sapere quale delle due scelete è quella giusta in modo da non dover fare backtracking.  

**first(A -> a)**: insieme di simboli, x appartiene a first(A->a) (dove a è un simbolo non terminale) se x è il primo carattere generato da a.

Una proprietà necessaria affinche si abbia una grammatica LL(1) è che per ogni derivazione, se A -> a e A -> b allora necessariamente:  
first(A->a) intersecato first(A->b) = vuoto.  

Caso speciale: epsilon-produzioni.  
È necessario definire prima di tutto la funzione follow.  
**follow(A)**: l'insieme di simboli terminali che seguono la derivazione A.

**first+(A->a)**:
- first(A->a) U follow(A) se a first(A->a) appartiene epsilon
- solo first(A->a) altrimenti

**Recursive descent parser**

Predictive parser.  
"Scorro" la derivazione, senza far nulla finchè non ho un errore o arrivo alla fine. A quel punto "risalgo" e inizio a ritornre true (o false in caso di errrore) e a costruire la sequenza partendo dal fondo.

**Trasformare una grammatica in LL(1)**
NON è sempre possibile, in alcuni casi si.

ES:  
`A -> xy` e `A -> xz` si ha first+(A->xy) intersecato first+(A->xz) != vuoto  
La grammatica non è LL(1), ma per portarla si può estrarre la parte comune:  
`A -> xA'`  
`A' -> y`  
`A' -> z`

**Costruire parser per LL(1)**
Input: LL(1) grammar, first() set and follow() set.  

Due opzioni: 
- una funzione per ciascun non terminale (cascata di if else...) con return true-false
- tabella TopDown (row per simboli non terminali, column per simboli terminali) + [algo (slide 71)](http://pages.di.unipi.it/gori/Linguaggi-Compilatori2020/ParsingI-II.pdf)

# Lezione 7

## Bottom up parser

Tutte le [slide e (algo slide 28)](http://pages.di.unipi.it/gori/Linguaggi-Compilatori2020/Bottomup.pdf).

Bottom up costruisce una derivazione "al contrario": invece di partire da lo stato iniziale e cercare di arrivare all'input scegliendo una derivazione piuttosto che un'altra, il bottom up parte dall'input e associa progressivamente la regola che ha condotto a quell'input.

Bottom un costruisce una rightmost derivation (al contrario) e per fare ciò deve sostituire la sottostringa sinistra.  

handle = < A -> B, k > dove k è simbolo più a sinistra della derivazione A -> B

Ogni sottostringa a destra di un handler è composta da soli simboli terminali (perchè è una derivazione destra!)

Se la grammatica non è ambigua, ogni iterazione ha un unico handle

Problema handler: come si generano senza calcolare troppe derivazioni? Soluzione: lookahead


# Lezione 8

## Costruzione tabelle ACTION e GOTO

Definire funzioni 
- goto(stato s, simbolo non terminale o terminale X)
- closure(stato s)

item: coppia P e delta (d)
- P è una produzione A -> B con un punto (°). Il punto indica il top dello stack!
- delta è il lookahead di lunghezza al più 1

Esempio:  
```
A -> beta  
con beta = By e lookahead a  
si hanno gli items:
[A -> °By, a]
[A -> B°y, a] B nel top dello stack!
[A -> By°, a] By è stato processato e il lookahead a è stato trovato
```

Il lookahead viene usato solo quando ° è alla fine.

High-level overview  
- Build the canonical collection of sets of LR(1) Items 
  - Start with an appropriate initial state, s0  
    - [S’ →•S,EOF], along with any equivalent items  
    - Derive equivalent items as closure( s0 )  
  - Repeatedly compute, for each sk, and each symbol X, goto(sk,X)  
    - If the set is not already in the collection, add it  
    - Record all the transitions created by goto( )  
	This eventually reaches a fixed point  
- Fill in the table from the collection of sets of LR(1) item  


### Computing Closures

`closure(s)` aggiunge ad `s` gli items diretta conseguenza degli elementi già in s:  
sia `[A→β•C δ,a]` un elemento di s, allora vengono aggiunti  
- `[C →•τ,x]`, uno per ciascuna derivazione di C
- ogni elemento `x ∈ FIRST(δa)`  

```
 Closure( s )
	while ( s is still changing )
		∀ items [A → β •C δ,a] ∈ s
			∀ productions C → τ ∈ P
			∀ x ∈ FIRST(δa) // δ might be ε
				if [C → • τ,x] ∉ s
				then s ← s ∪ { [C → • τ,x] }
 ```

 ### Computing Gotos

 ```
 Goto( s, X )
 	new ←Ø
 	∀ items [A→β•X δ,a] ∈ s
 		new ← new ∪ {[A→βX •δ,a]}
 	return closure(new)
 ```

 ### Filling Action and Goto tables

 ```
 ∀ set Sx ∈ S
 	∀ item i ∈ Sx
 		if i is [A→β • aδ,b] and goto(Sx,a) = Sk , a ∈ T
 			then ACTION[x,a] ← “shift k”
 		else if i is [S’→S •,EOF]
			 then ACTION[x ,EOF] ← “accept”
		else if i is [A→β •,a]
 			then ACTION[x,a] ← “reduce A→β”

	∀ n ∈ NT
 		if goto(Sx ,n) = Sk
 			then GOTO[x,n] ← k
 ```

# Lezione 9

http://pages.di.unipi.it/gori/Linguaggi-Compilatori2020/ContextSensitiveAnalysisII.pdf

# Lezione 10

http://pages.di.unipi.it/gori/Linguaggi-Compilatori2020/IntermediateRepresentations.pdf
