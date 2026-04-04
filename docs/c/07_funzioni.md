# Funzioni

---

## Perché le funzioni

Immagina di dover calcolare il massimo tra due numeri in dieci punti diversi del tuo programma. Potresti scrivere lo stesso `if` dieci volte — ma se ti accorgi di un errore, devi correggerlo in dieci posti. E se dimentichi uno?

Le **funzioni** risolvono questo problema: scrivi il codice una volta sola, gli dai un nome, e lo richiami ogni volta che ti serve.

Una funzione è un blocco di codice che:
- ha un **nome**
- riceve dei dati in ingresso (i **parametri**)
- svolge un compito preciso
- può restituire un risultato

---

## Il ruolo del main

Prima di vedere come si scrivono le funzioni, è utile chiarire il ruolo del `main`.

Il `main` serve per tre cose:
1. leggere i dati
2. chiamare le funzioni
3. stampare i risultati

Le funzioni invece si occupano di **calcolare** e **trasformare**. Tenere separati questi ruoli rende il codice più chiaro e più facile da correggere.

::: {.callout-note}
## Printf dentro una funzione
In generale è meglio evitare `printf` dentro una funzione. Una funzione dovrebbe fare un solo lavoro: calcolare o trasformare un dato. La stampa è responsabilità del `main`.

```c
/* Meglio così */
int massimo(int a, int b) {
    if (a > b) return a;
    return b;
}

int main() {
    int m = massimo(10, 7);
    printf("Il massimo è %d\n", m);   /* stampa nel main */
    return 0;
}
```
:::

---

## Struttura di una funzione

```c
tipo_ritorno nome_funzione(tipo parametro1, tipo parametro2, ...) {
    /* corpo della funzione */
    return valore;   /* se il tipo di ritorno non è void */
}
```

Tre decisioni da prendere ogni volta che scrivi una funzione:

**1. Cosa deve restituire?**
- un numero (`int`, `double`) — usa `return valore`
- niente (modifica una variabile, stampa qualcosa) — usa `void`

**2. Cosa riceve in ingresso?**
- i parametri — i dati su cui lavora

**3. Cosa fa?**
- il corpo — le istruzioni

---

## Funzioni che restituiscono un valore

Usa `int` quando la funzione deve restituire un conteggio, una posizione, un valore numerico, oppure 1 o 0 (vero/falso).

```c
/* Restituisce il massimo tra due interi */
int massimo(int a, int b) {
    if (a > b) {
        return a;
    }
    return b;
}

/* Restituisce 1 se n è pari, 0 altrimenti */
int isPari(int n) {
    return n % 2 == 0;
}

/* Restituisce la media di due valori */
double media(double a, double b) {
    return (a + b) / 2.0;
}
```

Come si usano nel `main`:

```c
int main() {
    int x = 10, y = 7;
    printf("Massimo: %d\n", massimo(x, y));   /* 10 */

    if (isPari(x)) {
        printf("%d è pari\n", x);
    }

    printf("Media: %.2f\n", media(3.5, 7.5));  /* 5.50 */

    return 0;
}
```

---

## Funzioni void

Usa `void` quando la funzione non restituisce un numero ma produce un risultato in altro modo — ad esempio modifica una stringa passata come parametro, o produce output.

```c
/* Stampa una riga di separazione */
void stampaSeparatore(int lunghezza) {
    int i;
    for (i = 0; i < lunghezza; i++) {
        printf("-");
    }
    printf("\n");
}

/* Stampa le statistiche (esempio di quando printf in funzione ha senso) */
void stampaStatistiche(int minimo, int massimo, double media) {
    printf("Minimo:  %d\n", minimo);
    printf("Massimo: %d\n", massimo);
    printf("Media:   %.2f\n", media);
}
```

Le funzioni `void` non hanno `return` (o hanno `return;` senza valore per uscire prima).

---

## Il prototipo

In C il compilatore legge il codice dall'alto verso il basso. Se chiami una funzione prima di averla definita, il compilatore non la conosce ancora e dà errore.

La soluzione è il **prototipo**: una dichiarazione della funzione che va messa in cima al file, prima del `main`. Dice al compilatore "questa funzione esiste, la trovi più avanti".

```c
#include <stdio.h>

/* Prototipi — dichiarazioni anticipate */
int massimo(int a, int b);
int isPari(int n);
void stampaSeparatore(int lunghezza);

int main() {
    /* Qui puoi chiamare le funzioni anche se sono definite dopo */
    printf("Massimo: %d\n", massimo(10, 7));
    stampaSeparatore(20);
    return 0;
}

/* Definizioni complete delle funzioni */
int massimo(int a, int b) {
    if (a > b) return a;
    return b;
}

int isPari(int n) {
    return n % 2 == 0;
}

void stampaSeparatore(int lunghezza) {
    int i;
    for (i = 0; i < lunghezza; i++) printf("-");
    printf("\n");
}
```

::: {.callout-tip}
## Struttura consigliata di un file C
```
1. #include delle librerie
2. #define delle costanti
3. Prototipi delle funzioni
4. main()
5. Definizioni delle funzioni
```

Seguire sempre questo ordine rende il codice prevedibile e facile da navigare.
:::

---

## Parametri: passaggio per valore

In C i parametri vengono passati **per valore**: la funzione riceve una **copia** del valore, non la variabile originale. Modificare un parametro dentro la funzione non cambia la variabile del chiamante.

```c
void raddoppia(int x) {
    x = x * 2;   /* modifica solo la copia locale */
}

int main() {
    int n = 5;
    raddoppia(n);
    printf("%d\n", n);   /* stampa 5 — n non è cambiato */
    return 0;
}
```

Se vuoi che la funzione modifichi la variabile originale, devi usare i **puntatori** — che vedremo nella prossima lezione.

---

## Variabili locali e scope

Le variabili dichiarate dentro una funzione si chiamano **variabili locali**. Esistono solo dentro quella funzione — non sono visibili dall'esterno.

```c
int somma(int a, int b) {
    int risultato = a + b;   /* risultato è locale a somma */
    return risultato;
}

int main() {
    int s = somma(3, 4);
    /* printf("%d\n", risultato);  ERRORE — risultato non esiste qui */
    printf("%d\n", s);   /* 7 */
    return 0;
}
```

Ogni funzione ha il proprio **scope** — il proprio spazio di variabili, separato da tutte le altre funzioni. Questo è uno dei vantaggi delle funzioni: non si intralciano a vicenda.

---

## Strategia per scrivere una funzione

Quando vedi un esercizio che richiede una funzione, procedi così:

```
1. Capisci cosa deve produrre
   → un numero?      → int o double
   → niente?         → void

2. Capisci cosa riceve
   → quali dati le servono per lavorare?
   → quanti parametri e di che tipo?

3. Scrivi lo scheletro
   tipo nome(parametri) {
       ...
       return ...;
   }

4. Scrivi il corpo
   → il ciclo, la condizione, il calcolo

5. Scrivi il prototipo sopra il main

6. Chiamala nel main
```

---

## Esempio completo

```c
#include <stdio.h>

/* Prototipi */
int massimo(int a, int b);
int minimo(int a, int b);
double media3(int a, int b, int c);
int isPari(int n);
void stampaRisultati(int a, int b, int c);

int main() {
    int a, b, c;

    printf("Inserisci tre numeri interi: ");
    scanf("%d %d %d", &a, &b, &c);

    stampaRisultati(a, b, c);

    return 0;
}

int massimo(int a, int b) {
    return (a > b) ? a : b;
}

int minimo(int a, int b) {
    return (a < b) ? a : b;
}

double media3(int a, int b, int c) {
    return (a + b + c) / 3.0;
}

int isPari(int n) {
    return n % 2 == 0;
}

void stampaRisultati(int a, int b, int c) {
    int max = massimo(massimo(a, b), c);
    int min = minimo(minimo(a, b), c);
    double m = media3(a, b, c);

    printf("\n--- RISULTATI ---\n");
    printf("Massimo: %d\n", max);
    printf("Minimo:  %d\n", min);
    printf("Media:   %.2f\n", m);
    printf("Pari:    ");
    if (isPari(a)) printf("%d ", a);
    if (isPari(b)) printf("%d ", b);
    if (isPari(c)) printf("%d ", c);
    printf("\n");
}
```

---

## Schema mentale

```
Cosa deve restituire la funzione?
    → un numero (conteggio, risultato, 1/0) → int o double + return
    → niente → void

Cosa riceve in ingresso?
    → una variabile → tipo nome
    → due variabili → tipo nome1, tipo nome2

Come la chiamo?
    → risultato = nomeFunzione(argomenti)   se restituisce qualcosa
    → nomeFunzione(argomenti)               se è void

Dove la metto nel file?
    → prototipo in cima
    → definizione dopo il main
    → chiamata nel main
```

---

## Esercizi

**Esercizio 1**
Scrivi una funzione `int potenza(int base, int esponente)` che calcola `base` elevato a `esponente` usando un ciclo (non usare `pow` della libreria).

**Esercizio 2**
Scrivi una funzione `int isPrimo(int n)` che restituisce 1 se `n` è primo, 0 altrimenti. Chiamala nel `main` per stampare tutti i numeri primi da 1 a 100.

**Esercizio 3**
Scrivi le funzioni:
- `int sommaDigiti(int n)` — restituisce la somma delle cifre di un numero
- `int inverti(int n)` — restituisce il numero con le cifre invertite

Esempio: `sommaDigiti(347)` → `14`; `inverti(347)` → `743`

**Esercizio 4**
Scrivi un programma con un menu che chiama funzioni diverse in base alla scelta dell'utente:
1. Calcola il massimo tra due numeri
2. Verifica se un numero è primo
3. Calcola la somma delle cifre
4. Esci