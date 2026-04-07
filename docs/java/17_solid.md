# SOLID

---

## Cos'è SOLID

Quando un programma è piccolo, quasi qualsiasi approccio funziona. Ma quando cresce — più classi, più funzionalità, più persone che ci lavorano, il codice scritto senza criteri diventa difficile da capire, da modificare e da testare.

SOLID è un insieme di cinque principi di progettazione che aiutano a scrivere codice orientato agli oggetti che sia:
- facile da leggere
- facile da modificare senza rompere quello che già funziona
- facile da estendere con nuove funzionalità

Ogni lettera di SOLID corrisponde a un principio.

| Lettera | Nome | In breve |
|---|---|---|
| **S** | Single Responsibility | una classe, un solo compito |
| **O** | Open/Closed | aperto alle estensioni, chiuso alle modifiche |
| **L** | Liskov Substitution | le sottoclassi devono poter sostituire la madre |
| **I** | Interface Segregation | interfacce piccole e specifiche |
| **D** | Dependency Inversion | dipendi dalle astrazioni, non dalle implementazioni |

Non devi memorizzare le definizioni. Devi capire il problema che ogni principio risolve.

---

## S - Single Responsibility Principle

> Una classe deve avere un solo motivo per cambiare.

In altre parole: una classe fa una cosa sola e la fa bene.

### Il problema

```java
public class Personaggio {
    private String nome;
    private int vita;
    private int livello;

    // logica del personaggio
    public void subirDanno(int danno) { ... }
    public void salaDiLivello() { ... }

    // logica di salvataggio -- non c'entra con il personaggio!
    public void salvaSuFile(String percorso) { ... }
    public void caricaDaFile(String percorso) { ... }

    // logica di stampa -- non c'entra con il personaggio!
    public void stampaScheda() { ... }
    public void stampaStatistiche() { ... }
}
```

`Personaggio` fa troppe cose: gestisce lo stato del personaggio, salva su file e stampa. Se cambia il formato del file di salvataggio, devi modificare `Personaggio` -- ma il personaggio non c'entra nulla con il file.

### La soluzione

Separa le responsabilità in classi diverse:

```java
public class Personaggio {
    private String nome;
    private int vita;
    private int livello;

    public void subirDanno(int danno) { ... }
    public void salaDiLivello() { ... }
    // solo la logica del personaggio
}

public class PersonaggioRepository {
    public void salva(Personaggio p, String percorso) { ... }
    public Personaggio carica(String percorso) { ... }
    // solo la logica di persistenza
}

public class PersonaggioView {
    public void stampaScheda(Personaggio p) { ... }
    public void stampaStatistiche(Personaggio p) { ... }
    // solo la logica di presentazione
}
```

::: {.callout-note}
## Schema mentale
Quando scrivi una classe, chiediti: "quanti motivi ho per modificare questa classe in futuro?". Se la risposta e piu di uno, probabilmente sta facendo troppe cose.
:::

---

## O - Open/Closed Principle

> Una classe deve essere aperta alle estensioni ma chiusa alle modifiche.

In altre parole: aggiungi funzionalita nuove senza toccare il codice che gia funziona.

### Il problema

```java
public class CalcolatoreTariffa {

    public double calcola(Personaggio p, String tipo) {
        if (tipo.equals("normale")) {
            return p.getLivello() * 10.0;
        } else if (tipo.equals("premium")) {
            return p.getLivello() * 10.0 * 0.8;
        } else if (tipo.equals("vip")) {
            return p.getLivello() * 10.0 * 0.5;
        }
        // ogni volta che aggiungi un tipo, devi modificare questo metodo
        return 0;
    }
}
```

Ogni nuovo tipo di tariffa richiede di aprire e modificare `CalcolatoreTariffa`. Rischi di rompere i tipi che gia funzionano.

### La soluzione

Usa un'interfaccia per definire il contratto, e una classe separata per ogni variante:

```java
public interface Tariffa {
    double calcola(Personaggio p);
}

public class TariffaNormale implements Tariffa {
    @Override
    public double calcola(Personaggio p) {
        return p.getLivello() * 10.0;
    }
}

public class TariffaPremium implements Tariffa {
    @Override
    public double calcola(Personaggio p) {
        return p.getLivello() * 10.0 * 0.8;
    }
}

public class TariffaVip implements Tariffa {
    @Override
    public double calcola(Personaggio p) {
        return p.getLivello() * 10.0 * 0.5;
    }
}
```

Per aggiungere un nuovo tipo di tariffa, crei una nuova classe che implementa `Tariffa` -- senza toccare nulla di quello che esiste gia.

::: {.callout-note}
## Schema mentale
Quando ti trovi ad aggiungere un `else if` a una catena gia lunga, fermati. Probabilmente stai violando Open/Closed. Chiediti: posso risolvere questo con una nuova classe invece di modificare quella esistente?
:::

---

## L - Liskov Substitution Principle

> Se B estende A, allora ovunque si usa A si deve poter usare B senza che il programma si comporti in modo inatteso.

In altre parole: una sottoclasse deve poter sostituire la classe madre senza sorprese.

### Il problema

```java
public class Rettangolo {
    protected int base;
    protected int altezza;

    public void setBase(int base)       { this.base = base; }
    public void setAltezza(int altezza) { this.altezza = altezza; }
    public int area()                   { return base * altezza; }
}

public class Quadrato extends Rettangolo {
    @Override
    public void setBase(int lato) {
        this.base = lato;
        this.altezza = lato;   // il quadrato ha i lati uguali
    }

    @Override
    public void setAltezza(int lato) {
        this.base = lato;
        this.altezza = lato;
    }
}
```

In apparenza sembra corretto. Ma:

```java
Rettangolo r = new Quadrato();
r.setBase(5);
r.setAltezza(3);
System.out.println(r.area());   // ci aspettiamo 15, otteniamo 9
```

`Quadrato` si comporta in modo diverso da `Rettangolo` -- viola il principio di Liskov.

### La soluzione

Se una sottoclasse non puo rispettare il contratto della madre, ripensa la gerarchia. In questo caso, `Quadrato` e `Rettangolo` non hanno un rapporto "e un" corretto -- entrambi possono estendere una classe `Figura` piu generica.

```java
public abstract class Figura {
    public abstract int area();
}

public class Rettangolo extends Figura {
    private int base, altezza;
    public Rettangolo(int base, int altezza) { ... }
    public int area() { return base * altezza; }
}

public class Quadrato extends Figura {
    private int lato;
    public Quadrato(int lato) { ... }
    public int area() { return lato * lato; }
}
```

::: {.callout-note}
## Schema mentale
Se una sottoclasse deve "sabotare" un metodo della madre -- svuotarlo, lanciare un'eccezione, cambiarne il significato -- stai probabilmente violando Liskov. Ripensa la gerarchia.
:::

---

## I - Interface Segregation Principle

> Non costringere una classe a implementare metodi che non usa.

In altre parole: e meglio avere tante interfacce piccole e specifiche che una sola grande e generica.

### Il problema

```java
public interface Personaggio {
    void attacca();
    void difendi();
    void lanciaIncantesimo();
    void cavalca();
    void nuota();
}

public class Guerriero implements Personaggio {
    public void attacca() { ... }
    public void difendi() { ... }
    public void lanciaIncantesimo() {
        throw new UnsupportedOperationException(); // il guerriero non lancia incantesimi!
    }
    public void cavalca() { ... }
    public void nuota() {
        throw new UnsupportedOperationException(); // il guerriero non sa nuotare!
    }
}
```

`Guerriero` e costretto a implementare metodi che non ha senso abbia. L'interfaccia e troppo grande.

### La soluzione

Dividi l'interfaccia in parti piu piccole e specifiche:

```java
public interface Combattente {
    void attacca();
    void difendi();
}

public interface Mago {
    void lanciaIncantesimo();
}

public interface Cavaliere {
    void cavalca();
}

public interface Nuotatore {
    void nuota();
}

// Il guerriero implementa solo quello che sa fare
public class Guerriero implements Combattente, Cavaliere {
    public void attacca() { ... }
    public void difendi() { ... }
    public void cavalca() { ... }
}

// Il mago implementa quello che sa fare lui
public class Mago implements Combattente, Mago {
    public void attacca() { ... }
    public void difendi() { ... }
    public void lanciaIncantesimo() { ... }
}
```

::: {.callout-note}
## Schema mentale
Se una classe implementa un'interfaccia ma alcuni metodi li lascia vuoti o lancia eccezioni, l'interfaccia e troppo grande. Spezzala.
:::

---

## D - Dependency Inversion Principle

> I moduli di alto livello non devono dipendere dai moduli di basso livello. Entrambi devono dipendere dalle astrazioni.

In altre parole: dipendi dalle interfacce, non dalle implementazioni concrete.

### Il problema

```java
public class GestorePartita {

    private SalvataggioSuFile salvataggio;   // dipende da una classe concreta!

    public GestorePartita() {
        this.salvataggio = new SalvataggioSuFile();
    }

    public void salva(Personaggio p) {
        salvataggio.salva(p);
    }
}
```

`GestorePartita` e legato a `SalvataggioSuFile`. Se vuoi cambiare il sistema di salvataggio -- per esempio salvare su database o sul cloud -- devi modificare `GestorePartita`.

### La soluzione

Introduce un'interfaccia e passa l'implementazione dall'esterno:

```java
public interface Salvataggio {
    void salva(Personaggio p);
    Personaggio carica();
}

public class SalvataggioSuFile implements Salvataggio {
    public void salva(Personaggio p) { ... }
    public Personaggio carica() { ... }
}

public class SalvataggioSuDatabase implements Salvataggio {
    public void salva(Personaggio p) { ... }
    public Personaggio carica() { ... }
}

public class GestorePartita {

    private Salvataggio salvataggio;   // dipende dall'interfaccia

    public GestorePartita(Salvataggio salvataggio) {
        this.salvataggio = salvataggio;   // riceve l'implementazione dall'esterno
    }

    public void salva(Personaggio p) {
        salvataggio.salva(p);
    }
}
```

Ora puoi cambiare il sistema di salvataggio senza toccare `GestorePartita`:

```java
GestorePartita g1 = new GestorePartita(new SalvataggioSuFile());
GestorePartita g2 = new GestorePartita(new SalvataggioSuDatabase());
```

::: {.callout-note}
## Schema mentale
Se dentro una classe scrivi `new NomeClasseConcreta()` per creare una dipendenza, chiediti: questa classe deve davvero sapere quale implementazione usare? Spesso la risposta e no -- ricevila dall'esterno tramite il costruttore.
:::

---

## I cinque principi insieme

SOLID non e una lista di regole da seguire ciecamente. E un insieme di domande da farti mentre progetti:

| Principio | La domanda da farti |
|---|---|
| **S** | Questa classe ha piu di un motivo per cambiare? |
| **O** | Posso aggiungere questa funzionalita senza modificare codice esistente? |
| **L** | Le mie sottoclassi si comportano come ci si aspetta dalla madre? |
| **I** | Le mie interfacce hanno metodi che alcune classi non useranno mai? |
| **D** | Le mie classi dipendono da implementazioni concrete invece che da astrazioni? |

Se la risposta a queste domande e si, c'e spazio per migliorare il design.

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- **S** - Single Responsibility: una classe, una sola responsabilita. Se fa troppe cose, spezzala.
- **O** - Open/Closed: estendi con nuove classi, non modificare quelle esistenti. Usa interfacce e polimorfismo.
- **L** - Liskov Substitution: una sottoclasse deve comportarsi come la madre. Se non puo, ripensa la gerarchia.
- **I** - Interface Segregation: interfacce piccole e specifiche. Nessuna classe deve implementare metodi inutili.
- **D** - Dependency Inversion: dipendi dalle interfacce, non dalle classi concrete. Passa le dipendenze dall'esterno.
:::