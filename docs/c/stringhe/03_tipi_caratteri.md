# Riconoscere i tipi di carattere in C


## Obiettivo

Saper distinguere i diversi tipi di caratteri all’interno di una stringa:

- lettere maiuscole  
- lettere minuscole  
- caratteri non alfabetici  

Questo è fondamentale per quasi tutti gli esercizi sulle stringhe.


## Schema base

Durante lo scorrimento della stringa, devi saper scrivere condizioni come queste:

```c
if (s[i] >= 'A' && s[i] <= 'Z') {
    // lettera maiuscola
} else if (s[i] >= 'a' && s[i] <= 'z') {
    // lettera minuscola
} else {
    // altro carattere
}
```
Idea fondamentale

Ogni carattere in C è in realtà un numero (codifica ASCII).
Quindi puoi confrontarlo come se fosse un numero.

Intervalli importanti
Tipo	Intervallo
Maiuscole	'A' → 'Z'
Minuscole	'a' → 'z'
Numeri	'0' → '9'

Esempio: distinguere i caratteri
```c
#include <stdio.h>

int main() {
    char s[100];
    int i = 0;

    fgets(s, sizeof(s), stdin);

    while (s[i] != '\0') {

        if (s[i] >= 'A' && s[i] <= 'Z') {
            printf("%c è maiuscola\n", s[i]);
        } else if (s[i] >= 'a' && s[i] <= 'z') {
            printf("%c è minuscola\n", s[i]);
        } else {
            printf("%c non è una lettera\n", s[i]);
        }

        i++;
    }

    return 0;
}
```
### Casi tipici
✔ Lettere
```c
if ((s[i] >= 'A' && s[i] <= 'Z') || (s[i] >= 'a' && s[i] <= 'z')) {
    // è una lettera
}
```
✔ Numeri
```c
if (s[i] >= '0' && s[i] <= '9') {
    // è una cifra
}
```
✔ Spazio
```c
if (s[i] == ' ') {
    // è uno spazio
}
```
✔ Caratteri speciali

Tutto ciò che non rientra nei casi precedenti:
```c
else {
    // carattere speciale
}
```

Errori tipici
Confondere caratteri e numeri
```c
if (s[i] == 5)   // SBAGLIATO
```
Devi scrivere:
```c
if (s[i] == '5')   // CORRETTO
```
Dimenticare le virgolette
```c
if (s[i] == a)   // SBAGLIATO
```
Corretto:
```c
if (s[i] == 'a')
```
### Esempio: contare le maiuscole
```c
#include <stdio.h>

int main() {
    char s[100];
    int i = 0;
    int cont = 0;

    fgets(s, sizeof(s), stdin);

    while (s[i] != '\0') {
        if (s[i] >= 'A' && s[i] <= 'Z') {
            cont++;
        }
        i++;
    }

    printf("Numero di maiuscole: %d\n", cont);

    return 0;
}
``` 
### Esempio: contare lettere e numeri
```c
#include <stdio.h>

int main() {
    char s[100];
    int i = 0;
    int lettere = 0;
    int numeri = 0;

    fgets(s, sizeof(s), stdin);

    while (s[i] != '\0') {

        if ((s[i] >= 'A' && s[i] <= 'Z') || (s[i] >= 'a' && s[i] <= 'z')) {
            lettere++;
        } else if (s[i] >= '0' && s[i] <= '9') {
            numeri++;
        }

        i++;
    }

    printf("Lettere: %d\n", lettere);
    printf("Numeri: %d\n", numeri);

    return 0;
}
```

## Schema mentale

Quando lavori sulle stringhe:

scorri la stringa
guarda il carattere s[i]
decidi a quale categoria appartiene
applica la logica