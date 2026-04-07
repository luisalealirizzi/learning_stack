# Caso d'uso SOLID

Vediamo come applicare i cinque principi SOLID su un esempio concreto. Partiamo da un codice che funziona ma è mal strutturato, e lo miglioriamo un passo alla volta.

---

## Il punto di partenza: codice che funziona ma non scala

Stiamo sviluppando un gioco. Abbiamo una classe `Gioco` che gestisce tutto: i personaggi, il salvataggio, le notifiche e il calcolo dei punteggi.

```java
public class Gioco {

    private String nomeGiocatore;
    private int livello;
    private int punteggio;
    private String tipoPersonaggio;

    public Gioco(String nomeGiocatore, String tipoPersonaggio) {
        this.nomeGiocatore = nomeGiocatore;
        this.tipoPersonaggio = tipoPersonaggio;
        this.livello = 1;
        this.punteggio = 0;
    }

    // calcola il punteggio
    public void aggiornaRisultato(int nemiciSconfitti) {
        if (tipoPersonaggio.equals("guerriero")) {
            punteggio += nemiciSconfitti * 10;
        } else if (tipoPersonaggio.equals("mago")) {
            punteggio += nemiciSconfitti * 15;
        } else if (tipoPersonaggio.equals("arciere")) {
            punteggio += nemiciSconfitti * 12;
        }
        livello = punteggio / 100 + 1;
    }

    // salva su file
    public void salva() {
        System.out.println("Salvo su file: " + nomeGiocatore +
            ", livello=" + livello + ", punteggio=" + punteggio);
        // logica di scrittura su file...
    }

    // invia notifica
    public void inviaNotifica() {
        System.out.println("Email inviata a " + nomeGiocatore +
            ": sei al livello " + livello);
    }

    // stampa lo stato
    public void stampa() {
        System.out.println("Giocatore: " + nomeGiocatore);
        System.out.println("Tipo: " + tipoPersonaggio);
        System.out.println("Livello: " + livello);
        System.out.println("Punteggio: " + punteggio);
    }
}
```

Questo codice funziona. Ma ha molti problemi:

- `Gioco` gestisce il calcolo del punteggio, il salvataggio, le notifiche e la stampa
- se aggiungi un nuovo tipo di personaggio devi modificare `aggiornaRisultato`
- se cambia il formato del salvataggio devi modificare `Gioco`
- se cambia il sistema di notifica devi modificare `Gioco`

Una sola classe ha troppi motivi per cambiare. Applichiamo SOLID.

---

## Passo 1: Single Responsibility

**Problema:** `Gioco` fa troppe cose. Ogni responsabilita va in una classe separata.

Separiamo le responsabilita:

```java
// Solo i dati e la logica del giocatore
public class Giocatore {

    private String nome;
    private String tipoPersonaggio;
    private int livello;
    private int punteggio;

    public Giocatore(String nome, String tipoPersonaggio) {
        this.nome = nome;
        this.tipoPersonaggio = tipoPersonaggio;
        this.livello = 1;
        this.punteggio = 0;
    }

    public String getNome()            { return nome; }
    public String getTipoPersonaggio() { return tipoPersonaggio; }
    public int getLivello()            { return livello; }
    public int getPunteggio()          { return punteggio; }

    public void setPunteggio(int punteggio) {
        this.punteggio = punteggio;
        this.livello = punteggio / 100 + 1;
    }
}

// Solo la logica di salvataggio
public class SalvataggioService {
    public void salva(Giocatore g) {
        System.out.println("Salvo: " + g.getNome() +
            ", livello=" + g.getLivello() +
            ", punteggio=" + g.getPunteggio());
    }
}

// Solo la logica di notifica
public class NotificaService {
    public void invia(Giocatore g) {
        System.out.println("Email a " + g.getNome() +
            ": sei al livello " + g.getLivello());
    }
}

// Solo la logica di stampa
public class StampaService {
    public void stampa(Giocatore g) {
        System.out.println("Giocatore: " + g.getNome());
        System.out.println("Tipo: "      + g.getTipoPersonaggio());
        System.out.println("Livello: "   + g.getLivello());
        System.out.println("Punteggio: " + g.getPunteggio());
    }
}
```

Ora ogni classe ha un solo motivo per cambiare.

---

## Passo 2: Open/Closed

**Problema:** il calcolo del punteggio usa una catena di `if-else`. Aggiungere un nuovo tipo di personaggio richiede di modificare il codice esistente.

Introduciamo un'interfaccia per il calcolo del punteggio:

```java
// Interfaccia che definisce il contratto
public interface CalcoloPunteggio {
    int calcola(int nemiciSconfitti);
}

// Una classe per ogni tipo di personaggio
public class PunteggioGuerriero implements CalcoloPunteggio {
    @Override
    public int calcola(int nemiciSconfitti) {
        return nemiciSconfitti * 10;
    }
}

public class PunteggioMago implements CalcoloPunteggio {
    @Override
    public int calcola(int nemiciSconfitti) {
        return nemiciSconfitti * 15;
    }
}

public class PunteggioArciere implements CalcoloPunteggio {
    @Override
    public int calcola(int nemiciSconfitti) {
        return nemiciSconfitti * 12;
    }
}
```

`Giocatore` ora riceve il calcolatore dall'esterno:

```java
public class Giocatore {

    private String nome;
    private int livello;
    private int punteggio;
    private CalcoloPunteggio calcolatore;   // dipende dall'interfaccia

    public Giocatore(String nome, CalcoloPunteggio calcolatore) {
        this.nome = nome;
        this.calcolatore = calcolatore;
        this.livello = 1;
        this.punteggio = 0;
    }

    public void aggiornaRisultato(int nemiciSconfitti) {
        punteggio += calcolatore.calcola(nemiciSconfitti);
        livello = punteggio / 100 + 1;
    }

    // getter...
}
```

Per aggiungere un Paladino basta creare `PunteggioPaladino` senza toccare nulla di esistente.

---

## Passo 3: Liskov Substitution

**Problema:** immagina di voler creare un `GiocatoreOspite` che non puo salire di livello. Se estendiamo `Giocatore` e sovrascriviamo `aggiornaRisultato` per non fare nulla, violare Liskov: chi usa `Giocatore` si aspetta che il punteggio si aggiorni.

La soluzione e ripensare la gerarchia. Introduciamo una classe base `Partecipante` con solo i comportamenti comuni:

```java
public abstract class Partecipante {
    protected String nome;

    public Partecipante(String nome) {
        this.nome = nome;
    }

    public String getNome() { return nome; }

    // metodo comune a tutti i partecipanti
    public abstract void stampaStato();
}

// Giocatore completo -- sale di livello e accumula punteggio
public class Giocatore extends Partecipante {

    private int livello;
    private int punteggio;
    private CalcoloPunteggio calcolatore;

    public Giocatore(String nome, CalcoloPunteggio calcolatore) {
        super(nome);
        this.calcolatore = calcolatore;
        this.livello = 1;
        this.punteggio = 0;
    }

    public void aggiornaRisultato(int nemiciSconfitti) {
        punteggio += calcolatore.calcola(nemiciSconfitti);
        livello = punteggio / 100 + 1;
    }

    public int getLivello()  { return livello; }
    public int getPunteggio(){ return punteggio; }

    @Override
    public void stampaStato() {
        System.out.println(nome + " | Lv." + livello + " | " + punteggio + " pt");
    }
}

// Ospite -- partecipa ma non accumula punteggio
public class GiocatoreOspite extends Partecipante {

    public GiocatoreOspite(String nome) {
        super(nome);
    }

    @Override
    public void stampaStato() {
        System.out.println(nome + " [ospite]");
    }
}
```

Ora `Giocatore` e `GiocatoreOspite` si comportano in modo coerente con quello che ci si aspetta da un `Partecipante`. Nessuna sorpresa.

---

## Passo 4: Interface Segregation

**Problema:** i service di salvataggio e notifica dipendono da `Giocatore`, ma potrebbero avere bisogno solo di alcune informazioni. Se `Giocatore` cresce, i service si trovano legati a metodi che non usano.

Introduciamo interfacce specifiche:

```java
// Solo quello che serve per salvare
public interface Salvabile {
    String getNome();
    int getLivello();
    int getPunteggio();
}

// Solo quello che serve per notificare
public interface Notificabile {
    String getNome();
    int getLivello();
}

// Giocatore implementa entrambe
public class Giocatore extends Partecipante implements Salvabile, Notificabile {
    // ...
}
```

I service ora dipendono da interfacce piccole e specifiche:

```java
public class SalvataggioService {
    public void salva(Salvabile s) {
        System.out.println("Salvo: " + s.getNome() +
            ", livello=" + s.getLivello() +
            ", punteggio=" + s.getPunteggio());
    }
}

public class NotificaService {
    public void invia(Notificabile n) {
        System.out.println("Email a " + n.getNome() +
            ": sei al livello " + n.getLivello());
    }
}
```

`SalvataggioService` non sa nulla di `getPunteggio` se non ne ha bisogno, e `NotificaService` non sa nulla del salvataggio.

---

## Passo 5: Dependency Inversion

**Problema:** il codice che orchestra tutto dipende da classi concrete. Se vogliamo cambiare il sistema di salvataggio o di notifica, dobbiamo modificare il codice principale.

Introduciamo interfacce per i service:

```java
public interface ISalvataggioService {
    void salva(Salvabile s);
}

public interface INotificaService {
    void invia(Notificabile n);
}

// Implementazioni concrete
public class SalvataggioSuFile implements ISalvataggioService {
    @Override
    public void salva(Salvabile s) {
        System.out.println("Salvo su file: " + s.getNome());
    }
}

public class NotificaEmail implements INotificaService {
    @Override
    public void invia(Notificabile n) {
        System.out.println("Email a: " + n.getNome());
    }
}
```

Il coordinatore riceve le dipendenze dall'esterno:

```java
public class GestorePartita {

    private ISalvataggioService salvataggio;
    private INotificaService notifica;

    // le dipendenze vengono iniettate dal costruttore
    public GestorePartita(ISalvataggioService salvataggio,
                          INotificaService notifica) {
        this.salvataggio = salvataggio;
        this.notifica = notifica;
    }

    public void fineTurno(Giocatore g, int nemiciSconfitti) {
        g.aggiornaRisultato(nemiciSconfitti);
        salvataggio.salva(g);
        notifica.invia(g);
    }
}
```

---

## Il risultato finale

Ecco come appare il `main` dopo aver applicato tutti i principi:

```java
public class Main {
    public static void main(String[] args) {

        // scelgo il tipo di personaggio (Open/Closed)
        CalcoloPunteggio calcolatore = new PunteggioMago();

        // creo il giocatore (Single Responsibility, Liskov)
        Giocatore g = new Giocatore("Gandalf", calcolatore);

        // scelgo le implementazioni (Dependency Inversion)
        ISalvataggioService salvataggio = new SalvataggioSuFile();
        INotificaService notifica = new NotificaEmail();

        // il gestore non sa nulla delle implementazioni concrete
        GestorePartita partita = new GestorePartita(salvataggio, notifica);

        // gioco
        partita.fineTurno(g, 5);
        partita.fineTurno(g, 3);

        g.stampaStato();
    }
}
```

Output:
```
Salvo su file: Gandalf
Email a: Gandalf
Salvo su file: Gandalf
Email a: Gandalf
Gandalf | Lv.2 | 120 pt
```

---

## Cosa e cambiato

| | Codice iniziale | Codice SOLID |
|---|---|---|
| **Classi** | 1 che fa tutto | tante, ognuna con un compito |
| **Nuovo tipo personaggio** | modifica `aggiornaRisultato` | nuova classe, zero modifiche |
| **Nuovo sistema di salvataggio** | modifica `Gioco` | nuova classe, zero modifiche |
| **Testabilita** | difficile | ogni classe si testa da sola |
| **Leggibilita** | bassa | alta |

Il codice e piu lungo, ma ogni pezzo e semplice, indipendente e sostituibile.

::: {.callout-note}
## Da ricordare
SOLID non significa scrivere piu codice per il gusto di farlo. Significa scrivere codice che possa crescere senza rompersi. Su progetti piccoli alcuni principi sembrano eccessivi -- ma su progetti grandi fanno la differenza tra codice che si puo mantenere e codice che diventa un problema.
:::