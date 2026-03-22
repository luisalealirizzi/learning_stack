# Variabili e controlli con Flowgorithm

Finora abbiamo costruito diagrammi di flusso. Ora iniziamo a usarli per lavorare con:

variabili
input e output
controlli

Questo è il primo passo verso la programmazione.

## Cos’è una variabile

Una variabile è uno spazio dove memorizzi un valore.

Esempi: un numero, un punteggio, una password...
In Flowgorithm una variabile deve essere: prima dichiarata e poi usata ed eventualmente modificata

## Tipi di variabili

In Flowgorithm (e nei linguaggi di programmazione) le variabili possono contenere tipi diversi di dati. Scegliere il tipo corretto è importante, perché determina che cosa puoi fare con quel valore.

I tipi principali sono:

### Integer (intero): 
contiene numeri senza virgola. Esempi: 3, 25, -10
Si usa per contatori, età, quantità...

### Real (decimale)
contiene numeri con la virgola. Esempi: 3.5, 2.75
Si usa per prezzi, misure, medie...

### String (testo)
contiene parole o frasi. Esempi: "ciao", "password123"
Si usa per nomi, messaggi, input dell’utente.

### Boolean (vero/falso)
contiene solo due valori: true oppure false.
Si usa per condizioni e controlli (es. accesso consentito sì/no). 

## Dichiarazione delle variabili

Prima di usare una variabile, bisogna dichiararla.

Esempi:

dichiara età come Integer
dichiara prezzo come Real
dichiara nome come String
dichiara accesso come Boolean

Questo serve per dire al programma che tipo di dato conterrà.

## Assegnazione di valori

Dopo aver dichiarato una variabile, possiamo assegnarle un valore.

Esempi:

età = 18
prezzo = 9.99
nome = "Luca"
accesso = true

L’assegnazione serve per memorizzare un valore nella variabile.

## Input (lettura)

Per leggere un valore dall’utente si usa un’istruzione di input.

Esempi:

leggi età
leggi nome

Questo significa che il programma chiede un valore all’utente e lo salva nella variabile.

## Output (stampa)

Per mostrare un risultato si usa un’istruzione di output.

Esempi:

stampa età
stampa nome

Attenzione alle virgolette
stampa nome → mostra il valore della variabile
stampa "nome" → mostra la parola nome

Esempio: 
nome = "Luca" 
stampa nome → Luca 
stampa "nome" → nome

## Decisioni (if)

Le decisioni servono per scegliere tra due possibilità.

Esempio: 
Se l’età è maggiore o uguale a 18, puoi entrare.

Schema:
```mermaid
flowchart TD
A([INIZIO]) --> B[Leggi età]
B --> C{età >= 18?}
C -- Sì --> D[Stampa "Accesso consentito"]
C -- No --> E[Stampa "Accesso negato"]
D --> F([FINE])
E --> F
```

## Ripetizioni (while)

Il ciclo while serve per ripetere un’azione finché una condizione è vera.

Esempio:

Inserisci una password finché è sbagliata.

Schema:

```mermaid
flowchart TD
A([INIZIO]) --> B[Leggi password]
B --> C{Password corretta?}
C -- No --> B
C -- Sì --> D[Stampa "Accesso consentito"]
D --> E([FINE])
```

## In sintesi

Un programma è fatto da:

dati (variabili)
operazioni (assegnazioni)
interazioni (input/output)
controlli (if, while)

Questi sono i mattoni fondamentali della programmazione.