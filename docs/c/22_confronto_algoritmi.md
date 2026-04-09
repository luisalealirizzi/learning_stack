# Confronto tra algoritmi

---

## Idea chiave

Abbiamo visto cinque algoritmi di ordinamento e due di ricerca. Tutti funzionano — ma non tutti funzionano bene nelle stesse situazioni. In questa lezione facciamo un passo indietro e guardiamo il quadro completo.

La domanda fondamentale e: **quanto lavoro fa un algoritmo al crescere della dimensione dell'array?**

---

## Blocco 1: Come si misura l'efficienza

Per confrontare gli algoritmi non usiamo il tempo in secondi — dipende troppo dalla macchina. Usiamo il **numero di operazioni** in funzione della dimensione N dell'array.

Le due operazioni principali sono:
- **confronti** — quante volte confrontiamo due elementi
- **scambi** — quante volte spostiamo elementi

Quello che ci interessa e come crescono queste operazioni quando N diventa grande.

### Crescita lineare — O(N)

Se raddoppio N, il lavoro raddoppia. La ricerca lineare su un array di 1000 elementi fa al massimo 1000 confronti. Su 2000 elementi ne fa 2000.

### Crescita quadratica — O(N²)

Se raddoppio N, il lavoro quadruplica. Bubble Sort, Selection Sort e Insertion Sort su 1000 elementi fanno circa 500.000 confronti. Su 2000 elementi ne fanno circa 2.000.000.

### Crescita logaritmica — O(log N)

Se raddoppio N, il lavoro cresce di uno solo. La ricerca binaria su 1000 elementi fa al massimo 10 confronti. Su 2.000.000 elementi ne fa al massimo 21.

### Crescita N log N

Via di mezzo tra lineare e quadratica. Merge Sort e Quick Sort su 1000 elementi fanno circa 10.000 operazioni. Su 2000 circa 22.000 — molto meno del quadratico.

---

## Blocco 2: Tabella comparativa — ordinamento

| Algoritmo | Confronti (caso medio) | Scambi | Memoria extra | Stabile | Note |
|---|---|---|---|---|---|
| **Bubble Sort** | N² / 2 | N² / 2 | no | si | ottimizzabile su array quasi ordinati |
| **Selection Sort** | N² / 2 | N - 1 | no | no | pochi scambi |
| **Insertion Sort** | N² / 4 | N² / 4 | no | si | veloce su array quasi ordinati |
| **Shell Sort** | N^1.5 circa | N^1.5 circa | no | no | dipende dalla sequenza di gap |
| **Merge Sort** | N log N | N log N | si | si | sempre veloce, usato in Java |
| **Quick Sort** | N log N | N log N | no | no | veloce in pratica, dipende dal pivot |

::: {.callout-note}
## Cosa significa "stabile"?
Un algoritmo e **stabile** se elementi uguali mantengono il loro ordine relativo originale dopo l'ordinamento.

Esempio: hai una lista di studenti ordinati per nome. Vuoi riordinarli per voto. Con un algoritmo stabile, a parita di voto gli studenti rimangono in ordine alfabetico. Con uno instabile no.

Bubble Sort e Insertion Sort sono stabili. Selection Sort e Quick Sort no.
:::

---

## Blocco 3: Quando usare quale algoritmo

### Array piccoli (N < 50)

Usa **Insertion Sort**. E semplice, stabile, e su array piccoli le differenze di prestazioni sono trascurabili. E anche il piu veloce su array quasi ordinati.

### Array medi o grandi, stabilita importante

Usa **Merge Sort**. Le prestazioni sono garantite indipendentemente dall'input. E stabile. E quello che usa Java in `Collections.sort()`.

### Array medi o grandi, memoria limitata

Usa **Quick Sort**. Non richiede memoria extra. E veloce in pratica su input casuali. E quello che usa la libreria C con `qsort()`.

### Array quasi ordinati

Usa **Insertion Sort** o **Bubble Sort ottimizzato**. Entrambi si fermano subito se l'array e gia ordinato.

### Mai ordinare per poi cercare una sola volta

Se devi cercare un elemento una volta sola in un array non ordinato, usa la **ricerca lineare** direttamente. Ordinare e poi usare la ricerca binaria ha senso solo se cerchi molte volte.

---

## Blocco 4: Visualizzare la differenza

Immagina di dover ordinare un array di N elementi. Quante operazioni fa ogni algoritmo?

| N | N² (quadratico) | N log₂N | log₂N (ricerca binaria) |
|---|---|---|---|
| 10 | 100 | 33 | 3 |
| 100 | 10.000 | 664 | 7 |
| 1.000 | 1.000.000 | 9.966 | 10 |
| 10.000 | 100.000.000 | 132.877 | 13 |
| 1.000.000 | 10^12 | 19.931.568 | 20 |

Su un milione di elementi, il Bubble Sort fa mille miliardi di operazioni. Il Merge Sort ne fa venti milioni. La ricerca binaria ne fa venti.

La differenza tra un algoritmo quadratico e uno N log N non e una questione di stile — e la differenza tra un programma che risponde in un secondo e uno che impiega ore.

---

## Blocco 5: Il TimSort — il meglio di tutto

Java non usa ne solo Merge Sort ne solo Insertion Sort. Usa il **TimSort** — un algoritmo ibrido progettato da Tim Peters nel 2002 e adottato da Java, Python e molti altri linguaggi moderni.

Il TimSort combina i punti di forza di entrambi:

```
Array grande
    |
    v
Dividi in blocchi piccoli (chiamati "run")
    |
    v
Ordina ogni blocco con Insertion Sort
(veloce su blocchi piccoli e quasi ordinati)
    |
    v
Fondi i blocchi con Merge Sort
(veloce su blocchi gia ordinati)
    |
    v
Array ordinato
```

Il risultato e un algoritmo che:
- e veloce su array casuali come il Merge Sort
- e velocissimo su array quasi ordinati come l'Insertion Sort
- e stabile come il Merge Sort
- funziona bene su dati reali, che spesso hanno gia una struttura parzialmente ordinata

Quando in Java scrivi:

```java
Collections.sort(lista);
Arrays.sort(array);
```

sta girando il TimSort — costruito sulle stesse idee che hai studiato in queste lezioni.

---

## Blocco 6: Ricerca — il quadro completo

| | Ricerca lineare | Ricerca binaria |
|---|---|---|
| **Requisito** | nessuno | array ordinato |
| **Caso peggiore** | N confronti | log₂N confronti |
| **Su 1.000 elementi** | 1.000 | 10 |
| **Su 1.000.000 elementi** | 1.000.000 | 20 |
| **Quando usarla** | array non ordinato, ricerca unica | array ordinato, ricerche frequenti |

### La strategia giusta

```
Devo cercare una volta sola?
    → ricerca lineare — semplice, nessun requisito

Devo cercare molte volte?
    → ordina una volta con Merge Sort o Quick Sort
    → poi usa ricerca binaria per ogni ricerca

L'array e gia ordinato?
    → usa subito la ricerca binaria
```

---

## Blocco 7: Il quadro completo — tutti gli algoritmi

```
ORDINAMENTO
                    semplici                    avanzati
              ┌─────────────────┐         ┌──────────────────┐
              │  Bubble Sort    │         │   Merge Sort     │
              │  Selection Sort │  →→→→   │   Quick Sort     │
              │  Insertion Sort │         │   Shell Sort     │
              │  (N²)          │         │   (N log N)      │
              └─────────────────┘         └──────────────────┘
                   su array piccoli           su array grandi
                   o quasi ordinati

                              ↓ combinati
                          TimSort
                    (usato in Java e Python)

RICERCA
              ┌──────────────────┐       ┌──────────────────┐
              │ Ricerca lineare  │       │ Ricerca binaria  │
              │ array qualsiasi  │       │ array ordinato   │
              │ (N)              │       │ (log N)          │
              └──────────────────┘       └──────────────────┘
```

---

## Schema mentale finale

```
Quanto e grande il mio array?
    piccolo (< 50)      → Insertion Sort
    grande              → Merge Sort o Quick Sort

Ho bisogno di stabilita?
    si                  → Merge Sort
    no                  → Quick Sort o Selection Sort

Ho poca memoria?
    si                  → Quick Sort (in-place)
    no                  → Merge Sort

L'array e quasi ordinato?
    si                  → Insertion Sort o Bubble Sort ottimizzato
    no                  → Merge Sort o Quick Sort

Devo cercare?
    una volta sola      → ricerca lineare
    molte volte         → ordina + ricerca binaria
    array gia ordinato  → ricerca binaria direttamente
```

---

## Esercizi

**Esercizio 1**
Scrivi un programma che genera array di dimensione 100, 1000 e 10000 con numeri casuali e conta quanti confronti fa il Bubble Sort e il Merge Sort su ognuno. Stampa i risultati in una tabella e verifica che corrispondano alle previsioni teoriche.

**Esercizio 2**
Genera un array di 1000 elementi gia quasi ordinati (ad esempio ordinato con solo 5 elementi scambiati casualmente). Conta i confronti di Bubble Sort ottimizzato e Merge Sort. Quale dei due e piu veloce in questo caso?

**Esercizio 3**
Scrivi un programma che:
- genera 500 numeri casuali
- li ordina con Quick Sort
- chiede all'utente 10 valori da cercare
- per ognuno usa la ricerca binaria e stampa il risultato
- alla fine stampa quante ricerche hanno avuto successo

**Esercizio 4 — riflessione**
Senza scrivere codice, rispondi a queste domande:
- Hai un array di 10 studenti da ordinare per cognome. Quale algoritmo usi? Perche?
- Hai un database di 10 milioni di prodotti gia ordinati per codice. Devi fare 100.000 ricerche. Quale strategia usi?
- Hai un array quasi ordinato di 50.000 elementi con 3 elementi fuori posto. Quale algoritmo di ordinamento e piu efficiente?
