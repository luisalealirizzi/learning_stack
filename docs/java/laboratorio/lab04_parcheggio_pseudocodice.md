# Lab 04 — Mini-simulatore di parcheggio

*Progettazione in pseudocodice: ereditarietà, `LocalDateTime`, `Scanner`, `Math`*

---

## Scenario

Un parcheggio cittadino registra l'ingresso e l'uscita dei veicoli, calcola il costo della sosta e applica regole diverse per veicoli speciali.

Questo laboratorio si svolge in due fasi:
1. **Oggi** — progettare lo pseudocodice in piccoli gruppi
2. **Prossimo laboratorio** — tradurre lo pseudocodice in Java

---

## Gerarchia delle classi

```
Veicolo
├── Auto
├── AutoElettrica
└── [a scelta: Moto / Camper / altro]
```

### Veicolo (superclasse)

| Tipo | Nome | Note |
|---|---|---|
| `private String` | `targa` | |
| `private String` | `marca` | |
| `private String` | `modello` | |
| `private double` | `pesoKg` | |

Metodi pubblici: `descrizione()`, getter necessari.
Metodi protetti: metodi di servizio che le sottoclassi possono usare.

### Auto (estende Veicolo)

Attributo aggiuntivo: `String classeAmbientale` (es. Euro 5, Euro 6).

### AutoElettrica (estende Veicolo)

| Tipo | Nome |
|---|---|
| `double` | `capacitaBatteriaKWh` |
| `boolean` | `ricaricaRichiesta` |

Override del metodo che calcola il costo — applica sconto o costo di ricarica.

### Terzo veicolo (a scelta del gruppo)

Scegli tra **Moto**, **Camper** o altro. Decidi gli attributi extra e se ha una tariffa speciale.

::: {.callout-note}
## Polimorfismo
Usa un riferimento di tipo `Veicolo` che può puntare a `Auto` o `AutoElettrica`:

```java
Veicolo v = new AutoElettrica(...);
v.calcolaCosto(...);  // eseguito nella versione di AutoElettrica
```

Puoi mettere il calcolo base del costo in `Veicolo` e ridefinirlo in `AutoElettrica` per applicare sconti o costi di ricarica.
:::

---

## Flusso del main (pseudocodice)

Scrivi i passi numerati che il tuo `main` dovrà seguire:

```
1. Avvia Scanner

2. Chiedi tipo veicolo ("auto" / "elettrica" / ...)

3. Chiedi dati comuni:
   - targa
   - marca
   - modello
   - peso (kg)

4. Se elettrica:
   - chiedi capacità batteria (kWh)
   - chiedi se vuole ricaricare (s/n)

5. Se [terzo tipo]:
   - chiedi [attributi specifici]

6. Crea l'oggetto con il costruttore appropriato

7. Chiedi tempo di sosta:
   - opzione A: inserire i minuti previsti
   - opzione B: inserire ora di uscita (anno/mese/giorno/ora/minuti)

8. Calcola ingresso = LocalDateTime.now()
   Calcola uscita = ingresso.plusMinutes(minuti)
              oppure = LocalDateTime inserito dall'utente

9. Chiedi tariffa base (€/ora)
   [facoltativo] chiedi tetto massimo giornaliero

10. Calcola il costo (vedi formule sotto)

11. Stampa il riepilogo

12. Chiudi Scanner
```

---

## Calcolo della tariffa

### Formula base

```
durataMin      = minuti tra ingresso e uscita
oreArrotondate = ceil(durataMin / 60.0)       ← Math.ceil
costoBase      = oreArrotondate * tariffaOraria
```

### Regole da applicare in ordine

**1. Minimo 30 minuti**
```
se durataMin < 30 → usa 30 minuti come base
```

**2. Sconto sera/notte**
```
se ora ingresso tra 20:00 e 07:00 → costo = costo * 0.8
```

**3. Auto elettrica**
```
se ricaricaRichiesta = true:
    opzione A → aggiungi costoRicarica = kWhRichiesti * €/kWh
    opzione B → sconto 20% sulla tariffa oraria
                ma minimo 1€ → Math.max(costoScontato, 1.0)
```

**4. Tetto massimo giornaliero**
```
costo = min(costo, tettoGiornaliero)    ← Math.min
```

**5. Arrotondamento a 2 decimali**
```
costoFinale = Math.round(costo * 100) / 100.0
```

---

## Tempo con `LocalDateTime`

```java
// Ingresso = adesso
LocalDateTime ingresso = LocalDateTime.now();

// Uscita = ingresso + tempo sosta
LocalDateTime uscita = ingresso.plusHours(H).plusMinutes(M);

// Durata in minuti
long durataMin = ChronoUnit.MINUTES.between(ingresso, uscita);

// Ore arrotondate per eccesso
long oreArrotondate = (long) Math.ceil(durataMin / 60.0);
```

---

## Esempio di output

```
===== RIEPILOGO SOSTA =====
Veicolo:   AutoElettrica | AB123CD | Tesla Model 3
Peso:      1800 kg | Batteria: 75 kWh | Ricarica: sì

Ingresso:  2025-11-13 09:30
Uscita:    2025-11-13 12:15
Durata:    165 minuti → 3 ore (arrotondate)

Tariffa base:    1.80 €/ora
Costo base:      5.40 €
Sconto ricarica: -1.08 €  (20%)
Minimo applicato: no
TOTALE:          4.32 €
===========================
```

---

## Attività di gruppo

Ogni gruppo deve produrre:

**1. Disegno delle classi**
Per ogni classe: lista attributi (tipo e nome) e lista metodi (firma e breve descrizione).

**2. Flusso del main**
Passi numerati come nello schema sopra, personalizzati con le scelte del gruppo.

**3. Formule di costo**
Quali regole avete scelto di applicare e in quale ordine? Come gestite i casi limite (sosta brevissima, notte, elettrica senza ricarica)?

**4. Esempio di output**
Scrivete a mano un output realistico per un caso concreto — un'auto normale e una elettrica.

---

## Aggiunte facoltative *(sfida)*

**Tariffa progressiva**
```
prime 2 ore    → tariffaOraria
ore successive → tariffaOraria * 1.5
```

**Multa per sosta eccessiva**
```
se durataMin > limiteOre * 60 → aggiungi penale fissa
```

**Sconto weekend**
```
se ingresso è sabato o domenica → sconto 30%
```
```java
DayOfWeek giorno = ingresso.getDayOfWeek();
if (giorno == DayOfWeek.SATURDAY || giorno == DayOfWeek.SUNDAY) {
    costo *= 0.7;
}
```

**Previsione ricavi giornalieri**
```
dato N ingressi stimati e durata media casuale:
ricavoStimato = somma di N * calcolaCosto(durataRandom)
```
```java
double durataRandom = 30 + Math.random() * 330;  // tra 30 e 360 minuti
```

---

::: {.callout-important}
## Obiettivo di oggi
Non scrivete codice Java — scrivete pseudocodice chiaro e leggibile. Più dettagliato è lo pseudocodice, più veloce sarà la traduzione in Java nel prossimo laboratorio.
:::