export default {
  async fetch(request, env) {
    const url = new URL(request.url);

    // GET all movies
    if (url.pathname === "/movies") {
      const { results } = await env.DB.prepare("SELECT * FROM movies").all();
      return Response.json(results);
    }

    // GET all episodes for a movie
    if (url.pathname.startsWith("/episodes")) {
      const movieId = url.searchParams.get("movie_id");
      const { results } = await env.DB.prepare(
        "SELECT * FROM episodes WHERE movie_id = ?"
      ).bind(movieId).all();
      return Response.json(results);
    }

    // POST add movie (admin)
    if (url.pathname === "/add-movie" && request.method === "POST") {
      const data = await request.json();
      await env.DB.prepare(
        "INSERT INTO movies (title, image, description, category, type) VALUES (?, ?, ?, ?, ?)"
      ).bind(data.title, data.image, data.description, data.category, data.type).run();
      return new Response("Movie Added");
    }

    // POST add episode (admin)
    if (url.pathname === "/add-episode" && request.method === "POST") {
      const data = await request.json();
      await env.DB.prepare(
        "INSERT INTO episodes (movie_id, episode_number, iframe_url) VALUES (?, ?, ?)"
      ).bind(data.movie_id, data.episode_number, data.iframe_url).run();
      return new Response("Episode Added");
    }

    return new Response("Not Found", { status: 404 });
  }
};
