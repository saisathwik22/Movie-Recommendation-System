# 🎬 Movie Recommendation System

A full-stack movie recommendation web application. It uses a content-based filtering model (TF-IDF + cosine similarity) built on movie metadata, served through a FastAPI backend, and displayed through a Streamlit frontend. Live movie data (posters, ratings, genres, overviews) is fetched from The Movie Database (TMDB) API.


🔗 **Live Demo:** https://movie-recommendation-system-bmjtdzdddrhrfeqtftgqjf.streamlit.app/


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
1. **Data loading & cleaning** — Reads `movies_metadata.csv`, removes duplicates, fills missing values in overview/tagline fields, and extracts genre names from nested JSON fields.
2. **Text preprocessing** — Lowercasing, punctuation removal, stopword removal, and lemmatization applied to overview/tagline text.
3. **TF-IDF vectorization** — Converts the cleaned text into numerical vectors representing word importance across all movies.
4. **Cosine similarity** — Compares the TF-IDF vector of a chosen movie against all others to find the most textually similar ones.
5. **Precomputed storage** — The TF-IDF matrix, vectorizer, and title-index mapping are saved as `.pkl` files so the app doesn't need to reprocess data on every request; it just loads these files and computes similarity directly.

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

**Home view:**
- Search bar with autocomplete dropdown (filtered by keyword match).
- Poster grid showing search results.
- Category selector to browse the home feed (trending/popular/top-rated/etc.).

**Details view:**
- Displays poster, backdrop, title, release date, genres, and overview.
- Shows both TF-IDF and genre-based recommendation lists.
- Back navigation to return to search/browse.
- Uses URL query parameters to keep state, so views can be shared as links.

---

## 🔌 API Response Models

**TMDBMovieCard**
- `tmdb_id`: int
- `title`: str
- `poster_url`: Optional[str]
- `release_date`: Optional[str]
- `vote_average`: Optional[float]

**TMDBMovieDetails**
- `tmdb_id`: int
- `title`: str
- `overview`: Optional[str]
- `release_date`: Optional[str]
- `poster_url`: Optional[str]
- `backdrop_url`: Optional[str]
- `genres`: List[dict]

**SearchBundleResponse**
- `query`: str
- `movie_details`: TMDBMovieDetails
- `tfidf_recommendations`: List[TFIDFRecItem]
- `genre_recommendations`: List[TMDBMovieCard]

---

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

### Rebuilding the recommendation model
If you want to regenerate the `.pkl` files from the raw dataset:
1. Place `movies_metadata.csv` in the project root.
2. Run `Movie_Recommendation_System.ipynb` cell by cell — it will clean the data, preprocess text, compute the TF-IDF matrix, and save all pickle files.

Example of using the model directly in Python:
```python
import pickle
import pandas as pd
from sklearn.metrics.pairwise import cosine_similarity

tfidf_matrix = pickle.load(open('tfidf_matrix.pkl', 'rb'))
indices = pickle.load(open('indices.pkl', 'rb'))
df = pd.read_pickle('df.pkl')

def recommend(title, n=10):
    if title not in indices:
        return ['Movie not found']
    idx = indices[title]
    similarity_score = cosine_similarity(tfidf_matrix[idx], tfidf_matrix).flatten()
    similar_idx = similarity_score.argsort()[::-1][1:n+1]
    return df['title'].iloc[similar_idx]

print(recommend('Toy Story'))
```

---

## 📁 Project Structure

```
movie-recommendation-system/
├── main.py                            # FastAPI backend
├── app.py                             # Streamlit frontend
├── Movie_Recommendation_System.ipynb  # Notebook: data cleaning, preprocessing, model building
├── movies_metadata.csv                # Raw input dataset
├── requirements.txt                   # Python dependencies
├── df.pkl                             # Preprocessed movie dataset
├── indices.pkl                        # Title-to-index mapping
├── tfidf_matrix.pkl                   # Pre-computed TF-IDF matrix
└── tfidf.pkl                          # TF-IDF vectorizer object
```

---

## 🔮 Possible Future Enhancements

- Collaborative filtering or a hybrid recommendation approach (combining content-based and user-behavior-based methods).
- Include additional features (director, cast) in the similarity calculation for richer recommendations.
- User feedback/rating mechanism to refine recommendations over time.

---

## 📝 License

MIT License
