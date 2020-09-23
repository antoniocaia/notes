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
- [Lezione 3](#lezione-3)
	- [Analisi lessicalse](#analisi-lessicalse)



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

| Grammatica |    Costo     |
| :--------: | :----------: |
|     3      |      P       |
|     2      |      P       |
|     1      | PSPACE (NP)  |
|     0      | undecidable  |



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

# Lezione 3

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