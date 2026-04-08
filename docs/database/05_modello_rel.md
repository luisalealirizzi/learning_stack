# Dal modello ER al modello relazionale

Il modello ER descrive la realtà in modo concettuale: entità, attributi, relazioni. Il modello relazionale traduce quella descrizione in una struttura che un DBMS può gestire concretamente: tabelle, colonne, chiavi. Questo capitolo copre il passaggio tra i due livelli.

---

## La tabella relazionale

L'unità fondamentale del modello relazionale è la **tabella**, chiamata anche **relazione** nella teoria formale.

Una tabella è una struttura che rappresenta un insieme di elementi dello stesso tipo. È composta da:

- **colonne** (o attributi): descrivono le proprietà degli elementi
- **righe** (o tuple): rappresentano i singoli elementi concreti
- un **dominio** per ogni colonna: indica i valori ammissibili (numero intero, testo, data...)
- una **chiave primaria**: identifica in modo univoco ogni riga

Esempio:

| idStudente (PK) | nome | cognome | dataNascita |
|---|---|---|---|
| 1 | Luca | Rossi | 2006-03-14 |
| 2 | Anna | Bianchi | 2005-11-27 |

Ogni riga è uno studente preciso. Ogni colonna è una caratteristica comune a tutti gli studenti. La chiave primaria `idStudente` garantisce che non esistano due righe identiche e che ogni studente sia identificabile con certezza.

::: {.callout-note}
## Tabella e relazione: stesso concetto, termini diversi
Nella teoria formale si usa il termine *relazione*. Nei DBMS e nella pratica si usa *tabella*. Il concetto è lo stesso. In questo corso useremo prevalentemente *tabella*, ricorrendo a *relazione* solo quando si parla di algebra relazionale o di teoria.
:::

---

## Schema e istanza

Nel modello relazionale occorre distinguere sempre tra due livelli.

Lo **schema** è la struttura del database: i nomi delle tabelle, le colonne, i tipi di dato, le chiavi e i vincoli. Lo schema cambia raramente: viene definito in fase di progettazione e modificato solo quando cambiano i requisiti del sistema.

L'**istanza** è il contenuto reale del database in un dato momento: le righe presenti nelle tabelle, i valori concreti dei campi. L'istanza cambia continuamente a ogni inserimento, modifica o cancellazione.

Esempio con la tabella `STUDENTE`:

Schema:
```
STUDENTE(idStudente INT, nome VARCHAR(50), cognome VARCHAR(50), dataNascita DATE)
```

Istanza (in un certo momento):
```
(1, 'Luca', 'Rossi', '2006-03-14')
(2, 'Anna', 'Bianchi', '2005-11-27')
```

Domani potrebbe esserci una terza riga, o la seconda potrebbe essere stata modificata. Lo schema resta invariato.

---

## Domini e tipi di dato

Ogni colonna di una tabella deve avere un **tipo di dato**, che definisce quali valori possono essere memorizzati in quel campo e come il DBMS li gestisce.

I tipi principali in MySQL/MariaDB:

| Tipo | Utilizzo tipico |
|---|---|
| `INT` | numeri interi (id, anni, quantità) |
| `DECIMAL(a,b)` | numeri con virgola precisi (prezzi, voti) |
| `CHAR(n)` | testo a lunghezza fissa (CAP, codici provincia) |
| `VARCHAR(n)` | testo a lunghezza variabile (nomi, descrizioni) |
| `DATE` | date nel formato YYYY-MM-DD |
| `DATETIME` | data e ora |
| `BOOLEAN` | vero/falso |

Due distinzioni che tornano spesso nelle domande:

**`CHAR` vs `VARCHAR`**: `CHAR(n)` occupa sempre esattamente n caratteri, anche se il valore è più corto. `VARCHAR(n)` occupa solo lo spazio necessario, fino a un massimo di n caratteri. `CHAR` è preferibile per campi sempre della stessa lunghezza (CAP: sempre 5 caratteri). `VARCHAR` è preferibile per campi di lunghezza variabile (cognome: da 2 a 50 caratteri).

**`DECIMAL` vs `FLOAT`**: per valori che devono essere precisi, come prezzi o importi bancari, si usa `DECIMAL(a,b)` dove `a` è il numero totale di cifre e `b` quante sono dopo la virgola. `FLOAT` può introdurre piccoli errori di arrotondamento, inaccettabili in contesti finanziari.

::: {.callout-tip}
## Scegliere il tipo giusto
Il tipo di dato non è solo una formalità tecnica. Scegliere `INT` per un campo che potrebbe contenere lettere, o `FLOAT` per un prezzo, sono errori progettuali che possono causare problemi difficili da correggere in seguito. Il tipo di dato è parte della progettazione, non una scelta da fare in fretta.
:::

---

## Le regole di traduzione dall'ER al relazionale

Il passaggio dal modello ER al modello logico segue regole precise e applicabili in modo sistematico.

### Regola 1 — Ogni entità diventa una tabella

Gli attributi dell'entità diventano le colonne della tabella. L'identificatore dell'entità diventa la chiave primaria.

```
Entità: STUDENTE (matricola, nome, cognome)
→ Tabella: STUDENTE(matricola PK, nome, cognome)
```

### Regola 2 — Le relazioni 1:N diventano chiavi esterne

Nella relazione 1:N, la chiave primaria dell'entità sul lato "1" viene aggiunta come colonna (chiave esterna) nella tabella dell'entità sul lato "N".

Esempio: `CORSO` *è tenuto da* `PROFESSORE`, cardinalità N:1 (molti corsi per un professore).

```
PROFESSORE(idProfessore PK, nome, cognome)
CORSO(idCorso PK, titolo, idProfessore FK)
```

La colonna `idProfessore` in `CORSO` è una chiave esterna che punta a `PROFESSORE`. Il DBMS garantisce che non si possa inserire un corso con un `idProfessore` che non esiste in `PROFESSORE`.

### Regola 3 — Le relazioni N:M diventano tabelle ponte

Una relazione N:M non può essere rappresentata con una semplice chiave esterna. Serve una **tabella ponte**: una nuova tabella che contiene le chiavi primarie delle due entità collegate.

Esempio: `STUDENTE` *frequenta* `CORSO`, cardinalità N:M.

```
STUDENTE(idStudente PK, nome, cognome)
CORSO(idCorso PK, titolo)
FREQUENTA(idStudente FK, idCorso FK)
```

La tabella `FREQUENTA` è la tabella ponte. La sua chiave primaria è composta dall'unione delle due chiavi esterne: `(idStudente, idCorso)`. Se la relazione ha attributi propri (per esempio `annoScolastico`), questi diventano colonne aggiuntive della tabella ponte.

### Regola 4 — Le relazioni ternarie diventano tabelle ponte a tre chiavi

Una relazione ternaria si traduce in una tabella che contiene le chiavi primarie di tutte e tre le entità coinvolte.

Esempio: `STUDENTE` *sostiene* `ESAME` con `DOCENTE`.

```
STUDENTE(idStudente PK, nome)
ESAME(idEsame PK, materia)
DOCENTE(idDocente PK, nome)
SOSTIENE(idStudente FK, idEsame FK, idDocente FK, data, voto)
```

La chiave primaria di `SOSTIENE` è `(idStudente, idEsame, idDocente)`.

---

## Un esempio completo

Riprendiamo il modello ER della biblioteca dal capitolo precedente e traduciamolo in tabelle.

**Modello ER (riepilogo):**
- Entità: `Libro` (ISBN, titolo, annoPubblicazione), `Autore` (id, nome, cognome), `Socio` (tessera, nome, cognome), `Prestito` (id, dataInizio, dataRestituzionePrevista)
- Relazioni: `Libro` — *scritto da* — `Autore` (N:M), `Socio` — *effettua* — `Prestito` (1:N), `Prestito` — *riguarda* — `Libro` (N:1)

**Modello logico:**

```
LIBRO(ISBN PK, titolo, annoPubblicazione)

AUTORE(idAutore PK, nome, cognome)

SCRIVE(ISBN FK, idAutore FK)          ← tabella ponte per N:M

SOCIO(tessera PK, nome, cognome)

PRESTITO(idPrestito PK, dataInizio, dataRestituzionePrevista,
         tessera FK, ISBN FK)
```

::: {.callout-important}
## La relazione N:M richiede sempre una tabella ponte
Non si può mettere una lista di ID in una singola colonna. Il modello relazionale non lo permette: ogni cella deve contenere un solo valore (questo è il principio della prima forma normale, che vedremo nel capitolo 07). La tabella ponte è l'unico modo corretto per rappresentare una relazione N:M.
:::

---

## Schema mentale

**Qual è la differenza tra schema e istanza?**
Lo schema è la struttura (tabelle, colonne, tipi, vincoli): cambia raramente. L'istanza è il contenuto reale in un dato momento: cambia a ogni operazione sui dati.

**Quando si usa `CHAR` e quando `VARCHAR`?**
`CHAR(n)` per valori sempre della stessa lunghezza (CAP, codici). `VARCHAR(n)` per testo di lunghezza variabile (nomi, descrizioni).

**Come si traduce una relazione 1:N nel modello logico?**
La chiave primaria dell'entità sul lato "1" diventa chiave esterna nella tabella dell'entità sul lato "N".

**Perché le relazioni N:M richiedono una tabella ponte?**
Perché non si può memorizzare una lista di valori in una singola colonna. La tabella ponte è l'unica rappresentazione corretta nel modello relazionale.

**Cosa contiene la tabella ponte di una relazione ternaria?**
Le chiavi primarie di tutte e tre le entità coinvolte, più gli eventuali attributi propri della relazione.