# Istruzioni di progetto — RalyaLands

Queste istruzioni valgono per gli agenti che lavorano in questa cartella, inclusi Codex e Claude Code. Rispettare sempre le istruzioni di sistema e le richieste esplicite dell'utente.

## Prima di lavorare

1. Leggere [CONSTITUTION.md](CONSTITUTION.md), fonte unica delle regole condivise.
2. Leggere [CombatPrototype.md](CombatPrototype.md) per lo stato attuale del prototipo.
3. Leggere le ultime voci di [CHANGELOG.md](CHANGELOG.md), e le voci precedenti pertinenti al lavoro richiesto.
4. Verificare eventuali istruzioni più specifiche nelle sottocartelle interessate e lo stato reale dei file e dell'editor. Non presumere che il progetto sia un repository Git.

## Regola obbligatoria del changelog

Ogni intervento che modifica asset, codice, configurazione, istruzioni o documentazione deve aggiornare `CHANGELOG.md` prima della risposta conclusiva. Vale anche per le modifiche ai soli documenti.

- Se l'intervento è eseguito da Codex, scrivere `Autore: Codex`.
- Se è eseguito da Claude Code, scrivere `Autore: Claude Code`.
- Non attribuire modifiche all'altro agente senza evidenza. Per un intervento condiviso, precisare il contributo di ciascuno.
- Descrivere risultato, motivo delle scelte, asset/file interessati e verifiche realmente eseguite. Segnalare esplicitamente ciò che non è stato verificato.
- Conservare la storia delle soluzioni sostituite; aggiungere una nuova voce che indichi cosa le sostituisce.

Il formato completo e il flusso di lavoro sono in `CONSTITUTION.md`. La guida corrente va aggiornata se cambia il comportamento del prototipo.

## Punto di ingresso

- Progetto: `RalyaLands.uproject`.
- Mappa del prototipo: `/Game/CombatPrototype/Lvl_CombatPrototype`.
- Asset del prototipo: `/Game/CombatPrototype/`.
- Per modificare o verificare gli asset Unreal, usare Unreal MCP quando disponibile.

Non confondere lo storico del changelog con le regole di gioco attuali: portata di 2 metri, corpi nascosti e aggiramento RVO appartengono a versioni precedenti.
