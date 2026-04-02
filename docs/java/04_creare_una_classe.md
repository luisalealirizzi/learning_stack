# Creare una classe

---

## Da struct a class

In C avevi le **struct** per raggruppare dati correlati. Una struct è un contenitore di variabili — niente di più. Le funzioni che operano su quei dati stanno altrove, separate.

```c
// In C
struct Personaggio {
    char nome[50];
    int vita;
    int attacco;
};

// La funzione sta fuori dalla struct
void subirDanno(struct Personaggio* p, int danno) {
    p->vita -= danno;
}
```

In Java la **classe** fa un passo avanti decisivo: raccoglie insieme i dati **e** i metodi che li gestiscono. Dati e comportamenti sono un'unica cosa coerente.

| | C (`struct`) | Java (`class`) |
|---|---|---|
| **Dati** | ✅ | ✅ |
| **Metodi** | ❌ (separati) | ✅ (dentro la classe) |
| **Accesso controllato** | ❌ | ✅ (public/private) |
| **Costruttore** | ❌ | ✅ |

---

## Struttura di una classe Java

Una classe ben scritta ha sempre questa struttura:

```java
public class NomeClasse {

    // 1. Attributi (privati)
    private tipo attributo1;
    private tipo attributo2;

    // 2. Costruttore/i
    public NomeClasse(parametri) {
        // inizializza gli attributi
    }

    // 3. Getter e setter
    public tipo getAttributo1() { return attributo1; }
    public void setAttributo1(tipo valore) { attributo1 = valore; }

    // 4. Altri metodi
    public void metodo() { ... }
}
```

Vediamo ogni parte nel dettaglio.

---

## Attributi e incapsulamento

Gli **attributi** sono le variabili che descrivono lo stato dell'oggetto. Per convenzione si dichiarano sempre **`private`** — non possono essere modificati direttamente dall'esterno.

```java
public class Personaggio {
    private String nome;
    private int vita;
    private int vitaMax;
    private int attacco;
    private int livello;
}
```

Perché `private`? Perché senza controllo, chiunque potrebbe scrivere:

```java
eroe.vita = -9999;   // vita negativa — stato non valido!
eroe.livello = 999;  // livello impossibile
```

Con gli attributi privati, questi accessi diretti sono **vietati dal compilatore**. I dati sono al sicuro dentro l'oggetto.

::: {.callout-note}
## La regola d'oro
**Attributi `private`, metodi `public`.**

I dati sono nascosti. Il comportamento è pubblico. Chi usa la classe interagisce solo attraverso i metodi — non tocca mai i dati direttamente.
:::

---

## Il costruttore

Il **costruttore** è un metodo speciale che viene chiamato automaticamente quando si crea un oggetto con `new`. Il suo compito è **inizializzare gli attributi** in modo che l'oggetto parta sempre da uno stato valido.

Caratteristiche del costruttore:
- Ha lo **stesso nome della classe**
- **Non ha tipo di ritorno** (nemmeno `void`)
- Viene chiamato automaticamente da `new`

```java
public class Personaggio {
    private String nome;
    private int vita;
    private int vitaMax;
    private int attacco;
    private int livello;

    // Costruttore
    public Personaggio(String nome, int vitaMax, int attacco, int livello) {
        this.nome = nome;
        this.vitaMax = vitaMax;
        this.vita = vitaMax;   // parte con vita piena
        this.attacco = attacco;
        this.livello = livello;
    }
}
```

Ora creare un oggetto è immediato e sicuro:

```java
Personaggio eroe = new Personaggio("Kratos", 100, 25, 1);
Personaggio nemico = new Personaggio("Drago", 200, 40, 5);
```

---

## La parola chiave `this`

Nel costruttore hai visto `this.nome = nome`. A cosa serve `this`?

`this` è un riferimento all'**oggetto corrente** — quello su cui il metodo sta operando. Serve per distinguere tra il parametro del costruttore e l'attributo della classe quando hanno lo stesso nome.

```java
public Personaggio(String nome, int vitaMax) {
    this.nome = nome;       // this.nome = attributo della classe
                            //       nome = parametro del costruttore
    this.vitaMax = vitaMax;
    this.vita = vitaMax;
}
```

Senza `this`, Java non saprebbe se `nome` si riferisce all'attributo o al parametro. Con `this` è esplicito: stai parlando dell'attributo dell'oggetto corrente.

::: {.callout-tip}
## Quando usare `this`
`this` è obbligatorio quando il parametro ha lo stesso nome dell'attributo. Molti sviluppatori lo usano sempre per chiarezza, anche quando non strettamente necessario — rende immediatamente chiaro che stai operando su un attributo dell'oggetto.
:::

---

## Tipi di costruttori

Una classe può avere **più costruttori** con parametri diversi — è l'overloading che abbiamo già visto per i metodi. Il compilatore sceglie quale usare in base agli argomenti passati.

**Costruttore vuoto** — inizializza con valori di default:

```java
public Personaggio() {
    this.nome = "Sconosciuto";
    this.vitaMax = 100;
    this.vita = 100;
    this.attacco = 10;
    this.livello = 1;
}
```

**Costruttore con parametri** — i valori vengono scelti al momento della creazione:

```java
public Personaggio(String nome, int vitaMax, int attacco, int livello) {
    this.nome = nome;
    this.vitaMax = vitaMax;
    this.vita = vitaMax;
    this.attacco = (attacco > 0) ? attacco : 10;
    this.livello = (livello > 0) ? livello : 1;
}
```

**Costruttore di copia** — crea un nuovo oggetto identico a uno esistente:

```java
public Personaggio(Personaggio altro) {
    this.nome = altro.nome;
    this.vitaMax = altro.vitaMax;
    this.vita = altro.vita;
    this.attacco = altro.attacco;
    this.livello = altro.livello;
}
```

Uso dei tre costruttori:

```java
Personaggio p1 = new Personaggio();                       // vuoto
Personaggio p2 = new Personaggio("Arthas", 150, 35, 3);   // con parametri
Personaggio p3 = new Personaggio(p2);                     // copia di p2
```
### Overloading
Hai appena visto tre costruttori con lo stesso nome Personaggio ma parametri diversi. Questo meccanismo si chiama overloading — e non vale solo per i costruttori, ma per qualsiasi metodo.
L'overloading permette di definire più metodi con lo stesso nome nella stessa classe, a patto che abbiano firme diverse — cioè numero o tipo di parametri diversi. Il compilatore sceglie quale versione chiamare in base agli argomenti passati.

::: {.callout-note}
Le regole dell'overloading

I metodi devono avere lo stesso nome
Devono differire per numero e/o tipo di parametri
Il tipo di ritorno da solo non basta — due metodi con stessa firma e tipi di ritorno diversi danno errore di compilazione
L'overloading è risolto a compile time — il compilatore sa già quale versione chiamare
:::

---

## Getter e setter

Con gli attributi privati, come si leggono e modificano i dati dall'esterno? Tramite metodi appositi:

- **Getter** — restituisce il valore di un attributo (`get` + nome attributo)
- **Setter** — modifica il valore, con eventuali controlli (`set` + nome attributo)

```java
public class Personaggio {
    private String nome;
    private int vita;
    private int vitaMax;
    private int attacco;
    private int livello;

    // Getter
    public String getNome()   { return nome; }
    public int getVita()      { return vita; }
    public int getVitaMax()   { return vitaMax; }
    public int getAttacco()   { return attacco; }
    public int getLivello()   { return livello; }

    // Setter con controlli
    public void setNome(String nome) {
        if (nome != null && !nome.isEmpty()) {
            this.nome = nome;
        }
    }

    public void setVita(int vita) {
        if (vita < 0) vita = 0;
        if (vita > vitaMax) vita = vitaMax;
        this.vita = vita;
    }

    public void setAttacco(int attacco) {
        if (attacco > 0) this.attacco = attacco;
    }
}
```

I setter sono le **porte d'ingresso controllate** ai dati. Possono contenere regole — ad esempio impedire vita negativa o attacco nullo — garantendo che l'oggetto mantenga sempre uno stato valido.

::: {.callout-important}
## Non tutti gli attributi hanno bisogno di un setter
Se un attributo non deve mai cambiare dopo la creazione (ad esempio il nome di un personaggio in certi giochi), puoi fornire solo il getter e nessun setter. Così l'attributo diventa di sola lettura dall'esterno.
:::

---

## La classe completa

Mettiamo tutto insieme: attributi privati, costruttori, getter, setter e metodi di comportamento.

```java
public class Personaggio {

    // Attributi
    private String nome;
    private int vita;
    private int vitaMax;
    private int attacco;
    private int livello;

    // Costruttore vuoto
    public Personaggio() {
        this.nome = "Sconosciuto";
        this.vitaMax = 100;
        this.vita = 100;
        this.attacco = 10;
        this.livello = 1;
    }

    // Costruttore con parametri
    public Personaggio(String nome, int vitaMax, int attacco, int livello) {
        this.nome = nome;
        this.vitaMax = (vitaMax > 0) ? vitaMax : 100;
        this.vita = this.vitaMax;
        this.attacco = (attacco > 0) ? attacco : 10;
        this.livello = (livello > 0) ? livello : 1;
    }

    // Costruttore di copia
    public Personaggio(Personaggio altro) {
        this.nome = altro.nome;
        this.vitaMax = altro.vitaMax;
        this.vita = altro.vita;
        this.attacco = altro.attacco;
        this.livello = altro.livello;
    }

    // Getter
    public String getNome()  { return nome; }
    public int getVita()     { return vita; }
    public int getVitaMax()  { return vitaMax; }
    public int getAttacco()  { return attacco; }
    public int getLivello()  { return livello; }

    // Setter
    public void setNome(String nome) {
        if (nome != null && !nome.isEmpty()) this.nome = nome;
    }

    public void setVita(int vita) {
        if (vita < 0) vita = 0;
        if (vita > vitaMax) vita = vitaMax;
        this.vita = vita;
    }

    public void setAttacco(int attacco) {
        if (attacco > 0) this.attacco = attacco;
    }

    // Metodi di comportamento
    public void subirDanno(int danno) {
        setVita(vita - danno);
        System.out.println(nome + " subisce " + danno +
            " danni. Vita: " + vita + "/" + vitaMax);
    }

    public void attacca(Personaggio bersaglio) {
        System.out.println(nome + " attacca " + bersaglio.getNome() + "!");
        bersaglio.subirDanno(attacco);
    }

    public void salaDiLivello() {
        livello++;
        vitaMax += 20;
        setVita(vitaMax);  // vita piena al cambio livello
        attacco += 5;
        System.out.println(nome + " sale al livello " + livello +
            "! Vita: " + vita + " | Attacco: " + attacco);
    }

    public boolean isMorto() {
        return vita == 0;
    }

    public void stampaScheda() {
        System.out.println("=== " + nome + " ===");
        System.out.println("Livello:  " + livello);
        System.out.println("Vita:     " + vita + "/" + vitaMax);
        System.out.println("Attacco:  " + attacco);
        System.out.println("Stato:    " + (isMorto() ? "morto" : "in vita"));
    }
}
```

---

## Usare la classe

```java
public class Main {
    public static void main(String[] args) {

        // Creo due personaggi con il costruttore con parametri
        Personaggio eroe = new Personaggio("Arthas", 120, 30, 1);
        Personaggio boss = new Personaggio("Lich King", 300, 50, 10);

        eroe.stampaScheda();
        System.out.println();
        boss.stampaScheda();

        System.out.println("\n--- COMBATTIMENTO ---\n");

        eroe.attacca(boss);
        boss.attacca(eroe);
        eroe.salaDiLivello();
        eroe.attacca(boss);

        System.out.println();
        eroe.stampaScheda();
        System.out.println();
        boss.stampaScheda();
    }
}
```

Output previsto:
```
=== Arthas ===
Livello:  1
Vita:     120/120
Attacco:  30
Stato:    in vita

=== Lich King ===
Livello:  10
Vita:     300/300
Attacco:  50
Stato:    in vita

--- COMBATTIMENTO ---

Arthas attacca Lich King!
Lich King subisce 30 danni. Vita: 270/300
Lich King attacca Arthas!
Arthas subisce 50 danni. Vita: 70/120
Arthas sale al livello 2! Vita: 140 | Attacco: 35
Arthas attacca Lich King!
Lich King subisce 35 danni. Vita: 235/300

=== Arthas ===
Livello:  2
Vita:     140/140
Attacco:  35
Stato:    in vita

=== Lich King ===
Livello:  10
Vita:     235/300
Attacco:  50
Stato:    in vita
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave 

- In Java la **classe** raccoglie insieme dati (attributi) e comportamenti (metodi) — a differenza della `struct` del C.
- Gli attributi si dichiarano **`private`** per proteggerli da accessi non controllati.
- Il **costruttore** inizializza l'oggetto al momento della creazione con `new`. Ha lo stesso nome della classe e nessun tipo di ritorno.
- **`this`** è un riferimento all'oggetto corrente, utile per distinguere attributi e parametri con lo stesso nome.
- Una classe può avere **più costruttori** con parametri diversi (overloading).
- I **getter** leggono gli attributi; i **setter** li modificano con eventuali controlli.
- La regola d'oro: **attributi `private`, metodi `public`**.
:::