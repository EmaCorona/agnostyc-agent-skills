# Compatibilità del runtime e configurazione dei subagenti (modello leggero + effort: high o default runtime)

**Leggi ed esegui questa verifica immediatamente all'attivazione della skill** — prima ancora che l'utente formuli la richiesta se invocata senza argomenti, e comunque prima di iniziare la ricognizione del progetto — e riconvalidala dopo cambiamenti di runtime, provider o policy. La skill non installa agenti, non cambia il modello principale e non modifica impostazioni globali.

## Rilevazione immediata all'attivazione

Appena la skill viene attivata (in sola lettura, senza scrivere file su disco e senza esporre segreti), accertati subito di:

1. **Strumento di delega disponibile**: controlla lo schema reale dello strumento di delega del runtime (`Agent`, `Task`, `invoke_subagent`, `define_subagent`, `SendMessage` o equivalenti). Non inventare chiamate né parametri non presenti nello schema.
2. **Parametro `model` esposto**: verifica se lo strumento di delega permette di selezionare il modello e quali valori/tier/versioni espone (es. `flash_lite`, alias leggero o ID modello specifico).
3. **Parametro `effort` / reasoning esposto**: verifica se lo strumento di delega (o il profilo subagente) espone un parametro per lo sforzo computazionale di ragionamento (`effort`, `reasoning`, `thinking` o equivalente).
4. **Configurazione locale e override pertinenti**: se hai accesso in lettura a configurazioni locali del runtime o nomi di variabili d'ambiente pertinenti, controllali subito in modo mirato (senza stampare `env` intero né chiavi/segreti) per sapere in anticipo quali parametri sono effettivamente impostabili e quali ricadono sul default del runtime.

Comunica sinteticamente l'esito di questa rilevazione già al momento dell'attivazione se manca ancora l'obiettivo dell'utente, così che eventuali parametri lasciati al default del runtime siano noti fin da subito e non scoperti dopo la richiesta.

## Regola di non-interruzione e precedenza di modello ed effort

- **Quando i parametri sono impostabili**: imposta sempre `model` sull'ultima versione disponibile del modello più economico, leggero e veloce esposto dal runtime e imposta `effort: "high"` (o valore massimo equivalente del parametro di reasoning) in ogni delega. Non selezionare mai deliberatamente un modello più pesante/costoso, una versione precedente o un effort inferiore quando il runtime consente di impostare i valori target.
- **Quando `model` o `effort`/reasoning NON sono impostabili dal modello AI che usa la skill**: **non interrompere e non bloccare mai l'esecuzione**.
  - Se `effort` / reasoning (o equivalente) non è esposto dallo schema o non può essere impostato, **prosegui normalmente con il default di quando viene staccato il subagente** nel runtime e registra `effort: default runtime (parametro non esposto/impostabile)`.
  - Se `model` non è esposto dallo schema o non può essere impostato, **prosegui normalmente con il modello di default del subagente** nel runtime e registra `model: default runtime (parametro non esposto/impostabile)`.
  - Se `model` espone solo un alias di fascia (es. `flash_lite`) senza esporre la versione esatta o la catena di fallback del provider, seleziona comunque quell'alias leggero e **prosegui senza bloccare**, annotando il limite di osservabilità.
  - Se un override globale del runtime forza un modello o un effort non modificabile dal modello AI che usa la skill, **non bloccare l'esecuzione** e non cambiare silenziosamente le impostazioni globali dell'utente: prosegui con il subagente avviato dal runtime e documenta l'override osservato in `state.md` e nel resoconto finale.
- **Quando bloccare davvero (`BLOCCATO`)**: marca i task esecutivi `BLOCCATO` **solo** se il runtime non espone alcuna primitiva funzionante per avviare subagenti (delega tecnicamente assente/rotta) o per mancanza di permessi/precondizioni operative.

Il solo testo del prompt non seleziona tecnicamente né il modello né l'effort nel runtime. Quando supportati dal framework, la risoluzione del modello e dell'effort del subagente segue questo ordine di precedenza (il primo livello attivo prevale):

1. **Override d'ambiente o di configurazione globale del runtime**: prevale sui parametri per chiamata e sulla definizione/frontmatter del profilo del subagente. Può nascondere o sovrascrivere `model` o `effort` per singola chiamata. Se è attivo e non modificabile dal modello AI che usa la skill, registralo esplicitamente in `state.md` e prosegui senza bloccare.
2. **Parametri per invocazione (`model` sul modello più economico, leggero e veloce disponibile + `effort: "high"` / equivalente, se esposti)**: passati esplicitamente nella chiamata allo strumento di delega (`Agent` / `invoke_subagent` o equivalente). Imposta sempre tutti quelli effettivamente esposti dallo schema, per qualunque `subagent_type`.
3. **Definizione o frontmatter del profilo del subagente**: usato se né l'ambiente né i parametri per invocazione specificano modello o effort.
4. **Default del runtime / ereditarietà al momento dello spawn**: usato automaticamente dal runtime per i parametri che il modello AI che usa la skill non può impostare. Documenta sempre in `state.md` e nel resoconto finale quali parametri sono stati impostati esplicitamente e quali sono rimasti al default del subagente.

Verifica attraverso gli strumenti e lo schema reali del runtime le capacità effettivamente installate. Consulta la documentazione ufficiale del runtime e del provider in uso relativa ai subagenti quando necessario.

## Profili `subagent_type`, capacità e permessi

Prima di assegnare lavoro, verifica lettura/scrittura, attesa, consegna e arresto. Ai subagenti vengono delegati solo task di esecuzione (`IMPLEMENTAZIONE` e recuperi); analisi e verifica restano all'orchestratore. Seleziona il profilo `subagent_type` più restrittivo e adatto al task:

| Tipologia task | Profilo runtime consigliato | Parametri `model` ed `effort` | Caratteristiche e isolamento |
| --- | --- | --- | --- |
| `IMPLEMENTAZIONE` | `subagent_type: "general-purpose"` (o custom worker equivalente) | Ultima versione disponibile del modello più economico, leggero e veloce + `effort: "high"` (o equivalente) **quando esposti dallo schema**; per i parametri non impostabili si usa il **default del runtime allo spawn** senza mai bloccare | Accesso agli strumenti di scrittura e verifica nel perimetro autorizzato; delega (`Agent` / spawn) disabilitata ai subagenti di primo livello. |

- **Foreground vs Background (`run_in_background`)**: un subagente con `run_in_background: true` (o modalità asincrona equivalente) non può chiedere autorizzazioni interattive all'utente (le operazioni non pre-autorizzate vengono negate). Usa `run_in_background: true` solo quando i permessi degli strumenti necessari sono già attivi; un rifiuto immediato dei permessi in background prima di qualsiasi modifica va riconciliato e può usare l'unico reinvio tecnico pre-esecuzione in foreground. Una richiesta di ampliamento del perimetro torna sempre all'orchestratore; non attivare bypass dei permessi.
- **Restrizione degli strumenti**: quando il runtime o i profili custom permettono di limitare gli strumenti (`tools` / `disallowedTools` o meccanismo equivalente), escludi `Agent` e `Skill` (o gli strumenti di sub-delega) dai worker; per sola lettura usa una allowlist minima (`Read, Grep, Glob` o equivalenti di sola lettura). Non includere shell o MCP mutanti in un profilo dichiarato tecnicamente di sola lettura. Se servono test con shell, esplicita effetti collaterali, esclusione di `<STATE-DIR>` e isolamento. Senza restrizioni sulla shell, dichiara che il divieto di sub-delega tramite CLI è procedurale (imposto dal brief). Non installare automaticamente definizioni di agenti o cambiare impostazioni globali.
- **Isolamento worktree (`isolation: "worktree"`)**: quando lo strumento di delega (`Agent` o equivalente) viene invocato con `isolation: "worktree"`, il runtime crea un worktree temporaneo da `HEAD` all'avvio (senza modifiche locali staged, unstaged o file non tracciati del workspace principale), lo imposta come `pwd` del subagente e lo rimuove se il subagente termina senza modifiche. Rispetta due regole: 1) se il task richiede input locali non committati, prepara prima un worktree/copia gestito dall'orchestratore con gli input sincronizzati (o trasferisci una patch verificata fuori da `<STATE-DIR>`); 2) con `isolation: "worktree"`, imponi nel brief di risolvere tutti i percorsi rispetto alla propria `pwd` del worktree isolato e di **non scrivere mai** sui percorsi assoluti del repository principale.

## Controllo all'attivazione e tracciamento operativo

- Con accesso alla shell, verifica all'attivazione solo i nomi delle variabili d'ambiente pertinenti del runtime/provider (override del modello dei subagenti, flag di forzatura globale, mapping del modello leggero e configurazione di `effort`/reasoning) e i file di configurazione del runtime, senza stampare l'intero ambiente (`env`) per non esporre chiavi o segreti. Senza shell, ispeziona lo schema degli strumenti e i file di configurazione leggibili del runtime e registra che le variabili di processo sono `NON OSSERVABILI SENZA SHELL`.
- Se alias, mapping, versione esatta o fallback del provider sono osservabili, registrali; se non sono osservabili o non sono configurabili dal modello AI che usa la skill, **non bloccare l'esecuzione** e registra esplicitamente il limite di osservabilità.
- Non cambiare mai silenziosamente le impostazioni globali dell'utente.
- Registra in `state.md` e nel resoconto finale la selezione effettivamente configurata (`model` impostato vs `default runtime`, `effort: high` impostato vs `default runtime`) e gli identificativi runtime esposti (anche nelle riprese tramite `resume` o `SendMessage`). Non dichiarare una prova del modello o dell'effort a livello provider se il runtime non la espone.

## Installazione e verifica documentale

Mantieni l'intera cartella (`SKILL.md`, `templates/`, `references/`) nella directory delle skill prevista dal runtime o framework di agenti AI in uso, a livello di progetto o utente (`<skills-dir>/agent-orchestrator/`). L'invocazione è `/agent-orchestrator`. Non aggiungere `context: fork` né `model:` al frontmatter di `SKILL.md`. Consulta la documentazione ufficiale delle skill del runtime in uso.

Consulta la documentazione ufficiale aggiornata del runtime e del provider in merito a subagenti, modelli/fallback, parametro `effort`/reasoning e skill quando occorrono conferme. Registra separatamente capacità documentate, configurazione osservata e limiti di osservabilità; imposta sempre modello leggero ultima versione ed `effort: high` quando esposti, proseguendo con il default del subagente per i parametri non impostabili.
