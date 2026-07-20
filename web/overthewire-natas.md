# OverTheWire: Natas (Web Security)

## Level 0 → 1

**Goal:** Find the password for natas1 on the page.

**Tools used:** Browser view-source (Ctrl+U)

**What I learned:**
- Anything the server sends to the browser is visible to the user. The password
  was in an HTML comment (`<!-- ... -->`) — hidden from the rendered page, but
  fully present in the source.
- Developers sometimes leave credentials, debug notes, or hints in comments
  assuming nobody inspects the source.
- Core web-security principle: client-side "hiding" is not security.

## Level 1 → 2

**Goal:** Same as level 0, but right-click is disabled.

**Tools used:** Ctrl+U / browser dev tools (F12)

**What I learned:**
- Disabling right-click via JavaScript blocks a *convenience*, not access —
  Ctrl+U and dev tools bypass it instantly.
- Reinforces the same lesson: client-side restrictions are trivially bypassed
  because the client controls the client.

## Level 2 → 3

**Goal:** Find the natas3 password on a page that appears to contain nothing.

**Tools used:** Browser view-source, URL manipulation

**What I learned:**
- The page was empty, but the source contained an image reference
  (`<img src="files/pixel.png">`), leaking the existence of a `files/` directory.
- Requesting the directory path directly returned a browsable listing, because
  the server had automatic directory indexing enabled — exposing files that were
  never linked from any page.
- **Unlinked ≠ inaccessible.** If a file is served, anyone who infers the path
  can reach it. Obscurity is not access control.
- Directory indexing should be disabled in production; it turns one leaked path
  into a full inventory of the folder.
- This is standard web recon — the same principle behind directory brute-forcing
  tools like `gobuster` and `dirb`.

## Level 3 → 4

**Goal:** Find the natas4 password. The hint said "not even Google will find it."

**Tools used:** Browser, `robots.txt`

**What I learned:**
- The Google hint pointed to `robots.txt` — a plain-text file at a site's root
  listing paths that crawlers should not index.
- `robots.txt` is publicly readable and purely advisory. Nothing enforces it, so
  a `Disallow` entry on a secret directory *advertises* that directory rather
  than protecting it.
- The file intended to hide the path is exactly what reveals it.
- **Always check `robots.txt` during web recon** — it's a free map of the paths
  the site owner considers sensitive.
- **Never rely on it for security.** It stops polite crawlers, not people.

## Level 4 → 5

**Goal:** Access a page that only allows visitors "coming from"
natas5.natas.labs.overthewire.org.

**Tools used:** `curl`, browser dev tools

**What I learned:**
- The server checked the `Referer` header to decide where the request came from.
- Every HTTP header is sent *by the client*, so it can be set to anything:
  `curl.exe -u natas4:PASS -H "Referer: http://natas5.natas.labs.overthewire.org/" http://natas4.natas.labs.overthewire.org/index.php`
- `-u user:pass` supplies HTTP basic auth credentials with the request.
- Core principle: **never trust client-supplied data.** Headers, cookies, form
  fields, and hidden inputs are all attacker-controlled.
- Trusting `Referer` for access control is broken authentication — it proves
  nothing.
- PowerShell gotcha: use `curl.exe`, since bare `curl` is an alias for
  `Invoke-WebRequest` and takes different syntax.
- Quirk: the header is misspelled "Referer" in the HTTP spec, and the typo is
  now permanent.
- First level requiring a hand-crafted HTTP request rather than just clicking —
  the foundation of practical web testing.

  ## Level 5 → 6

**Goal:** Page claims "You are not logged in" despite valid basic auth.

**Tools used:** Browser dev tools (cookie editor), `curl`

**What I learned:**
- The server set a cookie `loggedin=0` and then trusted that value on the next
  request to decide auth state. Changing it to `1` granted access:
  `curl.exe -u natas5:PASS -b "loggedin=1" http://natas5.natas.labs.overthewire.org/`
- Cookies are stored client-side, so they are fully attacker-controlled — editable
  via dev tools or settable with `curl -b`.
- Same failure as the Referer level, different vector: the server delegated a
  security decision to data the client owns.
- Correct pattern: a session cookie holding an **unguessable random token**, with
  the real auth state kept server-side. The cookie should be a lookup key, not
  the answer itself.
- A cookie that directly encodes `loggedin` or `isadmin` is trivially bypassed.

  ## Level 6 → 7

**Goal:** Submit the correct secret to a form to reveal the natas7 password.

**Tools used:** Browser, "View sourcecode" link, direct URL request

**What I learned:**
- The page offered a "View sourcecode" link — reading it revealed the check
  `if($secret == $_POST['secret'])`, with `$secret` pulled from
  `include "includes/secret.inc"`.
- Requesting `/includes/secret.inc` directly returned the file **as plain text**,
  exposing the secret.
- PHP only executes files the server is configured to parse (usually `.php`).
  A `.inc` file inside the webroot is served raw, dumping any credentials in it.
- Real-world fix: keep include files **outside the webroot**, or give them a
  `.php` extension so they're executed rather than displayed.
- Reinforces the Natas 2 lesson: anything under the webroot is fetchable by
  anyone who knows the path. Unlinked ≠ inaccessible.

  ## Level 7 → 8

**Goal:** Retrieve the natas8 password, hinted to be in
`/etc/natas_webpass/natas8`.

**Tools used:** Browser, URL parameter manipulation

**What I learned:**
- Navigation used a `page` parameter (`index.php?page=home`) fed straight into a
  PHP `include`.
- Supplying an absolute file path made the server read that file back:
  `index.php?page=/etc/natas_webpass/natas8`
- This is **Local File Inclusion (LFI)** — user input used unsanitised to build a
  filesystem path for an include.
- The same flaw could expose `/etc/passwd`, config files with DB credentials, or
  application source.
- Root cause: trusting user input to construct a file path. Fix: whitelist
  allowed pages and map to known files — never concatenate raw input into an
  include.
- More dangerous than earlier levels: it makes the *server* read arbitrary files,
  and under the right conditions LFI can escalate to remote code execution.

## Level 8 → 9

**Goal:** Submit the correct secret to a form; the expected value is stored
encoded in the page source.

**Tools used:** Browser source view, `xxd`, `rev`, `base64`

**What I learned:**
- The source revealed the encoding pipeline applied to the secret:
  `bin2hex(strrev(base64_encode($secret)))` — base64, then reverse, then hex.
- Reversing it means undoing each step in the opposite order — hex-decode,
  reverse, base64-decode:
  `echo "3d3d516343746d4d6d6c315669563362" | xxd -r -p | rev | base64 -d`
- Every step (base64, string reversal, hex) is a public, keyless, two-way
  function — encoding, not encryption.
- Encoding provides **zero security** once the algorithm is known, and here it
  was printed on the page via the source link.
- Same peel-back-the-layers logic as the Bandit compression level, applied to
  encodings instead of compression formats.

  
