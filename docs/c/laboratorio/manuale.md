# Manuale sintetico — Linguaggio C

---

## Struttura base

```c
#include <stdio.h>

int main() {
    /* codice */
    return 0;
}
```

Librerie comuni: `<stdio.h>` `<string.h>` `<math.h>` `<stdlib.h>` `<time.h>` `<stdbool.h>`

---

## Tipi e variabili

```c
int n = 5;
double x = 3.14;
char c = 'A';
bool b = true;          /* richiede <stdbool.h> */

#define MAX 100         /* costante */
```

---

## Input e output

```c
printf("Testo\n");
printf("Intero: %d, Decimale: %.2f, Char: %c\n", n, x, c);

scanf("%d", &n);
scanf("%lf", &x);
scanf(" %c", &c);       /* spazio prima di %c */

char s[100];
fgets(s, sizeof(s), stdin);                  /* legge riga intera */
s[strcspn(s, "\n")] = '\0';                  /* rimuove newline */
```

| Tipo | printf | scanf |
|---|---|---|
| `int` | `%d` | `%d` |
| `double` | `%.2f` | `%lf` |
| `char` | `%c` | ` %c` |
| stringa | `%s` | `%s` |

---

## Operatori

```c
/* Aritmetici */
+ - * / %

/* Relazionali */
== != < > <= >=

/* Logici */
&& || !

/* Assegnazione */
= += -= *= /= %=
i++   i--

/* Casting */
(double)a / b
```

---

## Selezione

```c
if (condizione) {
    /* ... */
} else if (condizione) {
    /* ... */
} else {
    /* ... */
}

switch (n) {
    case 1: /* ... */ break;
    case 2: /* ... */ break;
    default: break;
}

/* Operatore ternario */
int max = (a > b) ? a : b;
```

---

## Cicli

```c
/* for — numero di ripetizioni noto */
for (i = 0; i < n; i++) { /* ... */ }

/* while — condizione */
while (condizione) { /* ... */ }

/* do-while — almeno una volta */
do { /* ... */ } while (condizione);

break;      /* esci dal ciclo */
continue;   /* prossima iterazione */
```

---

## Funzioni

```c
/* Prototipo — va prima del main */
int somma(int a, int b);
void stampa(int n);

/* Definizione — va dopo il main */
int somma(int a, int b) {
    return a + b;
}

void stampa(int n) {
    printf("%d\n", n);
}

/* Funzione con puntatore — modifica la variabile originale */
void raddoppia(int *p) {
    *p = (*p) * 2;
}
/* Chiamata: raddoppia(&x); */
```

---

## Array

```c
#define N 5
int v[N] = {1, 2, 3, 4, 5};

/* Lettura */
for (i = 0; i < N; i++)
    scanf("%d", &v[i]);

/* Stampa */
for (i = 0; i < N; i++)
    printf("%d ", v[i]);

/* Passare a una funzione */
void funzione(int v[], int n) { /* ... */ }
/* Chiamata: funzione(v, N); */

/* Matrice */
int m[3][4];
for (i = 0; i < 3; i++)
    for (j = 0; j < 4; j++)
        m[i][j] = 0;

/* Passare matrice a funzione */
void f(int m[][4], int righe) { /* ... */ }
```

---

## Stringhe

```c
char s[100];

/* Scorrere */
int i = 0;
while (s[i] != '\0') {
    /* lavoro su s[i] */
    i++;
}

/* Riconoscere caratteri */
s[i] >= 'A' && s[i] <= 'Z'   /* maiuscola */
s[i] >= 'a' && s[i] <= 'z'   /* minuscola */
s[i] >= '0' && s[i] <= '9'   /* cifra */
s[i] - '0'                    /* cifra -> numero */

/* Funzioni (richiedono <string.h>) */
strlen(s)           /* lunghezza */
strcpy(dst, src)    /* copia */
strcmp(s1, s2)      /* confronto — 0 se uguali */
strstr(s1, s2)      /* cerca s2 in s1 */
strcspn(s, "\n")    /* posizione del \n */
```

---

## Puntatori

```c
int x = 10;
int *p = &x;      /* p punta a x */
*p = 20;          /* modifica x tramite p */

printf("%d\n", *p);    /* valore puntato */
printf("%p\n", p);     /* indirizzo */

/* Array e puntatori */
int v[5] = {1,2,3,4,5};
int *p = v;            /* p punta al primo elemento */
*(v + i) == v[i]       /* equivalenti */
```

---

## Struct

```c
typedef struct {
    char nome[50];
    int eta;
    double media;
} Studente;

Studente s;
strcpy(s.nome, "Mario");
s.eta = 20;
s.media = 8.5;

/* Array di struct */
Studente classe[30];
classe[0].eta = 18;

/* Passare a funzione */
void stampa(Studente s) { printf("%s\n", s.nome); }
```

---

## File

```c
/* Lettura */
FILE *f = fopen("dati.txt", "r");
if (f == NULL) { printf("Errore\n"); return 1; }

int n;
while (fscanf(f, "%d", &n) == 1) { /* ... */ }   /* parola per parola */

char riga[200];
while (fgets(riga, sizeof(riga), f) != NULL) { /* ... */ }   /* riga per riga */

fclose(f);

/* Scrittura */
FILE *f = fopen("output.txt", "w");    /* sovrascrive */
FILE *f = fopen("log.txt",    "a");    /* aggiunge in fondo */
fprintf(f, "Valore: %d\n", n);
fclose(f);
```

---

## Numeri casuali

```c
#include <stdlib.h>
#include <time.h>

srand(time(NULL));              /* inizializza — una volta sola nel main */
int r = rand() % 10;           /* numero tra 0 e 9 */
int r = rand() % 10 + 1;       /* numero tra 1 e 10 */
int r = rand() % (max - min + 1) + min;   /* tra min e max */
```

---

## Compilazione ed esecuzione

```bash
gcc programma.c -o programma     # compila
./programma                       # esegui
gcc programma.c -o programma -lm  # con libreria math.h
```
