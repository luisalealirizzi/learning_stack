# Buffer e scrittura

---

## Il principio del buffer

Nella lezione precedente abbiamo usato `Files.writeString()` e `Files.readAllLines()` — comode e leggibili. Ma c'è un principio importante che vale sia per i file che per le stringhe:

> **Evitare tante operazioni piccole e costose, accumulando i dati prima di scrivere o elaborare.**

Accedere al disco è costoso. Farlo molte volte per piccoli pezzi di dati è lento. Il **buffer** è una zona di memoria RAM usata come zona di appoggio tra il programma e il file: accumula i dati e li scarica sul disco in un'unica operazione, riducendo drasticamente il numero di accessi.

Quello che cambia tra le diverse API è **chi gestisce il buffer** e **quanto è visibile**.

---

## Buffer impliciti: le API di alto livello

Quando usi metodi come:

```java
Files.readString(p);
Files.readAllLines(p);
Files.writeString(p, testo);
```

il buffering esiste, ma è completamente **nascosto**. Java legge o scrive il file usando buffer interni, poi ti restituisce il risultato finito.

Questi metodi sono comodi, sicuri e leggibili. Ma implicano una scelta precisa: **tutto il contenuto viene caricato in memoria**. Sono indicati per file piccoli — configurazioni, testi brevi, output di report.

::: {.callout-note}
## L'idea di base in Java moderno
Non si parte dai buffer. Si parte da API di alto livello, già ottimizzate, e si scende solo quando serve — cioè quando il file è grande, quando si lavora riga per riga con logica complessa, o quando si vuole controllo fine sul flusso dei dati.
:::

---

## Il primo passo verso il buffer: `Files.lines()`

`Files.lines()` è già un passo verso il buffering: non carica tutto, consuma i dati mentre arrivano.

```java
try (Stream<String> righe = Files.lines(p)) {
    righe.forEach(System.out::println);
}
```

Questo codice può stampare un file enorme senza mai tenerlo tutto in memoria. Il buffer c'è, ma non lo vediamo.

---

## Buffer espliciti su testo: BufferedReader e BufferedWriter

Quando vogliamo controllo esplicito, usiamo `BufferedReader` e `BufferedWriter`. Il buffer non è più un dettaglio tecnico nascosto — diventa parte del nostro modello.

### BufferedReader: leggere riga per riga

```java
import java.nio.file.*;
import java.io.*;

Path p = Path.of("nomi.txt");

try (BufferedReader br = Files.newBufferedReader(p)) {
    String riga;
    while ((riga = br.readLine()) != null) {
        System.out.println("Letto: " + riga);
    }
}
```

Dichiarando `BufferedReader` stai dicendo esplicitamente tre cose:
1. il file viene letto **a righe**, non come blocco unico
2. il file viene **consumato progressivamente**
3. c'è un'area di memoria che accumula dati prima di consegnarteli — il buffer è parte del tuo modello

Il `try-with-resources` garantisce che il file venga aperto una sola volta, letto progressivamente, e **chiuso automaticamente** alla fine — anche se scatta un'eccezione nel mezzo.

### Gestione delle eccezioni dentro il try

```java
Path p = Path.of("numeri.txt");

try (BufferedReader br = Files.newBufferedReader(p)) {
    String riga;
    while ((riga = br.readLine()) != null) {
        int n = Integer.parseInt(riga);  // può lanciare NumberFormatException
        System.out.println(n * 2);
    }
} catch (IOException e) {
    System.out.println("Errore di lettura file");
}
```

Se una riga non è un numero, il programma va in errore. Ma il file viene comunque chiuso. Questo è il punto chiave: **la chiusura della risorsa non dipende dal fatto che il codice finisca bene**. La gestione delle risorse è separata dalla gestione degli errori.

### Due risorse nello stesso try

```java
Path src = Path.of("input.txt");
Path dst = Path.of("output.txt");

try (
    BufferedReader br = Files.newBufferedReader(src);
    BufferedWriter bw = Files.newBufferedWriter(dst)
) {
    String riga;
    while ((riga = br.readLine()) != null) {
        bw.write(riga.toUpperCase());
        bw.newLine();
    }
}
```

Java garantisce che prima venga chiuso il writer (svuotando i buffer di scrittura), poi il reader. Questo ordine è importante — e tu non devi ricordartelo: è parte del contratto del try-with-resources.

### BufferedWriter: scrivere con buffer

```java
Path file = Path.of("output.txt");

try (BufferedWriter bw = Files.newBufferedWriter(file)) {
    for (int i = 1; i <= 1000; i++) {
        bw.write("Riga " + i);
        bw.newLine();  // usa il separatore corretto per il sistema operativo
    }
}
```

Invece di mille accessi al disco, il buffer accumula le righe in memoria e le scarica in poche operazioni. Molto più efficiente.

::: {.callout-note}
## `newLine()` vs `"\n"`
Usa sempre `bw.newLine()` invece di `"\n"`. Il metodo scrive il carattere di a capo corretto per il sistema operativo: `\n` su Linux/Mac, `\r\n` su Windows.
:::

### Append con BufferedWriter

```java
try (BufferedWriter bw = Files.newBufferedWriter(file,
        StandardOpenOption.APPEND,
        StandardOpenOption.CREATE)) {
    bw.write("Riga aggiunta");
    bw.newLine();
}
```

### PrintWriter: testo formattato

`PrintWriter` aggiunge `println()` e `printf()` — ideale per report tabellari:

```java
try (PrintWriter pw = new PrintWriter(Files.newBufferedWriter(file))) {
    pw.println("=== REPORT ===");
    pw.printf("%-15s %5s %5s%n", "Nome", "Vita", "Lv");
    pw.printf("%-15s %5d %5d%n", "Arthas", 150, 5);
    pw.printf("%-15s %5d %5d%n", "Gandalf", 100, 7);
}
```

---

## L'analogia tra file e stringhe

Il principio del buffer vale in entrambi i contesti:

| Situazione | Senza buffer | Con buffer |
|---|---|---|
| **File** | scrittura continua byte per byte | `BufferedWriter` |
| **Stringhe** | concatenazione in ciclo con `+` | `StringBuilder` |

`BufferedWriter` bufferizza il **disco**. `StringBuilder` bufferizza la **memoria**. Stessa idea, contesti diversi.

---

## Buffer su flussi binari

Per i file di testo ha senso parlare di righe e caratteri. Per i **file binari** — immagini, audio, zip, file `.class` — no: sono solo sequenze di byte, senza significato testuale.

In quel caso si usano `BufferedInputStream` e `BufferedOutputStream`:

```java
import java.io.*;
import java.nio.file.*;

Path src = Path.of("immagine.png");
Path dst = Path.of("copia.png");

try (
    BufferedInputStream in  = new BufferedInputStream(Files.newInputStream(src));
    BufferedOutputStream out = new BufferedOutputStream(Files.newOutputStream(dst))
) {
    byte[] buffer = new byte[4096];  // il buffer è concreto e visibile
    int letti;
    while ((letti = in.read(buffer)) != -1) {
        out.write(buffer, 0, letti);
    }
}
```

Qui il buffer non è più astratto: è un array di byte che rappresenta fisicamente quanti dati leggi alla volta. `read()` restituisce il numero di byte effettivamente letti — l'ultima lettura può essere parziale, quindi si scrive solo `letti` byte, non tutti i 4096.

::: {.callout-tip}
## Perché 4096?
4096 byte (4 KB) è una dimensione comune perché corrisponde tipicamente alla dimensione di un blocco del filesystem. Leggere o scrivere blocchi allineati al filesystem è più efficiente.
:::

---

## Buffer specializzati: DataInputStream e DataOutputStream

`DataInputStream` e `DataOutputStream` non introducono un nuovo tipo di buffer — si appoggiano a uno già esistente. La loro specialità è leggere e scrivere **tipi primitivi in formato binario**: `int`, `double`, `boolean`, `long` e così via.

Senza questi stream, per salvare un intero dovresti convertirlo in stringa, scriverlo, poi rileggerlo e convertirlo di nuovo. Con `DataOutputStream` scrivi direttamente il valore binario — più compatto e più veloce.

```java
import java.io.*;
import java.nio.file.*;

Path file = Path.of("dati.bin");

// Scrittura — avvolge un BufferedOutputStream
try (DataOutputStream out = new DataOutputStream(
        new BufferedOutputStream(Files.newOutputStream(file)))) {

    out.writeInt(42);           // scrive 4 byte
    out.writeDouble(3.14);      // scrive 8 byte
    out.writeBoolean(true);     // scrive 1 byte
    out.writeUTF("Arthas");     // scrive la stringa in formato UTF-8 binario

}

// Lettura — avvolge un BufferedInputStream
try (DataInputStream in = new DataInputStream(
        new BufferedInputStream(Files.newInputStream(file)))) {

    int n       = in.readInt();
    double d    = in.readDouble();
    boolean b   = in.readBoolean();
    String nome = in.readUTF();

    System.out.println(n + " " + d + " " + b + " " + nome);
    // 42 3.14 true Arthas
}
```

::: {.callout-important}
## L'ordine di lettura deve essere identico all'ordine di scrittura
Il file binario non ha metadati — è solo una sequenza di byte. Java non sa cosa rappresenta ogni byte finché non glielo dici tu. Se scrivi `int`, `double`, `boolean` in quest'ordine, devi leggere esattamente nello stesso ordine. Un errore di ordine produce valori senza senso, senza nessun messaggio di errore.
:::

### Quando usarli

`DataInputStream`/`DataOutputStream` sono utili quando:
- vuoi salvare dati strutturati in formato **compatto e non leggibile** da un editor di testo
- stai implementando un **formato di salvataggio binario** (es. file di salvataggio di un gioco)
- lavori con **protocolli di rete** che trasmettono dati primitivi in formato binario

Non usarli quando i dati devono essere leggibili da un essere umano o da altri programmi — in quel caso usa CSV o JSON in formato testo.

---

## Il quadro completo dei buffer

Ricapitolando tutti i livelli che abbiamo visto:

| Strumento | Gestisce | Visibilità buffer | Quando usarlo |
|---|---|---|---|
| `Files.readString()` / `writeString()` | char | Implicito, nascosto | File piccoli, semplicità |
| `Files.lines()` | char | Implicito, lazy | File grandi con Stream |
| `BufferedReader` / `BufferedWriter` | char | Esplicito, riga per riga | File di testo grandi |
| `BufferedInputStream` / `BufferedOutputStream` | byte | Esplicito, array di byte | File binari, copie |
| `DataInputStream` / `DataOutputStream` | tipi primitivi | Eredita da Buffered* | Salvataggi binari strutturati |

::: {.callout-note}
## La progressione concettuale
```
Files.readString()              → buffer implicito, tutto in RAM
Files.lines()                   → buffer implicito, lazy
BufferedReader / Writer         → buffer esplicito, testo (char)
BufferedInputStream / Output    → buffer esplicito, binario (byte)
DataInputStream / Output        → buffer ereditato, tipi primitivi
```
Si scende di livello solo quando serve.
:::

## La codifica dei caratteri

Quando lavori con file di testo, specifica sempre **UTF-8** esplicitamente per evitare problemi con caratteri speciali su sistemi diversi:

```java
import java.nio.charset.StandardCharsets;

try (BufferedWriter bw = Files.newBufferedWriter(
        file, StandardCharsets.UTF_8)) {
    bw.write("Testo con accenti: à è ì ò ù");
    bw.newLine();
}

try (BufferedReader br = Files.newBufferedReader(
        file, StandardCharsets.UTF_8)) {
    System.out.println(br.readLine());
}
```

---

## Flush: svuotare il buffer manualmente

Normalmente il buffer viene svuotato automaticamente alla chiusura. Ma in alcuni casi, come un log in tempo reale, vuoi che i dati vengano scritti subito:

```java
try (BufferedWriter bw = Files.newBufferedWriter(
        Path.of("live.log"), StandardOpenOption.APPEND)) {

    for (int i = 0; i < 5; i++) {
        bw.write("Evento " + i);
        bw.newLine();
        bw.flush();  // scrive immediatamente sul disco

        Thread.sleep(1000);
    }
} catch (IOException | InterruptedException e) {
    System.out.println("Errore: " + e.getMessage());
}
```

Senza `flush()`, gli eventi rimarrebbero nel buffer e verrebbero scritti tutti insieme alla chiusura.

---

## Il quadro completo: quando usare cosa

| Strumento | Tipo | Quando usarlo |
|---|---|---|
| `Files.readString()` | testo, implicito | file piccoli, lettura rapida |
| `Files.readAllLines()` | testo, implicito | file piccoli, serve una List |
| `Files.writeString()` | testo, implicito | scrittura semplice |
| `Files.lines()` | testo, lazy | file grandi con Stream |
| `BufferedReader` | testo, esplicito | lettura riga per riga, file grandi |
| `BufferedWriter` | testo, esplicito | scrittura incrementale, molte righe |
| `PrintWriter` | testo, formattato | report con `printf` |
| `BufferedInputStream` | binario | copia file, immagini, audio |
| `BufferedOutputStream` | binario | scrittura file binari |

::: {.callout-note}
## La progressione concettuale
```
Files.readString()         → buffer implicito, tutto in RAM
Files.lines()              → buffer implicito, lazy (primo passo)
BufferedReader/Writer      → buffer esplicito su testo
BufferedInputStream/Output → buffer esplicito su byte
```
Si scende di livello solo quando serve: file grandi, prestazioni, dati binari.
:::

---

## Esempio completo: logger con buffer e rotazione giornaliera

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
    private final DateTimeFormatter fmtOra =
        DateTimeFormatter.ofPattern("HH:mm:ss");

    public Logger(String cartella) {
        this.cartella = Path.of(cartella);
        try {
            Files.createDirectories(this.cartella);
        } catch (IOException e) {
            System.out.println("Impossibile creare cartella log: " + e.getMessage());
        }
    }

    // Un file di log diverso per ogni giorno
    private Path fileOggi() {
        return cartella.resolve("gioco-" + LocalDate.now().format(fmtData) + ".log");
    }

    private void scrivi(String livello, String messaggio) {
        String riga = "[" + LocalTime.now().format(fmtOra) + "] "
                    + "[" + livello + "] " + messaggio;

        try (BufferedWriter bw = Files.newBufferedWriter(
                fileOggi(),
                StandardCharsets.UTF_8,
                StandardOpenOption.CREATE,
                StandardOpenOption.APPEND)) {

            bw.write(riga);
            bw.newLine();

        } catch (IOException e) {
            System.err.println("Errore log: " + e.getMessage());
        }
    }

    public void info(String msg)  { scrivi("INFO ", msg); }
    public void warn(String msg)  { scrivi("WARN ", msg); }
    public void error(String msg) { scrivi("ERROR", msg); }

    public void stampaLog() {
        Path file = fileOggi();
        if (!Files.exists(file)) {
            System.out.println("Nessun log per oggi.");
            return;
        }

        System.out.println("=== LOG " + LocalDate.now() + " ===");
        try (BufferedReader br = Files.newBufferedReader(file, StandardCharsets.UTF_8)) {
            String riga;
            while ((riga = br.readLine()) != null) {
                System.out.println(riga);
            }
        } catch (IOException e) {
            System.out.println("Errore lettura: " + e.getMessage());
        }
    }
}
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave

- Il **buffer** è una zona di RAM usata come appoggio tra il programma e il disco — riduce il numero di accessi.
- Le API di alto livello (`Files.readString`, `Files.writeString`) usano buffer **impliciti** — tutto in memoria, adatte per file piccoli.
- `Files.lines()` è **lazy** — primo passo verso il buffer, non carica tutto.
- **`BufferedReader`** e **`BufferedWriter`** rendono il buffer **esplicito** — per testo, riga per riga.
- Il **try-with-resources** chiude le risorse automaticamente, nell'ordine corretto, anche in caso di eccezione.
- **`BufferedInputStream`** e **`BufferedOutputStream`** per file **binari** — il buffer diventa un array di byte concreto e misurabile.
- Specifica sempre **`StandardCharsets.UTF_8`** per evitare problemi con caratteri speciali.
- Usa `flush()` per scrivere immediatamente senza aspettare la chiusura.
- Si parte dall'**alto livello** e si scende solo quando serve.
:::