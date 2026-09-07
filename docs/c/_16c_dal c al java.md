# Dal C al Java: lo stesso problema, due soluzioni

---

## Idea chiave

Non stai per imparare un linguaggio completamente nuovo. Stai per vedere che molte cose che finora hai dovuto costruire **a mano** in C, in Java il linguaggio le fa **al posto tuo**.

Hai già fatto la prova generale di questo passaggio quando hai progettato l'ADT con puntatore opaco (`stack.h` / `stack.c`): hai nascosto la struttura interna e hai esposto solo funzioni pubbliche. In Java questo si chiama incapsulamento, e ha una parola chiave dedicata (`private`). Non è un concetto nuovo: è lo stesso concetto, con meno lavoro manuale.

In questo modulo vediamo lo stesso schema applicato a un problema diverso: **gestire comportamenti diversi per tipi diversi di dato**.

---

## Blocco 1: Lo stesso problema, in C

Immagina di dover gestire forme geometriche diverse — cerchi e rettangoli — e calcolarne l'area. Senza classi, in C, il modo naturale è usare un campo che dice "che tipo di forma è questa":

```c
typedef enum { CERCHIO, RETTANGOLO } TipoForma;

typedef struct {
    TipoForma tipo;
    union {
        struct { double raggio; } cerchio;
        struct { double base, altezza; } rettangolo;
    } dati;
} Forma;

double area(Forma *f) {
    switch (f->tipo) {
        case CERCHIO:
            return 3.14159 * f->dati.cerchio.raggio * f->dati.cerchio.raggio;
        case RETTANGOLO:
            return f->dati.rettangolo.base * f->dati.rettangolo.altezza;
    }
    return 0;
}

void stampa(Forma *f) {
    switch (f->tipo) {
        case CERCHIO:
            printf("Cerchio, area = %.2f\n", area(f));
            break;
        case RETTANGOLO:
            printf("Rettangolo, area = %.2f\n", area(f));
            break;
    }
}
```

Funziona. Ma osserva cosa succede se devi aggiungere un terzo tipo, `TRIANGOLO`:

```c
typedef enum { CERCHIO, RETTANGOLO, TRIANGOLO } TipoForma;   /* 1. modifica l'enum */

typedef struct {
    TipoForma tipo;
    union {
        struct { double raggio; } cerchio;
        struct { double base, altezza; } rettangolo;
        struct { double base, altezza; } triangolo;         /* 2. modifica la union */
    } dati;
} Forma;

double area(Forma *f) {
    switch (f->tipo) {
        case CERCHIO:
            return 3.14159 * f->dati.cerchio.raggio * f->dati.cerchio.raggio;
        case RETTANGOLO:
            return f->dati.rettangolo.base * f->dati.rettangolo.altezza;
        case TRIANGOLO:                                      /* 3. modifica area() */
            return f->dati.triangolo.base * f->dati.triangolo.altezza / 2;
    }
    return 0;
}

void stampa(Forma *f) {
    switch (f->tipo) {
        case CERCHIO:
            printf("Cerchio, area = %.2f\n", area(f));
            break;
        case RETTANGOLO:
            printf("Rettangolo, area = %.2f\n", area(f));
            break;
        case TRIANGOLO:                                      /* 4. modifica stampa() */
            printf("Triangolo, area = %.2f\n", area(f));
            break;
    }
}
```

::: {.callout-important}
## Il problema non è la sintassi, è la fragilità
Ogni nuovo tipo di forma ti costringe a tornare a modificare **ogni singola funzione** che fa uno switch sul tipo. In un programma reale potrebbero essere dieci funzioni, scritte in file diversi, magari da colleghi diversi. Basta dimenticare un `case` in una sola di quelle funzioni — il compilatore in certi casi nemmeno te lo segnala — e hai un bug silenzioso.
:::

---

## Blocco 2: La stessa cosa in Java

In Java lo stesso problema si risolve con una gerarchia di classi:

```java
abstract class Forma {
    abstract double area();
    abstract void stampa();
}

class Cerchio extends Forma {
    double raggio;

    Cerchio(double raggio) {
        this.raggio = raggio;
    }

    double area() {
        return 3.14159 * raggio * raggio;
    }

    void stampa() {
        System.out.println("Cerchio, area = " + area());
    }
}

class Rettangolo extends Forma {
    double base, altezza;

    Rettangolo(double base, double altezza) {
        this.base = base;
        this.altezza = altezza;
    }

    double area() {
        return base * altezza;
    }

    void stampa() {
        System.out.println("Rettangolo, area = " + area());
    }
}
```

Uso:

```java
Forma[] forme = { new Cerchio(3), new Rettangolo(4, 5) };

for (Forma f : forme) {
    f.stampa();   /* chiama automaticamente la versione giusta */
}
```

Ora aggiungi `Triangolo`:

```java
class Triangolo extends Forma {
    double base, altezza;

    Triangolo(double base, double altezza) {
        this.base = base;
        this.altezza = altezza;
    }

    double area() {
        return base * altezza / 2;
    }

    void stampa() {
        System.out.println("Triangolo, area = " + area());
    }
}
```

Questo è tutto. **Nessuna riga del codice esistente è stata toccata** — non `Forma`, non `Cerchio`, non `Rettangolo`, non il ciclo che li stampa. Hai solo aggiunto codice nuovo.

::: {.callout-tip}
## Open/Closed Principle
Questo è uno dei principi SOLID che avete già visto: una classe dovrebbe essere **aperta all'estensione** (puoi aggiungere `Triangolo`) ma **chiusa alla modifica** (non devi toccare `Forma`, `Cerchio`, `Rettangolo` per farlo). La versione C con lo switch viola sistematicamente questo principio: ogni estensione richiede una modifica.
:::

---

## Blocco 3: Cosa hai già fatto senza saperlo

Confronta cosa hai costruito a mano nell'ADT con puntatore opaco e cosa il linguaggio Java ti dà con una parola chiave:

| Cosa serve fare | In C (ADT opaco) | In Java |
|---|---|---|
| Nascondere lo stato interno | `struct Stack` reale scritta solo nel `.c`, invisibile a chi include `stack.h` | campi dichiarati `private` |
| Esporre solo un'interfaccia pubblica | prototipi delle funzioni nell'header `.h` | metodi dichiarati `public` |
| Costruire un'istanza | `stack_create(...)` — funzione scritta a mano che alloca e inizializza | `new Stack(...)` — un costruttore, con sintassi dedicata |
| Distruggere un'istanza | `stack_destroy(...)` — funzione scritta a mano che chiama `free` | non serve — il garbage collector lo fa da solo |
| Comportamento diverso per tipo diverso | `switch` sul campo `tipo`, scritto e mantenuto a mano in ogni funzione | override di un metodo — il linguaggio sceglie la versione giusta a runtime |

Non hai imparato un concetto nuovo in questo modulo. Hai imparato **come si chiama** un concetto che hai già costruito con le tue mani, e hai visto che un linguaggio progettato per l'OOP te lo dà con una sintassi dedicata invece di dover organizzare tu stesso file `.h`/`.c` e scrivere `switch` ovunque.

---

## Blocco 4: Cosa il linguaggio ti dà gratis (e cosa no)

Vale la pena essere precisi su cosa cambia davvero e cosa no:

**Gestione della memoria**
In C hai dovuto ricordarti tu di chiamare `free`, gestire dangling pointer e memory leak. In Java il garbage collector libera automaticamente la memoria di un oggetto quando non è più raggiungibile — la classe di errori legata a `malloc`/`free` scompare, non perché tu sia diventato più bravo, ma perché il linguaggio se ne occupa al posto tuo.

**Incapsulamento**
In C l'opaque pointer è una **disciplina che ti sei imposto tu**: nulla ti impedisce tecnicamente di includere direttamente il file `.c` e violare il nascondimento. In Java `private` è imposto dal **compilatore**: se provi ad accedere a un campo privato da fuori la classe, il programma non compila.

**Scelta del comportamento giusto**
In C hai scritto tu, a mano, ogni `switch` che sceglie il comportamento in base al tipo. In Java questa scelta avviene automaticamente a runtime quando chiami un metodo su un oggetto — si chiama **dispatch dinamico** ed è il meccanismo dietro il polimorfismo.

::: {.callout-note}
## Se in futuro vedrai i puntatori a funzione
Esiste anche in C un modo per automatizzare la scelta del comportamento senza scrivere `switch` a mano — usando i puntatori a funzione, che magari incontrerai in un altro contesto quest'anno. Se e quando ci arrivi, quel meccanismo è concettualmente molto vicino a come Java implementa il dispatch dinamico "sotto il cofano". Non è necessario conoscerlo per capire questo modulo — è solo un collegamento in più, se capita.
:::

---

## Schema mentale

```
Devo gestire comportamenti diversi per tipi diversi?
    → in C: campo tag + switch, ripetuto in ogni funzione
    → in Java: classe astratta + override, il linguaggio sceglie da solo

Devo aggiungere un nuovo tipo?
    → in C: tocchi l'enum, la union, e ogni switch esistente
    → in Java: scrivi solo la nuova classe, zero modifiche al resto

Devo nascondere lo stato interno?
    → in C: disciplina manuale con .h/.c (opaque pointer)
    → in Java: private, imposto dal compilatore

Devo gestire la memoria?
    → in C: malloc/free a mano, rischio di leak e dangling pointer
    → in Java: garbage collector automatico
```

---

## Esercizi

### Esercizio 1: Estendi la versione C

Parti dal codice del Blocco 1 (con `CERCHIO` e `RETTANGOLO`, senza `TRIANGOLO`). Aggiungi tu il tipo `QUADRATO`. Conta quante righe hai dovuto modificare o aggiungere, e in quante funzioni diverse.

::: {.callout-tip collapse="true"}
## Soluzione
```c
typedef enum { CERCHIO, RETTANGOLO, QUADRATO } TipoForma;

typedef struct {
    TipoForma tipo;
    union {
        struct { double raggio; } cerchio;
        struct { double base, altezza; } rettangolo;
        struct { double lato; } quadrato;
    } dati;
} Forma;

double area(Forma *f) {
    switch (f->tipo) {
        case CERCHIO:
            return 3.14159 * f->dati.cerchio.raggio * f->dati.cerchio.raggio;
        case RETTANGOLO:
            return f->dati.rettangolo.base * f->dati.rettangolo.altezza;
        case QUADRATO:
            return f->dati.quadrato.lato * f->dati.quadrato.lato;
    }
    return 0;
}

void stampa(Forma *f) {
    switch (f->tipo) {
        case CERCHIO:
            printf("Cerchio, area = %.2f\n", area(f));
            break;
        case RETTANGOLO:
            printf("Rettangolo, area = %.2f\n", area(f));
            break;
        case QUADRATO:
            printf("Quadrato, area = %.2f\n", area(f));
            break;
    }
}
```
Punti toccati: l'enum, la union, **due** switch (`area` e `stampa`). Se ci fosse anche una terza funzione (es. `perimetro`), andrebbe modificata anche quella.
:::

### Esercizio 2: Estendi la versione Java

Parti dal codice del Blocco 2 (con `Cerchio` e `Rettangolo`, senza `Triangolo`). Aggiungi tu la classe `Quadrato`. Conta quante righe di codice **già esistente** hai dovuto modificare.

::: {.callout-tip collapse="true"}
## Soluzione
```java
class Quadrato extends Forma {
    double lato;

    Quadrato(double lato) {
        this.lato = lato;
    }

    double area() {
        return lato * lato;
    }

    void stampa() {
        System.out.println("Quadrato, area = " + area());
    }
}
```
Righe di codice esistente modificate: **zero**. Hai solo aggiunto una classe nuova. Confronta questo numero con quello dell'Esercizio 1.
:::

### Esercizio 3: Discussione guidata

Immagina che il tuo programma debba calcolare, oltre all'area, anche il **perimetro** di ogni forma. In entrambe le versioni (C e Java) aggiungi questa nuova operazione a tutti i tipi già esistenti (cerchio, rettangolo, quadrato). In quale versione ti senti più sicuro di non aver dimenticato un caso? Perché?

::: {.callout-tip collapse="true"}
## Traccia di risposta
In C, aggiungere `perimetro()` significa scrivere una nuova funzione con un nuovo `switch` che deve elencare di nuovo tutti i tipi esistenti — è facile dimenticarne uno, e il compilatore non sempre te lo segnala (dipende dai flag di compilazione usati). In Java, aggiungere `perimetro()` significa aggiungere un metodo astratto in `Forma`: se una sottoclasse non lo implementa, **il codice non compila affatto** — l'errore emerge subito, non a runtime. Questa è una differenza reale a favore di Java in questo scenario, non solo una questione di stile.
:::