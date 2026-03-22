# I simboli dei diagrammi di flusso

Introduciamo i **simboli standard** dei diagrammi di flusso.

Questi simboli servono per rappresentare in modo chiaro e universale:

- l’inizio e la fine
- le azioni
- le decisioni
- il flusso di esecuzione


## I simboli principali

### 1. Inizio / Fine

Indica il punto di partenza o di arrivo dell’algoritmo.

In Mermaid si rappresenta con una forma arrotondata:

```mermaid
flowchart TD
    A([INIZIO])
    B([FINE])
```

2. Azione (processo)

Rappresenta un’operazione da eseguire.

Esempi:

apri un’app
muovi un personaggio
calcola un valore

In Mermaid è un rettangolo:

flowchart TD
    A[Apri il gioco]
3. Decisione

Rappresenta una scelta, cioè una domanda con due possibili risposte.

Esempi:

il semaforo è verde?
il gioco si avvia?
il nemico è presente?

In Mermaid è un rombo:

4. Frecce (flusso)

Le frecce collegano i blocchi e indicano l’ordine delle operazioni.

Un esempio completo

Mettiamo insieme tutti i simboli.

Situazione:

Se il semaforo è verde attraversa, altrimenti aspetta.

Un esempio con videogioco

Situazione:

Avvia un gioco. Se parte, premi start. Altrimenti mostra errore.

Cosa ricordare

Ogni diagramma deve avere:

un solo punto di inizio
almeno un punto di fine
frecce chiare e ordinate
decisioni con due uscite (Sì / No)
Attenzione agli errori

Errori comuni:

frecce senza direzione chiara
decisioni con una sola uscita
blocchi scollegati
più inizi senza senso

Un diagramma deve poter essere “seguito” passo dopo passo senza dubbi.

In sintesi

I diagrammi di flusso usano simboli standard per rappresentare:

l’inizio e la fine
le azioni
le decisioni
il flusso

Ora che conosci i simboli, possiamo usarli per rappresentare qualsiasi algoritmo in modo chiaro e visivo.