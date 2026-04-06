# ULT - Dsquares B2C Loyalty App

## Project Overview
ULT is Dsquares' first B2C (Business-to-Consumer) loyalty points application. Dsquares is a major B2B loyalty solutions provider founded in 2012 in Cairo, Egypt, serving clients like Vodafone, Visa, Mastercard, Shell, Orange, PepsiCo, and major banks across 10+ countries in MENA & Africa.

This repo contains the **Flutter frontend** for the ULT mobile app.

---

## Chat History & Research Notes

### Context
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

## Confirmed Tech Stack

| Layer | Technology |
|---|---|
| Backend | .NET (C#) |
| Database | Microsoft SQL Server |
| Web Frontend | Angular |
| Mobile | Flutter + Dart |
| State Management | BLoC pattern |
| Architecture | Clean Architecture |
| Auth (likely) | OAuth 2.0 / IdentityServer / Duende |
| API Docs (likely) | Swagger / OpenAPI |

---

## Technical Call Preparation

### Architecture Approach
- **Clean Architecture**: Domain layer (entities, use cases), Data layer (repositories, API data sources), Presentation layer (UI + state management)
- **Feature-first modular structure**: Separate features (earn, burn, wallet, profile, offers) into self-contained modules for parallel development and testability
- **Repository pattern**: Abstract API calls behind repository interfaces; swap real/mock implementations for testing
- **DTO mapping**: Use dedicated Data Transfer Objects that mirror the .NET API contracts; map them to domain models inside the data layer
- **BFF (Backend for Frontend)**: A thin middleware layer tailored for mobile needs — ask if one exists
- **Error handling**: .NET APIs typically return ProblemDetails (RFC 7807) responses — build a centralized error parser in Dio interceptor

### Key Flutter Packages

| Category | Packages |
|---|---|
| State Management | flutter_bloc / Riverpod |
| API / HTTP | dio, retrofit (code-gen), connectivity_plus |
| Auth / Security | flutter_secure_storage, flutter_appauth, local_auth (biometrics) |
| Push Notifications | firebase_messaging, flutter_local_notifications |
| QR Codes | qr_flutter (display), mobile_scanner (scan) |
| Local DB / Cache | hive, isar, shared_preferences |
| Navigation | go_router / auto_route |
| DI | get_it + injectable |
| Analytics | firebase_analytics, mixpanel_flutter |

### Best Practices for B2C Loyalty Apps in Flutter
1. **Offline-first design**: Cache points balances, transaction history, and tier status locally so users see data instantly even with poor connectivity
2. **Security**: Use flutter_secure_storage for tokens; implement certificate pinning; never store sensitive PII in plain local storage
3. **Real-time feedback**: Animate points changes, tier progress bars, and reward unlocks to drive engagement
4. **Deep linking**: Support deep links for promotional offers, referral codes, and push notification payloads that land on specific screens
5. **White-labeling readiness**: Use theming and configuration files so the same codebase can serve multiple brands if needed

### Smart Questions to Ask on Technical Call
1. "Is there a Swagger/OpenAPI spec I can reference, or is the API contract still evolving?"
2. "How is authentication handled — JWT tokens with refresh? Biometric unlock?"
3. "Are points calculations server-authoritative, or do we do optimistic updates on the client?"
4. "Is there a design system/component library established, or are we building from scratch?"
5. "What environments exist — dev, staging, prod? Any CI/CD pipeline set up?"
6. "How are push notifications structured — are payload schemas defined yet?"
7. "What's the testing strategy? Unit tests, widget tests, integration tests?"
8. "How does the middleware layer between ULT and the main Dsquares backend work?"

---

## Loyalty Program Domain Knowledge

| Term | Meaning |
|---|---|
| **Earn / Accrual** | How users collect points (e.g., purchases, actions) |
| **Burn / Redemption** | How users spend points for rewards |
| **Earn ratio** | Points per currency spent (e.g., 1 point per 1 EGP) |
| **Burn ratio** | Points needed per currency value (e.g., 100 points = 1 EGP discount) |
| **Tiers** | Status levels (Silver/Gold/Platinum) with escalating benefits |
| **Breakage** | Points that expire or go unredeemed — expected revenue for the business |
| **Points liability** | Outstanding unredeemed points — actual financial liability on company books |
| **Qualification period** | Time window to earn enough for tier maintenance/upgrade |
| **Coalition program** | Multiple brands sharing one loyalty currency |
| **Gamification** | Challenges, streaks, badges layered on the core earn/burn loop |
| **Tokenization** | Representing loyalty value as digital tokens |

---

## Flutter + .NET Integration Notes

- **.NET APIs** typically use OAuth 2.0 with JWT tokens. Flutter handles this via `flutter_appauth` or Dio interceptors for automatic token refresh.
- **ProblemDetails (RFC 7807)** is the standard .NET error response format — build a centralized parser for this.
- **Swagger/OpenAPI** specs from the backend can be used to auto-generate API client code with `retrofit` or `openapi_generator`.
- Expect the middleware/BFF layer to aggregate data from the main Dsquares platform and shape it for mobile consumption.
