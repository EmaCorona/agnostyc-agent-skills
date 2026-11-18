# Piano persistente

Compila in `<STATE-DIR>/state.md` (o in chat se sola lettura); rimuovi opzioni inapplicabili. Solo l'orchestratore modifica questo file. Sostituisci `[...]` all'inizializzazione.

## Identità e stato globale

- RUN-ID, root canonica, `<STATE-DIR>` (o modalità in-chat sola lettura), proprietario:
- Richiesta originale (`$ARGUMENTS` / messaggio utente) e aggiornamenti in ambito:
- Runtime/versione, strumenti, persistenza (`mv` atomico da `state.md.tmp` vs `state.prev.md` + `Write`/`Edit`) e permessi:
- Git/non-Git, revisione iniziale e modifiche preesistenti (staged, unstaged, non tracciati, ignorati nei perimetri):
- Configurazione subagenti (rilevata all'attivazione): primitiva di delega, `model` (leggero ultima versione o `default runtime`), `effort`/reasoning (`high` o `default runtime allo spawn`), override d'ambiente, mapping provider, `subagent_type`, fallback e limiti di osservabilità:
- Stato globale: `PIANIFICAZIONE` / `IN CORSO` / `VERIFICA` / `COMPLETATO` / `INCOMPLETO` / `BLOCCATO`
- Fase/ondata, ultimo aggiornamento, prossima azione:
- Assunzioni, autorizzazioni esistenti e decisioni mancanti:
- Agenti attivi, risorse riservate, esclusioni obbligatorie (inclusa `<STATE-DIR>`), isolamento tra esecuzioni:
- Verifiche indispensabili e baseline fallimenti preesistenti:

## Requisiti

| ID | Requisito in ambito | Task/INT | Criterio osservabile | Artefatto | Evidenza | Stato |
| --- | --- | --- | --- | --- | --- | --- |
| REQ-01 | [Descrizione requisito] | [TASK-01 / INT-01] | [Criterio verificabile] | [Percorso o deliverable] | N/A (in attesa di verifica) | IN ATTESA |

Stati: `IN ATTESA`, `IN CORSO`, `IN VERIFICA`, `DA RIESAMINARE`, `VERIFICATO`, `INCOMPLETO`, `BLOCCATO`, `INVALIDATO`. Usa `N/A (in attesa di verifica)` prima della verifica. A chiusura: `VERIFICATO` richiede prova diretta completa; verifica parziale resta `INCOMPLETO` (dettagliando provata e mancante); blocco tecnico/permessi è `BLOCCATO` (o `INCOMPLETO` con causa). Su cambio input o difetti verificati, revoca i dipendenti (`DA RIESAMINARE`). Non marcare fallito un requisito non ancora eseguito.

## Task (ripetere per ciascuno)

### TASK-ID — Titolo

- Tipologia e generazione: `IMPLEMENTAZIONE` (subagente con parametri target o default runtime) | `GEN` (motivo eventuale `GEN+1`) | totale storico tentativi:
- Stato e tentativo: `IN ATTESA` / `PRONTO` / `IN CORSO` / `IN VERIFICA` / `DA CORREGGERE` / `DA RIESAMINARE` / `VERIFICATO` / `INVALIDATO` / `INCOMPLETO` / `BLOCCATO` | `iniziale` / `recupero 1` … `recupero 5` (eventuale unico reinvio tecnico pre-esecuzione):
- Agente e runtime: handle/ID subagente dedicato (nuovo per task, riusato via `SendMessage` per recuperi; eventuale subentro con motivo e recupero) | stato runtime (`attivo` / `fermo confermato` / `sconosciuto`) | `subagent_type` | `model` (impostato o `default runtime`), `effort` (`high` o `default runtime`) e prove/limiti di osservabilità:
- Budget e progresso: durata attesa, soglia inattività, `max_turns`, ultimo progresso:
- Scomposizione (se sotto-task): task padre e pool condiviso ($\text{ereditato} = 5 - R_{\text{padre}}$; $\text{residuo} = 5 - R_{\text{padre}} - \sum R_{\text{sotto-task}}$):
- Ultima intenzione pre-avvio, esito noto/incerto, consegna attesa:
- Requisiti (`REQ-ID`), obiettivo, contratto di interfaccia e criteri:
- Dipendenze e versioni/hash input verificati:
- Root e isolamento: `condiviso` / `worktree orchestratore` / `isolation: "worktree"` / `copia` (base e input non committati trasferiti):
- Letture autorizzate, istruzioni progetto ed esclusioni (inclusa `<STATE-DIR>`):
- Scritture autorizzate per operazione (`crea` / `modifica` / `elimina` / `rinomina`; `NESSUNA` se sola lettura):
- Risorse condivise, lettori/scrittori incompatibili e controlli isolati:
- Snapshot e manifest: `baseline/files/`, `gen-base/`, `attempts/<ATTEMPT-ID>/base/` e `manifest.md` (presenza iniziale, tipo, hash/target, permessi, copia, attribuzione, impronta finale; `impronte soli input` se sola lettura):
- Inventario prima/dopo (esclusi `.git/`, `<STATE-DIR>`, dipendenze massive fuori perimetro) e limiti di osservabilità:
- Report subagente (affermazione da verificare, non prova), modifiche reali verificate dall'orchestratore (file, diff, confronto col report) e verifiche dirette (`comando`, `cwd`, `exit code`, `esito`):
- Verifica locale, integrazione `INT-ID`, destinazione finale, verifica integrata e dipendenti sbloccati:
- Diagnosi, correzioni richieste, contatori e motivi blocchi:
- Rollback: `non necessario` / `pianificato` / `eseguito` / `parziale` / `bloccato`; `discarded/` ed esito per percorso (`ripristinato` / `conservato` / `in conflitto`):

## Integrazione e interventi dell'orchestratore (ripetere se necessari)

### INT-ID — Titolo

- Tipologia e stato: `INTEGRAZIONE` / `SBLOCCO CIRCOSCRITTO` / `AZIONE ESTERNA` | `PIANIFICATO` / `IN CORSO` / `VERIFICATO` / `INCOMPLETO` / `BLOCCATO` / `ANNULLATO`
- Requisiti (`REQ-ID`), task correlati, perimetro e input verificati:
- Snapshot/manifest in `backups/integrations/<INT-ID>/` (`base/`, `discarded/`, `manifest.md`) e conflitti col lavoro locale:
- Modifiche/operazioni eseguite e verifiche dirette (`comando`, `cwd`, `exit code`, `esito`):
- Solo `SBLOCCO CIRCOSCRITTO`: prova esecuzione subagente (parametri target o default allo spawn) ed esaurimento `iniziale` + 5 recuperi dello stesso subagente (`INCOMPLETO`), difetti residui verificati, perimetro minimo, esito (`VERIFICATO` oppure fallito → rollback selettivo e task `INCOMPLETO`):
- Solo `AZIONE ESTERNA`: autorizzazione, destinatario/risorsa, intenzione pre-invio, ricevuta/stato remoto e riconciliazione se incerto:

## Registro eventi e conclusione

- Modifiche richiesta, invalidazioni, arresti confermati e consegne obsolete conservate:
- Verifiche globali (esclusa `<STATE-DIR>`): comando, `cwd`, verifica/input, exit code, sintesi e log:
- Riepilogo requisiti: verificati direttamente, verificati parzialmente, non verificabili:
- Report/evidenze fuori dai temporanei; conferma agenti tutti `fermo confermato` e nessun `INT-ID` pendente; pulizia selettiva o motivi di conservazione:
- Stato finale (`COMPLETATO` / `INCOMPLETO` / `BLOCCATO`) e condizione concreta di ripresa se non completato:
