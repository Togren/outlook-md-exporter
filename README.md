# Outlook Email → Markdown Exporter

A minimal Office Add-in (no build tooling, no backend) that adds an "Export to MD" button to the message-read ribbon in Outlook on the web / New Outlook. Everything runs client-side in your browser — the email content never leaves your machine except to be written to the .md file you download.

## What it does

- Reads the open email's `To`, `From`, `Cc`, `Subject`, `Date` and writes them as YAML frontmatter.
- Converts the HTML body to Markdown using [Turndown](https://github.com/mixmark-io/turndown) + its GFM plugin (tables, strikethrough, task lists), preserving bold, italic, headings, links, ordered/unordered lists, and tables.
- Lets you choose how underline is handled (Markdown has no native underline syntax): keep as inline `<u>` HTML, convert to bold, or drop it.
- Downloads the result as a `.md` file named `YYYY-MM-DD Subject.md`.

## 1. Host the files

Outlook loads task panes over HTTPS, so this needs to be served from a real URL — a local `file://` path won't work.

**Simplest: GitHub Pages (recommended for a personal tool)**

1. Create a new GitHub repo (public is fine — there's nothing sensitive in these files) and push this folder's contents to it.
2. Repo Settings → Pages → Deploy from branch → `main` / root.
3. Your files will be served at `https://YOUR-USERNAME.github.io/REPO-NAME/...`.
4. In `manifest.xml`, replace every occurrence of `https://YOUR-USERNAME.github.io/outlook-md-exporter` with your actual Pages URL.

**Alternative: local HTTPS dev server**

If you'd rather not publish anything, use Microsoft's own dev-cert tool:
```
npx office-addin-dev-certs install
npx http-server . -p 3000 --ssl --cert ~/.office-addin-dev-certs/localhost.crt --key ~/.office-addin-dev-certs/localhost.key
```
Then point the manifest URLs at `https://localhost:3000/...` instead. This only works while your machine is running the server, so GitHub Pages is more convenient long-term.

## 2. Sideload the add-in

In Outlook on the web (or New Outlook desktop, which uses the same mechanism):

1. Open any email → the "..." (More actions) menu → **Get Add-ins**.
2. **My add-ins** → **Add a custom add-in** → **Add from file**.
3. Select your edited `manifest.xml`.
4. Open any email — you should see the "Export to MD" button in the ribbon or the "..." menu.

## 3. Use it

Open an email, click **Export to MD**, pick how you want underline handled, click the big button in the task pane. The .md file downloads through your browser's normal download flow.

## Notes / limitations

- Attachments are intentionally not exported (per your original ask).
- Very deeply nested lists or tables with merged cells can occasionally trip up Turndown's GFM table plugin — if you hit a case that converts badly, flag it and the conversion rule can be tightened.
- One email per click. If you want to batch-export a whole folder later, that's a bigger add-in (would need to enumerate items via the Outlook REST/Graph API rather than the simple `item` context this uses) — worth doing as a second iteration if the single-email flow proves useful first.
