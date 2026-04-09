# Ordinamento per selezione — Selection Sort

---

## Idea chiave

Il **Selection Sort** usa una strategia diversa dal Bubble Sort. Invece di confrontare coppie adiacenti e scambiarle, cerca il minimo dell'array e lo mette al posto giusto — una posizione alla volta.

L'idea e questa: dividi mentalmente l'array in due parti — la parte **ordinata** a sinistra e la parte **non ordinata** a destra. All'inizio la parte ordinata e vuota. Ad ogni passaggio, trova il minimo nella parte non ordinata e spostalo alla fine della parte ordinata.

```
[5, 3, 8, 1, 4]   parte ordinata: []        non ordinata: [5,3,8,1,4]
[1, 3, 8, 5, 4]   parte ordinata: [1]       non ordinata: [3,8,5,4]
[1, 3, 8, 5, 4]   parte ordinata: [1,3]     non ordinata: [8,5,4]
[1, 3, 4, 5, 8]   parte ordinata: [1,3,4]   non ordinata: [5,8]
[1, 3, 4, 5, 8]   parte ordinata: [1,3,4,5] non ordinata: [8]
[1, 3, 4, 5, 8]   ordinato!
```

---

## Blocco 1: Come funziona — passo per passo

Array di partenza: `[5, 3, 8, 1, 4]`

**Passaggio 1** — trova il minimo in tutto l'array:
```
minimo = 1 (posizione 3)
scambia v[0] con v[3]
[1, 3, 8, 5, 4]
 ^ordinato
```

**Passaggio 2** — trova il minimo da posizione 1 in poi:
```
minimo = 3 (posizione 1)
nessuno scambio necessario
[1, 3, 8, 5, 4]
    ^ordinato
```

**Passaggio 3** — trova il minimo da posizione 2 in poi:
```
minimo = 4 (posizione 4)
scambia v[2] con v[4]
[1, 3, 4, 5, 8]
       ^ordinato
```

**Passaggio 4** — trova il minimo da posizione 3 in poi:
```
minimo = 5 (posizione 3)
nessuno scambio necessario
[1, 3, 4, 5, 8]
          ^ordinato
```

Array ordinato: `[1, 3, 4, 5, 8]`

Schema mentale: **ad ogni passaggio trovo il minimo nella parte non ordinata e lo porto alla sua posizione finale**.

---

## Blocco 2: Il codice

```c
#include <stdio.h>

#define N 5

void selectionSort(int v[], int n) {
    int i, j, indMin, temp;

    for (i = 0; i < n - 1; i++) {

        /* trova l'indice del minimo nella parte non ordinata */
        indMin = i;
        for (j = i + 1; j < n; j++) {
            if (v[j] < v[indMin]) {
                indMin = j;
            }
        }

        /* scambia il minimo con il primo elemento non ordinato */
        if (indMin != i) {
            temp       = v[i];
            v[i]       = v[indMin];
            v[indMin]  = temp;
        }
    }
}

void stampa(int v[], int n) {
    int i;
    for (i = 0; i < n; i++)
        printf("%d ", v[i]);
    printf("\n");
}

int main() {
    int v[N] = {5, 3, 8, 1, 4};

    printf("Prima:  ");
    stampa(v, N);

    selectionSort(v, N);

    printf("Dopo:   ");
    stampa(v, N);

    return 0;
}
```

Output:
```
Prima:  5 3 8 1 4
Dopo:   1 3 4 5 8
```

---

## Blocco 3: Capire i due cicli

```c
for (i = 0; i < n - 1; i++) {        /* ciclo esterno: posizione da riempire */

    indMin = i;                        /* assume che il minimo sia in posizione i */
    for (j = i + 1; j < n; j++) {    /* ciclo interno: cerca il vero minimo */
        if (v[j] < v[indMin]) {
            indMin = j;
        }
    }

    if (indMin != i) {                 /* scambia solo se serve */
        temp      = v[i];
        v[i]      = v[indMin];
        v[indMin] = temp;
    }
}
```

**Ciclo esterno** — scorre le posizioni da riempire. Alla posizione `i` va il minimo degli elementi da `i` in poi.

**Ciclo interno** — cerca il minimo nella parte non ordinata. Non scambia subito — si limita a tenere traccia dell'indice del minimo trovato finora.

**Lo scambio** — avviene una sola volta per passaggio, solo se il minimo non e gia al posto giusto.

::: {.callout-note}
## La differenza con Bubble Sort
Il Bubble Sort fa tanti piccoli scambi ad ogni passaggio. Il Selection Sort fa **al massimo uno scambio per passaggio** — quello finale con il minimo. Per questo e piu efficiente in termini di numero di scambi, anche se fa lo stesso numero di confronti.
:::

---

## Blocco 4: Ordinamento decrescente

Per ordinare in ordine decrescente, cerca il **massimo** invece del minimo:

```c
void selectionSortDecrescente(int v[], int n) {
    int i, j, indMax, temp;

    for (i = 0; i < n - 1; i++) {
        indMax = i;
        for (j = i + 1; j < n; j++) {
            if (v[j] > v[indMax]) {   /* > invece di < */
                indMax = j;
            }
        }
        if (indMax != i) {
            temp       = v[i];
            v[i]       = v[indMax];
            v[indMax]  = temp;
        }
    }
}
```

---

## Blocco 5: Selection Sort su array di struct

Come per il Bubble Sort, basta confrontare il campo su cui vuoi ordinare:

```c
typedef struct {
    char nome[50];
    double media;
} Studente;

void ordinaPerMedia(Studente v[], int n) {
    int i, j, indMin;
    Studente temp;

    for (i = 0; i < n - 1; i++) {
        indMin = i;
        for (j = i + 1; j < n; j++) {
            if (v[j].media < v[indMin].media) {
                indMin = j;
            }
        }
        if (indMin != i) {
            temp      = v[i];
            v[i]      = v[indMin];
            v[indMin] = temp;
        }
    }
}
```

---

## Blocco 6: Bubble Sort vs Selection Sort

| | Bubble Sort | Selection Sort |
|---|---|---|
| **Strategia** | scambia coppie adiacenti | trova il minimo e lo porta in posizione |
| **Scambi per passaggio** | molti | al massimo uno |
| **Confronti** | N*(N-1)/2 | N*(N-1)/2 |
| **Ottimizzabile** | si — stop se nessuno scambio | no |
| **Quando preferirlo** | array quasi ordinati | quando gli scambi sono costosi |

Entrambi fanno lo stesso numero di confronti. Il Selection Sort fa meno scambi — utile quando copiare un elemento e costoso, ad esempio con struct grandi.

---

## Schema mentale

```
Come funziona il Selection Sort?
    -> trova il minimo nella parte non ordinata
    -> portalo alla prima posizione non ordinata
    -> ripeti per N-1 passaggi

Come si trova il minimo?
    -> indMin = i            assume che il minimo sia in posizione i
    -> scorre da i+1 a n-1   cerca se ce ne uno piu piccolo
    -> aggiorna indMin se trova un valore piu piccolo

Quanti scambi fa?
    -> al massimo uno per passaggio
    -> solo se il minimo non e gia al posto giusto

Come si ordina in decrescente?
    -> cerca il massimo invece del minimo
    -> > invece di <
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge 6 numeri interi, li ordina con Selection Sort in ordine crescente e li stampa.

**Esercizio 2**
Scrivi una funzione `int indicePiuVicino(int v[], int n, int target)` che restituisce l'indice dell'elemento piu vicino al valore `target`. Usala dopo aver ordinato l'array con Selection Sort.

**Esercizio 3**
Dichiara una struct `Prodotto` con nome e prezzo. Scrivi un programma che legge 4 prodotti, li ordina per prezzo crescente con Selection Sort e stampa la lista ordinata.

**Esercizio 4**
Scrivi un programma che legge un array di N interi, lo ordina con Selection Sort e poi cerca un valore inserito dall'utente. *(Preparazione alla ricerca binaria che vedremo piu avanti.)*
