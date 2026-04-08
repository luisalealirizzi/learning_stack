# Scrivere query: metodo e casistiche

Conoscere le clausole SQL non basta. La competenza reale è saper leggere una richiesta in linguaggio naturale e tradurla in una query corretta. Questo capitolo non introduce nuova teoria: è una palestra guidata con metodo, casistiche e soluzioni commentate.

---

## Lo schema di riferimento

Tutti gli esercizi usano questo schema. Leggilo con attenzione prima di procedere.

```sql
CREATE TABLE studente (
    idStudente  INT         PRIMARY KEY AUTO_INCREMENT,
    nome        VARCHAR(50) NOT NULL,
    cognome     VARCHAR(50) NOT NULL,
    classe      VARCHAR(10) NOT NULL,
    dataNascita DATE
);

CREATE TABLE materia (
    idMateria   INT         PRIMARY KEY AUTO_INCREMENT,
    nome        VARCHAR(50) NOT NULL,
    ore         INT         NOT NULL
);

CREATE TABLE professore (
    idProfessore INT         PRIMARY KEY AUTO_INCREMENT,
    nome         VARCHAR(50) NOT NULL,
    cognome      VARCHAR(50) NOT NULL
);

CREATE TABLE insegnamento (
    idInsegnamento INT PRIMARY KEY AUTO_INCREMENT,
    idMateria      INT NOT NULL,
    idProfessore   INT NOT NULL,
    FOREIGN KEY (idMateria)    REFERENCES materia(idMateria),
    FOREIGN KEY (idProfessore) REFERENCES professore(idProfessore)
);

CREATE TABLE voto (
    idVoto      INT          PRIMARY KEY AUTO_INCREMENT,
    idStudente  INT          NOT NULL,
    idMateria   INT          NOT NULL,
    valore      DECIMAL(4,1) NOT NULL CHECK (valore >= 0 AND valore <= 10),
    data        DATE         NOT NULL,
    FOREIGN KEY (idStudente) REFERENCES studente(idStudente),
    FOREIGN KEY (idMateria)  REFERENCES materia(idMateria)
);
```

---

## Il metodo in quattro passi

Prima di scrivere qualsiasi query, segui sempre questo percorso.

**Passo 1 — Individua le tabelle coinvolte**
Quali entità compaiono nella richiesta? Da quali tabelle provengono i dati?

**Passo 2 — Identifica i collegamenti**
Le tabelle sono collegate? Serve una JOIN? Su quale chiave?

**Passo 3 — Determina il filtro**
C'è una condizione da applicare? Va applicata alle singole righe (WHERE) o ai gruppi (HAVING)?

**Passo 4 — Stabilisci cosa mostrare**
Quali colonne vuoi nel risultato? Servono aggregazioni? Serve un ordinamento?

---

## Casistica 1: query su una sola tabella

### Richiesta
Mostra nome, cognome e classe di tutti gli studenti della classe 5A, ordinati per cognome.

### Ragionamento
- Tabella coinvolta: `studente`
- Nessun JOIN necessario
- Filtro su righe: `WHERE classe = '5A'`
- Ordinamento: `ORDER BY cognome`

### Soluzione

```sql
SELECT nome, cognome, classe
FROM   studente
WHERE  classe = '5A'
ORDER BY cognome ASC;
```

---

## Casistica 2: query con JOIN

### Richiesta
Mostra il nome e il cognome di ogni studente insieme al valore di ogni suo voto e alla materia a cui si riferisce.

### Ragionamento
- Tabelle coinvolte: `studente`, `voto`, `materia`
- Serve un doppio JOIN: studente → voto → materia
- Nessun filtro
- Le colonne da mostrare sono distribuite su tre tabelle

### Soluzione

```sql
SELECT s.nome, s.cognome, m.nome AS materia, v.valore
FROM   studente s
INNER JOIN voto v    ON s.idStudente = v.idStudente
INNER JOIN materia m ON v.idMateria  = m.idMateria
ORDER BY s.cognome, m.nome;
```

::: {.callout-tip}
## Quando usare gli alias
Quando due tabelle hanno una colonna con lo stesso nome (qui `nome` esiste sia in `studente` che in `materia`), gli alias di tabella (`s.nome`, `m.nome`) sono indispensabili per disambiguare. L'alias `AS materia` nella SELECT serve a dare un nome leggibile alla colonna nel risultato.
:::

---

## Casistica 3: query con aggregazione senza raggruppamento

### Richiesta
Qual è la media generale di tutti i voti registrati nel database?

### Ragionamento
- Tabella coinvolta: `voto`
- Nessun JOIN, nessun filtro
- Funzione aggregativa: `AVG(valore)`
- Risultato: una sola riga con un solo valore

### Soluzione

```sql
SELECT AVG(valore) AS media_generale
FROM   voto;
```

---

## Casistica 4: query con GROUP BY

### Richiesta
Mostra il nome di ogni materia e la media dei voti ricevuti in quella materia. Mostra solo le materie con media superiore a 6, ordinate dalla media più alta alla più bassa.

### Ragionamento
- Tabelle coinvolte: `materia`, `voto`
- JOIN su `idMateria`
- Raggruppamento per materia: `GROUP BY`
- Filtro sul gruppo (non sulla riga): `HAVING`
- Ordinamento decrescente sulla media

### Soluzione

```sql
SELECT m.nome AS materia, AVG(v.valore) AS media
FROM   materia m
INNER JOIN voto v ON m.idMateria = v.idMateria
GROUP BY m.idMateria, m.nome
HAVING AVG(v.valore) > 6
ORDER BY media DESC;
```

::: {.callout-note}
## Perché GROUP BY include m.idMateria e m.nome?
In MySQL è sufficiente raggruppare per `m.idMateria` (la chiave primaria determina univocamente il nome). Ma includere anche `m.nome` nel GROUP BY è una buona pratica: rende la query più esplicita e compatibile con altri DBMS più rigidi come PostgreSQL.
:::

---

## Casistica 5: trovare righe senza corrispondenza (LEFT JOIN + IS NULL)

### Richiesta
Quali studenti non hanno ancora ricevuto nessun voto?

### Ragionamento
- Tabelle coinvolte: `studente`, `voto`
- Voglio tutti gli studenti, anche quelli senza voti: `LEFT JOIN`
- Filtro: tengo solo le righe dove non c'è corrispondenza in `voto` → `WHERE v.idVoto IS NULL`

### Soluzione

```sql
SELECT s.nome, s.cognome
FROM   studente s
LEFT JOIN voto v ON s.idStudente = v.idStudente
WHERE  v.idVoto IS NULL;
```

::: {.callout-tip}
## Il pattern LEFT JOIN + IS NULL
Questo schema ricorre spesso: serve ogni volta che si vuole trovare "chi non ha fatto X". Si fa un LEFT JOIN sulla tabella X e si filtra dove la chiave di X è NULL, cioè dove non c'era corrispondenza.
:::

---

## Casistica 6: subquery nel WHERE

### Richiesta
Mostra nome e cognome degli studenti che hanno preso almeno un voto superiore alla media generale di tutti i voti.

### Ragionamento
- Prima devo calcolare la media generale: è un valore aggregato → subquery
- Poi uso quel valore come soglia nel WHERE della query principale
- La subquery restituisce un singolo valore: si usa con `>`

### Soluzione

```sql
SELECT DISTINCT s.nome, s.cognome
FROM   studente s
INNER JOIN voto v ON s.idStudente = v.idStudente
WHERE  v.valore > (SELECT AVG(valore) FROM voto);
```

`DISTINCT` elimina i duplicati: uno studente con più voti sopra la media apparirebbe più volte senza di esso.

::: {.callout-note}
## Cos'è una subquery
Una subquery è una query dentro un'altra query, racchiusa tra parentesi. Può comparire nel `WHERE`, nel `FROM` o nel `SELECT`. Quando restituisce un solo valore, si usa con operatori di confronto (`>`, `=`, `<`). Quando restituisce più righe, si usa con `IN`, `ANY` o `ALL`.
:::

---

## Casistica 7: subquery con IN

### Richiesta
Mostra il nome delle materie in cui almeno uno studente ha preso 10.

### Ragionamento
- Devo trovare gli `idMateria` associati a voti con valore = 10: subquery
- La subquery restituisce più righe (più materie potrebbero avere un 10)
- Uso `IN` per confrontare con un insieme di valori

### Soluzione

```sql
SELECT nome
FROM   materia
WHERE  idMateria IN (
    SELECT idMateria
    FROM   voto
    WHERE  valore = 10
);
```

---

## Casistica 8: subquery nel FROM

### Richiesta
Mostra la media per classe di tutti i voti, ma solo per le classi in cui sono presenti almeno 3 studenti.

### Ragionamento
- Prima calcolo la media per classe con un GROUP BY
- Poi filtro sul numero di studenti per classe: serve un secondo livello
- Una subquery nel FROM crea una tabella temporanea interrogabile

### Soluzione

```sql
SELECT classe, ROUND(media_voti, 2) AS media, num_studenti
FROM (
    SELECT s.classe,
           AVG(v.valore)            AS media_voti,
           COUNT(DISTINCT s.idStudente) AS num_studenti
    FROM   studente s
    INNER JOIN voto v ON s.idStudente = v.idStudente
    GROUP BY s.classe
) AS riepilogo_classi
WHERE  num_studenti >= 3
ORDER BY media DESC;
```

La subquery nel `FROM` deve avere un alias (`riepilogo_classi`). La query esterna la tratta come una tabella normale.

---

## Errori comuni

**Usare WHERE con una funzione aggregativa**

```sql
-- SBAGLIATO
SELECT idMateria, AVG(valore)
FROM   voto
WHERE  AVG(valore) > 6    -- ❌ non si può: AVG non è ancora calcolata
GROUP BY idMateria;

-- CORRETTO
SELECT idMateria, AVG(valore)
FROM   voto
GROUP BY idMateria
HAVING AVG(valore) > 6;   -- ✓ HAVING filtra dopo il raggruppamento
```

**Dimenticare DISTINCT con le subquery**

```sql
-- Può restituire lo stesso studente più volte
SELECT s.nome, s.cognome
FROM   studente s
INNER JOIN voto v ON s.idStudente = v.idStudente
WHERE  v.valore > 8;

-- Corretto se uno studente può avere più voti sopra 8
SELECT DISTINCT s.nome, s.cognome
FROM   studente s
INNER JOIN voto v ON s.idStudente = v.idStudente
WHERE  v.valore > 8;
```

**Mettere nel SELECT colonne non nel GROUP BY e non aggregate**

```sql
-- SBAGLIATO in modalità strict (e logicamente ambiguo)
SELECT s.nome, s.classe, AVG(v.valore)
FROM   studente s
INNER JOIN voto v ON s.idStudente = v.idStudente
GROUP BY s.classe;    -- s.nome non è nel GROUP BY: quale nome mostrare?

-- CORRETTO
SELECT s.classe, AVG(v.valore) AS media
FROM   studente s
INNER JOIN voto v ON s.idStudente = v.idStudente
GROUP BY s.classe;
```

**UPDATE o DELETE senza WHERE**

```sql
-- Cancella TUTTI i voti, non solo quello voluto
DELETE FROM voto;

-- Corretto: specifica sempre la condizione
DELETE FROM voto WHERE idVoto = 42;
```

Prima di ogni UPDATE o DELETE, esegui la stessa condizione con un SELECT per verificare quali righe verranno modificate.

---

## Schema mentale

**Come capire se serve una JOIN o una subquery?**
Se hai bisogno di colonne da più tabelle nel risultato finale, usa JOIN. Se hai bisogno di un valore calcolato da un'altra tabella solo come condizione di filtro, una subquery nel WHERE è spesso più leggibile.

**Quando usare DISTINCT?**
Quando un JOIN può produrre righe duplicate perché una riga di una tabella si collega a più righe dell'altra. Controllare sempre se il risultato atteso ha righe uniche per entità.

**Come verificare una query prima di eseguirla in produzione?**
Sostituire temporaneamente `DELETE` o `UPDATE` con `SELECT *` e la stessa clausola WHERE, per vedere esattamente quali righe verrebbero coinvolte.

**Quando una subquery restituisce più righe, quale operatore si usa?**
`IN` per verificare se un valore appartiene all'insieme restituito. `ANY` o `ALL` per confronti con ogni elemento dell'insieme. Se la subquery restituisce un solo valore, si usano i normali operatori di confronto (`=`, `>`, `<`).