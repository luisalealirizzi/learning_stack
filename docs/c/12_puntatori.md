# Puntatori

---

## Idea chiave

Un puntatore è una variabile che contiene un **indirizzo di memoria**.

Non contiene direttamente un valore come `5` o `'A'`, ma dice **dove si trova** quel valore.

```c
int x = 10;
int *p = &x;
```

Qui:
- `x` è una variabile normale che contiene `10`
- `&x` significa: indirizzo di `x`
- `p` è un puntatore a intero
- `p` contiene l'indirizzo di `x`

---

## Blocco 1: Capire cos'è un puntatore

Devi distinguere bene tre cose:

```
la variabile       →    x
il suo indirizzo   →    &x
il valore puntato  →    *p
```

Riprendiamo l'esempio:

```c
int x = 10;
int *p = &x;
```

| Espressione | Significato | Valore |
|---|---|---|
| `x` | la variabile | `10` |
| `&x` | indirizzo di `x` | es. `0x7fff...` |
| `p` | contiene `&x` | es. `0x7fff...` |
| `*p` | valore all'indirizzo puntato | `10` |

Cose da sapere bene:
- `*` nella **dichiarazione** significa: "questa variabile è un puntatore"
- `&` significa: "prendi l'indirizzo di questa variabile"
- `*p` nell'**uso** significa: "vai all'indirizzo contenuto in `p` e leggi/modifica il valore"

::: {.callout-note}
## Idea chiave
Se `p` punta a `x`, allora `*p` è un altro modo per accedere a `x`. Modificare `*p` modifica direttamente `x`.
:::

---

## Blocco 2: Saper leggere questa scrittura

Questa è la scrittura che devi saper leggere senza esitazione:

```c
int x = 10;
int *p = &x;
```

Si legge così: **`p` è un puntatore a `int` e contiene l'indirizzo di `x`**.

Se poi scrivi:

```c
printf("%d\n", *p);
```

significa: stampa il valore contenuto nella variabile puntata da `p` — quindi stampa `10`.

### Modificare una variabile tramite puntatore

```c
int x = 10;
int *p = &x;
*p = 25;
printf("%d\n", x);   /* stampa 25 */
```

Il risultato è `25` perché modificando `*p` stai modificando direttamente `x`.

---

## Blocco 3: Operatori fondamentali

Con i puntatori ci sono due operatori essenziali.

### Operatore `&` — prendi l'indirizzo

Serve per ottenere l'indirizzo di una variabile.

```c
int x = 7;
printf("%p\n", &x);   /* stampa l'indirizzo di x */
```

### Operatore `*` — due significati diversi

**1. Nella dichiarazione:**
```c
int *p;   /* p è un puntatore a intero */
```

**2. Nell'uso:**
```c
*p   /* valore contenuto nella cella di memoria puntata da p */
```

### Esempio completo

```c
int x = 7;
int *p = &x;

printf("%d\n", x);    /* 7          — valore di x */
printf("%p\n", &x);   /* indirizzo  — indirizzo di x */
printf("%p\n", p);    /* indirizzo  — p contiene lo stesso indirizzo */
printf("%d\n", *p);   /* 7          — valore puntato da p */
```

Cosa devi osservare: `&x` e `p` sono uguali come indirizzo. `x` e `*p` sono uguali come valore.

---

## Blocco 4: Errori da non fare

### Errore 1: usare un puntatore non inizializzato

```c
int *p;
*p = 5;   /* SBAGLIATO */
```

Questo è sbagliato perché `p` non punta a nessuna zona valida di memoria. Stai dicendo: "vai all'indirizzo contenuto in `p` e scrivi `5`" — ma quell'indirizzo potrebbe essere una zona di memoria qualsiasi, memoria di un altro programma, una zona non accessibile.

Questo comportamento si chiama **undefined behavior** — il linguaggio C non garantisce cosa succede. Ci sono tre scenari possibili:

**Caso 1: fortuna**
`p` contiene un indirizzo che esiste ed è scrivibile → il programma non crasha → sembra funzionare.

**Caso 2: crash immediato**
Segmentation fault / access violation.

**Caso 3: bug silenzioso** *(il peggiore)*
Scrivi in memoria "valida" ma sbagliata → il programma continua, ma è corrotto.

::: {.callout-important}
## Inizializza sempre i puntatori
Prima di usare un puntatore, fallo puntare a una variabile valida con `&`, oppure impostalo a `NULL` per indicare che non punta a niente.

```c
int *p = NULL;    /* puntatore nullo — non punta a niente */
int x = 10;
int *q = &x;     /* puntatore valido — punta a x */
```
:::

### Errore 2: confondere indirizzo e valore

```c
int x = 10;
int *p = &x;
printf("%d\n", p);    /* concettualmente sbagliato */
printf("%d\n", *p);   /* corretto per il valore */
```

Per stampare un indirizzo si usa `%p`:

```c
printf("%p\n", p);    /* corretto per l'indirizzo */
```

### Errore 3: tipo non compatibile

```c
int x = 10;
char *p = &x;   /* SBAGLIATO — p è puntatore a char, &x è indirizzo di int */
```

Regola fondamentale: **un puntatore deve puntare a una variabile del tipo corretto**.

---

## Blocco 5: Il legame tra puntatori e funzioni

Questo è il motivo per cui i puntatori sono importanti.

In C, una funzione riceve normalmente una **copia** dei parametri. Quindi se scrivo:

```c
void modifica(int x) {
    x = 20;
}

int a = 10;
modifica(a);
printf("%d\n", a);   /* stampa 10 — a non è cambiato */
```

La funzione ha modificato solo la copia locale di `x` — `a` è rimasto invariato.

### Come modificare davvero una variabile

Si passa il suo **indirizzo**.

```c
void modifica(int *p) {
    *p = 20;
}

int a = 10;
modifica(&a);
printf("%d\n", a);   /* stampa 20 — a è cambiato */
```

Schema mentale:

```
se passi a    →  la funzione riceve il valore — non può modificare a
se passi &a   →  la funzione riceve l'indirizzo
se usa *p     →  modifica direttamente a
```

---

## Blocco 6: Schema tipico di funzione con puntatore

### Caso 1: modificare una variabile

```c
void raddoppia(int *p) {
    *p = (*p) * 2;
}

int x = 5;
raddoppia(&x);
printf("%d\n", x);   /* 10 */
```

### Caso 2: scambiare due variabili

Questo è uno dei classici più importanti.

```c
void scambia(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int x = 3, y = 8;
scambia(&x, &y);
/* x vale 8, y vale 3 */
```

Perché servono i puntatori? Perché senza di essi la funzione scambierebbe solo le copie, non le variabili originali.

---

## Blocco 7: Puntatori e array

Il nome di un array è strettamente collegato a un puntatore.

```c
int v[5] = {10, 20, 30, 40, 50};
```

`v` rappresenta l'indirizzo del primo elemento dell'array — cioè `v` è l'indirizzo di `v[0]`.

### Equivalenze fondamentali

```c
v[0] == *v
v[1] == *(v + 1)
v[2] == *(v + 2)
```

```c
int v[3] = {4, 7, 9};

printf("%d\n", v[0]);      /* 4 */
printf("%d\n", *v);        /* 4 — stesso valore */

printf("%d\n", v[1]);      /* 7 */
printf("%d\n", *(v + 1));  /* 7 — stesso valore */
```

### Scorrere un array con un puntatore

```c
int v[5] = {1, 2, 3, 4, 5};
int *p = v;   /* p punta al primo elemento */

for (i = 0; i < 5; i++) {
    printf("%d\n", *p);
    p++;   /* sposta il puntatore al prossimo elemento */
}
```

---

## Schema mentale

```
Cosa contiene p?
    → un indirizzo di memoria

Come faccio puntare p a x?
    → p = &x

Come leggo il valore puntato?
    → *p

Come modifico il valore puntato?
    → *p = nuovo_valore

Voglio che una funzione modifichi una variabile?
    → passa &a, ricevi int *p, usa *p dentro la funzione

Puntatore e array?
    → v e &v[0] sono la stessa cosa
    → v[i] e *(v+i) sono equivalenti
```

---

## Esercizi

### Esercizio 1: Primo contatto

Dichiara una variabile `int x` e un puntatore `p`. Assegna a `x` un valore a tua scelta, fai puntare `p` a `x` e stampa: il valore di `x`, l'indirizzo di `x`, il valore di `p`, il valore puntato da `p`.

Soluzione:

```c
#include <stdio.h>

int main() {
    int x = 10;
    int *p = &x;

    printf("x = %d\n", x);
    printf("indirizzo di x = %p\n", &x);
    printf("p = %p\n", p);
    printf("valore puntato da p = %d\n", *p);

    return 0;
}
```

### Esercizio 2: Modifica tramite puntatore

Dichiara `int x = 10`, usa un puntatore per modificarla a `25`, stampa il valore finale.

Soluzione:

```c
#include <stdio.h>

int main() {
    int x = 10;
    int *p = &x;
    *p = 25;
    printf("%d\n", x);
    return 0;
}
```

### Esercizio 3: Funzione che azzera

Scrivi `void azzera(int *p)` che imposta a 0 il valore della variabile.

Soluzione:

```c
#include <stdio.h>

void azzera(int *p) {
    *p = 0;
}

int main() {
    int x = 7;
    azzera(&x);
    printf("%d\n", x);
    return 0;
}
```

### Esercizio 4: Raddoppio

Scrivi `void raddoppia(int *p)` che raddoppia il valore.

Soluzione:

```c
#include <stdio.h>

void raddoppia(int *p) {
    *p = (*p) * 2;
}

int main() {
    int x = 5;
    raddoppia(&x);
    printf("%d\n", x);
    return 0;
}
```

### Esercizio 5: Scambio di due variabili

Scrivi `void scambia(int *a, int *b)`.

Soluzione:

```c
#include <stdio.h>

void scambia(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main() {
    int x = 3, y = 8;
    scambia(&x, &y);
    printf("%d %d\n", x, y);
    return 0;
}
```

### Esercizio 6: Puntatore e array

Dato `int v[5] = {10, 20, 30, 40, 50}`, stampa il primo elemento usando sia `v[0]` sia un puntatore `*p`.

Soluzione:

```c
#include <stdio.h>

int main() {
    int v[5] = {10, 20, 30, 40, 50};
    int *p = v;

    printf("%d\n", v[0]);
    printf("%d\n", *p);

    return 0;
}
```

### Esercizio 7: Scorrere un array con puntatore

Stampa tutti gli elementi usando un puntatore.

Soluzione:

```c
#include <stdio.h>

int main() {
    int v[5] = {1, 2, 3, 4, 5};
    int *p = v;
    int i;

    for (i = 0; i < 5; i++) {
        printf("%d\n", *p);
        p++;
    }

    return 0;
}
```

### Esercizio 8: Somma con puntatore

Scrivi `int somma(int v[], int n)` usando un puntatore internamente.

Soluzione:

```c
#include <stdio.h>

int somma(int v[], int n) {
    int *p = v;
    int s = 0, i;

    for (i = 0; i < n; i++) {
        s += *p;
        p++;
    }

    return s;
}

int main() {
    int v[4] = {2, 4, 6, 8};
    printf("%d\n", somma(v, 4));
    return 0;
}
```

### Esercizio 9: Indice vs puntatore

Scrivi due versioni per stampare l'array: una con `v[i]`, una con `*(v + i)`.

Soluzione:

```c
#include <stdio.h>

int main() {
    int v[4] = {2, 4, 6, 8};
    int i;

    for (i = 0; i < 4; i++)
        printf("%d\n", v[i]);

    for (i = 0; i < 4; i++)
        printf("%d\n", *(v + i));

    return 0;
}
```

### Esercizio 10: Massimo con puntatore

Scrivi `int massimo(int v[], int n)` usando un puntatore.

Soluzione:

```c
#include <stdio.h>

int massimo(int v[], int n) {
    int *p = v;
    int max = *p;
    int i;

    for (i = 0; i < n; i++) {
        if (*p > max)
            max = *p;
        p++;
    }

    return max;
}

int main() {
    int v[5] = {3, 7, 2, 9, 5};
    printf("%d\n", massimo(v, 5));
    return 0;
}
```

### Esercizio 11: Restituire un puntatore

Scrivi `int* trovaPari(int v[], int n)` che restituisce il puntatore al primo elemento pari, `NULL` se non esiste.

Soluzione:

```c
#include <stdio.h>

int* trovaPari(int v[], int n) {
    int *p = v;
    int i;

    for (i = 0; i < n; i++) {
        if (*p % 2 == 0)
            return p;
        p++;
    }

    return NULL;
}

int main() {
    int v[5] = {1, 3, 5, 8, 9};
    int *ris = trovaPari(v, 5);

    if (ris != NULL)
        printf("%d\n", *ris);
    else
        printf("Nessun pari\n");

    return 0;
}
```