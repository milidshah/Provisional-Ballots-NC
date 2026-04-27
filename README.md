# Provisional Ballots & Election Access in North Carolina
A machine learning pipeline to predict whether a provisional ballot will be counted or rejected, trained on 10 elections of North Carolina State Board of Elections data. Includes a conversational RAG chatbot that answers questions about NC voting rules and provisional ballot procedures.

## What it Does
This project combines predictive modeling and natural language AI to analyze provisional ballot outcomes in North Carolina. The ML pipeline takes raw provisional ballot records from 10 NCSBE elections (September 2023 – March 2026), engineers policy-motivated features — including election type, days between vote and election date, provisional ballot reason categories, reasonable impediment flags, and race × reason interaction terms — and trains XGBoost and Random Forest classifiers with class imbalance handling, hyperparameter tuning, and importance-based feature selection. Separately, a LangChain RAG chatbot scrapes 19 pages from ncsbe.gov, indexes them into a FAISS vector store using sentence embeddings, and powers a multi-turn conversational interface via the Gemini LLM — letting users ask plain-English questions about voter registration, Voter ID requirements, early voting, and provisional ballot procedures.

## Quick Start
All notebooks run on **Google Colab**. No local installation required.

**Step 1: Clone the repo and install dependencies**
```python
!git clone https://github.com/milidshah/Provisional-Ballots-NC.git
%cd Provisional-Ballots-NC
!pip install -r requirements.txt
```

**Step 2: ML Model** — open `provisional_ballot_model_final.ipynb` and run all cells top to bottom to train model and view results.  
⚠️ The hyperparameter tuning cell takes **10–15 minutes** (~150 fits). Set `N_ITER = 10` to speed it up.


**Step 3: RAG Chatbot** — open `ncsbe_rag_chatbot.ipynb`. Requires a free Gemini API key:
1. Get one at https://aistudio.google.com/app/apikey
2. In Colab, open **Secrets** (🔑 sidebar) → Add secret named `GOOGLE_API_KEY` → toggle ON
3. Run all cells. Type questions at the chat prompt, `quit` to exit.
  - example questions:

```
What ID do I need to vote in North Carolina?
What is a provisional ballot?
How do I register to vote?
```

> See [`SETUP.md`](SETUP.md) for full setup details including API key configuration and runtime tips.

## Video Links
Project Demo: https://drive.google.com/file/d/1OfxQgMTgu-X2P3ESUPkWODZgImdEDZjV/view?usp=sharing

Technical Walkthrough: https://drive.google.com/file/d/1bDI1w8QrmL6fXAkhvD7EYEdUDZQPJW3w/view?usp=sharing

## Evaluation
present quantitative results, accuracy metrics, qualitative outcomes from testing

## Individual Contribution: Mili Shah

## Data Sources
North Carolina State Board of Elections Provisional Files: https://www.ncsbe.gov/results-data/absentee-and-provisional-data#Whatisthebestwaytojoinanabsenteefilewithanotherfile-1160
  - Data Dictionary: https://s3.amazonaws.com/dl.ncsbe.gov/ENRS/layout_provisional.txt



## Evaluation

### ML Model — Final Model Comparison

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| Dummy (majority class) | 0.6266 | 0.0000 | 0.0000 | 0.0000 | — |
| XGBoost (default) | 0.7450 | 0.6575 | 0.6620 | 0.6597 | 0.8054 |
| Random Forest (default) | 0.7335 | 0.6439 | 0.6404 | 0.6421 | 0.7796 |
| XGBoost (tuned) | 0.7506 | 0.6675 | 0.6616 | 0.6645 | 0.8092 |
| **XGBoost (tuned + selected features)** | **0.7523** | **0.6689** | **0.6664** | **0.6677** | **0.8096** |

**XGBoost (tuned + selected features) — per-class breakdown:**
```
              precision    recall  f1-score   support
 Not Counted       0.80      0.80      0.80     15098
    Approved       0.67      0.67      0.67      8997
    accuracy                           0.75     24095
   macro avg       0.74      0.73      0.74     24095
weighted avg       0.75      0.75      0.75     24095
```

**Key findings:**
- The final model improves on the dummy baseline by +12.6 points in accuracy (0.75 vs 0.63) and +0.81 in ROC-AUC
- XGBoost consistently outperforms Random Forest across all metrics (F1 0.66 vs 0.64, ROC-AUC 0.81 vs 0.78)
- Tuning and feature selection together produce the best model — marginal but consistent gains over the default (F1 +0.008, ROC-AUC +0.004) with faster training (4.5s vs 3.7s comparable)
- The Approved class remains harder to predict (F1 0.67) than Not Counted (F1 0.80), reflecting genuine outcome variability for approved ballots

### RAG Chatbot — Qualitative Evaluation

The chatbot correctly attributes answers to retrieved NCSBE source pages and declines to speculate when information is not in the indexed documents. Multi-turn context is maintained for at least 4 turns via question reformulation, allowing follow-up questions like "What about for absentee ballots?" to resolve correctly.

## Individual Contributions

This was an **individual project** completed by **Mili Shah**.

All components — data collection, feature engineering, model design, class imbalance handling, hyperparameter tuning, feature selection, RAG pipeline, web scraping, and documentation — were designed and implemented independently.

