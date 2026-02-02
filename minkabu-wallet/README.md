# Minkabu Wallet

[Website](https://minkabu.co.jp/) | [App Store](https://apps.apple.com/us/app/web3-wallet/id6446136975)

**Featured:** [Bloomberg](https://www.bloomberg.com/profile/company/2027325D:JP) | [THXNet Article](https://thxnet.medium.com/thxnet-use-case-minkabu-web3-wallet-introduce-innovative-employee-id-nft-solution-and-blockchain-fd9920058226)

## Overview

Minkabu Wallet is a Web3-focused mobile wallet and credential management application that enables users to securely manage digital credentials (NFTs), verify authenticity through QR codes, and interact with Web3 services. The app emphasizes privacy, security, and seamless credential management across multiple issuers and services.

## Tech Stack

- **Framework:** Flutter (Dart)
- **State Management:** Custom BLoC pattern with RxDart
- **API Client:** Chopper (REST)
- **Dependency Injection:** GetIt + Injectable
- **Local Storage:** Hive, Flutter Secure Storage
- **Authentication:** Local Auth (biometric), Magic Link (passwordless)
- **Backend Services:** Firebase (Core, Messaging)
- **Code Generation:** Freezed, JSON Serializable, Build Runner
- **Functional Programming:** Dartz (Either type for error handling)

## Platform

- iOS & Android

## Features

- **Web3 Wallet Management:** NFT collection viewing, digital credential storage, wallet visibility control
- **Passwordless Authentication:** Magic link email auth, phone verification with OTP, biometric unlock
- **NFT Operations:** Detail view with metadata, QR code generation/scanning for verification, favorite and visibility toggles
- **User Management:** Profile management, picture upload, account settings, language selection (EN/JA)
- **Push Notifications:** Firebase messaging, notification history, background handling
- **Security:** Biometric authentication, encrypted token storage, location-based verification

## Architecture

The application follows **Clean Architecture** with **BLoC Pattern**:

```
lib/
├── features/                          # Feature modules
│   ├── app/                          # App-level BLoC and routing
│   ├── auth/                         # Authentication logic
│   ├── wallet/                       # NFT wallet management
│   │   ├── models/                   # Freezed data models
│   │   ├── data/                     # Data managers
│   │   └── usecases/                 # Business logic
│   ├── nft_detail/                   # NFT detail view
│   │   ├── bloc/                     # State management
│   │   ├── model/                    # Response models
│   │   └── usecase/                  # QR, favorites, visibility
│   ├── user/                         # User profile management
│   ├── verify_magic_link/            # Email link verification
│   ├── verify_phone/                 # Phone OTP verification
│   ├── notifications/                # Push notification handling
│   ├── issuer/                       # NFT issuer information
│   ├── onboarding/                   # User onboarding flow
│   └── settings/                     # Theme and preferences
├── source_remote/                    # API clients
│   └── api/                          # Chopper services
│       ├── auth_service.dart
│       ├── account_service.dart
│       ├── nft_service.dart
│       └── verify_service.dart
├── di/                               # Dependency injection
└── ui/                               # Reusable components
```

**Key Technical Decisions:**

- **Feature-first Modularization:** Self-contained features with models, data, usecases, and BLoCs
- **Reactive Streams:** BehaviorSubject for state, PublishSubject for events
- **UseCase Pattern:** Business logic encapsulation with base UseCase classes
- **Immutable Models:** Freezed for type-safe, immutable data classes
- **QR Verification Flow:** Two-step verification with location-based gating
- **Hive Persistence:** User preferences, auth state, notification settings
