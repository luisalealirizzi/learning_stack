# Stream e pipeline

---

## Un nuovo modo di pensare alle collections

Nelle lezioni precedenti abbiamo visto come gestire le collections con cicli `for`, `while` e metodi espliciti. Funziona, ma il codice può diventare verboso. Considera questo esempio — trovare i nomi dei personaggi di livello superiore a 3, in ordine alfabetico:

```java
// Approccio tradizionale
List<Personaggio> risultato = new ArrayList<>();
for (Personaggio p : squadra) {
    if (p.getLivello() > 3) {
        risultato.add(p);
    }
}
Collections.sort(risultato, (a, b) -> a.getNome().compareTo(b.getNome()));

List<String> nomi = new ArrayList<>();
for (Personaggio p : risultato) {
    nomi.add(p.getNome());
}
```

Con gli **Stream** lo stesso risultato si scrive così:

```java
// Approccio con Stream
List<String> nomi = squadra.stream()
    .filter(p -> p.getLivello() > 3)
    .sorted((a, b) -> a.getNome().compareTo(b.getNome()))
    .map(Personaggio::getNome)
    .collect(Collectors.toList());
```

Stesso risultato, meno codice, più leggibile. Vediamo come funziona.

---

## Cos'è uno Stream

Uno **Stream** è una sequenza di elementi su cui si applicano operazioni in **pipeline** — una catena di trasformazioni che si eseguono una dopo l'altra.

Uno Stream non è una struttura dati — non memorizza elementi. È un flusso di dati che viene elaborato e trasformato.

::: {.callout-note}
## Le tre fasi di uno Stream
1. **Sorgente** — da dove arrivano i dati (`List`, `Set`, array, file...)
2. **Operazioni intermedie** — trasformazioni che producono un nuovo Stream (`filter`, `map`, `sorted`...)
3. **Operazione terminale** — produce il risultato finale (`collect`, `count`, `forEach`, `reduce`...)
:::

```
sorgente → [op. intermedia] → [op. intermedia] → [op. terminale] → risultato
squadra  →     filter()     →      map()        →    collect()   → List<String>
```

---

## Creare uno Stream

```java
import java.util.stream.*;

List<Personaggio> squadra = new ArrayList<>();
// ...

// Da una List o Set
Stream<Personaggio> s1 = squadra.stream();

// Da un array
String[] nomi = {"Arthas", "Gandalf", "Legolas"};
Stream<String> s2 = Arrays.stream(nomi);

// Stream di valori diretti
Stream<String> s3 = Stream.of("uno", "due", "tre");

// Stream di interi
IntStream numeri = IntStream.range(1, 10);  // 1, 2, 3, ..., 9
IntStream numeri2 = IntStream.rangeClosed(1, 10); // 1, 2, 3, ..., 10
```

---

## Lambda e method reference

Prima di vedere le operazioni, capiamo la sintassi che si usa con gli Stream.

**Lambda** — una funzione anonima compatta:

```java
// Sintassi: (parametri) -> espressione
p -> p.getLivello() > 3           // predicate: prende un Personaggio, restituisce boolean
p -> p.getNome()                   // function: prende un Personaggio, restituisce String
(a, b) -> a.getNome().compareTo(b.getNome())  // comparator
```

**Method reference** — abbreviazione quando il lambda chiama solo un metodo:

```java
// Invece di: p -> p.getNome()
Personaggio::getNome   // method reference su istanza

// Invece di: s -> System.out.println(s)
System.out::println    // method reference statico

// Invece di: s -> s.toUpperCase()
String::toUpperCase    // method reference su istanza
```

---

## Operazioni intermedie

Le operazioni intermedie trasformano lo Stream in un altro Stream. Sono **lazy** — non vengono eseguite finché non arriva un'operazione terminale.

### filter()

Mantiene solo gli elementi che soddisfano una condizione:

```java
// Solo personaggi di livello superiore a 3
squadra.stream()
    .filter(p -> p.getLivello() > 3)
    ...

// Solo personaggi in vita
squadra.stream()
    .filter(p -> !p.isMorto())
    ...

// Solo personaggi con nome che inizia per A
squadra.stream()
    .filter(p -> p.getNome().startsWith("A"))
    ...
```

### map()

Trasforma ogni elemento in qualcos'altro:

```java
// Da Stream<Personaggio> a Stream<String> (solo i nomi)
squadra.stream()
    .map(p -> p.getNome())
    ...

// Equivalente con method reference
squadra.stream()
    .map(Personaggio::getNome)
    ...

// Da Stream<Personaggio> a Stream<Integer> (solo i livelli)
squadra.stream()
    .map(Personaggio::getLivello)
    ...

// Per ottenere IntStream (più efficiente con interi)
squadra.stream()
    .mapToInt(Personaggio::getLivello)
    ...
```

### sorted()

Ordina gli elementi:

```java
// Ordine naturale (richiede Comparable)
numeri.stream()
    .sorted()
    ...

// Con Comparator
squadra.stream()
    .sorted((a, b) -> a.getNome().compareTo(b.getNome()))
    ...

// Con Comparator.comparing (più leggibile)
squadra.stream()
    .sorted(Comparator.comparing(Personaggio::getNome))
    ...

// Ordine inverso
squadra.stream()
    .sorted(Comparator.comparing(Personaggio::getLivello).reversed())
    ...
```

### limit() e skip()

```java
// Prendi solo i primi 3 elementi
squadra.stream()
    .limit(3)
    ...

// Salta i primi 2 elementi
squadra.stream()
    .skip(2)
    ...

// Paginazione: seconda pagina di 3 elementi
squadra.stream()
    .skip(3)
    .limit(3)
    ...
```

### distinct() e peek()

```java
// Rimuove i duplicati
List<Integer> livelli = squadra.stream()
    .map(Personaggio::getLivello)
    .distinct()
    .collect(Collectors.toList());

// peek() per debug — esegue un'azione senza modificare lo Stream
squadra.stream()
    .filter(p -> p.getLivello() > 2)
    .peek(p -> System.out.println("Dopo filter: " + p.getNome()))
    .map(Personaggio::getNome)
    .peek(n -> System.out.println("Dopo map: " + n))
    .collect(Collectors.toList());
```

---

## Operazioni terminali

Le operazioni terminali consumano lo Stream e producono un risultato. Dopo un'operazione terminale, lo Stream non può essere riusato.

### collect()

Raccoglie gli elementi in una struttura dati:

```java
import java.util.stream.Collectors;

// In una List
List<String> nomi = squadra.stream()
    .map(Personaggio::getNome)
    .collect(Collectors.toList());

// In un Set (rimuove duplicati)
Set<Integer> livelli = squadra.stream()
    .map(Personaggio::getLivello)
    .collect(Collectors.toSet());

// In una stringa con separatore
String nomiuniti = squadra.stream()
    .map(Personaggio::getNome)
    .collect(Collectors.joining(", "));  // "Arthas, Gandalf, Legolas"

// In una mappa nome → livello
Map<String, Integer> mappa = squadra.stream()
    .collect(Collectors.toMap(
        Personaggio::getNome,
        Personaggio::getLivello
    ));

// Raggruppati per livello
Map<Integer, List<Personaggio>> perLivello = squadra.stream()
    .collect(Collectors.groupingBy(Personaggio::getLivello));
```

### forEach()

Esegue un'azione su ogni elemento — non restituisce niente:

```java
squadra.stream()
    .filter(p -> !p.isMorto())
    .forEach(p -> System.out.println(p.getNome() + " è in vita"));

// Equivalente con method reference
squadra.stream()
    .forEach(Personaggio::stampaScheda);
```

### count(), sum(), average(), min(), max()

```java
// Conteggio
long numVivi = squadra.stream()
    .filter(p -> !p.isMorto())
    .count();

// Somma, media su interi
int vitaTotale = squadra.stream()
    .mapToInt(Personaggio::getVita)
    .sum();

double mediaLivelli = squadra.stream()
    .mapToInt(Personaggio::getLivello)
    .average()
    .orElse(0.0);  // orElse perché average restituisce Optional

// Minimo e massimo
Optional<Personaggio> piuForte = squadra.stream()
    .max(Comparator.comparing(Personaggio::getAttacco));

piuForte.ifPresent(p ->
    System.out.println("Più forte: " + p.getNome()));
```

### anyMatch(), allMatch(), noneMatch()

```java
// Almeno uno soddisfa la condizione?
boolean ceMagoVivo = squadra.stream()
    .anyMatch(p -> !p.isMorto());

// Tutti soddisfano la condizione?
boolean tuttiAlti = squadra.stream()
    .allMatch(p -> p.getLivello() >= 3);

// Nessuno soddisfa la condizione?
boolean nessunoMorto = squadra.stream()
    .noneMatch(Personaggio::isMorto);
```

### findFirst() e findAny()

```java
// Primo personaggio di livello > 3
Optional<Personaggio> primo = squadra.stream()
    .filter(p -> p.getLivello() > 3)
    .findFirst();

primo.ifPresent(p -> System.out.println("Trovato: " + p.getNome()));

// Gestione esplicita dell'Optional
String nome = primo
    .map(Personaggio::getNome)
    .orElse("Nessuno trovato");
```

---

## Optional: gestire l'assenza di valore

Molte operazioni terminali restituiscono `Optional<T>` — un contenitore che può contenere un valore oppure essere vuoto. Evita i `NullPointerException`.

```java
Optional<Personaggio> forse = squadra.stream()
    .filter(p -> p.getNome().equals("Sauron"))
    .findFirst();

// Verifica se presente
if (forse.isPresent()) {
    System.out.println(forse.get().getNome());
}

// Versione compatta
forse.ifPresent(p -> System.out.println(p.getNome()));

// Valore di default se assente
Personaggio p = forse.orElse(new Personaggio("Default", 1, 1, 1));

// Eccezione se assente
Personaggio p2 = forse.orElseThrow(() ->
    new IllegalStateException("Personaggio non trovato!"));
```

---

## Pipeline complete: esempi reali

### Classifica dei sopravvissuti

```java
List<String> classifica = squadra.stream()
    .filter(p -> !p.isMorto())
    .sorted(Comparator.comparing(Personaggio::getLivello).reversed())
    .map(p -> p.getNome() + " (Lv." + p.getLivello() + ")")
    .collect(Collectors.toList());

System.out.println("Sopravvissuti in ordine di livello:");
classifica.forEach(System.out::println);
```

### Statistiche della squadra

```java
IntSummaryStatistics stats = squadra.stream()
    .mapToInt(Personaggio::getVita)
    .summaryStatistics();

System.out.println("Vita minima:  " + stats.getMin());
System.out.println("Vita massima: " + stats.getMax());
System.out.println("Vita media:   " + stats.getAverage());
System.out.println("Vita totale:  " + stats.getSum());
System.out.println("Personaggi:   " + stats.getCount());
```

### Dal file al report

Uno stream può elaborare dati da qualsiasi sorgente — anche righe di testo che rappresentano dati:

```java
import java.util.stream.*;

// Simula righe lette da un file CSV: "nome,livello,vita"
List<String> righe = List.of(
    "Arthas,3,150",
    "Gandalf,7,100",
    "Legolas,5,120",
    "Sauron,10,500"
);

// Parse e filtra solo i personaggi di livello >= 5
Map<String, Integer> report = righe.stream()
    .map(riga -> riga.split(","))          // split ogni riga
    .filter(parti -> Integer.parseInt(parti[1]) >= 5) // livello >= 5
    .collect(Collectors.toMap(
        parti -> parti[0],                 // chiave: nome
        parti -> Integer.parseInt(parti[2]) // valore: vita
    ));

System.out.println("Personaggi di alto livello:");
report.forEach((nome, vita) ->
    System.out.println("  " + nome + " → vita: " + vita));
```

Output:
```
Personaggi di alto livello:
  Gandalf → vita: 100
  Legolas → vita: 120
  Sauron → vita: 500
```

---

## Stream paralleli

Per collezioni molto grandi puoi usare `parallelStream()` al posto di `stream()` — Java distribuisce automaticamente il lavoro su più thread:

```java
// Stream sequenziale
long count = squadra.stream()
    .filter(p -> p.getLivello() > 3)
    .count();

// Stream parallelo — più veloce su grandi collezioni
long countParallelo = squadra.parallelStream()
    .filter(p -> p.getLivello() > 3)
    .count();
```

::: {.callout-warning}
## Quando usare parallelStream
Il parallelismo ha un overhead — per collezioni piccole è spesso più lento. Usalo solo con collezioni di migliaia di elementi e operazioni computazionalmente costose. E mai con operazioni che hanno effetti collaterali (come modificare una lista condivisa).
:::

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- Uno **Stream** è un flusso di dati che si elabora in pipeline — non memorizza dati.
- Le tre fasi: **sorgente** → **operazioni intermedie** → **operazione terminale**.
- Le operazioni intermedie sono **lazy** — eseguite solo quando arriva la terminale.
- **`filter()`** seleziona, **`map()`** trasforma, **`sorted()`** ordina, **`limit()`** tronca.
- **`collect()`** raccoglie in una struttura, **`forEach()`** itera, **`count()`** conta.
- **`Optional`** gestisce l'assenza di valore in modo sicuro, senza `null`.
- I **lambda** e i **method reference** rendono il codice compatto e leggibile.
- **`parallelStream()`** per collezioni grandi — con cautela.
:::