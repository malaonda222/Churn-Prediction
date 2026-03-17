# Telco Customer Churn Prediction

> Progetto di Machine Learning per la previsione del rischio di abbandono clienti nel settore delle telecomunicazioni, sviluppato su **Dataiku DSS**.
> Questo progetto si concentra sulla previsione del churn dei clienti nel settore delle telecomunicazioni utilizzando tecniche di machine learning.
> Il churn dei clienti si verifica quando un cliente interrompe l'utilizzo dei servizi di un'azienda. L'obiettivo è **identificare in anticipo i clienti ad alto rischio**, consentendo interventi mirati di retention.

--- 

## Presentazione del Progetto
Una presentazione completa del progetto, creata con Gamma, è disponibile al seguente link:

[Visualizza Presentazione](https://gamma.app/docs/Analisi-del-Churn-in-unazienda-Telco-4y40srwy2oq6lfx?mode=doc)
---

## Panoramica del Progetto

| Attributo | Dettaglio |
|---|---|
| **Settore** | Telecomunicazioni |
| **Tipo di Problema** | Classificazione binaria |
| **Target** | `Churn` (Yes / No) |
| **Strumento** | Dataiku DSS |
| **Algoritmi** | Random Forest, XGBoost, Logistic Regression |

### Scenario Aziendale

Una società di telecomunicazioni (Telco) sta affrontando un problema diffuso nel settore: la perdita di clienti (**churn**). L'obiettivo è costruire un modello predittivo in grado di identificare con anticipo quali clienti sono più propensi ad abbandonare il servizio, per consentire interventi mirati prima che sia troppo tardi.

### Decisioni Supportate dal Modello

- **Campagne di Retention** — Identificare i clienti a rischio per includerli in campagne di fidelizzazione personalizzate.
- **Sconti e Offerte Mirate** — Proporre upgrade o sconti ai clienti con alta probabilità di abbandono.

### Valore per il Business

Mantenere un cliente esistente è quasi sempre meno costoso che acquisirne uno nuovo. Una riduzione anche piccola del tasso di churn può avere un impatto significativo sui ricavi e sulla stabilità del business.

---

## Dataset

Il progetto utilizza il noto dataset **"Telco Customer Churn"** di IBM, ampiamente utilizzato per problemi di classificazione.

- [Kaggle — Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)
- **~7.000 righe**, **21 variabili**

### Variabili Principali

| Categoria | Variabili | Descrizione |
|---|---|---|
| Dati Demografici | `gender`, `SeniorCitizen`, `Partner` | Informazioni anagrafiche sul cliente |
| Servizi Sottoscritti | `PhoneService`, `InternetService`, `OnlineSecurity` | Dettagli sui servizi attivi |
| Dati Contabili | `tenure`, `Contract`, `MonthlyCharges`, `TotalCharges` | Anzianità, contratto e importi fatturati |
| **Target** | **`Churn`** | Se il cliente ha abbandonato il servizio (`Yes` / `No`) |

---

## Pipeline e Flow

Il Flow in Dataiku rappresenta l'intera pipeline end-to-end, dalla raccolta dei dati grezzi fino alle previsioni sui nuovi clienti.

```
telco_future_customers ──────────────────────────────────────────────────────────┐
                                                                                  ▼
Telco_Raw → [Prepare] → Telco_prepared → [Prepare] → Telco_features → [Predict Churn] → telco_predictions
                                                              │
                                                              └─────────────────────────→ Telco_test_scored
```

**Dataset prodotti nel Flow:**

| Dataset | Descrizione |
|---|---|
| `Telco_Raw` | Dataset grezzo importato da Kaggle |
| `Telco_prepared` | Dati puliti e con encoding delle variabili categoriche |
| `Telco_features` | Dataset con feature engineering applicato |
| `telco_train` / `telco_test` | Split 80/20 per training e valutazione |
| `telco_future_customers` | Nuovi clienti da scorare |
| `telco_predictions` | Output finale con probabilità di churn |

---

## Preparazione dei Dati

A partire dal dataset grezzo `Telco_Raw`, è stata creata una **Prepare Recipe** per generare `Telco_prepared`.

### Sfide Affrontate

**1. Gestione dei valori mancanti in `TotalCharges`**

La colonna `TotalCharges`, sebbene numerica, era interpretata come stringa a causa di spazi vuoti per i clienti con `tenure = 0` (clienti nuovi, senza ancora addebiti). Il processo applicato:

- Conversione degli spazi vuoti in veri `missing values`
- Imputazione con `0` (nessun addebito per clienti nuovi)
- Conversione della colonna in tipo numerico

**2. Encoding delle Variabili Categoriche**

Applicato **Dummy Encoding** (One-Hot Encoding) su tutte le colonne testuali (`Contract`, `InternetService`, `OnlineSecurity`, ecc.) per renderle interpretabili dagli algoritmi di ML.

---

## Feature Engineering

A partire da `Telco_prepared`, è stato creato il dataset `Telco_features` con le seguenti nuove variabili:

| Nuova Feature | Descrizione | Encoding |
|---|---|---|
| `tenure_group` | Fascia di anzianità del cliente: 0–12, 13–24, 25–48, >48 mesi | One-Hot Encoding |
| `monthly_cost_bucket` | Fascia di spesa mensile: Bassa, Media, Alta | One-Hot Encoding |

Queste feature aiutano il modello a generalizzare meglio rispetto ai valori continui originali, catturando pattern di comportamento per gruppi di clienti.

---

## Modellazione

### Divisione del Dataset

Split **80% training / 20% test** applicato su `Telco_features` tramite Split Recipe.

### Algoritmi Addestrati

Tramite la funzionalità **AutoML Prediction** di Dataiku, con `Churn` come variabile target:

- Logistic Regression
- Random Forest *(modello selezionato)*
- XGBoost *(performance comparabile)*

---

## Risultati e Performance

### Curva ROC

La curva ROC si discosta nettamente dalla diagonale casuale, confermando un'ottima capacità discriminativa del modello. L'AUC dimostra che il modello riesce a separare efficacemente i clienti che fanno churn da quelli che restano.

### Matrice di Confusione (soglia = 0.40)

La soglia di classificazione è stata abbassata a **0.40** (default 0.50) per privilegiare la **Recall** rispetto alla Precision: in un contesto di retention, è preferibile contattare qualche falso positivo piuttosto che perdere un vero churner.

|  | Predicted Yes | Predicted No | Totale |
|---|---|---|---|
| **Actually Yes** | 330 | 37 | 367 |
| **Actually No** | 382 | 645 | 1027 |
| **Totale** | 712 | 682 | 1394 |

### Metriche Principali

| Metrica | Valore |
|---|---|
| **Accuracy** | 70% |
| **Precision** | 46% |
| **Recall** | **90%** |
| **F1-Score** | 61% |

> La Recall del **90%** è il dato più rilevante: il modello identifica correttamente 9 churner su 10, consentendo all'azienda di intervenire sulla grande maggioranza dei clienti a rischio.

### Distribuzione delle Probabilità

Il modello assegna probabilità basse ai clienti che non abbandonano e probabilità più alte a chi effettivamente fa churn. Esiste una zona intermedia (0.30–0.60) con maggiore incertezza, tipica dei problemi di churn reali, dove i clienti presentano caratteristiche ambigue.

---

## Interpretabilità del Modello

### Feature Importance (Top 10)

| Feature | Importanza | Interpretazione |
|---|---|---|
| `Contract - Month-to-month` | **18%** | Contratto mensile = massimo rischio churn |
| `Contract - Two years` | 12% | Contratto biennale = forte fattore protettivo |
| `IS - Fiber optic` | 12% | Fibra ottica associata a maggiore insoddisfazione |
| `OS - No` | 8% | Assenza di Online Security aumenta il rischio |
| `TG - 0-12 mesi` | 7% | Clienti nuovi molto vulnerabili |
| `TotalCharges` | 7% | Proxy della fedeltà (correlato a tenure >48 mesi) |
| `IS - No` | 5% | Assenza di internet riduce il rischio |
| `MonthlyCharges` | 5% | Bolletta alta associata a più churn |

### Grafico SHAP — Feature Effects

L'analisi SHAP conferma e approfondisce la feature importance:

- **Contract Month-to-month** (valore alto) spinge fortemente verso il churn
- **Contract Two years** (valore alto) riduce significativamente la probabilità di abbandono
- **IS - Fiber optic** e **OS - No** aumentano il rischio di churn
- **TotalCharges** alti (clienti fedeli da lungo tempo) abbassano il rischio
- **Tenure 0–12 mesi** ha un forte impatto positivo sul rischio di abbandono

### What-If Analysis — Impatto del Tipo di Contratto

Simulando lo stesso cliente con tre diversi tipi di contratto, si osserva come il tipo di contratto sia la leva più potente per la retention:

| Tipo di Contratto | Probabilità Churn | Previsione |
|---|---|---|
| Month-to-month | **44.20%** | Yes |
| One year | 24.67% | No |
| Two years | **13.41%** | No |

> Convertire un cliente da mensile a biennale riduce la probabilità di churn di oltre **30 punti percentuali**.

---

## Previsioni su Nuovi Clienti

Il modello è stato deployato nel Flow di Dataiku. Tramite la **Score Recipe**, è possibile applicare il modello a qualsiasi nuovo dataset di clienti. L'output `telco_predictions` contiene:

- Le colonne originali del cliente
- `proba_Yes` — probabilità di churn (0–1)
- `prediction` — previsione finale (`Yes` / `No`)

---

## Insight di Business

### Profilo Cliente ad Alto Rischio Churn 

- Contratto **mensile** (Month-to-month)
- Connessione **fibra ottica**
- **Nessun servizio** di sicurezza online (Online Security = No)
- Cliente da **0–12 mesi**

### Profilo Cliente Fedele / Basso Rischio 

- Contratto **biennale** (Two years)
- **Nessun servizio internet** aggiuntivo
- Cliente da **oltre 48 mesi**

### Raccomandazioni Operative

1. **Incentivare l'upgrade contrattuale** — Offrire vantaggi economici ai clienti mensili per passare a contratti annuali o biennali. La What-If Analysis dimostra che questa è la leva più impattante (-30pp di probabilità churn).
2. **Onboarding mirato nei primi 12 mesi** — I clienti nuovi sono i più vulnerabili: investire in un'esperienza di onboarding di qualità e nel monitoraggio proattivo della soddisfazione.
3. **Bundle servizi di sicurezza** — Includere Online Security nei pacchetti base per i clienti fibra, riducendo uno dei fattori di rischio più rilevanti.
4. **Segmentazione per campagne retention** — Utilizzare il modello in produzione per generare score periodici e prioritizzare le azioni di retention sui clienti con `proba_Yes > 0.40`.

---

## Stack Tecnologico

| Strumento | Utilizzo |
|---|---|
| **Dataiku DSS** | Orchestrazione pipeline ML end-to-end |
| **AutoML** | Addestramento e confronto automatico dei modelli |
| **SHAP** | Interpretabilità e spiegazione delle previsioni |
| **Random Forest / XGBoost** | Algoritmi di classificazione |

---

## Miglioramenti Futuri
- Ottimizzazione iperparametri  
- Gestione dello sbilanciamento delle classi  
- Test su modelli aggiuntivi (es. LightGBM)  
- Pipeline di predizione in tempo reale  

## Autore
Lisa Bandinelli 





