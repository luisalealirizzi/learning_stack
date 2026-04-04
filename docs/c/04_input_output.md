# Input e output

---

## Comunicare con l'utente

Un programma utile deve saper fare due cose: **ricevere dati** dall'utente e **restituire risultati**. In C questo si fa con due funzioni della libreria `stdio.h`:

- `printf` — stampa dati sullo schermo
- `scanf` — legge dati dalla tastiera

Le hai già incontrate nelle lezioni precedenti. Ora le vediamo in modo completo.

---

## printf — stampare sullo schermo

`printf` stampa testo e valori sullo schermo. La struttura è sempre:

```c
printf("stringa di formato", variabile1, variabile2, ...);
```

La **stringa di formato** è il testo da stampare. Può contenere testo normale e **segnaposto** — codici che indicano dove inserire i valori delle variabili.

```c
int eta = 17;
printf("Ho %d anni.\n", eta);
/* stampa: Ho 17 anni. */
```

### I segnaposto

| Segnaposto | Tipo | Esempio |
|---|---|---|
| `%d` | `int` | `printf("%d", 42)` → `42` |
| `%f` | `float` / `double` | `printf("%f", 3.14)` → `3.140000` |
| `%lf` | `double` (con scanf) | — |
| `%c` | `char` | `printf("%c", 'A')` → `A` |
| `%s` | stringa | `printf("%s", "ciao")` → `ciao` |

### Formattare i numeri decimali

Di default `%f` stampa 6 cifre decimali. Puoi controllare quante cifre vuoi:

```c
double media = 7.666666;

printf("%f\n",   media);    /* 7.666666 — default */
printf("%.2f\n", media);    /* 7.67     — 2 decimali */
printf("%.0f\n", media);    /* 8        — nessun decimale */
printf("%8.2f\n", media);   /* __7.67   — 8 caratteri totali, 2 decimali */
```

### I caratteri speciali

All'interno delle stringhe alcuni caratteri hanno un significato speciale:

| Sequenza | Effetto |
|---|---|
| `\n` | vai a capo |
| `\t` | tabulazione orizzontale |
| `\\` | stampa il carattere `\` |
| `\"` | stampa il carattere `"` |
| `%%` | stampa il carattere `%` |

```c
printf("Nome:\tArthas\n");
printf("Punteggio:\t%d%%\n", 85);
/* stampa:
   Nome:   Arthas
   Punteggio:  85%  */
```

### Stampare più valori

```c
int livello = 5;
double vita = 87.5;
char classe = 'G';

printf("Classe: %c | Livello: %d | Vita: %.1f%%\n", classe, livello, vita);
/* stampa: Classe: G | Livello: 5 | Vita: 87.5% */
```

I segnaposto vengono sostituiti nell'ordine: il primo `%c` con `classe`, il primo `%d` con `livello`, e così via.

---

## scanf — leggere dalla tastiera

`scanf` legge quello che l'utente digita e lo salva in una variabile. La struttura è:

```c
scanf("segnaposto", &variabile);
```

La `&` davanti alla variabile è obbligatoria — indica l'indirizzo in memoria dove salvare il valore. Vedremo meglio cosa significa quando studieremo i puntatori. Per ora: **con `scanf` si mette sempre `&`**.

```c
int eta;
printf("Inserisci la tua età: ");
scanf("%d", &eta);
```

### Leggere tipi diversi

```c
int n;
double x;
char c;
char nome[50];   /* stringa — la vediamo dopo */

scanf("%d", &n);      /* legge un intero */
scanf("%lf", &x);     /* legge un double — nota: %lf, non %f */
scanf(" %c", &c);     /* legge un carattere — nota lo spazio prima di %c */
scanf("%s", nome);    /* legge una parola — senza & per le stringhe */
```

::: {.callout-warning}
## Tre cose da ricordare con scanf
**1. Con `scanf` si usa `%lf` per i double, non `%f`.**
```c
double x;
scanf("%lf", &x);   /* corretto */
scanf("%f", &x);    /* sbagliato — comportamento indefinito */
```

**2. Lo spazio prima di `%c` è necessario.**
```c
int n;
char c;
scanf("%d", &n);
scanf(" %c", &c);   /* lo spazio consuma l'invio rimasto nel buffer */
```
Senza lo spazio, `scanf` leggerebbe il carattere `\n` rimasto dal `%d` precedente.

**3. Non mettere `\n` nella stringa di formato di scanf.**
```c
scanf("%d\n", &n);   /* sbagliato — aspetta un altro invio */
scanf("%d", &n);     /* corretto */
```
:::

### Leggere più valori con un solo scanf

```c
int a, b;
scanf("%d %d", &a, &b);   /* legge due interi separati da spazio */
```

L'utente può separare i valori con spazi o con invio — `scanf` li gestisce allo stesso modo.

---

## Il buffer di input

Quando l'utente preme invio, il testo digitato va in una zona di memoria chiamata **buffer di input**. `scanf` legge dal buffer — non direttamente dalla tastiera.

Questo causa un problema classico: quando `scanf` legge un intero con `%d`, lascia nel buffer il carattere `\n` (l'invio). Se subito dopo leggi un carattere con `%c`, `scanf` troverà quel `\n` nel buffer e lo leggerà — invece di aspettare che l'utente digiti qualcosa.

```c
int n;
char c;

scanf("%d", &n);      /* legge il numero, lascia \n nel buffer */
scanf("%c", &c);      /* legge \n invece di aspettare l'utente! */
scanf(" %c", &c);     /* lo spazio consuma il \n — corretto */
```

Schema mentale: **ogni volta che leggi un `char` dopo un `int` o un `double`, metti uno spazio prima di `%c`**.

---

## Allineare l'output

Quando stampi tabelle o elenchi, puoi allineare i valori con i modificatori di larghezza:

```c
printf("%-15s %5s %8s\n", "Nome", "Voto", "Media");
printf("%-15s %5d %8.2f\n", "Arthas", 8, 7.83);
printf("%-15s %5d %8.2f\n", "Gandalf", 9, 8.50);
printf("%-15s %5d %8.2f\n", "Legolas", 7, 6.67);
```

Output:
```
Nome             Voto    Media
Arthas              8     7.83
Gandalf             9     8.50
Legolas             7     6.67
```

- `%15s` — testo allineato a destra in 15 caratteri
- `%-15s` — testo allineato a sinistra in 15 caratteri (il `-` inverte l'allineamento)
- `%5d` — intero in 5 caratteri, allineato a destra
- `%8.2f` — decimale in 8 caratteri totali con 2 cifre dopo la virgola

---

## Esempio completo: scheda personaggio

```c
#include <stdio.h>

int main() {
    char nome[50];
    int livello;
    double vita;
    char classe;

    /* Lettura dei dati */
    printf("=== CREA IL TUO PERSONAGGIO ===\n\n");

    printf("Nome: ");
    scanf("%s", nome);

    printf("Classe (G=Guerriero, M=Mago, A=Arciere): ");
    scanf(" %c", &classe);

    printf("Livello (1-10): ");
    scanf("%d", &livello);

    printf("Punti vita: ");
    scanf("%lf", &vita);

    /* Stampa della scheda */
    printf("\n--- SCHEDA PERSONAGGIO ---\n");
    printf("%-10s %s\n",   "Nome:",    nome);
    printf("%-10s %c\n",   "Classe:",  classe);
    printf("%-10s %d\n",   "Livello:", livello);
    printf("%-10s %.1f\n", "Vita:",    vita);

    return 0;
}
```

---

## Schema mentale

```
Devo stampare qualcosa?
    → printf("...", valori)
    → scegli il segnaposto giusto: %d %f %c %s
    → controlla i decimali: %.2f
    → vai a capo con \n

Devo leggere qualcosa?
    → scanf("segnaposto", &variabile)
    → usa %lf per i double
    → metti spazio prima di %c se hai letto un int prima
    → non mettere \n nella stringa di formato

Devo allineare l'output?
    → usa la larghezza: %10d, %-15s, %8.2f
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge nome, cognome e anno di nascita di una persona e stampa:
```
Benvenuto, Mario Rossi!
Sei nato nel 1990.
```

**Esercizio 2**
Scrivi un programma che legge tre voti interi e stampa una tabella allineata con i voti e la media:
```
Voto 1:     7
Voto 2:     8
Voto 3:     6
Media:   7.00
```

**Esercizio 3**
Scrivi un programma che legge un intero e un carattere (in quest'ordine) e li stampa entrambi. Fai attenzione al problema del buffer.