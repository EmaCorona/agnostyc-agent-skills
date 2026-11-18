# Persistenza, snapshot e recupero

Leggi prima della 1ª mutazione e per ripresa, rollback, recupero $N \to N+1$ o invalidazione. Precondizioni operative, non garanzie del solo prompt.

## Identità, percorsi e proprietà

Con scritture autorizzate usa struttura disgiunta in `<STATE-DIR>` per separare metadati e copie sorgenti:

- `<STATE-DIR>/state.md`: registro principale (`state.md.tmp` atomico, `state.prev.md` ultimo stato valido verificato).
- `<STATE-DIR>/backups/baseline/files/` e `manifest.md`: stato pre-modifica esecuzione.
- `<STATE-DIR>/backups/tasks/<TASK-ID>/<GEN>/gen-base/` e `manifest.md`: input prima di `<GEN>`.
- `<STATE-DIR>/backups/tasks/<TASK-ID>/<GEN>/attempts/<ATTEMPT-ID>/base/`, `discarded/` e `manifest.md`: stato pre-tentativo (`base/`), modifiche scartate pre-rollback (`discarded/`) e manifest.
- `<STATE-DIR>/backups/integrations/<INT-ID>/base/`, `discarded/` e `manifest.md`: snapshot, scartati e manifest di ogni `INT-ID`.

Registra root canonica, `<RUN-ID>`, repo/revisione, proprietario e handle runtime. Un solo orchestratore scrive stato; per esecuzioni concorrenti usa workspace isolati o lock. Rifiuta symlink o collisioni estranee; crea dir privata (`0700` se supportato). Se test/linter/bundler scansionano root senza escludere `.orchestration/`, usa dir temporanea privata esterna alla root registrandola in chat.

Aggiornamento `state.md`:
- Rinomina atomica stesso filesystem: scrivi `<STATE-DIR>/state.md.tmp`, verifica e sostituisci `<STATE-DIR>/state.md` (`mv`).
- Solo `Write`/`Edit` nativi senza shell: salva prima ultimo stato valido in `<STATE-DIR>/state.prev.md`, aggiorna `<STATE-DIR>/state.md` e rileggi subito coda anti-troncamento.
- Se il salvataggio fallisce: ferma nuovi avvii, richiedi arresto scrittori attivi, attendi conferma, conserva handle ed evidenze in chat e riconcilia prima di riprendere.

## Snapshot e manifest

Prima della 1ª scrittura, e per nuovi percorsi autorizzati prima di tentativi successivi:

1. Registra per percorso operazione (`crea`, `modifica`, `elimina`, `rinomina` origine/destinazione), inclusi output/cache controlli. File ignorati da Git non sono sacrificabili.
2. Verifica percorsi vs root: rifiuta `..` e destinazioni esterne. Controlla alias, hard link modificabili e symlink nelle dir antenate; non seguire symlink in copie/ripristini (tratta symlink come oggetto, non target). Per hard link isola o verifica tutti gli alias prima di autorizzare.
3. Registra in `manifest.md`: presenza/assenza (anche symlink spezzati, es. `test -L`; assenza esplicita `assente`), tipo, hash o target link, permessi e copia in `files/`, `gen-base/` o `base/`. Per dir registra solo quelle create dalla sessione e contenuto attribuibile.
4. `backups/baseline/files/` conserva stato pre-modifica (per file preesistenti aggiunti dopo, acquisisci baseline prima della 1ª mutazione annotando ingresso). `gen-base/` conserva input prima di `<GEN>`. `attempts/<ATTEMPT-ID>/base/` conserva stato pre-tentativo (`iniziale` = `gen-base/`). Mai sovrascrivere snapshot precedenti.
5. Usa copie verificate byte/hash e metadati necessari; mai hard link come backup di modificabili. Se symlink o metadati non preservabili, isola task o blocca mutazione.
6. Conferma input invariati durante snapshot. Salva manifest e intenzione in `state.md` prima della delega (se incompleti, non avviare). Se perimetro, requisiti o input cambiano esternamente durante un tentativo, arresta scrittore e aggiorna autorizzazioni, snapshot e brief prima di proseguire.

A fine tentativo registra inventario e impronte finali confrontando prima/dopo aree interessate, inclusi ignorati nei perimetri ed esclusi `.git/`, `<STATE-DIR>` (`git status --porcelain --ignored -- ':!.orchestration'`, `find . -path './.orchestration' -prune -o ...`) e dipendenze massive fuori perimetro (`node_modules/`, `.venv/`, `target/`, verificando integrità top-level), o usa filesystem isolato; documenta limiti osservabilità. Non copiare interi repo, dipendenze o segreti.

## Sequenza recupero $N \to N+1$, subentro e `SBLOCCO CIRCOSCRITTO`

- **Sequenza $N \to N+1$**: 1) Diagnostica causa e aggiorna brief (mai chiamate identiche); 2) Rollback selettivo dei soli file difettosi di $N$ (salva in `attempts/<N>/discarded/`, ripristina da `attempts/<N>/base/`), preservando lavoro parziale valido; 3) Per ogni nuovo file preesistente entrato nel perimetro, acquisisci prima `baseline/files/`; 4) Fotografa stato risultante in `attempts/<N+1>/base/` e `manifest.md` prima di avviare $N+1$.
- **Subentro** (se subagente precedente non riattivabile per handle perso/ripresa non supportata, accertato/registrato, arresto confermato o nessuna scrittura in corso): avvia nuovo subagente di subentro (configurazione target/default allo spawn) dedicato allo stesso `TASK-ID` — nuovo unico titolare per recuperi successivi — compilando nel brief tutte le sezioni di [incarico subagente](../templates/subagent-brief.md) (`Solo per subentro`: motivo, stato attuale su disco vs base, lavoro valido da conservare, difetti aperti, cosa resta, senza codice risolutivo). Non azzera il contatore (consuma il prossimo recupero). L'orchestratore non sostituisce il subagente perso; se delega non più avviabile, `BLOCCATO`.
- **Reinvio tecnico pre-esecuzione**: per errori certamente pre-esecuzione (es. rifiuto immediato `run_in_background: true` per permessi non pre-autorizzati) è ammesso 1 solo reinvio tecnico registrato; se avvio incerto, riconcilia prima.
- **Procedura `SBLOCCO CIRCOSCRITTO` (`INT-ID`)**: dopo esaurimento `iniziale` + 5 recuperi (`INCOMPLETO`), esegui snapshot in `backups/integrations/<INT-ID>/base/` e `manifest.md`, applica il solo fix limitato ai difetti residui verificati ed esegui verifiche dirette. Pass $\rightarrow$ `VERIFICATO`; fail $\rightarrow$ salva in `discarded/`, esegui rollback selettivo delle sole modifiche dell'`INT-ID` (o conservazione se valide e documentate), lascia task `INCOMPLETO` senza altri tentativi e riporta difetti residui, specifiche violate, evidenze e condizione per riprendere.

## Ripresa e avvii incerti

Alla ripresa verifica `state.md`; se mancante/troncato, riconcilia da `state.prev.md` o `state.md.tmp` e filesystem. Riconcilia poi col runtime prima di cambiare stati:

| Osservazione | Azione |
| --- | --- |
| Agente ancora attivo | Attendi o richiedi arresto; nessun ripristino o duplicazione. |
| Consegna disponibile | Verifica `TASK-ID`/`GEN`/`ATTEMPT-ID`; passa a `IN VERIFICA`. |
| Arresto confermato senza consegna | Ispeziona artefatti parziali e valuta recupero col contatore registrato (`IN VERIFICA` $\rightarrow$ `DA CORREGGERE` / `INCOMPLETO`). |
| Stato runtime sconosciuto | Mantieni handle, marca `BLOCCATO` (stato runtime `sconosciuto`); nessun nuovo scrittore sulle risorse. |
| Intenzione salvata senza handle | Cerca avvio nel runtime e ispeziona perimetro; non presumere mancato avvio. |

Non riprendere vecchie esecuzioni senza verificare richiesta e progetto. Non rimuovere lock altrui senza prova cessazione.

## Rollback condizionale

Arresta prima tutti gli scrittori coinvolti attendendo conferma. Salva modifiche da annullare in `attempts/<ATTEMPT-ID>/discarded/` (o `backups/integrations/<INT-ID>/discarded/`) e registra diagnosi. Confronta file corrente con ultima impronta attribuita al tentativo/task: se coincide senza scrittori concorrenti, ripristina `attempts/<ATTEMPT-ID>/base/` (`gen-base/` solo se annulli intera generazione senza conservare lavoro parziale valido). Se manca impronta o file cambiato esternamente, non ripristinare alla cieca: confronto a tre vie (`base` vs `output del task` vs `stato corrente`) o sospendi in `BLOCCATO`.

- Ripristina solo oggetti del manifest e modifiche attribuibili, preservando lavoro utente, indice Git e risultati validi successivi.
- Rimuovi file nuovo solo se `assente` alla base, creato dal task e identico all'impronta finale attribuita.
- Per eliminazioni verifica percorso ancora assente prima di ricrearlo. Per rinomine tratta origine e destinazione come oggetti distinti, preservando modifiche successive.
- Elimina dir solo se create dalla sessione e ora vuote. Vietati: cancellazioni ricorsive indiscriminate, `git clean`, `reset --hard`, checkout alberi o ripristino globale baseline.
- Con dipendenze, annulla in ordine inverso solo output obsoleti; se file contiene anche lavoro valido successivo, usa modifica selettiva (`INT-ID`) o registra blocco.

Verifica stato finale e registra per percorso: `ripristinato`, `conservato` o `in conflitto`. `INVALIDATO` è esclusione logica, non cancellazione fisica.

## Chiusura

Salva report/evidenze fuori dai temporanei prima della pulizia. Verifica tutti agenti `fermo confermato`, proprietà `<STATE-DIR>` e assenza file estranei. Rimuovi solo temporanei attribuibili alla sessione; conserva snapshot e `state.md` se servono per conflitti, blocchi, tracciabilità o ripresa. Non modificare esclusioni Git utente.
