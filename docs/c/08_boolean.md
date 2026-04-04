# Il tipo booleano

---

## Vero e falso in C

In C non esiste un tipo booleano nativo come in altri linguaggi. Il C usa i numeri interi per rappresentare vero e falso:

- **0** significa **falso**
- **qualsiasi valore diverso da 0** significa **vero** (tipicamente si usa 1)

Questo è il motivo per cui le condizioni degli `if` e dei `while` sono espressioni numeriche:

```c
if (5 > 3) { ... }     /* 5 > 3 vale 1 — vero */
if (0) { ... }          /* 0 — falso — il blocco non viene mai eseguito */
if (42) { ... }         /* 42 — vero — il blocco viene sempre eseguito */
```

E questo è il motivo per cui le funzioni che restituiscono vero/falso usano `int`:

```c
int isPari(int n) {
    return n % 2 == 0;   /* restituisce 1 (vero) o 0 (falso) */
}
```

---

## La libreria stdbool.h

Dal C99 in poi esiste la libreria `stdbool.h` che introduce il tipo `bool` e le costanti `true` e `false`. Rende il codice più leggibile senza cambiare nulla nel funzionamento.

```c
#include <stdbool.h>
```

Con questa libreria puoi scrivere:

```c
bool trovato = false;
bool finito = true;
```

invece di:

```c
int trovato = 0;
int finito = 1;
```

---

## bool, true e false

| Con `stdbool.h` | Equivale a |
|---|---|
| `bool` | `int` |
| `true` | `1` |
| `false` | `0` |

Sono solo nomi più leggibili — sotto al cofano sono ancora interi.

```c
#include <stdio.h>
#include <stdbool.h>

bool isPari(int n) {
    return n % 2 == 0;
}

bool isPrimo(int n) {
    int i;
    if (n < 2) return false;
    for (i = 2; i * i <= n; i++) {
        if (n % i == 0) return false;
    }
    return true;
}

int main() {
    int n;
    printf("Inserisci un numero: ");
    scanf("%d", &n);

    if (isPari(n)) {
        printf("%d è pari.\n", n);
    } else {
        printf("%d è dispari.\n", n);
    }

    if (isPrimo(n)) {
        printf("%d è primo.\n", n);
    } else {
        printf("%d non è primo.\n", n);
    }

    return 0;
}
```

---

## Usare bool come flag

Un uso molto comune di `bool` è il **flag** — una variabile che tiene traccia di una condizione che può cambiare durante il programma.

```c
#include <stdio.h>
#include <stdbool.h>

int main() {
    int numeri[] = {3, 7, 12, 5, 8, 21, 4};
    int dimensione = 7;
    int cercato = 12;
    bool trovato = false;
    int posizione = -1;
    int i;

    for (i = 0; i < dimensione; i++) {
        if (numeri[i] == cercato) {
            trovato = true;
            posizione = i;
            break;
        }
    }

    if (trovato) {
        printf("%d trovato alla posizione %d.\n", cercato, posizione);
    } else {
        printf("%d non trovato.\n", cercato);
    }

    return 0;
}
```

Il flag `trovato` parte come `false`. Appena il numero viene trovato, diventa `true` e il ciclo si interrompe. Alla fine del ciclo, basta controllare il flag per sapere se la ricerca ha avuto successo.

---

## bool nelle funzioni

Le funzioni che rispondono a una domanda sì/no si scrivono naturalmente con `bool`:

```c
bool isVocale(char c) {
    c = tolower(c);   /* converte in minuscolo — da ctype.h */
    return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u';
}

bool isAlfa(char c) {
    return (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z');
}

bool isUguali(int a[], int b[], int n) {
    int i;
    for (i = 0; i < n; i++) {
        if (a[i] != b[i]) return false;
    }
    return true;
}
```

::: {.callout-note}
## Convenzione sui nomi
Le funzioni che restituiscono `bool` di solito iniziano con `is` o `ha` — per comunicare chiaramente che rispondono a una domanda:

- `isPari`, `isPrimo`, `isVocale` — "è...?"
- `isAlfa`, `isDigit` — "è...?"
- `haSpazi`, `haVocali` — "ha...?"

Questa convenzione rende il codice molto più leggibile:
```c
if (isPrimo(n)) { ... }   /* si legge quasi come italiano */
```
:::

---

## bool o int?

In pratica, molti programmi C usano ancora `int` per i valori booleani — soprattutto nel codice più vecchio. Entrambi gli approcci funzionano.

| | `int` | `bool` |
|---|---|---|
| **Disponibilità** | sempre | richiede `#include <stdbool.h>` |
| **Leggibilità** | meno chiaro | più chiaro |
| **Compatibilità** | tutti i compilatori | C99 o successivo |

::: {.callout-tip}
## Quale scegliere?
Usa `bool` quando vuoi rendere chiaro che una variabile o una funzione rappresenta un valore vero/falso. Usa `int` quando il risultato è usato anche come numero (conteggio, indice, ecc.).

In ogni caso, ricorda che sono la stessa cosa — `bool` è solo un nome più leggibile per `int`.
:::

---

## Schema mentale

```
La mia funzione risponde a una domanda sì/no?
    → usa bool come tipo di ritorno
    → chiamala con is... o ha...
    → return true / return false

Ho una variabile che tiene traccia di una condizione?
    → usala come flag: bool trovato = false
    → quando la condizione si verifica: trovato = true
    → alla fine controlla il flag con if (trovato)

Devo includere stdbool.h?
    → sì, se usi bool, true o false
    → no, se usi int, 1 e 0
```

---

## Esercizio

Scrivi le seguenti funzioni usando `bool`:

- `bool isLettera(char c)` — true se `c` è una lettera dell'alfabeto
- `bool isMaiuscola(char c)` — true se `c` è una lettera maiuscola
- `bool isMinuscola(char c)` — true se `c` è una lettera minuscola
- `bool isCifra(char c)` — true se `c` è una cifra da '0' a '9'

Scrivi un `main` che legge un carattere e stampa tutte le informazioni su di esso:
```
Carattere: A
È una lettera?   sì
È maiuscola?     sì
È minuscola?     no
È una cifra?     no
```