# XPOT

*[Website]* | *[iOS]* | *[Android]*

## Overview

XPOT is a community rewards and engagement platform that enables users to earn stamps by visiting different communities, track their progress across multiple community groups, and redeem rewards. The app features location-based community discovery, QR code scanning for stamp registration, and comprehensive user profile management with multi-language support.

## Screenshots

![App Screenshot](images/screenshot.png)

## Tech Stack

- **Framework:** Flutter SDK (>=2.18.0)
- **State Management:** Custom BLoC pattern with RxDart
- **API Client:** gRPC with Protocol Buffers
- **Dependency Injection:** GetIt + Injectable
- **Local Storage:** Hive, Shared Preferences
- **Backend Services:** Firebase (Auth, Messaging, Remote Config)
- **Authentication:** Google Sign-In, Apple Sign-In, Firebase Auth
- **Location:** Geolocator for GPS-based community discovery
- **Code Generation:** Protocol Buffers, Injectable, Intl Utils

## Platform

- iOS & Android (+ Desktop support: Linux, macOS, Windows)

## Features

- **Community Discovery:** Location-based nearby community listing with distance sorting
- **Rewards & Stamps:** Collect stamps by visiting communities via QR code scanning
- **Multi-Provider Auth:** Google, Apple, and Firebase authentication
- **Card Collections:** User cards display with grouped communities and score tracking
- **Push Notifications:** Firebase messaging for real-time updates
- **Multi-Language:** 8+ supported locales with runtime language switching
- **Profile Management:** Image editing, preferences, and account settings
- **Onboarding:** Introduction screen flow for new users

## Architecture

The application implements **Clean Architecture** with a custom **Data Manager Pattern**:

```
lib/
├── features/                    # Feature-based modules
│   ├── app/                     # App-level state and navigation
│   ├── auth/                    # Authentication & social login
│   ├── cards/                   # Card management
│   ├── communities/             # Community listing
│   ├── community/               # Community details
│   ├── home/                    # Home screen
│   ├── launcher/                # App initialization
│   ├── qr_code_scanner/         # QR scanning
│   ├── settings/                # User settings
│   ├── user/                    # User data & profile
│   └── user_rewards/            # Reward redemption
├── ui/
│   ├── screens/                 # Screen implementations
│   └── widgets/                 # Reusable components
├── di/                          # Dependency injection setup
├── generated/                   # i18n and localization
└── pb/                          # Generated Protocol Buffers
    ├── gateway-api/
    ├── user-service/
    ├── community-service/
    ├── stamp-service/
    └── tokentransfer-service/
```

**Key Architectural Decisions:**

- **Functional Programming:** Dartz for `Option<T>` and `Either<L, R>` types with pattern matching
- **Reactive Streams:** RxDart with `BehaviorSubject`, `switchMap()`, and `trackActivity()` operators
- **Data Manager Pattern:** Centralized data management with automatic state persistence via Hive
- **gRPC Microservices:** 8 service integrations (Auth, User, Community, Card, Reward, Visit, Notification, Employee)
- **Multi-Environment:** Develop, staging, production flavors with environment-specific configuration
