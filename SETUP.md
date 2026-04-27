# Setup Instructions

All notebooks in this project run on **Google Colab** (but can also be downloaded and run as a jupyter notebook). No local installation is required. Each notebook is self-contained and installs its own dependencies. Follow the steps below for each notebook.

---

## Repository

The `data/` folder contains all 10 provisional ballot CSV files already committed. **You do not need to run `data_preparation.ipynb` to use the ML model — the data is already in the repo.**

---

## Data Preparation Notebook: (`data_preparation.ipynb`)

> **Skip this notebook if you just want to run the ML model.** The CSVs it produces are already committed to `data/`. Only run this if you want to re-download raw data from NCSBE.

### Steps
1. Open `data_preparation.ipynb` in Google Colab
2. Run the git config cell (cell 1) — no changes needed
3. **Skip the git clone cell** (cell 2) — it is marked "comment after running once" and the repo is already cloned in Colab's working directory if you opened from GitHub. If running fresh, uncomment and run it, then enter your GitHub personal access token when prompted
4. Run the data download cell (cell 3) — this fetches 10 `.txt` files directly from the NCSBE S3 bucket. No API key needed; files are publicly accessible
5. Run the save & push cell (cell 4) — saves CSVs to `data/` and pushes to GitHub. Requires a GitHub personal access token with `repo` write access when prompted

### Requirements
- GitHub personal access token (only needed if re-pushing data to the repo)
- No other API keys required

---

##  ML Model Notebook: (`provisional_ballot_model.ipynb`)

> This is the main notebook. It trains and evaluates all models on the provisional ballot data.

### Steps
1. Open `provisional_ballot_model.ipynb` in Google Colab
2. Run the git clone cell at the top to pull the repo and data into Colab:
   ```python
   !git clone https://github.com/milidshah/Provisional-Ballots-NC.git
   %cd Provisional-Ballots-NC
   ```
   When prompted for a GitHub token, enter one with `repo` read access (or make the repo public and skip the token step)
3. Run all cells top to bottom — all dependencies (`xgboost`, `imbalanced-learn`, `umap-learn`) are installed via `!pip install` cells at the top of the notebook
4. The full pipeline takes approximately **15–25 minutes** end to end on a standard Colab CPU runtime. The hyperparameter tuning cell (Section 11) is the longest step (~10–15 min for 150 fits). You can reduce `N_ITER = 30` to `N_ITER = 10` for a faster run.

### Runtime recommendation
- CPU runtime is sufficient. GPU is not required.
- If Colab disconnects during tuning, re-run from the tuning cell — all earlier cells produce deterministic output with `random_state=372`

### Requirements
- No API keys required
- GitHub access to clone the repo (public repo = no token needed)

---

## RAG Election Chatbot Notebook: (`ncsbe_rag_chatbot.ipynb`)

> This notebook builds a conversational chatbot about NC voting information powered by Gemini and LangChain RAG. It requires a Google Gemini API key.

### Getting a Google Gemini API Key
1. Go to https://aistudio.google.com/app/apikey
2. Sign in with a Google account
3. Click **Create API key**
4. Copy the key — you will paste it into Colab Secrets (see below)

> The Gemini API has a **free tier** that is sufficient to run this notebook. No billing setup required.

### Adding the API Key to Colab
1. In your Colab notebook, click the **key icon** in the left sidebar to open **Secrets**
2. Click **Add new secret**
3. Set the name to exactly: `GOOGLE_API_KEY`
4. Paste your API key as the value
5. Toggle **Notebook access** to ON

### Steps
1. Open `ncsbe_rag_chatbot.ipynb` in Google Colab
2. Add your `GOOGLE_API_KEY` to Colab Secrets as described above
3. Run cell 1 — installs all dependencies (`langchain`, `faiss-cpu`, `sentence-transformers`, etc.). This takes ~2 minutes
4. Run cell 2 — loads the API key from Colab Secrets
5. Run cell 3 — scrapes 19 pages from ncsbe.gov. Takes ~30 seconds with polite crawl delays
6. Run cell 4 — splits documents into chunks
7. Run cell 5 — downloads the `all-MiniLM-L6-v2` embedding model (~90MB on first run) and builds the FAISS index
8. Run cells 6–7 — builds the conversational RAG chain
9. Run the chat cell (cell 8) — starts an interactive session. Type questions and press Enter. Type `quit` to exit.

### Example questions to test the chatbot
```
What ID do I need to vote in North Carolina?
How do I register to vote?
What is a provisional ballot?
When is early voting available?
```

### Requirements
- Google Gemini API key (free tier is sufficient)
- Internet access for scraping ncsbe.gov and downloading the embedding model
- No GPU required

---

## Environment & Dependencies

All package versions are pinned in `requirements.txt`. These are pre-installed on Colab or installed by the notebooks themselves — you do not need to run `pip install -r requirements.txt` manually unless running locally.

To run locally (advanced):
```bash
git clone https://github.com/milidshah/Provisional-Ballots-NC.git
cd Provisional-Ballots-NC
pip install -r requirements.txt
jupyter notebook
```

Note: local runs require manually setting `GOOGLE_API_KEY` as an environment variable for the RAG notebook:
```bash
export GOOGLE_API_KEY=your_key_here
```
And replacing the `userdata.get()` call in cell 2 with `os.environ.get('GOOGLE_API_KEY')`.

