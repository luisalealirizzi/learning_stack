# Big Data e NoSQL

Il modello relazionale ha dominato la gestione dei dati per decenni, ed è ancora oggi la scelta giusta per la maggior parte delle applicazioni. Ma a partire dagli anni 2000, l'esplosione di internet, dei social network e dei sistemi distribuiti ha generato tipi di dati e volumi che il modello relazionale fatica a gestire. Da questa pressione nasce il movimento **NoSQL**.

---

## I limiti del modello relazionale su larga scala

Un database relazionale funziona molto bene quando:

- i dati sono strutturati e omogenei
- lo schema cambia raramente
- le operazioni richiedono coerenza forte (ACID)
- il volume è gestibile da un singolo server

Quando una di queste condizioni viene meno, emergono i problemi.

**Scalabilità verticale vs orizzontale**: un database relazionale si scala verticalmente, cioè aggiungendo più potenza allo stesso server (CPU più veloce, più RAM, dischi più veloci). Questo ha un limite fisico ed economico. I sistemi web moderni preferiscono la scalabilità orizzontale: aggiungere più server. Il modello relazionale, con le sue garanzie di coerenza e i JOIN distribuiti, si adatta male a questo approccio.

**Schema rigido**: in un database relazionale ogni tabella ha uno schema fisso. Aggiungere una colonna richiede un `ALTER TABLE` che può bloccare la tabella per minuti su sistemi con milioni di righe. In contesti in cui il formato dei dati cambia continuamente (profili utente, configurazioni, log), questo rigidità diventa un problema.

**Dati eterogenei**: non tutti i dati si modellano bene come tabelle. Un post di un social network può avere zero commenti o mille, un'immagine o dieci video. Forzare questa variabilità in uno schema relazionale rigido produce tabelle sparse, piene di NULL, o gerarchie di tabelle complesse.

---

## Cos'è NoSQL

**NoSQL** non significa "senza SQL": significa **Not Only SQL**. È una famiglia eterogenea di database nati per rispondere ai limiti descritti sopra. Non condividono un modello unico, ma alcune caratteristiche comuni:

- schema flessibile o assente
- progettati per la scalabilità orizzontale
- coerenza spesso rilassata a favore della disponibilità e delle prestazioni
- ottimizzati per tipi specifici di dati o di accesso

Esistono quattro famiglie principali di database NoSQL.

---

## Le quattro famiglie NoSQL

### Documento (Document store)

I dati sono organizzati come **documenti**, tipicamente in formato JSON o BSON. Ogni documento è un'unità autonoma che può contenere strutture annidate, array e campi variabili. Non esiste uno schema fisso: due documenti nella stessa collezione possono avere campi diversi.

```json
{
  "id": "u001",
  "nome": "Luca Rossi",
  "email": "luca@example.com",
  "interessi": ["fotografia", "musica", "viaggi"],
  "indirizzo": {
    "città": "Milano",
    "cap": "20100"
  }
}
```

Esempi di DBMS: **MongoDB**, CouchDB.

Usato per: profili utente, cataloghi di prodotti, contenuti di CMS, configurazioni applicative.

### Chiave-valore (Key-value store)

I dati sono coppie **chiave: valore**. La struttura è minima: si accede a un valore tramite la sua chiave, senza possibilità di query complesse sul contenuto. Sono i più veloci in lettura e scrittura.

```
"sessione:abc123" → { "userId": 42, "scadenza": "2025-12-01" }
"cache:homepage"  → "<html>...</html>"
```

Esempi di DBMS: **Redis**, DynamoDB.

Usato per: cache, sessioni utente, code di messaggi, contatori in tempo reale.

### Colonnare (Column store)

I dati sono organizzati per **colonne** invece che per righe. In un database relazionale, una riga è l'unità di memorizzazione; in un colonnare, lo è la colonna. Questo permette di leggere solo le colonne necessarie per una query, senza caricare l'intera riga, ed è molto efficiente per aggregazioni su grandi volumi.

Esempi di DBMS: **Apache Cassandra**, HBase, Google Bigtable.

Usato per: analytics, serie storiche, sistemi di log, dati IoT.

### A grafo (Graph database)

I dati sono modellati come **nodi** (entità) e **archi** (relazioni tra entità). Le relazioni sono cittadini di prima classe del modello, non semplici chiavi esterne. Sono ottimizzati per navigare reti di connessioni complesse.

Esempi di DBMS: **Neo4j**, Amazon Neptune.

Usato per: social network (amici di amici), motori di raccomandazione, reti di frodi, knowledge graph.

---

## Il teorema CAP

Per capire i compromessi dei sistemi distribuiti, inclusi i NoSQL, è utile conoscere il **teorema CAP** (Consistency, Availability, Partition tolerance), formulato da Eric Brewer nel 2000.

Il teorema afferma che un sistema distribuito non può garantire simultaneamente tutte e tre queste proprietà:

**Coerenza (Consistency)**: ogni lettura restituisce il dato più aggiornato, o un errore.

**Disponibilità (Availability)**: ogni richiesta riceve sempre una risposta, anche se potrebbe non essere il dato più aggiornato.

**Tolleranza alle partizioni (Partition tolerance)**: il sistema continua a funzionare anche se alcuni nodi della rete non riescono a comunicare tra loro.

In un sistema distribuito reale, le partizioni di rete sono inevitabili. Quindi la scelta è tra coerenza e disponibilità.

| Sistema | Sceglie | Rinuncia |
|---|---|---|
| Database relazionale tradizionale | Coerenza + Disponibilità | Tolleranza alle partizioni (non è distribuito) |
| MongoDB (configurazione tipica) | Disponibilità + Tolleranza | Coerenza forte |
| Cassandra | Disponibilità + Tolleranza | Coerenza forte |
| HBase | Coerenza + Tolleranza | Disponibilità |

I database NoSQL nascono spesso accettando coerenza eventuale (*eventual consistency*): i dati si propagano tra i nodi nel tempo, e per un breve periodo diversi nodi potrebbero restituire valori diversi. Per molte applicazioni (un like su un post, una visualizzazione di un video) questo è accettabile. Per un conto bancario, no.

::: {.callout-note}
## ACID vs BASE
I database relazionali seguono il modello ACID (Atomicità, Coerenza, Isolamento, Durabilità). Molti NoSQL seguono invece il modello **BASE**: **B**asically **A**vailable (sempre disponibile), **S**oft state (lo stato può cambiare nel tempo), **E**ventually consistent (prima o poi i dati convergono). BASE non è sbagliato: è una scelta diversa, adatta a contesti diversi.
:::

---

## Confronto: relazionale vs NoSQL

| | Relazionale | NoSQL |
|---|---|---|
| Struttura dei dati | tabelle con schema fisso | documenti, chiavi-valore, colonne, grafi |
| Schema | rigido, definito prima | flessibile, può cambiare nel tempo |
| Linguaggio | SQL standard | API specifiche per ogni DBMS |
| Coerenza | forte (ACID) | spesso eventuale (BASE) |
| Scalabilità | verticale | orizzontale |
| Relazioni tra dati | JOIN esplicite | incorporate nei documenti o gestite dall'applicazione |
| Ideale per | dati strutturati, transazioni, coerenza critica | grandi volumi, dati variabili, alta disponibilità |

---

## Quando usare cosa

Non esiste una scelta universalmente migliore. La domanda giusta è: quali sono i requisiti del sistema?

**Usa un database relazionale quando:**
- i dati hanno una struttura stabile e ben definita
- le operazioni richiedono transazioni con coerenza forte
- le relazioni tra entità sono complesse e richiedono JOIN
- il volume è gestibile da un singolo server o da un cluster piccolo

**Usa un database NoSQL quando:**
- i dati sono eterogenei o la struttura cambia spesso
- il volume è molto grande e richiede distribuzione su più server
- la disponibilità è più importante della coerenza assoluta
- si tratta di un tipo di dato specifico (grafo, serie temporale, cache)

In molti sistemi reali si usano entrambi: un database relazionale per i dati critici (ordini, pagamenti, utenti) e un NoSQL per i dati ad alto volume o alta variabilità (log, sessioni, raccomandazioni).

---

## Schema mentale

**Cosa significa NoSQL?**
Not Only SQL: non "senza SQL" ma "non solo SQL". È una famiglia di database con modelli diversi da quello relazionale, nati per rispondere a esigenze di scalabilità, flessibilità e volume che il modello relazionale fatica a soddisfare.

**Quali sono le quattro famiglie NoSQL e quando si usa ciascuna?**
Documento (dati variabili, profili, contenuti), chiave-valore (cache, sessioni, alta velocità), colonnare (analytics, serie storiche, grandi aggregazioni), grafo (reti di connessioni, raccomandazioni, social network).

**Cosa dice il teorema CAP?**
Che un sistema distribuito non può garantire simultaneamente coerenza, disponibilità e tolleranza alle partizioni. Deve scegliere due delle tre. I NoSQL scelgono quasi sempre disponibilità e tolleranza, rinunciando alla coerenza forte.

**Qual è la differenza tra ACID e BASE?**
ACID garantisce coerenza forte a ogni transazione. BASE accetta che i dati siano eventualmente coerenti: per un periodo breve, nodi diversi potrebbero restituire valori diversi, ma alla fine convergeranno. BASE è adatto dove la disponibilità conta più della coerenza assoluta.

**Quando si usano relazionale e NoSQL insieme?**
Spesso nei sistemi reali: il relazionale gestisce i dati critici che richiedono coerenza (ordini, pagamenti), il NoSQL gestisce i dati ad alto volume o alta variabilità (log, sessioni, raccomandazioni).