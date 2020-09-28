# Note generali

- Per valutare l'efficienza di un algoritmo può convenire valutare il blocco di istruzioni piuttosto che il costo singolo. Questo perchè in alcuni casi (vedi sliding windows) il caso peggiore è ampiamente compensato dal resto dei casi.

# Algoritmi di sortting

## Bubble sort

- Sempre quadratico (O(N^2)), senza distinzione tra caso peggiore, caso medio e caso migliore.
- Unica utilità nel caso in cui l'unica operazione concessa sia l'invesione di due elementi consecutivi


## Insertion sort

- Usato per array corti
- Usato ibridamente 
- Caso peggiore O(N^2)

## Merge sort

- O(N log(N)) confronti. Non si può ottenere meno di O(N log(N)) con algoritmi basati sui confronti (comparison model) e senza aggiungere restrizioni particolari. 
- O(N) spazio (no inplace)
- Esistono versioni teoriche con O(N sqr(log(N)))

## Quick sort
- O(N^2) caso peggiore, scegliendo pivot vicino agli estremi del vettore
  - Scegliendo il pivot casualmente si ha una probabilità 1 - 1/N di avere un costo medio di O(N log(N))
  
### Quick select

Trovare il k-th elemento più piccolo.  
// TODO incollare algoritmo

## Counting sort

## Radix sort


# Algoritmi di ricerca

## Binary search
- Ricerca binaria richiede un vettore ordinato
- Controlla l'elemento centrale, e confrontandolo con l'elemento cercato procede ricorsivamente nella metà sx o dx
- t(N) = O(1) + t(N/2) da cui **O(log(N))**  
  O(1) è il costo di un confronto. Per valori interi per esempio questo valore è costante (O(1) appunto), ma per variabili come le stringhe il costo è proporzionale alla lunghezza della stringa, per cui si ha **O(|str|log(N))**.  
  E' possibile ottenere O(|str|+log(N)) utilizzando una struttura di appoggio. Quale?

## Exponential search

- partendo dalla posizione 1, raddoppiamo l'indice di posizione finchè non torviamo un valore nel vettore maggiore di quello ricercato.  
  Pertanto, il valore cercato sarà nell'intervallo 2^(i-1) - 2^i.
- Nell'intervallo individuato viene applicata la ricerca binaria
- O(log(N)) + O(log(N)) = **O(log(N))**
  Costo ricerca exp + costo ricerca bin
- In realtà, la ricerca esponenziale risultà più efficiente di quella binaria per elementi nelle prime posizioni del vettore. la ricerca exp non necessità di dividere a metà il vettore ogni volta, ma basta fermarsi appena trova un elemento più grande.
- Sia L la posizione dell'elemento cercato la ricerca exp ha un costo di:
  O(log(l)) + O(log(l)) = **O(log(l))**
- Exponential search è buono in particolar modo per le sequenze unbound
