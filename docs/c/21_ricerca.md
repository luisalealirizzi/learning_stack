# Ricerca

---

## Idea chiave

Ordinare un array e utile, ma spesso quello che vogliamo davvero e **trovare** un elemento. La ricerca e uno dei problemi piu fondamentali dell'informatica.

Ci sono due approcci principali:

- **Ricerca lineare** — scorro tutto l'array finche trovo l'elemento. Funziona sempre, ma e lenta su array grandi.
- **Ricerca binaria** — sfrutto il fatto che l'array e ordinato per dimezzare ad ogni passo lo spazio di ricerca. Molto piu veloce, ma richiede un array ordinato.

---

## Parte 1 — Ricerca lineare

## Blocco 1: Come funziona

La ricerca lineare e la piu semplice: scorro l'array dall'inizio alla fine e confronto ogni elemento con quello cercato. Se lo trovo, restituisco la posizione. Se arrivo alla fine senza trovarlo, restituisco -1.

```
Array: [5, 3, 8, 1, 4, 2, 7]
Cerco: 4

Confronto v[0]=5 con 4 → no
Confronto v[1]=3 con 4 → no
Confronto v[2]=8 con 4 → no
Confronto v[3]=1 con 4 → no
Confronto v[4]=4 con 4 → trovato! restituisco 4
```

Schema mentale: **scorro dall'inizio, confronto ogni elemento, mi fermo appena trovo**.

---

## Blocco 2: Il codice

```c
#include <stdio.h>

#define N 7

int ricercaLineare(int v[], int n, int cercato) {
    int i;
    for (i = 0; i < n; i++) {
        if (v[i] == cercato)
            return i;   /* trovato — restituisco la posizione */
    }
    return -1;          /* non trovato */
}

int main() {
    int v[N] = {5, 3, 8, 1, 4, 2, 7};
    int cercato, pos;

    printf("Elemento da cercare: ");
    scanf("%d", &cercato);

    pos = ricercaLineare(v, N, cercato);

    if (pos != -1)
        printf("Trovato alla posizione %d\n", pos);
    else
        printf("Elemento non trovato\n");

    return 0;
}
```

---

## Blocco 3: Varianti della ricerca lineare

La ricerca lineare si adatta facilmente a varianti diverse.

### Contare le occorrenze

```c
int contaOccorrenze(int v[], int n, int cercato) {
    int i, cont = 0;
    for (i = 0; i < n; i++) {
        if (v[i] == cercato)
            cont++;
    }
    return cont;
}
```

### Trovare l'ultima occorrenza

```c
int ultimaOccorrenza(int v[], int n, int cercato) {
    int i, ultima = -1;
    for (i = 0; i < n; i++) {
        if (v[i] == cercato)
            ultima = i;   /* aggiorna ogni volta che lo trova */
    }
    return ultima;
}
```

### Cercare in un array di struct

```c
typedef struct {
    char nome[50];
    int matricola;
} Studente;

int cercaPerMatricola(Studente v[], int n, int matricola) {
    int i;
    for (i = 0; i < n; i++) {
        if (v[i].matricola == matricola)
            return i;
    }
    return -1;
}
```

### Quando usare la ricerca lineare

La ricerca lineare funziona sempre — su array ordinati e non, su qualsiasi tipo di dato. E la scelta giusta quando:
- l'array non e ordinato
- l'array e piccolo
- cerchi una sola volta e non vale la pena ordinare prima

---

## Parte 2 — Ricerca binaria

## Blocco 4: Come funziona

La ricerca binaria sfrutta il fatto che l'array e **ordinato**. Invece di scorrere dall'inizio, guarda l'elemento centrale:

- se e quello cercato → trovato
- se e piu grande → l'elemento cercato e nella meta sinistra
- se e piu piccolo → l'elemento cercato e nella meta destra

Ad ogni passo dimezza lo spazio di ricerca. Su un array di 1000 elementi bastano al massimo 10 passi. Su un array di un milione, al massimo 20.

```
Array ordinato: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
Cerco: 7

sin=0, des=9, mid=4 → v[4]=5 < 7 → cerco a destra
sin=5, des=9, mid=7 → v[7]=8 > 7 → cerco a sinistra
sin=5, des=6, mid=5 → v[5]=6 < 7 → cerco a destra
sin=6, des=6, mid=6 → v[6]=7 == 7 → trovato!
```

Con 10 elementi, la ricerca lineare avrebbe fatto al massimo 10 confronti. La ricerca binaria ne ha fatti 4.

Schema mentale: **guarda il centro, scarta meta, ripeti**.

---

## Blocco 5: Il codice — versione iterativa

```c
#include <stdio.h>

#define N 10

int ricercaBinaria(int v[], int n, int cercato) {
    int sin = 0;
    int des = n - 1;
    int mid;

    while (sin <= des) {
        mid = (sin + des) / 2;

        if (v[mid] == cercato)
            return mid;          /* trovato */
        else if (v[mid] < cercato)
            sin = mid + 1;       /* cerca a destra */
        else
            des = mid - 1;       /* cerca a sinistra */
    }

    return -1;   /* non trovato */
}

int main() {
    int v[N] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int cercato, pos;

    printf("Elemento da cercare: ");
    scanf("%d", &cercato);

    pos = ricercaBinaria(v, N, cercato);

    if (pos != -1)
        printf("Trovato alla posizione %d\n", pos);
    else
        printf("Elemento non trovato\n");

    return 0;
}
```

---

## Blocco 6: Il codice — versione ricorsiva

La ricerca binaria si presta naturalmente a una versione ricorsiva — ogni chiamata lavora su una meta dell'array:

```c
int ricercaBinariaRicorsiva(int v[], int sin, int des, int cercato) {
    if (sin > des)
        return -1;   /* caso base: intervallo vuoto — non trovato */

    int mid = (sin + des) / 2;

    if (v[mid] == cercato)
        return mid;                                          /* trovato */
    else if (v[mid] < cercato)
        return ricercaBinariaRicorsiva(v, mid+1, des, cercato);  /* destra */
    else
        return ricercaBinariaRicorsiva(v, sin, mid-1, cercato);  /* sinistra */
}

/* Chiamata nel main */
pos = ricercaBinariaRicorsiva(v, 0, N-1, cercato);
```

::: {.callout-note}
## Iterativa o ricorsiva?
Per la ricerca binaria le due versioni sono equivalenti. La versione iterativa e leggermente piu efficiente perche non crea stack frame. La versione ricorsiva e piu elegante e piu vicina alla definizione matematica. In pratica si usa quasi sempre quella iterativa.
:::

---

## Blocco 7: Attenzione — l'array deve essere ordinato

La ricerca binaria funziona **solo su array ordinati**. Se l'array non e ordinato, i risultati sono imprevedibili.

```c
/* Array NON ordinato — la ricerca binaria da risultati sbagliati */
int v[] = {5, 3, 8, 1, 4};
ricercaBinaria(v, 5, 4);   /* potrebbe restituire -1 anche se 4 c'e! */

/* Ordina prima, poi cerca */
selectionSort(v, 5);
ricercaBinaria(v, 5, 4);   /* ora funziona correttamente */
```

::: {.callout-important}
## Ordina sempre prima di cercare con la ricerca binaria
Se devi cercare molte volte sullo stesso array, conviene ordinarlo una volta sola e poi usare la ricerca binaria per tutte le ricerche successive. Se devi cercare una volta sola, la ricerca lineare e piu semplice e non richiede ordinamento.
:::

---

## Blocco 8: Ricerca lineare vs ricerca binaria

| | Ricerca lineare | Ricerca binaria |
|---|---|---|
| **Array richiesto** | qualsiasi | ordinato |
| **Confronti (caso peggiore)** | N | log2(N) |
| **Su 1.000 elementi** | fino a 1.000 | fino a 10 |
| **Su 1.000.000 elementi** | fino a 1.000.000 | fino a 20 |
| **Quando usarla** | array non ordinato, ricerca unica | array ordinato, ricerche frequenti |

::: {.callout-note}
## Cosa significa log2(N)?
`log2(N)` e il numero di volte che devi dimezzare N per arrivare a 1.
- `log2(8) = 3` perche 8 → 4 → 2 → 1 (tre dimezzamenti)
- `log2(1024) = 10` perche servono 10 dimezzamenti
- `log2(1.000.000) ≈ 20`

Questo spiega perche la ricerca binaria e cosi veloce: raddoppiare la dimensione dell'array aggiunge solo un confronto in piu.
:::

---

## Esempio completo: rubrica telefonica

```c
#include <stdio.h>
#include <string.h>

#define N 5

typedef struct {
    char nome[50];
    char telefono[20];
} Contatto;

/* Ordina per nome con Bubble Sort */
void ordinaPerNome(Contatto v[], int n) {
    int i, j;
    Contatto temp;
    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - 1 - i; j++) {
            if (strcmp(v[j].nome, v[j+1].nome) > 0) {
                temp     = v[j];
                v[j]     = v[j+1];
                v[j+1]   = temp;
            }
        }
    }
}

/* Ricerca binaria per nome */
int cercaContatto(Contatto v[], int n, const char nome[]) {
    int sin = 0, des = n - 1, mid, cmp;
    while (sin <= des) {
        mid = (sin + des) / 2;
        cmp = strcmp(v[mid].nome, nome);
        if (cmp == 0)  return mid;
        if (cmp < 0)   sin = mid + 1;
        else           des = mid - 1;
    }
    return -1;
}

int main() {
    Contatto rubrica[N] = {
        {"Mario",   "333-1111"},
        {"Alice",   "333-2222"},
        {"Sara",    "333-3333"},
        {"Marco",   "333-4444"},
        {"Lucia",   "333-5555"}
    };

    /* Ordino prima di cercare */
    ordinaPerNome(rubrica, N);

    char cercato[50];
    printf("Cerca contatto: ");
    fgets(cercato, sizeof(cercato), stdin);
    cercato[strcspn(cercato, "\n")] = '\0';

    int pos = cercaContatto(rubrica, N, cercato);

    if (pos != -1)
        printf("Trovato: %s → %s\n",
            rubrica[pos].nome, rubrica[pos].telefono);
    else
        printf("Contatto non trovato.\n");

    return 0;
}
```

---

## Schema mentale

```
Devo cercare un elemento?

L'array e ordinato?
    NO  → ricerca lineare
          for (i=0; i<n; i++) if (v[i]==cercato) return i;
          return -1;

    SI  → ricerca binaria
          sin=0, des=n-1
          while (sin<=des):
              mid = (sin+des)/2
              se v[mid]==cercato → return mid
              se v[mid]<cercato  → sin = mid+1
              se v[mid]>cercato  → des = mid-1
          return -1

Devo cercare molte volte?
    → ordina una volta, poi usa ricerca binaria per tutte le ricerche
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge 10 numeri interi e un valore da cercare. Usa la ricerca lineare e stampa tutte le posizioni in cui il valore appare (non solo la prima).

**Esercizio 2**
Scrivi un programma che genera 20 numeri casuali tra 1 e 50, li ordina con Selection Sort e poi chiede all'utente un numero da cercare con la ricerca binaria. Mostra anche quanti confronti ha fatto la ricerca.

**Esercizio 3**
Scrivi una funzione `int primaMaggioreUguale(int v[], int n, int x)` che usa la ricerca binaria per trovare la posizione del primo elemento maggiore o uguale a `x` in un array ordinato. Restituisce `n` se tutti gli elementi sono minori di `x`.

**Esercizio 4**
Scrivi un programma che gestisce una piccola rubrica con almeno 8 contatti. Permette di:
- aggiungere un contatto
- cercare per nome con ricerca binaria (dopo aver ordinato)
- stampare tutta la rubrica in ordine alfabetico
