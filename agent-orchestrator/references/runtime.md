# Compatibilità del runtime e configurazione subagenti

**Esegui all'attivazione della skill** — prima della richiesta se invocata senza argomenti e comunque prima della ricognizione — e riconvalida a cambi di runtime, provider o policy. La skill non installa agenti, non cambia il modello principale né le impostazioni globali.

## Rilevazione immediata all'attivazione

All'attivazione (sola lettura, senza scrivere su disco né esporre segreti), verifica subito:

1. **Strumento di delega**: schema reale (`Agent`, `Task`, `invoke_subagent`, `define_subagent`, `SendMessage` o equivalenti). Non inventare chiamate o parametri assenti dallo schema.
2. **Parametro `model`**: se esposto e valori/tier/versioni accettati (es. `flash_lite`, alias leggero o ID modello).
3. **Parametro `effort` / reasoning**: se strumento o profilo espone `effort`, `reasoning`, `thinking` o equivalenti.
4. **Configurazione locale e override**: se leggibili, ispeziona configurazioni locali del runtime e nomi delle sole variabili d'ambiente pertinenti (mai `env` intero né chiavi/segreti) per sapere quali parametri sono impostabili e quali ricadono sul default runtime.

Se manca l'obiettivo dell'utente, comunica sinteticamente l'esito già all'attivazione.

## Non-interruzione e precedenza

- **Parametri impostabili**: imposta `model` sull'ultima versione del modello più economico, leggero e veloce e `effort: "high"` (o massimo equivalente) in ogni delega. Vietato scegliere deliberatamente modelli più pesanti/costosi, versioni precedenti o effort inferiori.
- **Parametri NON impostabili dal modello AI**: **mai interrompere o bloccare**:
  - `effort`/reasoning non esposto/impostabile $\rightarrow$ default allo spawn; registra `effort: default runtime (parametro non esposto/impostabile)`.
  - `model` non esposto/impostabile $\rightarrow$ default allo spawn; registra `model: default runtime (parametro non esposto/impostabile)`.
  - `model` solo come alias (es. `flash_lite`) senza versione esatta/fallback provider $\rightarrow$ seleziona l'alias, prosegui e annota il limite di osservabilità.
  - Override globale non modificabile $\rightarrow$ non bloccare né alterare silenziosamente le impostazioni utente; prosegui e documenta l'override in `state.md` e nel resoconto finale.
- **`BLOCCATO`**: **solo** se manca una primitiva funzionante di avvio subagenti o mancano permessi/precondizioni operative.

Il solo prompt non configura modello o effort nel runtime. Precedenza di risoluzione (il primo livello attivo prevale):

1. **Override d'ambiente o configurazione globale**: prevale su chiamate e profili; se non modificabile, registralo in `state.md` e prosegui senza bloccare.
2. **Parametri per invocazione** (`model` leggero ultima versione + `effort: "high"` / equivalente, se esposti): passati allo strumento di delega (`Agent` / `invoke_subagent` o equivalente) per qualunque `subagent_type`.
3. **Definizione/frontmatter del profilo subagente**: usato se 1 e 2 non specificano modello/effort.
4. **Default runtime / ereditarietà allo spawn**: usato per parametri non impostabili; documenta in `state.md` e resoconto finale quali sono espliciti e quali di default.

Verifica capacità tramite strumenti/schema reali e consulta la documentazione ufficiale di runtime e provider se necessario.

## Profili `subagent_type`, capacità e permessi

Prima di delegare verifica lettura/scrittura, attesa, consegna e arresto. Delega solo esecuzione (`IMPLEMENTAZIONE` e recuperi); analisi e verifica restano all'orchestratore. Usa il profilo più restrittivo adatto:

| Tipologia task | Profilo consigliato | Parametri `model` ed `effort` | Caratteristiche e isolamento |
| --- | --- | --- | --- |
| `IMPLEMENTAZIONE` | `subagent_type: "general-purpose"` (o custom worker equivalente) | Target esposti (modello leggero ultima versione + `effort: "high"` / equivalente); default allo spawn per non impostabili senza bloccare | Scrittura e verifica nel perimetro; delega (`Agent` / spawn) disabilitata ai subagenti di primo livello. |

- **`run_in_background`**: in background (`run_in_background: true` o asincrono equivalente) il subagente non può chiedere autorizzazioni interattive (operazioni non pre-autorizzate negate). Usalo solo con permessi già attivi; un rifiuto immediato pre-esecuzione va riconciliato e può usare l'unico reinvio tecnico in foreground. Ampliamenti di perimetro tornano sempre all'orchestratore; vietati bypass dei permessi.
- **Restrizione strumenti**: se supportato (`tools` / `disallowedTools` o equivalente), escludi `Agent` e `Skill` (o strumenti di sub-delega) dai worker; per sola lettura usa allowlist minima (`Read, Grep, Glob` o equivalenti), mai shell/MCP mutanti. Se servono test con shell, esplicita effetti collaterali, esclusione `<STATE-DIR>` e isolamento. Senza restrizioni shell, dichiara procedurale (da brief) il divieto di sub-delega via CLI. Non installare definizioni agenti né cambiare impostazioni globali.
- **`isolation: "worktree"`**: crea worktree temporaneo da `HEAD` (senza modifiche locali staged/unstaged/non tracciate), lo imposta come `pwd` e lo rimuove se esce senza modifiche. Regole: 1) se servono input locali non committati, prepara worktree/copia gestito dall'orchestratore con input sincronizzati (o patch verificata fuori da `<STATE-DIR>`); 2) imponi nel brief di risolvere i percorsi rispetto alla `pwd` del worktree isolato e di **non scrivere mai** sui percorsi assoluti del repo principale.

## Controllo all'attivazione e tracciamento operativo

- Con shell: controlla solo nomi delle variabili d'ambiente pertinenti (override modello subagenti, flag forzatura globale, mapping modello leggero, `effort`/reasoning) e file di configurazione runtime, mai `env` intero. Senza shell: ispeziona schemi strumenti e config leggibili e registra variabili di processo `NON OSSERVABILI SENZA SHELL`.
- Registra alias, mapping, versione esatta e fallback se osservabili; altrimenti non bloccare e annota il limite di osservabilità.
- Non alterare silenziosamente impostazioni globali utente.
- Registra in `state.md` e resoconto finale la configurazione effettiva (`model` vs `default runtime`, `effort: high` vs `default runtime`) e gli ID runtime esposti (anche su `resume` / `SendMessage`). Non dichiarare prove a livello provider non esposte dal runtime.

## Installazione e verifica documentale

Mantieni `SKILL.md`, `templates/` e `references/` in `<skills-dir>/agent-orchestrator/` (progetto o utente); invocazione `/agent-orchestrator`. Non aggiungere `context: fork` né `model:` al frontmatter di `SKILL.md`. Consulta la documentazione ufficiale di runtime e provider su subagenti, modelli/fallback, `effort`/reasoning e skill quando servono conferme, registrando separatamente capacità documentate, configurazione osservata e limiti di osservabilità.
