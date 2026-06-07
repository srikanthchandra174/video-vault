# Video Vault — Setup Guide

This guide rebuilds the app from scratch. It is safe to keep in a public repo — it contains **no passwords or private credentials**, only generic steps.

> Choose your own login email and a **strong** password during setup. Do **not** commit them to this repository.

---

## Prerequisites

- A Google account (for Firebase)
- A GitHub account (for hosting)
- The `index.html` app file

## 1. Create a Firebase project

1. Go to <https://console.firebase.google.com> → **Add project**.
2. Name it (e.g. `video-vault`). Analytics optional. Create.

## 2. Register a Web app & copy the config

1. In the project, click the web icon `</>` (**Add app**).
2. Give it a nickname → **Register app**.
3. Copy the `firebaseConfig` object shown.
4. Paste those values into the `firebaseConfig` block near the top of `index.html`.

> Ignore the "Install Firebase CLI / npm install / Deploy to Firebase Hosting" screens — none are needed. Click **Continue to console**. These config keys are safe to be public; security is enforced by the rules in Step 6.

## 3. Enable Firestore (the database)

1. **Build → Firestore Database → Create database**.
2. **Production mode** → pick a region (e.g. `asia-south1` Mumbai) → Enable.

## 4. Enable Email/Password authentication

1. **Build → Authentication → Get started**.
2. **Sign-in method** → **Email/Password** → Enable → Save.

## 5. Create the owner account

1. **Authentication → Users → Add user**.
2. Enter **your email** and a **strong password** — this becomes your login.
3. Use this same email as the owner in Step 6 and in the app's owner constant.

## 6. Security rules (authorization)

**Firestore Database → Rules**, replace everything with the following, set `OWNER_EMAIL_HERE` to the email from Step 5, then **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /vault/{document=**} {
      allow read:  if request.auth != null;
      allow write: if request.auth != null
                   && request.auth.token.email == "OWNER_EMAIL_HERE";
    }
  }
}
```

Any signed-in user can read the shared vault; only the owner can write. The owner email is also referenced in `index.html` (the `OWNER_EMAIL` constant) so the UI shows edit controls only to the owner.

## 7. Deploy to GitHub Pages

1. Ensure the file is named exactly `index.html` at the repo root.
2. Push to a **public** GitHub repository.
3. **Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)` → Save.**
4. Live at `https://<your-username>.github.io/video-vault/`.

## 8. Authorize your domain in Firebase

So login works on the live site:

1. **Authentication → Settings → Authorized domains → Add domain**.
2. Enter `<your-username>.github.io` → Add.

## 9. First run

1. Open the live URL → sign in with the owner account → you'll see **Owner** mode.
2. Paste a YouTube link → **Add**. Create a playlist and assign videos.
3. To add a read-only viewer, create another user in **Authentication → Users**; they will sign in to **Viewer (read-only)** mode automatically.

---

## Feature notes

- **Thumbnails & titles** load automatically from the YouTube URL (no API key). Thumbnails cascade `maxresdefault → hqdefault → mqdefault`.
- **Private vs Unlisted:** truly *Private* YouTube videos cannot be embedded or show thumbnails on external sites. Use **Unlisted** for personal videos you want in the vault (link-only, but embeddable).
- **Embedded playback** is most reliable on the deployed HTTPS URL. A "Watch on YouTube" link is provided as a fallback for videos whose owners disable embedding.

## Troubleshooting

| Symptom | Fix |
|--------|-----|
| Can't create a playlist (`permission-denied`) | Rules not updated to the `{document=**}` version (Step 6). Re-publish. |
| Login fails: "unauthorized domain" | Add `<username>.github.io` in Authentication → Settings → Authorized domains. |
| Login fails: "no user" | Create the user in Authentication → Users (Step 5). |
| Thumbnail blank for a video that plays on YouTube | The video is likely **Private** — set it to **Unlisted**. |
| Old version shows after an update | Hard-refresh (`Ctrl/Cmd + Shift + R`) or reopen the tab. |

## License

Released under the [MIT License](LICENSE).
