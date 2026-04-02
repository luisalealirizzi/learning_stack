# Eccezioni

---

## Cosa succede quando qualcosa va storto?

Fino ad ora abbiamo scritto codice che funziona, assumendo che i dati in ingresso siano sempre validi, che i file esistano, che la rete sia disponibile. Ma nella realtà non è così.

Un utente inserisce una lettera dove si aspetta un numero. Un file non esiste. La memoria finisce. La connessione cade.

Questi sono **errori a runtime**, situazioni anomale che si verificano durante l'esecuzione del programma, non durante la compilazione. Java gestisce queste situazioni con un meccanismo chiamato **gestione delle eccezioni**.

---

## Cos'è un'eccezione

Un'**eccezione** è un oggetto che rappresenta una situazione anomala verificatasi durante l'esecuzione. Quando si verifica un errore, Java **lancia** (throws) un'eccezione e interrompe il flusso normale e cerca qualcuno che la gestisca.

Se nessuno la gestisce, il programma termina stampando un messaggio di errore chiamato **stack trace**:

```
Exception in thread "main" java.lang.ArithmeticException: / by zero
    at Main.divide(Main.java:8)
    at Main.main(Main.java:3)
```

Lo stack trace dice: cosa è successo (`ArithmeticException: / by zero`), dove è successo (`Main.java:8`), e come ci siamo arrivati (la sequenza di chiamate).

---

## La gerarchia delle eccezioni

In Java le eccezioni sono classi organizzate in una gerarchia, Error ed Exception derivano entrambe dalla classe Throwable

```
Throwable
├── Error                    → errori gravi della JVM (non gestire)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception                → situazioni anomale gestibili
    ├── RuntimeException     → eccezioni non verificate (unchecked)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── ClassCastException
    │   ├── ArithmeticException
    │   └── NumberFormatException
    └── IOException          → eccezioni verificate (checked)
        ├── FileNotFoundException
        └── ...
```

::: {.callout-note}
## Checked vs Unchecked
**Unchecked** (RuntimeException e sottoclassi): il compilatore non ti obbliga a gestirle. Spesso indicano bug nel codice: accesso a un indice fuori range, divisione per zero, cast errato.

**Checked** (tutto il resto di Exception) — il compilatore ti **obbliga** a gestirle o a dichiararle. Tipicamente sono situazioni esterne al controllo del programma: file non trovato, errore di rete.
:::

---

## Le eccezioni più comuni

Prima di vedere come gestirle, impariamo a riconoscerle:

**`NullPointerException`** — accesso a un riferimento `null`:
```java
String s = null;
s.length();  // NullPointerException!
```

**`ArrayIndexOutOfBoundsException`** — indice fuori dai limiti dell'array:
```java
int[] arr = new int[3];
arr[5] = 10;  // ArrayIndexOutOfBoundsException!
```

**`ClassCastException`** — downcasting errato:
```java
Personaggio p = new Mago(...);
Guerriero g = (Guerriero) p;  // ClassCastException!
```

**`ArithmeticException`** — operazione aritmetica illegale:
```java
int risultato = 10 / 0;  // ArithmeticException: / by zero
```

**`NumberFormatException`** — conversione di stringa non valida:
```java
int n = Integer.parseInt("abc");  // NumberFormatException!
```

**`StackOverflowError`** — ricorsione infinita:
```java
void infinita() { infinita(); }  // StackOverflowError!
```

---

## Try-catch: gestire le eccezioni

Il blocco **`try-catch`** permette di intercettare un'eccezione e gestirla senza che il programma si blocchi:

```java
try {
    // codice che potrebbe lanciare un'eccezione
} catch (TipoEccezione e) {
    // cosa fare se quell'eccezione si verifica
}
```

Esempio concreto — divisione sicura:

```java
public static int dividi(int a, int b) {
    try {
        return a / b;
    } catch (ArithmeticException e) {
        System.out.println("Errore: divisione per zero!");
        return 0;
    }
}
```

Esempio con input utente:

```java
import java.util.Scanner;

Scanner sc = new Scanner(System.in);
System.out.print("Inserisci un numero: ");

try {
    int n = Integer.parseInt(sc.nextLine());
    System.out.println("Hai inserito: " + n);
} catch (NumberFormatException e) {
    System.out.println("Quello non è un numero valido!");
}
```

---

## Blocchi multipli catch

Puoi gestire più tipi di eccezione con più blocchi `catch`:

```java
try {
    String[] nomi = {"Arthas", "Gandalf"};
    String input = sc.nextLine();
    int indice = Integer.parseInt(input);
    System.out.println("Personaggio: " + nomi[indice]);

} catch (NumberFormatException e) {
    System.out.println("Inserisci un numero intero.");
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Indice fuori range! Scegli tra 0 e 1.");
} catch (Exception e) {
    // cattura qualsiasi altra eccezione
    System.out.println("Errore inaspettato: " + e.getMessage());
}
```

::: {.callout-important}
## Ordine dei catch
I blocchi `catch` vengono valutati dall'alto verso il basso — il primo che corrisponde viene eseguito. Metti sempre i tipi più specifici prima di quelli più generici. Se metti `Exception` per primo, cattura tutto e i blocchi successivi non vengono mai eseguiti.
:::

---

## Il blocco finally

Il blocco **`finally`** viene eseguito **sempre** — sia che l'eccezione si verifichi, sia che non si verifichi. È utile per operazioni di pulizia: chiudere file, connessioni, scanner.

```java
Scanner sc = new Scanner(System.in);

try {
    System.out.print("Vita del personaggio: ");
    int vita = Integer.parseInt(sc.nextLine());
    System.out.println("Vita impostata: " + vita);

} catch (NumberFormatException e) {
    System.out.println("Valore non valido!");

} finally {
    sc.close();  // viene eseguito sempre, eccezione o no
    System.out.println("Scanner chiuso.");
}
```

::: {.callout-note}
## Try-with-resources
Da Java 7 in poi puoi usare il **try-with-resources** per chiudere automaticamente le risorse senza `finally`:

```java
try (Scanner sc = new Scanner(System.in)) {
    int n = Integer.parseInt(sc.nextLine());
    System.out.println("Numero: " + n);
} catch (NumberFormatException e) {
    System.out.println("Non è un numero!");
}
// sc viene chiuso automaticamente
```
:::

---

## Lanciare eccezioni con throw

Non solo puoi gestire le eccezioni, puoi anche **lanciarle** tu stesso con `throw`. Questo è utile nei setter e nei costruttori quando ricevi valori non validi:

```java
public void setVita(int vita) {
    if (vita < 0) {
        throw new IllegalArgumentException("La vita non può essere negativa: " + vita);
    }
    if (vita > vitaMax) {
        throw new IllegalArgumentException("La vita non può superare vitaMax: " + vita);
    }
    this.vita = vita;
}
```

Chi chiama questo metodo deve gestire l'eccezione (o lasciarla propagare):

```java
try {
    eroe.setVita(-100);
} catch (IllegalArgumentException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

---

## Creare eccezioni personalizzate

Puoi creare le tue eccezioni estendendo `Exception` (checked) o `RuntimeException` (unchecked):

```java
// Eccezione personalizzata per il gioco
public class PersonaggioMortoException extends RuntimeException {

    public PersonaggioMortoException(String nomePersonaggio) {
        super("Il personaggio " + nomePersonaggio + " è morto e non può agire!");
    }
}
```

Usarla:

```java
public void attacca(Personaggio bersaglio) {
    if (isMorto()) {
        throw new PersonaggioMortoException(getNome());
    }
    bersaglio.subirDanno(attacco);
}
```

```java
try {
    fantasma.attacca(eroe);
} catch (PersonaggioMortoException e) {
    System.out.println(e.getMessage());
}
```

::: {.callout-tip}
## Quando creare eccezioni personalizzate
Crea eccezioni personalizzate quando vuoi dare un nome significativo a un errore specifico del tuo dominio. `PersonaggioMortoException` è molto più chiara di un generico `IllegalStateException`.
:::

---

## Propagare eccezioni con throws

Se un metodo può lanciare una **checked exception** ma non la gestisce internamente, deve propagarla nella firma con `throws`:

```java
public void caricaDaFile(String percorso) throws IOException {
    // questo metodo può lanciare IOException
    // chi lo chiama deve gestirla
}
```

Chi chiama il metodo è obbligato a gestirla:

```java
try {
    caricaDaFile("salvataggio.txt");
} catch (IOException e) {
    System.out.println("File non trovato: " + e.getMessage());
}
```

::: {.callout-note}
## `throw` vs `throws`
- **`throw`**  lancia effettivamente un'eccezione all'interno del codice
- **`throws`** dichiara nella firma del metodo che potrebbe lanciare quell'eccezione
:::

---

## Esempio completo: input robusto

Mettiamo tutto insieme con un esempio realistico: un programma che chiede la vita del personaggio e non si blocca su input errati:

```java
import java.util.Scanner;

public class InputRobusto {

    public static int chiediInteroPositivo(Scanner sc, String messaggio) {
        while (true) {
            System.out.print(messaggio);
            try {
                int valore = Integer.parseInt(sc.nextLine().trim());
                if (valore <= 0) {
                    System.out.println("Il valore deve essere positivo. Riprova.");
                    continue;
                }
                return valore;
            } catch (NumberFormatException e) {
                System.out.println("Inserisci un numero intero valido. Riprova.");
            }
        }
    }

    public static void main(String[] args) {
        try (Scanner sc = new Scanner(System.in)) {

            System.out.println("=== CREA IL TUO PERSONAGGIO ===\n");

            System.out.print("Nome: ");
            String nome = sc.nextLine().trim();

            int vitaMax  = chiediInteroPositivo(sc, "Vita massima: ");
            int attacco  = chiediInteroPositivo(sc, "Attacco: ");
            int livello  = chiediInteroPositivo(sc, "Livello: ");

            System.out.println("\nPersonaggio creato:");
            System.out.println("Nome:    " + nome);
            System.out.println("Vita:    " + vitaMax);
            System.out.println("Attacco: " + attacco);
            System.out.println("Livello: " + livello);
        }
    }
}
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave 

- Un'**eccezione** è un oggetto che rappresenta una situazione anomala a runtime.
- Le eccezioni **unchecked** (`RuntimeException`) indicano spesso bug nel codice — il compilatore non obbliga a gestirle.
- Le eccezioni **checked** (`Exception`) riguardano situazioni esterne — il compilatore obbliga a gestirle.
- Il blocco **`try-catch`** intercetta le eccezioni e permette di gestirle.
- Il blocco **`finally`** viene eseguito sempre — utile per chiudere risorse.
- Il **try-with-resources** chiude automaticamente le risorse al termine del blocco.
- **`throw`** lancia un'eccezione; **`throws`** la dichiara nella firma del metodo.
- Puoi creare **eccezioni personalizzate** estendendo `Exception` o `RuntimeException`.
:::