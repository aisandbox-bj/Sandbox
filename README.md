# Sandbox

A small launcher page that links to dashboards and analyses I've built. Public items open right away; private items are AES-encrypted and unlock with a shared passphrase.

Live site: https://aisandbox-bj.github.io/Sandbox/

---

## Repo layout

```
/index.html                       Launcher (the page everyone lands on)
/dashboards/                      Public dashboards — plain .html
/private/                         Encrypted dashboards — *.enc.json
/tools/encrypt.html               Local-only tool to produce .enc.json
/Showcase/                        Original source folder (kept for reference)
```

---

## ⚠ Before pushing for the first time

The previous `index.html` on GitHub was the **BD402 Parts Readiness** dashboard. Pushing this new launcher overwrites it. Save the old file before you push:

1. Open https://github.com/aisandbox-bj/Sandbox/raw/main/index.html in your browser.
2. Save the page as `bd402-parts-readiness.html` somewhere on your computer.
3. Then push the new files in this repo. BD402 won't be on the live site any more — that's fine, you can encrypt and re-add it later (see "Adding a private dashboard" below).

---

## Adding a public dashboard

1. Drop the `.html` file into `/dashboards/`.
2. Open `/index.html` in a text editor, find the `DASHBOARDS` array near the top of the `<script>` block, and add an entry:
   ```js
   {
     slug: 'my-thing',
     title: 'My Thing',
     description: 'One-line description that shows on the card.',
     type: 'public',
     href: './dashboards/my-thing.html',
     tags: ['Showcase']
   }
   ```
3. Commit + push.

---

## Adding a private (encrypted) dashboard

1. Open `/tools/encrypt.html` in your browser locally (just double-click).
2. Drop the dashboard `.html` file into the dropzone.
3. Type the **shared passphrase** (twice — must match).
4. Click **Encrypt & download**. A `*.enc.json` file downloads.
5. Move the `.enc.json` into `/private/` in this repo.
6. Add an entry to `DASHBOARDS` in `/index.html`:
   ```js
   {
     slug: 'bd402',
     title: 'BD402 Parts Readiness',
     description: 'Component readiness for the BD402 fleet rebuild.',
     type: 'private',
     href: './private/bd402.enc.json',
     tags: ['Operations']
   }
   ```
7. Commit + push.

The encrypt tool runs entirely in your browser — nothing is uploaded anywhere.

---

## Rotating the passphrase

If the passphrase leaks or you want to change it:

1. Gather the **plaintext** `.html` originals for every private dashboard. (You need these — the launcher only has the encrypted versions, which the OLD passphrase decrypts.)
2. Open `/tools/encrypt.html`.
3. Drop **all** the originals into the dropzone at once.
4. Enter the **new** passphrase.
5. Click Encrypt — get a new `.enc.json` for each.
6. To avoid CDN cache returning old ciphertext, rename each new file with a version suffix (e.g. `bd402.v2.enc.json`) and update the `href` in the launcher's `DASHBOARDS` array to match.
7. Push.
8. Delete the old `.enc.json` files after a day or two.

---

## How the encryption works

- **Algorithm**: AES-GCM 256-bit
- **Key derivation**: PBKDF2-SHA256 with 600,000 iterations
- **Salt**: 16 random bytes per file
- **IV**: 12 random bytes per file

The encrypted file is a JSON envelope with everything the launcher needs to decrypt. AES-GCM's authentication tag means a wrong passphrase fails cleanly — the launcher catches the failure and shows "Incorrect passphrase".

**Security note.** Anyone can download the `.enc.json` files from the public GitHub Pages site. Their security relies entirely on passphrase strength. The encrypt tool enforces a 12+ character minimum with at least two character classes, which combined with 600k PBKDF2 iterations puts brute-forcing well out of reach for casual attackers. Use a real passphrase (e.g. `maple-river-summer-2031`), not a 4-digit PIN.

---

## Running locally

For a final check before pushing, serve the folder over HTTP (the `private/*.enc.json` fetch needs http://, not file://):

```
python -m http.server 8000
```

Then open http://localhost:8000/ — that's exactly what GitHub Pages will serve.

---

## What's intentionally NOT here

- No analytics, no external fonts, no CDN scripts. Self-contained on purpose.
- No service worker. They cache aggressively and would serve stale ciphertext after rotation.
- No accounts or per-user access — GitHub Pages is static. The encrypted-payload + shared passphrase model is as far as it goes.
