# Digitax (formerly CashSnapp)

## What it is

A production mobile financial application built for iOS and Android that handles payment processing, digital transactions, and expense tracking. The app provides secure authentication, real-time transaction processing via Stripe, and offline-first data synchronization.

## My role

I architected and developed the entire mobile application from initial requirements through production deployment. Implemented Clean Architecture with dependency injection, integrated multiple payment gateways and authentication providers, and established offline-first data persistence strategies. Managed iOS and Android releases through TestFlight and Firebase App Distribution.

## Tech stack

**Framework & Language:**
- Flutter 3.2.2+ (Dart SDK >=3.2.2 <4.0.0)
- iOS and Android native platform integration

**Architecture & State Management:**
- Clean Architecture with feature-based modules
- GetIt + Injectable for dependency injection
- RxDart for reactive programming
- Dartz for functional error handling (Either monad)

**Data & Persistence:**
- Hive (local NoSQL database)
- Shared Preferences
- Offline-first architecture

**Network & API:**
- Chopper (HTTP client with code generation)
- JWT authentication
- Freezed + JSON Serializable for data models

**Payment & Authentication:**
- Stripe Flutter (v11.5.0)
- Google Sign-In
- Apple Sign-In
- Facebook App Events
- JWT token management

**Firebase Suite:**
- Firebase Core
- Crashlytics (error tracking)
- Analytics
- Cloud Messaging (push notifications)

**UI & Navigation:**
- Go Router (declarative routing)
- Material Design components
- Custom sliding segmented controls
- Pinput (PIN input widgets)
- Modal Bottom Sheet

**Additional Features:**
- Image Picker & File Picker
- Crisp Chat integration
- Branch SDK (deep linking)
- Pull to Refresh

## System architecture

### Clean Architecture Implementation

The project follows Clean Architecture with clear separation of concerns:

```
lib/
├── app/              # Application-level configuration
├── core/             # Shared utilities and DI setup
│   ├── di/          # Dependency injection configuration
│   ├── theme/       # App theming and styles
│   └── utils.dart   # Shared utilities
├── features/         # Feature modules (Clean Architecture)
│   ├── login/
│   │   ├── models/       # Data models (Freezed)
│   │   ├── usecases/     # Business logic
│   │   ├── bloc/         # Presentation logic (BLoC)
│   │   ├── pages/        # UI screens
│   │   └── widgets/      # Feature-specific widgets
│   ├── expenses/
│   ├── transactions/
│   └── ... (30+ features)
└── remote/           # API clients and services
```

### Dependency Injection Setup

```dart
// lib/core/di/config.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'package:cashsnapp/core/di/config.config.dart';

@InjectableInit(
  throwOnMissingDependencies: true,
)
Future<GetIt> configureInjector(GetIt getIt) {
  return getIt.init();
}
```

All dependencies are registered automatically using `@injectable` annotations, ensuring compile-time safety and preventing runtime injection errors.

### Feature Module Pattern

Each feature follows a consistent structure:
- **Models**: Immutable data classes (Freezed + JSON Serializable)
- **Usecases**: Business logic encapsulation extending `UseCase<Params, Result>`
- **Bloc**: Presentation logic using RxDart streams
- **Pages**: UI implementation
- **Widgets**: Reusable UI components

## Key technical decisions

**Clean Architecture over MVC/MVVM:**
Enforced strict separation between business logic (usecases), presentation (BLoC), and data layers. This enabled independent testing of business rules without UI dependencies and facilitated parallel feature development by multiple developers.

**GetIt + Injectable for dependency injection:**
Compile-time code generation eliminated runtime reflection overhead. `throwOnMissingDependencies: true` caught injection errors at build time rather than production runtime.

**Dartz for functional error handling:**
Used `Either<Failure, Success>` pattern to make error states explicit in the type system. Eliminated uncaught exceptions and forced explicit error handling at every API boundary.

**Hive over SQLite:**
Chose Hive for local persistence due to no-SQL flexibility, type-safe adapters, and zero native dependencies. Critical for offline-first architecture where data schema could evolve without complex migrations.

**RxDart streams over ChangeNotifier:**
Reactive programming with RxDart enabled complex state transformations (debounce, combineLatest, switchMap) and prevented callback hell in multi-step workflows like payment processing.

**Freezed for immutability:**
Code generation ensured all data models were immutable with value equality. Prevented subtle state mutation bugs in async workflows and made state changes explicit through `.copyWith()`.

## Notable challenges solved

**Offline-first payment processing:**
Challenge: Users needed to record transactions offline with eventual consistency. Solution: Implemented two-tier persistence with local Hive storage for pending transactions and background sync queue with exponential backoff. Used timestamp-based versioning to resolve conflicts server-side.

**Multi-provider OAuth flow:**
Challenge: Handling Google, Apple, and Facebook authentication with different token formats and platform-specific requirements. Solution: Created unified `AuthService` abstraction with platform-specific implementations. Used JWT decoder to normalize all provider tokens into consistent internal format.

**Deep linking with app state:**
Challenge: Branch.io deep links needed to work when app was closed, backgrounded, or active with different navigation states. Solution: Deferred deep link handling until after app initialization completed, then used Go Router's declarative routing to restore correct navigation stack state.

**Type-safe API contracts:**
Challenge: Runtime JSON deserialization errors in production. Solution: Freezed + JSON Serializable code generation caught schema mismatches at compile time. Chopper integration generated type-safe API clients from service definitions.

**Stripe 3D Secure flow:**
Challenge: Stripe 3DS requires WebView redirect flow that interrupts normal app navigation. Solution: Implemented modal WebView layer with custom URL scheme handling. Preserved payment context through WebView navigation and restored UI state on completion/cancellation.

## Code highlights

### [lib/features/login/usecases/login_usecase.dart](lib/features/login/usecases/login_usecase.dart)
Demonstrates Clean Architecture usecase pattern with functional error handling:

```dart
@injectable
class LoginUsecase
    extends UseCase<Tuple2<String, String>, Tuple2<String, User>> {
  LoginUsecase(this._authService);

  final AuthService _authService;

  @override
  Future<Either<Failure, Tuple2<String, User>>> execute(
    Tuple2<String, String> params,
  ) async {
    final req = {"email": params.value1, "password": params.value2};
    return await _authService.signin(req).then(_onRes).catchError(_onErr);
  }

  Either<Failure, Tuple2<String, User>> _onRes(
    Response<ProfileResponse> res,
  ) {
    final body = res.body;

    if (res.isSuccessful && body != null) {
      final user = body.user;
      final token = extractBearerToken(body.accessToken);

      if (token == null) {
        return left(
          UnableToLogin(
            error: const ErrorModel(
              message: 'Invalid authorization string format',
            ),
          ),
        );
      }

      return right(Tuple2(token, user));
    }

    final errorBody = res.error as Map<String, dynamic>;

    return left(UnableToLogin(error: ErrorModel.fromJson(errorBody)));
  }

  Either<Failure, Tuple2<String, User>> _onErr(dynamic err) {
    logger.w('Unable to login: $err');

    return left(
      UnableToLogin(error: ErrorModel(message: 'Unable to login: $err')),
    );
  }
}
```

**Why this matters:** Type-safe error handling using `Either<Failure, Success>` eliminates uncaught exceptions. All error paths are explicit in the return type, forcing callers to handle both success and failure cases.

### [lib/features/login/bloc/login_bloc.dart](lib/features/login/bloc/login_bloc.dart)
Demonstrates BLoC abstraction pattern:

```dart
abstract class LoginBloc extends BaseBloc {
  GlobalKey<FormState> get formKey;
  TextEditingController get emailController;
  TextEditingController get passwordController;

  Stream<bool> get loading;

  void onSignIn();
}
```

**Why this matters:** Interface segregation through abstract base classes. UI depends on interface, not implementation. Enables testability and allows swapping implementations without modifying UI code.

### [lib/core/di/](lib/core/di/)
Shows dependency injection setup with automatic registration. Injectable annotations (`@injectable`, `@lazySingleton`, `@singleton`) enable compile-time dependency graph validation.

### [lib/features/](lib/features/)
Contains 30+ feature modules demonstrating consistent architectural patterns:
- Expenses, Income tracking
- Transactions and payments
- Reports and analytics
- User profile management
- Settings and categories

## Deployment & environment

**Build & Release:**
- CI/CD pipeline with GitHub Actions
- Automated builds for iOS (TestFlight) and Android (Firebase App Distribution)
- Environment configurations via Flutter build flavors (dev, staging, production)

**Environment Management:**
- Build flavors for environment-specific configs
- Firebase configuration per environment
- Stripe publishable keys via environment variables

**Release Process:**
- Version management via pubspec.yaml (`1.0.37+49`)
- Code signing for iOS (Apple Developer certificates)
- Android signing with release keystore
- Firebase Crashlytics for production error tracking
- Firebase Analytics for user behavior tracking

**Production Monitoring:**
- Firebase Crashlytics for real-time crash reporting
- Custom logging with debug/release build separation
- Analytics events for critical user flows

## Public links

Private commercial project under NDA. Source code available upon request for portfolio review.
