# Path, Files e Log

---

## Lavorare con il filesystem

Fino ad ora i nostri programmi leggono dati da tastiera e scrivono sull'output. Ma nella realtà i dati vivono su file: configurazioni, salvataggi, log, report. Java offre un'API moderna e potente per gestire il filesystem — il package `java.nio.file`.

Le due classi principali sono:

- **`Path`** — rappresenta un percorso nel filesystem (file o directory)
- **`Files`** — classe di utilità con metodi statici per operare sui file

---

## Path: rappresentare un percorso

`Path` rappresenta un percorso nel filesystem, ma **non** implica che il file esista davvero — è solo una rappresentazione del percorso.

```java
import java.nio.file.*;

// Creare un Path
Path p1 = Path.of("dati/salvataggio.txt");           // relativo
Path p2 = Path.of("/home/utente/gioco/log.txt");     // assoluto (Linux/Mac)
Path p3 = Path.of("C:\\Gioco\\salvataggio.txt");     // assoluto (Windows)

// Operazioni sul percorso (senza toccare il filesystem)
Path file = Path.of("dati/livello/mappa.json");

System.out.println(file.getFileName());   // mappa.json
System.out.println(file.getParent());     // dati/livello
System.out.println(file.getRoot());       // null (percorso relativo)
System.out.println(file.isAbsolute());    // false

// Percorso assoluto (risolve rispetto alla directory corrente)
System.out.println(file.toAbsolutePath());

// Combinare percorsi
Path base = Path.of("dati");
Path completo = base.resolve("livello/mappa.json");  // dati/livello/mappa.json

// Normalizzare (rimuove . e ..)
Path norm = Path.of("dati/../dati/./mappa.json").normalize(); // dati/mappa.json
```

---

## Files: operare sui file

La classe `Files` offre metodi statici per leggere, scrivere, copiare, spostare e cancellare file.

### Verificare esistenza e tipo

```java
Path file = Path.of("salvataggio.txt");
Path dir  = Path.of("dati");

Files.exists(file);        // il file esiste?
Files.notExists(file);     // il file NON esiste?
Files.isRegularFile(file); // è un file normale?
Files.isDirectory(dir);    // è una directory?
Files.isReadable(file);    // è leggibile?
Files.isWritable(file);    // è scrivibile?
Files.size(file);          // dimensione in byte
```

### Leggere un file

```java
import java.nio.file.*;
import java.io.IOException;

Path file = Path.of("squadra.txt");

// Leggi tutte le righe in una List<String>
try {
    List<String> righe = Files.readAllLines(file);
    for (String riga : righe) {
        System.out.println(riga);
    }
} catch (IOException e) {
    System.out.println("Errore lettura: " + e.getMessage());
}

// Leggi come unica stringa
try {
    String contenuto = Files.readString(file);
    System.out.println(contenuto);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}

// Leggi come Stream di righe (lazy — utile per file grandi)
try (Stream<String> righe = Files.lines(file)) {
    righe
        .filter(r -> !r.isBlank())
        .forEach(System.out::println);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

### Scrivere un file

```java
Path file = Path.of("output.txt");

// Scrivi una stringa (crea o sovrascrive il file)
try {
    Files.writeString(file, "Contenuto del file\n");
} catch (IOException e) {
    System.out.println("Errore scrittura: " + e.getMessage());
}

// Scrivi più righe
List<String> righe = List.of("riga 1", "riga 2", "riga 3");
try {
    Files.write(file, righe);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}

// Aggiungi senza sovrascrivere (append)
try {
    Files.writeString(file, "Nuova riga\n", StandardOpenOption.APPEND);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

::: {.callout-note}
## StandardOpenOption
Le opzioni di apertura del file si specificano con `StandardOpenOption`:
- `CREATE` — crea il file se non esiste (default)
- `CREATE_NEW` — crea solo se non esiste, errore se esiste già
- `APPEND` — aggiunge in fondo senza sovrascrivere
- `TRUNCATE_EXISTING` — svuota il file prima di scrivere (default)
:::

### Operazioni su file e directory

```java
Path origine = Path.of("dati/mappa.json");
Path dest    = Path.of("backup/mappa.json");
Path dir     = Path.of("nuova_cartella");

// Creare directory
Files.createDirectory(dir);          // crea una directory
Files.createDirectories(dir);        // crea anche le parent mancanti

// Copiare
Files.copy(origine, dest);           // errore se dest esiste già
Files.copy(origine, dest, StandardCopyOption.REPLACE_EXISTING);

// Spostare / rinominare
Files.move(origine, dest);
Files.move(origine, dest, StandardCopyOption.REPLACE_EXISTING);

// Cancellare
Files.delete(file);                  // lancia eccezione se non esiste
Files.deleteIfExists(file);          // non lancia eccezione

// Listare contenuto di una directory
try (Stream<Path> contenuto = Files.list(dir)) {
    contenuto.forEach(System.out::println);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}

// Listare ricorsivamente
try (Stream<Path> tutti = Files.walk(dir)) {
    tutti
        .filter(Files::isRegularFile)
        .forEach(System.out::println);
} catch (IOException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

---

## Salvataggio e caricamento di dati

Vediamo un esempio pratico — salvare e caricare i dati di una squadra:

```java
import java.nio.file.*;
import java.io.IOException;
import java.util.*;

public class GestioneSalvataggio {

    // Salva la squadra in formato CSV
    public static void salva(List<Personaggio> squadra, Path file) {
        List<String> righe = new ArrayList<>();
        righe.add("nome,vita,vitaMax,attacco,livello");  // header

        for (Personaggio p : squadra) {
            righe.add(p.getNome() + "," +
                      p.getVita() + "," +
                      p.getVitaMax() + "," +
                      p.getAttacco() + "," +
                      p.getLivello());
        }

        try {
            Files.createDirectories(file.getParent());
            Files.write(file, righe);
            System.out.println("Squadra salvata in " + file);
        } catch (IOException e) {
            System.out.println("Errore nel salvataggio: " + e.getMessage());
        }
    }

    // Carica la squadra da CSV
    public static List<Personaggio> carica(Path file) {
        List<Personaggio> squadra = new ArrayList<>();

        try {
            List<String> righe = Files.readAllLines(file);
            // salta la prima riga (header)
            for (int i = 1; i < righe.size(); i++) {
                String[] parti = righe.get(i).split(",");
                String nome   = parti[0];
                int vita      = Integer.parseInt(parti[1]);
                int vitaMax   = Integer.parseInt(parti[2]);
                int attacco   = Integer.parseInt(parti[3]);
                int livello   = Integer.parseInt(parti[4]);

                Personaggio p = new Personaggio(nome, vitaMax, attacco, livello);
                p.setVita(vita);
                squadra.add(p);
            }
            System.out.println("Caricati " + squadra.size() + " personaggi.");
        } catch (IOException e) {
            System.out.println("File non trovato: " + e.getMessage());
        } catch (NumberFormatException e) {
            System.out.println("File corrotto: " + e.getMessage());
        }

        return squadra;
    }

    public static void main(String[] args) {
        Path fileSalvataggio = Path.of("salvataggi/squadra.csv");

        // Crea e salva una squadra
        List<Personaggio> squadra = new ArrayList<>();
        squadra.add(new Personaggio("Arthas", 150, 30, 5));
        squadra.add(new Personaggio("Gandalf", 100, 25, 7));
        squadra.add(new Personaggio("Legolas", 120, 28, 6));

        salva(squadra, fileSalvataggio);

        // Ricarica dal file
        List<Personaggio> caricata = carica(fileSalvataggio);
        caricata.forEach(Personaggio::stampaScheda);
    }
}
```

---

## Log: registrare gli eventi

Un **log** è un registro cronologico di eventi — fondamentale per il debug, il monitoraggio e l'analisi post-mortem. Ogni riga di log ha tipicamente: timestamp, livello di gravità e messaggio.

### Log semplice con Files

```java
import java.nio.file.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.io.IOException;

public class Logger {

    private final Path fileLog;
    private final DateTimeFormatter formatter =
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public Logger(String percorso) {
        this.fileLog = Path.of(percorso);
        try {
            Files.createDirectories(fileLog.getParent());
            if (!Files.exists(fileLog)) {
                Files.createFile(fileLog);
            }
        } catch (IOException e) {
            System.out.println("Impossibile creare il file di log: " + e.getMessage());
        }
    }

    public void log(String livello, String messaggio) {
        String riga = "[" + LocalDateTime.now().format(formatter) + "] " +
                      "[" + livello + "] " + messaggio + "\n";
        try {
            Files.writeString(fileLog, riga, StandardOpenOption.APPEND);
        } catch (IOException e) {
            System.out.println("Errore scrittura log: " + e.getMessage());
        }
    }

    public void info(String msg)    { log("INFO ", msg); }
    public void warn(String msg)    { log("WARN ", msg); }
    public void error(String msg)   { log("ERROR", msg); }
}
```

Uso del logger:

```java
Logger logger = new Logger("logs/gioco.log");

logger.info("Partita iniziata");
logger.info("Arthas ha raggiunto il livello 5");
logger.warn("Vita di Gandalf sotto il 20%");
logger.error("File di salvataggio corrotto");
```

Il file `logs/gioco.log` conterrà:
```
[2025-11-13 10:05:32] [INFO ] Partita iniziata
[2025-11-13 10:07:18] [INFO ] Arthas ha raggiunto il livello 5
[2025-11-13 10:09:45] [WARN ] Vita di Gandalf sotto il 20%
[2025-11-13 10:12:03] [ERROR] File di salvataggio corrotto
```

### Leggere e analizzare i log

Con gli Stream possiamo analizzare i log in modo elegante:

```java
public void analizza() {
    try {
        List<String> righe = Files.readAllLines(fileLog);

        System.out.println("Totale eventi: " + righe.size());

        long errori = righe.stream()
            .filter(r -> r.contains("[ERROR]"))
            .count();
        System.out.println("Errori: " + errori);

        long warning = righe.stream()
            .filter(r -> r.contains("[WARN ]"))
            .count();
        System.out.println("Warning: " + warning);

        System.out.println("\nUltimi 5 eventi:");
        righe.stream()
            .skip(Math.max(0, righe.size() - 5))
            .forEach(System.out::println);

    } catch (IOException e) {
        System.out.println("Errore lettura log: " + e.getMessage());
    }
}
```

---

## Gestione delle date e ore

Per i timestamp nei log e nei salvataggi, Java offre il package `java.time` — moderno, immutabile e chiaro.

```java
import java.time.*;
import java.time.format.*;

// Data e ora corrente
LocalDate oggi = LocalDate.now();           // solo data: 2025-11-13
LocalTime ora  = LocalTime.now();           // solo ora: 10:05:32
LocalDateTime adesso = LocalDateTime.now(); // data + ora

// Creare date specifiche
LocalDate natale = LocalDate.of(2025, 12, 25);
LocalTime mezzanotte = LocalTime.of(0, 0, 0);

// Operazioni sulle date
LocalDate domani = oggi.plusDays(1);
LocalDate mesiPrecedente = oggi.minusMonths(1);
boolean ePassato = natale.isBefore(oggi);

// Formattare
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
String formattata = adesso.format(fmt);  // "13/11/2025 10:05"

// Parsare
LocalDateTime parsed = LocalDateTime.parse("13/11/2025 10:05", fmt);

// Differenza tra date
long giorni = ChronoUnit.DAYS.between(oggi, natale);
System.out.println("Giorni a Natale: " + giorni);
```

::: {.callout-tip}
## Perché `java.time` e non `Date`?
La vecchia classe `java.util.Date` è piena di problemi: mutabile, con mesi che partono da 0, con gestione del fuso orario confusa. `java.time` (introdotto in Java 8) è la sostituzione moderna — immutabile, chiara e corretta. Usa sempre `java.time`.
:::

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- **`Path`** rappresenta un percorso nel filesystem — non implica che il file esista.
- **`Files`** è la classe di utilità per operare sui file: leggere, scrivere, copiare, spostare, cancellare.
- `Files.readAllLines()` e `Files.readString()` per lettura; `Files.writeString()` e `Files.write()` per scrittura.
- `StandardOpenOption.APPEND` per aggiungere senza sovrascrivere.
- Un **log** è un registro cronologico di eventi — scritto in append con timestamp e livello.
- **`java.time`** (`LocalDate`, `LocalTime`, `LocalDateTime`) per gestire date e ore — mai `java.util.Date`.
- Combina `Files.lines()` con gli **Stream** per analizzare file di grandi dimensioni in modo efficiente.
:::

Nella prossima lezione vedremo come gestire la scrittura con i **buffer** — strumento essenziale per scrivere file in modo efficiente.