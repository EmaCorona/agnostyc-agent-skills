# Skills

Raccolta di custom skill per agenti AI (Google Antigravity e ambienti compatibili).

## Skills disponibili

### [agent-orchestrator](./agent-orchestrator/SKILL.md)
Coordina attività complesse con task dipendenti o paralleli tramite subagenti configurati sull'ultima versione disponibile del modello più economico, leggero e veloce con `effort: high` (quando supportati dal runtime, altrimenti proseguendo con il default del subagente allo spawn senza mai bloccare l'esecuzione).
- **Rilevazione immediata all'attivazione**: rileva lo strumento di delega e i parametri di modello ed effort supportati prima di eseguire task o iniziare modifiche.
- **Architettura non bloccante**: prosegue con il default del runtime se parametri specifici non sono esposti o controllabili.
- **Piano persistente e verifica**: piano salvato atomicamente, snapshot, isolamento e verifica rigorosa di ogni consegna prima dell'integrazione.
