# Previsione del Churn dei Clienti Telco

## Panoramica del Progetto
Questo progetto si concentra sulla previsione del churn dei clienti nel settore delle telecomunicazioni utilizzando tecniche di machine learning.

Il churn dei clienti si verifica quando un cliente interrompe l'utilizzo dei servizi di un'azienda. L'obiettivo è **identificare in anticipo i clienti ad alto rischio**, consentendo interventi mirati di retention.

## Presentazione del Progetto
Una presentazione completa del progetto, creata con Gamma, è disponibile al seguente link:

[Visualizza Presentazione](https://gamma.app/docs/Analisi-del-Churn-in-unazienda-Telco-4y40srwy2oq6lfx?mode=doc)

### Contenuti della Presentazione:
- Problema di business e obiettivi
- Pipeline dati end-to-end
- Scelta e valutazione del modello
- Variabili chiave e driver del churn
- Raccomandazioni strategiche

## Contesto di Business
Le aziende di telecomunicazioni affrontano tassi elevati di churn a causa della forte concorrenza. Mantenere i clienti esistenti è molto più conveniente rispetto all'acquisizione di nuovi clienti.

### Obiettivi di Business
- Identificare i clienti a rischio di churn
- Supportare campagne di retention mirate
- Offrire promozioni e sconti personalizzati

### Valore per il Business
- Riduzione del tasso di churn
- Incremento del valore del cliente nel tempo
- Maggiore stabilità dei ricavi

## Dataset
- Fonte: [Telco Customer Churn Dataset (Kaggle)](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)
- Record: 7043 clienti
- Variabili: 21

### Variabili Chiave
| Categoria       | Esempi                                   | Descrizione                          |
|-----------------|-----------------------------------------|--------------------------------------|
| Demografiche    | gender, SeniorCitizen, Partner          | Informazioni sul cliente             |
| Servizi         | PhoneService, InternetService, OnlineSecurity | Servizi sottoscritti          |
| Account         | tenure, Contract, MonthlyCharges, TotalCharges | Informazioni su contratto e fatturato |
| Target          | Churn                                   | Indica se il cliente ha abbandonato (Yes/No) |

## Preparazione dei Dati

### Pulizia dei Dati
- Conversione della colonna `TotalCharges` da stringa a numerica
- Gestione dei valori mancanti (clienti con tenure = 0 → imputazione con 0)

### Codifica
- Applicazione di One-Hot Encoding alle variabili categoriche

## Feature Engineering
Per migliorare le performance del modello, sono state create nuove variabili:

- **Gruppi di Tenure**
  - 0–12 mesi
  - 13–24 mesi
  - 25–48 mesi
  - superiore a 48 mesi
- **Fasce di Costo Mensile**
  - Basso
  - Medio
  - Alto

## Pipeline dei Dati (Dataiku Flow)
Il progetto segue la seguente pipeline:
telco_raw → telco_prepared → telco_features → modello → predizioni


- `telco_prepared`: dataset pulito  
- `telco_features`: dataset arricchito con le feature ingegnerizzate  
- Predizioni generate su:
  - set di test (`telco_test_scored`)  
  - nuovi clienti (`telco_predictions`)  

## Modellazione

### Approccio
- Tipo di problema: Classificazione binaria  
- Suddivisione Train/Test: 80/20  
- Piattaforma: Dataiku AutoML  

### Modelli presi in considerazione
- Regressione Logistica  
- Random Forest  

Modello con le migliori performance:
- Random Forest  

## Performance del Modello

### Metriche
- Accuracy: 70%  
- Precision: 46%  
- Recall: 90%  
- F1-score: 61%  

### ROC Curve
- Buona separazione tra le classi  
- AUC elevata (0.854) → ottima capacità discriminante del modello  

## Ottimizzazione della Soglia (Business-Driven)
La soglia di classificazione è stata impostata a 0.400 (anziché 0.5) per privilegiare la **recall** rispetto alla precision:

- Massimizzare l'identificazione dei churner  
- Accettare un numero maggiore di falsi positivi  

**Ragione di business:** perdere un churner è più costoso che contattare un cliente che non avrebbe abbandonato.

## Analisi della Confusion Matrix

|                | Predetto Yes | Predetto No |
|----------------|--------------|-------------|
| Effettivo Yes  | 330 (TP)     | 37 (FN)     |
| Effettivo No   | 382 (FP)     | 645 (TN)    |

**Insight chiave:** bassi falsi negativi → forte rilevazione dei churner; falsi positivi maggiori → trade-off accettabile.

## Interpretazione del Modello

### Importanza delle Variabili
Le variabili più influenti:
- Tipo di contratto  
- Tenure  
- Servizio Internet  
- Online Security  
- Total Charges  

### Insight Chiave
- Contratti Month-to-Month → rischio di churn più alto  
- Tenure breve (0–12 mesi) → alta probabilità di churn  
- Bollette elevate → correlazione positiva con il churn  
- Utenti Fiber optic → maggiore probabilità di abbandono  
- Assenza di Online Security → rischio aumentato  
- Clienti con lunga tenure (>48 mesi) → più fedeli  

### Insight Comportamentali
- La probabilità di churn diminuisce con l’aumentare della tenure  
- Clienti con servizi base → maggiore rischio  
- Clienti con pacchetti di servizi più completi → più fedeli  

## Analisi Scenario (What-If)
Esempio:
- Cliente con contratto Month-to-Month → rischio alto  
- Cambio contratto:
  - One-year
  - Two-year
- Risultato: riduzione significativa della probabilità di churn  

## Deploy e Predizioni

### Deploy del Modello
- Modello migliore deployato in Dataiku Flow  

### Scoring Nuovi Clienti
- Dataset creato: `telco_future_customers`  
- Applicazione del modello tramite scoring pipeline  

### Output
- `prediction` → classificazione churn  
- `proba_Yes` → probabilità di churn  

## Strumenti e Tecnologie
- Dataiku DSS  
- Machine Learning (Classificazione)  
- AutoML  

## Conclusioni
- Tipo di contratto è il principale driver del churn  
- Clienti all’inizio della tenure sono i più a rischio  
- La scelta della soglia è fondamentale per allineare il modello al business  
- L’interpretazione del modello supporta le decisioni strategiche  

## Miglioramenti Futuri
- Ottimizzazione iperparametri  
- Gestione dello sbilanciamento delle classi  
- Test su modelli aggiuntivi (es. LightGBM)  
- Pipeline di predizione in tempo reale  

## Autore
Lisa Bandinelli 
