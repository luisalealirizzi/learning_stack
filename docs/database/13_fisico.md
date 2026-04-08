# Progettazione fisica e prestazioni

Fino a questo punto abbiamo lavorato al livello logico: tabelle, chiavi, vincoli, query. La progettazione fisica è il livello successivo: le stesse tabelle, sullo stesso schema, possono comportarsi in modo radicalmente diverso a seconda di come vengono implementate fisicamente. Questo capitolo non insegna a ottimizzare sistemi in produzione, ma a capire perché le scelte fisiche esistono e cosa determinano.

---

## Due database identici, comportamenti diversi

A livello logico, due database possono essere identici: stesse tabelle, stesse chiavi, stessi vincoli. A livello fisico possono comportarsi in modo completamente diverso in termini di velocità, affidabilità e capacità di recupero dopo un errore.

Questo perché ogni database, nella sua progettazione fisica, incarna una **filosofia**: una serie di scelte su cosa privilegiare e cosa sacrificare.

Un esempio è **PostgreSQL**. Non nasce per essere il database più veloce: nasce con una scelta precisa, meglio perdere prestazioni che perdere dati. Per questo adotta soluzioni fisiche costose: scrive prima su un log e poi sui dati, applica controlli rigorosi sulle transazioni, mantiene meccanismi di recupero affidabili dopo i crash. Il risultato è un sistema spesso più lento in scrittura, ma molto più affidabile nel tempo.

**MySQL** fa scelte diverse a seconda del motore di storage usato. Il motore InnoDB garantisce ACID e supporto alle chiavi esterne. Il motore MyISAM, più vecchio, è più veloce in lettura ma non garantisce l'integrità referenziale.

Nessuna scelta è giusta in assoluto. Ogni scelta è coerente con un contesto. Un sistema bancario ha bisogni diversi da un sito di e-commerce, che ha bisogni diversi da un sistema di log in tempo reale.

---

## Come i dati sono organizzati su disco

Un DBMS non legge i dati riga per riga ogni volta che riceve una query. I dati sono organizzati su disco in **pagine** (blocchi di dimensione fissa, tipicamente 8 o 16 KB). Quando il DBMS cerca un dato, carica in memoria l'intera pagina che lo contiene.

Questo ha una conseguenza importante: accedere a dati vicini fisicamente sul disco è molto più veloce che accedere a dati sparsi. È uno dei motivi per cui la progettazione fisica incide sulle prestazioni anche quando lo schema logico è identico.

Quando una tabella viene interrogata frequentemente su una certa colonna, il DBMS deve scorrere tutte le pagine per trovare le righe che corrispondono alla condizione. Su tabelle con milioni di righe, questo diventa lento. La soluzione sono gli **indici**.

---

## Indici

Un **indice** è una struttura dati separata dalla tabella che permette al DBMS di trovare rapidamente le righe che soddisfano una condizione, senza scorrere tutta la tabella.

Funziona in modo simile all'indice analitico di un libro: invece di leggere tutto il libro per trovare un argomento, si consulta l'indice che indica la pagina esatta.

La struttura più usata è il **B-Tree** (albero bilanciato): una struttura ad albero in cui ogni nodo contiene valori della colonna indicizzata e puntatori alle pagine del disco dove si trovano le righe corrispondenti. La ricerca su un B-Tree richiede un numero di operazioni proporzionale al logaritmo del numero di righe, invece che al numero di righe stesso.

```sql
-- Creare un indice sulla colonna cognome della tabella studente
CREATE INDEX idx_cognome ON studente(cognome);

-- Creare un indice su più colonne (indice composto)
CREATE INDEX idx_classe_cognome ON studente(classe, cognome);

-- Eliminare un indice
DROP INDEX idx_cognome ON studente;
```

La chiave primaria ha sempre un indice creato automaticamente dal DBMS. Le chiavi esterne di solito lo hanno anch'esse, perché vengono controllate a ogni INSERT e DELETE.

### Indici: vantaggi e costi

Gli indici non sono gratuiti. Accelerano le letture (`SELECT`), ma rallentano le scritture (`INSERT`, `UPDATE`, `DELETE`), perché ogni modifica ai dati richiede di aggiornare anche l'indice.

| Operazione | Con indice | Senza indice |
|---|---|---|
| SELECT con WHERE sulla colonna indicizzata | molto più veloce | lenta su tabelle grandi |
| INSERT / UPDATE / DELETE | più lento | più veloce |
| Spazio su disco | maggiore | minore |

::: {.callout-tip}
## Quando creare un indice
Un indice è utile su colonne che compaiono spesso in clausole `WHERE`, `JOIN ON` o `ORDER BY`, e che hanno molti valori distinti. Non ha senso indicizzare una colonna con pochi valori possibili (come un campo booleano): il DBMS troverebbe comunque metà delle righe e l'indice non aiuterebbe.
:::

---

## Partizionamento

Il **partizionamento** è una tecnica che divide fisicamente una tabella grande in parti più piccole chiamate partizioni, mantenendo una vista logica unica. Chi fa una query vede sempre la stessa tabella, ma il DBMS lavora su una porzione ridotta dei dati.

Esempio: una tabella `ordini` con milioni di righe può essere partizionata per anno. Una query che filtra su `anno = 2024` accede solo alla partizione di quell'anno, invece di scorrere tutta la tabella.

```sql
-- Esempio di partizionamento per anno in MySQL
CREATE TABLE ordini (
    idOrdine INT NOT NULL,
    data     DATE NOT NULL,
    importo  DECIMAL(10,2)
)
PARTITION BY RANGE (YEAR(data)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION pFuturi VALUES LESS THAN MAXVALUE
);
```

Il partizionamento è utile su tabelle che crescono continuamente nel tempo (log, transazioni, eventi) e che vengono interrogate spesso su un intervallo temporale o geografico.

---

## Prestazioni e compromessi

Le scelte di progettazione fisica comportano sempre dei compromessi. Non esiste una configurazione ottimale in assoluto: esiste la configurazione migliore per un determinato carico di lavoro.

I tre compromessi principali sono:

**Velocità di lettura vs velocità di scrittura**
Più indici accelerano le SELECT ma rallentano INSERT e UPDATE. Un sistema con molte letture e poche scritture (un sito di notizie) si configura diversamente da un sistema con molte scritture e poche letture (un sistema di logging).

**Spazio su disco vs velocità**
Gli indici occupano spazio. Le copie ridondanti dei dati (a volte usate per velocizzare certe query) occupano ancora più spazio. In alcuni sistemi si accetta ridondanza controllata per guadagnare prestazioni.

**Consistenza vs disponibilità**
Nei sistemi distribuiti (più server che gestiscono lo stesso database) garantire che tutti i nodi abbiano sempre gli stessi dati richiede comunicazione e coordinamento, che richiedono tempo. Alcuni sistemi rilassano la consistenza per restare sempre disponibili. Questo è uno dei temi centrali dei database NoSQL, che vedremo nel prossimo capitolo.

::: {.callout-note}
## La progettazione fisica non si improvvisa
Le ottimizzazioni fisiche si applicano *dopo* che lo schema logico è corretto e normalizzato, non prima. Ottimizzare uno schema mal progettato è come lubrificare un motore rotto: si può fare, ma non risolve il problema reale.
:::

---

## Schema mentale

**Perché due database con lo stesso schema logico possono avere prestazioni diverse?**
Perché le scelte fisiche (struttura su disco, indici, motore di storage, configurazione delle transazioni) incidono direttamente sulla velocità e sull'affidabilità, indipendentemente dallo schema.

**Cos'è un indice e quando conviene crearlo?**
È una struttura dati separata che permette ricerche rapide su una colonna. Conviene sulle colonne usate frequentemente in WHERE, JOIN e ORDER BY, con molti valori distinti. Non conviene su colonne con pochi valori possibili o su tabelle con molte scritture.

**Qual è il costo di un indice?**
Rallenta le scritture (ogni INSERT, UPDATE, DELETE deve aggiornare anche l'indice) e occupa spazio su disco. Il guadagno in lettura deve essere valutato rispetto a questi costi.

**Cos'è il partizionamento e quando si usa?**
È la divisione fisica di una tabella grande in parti più piccole. Si usa su tabelle che crescono continuamente e che vengono interrogate spesso su un sottoinsieme dei dati (per esempio per anno o per area geografica).

**Perché la progettazione fisica si fa dopo quella logica?**
Perché ottimizzare uno schema mal progettato non risolve i problemi strutturali. Prima si progetta correttamente il modello logico, poi si ottimizza il livello fisico in base al carico di lavoro reale.