---
name: compact-skill
description: >-
  Compatta altre skill (file SKILL.md ed eventuali file collegati) riducendone
  lunghezza del testo, token e byte, mantenendo invariati al 100% funzionamento,
  logica, vincoli e semantica. Usa questa skill quando l'utente chiede di
  compattare, sintetizzare, snellire, ridurre o ottimizzare il peso di una
  skill esistente.
---

# Compact Skill (`compact-skill`)

Riduci il peso in byte, caratteri e token dei file di una skill target (`SKILL.md` ed eventuali file collegati come `references/`, `examples/`, `resources/`), mantenendo invariati al 100% comportamento, logica, regole e semantica.

## Principi di intervento

- **Intervento mirato**: analizza il testo e modifica solo le parti verbose, ripetitive o ridondanti; lascia intatte le parti già essenziali.
- **Sintesi fedele**: riformula in modo diretto e conciso preservando esattamente significato, sfumature operative e tutti i vincoli (condizioni, eccezioni, divieti, priorità, soglie).

## Vincoli di sicurezza (cosa NON alterare)

- **Frontmatter YAML**: mantienilo invariato; puoi ottimizzare `description` solo se espressamente utile o richiesto, senza perdere alcuna parola chiave di attivazione.
- **Elementi tecnici e strutturali**: non alterare blocchi di codice, comandi CLI, percorsi di file, URL, placeholder/variabili e schemi di output.
- **Completezza operativa**: non eliminare passaggi operativi, regole condizionali, fallback, criteri di verifica o dettagli tecnici necessari alla corretta esecuzione della skill.

## Flusso di lavoro

### 1. Analisi e misurazione iniziale
1. Individua e leggi `SKILL.md` e gli eventuali file collegati della skill target.
2. Registra per ogni file la dimensione iniziale esatta (byte, caratteri e righe, es. tramite `wc -c -m -l`).
3. Conserva un riferimento della versione originale (es. copia temporanea nella directory scratch o `git diff` se il repository è tracciato) per consentire il confronto indipendente al punto 3.
4. Identifica ridondanze, giri di parole, ripetizioni tra sezioni e formule prolisse.

### 2. Compressione
1. Riscrivi in modo sintetico solo le sezioni migliorabili.
2. Mantieni la struttura logica, l'ordine operativo e la leggibilità.
3. Verifica che nessun vincolo di sicurezza sia stato violato.

### 3. Verifica indipendente tramite subagent (obbligatoria)
Al termine delle modifiche, lancia **obbligatoriamente** un subagent dedicato (`invoke_subagent`, es. `research` o `self`) fornendogli i percorsi o i contenuti della versione originale e di quella compattata.
Il subagent deve confrontare le due versioni in modo indipendente e verificare che:
- nessun comportamento, regola, condizione, eccezione, divieto o priorità sia stato alterato o perso;
- frontmatter, blocchi di codice, comandi CLI, percorsi, placeholder e schemi di output siano intatti;
- nessun dettaglio operativo necessario all'esecuzione sia stato rimosso.

**Ciclo di correzione**: se il subagent segnala qualsiasi alterazione semantica o funzionale, correggi tutte le parti indicate e ripeti la verifica con subagent fino a ottenere esito positivo senza riserve.

### 4. Output e report di riduzione
Solo dopo il via libera definitivo del subagent (e dopo aver rimosso eventuali copie temporanee di confronto):
1. Presenta in sintesi le modifiche applicate.
2. Riporta l'esito della verifica indipendente del subagent.
3. Fornisci un riepilogo puntuale della riduzione ottenuta per ogni file compattato (e il totale complessivo se i file sono più di uno), indicando:
   - Dimensione iniziale vs dimensione finale (byte, caratteri e righe)
   - Differenza assoluta e percentuale di riduzione (es. `-1.420 byte / -32%`)
