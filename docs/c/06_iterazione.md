# Iterazione

---

## Ripetere le istruzioni

Immagina di dover stampare i numeri da 1 a 100. Scrivere cento `printf` non è una soluzione — è lungo, noioso e impossibile da modificare.

Le strutture di iterazione — chiamate anche **cicli** o **loop** — permettono di ripetere un blocco di istruzioni più volte. In C ce ne sono tre: `for`, `while` e `do-while`.

---

## for — quando sai quante volte ripetere

Il ciclo `for` è la scelta giusta quando sai esattamente quante volte vuoi ripetere qualcosa.

```c
for (inizializzazione; condizione; aggiornamento) {
    /* istruzioni da ripetere */
}
```

Le tre parti dentro le parentesi:
- **inizializzazione** — eseguita una volta sola all'inizio
- **condizione** — controllata prima di ogni ripetizione; se è falsa il ciclo si ferma
- **aggiornamento** — eseguito alla fine di ogni ripetizione

```c
int i;
for (i = 1; i <= 5; i++) {
    printf("%d\n", i);
}
```

Output:
```
1
2
3
4
5
```

Passo per passo:
1. `i = 1` — inizializzazione
2. `i <= 5`? Sì → esegui il corpo → `i++` → `i` diventa 2
3. `i <= 5`? Sì → esegui il corpo → `i++` → `i` diventa 3
4. ...
5. `i <= 5`? No (i è 6) → il ciclo si ferma

::: {.callout-note}
## La variabile contatore
La variabile `i` si chiama **contatore** — tiene traccia di quante volte il ciclo ha girato. Per convenzione si usa `i` (poi `j`, `k` per cicli annidati). Deve essere dichiarata prima del `for`.

```c
int i;
for (i = 0; i < 10; i++) { ... }   /* da 0 a 9 — 10 ripetizioni */
for (i = 1; i <= 10; i++) { ... }  /* da 1 a 10 — 10 ripetizioni */
for (i = 10; i >= 1; i--) { ... }  /* da 10 a 1 — conto alla rovescia */
```
:::

### Varianti comuni del for

```c
int i;

/* Conta da 0 a N-1 — molto usato con gli array */
for (i = 0; i < 10; i++) {
    printf("%d ", i);   /* 0 1 2 3 4 5 6 7 8 9 */
}

/* Conta solo i pari */
for (i = 0; i <= 20; i += 2) {
    printf("%d ", i);   /* 0 2 4 6 8 10 12 14 16 18 20 */
}

/* Conta alla rovescia */
for (i = 5; i >= 1; i--) {
    printf("%d ", i);   /* 5 4 3 2 1 */
}

/* Somma da 1 a N */
int somma = 0;
int n = 10;
for (i = 1; i <= n; i++) {
    somma += i;
}
printf("Somma: %d\n", somma);   /* 55 */
```

---

## while — quando non sai quante volte ripetere

Il ciclo `while` ripete un blocco finché una condizione è vera. È la scelta giusta quando non sai in anticipo quante volte ripetere.

```c
while (condizione) {
    /* istruzioni da ripetere */
}
```

La condizione viene controllata **prima** di ogni ripetizione. Se è subito falsa, il corpo non viene mai eseguito.

```c
int n;
printf("Inserisci un numero positivo: ");
scanf("%d", &n);

while (n <= 0) {
    printf("Il numero deve essere positivo. Riprova: ");
    scanf("%d", &n);
}

printf("Hai inserito: %d\n", n);
```

Questo ciclo continua a chiedere il numero finché l'utente non inserisce un valore valido. Non sai quante volte l'utente sbaglierà — per questo usi `while` e non `for`.

### while per leggere finché l'utente vuole

```c
int valore;
int somma = 0;
int quanti = 0;

printf("Inserisci numeri (0 per finire):\n");
scanf("%d", &valore);

while (valore != 0) {
    somma += valore;
    quanti++;
    scanf("%d", &valore);
}

if (quanti > 0) {
    printf("Somma: %d\n", somma);
    printf("Media: %.2f\n", (double)somma / quanti);
} else {
    printf("Nessun numero inserito.\n");
}
```

::: {.callout-important}
## Il ciclo infinito
Se la condizione del `while` non diventa mai falsa, il programma non si ferma mai. Questo si chiama **ciclo infinito** ed è quasi sempre un errore.

```c
int i = 1;
while (i <= 10) {
    printf("%d\n", i);
    /* manca i++ — i resta sempre 1 — ciclo infinito! */
}
```

Assicurati sempre che qualcosa nel corpo del ciclo faccia progredire verso la condizione di uscita.

Per fermare un programma bloccato in un ciclo infinito: `Ctrl+C` nel terminale.
:::

---

## do-while — eseguire almeno una volta

Il ciclo `do-while` è simile al `while`, ma con una differenza importante: la condizione viene controllata **dopo** il corpo. Questo significa che il corpo viene eseguito **almeno una volta**, anche se la condizione è subito falsa.

```c
do {
    /* istruzioni da ripetere */
} while (condizione);
```

È perfetto per i menu e per chiedere input all'utente almeno una volta:

```c
int scelta;

do {
    printf("\n=== MENU ===\n");
    printf("1. Nuova partita\n");
    printf("2. Carica partita\n");
    printf("3. Esci\n");
    printf("Scelta: ");
    scanf("%d", &scelta);

    if (scelta == 1) {
        printf("Avvio nuova partita...\n");
    } else if (scelta == 2) {
        printf("Caricamento...\n");
    } else if (scelta != 3) {
        printf("Scelta non valida.\n");
    }

} while (scelta != 3);

printf("Arrivederci!\n");
```

Il menu viene mostrato almeno una volta. Se l'utente sceglie 3, il ciclo si ferma; altrimenti il menu viene ripresentato.

---

## Confronto tra i tre cicli

| | `for` | `while` | `do-while` |
|---|---|---|---|
| **Quando usarlo** | Numero di ripetizioni noto | Numero di ripetizioni ignoto | Corpo eseguito almeno una volta |
| **Condizione** | controllata prima | controllata prima | controllata dopo |
| **Caso tipico** | scorrere array, contare | validazione input, leggere fino a sentinella | menu, input utente |

---

## break e continue

Due parole chiave che modificano il comportamento del ciclo.

### break — esci dal ciclo

`break` interrompe immediatamente il ciclo e salta al codice successivo.

```c
int i;
for (i = 1; i <= 10; i++) {
    if (i == 5) {
        break;   /* esce quando i vale 5 */
    }
    printf("%d ", i);
}
/* stampa: 1 2 3 4 */
```

### continue — salta alla prossima iterazione

`continue` salta il resto del corpo e passa alla prossima iterazione.

```c
int i;
for (i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
        continue;   /* salta i numeri pari */
    }
    printf("%d ", i);
}
/* stampa: 1 3 5 7 9 */
```

::: {.callout-tip}
## Usa break e continue con moderazione
`break` e `continue` sono utili ma rendono il flusso del programma più difficile da seguire se usati troppo. Prima di usarli, chiediti se puoi ottenere lo stesso risultato modificando la condizione del ciclo.
:::

---

## Cicli annidati

Puoi mettere un ciclo dentro un altro ciclo. Per ogni iterazione del ciclo esterno, il ciclo interno completa tutte le sue iterazioni.

```c
int i, j;
for (i = 1; i <= 3; i++) {
    for (j = 1; j <= 3; j++) {
        printf("%d*%d=%d  ", i, j, i*j);
    }
    printf("\n");
}
```

Output:
```
1*1=1  1*2=2  1*3=3
2*1=2  2*2=4  2*3=6
3*1=3  3*2=6  3*3=9
```

I cicli annidati sono fondamentali per lavorare con le matrici (array bidimensionali) — lo vedremo nella lezione sugli array.

---

## Esempio completo: indovina il numero

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int segreto, tentativo, tentativi;

    /* Genera un numero casuale tra 1 e 100 */
    srand(time(NULL));
    segreto = rand() % 100 + 1;

    tentativi = 0;

    printf("Ho scelto un numero tra 1 e 100. Indovina!\n\n");

    do {
        printf("Tentativo: ");
        scanf("%d", &tentativo);
        tentativi++;

        if (tentativo < segreto) {
            printf("Troppo basso!\n");
        } else if (tentativo > segreto) {
            printf("Troppo alto!\n");
        } else {
            printf("Esatto! Hai indovinato in %d tentativi.\n", tentativi);
        }

    } while (tentativo != segreto);

    return 0;
}
```

---

## Schema mentale

```
So quante volte ripetere?
    → for (i = 0; i < n; i++)

Non so quante volte ripetere, ma so quando fermarmi?
    → while (condizione) { ... }
    → assicurati che qualcosa nel ciclo faccia progredire verso l'uscita

Devo eseguire il corpo almeno una volta (es. menu)?
    → do { ... } while (condizione);

Devo uscire dal ciclo prima della fine?
    → break

Devo saltare un'iterazione?
    → continue
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che stampa la tavola pitagorica del numero inserito dall'utente.
```
Inserisci un numero: 7
7 x 1 = 7
7 x 2 = 14
...
7 x 10 = 70
```

**Esercizio 2**
Scrivi un programma che legge numeri interi finché l'utente inserisce 0, poi stampa quanti numeri positivi e quanti negativi ha inserito.

**Esercizio 3**
Scrivi un programma che legge un numero intero `n` e stampa i primi `n` numeri di Fibonacci.
```
n=8 → 0 1 1 2 3 5 8 13
```

**Esercizio 4**
Scrivi un programma con un menu che permette di:
1. Calcolare la somma di N numeri
2. Trovare il massimo tra N numeri
3. Uscire

Il menu deve ripresentarsi finché l'utente non sceglie di uscire.