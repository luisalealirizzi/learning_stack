# Array

---

## Blocco 1: Dichiarare e accedere a un array

Un array è una sequenza di variabili dello stesso tipo, tutte con lo stesso nome, accessibili tramite un indice.

Senza array, per memorizzare 5 voti dovresti scrivere:

```c
int voto1, voto2, voto3, voto4, voto5;
```

Con un array:

```c
int voti[5];
```

Un solo nome, cinque celle. Se i voti fossero cento, cambieresti solo il numero.

### Dichiarazione

```c
int voti[5];           /* array di 5 interi */
double temperature[10]; /* array di 10 double */
char lettere[26];       /* array di 26 caratteri */
```

Il numero dentro le parentesi quadre è la **dimensione** — quante celle ha l'array. Deve essere un valore costante.

### Accesso agli elementi

Ogni elemento ha un **indice** — un numero che indica la sua posizione. In C gli indici partono da **0**, non da 1.

```c
int voti[5];

voti[0] = 7;   /* primo elemento */
voti[1] = 8;   /* secondo elemento */
voti[2] = 6;   /* terzo elemento */
voti[3] = 9;   /* quarto elemento */
voti[4] = 7;   /* quinto elemento */

printf("%d\n", voti[0]);   /* stampa 7 */
printf("%d\n", voti[2]);   /* stampa 6 */
```

::: {.callout-important}
## Gli indici partono da 0
Un array di dimensione `N` ha indici da `0` a `N-1`. L'indice `N` non esiste — accedervi è un errore che può causare comportamenti imprevedibili.

```c
int voti[5];
voti[5] = 10;   /* ERRORE — l'indice 5 è fuori dall'array */
```
:::

### Inizializzazione

Puoi inizializzare un array al momento della dichiarazione:

```c
int voti[5] = {7, 8, 6, 9, 7};

/* Se dai meno valori, il resto viene inizializzato a 0 */
int voti[5] = {7, 8};   /* voti[2], voti[3], voti[4] valgono 0 */

/* Trucco per azzerare tutto l'array */
int voti[5] = {0};
```

### Usare #define per la dimensione

È buona pratica definire la dimensione dell'array con una costante:

```c
#define N 5

int voti[N];
```

Così se devi cambiare la dimensione, la modifichi in un solo posto.

---

## Blocco 2: Saper scorrere un array

Questo è il cuore di quasi tutto: se non sai fare questo, non puoi fare né somme né ricerche né ordinamenti.

Lo schema mentale deve essere: **accedo a un elemento alla volta, dall'indice 0 fino all'ultimo**.

```c
#define N 5
int voti[N];
int i;

for (i = 0; i < N; i++) {
    /* lavoro su voti[i] */
}
```

Il ciclo `for` è perfetto per gli array perché sai sempre quanti elementi ci sono.

### Leggere un array dall'utente

```c
#define N 5
int voti[N];
int i;

for (i = 0; i < N; i++) {
    printf("Inserisci voto %d: ", i + 1);
    scanf("%d", &voti[i]);
}
```

### Stampare un array

```c
for (i = 0; i < N; i++) {
    printf("voti[%d] = %d\n", i, voti[i]);
}
```

### Calcolare la somma

```c
int somma = 0;
for (i = 0; i < N; i++) {
    somma += voti[i];
}
printf("Somma: %d\n", somma);
printf("Media: %.2f\n", (double)somma / N);
```

### Trovare il massimo

```c
int max = voti[0];   /* parti dal primo elemento */
for (i = 1; i < N; i++) {
    if (voti[i] > max) {
        max = voti[i];
    }
}
printf("Massimo: %d\n", max);
```

::: {.callout-note}
## Parti sempre dal primo elemento
Quando cerchi il massimo o il minimo, inizializza la variabile con `voti[0]` — non con 0 o con un numero a caso. Poi scorri dal secondo elemento (`i = 1`) in poi.
:::

---

## Blocco 3: Come si scrive una funzione che lavora sugli array

Una funzione che lavora su un array riceve l'array e la sua dimensione come parametri.

```c
tipo funzione(int v[], int n) {
    int i;
    for (i = 0; i < n; i++) {
        /* lavoro su v[i] */
    }
}
```

Questo schema si adatta a qualsiasi esercizio sugli array.

### Il tipo restituito

Si usa `int` quando la funzione deve restituire un conteggio, una posizione, un valore numerico, oppure 1 o 0 (trovato/non trovato).

```c
int somma(int v[], int n);
int massimo(int v[], int n);
int cercaElemento(int v[], int n, int cercato);
```

Si usa `void` quando la funzione modifica l'array o non restituisce nulla.

```c
void leggiArray(int v[], int n);
void stampaArray(int v[], int n);
void azzera(int v[], int n);
```

### I parametri

**Caso 1: array + dimensione**
```c
int somma(int v[], int n)
```
Serve per: analizzare, contare, trovare il massimo/minimo.

**Caso 2: due array + dimensione**
```c
void copia(int src[], int dst[], int n)
```
Serve per: copiare, confrontare due array.

**Caso 3: array + dimensione + valore**
```c
int cerca(int v[], int n, int elemento)
```
Serve per: cercare un elemento, contare le occorrenze di un valore.

### Schema tipico di una funzione sugli array

```c
tipo funzione(int v[], int n) {
    int i;
    for (i = 0; i < n; i++) {
        /* lavoro su v[i] */
    }
    return ...;
}
```

Questo è il cuore dell'algoritmo.

### Funzione che conta qualcosa

```c
int contaPositivi(int v[], int n) {
    int i, cont = 0;
    for (i = 0; i < n; i++) {
        if (v[i] > 0)
            cont++;
    }
    return cont;
}
```

Esempi possibili: `contaNegativi`, `contaPari`, `contaMaggioriDi`.

### Funzione che trova un valore

```c
int massimo(int v[], int n) {
    int i, max = v[0];
    for (i = 1; i < n; i++) {
        if (v[i] > max)
            max = v[i];
    }
    return max;
}
```

### Funzione che cerca un elemento

```c
int cerca(int v[], int n, int elemento) {
    int i;
    for (i = 0; i < n; i++) {
        if (v[i] == elemento)
            return i;   /* restituisce la posizione */
    }
    return -1;   /* non trovato */
}
```

::: {.callout-note}
## Restituire -1 per "non trovato"
Quando una funzione cerca qualcosa e non lo trova, restituisce `-1`. È una convenzione universale in C — -1 non è mai un indice valido, quindi non crea ambiguità.
:::

### Funzione che modifica un array

```c
void azzera(int v[], int n) {
    int i;
    for (i = 0; i < n; i++) {
        v[i] = 0;
    }
}
```

::: {.callout-important}
## Gli array si passano per riferimento
A differenza dei tipi semplici, gli array non vengono copiati quando li passi a una funzione. La funzione lavora direttamente sull'array originale. Questo significa che le modifiche fatte dentro la funzione si riflettono sull'array del chiamante.

```c
void raddoppia(int v[], int n) {
    int i;
    for (i = 0; i < n; i++) {
        v[i] *= 2;   /* modifica l'array originale */
    }
}
```
:::

---

## Blocco 4: Array bidimensionali

Un array bidimensionale è una tabella — righe e colonne.

```c
int matrice[3][4];   /* 3 righe, 4 colonne */
```

Si accede con due indici: prima la riga, poi la colonna.

```c
matrice[0][0] = 1;   /* prima riga, prima colonna */
matrice[1][2] = 7;   /* seconda riga, terza colonna */
```

### Scorrere una matrice con due cicli annidati

```c
#define RIGHE 3
#define COLONNE 4

int matrice[RIGHE][COLONNE];
int i, j;

for (i = 0; i < RIGHE; i++) {
    for (j = 0; j < COLONNE; j++) {
        /* lavoro su matrice[i][j] */
    }
}
```

Lo schema mentale: **per ogni riga, scorri tutte le colonne**.

### Leggere e stampare una matrice

```c
/* Lettura */
for (i = 0; i < RIGHE; i++) {
    for (j = 0; j < COLONNE; j++) {
        printf("Elemento [%d][%d]: ", i, j);
        scanf("%d", &matrice[i][j]);
    }
}

/* Stampa */
for (i = 0; i < RIGHE; i++) {
    for (j = 0; j < COLONNE; j++) {
        printf("%4d", matrice[i][j]);
    }
    printf("\n");   /* vai a capo dopo ogni riga */
}
```

---

## Blocco 5: Strategie per risolvere gli esercizi sugli array

Molti esercizi sugli array sembrano diversi, ma appartengono quasi sempre a quattro famiglie di problemi.

Quando vedi un esercizio, la prima cosa da fare è capire a quale famiglia appartiene.

Le quattro famiglie più comuni sono: **contare**, **trovare**, **cercare**, **trasformare**.

### Esercizi di tipo contare

Bisogna scorrere l'array e contare quante volte succede qualcosa.

Esempi: contare i pari, contare i negativi, contare i valori maggiori di una soglia.

Strategia:
- inizializzare un contatore
- scorrere l'array
- controllare se l'elemento soddisfa una condizione
- aumentare il contatore
- restituire il risultato

Schema mentale: **scorro l'array → se l'elemento soddisfa la condizione → aumento il contatore**

```c
int contaPari(int v[], int n) {
    int i, cont = 0;
    for (i = 0; i < n; i++) {
        if (v[i] % 2 == 0)
            cont++;
    }
    return cont;
}
```

Varianti possibili: cambia solo la condizione. La struttura rimane la stessa.

### Esercizi di tipo trovare

Bisogna trovare un valore particolare nell'array: il massimo, il minimo, la media, la somma.

Esempi: trovare il massimo, trovare il minimo, calcolare la media.

Strategia:
- inizializzare la variabile risultato con il primo elemento
- scorrere dal secondo elemento
- aggiornare il risultato se l'elemento corrente è migliore
- restituire il risultato

Schema mentale: **parto dal primo → confronto con il corrente → aggiorno se necessario**

```c
int minimo(int v[], int n) {
    int i, min = v[0];
    for (i = 1; i < n; i++) {
        if (v[i] < min)
            min = v[i];
    }
    return min;
}
```

Varianti possibili: massimo, minimo, somma, media. La struttura rimane la stessa.

### Esercizi di tipo cercare

Bisogna verificare se qualcosa è presente nell'array o trovare la sua posizione.

Esempi: verificare se un elemento è presente, trovare la prima occorrenza, verificare se l'array è ordinato.

Strategia:
- scorrere l'array
- confrontare ogni elemento con quello cercato
- restituire la posizione se trovato, -1 altrimenti

Schema mentale: **scorro → confronto → se trovato restituisco la posizione → altrimenti -1**

```c
int cerca(int v[], int n, int elemento) {
    int i;
    for (i = 0; i < n; i++) {
        if (v[i] == elemento)
            return i;
    }
    return -1;
}
```

Varianti possibili: trovare l'ultima occorrenza, contare le occorrenze, verificare se esiste. La logica è sempre confrontare elementi.

### Esercizi di tipo trasformare

Bisogna modificare gli elementi dell'array o crearne uno nuovo trasformato.

Esempi: raddoppiare tutti gli elementi, azzerare i negativi, copiare solo i pari.

Strategia:
- scorrere l'array
- applicare la trasformazione a ogni elemento
- scrivere il risultato nell'array originale o in uno nuovo

Schema mentale: **leggo un elemento → decido come trasformarlo → scrivo il risultato**

```c
void raddoppia(int v[], int n) {
    int i;
    for (i = 0; i < n; i++) {
        v[i] *= 2;
    }
}
```

Varianti possibili: azzerare i negativi, sostituire con il valore assoluto, copiare solo gli elementi che soddisfano una condizione.

---

Quando leggi una consegna, chiediti:
- Devo contare qualcosa? → esercizio di conteggio
- Devo trovare un valore (massimo, minimo, media)? → esercizio di ricerca del valore
- Devo verificare se qualcosa è presente? → esercizio di ricerca
- Devo modificare gli elementi? → esercizio di trasformazione

---

## Esercizio — Contare

Scrivere la funzione `int contaPari(int v[], int n)` che restituisce il numero di elementi pari nell'array.
Scrivere anche un main che:
- legge N interi
- chiama la funzione
- stampa il risultato

Soluzione:

```c
#include <stdio.h>
#define N 5

int contaPari(int v[], int n) {
    int i, cont = 0;
    for (i = 0; i < n; i++) {
        if (v[i] % 2 == 0)
            cont++;
    }
    return cont;
}

int main() {
    int voti[N];
    int i;
    for (i = 0; i < N; i++) {
        printf("Inserisci elemento %d: ", i + 1);
        scanf("%d", &voti[i]);
    }
    printf("Elementi pari: %d\n", contaPari(voti, N));
    return 0;
}
```

## Esercizio — Trovare

Scrivere la funzione `double media(int v[], int n)` che restituisce la media degli elementi dell'array.
Scrivere anche un main che:
- legge N interi
- chiama la funzione
- stampa il risultato con due decimali

Soluzione:

```c
#include <stdio.h>
#define N 5

double media(int v[], int n) {
    int i;
    double somma = 0;
    for (i = 0; i < n; i++) {
        somma += v[i];
    }
    return somma / n;
}

int main() {
    int voti[N];
    int i;
    for (i = 0; i < N; i++) {
        printf("Inserisci elemento %d: ", i + 1);
        scanf("%d", &voti[i]);
    }
    printf("Media: %.2f\n", media(voti, N));
    return 0;
}
```

## Esercizio — Cercare

Scrivere la funzione `int cerca(int v[], int n, int elemento)` che restituisce la posizione dell'elemento cercato, oppure -1 se non è presente.
Scrivere anche un main che:
- legge N interi
- legge il valore da cercare
- chiama la funzione
- stampa la posizione oppure "non trovato"

Soluzione:

```c
#include <stdio.h>
#define N 5

int cerca(int v[], int n, int elemento) {
    int i;
    for (i = 0; i < n; i++) {
        if (v[i] == elemento)
            return i;
    }
    return -1;
}

int main() {
    int voti[N];
    int i, cercato, pos;
    for (i = 0; i < N; i++) {
        printf("Inserisci elemento %d: ", i + 1);
        scanf("%d", &voti[i]);
    }
    printf("Elemento da cercare: ");
    scanf("%d", &cercato);
    pos = cerca(voti, N, cercato);
    if (pos != -1)
        printf("Trovato alla posizione %d\n", pos);
    else
        printf("Non trovato\n");
    return 0;
}
```