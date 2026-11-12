# Incidenti stradali in Italia: dove investire in sicurezza

Capstone Project del Master in Data Analytics di Boolean, consegnato ad agosto 2026.

> **Versione 2, novembre 2026.** Questa è una revisione che ho fatto dopo il corso, per esercitarmi e migliorare. Ho riletto il progetto consegnato, ho trovato degli errori e li ho corretti. In fondo c'è l'elenco di cosa è cambiato e perché. La versione consegnata resta nella storia del repository, al commit `0a4afcc`.

**La domanda:** un'azienda che si occupa di traffico e prevenzione vuole sapere in quali comuni italiani conviene investire in sicurezza stradale. Servono numeri aggiornati, confrontabili tra comuni grandi e piccoli, e una dashboard che racconti il risultato.

**La risposta, in breve:** ho scaricato i dati ISTAT degli incidenti, li ho uniti a popolazione e superficie di ogni comune e ho diviso i 1.202 comuni sopra i 10.000 abitanti in tre profili: **772 a bassa incidentalità, 369 ad alta incidenza, 61 ad alta densità**. Poi li ho messi in ordine con un punteggio di priorità. In testa ci sono Bergamo, Firenze, Genova, Milano e Riccione. Profili e punteggio usano la media degli ultimi tre anni, dal 2023 al 2025.

I numeri sono del 2025, l'ultimo anno pubblicato da ISTAT: 177.207 incidenti con feriti, 2.868 morti, 237.822 feriti.

![Comuni divisi in tre profili](images/cluster-rischio.png)

## Cosa contiene

| File | Cosa c'è |
| --- | --- |
| `main.ipynb` | Tutta l'analisi in Python: download, pulizia, esplorazione, indicatori, clustering, priorità |
| `Dashboard.pbix` | La dashboard Power BI, aggiornata ai dati di questa revisione. Tre pagine: la panoramica, la mappa dei profili e la classifica di priorità (`Where to Invest`), con i filtri per profilo e per regione |
| `Slide deck capstone project.pptx` | Le cinque slide della presentazione, aggiornate ai numeri di questa revisione |
| `data/raw/` | I dati come arrivano da ISTAT e da SITUAS |
| `data/clean/` | I dataset prodotti dal notebook, fino a quello con profili e priorità. Per chi li legge con pandas: la sigla della provincia di Napoli è `NA`, che `read_csv` scambia per un valore mancante se non si passa `keep_default_na=False` |
| `images/` | I grafici salvati dal notebook. Due sono in questa pagina |
| `requirements.txt` | Le librerie usate dal notebook, con le versioni con cui è stato eseguito |

## Come ho lavorato

1. **Dati ISTAT scaricati in automatico.** Il notebook chiama l'endpoint SDMX di ISTAT. Il server è lento, quindi chiede solo gli anni dall'ultimo già salvato in poi e li aggiunge agli altri. Ogni risposta viene controllata prima di scrivere: se il download fallisce, l'analisi riparte dal file salvato e lo dice.
2. **Tre misure per comune e per anno.** Dal dataset tengo gli incidenti (`ROADACC`), i morti e i feriti (`KILLINJ`), dal 2001 all'ultimo anno disponibile.
3. **Unione con SITUAS.** ISTAT non ha né il nome del comune né la popolazione. Li prendo da SITUAS insieme alla superficie, alla data dell'anno degli incidenti. L'unione parte dall'elenco dei comuni di SITUAS e non dalle righe degli incidenti: negli ultimi anni ISTAT non riporta i comuni senza incidenti, e così restano nell'analisi con uno zero. Dopo l'unione controllo quali codici ISTAT non hanno trovato il loro comune. In questa esecuzione nessuno.
4. **Indicatori.** Incidenti ogni 100.000 abitanti, per confrontare comuni di dimensioni diverse. Incidenti per chilometro quadrato, per misurare quanto sono concentrati sul territorio. In più le vittime, cioè morti e feriti insieme, ogni 100.000 abitanti, gli stessi due indicatori sulla media degli ultimi tre anni e la variazione rispetto al 2017-2019, i tre anni prima della pandemia.
5. **Solo i comuni sopra i 10.000 abitanti.** Nei paesi piccoli bastano pochi incidenti per far schizzare il tasso, e nel notebook c'è il grafico che lo mostra. Sopra la soglia il tasso di un anno somiglia a quello dell'anno prima (correlazione 0,88). Questi 1.202 comuni fanno l'80,8% degli incidenti del 2025.
6. **Nessun comune tolto.** Le grandi città hanno valori molto alti ma veri: restano nell'analisi.
7. **K-Means in tre profili, sulla media di tre anni.** Con un anno solo circa un comune su sette cambia profilo da un anno all'altro, con la media di tre anni circa uno su sedici. Prima porto i due indicatori sulla stessa scala con `StandardScaler`, poi scelgo il numero di gruppi guardando il gomito e la silhouette. La silhouette da sola direbbe due gruppi (0,53 contro 0,46): ne tengo tre perché il terzo, quello ad alta densità, ha un significato per chi deve investire. Do un nome ai gruppi in base a come sono fatti: bassa incidentalità, alta incidenza per abitante, alta densità.
8. **Punteggio di priorità.** Per ogni comune la media delle sue due posizioni in classifica, da 0 a 100. È una regola semplice e dichiarata, che chiunque può rifare. Ho provato a spostare il peso su una misura e poi sull'altra: in tutti e due i casi 13 dei primi 15 restano gli stessi.
9. **Dove gli incidenti crescono.** Per ogni comune confronto la media degli ultimi tre anni con quella del 2017-2019. In Italia il numero è quasi fermo (-0,5%). Nel notebook c'è la tabella dei dieci cresciuti di più tra i cento con la priorità più alta, con accanto la serie anno per anno: una percentuale da sola può ingannare, se uno degli anni di partenza è anomalo.

![I quindici comuni con il punteggio di priorità più alto](images/priorita.png)

## Strumenti

Python (pandas, NumPy, requests, Matplotlib, seaborn, scikit-learn), Jupyter, Power BI, Git.

## Limiti e cosa farei dopo

- **I residenti non sono chi sta sulla strada.** Il tasso divide per chi abita nel comune, ma gli incidenti li fa anche chi passa: turisti, pendolari, traffico autostradale. Una località di mare o un comune attraversato da un'autostrada possono salire in classifica per questo. Con questi dati non si distingue: servirebbero i volumi di traffico o gli incidenti per tipo di strada.
- **Quanti incidenti, non quanto gravi.** Morti e feriti per abitante seguono da vicino il tasso di incidenti dello stesso anno (correlazione 0,96), quindi qui cambiano poco il quadro. I morti nel singolo comune sono numeri piccoli e vanno letti su più anni.
- **La popolazione è del 2024, gli incidenti del 2025.** È quella che SITUAS restituisce chiedendo i comuni al 31 dicembre 2025.
- **I gruppi non sono separati in modo netto.** La silhouette con tre gruppi è 0,46: un comune vicino al confine tra due profili potrebbe stare da una parte o dall'altra. Spostando di un anno la finestra dei tre anni, 77 comuni su 1.202 cambiano profilo.
- **La variazione conta gli incidenti, non il tasso.** Un comune che ha guadagnato abitanti può crescere solo per quello, e con numeri piccoli una percentuale salta facilmente. Per cinque comuni non si calcola: quattro hanno un codice più recente del 2017, come i comuni nati da una fusione, e uno non ha incidenti nel 2017-2019.
- **Due comuni con numeri molto bassi.** Riesi e Mazzarino, in provincia di Caltanissetta, hanno più di 10.000 abitanti e pochissimi incidenti: dal 2017 Riesi ne ha da zero a due l'anno. Mazzarino ne ha da zero a quattro, e nel 2025 non compare nemmeno nei dati. Li ho lasciati come arrivano da ISTAT, ma per due comuni di quella grandezza sono numeri strani: il dato alla fonte potrebbe essere incompleto, e da qui non posso verificarlo.
- **Previsioni.** La traccia lasciava aperta l'idea di prevedere gli incidenti degli anni successivi. La variazione rispetto al 2017-2019 dice dove sono cresciuti, non dove cresceranno: con la serie storica dei comuni si può provare una regressione.

## Cosa è cambiato rispetto alla consegna

| Nella versione consegnata | In questa revisione | Perché |
| --- | --- | --- |
| Toglievo come valori estremi i comuni oltre 800 incidenti ogni 100.000 abitanti o 15 per chilometro quadrato | Nessun comune tolto | Uscivano 19 comuni tra cui Milano, Genova, Torino, Napoli e Firenze: proprio alcune delle città dove un investimento conta di più |
| K-Means sui due indicatori senza portarli sulla stessa scala | Indicatori standardizzati, numero di gruppi controllato con gomito e silhouette | Senza scala comune i gruppi erano uguali a quelli fatti con il solo tasso per abitante: la densità non contava |
| Tre classi: basso, medio, alto | Tre profili: bassa incidentalità, alta incidenza, alta densità. Più un punteggio di priorità | Ora i gruppi usano tutte e due le misure, e la priorità dice da dove cominciare |
| Solo il numero di incidenti | Anche morti e feriti | Erano già nel file ISTAT |
| Solo l'ultimo anno | Profili e priorità sulla media di tre anni, andamento dal 2001, confronto con il 2017-2019 | Un anno solo sposta troppi comuni da un profilo all'altro, e serve sapere anche dove gli incidenti crescono |
| Download con 5 secondi di attesa, che gestiva solo il tempo scaduto | Download a blocchi di anni, con controllo della risposta | Nelle mie prove di novembre il server ISTAT ha risposto in un tempo fra 27 e 69 secondi per un blocco di anni, e una volta non ha risposto per cinque minuti: con tempi così 5 secondi non bastano |
| Popolazione SITUAS del 2020, file scaricato a mano | Popolazione scaricata in automatico alla data degli incidenti | Tassi più precisi, e nessun comune resta senza nome |
| Dashboard con i titoli degli assi scambiati nel grafico a dispersione | Titoli corretti | Erano invertiti |
| Dashboard a due pagine, con i primi dieci comuni per tasso di incidenti | Tre pagine: i primi dieci per punteggio di priorità, il filtro per regione e una pagina nuova con la classifica completa e l'andamento dal 2019 | La dashboard deve raccontare la stessa risposta del notebook: da dove cominciare |
| Notebook consegnato senza rieseguirlo da capo | Eseguito dall'inizio alla fine prima di salvarlo | Le uscite devono essere quelle del codice che si legge |

## Come provarlo

```bash
git clone https://github.com/francesco-mauro/capston_project_final.git
cd capston_project_final
pip install -r requirements.txt
jupyter notebook main.ipynb
```

Il download da ISTAT può richiedere da un minuto a diversi minuti: il server è lento e il notebook ritenta da solo. La dashboard si apre con Power BI Desktop. Legge due file di `data/clean/` da un percorso fisso del mio PC: per aggiornare i dati va cambiato nelle origini dati (Transform data, Data source settings). I numeri seguono la lingua del PC di chi guarda: in italiano i decimali hanno la virgola.

---

La traccia originale del Master è in [`Capstone Project README.md`](Capstone%20Project%20README.md).
