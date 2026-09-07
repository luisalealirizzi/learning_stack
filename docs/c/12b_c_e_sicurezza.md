# C e sicurezza informatica

*Approfondimento opzionale*

---

## Perché il C è centrale nella sicurezza

Il C è il linguaggio più vicino alla macchina tra quelli ad alto livello. Questa vicinanza è la sua forza — ma è anche la fonte delle sue vulnerabilità più pericolose.

In C il programmatore ha il controllo diretto della memoria: decide dove allocare i dati, come accedervi, quando liberarli. Nessun altro linguaggio moderno ti dà questo controllo. Ma questo significa anche che se sbagli, le conseguenze possono essere gravi e sfruttabili da un attaccante.

Chi lavora in cybersecurity deve conoscere il C per due motivi:
- capire **come funzionano gli attacchi** a basso livello
- saper **leggere e analizzare** codice vulnerabile

---

## Idea chiave: il buffer overflow

Un **buffer overflow** è una delle vulnerabilità più antiche e più sfruttate nella storia della sicurezza informatica. Nasce da un errore semplice: scrivere più dati di quanti un array possa contenere.

Hai già visto questo concetto quando abbiamo parlato degli array, abbiamo detto che accedere oltre i limiti è un errore. Il buffer overflow è esattamente questo errore, sfruttato intenzionalmente da un attaccante.

---

## Blocco 1: Come funziona un buffer overflow

Considera questo codice:

```c
#include <stdio.h>
#include <string.h>

void leggiNome() {
    char nome[10];   /* buffer di 10 caratteri */
    printf("Inserisci il tuo nome: ");
    gets(nome);      /* PERICOLOSO — legge senza limiti */
    printf("Ciao, %s!\n", nome);
}

int main() {
    leggiNome();
    return 0;
}
```

`gets` legge caratteri dalla tastiera e li scrive in `nome` senza mai controllare quanti ne arrivano. Se l'utente scrive 50 caratteri invece di 10, i 40 caratteri in eccesso vengono scritti **oltre i confini dell'array** in zone di memoria che appartengono ad altre variabili o al sistema.

### Cosa succede in memoria

Quando una funzione viene chiamata, il sistema crea uno **stack frame**, una zona di memoria che contiene le variabili locali e l'indirizzo di ritorno (dove tornare dopo la funzione).

```
STACK FRAME di leggiNome():
┌─────────────────────┐
│ nome[0]  ... nome[9]│  ← buffer — 10 byte
├─────────────────────┤
│ altre variabili     │  ← qui arrivano i dati in eccesso
├─────────────────────┤
│ indirizzo di ritorno│  ← se sovrascritto, il programma va dove vuole l'attaccante
└─────────────────────┘
```

Se l'attaccante scrive abbastanza dati da raggiungere l'**indirizzo di ritorno** e lo sovrascrive con un indirizzo scelto da lui, il programma continuerà l'esecuzione da quel punto eseguendo codice arbitrario.

::: {.callout-important}
## gets è stata rimossa dal C moderno
La funzione `gets` è così pericolosa che è stata rimossa dallo standard C11. Il compilatore moderno ti avvisa quando la usi. Non usarla mai: usa sempre `fgets` con il limite di dimensione.

```c
gets(nome);                          /* PERICOLOSO — mai usare */
fgets(nome, sizeof(nome), stdin);    /* sicuro */
```
:::

---

## Blocco 2: Un esempio concreto e didattico

Vediamo un programma volutamente vulnerabile per capire cosa succede:

```c
#include <stdio.h>
#include <string.h>

int main() {
    int autorizzato = 0;
    char password[8];

    printf("Inserisci la password: ");
    scanf("%s", password);   /* non controlla la lunghezza */

    if (strcmp(password, "segreto") == 0) {
        autorizzato = 1;
    }

    if (autorizzato) {
        printf("Accesso consentito!\n");
    } else {
        printf("Accesso negato.\n");
    }

    return 0;
}
```

Questo programma sembra corretto. Ma considera la disposizione in memoria:

```
MEMORIA LOCALE:
┌─────────────┐
│ password[8] │  ← 8 byte per la password
├─────────────┤
│ autorizzato │  ← subito dopo in memoria
└─────────────┘
```

Se l'utente inserisce una stringa più lunga di 8 caratteri, `scanf` continua a scrivere oltre `password` sovrascrivendo `autorizzato`. Se `autorizzato` diventa diverso da 0, la condizione `if (autorizzato)` è vera e l'accesso è consentito anche senza la password corretta.

> **Prova tu**: cosa succede se inserisci esattamente 9 caratteri come password (uno in più della dimensione dell'array)? E se ne inserisci 20? Prova a immaginare, byte per byte, cosa finisce scritto in `autorizzato` prima ancora che il programma controlli la condizione.

::: {.callout-note}
## Questo è solo a scopo didattico
L'esempio mostra il principio. Nella realtà moderna i sistemi operativi hanno protezioni come ASLR (Address Space Layout Randomization) e stack canaries che rendono questi attacchi molto più difficili ma non impossibili.
:::

---

## Blocco 3: Altre vulnerabilità legate al C

Il buffer overflow è la più famosa, ma non è l'unica.

### Format string attack

`printf` con una stringa controllata dall'utente è pericolosa:

```c
char input[100];
fgets(input, sizeof(input), stdin);

printf(input);       /* PERICOLOSO */
printf("%s", input); /* sicuro */
```

Se l'utente inserisce `"%x %x %x"`, `printf` interpreta quei segnaposto e legge dati dallo stack — rivelando indirizzi di memoria o valori di variabili.

> **Prova tu**: nella prima riga di `printf` qui sopra, cosa succederebbe se un utente inserisse `%s%s%s%s` invece di un nome normale? `printf` andrebbe a cercare degli argomenti che non esistono: da dove pensi che li leggerebbe?

### Use-after-free

Usare memoria dopo averla liberata: un errore tipico con `malloc` e `free` che può portare a comportamenti imprevedibili e sfruttabili.

### Integer overflow

```c
unsigned int a = 4294967295;   /* valore massimo di unsigned int */
a = a + 1;                     /* diventa 0 — overflow */
```

Se questo valore viene usato come dimensione di un buffer, il buffer viene allocato troppo piccolo e i dati in eccesso sovrascrivono memoria.

> **Prova tu**: se una funzione calcola `dimensione = numero_elementi * sizeof(int)` e `numero_elementi` è un valore molto grande fornito dall'utente, cosa può succedere al valore di `dimensione` prima ancora che venga chiamata `malloc`?

---

## Blocco 4: Come si difende un programmatore

Conoscere le vulnerabilità serve per evitarle. Ecco le regole pratiche:

**Usa sempre funzioni con limite di dimensione**
```c
gets(s);                          /* mai */
fgets(s, sizeof(s), stdin);       /* sempre */

scanf("%s", s);                   /* pericoloso */
scanf("%99s", s);                 /* meglio — limita a 99 caratteri */
```

**Controlla sempre i limiti degli array**
```c
if (indice >= 0 && indice < N) {
    v[indice] = valore;   /* accesso sicuro */
}
```

**Valida sempre l'input dell'utente**
Non fidarti mai di quello che arriva dall'esterno (un file, la tastiera, la rete), controlla lunghezza, formato, range.

**Usa `strcmp` per confrontare stringhe**
```c
if (s1 == s2) { }              /* SBAGLIATO — confronta indirizzi */
if (strcmp(s1, s2) == 0) { }   /* corretto — confronta contenuto */
```

::: {.callout-tip}
## Strumenti usati dai professionisti
Chi lavora in sicurezza usa strumenti specifici per trovare vulnerabilità nel codice C:

- **valgrind** — rileva accessi fuori bounds e memory leak
- **AddressSanitizer** — compila con `-fsanitize=address` per rilevare buffer overflow a runtime
- **gdb** — debugger per analizzare il comportamento del programma in memoria
- **static analyzers** — come `cppcheck` o `clang-tidy` per trovare bug senza eseguire il codice
:::

---

## Blocco 5: Il C nel mondo della sicurezza informatica

Conoscere il C apre porte specifiche nel mondo della cybersecurity:

**Vulnerability research**
Ricerca di vulnerabilità in software esistente: molti programmi critici sono scritti in C (kernel Linux, OpenSSL, browser).

**Penetration testing**
I penetration tester scrivono exploit in C quando devono interagire direttamente con la memoria di un processo target.

**Reverse engineering**
Analizzare un programma senza il codice sorgente. Il codice compilato viene tradotto in assembly, chi conosce il C capisce molto più facilmente cosa sta succedendo.

**Sviluppo di malware e antivirus**
Molti malware sono scritti in C perché producono eseguibili piccoli e veloci. I ricercatori che li analizzano devono capire C.

**Sicurezza embedded e IoT**
Router, telecamere, sistemi industriali girano quasi sempre su C. Le vulnerabilità IoT sono una delle aree più attive della ricerca in sicurezza.

---

## Schema mentale

```
Perché il C è pericoloso?
    → il programmatore gestisce la memoria direttamente
    → nessun controllo automatico sui limiti degli array

Cos'è un buffer overflow?
    → scrittura oltre i confini di un array
    → sovrascrive memoria adiacente — variabili, indirizzi di ritorno

Come si previene?
    → usa fgets invece di gets
    → usa limiti espliciti con scanf: "%99s"
    → valida sempre l'input
    → controlla gli indici prima di accedere agli array

Perché studiarlo?
    → capire gli attacchi per difendersi
    → base per vulnerability research e penetration testing
```

---

## Per approfondire

Se questo argomento ti interessa, ecco da dove partire:

- **Progetto**: prova a compilare il codice vulnerabile di questo capitolo con `gcc -fsanitize=address` e osserva cosa rileva
- **Strumento**: installa `valgrind` e analizza un tuo programma C
- **Percorso**: piattaforme come **picoCTF** e **HackTheBox** hanno sfide che partono esattamente da queste vulnerabilità
- **Lettura**: cerca "smashing the stack for fun and profit", l'articolo storico del 1996 che ha cambiato la sicurezza informatica