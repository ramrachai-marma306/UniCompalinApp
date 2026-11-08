# UniFix Firebase setup

This project assumes Firebase project `universitycomplain` and its `(default)` Cloud Firestore database in Standard mode.

## Enable Firebase services

In the Firebase Console for project `universitycomplain`:

1. Authentication → Sign-in method → enable Email/Password.
2. Firestore Database → create or select the `(default)` database in Standard mode.
3. Storage → get started and select a bucket in the same region as Firestore where possible.
4. Confirm the Android app registered in Firebase uses application ID `com.example.universitycomplain`. Keep `app/google-services.json` private to this project and do not post its API key publicly.
5. Update the email verification template and sender details before distributing the app.

The Android app uses Firebase Auth, Firestore, Storage and Messaging SDKs. Client registration only creates a student profile. Admin authority is a Firebase Auth custom claim set with the Admin SDK; there is no admin registration UI.

## Deploy rules and notification functions

Install Node.js 20 or newer and Firebase CLI, authenticate with an account that can deploy to the project, then run from the project root:

```powershell
npm --prefix functions install
npm --prefix functions run build
firebase deploy --only firestore:rules,storage,functions --project universitycomplain
```

Cloud Functions require a billing-enabled Firebase project (Blaze plan) for deployment. The app can compile without deploying functions, but server-triggered push notifications and the callable admin-role manager will not run until deployment succeeds.

## Grant the first administrator

1. Register the person through the app as a student and have them verify their email.
2. Use a trusted operator machine with Application Default Credentials for the Firebase project (or set `GOOGLE_APPLICATION_CREDENTIALS` to a service-account key kept outside the repository).
3. Run `npm --prefix functions run bootstrap-admin -- <verified-user-uid>`.

The bootstrap script updates the profile role through Admin SDK and then sets the protected `admin` custom claim. Never ship service-account credentials in the Android app.

## Data paths

- `Users/{uid}`: private student profile.
- `Users/{uid}/Tokens/{sha256(fcmToken)}`: device token, owner write-only for client.
- `Complaints/{complaintId}`: student submission and admin-managed status/assignment fields.
- `Complaints/{complaintId}/StatusHistory/{eventId}`: admin-authored audit events.
- `Feedback/{complaintId}`: one student feedback per resolved complaint.
- `Notifications/{notificationId}`: server-created notification, owner can mark read.

Images use `complaints/{uid}/{complaintId}/problem`. The app downloads images through the authenticated Storage SDK; it does not publish public download-token URLs.
