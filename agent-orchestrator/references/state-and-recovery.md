# Persistenza, snapshot e recupero

Leggi prima di ogni prima mutazione e quando occorrono ripresa, rollback o invalidazione. Queste sono precondizioni operative, non garanzie fornite dal solo prompt.

## Identità, struttura dei percorsi e proprietà

In sessione ordinaria (con scritture autorizzate), usa una struttura disgiunta dentro `<STATE-DIR>` per evitare collisioni fra metadati dell'orchestratore e copie dei file di progetto:

- `<STATE-DIR>/state.md`: registro principale dell'esecuzione (`state.md.tmp` per scritture atomiche, `state.prev.md` come copia dell'ultimo stato valido verificato).
- `<STATE-DIR>/backups/baseline/files/` e `manifest.md`: stato originale dei file prima della prima modifica dell'esecuzione.
- `<STATE-DIR>/backups/tasks/<TASK-ID>/<GEN>/gen-base/` e `manifest.md`: snapshot degli input immediatamente precedenti all'avvio della generazione `<GEN>` del task.
- `<STATE-DIR>/backups/tasks/<TASK-ID>/<GEN>/attempts/<ATTEMPT-ID>/base/`, `discarded/` e `manifest.md`: snapshot immediatamente precedente al singolo tentativo (`base/`), copia delle modifiche scartate salvate prima di un eventuale rollback (`discarded/`) e manifest del tentativo.
- `<STATE-DIR>/backups/integrations/<INT-ID>/base/`, `discarded/` e `manifest.md`: snapshot, copie scartate e manifest per ogni intervento o integrazione dell'orchestratore (`INT-ID`).

Registra root canonica, `<RUN-ID>`, repository/revisione, proprietario e handle runtime. Un solo orchestratore scrive lo stato; per esecuzioni concorrenti usa workspace isolati o un lock disponibile nell'ambiente. Rifiuta percorsi di stato con symlink o collisioni con materiale estraneo e crea la directory privata (`0700` ove supportato). Se i runner di test, linter o bundler scansionano l'intera root senza permettere l'esclusione di `.orchestration/`, colloca `<STATE-DIR>` in una directory temporanea privata esterna alla root e registra il percorso in chat.

Per aggiornare `state.md`:
- Con rinomina atomica nello stesso filesystem, scrivi `<STATE-DIR>/state.md.tmp`, verifica il contenuto e sostituisci `<STATE-DIR>/state.md` atomicamente (`mv`).
- Solo con `Write`/`Edit` nativi senza shell, salva prima l'ultimo stato valido in `<STATE-DIR>/state.prev.md`, aggiorna `<STATE-DIR>/state.md` e rileggine subito la coda contro troncamenti.

## Snapshot e manifest

Prima del primo tentativo di scrittura, e per i nuovi percorsi autorizzati prima di ogni tentativo successivo:

1. Registra per ogni percorso autorizzato l'operazione (`crea`, `modifica`, `elimina`, `rinomina` con origine e destinazione), inclusi gli output/cache generati dai controlli. Un file ignorato da Git non è automaticamente sacrificabile.
2. Controlla i percorsi rispetto alla root: rifiuta `..` e destinazioni esterne. Verifica alias del filesystem, hard link dei file modificabili e symlink nelle directory antenate; non seguire symlink durante copie o ripristini (se un symlink va modificato, tratta il link come oggetto, non il target). Per file con hard link isola il lavoro o verifica tutti gli alias prima di autorizzare la scrittura.
3. Memorizza nel `manifest.md` presenza/assenza (usando un controllo che rilevi anche symlink spezzati, es. `test -L`), tipo di oggetto, hash o target del link, permessi e percorso della copia in `files/`, `gen-base/` o `base/`. Per directory registra solo quelle create dalla sessione e il loro contenuto attribuibile. Rappresenta l'assenza iniziale esplicitamente (`assente`).
4. `backups/baseline/files/` conserva lo stato precedente alla prima modifica dell'esecuzione (per un nuovo percorso preesistente aggiunto in seguito, acquisisci la baseline prima della sua prima mutazione annotando quando è entrato). `gen-base/` conserva gli input immediatamente precedenti alla generazione `<GEN>`. `attempts/<ATTEMPT-ID>/base/` conserva lo stato immediatamente precedente al singolo tentativo: nel tentativo `iniziale` coincide con `gen-base/`; tra un tentativo fallito $N$ e il recupero $N+1$, esegui **prima** il rollback selettivo dei soli file difettosi di $N$ (salvando in `attempts/<N>/discarded/` e ripristinando da `attempts/<N>/base/`), mantenendo le modifiche parziali valide, e **poi** acquisisci `attempts/<N+1>/base/` sullo stato risultante prima di avviare $N+1$. Non sovrascrivere mai snapshot precedenti.
5. Usa copie verificate byte/hash e metadati necessari; mai hard link come backup di file modificabili. Se il metodo non preserva symlink o metadati richiesti, isola il task o blocca la scrittura.
6. Conferma che gli input non siano cambiati durante lo snapshot. Salva manifest e intenzione in `state.md` prima della delega (se incompleti, non avviare lo scrittore). Se perimetro, requisiti o input cambiano per cause esterne durante un tentativo, arresta lo scrittore e aggiorna autorizzazioni, snapshot e brief prima di continuare.

A fine tentativo registra inventario e impronte finali. Confronta l'inventario prima/dopo delle aree interessate includendo i file ignorati nei perimetri del task ed escludendo `.git/`, `<STATE-DIR>` (`git status --porcelain --ignored -- ':!.orchestration'`, `find . -path './.orchestration' -prune -o ...`) e dipendenze massive fuori perimetro (`node_modules/`, `.venv/`, `target/`, di cui verifichi l'integrità top-level), oppure usa un filesystem isolato; documenta ogni limite di osservabilità. Non copiare interi repository, dipendenze o segreti.

## Ripresa e avvii incerti

Alla ripresa verifica l'integrità di `state.md`; se mancante o troncato, riconcilia da `state.prev.md` o `state.md.tmp` e dall'ispezione del filesystem. Riconcilia poi il registro con il runtime prima di cambiare stati:

| Osservazione | Azione |
| --- | --- |
| Agente ancora attivo | Attendi o richiedi arresto; nessun ripristino o duplicazione. |
| Consegna disponibile | Verifica corrispondenza con `TASK-ID`/`GEN`/`ATTEMPT-ID`; passa a `IN VERIFICA`. |
| Arresto confermato senza consegna | Ispeziona artefatti parziali e valuta recupero col contatore già registrato (`IN VERIFICA` $\rightarrow$ `DA CORREGGERE` / `INCOMPLETO`). |
| Stato runtime sconosciuto | Mantieni l'handle, marca il task `BLOCCATO` (stato runtime `sconosciuto`); nessun nuovo scrittore sulle risorse interessate. |
| Intenzione salvata ma handle mancante | Cerca l'avvio nel runtime e ispeziona il perimetro; non presumere che non sia avvenuto. |

Non riprendere una vecchia esecuzione senza verificare che richiesta e progetto corrispondano. Non rimuovere lock altrui senza prova di cessazione.

## Rollback condizionale

Prima arresta e attendi la conferma di arresto di tutti gli scrittori coinvolti. Conserva in `attempts/<ATTEMPT-ID>/discarded/` (o `backups/integrations/<INT-ID>/discarded/`) una copia delle modifiche correnti da annullare e registra la diagnosi. Confronta il file corrente con l'ultima impronta attribuita al tentativo/task: se coincide e non ci sono scrittori concorrenti, ripristina `attempts/<ATTEMPT-ID>/base/` (usa `gen-base/` solo se stai annullando l'intera generazione senza conservare lavoro parziale valido). Se manca l'impronta o il file è cambiato per intervento esterno, non ripristinare alla cieca: usa confronto a tre vie (`base` vs `output del task` vs `stato corrente`) o sospendi i file in conflitto (`BLOCCATO`).

- Ripristina solo gli oggetti del manifest e le modifiche attribuibili, preservando modifiche preesistenti dell'utente, indice Git e risultati validi successivi.
- Rimuovi un file nuovo solo se registrato `assente` alla base, creato con certezza dal task e ancora identico all'impronta finale attribuita.
- Per eliminazioni verifica che il percorso sia ancora assente prima di ricrearlo dallo snapshot. Per rinomine tratta origine e destinazione come due oggetti distinti, preservando modifiche successive.
- Elimina directory solo se create dall'esecuzione e ora vuote. Niente cancellazioni ricorsive indiscriminate, `git clean`, `reset --hard`, checkout di alberi o ripristino globale della baseline.
- Con dipendenze, annulla in ordine inverso solo gli output obsoleti; se un file contiene anche lavoro valido successivo, esegui una modifica selettiva (`INT-ID`) o registra il conflitto come blocco.

Verifica lo stato finale e registra per percorso: `ripristinato`, `conservato` o `in conflitto`. `INVALIDATO` indica l'esclusione logica del task, non la cancellazione fisica.

## Chiusura

Salva il report e le evidenze fuori dai temporanei prima della pulizia. Controlla che tutti gli agenti siano `fermo confermato`, la proprietà di `<STATE-DIR>` e l'assenza di file estranei. Rimuovi solo i temporanei attribuibili alla sessione; conserva snapshot e `state.md` se servono per conflitti, blocchi, tracciabilità o ripresa. Non modificare le esclusioni Git dell'utente.
