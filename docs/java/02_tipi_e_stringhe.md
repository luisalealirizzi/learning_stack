# Tipi, stringhe e input

---

## Tipi primitivi e oggetti

Java fa una distinzione netta tra due categorie di dati: i **tipi primitivi** e gli **oggetti**.

I **tipi primitivi** sono i mattoni fondamentali del linguaggio: numeri, caratteri, valori logici. Sono scritti in minuscolo, occupano una quantità fissa di memoria e contengono direttamente il loro valore.

Le **classi** descrivono oggetti complessi con dati e metodi. Si scrivono con l'iniziale maiuscola — proprio per segnare la differenza visiva rispetto ai tipi primitivi.

::: {.callout-note}
## La differenza fondamentale in memoria
Quando assegni un valore a una variabile primitiva, memorizzi **direttamente quel dato**.

Quando assegni un oggetto, memorizzi un **riferimento** — un indirizzo che punta a dove l'oggetto si trova in memoria.

```java
int vita = 100;          // il valore 100 è direttamente nella variabile
Personaggio eroe = ...;  // eroe contiene un indirizzo, non l'oggetto
```
:::

---

## Gli 8 tipi primitivi

Java ha **8 tipi primitivi**, divisi in quattro categorie:

| Categoria | Tipo | Dimensione | Intervallo / Note |
|---|---|---|---|
| **Interi** | `byte` | 8 bit | da -128 a 127 |
| | `short` | 16 bit | da -32.768 a 32.767 |
| | `int` | 32 bit | da -2 miliardi a +2 miliardi — il più usato |
| | `long` | 64 bit | numeri molto grandi, si scrive con `L` finale |
| **Reali** | `float` | 32 bit | precisione singola, si scrive con `f` finale |
| | `double` | 64 bit | precisione doppia — il più usato per i decimali |
| **Caratteri** | `char` | 16 bit | un singolo carattere Unicode, tra apici singoli |
| **Booleani** | `boolean` | — | solo `true` o `false` |

```java
int livello = 5;
long punteggio = 9_999_999_999L;  // L obbligatorio per i long grandi
double velocita = 3.14;
float peso = 72.5f;               // f obbligatorio per i float
char iniziale = 'A';
boolean inVita = true;
```

::: {.callout-tip}
## Quando usare int e quando long?
Nella maggior parte dei casi `int` è sufficiente. Usa `long` solo quando sai che il numero può superare circa 2 miliardi — ad esempio un punteggio globale online, un timestamp in millisecondi, o l'ID di un database.
:::

---

## Valori di default

Quando dichiari un attributo di una classe senza assegnare un valore, Java lo inizializza automaticamente:

| Tipo | Valore di default |
|---|---|
| `int`, `short`, `byte`, `long` | `0` |
| `float`, `double` | `0.0` |
| `char` | `'\u0000'` (carattere vuoto) |
| `boolean` | `false` |
| Qualsiasi oggetto | `null` |

::: {.callout-warning}
## Attenzione alle variabili locali
I valori di default valgono solo per gli **attributi di una classe**. Le variabili locali (dentro un metodo) **non** vengono inizializzate automaticamente — il compilatore ti dà errore se provi a usarle senza averle prima assegnato.

```java
int x;
System.out.println(x); // ERRORE: variable x might not have been initialized
```
:::

---

## Casting: convertire tra tipi

A volte devi convertire un tipo in un altro. Java distingue due casi:

**Casting implicito**  da un tipo "piccolo" a uno "grande", senza perdita di dati. Java lo fa automaticamente:

```java
int n = 42;
double d = n;  // automatico: int → double, nessun problema
```

**Casting esplicito**  da un tipo "grande" a uno "piccolo", con possibile perdita di dati. Devi dirlo esplicitamente con le parentesi:

```java
double punteggio = 97.8;
int puntiInteri = (int) punteggio;  // diventa 97 — la parte decimale viene troncata!
```

::: {.callout-important}
## Il casting tronca, non arrotonda
`(int) 97.8` dà `97`, non `98`. Se vuoi arrotondare usa `Math.round()`:

```java
long arrotondato = Math.round(97.8);  // dà 98
```
:::

---

## La classe String

`String` non è un tipo primitivo — è una **classe**. Si scrive con la maiuscola e ogni stringa è un oggetto a tutti gli effetti, con i propri metodi.

```java
String nomeGiocatore = "Arthas";
String messaggio = "Livello " + 5;  // concatenazione con +
```

Puoi chiedere a una stringa di fare cose:

```java
String s = "Hello, World!";

int lunghezza = s.length();           // 13
String maiuscolo = s.toUpperCase();   // "HELLO, WORLD!"
String minuscolo = s.toLowerCase();   // "hello, world!"
boolean inizia = s.startsWith("He"); // true
boolean contiene = s.contains("World"); // true
String rimpiazza = s.replace("World", "Java"); // "Hello, Java!"
String parte = s.substring(7, 12);   // "World"
int posizione = s.indexOf("World");  // 7
String pulita = "  spazi  ".trim();  // "spazi"
```

::: {.callout-note}
## Perché String con la maiuscola?
Per distinguerla visivamente dai tipi primitivi. In Java la convenzione è:
- **minuscolo** → tipo primitivo (`int`, `double`, `boolean`...)
- **Maiuscola** → classe o oggetto (`String`, `Scanner`, `Personaggio`...)
:::

---

## Confrontare le stringhe: `==` vs `equals()`

Questo è uno degli errori più comuni per chi viene dal C.

```java
String a = "ciao";
String b = "ciao";

// NON fare così:
if (a == b) { ... }      // confronta i riferimenti, non il contenuto!

// Fare così:
if (a.equals(b)) { ... } // confronta il contenuto — questo è corretto
```

`==` confronta se due variabili puntano **allo stesso oggetto in memoria**. `equals()` confronta se il **contenuto** delle due stringhe è uguale.

::: {.callout-warning}
## Regola d'oro per le stringhe
Per confrontare stringhe usa **sempre** `equals()`, mai `==`.

Se vuoi confrontare ignorando maiuscole/minuscole:
```java
if (a.equalsIgnoreCase(b)) { ... }  // "Ciao" == "ciao" → true
```
:::

---

## Il package `java.lang`

Java ha una libreria standard enorme. Il package **`java.lang`** è quello di base — viene importato automaticamente in ogni programma, senza bisogno di scrivere `import`.

Le classi più utili:

| Classe | A cosa serve |
|---|---|
| `String` | Gestione testi |
| `Math` | Funzioni matematiche |
| `Integer`, `Double` | Convertire tipi primitivi in oggetti (wrapper) |
| `Object` | La superclasse di tutte le classi Java |

Qualche metodo utile di `Math`:

```java
Math.abs(-5)          // valore assoluto → 5
Math.max(10, 20)      // massimo → 20
Math.min(10, 20)      // minimo → 10
Math.pow(2, 10)       // potenza → 1024.0
Math.sqrt(16)         // radice quadrata → 4.0
Math.round(3.7)       // arrotondamento → 4
Math.random()         // numero casuale tra 0.0 e 1.0
```

Per generare un numero intero casuale tra 0 e N:
```java
int casuale = (int)(Math.random() * N);
```

---

## Le classi wrapper

Ogni tipo primitivo ha una corrispondente **classe wrapper** con la maiuscola:

| Primitivo | Wrapper |
|---|---|
| `int` | `Integer` |
| `double` | `Double` |
| `boolean` | `Boolean` |
| `char` | `Character` |

Le wrapper sono utili soprattutto per convertire tra tipi:

```java
// Da stringa a numero
int n = Integer.parseInt("42");
double d = Double.parseDouble("3.14");

// Da numero a stringa
String s = Integer.toString(42);
String s2 = String.valueOf(3.14);

// Costanti utili
int max = Integer.MAX_VALUE;  // 2147483647
int min = Integer.MIN_VALUE;  // -2147483648
```

::: {.callout-tip}
## Autoboxing
Java converte automaticamente tra primitivi e wrapper quando necessario — questa si chiama **autoboxing**:
```java
Integer x = 42;   // autoboxing: int → Integer
int y = x;        // unboxing: Integer → int
```
Non devi farlo a mano, Java lo gestisce da solo.
:::

---

## Input da tastiera con Scanner

Per leggere input dall'utente si usa la classe `Scanner`, che si trova nel package `java.util` — quindi va importata:

```java
import java.util.Scanner;

public class InputEsempio {
    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);

        System.out.print("Inserisci il tuo nome: ");
        String nome = sc.nextLine();

        System.out.print("Inserisci la tua età: ");
        int eta = sc.nextInt();

        System.out.println("Ciao " + nome + ", hai " + eta + " anni.");

        sc.close();
    }
}
```

I metodi principali di `Scanner`:

| Metodo | Legge |
|---|---|
| `nextLine()` | Una riga intera (String) |
| `next()` | Una parola (String) |
| `nextInt()` | Un intero |
| `nextDouble()` | Un decimale |
| `nextBoolean()` | Un booleano |

::: {.callout-warning}
## Il problema del `nextLine()` dopo `nextInt()`
`nextInt()` legge il numero ma lascia nel buffer il carattere `\n` (invio). Se subito dopo chiami `nextLine()`, legge quella riga vuota invece di aspettare l'input.

La soluzione: aggiungi un `sc.nextLine()` extra per "consumare" il carattere rimasto.

```java
int eta = sc.nextInt();
sc.nextLine();          // consuma il \n rimasto nel buffer
String nome = sc.nextLine();  // ora legge correttamente
```
:::

---

## Esempio completo: scheda personaggio

Mettiamo tutto insieme con un esempio concreto:

```java
import java.util.Scanner;

public class Scheda {
    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);

        System.out.println("=== CREA IL TUO PERSONAGGIO ===\n");

        System.out.print("Nome: ");
        String nome = sc.nextLine();

        System.out.print("Classe (guerriero/mago/arciere): ");
        String classe = sc.nextLine();

        System.out.print("Livello iniziale: ");
        int livello = sc.nextInt();

        // Calcola vita iniziale in base al livello
        int vita = livello * 20 + 80;

        System.out.println("\n--- SCHEDA PERSONAGGIO ---");
        System.out.println("Nome:    " + nome.toUpperCase());
        System.out.println("Classe:  " + classe);
        System.out.println("Livello: " + livello);
        System.out.println("Vita:    " + vita);
        System.out.println("Pronto per l'avventura!");

        sc.close();
    }
}
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave

- Java ha **8 tipi primitivi**: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`.
- I tipi primitivi contengono direttamente il valore; gli oggetti contengono un **riferimento**.
- Il **widening casting** è automatico; il **narrowing casting** richiede le parentesi esplicite.
- `String` è una **classe**, non un tipo primitivo — si scrive con la maiuscola e ha metodi propri.
- Per confrontare stringhe usa sempre **`equals()`**, mai `==`.
- Il package **`java.lang`** (String, Math, Integer...) è importato automaticamente.
- Per leggere input da tastiera si usa **`Scanner`** da `java.util`.
:::