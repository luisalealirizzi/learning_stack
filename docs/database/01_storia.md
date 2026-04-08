# Archivi, dati e società

Prima di scrivere una sola riga di SQL, vale la pena chiedersi da dove vengono i database. Non sono nati per i siti web o per le app. Sono nati per risolvere un problema molto più antico: come si conserva, si organizza e si recupera la memoria di un'istituzione.

---

## Prima dei computer: archivi fisici

Per secoli, la memoria delle organizzazioni è stata fatta di carta. Registri, schedari, archivi a cassetti. Una biblioteca conservava migliaia di schede cartacee, una per ogni libro. Un ospedale teneva cartelle cliniche in faldoni ordinati per cognome. Una banca registrava i conti su libri mastri.

Questi sistemi funzionavano, ma avevano limiti chiari:

- cercare un dato richiedeva tempo e personale
- aggiornare le informazioni significava correggere a mano ogni copia
- perdere un archivio fisico significava perdere la memoria di anni di attività

Quando arrivano i computer negli anni '50, la prima cosa che si prova a fare è trasferire questi archivi su supporti digitali. Non per inventare qualcosa di nuovo, ma per fare la stessa cosa più velocemente.

::: {.callout-note}
## Da dove vengono i primi database
I primi grandi archivi digitali non nascono per scopi scientifici o commerciali generici. Nascono per gestire dati pubblici e industriali reali: anagrafi, ferrovie, compagnie aeree, eserciti, banche, sistemi fiscali.
:::

---

## Il sistema SABRE: quando i dati diventano infrastruttura

Un esempio storico chiarisce meglio di ogni definizione astratta cosa significa gestire dati su scala reale.

Negli anni '50, American Airlines gestisce le prenotazioni dei voli con telefonate, fogli di carta e tabelle scritte a mano. Con l'esplosione dei voli commerciali, questo sistema sta collassando: troppi dati, troppi errori, tempi troppo lunghi.

Nel 1959, American Airlines e IBM iniziano a sviluppare insieme **SABRE** (*Semi-Automatic Business Research Environment*): uno dei primi sistemi di gestione dati in tempo reale della storia.

SABRE deve essere in grado di:

- memorizzare lo stato di tutti i posti su tutti i voli
- aggiornare i dati in tempo reale quando qualcuno prenota o cancella
- rispondere alle richieste di più agenti contemporaneamente, senza errori
- garantire che due agenti non vendano lo stesso posto allo stesso passeggero

Questi non sono problemi banali. Sono esattamente i problemi che i moderni sistemi di database risolvono ancora oggi: **coerenza, concorrenza, affidabilità, recupero dopo errori**.

::: {.callout-tip}
## Il punto chiave
SABRE non nasce per "fare tecnologia". Nasce per rispondere a un'esigenza concreta: gestire milioni di prenotazioni senza errori e senza perdere dati. Questa è ancora oggi la ragione per cui esistono i database.
:::

---

## Da archivio a modello: Edgar F. Codd e le regole del modello relazionale

Negli anni '60 e '70, i database esistono, ma ognuno funziona in modo diverso. I dati sono organizzati in strutture fisiche (gerarchie, reti), e chi interroga il database deve sapere esattamente come i dati sono memorizzati su disco. Cambiare la struttura fisica significa riscrivere tutti i programmi che usano quei dati.

Nel 1970, Edgar F. Codd, un ricercatore di IBM, pubblica un articolo che cambia tutto: *A Relational Model of Data for Large Shared Data Banks*.

L'idea centrale è semplice ma radicale:

> I dati devono essere rappresentati come **relazioni matematiche** (tabelle), e devono poter essere interrogati con operazioni formali, indipendentemente da come sono fisicamente memorizzati.

Questo significa separare due cose che prima erano mescolate:

| Prima di Codd | Dopo Codd |
|---|---|
| L'utente deve sapere come i dati sono salvati su disco | L'utente descrive *cosa* vuole, non *come* ottenerlo |
| Cambiare la struttura fisica rompe i programmi | La struttura logica è indipendente da quella fisica |
| Ogni DBMS ha il suo linguaggio proprietario | Si può definire un linguaggio standard (SQL) |

Codd formalizza queste idee nelle cosiddette **12 regole di Codd**, un insieme di criteri che un sistema deve rispettare per essere considerato un vero database relazionale. In pratica, quasi tutti i DBMS moderni le rispettano almeno in parte.

::: {.callout-important}
## L'idea che cambia tutto
Codd non inventa solo una tecnica. Inventa un modo di pensare ai dati. Separare il *modello logico* (come i dati sono organizzati) dalla *struttura fisica* (come sono salvati su disco) è ancora oggi il fondamento di ogni database relazionale.
:::

---

## Cos'è un database

Un **database** è una raccolta organizzata di dati, strutturata in modo da permettere l'inserimento, la ricerca, la modifica e la cancellazione delle informazioni in modo efficiente e affidabile.

La parola chiave è *organizzata*. Un file di testo con migliaia di righe contiene dati, ma non è un database. Un foglio Excel può contenere dati strutturati, ma non è un database. Un database è una struttura progettata esplicitamente per:

- rappresentare la realtà in modo coerente
- rispondere a domande (query) sui dati
- garantire che i dati rimangano corretti nel tempo
- permettere a più utenti di lavorare contemporaneamente

::: {.callout-note}
## Database e DBMS non sono la stessa cosa
Il **database** è l'insieme dei dati. Il **DBMS** (*DataBase Management System*) è il software che gestisce il database: controlla gli accessi, esegue le query, garantisce l'integrità dei dati. MySQL, PostgreSQL, MariaDB, SQLite sono DBMS, non database.
:::

---

## Il ruolo sociale dei dati

Un database non è solo uno strumento tecnico. È una struttura che decide come la realtà viene rappresentata, organizzata e interrogata.

Ogni scuola conserva i voti degli studenti in un database. Ogni ospedale archivia le cartelle cliniche. Ogni comune gestisce l'anagrafe. Ogni banca registra i movimenti dei conti. Questi dati non sono solo numeri: sono la memoria ufficiale di istituzioni che hanno effetti reali sulla vita delle persone.

Questo significa che chi progetta un database fa scelte che non sono solo tecniche:

- quali dati raccogliere e quali no
- chi può accedere a quali informazioni
- per quanto tempo i dati vengono conservati
- cosa succede se i dati sono sbagliati o vengono persi

::: {.callout-warning}
## Dati sbagliati, conseguenze reali
Se il database di una scuola registra un voto in modo errato, uno studente può perdere un'opportunità. Se il database di un ospedale contiene un'allergia non aggiornata, le conseguenze possono essere gravi. La correttezza dei dati non è un problema solo tecnico.
:::

Torneremo su questi temi nel capitolo dedicato a dati, diritti e responsabilità. Per ora, tieni presente che ogni scelta tecnica nella progettazione di un database ha una dimensione etica e sociale.

---

## Schema mentale

Prima di proseguire, fermati un momento e rispondi mentalmente a queste domande.

**Perché i database non nascono per i siti web?**
Perché i problemi di gestione dei dati su scala esistevano decenni prima di internet. I primi sistemi nascono per gestire prenotazioni, anagrafi, conti bancari.

**Cosa cambia con il modello relazionale di Codd?**
Si separa il modello logico dei dati dalla struttura fisica. L'utente descrive cosa vuole, non come i dati sono memorizzati.

**Qual è la differenza tra database e DBMS?**
Il database è l'insieme dei dati. Il DBMS è il software che lo gestisce.

**Perché la progettazione di un database ha una dimensione etica?**
Perché i dati rappresentano persone reali e hanno effetti reali. Chi progetta decide cosa si raccoglie, chi può accedere e per quanto tempo.