# Ordinamento per inserzione — Insertion Sort e Shell Sort

---

## Idea chiave

L'**Insertion Sort** si ispira al modo in cui ordiniamo le carte da gioco a mano. Prendi una carta alla volta dalla pila e inseriscila nel posto giusto tra le carte che tieni gia in mano — che sono sempre ordinate.

In termini di array: scorri gli elementi uno alla volta. Per ogni elemento, trovane la posizione corretta nella parte gia ordinata a sinistra e inseriscilo li — spostando gli altri per fare posto.

```
[5, 3, 8, 1, 4]   prendo 3 → inserisco prima di 5
[3, 5, 8, 1, 4]   prendo 8 → gia al posto giusto
[3, 5, 8, 1, 4]   prendo 1 → inserisco in testa
[1, 3, 5, 8, 4]   prendo 4 → inserisco tra 3 e 5
[1, 3, 4, 5, 8]   ordinato!
```

---

## Blocco 1: Come funziona — passo per passo

Array di partenza: `[5, 3, 8, 1, 4]`

**Passaggio 1** — prendo `v[1] = 3`, lo confronto con la parte ordinata `[5]`:
```
chiave = 3
3 < 5 → sposto 5 a destra
inserisco 3 in posizione 0
[3, 5, 8, 1, 4]
```

**Passaggio 2** — prendo `v[2] = 8`, lo confronto con `[3, 5]`:
```
chiave = 8
8 > 5 → gia al posto giusto
[3, 5, 8, 1, 4]
```

**Passaggio 3** — prendo `v[3] = 1`, lo confronto con `[3, 5, 8]`:
```
chiave = 1
1 < 8 → sposto 8 a destra
1 < 5 → sposto 5 a destra
1 < 3 → sposto 3 a destra
inserisco 1 in posizione 0
[1, 3, 5, 8, 4]
```

**Passaggio 4** — prendo `v[4] = 4`, lo confronto con `[1, 3, 5, 8]`:
```
chiave = 4
4 < 8 → sposto 8 a destra
4 < 5 → sposto 5 a destra
4 > 3 → inserisco 4 in posizione 2
[1, 3, 4, 5, 8]
```

Array ordinato: `[1, 3, 4, 5, 8]`

Schema mentale: **prendo un elemento, lo confronto con quelli alla mia sinistra spostandoli a destra finche trovo il posto giusto, e lo inserisco**.

---

## Blocco 2: Il codice

```c
#include <stdio.h>

#define N 5

void insertionSort(int v[], int n) {
    int i, j, chiave;

    for (i = 1; i < n; i++) {
        chiave = v[i];       /* elemento da inserire */
        j = i - 1;

        /* sposta a destra gli elementi piu grandi della chiave */
        while (j >= 0 && v[j] > chiave) {
            v[j + 1] = v[j];
            j--;
        }

        /* inserisce la chiave nella posizione corretta */
        v[j + 1] = chiave;
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

    insertionSort(v, N);

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

## Blocco 3: Capire il codice

```c
for (i = 1; i < n; i++) {       /* parte da 1 — v[0] e gia "ordinato" */
    chiave = v[i];               /* salvo l'elemento da inserire */
    j = i - 1;                   /* parto dal precedente */

    while (j >= 0 && v[j] > chiave) {   /* scorro a sinistra */
        v[j + 1] = v[j];                 /* sposto l'elemento a destra */
        j--;
    }

    v[j + 1] = chiave;           /* inserisco la chiave */
}
```

**Perche si parte da `i = 1`?** Il primo elemento `v[0]` da solo e gia ordinato. Si parte dal secondo.

**Perche si salva `chiave = v[i]`?** Quando spostiamo gli elementi a destra, sovrascriviamo `v[i]`. Dobbiamo salvare il valore prima di perdere.

**Perche `j >= 0` nella condizione del while?** Per non uscire fuori dall'array quando l'elemento va inserito in testa.

::: {.callout-note}
## Il punto di forza dell'Insertion Sort
Se l'array e **quasi ordinato**, l'Insertion Sort e molto veloce — il while interno si ferma subito perche gli elementi sono gia nel posto giusto. Per questo Java lo usa come parte del TimSort su array piccoli o quasi ordinati.
:::

---

## Blocco 4: Ordinamento decrescente

```c
void insertionSortDecrescente(int v[], int n) {
    int i, j, chiave;

    for (i = 1; i < n; i++) {
        chiave = v[i];
        j = i - 1;

        while (j >= 0 && v[j] < chiave) {   /* < invece di > */
            v[j + 1] = v[j];
            j--;
        }

        v[j + 1] = chiave;
    }
}
```

---

## Blocco 5: Insertion Sort su array di struct

```c
typedef struct {
    char nome[50];
    double media;
} Studente;

void ordinaPerMedia(Studente v[], int n) {
    int i, j;
    Studente chiave;

    for (i = 1; i < n; i++) {
        chiave = v[i];
        j = i - 1;

        while (j >= 0 && v[j].media > chiave.media) {
            v[j + 1] = v[j];
            j--;
        }

        v[j + 1] = chiave;
    }
}
```

---

## Blocco 6: Shell Sort — una variante piu veloce

L'**Shell Sort** e un miglioramento dell'Insertion Sort. L'idea e semplice: invece di confrontare solo elementi adiacenti, confronta elementi lontani tra loro. Questo sposta velocemente gli elementi molto fuori posto, riducendo il lavoro dell'Insertion Sort finale.

### Come funziona

Si sceglie un **gap** — la distanza tra gli elementi da confrontare. Si ordina con Insertion Sort usando quel gap. Poi si dimezza il gap e si ripete. Quando il gap arriva a 1, e un normale Insertion Sort — ma l'array e gia quasi ordinato, quindi e veloce.

```
Array: [9, 1, 7, 3, 8, 2, 6, 4]    gap = 4
Confronta: v[0] e v[4], v[1] e v[5], v[2] e v[6], v[3] e v[7]
Risultato: [8, 1, 6, 3, 9, 2, 7, 4]  (quasi ordinato)

Gap = 2
Confronta a distanza 2 e ordina...

Gap = 1
Insertion Sort finale — pochissimo lavoro
```

### Il codice

```c
void shellSort(int v[], int n) {
    int gap, i, j, temp;

    /* parte con gap = meta dell'array, dimezza ad ogni iterazione */
    for (gap = n / 2; gap > 0; gap /= 2) {

        /* Insertion Sort con questo gap */
        for (i = gap; i < n; i++) {
            temp = v[i];
            j = i;

            while (j >= gap && v[j - gap] > temp) {
                v[j] = v[j - gap];
                j -= gap;
            }

            v[j] = temp;
        }
    }
}
```

Quando `gap = 1`, il codice diventa identico all'Insertion Sort classico.

::: {.callout-tip}
## Shell Sort vs Insertion Sort
Su array grandi lo Shell Sort e significativamente piu veloce dell'Insertion Sort, pur essendo basato sulla stessa idea. E un buon esempio di come una piccola modifica all'algoritmo puo cambiare molto le prestazioni.
:::

---

## Blocco 7: Confronto tra i tre algoritmi visti finora

| | Bubble Sort | Selection Sort | Insertion Sort |
|---|---|---|---|
| **Strategia** | scambia adiacenti | trova il minimo | inserisce in posizione |
| **Scambi** | molti | al massimo N-1 | spostamenti |
| **Migliore con** | array quasi ordinati | scambi costosi | array quasi ordinati |
| **Peggiore con** | array invertiti | sempre uguale | array invertiti |
| **Leggibilita** | alta | alta | media |

Tutti e tre sono semplici da capire e da implementare. Su array piccoli le differenze sono irrilevanti. Su array grandi esistono algoritmi molto piu veloci — li vedremo nella prossima lezione.

---

## Schema mentale

```
Come funziona l'Insertion Sort?
    -> prendo v[i] come chiave
    -> scorro a sinistra finche trovo un elemento piu piccolo
    -> sposto gli elementi a destra per fare posto
    -> inserisco la chiave

Cosa salvo prima di spostare?
    -> chiave = v[i]  — fondamentale, altrimenti perdo il valore

Quando si ferma il while?
    -> j < 0          ho raggiunto l'inizio dell'array
    -> v[j] <= chiave ho trovato dove inserire

Cos'e lo Shell Sort?
    -> Insertion Sort con gap > 1
    -> dimezza il gap ad ogni iterazione
    -> quando gap = 1 e un normale Insertion Sort
    -> piu veloce su array grandi
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge 7 numeri interi, li ordina con Insertion Sort e li stampa.

**Esercizio 2**
Scrivi un programma che legge un array di N interi gia quasi ordinati (ad esempio con un solo elemento fuori posto) e conta quanti spostamenti fa l'Insertion Sort. Confronta con il numero di spostamenti che farebbe il Bubble Sort sullo stesso array.

**Esercizio 3**
Scrivi una funzione `void ordinaStringhe(char v[][50], int n)` che ordina un array di stringhe in ordine alfabetico usando Insertion Sort e `strcmp`.

**Esercizio 4**
Scrivi un programma che:
- genera 10 numeri casuali tra 1 e 100
- li ordina prima con Insertion Sort, poi con Shell Sort
- stampa i risultati di entrambi e verifica che siano uguali
