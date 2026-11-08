# Firebase image storage change review

## Assumptions and affected paths

- Target project is `universitycomplain`; the existing setup guide targets its `(default)` Cloud Firestore database and Standard mode. Firebase CLI/Node is unavailable here, so the live database edition and deployed rules could not be queried.
- Student uploads use `Complaints/{complaintId}` and `complaints/{uid}/{complaintId}/problem` in Firebase Storage.
- Complaint ownership uses the immutable `studentUid` field. New complaint docs store `imageUrl: ""` until Storage upload and URL persistence both succeed. `imageBase64` remains blank for compatibility.
- My Complaints and Admin Dashboard both use `ComplaintAdapter`. Complaint detail uses `ComplaintDetailActivity` for both roles.

## Changes reviewed

- Upload the selected URI to Firebase Storage, request `getDownloadUrl()`, then update Firestore `imageUrl` and `updatedAt`.
- Upload or URL update failures remove the Storage object and pending Firestore draft where permissions permit; the app reports failure rather than success.
- Glide loads the Firestore URL in list thumbnails and detail views. Detail view uses fit-center scaling, a placeholder/error drawable, and readable state text for absent or invalid images. Existing Storage-path records can still request a download URL.
- Firestore rules allow a verified owner to add one Firebase Storage HTTPS URL to their own Pending complaint while preserving all other fields. Admin status/remarks updates preserve the image URL. Storage reads/uploads are scoped to the owner/admin, and cleanup deletion is limited to the owner of a Pending complaint.
- Firestore and Storage unmatched paths remain default-deny.

## Adversarial review (source-level)

- Unauthenticated or unverified callers: denied by `verified()` / `isStudent()` checks.
- Other student attaching a URL or uploading to another UID path: denied by UID ownership and linked complaint checks.
- Owner changing title, status, department, or remarks in the URL-only update: denied by the affected-key allowlist and unchanged-field checks.
- Owner replacing an existing URL: denied because only an empty `imageUrl` can be changed.
- Non-Firebase URL or oversized URL: denied by HTTPS Firebase Storage host validation and the 2048-character cap.
- Cleanup deletion of resolved complaint attachment: denied by the Pending-status check.
- Missing/invalid URL in the UI: Glide fallback/error placeholder; invalid host is not passed to Glide.

## Validation limits

- `:app:assembleDebug` passed after the Java/XML changes.
- Firebase Rules Emulator and live Firebase deployment were not run. The CLI query `npx -y firebase-tools@latest firestore:databases:list --project universitycomplain` could not run because `npx` is not installed. Deploy the updated `firestore.rules` and `storage.rules` before testing image upload on Firebase; review these prototype rules before broadly sharing the app.
