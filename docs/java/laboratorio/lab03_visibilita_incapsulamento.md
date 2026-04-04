# Lab 03 — Visibilità e incapsulamento

*Modificatori `private`, `protected`, `public`, `final` e uso di `Scanner`*

---

## Introduzione

In Java ogni attributo o metodo ha una **visibilità** che determina chi può accedervi:

| Modificatore | Chi può accedere |
|---|---|
| `public` | Tutti — qualsiasi classe |
| `protected` | La classe stessa + le sottoclassi (le "figlie") |
| `private` | Solo la classe stessa |

Esiste anche il modificatore **`final`**, che significa "non modificabile":
- un **attributo** `final` non può essere cambiato dopo l'inizializzazione
- un **metodo** `final` non può essere riscritto dalle sottoclassi

```java
final double FATTORE_CRESCITA = 1.2;  // resterà sempre 1.2
```

---

## Parte 1 — La classe Pianta

Crea una classe `Pianta` che rappresenti una pianta generica.

**Attributi privati:**
- `String nome`
- `double altezza` (in cm)
- `double livelloAcqua` (da 0 a 100%)

**Attributo final:**
- `double FATTORE_CRESCITA` — quanto velocemente la pianta cresce (es. `1.2`)

**Costruttori:**
- vuoto
- con parametri — inizializza `nome` e `altezza`; il livello d'acqua parte da 50%
- di copia

**Metodi pubblici:**
- `annaffia(double quantita)` — aumenta il livello d'acqua, ma non oltre 100
- `cresce()` — aumenta l'altezza in base all'acqua disponibile e al `FATTORE_CRESCITA`
- `toString()` — restituisce nome, altezza e livello d'acqua
- getter per tutti gli attributi

**Metodi protetti** *(accessibili alle sottoclassi)*:
- `setLivelloAcqua(double nuovoLivello)` — regola il livello d'acqua
- `setAltezza(double nuovaAltezza)` — modifica l'altezza

```java
public class Pianta {

    private String nome;
    private double altezza;
    private double livelloAcqua;
    final double FATTORE_CRESCITA = 1.2;  // scegli tu un valore realistico

    // Costruttore vuoto
    public Pianta() {
        // TODO
    }

    // Costruttore con parametri
    public Pianta(String nome, double altezza) {
        // TODO — livelloAcqua parte da 50%
    }

    // Costruttore di copia
    public Pianta(Pianta altra) {
        // TODO
    }

    // Getter
    public String getNome()        { return nome; }
    public double getAltezza()     { return altezza; }
    public double getLivelloAcqua(){ return livelloAcqua; }

    // Metodi protetti — accessibili alle sottoclassi
    protected void setLivelloAcqua(double nuovoLivello) {
        // TODO — controlla che sia tra 0 e 100
    }

    protected void setAltezza(double nuovaAltezza) {
        // TODO — controlla che sia > 0
    }

    // Metodi pubblici
    public void annaffia(double quantita) {
        // TODO — aumenta livelloAcqua, ma non oltre 100
    }

    public void cresce() {
        // TODO — aumenta altezza in base a livelloAcqua e FATTORE_CRESCITA
    }

    @Override
    public String toString() {
        // TODO — esempio: "Pianta[nome=Rosa, altezza=15.0cm, acqua=50.0%]"
    }
}
```

---

## Parte 2 — La classe PiantaCarnivora

Crea una classe `PiantaCarnivora` che estende `Pianta`.

**Attributo privato aggiuntivo:**
- `double livelloFame` (da 0 a 100%) — inizia a 50%

**Metodi:**
- `catturaInsetto()` — riduce il livello di fame, aumenta leggermente l'altezza (usa `setAltezza()`), consuma un po' d'acqua (usa `setLivelloAcqua()`)
- `toString()` ridefinito — richiama `super.toString()` e aggiunge il livello di fame
- `final void avvisa()` — stampa un messaggio di avvertimento; essendo `final`, nessuna sottoclasse potrà ridefinirlo

```java
public class PiantaCarnivora extends Pianta {

    private double livelloFame;

    public PiantaCarnivora(String nome, double altezza) {
        super(nome, altezza);
        this.livelloFame = 50.0;
    }

    public void catturaInsetto() {
        // TODO:
        // - riduci livelloFame
        // - aumenta leggermente l'altezza con setAltezza()
        // - consuma un po' d'acqua con setLivelloAcqua()
        System.out.println(getNome() + " ha catturato un insetto!");
    }

    @Override
    public String toString() {
        // TODO — super.toString() + " [fame=" + livelloFame + "%]"
    }

    public final void avvisa() {
        System.out.println("Attenzione: non toccare la pianta carnivora!");
    }
}
```

::: {.callout-note}
## Perché `protected` e non `private`?
I setter `setAltezza()` e `setLivelloAcqua()` sono `protected` perché `PiantaCarnivora` ne ha bisogno nel metodo `catturaInsetto()`. Se fossero `private`, la sottoclasse non potrebbe accedervi — e saremmo costretti a renderli `public`, esponendo i dati a tutti. `protected` è la via di mezzo: aperto alle figlie, chiuso al resto del mondo.
:::

---

## Programma di test

```java
public class Main {
    public static void main(String[] args) {

        // Crea una pianta normale e una carnivora
        Pianta p1 = new Pianta("Rosa", 10.0);
        PiantaCarnivora p2 = new PiantaCarnivora("Venere acchiappamosche", 5.0);

        // Stato iniziale
        System.out.println("=== STATO INIZIALE ===");
        System.out.println(p1);
        System.out.println(p2);

        // Annaffia e fai crescere la pianta normale
        p1.annaffia(30);
        p1.cresce();

        // Fai catturare un insetto alla carnivora e falla crescere
        p2.catturaInsetto();
        p2.cresce();

        // Stato finale
        System.out.println("\n=== STATO FINALE ===");
        System.out.println(p1);
        System.out.println(p2);

        // Messaggio di avvertimento
        p2.avvisa();

        // Prova a ridefinire avvisa() in una sottoclasse — il compilatore lo impedisce!
        // (final non può essere sovrascritto)
    }
}
```

---

## Variante con Scanner — input da utente

Modifica il `main` per creare le piante leggendo i dati dall'utente invece di scriverli nel codice.

**1. Importa Scanner** (in cima al file):
```java
import java.util.Scanner;
```

**2. Crea lo Scanner nel main:**
```java
Scanner input = new Scanner(System.in);
```

**3. Leggi i dati e crea la pianta:**
```java
System.out.print("Nome della pianta: ");
String nome = input.nextLine();

System.out.print("Altezza iniziale (cm): ");
double altezza = input.nextDouble();

Pianta p = new Pianta(nome, altezza);
System.out.println("Pianta creata: " + p);
```

**4. Chiudi lo Scanner alla fine:**
```java
input.close();
```

::: {.callout-warning}
## Attenzione a `nextDouble()` seguito da `nextLine()`
`nextDouble()` legge il numero ma lascia nel buffer il carattere `\n` (invio). Se subito dopo chiami `nextLine()`, legge quella riga vuota. La soluzione: aggiungi `input.nextLine()` extra dopo ogni `nextDouble()` o `nextInt()`.

```java
double altezza = input.nextDouble();
input.nextLine();  // consuma il \n rimasto
String note = input.nextLine();  // ora legge correttamente
```
:::

---

## Output atteso (esempio)

```
=== STATO INIZIALE ===
Pianta[nome=Rosa, altezza=10.0cm, acqua=50.0%]
Pianta[nome=Venere acchiappamosche, altezza=5.0cm, acqua=50.0%] [fame=50.0%]

Venere acchiappamosche ha catturato un insetto!

=== STATO FINALE ===
Pianta[nome=Rosa, altezza=12.5cm, acqua=80.0%]
Pianta[nome=Venere acchiappamosche, altezza=5.8cm, acqua=44.0%] [fame=35.0%]

Attenzione: non toccare la pianta carnivora!
```