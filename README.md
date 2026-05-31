2026 Pentecost Festival Display Board — Setup Guide
This is a single-page web app for the Cupertino Zion lobby board. With the free
Firebase step below, everyone who opens the page sees the same live data, and
any edit made on any device is shared with everyone instantly. No data is stored
on individual visitors' devices.
There are two parts:
Firestore — the free shared database (Cloud Firestore) that holds the board's data.
GitHub Pages — free hosting so anyone can open the page by its web link.
You only do the Firebase setup once. Total time: ~10 minutes.
---
Part 1 — Create the free shared database (Firebase)
Go to https://console.firebase.google.com and sign in with a Google account.
Click Add project → give it a name (e.g. `cupertino-zion-festival`) → continue.
You can disable Google Analytics. Click Create project.
In the left menu choose Build → Firestore Database → Create database.
Choose a location (e.g. us-west) → start in test mode for now → Enable.
Set who can read/write. This project already uses the Firebase CLI, so the
rules live in `firestore.rules` (included). They allow anyone to read and edit
the single board document:
```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /festival/{docId} {
         allow read, write: if true;
       }
       match /{document=**} {
         allow read, write: if false;
       }
     }
   }
   ```
Deploy them with the CLI:
```
   firebase deploy --only firestore:rules
   ```
(Or paste the same rules into Firestore Database → Rules in the console and
click Publish.)
Get your config keys. Click the gear icon → Project settings. Scroll to
Your apps and copy the `firebaseConfig` values. These are already filled in
for the `cupertino-zion-festival` project inside `index.html`.
> **Tip:** If you skip Firebase entirely, the board still works, but in "Local
> mode" — data is saved only on the device you're using and is **not** shared.
> The status pill at the top right tells you which mode you're in:
> **Live · shared** (green) = working and shared. **Local mode** (grey) = not shared yet.
If the status pill is stuck on "Connecting…" or shows "Connection issue"
This almost always means the Firestore security rules are blocking access.
Default rules (`allow read, write: if false;`) reject every request, so the page
hangs. Fix it by deploying the included open rules:
```
firebase deploy --only firestore:rules
```
Make sure your `firebase.json` points at the rules file:
```json
{
  "firestore": { "rules": "firestore.rules" }
}
```
Then hard-refresh the page (Ctrl/Cmd + Shift + R). Open the browser console
(F12) — a `permission-denied` error confirms it was the rules. Also confirm in the
Firebase console under Build → Firestore Database that the database has actually
been created (not just enabled), and that it's Firestore, not Realtime
Database.
---
Part 2 — Publish it free with GitHub Pages
Create a free account at https://github.com if you don't have one.
Click New repository. Name it (e.g. `pentecost-board`), set it Public,
and click Create repository.
On the new repo page click Add file → Upload files. Drag in `index.html`
(the one you edited with your Firebase keys). Optionally also upload `README.md`
and `database.rules.json`. Click Commit changes.
Go to Settings → Pages (left menu).
Under Source, choose Deploy from a branch.
Branch: main, folder: / (root) → Save.
Wait about 1 minute, then refresh. GitHub shows your live link, e.g.
`https://YOUR-USERNAME.github.io/pentecost-board/`
Share that link. Anyone who opens it sees the live board. Anyone who makes an
edit (add a participant, tap +/−, change a goal) updates it for everyone.
---
Using the board
Add Participant / Add Candidate — buttons on each panel header.
Update counts during the month — tap the + / − on each person; the
Cupertino Zion goals update live from those counts.
Customize — title, verse, dates, point values, goal targets, accent theme.
TV Mode — full single-screen display board for the lobby TV (press Esc to exit).
All changes save to the shared database automatically and appear on every screen.
---
Updating the board later
To change the design or text, edit `index.html` and re-upload it to the repo
(Add file → Upload files, replace, commit). GitHub Pages republishes in about
a minute. Your festival data lives in Firebase, so re-uploading the HTML never
erases the participant counts.
---
Locking it down (optional)
The rules above let anyone edit. For a lobby board that's usually fine. If you want
everyone to view but only you to edit, the simplest options are:
Change the rule to `".write": false` and do edits from the Firebase console, or
Ask me to add a passcode gate so the edit buttons only work after a code is entered, or
Turn on Firebase Authentication and restrict `.write` to your account.
Just let me know which you'd prefer and I'll set it up.
