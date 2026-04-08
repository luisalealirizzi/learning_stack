# Il DBMS: cos'è e cosa fa

Nel capitolo precedente abbiamo visto da dove vengono i database e perché esistono. In questo capitolo entriamo nel funzionamento del sistema che li gestisce: il **DBMS**.

---

## Database e DBMS: due cose diverse

È una distinzione che vale la pena fissare subito, perché si tende a confonderle.

Un **database** è un insieme organizzato di dati. È il contenuto: le tabelle, i record, le informazioni memorizzate.

Un **DBMS** (*DataBase Management System*) è il software che gestisce quel contenuto. È il sistema che permette di creare le strutture, inserire e modificare i dati, controllare chi può accedere a cosa, mantenere la coerenza delle informazioni ed eseguire le interrogazioni (query).

::: {.callout-note}
## Un'analogia utile
Il database è come il contenuto di una biblioteca: i libri, le schede, i documenti. Il DBMS è come il sistema di gestione della biblioteca: il catalogo, le regole di prestito, il personale che controlla gli accessi, il sistema antincendio che protegge il patrimonio.
:::

Senza un DBMS, i dati sarebbero solo file. Con un DBMS, diventano una memoria affidabile, condivisa e interrogabile.

MySQL, MariaDB, PostgreSQL, SQLite, Oracle sono tutti DBMS. Non sono database: sono i sistemi che gestiscono i database.

---

## Cosa fa concretamente un DBMS

Le funzioni di un DBMS si possono raggruppare in quattro aree principali.

### Gestione dei dati

Il DBMS permette di creare, leggere, modificare e cancellare i dati attraverso un linguaggio standard: SQL. Gestisce la struttura delle tabelle (schema) e il loro contenuto (istanza), e si occupa di tradurre le istruzioni dell'utente in operazioni fisiche sul disco.

### Controllo degli accessi

Non tutti gli utenti possono fare tutto. Il DBMS gestisce **utenti**, **ruoli** e **privilegi**.

| Privilegio | Significato |
|---|---|
| `SELECT` | leggere i dati |
| `INSERT` | inserire nuovi record |
| `UPDATE` | modificare record esistenti |
| `DELETE` | cancellare record |

Un ruolo è un insieme di privilegi che può essere assegnato a più utenti. In una scuola, ad esempio, il ruolo *docente* potrebbe includere solo `SELECT` sui voti dei propri studenti e `INSERT` per aggiungerne di nuovi.

### Garanzia dell'integrità

Il DBMS impone automaticamente le regole che i dati devono rispettare: un campo non può essere vuoto se è dichiarato obbligatorio, una chiave esterna deve riferirsi a un record che esiste, un voto non può essere al di fuori dell'intervallo ammesso. Se un'operazione viola una di queste regole, il DBMS la rifiuta.

### Backup e recovery

Un database contiene memoria ufficiale: voti, stipendi, anagrafi, cartelle cliniche. Se questa memoria viene persa o danneggiata, non è solo un problema tecnico, è un problema sociale e legale.

Il **backup** è una copia di sicurezza del database. Il **recovery** è l'insieme delle procedure che permettono di ripristinare i dati dopo un guasto, un errore o un incidente. Un DBMS ben configurato può recuperare i dati anche dopo un crash improvviso del sistema.

---

## Le transazioni

Una delle funzioni più importanti del DBMS è la gestione delle **transazioni**.

Una transazione è una sequenza di operazioni che il sistema considera come un'unica azione logica. Il DBMS non valuta le singole istruzioni una per una: valuta il risultato complessivo.

Una transazione può avere solo due esiti:

- **COMMIT**: tutte le operazioni vengono confermate e rese permanenti
- **ROLLBACK**: tutte le operazioni vengono annullate, come se non fossero mai avvenute

Non esiste uno stato intermedio. O tutto, o niente.

::: {.callout-tip}
## Perché le transazioni esistono
Nel mondo reale, molte operazioni hanno senso solo se complete. Un pagamento bancario richiede che il conto del mittente venga addebitato *e* quello del destinatario accreditato. Se solo una delle due operazioni va a buon fine, il risultato è sbagliato. La transazione garantisce che le due operazioni siano trattate come un blocco indivisibile.
:::

---

## ACID: le quattro proprietà di una transazione

Le transazioni di un DBMS affidabile devono rispettare quattro proprietà, riassunte nell'acronimo **ACID**.

**Atomicità (Atomicity)**
La transazione è indivisibile. O viene eseguita per intero, oppure non viene eseguita per niente. Non esistono "mezze operazioni".

**Coerenza (Consistency)**
Dopo ogni transazione, il database deve rispettare tutte le regole e i vincoli definiti nello schema. Non possono comparire dati in uno stato non valido.

**Isolamento (Isolation)**
Più transazioni possono essere eseguite contemporaneamente, ma non devono interferire tra loro. Ogni transazione deve comportarsi come se fosse l'unica in esecuzione, anche quando non lo è.

**Durabilità (Durability)**
Una volta confermata con un COMMIT, una transazione non può andare persa, nemmeno in caso di crash o spegnimento improvviso del sistema.

::: {.callout-important}
## ACID non è opzionale
Un sistema che non garantisce ACID non è un DBMS affidabile. Le proprietà ACID sono il motivo per cui i dati bancari, i voti scolastici e le cartelle cliniche possono essere gestiti digitalmente senza rischi di perdita o incoerenza sistematica.
:::

---

## Il controllo della concorrenza

In un DBMS reale, più utenti lavorano contemporaneamente sugli stessi dati. Questo crea il problema della **concorrenza**: due operazioni che accedono agli stessi dati nello stesso momento possono entrare in conflitto.

Il DBMS deve garantire che ogni utente abbia l'illusione di essere l'unico a usare il sistema, anche quando non è vero.

Un esempio classico: due persone prenotano l'ultimo posto disponibile su un volo nello stesso istante. Senza controllo della concorrenza, entrambe potrebbero ricevere la conferma. Il DBMS, attraverso meccanismi di blocco (*lock*) e gestione delle transazioni, garantisce che solo una delle due riesca a completare la prenotazione.

La concorrenza è anche il motivo per cui la proprietà di **isolamento** in ACID è così importante: non basta che ogni singola transazione sia corretta, bisogna che lo siano anche quando vengono eseguite in parallelo.

---

## Schema mentale

**Qual è la differenza tra database e DBMS?**
Il database è l'insieme dei dati. Il DBMS è il software che lo gestisce: crea le strutture, esegue le query, controlla gli accessi, garantisce l'integrità.

**Cosa significa che una transazione è atomica?**
Che è indivisibile: o tutte le operazioni vengono eseguite, o nessuna. Non esistono stati intermedi.

**Cosa garantisce la proprietà di isolamento?**
Che transazioni parallele non si interferiscono: ogni transazione si comporta come se fosse l'unica in esecuzione.

**Cosa succede se un DBMS va in crash durante una transazione?**
Il sistema usa il meccanismo di recovery per ripristinare lo stato coerente precedente al crash, annullando le transazioni incomplete e confermando quelle già committate.

**Perché il controllo della concorrenza è necessario?**
Perché in un DBMS reale più utenti lavorano contemporaneamente. Senza controllo, due operazioni sugli stessi dati potrebbero produrre risultati incoerenti.