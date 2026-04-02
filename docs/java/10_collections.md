# Java Collections

---

## Il problema degli array

Gli array che conosci dal C hanno un limite fondamentale: la **dimensione è fissa**. Devi decidere quanti elementi ci sono prima di creare l'array, e non puoi cambiare idea.

```java
int[] punteggi = new int[10];  // esattamente 10 elementi, né più né meno
```

Nella realtà i dati sono dinamici: una lista di giocatori connessi cresce e diminuisce, un inventario si riempie e si svuota, i risultati di una ricerca variano. Serve qualcosa di più flessibile.

Java risponde con il **Collections Framework**: un insieme di classi e interfacce per gestire collezioni di oggetti in modo dinamico, sicuro ed efficiente.

---

## La gerarchia delle Collections

Il Collections Framework è organizzato attorno a interfacce che definiscono i comportamenti, e classi che le implementano:

```
Collection
├── List          → sequenza ordinata, elementi duplicati ammessi
│   ├── ArrayList
│   └── LinkedList
├── Set           → nessun duplicato
│   ├── HashSet
│   └── TreeSet
└── Queue         → coda FIFO
    ├── LinkedList
    └── PriorityQueue

Map               → coppie chiave-valore (non estende Collection)
    ├── HashMap
    └── TreeMap
```

::: {.callout-note}
## Interfaccia vs implementazione
Nel Collections Framework programmi sempre verso l'**interfaccia** (`List`, `Set`, `Map`) e istanzi l'**implementazione** (`ArrayList`, `HashSet`, `HashMap`). Questo ti permette di cambiare implementazione senza modificare il resto del codice.

```java
List<String> nomi = new ArrayList<>();  // tipo dichiarato: List; tipo reale: ArrayList
```
:::

---

## Generics: collezioni tipizzate

Prima di usare le collections, capiamo i **generics**: la sintassi con le parentesi angolari `<>`.

I generics permettono di specificare il **tipo degli elementi** che una collezione può contenere. Il compilatore verifica che si inseriscano solo elementi del tipo corretto.

```java
List<String> nomi = new ArrayList<>();           // solo String
List<Integer> punteggi = new ArrayList<>();      // solo Integer
List<Personaggio> squadra = new ArrayList<>();   // solo Personaggio
```
Senza generics dovresti fare cast manuale ogni volta che estrai un elemento — con generics il compilatore lo fa per te e ti avvisa subito se inserisci il tipo sbagliato.

::: {.callout-tip}
## Tipi primitivi e generics
I generics funzionano solo con **oggetti**, non con tipi primitivi. Usa le classi wrapper:
- `List<int>` → ❌
- `List<Integer>` → ✅

Java si occupa dell'autoboxing automaticamente — puoi aggiungere `int` e li converte in `Integer` da solo.
:::

---

## ArrayList: la lista dinamica

`ArrayList` è la collezione più usata in Java. È come un array che si ridimensiona automaticamente.

```java
import java.util.ArrayList;
import java.util.List;

List<String> inventario = new ArrayList<>();

// Aggiungere elementi
inventario.add("Spada del Caos");
inventario.add("Scudo di Ferro");
inventario.add("Pozione di Vita");

// Dimensione
System.out.println("Oggetti: " + inventario.size()); // 3

// Accedere a un elemento per indice
String primo = inventario.get(0);  // "Spada del Caos"

// Modificare un elemento
inventario.set(1, "Scudo Leggendario");

// Rimuovere per indice
inventario.remove(2);  // rimuove "Pozione di Vita"

// Rimuovere per valore
inventario.remove("Spada del Caos");

// Verificare se contiene un elemento
boolean haSpada = inventario.contains("Spada del Caos");  // false

// Svuotare la lista
inventario.clear();

System.out.println("Vuota? " + inventario.isEmpty());  // true
```

---

## Iterare su una lista

Ci sono tre modi principali per scorrere una lista:

**For-each** — il più comune e leggibile:
```java
List<String> squadra = new ArrayList<>();
squadra.add("Arthas");
squadra.add("Gandalf");
squadra.add("Legolas");

for (String nome : squadra) {
    System.out.println("- " + nome);
}
```

**For classico con indice** — quando hai bisogno della posizione:
```java
for (int i = 0; i < squadra.size(); i++) {
    System.out.println(i + ": " + squadra.get(i));
}
```


::: {.callout-warning}
## Non rimuovere con for-each
Non rimuovere elementi da una lista mentre la stai scorrendo con un for-each — lancia `ConcurrentModificationException`. Accumula gli elementi da rimuovere e cancellali dopo.
:::

---

## HashMap: la mappa chiave-valore

`HashMap` associa a ogni **chiave** un **valore**. È perfetta quando vuoi cercare dati rapidamente per un identificatore.

```java
import java.util.HashMap;
import java.util.Map;

Map<String, Integer> punteggi = new HashMap<>();

// Inserire coppie chiave-valore
punteggi.put("Arthas", 1500);
punteggi.put("Gandalf", 2300);
punteggi.put("Legolas", 1800);

// Leggere un valore
int p = punteggi.get("Gandalf");  // 2300

// Verificare se una chiave esiste
boolean c = punteggi.containsKey("Arthas");  // true

// Aggiornare un valore
punteggi.put("Arthas", 1750);  // sovrascrive il valore esistente

// Rimuovere una coppia
punteggi.remove("Legolas");

// Valore di default se la chiave non esiste
int valore = punteggi.getOrDefault("Sconosciuto", 0);  // 0

// Dimensione
System.out.println("Giocatori: " + punteggi.size());
```

---

## Iterare su una HashMap

```java
Map<String, Integer> punteggi = new HashMap<>();
punteggi.put("Arthas", 1500);
punteggi.put("Gandalf", 2300);

// Iterare su tutte le coppie
for (Map.Entry<String, Integer> entry : punteggi.entrySet()) {
    System.out.println(entry.getKey() + " → " + entry.getValue());
}

// Iterare solo sulle chiavi
for (String nome : punteggi.keySet()) {
    System.out.println(nome);
}

// Iterare solo sui valori
for (int punti : punteggi.values()) {
    System.out.println(punti);
}
```

---

## Collections con oggetti personalizzati

Le collections funzionano benissimo con le tue classi:

```java
List<Personaggio> squadra = new ArrayList<>();

squadra.add(new Personaggio("Arthas", 150, 30, 1));
squadra.add(new Personaggio("Gandalf", 100, 20, 3));
squadra.add(new Personaggio("Legolas", 120, 25, 2));

// Stampa tutti i personaggi
for (Personaggio p : squadra) {
    p.stampaScheda();
}

// Trova i personaggi ancora in vita
List<Personaggio> inVita = new ArrayList<>();
for (Personaggio p : squadra) {
    if (!p.isMorto()) {
        inVita.add(p);
    }
}
System.out.println("Personaggi in vita: " + inVita.size());

// Cerca un personaggio per nome
Personaggio trovato = null;
for (Personaggio p : squadra) {
    if (p.getNome().equals("Gandalf")) {
        trovato = p;
        break;
    }
}
```

---

## `equals()` e `hashCode()`: il contratto fondamentale

Nella lezione sul polimorfismo hai visto che per confrontare oggetti si usa `equals()` invece di `==`. Ma c'è una regola fondamentale in Java che spesso viene dimenticata:

> **Se ridefinisci `equals()`, devi ridefinire anche `hashCode()`.**

Perché? Perché `HashMap` e `HashSet` usano `hashCode()` per organizzare gli oggetti in "bucket" — celle di memoria indicizzate. Quando cerchi un oggetto, Java prima calcola il suo `hashCode()` per trovare il bucket giusto, poi usa `equals()` per trovare l'oggetto esatto dentro quel bucket.

Se `equals()` dice che due oggetti sono uguali ma `hashCode()` restituisce valori diversi, `HashMap` e `HashSet` li mettono in bucket diversi e non riescono a trovarli correttamente.

```java
// Senza hashCode ridefinito — comportamento SBAGLIATO
Map<Personaggio, Integer> punteggi = new HashMap<>();
Personaggio p1 = new Personaggio("Arthas", 150, 30, 1);
punteggi.put(p1, 500);

Personaggio p2 = new Personaggio("Arthas", 150, 30, 1);
// p1.equals(p2) → true (stesso nome e livello)
// ma hashCode diversi → HashMap non trova p2!
System.out.println(punteggi.get(p2));  // null — non trovato!
```

La soluzione è ridefinire entrambi in modo coerente:

```java
public class Personaggio {

    private String nome;
    private int livello;
    // ...

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null) return false;
        if (!(obj instanceof Personaggio)) return false;

        Personaggio altro = (Personaggio) obj;
        return this.livello == altro.livello &&
               this.nome.equals(altro.nome);
    }

    @Override
    public int hashCode() {
        int risultato = nome.hashCode();
        risultato = 31 * risultato + livello;
        return risultato;
    }
}
```

Oppure, da Java 7 in poi, puoi usare `Objects.hash()` che fa il lavoro sporco per te:

```java
import java.util.Objects;

@Override
public int hashCode() {
    return Objects.hash(nome, livello);  // usa gli stessi campi di equals()
}
```

::: {.callout-important}
## Il contratto equals/hashCode
- Se `a.equals(b)` è `true`, allora `a.hashCode()` deve essere uguale a `b.hashCode()`
- Il contrario non è obbligatorio: due oggetti con lo stesso `hashCode` possono non essere uguali (collisione)
- Usa **sempre gli stessi campi** in `equals()` e `hashCode()`
- Se usi `Objects.hash()`, passa gli stessi campi che controlli in `equals()`
:::

Ora con `hashCode()` ridefinito correttamente:

```java
Map<Personaggio, Integer> punteggi = new HashMap<>();
Personaggio p1 = new Personaggio("Arthas", 150, 30, 1);
punteggi.put(p1, 500);

Personaggio p2 = new Personaggio("Arthas", 150, 30, 1);
System.out.println(punteggi.get(p2));  // 500 — trovato correttamente!
```

## La classe Collections (utility)

`java.util.Collections` è una classe di utilità con metodi statici per operare sulle collezioni:

```java
import java.util.Collections;

List<Integer> numeri = new ArrayList<>();
numeri.add(5);
numeri.add(2);
numeri.add(8);
numeri.add(1);

Collections.sort(numeri);          // ordina in modo crescente → [1, 2, 5, 8]
Collections.reverse(numeri);       // inverte → [8, 5, 2, 1]
Collections.shuffle(numeri);       // mescola casualmente
int max = Collections.max(numeri); // elemento massimo
int min = Collections.min(numeri); // elemento minimo
```

Per ordinare tipi primitivi e `String`, `Collections.sort()` funziona direttamente perché questi tipi implementano già `Comparable`. Per ordinare oggetti personalizzati come `Personaggio`, devi implementare `Comparable` nella tua classe — lo vediamo nella sezione successiva.

```java
// Funziona subito — String implementa già Comparable
List<String> nomi = new ArrayList<>(Arrays.asList("Legolas", "Arthas", "Gandalf"));
Collections.sort(nomi);  // ordine alfabetico → [Arthas, Gandalf, Legolas]

// Funziona dopo aver implementato Comparable in Personaggio
List<Personaggio> squadra = new ArrayList<>();
squadra.add(new Personaggio("Arthas", 150, 30, 5));
squadra.add(new Personaggio("Gandalf", 100, 20, 7));
squadra.add(new Personaggio("Legolas", 120, 25, 3));

Collections.sort(squadra);  // ordina per livello crescente (definito in compareTo)
squadra.forEach(p -> System.out.println(p.getNome() + " Lv." + p.getLivello()));
```

Output:
```
Legolas Lv.3
Arthas Lv.5
Gandalf Lv.7
```
## Comparable: definire l'ordine naturale

Nella sezione precedente hai visto che `Collections.sort()` ordina una lista. Ma come fa Java a sapere in che ordine mettere i tuoi oggetti personalizzati? Non lo sa — a meno che tu non glielo dica.

Puoi codificare l'ordine "naturale" dei tuoi oggetti direttamente nella classe implementando l'interfaccia **`Comparable`**.

### Implementare Comparable

`Comparable<T>` ha un solo metodo da implementare: `compareTo()`. Deve restituire:

- un numero **negativo** se `this` viene prima di `other`
- **zero** se sono equivalenti
- un numero **positivo** se `this` viene dopo di `other`

```java
public class Personaggio implements Comparable<Personaggio> {

    private String nome;
    private int vita;
    private int vitaMax;
    private int attacco;
    private int livello;

    // ... costruttori, getter, setter, metodi ...

    // Ordine naturale: per livello crescente
    @Override
    public int compareTo(Personaggio altro) {
        return this.livello - altro.livello;
        // negativo → this viene prima (livello più basso)
        // zero     → stesso livello
        // positivo → this viene dopo (livello più alto)
    }
}
```

Ora `Collections.sort()` sa come ordinare i personaggi senza bisogno di istruzioni aggiuntive:

```java
List<Personaggio> squadra = new ArrayList<>();
squadra.add(new Personaggio("Arthas", 150, 30, 5));
squadra.add(new Personaggio("Gandalf", 100, 20, 7));
squadra.add(new Personaggio("Legolas", 120, 25, 3));

// Funziona perché Personaggio implementa Comparable
Collections.sort(squadra);

for (Personaggio p : squadra) {
    System.out.println(p.getNome() + " (Lv." + p.getLivello() + ")");
}
```

Output:
```
Legolas (Lv.3)
Arthas (Lv.5)
Gandalf (Lv.7)
```

`TreeSet` e `TreeMap` usano anch'essi `compareTo()` per mantenere gli elementi ordinati automaticamente:

```java
// Funziona perché Personaggio implementa Comparable
TreeSet<Personaggio> squadraOrdinata = new TreeSet<>();
squadraOrdinata.add(new Personaggio("Arthas", 150, 30, 5));
squadraOrdinata.add(new Personaggio("Gandalf", 100, 20, 7));
squadraOrdinata.add(new Personaggio("Legolas", 120, 25, 3));

// Già ordinati per livello, senza sort esplicito
squadraOrdinata.forEach(p ->
    System.out.println(p.getNome() + " Lv." + p.getLivello()));
```

::: {.callout-note}
## La logica di compareTo()
Un trucco semplice per ricordare il segno:

```java
// Ordine crescente per un intero:
return this.livello - altro.livello;

// Ordine decrescente per un intero:
return altro.livello - this.livello;

// Per le stringhe, usa compareTo() già disponibile:
return this.nome.compareTo(altro.nome);  // ordine alfabetico
```
:::

::: {.callout-important}
## Comparable e equals() devono essere coerenti
Se due oggetti sono uguali secondo `equals()`, `compareTo()` dovrebbe restituire 0. E viceversa — se `compareTo()` restituisce 0, in genere ci si aspetta che `equals()` sia `true`. Mantieni i due metodi allineati usando gli stessi campi.
:::

## Array vs ArrayList — quando usare cosa

| | Array | ArrayList |
|---|---|---|
| **Dimensione** | Fissa | Dinamica |
| **Tipi** | Primitivi e oggetti | Solo oggetti (wrapper per primitivi) |
| **Performance** | Leggermente più veloce | Ottima per uso generale |
| **Metodi utili** | Solo `length` | `add`, `remove`, `contains`, `size`... |
| **Quando usarlo** | Dimensione nota e fissa | Dimensione variabile o sconosciuta |

---

## Esempio completo: gestione inventario

```java
import java.util.*;

public class GestioneGioco {
    public static void main(String[] args) {

        // Lista giocatori
        List<Personaggio> giocatori = new ArrayList<>();
        giocatori.add(new Personaggio("Arthas", 150, 30, 1));
        giocatori.add(new Personaggio("Gandalf", 100, 20, 3));
        giocatori.add(new Personaggio("Legolas", 120, 25, 2));

        // Mappa punteggi
        Map<String, Integer> punteggi = new HashMap<>();
        for (Personaggio p : giocatori) {
            punteggi.put(p.getNome(), 0);
        }

        // Simulazione: ogni personaggio guadagna punti per livello
        for (Personaggio p : giocatori) {
            int punti = p.getLivello() * 100;
            punteggi.put(p.getNome(), punti);
        }

        // Classifica ordinata per punteggio decrescente
        giocatori.sort((a, b) ->
            punteggi.get(b.getNome()) - punteggi.get(a.getNome()));

        System.out.println("=== CLASSIFICA ===");
        for (int i = 0; i < giocatori.size(); i++) {
            String nome = giocatori.get(i).getNome();
            System.out.println((i + 1) + ". " +
                nome + " → " + punteggi.get(nome) + " punti");
        }
    }
}
```

Output:
```
=== CLASSIFICA ===
1. Gandalf → 300 punti
2. Legolas → 200 punti
3. Arthas → 100 punti
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- Il **Collections Framework** offre strutture dati dinamiche e flessibili.
- I **generics** (`<T>`) specificano il tipo degli elementi e rendono il codice type-safe.
- **`ArrayList`** è una lista dinamica: elementi ordinati, duplicati ammessi, accesso per indice.
- **`HashMap`** associa chiavi a valori: ricerca rapida per chiave, nessun ordine garantito.
- Programma verso l'**interfaccia** (`List`, `Map`) e istanzia l'**implementazione** (`ArrayList`, `HashMap`).
- La classe **`Collections`** offre metodi statici utili: `sort`, `reverse`, `shuffle`, `max`, `min`.
- Per ordinare oggetti personalizzati usa un **`Comparator`** con un lambda.
:::

Nella prossima lezione approfondiamo le interfacce `List`, `Set` e `Queue` con le loro implementazioni e casi d'uso.