# dreamhack-XS-Search

**Vulnerability:**
- Method 1: [XS-Search](https://hacktricks.wiki/pentesting-web/xs-search/)
- Method 2: Code logic

# Source

- The global variable `notes` holds a dict storing the notes and a boolean value.

<img width="498" height="133" alt="image" src="https://github.com/user-attachments/assets/853d4a2a-1391-4ea4-86a4-f7c670bf37c2" />

- Functions in `app.py`:
  - `/search`:
    - Takes input from the `query` param.
    - If the input matches characters in the flag, the request must come from `127.0.0.1` OR carry a `Host` header with the value `127.0.0.1:8000`.
      (Because of `if private == True and request.remote_addr != "127.0.0.1" and request.headers.get("HOST") != "127.0.0.1:8000": continue`: if the conditions inside the `if` are satisfied, the server skips to `return render_template("search.html", query=query, result=None)`.)
    - If the input is non-empty and appears in any note, the server runs `return render_template("search.html", query=query, result=note)`.
  - `/submit`:
    - If the request method is POST and the submitted URL starts with `http`, the server uses `driver.get(url)` to visit the user-supplied URL.

- `search.html`:
  - If a match is found (meaning the data passed through `/search` in `app.py` appears in a note stored in `notes`), the server returns the entire note.

# Exploit

- **Method 1: XS-Search**
  - We can abuse `/submit` (SSRF) to make the bot send internal requests in order to brute-force the flag.
  - When a correct character is guessed, the server returns the full note inside an `iframe` tag => `Content.length = 1`. Otherwise there is no frame => `Content.length = 0`.
  - The server visits the URL via `driver.get`. Note that if the visited endpoint contains HTML tags or `<script>` tags, they are rendered and executed as well.
  - So we can host an HTML file whose tags send requests to `127.0.0.1:8000/search` to brute-force the flag, using the frame-count condition above as the oracle to check each guess.

- **Method 2: Code logic**

  <img width="887" height="128" alt="image" src="https://github.com/user-attachments/assets/36dabe3c-fb5d-4da4-a868-97a596a0e1ea" />

  - Looking closely at the first `if` statement, the `continue` is only reached when all conditions are satisfied simultaneously => bypassing just one of the last two conditions is enough to get past the first filter.
  - Here we only need to send a single request with the `Host` header set to `127.0.0.1:8000` and the flag characters as the query => the server returns the whole flag string.
