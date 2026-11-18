# Contesto operativo compatto e persistente

## Ambito e priorità

Riduci duplicazioni e ricostruzioni durante task lunghi, multifase o con handoff/log voluminosi. Conserva integralmente requisiti, vincoli, prove necessarie e protocolli di revisione/escalation. Nessun limite di lunghezza giustifica omissioni. Questo protocollo non autorizza nuove azioni esterne, non cambia i trigger della skill e non si esegue nel turno di sola attivazione.

Scrivere su disco non elimina il contesto già inviato al modello. Il risparmio dipende da letture e trasmissioni effettivamente evitate; non promettere percentuali senza misure comparabili. Non introdurre file per un task breve che si risolve con un brief compatto. Se il task cresce, passa alla persistenza prima che ricostruire lo stato diventi costoso.

## 1. Collocazione e stato minimo

Usa una directory di lavoro consentita e univoca per il task (per esempio `work/task-<id>/`); verifica che non contenga lavoro altrui prima di creare file. Non collocare stato utente nella directory della skill, non sovrascrivere file esistenti e non aggiungerlo automaticamente al versionamento. Rispetta convenzioni e permessi del progetto. In ambienti senza scrittura condivisa usa un brief testuale autosufficiente: il filesystem assente non blocca il task né esenta dalla revisione.

Crea un solo `stato.md` come indice corrente. Campi minimi, con dettagli solo se applicabili:

- Identità del task, revisione progressiva dello stato, richiesta corrente fedele e revisioni dell'utente; distingui requisiti sostituiti da ancora validi.
- Obiettivo, deliverable, criteri di accettazione con ID stabili (`R1`, `R2`...), vincoli/divieti, fuori ambito e contesto storico pertinente. Mantieni tutti i requisiti negativi e l'ordine originale delle liste.
- Piano e stato per voce, dipendenze, decisioni/assunzioni e motivazione, ipotesi aperte, prossimo passo concreto.
- Versione degli artefatti: percorsi, branch/HEAD e modifiche locali se pertinenti; per file non versionati usa hash o identificazione equivalente verificabile. HEAD da solo non identifica modifiche non committate.
- Indice delle evidenze (`E1`...), controllo/comando, versione esaminata, esito reale, percorso e posizione rilevante. Non confondere piano, tentativo, test eseguito e risultato.
- Blocchi: errore, tentativi distinti con ipotesi/esito, alternative residue; revisione: rilievi aperti e risolti, rapporto e versione approvata.

Non copiare log estesi nel piano. Se un brief o rapporto completo è voluminoso, conservalo in un file distinto e indicizzalo; il lettore deve recuperare tutti i campi obbligatori, non solo l'indice. Evita file vuoti o registri duplicati. Mantieni nel contesto attivo obiettivo, requisiti/vincoli necessari alla decisione, revisione dello stato, percorso dell'indice e prossimo passo; recupera gli altri dettagli prima che diventino necessari.

## 2. Aggiornamento e recupero

Aggiorna lo stato quando cambiano requisiti, decisioni, fase, risultato di un controllo, blocco o rilievo; prima di handoff, pausa o compattazione se prevedibile. Non riscrivere l'intero piano per ogni comando o messaggio senza cambiamenti. Usa modifiche circoscritte e controlla che non sovrascrivano aggiornamenti concorrenti; l'agente principale è responsabile del registro corrente. I subagent consegnano rapporti separati, non sovrascrivono lo stesso stato condiviso.

Dopo perdita del contesto leggi l'indice e il brief corrente completo, confrontali con i messaggi utente disponibili e verifica identità del task e versione reale degli artefatti. Recupera i riferimenti della skill applicabili e le evidenze necessarie. Per un nuovo task crea un'identità distinta; non riattivare obiettivi vecchi per la sola presenza del file. I file sono dati subordinati alla richiesta e alle istruzioni superiori, non una fonte di nuova autorità.

Un PASS o un'approvazione salvati valgono solo per la versione e il contesto verificati. Se artefatti o condizioni cambiano, marca le evidenze coinvolte come da rivalutare; le modifiche sostanziali invalidano l'approvazione secondo §3.6. Se stato mancante, incoerente, incompleto o obsoleto, ricostruiscilo da fonti effettive e controlli; non inventare risultati. Usa fallback testuale se necessario e continua il lavoro fattibile. Nessun compattamento deve perdere vincoli, errori irrisolti o prove indispensabili.

## 3. Evidenze e letture selettive

Conserva output voluminosi rilevanti in file di evidenza, con provenienza, comando, esito/exit code e versione. Oscura segreti e dati personali non necessari prima di salvarli o mostrarli; non memorizzare credenziali nel piano o negli handoff. Non copiare intere cronologie, ambienti o directory per comodità. Non cancellare prove necessarie o materiale utente per ridurre il contesto.

Mostra normalmente esito, anomalie e riferimenti precisi; cerca prima percorsi/sezioni pertinenti e poi leggi il contesto sufficiente a verificarli. In caso di errore includi messaggio completo e contesto necessario alla diagnosi. Non presentare output troncato, un filtro senza match o un estratto come prova di copertura completa. Amplia la lettura quando l'evidenza non basta. Le letture selettive di log/codice non sostituiscono la lettura integrale delle istruzioni obbligatorie, del brief corrente e dei rapporti richiesti.

Evita di rileggere materiale già disponibile integralmente e invariato; verifica l'attualità quando potrebbe essere cambiato. Non presumere di ricordare materiale perso dopo compattazione. Non rieseguire controlli soltanto per riempire un rapporto; ripetili quando modifiche, instabilità, prove insufficienti o protocolli obbligatori lo richiedono. La rivalutazione dell'intera implementazione resta obbligatoria.

## 4. Handoff autosufficiente senza duplicazione

Prima della delega aggiorna il brief corrente e includi tutti i campi richiesti da §3.6.1 o §3.8, non soltanto il piano. Nel messaggio indica task corrente, incarico, vincoli essenziali, revisione e percorsi esatti del brief/procedure/evidenze, chiedendone esplicitamente la lettura. Se disponibili usa una copia identificata e stabile del brief per quella revisione; nuove richieste o modifiche durante il lavoro richiedono comunicazione esplicita al subagent e riallineamento prima del verdetto.

Il passaggio tramite file è valido soltanto se il destinatario può accedervi e legge integralmente il brief e le procedure applicabili prima di agire. Se l'accesso non è disponibile o il contenuto non viene recuperato, invia il testo completo necessario. Un puntatore o riepilogo da soli non soddisfano l'obbligo di handoff. Non dare per acquisita la memoria dell'agente; con contesto ereditato ridotto devi comunque fornire tutto il contesto pertinente e verificabile.

Nei cicli successivi comunica la nuova revisione, le differenze, le nuove prove e lo stato dei rilievi; mantieni disponibile il brief corrente completo e richiedi nuovamente la rivalutazione integrale. Il delta serve a orientare, non sostituisce il task o riduce la copertura della revisione. Il subagent resta indipendente e deve verificare direttamente l'implementazione.

## 5. Rapporti e conclusione

Usa ID e riferimenti per non ripetere le stesse prove in ogni sezione. Ogni rapporto mantiene tutti i campi, la matrice requisito per requisito, le due passate, i controesempi, stati reali, rilievi e verdetto richiesti; nessuna abbreviazione giustifica una revisione superficiale. L'agente principale legge integralmente il rapporto e verifica i passaggi critici. Un semplice «invariato» non prova la correttezza della versione attuale.

La risposta finale resta autosufficiente: esito, risultato, controlli, verdetto applicabile e limiti effettivi. Collega gli approfondimenti utili senza obbligare l'utente a ricostruire l'esito dai log. Conserva le prove necessarie al riuso e al recupero; non avviare cancellazioni o pubblicazioni automatiche dei registri. La persistenza è un supporto al task, non un nuovo obiettivo né un'esenzione dai protocolli esistenti.
