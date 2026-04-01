# Paradigmi e oggetti

---

## Un nuovo modo di pensare

Passare dal C a Java non è solo imparare una nuova sintassi. È un **cambio di mentalità**.

Nel C il focus è sulla macchina:

> *"Cosa deve fare il computer, passo dopo passo?"*

Nella programmazione orientata agli oggetti (OOP) il focus si sposta sul problema:

> *"Quali entità esistono nel mio problema? Come interagiscono?"*

Un programma non è più una lista di comandi, ma un modo per **modellare la realtà**.

---

## Il paradigma imperativo

Un programma imperativo è una **sequenza di istruzioni** che modificano lo stato della memoria. Il C che hai usato finora funziona così:

```c
// Gestione di un personaggio — approccio imperativo in C
char nome[] = "Kratos";
int vita = 100;
int attacco = 25;
int livello = 1;

vita -= 30;    // subisce danno
livello++;     // sale di livello
attacco += 5;  // l'attacco aumenta

printf("%s - Vita: %d, Livello: %d\n", nome, vita, livello);
```

Il programma dice alla macchina **esattamente cosa fare**, in quale ordine, su quali variabili.

::: {.callout-note}
## Focus del paradigma imperativo
Il programmatore pensa in termini di **stato** (i valori in memoria) e **trasformazioni** (le operazioni che li modificano). Il computer esegue tutto fedelmente, passo dopo passo.
:::

---

## I limiti del paradigma imperativo

Su programmi piccoli tutto funziona. Ma appena la complessità cresce, emergono problemi seri.

**Complessità difficile da gestire.** Con decine di personaggi, nemici, oggetti e livelli, capire "chi modifica cosa" diventa un incubo.

**Poco riuso del codice.** Le funzioni sono spesso scritte per un caso specifico e difficili da riutilizzare.

**Manutenzione complicata.** Cambiare una parte del codice può romperne un'altra in modo imprevedibile.

Il risultato è lo **spaghetti code**: variabili globali accessibili da chiunque, funzioni sparse, nessuna organizzazione.

::: {.callout-warning}
## Il problema dello spaghetti code
```c
int vita_giocatore = 100;
int vita_nemico = 80;
int punteggio = 0;

void attacca() {
    vita_nemico -= 25;    // chiunque può modificare vita_nemico!
    punteggio += 10;
}

void subisci_danno() {
    vita_giocatore -= 30; // nessun controllo: vita può diventare negativa
}
```
Niente impedisce a una funzione qualsiasi di portare `vita_giocatore` a -500. Con decine di variabili globali il codice diventa fragile e impossibile da mantenere.
:::

---

## Il paradigma orientato agli oggetti

L'OOP propone una soluzione diversa: modellare il software come un insieme di **oggetti** che interagiscono tra loro.

Ogni oggetto:

- rappresenta un'**entità** del problema reale
- ha dei **dati** che lo descrivono — chiamati *attributi*
- sa compiere certe **azioni** — chiamate *metodi*
- comunica con gli altri oggetti scambiandosi **messaggi**

::: {.callout-tip}
## Un'analogia utile
Pensa a un videogioco multiplayer come un **mondo abitato da oggetti**: ogni giocatore, ogni nemico, ogni arma, ogni livello è un oggetto. Ognuno ha le sue caratteristiche e sa fare certe cose. Interagiscono tra loro, ma ognuno gestisce il proprio stato in autonomia.
:::

---

## I tre pilastri dell'OOP

L'OOP si regge su tre concetti fondamentali che esploreremo nelle prossime lezioni:

::: {.callout-note}
## 1 — Incapsulamento
Dati e metodi sono racchiusi dentro l'oggetto. I dettagli interni sono nascosti; si interagisce solo attraverso l'interfaccia pubblica.
:::

::: {.callout-note}
## 2 — Ereditarietà
Una classe può derivare da un'altra, ereditandone attributi e metodi. Permette di riusare il codice e organizzare le classi in gerarchie.
:::

::: {.callout-note}
## 3 — Polimorfismo
La stessa azione può comportarsi in modo diverso a seconda dell'oggetto che la esegue. Stessa interfaccia, comportamenti differenti.
:::

---

## Classi e oggetti

Gli oggetti non nascono dal nulla: vengono creati a partire da uno **schema** chiamato **classe**.

Una classe è come la **scheda tipo di un personaggio** in un GDR: descrive quali statistiche ha ogni personaggio di quel tipo e cosa sa fare. La scheda non è il personaggio — è il modello da cui si creano tanti personaggi diversi, ognuno con i propri valori.

| | Descrizione | Esempio |
|---|---|---|
| **Classe** | Il modello, lo schema generale | `Personaggio` |
| **Oggetto** | La realizzazione concreta | `eroe`, `nemico`, `boss` |

::: {.callout-note}
## Definizione
Una **classe** è un modello che descrive attributi e metodi di un tipo di oggetto.
Un **oggetto** è un'istanza concreta di una classe, con i propri valori.
:::

---

## Il primo oggetto in Java

Vediamo subito come si traduce tutto questo in codice. Modelliamo un personaggio di un videogioco.

**Cosa descrive un personaggio?**
`nome`, `vita`, `attacco`, `livello`

**Cosa sa fare un personaggio?**
Subire danno, attaccare, salire di livello.

```java
public class Personaggio {

    // Attributi — descrivono lo stato dell'oggetto
    String nome;
    int vita;
    int attacco;
    int livello;

    // Metodi — descrivono il comportamento
    void subirDanno(int danno) {
        vita -= danno;
        if (vita < 0) vita = 0;
        System.out.println(nome + " subisce " + danno +
            " danni. Vita rimasta: " + vita);
    }

    void attacca(Personaggio bersaglio) {
        System.out.println(nome + " attacca " + bersaglio.nome + "!");
        bersaglio.subirDanno(attacco);
    }

    void salaDiLivello() {
        livello++;
        attacco += 5;
        vita += 20;
        System.out.println(nome + " sale al livello " + livello + "!");
    }
}
```

---

## Creare e usare oggetti

Per creare un oggetto si usa la parola chiave `new`:

```java
Personaggio eroe = new Personaggio();
eroe.nome = "Kratos";
eroe.vita = 100;
eroe.attacco = 25;
eroe.livello = 1;

Personaggio nemico = new Personaggio();
nemico.nome = "Drago";
nemico.vita = 200;
nemico.attacco = 40;
nemico.livello = 5;
```

`new` fa tre cose:
1. **Alloca** la memoria necessaria per l'oggetto
2. **Inizializza** gli attributi con valori di default
3. **Restituisce** un riferimento all'oggetto appena creato

Per accedere ad attributi e metodi si usa la **notazione punto**:

```java
eroe.attacca(nemico);   // "Kratos attacca Drago! Drago subisce 25 danni."
nemico.attacca(eroe);   // "Drago attacca Kratos! Kratos subisce 40 danni."
eroe.salaDiLivello();   // "Kratos sale al livello 2!"
```

I due oggetti sono **indipendenti**: modificare la vita di uno non influenza l'altro.

---

## Confronto diretto: C vs Java

| | Paradigma imperativo (C) | Paradigma OOP (Java) |
|---|---|---|
| **Focus** | Come fa la macchina | Quali entità esistono |
| **Unità base** | Funzione + variabile | Oggetto (dati + metodi) |
| **Dati** | Spesso globali | Incapsulati nell'oggetto |
| **Organizzazione** | Funzioni sparse | Classi e gerarchie |
| **Riuso** | Difficile | Tramite ereditarietà |
| **Scalabilità** | Limitata | Alta |

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- Il paradigma **imperativo** si concentra su *come* fare le cose passo per passo.
- Il paradigma **OOP** si concentra su *quali entità* esistono e come interagiscono.
- Una **classe** è il modello; un **oggetto** è la realizzazione concreta.
- Ogni oggetto ha **attributi** (dati) e **metodi** (comportamenti).
- `new` crea un oggetto in memoria e restituisce un riferimento ad esso.
- La **notazione punto** permette di accedere ad attributi e metodi: `oggetto.metodo()`.
- I tre pilastri dell'OOP sono **incapsulamento**, **ereditarietà** e **polimorfismo**.
:::

Nella prossima lezione vedremo come scrivere una classe completa e ben strutturata, con costruttori, getter e setter.