# Ordinamento avanzato — Merge Sort e Quick Sort

---

## Idea chiave

Gli algoritmi che abbiamo visto finora — Bubble Sort, Selection Sort, Insertion Sort — sono semplici da capire ma lenti su array grandi. Con N elementi fanno circa N*N operazioni.

Gli algoritmi avanzati usano una strategia diversa: **divide et impera** — dividi il problema in sottoproblemi piu piccoli, risolvili separatamente, poi combina i risultati. Con questa strategia il numero di operazioni scende a N*log(N) — molto piu veloce.

In questa lezione vediamo i due algoritmi divide et impera piu importanti: **Merge Sort** e **Quick Sort**.

---

## Parte 1 — Merge Sort

## Blocco 1: Come funziona il Merge Sort

Il Merge Sort divide l'array a meta, ordina le due meta separatamente, poi le **fonde** insieme in ordine. La fusione di due array gia ordinati e semplice e veloce.

```
[5, 3, 8, 1, 4, 2, 7, 6]

Dividi:
[5, 3, 8, 1]    [4, 2, 7, 6]

Dividi ancora:
[5, 3]  [8, 1]  [4, 2]  [7, 6]

Dividi ancora:
[5] [3] [8] [1] [4] [2] [7] [6]

Fondi a coppie:
[3, 5]  [1, 8]  [2, 4]  [6, 7]

Fondi:
[1, 3, 5, 8]    [2, 4, 6, 7]

Fondi:
[1, 2, 3, 4, 5, 6, 7, 8]
```

Schema mentale: **dividi finche hai pezzi di un elemento, poi fondi in ordine risalendo**.

---

## Blocco 2: La fusione di due array ordinati

Il cuore del Merge Sort e la funzione `merge` — quella che fonde due meta gia ordinate in un unico array ordinato.

```
Array sinistro:  [1, 3, 5, 8]
Array destro:    [2, 4, 6, 7]

Confronto 1 e 2 → prendo 1
Confronto 3 e 2 → prendo 2
Confronto 3 e 4 → prendo 3
Confronto 5 e 4 → prendo 4
Confronto 5 e 6 → prendo 5
Confronto 8 e 6 → prendo 6
Confronto 8 e 7 → prendo 7
Rimane 8        → prendo 8

Risultato: [1, 2, 3, 4, 5, 6, 7, 8]
```

---

## Blocco 3: Il codice

```c
#include <stdio.h>
#include <stdlib.h>

#define N 8

/* Fonde v[sin..mid] e v[mid+1..des] in ordine */
void merge(int v[], int sin, int mid, int des) {
    int n1 = mid - sin + 1;
    int n2 = des - mid;
    int i, j, k;

    /* array temporanei */
    int *L = (int *)malloc(n1 * sizeof(int));
    int *R = (int *)malloc(n2 * sizeof(int));

    /* copia nelle meta temporanee */
    for (i = 0; i < n1; i++) L[i] = v[sin + i];
    for (j = 0; j < n2; j++) R[j] = v[mid + 1 + j];

    /* fonde le due meta */
    i = 0; j = 0; k = sin;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            v[k] = L[i];
            i++;
        } else {
            v[k] = R[j];
            j++;
        }
        k++;
    }

    /* copia gli elementi rimasti */
    while (i < n1) { v[k] = L[i]; i++; k++; }
    while (j < n2) { v[k] = R[j]; j++; k++; }

    free(L);
    free(R);
}

/* Ordina v[sin..des] con Merge Sort */
void mergeSort(int v[], int sin, int des) {
    if (sin < des) {
        int mid = (sin + des) / 2;
        mergeSort(v, sin, mid);       /* ordina la meta sinistra */
        mergeSort(v, mid + 1, des);   /* ordina la meta destra */
        merge(v, sin, mid, des);      /* fonde le due meta */
    }
}

void stampa(int v[], int n) {
    int i;
    for (i = 0; i < n; i++)
        printf("%d ", v[i]);
    printf("\n");
}

int main() {
    int v[N] = {5, 3, 8, 1, 4, 2, 7, 6};

    printf("Prima:  ");
    stampa(v, N);

    mergeSort(v, 0, N - 1);

    printf("Dopo:   ");
    stampa(v, N);

    return 0;
}
```

::: {.callout-note}
## La ricorsione
`mergeSort` chiama se stessa su pezzi sempre piu piccoli dell'array. Questa tecnica si chiama **ricorsione**. Il caso base e `sin >= des` — un array di un elemento e gia ordinato. Senza caso base la funzione chiamerebbe se stessa all'infinito.
:::

::: {.callout-note}
## malloc e free
Il Merge Sort ha bisogno di memoria extra per gli array temporanei durante la fusione. Usiamo `malloc` per allocare questa memoria e `free` per liberarla. E il primo esempio pratico di allocazione dinamica.
:::

---

## Blocco 4: I punti di forza del Merge Sort

- **Stabile** — elementi uguali mantengono il loro ordine relativo originale
- **Prestazioni garantite** — e sempre veloce, indipendentemente dall'input
- **Base del TimSort** — Java usa una versione ibrida di Merge Sort e Insertion Sort in `Collections.sort()`

Il punto debole: richiede memoria extra proporzionale alla dimensione dell'array.

---

## Parte 2 — Quick Sort

## Blocco 5: Come funziona il Quick Sort

Il **Quick Sort** usa anch'esso divide et impera, ma con una strategia diversa. Sceglie un elemento **pivot** e riorganizza l'array in modo che:
- tutti gli elementi minori del pivot stiano a sinistra
- tutti gli elementi maggiori stiano a destra

Poi applica ricorsivamente la stessa strategia alle due parti.

```
[5, 3, 8, 1, 4, 2, 7, 6]   pivot = 6

Partizione:
[5, 3, 1, 4, 2] | 6 | [8, 7]
 minori di 6        maggiori di 6

Applica Quick Sort a sinistra e a destra:
[1, 2, 3, 4, 5] | 6 | [7, 8]

Risultato: [1, 2, 3, 4, 5, 6, 7, 8]
```

Schema mentale: **metti il pivot al posto giusto, poi ordina i due pezzi ai lati**.

---

## Blocco 6: La partizione

Il cuore del Quick Sort e la funzione `partition` — quella che riorganizza l'array attorno al pivot.

```c
/* Riorganizza v[sin..des] attorno al pivot (ultimo elemento) */
/* Restituisce l'indice finale del pivot */
int partition(int v[], int sin, int des) {
    int pivot = v[des];   /* scegliamo l'ultimo elemento come pivot */
    int i = sin - 1;      /* indice dell'elemento piu piccolo */
    int j, temp;

    for (j = sin; j < des; j++) {
        if (v[j] <= pivot) {
            i++;
            /* scambia v[i] e v[j] */
            temp = v[i]; v[i] = v[j]; v[j] = temp;
        }
    }

    /* metti il pivot al posto giusto */
    temp = v[i + 1]; v[i + 1] = v[des]; v[des] = temp;
    return i + 1;   /* indice finale del pivot */
}
```

---

## Blocco 7: Il codice completo

```c
#include <stdio.h>

#define N 8

int partition(int v[], int sin, int des) {
    int pivot = v[des];
    int i = sin - 1;
    int j, temp;

    for (j = sin; j < des; j++) {
        if (v[j] <= pivot) {
            i++;
            temp = v[i]; v[i] = v[j]; v[j] = temp;
        }
    }

    temp = v[i + 1]; v[i + 1] = v[des]; v[des] = temp;
    return i + 1;
}

void quickSort(int v[], int sin, int des) {
    if (sin < des) {
        int p = partition(v, sin, des);   /* posizione finale del pivot */
        quickSort(v, sin, p - 1);         /* ordina la parte sinistra */
        quickSort(v, p + 1, des);         /* ordina la parte destra */
    }
}

void stampa(int v[], int n) {
    int i;
    for (i = 0; i < n; i++)
        printf("%d ", v[i]);
    printf("\n");
}

int main() {
    int v[N] = {5, 3, 8, 1, 4, 2, 7, 6};

    printf("Prima:  ");
    stampa(v, N);

    quickSort(v, 0, N - 1);

    printf("Dopo:   ");
    stampa(v, N);

    return 0;
}
```

---

## Blocco 8: I punti di forza del Quick Sort

- **Veloce in pratica** — su array casuali e spesso il piu veloce in assoluto
- **In-place** — non richiede memoria extra come il Merge Sort
- **Molto usato** — e la base di molte implementazioni di ordinamento nei linguaggi di sistema

Il punto debole: se il pivot e sempre il minimo o il massimo (caso peggiore), le prestazioni calano drasticamente. Si puo mitigare scegliendo il pivot in modo casuale o usando la mediana di tre elementi.

---

## Blocco 9: Merge Sort vs Quick Sort

| | Merge Sort | Quick Sort |
|---|---|---|
| **Strategia** | dividi, ordina, fondi | partiziona attorno al pivot |
| **Memoria extra** | si — array temporanei | no — in-place |
| **Caso peggiore** | sempre veloce | pivot mal scelto |
| **Stabile** | si | no |
| **Usato in** | Java `Collections.sort()` | librerie C `qsort()` |
| **Quando preferirlo** | stabilita garantita | memoria limitata |

---

## Il TimSort — il meglio dei due mondi

Java non usa ne solo Merge Sort ne solo Quick Sort. Usa il **TimSort** — un algoritmo ibrido che combina Merge Sort e Insertion Sort.

L'idea:
- divide l'array in blocchi piccoli (chiamati "run")
- ordina ogni blocco con Insertion Sort (veloce su array piccoli e quasi ordinati)
- fonde i blocchi con Merge Sort

Il risultato e un algoritmo che sfrutta i punti di forza di entrambi: veloce su array grandi come il Merge Sort, veloce su array piccoli e quasi ordinati come l'Insertion Sort.

Quando in Java scrivi `Collections.sort(lista)` o `Arrays.sort(array)`, sta girando il TimSort — costruito sulle stesse idee che hai appena studiato.

---

## Schema mentale

```
Merge Sort
    -> dividi l'array a meta
    -> ordina le due meta (ricorsione)
    -> fondi le due meta ordinate
    -> richiede memoria extra
    -> sempre veloce, stabile

Quick Sort
    -> scegli un pivot
    -> metti i minori a sinistra, i maggiori a destra
    -> applica ricorsivamente alle due parti
    -> in-place, no memoria extra
    -> veloce in pratica, ma dipende dal pivot

Entrambi usano:
    -> divide et impera
    -> ricorsione
    -> caso base: array di un elemento e gia ordinato
```

---

## Esercizi

**Esercizio 1**
Traccia a mano il funzionamento del Merge Sort sull'array `[4, 7, 2, 9, 1, 5]`. Disegna tutti i passi di divisione e fusione.

**Esercizio 2**
Traccia a mano il funzionamento del Quick Sort sull'array `[4, 7, 2, 9, 1, 5]` usando sempre l'ultimo elemento come pivot. Mostra la partizione ad ogni livello.

**Esercizio 3**
Modifica il Merge Sort per ordinare in ordine decrescente. Quale riga devi cambiare?

**Esercizio 4**
Scrivi un programma che genera 1000 numeri casuali, li ordina con Merge Sort e con Quick Sort, e verifica che i due risultati siano identici.

**Esercizio 5 — sfida**
La funzione `qsort` della libreria `stdlib.h` implementa il Quick Sort in C. Studia la sua firma:
```c
void qsort(void *base, size_t n, size_t size,
           int (*confronta)(const void *, const void *));
```
Scrivi un programma che usa `qsort` per ordinare un array di interi e uno di struct.
