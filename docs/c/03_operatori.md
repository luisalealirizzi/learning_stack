# Operatori

---

## Cosa sono gli operatori

Un operatore è un simbolo che esegue un'operazione su uno o più valori. Li conosci già dalla matematica: `+`, `-`, `*`, `/`. In C ce ne sono altri, specifici della programmazione.

Gli operatori si dividono in tre famiglie:
- **aritmetici** — fanno calcoli
- **relazionali** — confrontano due valori
- **logici** — combinano condizioni

---

## Operatori aritmetici

| Operatore | Significato | Esempio | Risultato |
|---|---|---|---|
| `+` | addizione | `5 + 3` | `8` |
| `-` | sottrazione | `5 - 3` | `2` |
| `*` | moltiplicazione | `5 * 3` | `15` |
| `/` | divisione | `5 / 2` | `2` |
| `%` | modulo (resto) | `5 % 2` | `1` |

```c
int a = 10;
int b = 3;

printf("%d\n", a + b);   /* 13 */
printf("%d\n", a - b);   /* 7  */
printf("%d\n", a * b);   /* 30 */
printf("%d\n", a / b);   /* 3  — divisione intera! */
printf("%d\n", a % b);   /* 1  — resto della divisione */
```

::: {.callout-important}
## La divisione intera
Quando dividi due `int`, il risultato è un `int` — la parte decimale viene troncata, non arrotondata.

```c
int a = 5;
int b = 2;
printf("%d\n", a / b);   /* stampa 2, non 2.5 */
```

Se vuoi il risultato decimale, almeno uno dei due deve essere `double`:

```c
double risultato = 5.0 / 2;    /* 2.5 */
double risultato = (double)5 / 2;  /* 2.5 — casting */
```
:::

### Il modulo `%`

Il modulo restituisce il **resto** della divisione intera. È uno degli operatori più utili in programmazione.

```c
10 % 3   /* 1  — perché 10 = 3*3 + 1 */
12 % 4   /* 0  — perché 12 = 4*3 + 0 */
7  % 2   /* 1  — perché 7  = 2*3 + 1 */
```

A cosa serve il modulo? A molte cose:

```c
/* Verificare se un numero è pari o dispari */
if (n % 2 == 0)   /* n è pari  */
if (n % 2 != 0)   /* n è dispari */

/* Estrarre l'ultima cifra di un numero */
int ultima = n % 10;   /* es: 347 % 10 = 7 */

/* Tenere un indice dentro un range */
int i = posizione % dimensione;
```

---

## Operatori di assegnazione

L'operatore di assegnazione base è `=`. Ma ci sono anche forme abbreviate molto usate:

| Operatore | Equivale a | Esempio |
|---|---|---|
| `=` | assegna | `x = 5` |
| `+=` | `x = x + n` | `x += 3` |
| `-=` | `x = x - n` | `x -= 3` |
| `*=` | `x = x * n` | `x *= 2` |
| `/=` | `x = x / n` | `x /= 2` |
| `%=` | `x = x % n` | `x %= 2` |

```c
int punteggio = 100;
punteggio += 50;    /* punteggio diventa 150 */
punteggio -= 20;    /* punteggio diventa 130 */
punteggio *= 2;     /* punteggio diventa 260 */
```

### Incremento e decremento

Due operatori molto usati nei cicli:

```c
int i = 0;
i++;    /* equivale a i = i + 1 */
i--;    /* equivale a i = i - 1 */
```

```c
/* Questi quattro sono tutti equivalenti */
i = i + 1;
i += 1;
i++;
++i;
```

::: {.callout-note}
## `i++` vs `++i`
La differenza emerge solo quando li usi dentro un'espressione:

```c
int i = 5;
int a = i++;   /* a = 5, poi i diventa 6 — prima assegna, poi incrementa */
int b = ++i;   /* i diventa 7, poi b = 7 — prima incrementa, poi assegna */
```

Nei cicli, dove li usi da soli, non fa differenza. Usa la forma che preferisci — `i++` è più comune.
:::

---

## Operatori di confronto

Gli operatori relazionali **confrontano** due valori e restituiscono `1` (vero) oppure `0` (falso).

| Operatore | Significato | Esempio | Risultato |
|---|---|---|---|
| `==` | uguale a | `5 == 5` | `1` (vero) |
| `!=` | diverso da | `5 != 3` | `1` (vero) |
| `<` | minore di | `3 < 5` | `1` (vero) |
| `>` | maggiore di | `3 > 5` | `0` (falso) |
| `<=` | minore o uguale | `5 <= 5` | `1` (vero) |
| `>=` | maggiore o uguale | `3 >= 5` | `0` (falso) |

```c
int a = 10;
int b = 20;

printf("%d\n", a == b);   /* 0 — falso  */
printf("%d\n", a != b);   /* 1 — vero   */
printf("%d\n", a < b);    /* 1 — vero   */
printf("%d\n", a > b);    /* 0 — falso  */
```

::: {.callout-warning}
## `=` e `==` sono cose completamente diverse
`=` è l'assegnazione — modifica il valore di una variabile.
`==` è il confronto — verifica se due valori sono uguali.

Questo è uno degli errori più comuni in C:

```c
if (x = 5)    /* SBAGLIATO — assegna 5 a x, poi valuta x */
if (x == 5)   /* CORRETTO  — confronta x con 5 */
```

Il primo non dà errore di compilazione ma si comporta in modo inatteso — stai assegnando, non confrontando.
:::

---

## Operatori logici

Gli operatori logici combinano più condizioni.

| Operatore | Significato | Esempio |
|---|---|---|
| `&&` | AND — entrambe vere | `a > 0 && a < 10` |
| `\|\|` | OR — almeno una vera | `a < 0 \|\| a > 100` |
| `!` | NOT — nega la condizione | `!(a == 0)` |

```c
int voto = 7;

/* AND — il voto deve essere tra 6 e 10 compresi */
if (voto >= 6 && voto <= 10) {
    printf("Promosso\n");
}

/* OR — il voto è insufficiente o fuori scala */
if (voto < 6 || voto > 10) {
    printf("Valore non valido\n");
}

/* NOT — il voto NON è sufficiente */
if (!(voto >= 6)) {
    printf("Insufficiente\n");
}
```

### Tabella della verità

| `a` | `b` | `a && b` | `a \|\| b` | `!a` |
|---|---|---|---|---|
| 0 (falso) | 0 (falso) | 0 | 0 | 1 |
| 0 (falso) | 1 (vero) | 0 | 1 | 1 |
| 1 (vero) | 0 (falso) | 0 | 1 | 0 |
| 1 (vero) | 1 (vero) | 1 | 1 | 0 |

---

## Precedenza degli operatori

Quando un'espressione contiene più operatori, il C segue un ordine preciso — come in matematica dove la moltiplicazione viene prima dell'addizione.

```
1. ! (NOT)
2. * / % (moltiplicazione, divisione, modulo)
3. + - (addizione, sottrazione)
4. < > <= >= (confronto)
5. == != (uguaglianza)
6. && (AND)
7. || (OR)
8. = += -= ... (assegnazione)
```

```c
int x = 2 + 3 * 4;      /* 14, non 20 — prima 3*4, poi +2 */
int y = (2 + 3) * 4;    /* 20 — le parentesi cambiano l'ordine */

int ok = a > 0 && b < 10;  /* prima valuta a>0 e b<10, poi && */
```

::: {.callout-tip}
## Usa le parentesi
Non devi memorizzare tutta la tabella di precedenza. Quando hai dubbi, usa le parentesi — rendono il codice più chiaro e prevengono errori.

```c
/* Questi due si leggono diversamente */
if (a == b && c == d || e == f)    /* difficile da leggere */
if ((a == b && c == d) || e == f)  /* molto più chiaro */
```
:::

---

## Il casting

A volte devi convertire un tipo in un altro. In C si usa il **casting** con le parentesi tonde:

```c
int a = 5;
int b = 2;

/* Senza casting: divisione intera */
double risultato1 = a / b;           /* 2.0 — sbagliato! */

/* Con casting: divisione decimale */
double risultato2 = (double)a / b;   /* 2.5 — corretto */
```

Il casting `(double)a` dice al compilatore: *"tratta `a` come se fosse un `double` per questa operazione"*. Il valore originale di `a` non cambia.

```c
/* Altri esempi di casting */
int x = (int)3.9;      /* x = 3 — la parte decimale viene troncata */
char c = (char)65;     /* c = 'A' — il numero 65 diventa il carattere A */
```

---

## Esempio completo

```c
#include <stdio.h>

int main() {
    int a, b;

    printf("Inserisci due numeri interi: ");
    scanf("%d %d", &a, &b);

    printf("\n--- OPERAZIONI ---\n");
    printf("Somma:       %d\n", a + b);
    printf("Differenza:  %d\n", a - b);
    printf("Prodotto:    %d\n", a * b);

    if (b != 0) {
        printf("Quoziente:   %d\n", a / b);
        printf("Resto:       %d\n", a % b);
        printf("Divisione:   %.2f\n", (double)a / b);
    } else {
        printf("Divisione impossibile: b è zero\n");
    }

    printf("\n--- CONFRONTI ---\n");
    printf("a == b?  %d\n", a == b);
    printf("a != b?  %d\n", a != b);
    printf("a < b?   %d\n", a < b);
    printf("a > b?   %d\n", a > b);

    printf("\n--- PARI O DISPARI ---\n");
    printf("a è %s\n", (a % 2 == 0) ? "pari" : "dispari");
    printf("b è %s\n", (b % 2 == 0) ? "pari" : "dispari");

    return 0;
}
```

---

## Schema mentale

```
Devo fare un calcolo?
    → usa + - * /
    → attento alla divisione intera tra int

Devo confrontare due valori?
    → usa == != < > <= >=
    → non confondere = (assegna) con == (confronta)

Devo combinare più condizioni?
    → entrambe vere      → &&
    → almeno una vera    → ||
    → negare             → !

Ho dubbi sulla precedenza?
    → usa le parentesi
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge un numero intero e stampa:
- se è pari o dispari
- se è positivo, negativo o zero
- le sue ultime due cifre (suggerimento: usa `%`)

**Esercizio 2**
Scrivi un programma che legge due numeri interi `a` e `b` e stampa quante volte `b` entra in `a` e quanto rimane (divisione con resto).
```
Esempio: a=17, b=5 → 17 = 5 * 3 + 2
```

**Esercizio 3**
Scrivi un programma che legge tre voti (interi da 0 a 10) e stampa la media con due decimali. Usa il casting per ottenere una divisione decimale.