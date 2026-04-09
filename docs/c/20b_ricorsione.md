# Ricorsione

---

## Idea chiave

Una funzione ricorsiva e una funzione che chiama se stessa. Il concetto può sembrare strano: come può una funzione chiamare se stessa senza andare avanti all'infinito?
La risposta e che ogni chiamata lavora su un problema **più piccolo** del precedente. Ad un certo punto il problema e cosi piccolo che la risposta è ovvia e la funzione smette di chiamarsi.

Ogni funzione ricorsiva ha due parti:
- il **caso base** — il problema e cosi piccolo che la risposta e immediata
- il **caso ricorsivo** — riduci il problema e richiama la funzione

::: {.callout-important}
## Il caso base e obbligatorio
Senza caso base la funzione si chiama all'infinito — il programma esaurisce la memoria e crasha. E il primo errore che si fa con la ricorsione.
:::

---

## Blocco 1: Il primo esempio — il fattoriale

Il fattoriale di N e il prodotto di tutti i numeri da 1 a N:

```
5! = 5 * 4 * 3 * 2 * 1 = 120
```

Puoi vederlo anche cosi:

```
5! = 5 * 4!
4! = 4 * 3!
3! = 3 * 2!
2! = 2 * 1!
1! = 1          ← caso base: la risposta e ovvia
```

Ogni fattoriale e definito in termini del fattoriale precedente — piu piccolo di uno. Questa struttura si traduce direttamente in codice ricorsivo:

```c
int fattoriale(int n) {
    if (n == 1)              /* caso base */
        return 1;
    return n * fattoriale(n - 1);   /* caso ricorsivo */
}
```

### Cosa succede quando chiami `fattoriale(4)`

```
fattoriale(4)
    → 4 * fattoriale(3)
            → 3 * fattoriale(2)
                    → 2 * fattoriale(1)
                                → 1        (caso base)
                    → 2 * 1 = 2
            → 3 * 2 = 6
    → 4 * 6 = 24
```

La funzione scende fino al caso base, poi risale calcolando i risultati.

---

## Blocco 2: Come la pensa il computer

Ogni volta che una funzione viene chiamata, il computer crea una zona di memoria chiamata **stack frame** che contiene le variabili locali e il punto dove tornare quando la funzione finisce.

Con la ricorsione, gli stack frame si accumulano uno sopra l'altro:

```
STACK durante fattoriale(4):

| fattoriale(1) |  ← in cima — il caso base
| fattoriale(2) |
| fattoriale(3) |
| fattoriale(4) |  ← in fondo — la prima chiamata
```

Quando `fattoriale(1)` restituisce 1, il suo stack frame viene rimosso. Poi `fattoriale(2)` calcola `2 * 1 = 2` e anche il suo stack frame viene rimosso. E cosi via, risalendo.

::: {.callout-warning}
## Stack overflow
Se la ricorsione va troppo in profondo — perche il problema e troppo grande o perche manca il caso base — gli stack frame si accumulano finche la memoria dello stack si esaurisce. Questo si chiama **stack overflow** — lo stesso nome del famoso sito per programmatori.
:::

---

## Blocco 3: Fibonacci

La sequenza di Fibonacci e un altro classico:

```
0, 1, 1, 2, 3, 5, 8, 13, 21, ...
```

Ogni numero e la somma dei due precedenti:
```
F(0) = 0          ← caso base
F(1) = 1          ← caso base
F(n) = F(n-1) + F(n-2)   ← caso ricorsivo
```

```c
int fibonacci(int n) {
    if (n == 0) return 0;   /* caso base */
    if (n == 1) return 1;   /* caso base */
    return fibonacci(n - 1) + fibonacci(n - 2);   /* caso ricorsivo */
}
```

### Il problema di Fibonacci ricorsivo

`fibonacci(5)` genera questo albero di chiamate:

```
                    fib(5)
                /          \
           fib(4)          fib(3)
          /      \         /    \
       fib(3)  fib(2)  fib(2) fib(1)
       /   \   /   \   /   \
    fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)
    /   \
 fib(1) fib(0)
```

`fib(3)` viene calcolato due volte, `fib(2)` tre volte. Su numeri grandi questo diventa enormemente lento.

::: {.callout-note}
## Ricorsione vs ciclo
Il Fibonacci ricorsivo e elegante ma inefficiente. La versione con il ciclo e molto piu veloce:

```c
int fibonacci(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1, temp, i;
    for (i = 2; i <= n; i++) {
        temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}
```

La ricorsione non e sempre la soluzione migliore — e potente quando il problema ha una struttura naturalmente ricorsiva, ma puo essere costosa se ricalcola le stesse cose.
:::

---

## Blocco 4: Ricorsione e array — somma degli elementi

La ricorsione funziona bene anche sugli array. La somma degli elementi da 0 a n-1 si puo vedere cosi:

```
somma([5,3,8,1], 4)
    = 5 + somma([3,8,1], 3)      ← primo elemento + somma del resto
    = 5 + 3 + somma([8,1], 2)
    = 5 + 3 + 8 + somma([1], 1)
    = 5 + 3 + 8 + 1 + somma([], 0)
    = 5 + 3 + 8 + 1 + 0          ← caso base: array vuoto vale 0
```

```c
int somma(int v[], int n) {
    if (n == 0)               /* caso base: array vuoto */
        return 0;
    return v[0] + somma(v + 1, n - 1);   /* primo + somma del resto */
}
```

`v + 1` e un puntatore al secondo elemento — la stessa cosa che usare `&v[1]`. La funzione lavora ogni volta su un pezzo piu piccolo dell'array.

---

## Blocco 5: Il collegamento con Merge Sort e Quick Sort

Ora che conosci la ricorsione, guarda di nuovo le funzioni che hai gia visto:

```c
void mergeSort(int v[], int sin, int des) {
    if (sin < des) {                      /* caso base: un solo elemento */
        int mid = (sin + des) / 2;
        mergeSort(v, sin, mid);           /* risolvi meta sinistra */
        mergeSort(v, mid + 1, des);       /* risolvi meta destra */
        merge(v, sin, mid, des);          /* combina i risultati */
    }
}
```

```c
void quickSort(int v[], int sin, int des) {
    if (sin < des) {                      /* caso base: un solo elemento */
        int p = partition(v, sin, des);
        quickSort(v, sin, p - 1);         /* risolvi parte sinistra */
        quickSort(v, p + 1, des);         /* risolvi parte destra */
    }
}
```

Entrambi seguono esattamente lo schema:
1. **caso base** — se il pezzo e di un elemento, e gia ordinato
2. **dividi** — spezza il problema in pezzi piu piccoli
3. **risolvi** — chiama se stesso sui pezzi
4. **combina** — metti insieme i risultati

Questa struttura si chiama **divide et impera** ed e uno dei pattern algoritmici piu importanti dell'informatica.

---

## Blocco 6: Quando usare la ricorsione

La ricorsione e la scelta naturale quando:

- il problema si divide in sottoproblemi della stessa natura (Merge Sort, Quick Sort)
- la struttura dei dati e ricorsiva per definizione (alberi, liste concatenate — che vedremo in futuro)
- la soluzione ricorsiva e molto piu chiara di quella iterativa

Non e la scelta giusta quando:
- il problema si risolve facilmente con un ciclo
- si ricalcolano gli stessi sottoproblemi piu volte (come Fibonacci)
- la profondita di ricorsione puo diventare molto grande

::: {.callout-tip}
## Come leggere una funzione ricorsiva
Quando incontri una funzione ricorsiva, non cercare di seguire mentalmente tutte le chiamate — ti perdi. Invece:

1. leggi il caso base — cosa succede quando il problema e minimo?
2. leggi il caso ricorsivo — come si riduce il problema?
3. fidati che la funzione funzioni sui sottoproblemi piu piccoli

Questo approccio si chiama **induzione** — e lo stesso modo in cui i matematici ragionano sulle dimostrazioni ricorsive.
:::

---

## Schema mentale

```
Come si scrive una funzione ricorsiva?

1. Individua il caso base
   -> il problema piu piccolo con risposta ovvia
   -> senza caso base → stack overflow

2. Individua il caso ricorsivo
   -> come riduco il problema di un passo?
   -> la chiamata ricorsiva deve avvicinarsi al caso base

3. Fidati della ricorsione
   -> non cercare di seguire tutte le chiamate a mente
   -> assume che la funzione funzioni sui sottoproblemi

Struttura:
    tipo funzione(parametri) {
        if (caso base)
            return risultato_immediato;
        return funzione(problema_piu_piccolo);
    }
```

---

## Esercizi

**Esercizio 1**
Scrivi una funzione ricorsiva `int potenza(int base, int esp)` che calcola `base` elevato a `esp`.
```
potenza(2, 0) = 1        ← caso base
potenza(2, n) = 2 * potenza(2, n-1)
```

**Esercizio 2**
Scrivi una funzione ricorsiva `int sommaDigiti(int n)` che calcola la somma delle cifre di un numero intero.
```
sommaDigiti(347) = 7 + sommaDigiti(34)
sommaDigiti(3)   = 3   ← caso base: numero a una cifra
```

**Esercizio 3**
Scrivi una funzione ricorsiva `int contaElementi(int v[], int n, int x)` che conta quante volte `x` appare nell'array.

**Esercizio 4**
Scrivi una funzione ricorsiva `void invertiArray(int v[], int sin, int des)` che inverte un array sul posto scambiando il primo e l'ultimo elemento e poi chiamandosi sul pezzo centrale.
```
inverti([1,2,3,4,5])
    scambia v[0] e v[4] → [5,2,3,4,1]
    inverti([2,3,4])
        scambia v[1] e v[3] → [5,4,3,2,1]
        inverti([3])  ← caso base: un elemento
```
