# Mixtape

A tiny site where people submit Spotify/YouTube links, you approve them, and
anyone can pull a random one back out. Static site + Firebase (Firestore +
Auth), meant to be hosted on GitHub Pages.

## Files

- `index.html` — public page: submit form + random picker
- `admin.html` — private moderation queue (login required)
- `style.css` — shared styling
- `app.js` — logic for `index.html`
- `admin.js` — logic for `admin.html`
- `firebase-config.js` — **you edit this** with your project's keys
- `firestore.rules` — paste this into the Firebase console
- `favicon.svg`

## 1. Firestore

In your Firebase project, open **Build → Firestore Database** and create a
database if you haven't already (production mode is fine — the rules below
handle access control). You don't need to create the `tracks` collection by
hand; it's created automatically the first time someone submits a track.

Open the **Rules** tab and replace the contents with everything in
`firestore.rules` from this folder, then **Publish**. Rules summary:

- Anyone can **submit** a track, but it's always forced to `status: "pending"`.
- Anyone can **read** a track once it's `approved`.
- Only a **signed-in** user (you) can approve or delete a track, or read the
  pending queue.

## 2. Authentication (your admin login)

Open **Build → Authentication → Sign-in method**, enable **Email/Password**.
Then go to the **Users** tab and **Add user** — use whatever email/password
you want to log into `admin.html` with. This is the only account that should
exist; there's no public sign-up.

## 3. Get your web config

Go to **Project settings** (gear icon) → **General**, scroll to **Your apps**.
If you don't have a web app yet, click **Add app → Web** (the `</>` icon) and
register one — you don't need Firebase Hosting, just the config object.

Copy the six values (`apiKey`, `authDomain`, `projectId`, `storageBucket`,
`messagingSenderId`, `appId`) into `firebase-config.js`, replacing the
placeholders. These values are meant to be public — they identify your
project, they don't grant access on their own. The Firestore rules are what
actually protect your data.

## 4. Put it on GitHub Pages

1. Create a new GitHub repo and push all the files in this folder to it
   (root of the repo, no subfolder).
2. In the repo, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to "Deploy from a branch",
   pick your default branch and `/ (root)`, then **Save**.
4. GitHub gives you a URL like `https://yourname.github.io/repo-name/`.
   Give it a minute after each push to rebuild.

## 5. Using it

- Public site: `yoursite/index.html` (or just the root URL).
- Moderation: `yoursite/admin.html` — sign in with the account from step 2.
  It's not linked anywhere except a small "Admin" link in the footer, so
  it's easy to find for you but not advertised.
- New submissions show up live in the **Pending review** list on the admin
  page — approve or reject each one. Approved tracks move to **Live in the
  pool** and become eligible for the random picker.

## Testing locally

Because the scripts use ES module imports, opening `index.html` directly as
a `file://` URL won't work — browsers block module imports over `file://`.
Run a tiny local server from the project folder instead, e.g.:

```
npx serve .
```

or

```
python3 -m http.server 8000
```

then open the printed `localhost` address.

## Notes / possible upgrades later

- The random picker currently fetches all approved tracks and picks one
  client-side. That's fine for a community-sized pool (hundreds of tracks);
  if it ever grows into the thousands, an admin-maintained counter + random
  offset query would scale better.
- Title/artist are entered manually on submit for reliability. If you want
  auto-filled titles later, Spotify and YouTube both have oEmbed endpoints
  you could call from `app.js` before writing the doc.
