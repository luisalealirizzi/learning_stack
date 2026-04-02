# Ereditarietà

---

## Il problema della duplicazione

Immagina di dover modellare i personaggi di un videogioco. Hai diversi tipi: guerriero, mago, arciere. Ognuno ha caratteristiche proprie, ma tutti condividono alcune cose di base: un nome, dei punti vita, un livello, la capacità di subire danno e di salire di livello.

Senza ereditarietà, dovresti riscrivere queste cose **in ogni classe**:

```java
public class Guerriero {
    private String nome;
    private int vita;
    private int livello;
    // ... stesse cose del Mago e dell'Arciere
    private int forza;      // specifico del Guerriero
    private int armatura;   // specifico del Guerriero
}

public class Mago {
    private String nome;    // duplicato!
    private int vita;       // duplicato!
    private int livello;    // duplicato!
    // ...
    private int mana;       // specifico del Mago
    private int intelligenza; // specifico del Mago
}
```

Codice duplicato è codice difficile da mantenere: se vuoi cambiare come funziona `subirDanno()`, devi modificarlo in ogni classe separatamente.

::: {.callout-warning}
## Il problema della duplicazione
Copiare e incollare codice tra classi è uno dei peggiori errori di progettazione. Se trovi un bug in `subirDanno()`, devi ricordarti di correggerlo in `Guerriero`, `Mago`, `Arciere`... e in ogni altro personaggio che hai creato.
:::

---

## La soluzione: ereditarietà

L'**ereditarietà** permette di creare una nuova classe a partire da una già esistente. La nuova classe — chiamata **classe figlia** o sottoclasse — eredita automaticamente tutti gli attributi e i metodi della **classe madre** o superclasse, e può aggiungerne di nuovi.

In Java si usa la parola chiave `extends`:

```java
public class ClasseFiglia extends ClasseMadre {
    // eredita tutto dalla madre
    // può aggiungere nuovi attributi e metodi
}
```

::: {.callout-note}
## Definizione
L'ereditarietà è un meccanismo che permette a una classe di **derivare** da un'altra, acquisendone attributi e metodi senza doverli riscrivere.
:::

---

## Costruiamo la gerarchia

Partiamo dalla classe madre `Personaggio`, che contiene tutto ciò che è comune a ogni tipo di personaggio:

```java
public class Personaggio {

    private String nome;
    private int vita;
    private int vitaMax;
    private int livello;

    // Getter
    public String getNome()  { return nome; }
    public int getVita()     { return vita; }
    public int getVitaMax()  { return vitaMax; }
    public int getLivello()  { return livello; }

    // Setter con controlli
    public void setNome(String n)    { nome = n; }
    public void setLivello(int l)    { if (l > 0) livello = l; }
    public void setVitaMax(int vm)   { if (vm > 0) vitaMax = vm; }
    public void setVita(int v) {
        if (v < 0) v = 0;
        if (v > vitaMax) v = vitaMax;
        vita = v;
    }

    // Metodi comuni
    public void subirDanno(int danno) {
        setVita(vita - danno);
        System.out.println(nome + " subisce " + danno +
            " danni. Vita: " + vita + "/" + vitaMax);
    }

    public boolean isMorto() {
        return vita == 0;
    }

    public void salaDiLivello() {
        livello++;
        vitaMax += 20;
        setVita(vitaMax);
        System.out.println(nome + " sale al livello " + livello + "!");
    }
}
```

Ora creiamo le classi figlie. Il `Guerriero` aggiunge forza e armatura:

```java
public class Guerriero extends Personaggio {

    private int forza;
    private int armatura;

    public int getForza()    { return forza; }
    public int getArmatura() { return armatura; }
    public void setForza(int f)    { if (f > 0) forza = f; }
    public void setArmatura(int a) { if (a >= 0) armatura = a; }

    // Metodo specifico del Guerriero
    public void colpoPoderoso(Personaggio bersaglio) {
        int danno = forza * 2;
        System.out.println(getNome() + " usa Colpo Poderoso!");
        bersaglio.subirDanno(danno);
    }
}
```

Il `Mago` aggiunge mana e intelligenza:

```java
public class Mago extends Personaggio {

    private int mana;
    private int manaMax;
    private int intelligenza;

    public int getMana()         { return mana; }
    public int getIntelligenza() { return intelligenza; }
    public void setMana(int m)   { if (m >= 0 && m <= manaMax) mana = m; }
    public void setManaMax(int mm) { if (mm > 0) manaMax = mm; }
    public void setIntelligenza(int i) { if (i > 0) intelligenza = i; }

    // Metodo specifico del Mago
    public void lanciaFirball(Personaggio bersaglio) {
        int costomana = 30;
        if (mana >= costomana) {
            int danno = intelligenza * 3;
            System.out.println(getNome() + " lancia Fireball!");
            bersaglio.subirDanno(danno);
            setMana(mana - costomana);
        } else {
            System.out.println(getNome() + " non ha abbastanza mana!");
        }
    }
}
```

---

## Usare la gerarchia

Creiamo i personaggi e usiamoli:

```java
Guerriero tank = new Guerriero();
tank.setNome("Arthas");
tank.setVitaMax(200);
tank.setVita(200);
tank.setLivello(5);
tank.setForza(40);
tank.setArmatura(20);

Mago mago = new Mago();
mago.setNome("Gandalf");
mago.setVitaMax(100);
mago.setVita(100);
mago.setLivello(8);
mago.setManaMax(150);
mago.setMana(150);
mago.setIntelligenza(60);

Guerriero nemico = new Guerriero();
nemico.setNome("Lich King");
nemico.setVitaMax(300);
nemico.setVita(300);
nemico.setForza(50);

// Arthas usa un metodo ereditato da Personaggio
tank.subirDanno(30);

// Arthas usa un metodo specifico del Guerriero
tank.colpoPoderoso(nemico);

// Gandalf usa un metodo specifico del Mago
mago.lanciaFirball(nemico);
```

::: {.callout-tip}
## Cosa ha ereditato il Guerriero?
`Guerriero` non ha un metodo `subirDanno()` — lo eredita da `Personaggio`. Idem per `isMorto()`, `salaDiLivello()`, tutti i getter e setter di base. Non è stato necessario riscriverli.
:::

---

## La parola chiave `super`

A volte la classe figlia vuole **estendere** il comportamento di un metodo della madre, non sostituirlo. Si usa la parola chiave `super` per chiamare il metodo della classe madre:

```java
public class Guerriero extends Personaggio {

    private int armatura;

    // Il Guerriero riduce il danno grazie all'armatura
    @Override
    public void subirDanno(int danno) {
        int dannoRidotto = Math.max(0, danno - armatura);
        System.out.println(getNome() + " blocca " +
            (danno - dannoRidotto) + " danni con l'armatura.");
        super.subirDanno(dannoRidotto); // chiama il metodo della madre
    }
}
```

Ora quando un `Guerriero` subisce danno, prima calcola la riduzione dell'armatura, poi delega il resto a `Personaggio.subirDanno()`.

---

## Il principio "è un"

Un modo semplice per capire se l'ereditarietà ha senso è chiedersi: *"X è un Y?"*

- Un `Guerriero` **è un** `Personaggio` ✅ → ereditarietà corretta
- Un `Mago` **è un** `Personaggio` ✅ → ereditarietà corretta
- Un `Guerriero` **è una** `Arma` ❌ → ereditarietà sbagliata

Se la frase non ha senso, l'ereditarietà non va usata.

::: {.callout-important}
## Ereditarietà non è composizione
Se un `Guerriero` **ha una** `Arma`, non deve estendere `Arma` — deve avere un attributo di tipo `Arma`. L'ereditarietà modella una relazione "è un", non "ha un".

```java
public class Guerriero extends Personaggio {
    private Arma armaPrincipale;  // "ha una" Arma → attributo
}
```
:::

---

## La gerarchia completa

Ecco la struttura che abbiamo costruito:

```
Personaggio
├── Guerriero
│     └── metodi aggiuntivi: colpoPoderoso()
└── Mago
      └── metodi aggiuntivi: lanciaFireball()
```

`Personaggio` definisce il contratto comune. `Guerriero` e `Mago` lo specializzano aggiungendo comportamenti propri — senza duplicare nemmeno una riga di codice comune.

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- L'**ereditarietà** permette a una classe figlia di estendere una classe madre con `extends`.
- La classe figlia eredita tutti gli attributi e metodi della madre.
- La classe figlia può aggiungere nuovi attributi e metodi specifici.
- `super` permette di chiamare i metodi della classe madre dalla figlia.
- Il test **"è un"** aiuta a capire se l'ereditarietà è appropriata.
- Non confondere ereditarietà ("è un") con composizione ("ha un").
:::
