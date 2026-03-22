# Guida al piano di indirizzamento IP


## PASSO 1: Calcolare gli host necessari

Per ogni rete:

- Prendi gli host richiesti  
- Aggiungi:
  - eventuale riserva (es. +10%)  
  - eventuali dispositivi extra (router, stampanti, server…)


## PASSO 2: Definire l’indirizzo di rete

Devi scegliere la rete iniziale solo se la traccia NON la fornisce.

### Scelta dello spazio privato e del prefisso

Quando la rete non è specificata, si deve scegliere uno spazio di indirizzamento privato tra:

- 192.168.x.x  
- 172.16–31.x.x  
- 10.x.x.x  

La scelta dello spazio dipende dal contesto:

- 192.168 → reti piccole (scuole, uffici)  
- 172 → reti intermedie  
- 10 → reti grandi o espandibili  

⚠️ Il numero di host NON dipende dallo spazio scelto, ma dal prefisso.

Esempio:

- 192.168.0.0/23 → circa 510 host  
- 10.0.0.0/23 → circa 510 host  

cambia il tipo di rete, non la dimensione

### Regole importanti

- lo spazio (192 / 172 / 10) è una scelta progettuale  
- il prefisso (/24, /23, /22) determina quanti host può contenere la rete  

Scegli una rete privata coerente con il contesto.

### Esempi

Rete: 192.168.0.0/25 Host disponibili: 126  (adatta per scuole o uffici con meno di 126 host)
Rete: 172.16.0.0/25 Host disponibili: 126  (adatta per medie imprese possibilimente in più sedi con meno di 126 host)
Rete: 10.0.0.0/25 Host disponibili: 126  (adatta per grandi aziende o reti diffuse ma con meno di 126 host)

Rete: 192.168.0.0/20 Host disponibili: 4094 (adatta per scuole o uffici con meno di 4094 host)
Rete: 172.16.0.0/20 Host disponibili: 4094  (adatta per medie imprese possibilimente in più sedi con meno di 4094 host)
Rete: 10.0.0.0/20 Host disponibili: 4094 (adatta per grandi aziende o reti diffuse ma con meno di 4094 host)


### Calcolo del prefisso

Trova la prima potenza di 2 ≥ (host + 2)

Esempio:

- 100 host → 100 + 2 = 102  
- 2⁶ = 64 no
- 2⁷ = 128 ok

 32 - 7 = 25   -> prefisso = /25  


## PASSO 3: Scegliere il blocco giusto

Per trovare il blocco:

- prendi il numero di host  
- aggiungi 2  
- scegli la prima potenza di 2 ≥  

Poi:

- sottrai da 32 → ottieni il prefisso  

---

### Tabella utile

| Host utili | Prefisso | Indirizzi totali |
|-----------|---------|------------------|
| 2 | /30 | 4 |
| 6 | /29 | 8 |
| 14 | /28 | 16 |
| 30 | /27 | 32 |
| 62 | /26 | 64 |
| 126 | /25 | 128 |
| 254 | /24 | 256 |


## PASSO 4: Ordinare le subnet

Se ci sono più reti: ordina dalla più grande alla più piccola

Esempio:

- prima /25  
- poi /26  
- poi /27  
- poi /28  


## PASSO 5: Capire il salto

Il salto serve per passare da una subnet alla successiva.

| Prefisso | Salto |
|---------|------|
| /25 | 128 |
| /26 | 64 |
| /27 | 32 |
| /28 | 16 |

Esempio:

Rete: 192.168.0.0/26  

salto = 64  

rete successiva = 192.168.0.64  


## PASSO 6: Assegnare le subnet

Per ogni subnet:

- scrivi la rete  
- trova il broadcast  
- trova gli host  

### Regole

- rete = primo indirizzo  
- primo host = rete + 1  
- broadcast = indirizzo prima della rete successiva  
- ultimo host = broadcast - 1  


### Esempio

Rete: 192.168.0.0/26  

- rete: 192.168.0.0  
- primo host: 192.168.0.1  
- ultimo host: 192.168.0.62  
- broadcast: 192.168.0.63  


## PASSO 7: Passare alla subnet successiva

Usa il salto.

Esempio:

192.168.0.0 + 64 = 192.168.0.64  -> questa è la nuova rete  


## PASSO 8: Ripetere

Ripeti:
- assegna subnet  
- calcola broadcast  
- vai avanti con il salto  

fino a completare tutte le subnet  


## PASSO 9: Controllo finale

Controlla sempre:

- ✔ ogni subnet ha abbastanza host  
- ✔ le subnet NON si sovrappongono  
- ✔ tutto sta dentro la rete iniziale  
- ✔ non hai saltato indirizzi  


## CASI PARTICOLARI

### Una sola subnet

Fai solo:

- passo 1  
- passo 2  
- calcoli rete, host, broadcast  


### Subnet tutte uguali

- usa sempre lo stesso prefisso  


### Subnet diverse (caso più comune)

- usa VLSM  
- ogni subnet ha dimensione diversa  
- ordina dalla più grande  


## STRATEGIA ANTIPANICO

Se ti blocchi:

- conta gli host  
- trova il numero nella tabella  
- scrivi il prefisso  
- parti dalla rete  
- usa il salto 