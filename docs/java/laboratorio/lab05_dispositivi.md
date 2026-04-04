# Lab 05 — Simulatore di dispositivi IoT

*Ereditarietà, `LocalDateTime`, `Scanner`, `Math`, enum, validazione input*

---

## Scenario

Stai configurando due dispositivi (`d1`, `d2`) di un piccolo impianto: un **sensore** (che misura) e/o un **attuatore** (che esegue un'azione). Alcuni dispositivi sono a batteria, altri a rete.

Il sistema deve:
- registrare l'ultimo ping (data/ora)
- aggiornare lo stato (`"ATTIVO"` / `"STANDBY"`)
- gestire misure/azioni e consumi batteria
- stimare l'autonomia residua in minuti
- stampare un riepilogo chiaro e segnalare avvisi

---

## Gerarchia delle classi

```
Dispositivo
├── Sensore
└── Attuatore
```

---

## Classe Dispositivo (superclasse)

**Attributi privati:**

| Tipo | Nome | Note |
|---|---|---|
| `String` | `id` | identificativo univoco |
| `String` | `nome` | |
| `LocalDateTime` | `installatoIl` | |
| `String` | `stato` | `"ATTIVO"` o `"STANDBY"` |
| `LocalDateTime` | `ultimoPing` | può essere `null` |
| `String` | `alimentazione` | `"RETE"` o `"BATTERIA"` |
| `double` | `batteria` | da 0 a 100%, solo se a batteria |

**Metodi pubblici:**
- `descrizione()` — stringa breve con le info principali
- `ping()` — aggiorna `ultimoPing = now()` e imposta `stato = "ATTIVO"`
- `minDaUltimoPing()` — minuti trascorsi dall'ultimo ping (se `null` → 0)
- `equals(Object o)` — `true` se stesso `id`
- getter/setter essenziali

```java
public class Dispositivo {

    private String id;
    private String nome;
    private LocalDateTime installatoIl;
    private String stato;
    private LocalDateTime ultimoPing;
    private String alimentazione;
    private double batteria;

    public Dispositivo(String id, String nome, LocalDateTime installatoIl,
                       String stato, String alimentazione, double batteria) {
        this.id = id;
        this.nome = nome;
        this.installatoIl = installatoIl;
        this.alimentazione = alimentazione;
        // TODO: validazione stato, batteria, installatoIl
    }

    public void ping() {
        // TODO: ultimoPing = LocalDateTime.now(), stato = "ATTIVO"
    }

    public long minDaUltimoPing() {
        // TODO: se ultimoPing == null → 0
        //       altrimenti ChronoUnit.MINUTES.between(ultimoPing, LocalDateTime.now())
    }

    @Override
    public boolean equals(Object o) {
        // TODO: true se stesso id
    }

    public String descrizione() {
        // TODO
    }

    // getter/setter...
}
```

---

## Classe Sensore (estende Dispositivo)

**Attributi extra:**
- `String tipoMisura` (es. `"TEMPERATURA"`)
- `String unita` (es. `"°C"`)
- `double ultimoValore`

**Metodi:**
- `misura(double nuovo)` — aggiorna `ultimoValore`, chiama `ping()`, se a batteria consuma 1%
- `descrizione()` — override, aggiunge info sulla misura

```java
public class Sensore extends Dispositivo {

    private String tipoMisura;
    private String unita;
    private double ultimoValore;

    public Sensore(String id, String nome, LocalDateTime installatoIl,
                   String stato, String alimentazione, double batteria,
                   String tipoMisura, String unita) {
        super(id, nome, installatoIl, stato, alimentazione, batteria);
        this.tipoMisura = tipoMisura;
        this.unita = unita.isEmpty() ? "(unità sconosciuta)" : unita;
    }

    public void misura(double nuovo) {
        // TODO:
        // - aggiorna ultimoValore
        // - chiama ping()
        // - se alimentazione = "BATTERIA": batteria -= 1
    }

    @Override
    public String descrizione() {
        // TODO: super.descrizione() + info misura
    }
}
```

---

## Classe Attuatore (estende Dispositivo)

**Attributi extra:**
- `String tipoAzione` (es. `"SIRENA"`, `"LUCE"`)
- `boolean statoAzione` (on/off)

**Metodi:**
- `esegui(String comando)` — se `"ON"`/`"OFF"` valido: cambia `statoAzione`, chiama `ping()`, se a batteria e `"ON"` consuma 5%; se comando non valido: avvisa e non cambia stato
- `descrizione()` — override, aggiunge info sull'azione

```java
public class Attuatore extends Dispositivo {

    private String tipoAzione;
    private boolean statoAzione;

    public Attuatore(String id, String nome, LocalDateTime installatoIl,
                     String stato, String alimentazione, double batteria,
                     String tipoAzione, String comandoIniziale) {
        super(id, nome, installatoIl, stato, alimentazione, batteria);
        this.tipoAzione = tipoAzione;
        esegui(comandoIniziale);
    }

    public void esegui(String comando) {
        // TODO:
        // - se comando.equalsIgnoreCase("ON") → statoAzione = true, ping()
        //   se a batteria: batteria -= 5
        // - se comando.equalsIgnoreCase("OFF") → statoAzione = false, ping()
        // - altrimenti: avvisa "comando non valido, stato invariato"
    }

    @Override
    public String descrizione() {
        // TODO: super.descrizione() + tipoAzione + statoAzione
    }
}
```

---

## Enum (facoltativo)

Un **enum** è un tipo con un insieme finito di costanti — più sicuro di una `String` perché il compilatore verifica i valori:

```java
enum Alimentazione { RETE, BATTERIA }
enum TipoMisura    { TEMPERATURA, UMIDITA, LUX, GAS }
enum TipoAzione    { LUCE, SIRENA, RELE }
```

Se non vuoi usarli, usa `String` e valida con `equalsIgnoreCase()`.

---

## Calcoli e logica

### Consumo batteria

| Dispositivo | Operazione | Consumo |
|---|---|---|
| Sensore | `misura()` | −1% |
| Attuatore | `esegui("ON")` | −5% |

### Stima autonomia residua

```
Sensore (1 misura/minuto):
    autonomiaMin = Math.floor(batteria / 1 * 60)
    es: 20% → Math.floor(20/1*60) = 1200 min

Attuatore (1 azione ON/minuto):
    autonomiaMin = Math.floor(batteria / 5 * 60)
    es: 60% → Math.floor(60/5*60) = 720 min
```

### Formattare i minuti in ore:min

```java
long ore = autonomiaMin / 60;
long min = autonomiaMin % 60;
System.out.println("Autonomia: " + ore + "h " + min + "min");
```

### Confronto autonomia

```java
// Quale dispositivo si esaurirà prima?
if (autonomia1 < autonomia2) {
    System.out.println(d1.getNome() + " si esaurirà prima.");
} else if (autonomia2 < autonomia1) {
    System.out.println(d2.getNome() + " si esaurirà prima.");
} else {
    System.out.println("Si esauriranno contemporaneamente.");
}
```

---

## Validazione input "soft" *(nessuna eccezione — solo avvisi)*

| Caso | Comportamento |
|---|---|
| `batteria` fuori da [0, 100] | clamp a 0 o 100 + avviso |
| stato diverso da `ATTIVO`/`STANDBY` | imposta `"STANDBY"` + avviso |
| `installatoIl` nel futuro | avviso, continua |
| `unita` vuota (sensore) | imposta `"(unità sconosciuta)"` |
| comando non valido (attuatore) | non cambia stato + avviso |

```java
// Esempio di validazione batteria
if (batteria < 0) {
    System.out.println("AVVISO: batteria < 0, impostata a 0.");
    batteria = 0;
} else if (batteria > 100) {
    System.out.println("AVVISO: batteria > 100, impostata a 100.");
    batteria = 100;
}

// Validazione data futura
if (installatoIl.isAfter(LocalDateTime.now())) {
    System.out.println("AVVISO: data di installazione futura, continuo.");
}
```

---

## `==` vs `equals()`

```java
Dispositivo a = new Sensore("X", "Temp1", ...);
Dispositivo b = new Attuatore("X", "Sirena1", ...);

System.out.println(a == b);        // false — referenze diverse
System.out.println(a.equals(b));   // true  — stesso id "X"
```

---

## Flusso del main (pseudocodice)

```
1.  Avvia Scanner

2.  Leggi dati d1:
    - id, nome, installatoIl (o now()), stato, alimentazione, batteria
    - tipo: "sensore" o "attuatore"
    - se sensore: tipoMisura, unita, valore da misurare
    - se attuatore: tipoAzione, comando iniziale

3.  Crea d1 (Sensore o Attuatore)

4.  Esegui operazione su d1:
    - se sensore: d1.misura(valore)
    - se attuatore: d1.esegui(comando)

5.  Ripeti passi 2-4 per d2

6.  Stampa descrizione() di d1 e d2

7.  Calcola e stampa autonomia residua (solo se a batteria)

8.  Confronta autonomie → stampa quale si esaurirà prima

9.  Mostra avvisi di validazione applicati

10. Chiudi Scanner
```

---

## Esempio di output

```
===== DISPOSITIVO 1 =====
Tipo:        Sensore
Nome:        Termostato Cucina
ID:          SENS-01
Alimentaz.:  BATTERIA
Stato:       ATTIVO
Installato:  2025-11-01T09:00
Ultimo ping: 2025-11-13T10:05:32
Misura:      TEMPERATURA = 22.5 °C
Batteria:    79.0%
Autonomia:   4740 min (79h 0min)

===== DISPOSITIVO 2 =====
Tipo:        Attuatore
Nome:        Sirena Ingresso
ID:          ATT-01
Alimentaz.:  BATTERIA
Stato:       ATTIVO
Installato:  2025-11-01T09:00
Ultimo ping: 2025-11-13T10:05:32
Azione:      SIRENA → ON
Batteria:    55.0%
Autonomia:   660 min (11h 0min)

>> SENS-01 si esaurirà dopo ATT-01.
>> ATT-01 si esaurirà prima!
```

---

## Estensioni facoltative *(sfida)*

**`toString()` personalizzato**
Ridefinisci `toString()` in tutte le classi per stampe più eleganti — poi usa `System.out.println(d1)` direttamente.

**Priorità intervento**
```java
// Se un dispositivo è critico (batteria < 10%), stampalo per primo
if (d1.getBatteria() < 10 && d2.getBatteria() >= 10) {
    // stampa d1 per primo
}
```

**Modalità risparmio**
```java
// Se sensore a batteria < 20%, avvisa di limitare le misure
if (alimentazione.equals("BATTERIA") && batteria < 20) {
    System.out.println("AVVISO: batteria bassa — limita le misure a 1 ogni 5 minuti.");
}
```

**Enum per lo stato**
```java
enum Stato { ATTIVO, STANDBY, GUASTO }
```