# Stringhe avanzate

---

## String è immutabile

Nella lezione sui tipi abbiamo visto che `String` è una classe. Ma ha una caratteristica fondamentale che spesso sorprende: le stringhe in Java sono **immutabili** — una volta create, non possono essere modificate.

Quando sembra che stai modificando una stringa, in realtà stai creando un nuovo oggetto:

```java
String s = "ciao";
s = s + " mondo";
```

Cosa succede realmente:
1. viene creata una **nuova stringa** `"ciao mondo"`
2. i caratteri della vecchia stringa vengono copiati nel nuovo oggetto
3. la vecchia stringa `"ciao"` resta in memoria finché il garbage collector non la elimina

Ogni concatenazione crea quindi una nuova allocazione, una copia dei dati e uno spreco di tempo e memoria.

---

## Il problema delle concatenazioni in ciclo

L'immutabilità diventa un problema serio quando concateni in un ciclo:

```java
String testo = "";
for (int i = 0; i < 10000; i++) {
    testo = testo + i;  // crea un nuovo oggetto ad ogni iterazione!
}
```

Ad ogni iterazione viene creata una nuova stringa e vengono ricopiati tutti i caratteri precedenti. La complessità cresce rapidamente: **effetto valanga**.

Con 10.000 iterazioni questo codice crea 10.000 oggetti `String` intermedi, quasi tutti subito scartati. È lento e spreca memoria.

---

## StringBuilder: il buffer delle stringhe

`StringBuilder` risolve questo problema. Funziona come un **buffer in memoria**:

- mantiene un array interno modificabile
- aggiunge caratteri senza ricreare tutto
- rialloca solo quando necessario

```java
// Approccio SBAGLIATO
String testo = "";
for (int i = 0; i < 10000; i++) {
    testo = testo + i;
}

// Approccio CORRETTO
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);
}
String risultato = sb.toString();  // converte in String solo alla fine
```

Un solo oggetto, nessuna copia continua, prestazioni molto migliori.


## Metodi principali di StringBuilder

```java
StringBuilder sb = new StringBuilder("Arthas");

// append() — aggiunge in fondo
sb.append(" il Re dei Lich");     // "Arthas il Re dei Lich"
sb.append(42);                     // funziona con qualsiasi tipo

// insert() — inserisce in posizione specifica
sb.insert(0, "Lord ");            // "Lord Arthas il Re dei Lich"

// delete() — rimuove una porzione
sb.delete(0, 5);                  // rimuove "Lord "

// replace() — sostituisce una porzione
sb.replace(0, 6, "Cavaliere");

// reverse() — inverte tutta la stringa
sb.reverse();

// length() e charAt()
int len = sb.length();
char c = sb.charAt(0);

// setCharAt() — modifica un carattere
sb.setCharAt(0, 'A');

// toString() — converte in String
String finale = sb.toString();
```
---

::: {.callout-note}
## Quando usare StringBuilder, quando String
Usa **`StringBuilder`** quando concateni dentro cicli, crei testo lungo o fai parsing riga per riga.

Usa **`String`** quando hai poche concatenazioni, lavori con stringhe costanti, o hai bisogno di codice semplice e leggibile.
:::

---

## Lo String Pool

Dopo aver capito `StringBuilder`, è utile capire un altro meccanismo interno di Java: lo **String Pool**.

Lo String Pool è una zona speciale della memoria in cui Java conserva le stringhe letterali per evitare duplicazioni inutili.

```java
String a = "ciao";
String b = "ciao";
```

In questo caso Java **non** crea due oggetti distinti. Esiste un solo oggetto nello String Pool, e sia `a` che `b` puntano allo stesso riferimento:

```java
System.out.println(a == b);      // true: stesso riferimento!
System.out.println(a.equals(b)); // true: stesso contenuto
```

Attenzione però a `new String()`:

```java
String a = "ciao";
String b = new String("ciao");

System.out.println(a == b);      // false: oggetti diversi!
System.out.println(a.equals(b)); // true: contenuto uguale
```

`new String()` crea sempre un nuovo oggetto nell'heap, anche se il contenuto esiste già nel pool.

::: {.callout-note}
## Perché esiste lo String Pool?
Per efficienza: riduce l'uso della memoria ed evita la creazione continua di oggetti identici. Immagina migliaia di stringhe `"OK"` o `"ERROR"` in un programma — senza pool avremmo migliaia di copie uguali in memoria.

Lo String Pool funziona **solo perché le stringhe sono immutabili**. Se una stringa potesse cambiare, modificare `a` cambierebbe anche `b` dato che condividono lo stesso oggetto.
:::

---

## Stack, Heap e String Pool: dove vivono gli oggetti

Per capire davvero cosa succede con le stringhe, è utile vedere come Java organizza la memoria durante l'esecuzione.

Java utilizza tre zone principali:

```
┌─────────────────────────────────────────┐
│                  HEAP                   │
│  ┌───────────────────────────────────┐  │
│  │          STRING POOL              │  │
│  │   [ "java" ] [ "ciao" ] ...      │  │
│  └───────────────────────────────────┘  │
│  [ oggetti ]  [ array ]  [ StringBuilder]│
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│                  STACK                  │
│   a → riferimento                       │
│   b → riferimento                       │
│   sb → riferimento                      │
└─────────────────────────────────────────┘
```

**Stack** — contiene variabili locali dei metodi, riferimenti agli oggetti e parametri. È molto veloce, gestito automaticamente, segue la logica "entra/esci" dei metodi. Nota: nello stack non vive la stringa, ma solo il riferimento ad essa.

**Heap** — contiene oggetti, array, `StringBuilder` e tutto ciò che si crea con `new`. È più grande ma più lento dello stack.

**String Pool** — zona speciale dell'heap dedicata alle stringhe letterali.

---

## Simulazione della memoria

```java
String a = "java";
String b = "java";
String c = new String("java");
```

**Passo 1** — `String a = "java"`: Java controlla lo String Pool. Non c'è ancora `"java"`, quindi lo crea.

```
STRING POOL: [ "java" ]
STACK: a → pool["java"]
```

**Passo 2** — `String b = "java"`: Java controlla lo String Pool. `"java"` esiste già — riusa lo stesso oggetto.

```
STRING POOL: [ "java" ]
STACK: a → pool["java"]
       b → pool["java"]   ← stesso riferimento!
```

**Passo 3** — `String c = new String("java")`: `new` crea sempre un nuovo oggetto nell'heap, fuori dal pool.

```
STRING POOL: [ "java" ]
HEAP:        [ "java" ]   ← oggetto nuovo e separato
STACK: a → pool["java"]
       b → pool["java"]
       c → heap["java"]
```

Ecco perché `a == b` è `true` (stesso riferimento nel pool) ma `a == c` è `false` (oggetti diversi).

---

## Esperimento guidato

Prevedi l'output prima di eseguire:

```java
// Caso 1: letterali
String a = "java";
String b = "ja" + "va";  // concatenazione di letterali → avviene a compile time!
String c = new String("java");

System.out.println(a == b);      // ?
System.out.println(a == c);      // ?
System.out.println(a.equals(c)); // ?
```

```java
// Caso 2: variabile + letterale
String a = "java";
String x = "ja";
String b = x + "va";  // concatenazione a runtime → crea un nuovo oggetto nell'heap!

System.out.println(a == b);      // ?
System.out.println(a.equals(b)); // ?
```

::: {.callout-tip}
## La chiave per rispondere
`"ja" + "va"` è una concatenazione di due **letterali** — il compilatore la risolve a compile time e produce un solo oggetto `"java"` nel pool.

`x + "va"` invece coinvolge una **variabile** — il compilatore non può prevederla, quindi la concatenazione avviene a runtime e crea un nuovo oggetto nell'heap.
:::

---

## Il metodo `intern()`

`intern()` forza una stringa a entrare nello String Pool (o restituisce il riferimento se esiste già):

```java
String s = new String("ciao");  // nell'heap
String p = s.intern();          // restituisce il riferimento nel pool

String a = "ciao";
System.out.println(a == p);     // true — ora puntano allo stesso oggetto nel pool
```

Utile in rari casi in cui vuoi garantire il riuso della memoria per stringhe create dinamicamente.

---

## Metodi avanzati di String

Alcuni metodi utili per l'elaborazione di testo:

```java
String s = "  Arthas il Re dei Lich  ";

// Pulizia
s.trim();              // rimuove spazi iniziali e finali
s.strip();             // come trim ma gestisce anche spazi Unicode
s.stripLeading();      // solo spazi iniziali
s.stripTrailing();     // solo spazi finali

// Controllo
s.isBlank();           // true se vuota o solo spazi
s.isEmpty();           // true solo se lunghezza zero
"abc".repeat(3);       // "abcabcabc"

// Split
String csv = "Arthas,150,5";
String[] parti = csv.split(",");     // ["Arthas", "150", "5"]
String[] prime = csv.split(",", 2);  // ["Arthas", "150,5"] — max 2 parti

// Confronto
"abc".compareTo("abd");              // negativo — "abc" viene prima
"abc".compareToIgnoreCase("ABC");    // 0 — uguali ignorando maiuscole
```

### Concatenare con join e formatted

```java
// String.join — unisce elementi con un separatore
String nomi = String.join(", ", "Arthas", "Gandalf", "Legolas");
// "Arthas, Gandalf, Legolas"

// Con una lista
List<String> lista = List.of("Arthas", "Gandalf", "Legolas");
String uniti = String.join(" | ", lista);
```
---

## Espressioni regolari

Le **espressioni regolari** (regex) sono un linguaggio per descrivere pattern di testo. Permettono di cercare, validare e trasformare stringhe in modo potente.

### I caratteri speciali principali

| Pattern | Significato | Esempio |
|---|---|---|
| `.` | Qualsiasi carattere | `a.c` → "abc", "a1c" |
| `*` | Zero o più del precedente | `ab*c` → "ac", "abc", "abbc" |
| `+` | Uno o più del precedente | `ab+c` → "abc", "abbc" (non "ac") |
| `?` | Zero o uno del precedente | `ab?c` → "ac", "abc" |
| `^` | Inizio della stringa | `^Arthas` → inizia con "Arthas" |
| `$` | Fine della stringa | `Lich$` → finisce con "Lich" |
| `\d` | Cifra (0-9) | `\d+` → uno o più cifre |
| `\w` | Carattere word (lettere, cifre, _) | `\w+` → una parola |
| `\s` | Spazio bianco | `\s+` → uno o più spazi |
| `[abc]` | Uno tra a, b, c | `[aeiou]` → una vocale |
| `[^abc]` | Nessuno tra a, b, c | `[^0-9]` → non una cifra |
| `[a-z]` | Range | `[a-zA-Z]` → qualsiasi lettera |
| `{n}` | Esattamente n volte | `\d{3}` → esattamente 3 cifre |
| `{n,m}` | Da n a m volte | `\d{2,4}` → da 2 a 4 cifre |
| `(abc)` | Gruppo | `(ab)+` → "ab", "abab" |
| `a\|b` | a oppure b | `gatto\|cane` |

::: {.callout-warning}
## Escape in Java
In Java i caratteri speciali delle regex vanno scritti con doppio backslash:
- `\d` in regex → `"\\d"` in Java
- `\w+` in regex → `"\\w+"` in Java
:::

### matches(), replaceAll(), split()

```java
// Validazione
"12345".matches("\\d+");               // true — solo cifre
"333-1234567".matches("\\d{3}-\\d{7}"); // true — formato telefono

// Sostituzione
String s = "Arthas ha 100 punti e Gandalf ha 250 punti";
s.replaceAll("\\d+", "###");           // "Arthas ha ### punti e Gandalf ha ### punti"
s.replaceFirst("\\d+", "###");         // solo la prima occorrenza
"Arthas   il    Re".replaceAll("\\s+", " "); // "Arthas il Re"

// Split
"Arthas   il Re".split("\\s+");        // ["Arthas", "il", "Re"]
"Arthas, Gandalf ,Legolas".split("\\s*,\\s*"); // ["Arthas", "Gandalf", "Legolas"]
```

### Pattern e Matcher: ricerca avanzata

```java
import java.util.regex.*;

String testo = "Arthas ha 150 vita e 30 di attacco";
Pattern pattern = Pattern.compile("\\d+");
Matcher matcher = pattern.matcher(testo);

while (matcher.find()) {
    System.out.println("Trovato: " + matcher.group() +
        " alla posizione " + matcher.start());
}
// Trovato: 150 alla posizione 10
// Trovato: 30 alla posizione 22
```

Con i **gruppi** puoi estrarre parti specifiche:

```java
String dati = "Arthas:livello=5,vita=150";
Pattern p = Pattern.compile("(\\w+):livello=(\\d+),vita=(\\d+)");
Matcher m = p.matcher(dati);

if (m.matches()) {
    System.out.println("Nome:    " + m.group(1));  // Arthas
    System.out.println("Livello: " + m.group(2));  // 5
    System.out.println("Vita:    " + m.group(3));  // 150
}
```

---

## Esempio completo: parser di un log di gioco

```java
import java.util.*;
import java.util.regex.*;
import java.util.stream.*;

public class AnalisiLog {
    public static void main(String[] args) {

        List<String> log = List.of(
            "[10:05] Arthas ha sconfitto Zombie (+50 punti)",
            "[10:07] Gandalf ha lanciato Fireball (-25 mana)",
            "[10:09] Arthas ha sconfitto Scheletro (+30 punti)",
            "[10:12] Legolas ha mancato il bersaglio",
            "[10:15] Gandalf ha sconfitto Drago (+200 punti)"
        );

        Pattern punteggio = Pattern.compile(
            "\\[(\\d+:\\d+)\\] (\\w+) ha sconfitto .+ \\(\\+(\\d+) punti\\)"
        );

        Map<String, Integer> totali = new HashMap<>();

        for (String riga : log) {
            Matcher m = punteggio.matcher(riga);
            if (m.matches()) {
                String nome = m.group(2);
                int punti = Integer.parseInt(m.group(3));
                totali.merge(nome, punti, Integer::sum);
            }
        }

        // Report finale con StringBuilder
        StringBuilder report = new StringBuilder();
        report.append("=== REPORT FINALE ===\n");

        totali.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .forEach(e -> report
                .append(e.getKey())
                .append(": ")
                .append(e.getValue())
                .append(" punti\n"));

        System.out.println(report.toString());
    }
}
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave

- `String` è **immutabile** — ogni operazione crea un nuovo oggetto in memoria.
- La concatenazione in ciclo è inefficiente — ogni iterazione crea e scarta un oggetto.
- `StringBuilder` è **mutabile** — mantiene un array interno modificabile, molto più efficiente per costruzione incrementale.
- Lo **String Pool** è una zona speciale dell'heap dove Java conserva le stringhe letterali per evitare duplicazioni — funziona grazie all'immutabilità.
- Nello **Stack** vivono i riferimenti; nell'**Heap** vivono gli oggetti; nel **String Pool** vivono le stringhe letterali.
- `new String("...")` bypassa il pool e crea sempre un nuovo oggetto nell'heap — usa `==` restituisce `false`.
- Le **espressioni regolari** descrivono pattern di testo: `matches()` valida, `replaceAll()` sostituisce, `split()` divide.
- `Pattern` e `Matcher` permettono ricerche avanzate con estrazione di gruppi.
:::