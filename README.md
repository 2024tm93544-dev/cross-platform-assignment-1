# Task Manager App

A Flutter-based mobile application that connects to Back4App as a cloud backend. The app allows users to register with their college email, manage personal tasks, and log out securely. All data is stored and retrieved from Back4App in real time without any custom server code.

---

## What This Project Does

The application gives each registered user a private list of tasks. A user can add a new task with a title and description, choose a priority level, mark it as complete, update its details, or delete it by swiping the card. Every action is saved directly to the Back4App cloud database and visible in the Back4App dashboard under the Task class.

The project was built as part of a mobile application development assignment to demonstrate practical use of Backend-as-a-Service in a Flutter application.

---

## Technology Used

**Flutter and Dart** handle the entire frontend. All screens, forms, navigation, and state management are written in Dart using Flutter widgets.

**Back4App** is the backend platform. It is a hosted Parse Server that provides user authentication, a cloud database, and an ACL-based access control system. No custom backend code was written for this project.

**parse_server_sdk_flutter** is the official Dart package used to communicate between the Flutter app and Back4App. It handles login sessions, database queries, and object persistence.

**GitHub** is used for version control and submission.

---

## Project Structure

```
lib/
    main.dart                   Application entry point, Parse initialization, splash screen
    models/
        task_model.dart         Task data class with Parse mapping methods
    services/
        parse_service.dart      All Back4App operations including auth and CRUD
    screens/
        login_screen.dart       Login form with validation
        register_screen.dart    Registration form with password confirmation
        home_screen.dart        Task list with stats, filter options, and swipe-to-delete
        task_form_screen.dart   Create and edit task form with priority selection
```

---

## Back4App Setup

Before running the app, create a free account at back4app.com and complete the following steps.

**Step 1.** Create a new app named TaskManagerApp from the Back4App dashboard.

**Step 2.** Go to App Settings, then Security and Keys. Copy the Application ID and Client Key.

**Step 3.** Open lib/main.dart and replace the placeholder values:

```dart
const String kApplicationId = 'YOUR_APPLICATION_ID';
const String kClientKey     = 'YOUR_CLIENT_KEY';
```

**Step 4.** In the Back4App dashboard, go to Database, then Browser. Create a new custom class named Task. Add these columns:

| Column Name | Type | Notes |
|---|---|---|
| title | String | Task title entered by user |
| description | String | Optional details |
| isCompleted | Boolean | Default value: false |
| priority | String | One of: low, medium, high |
| user | Pointer to _User | Links task to its owner |

**Step 5.** Click the lock icon on the Task class and set Read, Write, and Add Field permissions to Authenticated under Class Level Permissions. Save the changes.

---

## How to Run the Application

Install Flutter from flutter.dev and Android Studio from developer.android.com. Connect your Android phone via USB with USB Debugging enabled, or start an Android emulator through Android Studio.

Open a terminal in the project folder and run the following commands:

```bash
flutter pub get
flutter run
```

To verify the phone is detected before running:

```bash
flutter devices
```

---

## Application Flow

When the app opens, it checks whether a valid session token exists on the device. If one is found, the user is taken directly to the task list. If not, the login screen is shown.

On the login screen, users enter their registered college email and password. First-time users tap the register link and create an account. After registration, they return to the login screen to sign in.

The home screen displays all tasks belonging to the logged-in user. Tasks are fetched from Back4App using a query filtered by the user pointer stored on each task object. Users can filter the list by status, create a new task using the floating button, edit a task by tapping the pencil icon, toggle completion by tapping the circle on the left, or delete a task by swiping the card to the left.

Logout invalidates the session on the Back4App server and redirects to the login screen.

---

## CRUD Implementation Details

**Create.** A new ParseObject of class Task is built with the task fields, the current user set as a pointer, and an ACL restricting access to that user only. Calling save() on the object sends it to Back4App.

**Read.** A QueryBuilder on the Task class uses whereEqualTo with the user pointer to fetch only tasks belonging to the current user. Results are ordered by creation date, newest first.

**Update.** When editing a task, the existing objectId is kept on the ParseObject. Calling save() with an objectId already present on the server performs an update rather than creating a duplicate.

**Delete.** A ParseObject is constructed using only the objectId of the task to remove. Calling delete() on that object removes it from the Back4App database without needing to fetch the full object first.

---

## Issues Found and Fixed During Development

The initial version of the application had three problems that prevented any CRUD operation from working.

The first issue was in the ACL setup. The code used ParseUser.forQuery() to set the task owner, which is a placeholder object used for building queries and not the actual logged-in user. This caused every save operation to fail with a permission error. The fix was to fetch the current user explicitly using ParseUser.currentUser() and pass that to the ACL constructor.

The second issue was in the query that fetches tasks. The code used whereEqualTo on the ACL field, which is not a queryable field in Parse. Tasks were not loading at all. The fix was to store the user as a Pointer field on each task and query by that pointer instead.

The third issue was that no user pointer was being saved on task objects. This meant there was no way to associate a task with the user who created it. Adding a user pointer on creation and querying by that pointer resolved both the read and the per-user isolation requirements.

---

## Submission Details

**Name:** P Naveen Prabhath

**Email:** 2024tm93544@wilp.bits-pilani.ac.in

**YouTube Demo Link:** 

---

## Notes

The app was developed and tested on a OnePlus 12R running Android 14. The Back4App free tier was used throughout development. Debug mode is enabled in main.dart and should be set to false before any production use.