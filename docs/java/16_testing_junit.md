# Testing con JUnit

---

## Perché testare il codice?

Hai scritto una classe `Personaggio` con setter che controllano i valori. Come sai che funzionano davvero? Provi a mano ogni volta che modifichi qualcosa?

Il problema del testing manuale è che è:
- **lento** — devi ricordarti cosa testare ogni volta
- **inaffidabile** — dimentichi casi limite
- **non ripetibile** — non puoi rieseguirlo automaticamente dopo ogni modifica

I **test automatici** risolvono tutti questi problemi: li scrivi una volta, li esegui in un secondo ogni volta che modifichi il codice. Se qualcosa si rompe, lo scopri subito.

::: {.callout-note}
## Il principio di base
Un test automatico è un pezzo di codice che verifica che un altro pezzo di codice si comporti come previsto. Se il comportamento cambia inaspettatamente, il test fallisce e ti avvisa.
:::

---

## JUnit 5: il framework di testing

**JUnit** è il framework di testing più usato in Java. La versione attuale è JUnit 5. Permette di scrivere test in modo semplice e leggibile.

Per usarlo in un progetto Maven, aggiungi al `pom.xml`:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
</dependency>
```

Per un progetto semplice senza Maven, puoi scaricare il jar da [junit.org](https://junit.org).

---

## Struttura di una classe di test

Per convenzione, la classe di test ha lo stesso nome della classe testata con suffisso `Test`, e si trova nella cartella `src/test/java`:

```
src/
├── main/java/
│   └── Personaggio.java       ← classe da testare
└── test/java/
    └── PersonaggioTest.java   ← classe di test
```

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class PersonaggioTest {

    private Personaggio eroe;

    // Eseguito prima di OGNI test — ricrea l'oggetto pulito
    @BeforeEach
    void setUp() {
        eroe = new Personaggio("Arthas", 150, 30, 1);
    }

    // Eseguito dopo OGNI test — pulizia
    @AfterEach
    void tearDown() {
        // eventuali risorse da chiudere
    }

    // Eseguito una volta sola prima di tutti i test della classe
    @BeforeAll
    static void setUpClass() {
        System.out.println("Inizio test PersonaggioTest");
    }

    // Eseguito una volta sola dopo tutti i test
    @AfterAll
    static void tearDownClass() {
        System.out.println("Fine test PersonaggioTest");
    }

    @Test
    void testCreazione() {
        assertEquals("Arthas", eroe.getNome());
        assertEquals(150, eroe.getVitaMax());
        assertEquals(150, eroe.getVita());  // parte con vita piena
        assertEquals(30, eroe.getAttacco());
        assertEquals(1, eroe.getLivello());
    }
}
```

---

## Le asserzioni principali

Il cuore di ogni test è l'**asserzione** — una verifica che deve essere vera. Se è falsa, il test fallisce.

```java
// Uguaglianza
assertEquals(150, eroe.getVita());          // expected, actual
assertEquals("Arthas", eroe.getNome());

// Uguaglianza con delta (per i double)
assertEquals(3.14, Math.PI, 0.01);          // tolleranza di 0.01

// Disuguaglianza
assertNotEquals(0, eroe.getVita());

// Booleani
assertTrue(eroe.getVita() > 0);
assertFalse(eroe.isMorto());

// Null
assertNull(eroe.getArma());          // se Personaggio ha getArma()
assertNotNull(eroe.getNome());

// Stesso oggetto in memoria
assertSame(eroe, eroe);
assertNotSame(eroe, new Personaggio("Arthas", 150, 30, 1));

// Verifica che un'eccezione venga lanciata
assertThrows(IllegalArgumentException.class, () -> {
    eroe.setVita(-100);
});

// Tutti i test in un blocco (tutti eseguiti anche se uno fallisce)
assertAll("statistiche",
    () -> assertEquals("Arthas", eroe.getNome()),
    () -> assertEquals(150, eroe.getVitaMax()),
    () -> assertFalse(eroe.isMorto())
);
```

---

## Scrivere test significativi

Un buon test verifica **un comportamento specifico** — non tutto insieme. Il nome del metodo deve descrivere esattamente cosa si sta testando.

```java
class PersonaggioTest {

    private Personaggio eroe;

    @BeforeEach
    void setUp() {
        eroe = new Personaggio("Arthas", 150, 30, 1);
    }

    // --- Test costruttore ---

    @Test
    void nuovoPersonaggioParteConVitaPiena() {
        assertEquals(eroe.getVitaMax(), eroe.getVita());
    }

    @Test
    void vitaMaxNegativaVieneNormalizzata() {
        Personaggio p = new Personaggio("Test", -50, 10, 1);
        assertTrue(p.getVitaMax() > 0);
    }

    // --- Test subirDanno ---

    @Test
    void subirDannoRiduceVita() {
        eroe.subirDanno(30);
        assertEquals(120, eroe.getVita());
    }

    @Test
    void subirDannoMaggioreDellaVitaPortaAZero() {
        eroe.subirDanno(9999);
        assertEquals(0, eroe.getVita());
    }

    @Test
    void subirDannoZeroNonModificaVita() {
        int vitaPrima = eroe.getVita();
        eroe.subirDanno(0);
        assertEquals(vitaPrima, eroe.getVita());
    }

    @Test
    void dopoMortaIsMortoRestituisceTrue() {
        eroe.subirDanno(9999);
        assertTrue(eroe.isMorto());
    }

    // --- Test salaDiLivello ---

    @Test
    void salaDiLivelloAumentaIlLivello() {
        eroe.salaDiLivello();
        assertEquals(2, eroe.getLivello());
    }

    @Test
    void salaDiLivelloRipristinaVitaPiena() {
        eroe.subirDanno(50);
        eroe.salaDiLivello();
        assertEquals(eroe.getVitaMax(), eroe.getVita());
    }

    @Test
    void salaDiLivelloAumentaAttacco() {
        int attaccoPrima = eroe.getAttacco();
        eroe.salaDiLivello();
        assertTrue(eroe.getAttacco() > attaccoPrima);
    }

    // --- Test setter ---

    @Test
    void setVitaConValoreNegativoPortaAZero() {
        eroe.setVita(-10);
        assertEquals(0, eroe.getVita());
    }

    @Test
    void setVitaSopraVitaMaxVieneClampata() {
        eroe.setVita(9999);
        assertEquals(eroe.getVitaMax(), eroe.getVita());
    }

    @Test
    void setAttaccoNegativoNonModificaAttacco() {
        int attaccoPrima = eroe.getAttacco();
        eroe.setAttacco(-5);
        assertEquals(attaccoPrima, eroe.getAttacco());
    }
}
```

---

## Test delle eccezioni

Verificare che il codice lanci le eccezioni giuste è fondamentale:

```java
@Test
void setVitaConValoreNegativoLanciaEccezione() {
    // Se il setter lancia IllegalArgumentException con valori negativi:
    IllegalArgumentException ex = assertThrows(
        IllegalArgumentException.class,
        () -> eroe.setVita(-100)
    );

    // Puoi anche verificare il messaggio dell'eccezione
    assertTrue(ex.getMessage().contains("negativa") ||
               ex.getMessage().contains("negativo"));
}

@Test
void costruttoreConNomeNulloLanciaEccezione() {
    assertThrows(IllegalArgumentException.class, () ->
        new Personaggio(null, 150, 30, 1)
    );
}
```

---

## Test parametrizzati

Quando vuoi testare lo stesso comportamento con valori diversi, i **test parametrizzati** evitano la duplicazione:

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

class PersonaggioTest {

    private Personaggio eroe;

    @BeforeEach
    void setUp() {
        eroe = new Personaggio("Arthas", 150, 30, 1);
    }

    // Test con valori singoli
    @ParameterizedTest
    @ValueSource(ints = {0, -1, -100, Integer.MIN_VALUE})
    void setVitaNegativaPortaSempreAZero(int vitaNegativa) {
        eroe.setVita(vitaNegativa);
        assertEquals(0, eroe.getVita());
    }

    // Test con coppie di valori (danno, vitaAttesa)
    @ParameterizedTest
    @CsvSource({
        "0, 150",    // nessun danno → vita rimane 150
        "30, 120",   // danno 30 → vita 120
        "150, 0",    // danno = vita → vita 0
        "200, 0"     // danno > vita → vita 0
    })
    void subirDannoCalcolaVitaCorretta(int danno, int vitaAttesa) {
        eroe.subirDanno(danno);
        assertEquals(vitaAttesa, eroe.getVita());
    }
}
```

---

## Piano di test

Prima di scrivere i test, conviene pianificarli. Un **piano di test** elenca cosa verificare per ogni metodo, includendo i casi normali e i casi limite.

Per la classe `Personaggio`:

| Metodo | Caso da testare | Risultato atteso |
|---|---|---|
| Costruttore | Valori validi | Oggetto creato con vita = vitaMax |
| Costruttore | `vitaMax` negativo | `vitaMax` normalizzato a valore positivo |
| Costruttore | Nome null/vuoto | Eccezione o nome di default |
| `subirDanno()` | Danno < vita | Vita diminuisce esattamente di `danno` |
| `subirDanno()` | Danno = vita | Vita = 0, `isMorto()` = true |
| `subirDanno()` | Danno > vita | Vita = 0, non negativa |
| `subirDanno()` | Danno = 0 | Vita invariata |
| `setVita()` | Valore valido | Vita aggiornata |
| `setVita()` | Valore negativo | Vita = 0 |
| `setVita()` | Valore > vitaMax | Vita = vitaMax |
| `salaDiLivello()` | Prima chiamata | Livello = 2, vita piena |
| `isMorto()` | Vita > 0 | false |
| `isMorto()` | Vita = 0 | true |

::: {.callout-tip}
## I casi limite sono i più importanti
I test più utili sono quelli sui **casi limite** (edge cases): valori a zero, negativi, null, stringhe vuote, liste vuote. Questi sono i valori con cui il codice si comporta spesso in modo inatteso.
:::

---

## Test-Driven Development (TDD)

Il **TDD** è un approccio in cui scrivi i test **prima** del codice di produzione:

1. **Red** — scrivi un test che fallisce (il codice non esiste ancora)
2. **Green** — scrivi il minimo codice necessario per far passare il test
3. **Refactor** — migliora il codice senza rompere i test

```java
// 1. RED — scrivo il test prima di avere il metodo
@Test
void puntiEsperienzaBasatiSuLivello() {
    Personaggio p = new Personaggio("Arthas", 150, 30, 3);
    assertEquals(300, p.getPuntiEsperienza());  // il metodo non esiste ancora!
}

// 2. GREEN — aggiungo il metodo minimo per far passare il test
public int getPuntiEsperienza() {
    return livello * 100;
}

// 3. REFACTOR — miglioro se necessario
public int getPuntiEsperienza() {
    return livello * livello * 50;  // formula più interessante
}
// il test mi dice subito se ho rotto qualcosa
```

::: {.callout-note}
## Vantaggi del TDD
- Forza a pensare all'interfaccia prima dell'implementazione
- Garantisce che ogni riga di codice sia coperta da almeno un test
- Il refactoring diventa sicuro — i test ti dicono se hai rotto qualcosa
:::

---

## Esempio completo: test della classe Arma

```java
import org.junit.jupiter.api.*;
import org.junit.jupiter.params.*;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.api.Assertions.*;

class ArmaTest {

    private Arma spada;

    @BeforeEach
    void setUp() {
        spada = new Arma("Spada del Caos", 50);
    }

    @Test
    void nuovaArmaParte conDurabilita100() {
        assertEquals(100, spada.getDurabilita());
    }

    @Test
    void usaRiduceDurabilita() {
        spada.usa();
        assertEquals(99, spada.getDurabilita());
    }

    @Test
    void dannoCalaCon Durabilita() {
        int dannoIniziale = spada.getDanno();
        // Simula usura (99 utilizzi → durabilità 1)
        for (int i = 0; i < 99; i++) spada.usa();
        assertTrue(spada.getDanno() < dannoIniziale);
    }

    @Test
    void armaRottaNonInfliggeDanno() {
        for (int i = 0; i < 100; i++) spada.usa();
        assertTrue(spada.isRotta());
        // Con durabilità 0, il danno dovrebbe essere 0
        assertEquals(0, spada.getDanno());
    }

    @Test
    void riparaRipristinaDurabilita() {
        spada.usa();
        spada.usa();
        spada.ripara();
        assertEquals(100, spada.getDurabilita());
    }

    @ParameterizedTest
    @ValueSource(ints = {0, -1, -100})
    void dannoBas eNonPositivoVieneNormalizzato(int dannoInvalido) {
        Arma arma = new Arma("Test", dannoInvalido);
        assertTrue(arma.getDanno() > 0);
    }
}
```

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- I **test automatici** verificano il comportamento del codice in modo rapido e ripetibile.
- **JUnit 5** è il framework standard per il testing in Java.
- Le **asserzioni** (`assertEquals`, `assertTrue`, `assertThrows`...) verificano i risultati attesi.
- `@BeforeEach` ricrea l'oggetto pulito prima di ogni test — fondamentale per l'isolamento.
- I **casi limite** (zero, negativo, null, stringa vuota) sono i test più importanti.
- I **test parametrizzati** evitano la duplicazione quando testi lo stesso comportamento con valori diversi.
- Un **piano di test** elenca sistematicamente cosa verificare prima di scrivere i test.
- Il **TDD** inverte l'ordine: test prima, codice dopo — porta a design più puliti.
:::