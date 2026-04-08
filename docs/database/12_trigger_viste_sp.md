# Trigger, viste e stored procedure

Fino a questo punto abbiamo usato SQL in modo interattivo: scriviamo una query, il DBMS la esegue, otteniamo un risultato. Esistono però tre strumenti che permettono di spostare logica e automazione *dentro* il database stesso: le **viste**, i **trigger** e le **stored procedure**. Ognuno risponde a un'esigenza diversa.

---

## Viste

Una **vista** è una query salvata con un nome. Si comporta come una tabella: si può interrogare con SELECT, filtrare, unire ad altre tabelle. Ma non contiene dati propri: ogni volta che viene interrogata, il DBMS esegue la query sottostante e restituisce i dati aggiornati.

```sql
-- Definizione della vista
CREATE VIEW voti_per_studente AS
SELECT s.nome, s.cognome, m.nome AS materia, v.valore, v.data
FROM   studente s
INNER JOIN voto v    ON s.idStudente = v.idStudente
INNER JOIN materia m ON v.idMateria  = m.idMateria;

-- Uso della vista: si interroga come una tabella normale
SELECT nome, cognome, materia, valore
FROM   voti_per_studente
WHERE  valore < 6
ORDER BY cognome;

-- Eliminare una vista
DROP VIEW voti_per_studente;
```

### A cosa servono le viste

**Semplificazione**: una JOIN complessa su tre tabelle può essere incapsulata in una vista. Chi la usa scrive una SELECT semplice senza conoscere la struttura sottostante.

**Sicurezza e controllo degli accessi**: una vista può esporre solo alcune colonne o alcune righe. Un utente può avere il permesso di interrogare la vista ma non di accedere direttamente alle tabelle originali.

```sql
-- Vista che mostra solo i dati anagrafici, senza voti
CREATE VIEW anagrafica_studenti AS
SELECT idStudente, nome, cognome, classe
FROM   studente;

-- Il docente vede solo i voti della propria classe
CREATE VIEW voti_5a AS
SELECT s.nome, s.cognome, m.nome AS materia, v.valore
FROM   studente s
INNER JOIN voto v    ON s.idStudente = v.idStudente
INNER JOIN materia m ON v.idMateria  = m.idMateria
WHERE  s.classe = '5A';
```

**Astrazione**: se la struttura delle tabelle cambia, si aggiorna la vista. Le applicazioni che la usano non devono essere modificate.

::: {.callout-note}
## Vista vs tabella
Una tabella occupa spazio fisico su disco. Una vista no: è solo una query salvata. Il vantaggio è che mostra sempre i dati aggiornati. Lo svantaggio è che se la query sottostante è complessa, viene rieseguita ogni volta che la vista viene interrogata.
:::

---

## Trigger

Un **trigger** è un blocco di codice SQL che il DBMS esegue automaticamente in risposta a un evento su una tabella: un inserimento, una modifica o una cancellazione.

Non viene chiamato dall'utente: scatta da solo quando si verifica la condizione specificata.

### Struttura di un trigger

```sql
CREATE TRIGGER nome_trigger
    { BEFORE | AFTER }
    { INSERT | UPDATE | DELETE }
    ON nome_tabella
    FOR EACH ROW
BEGIN
    -- istruzioni SQL
END;
```

- `BEFORE`: il trigger si esegue prima dell'operazione
- `AFTER`: il trigger si esegue dopo l'operazione
- `FOR EACH ROW`: il trigger si esegue una volta per ogni riga coinvolta

All'interno del trigger si usano due riferimenti speciali:
- `NEW`: la riga con i nuovi valori (disponibile in INSERT e UPDATE)
- `OLD`: la riga con i vecchi valori (disponibile in UPDATE e DELETE)

### Esempio 1: validazione

Impedire l'inserimento di un voto fuori dall'intervallo ammesso, aggiungendo un controllo più esplicito rispetto al vincolo CHECK.

```sql
DELIMITER $$

CREATE TRIGGER controlla_voto
BEFORE INSERT ON voto
FOR EACH ROW
BEGIN
    IF NEW.valore < 0 OR NEW.valore > 10 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Il voto deve essere compreso tra 0 e 10';
    END IF;
END$$

DELIMITER ;
```

`SIGNAL SQLSTATE '45000'` genera un errore che interrompe l'operazione e restituisce il messaggio all'applicazione.

### Esempio 2: log automatico

Registrare automaticamente ogni modifica a un voto in una tabella di log.

```sql
CREATE TABLE log_voti (
    idLog       INT PRIMARY KEY AUTO_INCREMENT,
    idVoto      INT,
    vecchioValore DECIMAL(4,1),
    nuovoValore   DECIMAL(4,1),
    dataModifica  DATETIME DEFAULT NOW()
);

DELIMITER $$

CREATE TRIGGER log_modifica_voto
AFTER UPDATE ON voto
FOR EACH ROW
BEGIN
    INSERT INTO log_voti (idVoto, vecchioValore, nuovoValore)
    VALUES (OLD.idVoto, OLD.valore, NEW.valore);
END$$

DELIMITER $$
```

Ogni volta che un voto viene modificato, il trigger inserisce automaticamente una riga in `log_voti` con i valori prima e dopo la modifica.

### Esempio 3: aggiornamento automatico

Quando si cancella uno studente, impostare automaticamente a NULL il suo riferimento in un'altra tabella (alternativa al CASCADE).

```sql
DELIMITER $$

CREATE TRIGGER dopo_cancellazione_studente
AFTER DELETE ON studente
FOR EACH ROW
BEGIN
    UPDATE progetto
    SET    idStudente = NULL
    WHERE  idStudente = OLD.idStudente;
END$$

DELIMITER ;
```

::: {.callout-warning}
## Trigger e debugging
I trigger sono invisibili a chi esegue una query: scattano in automatico. Se un trigger contiene un errore o produce effetti inattesi, può essere difficile capire perché un'operazione fallisce o perché i dati cambiano in modo inaspettato. Documentare i trigger è essenziale.
:::

### Visualizzare ed eliminare i trigger

```sql
-- Vedere i trigger definiti nel database
SHOW TRIGGERS;

-- Eliminare un trigger
DROP TRIGGER log_modifica_voto;
```

---

## Stored procedure

Una **stored procedure** è un blocco di codice SQL con un nome, salvato nel database e richiamabile come una funzione. Può ricevere parametri in input, eseguire operazioni complesse (query, inserimenti, aggiornamenti, controlli) e restituire risultati.

A differenza dei trigger, una stored procedure viene chiamata esplicitamente dall'utente o dall'applicazione.

### Struttura di una stored procedure

```sql
DELIMITER $$

CREATE PROCEDURE nome_procedura (
    IN  parametro1 TIPO,
    OUT parametro2 TIPO
)
BEGIN
    -- istruzioni SQL
END$$

DELIMITER ;

-- Chiamata della procedura
CALL nome_procedura(valore1, @variabile_output);
```

I parametri possono essere:
- `IN`: valore passato alla procedura (sola lettura)
- `OUT`: valore restituito dalla procedura
- `INOUT`: valore passato e poi modificato dalla procedura

### Esempio 1: procedura senza parametri

Calcola e mostra la media voti per ogni classe.

```sql
DELIMITER $$

CREATE PROCEDURE media_per_classe()
BEGIN
    SELECT s.classe, ROUND(AVG(v.valore), 2) AS media
    FROM   studente s
    INNER JOIN voto v ON s.idStudente = v.idStudente
    GROUP BY s.classe
    ORDER BY s.classe;
END$$

DELIMITER ;

-- Chiamata
CALL media_per_classe();
```

### Esempio 2: procedura con parametro IN

Restituisce tutti i voti di uno studente dato il suo id.

```sql
DELIMITER $$

CREATE PROCEDURE voti_studente(IN p_idStudente INT)
BEGIN
    SELECT m.nome AS materia, v.valore, v.data
    FROM   voto v
    INNER JOIN materia m ON v.idMateria = m.idMateria
    WHERE  v.idStudente = p_idStudente
    ORDER BY v.data;
END$$

DELIMITER ;

-- Chiamata
CALL voti_studente(3);
```

### Esempio 3: procedura con parametro OUT

Calcola la media dei voti di uno studente e la restituisce come parametro.

```sql
DELIMITER $$

CREATE PROCEDURE media_studente(
    IN  p_idStudente INT,
    OUT p_media      DECIMAL(4,2)
)
BEGIN
    SELECT AVG(valore)
    INTO   p_media
    FROM   voto
    WHERE  idStudente = p_idStudente;
END$$

DELIMITER ;

-- Chiamata
CALL media_studente(3, @risultato);
SELECT @risultato AS media;
```

::: {.callout-tip}
## Quando usare una stored procedure
Le stored procedure sono utili quando la stessa sequenza di operazioni viene eseguita spesso, quando si vuole centralizzare la logica nel database invece che nell'applicazione, e quando si vuole limitare l'accesso diretto alle tabelle (l'applicazione chiama la procedura, non interroga le tabelle direttamente).
:::

### Visualizzare ed eliminare le stored procedure

```sql
-- Vedere le procedure definite nel database
SHOW PROCEDURE STATUS WHERE Db = 'nome_database';

-- Eliminare una procedura
DROP PROCEDURE media_studente;
```

---

## Confronto tra i tre strumenti

| | Vista | Trigger | Stored procedure |
|---|---|---|---|
| Quando si attiva | alla SELECT | automaticamente su INSERT/UPDATE/DELETE | chiamata esplicitamente |
| Riceve parametri | no | no (usa NEW/OLD) | sì |
| Modifica i dati | no (di solito) | sì | sì |
| Uso tipico | semplificare letture, controllo accessi | automazioni, log, validazioni | operazioni ripetute, logica complessa |

---

## Schema mentale

**Qual è la differenza tra una vista e una tabella?**
Una tabella contiene dati fisici su disco. Una vista è una query salvata con un nome: non occupa spazio autonomo e restituisce sempre dati aggiornati al momento dell'interrogazione.

**Quando scatta un trigger?**
Automaticamente, quando si verifica un evento specifico su una tabella: INSERT, UPDATE o DELETE. Non viene chiamato dall'utente: è il DBMS a eseguirlo.

**Qual è la differenza tra BEFORE e AFTER in un trigger?**
`BEFORE` si esegue prima dell'operazione: può modificare i valori di NEW o interrompere l'operazione. `AFTER` si esegue dopo: non può annullare l'operazione, ma può eseguire azioni conseguenti (log, aggiornamenti in altre tabelle).

**Qual è la differenza tra trigger e stored procedure?**
Il trigger scatta automaticamente in risposta a un evento. La stored procedure viene chiamata esplicitamente dall'utente o dall'applicazione, e può ricevere parametri.

**Quando conviene usare una stored procedure invece di una query diretta?**
Quando la stessa logica complessa viene eseguita spesso, quando si vuole centralizzare le operazioni nel database, o quando si vuole che l'applicazione acceda ai dati solo tramite procedure controllate, senza poter interrogare direttamente le tabelle.