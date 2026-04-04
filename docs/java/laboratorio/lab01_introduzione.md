# Lab 01 — Classi e oggetti

*Prima esercitazione guidata — classi, costruttori, getter e setter*

---

## Esercitazione guidata

In questa prima esercitazione creiamo due classi (`Rettangolo` e `Cerchio`) e un `Main` che le usa. L'obiettivo è familiarizzare con la struttura di un progetto Java da terminale.

### Crea la cartella di lavoro

Apri il terminale e digita:

```bash
mkdir -p ~/java-lab && cd ~/java-lab
pwd  # verifica dove sei
```

### Crea i file

Crea tre file separati nella cartella:

**`Rettangolo.java`**

```java
class Rettangolo {
    private int base;
    private int altezza;

    public Rettangolo(int b, int h) {
        base    = (b > 0) ? b : 1;
        altezza = (h > 0) ? h : 1;
    }

    public int getBase()    { return base; }
    public int getAltezza() { return altezza; }
    public int area()       { return base * altezza; }
    public int perimetro()  { return 2 * (base + altezza); }
}
```

**`Cerchio.java`**

```java
class Cerchio {
    private int r;

    public Cerchio(int r) {
        this.r = (r > 0) ? r : 1;
    }

    public int    getRaggio()       { return r; }
    public double area()            { return Math.PI * r * r; }
    public double circonferenza()   { return 2 * Math.PI * r; }
}
```

**`Main.java`**

```java
public class Main {
    public static void main(String[] args) {

        Rettangolo r = new Rettangolo(5, 3);
        Cerchio    c = new Cerchio(4);

        System.out.println("Rettangolo: base="   + r.getBase()   +
                           ", altezza="           + r.getAltezza() +
                           ", area="              + r.area()       +
                           ", perimetro="         + r.perimetro());

        System.out.println("Cerchio: r="          + c.getRaggio()     +
                           ", area="              + c.area()           +
                           ", circonferenza="     + c.circonferenza());
    }
}
```

### Compila

```bash
javac Rettangolo.java Cerchio.java Main.java
```

Verifica che siano comparsi i file `.class`:

```bash
ls
# Cerchio.class  Cerchio.java  Main.class  Main.java  Rettangolo.class  Rettangolo.java
```

### Esegui

```bash
java Main
```

### Pulizia

```bash
rm *.class
```

---

## Variante per compilatori online

Se usi OnlineGDB o un compilatore online, non puoi avere più file separati. Metti tutte le classi nello stesso file — ma **solo `Main` può essere `public`**:

```java
class Rettangolo {
    private int base;
    private int altezza;

    public Rettangolo(int b, int h) {
        base    = (b > 0) ? b : 1;  // operatore ternario
        altezza = (h > 0) ? h : 1;
    }

    public int getBase()    { return base; }
    public int getAltezza() { return altezza; }
    public int area()       { return base * altezza; }
    public int perimetro()  { return 2 * (base + altezza); }
}

class Cerchio {
    private int r;

    public Cerchio(int r) {
        this.r = (r > 0) ? r : 1;
    }

    public int    getRaggio()     { return r; }
    public double area()          { return Math.PI * r * r; }
    public double circonferenza() { return 2 * Math.PI * r; }
}

public class Main {
    public static void main(String[] args) {

        Rettangolo r = new Rettangolo(5, 3);
        Cerchio    c = new Cerchio(4);

        System.out.println("Rettangolo: base="  + r.getBase()    +
                           ", altezza="          + r.getAltezza() +
                           ", area="             + r.area()       +
                           ", perimetro="        + r.perimetro());

        System.out.println("Cerchio: r="         + c.getRaggio()     +
                           ", area="             + c.area()          +
                           ", circonferenza="    + c.circonferenza());
    }
}
```

::: {.callout-note}
## Perché solo una classe `public`?
In Java, se il file si chiama `Main.java`, solo la classe `Main` può essere dichiarata `public`. Le altre classi nello stesso file vanno senza `public` — il compilatore le vede comunque.
:::

---

## Esercizio 1 — Classe Quadrato

### Obiettivo

Creare una classe `Quadrato` che rappresenti un quadrato e permetta di calcolare area e perimetro. Il lato non può mai essere ≤ 0 — se succede, deve diventare 1.

### Specifiche

**Attributo privato:**
- `int lato`

**Costruttori:**
- con parametro `int l` — se `l <= 0`, assegna 1
- senza parametri — inizializza `lato = 1`
- di copia — prende un `Quadrato` in input e copia il lato

**Getter e setter:**
- `getLato()`
- `setLato(int l)` — se il valore è ≤ 0, imposta 1

**Metodi:**
- `area()` → `lato * lato`
- `perimetro()` → `4 * lato`

**Classe di test `TestQuadrato`:**
1. Crea un quadrato di lato 5 → stampa lato, area, perimetro
2. Crea un quadrato di lato 0 → stampa lato (deve diventare 1)
3. Modifica il lato a -3 → verifica che diventi 1

### Output atteso

```
Quadrato di lato 5
Area: 25
Perimetro: 20

Quadrato di lato 1
Area: 1
Perimetro: 4

Nuovo lato: 1
```

---

## Esercizio 2 — Classe PasswordChecker

### Obiettivo

Creare una classe `PasswordChecker` che rappresenti una password e permetta di controllarne la validità.

### Specifiche

**Attributo privato:**
- `String password`

**Costruttori:**
- con parametro `String p` — assegna la password
- senza parametri — imposta una password di default (es. `"default"`)

**Getter e setter:**
- `getPassword()`
- `setPassword(String p)`

**Metodi:**
- `boolean isLongEnough()` → `true` se `password.length() >= 8`
- `boolean hasUpperCase()` → `true` se contiene almeno una lettera maiuscola
- `boolean hasNumber()` → `true` se contiene almeno una cifra
- `boolean isValid()` → `true` se tutte e tre le condizioni sono soddisfatte

### Come implementare `hasNumber()`

**Con `charAt`**:
```java
public boolean hasNumber() {
    for (int i = 0; i < password.length(); i++) {
        if (Character.isDigit(password.charAt(i))) {
            return true;  // trovata una cifra
        }
    }
    return false;  // nessuna cifra trovata
}
```

### Classe di test `TestPassword`

Crea due password (`"ciao"` e `"Ciao2025"`) e verifica tutti i controlli. Stampa un messaggio finale: `"Password valida"` oppure `"Password NON valida"`.

### Output atteso

```
Password: ciao
Lunghezza sufficiente? false
Contiene maiuscole?    false
Contiene numeri?       false
Password NON valida

Password: Ciao2025
Lunghezza sufficiente? true
Contiene maiuscole?    true
Contiene numeri?       true
Password valida
```