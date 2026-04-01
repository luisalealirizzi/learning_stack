# Classi astratte e interfacce

---

## Il problema del metodo "generico"

Nella lezione sull'ereditarietà abbiamo costruito questa gerarchia:

```
Personaggio
├── Guerriero
├── Mago
└── Arciere
```

Ogni sottoclasse ridefinisce `attacca()` con il proprio comportamento. Ma la versione di `attacca()` in `Personaggio` — quella base — non ha molto senso da sola:

```java
public void attacca(Personaggio bersaglio) {
    System.out.println(getNome() + " attacca con un colpo generico.");
    bersaglio.subirDanno(10);
}
```

Un "attacco generico" non esiste nel nostro gioco. `Personaggio` è un concetto astratto — nella realtà esistono solo guerrieri, maghi, arcieri. Nessuno crea mai un `Personaggio` puro.

Come impediamo che qualcuno scriva `new Personaggio()`? E come forziamo ogni sottoclasse a implementare `attacca()`?

La risposta è la **classe astratta**.

---

## Classi astratte

Una **classe astratta** è una classe che non può essere istanziata direttamente — esiste solo per essere estesa. Si dichiara con la parola chiave `abstract`.

```java
public abstract class Personaggio {

    private String nome;
    private int vita;
    private int vitaMax;
    private int attacco;
    private int livello;

    public Personaggio(String nome, int vitaMax, int attacco, int livello) {
        this.nome = nome;
        this.vitaMax = (vitaMax > 0) ? vitaMax : 100;
        this.vita = this.vitaMax;
        this.attacco = (attacco > 0) ? attacco : 10;
        this.livello = (livello > 0) ? livello : 1;
    }

    // Getter
    public String getNome()  { return nome; }
    public int getVita()     { return vita; }
    public int getVitaMax()  { return vitaMax; }
    public int getAttacco()  { return attacco; }
    public int getLivello()  { return livello; }

    // Setter
    public void setVita(int vita) {
        if (vita < 0) vita = 0;
        if (vita > vitaMax) vita = vitaMax;
        this.vita = vita;
    }

    // Metodi concreti — ereditati da tutte le sottoclassi
    public void subirDanno(int danno) {
        setVita(vita - danno);
        System.out.println(nome + " subisce " + danno +
            " danni. Vita: " + vita + "/" + vitaMax);
    }

    public boolean isMorto() { return vita == 0; }

    public void salaDiLivello() {
        livello++;
        vitaMax += 20;
        setVita(vitaMax);
        attacco += 5;
        System.out.println(nome + " sale al livello " + livello + "!");
    }

    // Metodo astratto — ogni sottoclasse DEVE implementarlo
    public abstract void attacca(Personaggio bersaglio);

    public void stampaScheda() {
        System.out.println("=== " + nome + " [" + getClass().getSimpleName() + "] ===");
        System.out.println("Livello: " + livello + " | Vita: " +
            vita + "/" + vitaMax + " | Attacco: " + attacco);
    }
}
```

::: {.callout-note}
## Regole delle classi astratte
- Si dichiarano con `abstract class`
- **Non possono essere istanziate** direttamente — `new Personaggio()` dà errore
- Possono avere sia metodi **concreti** (con implementazione) sia metodi **astratti** (senza)
- Ogni classe figlia **deve** implementare tutti i metodi astratti, altrimenti deve essere astratta anche lei
:::

---

## Metodi astratti

Un **metodo astratto** è un metodo senza corpo — solo la firma, seguita da `;`. Dice alle sottoclassi: *"questo metodo deve esistere, ma tocca a te decidere come funziona"*.

```java
public abstract void attacca(Personaggio bersaglio);  // nessun corpo!
```

Se una sottoclasse non implementa tutti i metodi astratti, il compilatore dà errore. È un modo per **forzare un contratto**: ogni tipo concreto di personaggio deve saper attaccare.

---

## Le sottoclassi concrete

Ora le sottoclassi non possono più ignorare `attacca()` — devono implementarlo:

```java
public class Guerriero extends Personaggio {

    private int armatura;

    public Guerriero(String nome, int vitaMax, int attacco, int livello, int armatura) {
        super(nome, vitaMax, attacco, livello);
        this.armatura = (armatura >= 0) ? armatura : 0;
    }

    @Override
    public void attacca(Personaggio bersaglio) {
        int danno = getAttacco() + 10;
        System.out.println(getNome() + " colpisce con la spada!");
        bersaglio.subirDanno(danno);
    }

    @Override
    public void subirDanno(int danno) {
        int dannoRidotto = Math.max(0, danno - armatura);
        System.out.println(getNome() + " blocca " +
            (danno - dannoRidotto) + " danni con l'armatura.");
        super.subirDanno(dannoRidotto);
    }
}
```

```java
public class Mago extends Personaggio {

    private int mana;
    private int manaMax;

    public Mago(String nome, int vitaMax, int attacco, int livello, int manaMax) {
        super(nome, vitaMax, attacco, livello);
        this.manaMax = manaMax;
        this.mana = manaMax;
    }

    public int getMana() { return mana; }

    @Override
    public void attacca(Personaggio bersaglio) {
        int costoMana = 25;
        if (mana >= costoMana) {
            int danno = getAttacco() * 2;
            System.out.println(getNome() + " lancia Fireball!");
            bersaglio.subirDanno(danno);
            mana -= costoMana;
            System.out.println("Mana rimasto: " + mana + "/" + manaMax);
        } else {
            System.out.println(getNome() + " non ha mana! Attacco fisico.");
            bersaglio.subirDanno(getAttacco());
        }
    }
}
```

---

## Interfacce

Le classi astratte risolvono il problema dell'astrazione nella gerarchia. Ma Java non supporta l'**ereditarietà multipla** — una classe può estendere una sola classe madre.

E se volessimo aggiungere capacità trasversali? Ad esempio, alcune entità del gioco possono essere **curate**, altre possono essere **salvate** su file, altre ancora possono essere **osservate** da altri oggetti.

Queste capacità non appartengono alla gerarchia `Personaggio` — sono comportamenti che classi diverse possono condividere indipendentemente.

Per questo esistono le **interfacce**.

::: {.callout-note}
## Definizione
Un'interfaccia è un **contratto**: dichiara quali metodi una classe deve implementare, senza fornire nessuna implementazione. Una classe può implementare **quante interfacce vuole**.
:::

---

## Dichiarare un'interfaccia

```java
public interface Curabile {
    void cura(int quantita);
    int getVita();
    int getVitaMax();
}
```

```java
public interface Salvabile {
    String toStringSalvataggio();
    void caricaDaSalvataggio(String dati);
}
```

Un'interfaccia può anche avere **metodi default** — metodi con un'implementazione di base che le classi possono sovrascrivere o usare così com'è:

```java
public interface Curabile {
    void cura(int quantita);
    int getVita();
    int getVitaMax();

    // Metodo default — implementazione di base
    default boolean isFerito() {
        return getVita() < getVitaMax();
    }
}
```

---

## Implementare un'interfaccia

Una classe implementa un'interfaccia con la parola chiave `implements`:

```java
public class Guerriero extends Personaggio implements Curabile, Salvabile {

    private int armatura;

    public Guerriero(String nome, int vitaMax, int attacco, int livello, int armatura) {
        super(nome, vitaMax, attacco, livello);
        this.armatura = (armatura >= 0) ? armatura : 0;
    }

    @Override
    public void attacca(Personaggio bersaglio) {
        bersaglio.subirDanno(getAttacco() + 10);
    }

    // Implementazione di Curabile
    @Override
    public void cura(int quantita) {
        setVita(getVita() + quantita);
        System.out.println(getNome() + " recupera " + quantita +
            " punti vita. Vita: " + getVita() + "/" + getVitaMax());
    }

    // Implementazione di Salvabile
    @Override
    public String toStringSalvataggio() {
        return getNome() + "," + getVita() + "," +
            getVitaMax() + "," + getAttacco() + "," + getLivello();
    }

    @Override
    public void caricaDaSalvataggio(String dati) {
        String[] parti = dati.split(",");
        // parsing e ripristino dello stato...
        System.out.println("Personaggio caricato: " + parti[0]);
    }
}
```

::: {.callout-important}
## Una classe può implementare più interfacce
```java
public class Guerriero extends Personaggio implements Curabile, Salvabile {
//                     ↑ una sola classe madre    ↑ quante interfacce si vuole
```
Questo è il modo in cui Java simula l'ereditarietà multipla in modo sicuro.
:::

---

## Classe astratta vs interfaccia

È una domanda frequente. Ecco le differenze principali:

| | Classe astratta | Interfaccia |
|---|---|---|
| **Ereditarietà** | Una sola (`extends`) | Illimitata (`implements`) |
| **Metodi concreti** | ✅ Sì | ✅ Solo `default` |
| **Attributi** | ✅ Sì | ❌ Solo costanti |
| **Costruttore** | ✅ Sì | ❌ No |
| **Quando usarla** | Gerarchia con codice condiviso | Comportamento trasversale |

::: {.callout-tip}
## La regola pratica
Usa una **classe astratta** quando le classi condividono codice e appartengono alla stessa famiglia (`Personaggio` → `Guerriero`, `Mago`).

Usa un'**interfaccia** quando vuoi definire una capacità che classi diverse e indipendenti possono avere (`Curabile`, `Salvabile`, `Comparabile`).
:::

---

## Esempio completo

```java
public class Main {
    public static void main(String[] args) {

        // Non posso scrivere: new Personaggio(...) → ERRORE!
        Guerriero eroe = new Guerriero("Arthas", 150, 30, 1, 10);
        Mago mago = new Mago("Gandalf", 100, 25, 3, 120);

        eroe.stampaScheda();
        mago.stampaScheda();

        System.out.println("\n--- COMBATTIMENTO ---\n");

        mago.attacca(eroe);  // Fireball
        eroe.attacca(mago);  // Colpo con spada

        System.out.println("\n--- CURA ---\n");

        // Posso usare il tipo interfaccia come variabile
        Curabile paziente = eroe;
        if (paziente.isFerito()) {
            paziente.cura(50);
        }

        System.out.println("\n--- SALVATAGGIO ---\n");

        Salvabile salvataggio = eroe;
        String dati = salvataggio.toStringSalvataggio();
        System.out.println("Dati salvati: " + dati);
    }
}
```

Nota come `eroe` può essere trattato come `Guerriero`, come `Curabile` e come `Salvabile` — è lo stesso oggetto visto attraverso interfacce diverse. Questo è il polimorfismo applicato alle interfacce.

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- Una **classe astratta** non può essere istanziata — esiste solo per essere estesa.
- Un **metodo astratto** non ha corpo — ogni sottoclasse concreta deve implementarlo.
- Una **interfaccia** è un contratto puro: dichiara metodi senza implementazione (salvo i `default`).
- Una classe può estendere **una sola** classe madre ma implementare **quante interfacce vuole**.
- Usa la classe astratta per **gerarchie con codice condiviso**; usa l'interfaccia per **capacità trasversali**.
- Le interfacce permettono il **polimorfismo** anche tra classi non imparentate.
:::