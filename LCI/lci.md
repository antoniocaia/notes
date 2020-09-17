# LCI (Draft)

- [LCI (Draft)](#lci-draft)
- [Lezione 1](#lezione-1)
	- [Compilatore](#compilatore)
	- [Front end](#front-end)
	- [Ottimizzatore](#ottimizzatore)
	- [Back end](#back-end)
- [LEZIONE 2](#lezione-2)
	- [Linguaggi formali](#linguaggi-formali)



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

# LEZIONE 2

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
|     0      | undecidable) |



**Linguaggi regolari:**
Possibili rappresentazioni:
- Grammatica regolare  
- Determinist finite automata (DFA)  
- Non-determinist finite automata (NFA)  
- Non-determinist finite automata epsilon (epsilon-NFA)  
- Regular Expression (RE)  


**Grammatica regolare:**  
G = (E, N, S, P)  
ES:  
L = { a^n b^n | n,m > 0 }  
S->aS|B  
B->bB|b  


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
Per ogni simbolo terminale definisco un nuovo stato + un nuovo stato F (finale).
Per ogni regola della grammatica costruisco un collegamento tra due stati.

ETC

