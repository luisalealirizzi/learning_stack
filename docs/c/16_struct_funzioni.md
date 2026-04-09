# Struct e funzioni

---

## Idea chiave

Nella lezione precedente abbiamo visto come dichiarare una struct e usarla nel main. Ora vediamo come passarla alle funzioni — perche e qui che le struct diventano davvero utili.

Ci sono due modi per passare una struct a una funzione:

- **per valore** — la funzione riceve una copia
- **per puntatore** — la funzione riceve l'indirizzo e puo modificare l'originale

La scelta dipende da cosa deve fare la funzione.

---

## Blocco 1: Passare una struct per valore

Quando passi una struct per valore, la funzione riceve una **copia completa**. Le modifiche fatte dentro la funzione non si riflettono sull'originale.

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char nome[50];
    int eta;
    double media;
} Studente;

void stampa(Studente s) {
    printf("Nome:  %s\n",   s.nome);
    printf("Eta:   %d\n",   s.eta);
    printf("Media: %.2f\n", s.media);
}

int main() {
    Studente s;
    strcpy(s.nome, "Mario");
    s.eta = 20;
    s.media = 8.5;

    stampa(s);   /* passa una copia */
    return 0;
}
```

Usa il passaggio per valore quando la funzione deve solo **leggere** i dati della struct — non modificarli.

---

## Blocco 2: Passare una struct per puntatore

Quando passi una struct per puntatore, la funzione riceve l'indirizzo dell'originale e puo modificarlo direttamente.

```c
void aggiorna(Studente *p, double nuovaMedia) {
    p->media = nuovaMedia;   /* operatore -> per accedere tramite puntatore */
}

int main() {
    Studente s;
    strcpy(s.nome, "Mario");
    s.eta = 20;
    s.media = 8.5;

    aggiorna(&s, 9.0);   /* passa l'indirizzo */
    printf("%.2f\n", s.media);   /* stampa 9.00 */
    return 0;
}
```

### L'operatore ->

Quando lavori con un puntatore a struct, usi `->` invece di `.` per accedere ai campi:

```c
Studente s;
Studente *p = &s;

s.media = 8.5;    /* accesso diretto */
p->media = 8.5;   /* accesso tramite puntatore — equivalente */

(*p).media = 8.5; /* equivalente ma meno leggibile */
```

::: {.callout-note}
## Quando usare . e quando ->
La regola e semplice:
- hai una **variabile** struct → usa `.`
- hai un **puntatore** a struct → usa `->`
:::

---

## Blocco 3: Funzioni che restituiscono una struct

Una funzione puo anche **restituire** una struct. E utile quando vuoi costruire e inizializzare una struct in modo pulito.

```c
Studente creaStudente(const char nome[], int eta, double media) {
    Studente s;
    strcpy(s.nome, nome);
    s.eta = eta;
    s.media = media;
    return s;
}

int main() {
    Studente s1 = creaStudente("Alice", 19, 9.0);
    Studente s2 = creaStudente("Marco", 20, 7.5);

    stampa(s1);
    stampa(s2);
    return 0;
}
```

Questo pattern rende il codice molto piu leggibile — la creazione di una struct e incapsulata in una funzione dedicata.

---

## Blocco 4: Funzioni su array di struct

Quando hai un array di struct, le funzioni seguono lo stesso schema che hai gia visto con gli array normali — ma accedi ai campi con il punto.

### Schema tipico

```c
tipo funzione(Studente v[], int n) {
    int i;
    for (i = 0; i < n; i++) {
        /* lavoro su v[i].campo */
    }
}
```

### Funzione che stampa un array di struct

```c
void stampaClasse(Studente v[], int n) {
    int i;
    printf("%-20s %5s %6s\n", "Nome", "Eta", "Media");
    printf("-------------------------------\n");
    for (i = 0; i < n; i++) {
        printf("%-20s %5d %6.2f\n",
            v[i].nome,
            v[i].eta,
            v[i].media);
    }
}
```

### Funzione che trova il massimo

```c
int indiceMigliore(Studente v[], int n) {
    int i, indMax = 0;
    for (i = 1; i < n; i++) {
        if (v[i].media > v[indMax].media) {
            indMax = i;
        }
    }
    return indMax;
}

/* Uso nel main */
int idx = indiceMigliore(classe, N);
printf("Miglior studente: %s (%.2f)\n",
    classe[idx].nome,
    classe[idx].media);
```

### Funzione che conta

```c
int contaPromossi(Studente v[], int n, double soglia) {
    int i, cont = 0;
    for (i = 0; i < n; i++) {
        if (v[i].media >= soglia)
            cont++;
    }
    return cont;
}
```

### Funzione che cerca

```c
int cercaPerNome(Studente v[], int n, const char nome[]) {
    int i;
    for (i = 0; i < n; i++) {
        if (strcmp(v[i].nome, nome) == 0)
            return i;   /* restituisce la posizione */
    }
    return -1;          /* non trovato */
}
```

---

## Blocco 5: Esempio completo

```c
#include <stdio.h>
#include <string.h>

#define N 3

typedef struct {
    char nome[50];
    int matricola;
    double media;
} Studente;

/* Prototipi */
Studente creaStudente(const char nome[], int matricola, double media);
void stampaClasse(Studente v[], int n);
int  indiceMigliore(Studente v[], int n);
int  contaPromossi(Studente v[], int n, double soglia);

int main() {
    Studente classe[N];

    classe[0] = creaStudente("Alice",  1001, 9.0);
    classe[1] = creaStudente("Marco",  1002, 6.5);
    classe[2] = creaStudente("Sara",   1003, 8.0);

    stampaClasse(classe, N);

    int idx = indiceMigliore(classe, N);
    printf("\nMiglior studente: %s (%.2f)\n",
        classe[idx].nome, classe[idx].media);

    int promossi = contaPromossi(classe, N, 6.0);
    printf("Promossi: %d su %d\n", promossi, N);

    return 0;
}

Studente creaStudente(const char nome[], int matricola, double media) {
    Studente s;
    strcpy(s.nome, nome);
    s.matricola = matricola;
    s.media = media;
    return s;
}

void stampaClasse(Studente v[], int n) {
    int i;
    printf("%-20s %10s %6s\n", "Nome", "Matricola", "Media");
    printf("--------------------------------------\n");
    for (i = 0; i < n; i++) {
        printf("%-20s %10d %6.2f\n",
            v[i].nome, v[i].matricola, v[i].media);
    }
}

int indiceMigliore(Studente v[], int n) {
    int i, indMax = 0;
    for (i = 1; i < n; i++) {
        if (v[i].media > v[indMax].media)
            indMax = i;
    }
    return indMax;
}

int contaPromossi(Studente v[], int n, double soglia) {
    int i, cont = 0;
    for (i = 0; i < n; i++) {
        if (v[i].media >= soglia)
            cont++;
    }
    return cont;
}
```

Output:
```
Nome                 Matricola  Media
--------------------------------------
Alice                     1001   9.00
Marco                     1002   6.50
Sara                      1003   8.00

Miglior studente: Alice (9.00)
Promossi: 3 su 3
```

---

## Schema mentale

```
La funzione deve solo leggere la struct?
    -> passa per valore: void stampa(Studente s)

La funzione deve modificare la struct?
    -> passa per puntatore: void aggiorna(Studente *p)
    -> usa -> per accedere ai campi: p->campo
    -> chiama con &: aggiorna(&s)

La funzione lavora su un array di struct?
    -> void funzione(Studente v[], int n)
    -> accedi con v[i].campo
    -> stessi schemi degli array: contare, trovare, cercare

La funzione crea e restituisce una struct?
    -> Studente creaStudente(parametri)
    -> costruisci la struct, return s
```

---

## Esercizi

**Esercizio 1**
Dichiara una struct `Prodotto` con nome, prezzo e quantita. Scrivi le funzioni:
- `Prodotto creaProdotto(...)` — crea e restituisce un prodotto
- `void stampaProdotto(Prodotto p)` — stampa i dati
- `void applicaSconto(Prodotto *p, double percentuale)` — riduce il prezzo

**Esercizio 2**
Usando la struct `Prodotto`, scrivi le funzioni:
- `int cercaProdotto(Prodotto v[], int n, const char nome[])` — restituisce l'indice, -1 se non trovato
- `Prodotto prodottoPiuCaro(Prodotto v[], int n)` — restituisce il prodotto con prezzo massimo
- `int contaDisponibili(Prodotto v[], int n)` — conta i prodotti con quantita > 0

**Esercizio 3**
Dichiara una struct `Punto` con coordinate `x` e `y`. Scrivi:
- `double distanza(Punto a, Punto b)` — distanza euclidea tra due punti
- `Punto puntoMedio(Punto a, Punto b)` — restituisce il punto medio
- `void trasla(Punto *p, double dx, double dy)` — sposta il punto

**Esercizio 4**
Scrivi un programma completo che:
- legge N prodotti dalla tastiera usando `creaProdotto`
- salva tutti i prodotti su file con `fprintf`
- rilegge i prodotti dal file
- stampa quello con il prezzo piu alto
