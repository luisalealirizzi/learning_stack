# Polimorfismo, upcasting e downcasting

---

## Riprendiamo dal polimorfismo

Nella lezione sull'ereditarietà abbiamo costruito la gerarchia `Personaggio` → `Guerriero`, `Mago`, `Arciere`. Ogni sottoclasse ridefinisce `attacca()` con il proprio comportamento.

Il **polimorfismo** è la capacità di oggetti di classi diverse di rispondere alla stessa chiamata in modo diverso, ognuno secondo la propria natura.

::: {.callout-note}
## Definizione
Il polimorfismo permette di trattare oggetti di classi diverse attraverso un'unica interfaccia comune, lasciando che ogni oggetto risponda a modo suo.
:::

---

## Override: ridefinire un metodo

L'override è il meccanismo con cui una sottoclasse **ridefinisce** un metodo ereditato dalla madre. L'annotazione `@Override` dice al compilatore che stiamo intenzionalmente ridefinendo un metodo — e ci avvisa se sbagliamo il nome.

```java
public class Guerriero extends Personaggio {
    private int forza;

    @Override
    public void attacca(Personaggio bersaglio) {
        int danno = forza + 15;
        System.out.println(getNome() + " colpisce con la spada!");
        bersaglio.subirDanno(danno);
    }
}

public class Mago extends Personaggio {
    private int intelligenza;
    private int mana;

    @Override
    public void attacca(Personaggio bersaglio) {
        if (mana >= 20) {
            int danno = intelligenza * 2;
            System.out.println(getNome() + " lancia Fireball!");
            bersaglio.subirDanno(danno);
            mana -= 20;
        } else {
            System.out.println(getNome() + " non ha mana! Attacco fisico.");
            super.attacca(bersaglio);
        }
    }
}

public class Arciere extends Personaggio {
    private int destrezza;
    private int frecce;

    @Override
    public void attacca(Personaggio bersaglio) {
        if (frecce > 0) {
            int danno = destrezza + 10;
            System.out.println(getNome() + " scocca una freccia!");
            bersaglio.subirDanno(danno);
            frecce--;
        } else {
            System.out.println(getNome() + " ha esaurito le frecce!");
            super.attacca(bersaglio);
        }
    }
}
```

---

## Riferimenti polimorfici e il vantaggio reale

In Java una variabile di tipo `Personaggio` può contenere qualsiasi oggetto di una sua sottoclasse — perché un `Guerriero` **è un** `Personaggio`:

```java
Personaggio p1 = new Guerriero(...);
Personaggio p2 = new Mago(...);
Personaggio p3 = new Arciere(...);
```

Quando chiami `attacca()` su queste variabili, Java esegue il metodo della **classe reale dell'oggetto**, non quello della variabile. Questo si chiama **binding dinamico**: la scelta avviene a runtime.

```java
Personaggio p = new Mago(...);
p.attacca(nemico);  // chiama Mago.attacca(), non Personaggio.attacca()!
```

Il vero vantaggio: puoi mettere tutti i personaggi in un unico array e gestirli con un solo ciclo:

```java
Personaggio[] squadra = {
    new Guerriero("Arthas", 150, 30, 1, 15),
    new Mago("Gandalf", 100, 20, 3, 120),
    new Arciere("Legolas", 120, 25, 2, 20)
};

Personaggio boss = new Guerriero("Sauron", 500, 50, 10, 0);

for (Personaggio p : squadra) {
    p.attacca(boss);  // ognuno attacca a modo suo!
}
```

::: {.callout-tip}
## Perché è potente
Se domani aggiungi `Paladino extends Personaggio`, non tocchi nulla nel ciclo di combattimento. Aggiungi il `Paladino` all'array e il codice lo gestisce automaticamente. Questo è il vero vantaggio del polimorfismo.
:::

---

## Upcasting

L'**upcasting** è la conversione di un oggetto da un tipo figlio a un tipo padre. È sempre **sicuro** e avviene **automaticamente** — Java lo fa senza bisogno di parentesi esplicite.

```java
Guerriero guerriero = new Guerriero("Arthas", 150, 30, 1, 15);

// Upcasting implicito — automatico
Personaggio p = guerriero;
```

Dopo l'upcasting, attraverso la variabile `p` puoi accedere solo ai metodi di `Personaggio` — anche se l'oggetto reale è un `Guerriero`. I metodi specifici del `Guerriero` non sono visibili:

```java
p.attacca(nemico);    // OK — metodo di Personaggio (eseguito come Guerriero)
p.subirDanno(10);     // OK — metodo di Personaggio
// p.colpoPoderoso(); // ERRORE — metodo specifico di Guerriero, non visibile!
```

::: {.callout-note}
## Cosa succede con l'upcasting
L'oggetto in memoria è e rimane un `Guerriero`. L'upcasting cambia solo il **tipo della variabile** — la "lente" attraverso cui lo guardi. Il binding dinamico garantisce che i metodi override vengano comunque eseguiti nella versione corretta.
:::

Questo è esattamente quello che abbiamo fatto con l'array di `Personaggio[]` — tutti gli elementi erano tecnicamente upcastati a `Personaggio`.

---

## Downcasting

Il **downcasting** è l'operazione inversa: da tipo padre a tipo figlio. Non è automatico — richiede le **parentesi esplicite** — e può fallire a runtime se l'oggetto reale non è del tipo che ci aspettiamo.

```java
Personaggio p = new Guerriero("Arthas", 150, 30, 1, 15);

// Downcasting esplicito
Guerriero g = (Guerriero) p;
g.colpoPoderoso(nemico);  // ora posso accedere ai metodi specifici!
```

Ma cosa succede se l'oggetto reale non è un `Guerriero`?

```java
Personaggio p = new Mago("Gandalf", 100, 20, 3, 120);
Guerriero g = (Guerriero) p;  // ERRORE a runtime: ClassCastException!
```

Il compilatore non si accorge dell'errore — `p` è di tipo `Personaggio` e un `Guerriero` è un `Personaggio`, quindi la sintassi è valida. Ma a runtime Java controlla il tipo reale dell'oggetto e lancia un'eccezione.

::: {.callout-warning}
## Il downcasting è rischioso
Prima di fare un downcasting, verifica sempre il tipo reale dell'oggetto con **`instanceof`**.
:::

---

## L'operatore `instanceof`

`instanceof` controlla se un oggetto è un'istanza di una certa classe (o di una sua sottoclasse):

```java
Personaggio p = new Mago("Gandalf", 100, 20, 3, 120);

if (p instanceof Mago) {
    Mago m = (Mago) p;
    System.out.println("Mana: " + m.getMana());
} else if (p instanceof Guerriero) {
    Guerriero g = (Guerriero) p;
    g.colpoPoderoso(nemico);
}
```

Da Java 16 in poi esiste anche il **pattern matching** per `instanceof`, che rende il codice più compatto:

```java
if (p instanceof Mago m) {
    // m è già di tipo Mago, nessun cast esplicito!
    System.out.println("Mana: " + m.getMana());
}
```

---

## `==` vs `equals()`

Già visto per le stringhe, ma vale per tutti gli oggetti. È un errore classico che vale la pena approfondire.

`==` confronta i **riferimenti** — controlla se due variabili puntano allo stesso oggetto in memoria.

`equals()` confronta il **contenuto** — controlla se due oggetti sono logicamente equivalenti.

```java
String a = new String("ciao");
String b = new String("ciao");

System.out.println(a == b);        // false — sono due oggetti diversi in memoria
System.out.println(a.equals(b));   // true  — il contenuto è uguale
```

Con i tipi primitivi `==` funziona correttamente perché confronta direttamente i valori:

```java
int x = 5;
int y = 5;
System.out.println(x == y);  // true — confronto di valori, non riferimenti
```

::: {.callout-important}
## Regola d'oro
- Per i **tipi primitivi** (`int`, `double`, `boolean`...) usa sempre `==`
- Per gli **oggetti** (`String`, `Personaggio`, qualsiasi classe) usa sempre `equals()`
- L'unica eccezione accettabile con `==` sugli oggetti è il confronto con `null`: `if (oggetto == null)`
:::

---

## Implementare `equals()` in una classe

Quando scrivi le tue classi, puoi decidere cosa significa "uguale" ridefinendo `equals()`:

```java
public class Personaggio {
    private String nome;
    private int livello;
    // ...

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;           // stesso riferimento
        if (obj == null) return false;          // null non è uguale a niente
        if (!(obj instanceof Personaggio)) return false; // tipi diversi

        Personaggio altro = (Personaggio) obj;
        return this.nome.equals(altro.nome) && this.livello == altro.livello;
    }
}
```

```java
Personaggio p1 = new Personaggio("Arthas", 150, 30, 1);
Personaggio p2 = new Personaggio("Arthas", 150, 30, 1);

System.out.println(p1 == p2);        // false — oggetti diversi in memoria
System.out.println(p1.equals(p2));   // true  — stesso nome e livello
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- Il **polimorfismo** permette a oggetti di classi diverse di rispondere alla stessa chiamata in modo diverso.
- L'**override** (`@Override`) ridefinisce un metodo ereditato nella sottoclasse.
- Il **binding dinamico** sceglie a runtime quale versione del metodo eseguire.
- L'**upcasting** converte da tipo figlio a tipo padre — automatico e sempre sicuro.
- Il **downcasting** converte da tipo padre a tipo figlio — esplicito e potenzialmente rischioso.
- Usa sempre **`instanceof`** prima di fare un downcasting.
- `==` confronta i **riferimenti**; `equals()` confronta il **contenuto** — per gli oggetti usa sempre `equals()`.
:::

Nella prossima lezione vedremo le **classi astratte e le interfacce** — strumenti potenti per definire contratti e comportamenti condivisi tra classi non necessariamente imparentate.