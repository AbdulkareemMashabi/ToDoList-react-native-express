# ToDoList (React Native + Express)

A cross-platform (iOS + Android) to‑do app built with React Native, backed by a custom Node/Express REST API instead of Firebase. Tasks support due dates, colors, and sub‑tasks; a favorite task can be pinned to a native Home Screen / Lock Screen widget, checked off from the calendar, and celebrated with a small animation when it's done.

This is a variant of [ToDoList-react-native-firebase](https://github.com/AbdulkareemMashabi/ToDoList-react-native-firebase) with the Firebase Auth/Firestore backend swapped out for a self-hosted Express API (JWT auth over plain REST endpoints). See also the native SwiftUI version: [ToDoListIos](https://github.com/AbdulkareemMashabi/ToDoListIos).

## Features

- **Guest or account login** — sign up with a hashed/encrypted device ID for anonymous use, or create a real account (email/password) to sync across devices.
- **JWT-based auth** — the API returns a bearer token on login/signup, stored in `react-native-encrypted-storage`; a 401 response from any request automatically clears the session and redirects to a non-dismissible login screen.
- **Password reset & account deletion** self-service flows.
- **Tasks & sub‑tasks** — create tasks with a title, description, due date, and a color tag; break them down into sub‑tasks that roll up into the parent task's completion state.
- **Favorite task widget** — mark a task as favorite to surface it on a native Home Screen widget (iOS + Android) and an iOS Lock Screen widget, kept in sync via an App Group (iOS) / shared storage (Android) whenever the favorite task changes.
- **Calendar sync** — link a task to the device calendar (`react-native-calendar-events`) when it's created, and remove the event automatically when the task is deleted.
- **Swipe actions** — swipe a task row to see details, delete, or toggle favorite.
- **Localization** — English and Arabic, with automatic RTL layout switching (and an in‑app restart to apply it).
- **Feedback & polish** — Lottie splash/loading/press animations, a "task completed" celebration animation, a completion sound effect, toast messages, and a shared confirmation pop‑up.
- **Offline‑aware** — network status is tracked via `@react-native-community/netinfo`.

## Tech stack

- **Framework:** React Native 0.72 (bare workflow, not Expo)
- **State:** Redux Toolkit
- **Navigation:** React Navigation (native stack)
- **Backend:** a separate Node/Express REST API (not included in this repo) — auth, tasks, and sub‑tasks all go through JSON endpoints under `/auth/*` and `/task/*`, secured with a `Bearer` JWT
- **Local storage:** `react-native-encrypted-storage` for the auth token
- **Forms & validation:** Formik + Yup
- **Native widget bridges:** a custom `WidgetRefresh` native module (iOS, Swift/WidgetKit) and `RNSharedWidget` (Android, Java) to push the favorite task to each platform's widget
- **Other notables:** `react-native-calendar-events`, `lottie-react-native`, `react-native-sound`, `react-native-actions-sheet`, `react-native-toast-message`, `i18n-js`, `crypto-js` (client-side AES encryption of credentials/device ID before they're sent to the API)
- **Language:** JavaScript (screens/components) with some TypeScript (`localization.ts`, `tsconfig.json`)

> Note: `firebase` is still listed in `package.json` but isn't imported anywhere in `src/` — auth and task storage go through `authServices.js` / `taskServices.js` to the Express API instead. It can likely be removed as an unused dependency.

## Project structure

```
ToDoList-react-native-express/
├── src/
│   ├── App/                    # Root navigator (App.js) and screen-transition config
│   ├── Screens/                # Lottie splash, Login, Register, ForgetPassword,
│   │                           #   Dashboard, CreateNewTask, TaskDetailsScreen,
│   │                           #   AccountDeletion
│   ├── Components/              # Reusable UI: Task, SubTask, Swipeable, Form,
│   │                           #   TextField, PasswordInput, DatePicker, PopUp…
│   ├── helpers/
│   │   ├── authServices.js     # signUp / login / guest signup / delete-account (REST)
│   │   ├── taskServices.js     # CRUD + favorite/status updates for tasks (REST)
│   │   ├── localization.ts     # i18n-js wrapper
│   │   ├── utils.js            # callAPI fetch wrapper, token storage, widget data sync
│   │   └── Redux/               # Redux Toolkit store & main slice
│   ├── Language/                # en.json / ar.json translation files
│   └── assets/                  # Icons, images, Lottie JSON files
├── ios/                         # Xcode project, ToDoAppWidget (Home Screen widget)
│                                #   and lock_screen_widget (Lock Screen widget)
├── android/                     # Gradle project, ToDoListWidget (App Widget)
└── __tests__/                   # Jest tests
```

## Prerequisites

- Node.js >= 16
- A working [React Native environment](https://reactnative.dev/docs/set-up-your-environment) for the platform(s) you're targeting (Xcode + CocoaPods for iOS, Android Studio/SDK for Android)
- The companion Express API running and reachable (this repo only contains the client)

## Getting started

1. **Clone and install JS dependencies**

   ```bash
   git clone https://github.com/AbdulkareemMashabi/ToDoList-react-native-express.git
   cd ToDoList-react-native-express
   npm install
   ```

2. **Create a `.env` file** in the project root (git‑ignored, loaded via `react-native-dotenv`):

   ```
   PRIVATE_KEY=            # shared AES key used to encrypt credentials/device ID client-side
   STORAGE_KEY=            # AsyncStorage key the widget reads shared task data from
   APP_GROUP_KEY=          # iOS App Group identifier shared with the widget extension
   ```

3. **Point the app at your API.** The base URL is currently hardcoded to `http://10.0.2.2:8080` (the Android emulator's alias for your machine's `localhost`) in `src/helpers/authServices.js` and `src/helpers/taskServices.js`. Update these to your API's actual host — e.g. `http://localhost:8080` for an iOS simulator, or your deployed API's URL for a real device/production build.

4. **iOS only — install CocoaPods**

   ```bash
   cd ios && bundle install && bundle exec pod install && cd ..
   ```

   In Xcode, make sure the main app, `ToDoAppWidget`, and `lock_screen_widget` targets are all signed with your team and share the same App Group capability (matching `APP_GROUP_KEY`).

5. **Run the app**

   ```bash
   npm run ios       # or: npm run android
   ```

   To try the widget, add it to the Home Screen (iOS/Android) or Lock Screen (iOS) after running the app and marking a task as favorite.

## Scripts

| Command | Description |
| --- | --- |
| `npm start` | Start the Metro bundler |
| `npm run ios` | Build and run the iOS app |
| `npm run android` | Build and run the Android app |
| `npm run lint` | Run ESLint |
| `npm test` | Run Jest tests |

## Localization

Strings live in `src/Language/en.json` and `src/Language/ar.json`. The in‑app language button toggles between them, flips the layout direction with `I18nManager`, and restarts the app (`react-native-restart`) to apply the change.

## Screenshots

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-08 at 23 31 08](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/79df062b-b718-4cef-b6be-e320463963eb)

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-08 at 23 31 00](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/a7e9d5db-6054-428f-8638-bca3a8342a09)

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-09 at 00 10 35](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/11c3c03d-0941-45b3-b176-17593a43c5a6)

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-09 at 00 10 31](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/5d453396-9f6d-4b2d-9364-9fc40e869037)

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-09 at 00 10 28](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/93062ea3-ab9e-48d4-b5c2-2377ced48b7d)

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-08 at 23 28 10](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/5ff0f225-59ab-4632-acc1-496eb8ffb919)

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-08 at 23 29 36](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/862dce45-926d-44cf-ade6-d4caa8c7d88c)

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-08 at 23 29 52](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/a95025ef-978b-4f5c-b1bb-357a9bba1564)

![Simulator Screenshot - iPhone 13 Pro Max - 2024-07-09 at 00 00 12](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/fa071c32-3732-4ab0-8888-de4b8a0542c0)

## Videos

![Swipe actions demo](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/003be7b9-6abf-4f14-85ea-982a22e7c3dd)

![Task-completed animation demo](https://github.com/AbdulkareemMashabi/ToDoList/assets/106698136/12353f88-98d6-43b3-b6d4-575bf6aa333b)

## Author

Built by [Abdulkareem Mashabi](https://github.com/AbdulkareemMashabi).
