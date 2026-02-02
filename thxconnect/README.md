# THXConnect - Web3 Wallet & DAO Platform

[Website](https://thxnet.org/) | [App Store](https://apps.apple.com/us/app/thxconnect/id6471815589) | [Play Store](https://play.google.com/store/apps/details?id=org.thxnet.thxconnect&pcampaignid=web_share)

**Featured:** [Yahoo Finance](https://finance.yahoo.com/news/thxnet-announces-silver-sponsorship-teamz-060400912.html) | [HR Asia](https://hr.asia/media-outreach/thxnet-launches-thxconnect-a-revolutionary-mobile-app-for-blockchain-developers/) | [THXNet Medium](https://thxnet.medium.com/thxconnect-a-one-stop-app-platform-for-developers-to-explore-build-connect-and-discover-thxnet-524ca70836d4)

![THXConnect App](images/thxconnect.png)

## What it is

A comprehensive Flutter mobile application serving as a multi-currency crypto wallet with integrated DAO governance, NFT management, token staking, and social platform connections. Built for the Japanese market with My Number integration, the app provides blockchain explorer functionality, fungible token (FT) transactions, and secure biometric authentication.

## My role

I designed and developed the complete mobile application as the lead Flutter developer for THXLAB. Responsible for Clean Architecture implementation, gRPC/GraphQL integration, state management with BLoC pattern, dependency injection setup (GetIt/Injectable), secure local storage with Hive and Flutter Secure Storage, WebView bridge for web3 interactions, and multi-wallet support for THX, AETH, SAND, and other tokens.

## Tech stack

**Framework & Language:**
- Flutter 3.0.6+
- Dart SDK >=3.0.0 <4.0.0
- iOS and Android native platform integration

**Architecture & State Management:**
- Clean Architecture with feature-based modules
- BLoC pattern for state management (RxDart)
- GetIt + Injectable for dependency injection
- Repository pattern for data access

**Backend Communication:**
- **GraphQL**: Ferry 0.16.0+1 (GraphQL client with code generation)
- **gRPC**: grpc 4.0.0 (high-performance RPC)
- **Protocol Buffers**: protobuf 3.1.0 (schema definitions)
- **Ferry Generator**: Type-safe GraphQL queries with auto-generated models

**Local Storage:**
- Hive 2.2.3 (NoSQL database)
- Flutter Secure Storage 9.0.0 (encrypted credential storage)
- Shared Preferences

**Authentication & Security:**
- OAuth2 with yup_oauth2
- JWT Decoder
- Magic Links (passwordless authentication)
- Biometric authentication (local_auth)
- Google Sign-In, Apple Sign-In, Facebook Login

**Blockchain & Web3:**
- ss58 (Substrate address encoding for blockchain addresses)
- WebView Flutter 4.11.0 (web3 dApp integration)

**Firebase Suite:**
- Firebase Messaging 15.0.4 (push notifications)
- Firebase Analytics
- Firebase Crashlytics

**UI & Components:**
- Flex Color Scheme 8.2.0 (theming)
- Percent Indicator 4.2.3
- Shimmer 3.0.0 (loading states)
- Photo View 0.15.0 (image viewer)
- FL Chart 0.68.0 (charts/graphs)
- QR Flutter 4.1.0 (QR code generation)
- Mobile Scanner 6.0.0 (QR scanning)
- Flutter Animate 4.5.0 (animations)
- Pull To Refresh
- Cached Network Image

**Notifications:**
- Local Notifications 17.1.2
- Teledart 0.6.1 (Telegram bot API integration)

**Image & Media:**
- Image Picker (camera/gallery)
- Image Fade
- PDF viewer

**Code Generation:**
- Build Runner
- Ferry Generator (GraphQL)
- GQL Build
- Freezed + JSON Serializable (data models)

**Internationalization:**
- Flutter Intl with auto-generated localization classes
- Multi-language support

**Social Integration:**
- Twitter/X authentication
- Discord authentication
- Telegram integration

**Japanese Market Features:**
- My Number integration (Japanese national ID system)
- Local permissions handling (Permission Handler 11.3.1)

## System architecture

### Clean Architecture with Feature-Based Structure

The project follows Clean Architecture principles with strict layer separation:

```
lib/
├── app/                   # Application-level configuration
│   └── bloc/             # App-level BLoC (global state)
├── core/                  # Core infrastructure
│   ├── di/               # Dependency injection (GetIt + Injectable)
│   │   ├── config.dart            # DI configuration
│   │   ├── register_grpc_clients.dart
│   │   └── config.config.dart     # Generated DI
│   ├── theme.dart        # App theming
│   ├── failures.dart     # Error handling
│   ├── biometrics.dart   # Biometric auth
│   └── hive_box.dart     # Local storage config
├── features/              # Feature modules (Clean Architecture)
│   ├── wallet_THX/       # THX token wallet
│   ├── wallet_sand/      # SAND token wallet
│   ├── nft/              # NFT management
│   ├── dao/              # DAO governance
│   ├── staking/          # Token staking
│   ├── dashboard/        # Blockchain explorer
│   ├── user/             # User management
│   ├── magic_link/       # Magic link auth
│   ├── twitter/          # Twitter/X auth
│   ├── discord/          # Discord auth
│   ├── telegram/         # Telegram integration
│   ├── my_number_page/   # My Number (Japan)
│   ├── send_ft/          # Fungible token transfers
│   └── ... (25+ features total)
├── graphql/              # GraphQL setup
│   ├── schema.graphql
│   ├── graphql_client.dart
│   ├── get_blocks.graphql
│   ├── get_events.graphql
│   └── __generated__/    # Ferry generated code
├── pb/                   # Protocol Buffers (gRPC)
│   ├── auth-service/
│   ├── blockchain-info-service/
│   ├── ft-service/
│   ├── nft-service/
│   ├── dao-service/
│   └── notification-service/
└── shared/               # Shared utilities
    ├── validators.dart
    └── widgets/
```

### Feature Module Architecture

Each feature follows a consistent Clean Architecture structure:

```
feature_name/
├── bloc/                    # Presentation layer
│   ├── feature_bloc.dart           # Abstract interface
│   └── feature_bloc_impl.dart      # Implementation
├── pages/                   # UI screens
├── widgets/                 # Feature-specific UI components
├── models/                  # Data models (Freezed)
├── usecases/                # Business logic layer
│   ├── get_data_usecase.dart
│   └── update_data_usecase.dart
└── data/                    # Data layer
    └── data_manager.dart    # Repository pattern
```

### Data Flow Architecture

```
UI (Pages/Widgets)
    ↓
BLoC (State Management)
    ↓
Usecases (Business Logic)
    ↓
Data Managers (Repository Pattern)
    ↓
    ├→ GraphQL Client (Ferry) → Blockchain Explorer API
    ├→ gRPC Clients → Microservices (Auth, NFT, DAO, FT, Notification)
    └→ Hive/Secure Storage → Local Cache
```

### Dependency Injection Pattern

```dart
// lib/core/di/config.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';

@InjectableInit(
  throwOnMissingDependencies: true,
)
Future<GetIt> configureInjector(GetIt getIt) {
  return getIt.init();
}
```

All dependencies registered with `@injectable` annotations:
- `@singleton` - Single instance for app lifetime
- `@lazySingleton` - Created on first access
- `@injectable` - New instance per injection

### gRPC Service Integration

**Microservices Architecture:**
- **auth-service**: User authentication, OAuth flows
- **blockchain-info-service**: Blockchain data queries
- **ft-service**: Fungible token operations (transfer, balance)
- **nft-service**: NFT minting, transfers, metadata
- **dao-service**: DAO proposals, voting, governance
- **notification-service**: Push notifications, alerts
- **id-wallet-service**: Digital identity management
- **general-service**: Miscellaneous utilities

**Protocol Buffer Files:**
- `.proto` definitions compiled to `.pb.dart`, `.pbenum.dart`, `.pbjson.dart`, `.pbgrpc.dart`
- Type-safe request/response models
- Binary serialization for efficient network transfer

### GraphQL Integration

**Blockchain Explorer Queries:**
```graphql
# get_blocks.graphql
query GetBlocks($limit: Int!, $offset: Int!) {
  blocks(limit: $limit, offset: $offset, order_by: {number: desc}) {
    number
    hash
    timestamp
    extrinsics_count
  }
}

# get_events.graphql
query GetEvents($blockNumber: Int!) {
  events(where: {block_number: {_eq: $blockNumber}}) {
    event_id
    module
    event
    data
  }
}
```

**Ferry Code Generation:**
- Type-safe query builders
- Automatic cache normalization
- Subscription support for real-time updates
- Generated models in `graphql/__generated__/`

## Key technical decisions

**Clean Architecture with feature modules:**
Enforced strict layer separation preventing domain logic from depending on UI or data sources. Each feature is self-contained with its own BLoC, usecases, and data managers. Enables independent feature development and testing isolation.

**gRPC over REST for backend communication:**
Binary Protocol Buffers provide 5-7x smaller payload sizes than JSON. Typed service definitions prevent runtime serialization errors. HTTP/2 multiplexing reduces latency for multiple concurrent requests. Critical for mobile apps on limited bandwidth.

**GraphQL (Ferry) for blockchain data:**
Blockchain explorer requires flexible queries (filter by address, date range, transaction type). GraphQL enables client-specified query shapes reducing over-fetching. Ferry's normalized cache prevents redundant network requests.

**Hive + Flutter Secure Storage dual-layer persistence:**
Hive for non-sensitive data (UI state, cached NFT metadata, user preferences). Flutter Secure Storage for credentials (JWT tokens, private keys, OAuth tokens). Encrypted storage uses iOS Keychain and Android Keystore for hardware-backed security.

**BLoC pattern over Provider/Riverpod:**
RxDart streams enable complex state transformations (debounce, combineLatest, switchMap). Separation of BLoC interface from implementation enables testability with mock BLoCs. Explicit state transitions through events/states pattern.

**Magic Links over traditional password auth:**
Passwordless authentication reduces user friction (no password forgotten flows). Secure one-time tokens sent via email with expiration. Backend verifies token signature and issues JWT. Improved security as passwords can't be phished.

**Biometric authentication for transactions:**
High-value transactions (token transfers, NFT sales) require biometric confirmation. Uses iOS Face ID, Touch ID, or Android fingerprint. Prevents unauthorized transactions even if device is unlocked.

**WebView bridge for web3 dApps:**
Third-party dApps load in WebView with JavaScript bridge injecting wallet provider. Enables MetaMask-style interactions without custom integration per dApp. Security: domain whitelist + user confirmation for transactions.

## Notable challenges solved

**Multi-wallet architecture:**
Challenge: Supporting multiple token types (THX, AETH, SAND, LMT, MBT, Z28) with different blockchain protocols. Solution: Abstract wallet interface with protocol-specific implementations. Dependency injection provides correct wallet instance based on token type. Shared UI components with wallet-agnostic APIs.

**gRPC stream handling:**
Challenge: Maintaining persistent gRPC streams for real-time notifications without draining battery. Solution: Connection pooling with automatic reconnection on network changes. Exponential backoff for failed connections. Stream disposal tied to Flutter widget lifecycle.

**GraphQL cache normalization:**
Challenge: Blockchain data queries overlap (dashboard shows same blocks as detail view). Solution: Ferry's normalized cache uses GraphQL type + ID as cache key. Automatic cache updates when query refetches. Manual cache updates for optimistic UI.

**Secure credential storage:**
Challenge: Storing wallet private keys and OAuth tokens securely. Solution: Flutter Secure Storage with platform-specific encryption (iOS Keychain, Android KeyStore). Keys never leave secure storage - signing operations happen in-place. Biometric protection for key access.

**Offline-first NFT display:**
Challenge: NFT metadata on IPFS requires network requests, poor UX on slow connections. Solution: Two-tier caching - Hive stores metadata with timestamps. Display cached version immediately, fetch update in background. Shimmer loading state for missing images.

**Japanese My Number integration:**
Challenge: My Number system requires specific validation rules and security measures. Solution: Custom input formatters for My Number format (12 digits with hyphens). Checksum validation algorithm. Encrypted storage with biometric access. Compliance with Japanese privacy laws (APPI).

**Transaction signing with user confirmation:**
Challenge: Preventing accidental token transfers or NFT sales. Solution: Multi-step confirmation flow: (1) review transaction details, (2) biometric authentication, (3) backend verification. Transaction preview shows fiat-equivalent values. Hardware-backed key signing.

**Social platform OAuth flows:**
Challenge: Twitter, Discord, Telegram each have different OAuth implementations. Solution: Abstract OAuth provider interface. Platform-specific implementations handle redirect URIs, scope requests, token exchange. Unified user experience across platforms.

**DAO proposal voting:**
Challenge: On-chain voting requires gas fees for every vote. Solution: Off-chain signature-based voting (similar to Snapshot). Backend aggregates signatures and submits batch transaction. Users sign votes with wallet keys without gas cost. Final tally verified on-chain.

## Code highlights

### [lib/features/dashboard/usecase/get_blocks_usecase.dart](lib/features/dashboard/usecase/get_blocks_usecase.dart)
Demonstrates GraphQL integration with Ferry client for blockchain explorer functionality. Shows type-safe query execution with error handling.

### [lib/core/di/config.dart](lib/core/di/config.dart) - Dependency Injection

```dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'config.config.dart';

@InjectableInit(
  throwOnMissingDependencies: true,
)
Future<GetIt> configureInjector(GetIt getIt) {
  return getIt.init();
}

// Usage in main.dart
void main() async {
  final getIt = GetIt.instance;
  await configureInjector(getIt);
  runApp(MyApp());
}

// Injectable services
@injectable
class AuthService {
  final AuthGrpcClient _client;
  AuthService(this._client);
}

@lazySingleton
class UserBloc implements UserBlocInterface {
  final GetUserUsecase _getUserUsecase;
  UserBloc(this._getUserUsecase);
}
```

**Why this matters:** Compile-time dependency graph validation with `throwOnMissingDependencies: true`. All injection errors caught at build time. Clean separation between interface and implementation for testability.

### [lib/graphql/](lib/graphql/) - GraphQL Setup

**Schema Definition:**
- `schema.graphql` - GraphQL schema
- `get_blocks.graphql`, `get_events.graphql`, `get_extrinsics.graphql` - Queries

**Generated Code:**
- `__generated__/get_blocks.data.gql.dart` - Type-safe query builders
- `__generated__/serializers.gql.dart` - JSON serialization

**Client Configuration:**
```dart
final graphqlClient = GraphqlClient(
  baseUrl: 'https://api.thxlab.com/graphql',
  authToken: await getAuthToken(),
);
```

### [lib/pb/](lib/pb/) - gRPC Services

**Protocol Buffer Services:**
- `auth-service/` - Authentication gRPC
- `ft-service/` - Fungible token operations
- `nft-service/` - NFT management
- `dao-service/` - DAO governance
- `notification-service/` - Push notifications

**Example gRPC Call:**
```dart
@injectable
class NftService {
  final NftServiceClient _grpcClient;

  Future<NftList> getUserNfts(String address) async {
    final request = GetUserNftsRequest()..address = address;
    final response = await _grpcClient.getUserNfts(request);
    return response.nfts;
  }
}
```

### [lib/features/user/bloc/](lib/features/user/bloc/) - BLoC Pattern

**Abstract Interface:**
```dart
abstract class UserBloc extends BaseBloc {
  Stream<User?> get user;
  Stream<bool> get loading;

  void loadUser();
  void updateProfile(ProfileData data);
}
```

**Implementation:**
```dart
@lazySingleton
class UserBlocImpl implements UserBloc {
  final GetUserUsecase _getUserUsecase;
  final UpdateProfileUsecase _updateProfileUsecase;

  final _userSubject = BehaviorSubject<User?>();
  final _loadingSubject = BehaviorSubject<bool>.seeded(false);

  @override
  Stream<User?> get user => _userSubject.stream;

  @override
  Stream<bool> get loading => _loadingSubject.stream;

  @override
  void loadUser() {
    _loadingSubject.add(true);
    _getUserUsecase.execute().then((result) {
      result.fold(
        (failure) => handleError(failure),
        (user) => _userSubject.add(user),
      );
    }).whenComplete(() => _loadingSubject.add(false));
  }
}
```

**Why this matters:** Interface segregation for testability. RxDart BehaviorSubject provides current value to new listeners. Reactive streams enable UI to react to state changes automatically.

### [lib/features/wallet_THX/](lib/features/wallet_THX/) - Multi-Wallet Implementation

Demonstrates wallet abstraction with THX-specific implementation. Shows balance queries, transaction history, transfer operations with biometric confirmation.

### [lib/features/nft/](lib/features/nft/) - NFT Management

NFT listing, detail view, transfer functionality. IPFS metadata fetching with fallback to cache. QR code scanning for wallet addresses.

### [lib/features/dao/](lib/features/dao/) - DAO Governance

Proposal creation, voting interface, vote tallying. Off-chain signature collection with on-chain verification.

## Deployment & environment

**Build & Release:**
- Flutter build for iOS and Android
- Code signing with Apple Developer certificates
- Android signing with release keystore

**Environment Configuration:**
- Build flavors: development, staging, production
- Environment-specific API endpoints
- Firebase configuration per environment

**Backend Services:**
```
gRPC Endpoints:
- auth-service.thxlab.com:443
- ft-service.thxlab.com:443
- nft-service.thxlab.com:443
- dao-service.thxlab.com:443

GraphQL Endpoint:
- https://explorer.thxlab.com/graphql

Firebase:
- Cloud Messaging (push notifications)
- Analytics (user behavior tracking)
- Crashlytics (error reporting)
```

**Production Monitoring:**
- Firebase Crashlytics for crash reporting
- Custom error logging with user context
- Performance monitoring for GraphQL queries and gRPC calls

**Security Measures:**
- Certificate pinning for gRPC/HTTPS
- Biometric authentication for sensitive operations
- Encrypted local storage (Keychain/KeyStore)
- JWT token expiration and refresh

## Public links

Commercial project for THXLAB under NDA. Demo available upon request for portfolio review. Source code available with NDA agreement.
