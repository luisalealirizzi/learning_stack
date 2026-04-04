# Selezione

---

## Prendere decisioni

Fino ad ora i nostri programmi eseguono sempre le stesse istruzioni, dall'alto verso il basso, senza mai cambiare percorso. Ma nella realtà un programma deve saper **prendere decisioni**: fare una cosa se una condizione è vera, un'altra se è falsa.

Le strutture di selezione servono esattamente a questo — permettono al programma di scegliere quale blocco di istruzioni eseguire in base a una condizione.

---

## if — se

La struttura più semplice è `if`. Esegue un blocco di istruzioni solo se la condizione è vera.

```c
if (condizione) {
    /* istruzioni eseguite solo se la condizione è vera */
}
```

```c
int voto = 7;

if (voto >= 6) {
    printf("Promosso!\n");
}
```

Se `voto` è 7, la condizione `voto >= 6` è vera e viene stampato `Promosso!`. Se `voto` fosse 4, la condizione sarebbe falsa e `printf` non verrebbe eseguita.

---

## if-else — se... altrimenti

Quando vuoi fare una cosa se la condizione è vera e un'altra se è falsa, usi `if-else`.

```c
if (condizione) {
    /* eseguito se la condizione è vera */
} else {
    /* eseguito se la condizione è falsa */
}
```

```c
int voto = 4;

if (voto >= 6) {
    printf("Promosso!\n");
} else {
    printf("Bocciato.\n");
}
```

Uno dei due blocchi viene sempre eseguito — mai entrambi, mai nessuno.

---

## if-else if-else — catena di condizioni

Quando hai più casi da gestire, puoi concatenare più condizioni con `else if`.

```c
if (condizione1) {
    /* eseguito se condizione1 è vera */
} else if (condizione2) {
    /* eseguito se condizione1 è falsa e condizione2 è vera */
} else if (condizione3) {
    /* eseguito se le prime due sono false e condizione3 è vera */
} else {
    /* eseguito se tutte le condizioni sono false */
}
```

```c
int voto;
printf("Inserisci il voto: ");
scanf("%d", &voto);

if (voto >= 9) {
    printf("Ottimo\n");
} else if (voto >= 7) {
    printf("Buono\n");
} else if (voto >= 6) {
    printf("Sufficiente\n");
} else {
    printf("Insufficiente\n");
}
```

::: {.callout-note}
## Come funziona la catena
Le condizioni vengono valutate dall'alto verso il basso. Appena una è vera, viene eseguito quel blocco e le altre vengono ignorate — anche se sarebbero vere anch'esse.

Per questo l'ordine conta: nell'esempio sopra, se il voto è 9, la prima condizione `voto >= 9` è vera e si stampa `Ottimo`. Non viene più controllato se `voto >= 7` — anche se sarebbe vero.

Scrivi sempre le condizioni dalla più restrittiva alla meno restrittiva.
:::

---

## Le graffe sono obbligatorie?

Tecnicamente, se il blocco contiene una sola istruzione, le graffe si possono omettere:

```c
if (voto >= 6)
    printf("Promosso!\n");   /* funziona */
```

Ma è una cattiva abitudine. Se aggiungi una seconda istruzione dimenticandoti le graffe, il programma non fa quello che ti aspetti:

```c
if (voto >= 6)
    printf("Promosso!\n");
    printf("Complimenti!\n");  /* viene eseguita SEMPRE, non solo se promosso */
```

::: {.callout-important}
## Metti sempre le graffe
Anche quando il blocco ha una sola istruzione, usa sempre le graffe. Il codice è più leggibile e meno soggetto a errori quando lo modifichi.

```c
if (voto >= 6) {
    printf("Promosso!\n");
}
```
:::

---

## Condizioni composte

Puoi combinare più condizioni con gli operatori logici `&&` e `||`.

```c
int eta = 17;
int consenso = 1;   /* 1 = sì, 0 = no */

/* AND — entrambe le condizioni devono essere vere */
if (eta >= 18 && consenso == 1) {
    printf("Accesso consentito\n");
}

/* OR — almeno una condizione deve essere vera */
if (eta < 13 || eta > 65) {
    printf("Tariffa ridotta\n");
}

/* Range — il valore deve essere tra due estremi */
if (voto >= 6 && voto <= 10) {
    printf("Voto valido\n");
}
```

---

## if annidati

Puoi mettere un `if` dentro un altro `if`. Si chiamano **if annidati**.

```c
int voto;
char comportamento;

if (voto >= 6) {
    printf("Promosso in termini di voto.\n");
    if (comportamento == 'O') {   /* O = Ottimo */
        printf("Menzione speciale per il comportamento!\n");
    }
} else {
    printf("Bocciato.\n");
}
```

::: {.callout-tip}
## Non annidare troppo
Più livelli di annidamento rendono il codice difficile da leggere. Se ti trovi con tre o quattro livelli, probabilmente c'è un modo più chiaro di scrivere la stessa logica.

Regola pratica: oltre due livelli di annidamento, fermati e rifletti.
:::

---

## switch — selezionare tra casi precisi

Quando devi confrontare una variabile con molti valori precisi, `switch` è più leggibile di una lunga catena di `if-else if`.

```c
switch (variabile) {
    case valore1:
        /* istruzioni */
        break;
    case valore2:
        /* istruzioni */
        break;
    default:
        /* eseguito se nessun caso corrisponde */
        break;
}
```

```c
char classe;
printf("Inserisci la classe (G/M/A): ");
scanf(" %c", &classe);

switch (classe) {
    case 'G':
        printf("Sei un Guerriero.\n");
        break;
    case 'M':
        printf("Sei un Mago.\n");
        break;
    case 'A':
        printf("Sei un Arciere.\n");
        break;
    default:
        printf("Classe non riconosciuta.\n");
        break;
}
```

::: {.callout-warning}
## Non dimenticare il break
Senza `break`, dopo aver eseguito un caso il programma continua a eseguire i casi successivi — anche se non corrispondono. Questo si chiama **fall-through** ed è quasi sempre un errore.

```c
switch (voto) {
    case 10:
        printf("Ottimo\n");
        /* manca break — continua nel caso 9! */
    case 9:
        printf("Buono\n");
        break;
}
/* Se voto è 10, stampa sia "Ottimo" che "Buono" */
```

Metti sempre `break` alla fine di ogni caso, incluso `default`.
:::

### switch vs if-else if

| | `switch` | `if-else if` |
|---|---|---|
| **Confronta** | valori precisi (`==`) | qualsiasi condizione |
| **Tipi** | `int`, `char` | qualsiasi tipo |
| **Leggibilità** | molto chiaro con molti casi | preferibile con pochi casi o range |

Usa `switch` quando confronti una variabile con valori precisi e hai molti casi. Usa `if-else if` quando le condizioni coinvolgono range, operatori `<`, `>`, o tipi `double`.

---

## L'operatore ternario

L'operatore ternario è una forma compatta di `if-else` che restituisce un valore.

```c
condizione ? valore_se_vera : valore_se_falsa
```

```c
int voto = 7;

/* Con if-else */
if (voto >= 6) {
    printf("Promosso\n");
} else {
    printf("Bocciato\n");
}

/* Con operatore ternario — equivalente */
printf("%s\n", voto >= 6 ? "Promosso" : "Bocciato");
```

Usalo solo per casi semplici — quando la condizione e i valori sono brevi e chiari. Per logica complessa, usa `if-else`.

---

## Esempio completo: classificatore di temperature

```c
#include <stdio.h>

int main() {
    double temperatura;

    printf("Inserisci la temperatura: ");
    scanf("%lf", &temperatura);

    printf("Temperatura: %.1f°C → ", temperatura);

    if (temperatura < 0) {
        printf("Sotto zero — ghiaccio!\n");
    } else if (temperatura < 10) {
        printf("Molto freddo.\n");
    } else if (temperatura < 20) {
        printf("Fresco.\n");
    } else if (temperatura < 30) {
        printf("Ideale.\n");
    } else if (temperatura < 40) {
        printf("Caldo.\n");
    } else {
        printf("Molto caldo!\n");
    }

    /* Avviso separato con condizione composta */
    if (temperatura >= 37.0 && temperatura <= 38.5) {
        printf("Attenzione: possibile febbre.\n");
    }

    return 0;
}
```

---

## Schema mentale

```
Devo eseguire qualcosa solo se una condizione è vera?
    → if (condizione) { ... }

Devo fare una cosa o l'altra?
    → if (condizione) { ... } else { ... }

Ho più di due casi?
    → if ... else if ... else if ... else
    → ordina dalla condizione più restrittiva alla meno restrittiva

Confronto un valore con molti valori precisi (int o char)?
    → switch — più leggibile
    → non dimenticare break in ogni caso

Ho una condizione semplice e voglio un valore?
    → operatore ternario: condizione ? a : b
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che legge un numero intero e stampa se è positivo, negativo o zero.

**Esercizio 2**
Scrivi un programma che legge un voto da 0 a 10 e stampa il giudizio:
- 9-10: Ottimo
- 7-8: Buono
- 6: Sufficiente
- 4-5: Insufficiente
- 0-3: Gravemente insufficiente
- altro: valore non valido

**Esercizio 3**
Scrivi un programma che legge il numero del mese (1-12) e stampa il nome del mese e il numero di giorni che ha. Usa `switch`. Per febbraio stampa "28 o 29 giorni".

**Esercizio 4**
Scrivi un programma che legge tre numeri interi e stampa il maggiore tra i tre.