# Quickstart

Get a working search request in about five minutes.

## 1. Get an API key

Create a free account at [themoviedb.org](https://www.themoviedb.org/signup), then open **Settings → API** and request a developer key.

TMDB issues two credentials. The **API Key** is a 32-character string used as a query parameter — that's the one used throughout this guide. The **API Read Access Token** is a longer string sent in an `Authorization` header instead. Either works; don't mix them in the same request.

## 2. Store your key

Never paste your key directly into a script or commit it to a repository. Store it in an environment variable for the session.

PowerShell:

```powershell
$key = "your_api_key_here"
```

macOS and Linux:

```bash
export TMDB_KEY="your_api_key_here"
```

## 3. Make your first request

Search for a movie by title.

```powershell
curl.exe "https://api.themoviedb.org/3/search/movie?query=Heat&api_key=$key"
```

The same request in Python:

```python
import os, requests

response = requests.get(
    "https://api.themoviedb.org/3/search/movie",
    params={"query": "Heat", "api_key": os.environ["TMDB_KEY"]},
)
print(response.json())
```

And in JavaScript:

```javascript
const key = process.env.TMDB_KEY;
const res = await fetch(
  `https://api.themoviedb.org/3/search/movie?query=Heat&api_key=${key}`
);
console.log(await res.json());
```

!!! warning "Windows users: use `curl.exe`, not `curl`"

    In PowerShell, `curl` is an alias for `Invoke-WebRequest`, which tries to parse the response with a rendering engine that is no longer installed. You get a "Script Execution Risk" prompt instead of your data. Typing `curl.exe` calls the real curl that ships with Windows 10 and 11.

## 4. Read the response

A successful search returns a JSON object with `page`, `results`, `total_pages`, and `total_results`. Each entry in `results` includes the movie's `id`, `title`, `release_date`, and `overview`.

The `id` is what you'll use for every follow-up request — cast, images, recommendations — so it's usually the first thing you extract.

## Next steps

If your request failed, see [Errors](reference/errors.md) for what each response means.