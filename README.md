# Skills

Raccolta di custom skill per agenti AI (Google Antigravity e ambienti compatibili).

## Skills disponibili

### [agent-orchestrator](./agent-orchestrator/SKILL.md)
Coordina attività complesse con task dipendenti o paralleli tramite subagenti configurati sull'ultima versione disponibile del modello più economico, leggero e veloce con `effort: high` (quando supportati dal runtime, altrimenti proseguendo con il default del subagente allo spawn senza mai bloccare l'esecuzione).
- **Rilevazione immediata all'attivazione**: rileva lo strumento di delega e i parametri di modello ed effort supportati prima di eseguire task o iniziare modifiche.
- **Architettura non bloccante**: prosegue con il default del runtime se parametri specifici non sono esposti o controllabili.
- **Piano persistente e verifica**: piano salvato atomicamente, snapshot, isolamento e verifica rigorosa di ogni consegna prima dell'integrazione.

### [compact-skill](./compact-skill/SKILL.md)
Compatta altre skill (`SKILL.md` ed eventuali file collegati come `references/`, `examples/`, `resources/`) riducendone lunghezza del testo, token e byte, mantenendo invariati al 100% funzionamento, logica, vincoli e semantica.

### [task-resolver-auto](./task-resolver-auto/SKILL.md)
Attiva una modalità persistente di esecuzione autonoma end-to-end, con diagnosi degli ostacoli, delega obbligatoria a subagent per i blocchi, verifica delle soluzioni e recupero fino al massimo risultato verificabile.

### [task-resolver-pro-v2](./task-resolver-pro-v2/SKILL.md)
Attiva una modalità persistente di esecuzione autonoma end-to-end (versione v2 strutturata con riferimenti modulari in `references/`), con diagnosi degli ostacoli, delega obbligatoria a subagent per i blocchi, verifica delle soluzioni e recupero fino al massimo risultato verificabile.
