# SQL: il linguaggio dei database

L'algebra relazionale descrive *cosa* si può fare con i dati in modo matematico e astratto. SQL è il linguaggio con cui queste operazioni vengono eseguite concretamente nei database. Questo capitolo copre le basi: cos'è SQL, come è organizzato, e i comandi fondamentali per definire e manipolare i dati.

---

## Cos'è SQL

**SQL** (*Structured Query Language*) è il linguaggio standard per definire, modificare e interrogare i dati nei database relazionali. Nasce negli anni '70 nei laboratori IBM durante il progetto *System R*, inizialmente sotto il nome di SEQUEL (*Structured English Query Language*). Nel 1986 viene adottato come standard internazionale.

La caratteristica più importante di SQL è che è un linguaggio **dichiarativo**: l'utente descrive *cosa* vuole ottenere, non *come* il sistema deve trovarlo. Sarà il DBMS a scegliere il modo più efficiente per eseguire l'operazione.

Questo è un cambio di prospettiva rispetto alla programmazione tradizionale. In C o Java si scrive un algoritmo che attraversa i dati passo per passo. In SQL si formula una richiesta:

```sql
-- In un linguaggio imperativo: ciclo, confronto, raccolta risultati
-- In SQL: si descrive il risultato voluto
SELECT nome, cognome FROM studenti WHERE voto >= 6;
```

::: {.callout-note}
## SQL e i DBMS
Pur essendo standardizzato, SQL ha piccole differenze tra MySQL, MariaDB, PostgreSQL, Oracle e altri DBMS. In questo corso usiamo MySQL/MariaDB. La struttura fondamentale del linguaggio è identica in tutti.
:::

---

## Le tre famiglie di comandi

SQL si divide in tre categorie principali, ognuna con uno scopo preciso.

| Categoria | Nome completo | Scopo |
|---|---|---|
| **DDL** | Data Definition Language | definire la struttura del database |
| **DML** | Data Manipulation Language | inserire, modificare, cancellare dati |
| **DQL** | Data Query Language | interrogare e leggere i dati |

---

## DDL: costruire la struttura

Il DDL serve a definire lo **schema** del database: quali tabelle esistono, quali colonne contengono, quali chiavi e vincoli devono essere rispettati. I comandi DDL cambiano la struttura, non i dati.

I tre comandi principali sono `CREATE`, `ALTER` e `DROP`.

### CREATE TABLE

Crea una nuova tabella. Si specifica il nome di ogni colonna, il tipo di dato e gli eventuali vincoli.

```sql
CREATE TABLE studente (
    idStudente  INT           NOT NULL AUTO_INCREMENT,
    matricola   CHAR(6)       NOT NULL UNIQUE,
    nome        VARCHAR(50)   NOT NULL,
    cognome     VARCHAR(50)   NOT NULL,
    dataNascita DATE,
    PRIMARY KEY (idStudente)
);
```

Cosa succede qui:
- `AUTO_INCREMENT` genera automaticamente un nuovo valore intero per ogni riga inserita
- `NOT NULL` impedisce che il campo venga lasciato vuoto
- `UNIQUE` garantisce che due studenti non abbiano la stessa matricola
- `PRIMARY KEY` dichiara la chiave primaria della tabella

### Chiavi esterne con FOREIGN KEY

Per collegare due tabelle si dichiara una chiave esterna:

```sql
CREATE TABLE iscrizione (
    idIscrizione  INT  NOT NULL AUTO_INCREMENT,
    dataIscrizione DATE NOT NULL,
    idStudente    INT  NOT NULL,
    idCorso       INT  NOT NULL,
    PRIMARY KEY (idIscrizione),
    FOREIGN KEY (idStudente) REFERENCES studente(idStudente),
    FOREIGN KEY (idCorso)    REFERENCES corso(idCorso)
);
```

Il DBMS garantisce che ogni `idStudente` inserito in `iscrizione` esista realmente nella tabella `studente`. Se si prova a inserire un riferimento a uno studente inesistente, l'operazione viene rifiutata.

### Vincoli CHECK e DEFAULT

```sql
CREATE TABLE esame (
    idEsame  INT          NOT NULL AUTO_INCREMENT,
    voto     DECIMAL(4,1) NOT NULL CHECK (voto >= 0 AND voto <= 10),
    stato    VARCHAR(10)  DEFAULT 'in attesa',
    PRIMARY KEY (idEsame)
);
```

- `CHECK` impone una condizione sul valore: il DBMS la verifica a ogni inserimento o modifica
- `DEFAULT` assegna un valore predefinito se non viene specificato nulla

### ALTER TABLE

Modifica la struttura di una tabella esistente: aggiunge o rimuove colonne, modifica tipi di dato, aggiunge vincoli.

```sql
-- Aggiungere una colonna
ALTER TABLE studente ADD email VARCHAR(100) UNIQUE;

-- Modificare il tipo di una colonna
ALTER TABLE studente MODIFY nome VARCHAR(100) NOT NULL;

-- Rimuovere una colonna
ALTER TABLE studente DROP COLUMN email;
```

### DROP TABLE

Elimina una tabella e tutti i dati che contiene. È un'operazione irreversibile.

```sql
DROP TABLE iscrizione;
```

::: {.callout-warning}
## DROP è permanente
Un `DROP TABLE` non chiede conferma e non è annullabile. Prima di eseguirlo in un database di produzione, assicurarsi di avere un backup.
:::

---

## DML: lavorare sui dati

Il DML serve a modificare il **contenuto** delle tabelle: inserire nuovi record, aggiornare quelli esistenti, cancellare quelli non più necessari. I comandi DML rispettano sempre i vincoli definiti con il DDL.

### INSERT

Inserisce una o più righe in una tabella.

```sql
-- Inserimento con tutti i campi nell'ordine della tabella
INSERT INTO studente (matricola, nome, cognome, dataNascita)
VALUES ('S00123', 'Luca', 'Rossi', '2006-03-14');

-- Inserimento multiplo
INSERT INTO studente (matricola, nome, cognome)
VALUES ('S00124', 'Anna', 'Bianchi'),
       ('S00125', 'Marco', 'Verdi');
```

Se si tenta di inserire un record con una chiave primaria già esistente, o con una chiave esterna che non ha corrispondenza, il DBMS rifiuta l'operazione.

### UPDATE

Modifica i valori di uno o più campi in righe esistenti.

```sql
-- Aggiornare un singolo record
UPDATE studente
SET email = 'luca.rossi@scuola.it'
WHERE idStudente = 1;

-- Aggiornare più record contemporaneamente
UPDATE esame
SET stato = 'completato'
WHERE dataEsame < '2025-01-01';
```

::: {.callout-important}
## UPDATE senza WHERE aggiorna tutto
Un `UPDATE` senza clausola `WHERE` modifica **tutte** le righe della tabella. È uno degli errori più comuni e potenzialmente più distruttivi. Prima di eseguire un UPDATE, verificare sempre la condizione con un SELECT.
:::

### DELETE

Elimina righe da una tabella.

```sql
-- Eliminare un record specifico
DELETE FROM studente
WHERE idStudente = 5;

-- Eliminare tutti i record che soddisfano una condizione
DELETE FROM iscrizione
WHERE dataIscrizione < '2020-01-01';
```

Come per UPDATE, un `DELETE` senza `WHERE` elimina tutte le righe della tabella.

::: {.callout-note}
## SQL injection
Un rischio storico del DML è la **SQL injection**: quando l'input dell'utente viene inserito direttamente in una query SQL senza controlli, un utente malintenzionato può inserire istruzioni SQL che il database esegue. La difesa principale è usare query parametrizzate nei programmi che interagiscono con il database. Il fumetto xkcd 327 (*"Little Bobby Tables"*) è diventato l'illustrazione classica di questo problema.
:::

---

## DQL: interrogare i dati

Il DQL serve a **leggere** i dati. Non modifica nulla nel database: produce un risultato sotto forma di tabella temporanea. Il comando fondamentale è `SELECT`.

### Struttura base di SELECT

```sql
SELECT colonne
FROM   tabella
WHERE  condizione;
```

Il risultato è sempre una tabella. Anche se il risultato è vuoto (nessuna riga soddisfa la condizione), è comunque una tabella, con le colonne specificate e zero righe.

### Selezionare tutte le colonne

```sql
SELECT * FROM studente;
```

L'asterisco `*` seleziona tutte le colonne. Utile in fase di esplorazione, ma da evitare nei programmi: se la struttura della tabella cambia, la query potrebbe restituire colonne inaspettate.

### Selezionare colonne specifiche

```sql
SELECT nome, cognome FROM studente;
```

Corrisponde alla **proiezione** dell'algebra relazionale: si scelgono solo le colonne rilevanti.

### Filtrare con WHERE

```sql
SELECT nome, cognome
FROM   studente
WHERE  cognome = 'Rossi';
```

La clausola `WHERE` filtra le righe in base a una condizione. Corrisponde alla **selezione** dell'algebra relazionale.

Gli operatori disponibili nella condizione:

| Operatore | Significato |
|---|---|
| `=` | uguale |
| `<>` o `!=` | diverso |
| `>`, `<`, `>=`, `<=` | confronto numerico |
| `AND`, `OR`, `NOT` | operatori logici |
| `BETWEEN a AND b` | intervallo incluso |
| `LIKE 'pattern'` | confronto su testo con caratteri jolly |
| `IN (v1, v2, ...)` | valore in un insieme |
| `IS NULL` | valore assente |

Esempi:

```sql
-- Studenti nati dopo il 2005
SELECT nome, cognome
FROM   studente
WHERE  dataNascita > '2005-12-31';

-- Studenti con cognome che inizia per R
SELECT nome, cognome
FROM   studente
WHERE  cognome LIKE 'R%';

-- Esami con voto tra 6 e 8
SELECT *
FROM   esame
WHERE  voto BETWEEN 6 AND 8;

-- Studenti senza email registrata
SELECT nome, cognome
FROM   studente
WHERE  email IS NULL;
```

### Ordinare i risultati con ORDER BY

```sql
SELECT nome, cognome, dataNascita
FROM   studente
ORDER BY cognome ASC, nome ASC;
```

`ASC` ordina in modo crescente (default), `DESC` in modo decrescente. Si possono specificare più colonne di ordinamento.

### Alias con AS

```sql
SELECT nome AS 'Nome studente', cognome AS 'Cognome'
FROM   studente;
```

`AS` assegna un nome alternativo a una colonna nel risultato. Utile per rendere più leggibili i risultati e obbligatorio quando si usano espressioni o funzioni.

---

## Schema mentale

**Qual è la differenza tra DDL e DML?**
Il DDL definisce la struttura del database (tabelle, colonne, chiavi, vincoli). Il DML lavora sui dati: inserisce, modifica e cancella record. Il DDL cambia lo schema, il DML cambia l'istanza.

**Perché SQL è detto dichiarativo?**
Perché si descrive il risultato voluto, non la procedura per ottenerlo. È il DBMS a decidere come eseguire la query nel modo più efficiente.

**Cosa succede se si esegue un UPDATE o DELETE senza WHERE?**
L'operazione viene applicata a tutte le righe della tabella. È uno degli errori più pericolosi: verificare sempre la condizione con un SELECT prima di modificare o cancellare.

**Cosa garantisce FOREIGN KEY?**
Che ogni valore inserito come chiave esterna corrisponda a un valore esistente nella tabella collegata. Il DBMS rifiuta inserimenti o modifiche che violerebbero questa regola.

**Qual è il collegamento tra SELECT/WHERE e l'algebra relazionale?**
`SELECT colonne` corrisponde alla proiezione (π). `WHERE condizione` corrisponde alla selezione (σ). Una query SQL combina entrambe le operazioni in un'unica istruzione.