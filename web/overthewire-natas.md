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
