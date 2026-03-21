# Stringhe in input in C


## Obiettivo

Saper leggere correttamente una frase da input e capire quando è necessario gestire il newline.


## Schema base

Devi saper fare senza esitazione questo schema:

```c
char s[200];
fgets(s, sizeof(s), stdin);

// se dobbiamo eliminare l'andata a capo
s[strcspn(s, "\n")] = '\0';
```

## Cose da sapere bene
fgets legge anche gli spazi
salva anche \n, che in alcuni casi va eliminato
la stringa finisce comunque con '\0'

### Quando NON è necessario togliere \n

Non serve eliminarlo quando scorriamo la stringa carattere per carattere.

Il \n verrà semplicemente trattato come un carattere qualsiasi.

Esempio
```c
int i = 0;

while (s[i] != '\0') {
    // lavoro su s[i]
    i++;
}
```
Questo approccio va bene negli esercizi in cui:
si contano caratteri (es: frequenza dei caratteri)
si analizza il testo (es: conteggio vocali)
si modificano solo alcune lettere (es: crittografia delle sole lettere)
si ignorano i caratteri non alfabetici

In questi casi il \n non crea problemi

### Quando invece bisogna togliere \n

È necessario eliminarlo quando vogliamo che la stringa rappresenti esattamente il testo inserito, senza l’invio finale.

🔹Confronto tra stringhe strcmp(s, "ciao");
    Se la stringa è: "ciao\n" il confronto fallisce

🔹 Calcolo della lunghezza reale
    Il newline verrebbe contato come carattere in più.

🔹 Elaborazioni sensibili alla posizione
    palindromi
    parsing preciso del testo
    confronti esatti tra stringhe

Esempio completo
```c
#include <stdio.h>
#include <string.h>

int main() {
    char s[200];

    printf("Inserisci una frase: ");
    fgets(s, sizeof(s), stdin);

    // rimuovo il newline
    s[strcspn(s, "\n")] = '\0';

    printf("Hai scritto: %s\n", s);

    return 0;
}

```
## Schema mentale
leggo la stringa
controllo se devo togliere \n
poi la uso

## Mini esercizio

Scrivi un programma che:

legge una frase
stampa la lunghezza della stringa

prova prima senza togliere \n
poi togli il newline. cosa cambia?