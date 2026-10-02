# Working with Images

TMDB does not return image URLs. It returns image *file paths*, and it is up to you to build the full URL.

A movie record includes fields like this:

```json
"poster_path": "/jSziioSwPVrOy9Yow3XhWIBDjq1.jpg"
```

That is not a link. Pasting it into a browser gets you nothing. To display the poster, you assemble a URL from three parts:

```
base URL  +  size  +  file path
```

For example:

```
https://image.tmdb.org/t/p/  +  w500  +  /jSziioSwPVrOy9Yow3XhWIBDjq1.jpg
```

Which produces:

```
https://image.tmdb.org/t/p/w500/jSziioSwPVrOy9Yow3XhWIBDjq1.jpg
```

That URL returns the Fight Club poster at 500 pixels wide.

The rest of this page explains where each part comes from.

## Part 1: The base URL

The base URL comes from the `/configuration` endpoint:

```
GET https://api.themoviedb.org/3/configuration?api_key=YOUR_KEY
```

The response opens with an `images` object:

```json
"images": {
  "base_url": "http://image.tmdb.org/t/p/",
  "secure_base_url": "https://image.tmdb.org/t/p/",
  "backdrop_sizes": ["w300", "w780", "w1280", "original"],
  "logo_sizes": ["w45", "w92", "w154", "w185", "w300", "w500", "original"],
  "poster_sizes": ["w92", "w154", "w185", "w342", "w500", "w780", "original"],
  "profile_sizes": ["w45", "w185", "h632", "original"],
  "still_sizes": ["w92", "w185", "w300", "original"]
}
```

**Use `secure_base_url`, not `base_url`.** The two differ only in protocol. `base_url` is plain HTTP, which browsers will block as mixed content on any HTTPS page. There is no reason to use it.

**Do not call `/configuration` on every request.** These values change rarely. Fetch them when your application starts, cache them, and refresh every few days. TMDB publishes this endpoint on the assumption that you will store the result, not poll it.

## Part 2: The size

Each image type has its own list of valid sizes, shown above. Two naming conventions appear in those lists:

| Format | Meaning | Example |
|---|---|---|
| `w` + number | Constrain to this **width** in pixels. Height scales to preserve aspect ratio. | `w500` |
| `h` + number | Constrain to this **height** in pixels. Width scales to preserve aspect ratio. | `h632` |
| `original` | The unresized file as uploaded. Dimensions vary by image. | `original` |

Use the smallest size that looks right at your display dimensions. `original` files can be several megabytes, which is wasteful for a grid of thumbnails.

Sizes are not interchangeable across image types. `w342` is a valid poster size but is not in `backdrop_sizes`. Always pull from the list that matches the field you are rendering.

## Part 3: The file path

The file path comes from whatever endpoint returned the record you are displaying. Common fields:

| Field | Appears on | Use sizes from |
|---|---|---|
| `poster_path` | Movies, TV shows, seasons | `poster_sizes` |
| `backdrop_path` | Movies, TV shows | `backdrop_sizes` |
| `profile_path` | People (cast and crew) | `profile_sizes` |
| `logo_path` | Production companies, networks | `logo_sizes` |
| `still_path` | Episodes | `still_sizes` |

**The file path already begins with a slash.** The base URL already ends with one. Concatenate them directly. Adding your own separator produces a doubled slash and a broken URL.

**The path can be `null`.** Not every record has every image. A movie may have a poster but no backdrop; a supporting actor may have no profile photo at all. Check for `null` before building the URL, and fall back to a placeholder.

## Putting it together

```javascript
const IMAGE_BASE = "https://image.tmdb.org/t/p/";

function buildImageUrl(filePath, size) {
  if (!filePath) return null;
  return IMAGE_BASE + size + filePath;
}

// Fight Club (movie ID 550)
buildImageUrl("/jSziioSwPVrOy9Yow3XhWIBDjq1.jpg", "w500");
// "https://image.tmdb.org/t/p/w500/jSziioSwPVrOy9Yow3XhWIBDjq1.jpg"

buildImageUrl(null, "w500");
// null
```

In production, replace the hardcoded `IMAGE_BASE` with the cached `secure_base_url` from `/configuration`.

## When images do not load

Image requests fail silently. The image host does not return JSON errors the way the API does, so a malformed URL gives you a broken image and an HTTP status code, nothing more.

| Symptom | Cause |
|---|---|
| 404 | Invalid size string. `w400` is not a real size, but nothing will tell you so. Check the value against the list for that image type. |
| 404, URL contains `//` before the filename | A separator was added between the base URL and the file path, which already starts with a slash. |
| Broken image, URL ends in `null` | The record's path field was `null` and got concatenated anyway. |
| Blocked in console as mixed content | `base_url` was used instead of `secure_base_url`. |

Note that the image host does not check your API key. Image URLs are public, and a 404 always means a malformed URL rather than an authentication problem.