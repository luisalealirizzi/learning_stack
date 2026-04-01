# Java Collections

---

## Il problema degli array

Gli array che conosci dal C hanno un limite fondamentale: la **dimensione è fissa**. Devi decidere quanti elementi ci sono prima di creare l'array, e non puoi cambiare idea.

```java
int[] punteggi = new int[10];  // esattamente 10 elementi, né più né meno
```

Nella realtà i dati sono dinamici: una lista di giocatori connessi cresce e diminuisce, un inventario si riempie e si svuota, i risultati di una ricerca variano. Serve qualcosa di più flessibile.

Java risponde con il **Collections Framework** — un insieme di classi e interfacce per gestire collezioni di oggetti in modo dinamico, sicuro ed efficiente.

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

Prima di usare le collections, capiamo i **generics** — la sintassi con le parentesi angolari `<>`.

I generics permettono di specificare il **tipo degli elementi** che una collezione può contenere. Il compilatore verifica che si inseriscano solo elementi del tipo corretto.

```java
List<String> nomi = new ArrayList<>();      // solo String
List<Integer> punteggi = new ArrayList<>(); // solo Integer
List<Personaggio> squadra = new ArrayList<>(); // solo Personaggio
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

**Iterator** — utile quando devi rimuovere elementi durante l'iterazione:
```java
import java.util.Iterator;

Iterator<String> it = squadra.iterator();
while (it.hasNext()) {
    String nome = it.next();
    if (nome.equals("Gandalf")) {
        it.remove();  // rimozione sicura durante l'iterazione
    }
}
```

::: {.callout-warning}
## Non rimuovere con for-each
Non rimuovere elementi da una lista mentre la stai scorrendo con un for-each — lancia `ConcurrentModificationException`. Usa l'`Iterator` oppure accumula gli elementi da rimuovere e cancellali dopo.
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

Per ordinare oggetti personalizzati puoi usare un **lambda** con `Comparator`:

```java
List<Personaggio> squadra = new ArrayList<>();
// ... aggiunta personaggi ...

// Ordina per livello crescente
Collections.sort(squadra, (a, b) -> a.getLivello() - b.getLivello());

// Ordina per nome alfabetico
Collections.sort(squadra, (a, b) -> a.getNome().compareTo(b.getNome()));

// Ordina per vita decrescente
squadra.sort((a, b) -> b.getVita() - a.getVita());
```

---

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