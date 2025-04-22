
I refactored Indexer.py, modified search_similar method, which now returns a list of movie information strings. It ensure the query input is a string and raises a ValueError if it's a boolean.
I also add error handlingchanged __init__ method, which includes try-except blocks when attempting to read the 'movie_index' file and 'movie_metadata.json' is not found, an empty self.metadata dictionary is initialized, and a message is printed. This makes the class more robust to missing files.
I removed the dependency on GPU resources for Faiss indexing as my hardware was only able to use CPU. While GPUs can offer performance benefits for large-scale vector search, the current deployment environment necessitates a CPU-only solution. I’m planning to explore the possibility of using GPU as it would impact on performance especially that database is expected to grow.



I refactored the SemanticSearchBar to create search input (genre and query), improved UX and backend integration. The component is now more efficient, with loading states and error handling, and more extensible. I also introduced accessibility improvements and moved formatting logic into a dedicated hook, separating concerns for better maintainability.

I refactored the /api/search route to improve efficiency. It's now more maintainable and extensible, with clearly separated logic paths for genre- and search-based recommendations. The new structure also scales better thanks to proper async handling and improved error.

import { useState, useEffect } from "react";
import "../styles/SemanticSearchBar.css";

export function SemanticSearchBar() {
const [genre, setGenre] = useState(""); // State for the genre
const [query, setQuery] = useState(""); // State for the query
const [movies, setMovies] = useState([]); // State for the search results
const [isLoading, setIsLoading] = useState(false); // State for loading status
const [error, setError] = useState(null); // State for error handling
const [formattedMovies, setFormattedMovies] = useState([]);

// Handle form submission
const handleSearch = async (event) => {
event.preventDefault();

    if (!genre && !query) {
      setError("Please provide a genre or query.");
      return;
    }

    try {
      setIsLoading(true);
      setError(null);

      const response = await fetch("https://127.0.0.1:443/api/search", {
        method: "POST",
        credentials: "include",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          genre: genre,
          search: query,
        }),
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(
          data.message || data.error || "Failed to fetch results"
        );
      }

      setMovies(data.movies || "");
    } catch (err) {
      setError(err.message);
      setMovies([]);
    } finally {
      setIsLoading(false);
    }

};

useEffect(() => {
(async function formatMovies() {
let splitArray;
let moviesArray;
if (movies.length > 0) {
splitArray = movies.split("] [");
let lastIndex = splitArray.length - 1;

        splitArray[0] = await splitArray[0].slice(1, splitArray[0].length - 1);

        splitArray[lastIndex] = await splitArray[lastIndex].slice(
          splitArray[lastIndex][0],
          splitArray[lastIndex].length - 1
        );
        moviesArray = await splitArray.map((movie) => {
          return movie.split("§ ");
        });
        setFormattedMovies(moviesArray);
        console.log(moviesArray);
      }
      console.log(splitArray);
    })();

}, [movies]);

return (
<div className="semantic-search-container">
<form onSubmit={handleSearch} role="search">
<label htmlFor="genre-search">Search by Genre</label>
<input
id="genre-search"
type="text"
value={genre}
onChange={(e) => setGenre(e.target.value)}
placeholder="Enter genre (e.g., Action, Comedy)"
aria-label="Genre search"
/>

        <label htmlFor="query-search">Search by Plot/Description</label>
        <input
          id="query-search"
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Enter plot, keywords, or description"
          aria-label="Query search"
        />

        <button type="submit" disabled={isLoading}>
          {isLoading ? "Searching..." : "Search"}
        </button>
      </form>

      {error && <div className="error-message">{error}</div>}

      <div className="search-results">
        {movies && typeof movies === "string" ? (
          <p>{movies}</p>
        ) : (
          <p>No results found.</p>
        )}
      </div>
    </div>

);
}

@app.route('/api/search', methods=['POST'])
@login_required
async def get_recommendations():
recommender = Recommender()
req = request.get_json(silent=True) or {}
print("Parsed JSON:", req) # Debug line
genre = req.get('genre', False)
search = req.get('search', False)
user_id = current_user.id
try:
if genre:
select_movies_statement = "SELECT movies.\*, GROUP_CONCAT(DISTINCT keywords.keyword) AS keywords, GROUP_CONCAT(DISTINCT genres.genre) AS genres, GROUP_CONCAT(DISTINCT actors.actor) as actors FROM movies \
 JOIN users_movies ON users_movies.movies_id = movies.id \
 JOIN movies_keywords ON movies.id = movies_keywords.movies_id JOIN keywords ON movies_keywords.keywords_id = keywords.id \
 JOIN movies_genres ON movies.id = movies_genres.movies_id JOIN genres ON movies_genres.genres_id = genres.id \
 JOIN movies_actors ON movies.id = movies_actors.movies_id JOIN actors ON movies_actors.actors_id = actors.id \
 WHERE genres.genre = %s AND users_movies.users_id = %s GROUP BY movies.id;"
with connection_pool.get_connection() as connection:
with connection.cursor(dictionary=True) as cursor:
cursor.execute(select_movies_statement, (genre, user_id))
movies = cursor.fetchall()
if not movies:
return jsonify({'message': f'No movies favourited in selected genre, please favourite some {genre} movies or get recommendations by text'}), 400
result = await recommender.get_recommendation(movies)
else:
result = await recommender.get_recommendation(search)
for movie in result:
title = movie.get("title", "")
return jsonify({'movies': result})
except Exception as e:
return jsonify({'error: ': str(e)})
