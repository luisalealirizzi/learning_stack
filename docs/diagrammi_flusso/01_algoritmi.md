# Perché servono gli algoritmi?

Quando parliamo tra esseri umani, possiamo permetterci di essere vaghi.

Se dico:

> Attraversa la strada.

quasi tutti capiscono cosa fare.  
Non serve spiegare ogni singolo passo.

Ma un computer **non è una persona**.  
Non interpreta, non immagina, non completa le informazioni mancanti.

Ha bisogno di istruzioni:

- precise
- ordinate
- senza ambiguità

## Uomo e macchina non ragionano allo stesso modo

Una persona può capire una frase come:

> Quando il semaforo è verde, attraversa.

Per un essere umano il significato è abbastanza chiaro.  
Per una macchina, invece, questa frase non basta.

Mancano molte informazioni:

- cosa deve controllare?
- quando deve controllarlo?
- cosa deve fare se il semaforo non è verde?
- quando l'azione può considerarsi conclusa?

Per eseguire un compito, una macchina ha bisogno di passi più espliciti.

## Un'istruzione ambigua

Proviamo con un altro esempio:

> Avvia una canzone.

Per noi è semplice.  
Per un computer, invece, non lo è affatto.

Quale canzone?  
Dove deve cercarla?  
Cosa deve fare se non la trova?  
Deve fermarsi dopo una canzone o continuare?

Una frase che per una persona è normale, per una macchina può essere troppo vaga.

## Un'istruzione più precisa

Ecco la stessa idea scritta in modo più chiaro:

```text
INIZIO
osserva il semaforo
se il semaforo è verde
    attraversa la strada
altrimenti
    aspetta
FINE
```
In questo caso le istruzioni sono più adatte a essere eseguite, perché ogni passo è:

chiaro
ordinato
preciso
Un esempio legato alla musica

Anche una semplice azione come riprodurre un brano può essere descritta come una sequenza di passi:

INIZIO
apri l'app musicale
cerca la canzone
se la canzone esiste
    premi play
altrimenti
    mostra un messaggio di errore
FINE

Questo è molto più vicino al modo in cui ragiona un sistema informatico.

## Idea chiave

Un computer può eseguire bene un compito solo se riceve istruzioni che siano:

esplicite
in ordine logico
prive di ambiguità
Definizione semplice

Un algoritmo è una sequenza di istruzioni precise e ordinate per risolvere un problema o svolgere un compito.

## Perché è importante studiarli

Imparare gli algoritmi non serve solo a programmare.


Serve soprattutto a imparare a:

ragionare con ordine, scomporre un problema in passi, descrivere una procedura in modo chiaro ed evitare errori dovuti a istruzioni confuse

Gli algoritmi stanno dietro a molte situazioni quotidiane e digitali:

un videogioco che reagisce alle azioni del giocatore, un semaforo intelligente, un navigatore che calcola un percorso, un'app musicale che cerca e riproduce un brano

## Errore tipico

Uno degli errori più comuni è scrivere istruzioni come se chi legge fosse una persona.

Per esempio:

Gioca e prova a vincere
Vai dove devi andare
Metti la canzone giusta

Queste frasi possono avere senso per un essere umano, ma non per una macchina.

Una macchina ha bisogno di indicazioni più precise, come:

controlla la posizione del personaggio
se incontri un ostacolo, fermati
se trovi il brano, riproducilo
altrimenti segnala che non è disponibile
In sintesi

Gli algoritmi servono perché i computer non capiscono istruzioni vaghe.

Per funzionare correttamente, hanno bisogno di istruzioni:

precise
ordinate
non ambigue

## Definizione

Un **algoritmo** è una sequenza di passi che permette di risolvere un problema o svolgere un compito.

### Un algoritmo nella vita reale

Un algoritmo non è qualcosa che riguarda solo i computer. Ogni volta che descrivi un procedimento passo dopo passo, stai costruendo un algoritmo.

## Le 3 caratteristiche fondamentali

Un algoritmo funziona bene solo se ha queste caratteristiche:

### 1. Finito

Deve avere una fine. Non può andare avanti per sempre senza fermarsi.

### 2. Non ambiguo

Ogni istruzione deve avere un solo significato possibile. Non devono esserci interpretazioni diverse.

### 3. Ordinato

I passi devono essere eseguiti in un ordine preciso. Cambiare l’ordine può cambiare il risultato.

## Come costruire un algoritmo (metodo semplice)

Quando devi scrivere un algoritmo, segui questi passi:

pensa al problema
dividi il problema in azioni semplici
metti le azioni in ordine
controlla che ogni passo sia chiaro