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

## 014 — 2026-11-15 — Movimento 9 m e mischia a 1,5 m fra i bordi, in stile Baldur's Gate 3

Autore: Claude Code
Tipo: Modifica delle regole
Stato: Budget di 9 m attuale; misura della portata fra i bordi sostituita da 015

### Richiesta e scelte

L'utente ha chiesto valori ispirati a D&D. Claude Code aveva proposto una griglia da 1,5 m con aggancio alle celle e diagonali 5-10-5, e aveva iniziato a scriverla nel controller. Dopo il confronto con Baldur's Gate 3, l'utente ha preferito mantenere il movimento libero e naturale, anche di pochi passi. Il lavoro sulla griglia è stato scartato prima di qualsiasi salvataggio, ricaricando `BP_CombatController` dal disco; non ne resta traccia negli asset.

Scelte dell'utente: 9 m di movimento per turno, con velocità per razza rimandata (Ezio sarà halfling); mischia a 1,5 m misurata fra i bordi delle capsule; arena invariata.

### Modifiche

- `BP_CombatController`: aggiunte le funzioni pure `UnitRadius`, `MeleeCenterRange` (150 cm + raggi delle due capsule) e `InMelee` (distanza fra i centri ≤ `MeleeCenterRange` + 0,5 cm). Sostituiti con `InMelee` i cinque controlli a 100,5 cm in `FindAttackPlan`, `PlanAITurn`, `TryAttack`, `UpdateAI` e `UpdatePendingAttack`.
- `FindAttackPlan`: 24 candidati a passi di 15° (prima 16 a 22,5°), a 2 cm all'interno della portata (prima 98 cm). Con il raggio attuale di 42 cm il cerchio è a circa 232 cm fra i centri; i punti candidati sono più distanti fra loro, da qui l'aumento del loro numero.
- `StartTurn`: budget da 6,0 a 9,0 m.
- Testi di stato e HUD (`BP_CombatHUD`): `/ 9 m`, `Attacco: disponibile (portata 1,5 m)`, `IN PORTATA (1,5 m)`; messaggi sul "quadretto/casella adiacente" sostituiti con la portata di 1,5 m.
- `CONSTITUTION.md`: aggiornate le regole (9 m, 1,5 m fra i bordi) e registrato Baldur's Gate 3 come riferimento di stile: movimento libero, nessuna griglia. `CombatPrototype.md` aggiornata di conseguenza.

### Verifiche

- Lettura completa di controller, HUD e unità con il DSL Blueprint prima delle modifiche; dopo le modifiche, riletti `FindAttackPlan` e il conteggio delle chiamate a `InMelee` (5), con assenza residua di 100,5 / 98 / 22,5 / 6,0.
- Ripristino della griglia scartata verificato: revisione DSL identica a quella iniziale (`5e2bc26cf4`), grafi e variabili originali, nessun pacchetto modificato.
- Controller e HUD compilati con avvisi come errori, senza messaggi, e salvati. Durante le scritture DSL intermedie il log ha mostrato errori transitori `The current value (280.0) of the ' B ' pin is invalid` (sottrazione intera dell'area HUD in `ClickCommand`/`UpdatePreview`); la compilazione finale non li ripresenta.
- PIE, con cursore e clic simulati tramite `IA_SetDestination_Click`:
  - anteprima su Guardia 1 `Percorso ~6.5 m | Residuo ~2.5 m | RAGGIUNGI E ATTACCA`, ispezionata visivamente insieme all'HUD (`9.0 / 9 m`, `portata 1,5 m`);
  - clic: attacco a 233,5 cm fra i centri, cioè 149,5 cm fra i bordi; danno 40 dopo l'impatto; residuo effettivo 2,33 m contro i ~2,5 stimati; nessuno spostamento delle altre unità;
  - limite di portata: con Nephilim spostato via script a 236 cm fra i centri (152 fra i bordi) `InMelee` falso, attacco rifiutato con messaggio; a 234 cm attacco riuscito; secondo attacco nello stesso turno respinto;
  - clic sul terreno oltre budget: anteprima `~13.5 m | OLTRE IL BUDGET`, verde fino a 9 m e rosso oltre, verificata visivamente; percorsi 899,8 cm, residuo 0, velocità finale 0;
  - scontro completo con clic sui nemici: vittoria con HP `[75,50,0,0]`; comandi successivi senza effetto;
  - nuova partita senza attacchi del giocatore: le guardie aggirano il muro (circa 665 cm) e colpiscono a ~149 cm fra i bordi già al primo turno; sconfitta con HP `[0,0,100,100]`; nessun movimento delle guardie fuori turno; corpi visibili e senza collisione.
- Log controllati: nessun `Accessed None`, accesso fuori indice o errore Blueprint runtime durante le prove; presenti solo avvisi preesistenti di `LogAnimation` (animazione additiva `AN_CombatHit` su AnimSingleNodeInstance) e deprecazioni dei miei script Python.
- Non riverificati in questa voce: attraversabilità dei corpi da parte di un'unità in movimento e ordine su un punto completamente circondato. La logica dei corpi e degli ostacoli non è stata modificata.
- PIE fermato al termine.


## 015 — 2026-11-15 — Mischia a 1,5 m fra i centri

Autore: Claude Code
Tipo: Correzione delle regole
Stato: Attuale; sostituisce la misura fra i bordi della voce 014

### Richiesta e scelte

Provando la voce 014, l'utente ha osservato che attaccante e bersaglio apparivano troppo distanti per una mischia. Misurare 1,5 m fra i bordi delle capsule portava i centri a circa 2,33 m, una portata da arma lunga. È stata adottata la definizione di D&D dei quadretti adiacenti: 1,5 m fra i centri. La misura fra i bordi era stata raccomandata da Claude Code: si trattava di una raccomandazione errata, qui corretta.

### Modifiche

- `BP_CombatController`: `MeleeCenterRange` restituisce 150 cm fissi; `InMelee` resta con tolleranza di 0,5 cm. Eliminata la funzione `UnitRadius`, non più usata. `FindAttackPlan` prova quindi i suoi 24 punti a 148 cm dal bersaglio.
- Testi HUD invariati (`portata 1,5 m`). Aggiornati `CONSTITUTION.md` e `CombatPrototype.md`.

### Verifiche

- Riletto `MeleeCenterRange`; verificata l'assenza di riferimenti residui e del grafo `UnitRadius`. Controller compilato con avvisi come errori, senza messaggi, e salvato.
- PIE, con cursore e clic simulati: anteprima su Guardia 1 `Percorso ~7.3 m | Residuo ~1.7 m | RAGGIUNGI E ATTACCA`; attacco a 148,1 cm fra i centri, danno 40, residuo effettivo 1,49 m. Ispezione visiva della cattura durante il colpo: le unità appaiono vicine, in mischia.
- Limite: Nephilim spostato via script a 151 cm dalla Guardia 2, attacco rifiutato; a 150 cm attacco riuscito.
- Nuova partita senza attacchi del giocatore: guardie in movimento per circa 750 cm attorno al muro, attacchi a 148,1 cm, sconfitta con HP `[0,0,100,100]`, nessun movimento fuori turno, corpi visibili e senza collisione; nessun errore runtime pertinente nel log.
- Osservato, non corretto: durante il feedback dell'attacco l'HUD continua a mostrare l'ultima anteprima, perché `UpdatePreview` non la azzera quando `FeedbackTime > 0` e `DrawPathPreview` la nasconde solo quando il testo è vuoto. Il comportamento è preesistente e contraddice la guida, che dichiara l'anteprima nascosta durante il feedback.
- PIE fermato al termine.

## 016 — 2026-11-15 — Anteprima nascosta durante il feedback dell'attacco

Autore: Claude Code
Tipo: Correzione
Stato: Attuale

### Richiesta e scelte

Correzione del difetto osservato nella voce 015, approvata dall'utente: durante l'animazione d'attacco l'HUD continuava a disegnare l'ultima anteprima calcolata, contrariamente alla guida.

### Modifiche

- `BP_CombatController` / `UpdatePreview`: `PreviewValid`, `PreviewEnemy`, `PreviewCanAttack` e `PreviewText` vengono azzerati a ogni aggiornamento, anche quando `FeedbackTime > 0`. In precedenza l'azzeramento avveniva solo fuori dal feedback.
- `BP_CombatHUD`: `DrawPathPreview` e `DrawEnemyLimit` non disegnano nulla mentre `FeedbackTime > 0`, così l'anteprima scompare subito senza attendere l'aggiornamento successivo, che arriva entro circa 0,1 s.

### Verifiche

- Riletto l'inizio di `UpdatePreview` dopo la modifica. Controller e HUD compilati con avvisi come errori, senza messaggi, e salvati.
- PIE: clic di attacco su Guardia 1, poi cursore spostato sul terreno. Durante il feedback il testo di anteprima del controller risulta vuoto da 0,1 s dopo l'inizio, e la cattura a 0,5 s non mostra né il percorso né il riquadro di anteprima. Al termine del feedback l'anteprima ricompare (`Percorso ~3.5 m | Residuo ~0.0 m | OLTRE IL BUDGET`). Nessun errore runtime nel log. PIE fermato.

## 017 — 2026-11-15 — Schede D&D 5e, tiro per colpire, Attacco extra e Attacco furtivo

Autore: Claude Code
Tipo: Funzionalità / Modifica delle regole
Stato: Attuale; sostituisce i 100 HP e i danni fissi 40/25 (002, 011)

### Richiesta e scelte

L'utente ha chiesto schede personaggio e tiro per colpire. Scelte dell'utente: scala D&D 5e reale, Ezio halfling ladro, Nephilim guerriero, livello 5, tiri mostrati come in Baldur's Gate 3.

Scelte di Claude Code, modificabili nella tabella:
- Ezio: stocco +7, 1d8+4, CA 16 (cuoio borchiato e DES +4), HP 38.
- Nephilim: spada lunga e scudo +7, 1d8+6 con stile Duellare, CA 18 (cotta di maglia e scudo), HP 49. La razza non è stata indicata: "Da definire".
- Guardie: valori ispirati al Veterano (HP 58, CA 17, due attacchi +5, 1d8+3), scelti perché due guardie base sarebbero troppo deboli contro un gruppo di livello 5.

Al livello 5 l'Attacco extra del guerriero e l'Attacco furtivo del ladro (3d6) sono stati inclusi come elementi centrali delle classi. Velocità per razza, vantaggio, reazioni, azioni bonus e iniziativa sono rimasti esclusi; il movimento resta 9 m per tutti, come chiesto in precedenza.

### Modifiche

- Nuovi `Data/S_CombatSheet` (classe, razza, HP massimi, CA, bonus per colpire, arma, dadi e bonus di danno, attacchi per azione, dadi furtivi, velocità) e `Data/DT_CombatSheets` (righe `Ezio`, `Nephilim`, `Guardia`).
- `BP_CombatController`:
  - `LoadSheets`, chiamata in `ReceiveBeginPlay` al posto di `HP = [100,100,100,100]`, compila gli array `MaxHP`, `ArmorClass`, `AttackBonus`, `DamageCount`, `DamageSides`, `DamageBonus`, `AttacksPerAction`, `SneakDice`, `SpeedMeters`, `ClassNames`, `Weapons`;
  - `StartTurn` imposta movimento, `AttacksLeft` e `SneakUsed` dalla scheda;
  - `RollAttack` tira d20 e danni (critico con dadi raddoppiati, 1 naturale sempre mancato, furtivo una volta per turno con l'altro alleato vivo entro 1,5 m dal bersaglio) e compone `RollText`;
  - `TryAttack` decrementa `AttacksLeft`; `AttackUsed` diventa vero quando non restano attacchi;
  - `ResolveAttack` riceve colpito, danno e critico: applica il danno con minimo 0, oppure mostra `MANCATO` senza reazione;
  - `UpdateAI` ripianifica se il bersaglio cade e restano attacchi, e il limite di sicurezza del turno sale da 5 a 8 s;
  - `HasAllyNear` e `AttackOdds` aggiungono all'anteprima sul nemico `Colpire N% | XdY+Z` ed eventualmente `+3d6 furtivo`;
  - variabile `ForcedD20`, modificabile sull'istanza, per forzare il d20 nelle prove; vale 0 nell'uso normale.
- `BP_CombatUnit`: variabile `FloatText` ed evento `ShowFloat`, usati per `-N`, `CRITICO -N` e `MANCATO`.
- `BP_CombatHUD`: barre e testi HP sugli HP massimi; pannello con classe, CA, `Attacchi: X / Y | arma` e movimento della scheda; nuova funzione `DrawRollLog` per il riquadro del tiro, visibile 5 s.
- `CONSTITUTION.md` e `CombatPrototype.md` aggiornati.

### Verifiche

- Struttura e righe della tabella rilette dopo la scrittura. Unit, controller e HUD compilati con avvisi come errori, senza messaggi. Tutti e cinque gli asset salvati; nessun pacchetto in sospeso.
- PIE:
  - valori caricati: HP `[38,49,58,58]`, CA `[16,18,17,17]`, attacchi `[1,2,2,2]`, furtivo `[3,0,0,0]`, 9 m; HUD ispezionato visivamente;
  - anteprima su Guardia 1: `RAGGIUNGI E ATTACCA | Colpire 55% | 1d8+4`, coerente con +7 contro CA 17;
  - d20 forzato a 20, Ezio con clic reale: `d20 20 + 7 = 27 contro CA 17 - CRITICO! | Danno: dadi 9 + 4 = 13`, HP guardia 58 → 45, testo `CRITICO -13`, nessun furtivo con Nephilim lontano; cattura ispezionata;
  - Nephilim, posizionato via script accanto alla guardia: d20 10 (totale 17 = CA 17) colpito per 12; d20 9 mancato con `MANCATO` e HP invariati; terzo attacco rifiutato, `Attacchi: esauriti`;
  - turno delle guardie con tiri casuali: due tiri per guardia, contro CA 18 di Nephilim;
  - Ezio accanto alla guardia con Nephilim vicino: anteprima `+3d6 furtivo`; d20 15 colpito, `dadi 5 + 4 + furtivo 3d6: 12 = 21`; dopo l'uso il furtivo non compare più nell'anteprima; cattura ispezionata;
  - nuova partita completa con tiri casuali e clic reali: vittoria al round 5 con HP `[0,29,0,0]`. Ezio è caduto e la Guardia 1 ha poi attaccato Nephilim. I corpi sono senza collisione.
- Log: nessun errore runtime, `Accessed None`, accesso fuori indice o `Scheda mancante`. Presenti solo gli errori transitori `280.0` delle compilazioni intermedie già annotati nella voce 014.
- Non verificati: 1 naturale con un totale che supererebbe comunque la CA (impossibile con i valori attuali; logica controllata solo rileggendo il grafo); critico con furtivo; ripianificazione della guardia dopo la morte del bersaglio con attacchi rimasti, non osservata con certezza nei registri. Il primo script della partita completa ha superato il limite di tempo dello strumento, ma la partita è proseguita in PIE ed è stata completata con un secondo script.
- PIE fermato al termine.

## 018 — 2026-11-15 — Iniziativa e attacchi di opportunità

Autore: Claude Code
Tipo: Funzionalità / Modifica delle regole
Stato: Attuale; sostituisce l'ordine fisso Ezio → Nephilim → Guardia 1 → Guardia 2 (voci precedenti)

### Richiesta e scelte

L'utente ha scelto come passo successivo "iniziativa e attacco di opportunità". Scelte di Claude Code, coerenti con D&D 5e e Baldur's Gate 3:

- Iniziativa d20 + bonus della scheda, tirata una volta a inizio scontro. Bonus da DES: Ezio +4, Nephilim +0, guardie +1. Parità risolte dal bonus più alto, poi a favore del giocatore, poi dall'indice dell'unità.
- Una reazione per round, ricaricata all'inizio del turno della singola unità. L'attacco di opportunità è un solo attacco con l'arma, anche per chi ha Attacco extra.
- Provoca solo il movimento volontario dell'unità attiva fuori da 1,5 m fra i centri, la stessa portata della mischia. Disimpegno e Azione scaltra non sono ancora implementati.
- Il movimento viene interrotto per il feedback e poi riprende verso la stessa destinazione, come in Baldur's Gate 3. Se l'unità cade, il turno passa.
- L'Attacco furtivo resta "una volta per turno" e quindi può applicarsi anche a un attacco di opportunità di Ezio nel turno altrui.

### Modifiche

- `Data/S_CombatSheet`: campo `InitiativeBonus`; `Data/DT_CombatSheets`: Ezio 4, Nephilim 0, Guardia 1.
- `BP_CombatController`:
  - nuove variabili `InitBonus`, `InitRoll`, `TurnOrder`, `TurnPos`, `ReactionAvailable`, `ReachPrev`, `ResumeMotion`;
  - `LoadSheets` legge `InitiativeBonus`; `ReceiveBeginPlay` chiama `RollInitiative` al posto di `Active = 0`;
  - `RollInitiative` e la funzione pura `InitKey`: tiri, ordinamento di `TurnOrder` e riepilogo nel riquadro del tiro per 6 s;
  - `EndTurn` avanza in `TurnOrder`, salta le unità a 0 HP e incrementa `Round` al ritorno alla posizione 0;
  - `StartTurn` ricarica la reazione dell'unità attiva e inizializza `ReachPrev`;
  - `CheckOpportunity`, nel tick dopo `UpdateBudget`, rileva l'uscita dalla portata durante un ordine di movimento; `OpportunityAttack` ferma il movimento, consuma la reazione e tira l'attacco;
  - nel tick, al termine del feedback, `ResumeMotion` riprende `MoveTo(MotionGoal)` oppure chiude il turno se l'unità è caduta;
  - `RollAttack(Attacker, Target)` e `HasAllyNear(Attacker, Target)` ricevono l'attaccante esplicito; `ResolveAttack` riceve `Attacker` come sorgente della reazione animata; `TryAttack` e `AttackOdds` adeguati;
  - `ProvokesOpportunity` (pura) antepone `ATTACCO DI OPPORTUNITA! | ` all'anteprima se un vertice del percorso esce dalla portata di un nemico adiacente con reazione.
- `BP_CombatHUD`: titolo con il round; elenco delle unità in ordine di iniziativa con `>` sull'attiva, `[Init N]`, HP, CA e `Reazione` / `Reazione usata`.
- `CONSTITUTION.md` (regola dell'ordine dei turni) e `CombatPrototype.md` aggiornati.

### Verifiche

- Grafi del controller e dell'HUD riletti dopo la scrittura; controller e HUD compilati con avvisi come errori, stato `BS_UP_TO_DATE`. Salvati controller, HUD, struttura e tabella. `BP_CombatUnit` e la mappa non sono stati modificati in questo intervento.
- Prima delle compilazioni risultava ancora attivo un PIE avviato in precedenza: le ricompilazioni hanno prodotto nel log errori `Accessed None` / `pending kill` dell'istanza vecchia. PIE è stato fermato e nelle sessioni di prova successive non compaiono errori runtime.
- PIE, iniziativa: sei avvii con ordini diversi; ordinamento corretto, parità 6–6 e 2–2 vinte dalle guardie per bonus +1, parità 14–14 fra guardie in ordine di indice. Avvio con Guardia 1 prima: sequenza `[Guardia 1, Nephilim, Ezio, Guardia 2]` per 5 round, round incrementato al ritorno in testa, Ezio sconfitto saltato.
- PIE, attacchi di opportunità (unità posizionate via script, d20 forzato con `ForcedD20`, clic reali):
  - Ezio si allontana dalla Guardia 1: AdO colpito per 4, reazione consumata, movimento ripreso fino a 49 cm dalla destinazione;
  - Ezio adiacente a entrambe le guardie, Guardia 1 senza reazione: anteprima con `ATTACCO DI OPPORTUNITA!`, un solo AdO dalla Guardia 2 (1 naturale, mancato), movimento ripreso; catture dell'anteprima e del colpo ispezionate;
  - clic su Guardia 2 con Guardia 1 adiacente: anteprima con avviso e probabilità; AdO della Guardia 1 durante l'avvicinamento, poi l'attacco in attesa riparte e Ezio colpisce da 149,7 cm (con furtivo, Nephilim era vicino al bersaglio);
  - guardia che lascia Nephilim: AdO di Nephilim colpito per 11; la guardia, oltre il muro, ha poi esaurito il budget senza attaccare. La causa (percorso più lungo di 9 m) è dedotta dalla posizione e dal budget, non registrata direttamente;
  - Guardia 2 portata a 3 HP con una chiamata di prova a `ResolveAttack`: AdO di Nephilim, guardia a 0 HP senza collisione, turno chiuso al termine del feedback, round 2 con Ezio;
  - guardia che lascia Ezio (AdO mancato): ripresa del movimento, arrivo a 148,4 cm da Nephilim e attacchi; la reazione di Nephilim risulta ricaricata all'inizio del suo turno.
- Non verificati: due nemici lasciati nello stesso tick (gestiti in sequenza, logica solo riletta); AdO durante un tratto fra due vertici del percorso senza che un vertice esca dalla portata (l'anteprima non lo segnala, l'esecuzione sì); partita completa con tiri casuali dopo questa modifica.
- PIE fermato al termine.

## 019 — 2026-11-15 — Scatto, Disimpegno e anteprima degli attacchi di opportunità lungo il percorso

Autore: Claude Code
Tipo: Funzionalità / Modifica delle regole
Stato: Attuale; sostituisce il controllo dell'anteprima limitato ai vertici del percorso e ai nemici adiacenti (018)

### Richiesta e scelte

L'utente ha confermato il funzionamento della voce 018 e ha chiesto Scatto e Disimpegno. Scelte di Claude Code, secondo D&D 5e:

- Scatto e Disimpegno usano l'azione del turno, alternativa all'azione Attacco. Sono disponibili solo se non è ancora stato fatto alcun attacco: con Attacco extra, dopo il primo colpo non si può più scattare.
- Scatto aggiunge il movimento della scheda al residuo, anche a movimento parzialmente speso. Disimpegno vale per il resto del turno.
- Comandi: pulsanti dell'HUD sopra Fine turno e tasti Q / E.
- Non inclusi: Azione scaltra di Ezio (azione bonus), uso da parte delle guardie.

Durante le prove è emerso che un percorso lungo passava accanto a una guardia e la provocava senza avviso: l'anteprima della 018 controllava solo i nemici già adiacenti e i vertici del percorso. È stata quindi riscritta.

### Modifiche

- `BP_CombatController`:
  - variabili `TurnSpeed`, `Disengaged`, `ActionName` (categoria Actions), reimpostate in `StartTurn`;
  - `CanUseAction` (pura): turno del giocatore, comandi non bloccati, nessun attacco fatto;
  - `UseDash` e `UseDisengage`: consumano l'azione (`AttackUsed`, `AttacksLeft = 0`), annullano l'eventuale attacco in attesa e aggiornano lo stato; eventi `Key|Q` e `Key|E`;
  - `CheckOpportunity` ignora le uscite dalla portata con `Disengaged`;
  - `ProvokesOpportunity` riscritta: per ogni nemico vivo con reazione campiona il percorso ogni 25 cm entro il budget. Avvisa se, dopo essere entrato entro 1,5 m + 15 cm, il percorso si allontana oltre 1,5 m. Restituisce sempre falso con Disimpegno attivo;
  - testi `ATTACCO GIA USATO` / `Attacco gia usato` sostituiti da `AZIONE GIA USATA` / `Azione gia usata`.
- `BP_CombatHUD`: pulsanti `SCATTO [Q]` e `DISIMPEGNO [E]` con hitbox `Dash` / `Disengage`, grigi e senza hitbox se l'azione non è disponibile; movimento mostrato su `TurnSpeed`; riga `Azione usata: Scatto/Disimpegno`; `ReceiveHitBoxClick` gestisce i nuovi pulsanti.
- `CONSTITUTION.md` (azione del turno) e `CombatPrototype.md` aggiornati.

### Verifiche

- Grafi riletti dopo ogni scrittura; controller e HUD compilati con avvisi come errori, stato `BS_UP_TO_DATE`, salvati. Nessun errore runtime nelle ultime 1.500 righe di log dopo le prove.
- PIE:
  - tasto Q (input simulato): residuo e totale da 9 a 18 m, azione usata, pulsanti grigi. Il successivo E viene rifiutato con messaggio; sul nemico l'anteprima mostra `AZIONE GIA USATA`. Cattura ispezionata;
  - dopo lo Scatto, percorso reale di 15,3 m: arrivo a 44 cm dalla meta con 2,4 m residui. Il percorso passava accanto alla Guardia 1 e ha provocato un attacco di opportunità (mancato), seguito dalla ripresa del movimento;
  - Disimpegno sullo stesso percorso: passaggio a 148 cm dalla Guardia 1 senza attacco, reazioni intatte. Con la Guardia 2 adiacente: allontanamento a 3,47 m senza attacco;
  - anteprima, versione finale: avviso presente sul passaggio accanto alla Guardia 1 (percorso pianificato a 156 cm, eseguito a 148 cm) e con la Guardia 2 adiacente in allontanamento. Avviso assente per il clic su Guardia 1, su Guardia 2 (anche adiacente), per due tratti di terreno vicini e con Disimpegno attivo;
  - al turno seguente (Nephilim) `TurnSpeed` 9, `Disengaged` falso, azione disponibile.
- Il primo tentativo di anteprima, con margine ma senza il controllo di allontanamento, non avvisava: il percorso restava entro la fascia di margine fino al limite del budget. È stato sostituito prima del salvataggio finale.
- Non verificati:
  - clic reale del mouse sui pulsanti HUD: il clic simulato con `LeftMouseButton` non raggiunge le hitbox del Canvas. La gestione è stata provata chiamando `ReceiveHitBoxClick` con `Disengage`, e il tasto Q con input simulato;
  - Scatto rifiutato dopo il primo attacco di Nephilim (logica solo riletta);
  - avvisi superflui dovuti al margine di 15 cm, non misurati sistematicamente.
- Un clic di prova è fallito con "Destinazione occupata" perché il cursore non era valido dopo una cattura; ripetuto, è riuscito. Il messaggio di stato precedente resta visibile dopo un movimento riuscito, comportamento già esistente.
- PIE fermato al termine.

## 020 — 2026-11-15 — Azione, azione bonus, Azione scaltra e Recuperare energie

Autore: Claude Code
Tipo: Funzionalità / Modifica delle regole
Stato: Attuale; integra la 019 (Scatto e Disimpegno ora possono usare anche l'azione bonus)

### Richiesta e scelte

L'utente ha chiesto la logica di azione e azione bonus, rimandando il nascondersi a un passo successivo. Scelte di Claude Code, secondo D&D 5e:

- Risorse separate per azione e azione bonus. Il primo attacco consuma l'azione; gli attacchi di Attacco extra restano disponibili.
- Azione scaltra di Ezio (Ladro 5): Scatto e Disimpegno come azione bonus. Q ed E usano prima l'azione bonus, poi l'azione, quindi due Scatti danno 27 m. Nascondersi, terza opzione dell'Azione scaltra, arriverà con il nascondersi.
- Recuperare energie di Nephilim (Guerriero 5), aggiunta per dare un'azione bonus anche al guerriero: 1d10 + livello, 1 uso per scontro (i riposi non esistono), rifiutata a HP pieni. Tasto R.
- Livello in scheda (guardie 0, creature senza livello) per la cura e per regole future.

### Modifiche

- `Data/S_CombatSheet`: campi `Level`, `CunningAction`, `SecondWindUses`. `DT_CombatSheets`: Ezio 5 / sì / 0, Nephilim 5 / no / 1, Guardia 0 / no / 0.
- `BP_CombatController`:
  - variabili `ActionAvailable`, `BonusAvailable`, `BonusName` e array `Level`, `CunningAction`, `SecondWindLeft`, caricati da `LoadSheets`;
  - `StartTurn` ripristina azione e azione bonus; `TryAttack` consuma l'azione;
  - funzioni pure `CanAct`, `CanUseAction` (ora basata su `ActionAvailable`), `CanUseBonus`, `BonusMovement`, `CanDash`, `CanDisengage`, `CanSecondWind`;
  - `UseDash` / `UseDisengage` riscritte con la scelta azione bonus / azione; nuovo `UseSecondWind` (riquadro del tiro e testo flottante `+N`) ed evento `Key|R`;
  - zona dell'HUD esclusa dai clic di movimento e dall'anteprima: x < 880 invece di 650 (`ClickCommand`, `UpdatePreview`).
- `BP_CombatHUD`: pannello largo 860; pulsanti Scatto e Disimpegno su `CanDash` / `CanDisengage` con suffisso `- bonus`; pulsante `RECUPERARE ENERGIE [R]` (hitbox `SecondWind`) finché resta un uso; righe `Azione:` e `Azione bonus:`; `ReceiveHitBoxClick` esteso.
- `CONSTITUTION.md` e `CombatPrototype.md` aggiornati.

### Verifiche

- Struttura e tabella rilette (esportazione CSV). Controller e HUD riletti e compilati con avvisi come errori, stato `BS_UP_TO_DATE`. Salvati controller, HUD, struttura e tabella. Nessun errore runtime nelle ultime 2.000 righe di log.
- PIE (tasti con input simulato, attacchi con `TryAttack` e d20 forzato, danno di prova con `ResolveAttack`):
  - Ezio: Q → Scatto bonus (18 m, azione disponibile); Q → Scatto azione (27 m); E ed R rifiutati. Catture iniziale e finale ispezionate;
  - Nephilim: R a HP pieni rifiutato. Dopo il danno di prova (49 → 29) un attacco consuma l'azione, poi Q è rifiutato. R cura 1d10 (1) + 5 = +6, l'uso scende a 0, il pulsante scompare e un secondo R è rifiutato. Il secondo attacco di Attacco extra resta disponibile. Cattura ispezionata;
  - Ezio al round 2: E → Disimpegno bonus; attacco riuscito (d20 15, 5 danni); Q rifiutato. Allontanamento dalla guardia a 3,47 m senza attacco di opportunità né avviso nell'anteprima;
  - nuova partita: pulsante `RECUPERARE ENERGIE [R]` attivo nel turno di Nephilim ferito, accanto alle righe `Azione` e `Azione bonus`. Cattura ispezionata.
- Non verificati:
  - clic reale del mouse sui pulsanti HUD (stesso limite della 019);
  - clic sul terreno subito a destra del vecchio pannello (fra 650 e 880 px), ora ignorato.
- Limite noto: con Nephilim il messaggio di Scatto rifiutato cita anche l'azione bonus, sebbene Nephilim non possa usarla per scattare.
- PIE fermato al termine.

## 021 — 2026-11-15 — Nascondersi, vantaggio e rivelazione

Autore: Claude Code
Tipo: Funzionalità / Modifica delle regole
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto il nascondersi "come in D&D", con la possibilità di cambiarlo in seguito. Claude Code ha proposto le opzioni e l'utente ha scelto la versione D&D con linea di vista e vantaggio. Dettagli scelti da Claude Code:

- Nascondersi è un'azione, o un'azione bonus per Ezio (terza opzione dell'Azione scaltra), possibile solo se nessun nemico vivo ha linea di vista.
- Prova d20 + Furtività contro la Percezione passiva più alta fra i nemici vivi. Ezio +10 (DES +4 e Maestria), Nephilim +1 con svantaggio per la cotta di maglia, guardie Percezione passiva 12 (Veterano). Valori nella scheda.
- Effetti: non bersagliabile dalle guardie, nessun attacco di opportunità provocato, vantaggio al prossimo attacco, che attiva anche il furtivo.
- Rivelazione: attaccando, o restando fermo in vista di un nemico. Durante il proprio movimento si resta nascosti, così uscire da dietro la copertura e attaccare dà vantaggio. È un'interpretazione vicina a Baldur's Gate 3 e alla nota del manuale sul DM che può consentire di avvicinarsi senza essere visti; resta modificabile in `RevealCheck`.
- Non inclusi: azione Cercare, oscurità e coperture parziali, guardie che si nascondono.

### Modifiche

- `Data/S_CombatSheet`: campi `StealthBonus`, `StealthDisadvantage`, `PassivePerception`; righe `DT_CombatSheets` aggiornate.
- `BP_CombatController`:
  - array `Hidden`, `StealthBonus`, `StealthDisadv`, `PassivePerception` (categoria Hide); `LoadSheets` e `RollInitiative` li inizializzano;
  - `IsSeenBy` (pura): due `LineTraceSingle` su Visibility dagli occhi del nemico alla testa e al centro del bersaglio, ignorando le unità; `SeenByEnemies` e `CanHide` (pure);
  - `UseHide` (azione bonus se `BonusMovement`, altrimenti azione), con riquadro del tiro, testo flottante `NASCOSTO` / `VISTO` ed evento `Key|N`;
  - `RevealCheck` nel tick dopo `UpdatePendingAttack`: rivela i morti e le unità ferme viste da un nemico (`SCOPERTO`);
  - `RollAttack`: vantaggio per l'attaccante nascosto (due d20, testo `d20 con vantaggio (a, b)`), furtivo con vantaggio o alleato vicino, rivelazione dopo l'attacco (`| rivelato`);
  - `AttackOdds`: probabilità con vantaggio `1 - (1 - p)²` e furtivo anche con vantaggio;
  - `PlanAITurn` esclude i nascosti; messaggio "Nessun bersaglio visibile e raggiungibile";
  - `CheckOpportunity` e `ProvokesOpportunity` ignorano l'unità attiva nascosta.
- `BP_CombatHUD`: pulsante `NASCONDERSI [N]` (con `- bonus`, hitbox `Hide`) nella seconda colonna; righe Azione / Azione bonus spostate più in basso; `(nascosto)` accanto al nome sopra l'unità; `ReceiveHitBoxClick` esteso.
- `CONSTITUTION.md` e `CombatPrototype.md` aggiornati.

### Verifiche

- La prima scrittura del DSL è fallita per una parentesi mancante in `UseHide`; le modifiche di cablaggio erano già applicate. Corretta e riscritta; controller e HUD riletti e compilati con avvisi come errori, stato `BS_UP_TO_DATE`. Salvati controller, HUD, struttura e tabella. Nessun errore runtime nelle ultime 2.500 righe di log.
- PIE (tasti con input simulato, d20 forzato, clic reali):
  - posizioni iniziali: nessuna guardia vede gli alleati dietro il muro;
  - Ezio: N → `Furtivita d20 10 + 10 = 20 contro Percezione passiva 12 - NASCOSTO` come azione bonus; secondo N → "Gia nascosto";
  - anteprima su Guardia 1: `Colpire 80% (vantaggio) | 1d8+4 +3d6 furtivo`; catture con etichetta `(nascosto)` e pulsante grigio ispezionate;
  - clic sulla guardia: Ezio resta nascosto per tutto il percorso attorno al muro, poi `d20 con vantaggio (10, 10) ... COLPITO | ... furtivo 3d6: 12 = 22 | rivelato`. Dopo l'attacco N è rifiutato;
  - Nephilim: `Furtivita d20 con svantaggio 15 + 1 = 16 ... NASCOSTO` come azione. Con entrambi nascosti le guardie chiudono subito il turno senza bersaglio;
  - Ezio esce allo scoperto oltre il muro: resta nascosto in movimento anche quando una guardia lo vede, ed è scoperto all'arresto (1,35 s);
  - turno delle guardie: la Guardia 2 attacca Ezio, visibile; girando attorno al muro vede Nephilim fermo e lo scopre.
- Non verificati:
  - fallimento della prova (logica riletta; con Ezio +10 fallisce solo con 1);
  - attacco di opportunità di un reattore nascosto;
  - critico con vantaggio;
  - clic reale del mouse sul pulsante (stesso limite delle voci 019–020);
  - casi di linea di vista con i blocchi laterali bassi (altezza non misurata).
- PIE fermato al termine.

## 022 — 2026-11-15 — Cono di vista delle guardie

Autore: Claude Code
Tipo: Funzionalità / Modifica delle regole
Stato: Attuale

### Richiesta e scelte

L'utente ha confermato che il Nascondersi della voce 021 funziona, compreso il caso Disimpegno bonus più Nascondersi (verificato senza modifiche: il blocco segnalato dipendeva da azione già usata o da una guardia in vista). Ha poi chiesto di creare il cono di vista. D&D 5e non prevede l'orientamento; il cono è una regola della casa ispirata a Baldur's Gate 3. Valori scelti da Claude Code e modificabili nella scheda:

- angolo totale di 120°, distanza di 18 m, uguali per tutte le unità;
- il cono segue la rotazione dell'attore. Le guardie partono rivolte verso gli alleati (yaw 180); poi mantengono la direzione dell'ultimo movimento, attacco o colpo ricevuto, già gestita dagli eventi esistenti;
- il cono entra in `IsSeenBy`. Così Nascondersi, `CanHide`, `RevealCheck` e l'anteprima lo applicano senza logiche duplicate;
- coni sempre visibili durante lo scontro, rossi quando vedono l'unità alleata attiva.

### Modifiche

- `Data/S_CombatSheet`: campi `VisionAngle` e `VisionRange` (float); `DT_CombatSheets` impostato a 120 / 18 per Ezio, Nephilim e Guardia.
- `BP_CombatController`:
  - array `VisionAngle` e `VisionRange` (categoria Hide), caricati in `LoadSheets`;
  - `IsSeenBy`: oltre alle due tracce richiede distanza XY ≤ `VisionRange` × 100 e prodotto scalare fra la direzione dell'osservatore e quella normalizzata del bersaglio ≥ cos(`VisionAngle`/2);
  - `ReceiveBeginPlay`: le due guardie vengono ruotate a yaw 180 dopo lo spawn.
- `BP_CombatHUD`: nuova funzione `DrawVisionCones`, chiamata per prima in `ReceiveDrawHUD`. Per ogni guardia viva esegue 25 `LineTraceSingle` sul canale Visibility a +60 cm, ignorando le unità, e disegna l'arco tagliato dai muri, i bordi e raggi interni tenui, proiettati a 12 cm da terra.
- `CombatPrototype.md` e `CONSTITUTION.md` aggiornati.

### Verifiche

- Controller e HUD compilati con avvisi come errori, stato `BS_UP_TO_DATE`. Grafi `IsSeenBy`, `LoadSheets` e `DrawVisionCones` riletti. Salvati controller, HUD, struttura e tabella.
- PIE, con chiamate dirette a `IsSeenBy` e spostamento di Nephilim (non attivo):
  - posizioni iniziali: guardie a yaw 180, nessuna vede Ezio dietro il muro;
  - Nephilim 2 m davanti alle guardie: visto da entrambe;
  - dietro le guardie: non visto;
  - Guardia 1, 70° fuori asse: non visto; 50°: visto;
  - Guardia 1 girata verso Ezio: Ezio visto, cono rosso, Nascondersi grigio (`CanHide` falso).
- Un primo tentativo di spostare Ezio, unità attiva, è stato falsato dal controllo del budget e non conta come prova.
- Round completo con turni automatici: le guardie aggirano il muro, attaccano e restano girate verso il bersaglio; il round 2 parte regolarmente. Nessun errore runtime nel log, a parte un nome di toolset errato in un mio script.
- Catture PIE ispezionate:
  - coni tagliati dal muro centrale;
  - colore giallo e rosso;
  - raggi interni attenuati dopo le prime catture, per ridurre la confusione visiva; la cattura del round 2 mostra la versione attenuata. A distanza di 18 m i coni coprono comunque gran parte dell'arena.
- Non verificati:
  - clic reali del giocatore che portano Ezio fuori dal cono per nascondersi;
  - leggibilità con camera e risoluzioni diverse;
  - copertura dei blocchi bassi (il cono passa sopra gli ostacoli sotto 1,6 m, come la linea di vista).
- PIE fermato al termine.

## 023 — 2026-11-15 — Coni di vista attivabili e disattivabili

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto di poter attivare e disattivare i coni della voce 022. Claude Code ha scelto il tasto V e una riga di stato nel pannello, senza un nuovo pulsante, perché lo spazio dei pulsanti è già occupato. I coni sono visibili all'avvio, come nella voce 022. L'interruttore cambia solo il disegno; le regole di vista restano sempre attive.

### Modifiche

- `BP_CombatController`: variabile `HideCones` (bool, falsa di default, categoria Hide) ed evento `Key|V` che la inverte.
- `BP_CombatHUD`:
  - `DrawVisionCones` disegna solo se `HideCones` è falsa;
  - nuova riga gialla `Coni di vista [V]: visibili` / `nascosti` a 640, SizeY − 60, sotto Azione bonus.
- `CombatPrototype.md` aggiornato.

### Verifiche

- Controller e HUD compilati con avvisi come errori, stato `BS_UP_TO_DATE`. Evento `Key|V` e riferimenti a `HideCones` riletti nel DSL. Asset salvati.
- PIE con il tasto V simulato:
  - `HideCones` passa da falso a vero e torna falso;
  - la cattura con i coni nascosti, ispezionata, mostra l'arena senza coni e la riga `Coni di vista [V]: nascosti`;
  - nessun errore runtime Blueprint nel log.
- Nella stessa cattura la riga di stato mostrava `Passaggio occupato: scegli un'altra destinazione.` senza clic eseguiti. La causa non è stata indagata e non sembra legata a questa modifica.
- Non verificati: pressione reale del tasto da tastiera e cattura ispezionata dopo la seconda pressione (verificato solo il valore della variabile).
- PIE fermato al termine.

## 024 — 2026-11-15 — Modello Meshy di Ezio, Animation Blueprint e velocità per unità

Autore: Claude Code
Tipo: Funzionalità / Asset
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto di importare Ezio Tosswick da `SourceArt/Characters/Ezio/` (export Meshy) e di sostituire il suo manichino nel prototipo, riusando movimento al clic, camera e turni. Il combattimento è in `Lvl_CombatPrototype`, quindi Ezio è stato integrato lì.

Scelte:

- Ezio è il pawn predefinito di `BP_CombatGameMode`; le altre unità sono generate dal controller con la stessa classe `BP_CombatUnit`. Per limitare le modifiche a Ezio, Claude Code ha creato la sottoclasse `BP_CombatUnit_Ezio` e cambiato solo il pawn predefinito del GameMode del prototipo, non quello globale.
- Le clip sono in-place: i fianchi si spostano al massimo di 1–2 cm. Le velocità naturali, misurate dal piede in appoggio, sono Walk ≈ 88 cm/s e Run ≈ 290 cm/s, mentre tutte le unità si muovevano a 600 cm/s. All'utente sono state proposte tre opzioni; ha scelto **Ezio a 290 cm/s**, con la Run a 1x e le altre unità invariate.
- Animation Blueprint: BlendSpace 1D sulla velocità effettiva (Idle 0, Walk 88, Run 290). Il primo tentativo era un Blend Poses by int con PlayRate dinamico, ma il pin PlayRate del Sequence Player non si è potuto esporre via script; è stato sostituito dal BlendSpace prima del salvataggio finale.
- Run e Attack: nel prototipo non esiste un'azione "corsa" distinta (Scatto raddoppia il budget, non la velocità), quindi la Run è legata alla velocità di spostamento di Ezio. Attack usa `Axe_Spin_Attack` (ritorna al punto di partenza) a 2x, circa 1,25 s, vicino ai tempi di feedback esistenti. `Rising_Flying_Kick` è stato scartato perché sposta i fianchi di circa 1 m in avanti.
- Clip mancanti: **Hit** e **Death**. `Fall4` termina in piedi e non è una morte.

### Modifiche

- Importazione Interchange in `/Game/Characters/Ezio/`:
  - `SK_Ezio` dal FBX `character_mixamo` (radice `SmartRigArmature` con scala 100, 67 ossa Mixamo, altezza circa 110 cm);
  - `SK_Ezio_Skeleton` dedicato e `SK_Ezio_PhysicsAsset`;
  - texture incorporate nel FBX, identiche per dimensione (2048²) ai PNG separati, rinominate `T_Ezio_BaseColor/Normal/Roughness/Metallic`;
  - roughness e metallic portate a lineari con compressione Masks.
- `M_Ezio`: materiale PBR con quattro TextureSampleParameter2D (BaseColor, Normal, Roughness e Metallic in Masks), usato con skeletal mesh e assegnato allo slot `BakedMaterial`. Sostituisce il materiale Phong legacy creato da Interchange, che usava la roughness come shininess e ignorava la metallic; quel materiale non salvato è stato eliminato.
- `Animations/`: 14 `A_Ezio_*` importate solo come animazioni su `SK_Ezio_Skeleton`, tramite pipeline Interchange con `import_only_animations` e senza mesh: Idle_02, Walking, Running, Axe_Spin_Attack, Rising_Flying_Kick, Fall4, Jump_Over_Obstacle_2, Slow_Bar_Hang_Left, Walk_Backward, Walk_Backward_with_Grenade, Walk_Backward_with_Gun_1_inplace, Walk_Left_with_Gun_inplace, Cautious_Crouch_Walk_Forward_inplace, Cautious_Crouch_Walk_Left_inplace. Lo strumento MCP `SkeletalMeshToolset.ImportFile` non importava i FBX privi di mesh.
- `Animations/BS_Ezio_Locomotion` (BlendSpace 1D, asse Speed 0–300).
- `ABP_Ezio`: AnimGraph `GetSpeed → BlendSpacePlayer(BS_Ezio_Locomotion) → Output Pose`. EventGraph `BlueprintUpdateAnimation`: `Speed = VectorLengthXY(GetVelocity(TryGetPawnOwner))`, 0 senza pawn.
- `BP_CombatUnit`:
  - variabili `MoveSpeed` (600), `AttackPlayRate` (1), `AttackAnim` / `HitAnim` / `DeathAnim`, con default sulle copie `AN_Combat*`;
  - `PlayAttackFeedback` e `PlayHitFeedback` le usano; attesa = durata della clip / velocità + 0,02 s, pari ai 1,02 e 0,72 s precedenti; una clip vuota salta l'animazione.
- `BP_CombatUnit_Ezio` (nuovo): mesh `SK_Ezio` a −98 cm (suola a terra con il capsule a z 98,15), `ABP_Ezio`, `MoveSpeed` 290, `AttackAnim` Axe Spin a 2x, Hit/Death vuote.
- `BP_CombatController`:
  - nuova funzione pura `UnitSpeed(Index)`;
  - `StopUnitMotion` e `UpdateMotionProgress` usano la velocità dell'unità al posto di `CruiseSpeed`, che resta come ripiego.
- `BP_CombatGameMode`: pawn predefinito `BP_CombatUnit_Ezio`.
- `CombatPrototype.md` e `CONSTITUTION.md` aggiornati. Nessuna modifica a `Lvl_TopDown`, al GameMode globale o agli asset del template.

### Verifiche

- `ABP_Ezio`, `BP_CombatUnit`, `BP_CombatUnit_Ezio`, `BP_CombatController` e `BP_CombatGameMode` compilati con avvisi come errori: stato `BS_UP_TO_DATE`. Grafi riletti dopo le modifiche: feedback, `StopUnitMotion`, `UnitSpeed`, AnimGraph ed EventGraph. Salvati tutti gli asset in `/Game/Characters/Ezio/` e i quattro Blueprint del prototipo; nessun redirector rimasto.
- Materiale: la prima compilazione di `M_Ezio` falliva (campionatore Linear Grayscale con texture Masks); corretta con il campionatore Masks, statistiche valide. Le miniature di mesh e materiale, ispezionate, mostrano Ezio texturizzato.
- PIE (`Lvl_CombatPrototype`):
  - pawn `BP_CombatUnit_Ezio_C`, mesh `SK_Ezio`, anim `ABP_Ezio_C`;
  - `MaxWalkSpeed` 290 per Ezio e 600 per le guardie;
  - l'avanti della testa coincide con l'avanti dell'attore (+X).
- Movimento con `MoveTo`, lo stesso percorso del clic sul terreno: accelera a 290, rallenta, si ferma a 55 cm dal punto (soglia esistente) e `Speed` torna a 0 (Idle). Budget da 9,0 a 5,0 m, nessun movimento residuo.
- Piedi:
  - prima della correzione la suola era a circa +3 cm; dopo, a mesh −98, le punte in Idle sono a 2,8–3,1 cm (bone interno alla scarpa);
  - in corsa, con il piede in appoggio, la punta scorre a 12–30 cm/s contro 290 cm/s del corpo: scivolamento minimo.
- Catture ravvicinate ispezionate (Idle, corsa, attacco; camera avvicinata solo in PIE, nulla salvato): texture corrette, proporzioni da halfling accanto alla guardia, posa di corsa orientata nella direzione del moto. Dopo `PlayAttackFeedback` la mesh torna in modalità Animation Blueprint.
- Round completo: le guardie colpiscono Ezio (38 → 30 HP) e Nephilim con le clip del manichino; nessun errore runtime Blueprint nel log.
- Non verificati:
  - clic reale del mouse sul terreno (usato `MoveTo`, chiamato da `ClickCommand`);
  - attacco di Ezio tramite clic su una guardia (animazione provata con chiamata diretta);
  - sconfitta di Ezio (senza clip Death resta in piedi);
  - Walk osservata visivamente: appare solo nell'accelerazione e nella frenata.
- PIE fermato al termine.

## 025 — 2026-11-15 — Zoom morbido con rotellina e inclinazione progressiva

Autore: Codex
Tipo: Funzionalità / Camera
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto uno zoom morbido fra circa 4–5 m e i 22 m della vista esistente, mantenuto al cambio turno. Ha scelto di abbassare progressivamente l'inclinazione avvicinandosi, come in Baldur's Gate 3.

La proposta iniziale descriveva la camera ferma su Ezio, ma la verifica in PIE ha mostrato che segue l'unità posseduta. L'utente ha confermato di **mantenere il seguito dell'unità attiva**. Non sono state ripristinate le vecchie regole del prototipo: schede, iniziativa, reazioni, azioni bonus, furtività e modello Meshy della voce 024 sono stati preservati.

Scelte di implementazione: limite vicino 450 cm, lontano 2200 cm; passo proporzionale 0,85 e inverso; interpolazione `FInterpTo` a velocità 8. Pitch da −25° vicino a −50° lontano, yaw mantenuto. La prova ravvicinata mostrava Ezio parzialmente coperto dall'HUD: il punto d'inquadratura viene abbassato progressivamente fino a 80 cm, tornando all'offset zero nella vista lontana.

### Modifiche

- `BP_CombatController`: categoria `Camera` con `CameraZoomReady`, distanza corrente e destinazione, limiti, fattore del passo, velocità d'interpolazione, pitch vicino/lontano e `CameraFocusNearZ`.
- Nuove funzioni `InitializeCameraZoom`, `ChangeCameraZoom(Direction)` e `UpdateCameraZoom(Delta)`.
- Eventi `Key|MouseScrollUp` e `Key|MouseScrollDown`: verso l'alto avvicina, verso il basso allontana.
- Inizializzazione dopo la creazione delle unità, prima di `StartTurn`; aggiornamento nel tick indipendente dai controlli di azione e di feedback.
- Distanza, inclinazione e offset applicati agli Spring Arm di tutte le unità dalla stessa impostazione: il cambio di soggetto non resetta lo zoom. Una nuova partita riparte da 22 m.
- Guida e costituzione aggiornate. Nessuna modifica al modello o alle animazioni di Ezio, alle schede, al GameMode o agli altri Blueprint di combattimento in questo intervento. Le modifiche preesistenti di Claude Code sono state conservate; nessun commit o push eseguito.

### Verifiche

- Controller compilato con avvisi trattati come errori e salvato. Riletti eventi e grafi: un evento per ciascun verso della rotellina, una chiamata di aggiornamento nel tick; `LoadSheets`, `CheckOpportunity` e `RevealCheck` ancora presenti.
- PIE, input simulato: un passo verso l'interno porta la destinazione da 2200 a 1870 cm.
- Passaggio morbido misurato: partendo da 1870 cm, il comando imposta la destinazione a 1589,5 senza cambiare subito la distanza corrente; dopo circa 0,04 s la distanza è circa 1816 cm, intermedia.
- Rotellina reale del viewport tramite `SlateInspector.Scroll`: verificati entrambi i versi; destinazione 2200 → 1870 → 2200 cm. Risorse di Nephilim invariate: 9 m, azione disponibile, azione bonus disponibile e 2 attacchi.
- Comandi ripetuti rispettano i limiti 450 e 2200 cm. Inquadratura vicina ispezionata con Ezio visibile per intero sopra l'HUD; pitch circa −25°. Vista lontana: pitch circa −50° e offset verticale tornato a zero.
- Cambio turno Ezio → Guardia 1 → Nephilim: camera sul soggetto attivo, distanza 450 cm e pitch −25° mantenuti; tutte le camere condividono la stessa impostazione.
- Nel log restano errori precedenti dell'editor (ultimi rilevati alle 20:07 UTC, relativi a unità in distruzione e `AddMappingContext`); non sono errori introdotti dallo zoom. Nessun nuovo errore runtime rilevato durante le prove di questa voce, svolte successivamente.
- Non ripetuta l'intera suite delle regole D&D delle voci 017–024: le prove di questo intervento riguardano la camera, gli input, le risorse e il passaggio dei turni.
- PIE fermato al termine.

## 026 — 2026-11-15 — Ezio V2 a braccia dritte e modello dedicato di Nephilim

Autore: Codex
Tipo: Funzionalità / Personaggi / Pulizia / Verifica
Stato: Attuale

### Richiesta e scelte

Sostituire Ezio con la V2 e il manichino di Nephilim con il suo modello, tramite Unreal MCP nell'editor, mantenendo party, collisioni, movimento al clic e turni. Mappa effettiva ispezionata: `/Game/CombatPrototype/Lvl_CombatPrototype`; `Lvl_TopDown` resta la sorgente storica del template.

- Sorgente Ezio esclusivamente `SourceArt/Characters/Ezio/v2/Meshy_AI_Wily_Halfling_Wanderer_20261115_animated_mixamo/`: FBX `..._character_mixamo.fbx`, 11 FBX `..._without_skin_*.fbx`, quattro PNG separati (base color, normal, roughness, metallic).
- Sorgente Nephilim `SourceArt/Characters/Nephilim/Meshy_AI_Ironclad_Wanderer_20261115_animated_mixamo/`: FBX `..._character_mixamo.fbx`, 14 FBX di animazione senza skin e quattro PNG separati.
- Confronto V1/V2 su import temporaneo: 67 ossa, medesimi nomi e genitori, ordine interno differente e trasformazioni di riferimento diverse per braccia/avambracci/mani. Riutilizzato `SK_Ezio_Skeleton` aggiornando la posa di riferimento durante il reimport; confronto successivo dei sei segmenti delle braccia con lo Skeleton V2 temporaneo riuscito. Mesh alta ancora 110 cm. Nessun retarget al manichino.
- Conservati i percorsi finali di Ezio: reimport della mesh, delle texture e delle cinque clip comuni V1/V2. Importate anche le sei clip aggiuntive V2. Nephilim usa uno Skeleton proprio; mesh alta circa 190 cm. Entrambi scala 1, mesh Z −98 cm e yaw −90°.

### Modifiche

- `/Game/Characters/Ezio/`: aggiornati `SK_Ezio`, Skeleton, PhysicsAsset e quattro `T_Ezio_*`; mantenuti `M_Ezio`, `ABP_Ezio` e `Animations/BS_Ezio_Locomotion`. Tutte le sorgenti di mesh, texture e Animation Sequence finali risultano sotto `Ezio/v2/`.
- `/Game/Characters/Nephilim/`: `SK_Nephilim`, `SK_Nephilim_Skeleton`, `SK_Nephilim_PhysicsAsset`, `M_Nephilim`, quattro `T_Nephilim_*`, `ABP_Nephilim`, `Animations/BS_Nephilim_Locomotion` e 14 `A_Nephilim_*`.
- Tutti gli FBX di animazione importati come sole Animation Sequence sullo Skeleton scelto, senza mesh per clip. Materiali PBR con quattro texture del rispettivo personaggio; base color sRGB, normal non sRGB/Normalmap, roughness e metallic non sRGB/Masks. Compilazione materiali senza errori.
- Blend Space guidati dalla velocità orizzontale effettiva: Ezio Idle 0 / Walk 75 / Run 290 cm/s (Run 1,09x); Nephilim Idle 0 / Walk 167 / Run 600 cm/s. Velocità del gameplay mantenute 290 e 600 cm/s; budget delle schede invariato. Le clip sono in-place; le soglie sono state confrontate con lo spostamento dei piedi nelle clip.
- `BP_CombatUnit_Ezio` conserva identità, riferimenti e classe esistenti. Nuovo figlio `/Game/CombatPrototype/BP_CombatUnit_Nephilim`; nel `ReceiveBeginPlay` di `BP_CombatController` cambia solo la classe del primo spawn aggiuntivo (indice party 1). Gli spawn delle due guardie restano `BP_CombatUnit`.
- Attacchi: Ezio `A_Ezio_Axe_Spin_Attack` (2,5 s a 2x), Nephilim `A_Nephilim_Attack` (2,8 s a 2,24x), entrambi circa 1,25 s per rientrare nel feedback esistente di 1,35 s. Ritorno al proprio Animation Blueprint dopo la clip. Hit/Death vuoti per entrambi perché non forniti: non si usano le clip incompatibili del manichino; alla sconfitta rimangono in Idle. Non aggiunte armi separate o nuove azioni per le clip accessorie.
- Guida `CombatPrototype.md` e regole correnti in `CONSTITUTION.md` aggiornate. Preservati sorgenti originali, template condiviso, schede e modifiche preesistenti. Nessun commit o push.

### Pulizia e riferimenti

- Prima e dopo le modifiche cercati i referencer di Ezio nell'intero progetto, caricandoli per conferma. Mesh e Animation Blueprint finali referenziati soltanto da `BP_CombatUnit_Ezio`; i corrispondenti di Nephilim soltanto da `BP_CombatUnit_Nephilim`. Skeleton e Blend Space di Ezio restano usati dagli asset finali aggiornati.
- Eliminate tramite l'editor le nove clip V1 senza referencer, sotto `Ezio/Animations/`, prefisso `A_Ezio_`: `Cautious_Crouch_Walk_Forward_inplace`, `Cautious_Crouch_Walk_Left_inplace`, `Fall4`, `Jump_Over_Obstacle_2`, `Rising_Flying_Kick`, `Slow_Bar_Hang_Left`, `Walk_Backward_with_Grenade`, `Walk_Backward_with_Gun_1_inplace`, `Walk_Left_with_Gun_inplace`.
- Eliminata nell'editor l'intera cartella temporanea `/Game/Characters/_IntegrationAudit/` (mesh, Skeleton, PhysicsAsset e quattro texture embedded), dopo controllo riferimenti. Nessun riferimento esterno da preservare. Eseguito fixup nelle cartelle dei due personaggi e sull'asset della nuova unità; audit finale: nessun redirector nelle due cartelle, temporanea assente.
- Restano 21 asset finali di Ezio (11 clip) e 24 di Nephilim (14 clip). Nessun asset esclusivamente V1 conservato per referencer esterni: i nomi principali storici sono riutilizzati per i contenuti aggiornati V2. `SourceArt` non cancellata.

### Verifiche

- Compilati `ABP_Ezio`, `ABP_Nephilim`, `BP_CombatUnit_Ezio`, `BP_CombatUnit_Nephilim` e `BP_CombatController` con avvisi come errori. Salvati nell'editor i 45 asset finali dei personaggi e i tre Blueprint di gameplay interessati. Riletto il ramo di inizializzazione: party e guardie corretti, `LoadSheets`, iniziativa e camera preservati.
- PIE visivo: texture dei due personaggi, scala relativa halfling/umano, orientamento, piedi al pavimento e animazioni. Esaminata la Walk V2 di Ezio anche isolatamente tramite riproduzione temporanea sul componente in PIE, ripristinando poi l'Animation Blueprint; non osservate torsioni anomale di braccio o polso. Nessuna modifica di prova persistente.
- Input Enhanced Input del clic nel viewport su terreno per entrambi: movimento reale, consumo del budget e ritorno a `Speed=0` / Idle. Dopo pulizia Ezio si sposta da `(-350,-150)` a circa `(-272,-301)`, residuo 7,30 m; Nephilim da `(-350,150)` a circa `(-349,397)`, residuo 6,53 m, velocità campionata durante il tragitto circa 338 cm/s e zero all'arrivo.
- Turni: Ezio controllabile, guardie automatiche, Nephilim posseduto come `BP_CombatUnit_Nephilim_C`, movimento nel suo turno e Fine turno funzionanti; più passaggi e round verificati. Attese esplicitamente le ricostruzioni del NavMesh prima dei comandi finali. Nessun nuovo sistema di selezione fuori turno introdotto: è mantenuto il controllo dell'unità attiva.
- Attacco di Ezio tramite `TryAttack`: consumo dell'attacco, danno alla guardia e ritorno ad `ABP_Ezio`. Attacco di Nephilim tramite clic sul nemico in portata: attacchi da 2 a 1, clip dedicata, danno 7 alla guardia e ritorno ad `ABP_Nephilim`. Dopo la regolazione finale del play rate, verificato separatamente il feedback animato di Nephilim: SingleNode durante il colpo e AnimationBlueprint dopo circa 1,4 s.
- Controllo delle sorgenti di 35 asset importati finali: tutte corrette, nessuna sorgente Ezio fuori da `v2/`. Controllati nuovamente referencer, assenza dei temporanei e redirector dopo la pulizia; nuova apertura PIE carica entrambe le classi e tutti i riferimenti finali.
- Log: nessun nuovo `Blueprint Runtime Error`/`Accessed None` nelle prove dell'integrazione. Interchange segnala bind pose FBX non valide e applica automaticamente la posa a tempo zero; risultato controllato visivamente in PIE. Rimane l'avviso preesistente sulle reazioni additive `AN_CombatHit` delle guardie riprodotte come SingleNode, con limite per build cooked: non modificato il template né ampliato questo intervento alle reazioni delle guardie.
- Non ripetuta l'intera suite D&D, la conclusione completa dello scontro o una build packaged; non ispezionate visivamente tutte le clip accessorie importate. Hit/Death dei due personaggi assenti come indicato sopra.
- PIE fermato al termine; editor sulla mappa del prototipo.

## 027 — 2026-11-15 — Rotazione della camera con trascinamento del tasto centrale

Autore: Codex
Tipo: Funzionalità / Camera
Stato: Attuale

### Richiesta e scelte

L'utente ha confermato l'aggiunta della rotazione tenendo premuto il tasto centrale e trascinando il mouse, mantenendo orientamento e zoom al cambio turno. Rotazione orizzontale libera; l'inclinazione resta legata allo zoom già esistente. Nessun consumo di risorse di combattimento.

### Modifiche

- Solo `BP_CombatController` fra gli asset: nuove variabili nella categoria Camera `CameraYaw`, `CameraRotateSensitivity` (0,25 gradi/pixel), `CameraDragging`, `CameraDragLastX`; funzione `UpdateCameraRotation` chiamata nel tick prima di `UpdateCameraZoom`, fuori dai blocchi di turno e feedback.
- Il tasto centrale e la validità della posizione mouse sono letti ogni tick. Il primo frame del trascinamento registra la posizione; i successivi aggiungono lo spostamento orizzontale allo yaw, normalizzato a ±180°. Al rilascio o posizione non valida lo stato di trascinamento si azzera.
- `InitializeCameraZoom` inizializza lo yaw dal primo Spring Arm (45° nel template). `UpdateCameraZoom` applica lo yaw condiviso a tutti gli Spring Arm, che hanno già rotazione assoluta: il movimento o la rotazione dei pawn non cambiano l'orientamento della camera. Zoom, pitch e offset esistenti preservati.
- `ClickCommand` ignora i comandi mentre il tasto centrale è premuto, per evitare ordini durante la rotazione. Le azioni esistenti non vengono consumate dalla camera.
- Aggiornate guida corrente e costituzione. Nessuna modifica a mesh, animazioni, mappa, schede o regole D&D; nessun commit/push.

### Verifiche

- PIE su `Lvl_CombatPrototype`, con vero input Slate del viewport: trascinamento centrale di 160 pixel a destra porta lo yaw 45° → 85°; 120 pixel a sinistra lo porta 85° → 55°.
- Dopo il rilascio, movimento del mouse senza tasto centrale lascia lo yaw a 85° e `CameraDragging=false`. Un nuovo trascinamento riparte senza salto dal nuovo punto iniziale.
- Durante il trascinamento invocato `ClickCommand`: `MotionRequested=false` e residuo ancora 9 m. Zoom provato contemporaneamente: distanza da 2200 a 1870 cm, inclinazione circa −45,29°, yaw mantenuto e identico sulle quattro camere.
- Fine turno Ezio → Guardia 1 conserva yaw 55° e zoom 1870 cm. Trascinamento durante il turno automatico aggiunge 20°; passando a Nephilim restano yaw 75°, distanza 1870 cm e stessa inclinazione sulle quattro camere. Camera sul pawn attivo `BP_CombatUnit_Nephilim_C`.
- Immagine del viewport ispezionata dopo la rotazione. Compilazione finale del controller con avvisi trattati come errori riuscita; asset salvato. Nessun nuovo errore runtime pertinente nel log; gli errori storici precedenti restano nel file.
- Non ripetuta la suite completa D&D o una build packaged. PIE fermato al termine.

## 028 — 2026-11-22 — Ripristinato l'avvio PIE dopo errori nei default interi

Autore: Codex
Tipo: Correzione / Verifica
Stato: Attuale

### Richiesta e scelte

L'utente ha mostrato il dialogo di errori di compilazione prima del Play, dopo aver riaperto `Lvl_CombatPrototype`. Compilazione diagnostica e log hanno confermato 17 pin di sottrazione intera con default scritti come numeri decimali: `280.0`, `277.0`, ecc. Il compilatore richiede una rappresentazione intera. Il difetto era negli asset salvati di controller e HUD; non è stato attribuito a una specifica versione del motore o a un aggiornamento, non verificati come causa. Le verifiche storiche riuscite delle voci precedenti non avevano dimostrato questa compilazione dopo ricaricamento.

### Modifiche

- Cercati in tutti i grafi dei due Blueprint i nodi `K2Node_PromotableOperator` con pin di tipo Integer e default decimale senza parte frazionaria; ispezionati tipi, valori e riferimenti dei pin prima della modifica.
- `BP_CombatController`: corretti due pin `B` da `280.0` a `280`, in `ClickCommand` e `UpdatePreview` (calcolo del limite dell'area HUD).
- `BP_CombatHUD`: corretti 15 pin `B` nel disegno dell'interfaccia in `EventGraph`, valori 277, 233, 173, 157, 113, 89 e 60. Modificati esclusivamente i default tramite `SetPinValue` di Unreal MCP; stessi numeri, nodi e collegamenti preservati.
- Entrambi gli asset compilati e salvati tramite editor, ricaricati da disco con `ReloadAsset` e compilati nuovamente con avvisi trattati come errori. Aggiunta in costituzione la regola di verificare i default tipizzati e la loro persistenza dopo ricaricamento. Nessuna modifica intenzionale alla mappa, ai personaggi o alle regole dello scontro; nessun commit/push.

### Verifiche

- Prima della correzione riprodotti esattamente 2 errori nel controller e 15 nell'HUD. Dopo la correzione e dopo ricaricamento entrambi compilano senza errori/avvisi.
- Avvio PIE su `Lvl_CombatPrototype` riuscito senza il dialogo di compilazione. Ispezionata l'immagine: Ezio V2, Nephilim, due guardie, barre HP, iniziativa e pannello azioni presenti.
- Comando di movimento tramite input clic del template: Ezio si sposta, consuma il budget (residuo circa 7,20 m) e si arresta a velocità zero.
- Clic reale tramite Slate sul pulsante Canvas Fine turno: il turno passa da Ezio a Guardia 2 e, dopo il turno automatico, a Nephilim; quattro unità ancora presenti. Controllo pertinente alle coordinate intere corrette dell'HUD.
- Nessun nuovo `Blueprint Runtime Error` o `Accessed None` rilevato nel log durante queste prove; gli errori di compilazione iniziali restano nello storico del log. Salvati nuovamente i due asset al termine.
- Non ripetuti scontro completo, intera suite D&D o build packaged. Non riavviato l'intero editor: verificato invece il ricaricamento da disco dei due asset interessati. PIE fermato al termine.

## 029 — 2026-11-22 — Reazione ai colpi per Ezio e Nephilim

Autore: Codex
Tipo: Funzionalità / Animazioni / Pulizia / Verifica
Stato: Attuale

### Richiesta e scelte

Integrare le due `Face_Punch_Reaction_1` fornite in cartelle `Meshy_` sotto `Content/Characters`, usarle quando Ezio e Nephilim vengono colpiti ed eliminare quelle cartelle. Contenevano file sorgente FBX/PNG, non asset Unreal già importati.

- Conservati prima dell'import tutti i file originali in `SourceArt/Characters/Ezio/v2/HitReaction/` (FBX e PNG) e `SourceArt/Characters/Nephilim/HitReaction/` (FBX e quattro PNG), con confronto SHA256 prima della pulizia.
- Importate solo le animazioni sui rispettivi Skeleton esistenti; nessuna nuova mesh, Skeleton, texture o materiale di gioco. Le texture fornite sono conservate fra le sorgenti, senza sostituire quelle già corrette sui personaggi.
- Le clip complete durano 4,5 s e contengono un'attesa iniziale: campionata la posa della testa ogni 0,25 s, immobile fino a circa 1,5 s, con la reazione principale da circa 2 s. Reimportato l'intervallo 60–135 a 30 fps (2,5 s) e impostato il playback a 2,5x: reazione di 1 s, senza attesa iniziale, compatibile con il feedback esistente di 1,35 s e l'impatto dopo 0,22 s. Sorgenti complete conservate per eventuale nuova temporizzazione.

### Modifiche

- Asset finali `/Game/Characters/Ezio/Animations/A_Ezio_HitReaction` e `/Game/Characters/Nephilim/Animations/A_Nephilim_HitReaction`; animazioni non additive, root motion disattivato e `force_root_lock` attivo.
- `BP_CombatUnit_Ezio` e `BP_CombatUnit_Nephilim`: `HitAnim` assegnato alla rispettiva clip; nuova variabile ereditata `HitPlayRate=2,5`.
- `BP_CombatUnit`: `HitPlayRate` configurabile, default base 1 per conservare le reazioni esistenti delle guardie. `PlayHitFeedback` imposta esplicitamente il playback e calcola il ritorno all'Animation Blueprint con durata/play rate + 0,02 s. Evita di ereditare velocità di playback da altre animazioni.
- La chiamata resta nel ramo colpito di `ResolveAttack`: un mancato mostra il testo senza avviare la reazione. Per un colpo letale, Death ha precedenza se presente; altrimenti viene riprodotta Hit e mantenuta l'ultima posa, senza ripristinare la locomozione dell'unità sconfitta. Per Ezio e Nephilim manca ancora una clip di caduta.
- Rimosse le due cartelle sorgente `Content/Characters/Meshy_AI_*_Face_Punch_Reaction_1` dopo verifica dell'assenza di asset Unreal al loro interno, dei referencer finali e dell'integrità delle copie SourceArt. Pulizia filesystem limitata ai due percorsi assoluti verificati, contenenti esclusivamente FBX/PNG. Nessun redirector generato, perché non sono stati spostati asset Unreal.
- I referencer delle nuove clip sono esclusivamente i rispettivi Blueprint unità; i percorsi di reimport puntano a SourceArt, non alle cartelle eliminate. Aggiornata la guida corrente. Nessun commit/push.

### Verifiche

- Compilati base unità, due figli, controller e HUD con avvisi come errori. Salvati i tre Blueprint modificati e le due clip. Ricaricati da disco base e figli, ricompilati e riprovati in PIE dopo la pulizia; riletto il grafo della reazione.
- PIE, prove controllate tramite l'evento gameplay `ResolveAttack`: colpo a Ezio, HP 38 → 35, riproduzione di `A_Ezio_HitReaction`; colpo a Nephilim, HP 49 → 46, riproduzione di `A_Nephilim_HitReaction`. Posizioni degli attori invariate durante la reazione, poi ritorno ad `ABP_Ezio` / `ABP_Nephilim` con Speed 0. Immagini di entrambe le reazioni ispezionate.
- Mancato contro Nephilim: HP invariati e Animation Blueprint ancora attivo, senza reazione.
- Dopo pulizia, attacchi automatici delle guardie attraverso il gameplay normale, con `ForcedD20=15` solo sull'istanza PIE per garantire colpi: osservate entrambe le reazioni corrette, HP 27 / 42, turni proseguiti fino al round 2; entrambe tornate ai rispettivi Animation Blueprint. Il valore di prova è stato azzerato e non salvato.
- Nuova prova controllata dei colpi letali: entrambe le clip riprodotte con `IsDefeated=true`; dopo il termine restano SingleNode sull'ultima posa, HP `[0,0,58,58]`, `BattleOver=true`. Nessun ritorno alla locomozione dopo la sconfitta.
- Controllo finale: sotto `Content/Characters` restano soltanto Ezio, Nephilim e Mannequins; sorgenti originali in SourceArt, referencer finali corretti. Nessun nuovo errore runtime rilevato nelle prove; rimangono nel log gli errori storici risolti nella voce 028.
- Non eseguiti build packaged o intera suite D&D; le guardie mantengono le animazioni preesistenti del template. PIE fermato al termine.

## 030 — 2026-11-22 — Versionamento del prototipo e delle sorgenti dei personaggi

Autore: Codex
Tipo: Versionamento / Documentazione
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto il push. Preparato un commit delle modifiche del progetto ancora locali dopo il commit iniziale: regole D&D, schede, personaggi Ezio V2 e Nephilim, camera, correzioni di compilazione e reazioni ai colpi. Le attribuzioni dei singoli interventi rimangono nelle voci precedenti; Codex esegue questa operazione di versionamento.

### Modifiche

- Aggiunta in `.gitattributes` la gestione LFS di `SourceArt/**`, oltre a `.uasset` e `.umap` già configurati. Inclusi FBX e texture sorgente, anche storici, per conservare il materiale originale senza appesantire gli oggetti Git ordinari.
- Inclusi gli asset finali dei personaggi, i Blueprint del prototipo, le schede e la documentazione aggiornata. Remoto previsto `origin`, repository `PregioSky/RalyaLandsGame`, branch `main`.
- Esclusi dal commit `.mcp.json` (connessione locale) e la modifica automatica di `Config/DefaultEngine.ini` che contiene un token locale dell'Android File Server. Questi file restano sul computer; nessuna modifica al loro contenuto in questo intervento.

### Verifiche

- Verificati stato della repository, identità Git locale e remoto. Fetch riuscito: prima del nuovo commit, HEAD e `origin/main` sono allineati al commit iniziale.
- Esaminati lo staging e lo stato LFS: asset Unreal e sorgenti sono rappresentati da puntatori LFS, mentre documentazione e attributi restano testo.
- Nessuna nuova modifica al gameplay: valgono le verifiche PIE riportate nelle voci precedenti, in particolare 028 e 029. Nessuna build packaged eseguita per questa operazione.

## 031 — 2026-11-22 — Animazioni di morte per Ezio e Nephilim

Autore: Codex
Tipo: Funzionalità / Animazioni / Pulizia / Verifica
Stato: Attuale

### Richiesta e scelte

Integrare le due animazioni `Dead` fornite sotto `Content/Characters/Meshy_AI_*_without_skin_Dead`, applicarle alla morte dei personaggi ed eliminare le due cartelle sorgente da Content. Questo sostituisce per Ezio e Nephilim il ripiego sulla reazione Hit dei colpi letali introdotto nella voce 029.

- Copiati e conservati gli originali in `SourceArt/Characters/Ezio/v2/Death/` e `SourceArt/Characters/Nephilim/Death/`: ciascuna cartella contiene un FBX e quattro PNG. Confrontati gli hash SHA256 prima di cancellare le copie sotto Content.
- Importate esclusivamente le Animation Sequence sui rispettivi Skeleton esistenti: nessuna nuova mesh, Skeleton o texture di gioco. Materiali dei personaggi preservati; i PNG forniti restano fra le sorgenti.
- Clip complete di 2,9667 s, non additive, con posa finale effettivamente a terra (campionate testa e bacino lungo l'animazione). Riproduzione naturale a 1x, senza ritaglio o accelerazione. Root motion disattivato e radice bloccata per non spostare l'attore dalla posizione di gameplay.

### Modifiche

- `/Game/Characters/Ezio/Animations/A_Ezio_Death` e `/Game/Characters/Nephilim/Animations/A_Nephilim_Death`, assegnate a `DeathAnim` nei corrispondenti `BP_CombatUnit_Ezio` e `BP_CombatUnit_Nephilim`.
- Nel ramo Death di `BP_CombatUnit.PlayHitFeedback`, impostato esplicitamente `SetPlayRate(1.0)` dopo `PlayAnimation(DeathAnim,false)`: la morte non eredita la velocità 2,5x di un'eventuale reazione precedente. Il ramo Hit non letale e il ripiego in assenza di Death restano disponibili.
- Preservati `IsDefeated`, arresto/disabilitazione del movimento, esclusione da collisioni e navigazione, salto delle unità sconfitte nei turni e conclusione dello scontro. I callback delle reazioni precedenti non ripristinano l'Animation Blueprint di un'unità sconfitta. La clip termina senza loop e mantiene la posa finale, anche a scontro concluso.
- Eliminate le due cartelle `Content/Characters/Meshy_AI_*_without_skin_Dead` dopo controllo che contenessero solo FBX/PNG e nessun asset Unreal importato. Percorsi assoluti verificati prima della rimozione; tutti i dieci file originali preservati in SourceArt. Non generati redirector, perché non sono stati spostati asset Unreal.
- Verificati i referencer delle due clip finali (solo il rispettivo Blueprint unità) e i percorsi di reimport in SourceArt. Guida corrente aggiornata. Nessun commit o push in questo intervento.

### Verifiche

- Compilati i tre Blueprint modificati con avvisi come errori e salvati insieme alle due nuove clip. Dopo la pulizia, ricaricati da disco tutti e cinque gli asset, ricompilati i Blueprint e ripetuta la prova PIE. Riletto il ramo Death per confermare il playback esplicito a 1x.
- PIE su `Lvl_CombatPrototype`, prova controllata tramite `ResolveAttack`: prima colpo non letale a Ezio, poi colpo letale mentre Hit è ancora attiva. Riprodotta `A_Ezio_Death`; circa 0,26 s avanzati dopo circa 0,28 s dall'impatto, coerenti con 1x. Nessun ritorno anticipato all'Idle dovuto al precedente callback Hit.
- Colpo letale a Nephilim: riprodotta `A_Nephilim_Death`, completata anche dopo `BattleOver=true`. Ispezionate le immagini di entrambi i corpi a terra. Entrambe le clip restano ferme a 2,9667 s dopo il termine, senza loop o ritorno all'Animation Blueprint.
- Verificati posizione degli attori conservata, velocità zero, movimento disabilitato, collisioni e influenza sulla navigazione disattivate. HP `[0,0,58,58]` e stato di sconfitta del party corretti.
- Dopo pulizia e ricaricamento, prova con attacchi automatici delle guardie: HP dei due alleati preparati a 1 tramite l'evento di danno e `ForcedD20=15` solo nell'istanza PIE per garantire i colpi. Osservate le due animazioni di morte dal normale flusso degli attacchi, turni proseguiti e conclusione dello scontro; entrambi i corpi senza collisione e fermi sulla posa finale. Valore di prova azzerato, nessuna modifica alle schede salvata.
- Sotto `Content/Characters` restano soltanto Ezio, Nephilim e Mannequins. Nessun nuovo errore runtime rilevato nelle prove; nel log restano gli errori storici della voce 028.
- Non eseguita build packaged né ripetuta l'intera suite D&D. PIE fermato al termine.

## 032 — 2026-11-22 — Suite di regressione completa e correzione del secondo attacco delle guardie

Autore: Claude Code
Tipo: Verifica / Correzione
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto di eseguire il primo punto della lista di proseguimento: una prova completa dello scontro dopo le animazioni di morte della voce 031, perché l'intera suite D&D non era più stata ripetuta dalla voce 024 mentre modelli, camera e feedback erano cambiati. Prova in PIE su `Lvl_CombatPrototype` con il flusso normale dei turni (`EndTurn`), clic reali sul viewport tramite Slate e `ForcedD20` impostato solo sull'istanza PIE.

La suite ha individuato un difetto, che è stato corretto. In `UpdateMotionProgress` il contatore `NoProgressSeconds` cresceva anche con l'unità ferma, e dopo 0,8 s scattava il ramo "Passaggio occupato" senza alcun ordine di movimento attivo. Conseguenze:

- **Guardie:** il ramo chiamava `EndTurn` durante l'animazione del primo attacco. Al termine del feedback il turno passava prima che `UpdateAI` eseguisse il secondo attacco, quindi le guardie (Veterano, 2 attacchi) ne facevano sempre e solo uno. Il difetto è stato osservato in tre turni di guardia distinti.
- **Giocatore:** compariva il messaggio "Passaggio occupato: scegli un'altra destinazione." all'avvio e a unità ferma. È il messaggio spurio già notato nella voce 024 e non indagato allora.

Correzione minima: il controllo di stallo e l'arresto vicino alla destinazione valgono solo con `MotionRequested` attivo. `BeginMotion` azzera già il contatore a ogni nuovo ordine, quindi non serve altro. La rilevazione di un percorso realmente bloccato durante il movimento resta invariata.

### Modifiche

- `BP_CombatController.UpdateMotionProgress`: aggiunto `MotionRequested` come condizione sia al ramo di arresto entro 55 cm sia al ramo di stallo dopo 0,8 s. Nessun'altra funzione modificata.
- `CombatPrototype.md`: precisazione sul controllo di stallo e voce della suite fra le verifiche.
- Conseguenza di gioco: le guardie applicano ora davvero i 2 attacchi della scheda, quindi lo scontro è più pericoloso per il party rispetto alle prove precedenti. Nessuna scheda è stata cambiata.

### Verifiche

Prima della correzione (stessa sessione PIE):

- Nephilim: Attacco extra con 2 attacchi; il terzo viene rifiutato, e un attacco durante il feedback è ignorato.
- Uscita dalla portata: un solo attacco di opportunità a 150,8 cm, reazione consumata e ripresa del movimento verso la destinazione. Recuperare energie (+13 HP) rifiutato la seconda volta. Scatto come azione porta il budget a 18 m e blocca attacco e Disimpegno; il lungo tragitto si ferma a 0 m residui, e passando nella portata di entrambe le guardie provoca due attacchi di opportunità.
- Ezio: Azione scaltra Nascondersi (Furtività 25 contro 12) e Disimpegno come azione, poi uscita dalla mischia senza attacchi di opportunità.
  - Il Nascondersi è riuscito pur essendo adiacente alle guardie, perché si erano girate verso Nephilim (161° e 68° dall'asse, contro un limite di 60°). È coerente con la regola della casa dei coni documentata nella guida, non è un difetto.
  - Le guardie ignorano Ezio nascosto e puntano Nephilim.
- Clic reale su una guardia: avvicinamento, arresto a 149,1 cm, attacco con vantaggio (probabilità mostrata 80%), furtivo 3d6 e rivelazione. Azione scaltra Disimpegno dopo l'attacco e movimento con il budget residuo.
- Critico con dadi raddoppiati; 1 naturale mancato.
- Vittoria: `BattleOver` e messaggio corretti; corpi visibili e senza collisione; Nephilim a terra con `A_Nephilim_Death`; dopo la fine `EndTurn`, Scatto, Nascondersi, attacco e clic non hanno effetto. Immagine ispezionata.

Dopo la correzione:

- Controller compilato con avvisi come errori, salvato, ricaricato da disco con `ReloadAsset` e ricompilato: stato `UP_TO_DATE`, correzione presente nel grafo riletto. Il ricaricamento ha segnalato oggetti di binding degli input generati e non ritrovati; dopo la ricompilazione, in PIE, clic sul terreno (2,73 m percorsi, arresto a velocità zero) e tasto V (coni nascosti e di nuovo visibili) funzionano.
- Scontro intero fino alla sconfitta con `ForcedD20=15`:
  - ogni turno di guardia esegue 2 attacchi, con `AttacksLeft` che arriva a 0, in 7 turni osservati;
  - il messaggio "Passaggio occupato" non compare mai;
  - Ezio sconfitto viene saltato nell'ordine, e la guardia cambia bersaglio verso Nephilim quando Ezio cade;
  - il messaggio "SCONFITTA" è corretto e i due corpi sono senza collisione e in SingleNode sull'animazione di morte;
  - immagine ispezionata.
- Log della sessione: nessun `Blueprint Runtime Error`, `Accessed None` o errore di script durante le prove.

Limiti:

- Le guardie-manichino nella posa finale di `AN_CombatDeath` appaiono chine anziché stese; è la clip preesistente del template, non modificata qui.
- Dopo la vittoria un comando rifiutato sovrascrive il testo di stato, mentre l'HUD mantiene "SCONTRO TERMINATO". È un dettaglio minore, non corretto.
- Non ripetuti: anteprima del percorso con tratto rosso, attacco di opportunità degli alleati contro una guardia in fuga, rivelazione per una guardia che arriva in vista, rotazione della camera e build packaged.
- PIE fermato al termine.

## 033 — 2026-11-22 — Velocità da halfling per Ezio

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha approvato il punto 8 della lista dei passi successivi: Ezio, halfling, si muove di 7,5 m per turno come in D&D 5e, invece dei 9 m provvisori. Il valore resta nella scheda, così movimento, Scatto, anteprima e HUD non hanno numeri scritti nei grafi.

### Modifiche

- `Data/DT_CombatSheets`, riga `Ezio`: `SpeedMeters` da 9 a 7,5. Nessun grafo modificato: `StartTurn`, `UseDash`, anteprima e HUD leggevano già `SpeedMeters`; la ricerca nel controller e nell'HUD non ha trovato 9 m scritti a mano.
- `CONSTITUTION.md`: la regola del movimento indica 7,5 m per Ezio e 9 m per gli altri; rimossa la frase che rimandava la velocità per razza.
- `CombatPrototype.md`: tabella delle regole, Scatto (Ezio 22,5 m con doppio Scatto), configurazione di Ezio e nota sull'arena (il primo attacco attorno al muro costa 7,3–7,5 m: Ezio ci arriva appena o no).

### Verifiche

- Tabella riletta dopo il salvataggio: Ezio 7,5, Nephilim e guardie 9.
- PIE: all'inizio del turno di Ezio `Remaining` = `TurnSpeed` = 7,5; `SpeedMeters` = `[7.5, 9, 9, 9]`; l'HUD mostra `Movimento residuo: 7.5 / 7.5 m` (immagine ispezionata).
- Non ripetuta in questa voce una prova dedicata di Scatto di Ezio a 15 m: il codice di `UseDash` somma `SpeedMeters` ed è invariato.

## 034 — 2026-11-22 — Balestra leggera di Ezio e attacco a distanza

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

Punto 11 della lista, con la scelta dell'utente: Ezio usa una balestra leggera (non un arco corto). Valori D&D 5e per un Ladro 5 con Destrezza +4: +7 a colpire, 1d8+4, gittata 24/96 m (80/320 piedi), Caricamento. Nephilim e guardie restano in mischia. Scelte di interfaccia proposte da Claude Code: tasto **F** e pulsante per passare all'arma a distanza, modalità azzerata a ogni turno, tiro da fermo al clic sul nemico, linea di mira nell'anteprima.

### Modifiche

- `Data/S_CombatSheet`: nuovi campi `RangedWeaponName`, `RangedAttackBonus`, `RangedDiceCount`, `RangedDieSides`, `RangedDamageBonus`, `RangedNormalMeters`, `RangedLongMeters`. `DT_CombatSheets`: Ezio `Balestra leggera`, 7, 1, 8, 4, 24, 96; Nephilim e Guardia vuoti/0.
- `BP_CombatController`:
  - nuovi array `RangedNames`, `RangedAtk`, `RangedCount`, `RangedSides`, `RangedDmgBonus`, `RangedNormal`, `RangedLong` (riempiti da `LoadSheets`) e variabili `RangedMode`, `PreviewRanged`, `PreviewTarget`;
  - `ToggleRanged` (evento tasto F) e azzeramento di `RangedMode` in `StartTurn`;
  - `RangedThreatened` (nemico vivo entro 1,5 m che vede il tiratore con `IsSeenBy`) e `RangedCheck` (gittata lunga, copertura totale, svantaggio);
  - `TryRangedAttack`: consuma l'azione e azzera `AttacksLeft` (Caricamento), animazione d'attacco, linea del dardo di 0,45 s, `RollAttack` e `ResolveAttack`;
  - `RollAttack` ha il nuovo parametro `Ranged`: usa bonus e dadi dell'arma a distanza, gestisce lo svantaggio (anche annullato dal vantaggio del nascosto) e nega l'Attacco furtivo con svantaggio; il testo del tiro indica l'arma. Le chiamate di `TryAttack` e `OpportunityAttack` passano `false`;
  - `AttackOdds` calcola la probabilità con vantaggio o svantaggio, la copertura e i dadi dell'arma attiva;
  - `ClickCommand`: con `RangedMode` il clic su una guardia chiama `TryRangedAttack`; `UpdatePreview` chiama `RangedPreview` (arma, distanza, gittata, probabilità o motivo del rifiuto).
- `BP_CombatHUD`: pulsante `TIRO A DISTANZA [F]` / `DISTANZA ATTIVA [F]` (dorato) in alto a destra del pannello per chi ha un'arma a distanza, hit box `Ranged`, arma attiva nella riga degli attacchi, nuova funzione `DrawAimLine` (linea arancione, rossa se il tiro non è possibile).
- `CombatPrototype.md`: comandi, tabella, nuova sezione "Attacco a distanza e copertura", schede, HUD. `CONSTITUTION.md`: regola dell'attacco a distanza.

### Verifiche

- Compilazione di `BP_CombatController` e `BP_CombatHUD` con avvisi come errori; salvataggio di struttura, tabella e Blueprint; ricaricamento dal disco e nuova compilazione pulita; nessun pacchetto rimasto modificato.
- PIE con clic reali nel viewport e tasto F tramite input:
  - F attiva la modalità (`Arma a distanza: Balestra leggera...`), un secondo F torna a `Stocco`;
  - clic su una guardia a 8 m con `ForcedD20=15`: `Ezio con Balestra leggera: d20 15 + 7 = 22 contro CA 17 - COLPITO | Danno: dadi 7 + 4 = 11`, HP della guardia 58 → 47, Ezio non si muove, `AttackUsed` vero e `AttacksLeft` 0; un secondo clic è rifiutato con `Azione gia usata`;
  - anteprima con Nephilim adiacente al bersaglio: `Colpire 55% | 1d8+4 +3d6 furtivo`;
  - con una guardia adiacente a Ezio che lo guarda: `Colpire 20% (svantaggio) | copertura +2 CA | 1d8+4` (furtivo negato, la guardia sulla linea dà copertura); tiro reale `d20 con svantaggio (12, 12) 12 + 7 = 19 contro CA 19 (copertura +2) - COLPITO`;
  - `RangedThreatened` vero con la guardia adiacente rivolta verso il tiratore, falso se rivolta altrove;
  - immagini dell'anteprima ispezionate: linea di mira, pulsante dorato, arma nel pannello, testo leggibile.
- Mischia dopo la modifica di `RollAttack`: clic di Nephilim su una guardia, avvicinamento e colpo con il formato del tiro invariato; attacchi delle guardie e attacchi di opportunità eseguiti nelle prove dell'IA.
- Log della sessione PIE: nessun errore runtime, `Accessed None` o indice fuori intervallo.
- Non verificati: tiro oltre la gittata normale (l'arena è più piccola di 24 m), tiro di Ezio nascosto con vantaggio a distanza (stessa logica già provata in mischia), aspetto della linea del dardo in una build non di sviluppo.

## 035 — 2026-11-22 — Copertura secondo le regole di D&D 5e

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

Punto 13 della lista: l'utente ha chiesto la copertura "come nelle regole di 5e". Mezza copertura +2 CA, tre quarti +5 CA, copertura totale: non si può essere bersaglio diretto; anche una creatura, amica o nemica, fra attaccante e bersaglio dà mezza copertura. Per misurarla senza griglia Claude Code ha adattato la variante della DMG con le linee fra gli angoli: tre origini sull'attaccante e quattro punti sul bersaglio, contando le linee bloccate dall'origine migliore. La copertura vale per ogni tiro per colpire, come da regolamento; solo il tiro a distanza viene rifiutato con copertura totale.

### Modifiche

- `BP_CombatController`: nuova funzione `CoverBonus(Attacker, Target)` → `Bonus`, `Total`. Tracce Visibility, ignorando le unità, da tre origini (centro e ±37 cm laterali, +45 cm) a quattro punti (±28 cm laterali, −45 e +55 cm): 1–2 linee bloccate +2, 3 linee +5, 4 linee copertura totale; una creatura viva entro 45 cm dalla linea, fra i due, dà almeno +2.
- `RollAttack` somma la copertura alla CA e la mostra nel testo (`contro CA 19 (copertura +2)`); `AttackOdds` la include nella probabilità quando il tiro è a distanza o il bersaglio è già in mischia; `RangedCheck` rifiuta la copertura totale.
- `CombatPrototype.md` (sezione "Attacco a distanza e copertura", arena) e `CONSTITUTION.md` (regola della copertura).

### Verifiche

- Misure in PIE con Nephilim come attaccante (le unità non attive si possono spostare senza intaccare il budget):
  - linea libera: 0;
  - bersaglio a ridosso del blocco laterale (1,2 m): +2; sul bordo (y 120 e 150), in obliquo e 1,5 m dietro: +2;
  - dietro il muro centrale (1,8 m): totale; sul bordo del muro (y 330): +2; oltre il bordo (y 380): 0;
  - guardia esattamente sulla linea: +2; spostata di 1,2 m di lato: 0;
  - due unità adiacenti su pavimento libero: 0.
- Tiro reale con copertura da creatura: CA 17 + 2 = 19 nel testo e nella probabilità.
- Prima misura scartata: spostando Ezio, unità attiva, `UpdateBudget` lo riportava entro il budget, quindi le posizioni non erano quelle previste. Una prova di mischia con +5 è dovuta a una guardia teletrasportata dentro il muro centrale dal test, non a un errore del gioco.
- Limiti: tre quarti di copertura si ottengono solo con geometrie parziali particolari, perché le linee alte passano sopra i blocchi di 1,2 m; un attacco in mischia con copertura totale non è rifiutato (riceve +5), caso non ottenibile nell'arena attuale; le guardie non tengono conto della copertura nella scelta del bersaglio.

## 036 — 2026-11-22 — IA delle guardie: bersaglio, Scatto e Disimpegno

Autore: Claude Code
Tipo: Funzionalità / Correzione
Stato: Attuale

### Richiesta e scelte

Punto 10 della lista, approvato dall'utente: guardie più intelligenti. Il punto 9 (azione Cercare) è stato escluso dall'utente e non è implementato. Scelte proposte da Claude Code: punteggio che preferisce il bersaglio più fragile e vicino ed evita gli attacchi di opportunità non necessari; Scatto quando nessun nemico è raggiungibile; Disimpegno con ripiegamento quando la guardia è gravemente ferita in mischia. La penalità per la provocazione è stata portata da 40 a 60 durante il lavoro, perché con 40 una guardia avrebbe lasciato Nephilim per un Ezio in salute a 2 m.

### Modifiche

- `BP_CombatController`:
  - `PlanAITurn` riscritta: punteggio = HP / probabilità di colpire + 2 × metri + 60 se irraggiungibile in questo turno + 60 se il percorso provoca; ripiegamento se l'azione è libera, c'è almeno un nemico adiacente e HP ≤ 1/3; Scatto se il bersaglio scelto è oltre il movimento;
  - nuova `PlanRetreat` (16 direzioni a circa l'85% del movimento, percorso completo entro il budget, punto più lontano dagli alleati vivi);
  - `UpdateAI`: con `AIRetreat` chiude il turno all'arrivo, a movimento esaurito o dopo 8 s; con `TargetIndex` < 0 chiude il turno invece di leggere un indice non valido;
  - `StartTurn` azzera `AIRetreat`.
- Correzione durante il lavoro: la prima versione di `PlanAITurn` interrompeva il ramo del ripiegamento con `(return)`, che nella funzione non ha fermato il flusso. La guardia passava al ramo "in mischia" e chiudeva subito il turno senza muoversi. In rilettura anche il `MoveTo` finale risultava fuori dalla catena dei rami. La versione attuale usa variabili locali (`Retreat`, `RetreatGoal`) e una catena esplicita, con un `MoveTo` in ciascun ramo che deve muovere; struttura riletta dopo la scrittura.
- `CombatPrototype.md` (comportamento delle guardie) e `CONSTITUTION.md` (regola dell'IA).

### Verifiche

- Compilazione con avvisi come errori, salvataggio, ricaricamento dal disco e nuova compilazione dopo ogni modifica; struttura di `PlanAITurn` riletta.
- PIE, con `ForcedD20=1` per evitare danni casuali:
  - guardia a 13–18 m da entrambi gli alleati: `Scatto`, `TurnSpeed` 18, arriva adiacente a Ezio e non attacca (azione usata); provato due volte;
  - guardia a 8 m: raggiunge Ezio (CA più bassa) e fa entrambi gli attacchi;
  - guardia a 13/58 HP adiacente a un alleato: `Disimpegno`, `AIRetreat`, si allontana di circa 7,3 m, nessun attacco di opportunità (reazione dell'alleato intatta), turno chiuso all'arrivo;
  - guardia adiacente a Nephilim con Ezio sano a 3,3 m: resta su Nephilim (`TargetIndex` 1);
  - stessa posizione con Ezio a 13 HP: va su Ezio (`TargetIndex` 0) e lo raggiunge.
- Osservato e documentato, non corretto: la valutazione della provocazione considera il percorso entro il movimento normale. Una guardia in Scatto è passata accanto a Nephilim nel tratto aggiunto e ha subito un attacco di opportunità.
- Log PIE senza errori runtime. PIE fermato al termine.

## 037 — 2026-11-22 — Balestra visibile di Ezio e struttura Items riutilizzabile

Autore: Codex
Tipo: Funzionalità / Equipaggiamento / Asset / Pulizia
Stato: Attuale

### Richiesta e scelte

Integrare il modello fornito in `Content/Characters/Meshy_AI_Hand_Crossbow_20260401_texture_fbx`, equipaggiarlo a Ezio per gli attacchi a distanza e rimuovere la cartella Meshy. L'utente ha chiesto che gli oggetti equipaggiabili e scambiabili abbiano una gerarchia propria al livello di Characters. Lette e preservate le modifiche precedenti di Claude Code: velocità di Ezio 7,5 m, tiro a distanza con F, copertura e IA (voci 032–036).

- Creato `/Game/Items/`, separato dai personaggi, con definizione comune PrimaryDataAsset e sottocartelle per armi, mesh, materiali e texture. Oggetto visuale indipendente dal rig; socket e posa di mira restano sulla configurazione del personaggio.
- Mantenute le regole esistenti della balestra leggera (+7, 1d8+4, 24/96 m, Caricamento), anche se il nome della sorgente grafica è Hand Crossbow. Nessuna migrazione delle statistiche fuori da `DT_CombatSheets` in questo intervento.
- Introdotto uno slot a distanza con operazioni Blueprint di equipaggiamento, sostituzione e rimozione; non costruiti inventario grafico, raccolta, scambio del party o persistenza dell'inventario. La definizione permette di organizzare e riutilizzare ulteriori oggetti, senza incorporare la mesh in Ezio.

### Modifiche

- `Items/Core/BP_ItemDefinition`: PrimaryDataAsset con `ItemId`, `DisplayName`, `EquipmentSlot`, `Mesh`, `GripLocation`, `GripRotation`, `GripScale`, `MuzzleOffset`; campi modificabili sulle definizioni. Nessun riferimento allo Skeleton o alle animazioni di Ezio nella definizione.
- `Items/Weapons/Crossbows/HandCrossbow/`: `DA_HandCrossbow`, `Meshes/SM_HandCrossbow`, `Materials/M_HandCrossbow` e quattro `Textures/T_HandCrossbow_*`. Import statico unico dal FBX; materiale PBR con base color sRGB, normal Normalmap e roughness/metallic Masks non sRGB. Abilitati gli usi StaticMesh/Nanite del materiale per la mesh importata. Rimangono gli avvisi di import relativi a smoothing group e tangenti quasi nulle della sorgente; aspetto ispezionato in PIE.
- `BP_CombatUnit`: componente `RangedItemMesh` senza collisioni, scala assoluta; variabili `InitialRangedItem`, `EquippedRangedItem`, `RangedPresentationActive`, `RangedGripSocket`, `RangedAimPose`, `RangedAimPoseTime`. Funzioni `EquipRangedItem`, `UnequipRangedItem`, `SetRangedPresentation`, `GetRangedMuzzleLocation`; equip iniziale in BeginPlay, arma nascosta alla morte.
- `EquipRangedItem` accetta lo slot Ranged con mesh valida prima di sostituire il precedente oggetto, attacca il componente al socket dell'unità e applica grip e scala. Offset in cm compensati per la scala del socket: verificata scala osso 100 su Ezio. La scala 0,25 dell'oggetto produce ingombro di circa 47 cm sul piano; impugnatura calibrata sulla posa di mira, senza alterare lo Skeleton.
- `BP_CombatUnit_Ezio`: `InitialRangedItem=DA_HandCrossbow`, socket RightHand, posa di mira V2 `A_Ezio_Crouch_Walk_Left_with_Gun` al tempo 0. `PlayRangedAttackFeedback` orienta il tiratore, mantiene questa posa per 1,25 s e ripristina il suo Animation Blueprint se vivo. La mischia continua a usare il feedback preesistente. Posa provvisoria: non è stata creata una nuova animazione di sparo o ricarica.
- `BP_CombatUnit_Nephilim`: solo configurazione del socket RightHand per il suo rig; nessun oggetto iniziale assegnato. La classe base dei manichini usa hand_r. Le unità senza definizione restano senza mesh equipaggiata.
- `BP_CombatController`: ToggleRanged mostra/nasconde l'oggetto, StartTurn nasconde le armi a distanza di tutte le unità, TryRangedAttack usa il feedback di mira e l'origine del dardo dal `MuzzleOffset` dell'oggetto. `HasEquippedRangedItem` impedisce tiro e attivazione senza equipaggiamento; UpdatePreview disattiva la modalità se l'oggetto viene rimosso. Preservati risorse, tiri, copertura e comportamento dell'IA di Claude Code.
- Copiati i cinque originali FBX/PNG in `SourceArt/Items/Weapons/Crossbows/HandCrossbow/`, poi eliminata la cartella Meshy sotto Characters dopo confronto SHA256 e verifica che contenesse solo sorgenti, senza asset Unreal. L'import ha estratto anche le texture embedded nella sottocartella FBM di SourceArt. I percorsi di reimport puntano a SourceArt.
- Nessun redirector generato da spostamenti di asset. Definizione finale referenziata da `BP_CombatUnit_Ezio`; clip di mira referenziata dal personaggio, non dalla definizione. Eliminata anche la definizione temporanea usata per la prova di sostituzione (`Items/_Validation`). Guida e costituzione aggiornate. Nessun commit/push.

### Verifiche

- Compilati con avvisi come errori `BP_ItemDefinition`, unità base, figli Ezio/Nephilim, controller e HUD. Salvati gli otto asset Items e i quattro Blueprint modificati; ricaricati da disco definizione base, definizione della balestra, unità base/Ezio e controller, ricompilati e riprovati dopo pulizia.
- Ispezione visuale ravvicinata in PIE e catture diagnostiche da più angoli: texture, modello, impugnatura e posa di mira; verificato il verso della balestra anche con vista ortografica dall'alto. L'oggetto segue la mano e mantiene scala 0,25 indipendentemente dalla scala del rig.
- Input reale F e clic sul nemico: modalità attiva e mesh visibile; tiro a circa 4,24 m da fermo, consumo dell'azione e AttacksLeft 0, danno alla guardia e ritorno ad ABP_Ezio dopo il feedback. Budget ancora 7,5 m e posizione del tiratore invariata. Primo tiro osservato: guardia 58 → 53, CA 19 per copertura +2; prova finale dopo ricaricamento: 58 → 49. Secondo tentativo lascia gli HP a 49, coerente con Caricamento.
- Prova di sostituzione con una seconda definizione temporanea creata fuori da PIE: lo slot punta alla nuova definizione e la mesh resta visibile. Rimozione: EquippedRangedItem vuoto, componente nascosto, RangedMode false e F non attiva il tiro; riequipaggiamento della definizione originale e F ripristinano la mesh. CollisionEnabled NoCollision. Fine turno nasconde la balestra.
- Dopo ricaricamento, colpo letale controllato contro Ezio: l'oggetto viene nascosto e parte A_Ezio_Death. La posa di mira e il suo callback non sostituiscono l'animazione di morte.
- Durante authoring, un tentativo di modifica del controller con PIE attivo ha prodotto un errore AddMappingContext durante la ricompilazione; un tentativo di duplicare il DataAsset in PIE è stato rifiutato dall'editor. PIE fermato e operazioni ripetute nel contesto corretto; nessun asset di prova rimasto. Nelle sessioni finali dopo la ricompilazione fuori da PIE non rilevati nuovi errori runtime; gli errori di quei tentativi restano nel log.
- Non ripetuta l'intera suite D&D di Claude Code né una build packaged. Non introdotte nuove animazioni dedicate alla balestra o un sistema completo di inventario. PIE fermato al termine.

## 038 — 2026-11-22 — Spada lunga di Nephilim, stocco e affondo di Ezio

Autore: Codex
Tipo: Funzionalità / Equipaggiamento / Asset / Pulizia
Stato: Attuale

### Richiesta e scelte

Integrare le due armi Meshy fornite sotto Items, assegnando la spada lunga a Nephilim e lo stocco a Ezio; usare il nuovo affondo fornito sotto Characters per Ezio e conservare l'attacco attuale di Nephilim. Rimossi gli input Meshy da Content dopo integrazione e conservazione delle sorgenti. I tre input erano archivi ZIP, non cartelle già estratte.

- Esteso il sistema Items della voce 037 con uno slot `Melee`, usando la stessa definizione riutilizzabile delle armi a distanza. Armi separate dai personaggi e sostituibili via Blueprint, senza incorporarle nelle mesh o introdurre dipendenze degli oggetti dagli Skeleton.
- La sorgente della spada è denominata Twohandedsword; viene usata come modello della spada lunga richiesta, mantenendo le statistiche esistenti della scheda di Nephilim. Nessuno scudo grafico aggiunto. Stocco e spada lunga restano rispettivamente 1d8+4 e 1d8+6; le definizioni visive non cambiano le regole D&D.
- Importata solo l'animazione di affondo sullo Skeleton esistente di Ezio, senza mesh o texture del personaggio duplicate. La posa provvisoria della balestra resta separata dall'attacco in mischia.

### Modifiche

- `/Game/Items/Weapons/Swords/Longsword/`: `DA_Longsword`, `Meshes/SM_Longsword`, `Materials/M_Longsword` e cinque texture BaseColor, Normal, Roughness, Metallic, Emissive. `/Game/Items/Weapons/Swords/Rapier/`: `DA_Rapier`, `Meshes/SM_Rapier`, `Materials/M_Rapier` e quattro texture PBR. Materiali compilati, usi StaticMesh e Nanite abilitati e verificati anche dopo salvataggio/ricaricamento; emissive della spada collegata al materiale. Nessuna modifica alle texture dei personaggi.
- Definizioni `EquipmentSlot=Melee`, identificativi `Weapon.Sword.Longsword` e `Weapon.Sword.Rapier`; scala 0,58 per circa 110 cm e 0,36 per circa 69,5 cm. Impugnature calibrate sulla mano destra durante le rispettive clip. Compensata la scala osso 100 degli Skeleton, con scala assoluta degli oggetti. Durante la calibrazione corretta una rotazione costruita con argomenti posizionali di `Rotator`: le rotazioni finali sono state impostate esplicitamente per nome e ricontrollate in PIE.
- `BP_CombatUnit`: `MeleeItemMesh` senza collisioni, `InitialMeleeItem`, `EquippedMeleeItem`, `MeleeGripSocket`; funzioni `EquipMeleeItem`, `UnequipMeleeItem`, `UpdateEquippedItemVisibility`. Validazione di slot/mesh prima della sostituzione. Equipaggiamento in BeginPlay insieme allo slot a distanza; `SetRangedPresentation` aggiorna entrambi i componenti, mostrando una sola arma. Il feedback in mischia ripristina la presentazione da mischia; cambio turno e morte usano la logica già esistente per ripristinare la spada o nascondere entrambe le armi. Nessuna modifica ai grafi del controller in questo intervento.
- `BP_CombatUnit_Ezio`: `InitialMeleeItem=DA_Rapier`, socket RightHand; `AttackAnim=A_Ezio_ThrustSlash`, playback 2,16x. Clip in `/Game/Characters/Ezio/Animations/`, importata sullo `SK_Ezio_Skeleton` esistente; frame 9–90 a 30 fps, durata 2,7 s riprodotta in 1,25 s. Conservati impatto a 0,22 s e blocco dei comandi a 1,35 s. Root motion spento e radice bloccata. `A_Ezio_Axe_Spin_Attack` conservata tra le clip disponibili ma non più assegnata come attacco.
- `BP_CombatUnit_Nephilim`: `InitialMeleeItem=DA_Longsword`, socket RightHand. `A_Nephilim_Attack` e playback 2,24x invariati.
- Archivi originali copiati e confrontati SHA256 nelle rispettive cartelle `SourceArt/.../Archives/`; estratte sorgenti armi in `SourceArt/Items/Weapons/Swords/Longsword/` e `Rapier/`. Animazione in `SourceArt/Characters/Ezio/v2/ThrustSlash/ThrustSlash.fbx`, con PNG originali conservati: nomi estratti abbreviati per il limite di lunghezza dei percorsi Windows, ZIP originale intatto. Tutti i percorsi di reimport controllati puntano a SourceArt. Le texture FBM estratte dall'import restano con le sorgenti.
- Eliminati da Content i tre archivi: `Items/Meshy_AI_Realistic_Twohandedsword_20260726_texture_fbx.zip`, `Items/Meshy_AI_Spanish_Rapier_20251212_texture_fbx.zip` e `Characters/Meshy_AI_Wily_Halfling_Wanderer_20261115_animated_mixamo_without_skin_Thrust_Slash.zip`, dopo verifica dei backup. Nessun file/cartella Meshy rimasto sotto Content/Items o Content/Characters. Nessun asset Unreal temporaneo o redirector generato; nessuna cancellazione delle sorgenti in SourceArt.
- Guida corrente e costituzione aggiornate. Rettificata nella guida la frase ormai superata che indicava assenza del modello di balestra: il modello e la posa esistono dalla voce 037, mentre manca ancora una clip dedicata di sparo/ricarica. Nessun commit/push.

### Verifiche

- Riletti i grafi di equipaggiamento, visibilità, BeginPlay e attacco. Compilati base, Ezio e Nephilim con avvisi come errori; salvati i 15 nuovi asset Items, la clip e i tre Blueprint modificati. Ricaricati base/figli, entrambe le definizioni e la clip dal disco, poi ricompilati e ripetute le prove PIE. Controllati i riferimenti: `DA_Rapier` e affondo usati da `BP_CombatUnit_Ezio`, `DA_Longsword` da `BP_CombatUnit_Nephilim`.
- Ispezione visiva in PIE di texture, scala, impugnature, Idle e attacco per entrambi; controlli anche da vista frontale dopo la correzione dei materiali. Mesh equipaggiate corrette, guardie senza nuove armi. Nephilim mantiene l'animazione preesistente; Ezio usa quella appena importata e torna ad `ABP_Ezio` dopo il feedback.
- Fixture solo in PIE: guardie riposizionate in portata, `ForcedD20=15` per colpi certi e `1` durante le attese dei turni, ripristinato a 0. Ezio colpisce con TryAttack: guardia 58→50, attacchi 0, movimento ancora 7,5 m; al turno seguente clic reale tramite Enhanced Input: 50→45 e ritorno ad ABP_Ezio. Nephilim colpisce con la clip invariata: 58→45→35, attacchi 2→1→0; terzo tentativo lascia gli HP a 35. Ritorno ad ABP_Nephilim. Passaggio dei turni con guardie automatiche osservato.
- Input reale F: balestra visibile/stocco nascosto, poi stocco visibile/balestra nascosta. Cambio turno ripristina mischia. Sostituzione runtime stocco→spada lunga, rimozione e riequipaggiamento dello stocco riusciti; tentativo di mettere la balestra nello slot Melee respinto senza perdere lo stocco. Le prove non modificano le definizioni persistenti.
- Movimento di Nephilim tramite MoveTo: velocità non nulla durante il percorso, arma visibile sul socket RightHand, NoCollision; arresto con velocità zero e Animation Blueprint corretto, residuo circa 7,14 m su 9. Gli offset restano agganciati alla mano.
- Colpi letali controllati tramite ResolveAttack: Ezio in modalità balestra e Nephilim con spada diventano sconfitti, entrambi i componenti arma nascosti; proseguono le animazioni di morte e compare la sconfitta dello scontro. Nessun callback riporta la visibilità dell'arma.
- Import dell'affondo segnala bind pose non valide nella sorgente, con fallback alla posa del tempo zero; Skeleton scelto e clip controllati in PIE. Durante le prove osservato anche l'avviso preesistente del template `AN_CombatHit` additiva riprodotta direttamente sulle guardie, non supportata così nelle build cooked: asset non modificato da questo intervento. Risolti gli avvisi Nanite delle nuove armi, confermando i flag dopo ricaricamento. Alcune interrogazioni Python con nomi di proprietà/API non disponibili hanno restituito errori di authoring; corrette le interrogazioni, senza errori Blueprint runtime rilevati nelle prove descritte.
- Non ripetuta l'intera suite D&D né verificata una build packaged. Ultima ispezione dopo pulizia e ricaricamento materiali eseguita in PIE; PIE fermato al termine.

## 039 — 2026-11-22 — Reazioni solo in mischia e guardie più prudenti nei percorsi

Autore: Codex
Tipo: Correzione / Gameplay / IA
Stato: Attuale

### Richiesta e scelte

Impedire gli attacchi di opportunità con l'arma a distanza impugnata e correggere le guardie che sottovalutano il rischio di subirli. Ispezionati i grafi attuali tramite Unreal MCP: la previsione si fermava al movimento normale, la penalità piatta poteva essere superata dal punteggio di un bersaglio fragile e veniva valutato solo il percorso più corto per quel bersaglio. Inoltre StartTurn ripristinava la mischia per tutte le unità, rendendo impossibile conservare la balestra durante i turni nemici.

- Il blocco dipende dall'arma attualmente impugnata dalla singola unità, non dal semplice possesso di una balestra nello slot. Nessun cambio automatico dallo stocco alla balestra o viceversa per reagire. La risorsa reazione resta disponibile se il tiro d'opportunità è impedito dall'arma a distanza.
- L'arma scelta resta in mano durante i turni altrui; all'inizio del proprio nuovo turno si torna alla mischia, conservando questa parte del comportamento precedente. La modifica sostituisce il reset di tutte le armi a ogni passaggio di turno descritto nelle voci 037–038.
- Le guardie danno priorità agli approcci sicuri, anche contro un bersaglio più sano o con un punto d'arrivo leggermente più lontano. Se tutti provocano, usano Disimpegno rinunciando agli attacchi; con azione già spesa terminano il turno. Questa scelta sostituisce la penalità di 60 e l'eccezione del bersaglio gravemente ferito della voce 036. Conservato il ripiegamento delle guardie a pochi HP.

### Modifiche

- `BP_CombatController`: nuova funzione pura `CanOpportunityReact(Index)`, comune a `CheckOpportunity`, alla previsione e al controllo iniziale di `OpportunityAttack`. Richiede indice e unità validi, HP positivi, unità non sconfitta, reazione disponibile e `RangedPresentationActive=false`. I manichini mantengono il proprio attacco in mischia senza richiedere una nuova mesh di arma. Reazione, animazione, arresto e danni vengono avviati soltanto dopo il controllo.
- `StartTurn` aggiorna la presentazione da mischia solo per l'unità attiva, lasciando intatta quella delle altre unità. Il `RangedMode` del controller riparte spento per il nuovo turno, mentre la scelta fuori turno è conservata sull'unità.
- Nuova `ProvokesOpportunityWithinBudget(BudgetMeters, PathPoints, ExtraClearance)`: controllo geometrico dei segmenti in piano, con ultimo segmento tagliato al budget. Rileva attraversamento completo di una zona minacciata anche all'interno dello stesso segmento; elimina la dipendenza dai campioni ogni 25 cm e dalla precedente tolleranza di ingresso di 15 cm. Esclude unità sconfitte, reazioni spese e armi a distanza, oltre a Disimpegno e movimento da nascosti già esclusi prima. Portata effettiva invariata a 150,5 cm.
- `ProvokesOpportunity` diventa l'adattatore per l'anteprima: budget residuo reale e margine zero. `AIMovementBudget` considera anche lo Scatto quando il candidato richiederebbe movimento aggiuntivo e l'azione è libera.
- Nuova `FindAIAttackPlan`: stessi 24 candidati in portata e controlli NavMesh del giocatore, con priorità sicurezza e poi costo. Le chiamate dell'IA e il confronto tra bersagli in `PlanAITurn` usano il budget previsto e un margine di 20 cm sul passaggio accanto alla zona minacciata, per evitare percorsi sicuri solo per pochi millimetri. `FindAttackPlan` e le chiamate di clic/anteprima del giocatore sono rimaste identiche alla versione precedente.
- `PlanAITurn`: conserva il punteggio HP/probabilità di colpire + 2 per metro + 60 se oltre il movimento normale, ma lo applica solo a parità di rischio. Mantiene Scatto sui percorsi sicuri; nei percorsi inevitabilmente minacciati consuma l'azione per Disimpegno, azzera gli attacchi e muove entro il budget normale. Nessuna combinazione Disimpegno + Scatto con una sola azione. Se l'azione è già spesa, non inizia il percorso rischioso e chiude il turno.
- `BP_CombatHUD`: nell'ordine dei turni distingue `Reazione`, `Reazione usata` e `Reazione bloccata`, usando lo stesso controllo del gameplay. Guida corrente e costituzione aggiornate.
- Nessuna modifica a statistiche, mesh, animazioni, NavMesh, mappa o sorgenti. Una funzione temporanea `Validation039` ha preparato gli stati dei test in PIE senza esporre come modificabili le variabili di gameplay; eliminata prima del salvataggio finale. Nessun asset o funzione di test residua. Nessun commit/push.

### Verifiche

- Riletti tutti i rami modificati. Confronto DSL prima/dopo: modificati soltanto CheckOpportunity, OpportunityAttack, PlanAITurn, ProvokesOpportunity e StartTurn, aggiunte le quattro funzioni previste; nessuna voce precedente rimossa. `FindAttackPlan` del giocatore verificata identica. HUD modificato nel solo testo di stato della reazione.
- Controller e HUD compilati con avvisi come errori, salvati e ricaricati dal disco, poi ricompilati. La ricarica HUD inizialmente bloccata da un salvataggio in corso è stata ripetuta con successo. Verificata l'assenza di Validation039 dopo la rimozione. Avviato ancora PIE con gli asset finali ricaricati: nessuna ricompilazione richiesta e HUD corretto.
- Fixture in PIE, senza modificare le schede persistenti: Ezio con balestra, guardia inizialmente a 1,4 m che si allontana. Nessun attacco, HP invariati, reazione intatta e balestra ancora impugnata durante il turno della guardia. Anche la chiamata diretta a OpportunityAttack viene respinta senza feedback o consumo. Con lo stocco, lo stesso movimento provoca: reazione consumata, guardia 58→46 con d20 forzato a 15, arresto intorno a 150,5 cm e ripresa del percorso dopo il feedback. Reazione della guardia contro Ezio ancora funzionante, con consumo anche sul mancato forzato a 1.
- Geometria della previsione: attraversamento che provoca solo dopo i 9 m restituisce falso con budget 9 e vero con 18; arrivo dentro la portata senza uscita non provoca; uscita nel segmento finale tagliato al budget rilevata; budget zero e passaggio a 160 cm con margine zero non provocano. Reattore con arma a distanza, HP zero o reazione spesa ignorato. I test della geometria iniziali precedono l'aggiunta del margine IA; il margine zero conserva lo stesso calcolo e i casi di Scatto sono stati riprovati con la versione finale.
- IA, guardia adiacente a Nephilim con Ezio a 13 HP più lontano: sceglie Nephilim senza abbandonarne la portata. Con Nephilim nascosto come solo reattore lungo il percorso e Ezio come unico bersaglio, sceglie Disimpegno; movimento fino in portata, zero attacchi, HP invariati e reazione di Nephilim intatta. Con azione già spesa nello stesso assetto chiude il turno, posizione invariata e velocità zero. Quando il solo reattore ha la balestra, avanza senza consumare Disimpegno.
- Caso riprodotto di Scatto: percorso più corto 15,52 m, sicuro nei primi 9 ma provocante entro 18. Il nuovo piano sceglie un altro punto in portata: circa 15,99 m, sicuro anche con margine IA di 20 cm. Eseguito realmente in PIE con budget 18 m: la guardia arriva al nuovo punto e passa il turno, HP invariati e reazione di Nephilim ancora disponibile. Con reazione già spesa l'IA continua a poter usare Scatto senza considerare quella minaccia.
- Input reale F e Spazio: balestra conservata entrando nel turno di Nephilim, `CanOpportunityReact(Ezio)=false` e reazione non consumata; ispezione visiva dell'HUD con `Reazione bloccata`. Dopo rimozione della funzione di test e ricarica del controller, ripetuto su iniziativa casuale: turno di Ezio→Guardia 1 con balestra ancora attiva e reazione bloccata, poi round 2, nuovo turno di Ezio in mischia con reazione utilizzabile. Default ForcedD20 verificato a 0; le forzature runtime dei test ripristinate a 0 prima della chiusura.
- Durante authoring alcuni tentativi di modifica delle firme sono stati respinti transazionalmente per parametri array read-only e pin ricostruiti; l'approccio finale usa una funzione IA separata e copie locali dei punti del percorso. Tentativi di impostare direttamente variabili non modificabili sull'istanza sono stati rifiutati: usata la funzione temporanea Blueprint, poi rimossa. Gli errori di authoring restano nel log; nessun errore Blueprint runtime rilevato nelle sessioni finali. Gli avvisi di ricaricamento riguardano vecchi oggetti transitori di binding degli input; F e Spazio riprovati dopo la ricarica.
- Non ripetuta l'intera suite D&D né una partita fino a vittoria/sconfitta, e non eseguita una build packaged. Il controllo dei percorsi copre i candidati NavMesh valutati, senza introdurre un pianificatore generale di deviazioni attorno alle aree minacciate. PIE fermato al termine.

## 040 — 2026-11-22 — Arma mantenuta all'inizio del turno e arena più varia

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto che l'arma scelta non torni alla mischia all'inizio del proprio turno: chi aveva la balestra ricomincia con la balestra. Sostituisce il ripristino della mischia introdotto da Codex nella voce 039; la reazione resta bloccata finché si impugna la balestra, come in 039. L'utente ha anche chiesto un'arena più varia, da preparare prima della suite di regressione per provare le coperture.

### Modifiche

- `BP_CombatController.StartTurn`: rimossi l'azzeramento di `RangedMode` e il ripristino della presentazione da mischia. Per gli alleati `RangedMode` = oggetto a distanza equipaggiato, `RangedLong` > 0 e `RangedPresentationActive` dell'unità; poi `SetRangedPresentation(unit, RangedMode)`. Le guardie ricevono sempre `false` (vedi 043).
- Livello `Lvl_CombatPrototype`, cartella `CombatPrototype/Arena`: sei nuovi StaticMeshActor (cubo dell'Engine, materiali di prototipo esistenti), salvati come pacchetti esterni di World Partition. Rovine a nord: `Arena_NorthRuinWall` 4 × 0,4 m, alto 1,1 m, a (0; 5,6 m); colonne `Arena_NorthPillarW/E` 0,6 × 0,6 × 2,3 m a x ±2,3 m. A sud: casse `Arena_SouthCrateW/E` 1 × 1 × 0,9 m a (±2,6; −5,6 m) e colonna `Arena_SouthPillar` a (0; −6 m). Posizioni iniziali e muri esistenti invariati.
- `CombatPrototype.md` e `CONSTITUTION.md` aggiornati (persistenza dell'arma, arena).

### Verifiche

- PIE: con F attivo a fine turno, al turno successivo di Ezio `RangedMode` e presentazione restano veri; con F spento riparte in mischia. Con la balestra, `CanOpportunityReact(Ezio)=false`.
- Griglie di `CoverBonus` attorno alle nuove geometrie: a nord frequente +5 vicino alle colonne e totale dietro le colonne, +2 dietro il muretto; a sud +2 e totale dietro la colonna. Percorsi NavMesh verso il retro delle rovine (circa 8,3 m) e verso sud validi; un primo tentativo restituiva nessun percorso perché il punto di partenza era dentro l'ostacolo di navigazione di Nephilim, ripetuto da altri punti.
- Non verificato con il giocatore il bilanciamento dell'arena: le posizioni sono una proposta modificabile.

## 041 — 2026-11-22 — Suite di regressione con coperture reali e correzione dell'anteprima del tiro

Autore: Claude Code
Tipo: Verifica / Correzione
Stato: Attuale

### Richiesta e scelte

Suite di regressione completa richiesta dall'utente, eseguita dopo l'arena (040) per provare anche le coperture. La suite ha trovato un difetto, corretto qui.

### Modifiche

- `BP_CombatController.UpdatePreview`: con la balestra attiva il blocco finale dell'anteprima di mischia sovrascriveva il testo `TIRO: ...` di `RangedPreview`. Il blocco è ora racchiuso in un controllo esplicito `if not PreviewRanged`. Una prima forma con `else` veniva normalizzata dall'editor DSL in un `elif`, spostando fuori le istruzioni finali; sostituita dal controllo esplicito.
- `CombatPrototype.md`: rettificata la descrizione dell'avviso `ATTACCO DI OPPORTUNITA!`, che riportava ancora il campionamento ogni 25 cm con 15 cm di margine sostituito nella voce 039.

### Verifiche

PIE, iniziativa reale, input reali (clic, hover, tasti) e `ForcedD20` dove serviva un esito deterministico:

- Ordine di iniziativa e schede caricate (HP, CA, velocità, attacchi, armi a distanza di Ezio e guardie).
- Movimento: anteprima `Percorso ~4.0 m`, spostamento effettivo 3,48 m, residuo 4,02 m, nessun movimento residuo.
- Balestra: anteprima `TIRO: Balestra leggera | distanza 7.7 m ... | Colpire 30% | copertura +5 CA` su una guardia all'angolo delle rovine; clic su una guardia dietro la colonna (copertura totale) rifiutato senza consumare l'azione; tiro reale `d20 15 + 7 = 22 contro CA 22 (copertura +5) - COLPITO`, Caricamento rispettato. Nascondersi rifiutato perché visto.
- Mischia di Nephilim con clic reale: avvicinamento a 149,1 cm, due attacchi con Attacco extra, terzo rifiutato; Recuperare energie; anteprima di ritirata con `ATTACCO DI OPPORTUNITA!` e attacco di opportunità effettivo della guardia, reazione consumata.
- Turni IA delle guardie: due attacchi, Scatto su percorso sicuro.
- Arma di Ezio mantenuta al turno successivo (040); critico con furtivo tramite alleato `dadi 12 + 4 + furtivo 6d6: 16 = 32`; Disimpegno come azione bonus; Scatto di Nephilim a 18 m.
- Vittoria: corpi visibili, senza collisione, armi nascoste; dopo la vittoria clic, Fine turno, F e T non producono effetti. Sconfitta in una nuova sessione PIE: `SCONFITTA - Ezio e Nephilim sono caduti.`, corpi visibili senza collisione.
- Log runtime: nessun `Blueprint Runtime Error`, `Accessed None` o accesso fuori dai limiti.
- Artefatto di prova non risolto: un clic sullo schermo verso la guardia dietro la colonna, con camera inclinata, colpisce la colonna invece della guardia; il caso è stato verificato chiamando il controllo della copertura.

## 042 — 2026-11-22 — Schivata prodigiosa di Ezio e Azione impetuosa di Nephilim

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha approvato Schivata prodigiosa (Ladro 5) e Azione impetuosa (Guerriero 5), escludendo Elusione perché di livello 7. Aiutare e Schivare restano rimandate. Schivata prodigiosa è automatica, senza conferma come in Baldur's Gate 3, e consuma la reazione. Azione impetuosa richiede l'azione già spesa, per non accumulare due azioni non usate.

### Modifiche

- `S_CombatSheet`: campi `UncannyDodge` (bool) e `ActionSurgeUses` (int). `DT_CombatSheets`: Ezio `UncannyDodge=true`, Nephilim `ActionSurgeUses=1`.
- `BP_CombatController`: array `UncannyDodge` e `ActionSurgeLeft` riempiti in `LoadSheets`; variabili `AppliedDamage` e `DodgedHit`.
- `ResolveAttack`: se il colpo va a segno e il bersaglio ha la Schivata, è vivo, ha la reazione e vede l'attaccante (`IsSeenBy`), il danno è dimezzato per difetto e la reazione consumata. HP, feedback e numero flottante (`SCHIVATA -N`) usano `AppliedDamage`; il riepilogo del tiro e lo stato lo segnalano.
- Funzioni `CanActionSurge` e `UseActionSurge`; tasto **T**. L'uso rende di nuovo disponibile l'azione, azzera `AttackUsed` e aggiunge gli attacchi di un'azione; testo `AZIONE IMPETUOSA`.
- `BP_CombatHUD`: pannello inferiore allargato da 860 a 1080 px; pulsante `AZIONE IMPETUOSA [T]` in una terza colonna (blu se utilizzabile, grigio altrimenti, assente senza usi) e hitbox `ActionSurge`. La soglia dei clic ignorati sopra il pannello passa da 880 a 1100 px in `ClickCommand` e `UpdatePreview`.

### Verifiche

- PIE: Azione impetuosa rifiutata prima dell'azione; dopo due attacchi concede altri 2 attacchi (4 colpi totali, il quinto rifiutato); seconda richiesta `non disponibile`.
- Schivata prodigiosa: con attaccante non visto danno pieno (6) e reazione intatta; con attaccante visto 7 → 3 e reazione consumata; colpo successivo nello stesso round a danno pieno.
- Immagine dell'HUD ispezionata: pulsante nella terza colonna, leggibile, nessuna sovrapposizione.
- Blueprint compilati con avvisi come errori, salvati, ricaricati dal disco e ricompilati (`BS_UP_TO_DATE`); righe della tabella rilette dopo il ricaricamento.

## 043 — 2026-11-22 — Balestra pesante delle guardie e tiro nell'IA

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha approvato una balestra pesante per le guardie, con un'IA che la consideri insieme alla copertura. Valori del Veterano del Manuale dei Mostri: +3, 1d10+1, 30/120 m. La mischia raggiungibile resta preferita (due attacchi a +5); il tiro sostituisce Scatto o Disimpegno quando la guardia non potrebbe attaccare in mischia in questo turno. Non esiste un modello di balestra pesante: si riusa quello della balestra di Ezio come segnaposto.

### Modifiche

- `DT_CombatSheets`, riga Guardia: `Balestra pesante`, +3, 1d10+1, gittata 30/120 m.
- Nuovi asset: `/Game/Items/Weapons/Crossbows/HeavyCrossbow/DA_HeavyCrossbow` (duplicato di `DA_HandCrossbow`, `Weapon.Crossbow.Heavy`, scala 0,4, impugnatura calibrata sul socket `hand_r` del manichino nella posa di mira: rotazione (9; −11,3; −17,9), offset nullo, `MuzzleOffset` (−94; 0; 5)); `/Game/CombatPrototype/Animations/AN_CombatAimCrossbow` (copia locale di `MF_Rifle_Idle_ADS`, che resta invariato); `/Game/CombatPrototype/BP_CombatUnit_Guard`, figlio di `BP_CombatUnit` con `InitialRangedItem` e `RangedAimPose`.
- `BP_CombatController.ReceiveBeginPlay`: le guardie nascono da `BP_CombatUnit_Guard`.
- Funzione `PlanAIRanged` → (Found, Target): fra gli alleati vivi non nascosti con `RangedCheck` valido, minimizza HP / probabilità di colpire (CA + copertura, probabilità al quadrato con svantaggio).
- `PlanAITurn`: se non sta ripiegando, ha l'azione, non ha alleati adiacenti, ha un oggetto a distanza e la mischia non è utile in questo turno (nessun bersaglio, approccio rischioso o bersaglio oltre il movimento normale), chiama `PlanAIRanged`; con un tiro valido usa `TryRangedAttack` e imposta `AIRanged`. `UpdateAI` chiude il turno dopo il feedback del tiro.
- `StartTurn` riporta le guardie alla mischia all'inizio del loro turno; fra un turno e l'altro tengono la balestra e quindi non reagiscono, come Ezio.

### Verifiche

- Impugnatura controllata in immagini laterale, dall'alto e frontale: balestra a due mani, puntata in avanti, senza compenetrazioni evidenti.
- PIE: Guardia 1 lontana, `nessun attacco in mischia possibile, tira con Balestra pesante`, `d20 15 + 3 = 18 contro CA 20 (copertura +2) - MANCATO`, dardo visibile; poi `CanOpportunityReact` falso e `Reazione bloccata` nell'HUD. Guardia 2 senza tiro valido (copertura totale) ripiega su Scatto. Nella suite 041 le guardie vicine hanno continuato ad attaccare in mischia.
- Riga della tabella, CDO di `BP_CombatUnit_Guard` e classe di spawn riletti dopo il ricaricamento.
- Non verificato: le guardie non si spostano per cercare una linea di tiro migliore; con molte partite casuali la frequenza del tiro rispetto a Scatto/Disimpegno non è stata misurata.

## 044 — 2026-11-22 — Dardo visibile per i tiri a distanza

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto un proiettile visibile, se realizzabile senza nuovi asset esterni. Sostituisce la linea di debug delle voci 034–037, visibile solo in editor e build di sviluppo.

### Modifiche

- `/Game/CombatPrototype/Materials/M_CombatBolt`: colore base marrone ed emissivo arancione.
- `/Game/CombatPrototype/BP_CombatBolt` (Actor): componente `BoltMesh` (cilindro dell'Engine, 60 × 6 cm, senza collisioni né ombra). Variabili `Shooter`, `Target`, `Hit`. Nasce nascosto; dopo 0,05 s, con la posa di mira già applicata, parte dalla punta dell'arma (`GetRangedMuzzleLocation`) e vola in 0,2 s, in sincronia con l'impatto a 0,22 s. Colpo: termina sul bersaglio a +30 cm; mancato: passa 70 cm di lato e prosegue 2,5 m. Poi si distrugge.
- `BP_CombatController.TryRangedAttack`: rimosso `DrawDebugLine`; dopo `RollAttack` genera il dardo e ne imposta tiratore, bersaglio ed esito. Vale per Ezio e per le guardie.

### Verifiche

- PIE con dilatazione del tempo 0,05: dardo alla bocca dell'arma, poi in volo verso il bersaglio con orientamento coerente (yaw −177°, pitch 4,4°); su un mancato passa di lato; nessun dardo rimasto dopo il volo. Immagini del viewport e ravvicinate ispezionate; dopo l'ispezione il dardo è stato ingrandito da 50 × 4 a 60 × 6 cm per leggibilità, senza ripetere la cattura a quella misura.
- Blueprint compilato con avvisi come errori, salvato, ricaricato e ricompilato.

## 045 — 2026-11-22 — Build pacchettizzata Win64 e flag Nanite della balestra

Autore: Claude Code
Tipo: Verifica / Correzione
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto di provare una build pacchettizzata dopo le voci 040–044. Il pacchetto è servito come verifica, non come consegna: la configurazione del progetto non è stata modificata (mappa iniziale e GameMode globali invariati). La mappa del prototipo è stata indicata solo nel comando di cook.

### Modifiche

- `/Game/Items/Weapons/Crossbows/HandCrossbow/Materials/M_HandCrossbow` (asset della voce 037): attivato il flag d'uso `Used with Nanite`, già presente su `M_Longsword` e `M_Rapier`. Senza il flag, nel gioco pacchettizzato `SM_HandCrossbow` (mesh Nanite) usava il materiale di default, segnalato dal log runtime. Riguarda la balestra di Ezio e la balestra pesante delle guardie, che usa la stessa mesh. Materiale ricompilato e salvato.

### Verifiche

- `RunUAT BuildCookRun` Win64 Development (cook delle mappe `Lvl_TopDown` e `Lvl_CombatPrototype`, stage, pak), motore UE 5.9 installato: `BUILD SUCCESSFUL` in circa 4 minuti; cook con 0 errori e 0 avvisi, compreso l'avviso additivo di `AN_CombatHit` temuto nella voce 039, che non è comparso.
- Avvio dell'eseguibile con `/Game/CombatPrototype/Lvl_CombatPrototype`: mappa caricata, iniziativa tirata, HUD, coni di vista, turno automatico delle guardie avviato. Unico avviso pertinente: il materiale della balestra senza flag Nanite, corretto sopra.
- Secondo pacchetto dopo la correzione: cook e stage completati e avviso del materiale scomparso nel log runtime. La copia finale in una cartella di archivio temporanea è fallita perché una DLL era bloccata da un'istanza del primo avvio rimasta aperta: chiudere il launcher non chiude `UnrealGame.exe`. Istanze chiuse e archivio incoerente eliminato. La build valida è quella in `Saved/StagedBuilds/Windows/` (cartella ignorata da Git).
- Avvio pulito a istanza singola, 1280 × 720, dopo 20 s: arena con rovine e casse della voce 040, Ezio e Nephilim con le armi da mischia, guardie, coni e HUD visibili; nessun errore Blueprint runtime nel log. Una prima cattura a 1600 × 900 nei primi frame mostrava solo il cielo. Era avvenuta con altre istanze aperte e non si è ripetuta con l'istanza singola.
- Non provati nella build: partita completa, input del mouse, tiro della balestra in gioco (quindi il materiale corretto non è stato osservato a schermo, solo l'assenza dell'avviso) e prestazioni. A 1280 × 720 il pannello HUD di 1080 px a dimensione fissa occupa gran parte della metà inferiore: l'HUD non si adatta alla risoluzione.
- La cartella `Build/` nella radice del progetto non è tracciata da Git ed era già presente all'inizio della sessione: non è stata modificata intenzionalmente in questo intervento.

## 046 — 2026-11-22 — Alleati a terra, tiri contro la morte e Aiutare come in Baldur's Gate 3

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto di seguire Baldur's Gate 3 invece di Aiutare e Schivare di 5e: Schivare non esiste in BG3, mentre Aiutare rialza un alleato a terra. Serve quindi lo stato a terra con i tiri contro la morte di 5e, che BG3 adotta. Le guardie, come creature comuni, muoiono a 0 HP. La sconfitta resta "entrambi gli alleati a 0 HP", cioè tutti a terra come in BG3.

### Modifiche

- `BP_CombatController`: array `Downed`, `Stable`, `DeathSuccess`, `DeathFail` (inizializzati in `LoadSheets`) e variabile `DownedTurn`. Funzioni `DeathSave`, `ReviveUnit`, `UseHelp`, `CanHelp`, `HasDownedAlly`, `DownLabel`; evento tasto **H**.
- `ResolveAttack` mette a terra l'alleato a 0 HP; `EndTurn` non salta più gli alleati a terra non stabili; `StartTurn` chiama `DeathSave`; il tick chiude il turno a terra dopo il feedback. Messaggio di sconfitta: "Ezio e Nephilim sono a terra".
- `DeathSave` blocca il movimento del corpo prima del tiro: nella prima prova il `Possess` dell'unità a terra, senza collisione, la faceva cadere attraverso il pavimento. Una prima scrittura del grafo era stata normalizzata dall'editor DSL portando il blocco dell'esito fuori dal ramo del 20 naturale; riscritta con un controllo esplicito `Die != 20` e riletta.
- `BP_CombatUnit`: funzione `Revive`.
- `BP_CombatHUD`: pulsante `AIUTARE [H]` nella terza colonna, verde se utilizzabile; etichetta sopra il corpo da `DownLabel`; testo `A TERRA: TIRO CONTRO LA MORTE` durante il turno a terra; hitbox `Help`.
- `CombatPrototype.md` e `CONSTITUTION.md` aggiornati.

### Verifiche

PIE con l'ordine Ezio → Nephilim, turni naturali e `ForcedD20` (10 durante i turni delle guardie, che così mancano Ezio):

- Nephilim a 0 HP: a terra, senza collisione, scontro non concluso, etichetta `A TERRA  TS 0 ok / 0 ko`, Ezio vede l'alleato a terra.
- 20 naturale: 1 HP, in piedi, collisione e Animation Blueprint ripristinati, turno utilizzabile (`CanAct` vero, 9 m), nessuna chiusura automatica.
- Successo (12): 1/3, chiusura automatica del turno e passaggio alla guardia. Corpo fermo alla stessa quota durante il proprio turno dopo la correzione; immagine ispezionata.
- Aiutare: rifiutato a 6 m con messaggio, accettato a 1,2 m (1 HP, azione `Aiutare` consumata, attacchi azzerati); un secondo uso risponde che non c'è nessun alleato a terra.
- Tre successi: stabile, etichetta `A TERRA (stabile)`, turno saltato; Aiutare funziona anche dallo stato stabile.
- 1 naturale (due fallimenti) più un fallimento: morto, turno saltato, Aiutare non più disponibile, etichetta `KO`.
- Nessun errore Blueprint runtime nel log. Blueprint compilati con avvisi come errori, salvati, ricaricati dal disco e ricompilati.
- Non verificati: sconfitta con entrambi gli alleati a terra a partire da questo stato (la condizione su 0 HP è invariata e già provata nella voce 041), danni a un alleato a terra (non previsti dall'IA).

## 047 — 2026-11-22 — HUD adattivo, barra dell'iniziativa e registro di combattimento

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto di rifare l'HUD con un registro e una barra dell'iniziativa, prima di Spinta, Salto, tiri salvezza e condizioni. Il pannello della voce 042 aveva misure fisse in pixel, un elenco dei turni dentro il pannello e una zona morta per i clic di 1100 × 280 px. Sostituiti con una disposizione in stile Baldur's Gate 3: barra dei turni in alto al centro, azioni in basso a sinistra, registro in basso a destra, tutto scalato in base al viewport. L'HUD resta su Canvas in Blueprint; nessun widget UMG, per non cambiare l'architettura del prototipo.

### Modifiche

- `BP_CombatHUD`: variabili di layout (`UIScale`, `Panel*`, `Log*`, `Bar*`, `TopY`) e funzioni `ComputeLayout`, `DrawButton`, `DrawActionPanel`, `DrawInitiativeBar`, `DrawCombatLog`, `IsPointOverUI`. `ReceiveDrawHUD` riscritto: calcola il layout, disegna gli elementi di prima (coni, unità, percorso, mira, limite, riquadro del tiro) e poi barra, registro e pannello. `DrawPathPreview` e `DrawRollLog` spostati sotto la barra e scalati. Hitbox `Log` in `ReceiveHitBoxClick`.
- `BP_CombatController`: `CombatLog` (array di stringhe, massimo 80 righe), `LastLogged`, `LastActionName`, `LastBonusName`, `EndLogged`, `HideLog`; funzioni `AddLog`, `ToggleLog`, `UnitTag`; tasto **L**. Il tick registra ogni nuovo `RollText`, le azioni e azioni bonus nuove e l'esito finale; `BeginPlay` registra l'iniziativa e `StartTurn` l'intestazione del turno. `ClickCommand` e `UpdatePreview` usano `IsPointOverUI` invece della zona fissa. `DownLabel` mostra `MORTO` per un alleato morto invece di `KO`.
- `CombatPrototype.md` aggiornato (comandi, sezione Anteprima e HUD, tabella degli asset).

### Verifiche

- `BP_CombatHUD` e `BP_CombatController` compilati con avvisi come errori, salvati, ricaricati dal disco e ricompilati; grafi modificati riletti.
- PIE con viewport di 1926 × 1216 (scala 1,13): immagine ispezionata, con barra, anteprima sotto la barra, pannello, griglia dei pulsanti e registro leggibili e senza sovrapposizioni.
- Clic sul pannello e sul registro: l'unità non si muove. Clic sul pulsante Scatto: movimento da 7,5 a 15 m con Azione scaltra. **L** nasconde e rimostra il registro. Anteprima del percorso sul terreno ancora attiva.
- Registro dopo due round: iniziativa, intestazioni dei turni e `Ezio: azione bonus Scatto`. Nessun errore Blueprint runtime nel log.
- Non verificati: l'aspetto a 1280 × 720 o in finestra piccola (la scala minima 0,75 è calcolata, ma non vista in immagine); righe di attacco nel registro in questa sessione (le guardie non hanno attaccato nelle prove brevi; la registrazione usa lo stesso `RollText` già mostrato nel riquadro del tiro); build pacchettizzata.

## 048 — 2026-11-22 — Spinta e Salto

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto Spinta e Salto dopo l'HUD.

- **Spinta:** segue 5e. Sostituisce un attacco, con prova contrapposta di Atletica contro la migliore fra Atletica e Acrobazia del bersaglio, e spinge di 1,5 m. L'alternativa di 5e, buttare a terra, è rinviata al blocco delle condizioni.
- **Salto:** segue Baldur's Gate 3, come il resto del prototipo: azione bonus, senza attacchi di opportunità. Per restare nelle regole 5e, la gittata è il salto in lungo con rincorsa (Forza × 0,3 m) e la distanza si paga in movimento. Ezio ha già Disimpegno come azione bonus; il Salto dà a Nephilim una via d'uscita simile, al prezzo dell'azione bonus.
- **Spostamento:** in Blueprint, con interpolazione nel tick. Il salto è un arco senza collisione, validato prima di partire; la spinta usa una corsa con collisione, che si ferma contro muri e unità.

### Modifiche

- `S_CombatSheet` / `DT_CombatSheets`: campi `AthleticsBonus`, `AcrobaticsBonus`, `JumpMeters` (Ezio 0/7/3, Nephilim 7/1/5,4, Guardia 5/1/4,8).
- `BP_CombatController`:
  - array `Athletics`, `Acrobatics`, `JumpMeters` letti in `LoadSheets`;
  - variabili `JumpMode`, `ShoveMode`, `Jump*`, `Shift*`;
  - funzioni `CanJump`, `CanShove`, `ToggleJump`, `ToggleShove`, `EvalJump`, `ArcPoint`, `LandingOccupied`, `JumpPreview`, `TryJump`, `StartShift`, `UpdateShift`, `TryShove`, `ShoveOdds`, `ActionOdds`; tasti **J** e **G**;
  - `ClickCommand`: in modalità Salto il clic chiama `TryJump`; in modalità Spinta il clic sul nemico usa il piano di mischia anche con la balestra;
  - `UpdatePendingAttack` sceglie fra `TryShove` e `TryAttack`;
  - `UpdatePreview` mostra probabilità e testo della spinta e chiama `JumpPreview`;
  - `StartTurn` azzera le modalità; il tick chiama `UpdateShift`.
- `BP_CombatHUD`: pulsanti `SALTO [J] - bonus` (colonna 2, riga 3) e `SPINTA [G]` (colonna 3, riga 1), dorati quando la modalità è attiva; hitbox `Jump` e `Shove`.
- `CombatPrototype.md` (comandi, tabella delle regole, sezione Spinta e Salto, pulsanti) e `CONSTITUTION.md` aggiornati.

### Verifiche

- Struttura, tabella e Blueprint salvati, ricaricati dal disco e ricompilati con avvisi come errori; rilette le funzioni nuove e i rami modificati di `ClickCommand`, `UpdatePendingAttack`, `UpdatePreview`, `LoadSheets`.
- PIE, con `ForcedD20` 10 per la Spinta e 1 nei turni delle guardie:
  - **Valori della scheda:** letti correttamente.
  - **Anteprima della Spinta:** `Atletica +7 contro +5, riesce al 57%`.
  - **Spinta di Nephilim con clic reale:** 17 contro 15, riuscita. Guardia spostata di 137 cm, fermata da un ostacolo; in campo aperto, in una seconda prova, esattamente 150 cm. Attacchi da 2 a 1, azione non esaurita, guardia di nuovo in modalità camminata.
  - **Spinta di Ezio:** 10 contro 15, fallita, nessuno spostamento. Registro e riquadro del tiro corretti.
  - **Salto di Ezio con clic reale:** l'anteprima mostra l'arco verde, immagine ispezionata. Ezio salta di 2,3 m allontanandosi da una guardia adiacente: nessun attacco di opportunità, reazioni e HP invariati, movimento da 7,5 a 5,16 m, azione bonus `Salto`.
  - **Dopo il salto:** il secondo salto viene rifiutato e la camminata successiva consuma il budget normalmente. Campionamento: a metà salto 137 cm di quota senza modalità di movimento, poi atterraggio a camminare.
  - **Rifiuti di `EvalJump`:** 4 m (oltre i 3 m di Ezio) e 0,3 m (troppo vicino).
  - Nessun errore Blueprint runtime nel log.
- Non verificati: rifiuto per ostacolo sulla traiettoria, per dislivello e su superficie non piana (la logica è stata riletta, non provata in PIE); salto su un blocco basso; immagine del salto a metà arco (la cattura è arrivata dopo l'atterraggio, la traiettoria è verificata solo dalle coordinate).

## 049 — 2026-11-22 — Tiri salvezza, prono, spaventato e manovre di Nephilim

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha chiesto tiri salvezza e condizioni base dopo Spinta e Salto. Nel prototipo nessuna capacità imponeva tiri salvezza; le guardie Veterano non ne hanno. Per avere una fonte reale e in stile Baldur's Gate 3, Nephilim (Guerriero 5, sottoclasse finora non definita) diventa Maestro di battaglia, con 4d8 superiorità e CD 15 (8 + competenza 3 + Forza 4). Le manovre scelte impongono due condizioni utili e leggibili:

- **Attacco sbilanciante:** tiro salvezza su Forza, se fallisce il bersaglio è prono.
- **Attacco minaccioso:** tiro salvezza su Saggezza, se fallisce il bersaglio è spaventato.

La Spinta riceve l'opzione "a terra" di 5e, rinviata nella voce 048. Le altre condizioni non hanno fonti nel prototipo e restano fuori. È una scelta di progetto presa in autonomia, come autorizzato dall'utente per questo blocco; è facile da cambiare nella scheda.

### Modifiche

- `S_CombatSheet` / `DT_CombatSheets`: campi `SaveStr`…`SaveCha`, `SuperiorityDice`, `SuperiorityDie`, `ManeuverDC` (valori in `CombatPrototype.md`).
- `BP_CombatUnit`:
  - variabile `IsProne` e funzione `SetProne`;
  - `PlayHitFeedback` e `PlayAttackFeedback`, a fine animazione, tornano alla posa prona se serve;
  - `Revive` azzera `IsProne`.
- `BP_CombatController`:
  - array `SaveBonus`, `Prone`, `FearSource`, `FearTurns`, `SuperiorityLeft`, `SuperiorityDie`, `ManeuverDC`; variabili `ManeuverMode`, `ShoveProne`, `LastSaveText`;
  - funzioni `AbilityName`, `SavingThrow`, `IsProne`, `IsFrightened`, `CondAdv`, `CondDis`, `ApplyProne`, `ManeuverLine`, `ManeuverOnHit`, `ToggleManeuver`; tasti **B** e **M**;
  - `ToggleShove` riscritta con il ciclo spingi → a terra → annulla; `ActionOdds` e `UnitTag` riscritte;
  - modifiche a `RollAttack`, `AttackOdds`, `TryAttack`, `TryShove`, `ShoveOdds`, `ToggleJump`, `StartTurn` (rialzarsi, azzeramento modalità), `EndTurn` (durata della paura), `ResolveAttack` (azzeramento condizioni a 0 HP) e `LoadSheets`.
- `BP_CombatHUD`: pulsanti `SBILANCIARE [B] (n)` e `MINACCIARE [M] (n)` nella quarta colonna; etichetta `SPINTA: SPINGI` / `SPINTA: A TERRA`; hitbox `Trip` e `Menace`.
- `CombatPrototype.md` e `CONSTITUTION.md` aggiornati.

### Verifiche

- Struttura, tabella e Blueprint salvati, ricaricati dal disco e ricompilati con avvisi come errori; riletti `RollAttack`, `StartTurn`, `EndTurn`, `ManeuverOnHit`, `TryShove`, `SavingThrow`, `SetProne` e `PlayHitFeedback`.
- PIE con clic reali e `ForcedD20` 10 per Nephilim e 1 nei turni delle guardie:
  - **Valori della scheda:** `SaveBonus` e superiorità letti correttamente.
  - **Attacco sbilanciante:** l'anteprima mostra `Sbilanciare: +1d8, TS Forza CD 15, cade prono al 55%`. Il colpo va a segno (17 contro CA 17), +3 di dado, TS Forza 13 contro 15 fallito: guardia prona, dadi superiorità da 4 a 3, modello in animazione singola.
  - **Anteprima contro il prono:** `Colpire 80% (vantaggio)`. `CondAdv` in mischia, `CondDis` a distanza contro il prono e `CondDis` per l'attaccante prono risultano tutti veri.
  - **Attacco minaccioso:** tiro con vantaggio (10, 10), TS Saggezza 10 contro 15 fallito, guardia spaventata (`FearTurns` 2). Barra dell'iniziativa `HP 33/58 PRONO SPAVENTATO`.
  - **Spinta a terra:** con Azione impetuosa, G G, anteprima `SPINTA A TERRA: ... 57%`, 17 contro 15: seconda guardia prona senza spostamento.
  - **Turni delle guardie:** entrambe si rialzano a inizio turno (registro: `si rialza dalla posizione prona`, animazione di nuovo Animation Blueprint). La guardia spaventata attacca `con svantaggio (1, 1)`.
  - **Durata della paura:** al turno successivo di Nephilim la paura è ancora attiva (`FearTurns` 1); termina alla fine di quel turno (`FearSource` −1).
  - Nessun errore Blueprint runtime nel log. Immagini ispezionate.
- Limite visivo osservato: la posa prona delle guardie è quella china della clip `AN_CombatDeath` del template, già segnalata per i corpi; la posa distesa si vede solo con le animazioni di Ezio e Nephilim, che però nessuna fonte rende proni.
- Non verificati: critico con manovra (raddoppio del dado), tiro salvezza riuscito con la condizione non applicata (la logica è stata riletta), attacco di un alleato prono, build pacchettizzata.

## 050 — 2026-11-23 — Animazione di salto di Nephilim

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha segnalato di aver già preparato le animazioni per il salto. Dei due personaggi, solo Nephilim ha una clip importata: `A_Nephilim_Jump_Over_Obstacle_2` (sorgente `SourceArt/Characters/Nephilim/..._Jump_Over_Obstacle_2.fbx`). La clip di Ezio `Jump_Over_Obstacle_2` esiste solo fra le sorgenti V1 (`SourceArt/Characters/Ezio/`); la sua copia importata era stata eliminata con le clip V1 (voce precedente, riga "Eliminate tramite l'editor le nove clip V1"). La costituzione ammette per Ezio solo le sorgenti in `v2/`, dove il salto non c'è: Ezio resta senza animazione finché non arriva una sorgente v2. La clip di Nephilim è sul posto (radice ferma, bacino che si sposta di pochi centimetri), quindi è compatibile con l'arco che già muove l'attore; non sono state cambiate le impostazioni dell'asset.

### Modifiche

- `BP_CombatUnit`: variabile `JumpAnim` (AnimSequence) e funzioni `PlayJump(Duration)`, che riproduce la clip con velocità lunghezza / durata dell'arco, ed `EndJump`, che torna all'Animation Blueprint o alla posa prona. Senza clip non fanno nulla.
- `BP_CombatUnit_Nephilim`: `JumpAnim` = `A_Nephilim_Jump_Over_Obstacle_2`.
- `BP_CombatController`: `TryJump` chiama `PlayJump`; `UpdateShift`, alla fine di uno spostamento senza collisione (il salto), chiama `EndJump`.
- `CombatPrototype.md` aggiornato. Sostituisce il limite "nessuna animazione di salto" della voce 048 per Nephilim.

### Verifiche

- Blueprint compilati con avvisi come errori, salvati, ricaricati dal disco e ricompilati; rilette le chiamate in `TryJump` e `UpdateShift`; dopo il ricaricamento `JumpAnim` di Nephilim punta ancora alla clip.
- PIE: salto di Nephilim di 4 m (durata 0,9 s). Durante l'arco animazione singola, a 229 cm di quota; dopo l'atterraggio Animation Blueprint, nessuno spostamento ancora attivo. Immagine a metà salto ispezionata: posa di salto in aria. Nessun errore Blueprint runtime nel log.
- Verificato anche, nella stessa prova, il rifiuto di un salto per ostacolo sulla traiettoria (`un ostacolo blocca la traiettoria`), non provato nella voce 048.
- Non verificati: salto di Ezio con animazione (manca la clip v2); build pacchettizzata.

## 051 — 2026-11-23 — Rimozione delle sorgenti V1 di Ezio

Autore: Claude Code
Tipo: Documentazione / Pulizia sorgenti
Stato: Attuale

### Richiesta e scelte

L'utente ha confermato che la V1 di Ezio non si usa più e ha chiesto di cancellarla, usando solo la v2. Prima della cancellazione è stato verificato che nessun asset in `/Game/Characters/Ezio` la usi.

### Modifiche

- Cancellati da `SourceArt/Characters/Ezio/` tutti gli elementi fuori da `v2/`: 20 voci, cioè 23 file tracciati da git. Sono la mesh `..._character_mixamo.fbx` con la cartella `.fbm`, 4 texture PNG e 14 FBX di animazione V1, compreso `Jump_Over_Obstacle_2`. `v2/` non è stata toccata.
- I file sono stati aggiunti nel commit `e3d8162`: restano recuperabili dalla cronologia git. La cancellazione non è stata registrata in un commit.
- `CONSTITUTION.md` e `CombatPrototype.md` aggiornati.

### Verifiche

- Prima della cancellazione: i dati d'importazione di tutti gli asset di `/Game/Characters/Ezio` puntano a `SourceArt/Characters/Ezio/v2/`. Riguarda la mesh, le texture e le 14 animazioni; materiale, scheletro, Physics Asset, Animation Blueprint e blend space non hanno sorgente esterna.
- Dopo la cancellazione: in `SourceArt/Characters/Ezio/` resta solo `v2/`; `git status` mostra le 23 cancellazioni.
- Non sono stati toccati asset Unreal; nessuna prova PIE necessaria.
- Il salto di Ezio resta senza animazione finché non arriva una sorgente v2 (voce 050).

## 052 — 2026-11-23 — Animazione di salto di Ezio

Autore: Claude Code
Tipo: Funzionalità
Stato: Attuale

### Richiesta e scelte

L'utente ha messo in `Content/` una cartella `Meshy_AI_Wily_Halfling_Wanderer_20261115_animated_mixamo_without_skin_Jump_Over_Obstacle_2`. La cartella contiene l'FBX del salto di Ezio e quattro texture. L'utente ha chiesto di importare la clip, applicarla al salto e cancellare la cartella. Le sorgenti non devono stare sotto `Content/`, e la costituzione vuole quelle di Ezio in `v2/`: i file sono quindi stati spostati in `SourceArt/Characters/Ezio/v2/Jump/` invece di essere eliminati, come le altre sorgenti v2 (`Death/`, `HitReaction/`, `ThrustSlash/`). La cartella in `Content/` non esiste più.

### Modifiche

- Importata `/Game/Characters/Ezio/Animations/A_Ezio_Jump`: solo animazione su `SK_Ezio_Skeleton`, con la stessa pipeline Interchange delle altre clip di Ezio; nessuna mesh, materiale o texture creati. 0,87 s, radice ferma: compatibile con l'arco che muove l'attore.
- `BP_CombatUnit_Ezio`: `JumpAnim` = `A_Ezio_Jump`. Usa `PlayJump` / `EndJump` della voce 050.
- `CombatPrototype.md` aggiornato. Sostituisce il limite "Ezio senza animazione di salto" delle voci 050 e 051.

### Verifiche

- Dopo l'importazione non c'erano asset estranei in `/Game/Characters/Ezio` né nella radice di `/Game`.
- `A_Ezio_Jump` e `BP_CombatUnit_Ezio` salvati, ricaricati dal disco; Blueprint ricompilato con avvisi come errori. Dopo il ricaricamento `JumpAnim` punta alla clip.
- PIE: salto di Ezio di 2,8 m (durata 0,78 s, movimento da 7,5 a 4,7 m). Durante l'arco animazione singola, a 223 cm di quota; dopo l'atterraggio Animation Blueprint. Immagine a metà salto ispezionata: Ezio raccolto in aria. Nessun errore Blueprint runtime nel log.

## 053 — 2026-11-23 — Salto orientato verso la destinazione e clip sul posto

Autore: Claude Code
Tipo: Correzione
Stato: Attuale

### Richiesta e scelte

L'utente ha segnalato due difetti del salto delle voci 050 e 052:

1. L'animazione partiva con l'orientamento iniziale del personaggio: chi guardava a nord saltava verso est restando rivolto a nord.
2. Il personaggio atterrava oltre il punto d'arrivo e poi veniva come teletrasportato indietro.

Causa del secondo difetto: le clip Mixamo `Jump_Over_Obstacle_2` non sono sul posto. L'osso radice è fermo, ma il bacino (`Hips`) avanza in spazio componente: 171 cm per Ezio, 340 cm per Nephilim. Questo spostamento si sommava all'arco che già muove l'attore. Al ritorno all'Animation Blueprint il corpo tornava di scatto sull'attore. La misura precedente (voce 050, "bacino che si sposta di pochi centimetri") leggeva lo spazio locale dell'osso: con la scala 0,01 dell'armatura Mixamo quei valori sono metri. La valutazione era quindi errata.

Si conserva l'arco validato (atterraggio libero e traiettoria verificati prima di partire). L'animazione è stata resa sul posto, invece di passare al root motion, che non avrebbe garantito il punto di arrivo calcolato. Gli originali importati restano intatti e reimportabili; le versioni usate sono copie.

### Modifiche

- Nuove clip `/Game/Characters/Ezio/Animations/A_Ezio_Jump_InPlace` e `/Game/Characters/Nephilim/Animations/A_Nephilim_Jump_InPlace`, duplicate dagli originali. Sulla traccia `Hips` (27 chiavi) l'asse locale dell'avanzamento è fissato al valore del primo fotogramma; rotazioni, scala e alzata verticale restano invariate. Modifica eseguita con `AnimSequencerController.set_bone_track_keys`, cioè tramite le API dell'editor, come strumento di authoring.
- `BP_CombatUnit_Ezio` e `BP_CombatUnit_Nephilim`: `JumpAnim` punta alle copie sul posto.
- `BP_CombatController.TryJump`: prima di `StartShift` imposta la rotazione dell'unità attiva verso `JumpDest`. Usa `MakeRotFromZX` con Z verso l'alto, quindi solo yaw. La spinta non è toccata.
- `CombatPrototype.md` aggiornato.
- Sostituisce l'uso diretto di `A_Nephilim_Jump_Over_Obstacle_2` (voce 050) e `A_Ezio_Jump` (voce 052). Rettifica la descrizione "sul posto" data in quelle voci.

### Verifiche

- **Clip, in spazio componente:**
  - Prima: il bacino avanza da 1 a 171 cm (Ezio) e da 3 a 340 cm (Nephilim).
  - Dopo: il bacino resta a 1 cm (Ezio) e a 3 cm (Nephilim) per tutta la clip. L'altezza è invariata (Ezio 50→71→50 cm, Nephilim 103→145→103 cm).
- **Salvataggio:** clip, unità e controller salvati, ricaricati dal disco e ricompilati con avvisi come errori. Rilette la rotazione in `TryJump` e le assegnazioni di `JumpAnim`.
- **PIE, orientamento:** Ezio rivolto a yaw 0 salta verso −90° e Nephilim rivolto a 180° salta verso 90°. Dopo `TryJump` e dopo l'atterraggio lo yaw coincide con la direzione del salto. La chiamata ruota quindi l'unità, non il controller.
- **PIE, atterraggio senza scatto:** il bacino segue l'attore lungo l'arco. Dopo l'atterraggio resta fermo sopra il punto d'arrivo per tutto il resto del campionamento (Ezio −427 contro l'attore −430; Nephilim 540 contro 550), senza salti di posizione.
- **Immagini ispezionate:** Ezio in volo piegato verso la direzione del salto, a 223 cm di quota; entrambi a terra nel punto d'arrivo dopo il salto. Nessun errore Blueprint runtime nel log.
- **Limite:** il campionamento a passi di 0,05 s dello script è più lento del gioco, quindi pochi campioni cadono durante l'arco. La continuità all'atterraggio è verificata dai campioni successivi e dalle immagini, non fotogramma per fotogramma.
