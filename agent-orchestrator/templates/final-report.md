# Resoconto conclusivo

Adatta lunghezza alla complessità. Riporta solo dati da evidenze verificate; mai esiti positivi automatici né verifiche non eseguite.

- **Risultato e stato globale**: `COMPLETATO` / `INCOMPLETO` / `BLOCCATO`; persistenza (`<STATE-DIR>/state.md` o registro in chat per sola lettura); sintesi risultato e artefatti utilizzabili.
- **Copertura requisiti (`REQ-ID`)**: stato finale (`VERIFICATO`, `INCOMPLETO`, `BLOCCATO`, `INVALIDATO`), separando **verificato direttamente**, **verificato parzialmente** e **non verificabile**, con artefatti e prove.
- **Verifiche dirette orchestratore**: controlli locali/globali su risultato integrato (esclusa `<STATE-DIR>`), comandi esatti, `cwd`, exit code, esito; distingui controlli parziali/non eseguibili, regressioni corrette e problemi baseline.
- **Delega subagenti**: task delegati (`IMPLEMENTAZIONE` e recuperi; analisi/verifiche sempre orchestratore), `subagent_type`, generazioni/recuperi consumati (incluso pool condiviso sotto-task), configurazione `model` ed `effort`/reasoning rilevata all'attivazione e osservata in esecuzione (target o `default runtime allo spawn`, override env, mapping provider), limiti osservabilità o assenza deleghe. Non attribuire attività non eseguite.
- **Interventi orchestratore (`INT-ID`)**: `INTEGRAZIONE` da worktree/copie, eventuale unico `SBLOCCO CIRCOSCRITTO` dopo 5 recuperi falliti (con esito) e `AZIONE ESTERNA`, con stato finale, motivazione, snapshot e verifica diretta.
- **Azioni esterne** (se presenti): effetto autorizzato, destinatario/risorsa, ricevuta/stato remoto verificato e riconciliazioni su esiti incerti prima di ritentare.
- **Limiti, rollback e ripresa**: esito rollback per percorso (`ripristinato`, `conservato`, `in conflitto`), causa/impatto blocchi residui, assunzioni rilevanti e precondizione precisa ripresa.
- **Stato conservato e pulizia**: percorsi report/evidenze e (se conservato) `<STATE-DIR>/state.md`; conferma tutti gli agenti `fermo confermato`; pulizia selettiva effettuata o rinviata. Nessun link a temporanei eliminati.
