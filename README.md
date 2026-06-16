# 📊 Market-Pulse: E-Commerce Sentiment & Business Analytics

Market-Pulse is an interactive, Streamlit-powered dashboard designed to help e-commerce store owners and analysts extract actionable business intelligence from customer reviews. 

The application blends **local machine learning (fast & resource-friendly)** for sentiment classification with a **generative AI-powered Consultant (Google Gemini API)** using a Retrieval-Augmented Generation (RAG) approach to draft detailed business reports, all protected by custom local safety guardrails.

---

## ✨ Key Features & Functionality

### 1. Interactive Business Intelligence Dashboard
* **Product Insights**: Visualize the top 10 most popular products based on customer upvotes.
* **Customer Loyalty**: Analyze review distribution across departments with interactive pie charts.
* **Market Segmentation**: Track purchase volume and ratings across different age groups.
* **Pain Point Identification**: Spot categories with high complaint counts using automatic defect rate calculations.

### 2. Live Semantic Search & Filtering
* Filter and search through review databases in real-time.
* Look up specific keywords (e.g., "fabric", "size", "delivery") alongside star ratings to drill down into specific customer experiences.

### 3. Smart Dataset Uploader
* Drag-and-drop custom `.csv` or `.xlsx` files directly.
* **Auto-Detection**: Dynamically detects the text column and rating column using intelligent keyword matching.
* **Validation & Cleaning**: Automatically handles null values, drops duplicate reviews, filters out invalid rows, and samples large datasets to ensure smooth performance.

### 4. Hybrid Sentiment Engine
* **Train-on-the-Fly**: If the uploaded dataset contains a rating column, the system automatically trains a fresh **TF-IDF + Logistic Regression** classification model tailored to that specific product domain.
* **Fallback Mode**: If no ratings are present, the app seamlessly falls back to a pre-trained base model.
* **Context-Aware Preprocessing**: Preprocesses text by retaining negation words (*not, no, tidak, kurang*) to prevent reversing sentiment labels.
* **N-grams Optimization (1, 3)**: Extracts unigrams, bigrams, and trigrams in the TF-IDF vectorizer to capture multi-word sentiment cues like *"not very good"*.
* **Soft Weighted Ensemble Layer**: Blends the text probability ($P_{\text{text}}$, 60% weight) from the ML model with a prior probability based on the star rating ($P_{\text{rating}}$, 40% weight) to produce a combined confidence score:
  * Rating prior mapping: 1 Star = 0.10, 2 Stars = 0.25, 3 Stars = 0.50, 4 Stars = 0.75, 5 Stars = 0.90.
  * Sentiment thresholds: $\ge 0.60$ is classified as **Positive**, $\le 0.40$ is classified as **Negative**, and values in-between are classified as **Neutral**.

### 5. PostgreSQL & Export Integration
* Export analyzed datasets with predicted sentiment labels directly as a `.csv` file.
* Save the entire analysis results and metadata permanently to a local PostgreSQL database with automatic schema setup.

### 6. AI Business Consultant (RAG)
* Ask business questions (e.g., *"Why are customers leaving negative reviews about shipping?"*) in plain language.
* **Dual Search Methods**:
  * **Keyword Search (TF-IDF)**: Matches literal keyword overlaps in reviews. Extremely fast.
  * **Two-Stage Semantic Search (MiniLM)**:
    1. **Lexical Candidate Filtering**: Runs TF-IDF with bilingual query expansion (e.g. mapping Indonesian terms like *"kualitas"* to English *"quality"*) to extract the top 100 candidate reviews.
    2. **Semantic Re-ranking**: Uses `paraphrase-multilingual-MiniLM-L12-v2` Sentence Transformers to embed and re-rank only these 100 candidates. This prevents CPU hangs, reducing execution time from 15 minutes to **~3 seconds**.
* **Background Preloading**: Loads the embedding model asynchronously in a background thread when the app starts, preventing the UI from freezing when the first semantic query is made.
* **Report Generation**: Passes this context to **Google Gemini** to draft a structured report with:
  * **Executive Summary** (Key findings and dataset context)
  * **Pain Points** (Key issues complete with direct customer quotes)
  * **Action Items** (Actionable business recommendations based on the data)
* **Pre-flight Guardrail**: Rejects off-topic prompts (like coding questions or general knowledge) locally before calling the Gemini API to prevent unnecessary token usage.
* **Hallucination Guard**: Uses local quote matching and keyword overlap calculations to verify if the AI's report is grounded in the retrieved customer reviews, displaying a warning if the report strays from the actual data.

---

## 🛠️ Tech Stack & Libraries

* **Frontend & UI**: `streamlit` (v1.32.0+)
* **Visualization**: `plotly-express`
* **Data Manipulation**: `pandas`, `numpy`
* **Machine Learning**: `scikit-learn` (Logistic Regression, TF-IDF Vectorizer)
* **Sentence Embeddings**: `sentence-transformers` (`paraphrase-multilingual-MiniLM-L12-v2`)
* **Generative AI**: `google-genai` SDK
* **Database**: `psycopg2-binary` (PostgreSQL Connector)

---

## 🚀 Getting Started

### Prerequisites
Make sure you have **Python 3.10+** installed on your system.

### 1. Installation
Clone this repository and install the required packages:
```bash
pip install -r requirements.txt
```

### 2. Configuration & Secrets
Create a `.streamlit/secrets.toml` file in the root of the project folder:
```toml
# Required for the AI Consultant
GEMINI_API_KEY = "your_actual_gemini_api_key_here"

# Optional: Configuration for your PostgreSQL database
DB_PASSWORD = "your_postgresql_password"
```

### 3. Run the App
Launch the Streamlit dashboard:
```bash
streamlit run app.py
```

---

## 📁 Project Structure

```
market-pulse/
│
├── app.py                      # Main Streamlit dashboard application
├── requirements.txt            # Dependency list
├── pyrefly.toml                # Linter configuration
├── system_design.md            # Target system design reference
│
├── modules/
│   ├── ai_consultant.py        # Pre-flight guardrails, RAG, and Hallucination Guard
│   ├── clean_csv.py            # CSV preprocessing helper
│   ├── database.py             # PostgreSQL connectors and schema initialization
│   ├── ml_pipeline.py          # TF-IDF + Logistic Regression classification
│   ├── queries.py              # Pre-defined database SQL queries
│   └── upload_processor.py     # File validation and auto-column detection
│
└── models/
    ├── model_sentimen.pkl      # Pre-trained base Logistic Regression model
    ├── vectorizer.pkl          # Pre-trained TF-IDF Vectorizer
    └── embeddings_*.npy        # Cached vector embeddings for dataset semantic search
```
