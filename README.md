# University Complain App

A Java and XML Android app for university students to submit and track campus complaints, with a separate administrator dashboard.

## Author

Add your name and student ID before submitting this project.

## Features

- Firebase Authentication: registration, email verification, login, password reset, remember-email option, and logout.
- Student dashboard with live complaint counts and recent reports.
- Complaint create, read, edit, and delete; editing and deletion are limited to the owner's Pending reports.
- Realtime complaint status, history, notifications, admin search and filters.
- Admin assignment, status updates, remarks, dashboard statistics, and student call action.
- Gallery photo picker; complaint images upload to Firebase Storage, with the download URL saved to Firestore and rendered by Glide in lists and details.
- Feedback after resolution, shake-to-refresh, share action, and configurable university website link.
- Firestore security rules restrict ownership and admin operations.

## Open and run

1. Open this folder in Android Studio.
2. In Firebase Console, register Android app ID `com.example.universitycomplain`.
3. Place that Firebase project's `google-services.json` at `app/google-services.json`. The file is ignored by Git so project-specific keys are not committed.
4. Enable Email/Password in Firebase Authentication and create the project's (default) Cloud Firestore database.
5. Deploy `firestore.rules` before testing student or admin data operations. See [FIREBASE_SETUP.md](FIREBASE_SETUP.md).
6. Select a connected phone or emulator and press **Run**, or build using `gradlew.bat :app:assembleDebug`.

Firebase Storage must be enabled and its rules deployed to submit complaints with photos. The app stores only a download URL in Firestore; it does not save image bytes there. FCM and Cloud Functions are optional integrations.

## Manual verification checklist

- Register a student, verify the email, then sign in and confirm Remember Me only saves the email.
- Submit a complaint with and without an image; check Firestore and the details screen.
- Confirm lists and counts update after changing a complaint status.
- Confirm students can edit/delete only their own Pending complaints.
- Provision a trusted admin, confirm search/filter/assignment/status/remarks, and test the phone dialer.
- Resolve a complaint and submit one 1–5 rating plus feedback.
- Test the shake refresh, complaint share sheet, and university website link.

The app can build without Firebase Console setup, but Firebase-backed workflows require a correctly configured Firebase project and deployed security rules.
