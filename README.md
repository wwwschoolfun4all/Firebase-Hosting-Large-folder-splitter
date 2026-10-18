# Firebase Chunk Deploy

Deploys a big folder to **Firebase Hosting** one folder at a time, in size-capped chunks, skipping files you ignore.
Comes with a local GUI (runs on your own PC, opens in your browser) and a command-line version.

---

## What you need

- **Node.js 18 or newer** (check with `node -v`; download from nodejs.org if missing)
- A **Firebase project** with Hosting enabled
- A **service account key** (a JSON file, set up in Step 2)

You do **not** need the Firebase CLI or `gcloud`.

---

## Step 1: Install

Open PowerShell in this folder (in File Explorer, click the address bar, type `powershell`, press Enter) and run:

```powershell
npm install
```

You only do this once.

## Step 2: Create your key (one time)

1. Go to the Firebase console and open your project.
2. Click the gear icon, then **Project settings**, then the **Service accounts** tab.
3. Click **Generate new private key** and save the JSON file somewhere on your PC, for example `C:\keys\firebase-key.json`.
4. Keep it private. Do not put it on the USB stick, in a shared folder, or in a git repo.

If a deploy later fails with **403**: open Google Cloud Console, go to **IAM & Admin, IAM**, find the service account
(its email ends in `iam.gserviceaccount.com`), and add the role **Firebase Hosting Admin**.

## Step 3: Find your site ID

In the Firebase console go to **Build, Hosting**. Your site ID is the first part of your URL:
`my-project` in `my-project.web.app`. It is usually the same as your project ID.

## Step 4: Start the GUI

Every time you open a new PowerShell window, point the app at your key, then start it:

```powershell
$env:GOOGLE_APPLICATION_CREDENTIALS="C:\keys\firebase-key.json"
node gui.js
```

Your browser opens at **http://localhost:4173**. Leave the PowerShell window open while you use it (Ctrl+C to quit).

On Mac or Linux use `export GOOGLE_APPLICATION_CREDENTIALS=/path/to/firebase-key.json` instead.

## Step 5: Deploy

1. **Hosting site ID:** type the ID from Step 3.
2. **Channel:** leave as `live` to publish to your real site. (A preview channel must already exist.)
3. **Folder to deploy:** click **Browse...**, pick a drive or folder, click through to it, then **Use this folder**.
   You can also type the path, for example `E:\site`.
4. **Chunking:** defaults are fine (50 MB and 500 files per chunk, 5 parallel uploads).
   If uploads time out or your drive is slow, lower these.
5. **Ignore rules:** list anything that should not be uploaded, one per line, then click **Save ignore rules**.
6. Click **Dry run** first. It lists each folder and how many chunks it needs, without uploading anything.
7. Click **Deploy**. Watch the progress bar and log. When it says "Done", your site is live.

### If a deploy stops or fails
Click **Resume failed deploy**. It reuses the unfinished version and skips every file already uploaded.
Nothing goes live until all folders finish uploading, so a failed run never leaves your site half-updated.

---

## Using a USB drive

- Plug it in **before** clicking Browse, or click **Refresh drives**. It appears as `Drive E:` (Windows),
  under `USB/Volume:` (Mac) or `USB/Media:` (Linux).
- Keep it plugged in until the deploy finishes. If it gets unplugged, reconnect it and click **Resume failed deploy**.
- Large folders on slow USB sticks take longer. Try 2 or 3 parallel uploads if the drive struggles.
- Your ignore rules and resume state are stored on your PC, never on the USB.

---

## Ignore rules

Edit them in the GUI or in `.firebaseignore` (same format as `.gitignore`):

```
*.map        # every source map
*.zip        # every zip file
raw/         # a whole folder named raw
drafts/      # a whole folder named drafts
```

Always skipped automatically: `.git`, dotfiles (names starting with a dot), `node_modules`, `firebase.json`.

## Optional: your rewrites, redirects and headers

Put the path to your `firebase.json` in the GUI's "firebase.json" box. Its `rewrites`, `redirects`, `headers`,
`cleanUrls` and related settings are copied into the new version. Leave it blank if you don't use them.

---

## Command-line version (no GUI)

```powershell
node deploy.js --site my-project --dir E:\site --dry-run
node deploy.js --site my-project --dir E:\site
node deploy.js --site my-project --dir E:\site --resume
```

| Option | Meaning | Default |
|---|---|---|
| `--site` | Hosting site ID (required) | |
| `--dir` | Folder to deploy | `public` |
| `--channel` | `live` or an existing preview channel | `live` |
| `--chunk-mb` | Max MB per chunk | 50 |
| `--max-files` | Max files per chunk | 500 |
| `--concurrency` | Parallel uploads | 5 |
| `--ignore` | Ignore file | `.firebaseignore` |
| `--config` | `firebase.json` to copy settings from | `firebase.json` |
| `--message` | Release message | `chunked deploy` |
| `--resume` | Continue a failed deploy | |
| `--dry-run` | Preview only | |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Cannot find module 'google-auth-library'` | Run `npm install` in this folder. |
| `Could not load the default credentials` | Set `GOOGLE_APPLICATION_CREDENTIALS` (Step 4) in the same PowerShell window, then run `node gui.js` again. |
| `Test-Path` prints `False` | The key file path is wrong. Check the spelling and the folder. |
| `403` / permission denied | Add the **Firebase Hosting Admin** role to the service account (Step 2). |
| `404` on the site | Wrong site ID, or Hosting isn't set up. Check Build, Hosting in the console. |
| Channel error | Preview channels must exist first. Use `live`, or create the channel in the console. |
| `gcloud` not recognized | You don't need it. Use the service account key instead. |
| USB drive missing from the list | Plug it in, then click **Refresh drives**, or type the path by hand. |
| Deploy stopped halfway | Click **Resume failed deploy**. |
| Page won't load | Make sure `gui.js` and `gui.html` are in the same folder and `node gui.js` is still running. |

---

## How it works

1. Creates a Hosting version.
2. For each top-level folder, then each chunk inside it: gzips the files, hashes them, asks Firebase which ones it
   doesn't already have, and uploads only those.
3. Finalizes the version and releases it to your channel.

Each file is gzipped in memory, so a single file of several hundred MB needs about that much free RAM.

## Files

- `gui.js` and `gui.html`: the local GUI
- `deploy.js`: the deploy engine and command-line tool
- `.firebaseignore`: your ignore rules
- `package.json`: dependencies
