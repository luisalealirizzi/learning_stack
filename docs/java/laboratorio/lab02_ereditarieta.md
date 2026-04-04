# Lab 02 — Ereditarietà e override

*Ereditarietà, override di `toString()` e `equals()`, upcasting e downcasting*

---

## Obiettivo

Creare due classi (`Giocatore` e `GiocatorePremium`) con override di `toString()` e `equals()`, più alcuni metodi per la gestione dei punteggi. Infine scrivere un programma di test per verificare il comportamento.

---

## Classe Giocatore

**Attributi:**
- `int codiceGiocatore`
- `String nickname`
- `int punteggio`

**Metodi da implementare:**
- Costruttori (tutti quelli che ritieni opportuni)
- Getter e setter
- `incrementaPunteggio(int punti)`
- `toString()` — descrive l'oggetto in modo leggibile
- `equals(Object o)` — due giocatori sono uguali solo se hanno lo stesso `codiceGiocatore`

```java
public class Giocatore {

    private int codiceGiocatore;
    private String nickname;
    private int punteggio;

    // costruttori, getter, setter...

    public void incrementaPunteggio(int punti) {
        // TODO
    }

    @Override
    public String toString() {
        // TODO — esempio: "Giocatore[codice=1, nickname=Arthas, punteggio=1500]"
    }

    @Override
    public boolean equals(Object o) {
        // TODO — uguali se stesso codiceGiocatore
    }
}
```

---

## Classe GiocatorePremium

`GiocatorePremium` estende `Giocatore` e aggiunge un livello VIP.

**Attributo aggiuntivo:**
- `int livelloVIP`

**Metodi:**
- `toString()` ridefinito — richiama `super.toString()` e aggiunge il livello VIP

```java
public class GiocatorePremium extends Giocatore {

    private int livelloVIP;

    // costruttori, getter, setter...

    @Override
    public String toString() {
        // TODO — esempio: super.toString() + " [VIP livello 2]"
    }
}
```

---

## Metodo extra — Sfida casuale *(facoltativo)*

Aggiungi questo metodo alla classe `Giocatore`:

```java
public Giocatore sfida(Giocatore altro) {
    int mioEffettivo    = this.getPunteggio()  + (int)(Math.random() * 101);
    int altroEffettivo  = altro.getPunteggio() + (int)(Math.random() * 101);

    if (mioEffettivo > altroEffettivo)  return this;
    if (altroEffettivo > mioEffettivo)  return altro;
    return null;  // pareggio
}
```

- Il punteggio "effettivo" = punteggio + numero casuale tra 0 e 100
- Vince chi ha il punteggio effettivo più alto
- In caso di pareggio → restituisce `null`

---

## Programma di test

### Struttura base

```java
public class Main {
    public static void main(String[] args) {

        // TODO: creare due Giocatore con stesso codice ma punteggi diversi
        //       → verifica equals()

        // TODO: creare un GiocatorePremium e stamparlo con toString()

        // TODO: dimostrare upcasting e downcasting

        // TODO: se implementato, provare il metodo sfida()
    }
}
```

### Upcasting e downcasting

```java
public class Main {
    public static void main(String[] args) {

        // Creo un GiocatorePremium
        GiocatorePremium gp = new GiocatorePremium(100, "ProPlayer", 3000, 2);

        // === Upcasting — implicito, sempre sicuro ===
        Giocatore g = gp;
        System.out.println("Stampo g (upcasting): " + g.toString());

        // Non posso accedere a getLivelloVIP() tramite g:
        // System.out.println(g.getLivelloVIP()); // ERRORE di compilazione!

        // === Downcasting corretto ===
        if (g instanceof GiocatorePremium) {
            GiocatorePremium gp2 = (GiocatorePremium) g;  // sicuro
            System.out.println("Livello VIP dopo il downcasting: " + gp2.getLivelloVIP());
        }

        // === Downcasting sbagliato — prova a decommentare e osserva l'errore ===
        // Giocatore g2 = new Giocatore(200, "NoobMaster", 100);
        // GiocatorePremium gp3 = (GiocatorePremium) g2;  // ClassCastException a runtime!
    }
}
```

::: {.callout-note}
## Cosa osservare
- Dopo l'upcasting `g` vede solo i metodi di `Giocatore` — anche se l'oggetto reale è un `GiocatorePremium`
- `toString()` viene comunque eseguito nella versione di `GiocatorePremium` — binding dinamico
- `instanceof` prima del downcasting evita la `ClassCastException`
- Il downcasting sbagliato compila ma crasha a runtime — decommentalo e osserva l'errore
:::

---

## Output atteso (esempio)

```
=== equals() ===
g1.equals(g2): true   ← stesso codice, punteggi diversi

=== toString() ===
Giocatore[codice=1, nickname=Arthas, punteggio=1500]
GiocatorePremium[codice=100, nickname=ProPlayer, punteggio=3000] [VIP livello 2]

=== Upcasting e downcasting ===
Stampo g (upcasting): GiocatorePremium[codice=100, nickname=ProPlayer, punteggio=3000] [VIP livello 2]
Livello VIP dopo il downcasting: 2

=== Sfida ===
Vincitore: Arthas
```