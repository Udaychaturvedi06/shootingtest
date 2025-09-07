
Lucknow Shooting Academy — Admin & Student Webapp (Firebase, plug-and-play)

Drop these alongside your existing `index1.html`.

Files
- login.html — sign in / sign up (students by default; admins via Admin Access Key)
- admin.html — manage Gallery (Firebase Storage) + News (Firestore) + toggle student status
- student.html — profile + announcements

Setup
1) Create a Firebase project; enable Email/Password auth.
2) Create Firestore + Storage.
3) In each HTML file, fill the firebaseConfig with your values.
4) Optional: apply the Security Rules included below.

Security Rules (starter)
Firestore:
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isSignedIn() { return request.auth != null; }
    function isAdmin() {
      return isSignedIn() && get(/databases/{database}/documents/users/$(request.auth.uid)).data.role == "admin";
    }
    match /users/{uid} {
      allow read: if isSignedIn() && (uid == request.auth.uid || isAdmin());
      allow update: if isSignedIn() && uid == request.auth.uid;
      allow create: if true;
    }
    match /news/{id} {
      allow read: if true;
      allow create, update: if isAdmin();
    }
    match /gallery/{id} {
      allow read: if true;
      allow create, update: if isAdmin();
    }
    match /queries/{id} {
      allow create: if true;
      allow read: if isAdmin();
    }
  }
}

Storage:
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    function isAdmin() {
      return request.auth != null && 
        get(/databases/(default)/documents/users/$(request.auth.uid)).data.role == "admin";
    }
    match /gallery/{allPaths=**} {
      allow read: if true;
      allow write: if isAdmin();
    }
  }
}

Routing
- After login: admin -> admin.html, student -> student.html
- Front site index1.html already reads news & gallery; admin changes reflect live.
