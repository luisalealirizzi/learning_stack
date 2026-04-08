# Normalizzazione

Avere le tabelle giuste non basta. È possibile costruire un database strutturalmente corretto — con chiavi primarie, chiavi esterne e vincoli — che produce comunque problemi quando i dati vengono modificati. La **normalizzazione** è il processo che elimina questi problemi alla radice.

---

## Il problema: la ridondanza

Un database non normalizzato può contenere dati corretti, ma comportarsi in modo sbagliato quando i dati cambiano. La causa quasi sempre è la stessa: la **ridondanza**, cioè la stessa informazione memorizzata in più posti.

Considera questa tabella:

| idIscrizione | nomeStudente | emailStudente | nomeCorso | aula |
|---|---|---|---|---|
| 1 | Luca Rossi | luca@scuola.it | Informatica | Lab3 |
| 2 | Luca Rossi | luca@scuola.it | Matematica | A12 |
| 3 | Anna Bianchi | anna@scuola.it | Informatica | Lab3 |

Il nome e l'email di Luca Rossi compaiono due volte. L'aula del corso di Informatica compare due volte. Questo crea tre tipi di **anomalie**.

**Anomalia di aggiornamento**: se Luca Rossi cambia email, bisogna aggiornarla in tutte le righe che lo riguardano. Se se ne aggiorna solo una, il database diventa incoerente.

**Anomalia di inserimento**: non si può registrare un corso senza avere almeno uno studente iscritto, perché le informazioni del corso non hanno dove stare da sole.

**Anomalia di cancellazione**: se Anna Bianchi si cancella dall'unica iscrizione a Informatica, si perde anche l'informazione che Informatica si tiene in Lab3.

::: {.callout-important}
## Anomalie e affidabilità
Le anomalie non sono errori visibili immediatamente. Il database continua a funzionare, ma i dati diventano progressivamente incoerenti. In un sistema usato da più persone, questo tipo di degrado può essere difficile da individuare e correggere.
:::

---

## Le dipendenze funzionali

Prima di applicare la normalizzazione, occorre capire le **dipendenze funzionali**: le regole che descrivono cosa determina cosa in una tabella.

Si scrive `A → B` e si legge *A determina B*. Significa: se due righe hanno lo stesso valore di A, devono avere lo stesso valore di B.

Esempio nella tabella `STUDENTE(idStudente, nome, cognome, comune, CAP)`:

```
idStudente → nome
idStudente → cognome
idStudente → comune
comune → CAP
```

L'ultima dipendenza, `comune → CAP`, dice che il CAP è determinato dal comune, non dallo studente. Questo è il segnale di un problema: `CAP` dipende da `comune`, che non è la chiave. Questa struttura genera ridondanza e anomalie.

::: {.callout-note}
## Quando una dipendenza è violata
La dipendenza `idStudente → nome` è violata se esistono due righe con lo stesso `idStudente` ma nomi diversi. In una tabella ben progettata, questo non dovrebbe mai accadere.
:::

---

## La normalizzazione: un percorso a tappe

La normalizzazione si applica attraverso una sequenza di forme normali, ognuna più restrittiva della precedente. Ogni forma normale elimina una specifica categoria di ridondanza.

---

### Prima Forma Normale (1NF)

Una tabella è in **1NF** se:

- ogni campo contiene un solo valore (valori atomici)
- non esistono liste o gruppi ripetuti in una colonna
- ogni riga è identificabile in modo univoco

**Esempio NON in 1NF:**

| idStudente | telefoni |
|---|---|
| 1 | 333123, 347456 |
| 2 | 320789 |

La colonna `telefoni` contiene una lista di valori. Non è atomica: NON è 1NF.

**Trasformazione in 1NF:**

| idStudente | telefono |
|---|---|
| 1 | 333123 |
| 1 | 347456 |
| 2 | 320789 |

Ogni cella contiene un solo valore. La tabella è in 1NF. L'attributo multivalore è diventato righe separate.

---

### Seconda Forma Normale (2NF)

Una tabella è in **2NF** se:

- è già in 1NF
- ogni attributo non chiave dipende **dall'intera chiave primaria**, non solo da una parte di essa

Il problema che la 2NF elimina si chiama **dipendenza parziale**: un attributo che dipende solo da una parte della chiave composta.

::: {.callout-note}
## Quando si applica la 2NF
La 2NF è rilevante solo quando la chiave primaria è **composta** (formata da più attributi). Se la chiave è semplice (un solo attributo), una tabella in 1NF è automaticamente in 2NF.
:::

**Esempio NON in 2NF:**

```
ISCRIZIONE(idStudente, idCorso, nomeStudente, nomeCorso)
Chiave primaria: (idStudente, idCorso)
```

Dipendenze:
```
idStudente → nomeStudente    ← dipendenza parziale
idCorso    → nomeCorso       ← dipendenza parziale
```

`nomeStudente` dipende solo da `idStudente`, non dall'intera chiave. Stessa cosa per `nomeCorso`. La tabella NON è in 2NF.

**Trasformazione in 2NF:**

```
STUDENTE(idStudente, nomeStudente)
CORSO(idCorso, nomeCorso)
ISCRIZIONE(idStudente FK, idCorso FK)
```

Ora ogni attributo dipende dall'intera chiave della propria tabella. La tabella è in 2NF.

---

### Terza Forma Normale (3NF)

Una tabella è in **3NF** se:

- è già in 2NF
- non esistono **dipendenze transitive** tra attributi non chiave

Una **dipendenza transitiva** esiste quando un attributo non chiave determina un altro attributo non chiave:

```
A → B  e  B → C  (con B non chiave)  →  A → C è transitiva
```

**Esempio NON in 3NF:**

```
STUDENTE(idStudente, nome, comune, CAP)
```

Dipendenze:
```
idStudente → comune
comune     → CAP
```

Quindi `idStudente → CAP` attraverso `comune`. Il `CAP` dipende dal `comune`, non direttamente dalla chiave. La tabella NON è in 3NF.

**Trasformazione in 3NF:**

```
STUDENTE(idStudente, nome, comune FK)
COMUNE(comune PK, CAP)
```

Ora ogni attributo dipende solo dalla chiave della propria tabella. La tabella è in 3NF.

---

### Forma Normale di Boyce-Codd (BCNF)

La **BCNF** è una versione più restrittiva della 3NF. Una tabella è in BCNF se:

- è già in 3NF
- per ogni dipendenza funzionale `A → B`, A è una chiave candidata

In pratica: solo le chiavi possono determinare altri attributi. La BCNF elimina ridondanze residue che la 3NF lascia in casi particolari con più chiavi candidate.

**Esempio NON in BCNF:**

```
ORARIO(professore, corso, aula)
```

Dipendenze:
```
(professore, corso) → aula    ← chiave primaria composta
corso → aula                  ← ma corso da solo determina aula
```

`corso` determina `aula`, ma `corso` da solo non è una chiave candidata. La tabella NON è in BCNF.

**Trasformazione in BCNF:**

```
CORSO_AULA(corso PK, aula)
INSEGNA(professore, corso FK)
```

Ora ogni determinante è una chiave candidata. La tabella è in BCNF.

::: {.callout-tip}
## La 3NF è spesso sufficiente
In contesti reali, si arriva generalmente fino alla 3NF. La BCNF è più restrittiva ma può rendere alcune query più complesse e rendere impossibile preservare tutte le dipendenze funzionali nello schema. Non è sempre necessario spingersi fino ad essa.
:::

---

## Il quadro d'insieme

Ogni forma normale risolve un problema specifico:

| Forma normale | Problema eliminato |
|---|---|
| **1NF** | valori non atomici, liste in una cella |
| **2NF** | dipendenze parziali dalla chiave composta |
| **3NF** | dipendenze transitive tra attributi non chiave |
| **BCNF** | determinanti che non sono chiavi candidate |

Il percorso è sempre cumulativo: per essere in 3NF bisogna essere già in 2NF, che richiede di essere già in 1NF.

---

## Schema mentale

**Cos'è la ridondanza e perché è pericolosa?**
È la stessa informazione memorizzata in più posti. Genera anomalie di aggiornamento, inserimento e cancellazione: il database diventa incoerente senza che nessuno se ne accorga subito.

**Cosa significa `A → B`?**
Che A determina B: se due righe hanno lo stesso valore di A, devono avere lo stesso valore di B.

**Quando la 2NF non è rilevante?**
Quando la chiave primaria è semplice (un solo attributo). In quel caso una tabella in 1NF è automaticamente in 2NF.

**Qual è la differenza tra dipendenza parziale e dipendenza transitiva?**
La dipendenza parziale riguarda la chiave: un attributo dipende solo da una parte della chiave composta. La dipendenza transitiva riguarda attributi non chiave: un attributo non chiave ne determina un altro.

**Perché non si normalizza sempre fino alla BCNF?**
Perché può rendere alcune query più complesse e non sempre è necessaria. La 3NF è sufficiente nella maggior parte dei casi reali.