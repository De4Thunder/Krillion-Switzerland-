# Krillion Crew: setup and maintenance

Live site: https://de4thunder.github.io/Krillion-Switzerland-/

## Database rules (v2)
Firebase console > Firestore > Rules: replace everything with this and tap Publish.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function okName(n) { return n is string && n.size() > 0 && n.size() <= 20; }

    match /players/{id} {
      allow read: if true;
      allow create, update: if okName(request.resource.data.name);
      allow delete: if true;
    }
    match /days/{day} {
      allow read: if true;
      allow create, update: if request.resource.data.day is int
                            && string(request.resource.data.day) == day;
      allow delete: if false;
    }
    match /meta/admin {
      allow read: if true;
      allow create: if request.resource.data.pinHash is string;
      allow update, delete: if false;
    }
    match /meta/group {
      allow read: if true;
      allow create, update: if request.resource.data.title is string
                            && request.resource.data.title.size() <= 30;
    }
    // Old storage from v1. Kept so older copies of the app keep working while scores move over.
    match /scores/{id} {
      allow read, delete: if true;
      allow create, update: if request.resource.data.score is int
                            && request.resource.data.day is int;
    }
  }
}
```

## How the data is stored
- `players`: one document per diver (name, color, avatar).
- `days`: one document per Krillion day, holding everyone's result for that day, plus reactions and chat.
  One document per day keeps reads low: a visit reads about one document per day played, not one per score.
- `meta/group`: the group name. `meta/admin`: the admin PIN, stored only as a hash.
- `scores`: the old v1 format. The app moves anything in there into `days` automatically.

## Admin PIN
The first person to set a PIN in Settings becomes the admin. To reset it, delete the document
`meta/admin` in Firebase > Firestore > Data, then set a new one in the app.
The PIN protects against accidents, not against someone who knows how to edit a web page.

## Files in the GitHub repo
- `index.html`: the whole app
- `manifest.webmanifest`, `sw.js`, `icon-*.png`: make it installable as a home-screen app
