# Path, Files e Log

---

## Lavorare con i file: una premessa

Finora i nostri programmi hanno lavorato con dati scritti direttamente nel codice. Quei dati sono sempre disponibili e sotto il nostro controllo.

Nel mondo reale le applicazioni lavorano quasi sempre con dati esterni: file di testo, log, configurazioni, CSV, dati prodotti da altri programmi. Un file non è mai garantito: può non esistere, può essere vuoto, può essere stato spostato, può non essere leggibile.

Per questo lavorare con i file significa accettare l'idea che qualcosa possa andare storto anche se il codice è corretto. Ed è qui che le **eccezioni checked** smettono di essere teoria e diventano necessarie.

---

## Path: non è una stringa

In Java un file non si rappresenta con una semplice stringa, ma con un oggetto di tipo **`Path`**.

`Path` non è il file in sé — è il percorso che indica **dove si trova**. Possiamo pensarlo come un indirizzo o un puntatore a una posizione sul disco.

```java
Path p = Path.of("input.txt");  // descrive una posizione, non legge nulla
```

Creare un `Path` non genera errori — anche se il file non esiste. L'errore arriva solo quando proviamo davvero ad accedere al file tramite `Files`.

::: {.callout-important}
## Path NON è una stringa
`Path` è un oggetto che rappresenta la posizione di una risorsa in modo **astratto e portabile**. A differenza di una stringa:

- conosce i separatori corretti per ogni sistema operativo
- permette di costruire percorsi in modo sicuro
- evita errori di concatenazione manuale
- non dipende da Windows, Linux o macOS
:::

---

## Percorsi relativi e assoluti

Un **percorso relativo** indica dove si trova un file a partire dalla cartella di lavoro del programma — non dal disco. Il programma non dipende dalla macchina e funziona anche se sposti il progetto.

Un **percorso assoluto** parte dalla radice del disco — è fragile e dipende dalla macchina.

```java
// Percorso relativo — preferibile
Path p1 = Path.of("dati.txt");
Path p2 = Path.of("Resources", "dati.txt");  // cartella + file separati

// Percorso assoluto — da evitare nei progetti
Path p3 = Path.of("/home/utente/gioco/dati.txt");
```

Quando usi `Path.of("Resources", "dati.txt")` stai passando cartella e file come argomenti separati — Java costruisce il percorso corretto per il sistema operativo.

::: {.callout-tip}
## Quale sintassi preferire?
```java
// Preferibile — Java gestisce i separatori
Path p = Path.of("Resources", "dati.txt");

// Funziona ma è meno portabile
Path p = Path.of("Resources/dati.txt");
```
Entrambe funzionano, ma la prima è più robusta se il programma gira su sistemi diversi.
:::

---

## Path vs Files: separazione delle responsabilità

Java separa nettamente due concetti:

- **`Path`** — descrive **dove** si trova una risorsa
- **`Files`** — descrive **cosa** vogliamo fare con quella risorsa

```java
import java.nio.file.*;

Path file = Path.of("dati.txt");   // posizione
Files.readAllLines(file);          // operazione
```

`Files` è una classe di soli metodi statici — non si creano oggetti `Files`, si usano i suoi metodi passando un `Path`. Questo rende il codice molto leggibile: prima dichiari dove si trova il file, poi dici cosa vuoi farci.

Poiché `Files` lavora davvero con il filesystem, i suoi metodi possono fallire — e Java ci obbliga a gestire la `IOException`.

---

## Operazioni di base su Path

```java
Path file = Path.of("dati", "livello", "mappa.json");

file.getFileName();       // mappa.json
file.getParent();         // dati/livello
file.toAbsolutePath();    // percorso assoluto completo

// Combinare percorsi
Path base = Path.of("dati");
Path completo = base.resolve("livello/mappa.json");  // dati/livello/mappa.json

// Normalizzare (rimuove . e ..)
Path.of("dati/../dati/./mappa.json").normalize();    // dati/mappa.json
```

---

## Verificare esistenza e tipo

```java
Path file = Path.of("salvataggio.txt");

Files.exists(file);         // il file esiste?
Files.notExists(file);      // il file NON esiste?
Files.isRegularFile(file);  // è un file normale?
Files.isDirectory(file);    // è una directory?
Files.isReadable(file);     // è leggibile?
Files.isWritable(file);     // è scrivibile?
Files.size(file);           // dimensione in byte
```

---

## Leggere un file: tre approcci

### `readAllLines()` — tutto in memoria

```java
try {
    List<String> righe = Files.readAllLines(Path.of("squadra.txt"));
    for (String riga : righe) {
        System.out.println(riga);
    }
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

Comodo e leggibile. Ma carica tutto il file in RAM — adatto solo per file piccoli.

### `readString()` — come unica stringa

```java
try {
    String contenuto = Files.readString(Path.of("config.txt"));
    System.out.println(contenuto);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

### `Files.lines()` — stream lazy

```java
try (Stream<String> righe = Files.lines(Path.of("log.txt"))) {
    righe
        .filter(r -> !r.isBlank())
        .forEach(System.out::println);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

Qui non stiamo dicendo "leggimi tutto" — stiamo dicendo "forniscimi una sequenza di righe". Il file viene letto un po' alla volta, man mano che lo stream avanza. Può elaborare un file enorme senza mai tenerlo tutto in memoria. È il primo ponte verso il concetto di buffer.

::: {.callout-important}
## Lo stream collegato a un file è una risorsa
`Files.lines()` restituisce uno stream che tiene aperto il file. Va chiuso quando hai finito. Per questo lo mettiamo tra le parentesi del `try-with-resources` — Java lo chiude automaticamente all'uscita dal blocco, anche in caso di errore.

Inoltre: **uno stream si può usare una sola volta**. Se vuoi fare più analisi sullo stesso file, devi riaprirlo ogni volta.
:::

---

## Scrivere un file

```java
// Scrittura semplice (crea o sovrascrive)
try {
    Files.writeString(Path.of("output.txt"), "Contenuto\n");
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}

// Scrittura di più righe
try {
    Files.write(Path.of("output.txt"), List.of("riga 1", "riga 2", "riga 3"));
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}

// Aggiunta in fondo (append)
try {
    Files.writeString(Path.of("log.txt"), "Nuova riga\n",
        StandardOpenOption.APPEND, StandardOpenOption.CREATE);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

::: {.callout-note}
## Sovrascrittura vs append
Quando scrivi su file, poniti sempre questa domanda: sto sostituendo il contenuto o lo sto aggiungendo?

- **Sovrascrittura** — il file viene ricreato ad ogni esecuzione. Tipico dei **report** (voglio l'ultimo risultato).
- **Append** — il nuovo testo si aggiunge al contenuto esistente. Tipico dei **log** (voglio una cronologia).

Questa scelta è importante perché cambia il significato del file prodotto.
:::

---

## Cartelle di output: buona pratica

È buona pratica separare i file di input da quelli di output usando cartelle dedicate (`Reports/`, `Output/`, `Logs/`). Un programma robusto controlla che la cartella esista e la crea se non c'è:

```java
private static final Path INPUT       = Path.of("accessi.txt");
private static final Path REPORTS_DIR = Path.of("Reports");
private static final Path REPORT_FILE = REPORTS_DIR.resolve("report.txt");

// Prima creo la cartella, poi scrivo
Files.createDirectories(REPORTS_DIR);
Files.writeString(REPORT_FILE, contenuto, StandardCharsets.UTF_8);
```

---

## Esempio completo: analisi log di accessi

Un programma che legge un file di log, calcola statistiche e genera un report.

Il file `accessi.txt`:
```
2025-12-18 08:01;alice;LOGIN
2025-12-18 08:05;bob;LOGIN
2025-12-18 08:20;alice;LOGOUT
2025-12-18 08:45;carla;LOGIN
2025-12-18 09:10;bob;LOGOUT
2025-12-18 09:30;alice;LOGIN
```

```java
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.time.LocalDateTime;
import java.util.stream.Stream;

public class AnalisiAccessi {

    private static final Path INPUT       = Path.of("accessi.txt");
    private static final Path REPORTS_DIR = Path.of("Reports");
    private static final Path REPORT_FILE = REPORTS_DIR.resolve("report.txt");

    public static void main(String[] args) {

        if (!Files.exists(INPUT)) {
            System.out.println("Errore: file non trovato → " + INPUT);
            return;
        }

        // Ogni calcolo riapre il file — uno stream non si riusa
        long righeTotali    = contaRighe(INPUT);
        long righeValide    = contaRigheValide(INPUT);
        long login          = contaEvento(INPUT, "LOGIN");
        long logout         = contaEvento(INPUT, "LOGOUT");
        long utenti         = contaUtentiDistinti(INPUT);
        long malformate     = righeTotali - righeValide;

        // Prima costruisco il report in memoria, poi lo scrivo su file
        String report = buildReport(righeTotali, righeValide,
            malformate, login, logout, utenti);

        try {
            Files.createDirectories(REPORTS_DIR);
            Files.writeString(REPORT_FILE, report, StandardCharsets.UTF_8);
            System.out.println("Report salvato in: " + REPORT_FILE);
        } catch (IOException e) {
            System.out.println("Errore scrittura: " + e.getMessage());
        }
    }

    private static long contaRighe(Path p) {
        try (Stream<String> righe = Files.lines(p, StandardCharsets.UTF_8)) {
            return righe.count();
        } catch (IOException e) { return 0; }
    }

    private static long contaRigheValide(Path p) {
        try (Stream<String> righe = Files.lines(p, StandardCharsets.UTF_8)) {
            return righe.map(r -> r.split(";"))
                        .filter(c -> c.length == 3).count();
        } catch (IOException e) { return 0; }
    }

    private static long contaEvento(Path p, String evento) {
        try (Stream<String> righe = Files.lines(p, StandardCharsets.UTF_8)) {
            return righe.map(r -> r.split(";"))
                        .filter(c -> c.length == 3)
                        .filter(c -> c[2].equals(evento)).count();
        } catch (IOException e) { return 0; }
    }

    private static long contaUtentiDistinti(Path p) {
        try (Stream<String> righe = Files.lines(p, StandardCharsets.UTF_8)) {
            return righe.map(r -> r.split(";"))
                        .filter(c -> c.length == 3)
                        .map(c -> c[1].trim())
                        .filter(s -> !s.isEmpty())
                        .distinct().count();
        } catch (IOException e) { return 0; }
    }

    private static String buildReport(long tot, long valid, long bad,
                                      long login, long logout, long utenti) {
        return "REPORT ACCESSI\n"
             + "==============\n"
             + "Generato: " + LocalDateTime.now() + "\n\n"
             + "Righe totali:     " + tot    + "\n"
             + "Righe valide:     " + valid  + "\n"
             + "Righe malformate: " + bad    + "\n\n"
             + "LOGIN:            " + login  + "\n"
             + "LOGOUT:           " + logout + "\n\n"
             + "Utenti distinti:  " + utenti + "\n";
    }
}
```

---

## Log: registrare gli eventi

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.time.*;
import java.time.format.*;
import java.io.IOException;

public class Logger {

    private final Path fileLog;
    private final DateTimeFormatter formatter =
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public Logger(String percorso) {
        this.fileLog = Path.of(percorso);
        try {
            Files.createDirectories(fileLog.getParent());
        } catch (IOException e) {
            System.out.println("Impossibile creare cartella log: " + e.getMessage());
        }
    }

    public void log(String livello, String messaggio) {
        String riga = "[" + LocalDateTime.now().format(formatter) + "] "
                    + "[" + livello + "] " + messaggio + "\n";
        try {
            Files.writeString(fileLog, riga,
                StandardCharsets.UTF_8,
                StandardOpenOption.CREATE,
                StandardOpenOption.APPEND);
        } catch (IOException e) {
            System.out.println("Errore log: " + e.getMessage());
        }
    }

    public void info(String msg)  { log("INFO ", msg); }
    public void warn(String msg)  { log("WARN ", msg); }
    public void error(String msg) { log("ERROR", msg); }
}
```

---

## Gestione delle date con `java.time`

```java
import java.time.*;
import java.time.format.*;
import java.time.temporal.ChronoUnit;

LocalDate oggi       = LocalDate.now();
LocalTime ora        = LocalTime.now();
LocalDateTime adesso = LocalDateTime.now();

LocalDate natale = LocalDate.of(2025, 12, 25);

LocalDate domani = oggi.plusDays(1);
boolean passato  = natale.isBefore(oggi);

DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
String formattata = adesso.format(fmt);

long giorni = ChronoUnit.DAYS.between(oggi, natale);
```

::: {.callout-tip}
## Perché `java.time` e non `Date`?
La vecchia classe `java.util.Date` è mutabile, con mesi che partono da 0 e gestione del fuso orario confusa. `java.time` (Java 8+) è immutabile, chiara e corretta. Usa sempre `java.time`.
:::

---

## Riepilogo

::: {.callout-note}
## I concetti chiave

- **`Path`** rappresenta **dove** si trova una risorsa — non verifica nulla, non legge nulla.
- **`Files`** rappresenta **cosa** fare con quella risorsa — può fallire con `IOException`.
- Usa `Path.of("cartella", "file.txt")` — Java gestisce i separatori in modo portabile.
- I percorsi **relativi** sono preferibili — il codice funziona su qualsiasi macchina.
- `Files.readAllLines()` carica tutto in RAM — adatto per file piccoli.
- `Files.lines()` è **lazy** — legge man mano, adatto per file grandi. Va chiuso con try-with-resources.
- Uno stream collegato a un file si usa **una sola volta** — riaprilo per ogni analisi.
- Separa la **costruzione dei dati** dalla loro **scrittura su file**.
- Usa **`java.time`** per gestire date e ore — mai `java.util.Date`.
:::
