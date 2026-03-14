# Amura Tracker — Two-Repo Setup

Firebase config is held in a **private repo** and injected at build time.
The source code in this public repo contains zero secrets.

-----

## Repos

|Repo                        |Visibility |Contents                                              |
|----------------------------|-----------|------------------------------------------------------|
|`sathyan-tech/amura-tracker`|**Public** |`index.html` (placeholders), `deploy.yml`, this README|
|`sathyan-tech/amura-config` |**Private**|`firebase-config.json` (real keys)                    |

-----

## One-Time Setup

### 1. Create the private config repo

1. Go to https://github.com/new
1. Name it `amura-config`
1. Set visibility to **Private**
1. Create it (no README needed)
1. Add `firebase-config.json` with the real Firebase values (file provided separately)

### 2. Create a Personal Access Token (PAT)

The deploy workflow needs permission to read the private repo.

1. Go to https://github.com/settings/tokens?type=beta (fine-grained PAT)
1. Click **Generate new token**
1. Set **Resource owner** to `sathyan-tech`
1. Set **Repository access** → **Only select repositories** → choose `amura-config`
1. Under **Permissions → Contents** → set to **Read-only**
1. Generate and copy the token

### 3. Add the PAT as a secret in the PUBLIC repo

1. Go to https://github.com/sathyan-tech/amura-tracker/settings/secrets/actions
1. Click **New repository secret**
1. Name: `CONFIG_REPO_TOKEN`
1. Value: paste the PAT from step 2
1. Save

### 4. Push to main

That’s it. Every push to `main` in the public repo will:

1. Check out this public repo
1. Check out `amura-config` (private) using the PAT
1. Read `firebase-config.json` and inject values into `index.html`
1. Verify no `__PLACEHOLDER__` values remain (build fails if any are missed)
1. Write `version.json` for auto-refresh
1. Deploy to GitHub Pages

-----

## How It Works

```
Public repo (source):          Private repo (secret):
index.html                     firebase-config.json
  apiKey: "__FIREBASE_         {
    API_KEY__"         ──►       "apiKey": "AIzaSy...",
  authDomain: "__FIRE            "authDomain": "...",
    BASE_AUTH_DOMAIN__"          ...
  ...                          }
        │
        ▼ GitHub Actions (deploy.yml)
        
Deployed page (GitHub Pages):
  apiKey: "AIzaSy..."   ← real value, injected by Actions
  authDomain: "..."     ← never in git history
```

-----

## Security Notes

- The deployed HTML at `sathyan-tech.github.io/amura-tracker` is public
  (it’s a website) so keys are visible in browser DevTools — that’s normal
  for client-side Firebase apps. The PAT and source code remain private.
- Your **Firestore security rules** are the real protection. Even with the
  keys, nobody can read or write data without being authenticated.
- Rotate keys in Firebase Console → Project Settings if ever compromised.
- The PAT only has read access to the private config repo — minimum privilege.
