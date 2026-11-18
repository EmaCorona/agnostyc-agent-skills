# Persistenza, snapshot e recupero

Leggi prima della prima mutazione e per ripresa, rollback o invalidazione. Sono precondizioni operative, non garanzie del solo prompt.

## Identità, struttura dei percorsi e proprietà

Con scritture autorizzate usa una struttura disgiunta in `<STATE-DIR>` per evitare collisioni tra metadati e copie sorgenti:

- `<STATE-DIR>/state.md`: registro principale (`state.md.tmp` per scritture atomiche, `state.prev.md` copia dell'ultimo stato valido verificato).
- `<STATE-DIR>/backups/baseline/files/` e `manifest.md`: stato originale prima della prima modifica dell'esecuzione.
- `<STATE-DIR>/backups/tasks/<TASK-ID>/<GEN>/gen-base/` e `manifest.md`: input immediatamente precedenti alla generazione `<GEN>`.
- `<STATE-DIR>/backups/tasks/<TASK-ID>/<GEN>/attempts/<ATTEMPT-ID>/base/`, `discarded/` e `manifest.md`: stato prima del tentativo (`base/`), modifiche scartate salvate prima di rollback (`discarded/`) e manifest.
- `<STATE-DIR>/backups/integrations/<INT-ID>/base/`, `discarded/` e `manifest.md`: snapshot, scartati e manifest di ogni intervento/integrazione (`INT-ID`).

Registra root canonica, `<RUN-ID>`, repo/revisione, proprietario e handle runtime. Un solo orchestratore scrive lo stato; per esecuzioni concorrenti usa workspace isolati o un lock. Rifiuta percorsi con symlink o collisioni estranee; crea la directory privata (`0700` se supportato). Se test/linter/bundler scansionano la root senza escludere `.orchestration/`, usa una directory temporanea privata esterna alla root registrandone il percorso in chat.

Aggiornamento di `state.md`:
- Con rinomina atomica sullo stesso filesystem: scrivi `<STATE-DIR>/state.md.tmp`, verifica e sostituisci `<STATE-DIR>/state.md` (`mv`).
- Con soli `Write`/`Edit` nativi senza shell: salva prima l'ultimo stato valido in `<STATE-DIR>/state.prev.md`, aggiorna `<STATE-DIR>/state.md` e rileggine subito la coda contro troncamenti.

## Snapshot e manifest

Prima della prima scrittura, e per nuovi percorsi autorizzati prima di tentativi successivi:

1. Registra per percorso l'operazione (`crea`, `modifica`, `elimina`, `rinomina` con origine/destinazione), inclusi output/cache dei controlli. File ignorati da Git non sono automaticamente sacrificabili.
2. Verifica i percorsi rispetto alla root: rifiuta `..` e destinazioni esterne. Controlla alias, hard link dei file modificabili e symlink nelle directory antenate; non seguire symlink in copie/ripristini (tratta il symlink modificato come oggetto, non il target). Per hard link isola il lavoro o verifica tutti gli alias prima di autorizzare.
3. Registra in `manifest.md`: presenza/assenza (rilevando anche symlink spezzati, es. `test -L`; assenza esplicita `assente`), tipo, hash o target del link, permessi e percorso copia in `files/`, `gen-base/` o `base/`. Per directory registra solo quelle create dalla sessione e il loro contenuto attribuibile.
4. `backups/baseline/files/` conserva lo stato pre-modifica (per file preesistenti aggiunti dopo, acquisisci la baseline prima della prima mutazione annotando l'ingresso). `gen-base/` conserva gli input prima di `<GEN>`. `attempts/<ATTEMPT-ID>/base/` conserva lo stato prima del tentativo: per `iniziale` coincide con `gen-base/`; tra tentativo fallito $N$ e recupero $N+1$ esegui **prima** il rollback selettivo dei soli file difettosi di $N$ (salva in `attempts/<N>/discarded/`, ripristina da `attempts/<N>/base/`), conservando le modifiche valide, e **poi** fotografa `attempts/<N+1>/base/` prima di avviare $N+1$. Non sovrascrivere mai snapshot precedenti.
5. Usa copie verificate byte/hash e metadati necessari; mai hard link come backup di file modificabili. Se symlink o metadati non sono preservabili, isola il task o blocca la scrittura.
6. Conferma che gli input non siano cambiati durante lo snapshot. Salva manifest e intenzione in `state.md` prima della delega (se incompleti, non avviare). Se perimetro, requisiti o input cambiano esternamente durante un tentativo, arresta lo scrittore e aggiorna autorizzazioni, snapshot e brief prima di proseguire.

A fine tentativo registra inventario e impronte finali confrontando prima/dopo le aree interessate, inclusi ignorati nei perimetri ed esclusi `.git/`, `<STATE-DIR>` (`git status --porcelain --ignored -- ':!.orchestration'`, `find . -path './.orchestration' -prune -o ...`) e dipendenze massive fuori perimetro (`node_modules/`, `.venv/`, `target/`, verificandone l'integrità top-level), o usa un filesystem isolato; documenta limiti di osservabilità. Non copiare interi repo, dipendenze o segreti.

## Ripresa e avvii incerti

Alla ripresa verifica `state.md`; se mancante o troncato, riconcilia da `state.prev.md` o `state.md.tmp` e filesystem. Riconcilia poi il registro col runtime prima di cambiare stati:

| Osservazione | Azione |
| --- | --- |
| Agente ancora attivo | Attendi o richiedi arresto; nessun ripristino o duplicazione. |
| Consegna disponibile | Verifica `TASK-ID`/`GEN`/`ATTEMPT-ID`; passa a `IN VERIFICA`. |
| Arresto confermato senza consegna | Ispeziona artefatti parziali e valuta recupero col contatore registrato (`IN VERIFICA` $\rightarrow$ `DA CORREGGERE` / `INCOMPLETO`). |
| Stato runtime sconosciuto | Mantieni handle, marca `BLOCCATO` (stato runtime `sconosciuto`); nessun nuovo scrittore sulle risorse. |
| Intenzione salvata senza handle | Cerca l'avvio nel runtime e ispeziona il perimetro; non presumere mancato avvio. |

Non riprendere vecchie esecuzioni senza verificare corrispondenza di richiesta e progetto. Non rimuovere lock altrui senza prova di cessazione.

## Rollback condizionale

Arresta prima tutti gli scrittori coinvolti attendendo conferma. Salva le modifiche da annullare in `attempts/<ATTEMPT-ID>/discarded/` (o `backups/integrations/<INT-ID>/discarded/`) e registra la diagnosi. Confronta il file corrente con l'ultima impronta attribuita al tentativo/task: se coincide senza scrittori concorrenti, ripristina `attempts/<ATTEMPT-ID>/base/` (`gen-base/` solo se annulli l'intera generazione senza conservare lavoro parziale valido). Se manca l'impronta o il file è cambiato esternamente, non ripristinare alla cieca: confronto a tre vie (`base` vs `output del task` vs `stato corrente`) o sospendi in `BLOCCATO`.

- Ripristina solo oggetti del manifest e modifiche attribuibili, preservando lavoro utente, indice Git e risultati validi successivi.
- Rimuovi un file nuovo solo se `assente` alla base, creato dal task e ancora identico all'impronta finale attribuita.
- Per eliminazioni verifica che il percorso sia ancora assente prima di ricrearlo. Per rinomine tratta origine e destinazione come oggetti distinti, preservando modifiche successive.
- Elimina directory solo se create dalla sessione e ora vuote. Vietati: cancellazioni ricorsive indiscriminate, `git clean`, `reset --hard`, checkout di alberi o ripristino globale della baseline.
- Con dipendenze, annulla in ordine inverso solo gli output obsoleti; se un file contiene anche lavoro valido successivo, usa modifica selettiva (`INT-ID`) o registra blocco.

Verifica lo stato finale e registra per percorso: `ripristinato`, `conservato` o `in conflitto`. `INVALIDATO` è esclusione logica, non cancellazione fisica.

## Chiusura

Salva report ed evidenze fuori dai temporanei prima della pulizia. Verifica tutti gli agenti `fermo confermato`, proprietà di `<STATE-DIR>` e assenza di file estranei. Rimuovi solo temporanei attribuibili alla sessione; conserva snapshot e `state.md` se servono per conflitti, blocchi, tracciabilità o ripresa. Non modificare le esclusioni Git dell'utente.
