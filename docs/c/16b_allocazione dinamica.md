# Allocazione dinamica della memoria

---

## Idea chiave

Fino ad ora abbiamo sempre dichiarato array con una dimensione fissa, decisa **prima** di eseguire il programma:

```c
int v[10];        /* 10 è deciso a compile-time — non puoi cambiarlo */
Studente classe[30];
```

Ma spesso la dimensione giusta si conosce solo **mentre il programma gira** — dipende da cosa inserisce l'utente, da quanti record ci sono in un file, da quanti elementi servono davvero. Se dichiari un array troppo grande sprechi memoria; se lo dichiari troppo piccolo il programma si rompe quando i dati non ci stanno più.

L'allocazione dinamica risolve questo problema: puoi chiedere memoria al sistema operativo **a runtime**, esattamente della dimensione che ti serve in quel momento.

::: {.callout-note}
## Due zone di memoria diverse
- **Stack**: dove vivono le variabili locali "normali" (`int x;`, `Studente s;`). Gestito automaticamente — quando la funzione finisce, la memoria si libera da sola.
- **Heap**: dove vive la memoria allocata dinamicamente. Gestita **da te**: se non la liberi esplicitamente, resta occupata finché il programma non termina.
:::

---

## Blocco 1: malloc e free

`malloc` chiede al sistema un blocco di memoria di una certa dimensione in byte, e restituisce un puntatore a quel blocco (oppure `NULL` se la richiesta fallisce).

```c
#include <stdlib.h>

int *v;
int n = 5;

v = malloc(n * sizeof(int));   /* chiede spazio per 5 interi */

if (v == NULL) {
    printf("Allocazione fallita\n");
    return 1;
}

for (int i = 0; i < n; i++) {
    v[i] = i * 10;
}

for (int i = 0; i < n; i++) {
    printf("%d\n", v[i]);
}

free(v);   /* restituisce la memoria al sistema */
```

::: {.callout-important}
## Controlla sempre il valore restituito da malloc
`malloc` può fallire (poca memoria disponibile) e restituire `NULL`. Usare un puntatore `NULL` per leggere o scrivere causa un crash immediato — è lo stesso errore del "puntatore non inizializzato" che hai già visto, solo con un'altra causa.
:::

### Perché `sizeof(int)` e non un numero fisso

```c
v = malloc(n * sizeof(int));    /* corretto — portabile */
v = malloc(n * 4);              /* SBAGLIATO — assume che int sia sempre 4 byte */
```

`sizeof` calcola la dimensione reale del tipo su quella macchina — non devi mai indovinarla a mano.

### Perché serve `free`

Ogni blocco allocato con `malloc` resta occupato finché non chiami `free` su di esso, anche se la funzione che l'ha allocato è già terminata. Se perdi il puntatore a un blocco senza averlo liberato, quella memoria resta occupata per sempre — è una **memory leak** (perdita di memoria).

```c
void esempio() {
    int *p = malloc(100 * sizeof(int));
    /* uso p... */
}   /* p esce di scope, ma la memoria allocata NON viene liberata automaticamente */
```

Questo è diverso dallo stack: quando `esempio()` finisce, la variabile `p` sparisce, ma il blocco di memoria nell'heap a cui puntava resta lì, occupato, e ormai irraggiungibile.

---

## Blocco 2: calloc e realloc

### calloc — alloca e azzera

```c
int *v = calloc(n, sizeof(int));   /* n elementi, tutti inizializzati a 0 */
```

Equivalente a `malloc` ma garantisce che la memoria parta azzerata — utile quando vuoi essere sicuro che non ci siano valori casuali residui.

### realloc — ridimensiona un blocco già allocato

```c
int *v = malloc(5 * sizeof(int));
/* ... uso v con 5 elementi ... */

v = realloc(v, 10 * sizeof(int));   /* ora v ha spazio per 10 elementi */
```

`realloc` prova ad allargare (o restringere) il blocco esistente. Se non riesce a farlo nello stesso punto di memoria, ne alloca uno nuovo altrove, copia automaticamente i dati vecchi, e libera quello vecchio — a te resta solo il nuovo puntatore.

::: {.callout-important}
## Occhio al valore di ritorno di realloc
```c
v = realloc(v, nuova_dim);              /* PERICOLOSO */
int *tmp = realloc(v, nuova_dim);       /* corretto */
if (tmp == NULL) {
    /* v è ancora valido — realloc non l'ha toccato */
} else {
    v = tmp;
}
```
Se `realloc` fallisce, restituisce `NULL` — ma il blocco originale resta ancora valido. Se sovrascrivi direttamente `v` con `NULL`, perdi per sempre il riferimento al blocco originale: è una memory leak garantita.
:::

---

## Blocco 3: Errori tipici con la memoria dinamica

Hai già visto nel modulo sulla sicurezza come un accesso fuori dai limiti di un array possa essere sfruttato. Con la memoria dinamica ci sono altri tre errori classici, specifici di `malloc`/`free`:

### Dangling pointer (puntatore penzolante)

```c
int *p = malloc(sizeof(int));
*p = 5;
free(p);
printf("%d\n", *p);   /* SBAGLIATO — p punta a memoria già liberata */
```

Dopo `free`, `p` contiene ancora lo stesso indirizzo, ma quella memoria non è più tua — può essere già stata riassegnata ad altro.

### Double free

```c
free(p);
free(p);   /* SBAGLIATO — liberi due volte lo stesso blocco */
```

::: {.callout-important}
## L'abitudine che previene entrambi gli errori
```c
free(p);
p = NULL;   /* ora p non punta più a niente di valido */
```
Con `p == NULL`, un secondo `free(p)` non fa danni, e un tentativo di dereferenziare `p` per errore crasha subito in modo diagnosticabile invece di corrompere silenziosamente la memoria.
:::

### Memory leak

```c
int *p = malloc(sizeof(int));
p = malloc(sizeof(int));   /* il primo blocco è perso — nessuno lo libera più */
```

Il primo indirizzo restituito da `malloc` viene sovrascritto prima di chiamare `free` su di esso: quel blocco resta occupato per sempre, senza che nessun puntatore lo raggiunga più.

---

## Blocco 4: Allocazione dinamica di struct

Le struct si allocano dinamicamente esattamente come i tipi semplici — e qui l'operatore `->` (che hai visto nel modulo *Struct e funzioni*) diventa lo strumento con cui ci lavori sempre.

```c
typedef struct {
    char nome[50];
    int matricola;
    double media;
} Studente;

Studente *s = malloc(sizeof(Studente));

if (s == NULL) {
    return 1;
}

strcpy(s->nome, "Mario");
s->matricola = 12345;
s->media = 8.5;

printf("%s\n", s->nome);

free(s);
```

### Array dinamico di struct

```c
int n = 5;
Studente *classe = malloc(n * sizeof(Studente));

for (int i = 0; i < n; i++) {
    classe[i].matricola = 1000 + i;   /* con l'array, torna comodo il punto */
}

free(classe);
```

Nota: `classe` è un puntatore, ma `classe[i]` funziona esattamente come con un array normale — è la stessa equivalenza puntatore/array che hai già visto (`v[i]` è `*(v + i)`), solo che qui la memoria puntata è stata allocata a runtime invece che dichiarata a compile-time.

---

## Blocco 5: Un ADT con puntatore opaco

Fin qui hai sempre visto lo `struct` intero: chiunque usi `Studente` vede e può toccare direttamente tutti i suoi campi. C'è un modo per **nascondere** la struttura interna e costringere chi usa il tuo codice a passare solo attraverso funzioni che tu hai scritto. Questa tecnica si chiama **puntatore opaco**, ed è il modo in cui il C simula quello che in un linguaggio OOP si chiama *incapsulamento* — `private` non esiste in C, ma il risultato che ottieni è lo stesso.

L'idea: separi l'**interfaccia** (cosa si può fare) dall'**implementazione** (come è fatta davvero la struttura), mettendole in due file diversi.

### Il file header — `stack.h` (l'interfaccia pubblica)

```c
#ifndef STACK_H
#define STACK_H

typedef struct Stack Stack;   /* tipo dichiarato, ma la sua struttura NON è visibile qui */

Stack *stack_create(int capacita);
void   stack_push(Stack *s, int valore);
int    stack_pop(Stack *s);
int    stack_is_empty(Stack *s);
void   stack_destroy(Stack *s);

#endif
```
Chi include `stack.h` sa che esiste un tipo `Stack` e sa quali funzioni può chiamare — ma non ha idea di come sia fatto internamente. Non può scrivere `s.dati[0] = 5;` perché non sa nemmeno che campo si chiami `dati`, né se esista.

### Il file di implementazione — `stack.c` (i dettagli nascosti)

```c
#include <stdlib.h>
#include "stack.h"

struct Stack {              /* la struttura reale vive SOLO qui */
    int *dati;
    int capacita;
    int top;
};

Stack *stack_create(int capacita) {
    Stack *s = malloc(sizeof(Stack));
    s->dati = malloc(capacita * sizeof(int));
    s->capacita = capacita;
    s->top = -1;
    return s;
}

void stack_push(Stack *s, int valore) {
    if (s->top < s->capacita - 1) {
        s->top++;
        s->dati[s->top] = valore;
    }
}

int stack_pop(Stack *s) {
    int valore = s->dati[s->top];
    s->top--;
    return valore;
}

int stack_is_empty(Stack *s) {
    return s->top == -1;
}

void stack_destroy(Stack *s) {
    free(s->dati);   /* prima libera la memoria interna... */
    free(s);         /* ...poi la struct stessa */
}
```

### Come lo usa chi scrive il main

```c
#include <stdio.h>
#include "stack.h"

int main() {
    Stack *s = stack_create(10);

    stack_push(s, 1);
    stack_push(s, 2);
    stack_push(s, 3);

    while (!stack_is_empty(s)) {
        printf("%d\n", stack_pop(s));   /* stampa 3, 2, 1 */
    }

    stack_destroy(s);
    return 0;
}
```

Chi scrive `main` non sa, e non deve sapere, che dentro `Stack` c'è un array e due interi. Sa solo che esiste un `create`, un `push`, un `pop`, un `destroy`. Se domani decidi di cambiare l'implementazione interna (es. usare una lista collegata invece di un array), il codice di `main` non deve cambiare di una virgola: è esattamente il vantaggio dell'incapsulamento.


---

## Schema mentale

```
Non conosco la dimensione a compile-time?
    → malloc(n * sizeof(tipo))
    → controlla sempre se restituisce NULL

Ho finito di usare la memoria?
    → free(p)
    → poi p = NULL, per evitare dangling pointer e double free

Devo ridimensionare un blocco già allocato?
    → usa realloc, MAI sovrascrivendo direttamente il puntatore originale

Alloco una struct dinamicamente?
    → Studente *s = malloc(sizeof(Studente));
    → accedi con s->campo

Voglio nascondere la struttura interna di un tipo?
    → typedef struct NomeStruct NomeStruct; nell'header (.h)
    → la vera struct { ... } sta solo nel .c
    → chi usa il tipo passa solo attraverso le funzioni che esponi
```

---

## Esercizi

### Esercizio 1: Array dinamico

Scrivi un programma che chiede all'utente quanti numeri vuole inserire, alloca dinamicamente un array di quella dimensione, legge i numeri, ne stampa la somma e la media, poi libera la memoria.

::: {.callout-tip collapse="true"}
## Soluzione
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int n;
    printf("Quanti numeri? ");
    scanf("%d", &n);

    int *v = malloc(n * sizeof(int));
    if (v == NULL) return 1;

    int somma = 0;
    for (int i = 0; i < n; i++) {
        scanf("%d", &v[i]);
        somma += v[i];
    }

    printf("Somma: %d\n", somma);
    printf("Media: %.2f\n", (double) somma / n);

    free(v);
    return 0;
}
```
:::

### Esercizio 2: Ridimensionare con realloc

Parti da un array dinamico di 5 interi, riempilo, poi usa `realloc` per portarlo a 10 elementi e riempi anche i nuovi 5. Stampa tutto l'array alla fine.

::: {.callout-tip collapse="true"}
## Soluzione
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *v = malloc(5 * sizeof(int));
    for (int i = 0; i < 5; i++) v[i] = i;

    int *tmp = realloc(v, 10 * sizeof(int));
    if (tmp == NULL) { free(v); return 1; }
    v = tmp;

    for (int i = 5; i < 10; i++) v[i] = i;

    for (int i = 0; i < 10; i++) printf("%d\n", v[i]);

    free(v);
    return 0;
}
```
:::

### Esercizio 3: Struct dinamica

Alloca dinamicamente uno `Studente` (nome, matricola, media), riempilo tramite un puntatore, stampalo, poi liberalo.

::: {.callout-tip collapse="true"}
## Soluzione
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char nome[50];
    int matricola;
    double media;
} Studente;

int main() {
    Studente *s = malloc(sizeof(Studente));
    if (s == NULL) return 1;

    strcpy(s->nome, "Mario");
    s->matricola = 12345;
    s->media = 8.5;

    printf("%s - %d - %.2f\n", s->nome, s->matricola, s->media);

    free(s);
    return 0;
}
```
:::

### Esercizio 4: Array dinamico di struct

Alloca dinamicamente un array di `n` `Studente` (chiesto all'utente), riempi solo il campo `matricola` con valori progressivi, stampa l'array, poi liberalo.

::: {.callout-tip collapse="true"}
## Soluzione
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    char nome[50];
    int matricola;
    double media;
} Studente;

int main() {
    int n;
    printf("Quanti studenti? ");
    scanf("%d", &n);

    Studente *classe = malloc(n * sizeof(Studente));
    if (classe == NULL) return 1;

    for (int i = 0; i < n; i++) {
        classe[i].matricola = 1000 + i;
    }

    for (int i = 0; i < n; i++) {
        printf("%d\n", classe[i].matricola);
    }

    free(classe);
    return 0;
}
```
:::

### Esercizio 5: Progetta un ADT

Usando lo schema di `stack.h` / `stack.c` visto sopra, progetta un ADT `Coda` (queue) con puntatore opaco, con le funzioni `coda_create`, `coda_enqueue`, `coda_dequeue`, `coda_is_empty`, `coda_destroy`. Non serve gestire il caso di coda piena in modo sofisticato — puoi assumere una capacità fissa decisa alla creazione, come nello Stack.

::: {.callout-tip collapse="true"}
## Soluzione — coda.h
```c
#ifndef CODA_H
#define CODA_H

typedef struct Coda Coda;

Coda *coda_create(int capacita);
void  coda_enqueue(Coda *c, int valore);
int   coda_dequeue(Coda *c);
int   coda_is_empty(Coda *c);
void  coda_destroy(Coda *c);

#endif
```

## Soluzione — coda.c
```c
#include <stdlib.h>
#include "coda.h"

struct Coda {
    int *dati;
    int capacita;
    int inizio;
    int fine;
    int conteggio;
};

Coda *coda_create(int capacita) {
    Coda *c = malloc(sizeof(Coda));
    c->dati = malloc(capacita * sizeof(int));
    c->capacita = capacita;
    c->inizio = 0;
    c->fine = 0;
    c->conteggio = 0;
    return c;
}

void coda_enqueue(Coda *c, int valore) {
    if (c->conteggio < c->capacita) {
        c->dati[c->fine] = valore;
        c->fine = (c->fine + 1) % c->capacita;
        c->conteggio++;
    }
}

int coda_dequeue(Coda *c) {
    int valore = c->dati[c->inizio];
    c->inizio = (c->inizio + 1) % c->capacita;
    c->conteggio--;
    return valore;
}

int coda_is_empty(Coda *c) {
    return c->conteggio == 0;
}

void coda_destroy(Coda *c) {
    free(c->dati);
    free(c);
}
```
:::