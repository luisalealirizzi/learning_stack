# Introduzione a Java

---

## Perché Java?

Prima di scrivere una sola riga di codice, vale la pena capire **perché esiste Java** e cosa lo rende diverso dai linguaggi che conosci già.

Con il C hai imparato a ragionare in modo imperativo: istruzioni, variabili, funzioni. Ma il C ha un limite importante — il codice compilato funziona **solo** sulla macchina per cui è stato compilato. Un programma compilato su Linux non gira su Windows, e viceversa.

Java nasce negli anni '90 con un obiettivo preciso:

> *"Write once, run anywhere."*
> Scrivi il codice una volta, eseguilo ovunque.

---

## Compilatori e interpreti

Per capire come Java risolve il problema della portabilità, dobbiamo fare un passo indietro.

Esistono due modi fondamentali per eseguire un programma:

**Il compilatore** traduce l'intero sorgente in codice macchina *prima* dell'esecuzione. Il risultato è un file eseguibile velocissimo, ma legato a uno specifico sistema operativo e processore. È quello che fa `gcc` con il C.

**L'interprete** legge ed esegue il sorgente riga per riga, senza compilare prima. È più flessibile — lo stesso file gira su qualsiasi macchina che ha l'interprete installato — ma in genere più lento. Python funziona così.

::: {.callout-note}
## Compilatore vs interprete in sintesi

| | Compilatore | Interprete |
|---|---|---|
| **Quando traduce** | Prima dell'esecuzione | Durante l'esecuzione |
| **Velocità** | Più veloce | Più lenta |
| **Portabilità** | Limitata | Alta |
| **Esempio** | C, C++ | Python, JavaScript |
:::

Java fa qualcosa di più intelligente: **unisce i vantaggi di entrambi**.

---

## La JVM e il bytecode

Ecco come funziona il ciclo di vita di un programma Java:

**1. Tu scrivi** il codice sorgente in un file `.java`.

**2. Il compilatore `javac`** traduce il sorgente non in codice macchina, ma in **bytecode** — un linguaggio intermedio, portabile e standardizzato, contenuto in file `.class`.

**3. La Java Virtual Machine (JVM)** legge il bytecode ed esegue il programma sulla macchina reale. Ogni sistema operativo ha la sua JVM, ma il bytecode è sempre lo stesso.

```
Codice sorgente (.java)
        ↓  javac
    Bytecode (.class)
        ↓  JVM
   Esecuzione sul sistema
```

::: {.callout-tip}
## Il compilatore JIT
La JVM non si limita a interpretare il bytecode riga per riga. Al suo interno c'è anche un compilatore **Just-In-Time (JIT)** che, mentre il programma gira, traduce le parti più usate direttamente in codice macchina nativo. Risultato: la velocità si avvicina a quella del C, mantenendo la portabilità.
:::

Il bytecode non è codice macchina — non può girare direttamente sul processore. Ma è uguale su tutte le piattaforme. Basta avere la JVM installata, e il programma gira.

---

## Perché si chiama Java?

Una curiosità che non trovi sul libro. Il nome originale del linguaggio era **Oak** ("quercia"). Peccato che esistesse già un altro linguaggio con quel nome — serviva un'alternativa.

Durante una riunione che stava diventando molto lunga, il team di sviluppatori stava bevendo grandi quantità di caffè. Qualcuno propose **Java** — slang americano per "caffè", che prende il nome dall'isola indonesiana famosa per le sue piantagioni.

Il nome piacque e rimase. Da qui il logo della tazzina fumante che ancora oggi identifica Java.

::: {.callout-note}
## JDK, JRE e JVM — le sigle che vedrai spesso

- **JVM** (Java Virtual Machine) — esegue il bytecode
- **JRE** (Java Runtime Environment) — JVM + librerie standard, per chi vuole solo *eseguire* programmi Java
- **JDK** (Java Development Kit) — JRE + compilatore + strumenti di sviluppo, per chi vuole *scrivere* programmi Java

Se stai sviluppando, ti serve il JDK.
:::

---

## File e programmi: Java vs C

In C i programmi erano divisi in file `.h` (intestazioni con le dichiarazioni) e file `.c` (implementazioni). Per collegare tutto si usava `#include`, che funziona in modo molto meccanico: il compilatore copia letteralmente il contenuto del file incluso dentro al programma.

Java funziona diversamente:

- Ogni classe sta in un **file separato** chiamato esattamente come la classe: `NomeClasse.java`
- Non esistono file header `.h`
- Per usare classi già pronte si usa `import`

```java
import java.util.Scanner;  // importa la classe Scanner
```

::: {.callout-important}
## `import` non è `#include`
Quando scrivi `import`, non stai copiando del codice nel tuo file. Stai solo dicendo al compilatore **dove trovare** quella classe. È come aggiungere un indirizzo a una rubrica — non come fare una fotocopia del contenuto.
:::

---

## Il primo programma Java

In Java, anche il programma più semplice deve avere almeno una **classe** e un metodo **`main`**. Il `main` è il punto di ingresso: è da lì che la JVM comincia a eseguire le istruzioni.

```java
public class PrimoProgramma {
    public static void main(String[] args) {
        System.out.println("Ciao, Java!");
    }
}
```

Tre regole fondamentali:

- Tutto il codice sta **dentro una classe**
- L'esecuzione parte sempre dal **`main`**
- Il file deve chiamarsi **esattamente come la classe**: `PrimoProgramma.java`

---

## Il metodo main — riga per riga

La firma del main è sempre questa:

```java
public static void main(String[] args)
```

Analizziamola pezzo per pezzo:

| Parola | Significato |
|---|---|
| `public` | Visibile alla JVM — deve poterlo chiamare dall'esterno |
| `static` | Non serve creare un oggetto per chiamarlo — la JVM lo chiama direttamente |
| `void` | Non restituisce nessun valore |
| `String[] args` | Può ricevere argomenti da riga di comando |

::: {.callout-note}
## Il main non va in tutte le classi
Il `main` serve **solo nella classe da cui parte il programma**. Tutte le altre classi (come `Personaggio`, `Arma`, ecc. che hai visto nel modulo OOP) non hanno un `main` — hanno solo attributi e metodi.
:::

---

## `System.out.println` — OOP in azione

La prima istruzione che scrivi in Java è già un esempio perfetto di programmazione a oggetti:

```java
System.out.println("Ciao, Java!");
```

Scomponila così:

- `System` — una **classe** della libreria standard Java
- `out` — un **oggetto** di tipo `PrintStream` che gestisce l'output
- `println()` — un **metodo** che stampa il testo e va a capo

In C usavi `printf()` — una funzione globale. In Java invece stai chiamando un **metodo su un oggetto**. Anche per stampare una riga, Java ragiona in termini di oggetti che collaborano.

```java
System.out.println("Testo con a capo");  // va a capo
System.out.print("Testo senza a capo"); // non va a capo
System.out.printf("Formattato: %d\n", 42); // come printf in C
```

---

## Compilare ed eseguire da terminale

Se vuoi fare tutto da terminale senza IDE:

```bash
# Compila il file sorgente → genera PrimoProgramma.class
javac PrimoProgramma.java

# Esegui il bytecode con la JVM
java PrimoProgramma
```

::: {.callout-warning}
## Attenzione al nome
Il comando `java` vuole il **nome della classe**, non il nome del file. Quindi `java PrimoProgramma`, non `java PrimoProgramma.class`.
:::

---

## Riepilogo

::: {.callout-note}
## I concetti chiave di questa lezione

- Java compila il sorgente `.java` in **bytecode** `.class`, non in codice macchina.
- La **JVM** esegue il bytecode su qualsiasi sistema operativo — *write once, run anywhere*.
- Il compilatore **JIT** ottimizza l'esecuzione traducendo il bytecode in codice nativo al volo.
- Ogni classe sta in un file `.java` con lo stesso nome della classe.
- `import` indica dove trovare una classe, non copia il suo codice.
- Ogni programma ha un punto di ingresso: il metodo **`main`**.
- `System.out.println()` è già OOP: classe, oggetto e metodo.
:::

Nella prossima lezione vedremo i **tipi di dato** di Java — dai tipi primitivi alla classe `String` — e come leggere input da tastiera.