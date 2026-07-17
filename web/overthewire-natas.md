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



  
