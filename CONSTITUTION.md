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
- Camera e base del movimento al clic provengono dal template TopDown. Ezio V2 e Nephilim usano i propri modelli Meshy (`BP_CombatUnit_Ezio` e `BP_CombatUnit_Nephilim`, asset in `/Game/Characters/Ezio/` e `/Game/Characters/Nephilim/`); le guardie restano manichini. Per Ezio usare esclusivamente le sorgenti in `SourceArt/Characters/Ezio/v2/`; le sorgenti V1 sono state cancellate su richiesta dell'utente (voce 051) e restano solo nella cronologia git. La velocità visiva di spostamento può differire per unità (Ezio 290 cm/s, Nephilim 600 cm/s), senza cambiare il budget in metri.
- Due alleati controllati dal giocatore, Ezio e Nephilim, contro due guardie automatiche.
- Gli oggetti riutilizzabili ed equipaggiabili hanno una gerarchia propria in `/Game/Items/`, allo stesso livello di `Characters`: definizioni comuni in `Items/Core`, contenuti per categoria (per esempio `Items/Weapons/Crossbows/HandCrossbow` e `Items/Weapons/Swords` con mesh, materiali e texture). Sorgenti in `SourceArt/Items/`. Non incorporare armi nelle mesh dei personaggi. Slot, equipaggiamento e rimozione sono esposti in Blueprint; socket e pose dipendenti dal rig restano nella configurazione dell'unità. Slot da mischia e a distanza: stocco di Ezio, spada lunga di Nephilim e balestra di Ezio (voci 037–038). La modalità di attacco alterna i modelli visibili; l'arma scelta resta in mano durante i turni altrui e anche all'inizio del proprio nuovo turno, per scelta dell'utente (voce 040, sostituisce il ripristino della mischia della voce 039); la morte nasconde le armi. Le guardie (`BP_CombatUnit_Guard`) portano una balestra pesante e tornano alla mischia all'inizio del proprio turno (voce 043).
- La camera segue l'unità attiva. Zoom morbido con rotellina fra 4,5 e 22 m, mantenuto al cambio turno; avvicinandosi l'inclinazione si abbassa da −50° a −25°, scelta confermata dall'utente nella voce 025. L'inquadratura ravvicinata deve mantenere Ezio visibile sopra l'HUD.
- Rotazione orizzontale della camera con tasto centrale premuto e trascinamento del mouse. Orientamento condiviso e conservato al cambio turno; disponibile anche durante i turni delle guardie. L'inclinazione resta controllata dallo zoom. Il trascinamento non impartisce ordini di movimento o attacco (voce 027).
- Ordine dei turni per iniziativa D&D 5e (d20 + bonus della scheda) tirata a inizio scontro; saltare le unità sconfitte. Ogni unità ha una reazione per round, usata per l'attacco di opportunità quando un nemico esce volontariamente dalla sua portata di 1,5 m. Impugnare l'arma a distanza impedisce l'attacco di opportunità, senza consumare la reazione; non si cambia automaticamente arma per reagire (voce 039). I manichini conservano la loro configurazione di attacco in mischia anche senza un modello di arma.
- Ogni turno concede il movimento della scheda (Ezio 7,5 metri da halfling, Nephilim e guardie 9 metri; voce 033) e un'azione: Attacco (un attacco, o più attacchi con Attacco extra, anche intervallati dal movimento), Scatto (movimento raddoppiato) oppure Disimpegno (nessun attacco di opportunità per il resto del turno); inoltre un'azione bonus, se l'unità ne ha una (Azione scaltra di Ezio, Recuperare energie di Nephilim). Nephilim ha Azione impetuosa (un'azione in più, una volta per scontro, dopo aver speso la prima); Ezio ha Schivata prodigiosa (reazione automatica che dimezza il danno di un colpo da un attaccante visibile). Elusione è di livello 7 ed è esclusa (voce 042). Spinta (al posto di un attacco, Atletica contrapposta, 1,5 m) e Salto (azione bonus come in BG3, gittata dalla Forza, pagato in movimento, senza attacchi di opportunità) dalla voce 048. Come in Baldur's Gate 3, per scelta dell'utente (voce 046): gli alleati a 0 HP vanno a terra e fanno i tiri contro la morte, Aiutare (azione, entro 1,5 m) li rialza con 1 HP, Schivare non esiste; le guardie a 0 HP muoiono. È consentito muoversi dopo aver attaccato, se resta budget.
- Riferimento di stile: Baldur's Gate 3. Movimento libero e naturale sul NavMesh, anche di pochi passi, misurato in metri; nessuna griglia né aggancio alle celle, scelta confermata dall'utente.
- Corpo a corpo a 1,5 metri fra i centri delle unità, come i quadretti adiacenti di D&D, con tolleranza tecnica di 0,5 cm. La misura fra i bordi è stata provata e scartata perché visivamente troppo distante.
- Clic sul nemico: pianificare un punto libero in portata, avvicinarsi entro il budget e attaccare. Il punto scelto deve rimanere stabile per quell'ordine.
- Le unità ferme devono essere ostacoli già nel NavMesh, come i muri. L'unità attiva non deve ostacolare se stessa. RVO è disattivato: non reintrodurre deviazioni locali che sostituiscano la pianificazione degli ostacoli.
- Anteprima e ordine effettivo devono usare lo stesso criterio di pianificazione. Il tratto oltre budget è rosso; sul nemico la barra rossa appare solo se il movimento non basta.
- Regole D&D 5e semplificate, con valori nelle schede `DT_CombatSheets`: Ezio halfling Ladro 5, Nephilim Guerriero 5 Maestro di battaglia (voce 049), guardie Veterano. Tiri salvezza con i bonus della scheda; condizioni prono (Attacco sbilanciante, Spinta a terra) e spaventato (Attacco minaccioso), con vantaggio e svantaggio come in 5e (voce 049). Tiro per colpire d20 + bonus contro la CA; 20 naturale critico (dadi raddoppiati), 1 naturale mancato; danno a dadi; Attacco extra e Attacco furtivo (una volta per turno, con vantaggio o con un alleato entro 1,5 m dal bersaglio). Nascondersi come in D&D: solo fuori dalla vista dei nemici, che vedono entro un cono orientato (regola della casa in stile Baldur's Gate 3, valori nella scheda) e senza ostacoli; prova di Furtività contro Percezione passiva; il nascosto non è bersagliabile e attacca con vantaggio, e viene rivelato attaccando o fermandosi in vista. Anteprima e HUD mostrano probabilità e tiri come in Baldur's Gate 3. Attacco a distanza per Ezio (balestra leggera, voce 034) e per le guardie (balestra pesante del Veterano, usata dall'IA, voce 043): da fermo, con linea di tiro, svantaggio oltre la gittata normale o con un nemico adiacente che vede il tiratore, un tiro per azione. Copertura come in 5e (voce 035): mezza +2 CA, tre quarti +5 CA, totale non bersagliabile a distanza; anche una creatura fra attaccante e bersaglio dà mezza copertura. L'azione Cercare non è prevista, per scelta dell'utente.
- Le guardie preferiscono bersagli e punti d'arrivo che non provocano attacchi di opportunità, poi valutano fragilità e distanza. Il rischio comprende il movimento effettivamente disponibile, incluso Scatto, e un margine prudenziale di 20 cm per gli approcci dell'IA. Se tutti gli approcci provocano, usano Disimpegno rinunciando agli attacchi; se l'azione è già spesa, terminano il turno invece di iniziare un percorso rischioso. Mantengono Scatto sui percorsi sicuri e Disimpegno per ripiegare quando sono gravemente ferite (voce 039, aggiorna 036). Se non possono attaccare in mischia in questo turno e nessun alleato è adiacente, tirano con la balestra al bersaglio con il miglior rapporto HP / probabilità, copertura inclusa (voce 043).
- I corpi sconfitti, e gli alleati a terra, restano visibili a terra, senza collisione o ostacolo di navigazione.
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
- Nei pin numerici rispettare il tipo effettivo: i default interi devono essere scritti come `280`, non `280.0`. Dopo correzioni ai default o alla tipizzazione dei nodi generati, salvare e ricaricare dall'editor gli asset interessati, poi ricompilarli: il successo nella sola sessione di authoring non verifica la persistenza su disco (voce 028).
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
