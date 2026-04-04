# Variabili e tipi

---

## Cos'è una variabile

Un programma ha bisogno di ricordare dei dati mentre lavora: il nome di un utente, il risultato di un calcolo, l'età di una persona. Per farlo usa le **variabili**.

Una variabile è una zona di memoria con un nome. Quando dichiari una variabile, stai dicendo al computer: *"tieni da parte un po' di memoria e chiamala con questo nome"*.

```c
int eta;
```

Questa riga dichiara una variabile chiamata `eta` che può contenere un numero intero. Per ora è vuota — non ha ancora un valore.

---

## Dichiarazione e assegnazione

In C devi sempre **dichiarare** una variabile prima di usarla. La dichiarazione dice due cose: il tipo e il nome.

```c
int eta;           /* dichiarazione */
eta = 17;          /* assegnazione */
```

Puoi anche fare le due cose insieme:

```c
int eta = 17;      /* dichiarazione + inizializzazione */
```

::: {.callout-important}
## Non usare una variabile senza inizializzarla
In C, una variabile dichiarata ma non inizializzata contiene un valore casuale — quello che si trovava in quella zona di memoria prima. Usarla porta a risultati imprevedibili.

```c
int x;
printf("%d\n", x);  /* stampa un valore casuale — comportamento non definito */
```

Inizializza sempre le variabili prima di usarle.
:::

---

## I tipi di dato

Il tipo di una variabile determina:
- che **tipo di valore** può contenere (numero intero, numero decimale, carattere...)
- quanta **memoria** occupa

I tipi fondamentali del C sono quattro.

### int: numeri interi

```c
int punteggio = 1500;
int livello = 3;
int temperatura = -5;
```

Contiene numeri interi, positivi o negativi. Occupa tipicamente 4 byte.

### float e double: numeri decimali

```c
float media = 7.5;
double pigreco = 3.14159265358979;
```

`float` è a precisione singola (4 byte), `double` a precisione doppia (8 byte). Nella pratica si usa quasi sempre `double` perché è più preciso.

### char: un singolo carattere

```c
char iniziale = 'A';
char risposta = 's';
```

Contiene un singolo carattere. Si scrive sempre tra **apici singoli** `' '` — non tra virgolette. Occupa 1 byte.

::: {.callout-note}
## char e numeri
In C i caratteri sono in realtà numeri — ogni carattere ha un codice numerico secondo la tabella ASCII. `'A'` corrisponde al numero 65, `'a'` al numero 97, `'0'` al numero 48.

Questo significa che puoi fare operazioni aritmetiche sui caratteri:
```c
char c = 'A';
printf("%c\n", c + 1);  /* stampa 'B' */
```
Questa proprietà è fondamentale quando lavori con le stringhe.
:::

---

## Tabella riassuntiva

| Tipo | Cosa contiene | Esempio |
|---|---|---|
| `int` | numero intero | `42`, `-7`, `0` |
| `float` | numero decimale (precisione singola) | `3.14` |
| `double` | numero decimale (precisione doppia) | `3.14159265` |
| `char` | singolo carattere | `'A'`, `'z'`, `'3'` |

---

## Le costanti

A volte hai un valore che non deve mai cambiare — per esempio il numero di giorni in una settimana o il valore del pi greco. In quel caso usi una **costante**.

```c
#define PI 3.14159
#define GIORNI_SETTIMANA 7
#define DIMENSIONE 100
```

`#define` non è una variabile — è una sostituzione: ogni volta che il compilatore trova `PI` nel codice, lo sostituisce con `3.14159`. Per convenzione i nomi delle costanti si scrivono in maiuscolo.

---

## I nomi delle variabili

Puoi chiamare le variabili come vuoi, ma ci sono delle regole:

**Regole obbligatorie:**
- può contenere lettere, cifre e underscore `_`
- non può iniziare con una cifra
- non può essere una parola riservata del C (`int`, `if`, `while`...)

**Convenzioni (buone abitudini):**
- usa nomi significativi: `eta` è meglio di `x`, `punteggioGiocatore` è meglio di `pg`
- usa il camelCase per nomi composti: `punteggioTotale`, `nomeUtente`
- evita nomi di una sola lettera (tranne `i`, `j`, `k` nei cicli)

```c
int eta = 17;              /* buono */
int e = 17;                /* scarso — cosa vuol dire e? */
int punteggio_totale = 0;  /* accettabile */
int punteggioTotale = 0;   /* preferibile */
int 2giocatori = 0;        /* ERRORE — inizia con un numero */
int int = 0;               /* ERRORE — int è una parola riservata */
```

---

## Stampare le variabili con printf

Per stampare una variabile con `printf` devi usare un **segnaposto** — un codice che indica il tipo della variabile.

| Tipo | Segnaposto |
|---|---|
| `int` | `%d` |
| `float` | `%f` |
| `double` | `%lf` oppure `%f` |
| `char` | `%c` |
| stringa | `%s` |

```c
int eta = 17;
double media = 8.5;
char voto = 'A';

printf("Età: %d\n", eta);
printf("Media: %.2f\n", media);   /* .2 = 2 cifre decimali */
printf("Voto: %c\n", voto);
```

Output:
```
Età: 17
Media: 8.50
Voto: A
```

Puoi stampare più variabili nella stessa `printf`:

```c
printf("Età: %d, media: %.1f\n", eta, media);
```

I segnaposto vengono sostituiti nell'ordine in cui compaiono — il primo `%d` con il primo valore, il secondo con il secondo, e così via.

---

## Leggere le variabili con scanf

Per leggere un valore inserito dall'utente si usa `scanf`.

```c
int eta;
printf("Inserisci la tua età: ");
scanf("%d", &eta);
printf("Hai %d anni.\n", eta);
```

La `&` davanti al nome della variabile è obbligatoria — dice a `scanf` dove in memoria salvare il valore letto. Per ora ricordala come una regola: con `scanf` si mette sempre `&`.

```c
int n;
double x;
char c;

scanf("%d", &n);    /* legge un intero */
scanf("%lf", &x);   /* legge un double */
scanf(" %c", &c);   /* legge un carattere — nota lo spazio prima di %c */
```

::: {.callout-warning}
## Lo spazio prima di `%c`
Quando leggi un carattere con `scanf`, metti sempre uno spazio prima di `%c`:
```c
scanf(" %c", &c);
```
Senza lo spazio, `scanf` potrebbe leggere il carattere di invio rimasto nel buffer dalla lettura precedente invece del carattere che vuoi leggere.
:::

---

## Esempio completo

```c
#include <stdio.h>

int main() {
    /* Dichiarazione delle variabili */
    int eta;
    double altezza;
    char iniziale;

    /* Lettura dei dati */
    printf("Inserisci la tua età: ");
    scanf("%d", &eta);

    printf("Inserisci la tua altezza in metri (es. 1.75): ");
    scanf("%lf", &altezza);

    printf("Inserisci l'iniziale del tuo nome: ");
    scanf(" %c", &iniziale);

    /* Stampa dei risultati */
    printf("\n--- RIEPILOGO ---\n");
    printf("Iniziale: %c\n", iniziale);
    printf("Età:      %d anni\n", eta);
    printf("Altezza:  %.2f m\n", altezza);

    return 0;
}
```

---

## Schema mentale

Quando dichiari una variabile, chiediti sempre tre cose:

```
1. Che tipo di dato devo memorizzare?
   → numero intero?    usa int
   → numero decimale?  usa double
   → singolo carattere? usa char

2. Che nome le do?
   → significativo, che descriva il contenuto

3. Ha già un valore oppure lo leggo dall'utente?
   → ce l'ha già → inizializzala subito
   → lo leggo    → dichiara, poi usa scanf
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che:
- dichiara una variabile `int` per l'anno di nascita e una `double` per la media scolastica
- legge entrambi i valori dall'utente
- stampa: `"Sei nato nel ANNO e hai una media di MEDIA"`

**Esercizio 2**
Scrivi un programma che legge due numeri interi e stampa la loro somma, differenza, prodotto e quoziente.

**Esercizio 3**
Scrivi un programma che legge un carattere e stampa il suo valore numerico ASCII.
```
Esempio: inserisci 'A' → stampa 65
```
*Suggerimento: puoi stampare un `char` sia con `%c` (come carattere) sia con `%d` (come numero).*