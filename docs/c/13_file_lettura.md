# File

---

## Idea chiave

Fino ad ora i nostri programmi leggono dati dalla tastiera e li perdono quando il programma finisce. I **file** permettono di leggere dati che esistono già sul disco — dati che persistono tra un'esecuzione e l'altra.

Lavorare con i file in C segue sempre lo stesso schema:

```
1. apri il file
2. leggi i dati
3. chiudi il file
```

Se dimentichi di chiudere il file, rischi di perdere dati o di lasciare risorse occupate.

---

## Blocco 1: Aprire e chiudere un file

Per lavorare con un file in C si usa un **puntatore a FILE** — un tipo speciale definito in `stdio.h`.

```c
#include <stdio.h>

FILE *f;
f = fopen("dati.txt", "r");   /* apri in lettura */
```

`fopen` riceve due argomenti:
- il **nome del file** — il percorso sul disco
- la **modalita** — cosa vuoi fare con il file

Le modalita principali:

| Modalita | Significato |
|---|---|
| `"r"` | lettura (read) — il file deve esistere |
| `"w"` | scrittura (write) — crea o sovrascrive |
| `"a"` | aggiunta (append) — aggiunge in fondo |

Per questa lezione usiamo solo `"r"`.

### Verificare se il file esiste

`fopen` restituisce `NULL` se il file non esiste o non e accessibile. Devi sempre controllare:

```c
FILE *f = fopen("dati.txt", "r");

if (f == NULL) {
    printf("Errore: impossibile aprire il file.\n");
    return 1;
}
```

Non controllare questo e uno degli errori piu comuni — il programma crasha in modo imprevedibile.

### Chiudere il file

Quando hai finito, chiudi sempre il file con `fclose`:

```c
fclose(f);
```

::: {.callout-important}
## Schema obbligatorio
```c
FILE *f = fopen("nome.txt", "r");
if (f == NULL) {
    printf("Errore apertura file.\n");
    return 1;
}

/* ... lettura ... */

fclose(f);
```
Questo schema va sempre rispettato. Apri, controlla, leggi, chiudi.
:::

---

## Blocco 2: Leggere parola per parola con fscanf

`fscanf` funziona come `scanf`, ma legge da un file invece che dalla tastiera.

```c
fscanf(f, "formato", &variabile);
```

Legge un valore alla volta, saltando automaticamente spazi e andate a capo.

### Esempio: leggere numeri interi

File `numeri.txt`:
```
10 25 7 42 3
```

```c
#include <stdio.h>

int main() {
    FILE *f = fopen("numeri.txt", "r");
    if (f == NULL) {
        printf("Errore apertura file.\n");
        return 1;
    }

    int n;
    while (fscanf(f, "%d", &n) == 1) {
        printf("Letto: %d\n", n);
    }

    fclose(f);
    return 0;
}
```

Output:
```
Letto: 10
Letto: 25
Letto: 7
Letto: 42
Letto: 3
```

### Come funziona il ciclo

`fscanf` restituisce il numero di valori letti con successo. Quando arriva alla fine del file, non riesce a leggere nulla e restituisce un valore diverso da 1. Il ciclo si ferma.

```c
while (fscanf(f, "%d", &n) == 1) {
    /* continua finche riesce a leggere un intero */
}
```

Schema mentale: **leggo un valore alla volta finche il file non e finito**.

### Esempio: leggere parole

File `parole.txt`:
```
ciao mondo come stai
```

```c
char parola[50];
while (fscanf(f, "%s", parola) == 1) {
    printf("Parola: %s\n", parola);
}
```

Output:
```
Parola: ciao
Parola: mondo
Parola: come
Parola: stai
```

::: {.callout-note}
## fscanf si ferma agli spazi
`fscanf` con `%s` legge fino al primo spazio o andata a capo. Se una riga contiene piu parole, le legge una alla volta ad ogni chiamata. Non riesce a leggere una riga intera in un colpo solo.
:::

---

## Blocco 3: Leggere riga per riga con fgets

`fgets` legge una riga intera dal file, spazi inclusi. Funziona esattamente come quando la usi con `stdin`, ma al posto di `stdin` passi il puntatore al file.

```c
fgets(s, dimensione, f);
```

### Esempio: leggere righe

File `frasi.txt`:
```
ciao mondo
oggi fa bello
terza riga
```

```c
#include <stdio.h>
#include <string.h>

int main() {
    FILE *f = fopen("frasi.txt", "r");
    if (f == NULL) {
        printf("Errore apertura file.\n");
        return 1;
    }

    char riga[200];
    while (fgets(riga, sizeof(riga), f) != NULL) {
        printf("Riga: %s", riga);   /* \n gia incluso nella riga */
    }

    fclose(f);
    return 0;
}
```

Output:
```
Riga: ciao mondo
Riga: oggi fa bello
Riga: terza riga
```

### Come funziona il ciclo

`fgets` restituisce `NULL` quando arriva alla fine del file. Il ciclo si ferma.

```c
while (fgets(riga, sizeof(riga), f) != NULL) {
    /* continua finche ci sono righe da leggere */
}
```

Schema mentale: **leggo una riga alla volta finche il file non e finito**.

### Il newline nelle righe

Come con `stdin`, `fgets` include il `\n` finale nella stringa. Se vuoi elaborare la riga senza il newline, eliminalo:

```c
riga[strcspn(riga, "\n")] = '\0';
```

---

## Blocco 4: fscanf vs fgets

Queste due funzioni non sono intercambiabili. La scelta dipende da come sono strutturati i dati nel file.

| | `fscanf` | `fgets` |
|---|---|---|
| **Legge** | un valore alla volta | una riga intera |
| **Si ferma a** | spazi e andate a capo | solo andata a capo |
| **Legge gli spazi** | no | si |
| **Quando usarlo** | dati separati da spazi (numeri, parole singole) | righe di testo, frasi |
| **Condizione di uscita** | `fscanf(...) == 1` | `fgets(...) != NULL` |

### Schema mentale

```
Il file contiene numeri o parole separati da spazi?
    → usa fscanf
    → while (fscanf(f, "%d", &n) == 1)

Il file contiene righe di testo con spazi?
    → usa fgets
    → while (fgets(riga, sizeof(riga), f) != NULL)
```

---

## Blocco 5: Elaborare i dati letti

Leggere un file non basta — di solito vuoi fare qualcosa con i dati. Lo schema e sempre lo stesso che hai visto con gli array e le stringhe: leggi un elemento, elaboralo, passa al successivo.

### Somma di numeri da file

File `numeri.txt`:
```
10 25 7 42 3
```

```c
#include <stdio.h>

int main() {
    FILE *f = fopen("numeri.txt", "r");
    if (f == NULL) {
        printf("Errore apertura file.\n");
        return 1;
    }

    int n, somma = 0, quanti = 0;

    while (fscanf(f, "%d", &n) == 1) {
        somma += n;
        quanti++;
    }

    fclose(f);

    if (quanti > 0) {
        printf("Numeri letti: %d\n", quanti);
        printf("Somma:        %d\n", somma);
        printf("Media:        %.2f\n", (double)somma / quanti);
    } else {
        printf("File vuoto.\n");
    }

    return 0;
}
```

### Contare le righe di un file

```c
#include <stdio.h>

int main() {
    FILE *f = fopen("frasi.txt", "r");
    if (f == NULL) {
        printf("Errore apertura file.\n");
        return 1;
    }

    char riga[200];
    int righe = 0;

    while (fgets(riga, sizeof(riga), f) != NULL) {
        righe++;
    }

    fclose(f);
    printf("Righe nel file: %d\n", righe);

    return 0;
}
```

### Cercare una parola in un file

```c
#include <stdio.h>
#include <string.h>

int main() {
    FILE *f = fopen("frasi.txt", "r");
    if (f == NULL) {
        printf("Errore apertura file.\n");
        return 1;
    }

    char riga[200];
    char cercata[] = "bello";
    int trovata = 0;
    int numRiga = 0;

    while (fgets(riga, sizeof(riga), f) != NULL) {
        numRiga++;
        if (strstr(riga, cercata) != NULL) {
            printf("Trovata alla riga %d: %s", numRiga, riga);
            trovata = 1;
        }
    }

    if (!trovata) {
        printf("Parola non trovata.\n");
    }

    fclose(f);
    return 0;
}
```

::: {.callout-note}
## strstr
`strstr(s1, s2)` cerca la stringa `s2` dentro `s1`. Restituisce un puntatore alla prima occorrenza, oppure `NULL` se non la trova. E definita in `string.h`.
:::

---

## Blocco 6: Errori comuni

**Dimenticare di controllare NULL dopo fopen**
```c
FILE *f = fopen("dati.txt", "r");
fscanf(f, "%d", &n);   /* ERRORE — se f e NULL, il programma crasha */
```

**Dimenticare fclose**
```c
FILE *f = fopen("dati.txt", "r");
/* ... lettura ... */
/* manca fclose — risorsa non rilasciata */
```

**Usare fscanf quando servono le righe intere**
```c
/* file contiene: "ciao mondo" */
fscanf(f, "%s", parola);   /* legge solo "ciao" — mondo viene perso */
fgets(riga, sizeof(riga), f);  /* legge "ciao mondo" — corretto */
```

**Usare il percorso sbagliato**
```c
FILE *f = fopen("C:\\Users\\nome\\dati.txt", "r");  /* percorso assoluto — non portabile */
FILE *f = fopen("dati.txt", "r");                   /* percorso relativo — preferibile */
```

---

## Schema mentale generale

```
Devo leggere dati da un file?

1. Apri il file
   FILE *f = fopen("nome.txt", "r");

2. Controlla sempre
   if (f == NULL) { ... return 1; }

3. Scegli come leggere
   dati separati da spazi?  → fscanf(f, "%d", &n) == 1
   righe intere?            → fgets(riga, sizeof(riga), f) != NULL

4. Elabora i dati nel ciclo

5. Chiudi il file
   fclose(f);
```

---

## Esercizi

**Esercizio 1**
Dato un file `voti.txt` che contiene una serie di interi (uno per riga), scrivi un programma che legge tutti i voti e stampa il massimo, il minimo e la media.

**Esercizio 2**
Dato un file `testo.txt` che contiene alcune righe di testo, scrivi un programma che stampa solo le righe che contengono piu di 20 caratteri.

**Esercizio 3**
Dato un file `parole.txt` che contiene una parola per riga, scrivi un programma che conta quante parole iniziano con una lettera maiuscola.

**Esercizio 4**
Dato un file `numeri.txt` che contiene numeri interi separati da spazi, scrivi un programma che li legge tutti, li salva in un array e stampa quanti sono pari e quanti sono dispari.
