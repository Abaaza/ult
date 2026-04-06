# ULT - Dsquares B2C Loyalty App

## Project Overview

ULT is Dsquares' first B2C (Business-to-Consumer) loyalty points application. Dsquares is a major B2B loyalty solutions provider founded in 2012 in Cairo, Egypt, serving clients like Vodafone, Visa, Mastercard, Shell, Orange, PepsiCo, and major banks across 10+ countries in MENA & Africa.

**What does B2C vs B2B mean here?**
- **B2B (Business-to-Business)**: Dsquares builds loyalty systems *for* other companies (e.g., Vodafone's rewards program). The client is a business.
- **B2C (Business-to-Consumer)**: ULT is Dsquares' own app that regular people download and use directly. The user is the end consumer.

This repo contains the **Flutter frontend** for the ULT mobile app. The Flutter app is what users see and interact with. The backend (built by a separate team in .NET) handles all the business logic, databases, and connections to Dsquares' existing platform. The app communicates with the backend through APIs.

---

## Context

- Dsquares wants to hire a Flutter frontend developer for ULT
- The project is ~20% complete (as of April 2026)
- Backend team uses .NET with middleware connecting ULT to the main Dsquares platform
- There is a dedicated UX/UI designer on the team

---

## Dsquares Company Research

- **Founded**: 2012, Cairo, Egypt
- **Founders**: Marwan Kenawy, Ayman Essawy, Momtaz Moussa
- **What they do**: End-to-end B2B loyalty program management — points systems, tiers, earn/burn mechanics, e-vouchers, digital rewards, gamification, merchant management, AI analytics
- **Clients**: Vodafone, Orange, Vodacom, Visa, Mastercard, ExxonMobil, PepsiCo, P&G, Credit Agricole, Shell, Uber, Souq, major Egyptian banks
- **Reach**: 10+ countries (Egypt, Jordan, Saudi, UAE, Morocco, Kenya, Tanzania, Romania), 900+ brands, 25,000+ stores
- **Funding**: Backed by Algebra Ventures and Ezdehar
- **Recent moves**: Acquired Prepit (B2B SaaS loyalty platform) in early 2025 — consolidating market position
- **Partnership**: Infobip for conversational loyalty experiences

---

## Confirmed Tech Stack (Detailed)

| Layer | Technology | What It Does |
|---|---|---|
| Backend | .NET (C#) | Microsoft's framework for building server-side applications. The backend team writes the APIs (endpoints) your app calls to get/send data. C# is the programming language. |
| Database | Microsoft SQL Server | Where all data lives — user accounts, points balances, transaction history, rewards catalog. The backend reads/writes to this; the Flutter app never talks to it directly. |
| Web Frontend | Angular | A JavaScript framework by Google used for the web version of the app or admin dashboard. Separate from the Flutter mobile work. |
| Mobile | Flutter + Dart | **Flutter** is Google's framework for building cross-platform mobile apps (iOS + Android) from a single codebase. **Dart** is the programming language Flutter uses, similar to JavaScript/TypeScript in feel. |
| State Management | BLoC pattern | See detailed explanation below. |
| Architecture | Clean Architecture | See detailed explanation below. |
| Auth (likely) | OAuth 2.0 / IdentityServer / Duende | See detailed explanation below. |
| API Docs (likely) | Swagger / OpenAPI | See detailed explanation below. |

### BLoC Pattern (Business Logic Component)

State = the data your app is currently showing (e.g., "user has 500 points", "this screen is loading").

How BLoC works:
1. The UI sends **events** (e.g., "user tapped refresh")
2. The BLoC processes the event (calls API, does calculations)
3. The BLoC emits a new **state** (e.g., "here's the updated points balance")
4. The UI rebuilds to show the new state

This keeps business logic completely separated from UI code, making the app testable and maintainable.

### Clean Architecture

A way of organizing code into **layers** that don't depend on each other:

```
Presentation Layer (UI + BLoCs)
       ↓ depends on
Domain Layer (Business rules, entities, use cases)
       ↓ depends on
Data Layer (API calls, local storage, repositories)
```

Key rule: **inner layers never know about outer layers**. Your business rules don't care whether data comes from an API or a local database. This makes the code testable, swappable, and maintainable.

### OAuth 2.0 / IdentityServer / Duende

- **OAuth 2.0** is the industry standard protocol for authentication. When you "Sign in with Google" anywhere, that's OAuth.
- **IdentityServer / Duende** is a .NET library that implements OAuth. The backend team uses this to issue **JWT tokens** (JSON Web Tokens) — encrypted strings that prove "this user is logged in."
- The Flutter app stores that token securely and sends it with every API request so the server knows who you are.

### Swagger / OpenAPI

- **Swagger** is a tool that auto-generates documentation for APIs. The backend team writes their .NET endpoints, and Swagger creates a webpage where you can see every endpoint, what parameters it expects, and what it returns.
- **OpenAPI** is the specification format behind Swagger. You can use this spec file to **auto-generate Dart code** that calls those APIs, so you don't write HTTP calls manually.

---

## Technical Call Preparation

### Architecture Approach (Detailed)

#### Clean Architecture Layers
- **Presentation Layer**: UI widgets + BLoC state management. What the user sees and interacts with.
- **Domain Layer**: Pure business rules, entities (data models), and use cases (e.g., "redeem points for reward"). No dependencies on Flutter or any framework.
- **Data Layer**: Repositories, API data sources, local storage. Handles where data comes from.

#### Feature-First Modular Structure

Instead of organizing by file type (bad):
```
/models/
/screens/
/blocs/
/repositories/
```

Organize by **feature** (good):
```
/features/
  /wallet/
    /data/
    /domain/
    /presentation/
  /offers/
    /data/
    /domain/
    /presentation/
  /profile/
    ...
```

Each feature is self-contained. One developer can work on "wallet" while another works on "offers" without conflicts.

#### Repository Pattern

An abstraction layer between your app and data sources:

```
BLoC → calls → Repository (interface) → implements → ApiRepository (real API)
                                       → implements → MockRepository (fake data for testing)
```

The BLoC just says "get me the user's points." It doesn't know or care if that data comes from a real server or a test mock.

#### DTO Mapping

- **DTO** = Data Transfer Object. A Dart class that exactly mirrors what the API sends back as JSON.
- **Domain Model** = A cleaner Dart class that your app actually uses.

Example:
```
API returns:  { "pts_bal": 500, "usr_nm": "Ahmed", "acct_sts": 1 }
DTO:          PtsBalanceDto(ptsBal: 500, usrNm: "Ahmed", acctSts: 1)
Domain Model: UserBalance(points: 500, name: "Ahmed", isActive: true)
```

The DTO handles the ugly API naming; the domain model is clean for your app to use.

#### BFF (Backend for Frontend)

A **thin middleware layer** that sits between the Flutter app and the main backend. Instead of the mobile app calling 5 different microservices to build one screen, the BFF aggregates that into one call optimized for mobile. Need to ask if Dsquares has one — if not, the main .NET API serves this role.

#### Error Handling: ProblemDetails (RFC 7807)

.NET APIs return errors in a standard format called ProblemDetails:
```json
{
  "type": "https://example.com/errors/insufficient-points",
  "title": "Insufficient Points",
  "status": 400,
  "detail": "You need 200 more points for this reward."
}
```

Build a **Dio interceptor** (middleware that intercepts every HTTP response) that catches these errors and converts them into user-friendly messages automatically, instead of handling errors in every single API call.

---

### Key Flutter Packages (Detailed)

#### State Management
| Package | Purpose |
|---|---|
| **flutter_bloc** | Implements the BLoC pattern — separates business logic from UI using events and states |
| **Riverpod** | Alternative state management solution (listed as an option if team prefers it over BLoC) |

#### API / HTTP
| Package | Purpose |
|---|---|
| **dio** | The HTTP client for making API calls (like Axios in JavaScript). Supports interceptors, retry logic, file uploads, timeout handling |
| **retrofit** | Code-generation tool that turns your API interface definitions into actual HTTP calls automatically. Define the endpoint, it writes the boilerplate |
| **connectivity_plus** | Detects whether the phone has internet connection. Useful for showing "offline mode" UI and triggering sync when back online |

#### Auth / Security
| Package | Purpose |
|---|---|
| **flutter_secure_storage** | Stores tokens in the phone's secure keychain (iOS Keychain / Android Keystore). Unlike SharedPreferences, this is encrypted |
| **flutter_appauth** | Handles the OAuth login flow — opens a browser, user logs in, app receives a token back |
| **local_auth** | Fingerprint / Face ID unlock. Lets users skip entering passwords on trusted devices |

#### Push Notifications
| Package | Purpose |
|---|---|
| **firebase_messaging** | Receives push notifications from Firebase Cloud Messaging (FCM). E.g., "You earned 50 points!" or "New offer available!" |
| **flutter_local_notifications** | Shows notification banners on the device. Works together with firebase_messaging to display the notification visually |

#### QR Codes
| Package | Purpose |
|---|---|
| **qr_flutter** | Generates QR code images. The user shows their QR code at a store to earn/redeem points |
| **mobile_scanner** | Uses the camera to scan QR codes. The user scans a merchant's code to initiate a transaction |

#### Local DB / Cache
| Package | Purpose |
|---|---|
| **hive** | A fast, lightweight local database. Good for caching data like points balance so the app opens instantly |
| **isar** | A more powerful local database (alternative to hive) with advanced querying capabilities |
| **shared_preferences** | Simple key-value storage for settings (e.g., "dark mode on", "language = Arabic") |

#### Navigation
| Package | Purpose |
|---|---|
| **go_router / auto_route** | Handle screen navigation. Support deep links (opening a specific screen from a push notification URL), nested navigation, route guards (redirect to login if not authenticated) |

#### Dependency Injection (DI)
| Package | Purpose |
|---|---|
| **get_it** | A service locator. Register your classes at app startup, then retrieve them anywhere without passing them through constructors manually |
| **injectable** | Code generation that auto-registers your classes with get_it using annotations, eliminating boilerplate registration code |

#### Analytics
| Package | Purpose |
|---|---|
| **firebase_analytics** | Tracks user behavior (screen views, button taps, conversion events) and sends it to Firebase/Google Analytics |
| **mixpanel_flutter** | Alternative/additional analytics platform. Often used for more detailed product analytics (funnels, retention, A/B testing) |

---

### Best Practices for B2C Loyalty Apps in Flutter

#### 1. Offline-First Design
Users in MENA/Africa may have unreliable internet. The app should:
- Cache the user's points, history, and tier locally
- Show cached data immediately when the app opens
- Sync with the server in the background
- Show a subtle indicator when data might be stale

#### 2. Security
- Tokens go in **secure storage**, not plain text
- **Certificate pinning** — the app only trusts your specific server's SSL certificate, preventing man-in-the-middle attacks
- Never store sensitive data (national IDs, payment info) in local storage

#### 3. Real-Time Feedback
Loyalty apps need to feel **rewarding**. When a user earns points:
- Animate the points counter going up
- Show a tier progress bar filling
- Display confetti or a badge when they level up
This drives engagement and makes users want to come back.

#### 4. Deep Linking
URLs like `ult://offers/summer-sale` should open directly to that offer screen. This enables:
- Push notifications that land on the right screen
- Referral links shared between friends
- Marketing campaigns that link into specific app sections

#### 5. White-Labeling Readiness
Dsquares serves many brands. If they want to reuse this app for another client:
- Colors, fonts, logos come from a config file
- Feature flags enable/disable modules per brand
- One codebase, many branded versions

---

### Smart Questions to Ask on Technical Call

| # | Question | Why It Matters |
|---|---|---|
| 1 | "Is there a Swagger/OpenAPI spec I can reference, or is the API contract still evolving?" | Determines if you can code-gen your API layer or need to wait for the backend team |
| 2 | "How is authentication handled — JWT tokens with refresh? Biometric unlock?" | Defines how exactly login works and what security features to build |
| 3 | "Are points calculations server-authoritative, or do we do optimistic updates on the client?" | Server-authoritative = always wait for server. Optimistic = show estimated points instantly, confirm later |
| 4 | "Is there a design system/component library established, or are we building from scratch?" | Determines if there's a Figma with reusable components or if you're building UI from zero |
| 5 | "What environments exist — dev, staging, prod? Any CI/CD pipeline set up?" | Tells you if there are separate servers for dev/testing/production and automated build/deploy pipelines |
| 6 | "How are push notifications structured — are payload schemas defined yet?" | Defines what data comes with a notification so you know which screen to open |
| 7 | "What's the testing strategy? Unit tests, widget tests, integration tests?" | Sets expectations for how much testing is required |
| 8 | "How does the middleware layer between ULT and the main Dsquares backend work?" | Understanding the BFF architecture affects how you structure API calls |

---

## Loyalty Program Domain Knowledge (Detailed)

| Term | Meaning | Plain English Example |
|---|---|---|
| **Earn / Accrual** | How users collect points (e.g., purchases, actions) | Buy coffee for 50 EGP → get 50 points |
| **Burn / Redemption** | How users spend points for rewards | Spend 500 points → get a free drink |
| **Earn ratio** | Points per currency spent (e.g., 1 point per 1 EGP) | Spending 100 EGP at 1:1 ratio = 100 points |
| **Burn ratio** | Points needed per currency value (e.g., 100 points = 1 EGP discount) | 1000 points at 100:1 ratio = 10 EGP discount |
| **Tiers** | Status levels (Silver/Gold/Platinum) with escalating benefits | Gold members get 2x points per purchase + free delivery |
| **Breakage** | Points that expire or go unredeemed — expected revenue for the business | User had 500 points, they expired. That's profit for the company (gave nothing away) |
| **Points liability** | Outstanding unredeemed points — actual financial liability on company books | 1 million unredeemed points across all users = debt the company might have to pay out |
| **Qualification period** | Time window to earn enough for tier maintenance/upgrade | "Spend 5000 EGP in 6 months to keep Gold status" — that 6-month window |
| **Coalition program** | Multiple brands sharing one loyalty currency | Earn points at Shell, spend them at Carrefour. Dsquares specializes in this |
| **Gamification** | Challenges, streaks, badges layered on the core earn/burn loop | "Buy 3 coffees this week" challenge, daily login streaks, leaderboards |
| **Tokenization** | Representing loyalty value as digital tokens | Loyalty points as blockchain tokens that users can potentially transfer or trade |

---

## Flutter + .NET Integration Notes (Detailed)

### Authentication Flow
1. User opens app → app checks secure storage for existing JWT token
2. No token or expired → redirect to login screen
3. User logs in → backend returns JWT access token + refresh token
4. App stores both tokens in `flutter_secure_storage` (encrypted)
5. Every API call attaches the access token via a Dio interceptor
6. When the access token expires, the Dio interceptor automatically uses the refresh token to get a new one (the user never notices)

### Error Handling
- **.NET APIs** return errors in **ProblemDetails (RFC 7807)** format — a standardized JSON error structure
- Build one centralized Dio interceptor that catches all ProblemDetails responses and converts them into user-friendly error messages
- This avoids handling errors in every single API call individually

### API Code Generation
- The backend team generates a **Swagger/OpenAPI spec** file describing all their endpoints
- Feed this spec into `retrofit` or `openapi_generator` to auto-create Dart API client classes
- This eliminates writing HTTP calls manually and keeps the app in sync with backend changes

### Middleware / BFF Layer
- Expect a middleware layer that sits between ULT's Flutter app and the main Dsquares platform
- This layer aggregates data from multiple backend services and shapes it for mobile consumption
- Instead of calling 5 endpoints to build one screen, the middleware provides one optimized endpoint

---

## Questions to Ask on the Technical Call

### About the BFF/Middleware (Most Critical)

| # | Question | Why It Matters |
|---|---|---|
| 1 | "Who owns the middleware layer — your backend team or a separate team?" | You need to know who to go to when an endpoint is missing or broken |
| 2 | "What endpoints are ready right now, and what's still being built?" | Determines what you can start coding against vs. what needs mocking |
| 3 | "Is there a Swagger/OpenAPI spec, or how do I know what the API returns?" | If there's no documentation, you'll waste time guessing |
| 4 | "Does the middleware just pass through data from the B2B platform, or does it reshape it for mobile?" | Tells you if the responses will be clean or if you'll need heavy DTO mapping |
| 5 | "How do I report a bug or request a new endpoint? Jira? Slack? Direct message?" | Communication workflow matters more than people think |

### About the Existing 20%

| # | Question | Why It Matters |
|---|---|---|
| 6 | "Can I see the current codebase? What's been built so far?" | You need to know if you're building on top of existing code or starting fresh |
| 7 | "Who wrote the existing code, and will they be on the team?" | If the original dev left, you might inherit messy code with no context |
| 8 | "What architecture decisions have already been locked in?" | BLoC might be confirmed, but maybe navigation or DI hasn't been decided yet |

### About the Design

| # | Question | Why It Matters |
|---|---|---|
| 9 | "Do I have access to Figma/design files? Is the full app designed or just some screens?" | You can't build UI without designs |
| 10 | "Is there a design system (colors, fonts, spacing, component library) or am I extracting that from mockups?" | Huge time difference between having a system vs. guessing from screenshots |
| 11 | "How does the handoff work between the designer and me? Do they update designs mid-sprint?" | Prevents scope creep and rework |

### About Auth & Security

| # | Question | Why It Matters |
|---|---|---|
| 12 | "Walk me through the exact login flow — what happens step by step?" | OAuth has many variations. You need the specific one they use |
| 13 | "Is there social login (Google, Apple, Facebook) or just email/password?" | Completely different implementation paths |
| 14 | "How does token refresh work? What's the token expiry time?" | If it's 5 minutes vs. 24 hours, your interceptor logic changes significantly |
| 15 | "Is there biometric login (fingerprint/Face ID) on the roadmap?" | Affects auth architecture decisions early on |

### About Team & Workflow

| # | Question | Why It Matters |
|---|---|---|
| 16 | "What's the sprint cycle? 1 week? 2 weeks?" | Sets your delivery rhythm |
| 17 | "What git workflow do you use? GitFlow? Trunk-based? PR reviews?" | You need to match their process from day one |
| 18 | "Who reviews my PRs?" | Tells you the feedback loop speed |
| 19 | "What tools does the team use? Jira, Slack, Teams, Notion?" | Logistics you need to know before starting |
| 20 | "Am I the only Flutter developer, or is there a team?" | Solo dev vs. team changes everything about how you structure code |

### About Features & Priority

| # | Question | Why It Matters |
|---|---|---|
| 21 | "What's the MVP feature set? What ships first?" | Don't build offers if wallet is the priority |
| 22 | "Is there a product roadmap I can see?" | Helps you make architecture decisions that won't break later |
| 23 | "Which features are must-haves vs. nice-to-haves for launch?" | Focus your effort on what actually matters for release |

### About Environments & Deployment

| # | Question | Why It Matters |
|---|---|---|
| 24 | "Are there dev/staging/prod environments set up?" | If there's only prod, that's a red flag |
| 25 | "Is there a CI/CD pipeline, or am I building and deploying manually?" | Affects your workflow and release speed |
| 26 | "Are the App Store and Google Play accounts set up? Who manages them?" | Deployment logistics that can block releases |
| 27 | "How do I get access to the dev/staging server?" | You need API URLs, test credentials, VPN access if any |

### About Testing

| # | Question | Why It Matters |
|---|---|---|
| 28 | "What's the expected test coverage? Unit tests? Widget tests? Integration tests?" | Some teams want 80% coverage, some want zero |
| 29 | "Is there a QA person, or am I also responsible for testing?" | Defines your role scope |

### About Timeline & Expectations

| # | Question | Why It Matters |
|---|---|---|
| 30 | "What's the launch target date?" | The most important question for managing expectations |
| 31 | "What does 'done' look like for my first 2 weeks?" | Sets clear short-term expectations so you can hit the ground running |
| 32 | "Is this a contract/freelance role or full-time?" | Affects how much ownership you take over architecture decisions |

### Red Flag Questions (Protect Yourself)

| # | Question | Why It Matters |
|---|---|---|
| 33 | "What happened with the previous developer?" | If they left or were fired, understanding why helps you avoid the same issues |
| 34 | "How often do requirements change mid-sprint?" | Frequent changes = stressful project with constant rework |
| 35 | "Is the backend stable, or is it still heavily changing?" | If APIs keep breaking, your work doubles |
