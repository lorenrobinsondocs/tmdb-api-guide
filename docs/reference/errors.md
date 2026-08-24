# Errors

TMDB returns errors in a consistent JSON shape. Every failed request includes three fields.

The `status_code` in the body is **not** the HTTP status code. It is TMDB's own internal error code, and the two do not map one-to-one. A single HTTP 401 can carry several different TMDB status codes depending on what was wrong with your credentials.

!!! note "Field order is not stable"

    The order of `success`, `status_code`, and `status_message` varies between endpoints. Parse by key name, never by position.

## Invalid API key

**HTTP 401** · **TMDB status code 7**

```json
{"status_code":7,"status_message":"Invalid API key: You must be granted a valid key.","success":false}
```

The key was rejected. Common causes:

- The key was never activated — new keys require confirming your account email
- You sent the Read Access Token as `api_key` instead of in an `Authorization` header
- Extra whitespace or a trailing newline was copied along with the key

To confirm the key itself is the problem, check its length. A v3 API key is exactly 32 characters. In PowerShell:

```powershell
$key.Length
```

## Resource not found

**HTTP 404** · **TMDB status code 34**

```json
{"success":false,"status_code":34,"status_message":"The resource you requested could not be found."}
```

The endpoint is valid but the record does not exist. Usually a movie or TV ID that was never issued, or one that has since been removed from the database.

Worth handling separately from a 401. A 404 means your authentication worked — only the ID was wrong.

## Errors that do not look like errors

If you send a request from PowerShell using `curl` rather than `curl.exe`, you may see a "Script Execution Risk" prompt rather than any response at all. That is not a TMDB error. See the [Quickstart](../quickstart.md) for why.