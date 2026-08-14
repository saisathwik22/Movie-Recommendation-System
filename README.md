# 🎬 Movie Recommendation System

A full-stack movie recommendation web application. It uses a content-based filtering model (TF-IDF + cosine similarity) built on movie metadata, served through a FastAPI backend, and displayed through a Streamlit frontend. Live movie data (posters, ratings, genres, overviews) is fetched from The Movie Database (TMDB) API.


🔗 **Live Demo:** https://movie-recommendation-system-bmjtdzdddrhrfeqtftgqjf.streamlit.app/


<img width="426" height="240" alt="demo" src="https://github.com/user-attachments/assets/c65e0c02-ad5c-49ab-96f2-880008ed0c4a" />




---

## 📌 What It Does
 
1. User searches for a movie by title.
2. The app shows autocomplete suggestions as they type.
3. On selecting a movie, its details (poster, backdrop, overview, genres, release date) are displayed.
4. Two sets of recommendations are shown:
   - **TF-IDF based** — movies with similar text content (overview/description similarity).
   - **Genre based** — movies from the same genre, sorted by popularity.
5. Users can click any recommended movie to view its details and get new recommendations, continuing the exploration.
---
 
## 🧠 How the Recommendation Model Works
 
The recommendation logic was developed and tested in a Jupyter notebook, then reused in the backend.
 
**Steps:**
1. **Data loading & cleaning** — Reads `movies_metadata.csv` (45,466 rows), removes 13 duplicate rows and 6 rows with missing titles, and extracts genre names from nested JSON fields. Final cleaned dataset: **45,447 movies**.
2. **Feature combination** — Merges `overview`, `genres`, and `tagline` into a single unified `tags` field per movie, so all textual signals are vectorized together instead of separately.
3. **Text preprocessing** — A custom `preprocess_text()` function applies lowercasing, regex-based punctuation removal, stopword removal, and lemmatization (via NLTK) to the combined text.
4. **TF-IDF vectorization** — Converts the cleaned text into numerical vectors using `TfidfVectorizer` with `ngram_range=(1,2)` (captures both single words and two-word phrases) and `max_features=50000` (vocabulary capped at 50K terms). The resulting matrix has shape **(45447, 50000)** and is ~99.93% sparse (~1.55M non-zero entries), stored efficiently as a compressed sparse row matrix.
5. **Cosine similarity** — Compares the TF-IDF vector of a chosen movie against all others to find the most textually similar ones.
6. **Precomputed storage** — The TF-IDF matrix, vectorizer, and title-index mapping are saved as `.pkl` files so the app doesn't need to reprocess data on every request; it just loads these files and computes similarity directly.
---
 
## 🏗️ Architecture
 
### Backend — FastAPI (`main.py`)
 
| Endpoint | Description |
|---|---|
| `GET /health` | Health check |
| `GET /home` | Curated home feed (trending, popular, top-rated, now-playing, upcoming) |
| `GET /tmdb/search` | Keyword search on TMDB, returns multiple matches |
| `GET /movie/id/{tmdb_id}` | Movie details by TMDB ID |
| `GET /recommend/genre` | Genre-based recommendations |
| `GET /recommend/tfidf` | TF-IDF similarity-based recommendations |
| `GET /movie/search` | Combined response: movie details + both recommendation types |
 
**Details:**
- Uses `httpx` for async HTTP requests to TMDB.
- Loads the pre-computed TF-IDF matrix and index files at startup for fast lookups.
- Normalizes movie titles for matching so minor formatting differences don't break searches.
- CORS enabled so the frontend (or any client) can call it directly.
- Error handling with fallback responses if TMDB or the model lookup fails.
### Frontend — Streamlit (`app.py`)
 

 
## 🛠️ Tech Stack
 
| Layer | Tools |
|---|---|
| Backend API | FastAPI, httpx, Pydantic |
| Frontend | Streamlit |
| Model / Data processing | pandas, NumPy, scikit-learn (TF-IDF, cosine similarity), NLTK (stopwords, lemmatization) |
| External data | TMDB API |
| Storage | pickle (`.pkl` files for matrix, vectorizer, indices, dataframe) |
 
---
 
## 🚀 Setup & Usage
 
### Prerequisites
- Python 3.8+
- A TMDB API key ([get one here](https://www.themoviedb.org/settings/api))
### Install dependencies
```bash
pip install -r requirements.txt
```
 
### Environment variables
Create a `.env` file in the project root:
```
TMDB_API_KEY=your_api_key_here
```
 
### Run the app
Open two terminals:
 
```bash
# Terminal 1 — backend
uvicorn main:app --reload
```
 
```bash
# Terminal 2 — frontend
streamlit run app.py
```
 
Then open `http://localhost:8501` in your browser.
 
