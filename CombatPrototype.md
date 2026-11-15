# Combat Prototype — stato attuale

Questa guida descrive il comportamento corrente. La storia delle scelte, delle prove e delle soluzioni sostituite è in [CHANGELOG.md](CHANGELOG.md). Le regole di lavoro condivise sono in [CONSTITUTION.md](CONSTITUTION.md).

## Avvio e controlli

Aprire `/Game/CombatPrototype/Lvl_CombatPrototype` e premere **Play**. La mappa iniziale globale del progetto resta `Lvl_TopDown`: il prototipo va aperto esplicitamente.

- **Clic sinistro sul terreno:** muove l'unità attiva lungo la navigazione, entro il movimento residuo.
- **Clic sinistro su una guardia:** sceglie un punto adiacente libero, avvicina l'unità e attacca appena entra in portata. Se il budget non basta, si ferma senza attaccare.
- **Altro clic sul terreno:** annulla l'attacco in attesa e impartisce un nuovo ordine.
- **Fine turno** oppure **Spazio:** passa alla prossima unità viva.
- **Riavviare PIE:** ricomincia lo scontro.

Durante l'animazione di attacco i comandi sono brevemente bloccati e l'HUD lo segnala.

## Regole dello scontro

| Regola | Valore attuale |
| --- | --- |
| Squadra del giocatore | Ezio e Nephilim |
| Avversari | Guardia 1 e Guardia 2, automatiche |
| Ordine | Ezio → Nephilim → Guardia 1 → Guardia 2 |
| Movimento per turno | 6 metri effettivamente percorsi in piano |
| Attacchi per turno | Uno; è possibile muoversi ancora dopo il colpo |
| Corpo a corpo | 1 metro fra i centri, con tolleranza tecnica di 0,5 cm |
| HP iniziali | 100 per unità |
| Danno alleato / guardia | 40 / 25 |
| Impatto | Dopo 0,22 secondi dall'inizio dell'attacco |
| Blocco dei comandi per il feedback | 1,35 secondi |
| Fine dello scontro | Una squadra interamente a 0 HP |

Il “quadretto adiacente” è rappresentato dalla distanza di corpo a corpo. Il movimento è libero sul NavMesh: non esistono ancora una griglia con aggancio alle celle, coperture con bonus difensivi o abilità aggiuntive.

Le unità sconfitte vengono saltate nei turni. Vittoria e sconfitta bloccano nuovi comandi di gioco.

## Movimento, unità ferme e attacco

- Il controller usa il movimento e la camera del template TopDown. La camera segue l'unità posseduta, con distanza aumentata a 2.200 cm.
- Muri e capsule delle unità ferme partecipano al NavMesh. Le capsule ferme usano `NavArea_Null`; l'unità attiva è esclusa dal proprio ostacolo.
- Al cambio turno si attende almeno 0,15 secondi e il completamento dell'aggiornamento della navigazione.
- RVO è **disattivato**: l'aggiramento dei personaggi è pianificato prima del movimento, invece di essere aggiunto da un evitamento locale.
- `FindAttackPlan` valuta 16 punti a 98 cm dal bersaglio, scarta quelli occupati o non raggiungibili e sceglie il percorso più corto fra i candidati validi.
- L'ordine mantiene il punto scelto. L'avvicinamento finale è rifinito con il movimento del template per compensare la tolleranza di arrivo della navigazione.
- Se non esiste un punto adiacente libero, l'attacco non parte. Un clic su un nemico dopo aver già usato l'attacco non ne avvia un secondo.
- Il budget usa la distanza realmente percorsa: anche le deviazioni consumano metri. Al limite il movimento viene fermato e corretto.
- Dopo 0,8 secondi senza almeno 2 cm di progresso si interrompe l'ordine: la guardia passa il turno, mentre il giocatore può scegliere una nuova destinazione. Resta anche un limite temporale di sicurezza per il turno automatico.

Le guardie scelgono fra gli alleati vivi in base al costo di un piano d'attacco raggiungibile, si avvicinano, colpiscono e terminano il turno dopo il feedback.

## Anteprima e HUD

Passare il cursore sul terreno o sul nemico per vedere il percorso prima del clic.

- **Verde:** tratto entro il budget.
- **Rosso:** tratto oltre il budget.
- **Barra rossa sul nemico:** solo se il movimento non basta; nessuna barra quando il bersaglio è raggiungibile per attaccare.
- **Riquadro superiore:** costo e residuo stimati in metri, con un decimale, e stato dell'attacco.
- **Pannello inferiore:** unità attiva, movimento, attacco disponibile, HP e Fine turno.
- **Presso le unità:** nome, indicatore dell'unità attiva, barra HP e valore numerico; colori distinti per squadra.
- **Al colpo:** numero del danno flottante e reazione animata.

L'anteprima usa la stessa pianificazione dell'ordine ed è aggiornata circa 10 volte al secondo. Le stime sono indicate con `~`: tolleranze del percorso, curve di movimento e tick possono produrre piccoli scostamenti dal consumo effettivo. L'anteprima è nascosta sull'HUD, durante il feedback, nei turni delle guardie e a scontro concluso.

## Animazioni e sconfitta

L'attaccante si volta verso il bersaglio e usa un attacco disarmato del template. Al momento dell'impatto vengono applicati danno, reazione e numero flottante. Le unità vive tornano a `ABP_Unarmed` al termine dell'animazione.

A 0 HP il corpo rimane visibile nell'ultima posa di caduta, senza collisione, movimento o ostacolo di navigazione. Il personaggio può passare attraverso la sua posizione. Questo sostituisce il nascondimento usato nella prima versione.

Le copie locali delle animazioni hanno root motion disattivato e radice bloccata per non spostare gli attori durante il colpo. Le animazioni originali del template non sono state modificate.

## Arena

Sette attori nella cartella Outliner `CombatPrototype/Arena`:

- Muro centrale di 1 × 7 metri, alto 1,8 metri.
- Quattro muri perimetrali bassi.
- Due blocchi laterali.

Interno di circa 21 × 16 metri e due passaggi a nord e sud del muro, larghi circa 4,6 metri. Sono ostacoli di movimento; non implementano una meccanica di copertura. La configurazione iniziale richiede circa 7,8 metri per raggiungere il primo attacco attorno al muro, quindi il primo turno serve a posizionarsi.

## Asset e responsabilità

Tutti gli asset specifici sono sotto `/Game/CombatPrototype/`.

| Asset | Responsabilità |
| --- | --- |
| `Lvl_CombatPrototype` | Arena, illuminazione, navigazione dinamica e override del GameMode. |
| `BP_CombatGameMode` | Selezione di controller, personaggio e HUD del prototipo. |
| `BP_CombatController` | Inizializzazione, turni, budget, navigazione, pianificazione degli attacchi, IA, danno e conclusione dello scontro. |
| `BP_CombatUnit` | Manichino e camera del template, impostazioni di collisione/navigazione, animazioni e stato del feedback. |
| `BP_CombatHUD` | Interfaccia Canvas, percorso, barre HP, danno flottante e Fine turno. |
| `Animations/AN_CombatAttack` | Copia locale di `MM_Attack_01`. |
| `Animations/AN_CombatHit` | Copia locale di `MM_HitReact_Front_Lgt_01`. |
| `Animations/AN_CombatDeath` | Copia locale di `MM_Death_Front_01`. |

Le regole sono nei grafi Blueprint. Python è stato usato solo per authoring, ispezione e prove nell'editor tramite Unreal MCP.

## Verifiche già eseguite

Le verifiche sono documentate con il contesto della rispettiva versione nel changelog; non sono una suite automatica rieseguita a ogni apertura del progetto.

- Budget, arresto a zero metri, clic e passaggio dei turni.
- Assenza di movimento fuori turno e ritorno all'animazione di riposo.
- Pianificazione attorno a muri e unità ferme, punto adiacente libero, attacco singolo e portata di mischia.
- Attacco e reazione senza spostare le coordinate degli attori; blocco temporaneo dei comandi e ritorno all'Animation Blueprint.
- Corpi visibili e attraversabili, esclusi dalla navigazione.
- Scontro completo con i feedback attuali: vittoria con HP `[25,25,0,0]`.
- Partita senza attacchi del giocatore: sconfitta con HP `[0,0,100,100]`.
- Compilazione dei Blueprint modificati con avvisi trattati come errori e controllo dei log runtime delle prove.

Per nuove modifiche seguire `AGENTS.md` e `CONSTITUTION.md`, aggiornando `CHANGELOG.md` con l'autore effettivo.

## Versionamento

La cartella è una repository Git sul branch `main`, con remoto `origin` su `https://github.com/PregioSky/RalyaLandsGame.git`.

Gli asset `.uasset` e `.umap` sono gestiti con Git LFS. Su un altro computer serve Git LFS installato per scaricare gli asset effettivi durante il clone. `.gitignore` esclude cache, cartelle generate e impostazioni locali; `Content`, `Config`, il `.uproject` e i documenti condivisi vengono versionati.

Le modifiche al gameplay restano soggette alle regole di verifica e aggiornamento del changelog. Per controllare il lavoro locale usare `git status`; i confronti visuali dei Blueprint richiedono gli strumenti dell'editor, poiché gli asset sono binari.

