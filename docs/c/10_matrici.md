# Matrici

---

## Idea chiave

Una matrice in C è un **array di array**. Pensala come una tabella: righe e colonne. Ogni elemento è identificato da due indici: `[riga][colonna]`.

```c
int m[3][4];
```

Significa:
- 3 righe
- 4 colonne
- totale elementi = 3 × 4 = 12
- ogni elemento è `int`

È memoria contigua: come un array monodimensionale ma più grande. Gli elementi sono disposti riga per riga in memoria.

---

## Blocco 1: Dichiarare e inizializzare una matrice

### Dichiarazione

```c
int m[3][4];          /* matrice di interi, 3 righe, 4 colonne */
double tabella[5][5]; /* matrice quadrata di double */
```

Con `#define` per le dimensioni — sempre consigliato:

```c
#define RIGHE 3
#define COLONNE 4

int m[RIGHE][COLONNE];
```

### Inizializzazione

**Forma "tabella"** — più leggibile:

```c
int m[3][3] = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};
```

**Forma lineare** — equivalente:

```c
int m[2][3] = {1, 2, 3, 4, 5, 6};
```

Entrambe generano la stessa struttura in memoria.

**Azzerare tutto:**

```c
int m[3][4] = {0};   /* tutti gli elementi a 0 */
```

### Accesso a un elemento

```c
m[0][2]   /* riga 0, colonna 2 */
m[1][3]   /* riga 1, colonna 3 */
```

Prima l'indice riga, poi l'indice colonna. Come dire: "vai alla riga 0, poi alla colonna 2".

::: {.callout-important}
## Gli indici partono da 0
Una matrice `m[RIGHE][COLONNE]` ha righe da `0` a `RIGHE-1` e colonne da `0` a `COLONNE-1`. Uscire da questi limiti è un errore.

```c
int m[3][4];
m[3][0] = 1;   /* ERRORE — la riga 3 non esiste */
m[0][4] = 1;   /* ERRORE — la colonna 4 non esiste */
```
:::

---

## Blocco 2: Saper scorrere una matrice

Questo è il cuore di quasi tutto: se non sai fare questo, non puoi fare né somme né ricerche.

Lo schema mentale deve essere: **per ogni riga, scorri tutte le colonne**.

```c
int i, j;
for (i = 0; i < RIGHE; i++) {
    for (j = 0; j < COLONNE; j++) {
        /* lavoro su m[i][j] */
    }
}
```

### Leggere una matrice dall'utente

```c
int i, j;
for (i = 0; i < RIGHE; i++) {
    for (j = 0; j < COLONNE; j++) {
        printf("m[%d][%d]: ", i, j);
        scanf("%d", &m[i][j]);
    }
}
```

### Stampare una matrice

```c
int i, j;
for (i = 0; i < RIGHE; i++) {
    for (j = 0; j < COLONNE; j++) {
        printf("%4d", m[i][j]);
    }
    printf("\n");   /* a capo dopo ogni riga */
}
```

Output (per la matrice 3×3 con valori 1-9):
```
   1   2   3
   4   5   6
   7   8   9
```

### Somma di tutti gli elementi

```c
int somma = 0;
for (i = 0; i < RIGHE; i++) {
    for (j = 0; j < COLONNE; j++) {
        somma += m[i][j];
    }
}
printf("Somma: %d\n", somma);
```

### Ricerca del massimo

```c
int max = m[0][0];
for (i = 0; i < RIGHE; i++) {
    for (j = 0; j < COLONNE; j++) {
        if (m[i][j] > max)
            max = m[i][j];
    }
}
printf("Massimo: %d\n", max);
```

### Somma di una riga specifica

```c
int r = 1;   /* seconda riga */
int s = 0;
for (j = 0; j < COLONNE; j++) {
    s += m[r][j];
}
printf("Somma riga %d: %d\n", r, s);
```

### Somma di una colonna specifica

```c
int c = 2;   /* terza colonna */
int s = 0;
for (i = 0; i < RIGHE; i++) {
    s += m[i][c];
}
printf("Somma colonna %d: %d\n", c, s);
```

### Diagonale principale (solo matrici quadrate)

```c
/* La diagonale principale ha i == j */
int s = 0;
for (i = 0; i < RIGHE; i++) {
    s += m[i][i];
}
printf("Somma diagonale: %d\n", s);
```

---

## Blocco 3: Funzioni con le matrici

Quando passi una matrice a una funzione, devi specificare almeno la dimensione delle colonne.

```c
void stampa(int m[][4], int righe) {
    int i, j;
    for (i = 0; i < righe; i++) {
        for (j = 0; j < 4; j++) {
            printf("%4d", m[i][j]);
        }
        printf("\n");
    }
}
```

Uso: `stampa(m, 3);`

::: {.callout-note}
## Perché bisogna specificare le colonne?
La memoria è contigua riga per riga. Per sapere dove inizia la riga `i`, il compilatore deve calcolare l'offset: `i * numero_colonne`. Senza sapere quante colonne ci sono, non può fare questo calcolo.

Il numero di righe invece puoi passarlo come parametro, come facevi con gli array monodimensionali.
:::

### Schema delle funzioni più comuni

```c
/* Somma tutti gli elementi */
int somma(int m[][COLONNE], int righe) {
    int i, j, s = 0;
    for (i = 0; i < righe; i++)
        for (j = 0; j < COLONNE; j++)
            s += m[i][j];
    return s;
}

/* Trova il massimo */
int massimo(int m[][COLONNE], int righe) {
    int i, j, max = m[0][0];
    for (i = 0; i < righe; i++)
        for (j = 0; j < COLONNE; j++)
            if (m[i][j] > max)
                max = m[i][j];
    return max;
}

/* Stampa la matrice */
void stampa(int m[][COLONNE], int righe) {
    int i, j;
    for (i = 0; i < righe; i++) {
        for (j = 0; j < COLONNE; j++)
            printf("%4d", m[i][j]);
        printf("\n");
    }
}
```

---

## Blocco 4: sizeof e matrici

Puoi usare `sizeof` per calcolare dimensioni e indici — ma solo nello stesso scope in cui la matrice è dichiarata, non dentro le funzioni.

```c
int m[3][4];

int totale_byte  = sizeof(m);             /* 3*4*4 = 48 byte */
int byte_riga    = sizeof(m[0]);          /* 4*4   = 16 byte */
int byte_elem    = sizeof(m[0][0]);       /* 4 byte */

int righe    = sizeof(m)    / sizeof(m[0]);       /* 3 */
int colonne  = sizeof(m[0]) / sizeof(m[0][0]);    /* 4 */
```

::: {.callout-warning}
## sizeof non funziona dentro le funzioni
Quando passi una matrice a una funzione, il parametro decade a puntatore e `sizeof` non restituisce più la dimensione reale della matrice. Per questo si passa sempre il numero di righe come parametro separato.

```c
void funzione(int m[][4], int righe) {
    /* sizeof(m) qui NON dà la dimensione della matrice */
    /* usa il parametro righe e la costante 4 */
}
```
:::

---

## Esempio completo

```c
#include <stdio.h>

#define RIGHE   3
#define COLONNE 3

void leggi(int m[][COLONNE], int righe);
void stampa(int m[][COLONNE], int righe);
int  somma(int m[][COLONNE], int righe);
int  massimo(int m[][COLONNE], int righe);
void sommaDiagonale(int m[][COLONNE], int righe);

int main() {
    int m[RIGHE][COLONNE];

    printf("Inserisci la matrice %dx%d:\n", RIGHE, COLONNE);
    leggi(m, RIGHE);

    printf("\nMatrice inserita:\n");
    stampa(m, RIGHE);

    printf("\nSomma totale:  %d\n", somma(m, RIGHE));
    printf("Massimo:       %d\n",  massimo(m, RIGHE));
    sommaDiagonale(m, RIGHE);

    return 0;
}

void leggi(int m[][COLONNE], int righe) {
    int i, j;
    for (i = 0; i < righe; i++)
        for (j = 0; j < COLONNE; j++) {
            printf("m[%d][%d]: ", i, j);
            scanf("%d", &m[i][j]);
        }
}

void stampa(int m[][COLONNE], int righe) {
    int i, j;
    for (i = 0; i < righe; i++) {
        for (j = 0; j < COLONNE; j++)
            printf("%4d", m[i][j]);
        printf("\n");
    }
}

int somma(int m[][COLONNE], int righe) {
    int i, j, s = 0;
    for (i = 0; i < righe; i++)
        for (j = 0; j < COLONNE; j++)
            s += m[i][j];
    return s;
}

int massimo(int m[][COLONNE], int righe) {
    int i, j, max = m[0][0];
    for (i = 0; i < righe; i++)
        for (j = 0; j < COLONNE; j++)
            if (m[i][j] > max)
                max = m[i][j];
    return max;
}

void sommaDiagonale(int m[][COLONNE], int righe) {
    int i, s = 0;
    for (i = 0; i < righe; i++)
        s += m[i][i];
    printf("Somma diagonale: %d\n", s);
}
```

---

## Schema mentale

```
Devo lavorare su tutta la matrice?
    → due cicli annidati: for i (righe) → for j (colonne)

Devo lavorare su una sola riga r?
    → un ciclo: for j (colonne) → usa m[r][j]

Devo lavorare su una sola colonna c?
    → un ciclo: for i (righe) → usa m[i][c]

Devo lavorare sulla diagonale? (solo matrici quadrate)
    → un ciclo: for i → usa m[i][i]

Devo passare la matrice a una funzione?
    → specifica le colonne: int m[][COLONNE]
    → passa le righe come parametro
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge una matrice 3×3 e stampa la somma di ogni riga e di ogni colonna.

**Esercizio 2**
Scrivi una funzione `int contieneValore(int m[][COLONNE], int righe, int val)` che restituisce 1 se il valore `val` è presente nella matrice, 0 altrimenti.

**Esercizio 3**
Scrivi una funzione `void trasposta(int m[][COLONNE], int t[][RIGHE], int righe)` che calcola la trasposta di una matrice (scambia righe e colonne).

**Esercizio 4**
Scrivi un programma che legge una matrice quadrata NxN e verifica se è simmetrica (cioè se `m[i][j] == m[j][i]` per ogni `i` e `j`).