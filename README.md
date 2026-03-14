# ⚔ Minecraft Hardcore Log

A public live dashboard for tracking co-op Minecraft Hardcore runs between **belleke03**, **zwana** and **Guntter78NL** — powered by Firebase and a Paper server plugin.

**Live site:** [emmaoudhof.github.io/Minecraft-Hardcore-Log](https://emmaoudhof.github.io/Minecraft-Hardcore-Log/)

---

## What it does

The dashboard updates automatically — no buttons, no manual input, everything is driven by the server mod.

| Event | What happens |
|---|---|
| `belleke03` or `zwana` joins the server | New run created, timer starts |
| Either tracked player dies | Run marked as 💀 Dead, death message shown |
| Ender Dragon is killed | Run marked as 🏆 Completed, added to leaderboard |
| Every 30 seconds | Elapsed time saved to Firebase |

Anyone can view the dashboard — no login required.

---

## Pages

- **Dashboard** — live view of the current run with a real-time ticking timer, player chips, death messages, and overall stats
- **Runs** — full history of every attempt (alive, dead, completed)
- **Leaderboard** — completed runs ranked by fastest time

---

## Tech stack

| | |
|---|---|
| Hosting | GitHub Pages (static) |
| Database | Cloud Firestore (public read) |
| Server mod | Paper plugin (writes to Firestore via REST API) |
| Frontend | Vanilla HTML/CSS/JS — single file, no build step |

---

## Repository structure

```
/
├── index.html              ← the entire frontend (single file)
└── hardcore-tracker/       ← Paper plugin source
    ├── pom.xml
    ├── README.md
    ├── hardcore_tracker.json
    └── src/main/java/com/hardcoretracker/
        ├── HardcoreTrackerPlugin.java
        ├── FirebaseClient.java
        └── TrackerConfig.java
```

---

## Server plugin setup

The plugin is a Paper server-side mod. Players do not need to install anything.

### Requirements
- Paper server 1.21.1
- Java 21
- Maven (to build)

### Build
```bash
cd hardcore-tracker
mvn clean package
# Output: target/hardcore-tracker-1.0.0.jar
```

### Install
1. Copy `hardcore-tracker-1.0.0.jar` to the server's `plugins/` folder
2. Restart the server — it generates `plugins/HardcoreTracker/config.yml`
3. Fill in `owner_uid` (see below) and restart again

### Config (`plugins/HardcoreTracker/config.yml`)
```yaml
firebase:
  project_id: "minecraft-hardcore-log"
  api_key: "your-firebase-api-key"

tracked_players:
  - "belleke03"
  - "zwana"
  - "Guntter78NL"

run:
  owner_uid: "your-firebase-uid"
  owner_username: "Emma"
  version: "1.21.1"
  goal: "ender"
```

`run.name` is set **automatically** — the plugin counts existing runs in Firestore and names the next one `Attempt N`. You never need to change anything between attempts.

### Finding your Firebase UID
Since the site no longer has a login page, the easiest way is:
1. Go to **Firebase Console → Authentication → Users**
2. Find your account and copy the **User UID** column

---

## Firebase setup

### Firestore rules
The dashboard is fully public — no login needed to view. Set these rules in **Firestore → Rules**:

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read:  if true;
      allow write: if true;
    }
  }
}
```

### Firestore collections

```
/runs/{runId}
  uid            string   — owner Firebase UID
  name           string   — "Attempt 1", "Attempt 2", etc. (auto-set by plugin)
  version        string   — Minecraft version
  goal           string   — ender | wither | elder | survive | speedrun
  seed           string
  notes          string
  status         string   — alive | dead | completed
  startTime      number   — epoch ms
  endTime        number   — epoch ms (null while alive)
  elapsed        number   — seconds (saved every 30s by plugin)
  lastDeath      string   — death message from plugin
  trackedPlayers array    — ["belleke03", "zwana", "Guntter78NL"]

/leaderboard/{docId}
  uid            string
  runName        string
  version        string
  goal           string
  elapsed        number   — seconds
  date           number   — epoch ms
```

---

## Starting a new attempt

Nothing. Just join the server.

The plugin automatically counts how many runs exist in Firestore and names the next one `Attempt N`. The dashboard updates live the moment the run is created.
