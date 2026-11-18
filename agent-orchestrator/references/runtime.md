# Compatibilità runtime e configurazione subagenti

**Esegui all'attivazione** — prima della richiesta se senza argomenti e comunque prima della ricognizione — e riconvalida a cambi runtime/provider/policy. La skill non installa agenti né modifica modello principale o impostazioni globali.

## Rilevazione immediata all'attivazione

In sola lettura (zero scritture/segreti), verifica:

1. **Strumento delega**: schema reale (`Agent`, `Task`, `invoke_subagent`, `define_subagent`, `SendMessage` o equivalenti). Non inventare chiamate/parametri assenti.
2. **Parametro `model`**: se esposto e valori/tier/versioni accettati (es. `flash_lite`, alias leggero o ID modello).
3. **Parametro `effort`/reasoning**: se strumento/profilo espone `effort`, `reasoning`, `thinking` o equivalenti.
4. **Config locale e override**: se leggibili, ispeziona config runtime e nomi sole env pertinenti (mai `env` intero né chiavi/segreti) per distinguere impostabili da default runtime.

Se manca obiettivo utente, comunica sinteticamente l'esito all'attivazione.

## Non-interruzione e precedenza

- **Parametri impostabili**: imposta `model` su ultima versione modello più economico, leggero e veloce ed `effort: "high"` (o massimo equivalente) in ogni delega. Vietati modelli più pesanti/costosi, versioni precedenti o effort inferiori.
- **Parametri NON impostabili dal modello AI**: **mai bloccare**:
  - `effort`/reasoning non esposto/impostabile $\rightarrow$ default allo spawn; registra `effort: default runtime (parametro non esposto/impostabile)`.
  - `model` non esposto/impostabile $\rightarrow$ default allo spawn; registra `model: default runtime (parametro non esposto/impostabile)`.
  - `model` solo alias (es. `flash_lite`) senza versione/fallback provider $\rightarrow$ seleziona alias, prosegui, annota limite osservabilità.
  - Override globale non modificabile $\rightarrow$ non bloccare né alterare impostazioni utente; prosegui e documenta in `state.md` e resoconto finale.
- **`BLOCCATO`**: **solo** se manca primitiva funzionante avvio subagenti o permessi/precondizioni operative.

Il solo prompt non configura modello/effort. Precedenza (prevale il primo livello attivo):
1. **Override env o config globale**: prevale su chiamate/profili; se non modificabile, registra in `state.md` e prosegui.
2. **Parametri invocazione** (`model` leggero ultima versione + `effort: "high"` / equivalente, se esposti): passati allo strumento delega (`Agent` / `invoke_subagent` o equivalente) per qualunque `subagent_type`.
3. **Definizione/frontmatter profilo subagente**: usato se 1 e 2 non specificano modello/effort.
4. **Default runtime / ereditarietà allo spawn**: usato per parametri non impostabili; documenta in `state.md` e resoconto finale espliciti vs default.

Verifica capacità da schemi reali; consulta docs ufficiali runtime/provider se serve.

## Profili `subagent_type`, capacità e permessi

Prima di delegare verifica lettura/scrittura, attesa, consegna, arresto. Delega solo esecuzione (`IMPLEMENTAZIONE` e recuperi); analisi/verifica restano all'orchestratore. Usa profilo più restrittivo adatto:

| Tipologia task | Profilo consigliato | Parametri `model` ed `effort` | Caratteristiche e isolamento |
| --- | --- | --- | --- |
| `IMPLEMENTAZIONE` | `subagent_type: "general-purpose"` (o custom worker equivalente) | Target esposti (modello leggero ultima versione + `effort: "high"` / equivalente); default allo spawn per non impostabili senza bloccare | Scrittura/verifica nel perimetro; delega (`Agent`/spawn) disabilitata a subagenti 1° livello. |

- **`run_in_background`**: in background (`run_in_background: true` o asincrono equivalente) il subagente non chiede autorizzazioni interattive (operazioni non pre-autorizzate negate). Usalo solo con permessi attivi; rifiuto immediato pre-esecuzione va riconciliato e può usare l'unico reinvio tecnico in foreground. Ampliamenti perimetro tornano all'orchestratore; vietati bypass permessi.
- **Restrizione strumenti**: se supportato (`tools` / `disallowedTools` o equivalente), escludi `Agent` e `Skill` (o strumenti sub-delega) dai worker; per sola lettura allowlist minima (`Read, Grep, Glob` o equivalenti), mai shell/MCP mutanti. Per test con shell esplicita effetti collaterali, esclusione `<STATE-DIR>` e isolamento. Senza restrizioni shell, dichiara procedurale (da brief) divieto sub-delega via CLI. Non installare agenti né cambiare impostazioni globali.
- **`isolation: "worktree"`**: crea worktree temporaneo da `HEAD` (senza modifiche locali staged/unstaged/non tracciate), lo imposta come `pwd` e lo rimuove se esce senza modifiche. Regole: 1) se servono input locali non committati, prepara worktree/copia orchestratore con input sincronizzati (o patch verificata fuori `<STATE-DIR>`); 2) imponi nel brief di risolvere percorsi rispetto a `pwd` worktree isolato e **mai scrivere** su percorsi assoluti repo principale.

## Controllo all'attivazione e tracciamento operativo

- Con shell: controlla solo nomi env pertinenti (override modello subagenti, flag forzatura globale, mapping modello leggero, `effort`/reasoning) e config runtime, mai `env` intero. Senza shell: ispeziona schemi strumenti e config leggibili, registrando variabili processo `NON OSSERVABILI SENZA SHELL`.
- Registra alias, mapping, versione esatta e fallback se osservabili; altrimenti non bloccare e annota limite osservabilità.
- Non alterare silenziosamente impostazioni globali utente.
- Registra in `state.md` e resoconto finale configurazione effettiva (`model` vs `default runtime`, `effort: high` vs `default runtime`) e ID runtime esposti (anche su `resume`/`SendMessage`). Non dichiarare prove provider non esposte dal runtime.

## Installazione e verifica documentale

Mantieni `SKILL.md`, `templates/` e `references/` in `<skills-dir>/agent-orchestrator/` (progetto/utente); invocazione `/agent-orchestrator`. Non aggiungere `context: fork` né `model:` al frontmatter `SKILL.md`. Consulta docs ufficiali runtime/provider su subagenti, modelli/fallback, `effort`/reasoning e skill se servono conferme, registrando separatamente capacità documentate, configurazione osservata e limiti osservabilità.
