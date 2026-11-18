### 3.6 Revisione indipendente obbligatoria e rigorosa di ogni modifica

**Vincolo non negoziabile:** ogni volta che l'agente principale completa una modifica concreta — per esempio al codice, alla documentazione, a una configurazione, a uno script, a un test o a un altro artefatto — deve avviare un subagent distinto per una revisione indipendente. La modifica e il task che la contiene **non possono essere dichiarati conclusi** finché il subagent non ha approvato esplicitamente l'implementazione finale.

Questa regola si applica anche quando la modifica sembra semplice, i test passano o l'agente principale è già sicuro della soluzione. Non è un'escalation facoltativa riservata ai casi difficili: è una fase obbligatoria del ciclo di implementazione e prevale sulle disposizioni più generali che consentono di proseguire senza delega dopo un'autovalutazione.

#### 3.6.1 Identifica il task giusto prima di avviare la revisione

Prima di avviare il subagent, l'agente principale deve identificare la richiesta dell'utente che ha generato **la modifica attualmente sottoposta a revisione**. Non deve usare automaticamente la prima domanda dopo l'attivazione della skill, il task precedente o un vecchio handoff.

Il brief da passare al revisore deve riportare esplicitamente:

1. **Richiesta utente corrente**, riportata testualmente o con una trascrizione fedele che mantenga tutti i requisiti e i vincoli. Se il task è stato aggiornato da messaggi successivi, includi la richiesta iniziale pertinente e tutte le revisioni successive, chiarendo quali requisiti sono stati aggiunti, modificati o sostituiti.
2. **Obiettivo e criteri di accettazione correnti**, ricavati dalla richiesta che si sta eseguendo, compresi requisiti negativi come «non modificare», «mantenere», «non rimuovere» o «non introdurre».
3. **Contesto precedente pertinente**, solo se necessario a capire il progetto o il seguito richiesto, separandolo chiaramente dai requisiti della richiesta corrente.
4. **Implementazione effettiva**, file modificati, diff aggiornato, aree correlate e stato corrente del workspace/branch, quando disponibili.
5. **Verifiche realmente eseguite**, risultati grezzi rilevanti, test mancanti e limiti noti.
6. **Rilievi precedenti del revisore** e modo in cui sono stati affrontati, se si tratta di un ciclo di rivalutazione.

Il brief deve dire chiaramente al subagent: **«Revisiona la richiesta corrente riportata in questo brief, non la prima richiesta della sessione né un task precedente. Usa il contesto storico esclusivamente nei limiti esplicitamente indicati. Non presumere che la descrizione dell'agente principale sia completa o corretta: verifica direttamente i requisiti e l'implementazione.»** Non affidarti a una memoria implicita della conversazione o a un generico invito a «controllare la modifica».

#### 3.6.2 Revisione indipendente: controlli obbligatori

Il subagent deve eseguire una valutazione sostanziale e scettica, non una conferma rapida o benevola del lavoro svolto. Deve leggere la richiesta corrente, ispezionare l'implementazione reale e verificare almeno i seguenti aspetti, adattandoli alla natura del task:

1. **Matrice di conformità requisito per requisito.** Scomponi la richiesta corrente in tutti i requisiti espliciti, vincoli, divieti, deliverable e criteri di accettazione. Per ognuno riporta requisito, prova concreta, metodo di verifica e stato: `PASS`, `FAIL` oppure `NON VERIFICATO`. Non accorpare requisiti distinti in una valutazione generica e non ignorare requisiti negativi o dettagli apparentemente minori.
2. **Conformità del 100% ai requisiti.** L'obiettivo della revisione è verificare che l'implementazione rispetti il 100% dei requisiti applicabili della richiesta corrente. Un requisito `FAIL` o `NON VERIFICATO`, un'area della richiesta non esaminata o una prova insufficiente impedisce l'approvazione. Non convertire l'assenza di prove in prova di correttezza e non approvare solo perché il risultato «sembra funzionare».
3. **Ispezione indipendente dell'implementazione.** Esamina direttamente il diff corrente e i file pertinenti nel loro contesto, incluse le funzioni, i moduli, le configurazioni, i test e la documentazione adiacenti. Non basarti solo sul riepilogo dell'agente principale, sui nomi dei file, sulle intenzioni dichiarate o sul fatto che un test esistente passi.
4. **Integrazione nel progetto.** Valuta se la modifica rispetta architettura, responsabilità dei moduli, API e interfacce, convenzioni, dipendenze, configurazioni, lifecycle, compatibilità delle versioni, flussi di esecuzione e documentazione del progetto. Cerca duplicazioni, logica collocata nel modulo sbagliato, assunzioni non valide e incoerenze con il comportamento preesistente.
5. **Regressioni ed effetti collaterali.** Cerca attivamente scenari in cui la modifica possa rompere funzionalità esistenti o introdurre errori: casi limite, percorsi di errore, risorse mancanti, condizioni di race/concorrenza, compatibilità retroattiva, interazioni con altri moduli, stati iniziali/finali e configurazioni alternative, quando pertinenti. Valuta probabilità e gravità dei rischi e indica le aree che richiedono test mirati.
6. **Qualità delle verifiche.** Controlla che i test eseguiti coprano il comportamento richiesto e i rischi della modifica. Quando possibile, esegui direttamente test pertinenti o chiedi evidenze riproducibili; non accettare come prova un test non eseguito, un risultato soltanto dichiarato o un controllo che non verifica il requisito in questione. Per documenti, ricerca o altri task non di codice, usa controlli equivalenti e proporzionati.
7. **Rispetto dell'ambito e dei vincoli.** Verifica che non siano state modificate aree che dovevano restare intatte, che non siano stati introdotti cambiamenti non richiesti e che siano preservati i vincoli di sicurezza, reversibilità e autorizzazione.
8. **Ricerca attiva di controesempi.** Prova a falsificare la soluzione: cerca almeno i punti più plausibili in cui fallirebbe, controlla spiegazioni alternative e non assumere che la diagnosi iniziale sia corretta soltanto perché l'implementazione la rispecchia.

Il revisore deve motivare i rilievi con riferimenti precisi a requisiti, file, righe, comportamenti, output o test quando disponibili. Espressioni vaghe come «sembra corretto», «non vedo problemi» o «i test passano» non sono una revisione sufficiente.

#### 3.6.2.1 Protocollo anti-approvazione superficiale

Il subagent opera come **gatekeeper tecnico indipendente**, non come assistente incaricato di convalidare il lavoro dell'agente principale. Il suo obiettivo non è approvare la modifica né dimostrare che l'agente principale ha ragione: è stabilire, sulla base di prove, se la versione esaminata soddisfa integralmente la richiesta corrente e se può essere integrata senza difetti noti inaccettabili. Non deve essere compiacente, ma non deve nemmeno inventare difetti per sembrare rigoroso.

Per ogni revisione, il subagent deve completare entrambe le passate seguenti prima di emettere un verdetto:

1. **Passata di conformità:** ricostruisci la richiesta corrente requisito per requisito e confrontala con il risultato effettivo. Verifica anche i vincoli negativi, le condizioni limite, i comportamenti richiesti in caso di errore e ciò che doveva restare invariato. Non basarti soltanto sulla descrizione della soluzione fornita dall'agente principale.
2. **Passata avversaria d'integrazione:** considera temporaneamente la modifica come potenzialmente difettosa e cerca scenari realistici che la facciano fallire. Ripercorri il flusso di controllo e i dati attraverso il codice o l'artefatto interessato e i moduli vicini; controlla i percorsi alternativi, gli stati parziali, le identità multiple, gli input non validi, i timeout, gli errori e gli effetti collaterali pertinenti al dominio. Poi verifica se test o controlli esistenti intercettano davvero questi scenari.

**Non è sufficiente che il nuovo codice sia migliore del precedente.** Il fatto che un commit risolva una parte del problema, introduca test o migliori la selettività non dimostra che abbia risolto tutti i difetti né che non ne abbia introdotti altri. Il subagent deve separare chiaramente ciò che è stato corretto, ciò che rimane problematico e ciò che non può ancora essere dimostrato.

Quando la revisione riguarda un repository o una modifica versionata, verifica direttamente, quando le informazioni sono accessibili:

- repository e workspace esaminati, branch corrente e stato delle modifiche locali;
- commit/HEAD effettivamente sottoposto a revisione e commit di base usato per il confronto;
- relazione con il branch di destinazione (per esempio commit avanti/indietro), senza presumere che il branch di destinazione sia `main` se il task ne indica un altro;
- elenco dei file modificati e diff completo, più il contesto dei moduli e dei chiamanti coinvolti;
- eventuali pipeline CI e relativo stato osservato al momento della revisione.

Se un dato non è accessibile o non è stato verificato, dichiaralo `NON VERIFICATO` o `NON DISPONIBILE`: non inventare hash, date, numero di commit, stato CI o risultati. Se il task non riguarda un repository, usa le informazioni equivalenti pertinenti all'artefatto revisionato.

Per ogni difetto o rischio, descrivi la catena causale: **condizione d'innesco → percorso del codice/comportamento → risultato errato → impatto**. Indica il file e la posizione specifica quando disponibili, spiega perché i controlli attuali non intercettano il caso e proponi una correzione concreta accompagnata dal test che dovrebbe impedire la regressione. Non limitarti a dire che «servono più test».

I test e le dichiarazioni devono essere classificati in modo rigoroso:

- `PASS`: eseguito direttamente o osservato da una fonte verificabile e pertinente, con risultato disponibile;
- `FAIL`: eseguito e fallito, oppure requisito chiaramente contraddetto dal comportamento osservato;
- `RUNNING`: la pipeline o il test è ancora in esecuzione;
- `NOT RUN`: il test non è stato eseguito;
- `UNAVAILABLE`: non è possibile eseguirlo o consultarne il risultato nell'ambiente corrente;
- `NON VERIFICATO`: le prove raccolte non bastano a stabilire la correttezza.

Un risultato riportato nella documentazione, in un commento, nel messaggio dell'agente principale o in una conversazione precedente **non prova da solo** che il controllo sia stato eseguito sulla versione corrente. Se la CI è ancora in esecuzione, non dichiararla verde. Se i test sono mockati, non usarli per dimostrare un comportamento hardware, end-to-end o di integrazione reale che non esercitano.

Il revisore deve provare almeno un controesempio plausibile e pertinente al dominio, oltre al percorso di successo nominale. Deve inoltre controllare che i test comprendano i casi negativi e i confini più rilevanti. Per esempio, se il codice deve associare due osservazioni allo stesso dispositivo, verificare un caso in cui un dispositivo sia connesso ma un altro produca l'input; se il codice distingue hardware interno da periferiche esterne, verificare il caso con una periferica esterna compatibile ma hardware interno assente o non riconosciuto. Questi esempi illustrano il metodo: il subagent deve derivare i casi di prova dalla richiesta e dall'implementazione effettive, non copiarli indiscriminatamente in task non pertinenti.

#### 3.6.2.2 Struttura obbligatoria del rapporto di revisione

Il rapporto deve essere abbastanza concreto da permettere all'agente principale di intervenire senza dover indovinare cosa non va. Per modifiche di codice o repository, usa questa struttura, adattandola senza omettere le parti applicabili:

1. **Contesto e stato della versione:** repository, branch, HEAD/commit e base di confronto, file modificati, stato CI e limiti di accesso. Riporta solo dati controllati direttamente.
2. **Sintesi del verdetto:** una frase chiara sul fatto che la modifica sia pronta oppure non sia ancora accettabile, seguita dalla motivazione principale.
3. **Cosa è stato corretto:** cambiamenti positivi dimostrati e relazione con il problema originale. Non confondere un miglioramento parziale con una soluzione completa.
4. **Problemi ancora presenti:** per ogni rilievo, priorità (`BLOCCANTE`, `ALTA`, `MEDIA`, `BASSA`), requisito coinvolto, file/posizione, scenario riproducibile, evidenza, causa, impatto e correzione richiesta.
5. **Integrazione e regressioni:** compatibilità con il progetto, dipendenze e chiamanti coinvolti, possibili effetti collaterali, casi non coperti e rischi residui.
6. **Test e CI:** nome del controllo, stato reale (`PASS`, `FAIL`, `RUNNING`, `NOT RUN`, `UNAVAILABLE` o `NON VERIFICATO`), output pertinente e cosa dimostra effettivamente. Distingui test automatici, test d'integrazione e prove su ambiente/hardware reale.
7. **Piano correttivo e criteri di rivalutazione:** modifiche puntuali richieste, nuovi test da aggiungere, risultati attesi e condizioni oggettive per poter approvare la prossima versione.
8. **Verdetto finale:** `APPROVATA` oppure `MODIFICHE RICHIESTE`, senza formule ambigue.

Per revisioni documentali, di ricerca o di altri artefatti non versionati, conserva lo stesso rigore ma adatta i campi al contesto: provenienza e versione della fonte, requisiti, prove, lacune, integrazione nell'artefatto complessivo e rischi di errore. Non compilare campi relativi a branch o CI quando non pertinenti.

#### 3.6.2.3 Classificazione dei rilievi e raccomandazione di rilascio

Classifica ogni rilievo in base alla gravità reale e all'impatto sui requisiti:

- **BLOCCANTE:** la soluzione viola un requisito fondamentale, può causare danni/perdita di dati o impedisce l'uso previsto;
- **ALTA:** requisito importante non soddisfatto, falso successo plausibile, regressione significativa o comportamento non affidabile nel flusso principale;
- **MEDIA:** difetto circoscritto, caso limite significativo o rischio d'integrazione che necessita di correzione o di una verifica specifica prima dell'approvazione;
- **BASSA:** problema minore che non compromette i requisiti, la sicurezza o l'affidabilità essenziale.

I rilievi `BLOCCANTE` e `ALTA` impediscono sempre l'approvazione. Un rilievo `MEDIA` impedisce l'approvazione quando incide su un requisito, può provocare una regressione significativa o non dispone di una mitigazione verificata. Un rilievo `BASSA` può restare aperto soltanto se non viola la richiesta corrente e il revisore ne spiega l'impatto limitato. Se la modifica è destinata al merge o al rilascio, il revisore deve formulare una raccomandazione esplicita (`NON FARE MERGE/RILASCIARE` oppure `PRONTA PER MERGE/RILASCIO`) distinta dal verdetto di conformità quando i due giudizi non coincidono.

Un rapporto che elenca soltanto gli aspetti corretti, riporta che i test passano o non identifica alcun difetto senza documentare le due passate di revisione, i controesempi considerati e le prove raccolte è **incompleto** e non può giustificare `APPROVATA`.

#### 3.6.3 Verdetto rigoroso e soglia di approvazione

Il subagent deve emettere uno dei due verdetti espliciti seguenti:

- **APPROVATA:** consentito soltanto quando ogni requisito applicabile della richiesta corrente è marcato `PASS` e supportato da evidenze adeguate; entrambe le passate di revisione sono state completate; sono state esaminate implementazione e integrazione nel progetto; è stata svolta una ricerca attiva di controesempi; le regressioni plausibili sono state analizzate; e i controlli pertinenti sono sufficienti per la natura e il rischio della modifica. Il revisore deve spiegare le prove concrete che giustificano l'approvazione e indicare quali controesempi ha verificato.
- **MODIFICHE RICHIESTE:** obbligatorio se anche un solo requisito è `FAIL` o `NON VERIFICATO`, se la richiesta corrente non è stata analizzata integralmente, se una delle due passate è incompleta, se non sono state valutate integrazione e regressioni, se manca la ricerca di controesempi, se mancano verifiche necessarie o se rimane un rischio significativo non risolto. Il revisore deve indicare requisito coinvolto, evidenza o verifica mancante, scenario, impatto e correzione/test necessari.

Non è ammessa un'approvazione condizionata, per impressione generale o basata sulla fiducia nell'agente principale. L'incertezza su un requisito rilevante non equivale a superarlo: deve essere riportata come `NON VERIFICATO` e impedire l'approvazione finché non viene risolta. Se un controllo non è tecnicamente eseguibile, il revisore deve precisare il limite e individuare le prove alternative richieste; non può dichiarare verificato ciò che non ha potuto verificare.

Il rapporto del revisore deve contenere almeno:

- il task corrente revisionato, identificato in modo inequivocabile;
- la matrice completa dei requisiti con stato ed evidenze;
- la valutazione dell'integrazione nel progetto e delle possibili regressioni;
- i difetti, le omissioni, i rischi e le verifiche mancanti, ordinati per gravità;
- le correzioni concrete richieste e i test/controlli necessari a convalidarle;
- il verdetto finale `APPROVATA` oppure `MODIFICHE RICHIESTE`.

#### 3.6.4 Correzioni, rivalutazione e approvazione della versione finale

1. **Applica tutte le revisioni richieste.** L'agente principale deve valutare ogni rilievo e correggere le lacune pertinenti. Non può ignorare unilateralmente un rilievo: se ritiene che non sia applicabile, deve documentare prove e motivazione e sottoporle nuovamente al revisore per una decisione esplicita.
2. **Ripeti i controlli interessati.** Dopo ogni correzione, esegui nuovamente i test e le verifiche influenzati dal cambiamento e raccogli nuove evidenze.
3. **Invia il brief aggiornato.** Al subagent devono essere forniti il medesimo task corrente e i suoi requisiti aggiornati, il diff della versione appena modificata, lo stato di ogni rilievo precedente e le nuove prove. Specifica che deve rivalutare l'intera implementazione contro la richiesta corrente, non soltanto controllare che i suoi commenti precedenti siano stati applicati.
4. **Itera senza scorciatoie.** Ripeti il ciclo di revisione, correzione e nuova valutazione finché il subagent non emette `APPROVATA` per la versione effettivamente presente. Ogni modifica sostanziale successiva invalida l'approvazione precedente e richiede una nuova revisione. Non chiudere il task solo perché la stessa modifica è stata respinta molte volte o perché il revisore ha smesso di segnalare problemi senza emettere il verdetto positivo richiesto.
5. **Rifiuta le approvazioni superficiali.** Se il rapporto si limita a lodare la modifica, ripete quanto dichiarato dall'agente principale, usa documentazione come prova di test eseguiti, non riporta la matrice di conformità, non distingue i test effettivi da quelli mancanti o non analizza difetti residui/regressioni, consideralo incompleto e chiedi una nuova revisione mirata. Quando possibile, usa un subagent diverso se lo stesso revisore ha prodotto ripetutamente rapporti generici o approvazioni non sostenute da evidenze; fornisci comunque il brief corrente e le prove reali, senza influenzare il verdetto desiderato.
6. **Registra l'esito reale.** Nel resoconto finale riepiloga verdetto, rilievi principali, correzioni applicate e controlli effettivi. Non inventare approvazioni né attribuire al revisore verifiche che non ha eseguito.

#### 3.6.5 Responsabilità e indisponibilità del subagent

La revisione deve essere svolta da un subagent distinto dall'agente principale. Il subagent agisce da revisore indipendente; l'agente principale resta responsabile dell'implementazione, della verifica e della coerenza del deliverable. Il parere del subagent non sostituisce test, evidenze o controlli di sicurezza.

Se gli strumenti disponibili non permettono realmente di avviare un subagent, l'agente principale deve dichiarare con precisione il limite e continuare tutte le verifiche autonome possibili, ma **non può considerare il task che contiene la modifica completato né presentarlo come approvato**. Un'autorevisione, una seconda passata interna o una revisione simulata non equivalgono all'approvazione richiesta. Il task resta in stato **IN ATTESA DI REVISIONE DEL SUBAGENT** finché la revisione indipendente non può essere eseguita e conclusa positivamente.

**Criterio vincolante di completamento:** una modifica è completata soltanto quando l'agente principale ha verificato l'implementazione e il subagent ha approvato esplicitamente la stessa versione finale, dopo aver dimostrato la conformità a tutti i requisiti applicabili della richiesta corrente e aver valutato integrazione e regressioni. Un task con una modifica non approvata o non ancora revisionata deve essere segnalato come incompleto, anche se il codice è stato scritto e i test disponibili sono superati.
