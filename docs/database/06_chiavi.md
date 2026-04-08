# Chiavi e vincoli: rendere i dati affidabili

Nel capitolo precedente abbiamo visto come tradurre un modello ER in tabelle. Ma avere le tabelle giuste non basta: occorre che i dati al loro interno si comportino correttamente. Chiavi e vincoli sono i meccanismi con cui il DBMS lo garantisce.

---

## Un'idea prima della tecnica

Nel 1969, a pochi minuti dall'atterraggio della missione Apollo 11 sulla Luna, il computer di bordo inizia a segnalare una serie di allarmi. Un errore umano aveva attivato un radar che non doveva essere operativo in quella fase, e il carico di lavoro supera i limiti previsti.

Il sistema non va in crash. Rifiuta semplicemente le operazioni non compatibili e continua a eseguire solo ciò che è ammesso in quello stato. L'atterraggio riesce.

Il software era stato progettato da **Margaret Hamilton**, con un principio preciso:

> Non puoi impedire all'essere umano di sbagliare, ma puoi impedire al sistema di entrare in stati impossibili.

Un database ben progettato funziona allo stesso modo. Le chiavi servono a riconoscere i dati senza ambiguità. I vincoli servono a bloccare operazioni che violerebbero la coerenza. Non rendono il sistema più rigido: lo rendono affidabile.

---

## Le chiavi nel modello relazionale

Una **chiave** è un insieme di uno o più attributi che permette di identificare in modo univoco ogni riga di una tabella. Non tutte le chiavi sono uguali: esiste una gerarchia precisa.

### Superchiave

Una **superchiave** è qualsiasi insieme di attributi che identifica univocamente ogni riga. Può contenere attributi in più rispetto al necessario.

Esempio: nella tabella `STUDENTE(idStudente, nome, cognome)`, se `idStudente` è univoco, allora anche `(idStudente, nome)` è una superchiave. Ma contiene un attributo inutile: `nome` non serve per garantire l'univocità.

### Chiave candidata

Una **chiave candidata** è una superchiave *minimale*: identifica univocamente ogni riga e non contiene attributi ridondanti. Se si toglie un qualsiasi attributo, perde la proprietà di identificare in modo univoco.

Una tabella può avere più chiavi candidate. Esempio: nella tabella `STUDENTE(idStudente, matricola, nome, cognome)`, sia `idStudente` sia `matricola` potrebbero essere chiavi candidate, se entrambi sono univoci.

### Chiave primaria

La **chiave primaria** (PK, *Primary Key*) è la chiave candidata scelta dal progettista come identificatore ufficiale della tabella. Deve rispettare tre regole:

- il suo valore deve essere **unico** per ogni riga
- non può contenere valori **NULL**
- deve essere **stabile nel tempo**: una volta assegnato, il valore non dovrebbe cambiare

Per questo motivo si preferiscono chiavi artificiali generate dal sistema (`idStudente INT AUTO_INCREMENT`) rispetto a chiavi naturali come il nome o il codice fiscale, che potrebbero cambiare o richiedere correzioni.

### Chiave esterna

Una **chiave esterna** (FK, *Foreign Key*) è un attributo che fa riferimento alla chiave primaria di un'altra tabella. Serve a creare e mantenere i collegamenti tra tabelle.

Esempio:

```
STUDENTE(idStudente PK, nome, cognome)

ISCRIZIONE(idIscrizione PK, dataIscrizione, idStudente FK)
```

La colonna `idStudente` in `ISCRIZIONE` è una chiave esterna: ogni valore che appare lì deve corrispondere a un `idStudente` esistente in `STUDENTE`. Se si prova a inserire un'iscrizione con un `idStudente` che non esiste, il DBMS rifiuta l'operazione.

::: {.callout-note}
## Il lessico in sintesi
- **Superchiave**: identifica univocamente, ma può contenere attributi in più
- **Chiave candidata**: superchiave minimale, senza attributi ridondanti
- **Chiave primaria**: la chiave candidata scelta come identificatore ufficiale
- **Chiave esterna**: collega una tabella a un'altra tramite la PK di quest'ultima
:::

---

## I vincoli

Un **vincolo** è una regola che il DBMS applica automaticamente per garantire la correttezza e la coerenza dei dati. Se un'operazione viola un vincolo, il DBMS la rifiuta. Non corregge: impedisce.

Esistono due categorie principali.

### Vincoli di dominio

I vincoli di dominio controllano i valori che possono comparire in una singola colonna. I più comuni in MySQL/MariaDB:

| Vincolo | Effetto |
|---|---|
| `NOT NULL` | il campo non può essere lasciato vuoto |
| `UNIQUE` | i valori nella colonna non possono ripetersi |
| `CHECK` | il valore deve soddisfare una condizione |
| `DEFAULT` | se non specificato, il DBMS usa un valore predefinito |

Esempi:

```sql
eta INT CHECK (eta >= 0)
email VARCHAR(100) UNIQUE
stato VARCHAR(10) DEFAULT 'attivo'
voto DECIMAL(4,2) CHECK (voto >= 0 AND voto <= 10)
```

### Vincoli referenziali

I vincoli referenziali controllano la coerenza tra tabelle collegate tramite chiavi esterne. Garantiscono due cose:

- una FK deve sempre fare riferimento a una PK che esiste nella tabella collegata
- non possono esistere record "orfani": righe che fanno riferimento a qualcosa che non c'è più

Esempio: se si prova a cancellare uno studente che ha iscrizioni attive, il DBMS deve decidere cosa fare. Le opzioni principali sono:

- **RESTRICT** (default): rifiuta la cancellazione
- **CASCADE**: cancella automaticamente anche le iscrizioni collegate
- **SET NULL**: imposta la FK a NULL nelle righe collegate

La scelta dipende dalla logica del sistema: in una scuola, cancellare uno studente e perdere tutte le sue iscrizioni potrebbe essere accettabile; in un sistema bancario, non lo sarebbe mai.

::: {.callout-important}
## Vincoli e integrità referenziale
L'integrità referenziale è la proprietà che garantisce che tutti i riferimenti tra tabelle siano validi. È il DBMS a mantenerla, non il programma che usa il database. Questo significa che anche un'applicazione scritta male non può corrompere i riferimenti, a patto che i vincoli siano definiti correttamente nello schema.
:::

---

## Perché chiavi e vincoli non sono opzionali

È tecnicamente possibile creare tabelle senza chiavi primarie e senza vincoli. Il DBMS lo permette. Ma il risultato è un database fragile: nulla impedisce di inserire dati duplicati, riferimenti a record inesistenti o valori fuori range.

Chiavi e vincoli non descrivono *cosa* rappresentano i dati, ma *come devono comportarsi* all'interno del sistema. Definirli correttamente è parte integrante della progettazione, non un dettaglio da aggiungere dopo.

Un sistema affidabile non corregge l'errore. Lo rende impossibile.

---

## Schema mentale

**Qual è la differenza tra superchiave e chiave candidata?**
La superchiave può contenere attributi ridondanti. La chiave candidata è minimale: se si toglie un attributo, non identifica più univocamente.

**Perché la chiave primaria non può contenere NULL?**
Perché deve identificare ogni riga in modo certo. Un valore NULL significa "sconosciuto": non può essere usato come identificatore.

**Cosa fa un vincolo referenziale quando si cancella un record collegato?**
Dipende dalla configurazione: può rifiutare la cancellazione (RESTRICT), propagarla alle righe collegate (CASCADE), oppure impostare la FK a NULL (SET NULL).

**Qual è la differenza tra `UNIQUE` e chiave primaria?**
Entrambi garantiscono l'univocità. Ma la chiave primaria non può essere NULL e ce n'è una sola per tabella. `UNIQUE` può ammettere NULL (in alcuni DBMS) e una tabella può averne più di uno.

**Perché si preferiscono chiavi artificiali alle chiavi naturali?**
Le chiavi naturali (nome, codice fiscale) possono cambiare nel tempo o risultare duplicate in casi limite. Le chiavi artificiali generate dal sistema sono stabili, univoche e non dipendono dai dati reali.