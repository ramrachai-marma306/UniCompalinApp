# UniFix Firestore Security Audit Notes

Scope: Android client source under app/src/main/java/com/example/universitycomplain and the requested UniFix data model. Target is the Firebase default database in Standard mode (assumed because no database ID was provided and Firebase CLI is unavailable in this environment).

## Collections and document shapes

- Users/{uid}: uid, fullName, studentId, department, email, phone, role, createdAt. Private profile; only owner reads. Client registration may create role=student only. Admin role is represented by a trusted Firebase Auth custom claim and synchronized to this profile only by Admin SDK code.
- Users/{uid}/Tokens/{sha256(token)}: token, updatedAt. Owner writes; no client reads. Admin SDK notification sender reads it.
- Complaints/{complaintId}: complaintId, studentId (Auth uid), studentName (copied from owner profile), title, category, description, department, location, imageUrl (empty for private SDK-mediated image download), imagePath (complaints/{uid}/{complaintId}/problem or empty), assignedDepartment, status, adminRemarks, createdAt, updatedAt.
- Complaints/{complaintId}/StatusHistory/{eventId}: status, assignedDepartment, remarks, updatedBy (admin uid), createdAt.
- Feedback/{complaintId}: feedbackId, complaintId, studentId, rating (integer 1..5), comment, createdAt. Document ID equals complaint ID, allowing at most one feedback per complaint.
- Notifications/{notificationId}: notificationId, userId, title, message, isRead, createdAt. Admin SDK only creates records; owner reads and marks read.

## Queries and operations found in Java source

- AdminDashboardActivity: Complaints orderBy createdAt descending, limit 100; aggregate count of all complaints; aggregate count where status == Resolved.
- StudentDashboardActivity: Users/{uid} get; Complaints where studentId == current uid (all matching docs).
- MyComplaintsActivity: Complaints where studentId == current uid (client-side search over complaint ID/title/status/category).
- ComplaintDetailActivity: get Complaints/{id}; get its StatusHistory subcollection; get Feedback/{complaintId}; admin batch-updates complaint and creates history event; owner creates Feedback/{complaintId}.
- RegisterActivity, ProfileActivity, AuthRouter: get/create Users/{uid}; no client-selected role values are accepted other than student.
- NotificationsActivity: Notifications where userId == current uid, limit 100; owner updates only isRead.
- UniFixMessagingService: owner writes Users/{uid}/Tokens/{sha256(token)}.
- ComplaintFormActivity: reads owner profile and creates Complaints/{generatedId}.
- Storage paths: complaints/{uid}/{complaintId}/problem; read/write is scoped to a verified owner or trusted admin and linked to the Firestore complaint.

## Constraints and authorization assumptions

Strings have explicit client and Rules limits; timestamps use serverTimestamp and Rules require request.time. Student complaint creation requires verified email, a matching student profile, owner UID, exact allowed category, and initial Pending status. Only verified users with the Auth custom claim admin=true can administer complaints. User-supplied profile role is never an admin authority. Status changes are Pending -> In Progress/Rejected and In Progress -> Resolved/Rejected; admin remarks/assignment updates may keep status unchanged. Client deletes are denied. Notifications are emitted by Cloud Functions using Admin SDK and FCM, which bypass client Rules by design.
