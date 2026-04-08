# Il modello ER: dalla descrizione al diagramma

Nel capitolo precedente abbiamo introdotto i tre elementi fondamentali del modello ER: entità, attributi e relazioni. In questo capitolo approfondiremo i dettagli che contano in fase di progettazione reale, e vedremo come passare da una descrizione in linguaggio naturale a un diagramma ER completo.

---

## Le fasi della progettazione

Prima di disegnare qualsiasi diagramma, vale la pena avere chiaro il percorso completo di progettazione di un database. Si articola in quattro fasi.

**Analisi dei requisiti**
Si raccolgono le informazioni su cosa il database dovrà rappresentare e gestire. Di solito si parte da una descrizione in linguaggio naturale fornita dal committente. Questa fase non produce ancora nessun diagramma: produce una lista di oggetti, caratteristiche e relazioni da rappresentare.

**Modello concettuale (ER)**
Si costruisce il diagramma ER. È la traduzione della realtà in un linguaggio visivo e formale, ancora indipendente dal DBMS. L'obiettivo è che il diagramma sia comprensibile anche da chi non conosce l'informatica.

**Modello logico (relazionale)**
Il diagramma ER viene tradotto in tabelle relazionali. È qui che compaiono le chiavi primarie nel senso tecnico, le chiavi esterne, i tipi di dato. Lo vedremo nel capitolo 05.

**Modello fisico**
Si scelgono le ottimizzazioni specifiche per il DBMS usato: indici, partizionamento, struttura su disco. Lo affronteremo nel capitolo 11.

::: {.callout-note}
## Perché l'ordine conta
Saltare l'analisi dei requisiti e il modello concettuale per passare direttamente alle tabelle è l'errore più comune nella progettazione di database. Un errore concettuale scoperto in fase ER si corregge in pochi minuti. Lo stesso errore scoperto dopo aver scritto centinaia di query SQL può costare ore di lavoro.
:::

---

## Tipi di attributi

Nel capitolo precedente abbiamo visto gli attributi nella loro forma più semplice. In realtà ne esistono varianti che compaiono spesso nei diagrammi ER reali.

**Attributo semplice**
È indivisibile: non ha senso scomporlo ulteriormente. Esempi: `matricola`, `voto`, `prezzo`.

**Attributo composto**
Può essere scomposto in parti più piccole, ognuna con un significato proprio. Esempio: `indirizzo` può essere scomposto in `via`, `civico`, `CAP`, `città`. Nel diagramma ER, gli attributi composti hanno a loro volta delle ellissi figlie.

La scelta tra attributo semplice e composto dipende dall'uso: se il sistema dovrà mai cercare per `città` separatamente, conviene renderlo un attributo autonomo. Se `indirizzo` viene trattato sempre come blocco unico, può restare semplice.

**Attributo multivalore**
Un'entità può avere più valori per lo stesso attributo. Esempio: `Studente` può avere più `numeroDiTelefono`. Nel diagramma ER, gli attributi multivalore si indicano con una doppia ellisse.

::: {.callout-warning}
## Attributi multivalore e il modello relazionale
Gli attributi multivalore non si traducono direttamente in una colonna nel modello relazionale. Richiedono una tabella separata. Anticipiamo qui il problema: una colonna SQL non può contenere una lista di valori, perché questo violerebbe la prima forma normale (1NF), che vedremo nel capitolo 07.
:::

**Attributo derivato**
È un attributo calcolabile a partire da altri dati già presenti. Esempio: `età` è derivabile da `dataNascita` e dalla data corrente. Nel diagramma ER si indica con una ellisse tratteggiata. Di solito non viene memorizzato nel database: si calcola quando serve.

---

## Identificatori

Ogni entità deve avere un **identificatore**: un attributo (o un insieme di attributi) che permette di distinguere in modo univoco ogni istanza da tutte le altre.

L'identificatore nel modello ER corrisponderà alla chiave primaria nel modello logico.

Alcune regole pratiche:

- l'identificatore non deve cambiare nel tempo: il `nome` non è un buon identificatore perché due persone possono avere lo stesso nome, e un nome può cambiare
- per questo motivo si usano spesso identificatori artificiali: `matricola`, `codice`, `id` — valori generati dal sistema, senza significato reale ma stabili e univoci
- in rari casi l'identificatore è composto da più attributi: ad esempio, una tupla `(idStudente, idCorso)` può identificare un'iscrizione in modo univoco

---

## Leggere una descrizione testuale

La competenza più pratica nella progettazione ER è saper estrarre entità, attributi e relazioni da una descrizione in linguaggio naturale. Ecco alcune indicazioni operative.

**Le entità** sono spesso i sostantivi principali della descrizione: le cose di cui si vogliono memorizzare informazioni. Un buon test: "ci sono più esemplari di questa cosa?" e "ha caratteristiche proprie?". Se la risposta è sì a entrambe, è probabilmente un'entità.

**Gli attributi** sono le caratteristiche associate a un'entità: di solito aggettivi o complementi di specificazione nel testo.

**Le relazioni** sono spesso i verbi: *frequenta*, *insegna*, *effettua*, *appartiene a*.

**I moltiplicatori** si trovano nelle frasi come "ogni studente può iscriversi a più corsi" (N lato studente) e "ogni corso può avere più studenti" (N lato corso) → cardinalità N:M.

---

## Un esempio guidato

Partiamo da questa descrizione:

> Una biblioteca gestisce libri e soci. Ogni libro ha un codice ISBN, un titolo e un anno di pubblicazione. Ogni libro è scritto da uno o più autori. Ogni autore ha un nome e un cognome. I soci possono prendere in prestito i libri: ogni prestito registra la data di inizio e la data di restituzione prevista. Un socio può avere più prestiti attivi contemporaneamente, ma ogni copia fisica di un libro può essere in prestito a un solo socio per volta.

**Passo 1 — Identificare le entità**

I sostantivi principali: `Libro`, `Autore`, `Socio`. Anche `Prestito` emerge come entità perché ha attributi propri (`dataInizio`, `dataRestituzionePrevista`) e non è riducibile a un semplice collegamento.

**Passo 2 — Definire gli attributi e gli identificatori**

| Entità | Attributi | Identificatore |
|---|---|---|
| `Libro` | ISBN, titolo, annoPubblicazione | ISBN |
| `Autore` | nome, cognome | id (artificiale) |
| `Socio` | nome, cognome, tessera | tessera |
| `Prestito` | dataInizio, dataRestituzionePrevista | id (artificiale) |

**Passo 3 — Definire le relazioni e le cardinalità**

- `Libro` — *è scritto da* — `Autore`: N:M (un libro ha più autori, un autore scrive più libri)
- `Socio` — *effettua* — `Prestito`: 1:N (un socio può avere più prestiti, ogni prestito appartiene a un solo socio)
- `Prestito` — *riguarda* — `Libro`: N:1 (ogni prestito riguarda una copia di un libro, ma un libro può avere più prestiti nel tempo)

::: {.callout-tip}
## Una regola pratica sulla cardinalità
Quando si è incerti sulla cardinalità, conviene chiedersi: "nel caso più generale possibile, quante istanze di A possono essere collegate a una singola istanza di B?". Se la risposta è "più di una", la cardinalità è N su quel lato.
:::

---

## Errori comuni nella progettazione ER

**Mettere troppo in una sola entità**
Creare una tabella `Tutto` con decine di colonne è l'errore più comune. Se un gruppo di attributi riguarda una cosa distinta che ha senso autonomo, probabilmente è un'entità separata.

**Confondere relazione e attributo**
Se qualcosa ha attributi propri (come `dataInizio` nel prestito), non è un semplice collegamento: è un'entità o una relazione con attributi.

**Scegliere come identificatore un attributo instabile**
Il nome di una persona non è un buon identificatore: può cambiare, può non essere unico. Meglio usare un codice generato dal sistema.

**Ignorare la partecipazione**
Specificare solo la cardinalità massima (1:N, N:M) senza indicare se la partecipazione è obbligatoria o facoltativa porta a problemi nel modello logico. "Un corso deve avere almeno uno studente" è una regola diversa da "un corso può esistere anche senza studenti".

---

## Schema mentale

**Quali sono le quattro fasi della progettazione di un database?**
Analisi dei requisiti, modello concettuale (ER), modello logico (relazionale), modello fisico.

**Qual è la differenza tra attributo composto e attributo multivalore?**
L'attributo composto può essere scomposto in parti (indirizzo → via, città, CAP). L'attributo multivalore ha più valori per la stessa istanza (più numeri di telefono).

**Perché gli attributi derivati di solito non si memorizzano?**
Perché sono calcolabili da altri dati già presenti. Memorizzarli creerebbe ridondanza e rischio di incoerenza.

**Come si riconosce una relazione N:M in una descrizione testuale?**
Quando su entrambi i lati si può rispondere "più di uno": "uno studente frequenta più corsi" e "un corso è frequentato da più studenti".

**Perché un nome non è un buon identificatore?**
Perché può non essere univoco (due persone con lo stesso nome) e può cambiare nel tempo. Un identificatore deve essere stabile e univoco.