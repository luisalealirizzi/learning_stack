# List, Set e Queue

---

## Tre strutture, tre problemi diversi

Nella lezione precedente abbiamo visto `ArrayList` e `HashMap`. Il Collections Framework offre però molte altre strutture, ognuna progettata per un caso d'uso specifico.

La scelta della struttura giusta non è un dettaglio — influenza la leggibilità del codice, la correttezza logica e le performance. La domanda da farsi è sempre:

- Ho bisogno di **ordine** e **duplicati**? → **List**
- Ho bisogno di **unicità**? → **Set**
- Ho bisogno di processare elementi in **sequenza**? → **Queue**

---

## List: sequenza ordinata

L'interfaccia `List` rappresenta una sequenza ordinata di elementi dove i **duplicati sono ammessi**. Ogni elemento ha un indice.

### ArrayList vs LinkedList

Le due implementazioni principali hanno caratteristiche diverse:

| | ArrayList | LinkedList |
|---|---|---|
| **Accesso per indice** | ✅ Veloce O(1) | ❌ Lento O(n) |
| **Inserimento in mezzo** | ❌ Lento O(n) | ✅ Veloce O(1) |
| **Inserimento in fondo** | ✅ Veloce | ✅ Veloce |
| **Memoria** | Meno overhead | Più overhead |
| **Quando usarla** | Lettura frequente | Inserimenti/rimozioni frequenti |

Per la maggior parte dei casi d'uso, `ArrayList` è la scelta migliore. `LinkedList` conviene solo quando fai molti inserimenti e rimozioni in posizioni intermedie.

```java
import java.util.*;

// ArrayList — accesso rapido per indice
List<String> storico = new ArrayList<>();
storico.add("Arthas ha ucciso il Lich King");
storico.add("Gandalf ha trovato l'Anello");
storico.add("Legolas ha abbattuto l'elefante");

System.out.println(storico.get(1));  // accesso rapido per indice

// LinkedList — inserimenti rapidi
List<String> coda = new LinkedList<>();
coda.add("evento1");
coda.add(0, "evento0");  // inserimento in testa — efficiente con LinkedList
```

### Metodi aggiuntivi di List

Oltre a quelli visti nella lezione precedente, `List` offre:

```java
List<String> nomi = new ArrayList<>(Arrays.asList("Arthas", "Gandalf", "Legolas"));

// Sottolist (vista sulla lista originale)
List<String> primi = nomi.subList(0, 2);  // ["Arthas", "Gandalf"]

// Indice dell'ultima occorrenza
nomi.add("Arthas");
int ultimo = nomi.lastIndexOf("Arthas");  // 3

// Convertire in array
String[] arr = nomi.toArray(new String[0]);

// Sostituire tutti gli elementi
nomi.replaceAll(String::toUpperCase);  // ["ARTHAS", "GANDALF", "LEGOLAS", "ARTHAS"]

// Rimuovere con condizione
nomi.removeIf(n -> n.startsWith("A")); // rimuove tutti che iniziano con A
```

---

## Set: nessun duplicato

L'interfaccia `Set` rappresenta una collezione **senza duplicati**. Perfetta quando vuoi garantire l'unicità degli elementi.

### HashSet

`HashSet` è l'implementazione più comune — veloce ma senza ordine garantito:

```java
import java.util.*;

Set<String> giocatoriConnessi = new HashSet<>();

giocatoriConnessi.add("Arthas");
giocatoriConnessi.add("Gandalf");
giocatoriConnessi.add("Arthas");  // ignorato — già presente!

System.out.println(giocatoriConnessi.size());           // 2, non 3
System.out.println(giocatoriConnessi.contains("Arthas")); // true

// Iterazione — l'ordine NON è garantito
for (String nome : giocatoriConnessi) {
    System.out.println(nome);
}
```

### TreeSet

`TreeSet` mantiene gli elementi **ordinati** (ordine naturale o con Comparator):

```java
Set<String> nomiOrdinati = new TreeSet<>();
nomiOrdinati.add("Legolas");
nomiOrdinati.add("Arthas");
nomiOrdinati.add("Gandalf");
nomiOrdinati.add("Arthas");  // ignorato

// Stampa in ordine alfabetico: Arthas, Gandalf, Legolas
for (String nome : nomiOrdinati) {
    System.out.println(nome);
}

// Metodi specifici di TreeSet
TreeSet<Integer> punteggi = new TreeSet<>();
punteggi.add(150);
punteggi.add(320);
punteggi.add(80);
punteggi.add(210);

System.out.println(punteggi.first());  // 80 — minimo
System.out.println(punteggi.last());   // 320 — massimo
System.out.println(punteggi.headSet(200)); // elementi < 200: [80, 150]
System.out.println(punteggi.tailSet(200)); // elementi >= 200: [210, 320]
```

### Operazioni tra insiemi

I `Set` supportano naturalmente le operazioni insiemistiche:

```java
Set<String> squadraA = new HashSet<>(Arrays.asList("Arthas", "Gandalf", "Legolas"));
Set<String> squadraB = new HashSet<>(Arrays.asList("Gandalf", "Aragorn", "Frodo"));

// Unione
Set<String> unione = new HashSet<>(squadraA);
unione.addAll(squadraB);  // tutti i giocatori unici

// Intersezione
Set<String> intersezione = new HashSet<>(squadraA);
intersezione.retainAll(squadraB);  // solo i giocatori in entrambe le squadre

// Differenza
Set<String> differenza = new HashSet<>(squadraA);
differenza.removeAll(squadraB);  // giocatori in A ma non in B

System.out.println("Unione: " + unione);             // [Arthas, Gandalf, Legolas, Aragorn, Frodo]
System.out.println("Intersezione: " + intersezione); // [Gandalf]
System.out.println("Differenza: " + differenza);     // [Arthas, Legolas]
```

::: {.callout-tip}
## HashSet vs TreeSet: quando usare quale

Usa **`HashSet`** quando:
- non ti interessa l'ordine
- vuoi la massima velocità (ricerca, inserimento, rimozione in O(1))

Usa **`TreeSet`** quando:
- vuoi gli elementi sempre ordinati
- hai bisogno di operazioni come "dammi tutti gli elementi tra X e Y"
:::

---

## Queue: la coda

L'interfaccia `Queue` rappresenta una struttura **FIFO** (First In, First Out) — il primo elemento inserito è il primo ad essere estratto. Come una fila al supermercato.

```
Inserimento → [C] [B] [A] → Estrazione
               ^             ^
             coda           testa
```

### Metodi principali di Queue

| Metodo | Comportamento se vuota |
|---|---|
| `add(e)` / `offer(e)` | Inserisce in coda (`add` lancia eccezione, `offer` no) |
| `remove()` / `poll()` | Estrae dalla testa (`remove` lancia eccezione, `poll` no) |
| `element()` / `peek()` | Legge la testa senza estrarre (`element` lancia eccezione, `peek` no) |

La convenzione è usare `offer`, `poll` e `peek` — non lanciano eccezioni ma restituiscono `null`.

```java
import java.util.*;

Queue<String> codaEventi = new LinkedList<>();

// Inserimento in coda
codaEventi.offer("Arthas attacca");
codaEventi.offer("Gandalf lancia incantesimo");
codaEventi.offer("Legolas scocca freccia");

System.out.println("Prossimo evento: " + codaEventi.peek()); // legge senza rimuovere

// Elaborazione degli eventi in ordine FIFO
while (!codaEventi.isEmpty()) {
    String evento = codaEventi.poll();  // estrae dalla testa
    System.out.println("Elaboro: " + evento);
}
```

### PriorityQueue: coda con priorità

`PriorityQueue` elabora gli elementi non in ordine di inserimento, ma in base a una **priorità**:

```java
// PriorityQueue di default: ordine naturale (minimo prima)
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(50);
pq.offer(10);
pq.offer(30);

while (!pq.isEmpty()) {
    System.out.println(pq.poll());  // 10, 30, 50 — ordine crescente
}
```

Con oggetti personalizzati puoi definire la priorità con un `Comparator`:

```java
// Coda di personaggi ordinata per vita (prima i più feriti)
PriorityQueue<Personaggio> codaCura = new PriorityQueue<>(
    (a, b) -> a.getVita() - b.getVita()  // vita minore = priorità più alta
);

codaCura.offer(new Personaggio("Arthas", 150, 30, 1));
codaCura.offer(new Personaggio("Gandalf", 100, 20, 3));
codaCura.offer(new Personaggio("Legolas", 120, 25, 2));

// Il mago con 20 di vita viene curato per primo
while (!codaCura.isEmpty()) {
    Personaggio p = codaCura.poll();
    System.out.println("Curo " + p.getNome() + " (vita: " + p.getVita() + ")");
}
```

Output:
```
Curo Gandalf (vita: 20)
Curo Legolas (vita: 25)
Curo Arthas (vita: 30)
```

::: {.callout-note}
## Queue come Deque
`LinkedList` implementa anche l'interfaccia `Deque` (Double Ended Queue) — una coda a doppia estremità da cui si può inserire ed estrarre sia dalla testa che dalla coda. Questo la rende utile anche come **stack** (LIFO):

```java
Deque<String> stack = new LinkedList<>();
stack.push("primo");   // inserisce in testa
stack.push("secondo");
stack.push("terzo");

System.out.println(stack.pop());  // "terzo" — LIFO
System.out.println(stack.pop());  // "secondo"
```
:::

---

## Confronto finale: quale struttura scegliere?

| Struttura | Ordine | Duplicati | Accesso | Caso d'uso tipico |
|---|---|---|---|---|
| `ArrayList` | Inserimento | ✅ Sì | Per indice | Lista generica, accesso frequente |
| `LinkedList` | Inserimento | ✅ Sì | Sequenziale | Molti inserimenti/rimozioni |
| `HashSet` | Nessuno | ❌ No | Per valore | Unicità, ricerca rapida |
| `TreeSet` | Ordinato | ❌ No | Per valore | Unicità + ordine |
| `HashMap` | Nessuno | Chiavi uniche | Per chiave | Dizionario, lookup rapido |
| `TreeMap` | Chiavi ordinate | Chiavi uniche | Per chiave | Dizionario + ordine |
| `LinkedList` (Queue) | FIFO | ✅ Sì | Testa/coda | Coda di eventi |
| `PriorityQueue` | Priorità | ✅ Sì | Per priorità | Scheduling, cura prioritaria |

---

## Esempio completo: sistema di turni

Un sistema di turni per un gioco — i personaggi agiscono in ordine di iniziativa (più alta prima):

```java
import java.util.*;

public class SistemaTurni {
    public static void main(String[] args) {

        // Coda con priorità: più alta l'iniziativa, prima si agisce
        PriorityQueue<Personaggio> turni = new PriorityQueue<>(
            (a, b) -> b.getLivello() - a.getLivello()  // livello più alto = prima
        );

        turni.offer(new Personaggio("Arthas", 150, 30, 3));
        turni.offer(new Personaggio("Gandalf", 100, 20, 7));
        turni.offer(new Personaggio("Legolas", 120, 25, 5));

        Personaggio boss = new Personaggio("Lich King", 500, 40, 10);

        System.out.println("=== TURNO DI COMBATTIMENTO ===\n");

        // Set per tenere traccia dei nomi già estratti (solo per demo)
        Set<String> haAgito = new HashSet<>();

        while (!turni.isEmpty() && !boss.isMorto()) {
            Personaggio attaccante = turni.poll();
            System.out.print("[Lv." + attaccante.getLivello() + "] ");
            attaccante.attacca(boss);
            haAgito.add(attaccante.getNome());
        }

        System.out.println("\nHanno agito: " + haAgito);
        System.out.println("Vita del boss: " + boss.getVita() + "/" + boss.getVitaMax());
    }
}
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- **`List`** — sequenza ordinata con duplicati. `ArrayList` per accesso rapido, `LinkedList` per inserimenti frequenti.
- **`Set`** — nessun duplicato. `HashSet` per velocità, `TreeSet` per ordine.
- **`Queue`** — elaborazione FIFO. `LinkedList` come coda semplice, `PriorityQueue` per priorità.
- Le operazioni insiemistiche (`addAll`, `retainAll`, `removeAll`) funzionano su tutti i `Set`.
- Preferisci i metodi "safe" (`offer`, `poll`, `peek`) a quelli che lanciano eccezioni (`add`, `remove`, `element`).
- La scelta della struttura giusta dipende dal **caso d'uso**: ordine, unicità, tipo di accesso.
:::