# Firebase Setup

The app expects Firebase Authentication and Cloud Firestore. The HTML contains the client configuration; `firestore.rules` contains the database access controls.

## 1. Add the Firebase web app config

The `firebaseConfig` block in `index.html` is now populated with the Web app config for the `keys-voice` project. If the Firebase project changes, update that block from **Project settings > General > Your apps**. Firebase web config identifies the project; it is not a server credential. Do not put service-account keys in the page. Analytics is not initialized by this app, so its `measurementId` is not needed here.

## 2. Enable Google sign-in

In **Authentication > Sign-in method**, enable Google. In **Authentication > Settings > Authorized domains**, add the exact hostname used to open the website. The `hd` OAuth parameter only suggests the school domain; the page validates the returned email and Firestore rules independently enforce it.

## 3. Create the Firestore database and deploy rules

Create the project's Cloud Firestore database if it does not exist. Install/use the Firebase CLI, sign in, and select the same project as the web config:

```powershell
npm install -g firebase-tools
firebase login
firebase use --add
firebase deploy --only firestore:rules
```

Run those commands from this folder. `firebase.json` deploys `firestore.rules`. Review the target project carefully before deploying; rules apply to that project's Firestore database.

## 4. Keep the admin list in sync

The exact admin email list is in two places: `ADMIN_EMAILS` in `index.html` controls the page, and `isAdmin()` in `firestore.rules` controls database permissions. Update both lists whenever admins change.

## 5. Publish the website

The site is a static HTML page. `firebase.json` serves the root `index.html` using Firebase Hosting and excludes the setup/rules files. From this folder, deploy it with:

```powershell
firebase deploy --only hosting
```

Share the Hosting URL printed by the CLI, normally `https://keys-voice.web.app`. A `localhost` URL only works on the computer running the local server; it is not shareable with friends.

## 6. Verify sign-in and saving

Test with a listed admin, a school-domain account not on the admin list, and an account outside `@keysschool.org`. Confirm that the first can create/archive polls and moderate submissions, the second can submit ideas/feedback and vote, and the outside account is signed out and cannot read/write Firestore. Test a page refresh to confirm data is coming back from Firestore.

Failed Firestore writes now show an error instead of claiming the data was saved locally. Firestore is the shared source of truth; data that existed only in a browser's old `kms_*` localStorage keys is not automatically migrated.

## Current limitations

Poll option counts and idea upvotes are shared counters, not one-vote-per-account records. The Firestore rules limit which fields students can change and require upvotes to increase by one, but a school account can still vote repeatedly or alter poll option content. Preventing repeat votes requires storing individual votes keyed by user and aggregating them separately.
