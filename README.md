# The Oracle — D&D Live Polling App

Two standalone HTML files. No build step required.

---

## Files

- `dm.html` — Dungeon Master screen (control panel + QR code)
- `audience.html` — Audience screen (button + poll voting)

---

## Setup

### 1. Create a Firebase project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** and follow the steps
3. Once created, go to **Build → Realtime Database** and click **Create database**
4. Start in **test mode** (you can add security rules later)

### 2. Get your Firebase config

1. In Firebase Console, go to **Project Settings** (gear icon)
2. Under **Your apps**, click the `</>` (Web) icon to register a web app
3. Copy the `firebaseConfig` object shown

### 3. Paste config into both HTML files

In **both** `dm.html` and `audience.html`, find this block near the top and replace the placeholder values:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

### 4. Initialize the database structure (optional)

You can manually seed the database in the Firebase Console with:

```json
{
  "crowd": {
    "total_users": 0,
    "button_count": 0
  },
  "poll": {
    "active": false
  }
}
```

### 5. Deploy

**Option A — Firebase Hosting (recommended)**
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# Set public directory to the folder containing your HTML files
firebase deploy
```
After deploy, your audience URL will be something like:
`https://your-project.web.app/audience.html`

**Option B — Any static host**
Upload both HTML files to any static file host (Netlify, GitHub Pages, Vercel, etc.). The files must be served from the same domain so the QR code links correctly.

**Option C — Local testing**
Use a local server (not file:// directly, as Firebase SDK requires HTTP):
```bash
npx serve .
# or
python3 -m http.server 8080
```

---

## How It Works

### Dungeon Master (`dm.html`)

**Control Panel tab:**
- Live crowd stats: total audience members who scanned the QR, how many pressed the button, and the press ratio
- Live poll results with animated bars (updates in real time)
- Create a poll: enter a question and 2–4 options, then click **Send Poll to Audience**
- Close the poll to reset votes and return audience screens to the button

**QR Code tab:**
- Shows a scannable QR code pointing to `audience.html`
- The URL is auto-generated based on where `dm.html` is hosted

### Audience (`audience.html`)

- On first visit, increments `crowd/total_users` (tracked per browser session)
- Shows the **Do Something** button — press it once to add to `crowd/button_count`
- When the DM sends a poll, audience screens automatically switch to the voting UI
- Each device can only vote once per poll; vote choice is locked in immediately
- When the DM closes the poll, screens return to the button (button press state is reset for the new round)

---

## Firebase Data Structure

```json
{
  "crowd": {
    "total_users": 45,
    "button_count": 8
  },
  "poll": {
    "active": true,
    "question": "Which path do you take?",
    "options": {
      "A": "Enter the cave",
      "B": "Cross the bridge",
      "C": "Turn back"
    },
    "votes": {
      "A": 12,
      "B": 13,
      "C": 17
    }
  }
}
```

---

## Security Rules (for production)

Once you're ready to go live, replace the default test rules in Firebase Console → Realtime Database → Rules:

```json
{
  "rules": {
    "crowd": {
      "total_users": { ".read": true, ".write": true },
      "button_count": { ".read": true, ".write": true }
    },
    "poll": {
      ".read": true,
      "active":   { ".write": true },
      "question": { ".write": true },
      "options":  { ".write": true },
      "votes":    { ".write": true }
    }
  }
}
```

For stricter rules, consider adding Firebase Authentication so only the DM can write to `poll/active`, `poll/question`, and `poll/options`.
