# ⚔ Minecraft Hardcore Log

A web app for tracking your Minecraft Java Edition Hardcore runs — built as a static GitHub Pages site powered by Firebase. Log runs, track deaths, time your sessions, and compete on a shared leaderboard with other players.

![Dashboard preview](https://img.shields.io/badge/status-live-brightgreen) ![Firebase](https://img.shields.io/badge/backend-Firebase-orange) ![GitHub Pages](https://img.shields.io/badge/hosted-GitHub%20Pages-blue)

---

## Features

- **Run tracking** — create runs with name, version, seed, goal, and notes
- **Live session timer** — start a timer tied to your active run, persists through page updates
- **Death logging** — record cause of death when a run ends
- **Run completion** — mark runs as finished and submit them to the leaderboard
- **Shared leaderboard** — all completed runs ranked by time, visible to every user
- **User accounts** — register with a username, email and password; data is private per user
- **Full version support** — version picker covers every Java Edition release from 1.0 to 1.21.11

---

## Tech Stack

| Layer | Technology |
|---|---|
| Hosting | GitHub Pages (static) |
| Auth | Firebase Authentication (email/password) |
| Database | Cloud Firestore |
| Frontend | Vanilla HTML/CSS/JS — single file, no build step |
| Fonts | Press Start 2P · Share Tech Mono · Nunito (Google Fonts) |

---

## Getting Started

### Prerequisites

- A [Firebase](https://firebase.google.com/) project with:
  - **Authentication** → Email/Password provider enabled
  - **Firestore Database** created (start in test mode, then apply rules below)
- A GitHub repository with Pages enabled (Settings → Pages → Deploy from branch `main`, folder `/root`)

### Setup

1. **Clone the repo**
   ```bash
   git clone https://github.com/yourusername/your-repo-name.git
   cd your-repo-name
   ```

2. **Add your Firebase config** — open `index.html` and replace the config object near the top of the `<script>` block:
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT.firebasestorage.app",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```

3. **Add your GitHub Pages domain to Firebase Auth**
   - Firebase Console → Authentication → Settings → Authorized domains
   - Add `yourusername.github.io`

4. **Push to GitHub**
   ```bash
   git add index.html
   git commit -m "Initial deploy"
   git push
   ```

Your site will be live at `https://yourusername.github.io/your-repo-name/` within a minute or two.

---

## Firestore Structure

```
/users/{uid}
  username: string
  email:    string
  created:  timestamp

/usernames/{username_lowercase}
  uid: string

/runs/{runId}
  uid:       string
  username:  string
  name:      string
  version:   string
  seed:      string
  goal:      string   // "ender" | "wither" | "elder" | "survive" | "speedrun"
  notes:     string
  status:    string   // "alive" | "dead" | "completed"
  startTime: timestamp
  endTime:   timestamp | null
  elapsed:   number   // seconds
  deaths:    array
    - cause: string
      notes: string
      time:  number

/leaderboard/{docId}
  uid:      string
  username: string
  runName:  string
  version:  string
  goal:     string
  elapsed:  number
  date:     timestamp
```

---

## Firestore Security Rules

Once you're done testing, replace the default rules in Firebase Console → Firestore → Rules:

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Users can read/write their own profile
    match /users/{uid} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == uid;
    }

    // Username reservations — anyone can read, only the owning user can write
    match /usernames/{username} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }

    // Runs — users can only access their own
    match /runs/{runId} {
      allow read, write: if request.auth.uid == resource.data.uid;
      allow create: if request.auth != null;
    }

    // Leaderboard — anyone logged in can read; only authenticated users can add
    match /leaderboard/{docId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
    }
  }
}
```

---

## Usage

### Starting a Run
1. Go to the **Dashboard** tab
2. Click **+ Start New Run**
3. Fill in the run name, version, seed (optional), and goal
4. Click **Start Run** — the timer starts automatically

### Ending a Run
From the Dashboard, your active run shows two buttons:
- **🏆 Finish** — marks the run complete and submits it to the leaderboard
- **💀 I Died** — opens a form to log the cause of death and ends the run

### My Runs Tab
View all your past and active runs. You can also mark them finished, log a death, or delete them from here.

### Leaderboard
Shows all completed runs across every user, sorted by fastest time. Updates live when new runs are completed.

---

## Known Limitations

- The session timer is local to your browser tab — if you close the tab, the timer resets (run elapsed time is still saved to Firestore every 10 seconds)
- Deleting your Firebase Auth account does not automatically clean up Firestore data — username reservations are freed automatically when someone else tries to register the same name

---

## License

MIT — do whatever you want with it.
