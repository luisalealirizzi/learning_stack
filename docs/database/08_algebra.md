# Algebra relazionale

SQL permette di interrogare un database descrivendo *cosa* si vuole ottenere. Ma da dove viene questa idea? Prima del linguaggio c'è la teoria: l'**algebra relazionale** è il sistema di operazioni matematiche su cui SQL è fondato.

---

## Prima delle query c'è il calcolo

Negli anni '30 non esistono computer. Eppure matematici come Alan Turing e Alonzo Church si pongono una domanda fondamentale: cos'è un calcolo? Non come si esegue su una macchina, ma che cosa significa calcolare.

Scoprono qualcosa di decisivo: un calcolo può essere descritto come una **trasformazione formale di simboli**, indipendente dall'hardware che lo esegue. Il calcolo precede la macchina.

Quando Edgar F. Codd affronta il problema dei dati, fa la stessa mossa concettuale. Non si chiede: come scorro i record? In che ordine leggo la memoria? Si chiede: **quali trasformazioni matematiche posso applicare ai dati?**

Nasce così l'algebra relazionale: un sistema di operazioni astratte che descrive *cosa* ottenere, non *come* ottenerlo. Selezione, proiezione, join non sono comandi procedurali. Sono operazioni matematiche applicate a insiemi di dati.

Per questo l'algebra relazionale è indipendente dal linguaggio, dal DBMS e dalla struttura fisica dei dati. Prima delle query c'è il calcolo. Prima della macchina c'è l'algebra.

---

## Le operazioni fondamentali

Ogni operazione dell'algebra relazionale prende una o più relazioni come input e restituisce sempre una nuova relazione come output. Questo significa che le operazioni si possono **comporre**: il risultato di una può diventare l'input di un'altra.

---

### Selezione (σ)

La **selezione** filtra le righe di una relazione in base a una condizione. Non modifica le colonne: agisce solo sul numero di righe.

Notazione formale:

```
σ_condizione(R)
```

Esempio: dalla relazione `STUDENTE(idStudente, nome, voto)`, voglio solo gli studenti con voto maggiore o uguale a 6:

```
σ_voto >= 6 (STUDENTE)
```

Il risultato è una nuova relazione con lo stesso schema di `STUDENTE`, ma solo le righe che soddisfano la condizione.

Osservazioni:
- la selezione non cambia lo schema della relazione
- può usare operatori di confronto: `>`, `<`, `=`, `>=`, `<=`, `<>`
- può combinare condizioni con `AND`, `OR`, `NOT`

---

### Proiezione (π)

La **proiezione** seleziona alcune colonne di una relazione. Non modifica le righe: agisce solo sull'insieme degli attributi.

Notazione formale:

```
π_lista_attributi(R)
```

Esempio: dalla relazione `STUDENTE(idStudente, nome, cognome, voto)`, voglio solo nome e voto:

```
π_nome, voto (STUDENTE)
```

Il risultato è una nuova relazione con solo le colonne richieste. Se la proiezione produce righe duplicate, queste vengono eliminate: una relazione non può contenere tuple identiche.

Osservazioni:
- la proiezione elimina le colonne non indicate
- può eliminare duplicati come effetto collaterale
- l'ordine delle colonne nel risultato non è significativo nella teoria formale

---

### Selezione e proiezione combinate

Le due operazioni si compongono facilmente. Per ottenere solo il nome degli studenti con voto maggiore o uguale a 6:

```
π_nome (σ_voto >= 6 (STUDENTE))
```

Si legge dall'interno verso l'esterno: prima si selezionano le righe con voto >= 6, poi si proietta sul solo attributo `nome`.

---

### Prodotto cartesiano (×)

Il **prodotto cartesiano** combina ogni riga di una relazione con ogni riga di un'altra. Se R ha m righe e S ha n righe, il risultato ha m × n righe.

Notazione formale:

```
R × S
```

Esempio:

Relazione `STUDENTE`:

| idStudente | nome |
|---|---|
| 1 | Luca |
| 2 | Anna |

Relazione `CORSO`:

| idCorso | titolo |
|---|---|
| A | Informatica |
| B | Matematica |

`STUDENTE × CORSO`:

| idStudente | nome | idCorso | titolo |
|---|---|---|---|
| 1 | Luca | A | Informatica |
| 1 | Luca | B | Matematica |
| 2 | Anna | A | Informatica |
| 2 | Anna | B | Matematica |

Il prodotto cartesiano genera molte tuple, la maggior parte prive di significato. Viene usato raramente da solo: il suo ruolo principale è essere la **base teorica della join**.

---

### Join (⨝)

La **join** combina due relazioni mettendo insieme le tuple che soddisfano una condizione. È l'operazione più importante dell'algebra relazionale, perché permette di ricostruire informazioni distribuite su più tabelle.

Notazione formale:

```
R ⨝_condizione S
```

La join può essere vista come un prodotto cartesiano seguito da una selezione:

```
R ⨝_condizione S  =  σ_condizione (R × S)
```

Esempio: voglio sapere il nome degli studenti e i titoli dei corsi a cui sono iscritti.

```
STUDENTE(idStudente, nome)
ISCRIZIONE(idStudente, idCorso)
CORSO(idCorso, titolo)
```

```
STUDENTE ⨝_STUDENTE.idStudente = ISCRIZIONE.idStudente ISCRIZIONE
```

Il risultato contiene le informazioni combinate delle due tabelle, solo per le righe in cui la condizione è soddisfatta.

**Join naturale**: se le due relazioni hanno attributi con lo stesso nome e stesso significato, si può usare la join naturale, che applica automaticamente la condizione di uguaglianza sugli attributi comuni:

```
STUDENTE ⨝ ISCRIZIONE
```

L'attributo comune (`idStudente`) compare una sola volta nel risultato.

Tipi di join nell'algebra relazionale:

| Tipo | Condizione |
|---|---|
| **Natural join** | uguaglianza sugli attributi con lo stesso nome |
| **Equi-join** | condizione di uguaglianza esplicita tra attributi |
| **Theta-join** | qualsiasi condizione di confronto |

::: {.callout-note}
## Join e SQL
In SQL troveremo `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`. Questi corrispondono a varianti della join che gestiscono in modo diverso le tuple senza corrispondenza. Le vedremo nel capitolo 10.
:::

---

### Differenza (−)

La **differenza** restituisce le tuple presenti in una relazione ma assenti in un'altra. Si applica solo a relazioni **compatibili**: stesso numero di attributi, stessi domini, stesso significato.

Notazione formale:

```
R − S
```

Esempio: voglio gli studenti che non hanno ancora consegnato il progetto.

```
STUDENTI_TUTTI(idStudente)  −  STUDENTI_CON_PROGETTO(idStudente)
```

| STUDENTI_TUTTI | | STUDENTI_CON_PROGETTO | | Risultato |
|---|---|---|---|---|
| 1 | | 1 | | 2 |
| 2 | | 3 | | |
| 3 | | | | |

La differenza è un'operazione insiemistica: l'ordine delle tuple non conta, e il risultato è sempre una relazione.

---

## Il collegamento con SQL

L'algebra relazionale non è un linguaggio che si usa direttamente: è la teoria che sta sotto SQL. Ogni query SQL può essere descritta come una combinazione di operazioni relazionali.

| Algebra relazionale | SQL equivalente |
|---|---|
| `σ_condizione(R)` | `SELECT * FROM R WHERE condizione` |
| `π_attributi(R)` | `SELECT attributi FROM R` |
| `R ⨝ S` | `SELECT ... FROM R JOIN S ON ...` |
| `R − S` | `SELECT ... EXCEPT SELECT ...` |

Capire l'algebra relazionale significa capire *perché* SQL funziona come funziona, non solo *come* si usa.

---

## Schema mentale

**Qual è la differenza tra selezione e proiezione?**
La selezione agisce sulle righe (le filtra in base a una condizione). La proiezione agisce sulle colonne (ne seleziona un sottoinsieme). Il risultato di entrambe è sempre una relazione.

**Perché il prodotto cartesiano è raramente utile da solo?**
Perché combina ogni riga con ogni altra, producendo molte tuple senza significato. Diventa utile quando viene filtrato con una condizione: in quel caso si ottiene una join.

**Cosa si intende per relazioni compatibili nella differenza?**
Che hanno lo stesso numero di attributi, gli stessi domini e lo stesso significato. Non si può fare la differenza tra una relazione di studenti e una di corsi.

**Perché la join è l'operazione più importante?**
Perché i dati in un database normalizzato sono distribuiti su più tabelle. La join è l'unico modo per ricostruire le informazioni collegate, senza duplicare i dati.

**Qual è il legame tra algebra relazionale e SQL?**
L'algebra relazionale è la teoria su cui SQL è fondato. Ogni query SQL corrisponde a una combinazione di operazioni relazionali. Capire l'algebra aiuta a capire perché SQL si comporta come si comporta.