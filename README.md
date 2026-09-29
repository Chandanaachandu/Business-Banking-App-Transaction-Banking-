# Business Banking App (Transaction Banking)

Native mobile banking app for a **bank in Denmark**, made for **business customers** ("Banking for modern businesses").
Companies can view their accounts and transactions, pay recipients in Denmark and abroad, **approve payments and recipients**, manage loans, and message the bank securely. Login uses **MitID** or **Freja eID**.

The app is built natively on **both platforms**:

| Platform | Language | UI | IDE | Architecture |
|---|---|---|---|---|
| **Android** (this repository) | Kotlin | XML layouts + ViewBinding, Material Components | Android Studio | MVVM + Clean Architecture |
| **iOS** | Swift | SwiftUI | Xcode | MVVM |

Both apps use the same backend REST APIs, the same login and approval flows, and the same design, and both support **English and Danish**.

---

## Screenshots

| Splash | Login – Denmark | Region selection | Login – Europe |
|:---:|:---:|:---:|:---:|
| <img src="screenshots/01_splash.png" width="190"/> | <img src="screenshots/02_login_denmark.png" width="190"/> | <img src="screenshots/03_region_select.png" width="190"/> | <img src="screenshots/04_login_europe.png" width="190"/> |

<!-- The screens below require a MitID / Freja eID login. Add them to /screenshots and uncomment the rows.
| Overview | Account details | Transfer | Recipients |
|:---:|:---:|:---:|:---:|
| <img src="screenshots/05_overview.png" width="190"/> | <img src="screenshots/06_account_details.png" width="190"/> | <img src="screenshots/07_transfer.png" width="190"/> | <img src="screenshots/08_recipients.png" width="190"/> |

| Add recipient | Pending approvals | Loans | Messages |
|:---:|:---:|:---:|:---:|
| <img src="screenshots/09_add_recipient.png" width="190"/> | <img src="screenshots/10_pending_approvals.png" width="190"/> | <img src="screenshots/11_loans.png" width="190"/> | <img src="screenshots/12_messages.png" width="190"/> |
-->

---

## Features

### Login and security
- Choose a **region**: *Denmark* (MitID or Freja eID) or *Europe* (Freja eID)
- **OpenID Connect login via Criipto** in a secure Chrome Custom Tab, returning to the app through an Android App Link
- The same secure flow is used to **approve payments and recipients** (strong customer authentication)
- Access and refresh tokens, with automatic token refresh
- Session data stored in **EncryptedSharedPreferences** (AES-256)
- **Privilege-based access**: features are shown or hidden based on the user's rights, for example viewing recipients or approving payments

### Overview and accounts
- Overview of the company's **account cards** and recent transactions
- **Paged transaction list** grouped by date, with account details
- **Transaction search**
- **Receipt / transaction advice**, which can be downloaded as a PDF and shared

### Payments and transfers
- **Transfer to a recipient** or **transfer between own accounts**
- Amount entry with an **exchange-rate (FX) lookup** for foreign currencies
- **Schedule payments** with a calendar date picker
- Insufficient-funds warning and a payment success screen
- **Upcoming payments**: view, edit (amount or date), delete or decline

### Approvals
- **Pending approvals** for payments, with single or **bulk** approval
- Approve or decline with a **swipe-to-confirm** button

### Recipients (beneficiaries)
- Tabs: **All, Favourites, Recent, Pending approval**, and mark recipients as favourite
- Multi-step **add recipient** flow: Corporate or Private → country → currency → bank details → address → extra information → summary, then swipe to create
- **IBAN validation** and **SWIFT/BIC bank lookup**
- Edit, delete, approve or reject recipients

### Loans
- Loan list, loan details with repayment progress, and **loan documents** (download)

### Secure messages
- Inbox with **All / Unread / Archive** tabs and an unread badge
- Write new messages by topic and chat with the bank in a message thread
- Mark as read and archive several messages at once

### Profile and general
- **Switch between client profiles** (companies), see admin status, app version, and log out
- **Push notifications** with Firebase Cloud Messaging
- **English and Danish**, following the device language
- **Offline detection** with a "no internet" dialog
- Environments: **QA, UAT, Prod**

---

## Tech stack – Android

| Area | Technology |
|---|---|
| Language | **Kotlin** 2.0 (JVM 11) |
| IDE / build | **Android Studio**, Gradle Kotlin DSL (AGP 8.7), version catalog |
| SDK | minSdk 26, target / compile SDK 36 |
| Architecture | **MVVM + Clean Architecture**: data / domain / presentation layers, about 35 use cases, sealed `UiState` classes |
| Dependency injection | **Dagger-Hilt** |
| Asynchronous work | **Kotlin Coroutines**, **Flow / StateFlow**, LiveData |
| Networking | **Retrofit** 2.11, **OkHttp** (auth interceptor with token refresh, no-network interceptor, logging), **Gson** |
| Lists | RecyclerView, **Paging 3** (transactions, recipients, conversations, pending payments) |
| Jetpack / AndroidX | ViewModel, Lifecycle, Fragment, ViewBinding, DataBinding, Browser (Custom Tabs), **Security-Crypto** |
| Authentication | **Criipto (OpenID Connect)** with **MitID** and **Freja eID**, Android App Links |
| Firebase | **Cloud Messaging (FCM)**, Analytics |
| UI | Material Components, BottomNavigationView, bottom sheets, **Glide**, AndroidSVG, **Lottie**, swipe button, calendar view, dots indicator |
| Testing | **JUnit 4, MockK, kotlinx-coroutines-test** (ViewModel and use case tests), Espresso |

## Tech stack – iOS

| Area | Technology |
|---|---|
| Language | **Swift** |
| IDE | **Xcode** |
| UI | **SwiftUI**, with reusable components and property wrappers for state management |
| Architecture | **MVVM** |
| Networking | **URLSession** with **async/await**, **Codable** |
| Security | Keychain, OpenID login with `ASWebAuthenticationSession` |
| Push | Firebase Cloud Messaging / APNs |
| Environments | Xcode build configurations / schemes (QA, UAT, Prod) |
| Distribution | **TestFlight** via App Store Connect |

---

## Project structure (Android)

```
app/src/main/java/<package>/
├── App.kt                    # Application class (Hilt)
├── MainActivity.kt           # Bottom navigation host
├── data/
│   ├── data_source/          # Retrofit API interface, Paging sources
│   ├── local/                # SessionManager (encrypted storage)
│   ├── remote/               # AuthInterceptor (headers, token refresh)
│   └── repository/           # Repository implementation
├── domain/
│   ├── model/                # Request / response models
│   ├── repository/           # Repository interface
│   └── use_cases/            # One use case per action
├── di/                       # Hilt modules (Retrofit, Repository)
├── fcm/                      # Firebase Messaging service
├── network/                  # Network monitor, no-internet handling
├── presentation/             # Activities, Fragments, ViewModels, bottom sheets
│   ├── login/  home/  accounts/  search/  receipt/
│   ├── transfer/  upcomingPayment/  editPayment/  pendingApproval/
│   ├── recipients/  createRecipient/  editRecipient/
│   ├── loans/  messages/  createMessage/  chat/
│   └── notification/  profile/
└── utils/                    # Helpers, validation
```

**Data flow:** Fragment/Activity → ViewModel (`StateFlow<UiState>`) → UseCase (Flow) → Repository → Retrofit API.

**Bottom navigation:** Overview · Payments (approvals) · Transfer · Recipients · Profile

---

## Getting started (Android)

1. Clone the repository and open it in **Android Studio** (latest stable version).
2. Make sure the Firebase `google-services.json` file is in place for the flavor you build (`app/src/qa/`, `app/src/uat/`).
3. Pick a build variant, for example **qaDebug**, and run the app.

```bash
./gradlew assembleQaDebug        # build the QA debug APK
./gradlew testQaDebugUnitTest    # run unit tests
```

### Build variants
| Flavor | Use |
|---|---|
| `qa` | QA / test environment |
| `uat` | User acceptance testing |
| `prod` | Production |

---

## My role

I worked on this project as an **Application Developer**, building the app natively for **Android (Kotlin, Android Studio)** and **iOS (Swift, SwiftUI, Xcode)**:
- built the **recipients and payments** module: add, edit and delete recipients with address and bank-detail validation (IBAN, SWIFT/BIC, sort code);
- built the **multi-step domestic and international transfer** flows: own-account transfer, recipient selection, amount entry and swipe-to-confirm;
- implemented **payment editing, upcoming payments and pending approvals** for scheduled and recurring transfers;
- built the **loan details, documents and account overview** screens using REST API data;
- implemented **secure in-app chat** with bank support and **Firebase push notifications**;
- followed **MVVM** with Hilt, Coroutines, LiveData/ViewModel and Paging 3 on Android, and built the matching **SwiftUI** views and navigation on iOS;
- distributed iOS beta builds through **TestFlight**, and worked in an Agile team using Git, Jira and code reviews.
