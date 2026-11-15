# Changelog — RalyaLands

Le voci sono in ordine cronologico, dalla più vecchia alla più recente. Aggiungere i nuovi interventi in fondo, con ID progressivo e autore esplicito secondo `CONSTITUTION.md`.

## Provenienza dello storico

Le voci 001–011 sono una ricostruzione retrospettiva della sessione di lavoro del 2026-11-15, basata sulla conversazione, sugli esiti degli strumenti e sulla precedente guida `CombatPrototype.md`. Non sono stati inventati orari o commit. Tutti questi interventi sono stati eseguiti da **Codex**; non risultano interventi di Claude Code in questa sessione.

Le verifiche riportate nelle voci storiche si riferiscono alla versione di quel momento. I cambiamenti successivi possono sostituire regole e risultati precedenti. La guida `CombatPrototype.md` descrive il comportamento corrente.

## 001 — 2026-11-15 — Ispezione del template e scelta della mappa di base

Autore: Codex
Tipo: Verifica
Stato: Storico

### Richiesta e scelte

L'utente ha chiesto di identificare il livello aperto tramite Unreal MCP, poi di costruire uno scontro a turni riusando il template. Un primo riferimento a `Lvl_Strategy` è stato sostituito dall'utente con `Lvl_TopDown` prima della creazione degli asset del prototipo.

### Modifiche

- Nessuna modifica nella prima ispezione: il livello aperto era `/Game/TopDown/Lvl_TopDown`.
- Ispezionati `BP_TopDownController`, `BP_TopDownCharacter` e `BP_TopDownGameMode`.
- Individuati camera sul personaggio, Enhanced Input, movimento diretto del template e `SimpleMoveToLocation`.
- Scelta una mappa e Blueprint separati per sviluppare il prototipo.

### Verifiche

- Livello corrente letto con `GetCurrentLevel` di Unreal MCP.
- Letti i grafi del controller e i riferimenti del GameMode del template.

## 002 — 2026-11-15 — Primo prototipo di combattimento a turni

Autore: Codex
Tipo: Funzionalità
Stato: Storico; portata, navigazione e sconfitta aggiornate nelle voci successive

### Richiesta e scelte

Creare Ezio e Nephilim contro due guardie, con manichini del template, movimento limitato, attacco base, turni automatici delle guardie e fine dello scontro.

### Modifiche

- Creati `/Game/CombatPrototype/Lvl_CombatPrototype`, `BP_CombatUnit`, `BP_CombatController`, `BP_CombatGameMode` e `BP_CombatHUD`.
- Duplicata la mappa TopDown e rimossi gli ostacoli della copia per partire da un'arena libera. Conservati illuminazione, pavimento e navigazione.
- GameMode del prototipo assegnato come override della mappa; configurazione globale del template mantenuta.
- Ordine Ezio → Nephilim → Guardia 1 → Guardia 2, saltando gli sconfitti.
- Budget 6 metri per turno, un attacco, 100 HP iniziali; danni 40 per gli alleati e 25 per le guardie.
- Prima portata sperimentale di 2 metri. In questa versione gli sconfitti erano nascosti.
- Movimento conteggiato sulla distanza effettiva in piano, con correzione al limite del budget.
- HUD Blueprint Canvas con unità attiva, HP, movimento, disponibilità dell'attacco, pulsante Fine turno e scorciatoia Spazio.
- Camera del template allontanata prima a 1.800, poi a 2.200 cm per inquadrare lo scontro; pannello HUD spostato in basso per non coprire i nemici iniziali.
- Navigazione impostata per la generazione dinamica. Regole di gioco in Blueprint; Python usato per authoring e prove tramite MCP.

### Verifiche

- Quattro Blueprint compilati e salvati, con avvisi trattati come errori.
- PIE: 600 cm percorsi e 0 cm aggiuntivi a budget esaurito; reset a 6 metri al cambio turno.
- Movimento e attacchi automatici delle guardie, attacchi fuori portata respinti, secondo attacco nello stesso turno respinto.
- Vittoria e sconfitta raggiunte; comandi finali bloccati. Prima partita vittoriosa registrata con HP `[50,50,0,0]`.
- Ispezione visiva e controllo dei log runtime.
- Le prove iniziali non avevano rilevato i rami residui dell'input, corretti nella voce 003 dopo la segnalazione dell'utente.

## 003 — 2026-11-15 — Rimozione dei movimenti fuori turno e arresto delle animazioni

Autore: Codex
Tipo: Correzione
Stato: Attuale

### Richiesta e scelte

L'utente ha segnalato Nephilim in movimento senza comando e guardie che continuavano a camminare o mantenevano l'animazione di corsa dopo l'attacco.

### Modifiche

- Individuati e rimossi i rami `Triggered` e di rilascio ancora collegati all'input originale del template.
- Ricreato l'evento del clic con il solo ramo `Started` verso `ClickCommand`; riletta la struttura per verificarne l'effettiva rimozione.
- Aggiunta `StopUnitMotion`: interruzione della navigazione, velocità azzerata e consumo dell'input pendente.
- Applicato l'arresto ad attacco, cambio turno e fine budget.

### Verifiche

- PIE: Nephilim immobile senza input per 1,5 secondi e controllabile al clic.
- Dopo gli attacchi, tutte le unità con spostamento residuo 0 cm, velocità zero, `GroundSpeed = 0` e `ShouldMove = false`.
- Guardie fermate dopo 600 cm anche quando il bersaglio rimaneva distante.
- Controller compilato e salvato.

## 004 — 2026-11-15 — Avvicinamento e attacco con un solo clic

Autore: Codex
Tipo: Funzionalità
Stato: Attuale nel principio; pianificazione sostituita dalla voce 010

### Richiesta e scelte

L'utente ha chiesto che il clic diretto sul nemico avviasse il movimento necessario e poi l'attacco, entro il budget disponibile.

### Modifiche

- Aggiunti bersaglio e attacco in attesa, controllati durante il movimento.
- Arrivo in portata seguito da attacco singolo; budget insufficiente seguito da arresto senza danno.
- Un altro comando o il cambio turno annulla l'attacco in attesa.
- Capsule dei manichini abilitate a bloccare la traccia Visibility; riconoscimento diretto dell'attore colpito dal cursore.

### Verifiche

- Alla portata di allora, 2 metri: da 7 metri Ezio arrivava a circa 192,54 cm, infliggeva 40 danni e conservava circa 0,93 metri.
- Nephilim con bersaglio troppo distante percorreva 600 cm, si fermava a 250 cm e non consumava l'attacco.
- Unità ferme al termine, Blueprint compilati e salvati.

## 005 — 2026-11-15 — Anteprima del percorso, costo e residuo

Autore: Codex
Tipo: Funzionalità
Stato: Attuale; costo dell'attacco aggiornato dalla voce 010

### Richiesta e scelte

Rendere prevedibile il comando prima del clic, mostrando percorso e possibilità di raggiungere e attaccare un nemico.

### Modifiche

- Percorso calcolato sul NavMesh e disegnato tramite HUD Canvas, con aggiornamento circa 10 volte al secondo.
- Tratto verde entro budget, rosso oltre budget e indicatore del punto di arresto.
- Riquadro con costo e residuo stimati, `RAGGIUNGI E ATTACCA`, `MOVIMENTO INSUFFICIENTE` o `ATTACCO GIA USATO`.
- Anteprima nascosta sull'HUD, nei turni delle guardie e a scontro concluso.
- Corretto il formato dei decimali usando una conversione esplicita da intero a double prima della divisione.
- Eliminato il precedente taglio in linea retta della destinazione: il limite viene applicato alla distanza effettivamente percorsa.
- Nella prima implementazione il costo dell'attacco era il percorso al centro del nemico meno la portata; questo criterio è stato sostituito dal percorso a un punto adiacente libero nella voce 010.

### Verifiche

- PIE e ispezione visiva dei tratti verde/rosso e delle indicazioni del costo.
- Alla portata di allora: stima di 5,05 metri, mostrata come 5,1, e attacco con circa 0,93 metri residui.
- Bersaglio oltre budget: arresto senza attacco. Controllati anche terreno, attacco già usato e occultamento dell'anteprima.
- Controller e HUD compilati e salvati; log runtime controllati.

## 006 — 2026-11-15 — Barra di arresto sul nemico e chiarimento dell'indicazione

Autore: Codex
Tipo: Funzionalità / Correzione
Stato: Parzialmente sostituito da 007

### Richiesta e scelte

L'utente non vedeva il limite rosso mirando un nemico. La prima risposta è stata rendere esplicito il punto di arresto anche quando il nemico era raggiungibile; la successiva precisazione dell'utente ha chiarito che la distanza osservata dipendeva dalla portata di 2 metri.

### Modifiche

- Aggiunta `DrawEnemyLimit`, una barra trasversale sul percorso con etichetta di portata o limite del movimento.
- In questa fase la barra compariva anche sui bersagli raggiungibili.
- Aggiornata la legenda ed eliminata l'etichetta duplicata sullo stesso punto.

### Verifiche

- Ispezione visiva in PIE sia del bersaglio raggiungibile sia di quello oltre budget.
- HUD compilato e salvato. La barra sui bersagli raggiungibili è stata rimossa nella voce seguente su richiesta dell'utente.

## 007 — 2026-11-15 — Corpo a corpo nel quadretto adiacente

Autore: Codex
Tipo: Modifica delle regole / Correzione
Stato: Attuale

### Richiesta e scelte

L'utente ha scelto un attacco da mischia nel quadretto adiacente e ha chiarito che la barra rossa non serviva quando era possibile colpire.

### Modifiche

- Portata portata a 1 metro fra i centri, con tolleranza di 0,5 cm, per alleati e guardie.
- Aggiornati attacco, avvicinamento, IA, anteprima e testi dell'HUD.
- Barra rossa sul nemico solo quando il movimento è insufficiente.
- Conservato il movimento libero: nessun aggancio a una griglia è stato introdotto.

### Verifiche

- PIE: nessun danno a 150 cm; attacco valido a 95 cm.
- Un clic da 7 metri ha portato Ezio a 100 cm dopo 600 cm di movimento, seguito dall'attacco.
- Anteprima di Nephilim raggiungibile senza barra rossa; guardie automatiche ferme dopo il colpo.
- Controller e HUD compilati e salvati.

## 008 — 2026-11-15 — Arena con muro centrale e passaggi laterali

Autore: Codex
Tipo: Funzionalità / Level design
Stato: Attuale

### Richiesta e scelte

Mettere alla prova il posizionamento e il conteggio delle deviazioni prima di introdurre abilità o coperture.

### Modifiche

- Aggiunti sette elementi nella cartella Outliner `CombatPrototype/Arena`: muro centrale, quattro muri perimetrali e due blocchi laterali.
- Muro centrale 1 × 7 metri, alto 1,8 metri; interno circa 21 × 16 metri; passaggi a nord e sud larghi circa 4,6 metri.
- Riutilizzati cubi e materiali del progetto. Ostacoli con collisione e navigazione dinamica, senza bonus di copertura.
- Mappa salvata e vista dell'editor centrata sull'arena.

### Verifiche

- Anteprima di Ezio con cinque punti attorno al lato sud; Nephilim attorno al lato nord.
- Percorsi iniziali per attaccare di circa 7,8 metri: insufficienza del primo budget di 6 metri mostrata correttamente.
- Movimento campionato: circa 599,73 cm lungo la deviazione, residuo zero, arresto fuori portata senza danno.
- Partita completa fino alla vittoria con HP `[25,25,0,0]`; IA e turni funzionanti fra gli ostacoli.
- Nessun errore runtime pertinente nei log delle prove.

## 009 — 2026-11-15 — Primo tentativo di aggiramento fra unità e protezione dagli stalli

Autore: Codex
Tipo: Correzione
Stato: RVO sostituito da 010; protezione dagli stalli mantenuta

### Richiesta e scelte

L'utente ha segnalato Guardia 2 bloccata da Guardia 1. Il primo tentativo ha usato l'evitamento locale RVO, oltre alla collisione già presente.

### Modifiche

- Abilitato temporaneamente RVO sul movimento delle unità.
- Aggiunti monitoraggio dell'ordine e avanzamento: dopo 0,8 secondi senza almeno 2 cm di progresso, arresto di navigazione, input e velocità.
- Guardia bloccata: fine turno; unità del giocatore bloccata: controllo mantenuto e possibilità di scegliere un'altra destinazione.
- Prevista l'esclusione degli sconfitti dall'evitamento.

### Verifiche

- Guardia 2 aggirava Guardia 1 con circa 193 cm di deviazione laterale; il bloccante rimaneva fermo.
- Stallo forzato nella sessione di test: turno restituito a Ezio entro 1,25 secondi, velocità e animazione azzerate.
- Scontro completo fino alla vittoria.
- L'utente ha poi rilevato deviazioni e giri innaturali vicino all'attacco: questo ha motivato la sostituzione della soluzione nella voce 010.

## 010 — 2026-11-15 — Unità come ostacoli nel calcolo del percorso

Autore: Codex
Tipo: Correzione / Navigazione
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto di considerare i personaggi già nel calcolo del percorso, come i muri, per eliminare le correzioni locali innaturali e i giri attorno al bersaglio.

### Modifiche

- RVO disattivato.
- Capsule delle unità ferme registrate nel NavMesh come ostacoli dinamici `NavArea_Null`.
- Unità attiva esclusa dal proprio ostacolo. Al cambio turno attesa minima di 0,15 secondi e completamento dell'aggiornamento della navigazione prima di accettare ordini o avviare l'IA.
- `FindAttackPlan` valuta 16 punti a 98 cm dal bersaglio, a passi di 22,5°, scarta quelli occupati o non raggiungibili e sceglie il percorso più corto fra i candidati validi.
- Anteprima, clic del giocatore e IA usano la stessa pianificazione verso un punto adiacente libero, mantenuto per l'ordine.
- Rifinito l'avvicinamento finale con il movimento del template per compensare la tolleranza di arrivo della navigazione.
- Nessun attacco avviato se manca un punto adiacente libero. Sconfitti rimossi dal NavMesh; protezione dagli stalli mantenuta.

### Verifiche

- Con un alleato in mezzo, anteprima con quattro punti già deviati attorno all'ostacolo.
- Movimento a oltre 84 cm dal centro del bloccante, che resta fermo; attacco a circa 99,5 cm dal nemico e un solo danno.
- Guardia 2 con Guardia 1 sul percorso: distanza minima oltre 85 cm, attacco a circa 99,8 cm, velocità finale zero e turno restituito a Ezio.
- Bersaglio chiuso fra muri e due unità: nessun piano valido.
- Partita completa fino alla vittoria con HP `[25,25,0,0]` e navigazione disabilitata sui caduti.
- Controller e unità compilati e salvati; impostazioni delle capsule verificate dopo compilazione; log runtime controllati.

## 011 — 2026-11-15 — Feedback dell'attacco, barre HP e corpi a terra

Autore: Codex
Tipo: Funzionalità / Feedback visivo
Stato: Attuale

### Richiesta e scelte

Rendere leggibile l'attacco base con orientamento, animazione, reazione, danno visibile, HP e sconfitta animata, preservando movimento e turni.

### Modifiche

- Attaccante orientato verso il bersaglio; attacco disarmato e reazione al colpo.
- Danno applicato dopo 0,22 secondi tramite `ResolveAttack`, insieme al feedback della vittima.
- Blocco dei comandi di movimento, attacco e fine turno per 1,35 secondi; indicazione nell'HUD. Dopo il feedback torna disponibile il movimento residuo.
- Numero del danno flottante, barre HP e valori numerici vicini alle unità, con colori per squadra e spaziatura laterale per leggibilità.
- Elementi dell'interfaccia principale disegnati sopra le indicazioni nel mondo.
- Animazione di sconfitta conservata sull'ultimo fotogramma. Corpi visibili, movimento disabilitato, collisione e navigazione rimosse.
- Create copie locali `AN_CombatAttack`, `AN_CombatHit`, `AN_CombatDeath` in `/Game/CombatPrototype/Animations/`, con root motion disattivato e radice bloccata per evitare spostamenti durante il colpo.
- Unità vive ripristinate a `ABP_Unarmed` dopo l'animazione. Sorgenti del template conservate.

### Verifiche

- Prima dell'impatto HP invariati e Fine turno bloccato; dopo l'impatto HP da 100 a 60 e danno `-40` visibile.
- Attaccante e vittima nelle stesse coordinate prima e dopo il colpo; ritorno di entrambi all'Animation Blueprint.
- Ispezione visiva di animazione, barre, danno e caduta.
- Corpo a 0 HP visibile, senza collisione e navigazione, animazione ferma a 1,1 secondi; passaggio del giocatore attraverso la sua posizione verificato.
- Scontro completo: vittoria con HP `[25,25,0,0]`.
- Partita senza attacchi del giocatore: sconfitta con HP `[0,0,100,100]`, entrambi i corpi visibili e attraversabili.
- Unit, controller e HUD compilati senza errori né avvisi e salvati; nessun errore runtime pertinente nei log delle prove.

## 012 — 2026-11-15 — Changelog e costituzione condivisa per Codex e Claude Code

Autore: Codex
Tipo: Documentazione / Istruzioni condivise
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto uno storico completo delle scelte e dei passaggi, istruzioni comuni per lavorare anche con Claude Code e aggiornamenti del changelog con attribuzione dell'agente.

### Modifiche

- Creato questo `CHANGELOG.md`, con ricostruzione trasparente delle fasi precedenti e dei tentativi sostituiti.
- Creata `CONSTITUTION.md` come unica fonte delle regole condivise e delle scelte fondamentali.
- Creati `AGENTS.md` e `CLAUDE.md` come ingressi per i due agenti.
- Reso obbligatorio aggiornare il changelog per ogni intervento con modifiche persistenti, anche documentali, usando `Autore: Codex` oppure `Autore: Claude Code`.
- Definiti formato delle voci, obbligo di verifiche oneste, conservazione dello storico e aggiornamento della guida corrente.
- Riorganizzato `CombatPrototype.md` come guida dello stato attuale; lo storico delle versioni precedenti è raccolto nel changelog.

### Verifiche

- Letti i documenti preesistenti e verificata l'assenza di istruzioni di progetto concorrenti nella cartella.
- Ispezionati in sola lettura i Blueprint correnti con Unreal MCP e l'inventario degli asset del prototipo.
- Verificate le configurazioni globali: avvio e GameMode restano quelli TopDown, con override nella mappa del prototipo.
- Controllati collegamenti locali, ID, attribuzioni e coerenza dei documenti.
- Nessuna modifica agli asset Unreal in questo intervento; nessuna nuova prova PIE richiesta o dichiarata per la sola documentazione.
- Il caricamento delle istruzioni in una nuova sessione di Claude Code non è stato provato da questa sessione Codex.

## 013 — 2026-11-15 — Inizializzazione Git e Git LFS

Autore: Codex
Tipo: Configurazione / Versionamento
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto di trasformare la cartella esistente in una repository Git e ha indicato `https://github.com/PregioSky/RalyaLandsGame.git` come repository remota. Il progetto resta nella stessa cartella.

### Modifiche

- Inizializzata la repository locale sul branch `main`.
- Aggiunto `.gitignore` per cache, cartelle generate da Unreal, file locali degli IDE e configurazioni locali degli agenti; asset, configurazione del progetto e istruzioni condivise restano versionati.
- Aggiunto `.gitattributes`: file `.uasset` e `.umap` gestiti con Git LFS, file di testo normalizzati automaticamente.
- Installati i filtri e gli hook LFS solo per questa repository.
- Impostata l'identità Git locale come `PregioSky`, con l'email fornita dall'utente, senza modificare l'identità globale.
- Configurato `origin` verso la repository indicata, per il primo commit e la pubblicazione su GitHub.
- Assegnata la proprietà della cartella `.git` all'account Windows dell'utente, dopo la creazione iniziale nell'ambiente isolato.
- Spostato il token locale di Android File Server da `Config/DefaultEngine.ini` a `Saved/Config/WindowsEditor/Engine.ini`, escluso da Git, conservando il valore sul computer senza pubblicarlo.
- Aggiornata la guida con le informazioni di versionamento.

### Verifiche

- Git e Git LFS disponibili; repository remota raggiungibile e inizialmente vuota.
- Controlli del contenuto da versionare, delle esclusioni e dei filtri LFS prima del primo commit.
- Verificata la conservazione del token nel file locale e la sua assenza dalla configurazione versionabile.
- L'esito del commit e del push viene verificato con Git al completamento dell'intervento.
- Nessuna modifica al gameplay; nessuna nuova prova PIE necessaria per questa configurazione.
