# Combat Prototype — stato attuale

Questa guida descrive il comportamento corrente. La storia delle scelte, delle prove e delle soluzioni sostituite è in [CHANGELOG.md](CHANGELOG.md). Le regole di lavoro condivise sono in [CONSTITUTION.md](CONSTITUTION.md).

## Avvio e controlli

Aprire `/Game/CombatPrototype/Lvl_CombatPrototype` e premere **Play**. La mappa iniziale globale del progetto resta `Lvl_TopDown`: il prototipo va aperto esplicitamente.

- **Clic sinistro sul terreno:** muove l'unità attiva lungo la navigazione, entro il movimento residuo.
- **Clic sinistro su una guardia:** sceglie un punto libero a 1,5 m dal nemico, avvicina l'unità e attacca appena entra in portata. Se il budget non basta, si ferma senza attaccare.
- **Altro clic sul terreno:** annulla l'attacco in attesa e impartisce un nuovo ordine.
- **Scatto** (pulsante o **Q**) e **Disimpegno** (pulsante o **E**): per Ezio usano prima l'azione bonus (Azione scaltra), poi l'azione; per gli altri usano l'azione, se non è ancora stato fatto alcun attacco.
- **Recuperare energie** (pulsante o **R**, solo Nephilim): azione bonus, una volta per scontro.
- **Nascondersi** (pulsante o **N**): azione, o azione bonus per Ezio; possibile solo fuori dalla vista di tutti i nemici.
- **V:** mostra o nasconde i coni di vista delle guardie (visibili all'avvio). Cambia solo la visualizzazione: le regole di vista restano attive.
- **Rotellina su / giù:** avvicina / allontana dolcemente la camera dell'unità attiva, fra 4,5 e 22 metri. Avvicinandosi la camera si abbassa; lo zoom resta uguale al cambio turno.
- **Fine turno** oppure **Spazio:** passa alla prossima unità viva nell'ordine di iniziativa.
- **Riavviare PIE:** ricomincia lo scontro.

Durante l'animazione di attacco i comandi sono brevemente bloccati e l'HUD lo segnala.

## Regole dello scontro

| Regola | Valore attuale |
| --- | --- |
| Squadra del giocatore | Ezio e Nephilim |
| Avversari | Guardia 1 e Guardia 2, automatiche |
| Ordine | Iniziativa tirata a inizio scontro: d20 + bonus della scheda (Ezio +4, Nephilim +0, guardie +1); parità al bonus più alto, poi al giocatore, poi all'indice |
| Reazione | Una per round, ricaricata all'inizio del proprio turno; usata per l'attacco di opportunità |
| Attacco di opportunità | Quando un nemico vivo, che si muove volontariamente nel proprio turno, esce dai 1,5 m: un attacco con l'arma (un solo tiro anche per chi ha Attacco extra) |
| Movimento per turno | 9 metri effettivamente percorsi in piano |
| Azione per turno | Una fra Attacco, Scatto e Disimpegno; il primo attacco consuma l'azione |
| Azione bonus per turno | Una: Azione scaltra di Ezio (Scatto, Disimpegno o Nascondersi) oppure Recuperare energie di Nephilim (1d10 + livello, 1 uso per scontro) |
| Attacchi per turno | Azione Attacco: 1 (Ezio) o 2 con Attacco extra (Nephilim, guardie); movimento libero fra un attacco e l'altro |
| Scatto | Aggiunge il movimento della scheda (9 m) al residuo e al totale del turno |
| Disimpegno | Per il resto del turno il movimento non provoca attacchi di opportunità |
| Corpo a corpo | 1,5 metri fra i centri (circa 66 cm fra le capsule), tolleranza tecnica di 0,5 cm |
| Schede | `Data/DT_CombatSheets` (struttura `Data/S_CombatSheet`) |
| Ezio | Halfling Ladro 5: HP 38, CA 16, stocco +7, 1d8+4, Attacco furtivo 3d6 |
| Nephilim | Guerriero 5 (razza da definire): HP 49, CA 18, spada lunga e scudo +7, 1d8+6 (Duellare), 2 attacchi |
| Guardie | Veterano: HP 58, CA 17, spada lunga +5, 1d8+3, 2 attacchi |
| Tiro per colpire | d20 + bonus ≥ CA; 20 naturale critico (dadi raddoppiati, compreso il furtivo), 1 naturale sempre mancato |
| Impatto | Dopo 0,22 secondi dall'inizio dell'attacco |
| Blocco dei comandi per il feedback | 1,35 secondi |
| Fine dello scontro | Una squadra interamente a 0 HP |

Il riferimento è Baldur's Gate 3: movimento libero e naturale sul NavMesh, anche di pochi passi, misurato in metri, senza griglia né regole sulle diagonali. Non esistono ancora coperture con bonus difensivi, abilità aggiuntive o velocità diverse per razza.

Le unità sconfitte vengono saltate nei turni. Il round aumenta quando l'ordine torna alla prima posizione. Vittoria e sconfitta bloccano nuovi comandi di gioco. Senza Disimpegno, ogni uscita dalla portata di un nemico con la reazione disponibile provoca, anche passandogli accanto. Le guardie non usano Scatto, Disimpegno, Nascondersi né azioni bonus.

## Nascondersi

Regole D&D 5e, scelte con l'utente come base modificabile:

- **Cono di vista** (regola della casa in stile Baldur's Gate 3, voce 022): un'unità vede solo entro il proprio cono, `VisionAngle` gradi totali centrati sulla direzione dell'attore e `VisionRange` metri (oggi 120° e 18 m per tutti, nella scheda). Le guardie partono rivolte verso gli alleati; poi conservano l'orientamento dell'ultimo movimento, attacco o colpo ricevuto. Alle spalle o di fianco a una guardia, oltre 60° dall'asse, ci si può nascondere anche allo scoperto.
- **Requisito:** nessun nemico vivo ha linea di vista sull'unità, cioè la vede entro il cono e senza ostacoli. `IsSeenBy` traccia sul canale Visibility dagli occhi del nemico (+60 cm) alla testa (+60 cm) e al centro del bersaglio, ignorando le unità; basta una traccia libera per vedere. Il muro centrale (1,8 m) blocca la vista; all'inizio dello scontro separa già le due squadre.
- **Prova:** d20 + `StealthBonus` contro la Percezione passiva più alta fra i nemici vivi; con `StealthDisadvantage` si tirano due d20 e si tiene il minore. Ezio +10 (Maestria), Nephilim +1 con svantaggio (cotta di maglia), guardie Percezione passiva 12. L'azione o l'azione bonus è spesa anche se la prova fallisce. `ForcedD20` vale anche per questa prova.
- **Effetti:** il nascosto non può essere scelto come bersaglio dalle guardie (`PlanAITurn`), non provoca attacchi di opportunità e il suo prossimo attacco ha vantaggio (due d20, il maggiore), che attiva anche l'Attacco furtivo. L'anteprima mostra la probabilità con vantaggio.
- **Rivelazione** (`RevealCheck`, a ogni tick): attaccando, anche con una reazione, o restando fermo in vista di un nemico. Durante il proprio movimento l'unità resta nascosta, quindi può uscire dalla copertura e attaccare con vantaggio, come in Baldur's Gate 3; se si ferma in vista viene scoperta. Un nemico che durante il suo turno arriva in vista di un'unità ferma la scopre.
- Il nascosto è indicato con `(nascosto)` accanto al nome sopra l'unità.
- **Coni sul terreno** (`DrawVisionCones` nell'HUD): per ogni guardia viva, contorno e raggi del cono proiettati a terra, visibili durante lo scontro finché non vengono nascosti con **V** (`HideCones` nel controller; voce 023). 25 tracce per guardia all'altezza degli occhi tagliano il cono sui muri più alti di 1,6 m; i blocchi più bassi non lo fermano. Il cono è giallo, e diventa rosso se vede l'unità alleata attiva.

## Azione e azione bonus

- `StartTurn` imposta `ActionAvailable` e `BonusAvailable`; `ActionName` e `BonusName` registrano cosa è stato usato.
- `TryAttack` consuma l'azione al primo attacco; gli attacchi successivi di Attacco extra restano disponibili.
- `CanAct` riunisce i controlli comuni (turno del giocatore, comandi non bloccati); `CanUseAction` e `CanUseBonus` vi aggiungono la risorsa. `BonusMovement` è vero se l'unità ha Azione scaltra e l'azione bonus libera.
- `UseDash` / `UseDisengage` scelgono automaticamente l'azione bonus quando `BonusMovement` è vero, altrimenti l'azione. Ezio può quindi scattare due volte (27 m) oppure disimpegnarsi e attaccare nello stesso turno. Il Disimpegno già attivo non viene ripetuto.
- `UseSecondWind` cura 1d10 + `Level`, senza superare gli HP massimi, e scala `SecondWindLeft`; è rifiutato a HP pieni.

## Camera e zoom

La camera segue l'unità attiva, comportamento verificato in PIE e confermato dall'utente nella voce 025. Non è fissata su Ezio: passa anche a Nephilim e alle guardie con la stessa distanza e inclinazione.

- Rotellina verso l'alto per avvicinarsi, verso il basso per allontanarsi.
- Tenere premuto il **tasto centrale del mouse** e trascinare a destra/sinistra per ruotare la camera attorno all'unità attiva. Al rilascio la rotazione si ferma. Lo spostamento verticale del mouse non cambia l'inclinazione, che resta legata allo zoom.
- La rotazione è condivisa fra tutte le camere e resta uguale al cambio turno, anche durante i turni delle guardie. Non consuma risorse; mentre si tiene premuto il tasto centrale, i clic di movimento/attacco sono ignorati.
- Distanza iniziale 22 m; limiti 4,5–22 m, misurati come lunghezza dello Spring Arm.
- Ogni scatto modifica la destinazione dello zoom in proporzione (fattore 0,85 avvicinandosi, inverso allontanandosi). `FInterpTo` con velocità 8 effettua il passaggio morbido.
- L'inclinazione varia progressivamente da −50° a 22 m a −25° a 4,5 m, mantenendo l'orientamento orizzontale della camera.
- Durante l'avvicinamento il punto d'inquadratura si abbassa gradualmente fino a 80 cm, così Ezio è visibile per intero sopra il pannello HUD; a 22 m l'offset torna a zero.
- Distanza, inclinazione e inquadratura vengono applicate a tutti gli Spring Arm dalla stessa impostazione del controller. Cambiare turno o unità non resetta lo zoom; una nuova partita riparte da 22 m.
- Lo zoom è disponibile anche durante i turni delle guardie e non consuma risorse di combattimento.

Le variabili sono nei default di `BP_CombatController`, categoria `Camera`: `CameraZoomMin` / `CameraZoomMax` (450 / 2200 cm), `CameraZoomStepFactor` (0,85), `CameraZoomInterpSpeed` (8), `CameraPitchNear` / `CameraPitchFar` (−25 / −50 gradi), `CameraFocusNearZ` (−80 cm). `InitializeCameraZoom`, `ChangeCameraZoom` e `UpdateCameraZoom` gestiscono inizializzazione, input e interpolazione.

`UpdateCameraRotation` legge ogni tick il tasto centrale e la posizione orizzontale del mouse: il primo frame registra il punto iniziale, i successivi aggiornano `CameraYaw`, normalizzato a ±180°. `CameraRotateSensitivity` vale 0,25 gradi per pixel; `CameraDragging` e `CameraDragLastX` conservano lo stato del trascinamento. `InitializeCameraZoom` legge lo yaw iniziale dello Spring Arm del primo pawn (45°); `UpdateCameraZoom` applica lo stesso yaw a tutte le camere. Una nuova partita ripristina l'orientamento iniziale.

## Movimento, unità ferme e attacco

- Il controller usa il movimento e la camera del template TopDown. La camera segue l'unità posseduta con zoom condiviso fra 4,5 e 22 m, come descritto sopra.
- Muri e capsule delle unità ferme partecipano al NavMesh. Le capsule ferme usano `NavArea_Null`; l'unità attiva è esclusa dal proprio ostacolo.
- Al cambio turno si attende almeno 0,15 secondi e il completamento dell'aggiornamento della navigazione.
- RVO è **disattivato**: l'aggiramento dei personaggi è pianificato prima del movimento, invece di essere aggiunto da un evitamento locale.
- La portata è calcolata da `InMelee`: distanza fra i centri ≤ `MeleeCenterRange` (150 cm) + 0,5 cm.
- `FindAttackPlan` valuta 24 punti, a passi di 15°, a 2 cm all'interno della portata dal bersaglio; scarta quelli occupati o non raggiungibili e sceglie il percorso più corto fra i candidati validi. L'attacco parte appena l'unità entra in portata durante l'avvicinamento.
- L'ordine mantiene il punto scelto. L'avvicinamento finale è rifinito con il movimento del template per compensare la tolleranza di arrivo della navigazione.
- Se non esiste un punto libero in portata, l'attacco non parte. Un clic su un nemico dopo aver già usato l'attacco non ne avvia un secondo.
- Il budget usa la distanza realmente percorsa: anche le deviazioni consumano metri. Al limite il movimento viene fermato e corretto.
- Dopo 0,8 secondi senza almeno 2 cm di progresso si interrompe l'ordine: la guardia passa il turno, mentre il giocatore può scegliere una nuova destinazione. Resta anche un limite temporale di sicurezza per il turno automatico.

Le guardie scelgono fra gli alleati vivi in base al costo di un piano d'attacco raggiungibile, si avvicinano, colpiscono e terminano il turno dopo il feedback. La scelta del bersaglio non considera ancora gli attacchi di opportunità.

## Iniziativa e attacchi di opportunità

- `RollInitiative`, chiamata in `ReceiveBeginPlay`, riempie `InitRoll` e ordina `TurnOrder` con la chiave `InitKey` (tiro × 100 + bonus × 2 + 1 per i giocatori); `TurnPos` è la posizione corrente. Il riepilogo appare nel riquadro del tiro per 6 secondi.
- `EndTurn` avanza `TurnPos`, salta le unità a 0 HP e incrementa `Round` al ritorno alla posizione 0.
- `StartTurn` ricarica `ReactionAvailable` dell'unità attiva e registra in `ReachPrev` quali unità sono entro 1,5 m.
- `CheckOpportunity`, a ogni tick dopo il budget, confronta la portata precedente con quella attuale mentre l'unità attiva ha un ordine di movimento. All'uscita dalla portata di un nemico vivo con reazione chiama `OpportunityAttack`.
- `OpportunityAttack` ferma il movimento, consuma la reazione, avvia l'animazione del reattore e tira con `RollAttack(Reactor, Active)`; il testo del tiro inizia con `ATTACCO DI OPPORTUNITA`. Il colpo si applica con `ResolveAttack` come gli altri.
- Al termine del feedback (`ResumeMotion`), se l'unità è viva riprende verso la stessa destinazione e l'eventuale attacco in attesa resta valido; se è caduta, il turno passa. Il movimento interrotto conserva il budget non speso.
- Al massimo un attacco di opportunità per tick: se due nemici vengono lasciati nello stesso istante, il secondo reagisce dopo la ripresa, se la portata risulta ancora persa.
- L'Attacco furtivo può applicarsi anche a un attacco di opportunità di Ezio, perché vale una volta per turno, anche nel turno altrui.

## Schede e tiri

Le schede sono nella tabella `/Game/CombatPrototype/Data/DT_CombatSheets` e si modificano nell'editor senza toccare i grafi. Righe `Ezio`, `Nephilim` e `Guardia` (usata da entrambe le guardie). `LoadSheets` le legge all'avvio e le copia negli array del controller (`MaxHP`, `ArmorClass`, `AttackBonus`, dadi di danno, `AttacksPerAction`, `SneakDice`, `SpeedMeters`).

- `RollAttack(Attacker, Target)` tira il d20 e i dadi di danno, scrive il riepilogo in `RollText` e restituisce colpito, critico e danno; `ResolveAttack` lo applica all'impatto, con l'attaccante come sorgente della reazione animata.
- L'Attacco furtivo si applica una volta per turno, al primo colpo andato a segno con vantaggio oppure con l'altro alleato vivo entro 1,5 m dal bersaglio. Per ora il vantaggio deriva solo dal Nascondersi; lo svantaggio solo dalla Furtività in armatura pesante.
- La velocità per razza (7,5 m per l'halfling) è prevista ma rimandata: la scheda di Ezio usa 9 m.
- Campi `Level`, `CunningAction` e `SecondWindUses`: Ezio livello 5 con Azione scaltra, Nephilim livello 5 con 1 uso di Recuperare energie, guardie livello 0 (creature senza livello).
- Campi `StealthBonus`, `StealthDisadvantage`, `PassivePerception`: Ezio +10 / no / 14, Nephilim +1 / sì / 11, guardie +1 / no / 12.
- Non implementati: tiri salvezza, altre reazioni (Schivata prodigiosa), Cercare (prova attiva di Percezione), oscurità e coperture parziali, Azione impetuosa, sorpresa, riposi.
- `ForcedD20` (variabile del controller modificabile sull'istanza) forza il d20 dei tiri per colpire nelle prove se è maggiore di 0; normalmente vale 0. Non influisce sull'iniziativa.

## Anteprima e HUD

Passare il cursore sul terreno o sul nemico per vedere il percorso prima del clic.

- **Verde:** tratto entro il budget.
- **Rosso:** tratto oltre il budget.
- **Barra rossa sul nemico:** solo se il movimento non basta; nessuna barra quando il bersaglio è raggiungibile per attaccare.
- **Riquadro superiore:** costo e residuo stimati in metri, con un decimale, e stato dell'attacco; sul nemico anche la probabilità di colpire e i dadi di danno (`+3d6 furtivo` quando applicabile). Il prefisso `ATTACCO DI OPPORTUNITA!` (`ProvokesOpportunity`) segue il percorso entro il budget, campionato ogni 25 cm. Per ogni nemico con reazione avvisa se il percorso, dopo essere entrato entro 1,5 m + 15 cm di margine, se ne allontana oltre 1,5 m. Il margine compensa i piccoli scarti fra percorso pianificato e movimento reale (osservati circa 8 cm), quindi un passaggio molto ravvicinato può essere segnalato anche se poi non provoca. Avvicinarsi al bersaglio non genera avvisi. Con Disimpegno attivo l'avviso non compare.
- **Riquadro del tiro:** sotto l'anteprima per 5 secondi, con d20, bonus, totale contro CA, esito e dettaglio del danno; all'avvio mostra l'iniziativa.
- **Pannello inferiore:** round, unità attiva e classe, movimento residuo sul totale del turno (raddoppiato da Scatto), attacchi rimasti e arma; dopo Scatto o Disimpegno `Azione usata: ...`; pulsanti Scatto e Disimpegno (con `- bonus` quando useranno l'azione bonus), grigi e non cliccabili se non disponibili; pulsante Recuperare energie finché resta un uso; pulsante Nascondersi (con `- bonus` per Ezio), attivo solo fuori vista; stato di azione e azione bonus; riga `Coni di vista [V]: visibili/nascosti`; il pannello è largo 860 px e i clic sopra di esso non muovono l'unità; elenco in ordine di iniziativa con `>` sull'unità attiva, iniziativa, HP/HP massimi, CA e stato della reazione; Fine turno.
- **Presso le unità:** nome, indicatore dell'unità attiva, barra HP e valore numerico; colori distinti per squadra.
- **Al colpo:** testo flottante `-N`, `CRITICO -N` oppure `MANCATO`; reazione animata solo se il colpo va a segno.

L'anteprima usa la stessa pianificazione dell'ordine ed è aggiornata circa 10 volte al secondo. Le stime sono indicate con `~`: tolleranze del percorso, curve di movimento e tick possono produrre piccoli scostamenti dal consumo effettivo. L'anteprima è nascosta sull'HUD, durante il feedback, nei turni delle guardie e a scontro concluso.

## Animazioni e sconfitta

L'attaccante si volta verso il bersaglio e riproduce la propria `AttackAnim`, a `AttackPlayRate`; le unità vive tornano al proprio Animation Blueprint al termine della clip. Al momento dell'impatto vengono applicati danno, reazione e numero flottante. Le clip di feedback sono variabili di `BP_CombatUnit` (`AttackAnim`, `HitAnim`, `DeathAnim`): le guardie mantengono le copie del template con `ABP_Unarmed`; Ezio e Nephilim hanno le proprie animazioni. La reazione usa `HitPlayRate` (default base 1) e attende la durata effettiva prima di ripristinare l'Animation Blueprint. Un mancato non avvia la reazione. Una variabile vuota salta l'animazione; se manca Death, un colpo letale riproduce Hit e resta sull'ultimo fotogramma, senza tornare alla locomozione.

### Ezio V2 e Nephilim (voce 026, sostituisce la configurazione della voce 024)

- Pawn `BP_CombatUnit_Ezio`, figlio di `BP_CombatUnit` e pawn predefinito di `BP_CombatGameMode`. Mesh `SK_Ezio` (alta circa 1,1 m, scala 1, mesh a −98 cm nel capsule) e `ABP_Ezio`; capsule, camera, collisione e logica dei turni sono quelle del genitore.
- `SK_Ezio` e tutte le sue clip provengono esclusivamente da `SourceArt/Characters/Ezio/v2/`. Le 67 ossa hanno gli stessi nomi e genitori della V1; lo Skeleton esistente è stato aggiornato alla posa V2, verificata per braccia, avambracci e mani.
- `ABP_Ezio`: `Speed` = velocità orizzontale effettiva del pawn → `BS_Ezio_Locomotion` (BlendSpace 1D: Idle 0, Walk 75, Run 290 cm/s). La Run V2 gira a 1,09x; la velocità di gameplay resta 290 cm/s.
- Nephilim usa `BP_CombatUnit_Nephilim`, figlio della stessa classe base: `SK_Nephilim`, Skeleton e `ABP_Nephilim` dedicati, altezza circa 1,9 m, scala 1, offset mesh −98 cm e yaw −90°. Solo il suo spawn nel controller usa la nuova classe; le guardie sono invariate.
- `ABP_Nephilim` usa la velocità orizzontale e `BS_Nephilim_Locomotion`: Idle 0, Walk 167, Run 600 cm/s, clip a 1x. `MoveSpeed` resta 600 cm/s. Per entrambi il budget della scheda resta 9 m; Walk compare durante accelerazione/frenata, Run a velocità piena.
- Attacco: `A_Ezio_Axe_Spin_Attack` a 2x (circa 1,25 s). Reazione: `A_Ezio_HitReaction`, aggiunta nella voce 029, assegnata a `HitAnim`.
- Nephilim usa `A_Nephilim_Attack` a 2,24x (circa 1,25 s), entro il blocco di feedback esistente, e `A_Nephilim_HitReaction` per la reazione al colpo.
- Le due nuove reazioni provengono da `Face_Punch_Reaction_1`, ciascuna sullo Skeleton del relativo personaggio. Importati i frame 60–135 (2,5 s), eliminando l'attesa iniziale di circa 2 s; `HitPlayRate=2,5` dà una reazione di 1 s, con ritorno all'Idle prima del termine del feedback del combattimento. Root motion disattivato e radice bloccata: la capsula non viene spostata dalla clip. Le texture dei personaggi restano quelle già in uso.
- Mancano ancora clip Death per entrambi. Su un colpo letale viene riprodotta la reazione e conservata l'ultima posa: non c'è ancora un'animazione di caduta. I corpi restano visibili e attraversabili, esclusi dai turni.

A 0 HP il corpo rimane visibile nell'ultima posa di caduta, senza collisione, movimento o ostacolo di navigazione. Il personaggio può passare attraverso la sua posizione. Questo sostituisce il nascondimento usato nella prima versione.

Le copie locali delle animazioni hanno root motion disattivato e radice bloccata per non spostare gli attori durante il colpo. Le animazioni originali del template non sono state modificate.

## Arena

Sette attori nella cartella Outliner `CombatPrototype/Arena`:

- Muro centrale di 1 × 7 metri, alto 1,8 metri.
- Quattro muri perimetrali bassi.
- Due blocchi laterali.

Interno di circa 21 × 16 metri e due passaggi a nord e sud del muro, larghi circa 4,6 metri. Sono ostacoli di movimento; non implementano una meccanica di copertura. Con 9 metri e portata di 1,5 m, il primo attacco attorno al muro costa circa 7,3–7,5 metri: già dal primo turno sia gli alleati sia le guardie possono colpire. L'utente ha scelto di mantenere l'arena invariata.

## Asset e responsabilità

Tutti gli asset specifici sono sotto `/Game/CombatPrototype/`.

| Asset | Responsabilità |
| --- | --- |
| `Lvl_CombatPrototype` | Arena, illuminazione, navigazione dinamica e override del GameMode. |
| `BP_CombatGameMode` | Selezione di controller, personaggio e HUD del prototipo. |
| `BP_CombatController` | Inizializzazione, caricamento delle schede, iniziativa e turni, reazioni e attacchi di opportunità, budget, navigazione, pianificazione degli attacchi, tiri, IA, danno e conclusione dello scontro. |
| `BP_CombatUnit` | Manichino e camera del template, impostazioni di collisione/navigazione, animazioni e stato del feedback; configurazione per unità `MoveSpeed`, `AttackAnim`, `AttackPlayRate`, `HitAnim`, `HitPlayRate`, `DeathAnim`. |
| `BP_CombatUnit_Ezio` | Configurazione di Ezio: mesh, Animation Blueprint, 290 cm/s, attacco. |
| `BP_CombatUnit_Nephilim` | Configurazione di Nephilim: mesh e Skeleton propri, Animation Blueprint, 600 cm/s, attacco. |
| `BP_CombatHUD` | Interfaccia Canvas, percorso, probabilità, riquadro del tiro, barre HP, testo flottante e Fine turno. |
| `Data/S_CombatSheet` | Struttura della scheda di combattimento. |
| `Data/DT_CombatSheets` | Schede di Ezio, Nephilim e Guardia. |
| `Animations/AN_CombatAttack` | Copia locale di `MM_Attack_01`. |
| `Animations/AN_CombatHit` | Copia locale di `MM_HitReact_Front_Lgt_01`. |
| `Animations/AN_CombatDeath` | Copia locale di `MM_Death_Front_01`. |

Asset di Ezio in `/Game/Characters/Ezio/` (sorgenti esclusivamente in `SourceArt/Characters/Ezio/v2/`, compresa la nuova sottocartella `HitReaction/`): `SK_Ezio`, `SK_Ezio_Skeleton`, `SK_Ezio_PhysicsAsset`, `M_Ezio` (PBR: base color, normal, roughness e metallic come Masks), `T_Ezio_*`, `ABP_Ezio`, `Animations/BS_Ezio_Locomotion` e 12 clip `Animations/A_Ezio_*`. I percorsi principali sono stati riusati; le 9 clip esclusivamente V1 non referenziate sono state eliminate nell'editor.

Nephilim ha la stessa struttura in `/Game/Characters/Nephilim/`: `SK_Nephilim`, `SK_Nephilim_Skeleton`, `SK_Nephilim_PhysicsAsset`, `M_Nephilim`, quattro `T_Nephilim_*`, `ABP_Nephilim`, `Animations/BS_Nephilim_Locomotion` e 15 `Animations/A_Nephilim_*`. Sorgenti in `SourceArt/Characters/Nephilim/`, compresa `HitReaction/`. Le clip aggiuntive sono importate ma non introducono nuove azioni di gameplay.

Le regole sono nei grafi Blueprint. Python è stato usato solo per authoring, ispezione e prove nell'editor tramite Unreal MCP.

## Verifiche già eseguite

Le verifiche sono documentate con il contesto della rispettiva versione nel changelog; non sono una suite automatica rieseguita a ogni apertura del progetto.

- Budget, arresto a zero metri, clic e passaggio dei turni.
- Assenza di movimento fuori turno e ritorno all'animazione di riposo.
- Pianificazione attorno a muri e unità ferme, punto libero in portata, limite degli attacchi per azione e portata di mischia (1,5 m fra i centri dalla voce 015).
- Tiri (voce 017): soglia della CA, critico, mancato, doppio attacco, Attacco furtivo con e senza alleato vicino, probabilità nell'anteprima.
- Attacco e reazione senza spostare le coordinate degli attori; blocco temporaneo dei comandi e ritorno all'Animation Blueprint.
- Corpi visibili e attraversabili, esclusi dalla navigazione.
- Scontro completo con tiri casuali: vittoria al round 5 con HP `[0,29,0,0]` (voce 017). Le prove con 100 HP fissi delle voci 011–015 appartengono alla versione precedente.
- Iniziativa, rotazione per `TurnOrder`, round, attacchi di opportunità di guardie e alleati, ripresa del movimento e dell'attacco in attesa, morte durante il movimento (voce 018).
- Scatto oltre i 9 m, Disimpegno senza attacchi di opportunità, avviso dell'anteprima sul passaggio accanto a un nemico (voce 019).
- Azione scaltra (doppio Scatto, Disimpegno bonus più attacco), Recuperare energie, azione consumata dal primo attacco (voce 020).
- Nascondersi dietro il muro, vantaggio con furtivo, rivelazione all'attacco e da fermi in vista, guardie che ignorano i nascosti (voce 021).
- Cono di vista: davanti, dietro e di lato alle guardie (50° visto, 70° non visto), cono rosso con Nascondersi disattivato, round completo delle guardie (voce 022).
- Compilazione dei Blueprint modificati con avvisi trattati come errori e controllo dei log runtime delle prove.

Per nuove modifiche seguire `AGENTS.md` e `CONSTITUTION.md`, aggiornando `CHANGELOG.md` con l'autore effettivo.

## Versionamento

La cartella è una repository Git sul branch `main`, con remoto `origin` su `https://github.com/PregioSky/RalyaLandsGame.git`.

Gli asset `.uasset` e `.umap` sono gestiti con Git LFS. Su un altro computer serve Git LFS installato per scaricare gli asset effettivi durante il clone. `.gitignore` esclude cache, cartelle generate e impostazioni locali; `Content`, `Config`, il `.uproject` e i documenti condivisi vengono versionati.

Le modifiche al gameplay restano soggette alle regole di verifica e aggiornamento del changelog. Per controllare il lavoro locale usare `git status`; i confronti visuali dei Blueprint richiedono gli strumenti dell'editor, poiché gli asset sono binari.

