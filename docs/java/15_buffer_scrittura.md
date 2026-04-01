# Buffer e scrittura

---

## Perché i buffer?

Nella lezione precedente abbiamo usato `Files.writeString()` e `Files.write()` — comode e semplici. Ma hanno un limite: ogni chiamata apre il file, scrive, e lo chiude. Se devi scrivere molte righe una alla volta — come in un log in tempo reale — questo approccio è inefficiente.

Immagina di dover costruire un muro mattone per mattone: ogni volta che posi un mattone, scendi dal ponteggio, vai al magazzino, torni su. È molto più efficiente caricare un carrello di mattoni, salire una volta sola e posarli tutti.

Il **buffer** è quel carrello: raccoglie i dati in memoria e li scrive sul disco in un'unica operazione quando il buffer è pieno o quando lo chiudi esplicitamente.

```
Scrittura senza buffer:          Scrittura con buffer:
[dato] → disco                   [dato] → buffer
[dato] → disco        vs         [dato] → buffer
[dato] → disco                   [dato] → buffer → disco (un'unica scrittura)
```

---

## BufferedWriter: scrivere con buffer

`BufferedWriter` avvolge uno scrittore base e aggiunge il buffering. Si usa tipicamente con `Files.newBufferedWriter()`:

```java
import java.nio.file.*;
import java.io.*;
import java.nio.charset.StandardCharsets;

Path file = Path.of("output.txt");

try (BufferedWriter writer = Files.newBufferedWriter(file)) {

    writer.write("Prima riga");
    writer.newLine();  // aggiunge il carattere di a capo corretto per il SO
    writer.write("Seconda riga");
    writer.newLine();
    writer.write("Terza riga");

} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
// il try-with-resources chiude e svuota il buffer automaticamente
```

Per aggiungere in append senza sovrascrivere:

```java
try (BufferedWriter writer = Files.newBufferedWriter(file,
        StandardOpenOption.APPEND,
        StandardOpenOption.CREATE)) {

    writer.write("Riga aggiunta");
    writer.newLine();

} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

::: {.callout-note}
## `newLine()` vs `\n`
Usa sempre `writer.newLine()` invece di `"\n"`. Il metodo scrive il carattere di a capo corretto per il sistema operativo corrente: `\n` su Linux/Mac, `\r\n` su Windows. Così il file funziona correttamente su tutte le piattaforme.
:::

---

## PrintWriter: scrivere con formattazione

`PrintWriter` aggiunge metodi comodi come `println()` e `printf()` — simili a `System.out`. È ideale quando vuoi scrivere testo formattato su file:

```java
import java.io.*;
import java.nio.file.*;

Path file = Path.of("report.txt");

try (PrintWriter pw = new PrintWriter(
        Files.newBufferedWriter(file))) {

    pw.println("=== REPORT SQUADRA ===");
    pw.println();

    // printf-style formatting
    pw.printf("%-15s %5s %5s %5s%n", "Nome", "Vita", "Att", "Lv");
    pw.printf("%-15s %5d %5d %5d%n", "Arthas", 150, 30, 5);
    pw.printf("%-15s %5d %5d %5d%n", "Gandalf", 100, 25, 7);
    pw.printf("%-15s %5d %5d %5d%n", "Legolas", 120, 28, 6);

} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

Output nel file:
```
=== REPORT SQUADRA ===

Nome             Vita   Att    Lv
Arthas            150    30     5
Gandalf           100    25     7
Legolas           120    28     6
```

---

## BufferedReader: leggere con buffer

`BufferedReader` fa il contrario — legge con buffer per efficienza. Il metodo `readLine()` legge una riga alla volta:

```java
import java.io.*;
import java.nio.file.*;

Path file = Path.of("squadra.csv");

try (BufferedReader reader = Files.newBufferedReader(file)) {

    String riga;
    while ((riga = reader.readLine()) != null) {
        System.out.println(riga);
    }

} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

::: {.callout-tip}
## `BufferedReader` vs `Files.readAllLines()`
`Files.readAllLines()` carica **tutto** il file in memoria. Con file piccoli va benissimo. Con file grandi (log di milioni di righe) è un problema.

`BufferedReader` legge una riga alla volta — usa solo la memoria necessaria per una riga. Scegli `BufferedReader` quando il file può essere molto grande.

`Files.lines()` (lo Stream lazy che abbiamo visto) è ancora più elegante di `BufferedReader` per la maggior parte dei casi.
:::

---

## La gerarchia di I/O di Java

Java ha due famiglie di classi per l'I/O:

```
Stream (byte)                    Reader/Writer (caratteri)
├── InputStream                  ├── Reader
│   ├── FileInputStream          │   ├── FileReader
│   └── BufferedInputStream      │   └── BufferedReader
└── OutputStream                 └── Writer
    ├── FileOutputStream             ├── FileWriter
    └── BufferedOutputStream         ├── BufferedWriter
                                     └── PrintWriter
```

**Stream** — lavorano con byte grezzi. Utili per file binari (immagini, audio, dati serializzati).

**Reader/Writer** — lavorano con caratteri. Utili per file di testo, gestendo correttamente la codifica (UTF-8, ecc.).

Per i file di testo usa sempre `Reader`/`Writer` — `Files.newBufferedReader()` e `Files.newBufferedWriter()` sono il punto di ingresso consigliato.

---

## Codifica dei caratteri

Quando lavori con file di testo, la **codifica** (charset) è importante. La codifica di default dipende dal sistema operativo — può causare problemi quando il file viene aperto su un sistema diverso.

La soluzione: specifica sempre **UTF-8** esplicitamente.

```java
import java.nio.charset.StandardCharsets;

// Specifica UTF-8 esplicitamente — comportamento identico su tutti i sistemi
try (BufferedWriter writer = Files.newBufferedWriter(
        file, StandardCharsets.UTF_8)) {
    writer.write("Testo con accenti: à è ì ò ù");
    writer.newLine();
}

try (BufferedReader reader = Files.newBufferedReader(
        file, StandardCharsets.UTF_8)) {
    System.out.println(reader.readLine());
}
```

::: {.callout-important}
## Usa sempre UTF-8
Specifica `StandardCharsets.UTF_8` ogni volta che apri un file di testo. Evita problemi con caratteri speciali, accenti e simboli su sistemi diversi.
:::

---

## Flush: svuotare il buffer manualmente

Normalmente il buffer viene svuotato (flushed) automaticamente quando il writer viene chiuso. Ma in alcuni casi — come un log in tempo reale — vuoi che i dati vengano scritti subito, senza aspettare la chiusura.

```java
try (BufferedWriter writer = Files.newBufferedWriter(
        Path.of("live.log"), StandardOpenOption.APPEND)) {

    for (int i = 0; i < 5; i++) {
        writer.write("Evento " + i);
        writer.newLine();
        writer.flush();  // scrive immediatamente sul disco

        Thread.sleep(1000); // aspetta 1 secondo
    }
} catch (IOException | InterruptedException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

Senza `flush()`, gli eventi rimarrebbero nel buffer e verrebbero scritti tutti insieme alla chiusura. Con `flush()` ogni evento è immediatamente visibile nel file.

---

## Esempio completo: logger con buffer

Riscriviamo il logger della lezione precedente usando `BufferedWriter` per maggiore efficienza, e aggiungiamo la rotazione del log (un file per giorno):

```java
import java.nio.file.*;
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.time.*;
import java.time.format.*;

public class Logger {

    private final Path cartella;
    private final DateTimeFormatter fmtData =
        DateTimeFormatter.ofPattern("yyyy-MM-dd");
    private final DateTimeFormatter fmtTimestamp =
        DateTimeFormatter.ofPattern("HH:mm:ss");

    public Logger(String cartella) {
        this.cartella = Path.of(cartella);
        try {
            Files.createDirectories(this.cartella);
        } catch (IOException e) {
            System.out.println("Impossibile creare cartella log: " + e.getMessage());
        }
    }

    // Il file di log cambia ogni giorno: gioco-2025-11-13.log
    private Path fileOggi() {
        String nomeFile = "gioco-" + LocalDate.now().format(fmtData) + ".log";
        return cartella.resolve(nomeFile);
    }

    private void scrivi(String livello, String messaggio) {
        String riga = "[" + LocalTime.now().format(fmtTimestamp) + "] " +
                      "[" + livello + "] " + messaggio;

        try (BufferedWriter writer = Files.newBufferedWriter(
                fileOggi(),
                StandardCharsets.UTF_8,
                StandardOpenOption.CREATE,
                StandardOpenOption.APPEND)) {

            writer.write(riga);
            writer.newLine();

        } catch (IOException e) {
            System.err.println("Errore log: " + e.getMessage());
        }
    }

    public void info(String msg)  { scrivi("INFO ", msg); }
    public void warn(String msg)  { scrivi("WARN ", msg); }
    public void error(String msg) { scrivi("ERROR", msg); }

    // Legge e stampa il log di oggi
    public void stampaLog() {
        Path file = fileOggi();
        if (!Files.exists(file)) {
            System.out.println("Nessun log per oggi.");
            return;
        }

        System.out.println("=== LOG " + LocalDate.now() + " ===");
        try (BufferedReader reader = Files.newBufferedReader(
                file, StandardCharsets.UTF_8)) {

            String riga;
            while ((riga = reader.readLine()) != null) {
                System.out.println(riga);
            }

        } catch (IOException e) {
            System.out.println("Errore lettura: " + e.getMessage());
        }
    }
}
```

Uso:

```java
public class Main {
    public static void main(String[] args) {

        Logger logger = new Logger("logs");

        logger.info("Partita iniziata");
        logger.info("Arthas ha raggiunto il livello 5");
        logger.warn("Vita di Gandalf sotto il 20%");
        logger.error("File di salvataggio corrotto");
        logger.info("Partita terminata");

        logger.stampaLog();
    }
}
```

Output del log `logs/gioco-2025-11-13.log`:
```
[10:05:32] [INFO ] Partita iniziata
[10:05:32] [INFO ] Arthas ha raggiunto il livello 5
[10:05:32] [WARN ] Vita di Gandalf sotto il 20%
[10:05:32] [ERROR] File di salvataggio corrotto
[10:05:32] [INFO ] Partita terminata
```

---

## Confronto finale: quando usare cosa

| Strumento | Caso d'uso |
|---|---|
| `Files.writeString()` | Scrittura semplice di una stringa |
| `Files.write()` | Scrittura di una lista di righe |
| `Files.readAllLines()` | Lettura di file piccoli/medi |
| `Files.lines()` | Lettura lazy con Stream (file grandi) |
| `BufferedWriter` | Scrittura incrementale efficiente |
| `BufferedReader` | Lettura riga per riga (file grandi) |
| `PrintWriter` | Testo formattato con `printf` |

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- Il **buffer** raccoglie i dati in memoria e li scrive sul disco in un'unica operazione — molto più efficiente di scrivere un byte alla volta.
- **`BufferedWriter`** scrive con buffer; usa `newLine()` per l'a capo multipiattaforma.
- **`PrintWriter`** aggiunge `println()` e `printf()` per testo formattato.
- **`BufferedReader`** legge riga per riga con `readLine()` — ideale per file grandi.
- Specifica sempre **`StandardCharsets.UTF_8`** per evitare problemi con caratteri speciali.
- Usa `flush()` per scrivere immediatamente sul disco senza aspettare la chiusura.
- Il **try-with-resources** garantisce che il buffer venga svuotato e il file chiuso correttamente.
:::