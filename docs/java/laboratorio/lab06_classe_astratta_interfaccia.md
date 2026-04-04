# Lab 06 — Classi astratte e interfacce

*Esercitazione guidata — classe astratta, interfaccia, polimorfismo, late binding*

---

## Obiettivo

Costruire un mini sistema che gestisca 2 dispositivi dove:
- entrambi **"sono un tipo di dispositivo"** → classe astratta
- solo alcuni **"possono consumare batteria"** → interfaccia
- ogni dispositivo **fa qualcosa di diverso** → override reale

---

## Fase 1 — Rifletti prima di scrivere

Prima di aprire l'editor, rispondi a queste domande:

**Ha senso scrivere `new Dispositivo()` da solo?**
Un dispositivo generico non esiste nella realtà — esistono sensori, attuatori, telecamere. `Dispositivo` è un concetto astratto, non un oggetto concreto.
→ Questa è una **classe astratta**.

**Ha senso che tutti abbiano la batteria?**
Un sensore wireless ce l'ha. Un attuatore collegato alla rete no. La batteria non è una caratteristica di tutti i dispositivi — è una capacità che solo alcuni hanno.
→ Questa è una **interfaccia**.

::: {.callout-note}
## Classe astratta vs interfaccia — la regola pratica
Usa la **classe astratta** quando le classi condividono codice e appartengono alla stessa famiglia (`Dispositivo` → `Sensore`, `Attuatore`).

Usa l'**interfaccia** quando vuoi definire una capacità trasversale che solo alcune classi hanno (`Ricaricabile`).
:::

---

## Fase 2 — La classe astratta

```java
import java.time.LocalDateTime;

abstract class Dispositivo {

    // Attributi comuni a tutti i dispositivi
    private String id;
    private String nome;
    private LocalDateTime ultimoPing;

    // Costruttore
    public Dispositivo(String id, String nome) {
        this.id = id;
        this.nome = nome;
        this.ultimoPing = null;
    }

    // Getter
    public String getId()   { return id; }
    public String getNome() { return nome; }
    public LocalDateTime getUltimoPing() { return ultimoPing; }

    // Metodo CONCRETO — uguale per tutti, non va ridefinito
    public void ping() {
        this.ultimoPing = LocalDateTime.now();
        System.out.println("[" + nome + "] ping alle " + ultimoPing);
    }

    // Metodo ASTRATTO — ogni sottoclasse DEVE implementarlo
    public abstract void azionePrincipale();
}
```

::: {.callout-important}
## Cosa significa `abstract`?
- La classe `Dispositivo` **non può essere istanziata** — `new Dispositivo(...)` dà errore di compilazione
- Il metodo `azionePrincipale()` **non ha corpo** — ogni sottoclasse concreta deve implementarlo
- `ping()` invece è concreto — ereditato da tutti senza doverlo riscrivere
:::

---

## Fase 3 — Le sottoclassi

```java
class Sensore extends Dispositivo {

    private String tipoMisura;
    private double ultimoValore;

    public Sensore(String id, String nome, String tipoMisura) {
        super(id, nome);  // chiama il costruttore di Dispositivo
        this.tipoMisura = tipoMisura;
        this.ultimoValore = 0.0;
    }

    @Override
    public void azionePrincipale() {
        // Il sensore misura
        ultimoValore = 20 + Math.random() * 10;  // simulazione lettura
        ping();
        System.out.println("[" + getNome() + "] " + tipoMisura +
            " = " + String.format("%.1f", ultimoValore));
    }
}
```

```java
class Attuatore extends Dispositivo {

    private String tipoAzione;
    private boolean statoOn;

    public Attuatore(String id, String nome, String tipoAzione) {
        super(id, nome);
        this.tipoAzione = tipoAzione;
        this.statoOn = false;
    }

    @Override
    public void azionePrincipale() {
        // L'attuatore esegue il suo comando
        statoOn = !statoOn;  // toggle on/off
        ping();
        System.out.println("[" + getNome() + "] " + tipoAzione +
            " → " + (statoOn ? "ON" : "OFF"));
    }
}
```

---

## Fase 4 — L'interfaccia Ricaricabile

```java
interface Ricaricabile {
    void consuma(int percento);
    int getBatteria();
}
```

Ora decidete in gruppo: **chi implementa `Ricaricabile`?**

In questo esempio il `Sensore` è a batteria, l'`Attuatore` è collegato alla rete:

```java
class Sensore extends Dispositivo implements Ricaricabile {

    private String tipoMisura;
    private double ultimoValore;
    private int batteria;  // attributo aggiunto per la batteria

    public Sensore(String id, String nome, String tipoMisura, int batteriaIniziale) {
        super(id, nome);
        this.tipoMisura = tipoMisura;
        this.batteria = batteriaIniziale;
    }

    @Override
    public void azionePrincipale() {
        ultimoValore = 20 + Math.random() * 10;
        ping();
        consuma(1);  // ogni misura consuma 1%
        System.out.println("[" + getNome() + "] " + tipoMisura +
            " = " + String.format("%.1f", ultimoValore) +
            " | Batteria: " + batteria + "%");
    }

    // Implementazione dell'interfaccia Ricaricabile
    @Override
    public void consuma(int percento) {
        batteria = Math.max(0, batteria - percento);
        if (batteria == 0) {
            System.out.println("AVVISO: " + getNome() + " — batteria esaurita!");
        }
    }

    @Override
    public int getBatteria() { return batteria; }
}
```

---

## Fase 5 — Il main

```java
public class Main {
    public static void main(String[] args) {

        // Polimorfismo — riferimento Dispositivo, oggetto concreto
        Dispositivo d1 = new Sensore("S01", "Termostato", "TEMPERATURA", 80);
        Dispositivo d2 = new Attuatore("A01", "Sirena", "SIRENA");

        // Late binding — viene chiamato l'override della classe reale
        System.out.println("=== Azione d1 ===");
        d1.azionePrincipale();

        System.out.println("\n=== Azione d2 ===");
        d2.azionePrincipale();

        // instanceof — controlla se il dispositivo è Ricaricabile
        System.out.println("\n=== Controllo batteria ===");
        if (d1 instanceof Ricaricabile) {
            Ricaricabile r = (Ricaricabile) d1;  // downcasting
            System.out.println(d1.getNome() + " — batteria: " + r.getBatteria() + "%");
        } else {
            System.out.println(d1.getNome() + " — non a batteria");
        }

        if (d2 instanceof Ricaricabile) {
            Ricaricabile r = (Ricaricabile) d2;
            System.out.println(d2.getNome() + " — batteria: " + r.getBatteria() + "%");
        } else {
            System.out.println(d2.getNome() + " — non a batteria");
        }
    }
}
```

Output atteso:
```
=== Azione d1 ===
[Termostato] ping alle 2025-11-13T10:05:32
[Termostato] TEMPERATURA = 24.3 | Batteria: 79%

=== Azione d2 ===
[Sirena] ping alle 2025-11-13T10:05:32
[Sirena] SIRENA → ON

=== Controllo batteria ===
Termostato — batteria: 79%
Sirena — non a batteria
```

::: {.callout-note}
## Late binding in azione
`d1` è dichiarato come `Dispositivo`, ma a runtime Java sa che l'oggetto reale è un `Sensore`. Quando chiama `d1.azionePrincipale()`, esegue la versione di `Sensore` — non quella astratta di `Dispositivo`. Questo è il **late binding** (o binding dinamico).
:::

---

## Fase 6 — Estensione *(scegli una)*

### Opzione A — Terzo tipo di dispositivo

Aggiungi una classe `Telecamera` che estende `Dispositivo`:
- attributo extra: `int fps` (fotogrammi al secondo)
- `azionePrincipale()` — registra un frame e stampa un messaggio
- decide il gruppo se implementa `Ricaricabile`

### Opzione B — Logging con `toString()`

Ridefinisci `toString()` in tutte le classi:

```java
// In Dispositivo
@Override
public String toString() {
    return "[" + getId() + "] " + getNome() +
        " | ultimo ping: " + (getUltimoPing() != null ? getUltimoPing() : "mai");
}

// In Sensore — aggiunge info batteria
@Override
public String toString() {
    return super.toString() + " | batteria: " + batteria + "%";
}
```

Poi nel main puoi scrivere direttamente `System.out.println(d1)`.

### Opzione C — Enum

Sostituisci le stringhe con enum per i tipi di misura e azione:

```java
enum TipoMisura  { TEMPERATURA, UMIDITA, LUX, GAS }
enum TipoAzione  { LUCE, SIRENA, RELE }
enum StatoSistema { ATTIVO, STANDBY, EMERGENZA }
```

### Opzione D — Priorità e emergenza

```java
// Se la batteria scende sotto il 10%, segnala emergenza
if (d1 instanceof Ricaricabile) {
    Ricaricabile r = (Ricaricabile) d1;
    if (r.getBatteria() < 10) {
        System.out.println("EMERGENZA: " + d1.getNome() +
            " — batteria critica (" + r.getBatteria() + "%)!");
    }
}
```