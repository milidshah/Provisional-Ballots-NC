# Attribution

## Data Sources

### Provisional Ballot Data
- **North Carolina State Board of Elections — Provisional Files**
  - Portal: https://www.ncsbe.gov/results-data/absentee-and-provisional-data
  - Data Dictionary: https://s3.amazonaws.com/dl.ncsbe.gov/ENRS/layout_provisional.txt
  - 10 elections downloaded as `.txt` files directly from the NCSBE S3 bucket, covering 10 elections from September 2023 through March 2026. Files were converted to CSV and stored in the `data/` directory via `data_preparation.ipynb`.

### Election & Voting Information (RAG Chatbot)
- **North Carolina State Board of Elections — Website**
  - https://www.ncsbe.gov/
  - 19 pages scraped from ncsbe.gov covering voter registration, Voter ID requirements, early voting, provisional voting, polling place data, and election types. Scraped using BeautifulSoup in `ncsbe_rag_chatbot.ipynb`.

---

## Libraries

### Data & ML (`provisional_ballot_model.ipynb`)
| Library | Version | Use |
|---|---|---|
| pandas | 2.2.2 | Data loading, merging, and manipulation |
| numpy | 2.0.2 | Numerical operations and array handling |
| matplotlib | 3.10.0 | Visualizations, confusion matrices, feature importance plots |
| scikit-learn | 1.6.1 | Preprocessing, train/test split, logistic regression, dummy classifier, pipelines, metrics |
| xgboost | 3.2.0 | XGBoost classifier |
| imbalanced-learn | 0.14.1 | SMOTE oversampling for class imbalance |
| umap-learn | 0.5.12 | UMAP dimensionality reduction for nonlinear structure visualization |

### RAG Chatbot (`ncsbe_rag_chatbot.ipynb`)
| Library | Version | Use |
|---|---|---|
| langchain | 1.2.15 | RAG chain orchestration |
| langchain-core | 1.2.28 | Prompts, runnables, output parsers |
| langchain-community | 0.4.1 | HuggingFace embeddings, FAISS vector store integration |
| langchain-text-splitters | 1.1.2 | Recursive character text splitting for document chunking |
| langchain-google-genai | 4.2.2 | Gemini LLM integration |
| google-generativeai | 0.8.6 | Google Generative AI Software Development Kit (SDK) |
| faiss-cpu | 1.13.2 | Local vector store for semantic retrieval |
| sentence-transformers | 5.4.0 | HuggingFace sentence embeddings (all-MiniLM-L6-v2) |
| beautifulsoup4 | 4.13.5 | Web scraping of ncsbe.gov pages |
| requests | 2.32.4 | HTTP requests for web scraping |

---

## AI Usage

Claude & ChatGPT was used as a development assistant throughout this project. The following describes what was generated, what was modified, and what required substantial rework:

### What AI assisted with
- **Boilerplate and scaffolding**: initial structure of the evaluate_model() utility function, the ColumnTransformer/Pipeline setup for one-hot encoding, and the RandomizedSearchCV block were drafted with AI assistance and then modified to fit the specific dataset and experimental design.
- **Debugging**: AI assisted in diagnosing issues with date parsing (the `01/10/4125` invalid date row), OneHotEncoder `sparse_output` deprecation, and UMAP scaling decisions.
- **Documentation**: markdown cell descriptions, inline code comments, and this ATTRIBUTION.md were drafted with AI assistance and reviewed and edited by the author.
- **Formatting**: requirements.txt, README.md, ATTRIBUTION.md, and SETUP.md were formatted and edited by ChatGPT.

### What was designed and implemented independently
- The overall experimental design: stratified split strategy, shared index reuse across all models, class imbalance comparison framework (3 strategies), and the model progression from dummy → logistic regression → tree models → tuned → feature-selected.
- Feature engineering decisions: the election type taxonomy, `days_diff_vote_election`, impediment flag groupings, `pv_voted_reason` policy categories, and the race × provisional reason interaction terms were all motivated by subject-matter knowledge of voting rights.
- Target encoding strategy for county-level approval rates and the leakage prevention design (fit on train fold only).
- The 95% cumulative importance threshold for feature selection.

### What required rework or debugging after AI generation
- The RAG chain required multiple iterations to get question contextualization working correctly with chat history — the initial draft did not correctly pass history through the standalone question reformulation step.
- The shared train/test index design required rethinking after the initial approach applied transformations before splitting, introducing potential leakage.

---

## References

### Provisional Ballot Feature Engineering
- Alvarez, R. M., Bailey, D., & Katz, J. N. (2008). *The Effect of Voter Identification Laws on Turnout*. California Institute of Technology Social Science Working Paper. — Informed the ID issue feature groupings and the `pvr_id_exception_claimed` flag.
- Foley, E. B. (2008). *The Analysis of Election Law*. — Background on provisional ballot adjudication standards, including the precinct vs. jurisdiction dispute distinction.
- Voting rights literature on racial disparities in provisional ballot resolution informed the inclusion of race × provisional reason interaction terms (see comment in notebook Section 6c).

### Machine Learning Methods
- documentation + additional resources on using SMOTE and threshold tuning to address class imbalance
- scikit learn documentation
- tutorial for using UMAP and interpeting it

### RAG Chatbot
- LangChain documentation
- Documentation on how to use Gemini API keys
