# Stringhe avanzate

---

## String è immutabile

Nella lezione sui tipi abbiamo visto che `String` è una classe. Ma ha una caratteristica fondamentale che spesso sorprende: le stringhe in Java sono **immutabili** — una volta create, non possono essere modificate.

Quando sembra che stai modificando una stringa, in realtà stai creando un nuovo oggetto:

```java
String s = "ciao";
s = s.toUpperCase();  // NON modifica "ciao" — crea un nuovo oggetto "CIAO"
                      // "ciao" rimane in memoria finché il garbage collector non lo elimina
```

Ogni operazione su una `String` — concatenazione, `replace`, `substring` — crea un nuovo oggetto. L'originale non viene mai toccato.

::: {.callout-note}
## Perché immutabile?
L'immutabilità porta vantaggi concreti:
- **Thread safety** — più thread possono leggere la stessa stringa senza problemi
- **Cache** — Java può riusare stringhe identiche in memoria (String pool)
- **Sicurezza** — una stringa passata come parametro non può essere modificata dal chiamato
:::

---

## Il problema della concatenazione in loop

L'immutabilità diventa un problema quando concateni stringhe in un ciclo:

```java
// Approccio SBAGLIATO per grandi quantità di dati
String risultato = "";
for (int i = 0; i < 10000; i++) {
    risultato += "elemento" + i + "\n";  // crea un nuovo oggetto ad ogni iterazione!
}
```

Con 10.000 iterazioni, questo codice crea 10.000 oggetti `String` intermedi — quasi tutti subito scartati. È lento e spreca memoria.

---

## StringBuilder: concatenazione efficiente

`StringBuilder` è una stringa **mutabile** — puoi modificarla senza creare nuovi oggetti. È la soluzione per concatenazioni intensive.

```java
StringBuilder sb = new StringBuilder();

for (int i = 0; i < 10000; i++) {
    sb.append("elemento").append(i).append("\n");
}

String risultato = sb.toString();  // converte in String solo alla fine
```

Un solo oggetto, nessuno spreco. Molto più veloce.

### Metodi principali di StringBuilder

```java
StringBuilder sb = new StringBuilder("Arthas");

// Aggiungere in fondo
sb.append(" il Re dei Lich");           // "Arthas il Re dei Lich"

// Inserire in una posizione
sb.insert(6, " Menethil");              // "Arthas Menethil il Re dei Lich"

// Sostituire una porzione
sb.replace(7, 15, "");                  // rimuove "Menethil"

// Eliminare una porzione
sb.delete(0, 7);                        // rimuove "Arthas "

// Invertire
sb.reverse();                           // inverte tutta la stringa

// Lunghezza e carattere singolo
int len = sb.length();
char c = sb.charAt(0);

// Modificare un carattere
sb.setCharAt(0, 'A');

// Convertire in String
String s = sb.toString();
```

### Concatenare con join e formatted

Per casi semplici, Java offre alternative moderne:

```java
// String.join — unisce elementi con un separatore
String nomi = String.join(", ", "Arthas", "Gandalf", "Legolas");
// "Arthas, Gandalf, Legolas"

// Con una lista
List<String> lista = List.of("Arthas", "Gandalf", "Legolas");
String uniti = String.join(" | ", lista);
// "Arthas | Gandalf | Legolas"

// String.formatted — alternativa moderna a String.format
String scheda = "Nome: %s | Livello: %d | Vita: %d"
    .formatted("Arthas", 5, 150);
// "Nome: Arthas | Livello: 5 | Vita: 150"
```

::: {.callout-tip}
## Quando usare cosa
- **`+`** — per concatenazioni semplici e occasionali (1-2 operazioni)
- **`StringBuilder`** — per cicli e costruzione incrementale di stringhe
- **`String.join()`** — per unire elementi di una collezione con un separatore
- **`String.formatted()`** / **`String.format()`** — per stringhe con valori formattati
:::

---

## Metodi avanzati di String

Alcuni metodi utili che non abbiamo ancora visto:

```java
String s = "  Arthas il Re dei Lich  ";

// Pulizia
s.trim();                    // rimuove spazi iniziali e finali
s.strip();                   // come trim ma gestisce anche spazi Unicode
s.stripLeading();            // solo spazi iniziali
s.stripTrailing();           // solo spazi finali

// Controllo
s.isBlank();                 // true se vuota o solo spazi
s.isEmpty();                 // true solo se lunghezza zero
"abc".repeat(3);             // "abcabcabc"

// Split
String csv = "Arthas,150,5";
String[] parti = csv.split(",");  // ["Arthas", "150", "5"]

// Split con limite
String[] prime = csv.split(",", 2); // ["Arthas", "150,5"] — max 2 parti

// Confronto
"abc".compareTo("abd");      // negativo — "abc" viene prima
"abc".compareToIgnoreCase("ABC"); // 0 — uguali ignorando maiuscole

// chars() come IntStream
"Arthas".chars()
    .filter(c -> c == 'a')
    .count();  // conta le 'a' minuscole → 1
```

---

## Espressioni regolari

Le **espressioni regolari** (regex) sono un linguaggio per descrivere pattern di testo. Permettono di cercare, validare e trasformare stringhe in modo potente.

### I caratteri speciali principali

| Pattern | Significato | Esempio |
|---|---|---|
| `.` | Qualsiasi carattere | `a.c` → "abc", "a1c", "a-c" |
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
| `a\|b` | a oppure b | `gatto\|cane` → "gatto" o "cane" |

::: {.callout-warning}
## Escape in Java
In Java i caratteri speciali delle regex vanno scritti con doppio backslash perché il backslash è già un carattere speciale nelle stringhe Java:
- `\d` in regex → `"\\d"` in Java
- `\w+` in regex → `"\\w+"` in Java
:::

### matches(): validare una stringa

```java
// Verifica se tutta la stringa corrisponde al pattern
"Arthas123".matches("[A-Za-z]+\\d+");  // true — lettere seguite da cifre
"12345".matches("\\d+");               // true — solo cifre
"abc".matches("\\d+");                 // false — non sono cifre

// Validazione numero di telefono (semplificata)
String tel = "333-1234567";
boolean valido = tel.matches("\\d{3}-\\d{7}");  // true

// Validazione email (semplificata)
String email = "utente@esempio.it";
boolean emailValida = email.matches("[\\w.]+@[\\w.]+\\.[a-z]{2,}");
```

### replaceAll() e replaceFirst()

```java
String testo = "Arthas ha 100 punti e Gandalf ha 250 punti";

// Sostituisce tutte le occorrenze del pattern
String senzaNumeri = testo.replaceAll("\\d+", "###");
// "Arthas ha ### punti e Gandalf ha ### punti"

// Sostituisce solo la prima occorrenza
String primoSostituito = testo.replaceFirst("\\d+", "###");
// "Arthas ha ### punti e Gandalf ha 250 punti"

// Rimuove tutti gli spazi multipli
String pulita = "Arthas   il    Re".replaceAll("\\s+", " ");
// "Arthas il Re"

// Rimuove tutti i caratteri non alfanumerici
String soloAlpha = "Art#has-123!".replaceAll("[^\\w]", "");
// "Arthas123"
```

### split() con regex

```java
// Split su qualsiasi spazio bianco
String[] parole = "Arthas   il Re".split("\\s+");
// ["Arthas", "il", "Re"]

// Split su virgola con spazi opzionali
String[] nomi = "Arthas, Gandalf ,Legolas".split("\\s*,\\s*");
// ["Arthas", "Gandalf", "Legolas"]

// Split su più separatori
String[] parti = "uno:due;tre,quattro".split("[;:,]");
// ["uno", "due", "tre", "quattro"]
```

### Pattern e Matcher: ricerca avanzata

Per ricerche più sofisticate si usano le classi `Pattern` e `Matcher`:

```java
import java.util.regex.*;

String testo = "Arthas ha 150 vita e 30 di attacco";

Pattern pattern = Pattern.compile("\\d+");
Matcher matcher = pattern.matcher(testo);

// Trova tutte le occorrenze
while (matcher.find()) {
    System.out.println("Trovato: " + matcher.group() +
        " alla posizione " + matcher.start());
}
// Trovato: 150 alla posizione 10
// Trovato: 30 alla posizione 22
```

Con i **gruppi** puoi estrarre parti specifiche del match:

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

Mettiamo tutto insieme — leggiamo e analizziamo righe di log di un gioco:

```java
import java.util.*;
import java.util.regex.*;
import java.util.stream.*;

public class AnalisiLog {
    public static void main(String[] args) {

        // Righe di log simulate
        List<String> log = List.of(
            "[10:05] Arthas ha sconfitto Zombie (+50 punti)",
            "[10:07] Gandalf ha lanciato Fireball (-25 mana)",
            "[10:09] Arthas ha sconfitto Scheletro (+30 punti)",
            "[10:12] Legolas ha mancato il bersaglio",
            "[10:15] Gandalf ha sconfitto Drago (+200 punti)"
        );

        // Pattern per estrarre: timestamp, nome, punti
        Pattern punteggio = Pattern.compile(
            "\\[(\\d+:\\d+)\\] (\\w+) ha sconfitto .+ \\(\\+(\\d+) punti\\)"
        );

        // Costruisci mappa nome → punti totali
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

Output:
```
=== REPORT FINALE ===
Gandalf: 200 punti
Arthas: 80 punti
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- `String` è **immutabile** — ogni operazione crea un nuovo oggetto.
- `StringBuilder` è **mutabile** — usalo per concatenazioni in cicli o costruzioni incrementali.
- Usa `String.join()` per unire elementi e `String.formatted()` per stringhe con valori.
- Le **espressioni regolari** descrivono pattern di testo: `.`, `*`, `+`, `\d`, `\w`, `[]`, `{}`.
- `matches()` valida, `replaceAll()` sostituisce, `split()` divide — tutti accettano regex.
- `Pattern` e `Matcher` permettono ricerche avanzate con estrazione di gruppi.
- In Java i backslash nelle regex vanno raddoppiati: `\d` → `"\\d"`.
:::