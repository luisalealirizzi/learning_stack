# Struct

---

## Idea chiave

Fino ad ora abbiamo usato variabili separate per memorizzare dati correlati:

```c
char nome[50];
int eta;
double altezza;
```

Se abbiamo dieci persone, servono trenta variabili. E se vogliamo passarle a una funzione, dobbiamo passare trenta parametri.

Una **struct** raggruppa piu variabili correlate in un unico tipo. E come creare un tipo di dato su misura.

```c
struct Persona {
    char nome[50];
    int eta;
    double altezza;
};
```

Ora `Persona` e un tipo come `int` o `double` — puoi dichiarare variabili di quel tipo, passarle alle funzioni, metterle in array.

---

## Blocco 1: Dichiarare una struct

```c
struct NomeStruct {
    tipo campo1;
    tipo campo2;
    /* ... */
};
```

Esempio:

```c
struct Studente {
    char nome[50];
    int matricola;
    double media;
};
```

::: {.callout-important}
## Il punto e virgola finale
La dichiarazione di una struct termina con `;` dopo la graffa chiusa. E uno degli errori piu comuni dimenticarlo.

```c
struct Studente {
    char nome[50];
    int matricola;
};        /* <- punto e virgola obbligatorio */
```
:::

### typedef — un nome piu comodo

Con `typedef` puoi dare un nome alternativo alla struct e usarla senza scrivere `struct` ogni volta:

```c
typedef struct {
    char nome[50];
    int matricola;
    double media;
} Studente;
```

Da questo momento puoi scrivere `Studente` invece di `struct Studente`:

```c
struct Studente s1;   /* senza typedef */
Studente s1;          /* con typedef — piu comodo */
```

---

## Blocco 2: Dichiarare e inizializzare una variabile struct

```c
Studente s;                                    /* dichiarazione */
Studente s = {"Mario Rossi", 12345, 8.5};      /* inizializzazione */
```

### Accedere ai campi con il punto

Si usa l'operatore `.` per accedere ai singoli campi:

```c
Studente s;

strcpy(s.nome, "Mario Rossi");   /* copia una stringa nel campo nome */
s.matricola = 12345;
s.media = 8.5;

printf("Nome: %s\n",     s.nome);
printf("Matricola: %d\n", s.matricola);
printf("Media: %.2f\n",   s.media);
```

::: {.callout-note}
## Perche strcpy per le stringhe?
Non puoi assegnare una stringa con `=` in C. Devi usare `strcpy` dalla libreria `string.h`.

```c
s.nome = "Mario";          /* ERRORE */
strcpy(s.nome, "Mario");   /* corretto */
```
:::

### Esempio completo

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char nome[50];
    int matricola;
    double media;
} Studente;

int main() {
    Studente s;

    printf("Nome: ");
    fgets(s.nome, sizeof(s.nome), stdin);
    s.nome[strcspn(s.nome, "\n")] = '\0';

    printf("Matricola: ");
    scanf("%d", &s.matricola);

    printf("Media: ");
    scanf("%lf", &s.media);

    printf("\n--- SCHEDA STUDENTE ---\n");
    printf("Nome:      %s\n",   s.nome);
    printf("Matricola: %d\n",   s.matricola);
    printf("Media:     %.2f\n", s.media);

    return 0;
}
```

---

## Blocco 3: Array di struct

Il vero vantaggio delle struct emerge quando devi gestire piu entita dello stesso tipo. Invece di tante variabili separate, usi un array di struct.

```c
Studente classe[30];   /* array di 30 studenti */
```

### Accesso agli elementi

```c
classe[0].matricola = 1001;
strcpy(classe[0].nome, "Alice");
classe[0].media = 8.5;

printf("%s: %.2f\n", classe[0].nome, classe[0].media);
```

### Leggere un array di struct

```c
#define N 5

Studente classe[N];
int i;

for (i = 0; i < N; i++) {
    printf("Studente %d\n", i + 1);

    printf("  Nome: ");
    fgets(classe[i].nome, sizeof(classe[i].nome), stdin);
    classe[i].nome[strcspn(classe[i].nome, "\n")] = '\0';

    printf("  Matricola: ");
    scanf("%d", &classe[i].matricola);

    printf("  Media: ");
    scanf("%lf", &classe[i].media);

    /* consuma il newline rimasto nel buffer */
    getchar();
}
```

### Stampare un array di struct

```c
for (i = 0; i < N; i++) {
    printf("%-20s  %5d  %.2f\n",
        classe[i].nome,
        classe[i].matricola,
        classe[i].media);
}
```

---

## Blocco 4: Cercare e confrontare struct

Con un array di struct puoi applicare gli stessi schemi che hai visto con gli array normali.

### Trovare la media piu alta

```c
int indiceMax = 0;
for (i = 1; i < N; i++) {
    if (classe[i].media > classe[indiceMax].media) {
        indiceMax = i;
    }
}
printf("Miglior studente: %s (%.2f)\n",
    classe[indiceMax].nome,
    classe[indiceMax].media);
```

### Contare quanti hanno la sufficienza

```c
int promossi = 0;
for (i = 0; i < N; i++) {
    if (classe[i].media >= 6.0) {
        promossi++;
    }
}
printf("Promossi: %d su %d\n", promossi, N);
```

### Cercare per matricola

```c
int cerca = 12345;
int trovato = -1;

for (i = 0; i < N; i++) {
    if (classe[i].matricola == cerca) {
        trovato = i;
        break;
    }
}

if (trovato != -1) {
    printf("Trovato: %s\n", classe[trovato].nome);
} else {
    printf("Matricola non trovata.\n");
}
```

Schema mentale: **gli stessi schemi degli array — cambia solo che accedi ai campi con il punto**.

---

## Blocco 5: Struct e file

Le struct si combinano naturalmente con i file — puoi salvare e ricaricare i dati di piu entita.

### Salvare un array di struct su file

```c
FILE *f = fopen("studenti.txt", "w");
if (f == NULL) { return 1; }

for (i = 0; i < N; i++) {
    fprintf(f, "%s;%d;%.2f\n",
        classe[i].nome,
        classe[i].matricola,
        classe[i].media);
}

fclose(f);
```

Il file `studenti.txt` conterra:
```
Alice;1001;8.50
Marco;1002;7.30
Sara;1003;9.10
```

### Leggere struct da file

```c
FILE *f = fopen("studenti.txt", "r");
if (f == NULL) { return 1; }

Studente classe[N];
int n = 0;

while (fscanf(f, "%49[^;];%d;%lf\n",
    classe[n].nome,
    &classe[n].matricola,
    &classe[n].media) == 3) {
    n++;
}

fclose(f);
printf("Letti %d studenti.\n", n);
```

::: {.callout-note}
## Il formato %49[^;]
`%49[^;]` legge fino a 49 caratteri o fino al primo `;` — serve per leggere stringhe che contengono spazi ma sono delimitate da un separatore.
:::

---

## Schema mentale

```
Devo raggruppare dati correlati?
    -> usa una struct
    -> typedef struct { campi; } NomeTipo;

Devo accedere a un campo?
    -> variabile.campo

Devo gestire molte entita dello stesso tipo?
    -> array di struct: NomeTipo v[N];
    -> accesso: v[i].campo

Devo cercare, trovare il massimo, contare?
    -> stessi schemi degli array
    -> confronta v[i].campo con la condizione

Devo salvare su file?
    -> fprintf con separatore: fprintf(f, "%s;%d\n", s.nome, s.matricola)
```

---

## Esercizi

**Esercizio 1**
Dichiara una struct `Prodotto` con i campi: `nome` (stringa), `prezzo` (double), `quantita` (int). Scrivi un programma che legge 5 prodotti dalla tastiera e stampa quello con il prezzo piu alto.

**Esercizio 2**
Usando la struct `Prodotto` dell'esercizio 1, scrivi un programma che:
- legge 5 prodotti dalla tastiera
- salva tutti i prodotti su file `magazzino.txt`
- rilegge il file e stampa solo i prodotti con quantita maggiore di 0

**Esercizio 3**
Dichiara una struct `Punto` con i campi `x` e `y` (double). Scrivi una funzione `double distanza(Punto a, Punto b)` che calcola la distanza euclidea tra due punti. Scrivila nel main che legge due punti e stampa la loro distanza.

**Esercizio 4**
Dichiara una struct `Studente` con nome, matricola e array di 5 voti (int). Scrivi un programma che legge 3 studenti, calcola la media dei voti per ognuno e stampa il nome dello studente con la media piu alta.
