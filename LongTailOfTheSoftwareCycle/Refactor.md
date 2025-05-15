### Profile the performance of existing code and be able to refactor where appropriate, considering improvements to efficiency, scalability, maintainability and extensibility.

I refactored Indexer.py, modified search_similar method, which now returns a list of movie information strings. It ensure the query input is a string and raises a ValueError if it's a boolean.
I also add error handlingchanged **init** method, which includes try-except blocks when attempting to read the 'movie_index' file and 'movie_metadata.json' is not found, an empty self.metadata dictionary is initialized, and a message is printed. This makes the class more robust to missing files.
I removed the dependency on GPU resources for Faiss indexing as my hardware was only able to use CPU. While GPUs can offer performance benefits for large-scale vector search, the current deployment environment necessitates a CPU-only solution. I’m planning to explore the possibility of using GPU as it would impact on performance especially that database is expected to grow. (Kloda, 2025a)

I refactored the SemanticSearchBar to create search input (genre and query), improved UX and backend integration. The component use handleSearch function with loading states and error handling. It also prevents the default form submission. I moved formatting logic into a useEffect hook. It handle the formatting of data, the movies data is a string with movies separated by "] [". It use map to split each movie string by "§ " and create array of movies.(Kloda, 2025b) I also refactored jest test, where i tested SemanticSearchBar ccomponent. (Kloda, 2025c)

I refactored the /api/search route to improve efficiency. It's now more maintainable and extensible, with clearly separated logic paths for genre- and search-based recommendations. The new structure also scales better thanks to proper async handling and improved error.

part of code from app.py (Kloda, 2025d)

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

### References:
Kloda (2025). [online] github. Available at: https://github.com/Jkloda/movie_recommendation_system/blob/main/server/faiss/Indexer.py [Accessed 2 May 2025].

Kloda (2025). [online] github. Available at: https://github.com/Jkloda/movie_recommendation_system/blob/main/client/src/components/SemanticSearchBar.js [Accessed 22 Apr. 2025].

Kloda (2025). [online] github. Available at: https://github.com/Jkloda/movie_recommendation_system/blob/main/client/src/tests/SemanticSearchBar.test.js [Accessed 10 May 2025].

Kloda (2025). [online] github. Available at: https://github.com/Jkloda/movie_recommendation_system/blob/main/server/app.py [Accessed 2 May 2025].