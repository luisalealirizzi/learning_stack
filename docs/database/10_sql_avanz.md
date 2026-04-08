# SQL avanzato: JOIN, GROUP BY e viste

Nel capitolo precedente abbiamo visto come costruire e interrogare una singola tabella. Ma un database normalizzato distribuisce le informazioni su più tabelle collegate. In questo capitolo vediamo come ricombinarle, come aggregare i risultati e come costruire viste che semplificano le interrogazioni ricorrenti.

---

## JOIN: interrogare più tabelle insieme

La **JOIN** è la clausola SQL che combina righe provenienti da due o più tabelle, in base a una condizione di collegamento. Corrisponde alla join dell'algebra relazionale vista nel capitolo 08.

La condizione di collegamento si basa quasi sempre su una chiave primaria e la corrispondente chiave esterna. È esattamente il collegamento che abbiamo definito con `FOREIGN KEY` nello schema.

Partiamo da queste tabelle:

```
studente(idStudente, nome, cognome)
corso(idCorso, titolo, idProfessore)
iscrizione(idIscrizione, idStudente FK, idCorso FK, dataIscrizione)
```

### INNER JOIN

L'**INNER JOIN** restituisce solo le righe che hanno una corrispondenza in entrambe le tabelle. Le righe senza corrispondenza vengono escluse da entrambi i lati.

```sql
SELECT s.nome, s.cognome, c.titolo
FROM   studente s
INNER JOIN iscrizione i ON s.idStudente = i.idStudente
INNER JOIN corso c      ON i.idCorso    = c.idCorso;
```

Questo restituisce il nome e cognome di ogni studente insieme al titolo del corso a cui è iscritto. Gli studenti senza iscrizioni e i corsi senza iscritti non compaiono nel risultato.

::: {.callout-note}
## Alias di tabella
`studente s` assegna l'alias `s` alla tabella `studente`. Serve a scrivere `s.nome` invece di `studente.nome`: indispensabile quando le stesse colonne compaiono in più tabelle, e utile per rendere la query più leggibile.
:::

### LEFT JOIN

Il **LEFT JOIN** restituisce tutte le righe della tabella di sinistra, anche quelle senza corrispondenza nella tabella di destra. Per le righe senza corrispondenza, le colonne della tabella di destra sono `NULL`.

```sql
SELECT s.nome, s.cognome, c.titolo
FROM   studente s
LEFT JOIN iscrizione i ON s.idStudente = i.idStudente
LEFT JOIN corso c      ON i.idCorso    = c.idCorso;
```

Questo restituisce tutti gli studenti, inclusi quelli che non hanno nessuna iscrizione. Per questi ultimi, `c.titolo` sarà `NULL`.

Uso tipico: trovare le righe "orfane", cioè quelle senza corrispondenza.

```sql
-- Studenti che non sono iscritti ad alcun corso
SELECT s.nome, s.cognome
FROM   studente s
LEFT JOIN iscrizione i ON s.idStudente = i.idStudente
WHERE  i.idStudente IS NULL;
```

### RIGHT JOIN

Il **RIGHT JOIN** è simmetrico al LEFT JOIN: restituisce tutte le righe della tabella di destra, anche quelle senza corrispondenza a sinistra. Meno usato in pratica, perché si può sempre riscrivere come LEFT JOIN invertendo l'ordine delle tabelle.

### Riepilogo visivo

Immagina due insiemi sovrapposti: le righe della tabella A a sinistra, le righe della tabella B a destra, le righe con corrispondenza al centro.

| Tipo | Restituisce |
|---|---|
| `INNER JOIN` | solo le righe al centro (corrispondenza in entrambe) |
| `LEFT JOIN` | tutte le righe di A, con NULL dove non c'è corrispondenza in B |
| `RIGHT JOIN` | tutte le righe di B, con NULL dove non c'è corrispondenza in A |

---

## Funzioni aggregative

Le **funzioni aggregative** elaborano un insieme di righe e producono un singolo valore riassuntivo. Non restituiscono ogni singola riga: sintetizzano i dati.

| Funzione | Risultato |
|---|---|
| `COUNT(*)` | numero di righe |
| `COUNT(colonna)` | numero di righe con valore non NULL |
| `SUM(colonna)` | somma dei valori |
| `AVG(colonna)` | media dei valori |
| `MIN(colonna)` | valore minimo |
| `MAX(colonna)` | valore massimo |

Esempi:

```sql
-- Quanti studenti ci sono nel database?
SELECT COUNT(*) AS totale_studenti
FROM studente;

-- Voto medio di tutti gli esami
SELECT AVG(voto) AS media_voti
FROM esame;

-- Voto più alto e più basso registrati
SELECT MAX(voto) AS voto_massimo,
       MIN(voto) AS voto_minimo
FROM esame;
```

---

## GROUP BY: aggregare per gruppi

Le funzioni aggregative applicate all'intera tabella producono un solo valore. Con **GROUP BY** si suddivide prima la tabella in gruppi, e poi si calcola il valore aggregato per ciascun gruppo.

```sql
SELECT idCorso, AVG(voto) AS media_voto
FROM   esame
GROUP BY idCorso;
```

Questo restituisce una riga per ogni corso, con la media dei voti degli esami di quel corso.

::: {.callout-important}
## La regola del GROUP BY
Nella clausola `SELECT`, quando si usa `GROUP BY`, si possono mettere solo due tipi di elementi: le colonne che compaiono nel `GROUP BY`, e funzioni aggregative. Qualsiasi altra colonna genererebbe ambiguità (quale valore mostrare tra le tante righe del gruppo?).
:::

Esempio più articolato — nome del corso e numero di iscritti, solo per i corsi con più di 5 iscritti:

```sql
SELECT c.titolo, COUNT(i.idStudente) AS num_iscritti
FROM   corso c
INNER JOIN iscrizione i ON c.idCorso = i.idCorso
GROUP BY c.idCorso, c.titolo
HAVING COUNT(i.idStudente) > 5
ORDER BY num_iscritti DESC;
```

### HAVING: filtrare i gruppi

**HAVING** è il filtro che si applica *dopo* il raggruppamento. Si usa per selezionare solo i gruppi che soddisfano una condizione aggregata.

La distinzione fondamentale:

| Clausola | Quando si applica | Cosa filtra |
|---|---|---|
| `WHERE` | prima del raggruppamento | singole righe |
| `HAVING` | dopo il raggruppamento | gruppi aggregati |

```sql
-- WHERE: esclude le righe prima di raggruppare
-- HAVING: esclude i gruppi dopo aver calcolato l'aggregato

SELECT idCorso, AVG(voto) AS media
FROM   esame
WHERE  voto IS NOT NULL          -- escludo esami senza voto
GROUP BY idCorso
HAVING AVG(voto) > 7;            -- tengo solo i corsi con media > 7
```

---

## L'ordine delle clausole in una query completa

Una query SQL completa segue un ordine preciso di clausole. Il DBMS le elabora in un ordine diverso da come sono scritte:

```sql
SELECT   colonne o aggregazioni          -- 5. cosa mostrare
FROM     tabella                         -- 1. da dove
JOIN     altra_tabella ON condizione     -- 2. come collegare
WHERE    condizione_riga                 -- 3. filtrare le righe
GROUP BY colonne                         -- 4. raggruppare
HAVING   condizione_gruppo               -- 5. filtrare i gruppi
ORDER BY colonna                         -- 6. ordinare
LIMIT    n;                              -- 7. limitare i risultati
```

Non tutte le clausole sono obbligatorie: l'unica sempre presente è `SELECT ... FROM ...`. Le altre si aggiungono secondo necessità.

---

## Le viste

Una **vista** è una tabella virtuale definita da una query. Non contiene dati propri: ogni volta che viene interrogata, il DBMS esegue la query sottostante e restituisce il risultato.

```sql
-- Creazione di una vista
CREATE VIEW voti_studenti AS
SELECT s.nome, s.cognome, c.titolo, e.voto
FROM   studente s
INNER JOIN esame e    ON s.idStudente = e.idStudente
INNER JOIN corso c    ON e.idCorso    = c.idCorso;

-- Uso della vista: si interroga come una tabella normale
SELECT nome, cognome, voto
FROM   voti_studenti
WHERE  voto >= 6;
```

Le viste servono a tre scopi principali.

**Semplificazione**: una query complessa con più JOIN può essere incapsulata in una vista. Chi la usa non deve conoscere la struttura sottostante.

**Sicurezza**: una vista può esporre solo alcune colonne o alcune righe di una tabella. Un utente può avere accesso alla vista ma non alle tabelle originali. In una scuola, il docente può avere accesso a una vista che mostra solo i voti della propria classe.

**Astrazione**: se la struttura delle tabelle cambia, basta aggiornare la vista. Le applicazioni che usano la vista non devono essere modificate.

```sql
-- Eliminare una vista
DROP VIEW voti_studenti;
```

::: {.callout-note}
## Vista vs tabella
Una tabella occupa spazio fisico sul disco. Una vista no: è solo una query salvata con un nome. Il vantaggio è la freschezza dei dati: una vista mostra sempre il contenuto attuale delle tabelle sottostanti, non una copia.
:::

---

## Schema mentale

**Qual è la differenza tra INNER JOIN e LEFT JOIN?**
L'INNER JOIN restituisce solo le righe con corrispondenza in entrambe le tabelle. Il LEFT JOIN restituisce tutte le righe della tabella di sinistra, mettendo NULL dove non c'è corrispondenza a destra.

**Quando si usa HAVING invece di WHERE?**
WHERE filtra le singole righe prima del raggruppamento. HAVING filtra i gruppi dopo che le funzioni aggregative sono state calcolate. Non si può usare un'aggregazione come `AVG(voto)` dentro un WHERE.

**Cosa si può mettere nel SELECT quando si usa GROUP BY?**
Solo le colonne presenti nel GROUP BY e le funzioni aggregative. Qualsiasi altra colonna è ambigua: il DBMS non saprebbe quale valore scegliere tra le righe del gruppo.

**Qual è la differenza tra una vista e una tabella?**
Una tabella contiene dati fisici memorizzati su disco. Una vista è una query salvata con un nome: ogni volta che viene interrogata, il DBMS esegue la query e restituisce i dati aggiornati. Non occupa spazio autonomo.

**Perché le viste sono utili per la sicurezza?**
Perché permettono di esporre solo una parte delle informazioni presenti nelle tabelle. Un utente può avere accesso alla vista senza avere accesso diretto alle tabelle sottostanti.