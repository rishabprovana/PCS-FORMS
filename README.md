# Team Break Board — Setup

This app is a single static HTML file (`break.html`). It has no server of its
own, so to make break status shared across everyone's devices (instead of
just saved locally in each person's browser), it uses a free **Firebase
Realtime Database** as the shared backend.

If you skip this setup, the app still works, but only locally — each
browser keeps its own copy of the data and nothing syncs between people.
You'll see a banner on the board saying "Not connected to a shared database
yet" until this is done.

## 1. Create a Firebase project

1. Go to <https://console.firebase.google.com/>.
2. Click **Add project**, give it any name (e.g. `team-break-board`), and
   finish the wizard. You can decline Google Analytics — it's not needed.

## 2. Create a Realtime Database

1. In the left sidebar of your new project, go to **Build → Realtime
   Database**.
2. Click **Create Database**.
3. Pick a location (this affects your URL — see step 3).
4. Start in **test mode** for now so read/write works immediately. You'll
   lock this down in step 4.

## 3. Get the database URL

After creation, the Realtime Database page shows your database's base URL
at the top, directly above the data viewer. It'll look like one of:

- `https://YOUR-PROJECT-default-rtdb.firebaseio.com` (US-based databases)
- `https://YOUR-PROJECT-default-rtdb.REGION.firebasedatabase.app` (regional
  databases, e.g. `europe-west1`, `asia-southeast1`)

Copy that **exact** URL — don't use the URL of the Firebase console page
itself (the one with `console.firebase.google.com` in it); that's a common
mix-up and it's what caused the original bug in `break.html`.

Open `break.html` and find this line near the top of the `<script>` block:

```js
const DATABASE_URL = 'https://pcs-forms-matrix-default-rtdb.firebaseio.com';
```

Replace the placeholder URL with your own, keeping the quotes:

```js
const DATABASE_URL = 'https://YOUR-PROJECT-default-rtdb.firebaseio.com';
```

Once this is set, the app detects it's configured and switches from local
browser storage to the shared database automatically — no other code
changes needed.

## 4. Lock down access rules

Test mode leaves your database open to anyone on the internet who has the
URL. Before sharing this with your team, tighten the rules:

1. In the Realtime Database page, go to the **Rules** tab.
2. Replace the rules with something like:

```json
{
  "rules": {
    "employees": {
      ".read": true,
      ".write": true
    }
  }
}
```

This keeps things simple (no auth) while scoping access to just the
`employees` path the app uses. For real access control, Firebase supports
adding authentication — but that's beyond what this simple board needs for
most teams, since the app's own `BOARD_KEY` and personal links already keep
casual visitors out (see below).

## 5. Set your own board key

In `break.html`, find:

```js
const BOARD_KEY = 'board';
```

Change `'board'` to your own private word (letters/numbers, no spaces).
This word becomes part of the manager view's URL:
`break.html#board-YOURWORD`. It's not real security — anyone who views the
page source can still read it — but it keeps casual visitors from
stumbling onto the full board.

Bookmark `break.html#board-YOURWORD` for yourself. Don't share that link
with employees.

## 6. Share personal links with employees

Each employee gets their own link instead of the board link, e.g.
`break.html#112590`. On the board (`#board-YOURWORD` view), click **Show
personal links to share** to see and copy every employee's individual
link. Employees only need their own link to start/stop their breaks and
see their own history — they never see the board or anyone else's status.

## 7. Host the file

Since this is a static file, you can host it anywhere that serves plain
HTML — GitHub Pages, Netlify, or even just opening it locally for testing.
For GitHub Pages:

1. Push `break.html` to a GitHub repo.
2. In the repo settings, enable **Pages** and point it at the branch/folder
   containing the file.
3. Your board will be live at
   `https://YOUR-USERNAME.github.io/YOUR-REPO/break.html`.

## Troubleshooting

- **Board shows "Not connected to a shared database yet"** — `DATABASE_URL`
  still contains the placeholder, or doesn't match the URL shown at the top
  of your Realtime Database page.
- **Data doesn't sync between devices** — double check the URL region
  suffix (`.firebaseio.com` vs `.firebasedatabase.app`) matches what
  Firebase actually shows for your project.
- **"Page not available" for everyone** — the URL hash doesn't match
  `BOARD_KEY` or any employee ID. Check the link you're using against
  `BOARD_KEY` in the code and the `id` values in the `EMPLOYEES` list.
