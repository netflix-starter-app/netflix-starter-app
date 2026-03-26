netflix-starter-app/
 ├── frontend/          ← put all frontend files here (or `public/` if template uses that)
 │    ├── index.html
 │    ├── movie.html
 │    ├── admin.html
 │    ├── style.css
 │    └── app.js
 └── backend/
      └── worker.js      ← Worker code (0cc4f4e7-bff7-4edc-af55-88f25cac8204)
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
<!DOCTYPE html>
<html>
<head>
  <title>streamflix</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
<h1>🎬 Movies</h1>
<div id="movies"></div>

<script src="app.js"></script>
<script>
fetch("https://netflix-starter-app.bilalakhtar0308ml.workers.dev/movies")
  .then(res => res.json())
  .then(data => {
    let html = "";
    data.forEach(movie => {
      html += `
        <div class="movie-card">
          <img src="${movie.image}" width="150">
          <h3>${movie.title}</h3>
          <a href="movie.html?id=${movie.id}">Watch</a>
        </div>
      `;
    });
    document.getElementById("movies").innerHTML = html;
  });
</script>
</body>
</html>
    return new Response("Not Found", { status: 404 });
  }
};
<h2>Admin Panel</h2>

<h3>Add Movie</h3>
<input id="title" placeholder="Title"><br>
<input id="image" placeholder="Image URL"><br>
<input id="category" placeholder="Category"><br>
<select id="type">
  <option value="movie">Movie</option>
  <option value="series">Series</option>
</select><br>
<button onclick="addMovie()">Add Movie</button>

<h3>Add Episode</h3>
<input id="movie_id" placeholder="Movie ID"><br>
<input id="episode_number" placeholder="Episode Number"><br>
<input id="iframe_url" placeholder="Iframe URL"><br>
<button onclick="addEpisode()">Add Episode</button>

<script src="app.js"></script>
<script>
function addMovie() {
  fetch("https://netflix-starter-app.bilalakhtar0308ml.workers.dev/add-movie", {
    method: "POST",
    headers: {"Content-Type": "application/json"},
    body: JSON.stringify({
      title: document.getElementById("title").value,
      image: document.getElementById("image").value,
      description: "",
      category: document.getElementById("category").value,
      type: document.getElementById("type").value
    })
  }).then(()=>alert("Movie Added!"));
}

function addEpisode() {
  fetch("https://netflix-starter-app.bilalakhtar0308ml.workers.dev/add-episode", {
    method: "POST",
    headers: {"Content-Type": "application/json"},
    body: JSON.stringify({
      movie_id: document.getElementById("movie_id").value,
      episode_number: document.getElementById("episode_number").value,
      iframe_url: document.getElementById("iframe_url").value
    })
  }).then(()=>alert("Episode Added!"));
}
</script>
body { font-family: Arial, sans-serif; padding: 20px; background: #f5f5f5; }
.movie-card { display: inline-block; margin: 10px; padding: 10px; background: white; border-radius: 5px; }
button { margin: 5px; padding: 5px 10px; cursor: pointer; }
input, select { margin: 5px 0; padding: 5px; }
// Currently empty, you can add common JS functions here
