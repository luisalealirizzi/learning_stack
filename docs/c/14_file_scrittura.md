# Scrittura su file

---

## Idea chiave

Nella lezione precedente abbiamo imparato a **leggere** dati da un file. Ora impariamo a **scrivere** — a produrre un risultato che persiste sul disco anche dopo che il programma finisce.

Lo schema e sempre lo stesso:

```
1. apri il file in scrittura
2. scrivi i dati
3. chiudi il file
```

La differenza rispetto alla lettura sta nella **modalita** con cui apri il file.

---

## Blocco 1: Aprire un file in scrittura

Usiamo sempre `fopen`, ma con una modalita diversa:

```c
FILE *f = fopen("output.txt", "w");   /* scrittura — crea o sovrascrive */
FILE *f = fopen("log.txt", "a");      /* append — aggiunge in fondo */
```

| Modalita | Cosa fa | Se il file esiste | Se il file non esiste |
|---|---|---|---|
| `"w"` | scrittura | lo sovrascrive | lo crea |
| `"a"` | aggiunta in fondo | aggiunge in fondo | lo crea |

::: {.callout-important}
## Attenzione alla modalita "w"
Con `"w"` il file viene **azzerato** ogni volta che lo apri — anche se non scrivi nulla. Se vuoi conservare il contenuto precedente usa `"a"`.

```c
FILE *f = fopen("dati.txt", "w");   /* il contenuto precedente e perso */
FILE *f = fopen("dati.txt", "a");   /* il contenuto precedente e conservato */
```
:::

Anche in scrittura devi sempre controllare `NULL`:

```c
FILE *f = fopen("output.txt", "w");
if (f == NULL) {
    printf("Errore: impossibile aprire il file.\n");
    return 1;
}
```

---

## Blocco 2: Scrivere con fprintf

`fprintf` funziona esattamente come `printf`, ma scrive su file invece che sullo schermo.

```c
fprintf(f, "formato", valori);
```

Tutto quello che sai su `printf` vale anche per `fprintf` — stessi segnaposto, stessi modificatori, stessi caratteri speciali.

```c
FILE *f = fopen("output.txt", "w");
if (f == NULL) { return 1; }

fprintf(f, "Ciao, mondo!\n");
fprintf(f, "Il risultato e: %d\n", 42);
fprintf(f, "Media: %.2f\n", 7.85);

fclose(f);
```

Il file `output.txt` conterra:
```
Ciao, mondo!
Il risultato e: 42
Media: 7.85
```

### Esempio: salvare i risultati di un calcolo

```c
#include <stdio.h>

int main() {
    int voti[] = {7, 8, 6, 9, 7};
    int n = 5;
    int i, somma = 0;

    for (i = 0; i < n; i++)
        somma += voti[i];

    double media = (double)somma / n;

    FILE *f = fopen("risultati.txt", "w");
    if (f == NULL) {
        printf("Errore apertura file.\n");
        return 1;
    }

    fprintf(f, "=== RISULTATI ===\n");
    fprintf(f, "Voti: ");
    for (i = 0; i < n; i++)
        fprintf(f, "%d ", voti[i]);
    fprintf(f, "\n");
    fprintf(f, "Somma: %d\n", somma);
    fprintf(f, "Media: %.2f\n", media);

    fclose(f);
    printf("Risultati salvati in risultati.txt\n");
    return 0;
}
```

---

## Blocco 3: Scrivere con fputs

`fputs` scrive una stringa su file senza formattazione — piu semplice di `fprintf` quando devi scrivere solo testo.

```c
fputs("testo da scrivere\n", f);
```

::: {.callout-note}
## fprintf vs fputs
Usa `fprintf` quando devi scrivere valori formattati (numeri, variabili).
Usa `fputs` quando devi scrivere solo stringhe gia pronte.

```c
fprintf(f, "Punteggio: %d\n", punteggio);   /* con variabile */
fputs("Fine del file.\n", f);               /* solo testo */
```
:::

---

## Blocco 4: Append — aggiungere senza sovrascrivere

Con la modalita `"a"` ogni scrittura si aggiunge in fondo al file senza cancellare il contenuto precedente. E il modo giusto per tenere un **log** — un registro cronologico di eventi.

```c
#include <stdio.h>

void scriviLog(const char messaggio[]) {
    FILE *f = fopen("log.txt", "a");
    if (f == NULL) return;
    fprintf(f, "%s\n", messaggio);
    fclose(f);
}

int main() {
    scriviLog("Programma avviato.");
    scriviLog("Lettura dati completata.");
    scriviLog("Elaborazione terminata.");
    scriviLog("Programma chiuso.");
    return 0;
}
```

Ogni volta che esegui il programma, le righe vengono aggiunte in fondo. Il file `log.txt` cresce ad ogni esecuzione:

```
Programma avviato.
Lettura dati completata.
Elaborazione terminata.
Programma chiuso.
Programma avviato.
Lettura dati completata.
...
```

::: {.callout-note}
## Quando usare "w" e quando "a"
Usa `"w"` quando vuoi il risultato dell'ultima esecuzione — ad esempio un report aggiornato.
Usa `"a"` quando vuoi tenere uno storico — ad esempio un log o un diario di bordo.
:::

---

## Blocco 5: Leggere e scrivere insieme

Spesso un programma legge dati da un file, li elabora e scrive i risultati su un altro file. E il pattern piu comune nel mondo reale.

```c
#include <stdio.h>

int main() {
    FILE *in  = fopen("input.txt",  "r");
    FILE *out = fopen("output.txt", "w");

    if (in == NULL) {
        printf("Errore apertura input.\n");
        return 1;
    }
    if (out == NULL) {
        printf("Errore apertura output.\n");
        fclose(in);   /* chiudi quello gia aperto */
        return 1;
    }

    int n, totale = 0, quanti = 0;

    while (fscanf(in, "%d", &n) == 1) {
        totale += n;
        quanti++;
    }

    if (quanti > 0) {
        fprintf(out, "Numeri letti: %d\n", quanti);
        fprintf(out, "Somma:        %d\n", totale);
        fprintf(out, "Media:        %.2f\n", (double)totale / quanti);
    } else {
        fprintf(out, "File vuoto.\n");
    }

    fclose(in);
    fclose(out);

    printf("Elaborazione completata.\n");
    return 0;
}
```

::: {.callout-important}
## Chiudi sempre tutti i file aperti
Se apri due file, devi chiuderli entrambi. Se il secondo `fopen` fallisce, chiudi quello gia aperto prima di uscire — altrimenti lasci una risorsa occupata.

```c
if (out == NULL) {
    fclose(in);   /* chiudi in prima di uscire */
    return 1;
}
```
:::

---

## Blocco 6: Errori comuni

**Scrivere su un file aperto in lettura**
```c
FILE *f = fopen("dati.txt", "r");
fprintf(f, "ciao\n");   /* ERRORE — il file e aperto in lettura */
```

**Dimenticare il newline**
```c
fprintf(f, "prima riga");
fprintf(f, "seconda riga");
/* il file conterra: "prima rigaseconda riga" — tutto sulla stessa riga */
```

**Usare "w" invece di "a" per i log**
```c
FILE *f = fopen("log.txt", "w");   /* ogni esecuzione azzera il log! */
FILE *f = fopen("log.txt", "a");   /* corretto — aggiunge in fondo */
```

**Non chiudere il file prima della fine del programma**
```c
FILE *f = fopen("output.txt", "w");
fprintf(f, "dati...\n");
return 0;   /* manca fclose — i dati potrebbero non essere scritti sul disco */
```

::: {.callout-warning}
## Il buffer di scrittura
Quando scrivi con `fprintf`, i dati non vanno subito sul disco — vengono accumulati in un buffer e scritti in blocco. `fclose` svuota il buffer e garantisce che tutto sia scritto. Senza `fclose`, parte dei dati potrebbe andare persa.
:::

---

## Schema mentale

```
Devo scrivere dati su un file?

1. Scegli la modalita
   voglio sovrascrivere?   -> "w"
   voglio aggiungere?      -> "a"

2. Apri il file
   FILE *f = fopen("nome.txt", "w");

3. Controlla sempre
   if (f == NULL) { ... return 1; }

4. Scrivi
   con variabili?   -> fprintf(f, "formato", valori)
   solo testo?      -> fputs("testo\n", f)

5. Chiudi sempre
   fclose(f);
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge 5 numeri interi dalla tastiera e li salva su un file `numeri.txt`, uno per riga.

**Esercizio 2**
Scrivi un programma che legge i numeri da `numeri.txt` (creato nell'esercizio 1), calcola la somma e la media e salva i risultati su `risultati.txt`.

**Esercizio 3**
Scrivi una funzione `void scriviLog(const char messaggio[])` che aggiunge una riga al file `log.txt` ogni volta che viene chiamata. Chiamala piu volte dal main con messaggi diversi e verifica che il file cresca ad ogni esecuzione.

**Esercizio 4**
Scrivi un programma che legge un file di testo riga per riga e scrive su un altro file solo le righe che contengono almeno una cifra numerica.
