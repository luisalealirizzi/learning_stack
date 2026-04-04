# Stringhe

---

## Idea chiave

In C non esiste un tipo stringa. Una stringa è un **array di caratteri** che termina con il carattere speciale `'\0'` (chiamato terminatore nullo o null terminator).

```c
char nome[10] = "Arthas";
```

In memoria si trova così:

```
'A' 'r' 't' 'h' 'a' 's' '\0' ? ? ?
 0    1    2    3    4    5    6
```

Il `'\0'` segnala la fine della stringa. Senza di esso il C non saprebbe dove finisce il testo.

::: {.callout-important}
## La dimensione deve contenere anche '\0'
Se la parola ha 6 caratteri, l'array deve avere almeno 7 celle — una in più per il terminatore.

```c
char nome[7] = "Arthas";   /* 6 lettere + '\0' */
char nome[6] = "Arthas";   /* ERRORE — non c'è spazio per '\0' */
```
:::

---

## Perché non si usa scanf per leggere le stringhe

Prima di vedere come si legge una stringa, è importante capire perché `scanf` da solo non basta.

`scanf("%s", s)` legge caratteri fino al primo spazio o invio — poi si ferma. Se l'utente scrive `"ciao mondo"`, `scanf` legge solo `"ciao"` e lascia `" mondo"` nel buffer.

```c
char s[100];
scanf("%s", s);
/* l'utente scrive: "ciao mondo" */
/* s contiene solo: "ciao" */
```

Per leggere una frase intera — spazi inclusi — si usa `fgets`.

---

## Come funziona fgets

```c
fgets(s, dimensione, stdin);
```

`fgets` riceve tre argomenti:

- `s` — l'array dove salvare la stringa
- `dimensione` — il numero massimo di caratteri da leggere, spazi inclusi
- `stdin` — da dove leggere (standard input = tastiera)

`fgets` legge caratteri dalla tastiera finché non incontra un invio (`\n`) oppure raggiunge il limite di `dimensione - 1` caratteri. Aggiunge sempre `'\0'` alla fine.

```c
char s[10];
fgets(s, 10, stdin);
/* l'utente scrive: "ciao" + invio */
/* s contiene: "ciao\n\0" */
```

::: {.callout-note}
## fgets salva anche il newline
`fgets` include il `\n` dell'invio nella stringa. Quindi se l'utente scrive `"ciao"` e preme invio, la stringa contiene `"ciao\n"` — non solo `"ciao"`.

In alcuni esercizi questo non dà fastidio. In altri bisogna eliminarlo. Lo vediamo nel Blocco 1.
:::

### fgets protegge dall'overflow

Il secondo argomento di `fgets` è fondamentale: dice quanti caratteri al massimo leggere. Se l'utente scrive più caratteri del previsto, `fgets` si ferma al limite — l'array non può andare in overflow.

```c
char s[10];
fgets(s, 10, stdin);
/* anche se l'utente scrive 100 caratteri, fgets ne legge al massimo 9 + '\0' */
```

Con `scanf("%s", s)` invece non c'è nessun limite — se l'utente scrive troppo, si va fuori dall'array con conseguenze imprevedibili.

### Usare sizeof per la dimensione

Invece di scrivere il numero a mano, usa `sizeof(s)` — così se cambi la dimensione dell'array non devi ricordare di aggiornare anche `fgets`:

```c
char s[200];
fgets(s, sizeof(s), stdin);   /* preferibile a fgets(s, 200, stdin) */
```

### Confronto scanf vs fgets

| | `scanf("%s", s)` | `fgets(s, dim, stdin)` |
|---|---|---|
| **Legge gli spazi** | ❌ si ferma allo spazio | ✅ sì |
| **Legge il newline** | ❌ lo lascia nel buffer | ✅ lo include nella stringa |
| **Protegge dall'overflow** | ❌ no | ✅ sì, con il limite |
| **Quando usarlo** | una singola parola | una frase intera |

::: {.callout-tip}
## Regola pratica
Quando devi leggere una stringa, usa sempre `fgets`. È più sicuro e legge anche le frasi con spazi.

```c
char s[200];
fgets(s, sizeof(s), stdin);
```

Se vuoi solo una parola senza spazi, `scanf("%s", s)` funziona — ma tieni presente che non ha protezione dall'overflow.
:::

## Blocco 1: Saper leggere correttamente una stringa

Devi saper fare senza esitazione questo schema: **leggo con `fgets`, se serve tolgo il newline**.

```c
char s[200];
fgets(s, sizeof(s), stdin);

/* se dobbiamo eliminare l'andata a capo */
s[strcspn(s, "\n")] = '\0';
```

Cose da sapere bene:
- `fgets` legge anche gli spazi — a differenza di `scanf("%s", ...)`
- salva anche `\n` alla fine — in alcuni casi va eliminato
- la stringa finisce comunque con `'\0'`

### Quando NON è necessario togliere `\n`

Non serve eliminarlo quando scorriamo la stringa carattere per carattere. Il `\n` verrà semplicemente trattato come un carattere qualsiasi.

```c
int i = 0;
while (s[i] != '\0') {
    /* lavoro su s[i] */
    i++;
}
```

Questo va bene negli esercizi in cui:
- si contano caratteri (es: frequenza dei caratteri)
- si analizza il testo (es: conteggio vocali)
- si modificano solo alcune lettere (es: crittografia delle sole lettere)
- si ignorano i caratteri non alfabetici

In questi casi il `\n` non crea problemi.

### Quando invece bisogna togliere `\n`

È necessario eliminarlo quando vogliamo che la stringa rappresenti esattamente il testo inserito, senza l'invio finale. Questo succede quando:

**confrontiamo stringhe**
```c
strcmp(s, "ciao")
```
Se la stringa è `"ciao\n"` il confronto fallisce.

**calcoliamo la lunghezza "reale"**
Il newline verrebbe contato come carattere in più.

**facciamo elaborazioni sensibili alla posizione**
Per esempio: palindromi, parsing preciso del testo, confronti esatti tra stringhe.

---

## Blocco 2: Saper scorrere tutta la stringa

Questo è il cuore di quasi tutto: se non sai fare questo, non puoi fare né frequenze né crittografia.

```c
int i = 0;
while (s[i] != '\0') {
    /* lavoro su s[i] */
    i++;
}
```

Lo schema mentale deve essere: **leggo un carattere alla volta finché trovo `'\0'`**.

---

## Blocco 3: Saper riconoscere il tipo di carattere

Negli esercizi questo è fondamentale. Devi saper distinguere lettera maiuscola, lettera minuscola, carattere non alfabetico.

**Schema minimo — senza librerie:**

```c
if (s[i] >= 'A' && s[i] <= 'Z') {
    /* maiuscola */
} else if (s[i] >= 'a' && s[i] <= 'z') {
    /* minuscola */
} else {
    /* altro */
}
```

**Con la libreria `ctype.h`** — più leggibile:

```c
#include <ctype.h>

isupper(s[i])    /* 1 se maiuscola */
islower(s[i])    /* 1 se minuscola */
isalpha(s[i])    /* 1 se lettera */
isdigit(s[i])    /* 1 se cifra 0-9 */
isspace(s[i])    /* 1 se spazio, tab, \n */
toupper(s[i])    /* converte in maiuscolo */
tolower(s[i])    /* converte in minuscolo */
```

**Trasformazione manuale — senza ctype.h:**

```c
/* Da minuscolo a maiuscolo */
if (s[i] >= 'a' && s[i] <= 'z') {
    s[i] = s[i] - 'a' + 'A';   /* oppure: s[i] -= 32; */
}

/* Da maiuscolo a minuscolo */
if (s[i] >= 'A' && s[i] <= 'Z') {
    s[i] = s[i] - 'A' + 'a';   /* oppure: s[i] += 32; */
}
```

::: {.callout-note}
## Perché funziona la trasformazione con i numeri?
I caratteri sono numeri — seguono la tabella ASCII. `'A'` vale 65, `'a'` vale 97. La differenza è sempre 32. Quindi `'a' - 'A' = 32` e `'A' + 32 = 'a'`. Per lo stesso motivo `'0'` vale 48, e `s[i] - '0'` converte un carattere cifra nel suo valore numerico.
:::

---

## Blocco 4: Le funzioni della libreria string.h

```c
#include <string.h>
```

Le funzioni più usate:

| Funzione | Cosa fa | Esempio |
|---|---|---|
| `strlen(s)` | lunghezza della stringa (senza `\0`) | `strlen("ciao")` → `4` |
| `strcpy(dst, src)` | copia `src` in `dst` | `strcpy(s2, s1)` |
| `strcat(dst, src)` | aggiunge `src` in fondo a `dst` | `strcat(s, " mondo")` |
| `strcmp(s1, s2)` | confronta due stringhe | `0` se uguali |
| `strcspn(s, rif)` | posizione del primo char di `rif` in `s` | trova `\n` |

### strcmp — confrontare due stringhe

`strcmp` restituisce:
- `0` se le stringhe sono uguali
- un valore negativo se `s1` viene prima di `s2` alfabeticamente
- un valore positivo se `s1` viene dopo

```c
if (strcmp(s1, s2) == 0) {
    printf("Stringhe uguali\n");
}
```

::: {.callout-warning}
## Non usare == per confrontare stringhe
In C non puoi confrontare stringhe con `==` — confronteresti gli indirizzi in memoria, non il contenuto.

```c
if (s1 == s2) { ... }         /* SBAGLIATO */
if (strcmp(s1, s2) == 0) { }  /* CORRETTO */
```
:::

---

## Blocco 5: Come si scrive una funzione che lavora sulle stringhe

Una funzione che lavora sulle stringhe svolge un compito preciso: contare qualcosa, trasformare una stringa, confrontare, cercare, convertire.

### Le tre cose da decidere

**1. Il tipo restituito**

Si usa `int` quando la funzione deve restituire un conteggio, una posizione, un valore numerico, 1 oppure 0:

```c
int contaMaiuscole(const char s[]);
int contaVocali(const char s[]);
int sottoStringa(const char s1[], const char s2[]);
```

Si usa `void` quando la funzione produce un risultato modificando una stringa:

```c
void maiuscolo(const char s1[], char s2[]);
```

**2. I parametri**

*Caso 1: una stringa*
```c
int contaMaiuscole(const char s[])
```
Serve per: contare, analizzare, convertire in place.

*Caso 2: due stringhe*
```c
void maiuscolo(const char s1[], char s2[])
```
`s1` è la stringa di partenza, `s2` è la stringa risultato.

*Caso 3: stringa + numero*
```c
void crittografa(const char s[], int k)
```
Serve per: traslare i caratteri, tagliare, cercare una posizione.

**3. Lo schema del corpo**

```c
tipo funzione(const char s[]) {
    int i = 0;
    while (s[i] != '\0') {
        /* lavoro su s[i] */
        i++;
    }
}
```

Questo è il cuore dell'algoritmo.

### Funzione che conta qualcosa

```c
int funzione(const char s[]) {
    int i = 0, cont = 0;
    while (s[i] != '\0') {
        if (/* condizione */)
            cont++;
        i++;
    }
    return cont;
}
```

Esempi possibili: `contaMaiuscole`, `contaVocali`, `contaSpazi`.

### Funzione che trasforma una stringa

```c
void funzione(const char s1[], char s2[]) {
    int i = 0;
    while (s1[i] != '\0') {
        s2[i] = /* trasformazione di s1[i] */;
        i++;
    }
    s2[i] = '\0';   /* fondamentale — termina la nuova stringa */
}
```

### Funzione che confronta due stringhe

```c
int funzione(const char s1[], const char s2[]) {
    int i = 0;
    while (s1[i] != '\0' && s2[i] != '\0') {
        if (s1[i] != s2[i])
            return 0;
        i++;
    }
    return s1[i] == s2[i];   /* vero solo se entrambe finite insieme */
}
```

### Il ruolo del main

Il main serve per: leggere i dati, chiamare le funzioni, stampare i risultati.

```c
int main() {
    char s[100];
    printf("Inserisci una frase: ");
    fgets(s, 100, stdin);
    int risultato = contaMaiuscole(s);
    printf("Maiuscole: %d\n", risultato);
    return 0;
}
```

In generale è meglio evitare `printf` dentro una funzione. Una funzione dovrebbe fare un solo lavoro: calcolare o trasformare. La stampa è responsabilità del main. Separare queste due cose rende il programma più chiaro, più riutilizzabile, più facile da testare.

---

## Blocco 6: Strategie per risolvere gli esercizi sulle stringhe

Molti esercizi sulle stringhe sembrano diversi, ma appartengono quasi sempre a quattro famiglie di problemi. Quando vedi un esercizio, la prima cosa da fare è capire a quale famiglia appartiene.

Le quattro famiglie sono: **contare**, **trasformare**, **cercare**, **convertire**.

### Esercizi di tipo contare

Bisogna scorrere la stringa e contare quante volte succede qualcosa.

Esempi: contare le maiuscole, contare le vocali, contare gli spazi, contare la frequenza delle lettere, contare i caratteri speciali.

Strategia:
- inizializzare un contatore
- scorrere la stringa carattere per carattere
- controllare se il carattere soddisfa una condizione
- aumentare il contatore
- restituire il risultato

Schema mentale: **scorro la stringa → se il carattere soddisfa la condizione → aumento il contatore**

Varianti possibili: la consegna può cambiare solo la condizione — se è una vocale, se è uno spazio, se è un numero, se è un carattere alfabetico. La struttura dell'algoritmo rimane la stessa.

### Esercizi di tipo trasformare

Bisogna modificare i caratteri della stringa oppure creare una nuova stringa trasformata.

Esempi: convertire in maiuscolo, convertire in minuscolo, cifrare una stringa, sostituire un carattere, eliminare certi caratteri.

Strategia:
- scorrere la stringa
- controllare il carattere
- se serve trasformarlo
- scrivere il risultato nella nuova stringa (o nella stessa)

Schema mentale: **leggo un carattere → decido se modificarlo → scrivo il risultato**

Varianti possibili: sostituire le vocali con `*`, cifrare con uno spostamento, eliminare gli spazi, trasformare minuscole in maiuscole. La struttura resta sempre la stessa.

### Esercizi di tipo cercare

Bisogna verificare se qualcosa è presente nella stringa.

Esempi: verificare se una parola è contenuta in una frase, trovare la posizione di un carattere, controllare se due stringhe sono uguali, verificare se una stringa è palindroma.

Strategia:
- scorrere la stringa principale
- confrontare i caratteri
- verificare se la sequenza cercata appare
- restituire 1 oppure 0

Schema mentale: **parto da una posizione → confronto i caratteri → se coincidono fino alla fine → trovato**

Varianti possibili: verificare se una parola è presente, trovare la prima posizione di una lettera, verificare se due stringhe sono uguali. La logica è sempre confrontare caratteri.

### Esercizi di tipo convertire

Bisogna trasformare una stringa in un numero o in un altro formato.

Esempi: convertire un numero romano in intero, convertire una stringa numerica in un numero, interpretare un codice.

Strategia:
- scorrere la stringa
- riconoscere il simbolo corrente
- assegnare il valore numerico
- aggiornare il risultato finale

Schema mentale: **leggo simbolo → lo trasformo in numero → aggiorno il totale**

---

Quando leggi una consegna, chiediti:
- Devo contare qualcosa? → esercizio di conteggio
- Devo modificare la stringa? → esercizio di trasformazione
- Devo verificare se qualcosa è presente? → esercizio di ricerca
- Devo trasformare il testo in un numero? → esercizio di conversione

---

## Esercizio — Contare

Scrivere la funzione `int contaSpazi(const char s[])` che restituisce il numero di spazi presenti nella stringa `s`.
Scrivere anche un main che legge una frase, chiama la funzione e stampa il risultato.

Soluzione:

```c
#include <stdio.h>

int contaSpazi(const char s[]) {
    int i = 0, cont = 0;
    while (s[i] != '\0') {
        if (s[i] == ' ')
            cont++;
        i++;
    }
    return cont;
}

int main() {
    char frase[200];
    printf("Inserisci una frase: ");
    fgets(frase, sizeof(frase), stdin);
    printf("Numero di spazi: %d\n", contaSpazi(frase));
    return 0;
}
```

## Esercizio — Trasformare

Scrivere la funzione `void sostituisciVocali(const char s1[], char s2[])` che copia la stringa `s1` in `s2`, sostituendo tutte le vocali con `*`.
Scrivere anche un main che legge una parola, chiama la funzione e stampa la stringa trasformata.

Soluzione:

```c
#include <stdio.h>

void sostituisciVocali(const char s1[], char s2[]) {
    int i = 0;
    while (s1[i] != '\0') {
        char c = s1[i];
        if (c=='a'||c=='e'||c=='i'||c=='o'||c=='u'||
            c=='A'||c=='E'||c=='I'||c=='O'||c=='U')
            s2[i] = '*';
        else
            s2[i] = c;
        i++;
    }
    s2[i] = '\0';
}

int main() {
    char parola[100], risultato[100];
    printf("Inserisci una parola: ");
    fgets(parola, sizeof(parola), stdin);
    sostituisciVocali(parola, risultato);
    printf("Risultato: %s\n", risultato);
    return 0;
}
```

## Esercizio — Cercare

Scrivere la funzione `int contieneCarattere(const char s[], char c)` che restituisce 1 se il carattere `c` è presente nella stringa `s`, 0 altrimenti.
Scrivere anche un main che legge una parola e un carattere, chiama la funzione e stampa il risultato.

Soluzione:

```c
#include <stdio.h>

int contieneCarattere(const char s[], char c) {
    int i = 0;
    while (s[i] != '\0') {
        if (s[i] == c)
            return 1;
        i++;
    }
    return 0;
}

int main() {
    char parola[100];
    char c;
    printf("Inserisci una parola: ");
    fgets(parola, sizeof(parola), stdin);
    printf("Inserisci un carattere: ");
    scanf(" %c", &c);
    printf("Risultato: %d\n", contieneCarattere(parola, c));
    return 0;
}
```

## Esercizio — Convertire

Scrivere la funzione `int sommaCifre(const char s[])` che riceve una stringa contenente solo cifre numeriche e restituisce la somma delle cifre.
Scrivere anche un main che legge la stringa, chiama la funzione e stampa il risultato.

Soluzione:

```c
#include <stdio.h>

int sommaCifre(const char s[]) {
    int i = 0, somma = 0;
    while (s[i] != '\0') {
        somma += s[i] - '0';
        i++;
    }
    return somma;
}

int main() {
    char numero[100];
    printf("Inserisci un numero: ");
    fgets(numero, sizeof(numero), stdin);
    printf("Somma cifre: %d\n", sommaCifre(numero));
    return 0;
}
```

::: {.callout-note}
## Noterai che tutti e quattro gli esercizi hanno lo stesso cuore
```c
int i = 0;
while (s[i] != '\0') {
    /* lavoro su s[i] */
    i++;
}
```
Cambia solo cosa fai dentro il ciclo.
:::