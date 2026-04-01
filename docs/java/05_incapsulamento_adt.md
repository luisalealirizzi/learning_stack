# Incapsulamento e ADT

---

## Riprendiamo da dove eravamo

Nella lezione precedente hai imparato a scrivere una classe con attributi privati, costruttori, getter e setter. Hai visto che dichiarare gli attributi `private` impedisce accessi non controllati dall'esterno.

Ma perché è così importante? E c'è un nome per questo principio?

Sì — si chiama **incapsulamento**, ed è uno dei tre pilastri dell'OOP.

---

## Il problema degli attributi pubblici

Prima di parlare della soluzione, guardiamo ancora una volta il problema. Se gli attributi fossero pubblici:

```java
public class Personaggio {
    public String nome;
    public int vita;
    public int vitaMax;
    public int attacco;
    public int livello;
}
```

Chiunque potrebbe scrivere queste cose dall'esterno:

```java
eroe.vita = -9999;      // vita negativa — il personaggio è "morto" ma non lo sa
eroe.vitaMax = 0;       // vitaMax a zero — nessun senso
eroe.livello = -5;      // livello negativo — impossibile
eroe.attacco = 0;       // personaggio inutile
```

L'oggetto entra in uno **stato inconsistente** — una configurazione che non ha senso nel contesto del problema. E nessuno lo impedisce.

::: {.callout-warning}
## Stato inconsistente
Uno stato inconsistente è quando un oggetto contiene valori che violano le regole del dominio: vita negativa, livello zero, nome vuoto. Con attributi pubblici è impossibile garantire che queste situazioni non accadano.
:::

---

## Incapsulamento

L'**incapsulamento** è il principio per cui i dati interni di un oggetto sono **nascosti** e accessibili solo attraverso metodi controllati.

In pratica:
- Gli attributi sono `private` — invisibili dall'esterno
- I metodi sono `public` — l'interfaccia attraverso cui si interagisce
- I setter contengono **controlli** che garantiscono la validità dei dati

```java
public void setVita(int vita) {
    if (vita < 0) vita = 0;
    if (vita > vitaMax) vita = vitaMax;
    this.vita = vita;
}

public void setLivello(int livello) {
    if (livello > 0) this.livello = livello;
}
```

Ora è impossibile portare la vita sotto zero o il livello a un valore negativo — il setter lo impedisce silenziosamente.

::: {.callout-tip}
## L'analogia del controller
Pensa a un controller di una console: hai dei pulsanti (l'interfaccia pubblica) e non ti interessa cosa succede dentro i circuiti quando li premi. Sai solo che premendo X salti, premendo B attacchi.

Allo stesso modo, chi usa un oggetto `Personaggio` non deve sapere come sono gestiti vita e livello internamente — usa i metodi pubblici e ottiene i risultati.
:::

---

## Information hiding

L'incapsulamento realizza un principio più generale chiamato **information hiding**: l'oggetto espone solo ciò che serve all'esterno e nasconde tutti i dettagli interni.

I vantaggi sono concreti:

**Sicurezza** — i dati non possono essere corrotti dall'esterno. Nessuno può portare `vita` a -9999.

**Flessibilità** — puoi cambiare completamente l'implementazione interna senza che il resto del programma se ne accorga. Se decidi di calcolare la vita in modo diverso, basta modificare i metodi interni — chi usa la classe non deve cambiare nulla.

**Leggibilità** — chi usa la classe sa esattamente cosa può e non può fare. L'interfaccia pubblica è il contratto.

---

## Tipi di dato astratti (ADT)

Arriviamo ora al concetto teorico che sta dietro a tutto questo: il **tipo di dato astratto** (ADT — Abstract Data Type).

Finora hai usato tipi già pronti: `int`, `double`, `String`, `boolean`. Ma con le classi puoi **definire i tuoi tipi**, costruiti su misura per il problema che vuoi risolvere.

Un ADT è definito da due cose:

- i **valori possibili** — le configurazioni valide per quel tipo
- le **operazioni disponibili** — le azioni che hanno senso su quei valori

::: {.callout-note}
## Definizione
Un tipo di dato astratto descrive **cosa** un tipo rappresenta e **cosa** si può fare con esso, senza specificare **come** è implementato internamente.
:::

L'idea chiave è la separazione tra **interfaccia** e **implementazione**: chi usa un ADT non deve sapere come funziona dentro, gli basta sapere cosa può fare.

---

## Il Personaggio come ADT

Pensiamo al nostro `Personaggio` come ADT.

**Valori possibili:**
Un personaggio valido ha un nome non vuoto, una vita tra 0 e `vitaMax`, un livello positivo, un attacco positivo.

**Operazioni disponibili:**
- creare un personaggio con nome e statistiche iniziali
- farlo subire danno
- farlo attaccare un bersaglio
- farlo salire di livello
- leggere le sue statistiche

Non ci interessa *come* queste operazioni sono implementate. Ci interessa *cosa* fanno e *quali valori accettano*.

```java
// Chi usa la classe sa solo questo — l'interfaccia:
Personaggio eroe = new Personaggio("Kratos", 120, 30, 1);
eroe.subirDanno(25);
eroe.attacca(nemico);
eroe.salaDiLivello();
int v = eroe.getVita();
```

I dettagli interni — come viene calcolata la vita, come funziona il setter — sono irrilevanti per chi usa la classe.

::: {.callout-tip}
## ADT e cambiamento
Possiamo cambiare completamente il modo in cui calcoliamo il danno o gestiamo la vita, senza che il resto del programma se ne accorga — purché l'interfaccia pubblica rimanga la stessa. Questa è la potenza dell'astrazione.
:::

---

## Un secondo esempio: l'arma come ADT

Modelliamo un'arma che si consuma con l'uso, come in molti RPG.

**Valori possibili:** nome non vuoto, danno positivo, durabilità tra 0 e 100.

**Operazioni:** usare l'arma, ripararla, leggere il danno attuale.

```java
public class Arma {

    private String nome;
    private int dannoBase;
    private int durabilita;

    public Arma(String nome, int dannoBase) {
        this.nome = nome;
        this.dannoBase = (dannoBase > 0) ? dannoBase : 1;
        this.durabilita = 100;
    }

    public String getNome()      { return nome; }
    public int getDurabilita()   { return durabilita; }

    // Il danno cala proporzionalmente alla durabilità
    public int getDanno() {
        return (dannoBase * durabilita) / 100;
    }

    public void usa() {
        if (durabilita > 0) {
            durabilita--;
            System.out.println(nome + " usata. Durabilità: " + durabilita + "/100");
        } else {
            System.out.println(nome + " è rotta! Ripararla prima di usarla.");
        }
    }

    public void ripara() {
        durabilita = 100;
        System.out.println(nome + " riparata! Durabilità: 100/100");
    }

    public boolean isRotta() {
        return durabilita == 0;
    }
}
```

Chi usa `Arma` non sa come viene calcolato `getDanno()`. Sa solo che il danno dipende dalla durabilità — questo è tutto ciò che gli serve.

---

## Incapsulamento in azione: simulazione completa

Vediamo tutto insieme — personaggio e arma che collaborano:

```java
public class Main {
    public static void main(String[] args) {

        Personaggio eroe = new Personaggio("Arthas", 150, 0, 1);
        Arma spada = new Arma("Spada del Caos", 50);

        Personaggio nemico = new Personaggio("Lich King", 300, 45, 10);

        System.out.println("=== INIZIO COMBATTIMENTO ===\n");

        // Turno 1
        System.out.println("--- Turno 1 ---");
        int danno = spada.getDanno();
        System.out.println(eroe.getNome() + " colpisce con " +
            spada.getNome() + " per " + danno + " danni!");
        nemico.subirDanno(danno);
        spada.usa();
        nemico.subirDanno(nemico.getAttacco()); // il nemico contrattacca

        System.out.println();

        // Turno 2
        System.out.println("--- Turno 2 ---");
        danno = spada.getDanno();
        System.out.println(eroe.getNome() + " colpisce con " +
            spada.getNome() + " per " + danno + " danni!");
        nemico.subirDanno(danno);
        spada.usa();

        System.out.println();
        System.out.println("Vita di " + nemico.getNome() +
            ": " + nemico.getVita() + "/" + nemico.getVitaMax());
        System.out.println("Vita di " + eroe.getNome() +
            ": " + eroe.getVita() + "/" + eroe.getVitaMax());
        System.out.println("Durabilità " + spada.getNome() +
            ": " + spada.getDurabilita() + "/100");
    }
}
```

::: {.callout-important}
## Osserva cosa non vedi
Nel `main` non c'è nessun accesso diretto agli attributi. Tutto passa attraverso i metodi pubblici. L'implementazione interna di `Personaggio` e `Arma` è completamente nascosta — questo è l'incapsulamento in azione.
:::

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- L'**incapsulamento** nasconde gli attributi (`private`) e li espone solo tramite metodi controllati (`public`).
- L'**information hiding** garantisce che i dettagli implementativi restino nascosti — si espone solo l'interfaccia necessaria.
- Uno **stato inconsistente** è quando un oggetto contiene valori non validi — l'incapsulamento lo previene.
- Un **ADT** definisce un tipo tramite i suoi valori possibili e le sue operazioni, senza specificare l'implementazione.
- La separazione tra **interfaccia** e **implementazione** permette di cambiare il codice interno senza impatto su chi usa la classe.
- La regola d'oro rimane: **attributi `private`, metodi `public`**.
:::

Nella prossima lezione vedremo l'**ereditarietà**: come creare nuove classi a partire da quelle esistenti, riutilizzando il codice già scritto.