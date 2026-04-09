# Ordinamento per scambio — Bubble Sort

---

## Idea chiave

Ordinare un array significa riorganizzare i suoi elementi in ordine crescente o decrescente. E uno dei problemi fondamentali dell'informatica — e ci sono molti modi per risolverlo, ognuno con caratteristiche diverse.

Il **Bubble Sort** e il piu semplice da capire. L'idea e questa: scorri l'array piu volte, e ad ogni passaggio confronta due elementi adiacenti. Se sono nell'ordine sbagliato, scambiali. Ripeti finche l'array e ordinato.

Il nome viene dal fatto che gli elementi piu grandi "galleggiano" verso la fine dell'array come bolle d'aria in un liquido — ad ogni passaggio il valore massimo si sposta all'ultima posizione libera.

---

## Blocco 1: Come funziona — passo per passo

Partiamo da un array di esempio:

```
[5, 3, 8, 1, 4]
```

**Primo passaggio — confronta coppie adiacenti:**

```
[5, 3, 8, 1, 4]   → confronta 5 e 3 → scambia → [3, 5, 8, 1, 4]
[3, 5, 8, 1, 4]   → confronta 5 e 8 → ok       → [3, 5, 8, 1, 4]
[3, 5, 8, 1, 4]   → confronta 8 e 1 → scambia → [3, 5, 1, 8, 4]
[3, 5, 1, 8, 4]   → confronta 8 e 4 → scambia → [3, 5, 1, 4, 8]
```

Dopo il primo passaggio: `[3, 5, 1, 4, 8]` — l'8 e in posizione corretta.

**Secondo passaggio:**
```
[3, 5, 1, 4, 8]   → confronta 3 e 5 → ok       → [3, 5, 1, 4, 8]
[3, 5, 1, 4, 8]   → confronta 5 e 1 → scambia → [3, 1, 5, 4, 8]
[3, 1, 5, 4, 8]   → confronta 5 e 4 → scambia → [3, 1, 4, 5, 8]
```

Dopo il secondo passaggio: `[3, 1, 4, 5, 8]` — 5 e 8 sono in posizione corretta.

**Terzo passaggio:**
```
[3, 1, 4, 5, 8]   → confronta 3 e 1 → scambia → [1, 3, 4, 5, 8]
[1, 3, 4, 5, 8]   → confronta 3 e 4 → ok
```

Dopo il terzo passaggio: `[1, 3, 4, 5, 8]` — array ordinato.

Schema mentale: **ad ogni passaggio il valore piu grande non ancora sistemato sale alla sua posizione finale**.

---

## Blocco 2: Il codice

```c
#include <stdio.h>

#define N 5

void bubbleSort(int v[], int n) {
    int i, j, temp;

    for (i = 0; i < n - 1; i++) {           /* passaggi */
        for (j = 0; j < n - 1 - i; j++) {   /* confronti */
            if (v[j] > v[j + 1]) {
                /* scambio */
                temp     = v[j];
                v[j]     = v[j + 1];
                v[j + 1] = temp;
            }
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

    bubbleSort(v, N);

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

Il cuore del Bubble Sort sono due cicli annidati. E importante capire cosa fa ognuno.

```c
for (i = 0; i < n - 1; i++) {           /* ciclo esterno — passaggi */
    for (j = 0; j < n - 1 - i; j++) {   /* ciclo interno — confronti */
        if (v[j] > v[j + 1]) {
            /* scambio */
        }
    }
}
```

**Ciclo esterno** — conta i passaggi. Con N elementi servono al massimo N-1 passaggi. Dopo ogni passaggio, un elemento in piu e nella posizione corretta.

**Ciclo interno** — fa i confronti. Ad ogni passaggio `i`, gli ultimi `i` elementi sono gia ordinati — non serve confrontarli di nuovo. Per questo il limite e `n - 1 - i` invece di `n - 1`.

**Lo scambio** — usa sempre una variabile temporanea `temp`. Senza di essa perderesti uno dei due valori.

```c
/* Scambio corretto */
temp     = v[j];
v[j]     = v[j + 1];
v[j + 1] = temp;

/* Scambio sbagliato — perde il valore originale di v[j] */
v[j]     = v[j + 1];   /* v[j] e sovrascritto */
v[j + 1] = v[j];       /* copia lo stesso valore */
```

::: {.callout-note}
## Perche n-1-i?
Dopo il primo passaggio, l'elemento piu grande e in fondo — non serve confrontarlo.
Dopo il secondo, i due piu grandi sono in fondo — non serve confrontarli.
E cosi via. Ogni passaggio fa meno confronti del precedente.
:::

---

## Blocco 4: Versione ottimizzata

Il Bubble Sort base fa sempre lo stesso numero di passaggi, anche se l'array e gia ordinato. Una piccola ottimizzazione: se in un passaggio non avviene nessuno scambio, l'array e gia ordinato e possiamo fermarci.

```c
void bubbleSortOttimizzato(int v[], int n) {
    int i, j, temp;
    int scambiato;

    for (i = 0; i < n - 1; i++) {
        scambiato = 0;   /* nessuno scambio per ora */

        for (j = 0; j < n - 1 - i; j++) {
            if (v[j] > v[j + 1]) {
                temp     = v[j];
                v[j]     = v[j + 1];
                v[j + 1] = temp;
                scambiato = 1;   /* e avvenuto almeno uno scambio */
            }
        }

        if (scambiato == 0)
            break;   /* nessuno scambio — array gia ordinato */
    }
}
```

Con un array gia ordinato, la versione ottimizzata fa un solo passaggio invece di N-1.

---

## Blocco 5: Ordinamento decrescente

Per ordinare in ordine decrescente basta invertire la condizione del confronto:

```c
/* Ordine crescente */
if (v[j] > v[j + 1]) { /* scambio */ }

/* Ordine decrescente */
if (v[j] < v[j + 1]) { /* scambio */ }
```

---

## Blocco 6: Bubble Sort su array di struct

Il Bubble Sort funziona anche su array di struct — basta confrontare il campo su cui vuoi ordinare.

```c
typedef struct {
    char nome[50];
    double media;
} Studente;

void ordinaPerMedia(Studente v[], int n) {
    int i, j;
    Studente temp;

    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - 1 - i; j++) {
            if (v[j].media > v[j + 1].media) {
                temp     = v[j];
                v[j]     = v[j + 1];
                v[j + 1] = temp;
            }
        }
    }
}
```

Lo scambio funziona allo stesso modo — ma invece di scambiare un intero, scambi una struct intera con la variabile temporanea `Studente temp`.

---

## Schema mentale

```
Come funziona il Bubble Sort?
    -> confronta coppie adiacenti
    -> se sono nell'ordine sbagliato, scambiale
    -> ripeti per N-1 passaggi
    -> ad ogni passaggio il valore piu grande sale in fondo

Quanti cicli servono?
    -> ciclo esterno: N-1 passaggi
    -> ciclo interno: N-1-i confronti

Come si scambia?
    -> temp = v[j]; v[j] = v[j+1]; v[j+1] = temp;
    -> serve sempre una variabile temporanea

Come si ordina in modo decrescente?
    -> inverti la condizione: < invece di >

Come si ordina un array di struct?
    -> confronta v[j].campo
    -> scambia l'intera struct con Struttura temp
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge 8 numeri interi dall'utente, li ordina con Bubble Sort in ordine crescente e li stampa.

**Esercizio 2**
Modifica il programma dell'esercizio 1 per ordinare in ordine decrescente.

**Esercizio 3**
Scrivi una funzione `void ordinaStringhe(char v[][50], int n)` che ordina un array di stringhe in ordine alfabetico usando Bubble Sort e `strcmp`.

**Esercizio 4**
Dichiara una struct `Studente` con nome e media. Scrivi un programma che:
- legge 5 studenti
- li ordina per media crescente con Bubble Sort
- stampa la classifica finale
