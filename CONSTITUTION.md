# Costituzione condivisa — RalyaLands

## 1. Scopo e fonti

Questo documento raccoglie le regole comuni per Codex e Claude Code nel progetto RalyaLands. Le richieste esplicite dell'utente possono cambiare le scelte di progetto qui descritte; in quel caso applicare la richiesta e aggiornare i documenti pertinenti, senza chiedere una conferma aggiuntiva per il solo fatto che la scelta precedente era documentata.

Le fonti hanno ruoli diversi:

| File | Ruolo |
| --- | --- |
| `AGENTS.md` | Ingresso alle istruzioni di progetto per gli agenti. |
| `CLAUDE.md` | Ingresso per Claude Code; richiama le stesse regole condivise. |
| `CONSTITUTION.md` | Regole di lavoro e decisioni fondamentali da preservare. |
| `CombatPrototype.md` | Guida operativa e stato attuale del prototipo. |
| `CHANGELOG.md` | Storia cronologica, con autori, scelte, correzioni e verifiche. |

Gli asset e l'editor sono l'evidenza dell'implementazione effettiva. Se contraddicono la documentazione, ispezionare la differenza e registrare la rettifica; non assumere che una descrizione storica rappresenti lo stato corrente.

## 2. Regole attuali del prototipo

Preservare queste scelte negli interventi che non ne richiedono il cambiamento:

- Mappa di prova separata: `/Game/CombatPrototype/Lvl_CombatPrototype`, derivata da `Lvl_TopDown`.
- Implementazione delle regole di gioco in Blueprint. Python è uno strumento di authoring e verifica nell'editor, non una dipendenza del gameplay.
- Camera, manichini e base del movimento al clic provengono dal template TopDown.
- Due alleati controllati dal giocatore, Ezio e Nephilim, contro due guardie automatiche.
- Ordine dei turni: Ezio, Nephilim, Guardia 1, Guardia 2; saltare le unità sconfitte.
- Ogni turno concede 6 metri di movimento effettivo e un attacco. È consentito muoversi dopo aver attaccato, se resta budget.
- Corpo a corpo nel quadretto adiacente: 1 metro fra i centri, con tolleranza tecnica di 0,5 cm. Il movimento resta libero sul NavMesh: non è stata implementata una griglia con aggancio alle celle.
- Clic sul nemico: pianificare un punto adiacente libero, avvicinarsi entro il budget e attaccare. Il punto scelto deve rimanere stabile per quell'ordine.
- Le unità ferme devono essere ostacoli già nel NavMesh, come i muri. L'unità attiva non deve ostacolare se stessa. RVO è disattivato: non reintrodurre deviazioni locali che sostituiscano la pianificazione degli ostacoli.
- Anteprima e ordine effettivo devono usare lo stesso criterio di pianificazione. Il tratto oltre budget è rosso; sul nemico la barra rossa appare solo se il movimento non basta.
- Tutti iniziano con 100 HP. L'attacco alleato infligge 40 danni, quello delle guardie 25.
- I corpi sconfitti restano visibili a terra, senza collisione o ostacolo di navigazione.
- Vittoria o sconfitta bloccano ulteriori azioni di gioco. Le animazioni già avviate possono concludersi.

Gli altri parametri tecnici sono descritti nella guida corrente. Modificarli quando necessario per la richiesta, mantenendo coerenti implementazione, anteprima e documentazione.

## 3. Flusso di lavoro

### Inizio dell'intervento

1. Leggere le istruzioni, la guida corrente e lo storico pertinente.
2. Identificare l'applicazione che sta eseguendo il lavoro: Codex oppure Claude Code.
3. Ispezionare lo stato reale prima di scrivere: mappa aperta, eventuale PIE, Blueprint interessati e file esistenti.
4. Distinguere modifiche preesistenti dell'utente da quelle dell'intervento corrente. Non ripristinare né sovrascrivere lavoro estraneo alla richiesta.
5. Comunicare brevemente l'azione prevista e gli esiti rilevanti durante lavori lunghi.

### Modifiche Unreal

- Preferire Unreal MCP e le API dell'editor per leggere, creare e modificare gli asset. Non modificare direttamente i byte dei file `.uasset` o `.umap`.
- Prima di usare un nuovo strumento o una nuova API, leggerne schema e documentazione disponibili nella sessione.
- Leggere i grafi prima delle modifiche e rileggere i rami interessati dopo. Una compilazione riuscita non dimostra che un vecchio collegamento sia stato eliminato.
- In particolare, verificare la rimozione effettiva dei rami ereditati degli input: scrivere un ramo vuoto non è prova che il cablaggio precedente sia scomparso.
- Mantenere gli asset specifici del prototipo in `/Game/CombatPrototype/`, comprese le copie delle animazioni. Preservare le sorgenti del template quando la richiesta non ne richiede la modifica.
- Non cambiare la mappa iniziale o il GameMode globale solo per lavorare sul prototipo. La mappa di prova ha il proprio override.
- Fermare PIE quando serve per le modifiche autorizzate, informando brevemente l'utente. Al termine lasciare una situazione chiara: indicare se PIE è fermato o ancora attivo.
- Salvare gli asset realmente modificati. Non usare salvataggi globali per includere inconsapevolmente modifiche estranee dell'utente.

### Verifiche proporzionate

- Compilare i Blueprint modificati; nel prototipo usare gli avvisi come errori quando possibile e risolvere quelli introdotti.
- Per modifiche al gameplay, eseguire prove PIE pertinenti al comportamento cambiato. Per modifiche solo documentali, sono sufficienti controlli di coerenza, collegamenti e contenuto.
- Per movimento e navigazione controllare: percorso pianificato, ostacoli statici e unità ferme, budget effettivo, punto di arresto e assenza di movimento fuori turno.
- Per attacchi e turni controllare, secondo il caso: portata, unicità dell'attacco, punto adiacente libero, blocco durante il feedback, turni automatici e conclusione dello scontro.
- Per i corpi verificare che rimangano visibili e non blocchino collisione o navigazione.
- Per modifiche visive ispezionare anche l'immagine in PIE: i valori numerici da soli non verificano pose, sovrapposizioni o leggibilità.
- Controllare i log per gli errori runtime pertinenti. Non dichiarare assenza generale di problemi oltre l'ambito effettivamente provato.
- Se una verifica non è eseguibile, riportare il motivo e non presentarla come superata. Le prove storiche non diventano automaticamente prove della nuova versione.

## 4. Changelog obbligatorio e attribuzione

Ogni intervento con modifiche persistenti deve aggiungere almeno una voce a `CHANGELOG.md` prima della risposta conclusiva. Sono inclusi asset, codice, configurazioni, documentazione e queste stesse istruzioni.

Una voce può raccogliere i passaggi della stessa attività. Non occorre registrare ogni chiamata a uno strumento. Le conversazioni senza modifiche non devono generare voci fittizie; le verifiche senza modifiche possono essere registrate se utili, indicando che sono sole verifiche.

Usare questo formato, proseguendo l'ID numerico e aggiungendo le voci in fondo in ordine cronologico:

```markdown
## NNN — YYYY-MM-DD — Titolo concreto

Autore: Codex
Tipo: Funzionalità / Correzione / Documentazione / Verifica
Stato: Attuale / Storico / Sostituito da NNN

### Richiesta e scelte

Motivo dell'intervento e decisioni che ne spiegano il risultato.

### Modifiche

- Comportamento aggiunto o corretto e asset/file interessati.

### Verifiche

- Controlli realmente eseguiti ed esito.
- Eventuali verifiche non eseguite, con motivo.
```

Regole di attribuzione:

- Codex scrive `Autore: Codex`.
- Claude Code scrive `Autore: Claude Code`.
- Se più soggetti hanno lavorato alla stessa modifica, elencarli e distinguere i contributi verificabili. Non dedurre coautorie dal semplice uso alternato dei due strumenti.
- L'utente può avere richiesto o corretto una scelta senza essere l'autore materiale della modifica: distinguere le due cose.
- Registrare la data reale nota; non inventare orari, commit, versioni o risultati.
- Per una ricostruzione retrospettiva dichiararne le fonti e i limiti.
- Conservare i tentativi sostituiti, specificando quale soluzione li ha superati. Non riscrivere silenziosamente la storia per far sembrare che una correzione fosse già presente.
- Le rettifiche di errori storici devono essere esplicite e riconoscibili.

## 5. Coerenza e consegna

Prima di concludere:

1. Verificare che gli asset e i file necessari siano salvati.
2. Aggiornare la guida corrente se è cambiato il comportamento; aggiornare questa costituzione se sono cambiate le scelte fondamentali o le regole comuni.
3. Aggiungere la voce di changelog con l'autore corretto e verifiche oneste.
4. Riportare all'utente risultato, modo di provarlo, verifiche eseguite ed eventuali limiti materiali. Citare gli ID del changelog quando utile.

Non duplicare regole divergenti nei file di ingresso dei due agenti. Le modifiche alle regole comuni devono essere effettuate in questo documento e registrate nel changelog.
