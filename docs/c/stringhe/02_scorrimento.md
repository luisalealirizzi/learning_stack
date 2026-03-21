# Scorrimento di una stringa in C


## Obiettivo

Saper scorrere una stringa carattere per carattere fino alla fine.

Questo è il cuore di quasi tutti gli esercizi sulle stringhe.  
Se non sai fare questo, non puoi risolvere bene esercizi di conteggio, trasformazione, ricerca o crittografia.


## Schema base

Devi saper fare senza esitazione questo schema:

```c
int i = 0;

while (s[i] != '\0') {
    // lavoro su s[i]
    i++;
}
```

Idea fondamentale

Una stringa in C è un array di caratteri che termina con il carattere speciale: '\0'

Quindi, per scorrerla tutta:

parti da indice 0
guardi un carattere alla volta
continui finché non trovi '\0'

## Schema mentale

Devi pensare così: leggo un carattere alla volta finché trovo '\0'

Cosa rappresenta s[i]
s è la stringa
i è la posizione
s[i] è il carattere nella posizione i

Esempio: 

Se la stringa è: casa

allora in memoria hai:

Indice	Contenuto
0	'c'
1	'a'
2	's'
3	'a'
4	'\0'


### Esempio minimo
```c
#include <stdio.h>

int main() {
    char s[] = "ciao";
    int i = 0;

    while (s[i] != '\0') {
        printf("%c\n", s[i]);
        i++;
    }

    return 0;
}
```
Cosa fa questo programma?

Stampa un carattere per volta:

c
i
a
o


###Esempio: contare i caratteri
```c
#include <stdio.h>

int main() {
    char s[100];
    int i = 0;
    int cont = 0;

    fgets(s, sizeof(s), stdin);

    while (s[i] != '\0') {
        cont++;
        i++;
    }

    printf("Numero di caratteri letti: %d\n", cont);

    return 0;
}
```
Attenzione al newline: se hai letto la stringa con fgets, potrebbe esserci anche \n.
Quindi questo programma potrebbe contare anche l’invio finale.

Per questo motivo, quando serve una lunghezza “pulita”, bisogna prima eliminare \n.

### Esempio: contare le vocali
```c
#include <stdio.h>

int main() {
    char s[100];
    int i = 0;
    int cont = 0;

    fgets(s, sizeof(s), stdin);

    while (s[i] != '\0') {
        if (s[i] == 'a' || s[i] == 'e' || s[i] == 'i' || s[i] == 'o' || s[i] == 'u' ||
            s[i] == 'A' || s[i] == 'E' || s[i] == 'I' || s[i] == 'O' || s[i] == 'U') {
            cont++;
        }
        i++;
    }

    printf("Numero di vocali: %d\n", cont);

    return 0;
}
```
Cosa devi riconoscere in tutti questi esercizi: la parte che cambia è la logica dentro il ciclo.

La struttura invece resta sempre questa:

```c
int i = 0;

while (s[i] != '\0') {
    // qui cambia il lavoro da fare
    i++;
}
```

### Schema generale

Quasi tutti gli esercizi sulle stringhe seguono questa idea:

inizializza i = 0
scorri la stringa finché non trovi '\0'
fai qualcosa su s[i]
incrementa i

### Esempi di esercizi che usano questo schema
contare le vocali, maiuscole e spazi...
cercare un carattere
sostituire lettere
cifrare una stringa
controllare certe condizioni sui caratteri