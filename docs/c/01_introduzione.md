# Introduzione al linguaggio C

---

## Cos'è un programma

Un programma è un insieme di istruzioni che il computer esegue una dopo l'altra. Tu scrivi le istruzioni, il computer le esegue.

Il problema è che il computer non capisce il linguaggio naturale (come l'italiano), capisce solo sequenze di numeri chiamate **codice macchina**. Noi però non scriviamo in codice macchina: scriviamo in un linguaggio più leggibile, come il C, e poi lo facciamo tradurre.

---

## Il ciclo di vita di un programma C

Scrivere ed eseguire un programma C richiede tre passi:

```
1. Scrivi il codice sorgente   →   file .c
2. Compila                     →   file eseguibile
3. Esegui
```

**Passo 1 — scrivi il codice sorgente**
Il codice sorgente è il testo che scrivi tu — le istruzioni in linguaggio C, salvate in un file con estensione `.c`.

**Passo 2 — compila**
Il **compilatore** è un programma che traduce il tuo codice sorgente in un file eseguibile che il computer sa eseguire. In C usiamo il compilatore `gcc`.

**Passo 3 — esegui**
Lanci il programma e il computer esegue le istruzioni.

::: {.callout-note}
## Cosa succede se c'è un errore?
Se il codice ha errori di sintassi, il compilatore si ferma e ti segnala il problema — il file eseguibile non viene creato. Devi correggere l'errore e ricompilare.

Un programma che **compila** non significa che funziona correttamente — significa solo che la sintassi è corretta. Gli errori di logica si scoprono solo eseguendo il programma.
:::

---

## Il primo programma

Ogni programma C, anche il più semplice, ha questa struttura:

```c
#include <stdio.h>

int main() {
    printf("Ciao, mondo!\n");
    return 0;
}
```

Analizziamolo riga per riga.

**`#include <stdio.h>`**
Dice al compilatore di includere la libreria `stdio.h` — la libreria standard per l'input e l'output. Senza questa riga non puoi usare `printf` e `scanf`. Va sempre messa all'inizio.

**`int main()`**
Ogni programma C ha un punto di partenza: il blocco `main`. Il computer inizia sempre da qui. `int` indica che `main` restituisce un numero intero alla fine.

**`{ ... }`**
Le graffe delimitano il corpo del `main` — tutto il codice che vuoi eseguire va dentro di esse.

**`printf("Ciao, mondo!\n");`**
Stampa il testo sullo schermo. `\n` è il carattere di a capo — senza di esso il cursore resta sulla stessa riga. Il punto e virgola `;` termina ogni istruzione.

**`return 0;`**
Segnala al sistema operativo che il programma è terminato correttamente. 0 significa "nessun errore".

---

## Come compilare ed eseguire

Apri il terminale e posizionati nella cartella dove hai salvato il file.

```bash
# Compila
gcc programma.c -o programma

# Esegui
./programma
```

`gcc` è il compilatore. `programma.c` è il tuo file sorgente. `-o programma` dice al compilatore di chiamare il file eseguibile `programma`. Se non metti `-o nome`, il compilatore crea un file chiamato `a.out`.

::: {.callout-tip}
## Errori comuni di compilazione
Il compilatore segnala gli errori con riga e messaggio. Impara a leggere questi messaggi — sono il tuo strumento principale per correggere il codice.

```
programma.c:4:5: error: expected ';' before 'return'
```

Questo dice: nel file `programma.c`, alla riga 4, colonna 5, manca un punto e virgola prima di `return`.
:::

---

## Le regole fondamentali del C

Prima di andare avanti, ci sono alcune regole che devi sempre rispettare.

**Ogni istruzione termina con `;`**
```c
printf("Ciao\n");   // corretto
printf("Ciao\n")    // ERRORE — manca il punto e virgola
```

**Il C distingue maiuscole e minuscole**
```c
int x = 5;    // corretto
Int x = 5;    // ERRORE — Int non esiste, si scrive int
Printf(".."); // ERRORE — Printf non esiste, si scrive printf
```

**Le graffe devono sempre essere aperte e chiuse**
```c
int main() {
    printf("Ciao\n");
}   // la graffa chiusa corrisponde a quella aperta
```

**Le istruzioni si eseguono in ordine, dall'alto verso il basso**
```c
printf("Prima riga\n");
printf("Seconda riga\n");
printf("Terza riga\n");
```
Il computer esegue la prima, poi la seconda, poi la terza — sempre in quest'ordine.

---

## I commenti

I commenti sono note che scrivi nel codice per spiegare cosa fa — il compilatore li ignora completamente.

```c
// Questo è un commento su una riga sola

/* Questo è un commento
   su più righe */

int main() {
    printf("Ciao\n");  // stampa il saluto
    return 0;
}
```

Usare i commenti è una buona abitudine — rende il codice più leggibile, sia per chi lo legge sia per te quando lo rileggi dopo qualche giorno.

---

## Cosa sono le librerie

Il C di base sa fare pochissimo. Quasi tutto quello che ti serve — stampare, leggere, calcolare, gestire file — sta dentro delle **librerie**.

Una libreria è una raccolta di funzioni già scritte che puoi usare nel tuo programma. Per usarla devi includerla con `#include`.

Le librerie più comuni che userai:

| Libreria | Cosa contiene |
|---|---|
| `stdio.h` | `printf`, `scanf`, gestione file |
| `stdlib.h` | `exit`, conversioni, numeri casuali |
| `string.h` | funzioni per le stringhe |
| `math.h` | funzioni matematiche (`sqrt`, `pow`...) |
| `ctype.h` | funzioni sui caratteri (`isupper`, `islower`...) |

---

## Schema mentale da portarsi dietro

Ogni volta che scrivi un programma C, la struttura è sempre questa:

```c
#include <stdio.h>      /* librerie necessarie */

int main() {
    /* 1. dichiara le variabili */
    /* 2. leggi i dati (se servono) */
    /* 3. elabora */
    /* 4. stampa il risultato */
    return 0;
}
```

Non devi memorizzare tutto subito. L'obiettivo di questa prima lezione è capire il ciclo: **scrivi → compila → esegui**.

---

## Esercizio

Scrivi un programma che stampa le seguenti righe:

```
Benvenuto nel corso di informatica
Linguaggio: C
```

Suggerimenti:
- ogni riga vuole un `printf` separato
- non dimenticare `\n` alla fine di ogni riga
- compila con `gcc` e verifica l'output