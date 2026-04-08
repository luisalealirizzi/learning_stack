# Il modello ER: progettare prima di costruire

Prima di creare una sola tabella, prima di scrivere una sola riga di SQL, bisogna capire cosa si vuole rappresentare. Il **modello ER** è lo strumento che permette di farlo.

---

## Perché serve un modello concettuale

Quando si progetta un database per una scuola, un ospedale o un e-commerce, il primo problema non è tecnico: è concettuale. Bisogna rispondere a domande come: quali informazioni contano? Come sono collegate tra loro? Cosa cambia nel tempo e cosa rimane stabile?

Rispondere direttamente in SQL sarebbe come costruire una casa senza planimetria. Si può fare, ma il risultato sarà quasi certamente sbagliato — e difficile da correggere.

Il **modello ER** (*Entity-Relationship*) è la planimetria del database. Descrive la realtà che il sistema dovrà rappresentare, usando un linguaggio visivo e concettuale, indipendente dal DBMS che si userà poi.

::: {.callout-note}
## Peter Chen e la nascita del modello ER
Nel 1976, l'informatico Peter Chen pubblica un articolo destinato a cambiare per sempre il modo di progettare i database: *The Entity–Relationship Model — Toward a Unified View of Data*. Chen capisce che prima delle tabelle servono parole, e prima delle colonne servono concetti. Il modello ER introduce una grammatica condivisa per descrivere la realtà: entità, attributi, relazioni.
:::

---

## I tre elementi fondamentali

Il modello ER si costruisce con tre elementi: entità, attributi e relazioni.

### Entità

Un'**entità** è una "cosa" del mondo reale che il sistema vuole rappresentare. Deve avere un'esistenza autonoma e caratteristiche proprie.

Esempi: `Studente`, `Corso`, `Professore`, `Prodotto`, `Ordine`.

Come riconoscere un'entità in una descrizione testuale: di solito è un sostantivo che indica qualcosa di cui esistono più esemplari e di cui si vogliono memorizzare informazioni.

Attenzione alla distinzione tra **entità** e **istanza**: `Studente` è l'entità (il tipo), mentre *Mario Rossi, matricola 12345* è una sua istanza (un elemento concreto).

Nel diagramma ER, le entità si rappresentano con un **rettangolo**.

### Attributi

Gli **attributi** sono le caratteristiche che descrivono un'entità. Ogni attributo ha un **dominio**, cioè l'insieme dei valori che può assumere.

Esempio: l'entità `Studente` può avere gli attributi `matricola`, `nome`, `cognome`, `dataNascita`. Il dominio di `dataNascita` è l'insieme delle date valide.

Tra gli attributi, uno (o un insieme) viene scelto come **identificatore**: è l'attributo che permette di distinguere un'istanza da tutte le altre. La `matricola` identifica ogni studente in modo univoco.

Nel diagramma ER, gli attributi si rappresentano con un **ellisse** collegata all'entità.

### Relazioni

Una **relazione** è il legame logico che collega due o più entità. Non rappresenta una cosa, ma un fatto che riguarda più entità.

Esempi:
- `Studente` — *frequenta* — `Corso`
- `Libro` — *è scritto da* — `Autore`
- `Cliente` — *effettua* — `Ordine`

Una relazione può avere attributi propri, se il fatto che descrive ha caratteristiche proprie. Ad esempio, la relazione *frequenta* potrebbe avere l'attributo `annoScolastico`.

Nel diagramma ER, le relazioni si rappresentano con un **rombo**.

---

## Cardinalità e partecipazione

Non basta sapere che due entità sono collegate: bisogna specificare *quante* istanze di ciascuna entità possono partecipare alla relazione. Questo si chiama **cardinalità**.

Le cardinalità principali sono tre:

| Cardinalità | Significato | Esempio |
|---|---|---|
| **1:1** | ogni istanza di A è collegata a una sola di B, e viceversa | ogni studente ha un solo libretto, ogni libretto appartiene a uno studente |
| **1:N** | ogni istanza di A è collegata a molte di B, ma ogni B è collegata a una sola A | un corso ha molti studenti, ogni studente appartiene a una sola classe |
| **N:M** | ogni istanza di A è collegata a molte di B, e viceversa | uno studente frequenta molti corsi, un corso è frequentato da molti studenti |

Accanto alla cardinalità, si specifica anche la **partecipazione**: indica se la presenza in una relazione è obbligatoria o facoltativa.

La cardinalità completa si scrive nella forma `min..max`:

- `0..1` — partecipazione facoltativa, al massimo una
- `1..1` — partecipazione obbligatoria, esattamente una
- `0..N` — partecipazione facoltativa, molte possibili
- `1..N` — partecipazione obbligatoria, almeno una

::: {.callout-tip}
## Come leggere la cardinalità
Nella relazione `Studente — frequenta — Corso`, se la cardinalità è `Studente: 0..N` e `Corso: 1..N`, significa: uno studente può anche non frequentare alcun corso (0 è il minimo), ma un corso deve avere almeno uno studente (1 è il minimo).
:::

---

## Un esempio completo

Supponiamo di dover progettare il database di una scuola. La descrizione iniziale è:

> La scuola ha studenti e corsi. Ogni studente può essere iscritto a più corsi. Ogni corso è tenuto da un professore. Ogni studente ha una matricola, un nome e un cognome. Ogni corso ha un codice e un titolo.

Dal testo si estraggono:

**Entità:** `Studente`, `Corso`, `Professore`

**Attributi:**
- `Studente`: matricola *(identificatore)*, nome, cognome
- `Corso`: codice *(identificatore)*, titolo
- `Professore`: id *(identificatore)*, nome, cognome

**Relazioni:**
- `Studente` — *è iscritto a* — `Corso` → cardinalità N:M (uno studente si iscrive a molti corsi, un corso ha molti studenti)
- `Corso` — *è tenuto da* — `Professore` → cardinalità N:1 (un corso ha un professore, un professore può tenere più corsi)

::: {.callout-warning}
## Il modello ER non è ancora il database
Il diagramma ER descrive la realtà in modo concettuale. Non contiene ancora tabelle, chiavi primarie nel senso SQL del termine, o tipi di dato. La traduzione in struttura relazionale avviene nel passo successivo: il modello logico, che vedremo nel capitolo 05.
:::

---

## Relazioni ternarie

La maggior parte delle relazioni collega due entità (relazioni binarie). In alcuni casi, però, un fatto riguarda tre entità contemporaneamente: si parla allora di **relazione ternaria**.

Una relazione ternaria non può essere spezzata in due relazioni binarie senza perdere significato.

Esempio: uno studente sostiene un esame con uno specifico docente. La relazione *sostiene* collega `Studente`, `Esame` e `Docente` contemporaneamente. Sapere che uno studente ha sostenuto un esame, e che un docente ha tenuto quell'esame, non dice ancora *chi ha valutato chi*: serve la relazione ternaria.

Nel modello logico, una relazione ternaria diventa una tabella con le chiavi delle tre entità coinvolte — lo vedremo nel capitolo 05.

---

## Schema mentale

**Cos'è un'entità e come si riconosce?**
È una "cosa" del mondo reale con esistenza autonoma e caratteristiche proprie. Di solito è un sostantivo di cui esistono più esemplari (Studente, Corso, Prodotto).

**Qual è la differenza tra entità e istanza?**
L'entità è il tipo (Studente), l'istanza è l'elemento concreto (Mario Rossi, matricola 12345).

**Cosa indica la cardinalità N:M?**
Che ogni istanza di un'entità può essere collegata a molte istanze dell'altra, e viceversa. Richiede una gestione speciale nel modello logico.

**Cosa significa partecipazione obbligatoria?**
Che il valore minimo della cardinalità è 1: ogni istanza deve partecipare alla relazione. Se il minimo è 0, la partecipazione è facoltativa.

**Perché si usa il modello ER prima di costruire le tabelle?**
Perché permette di capire e rappresentare la realtà in modo condiviso, prima di fare scelte tecniche. Un errore concettuale nel modello ER si propaga in tutto il database.