# Lsync - Localization Sync CLI Tool

## What it is

A blazing-fast command-line tool written in Rust that synchronizes translations from Google Sheets and generates type-safe localization files for Flutter (.arb + Dart) and React/Next.js projects (JSON). Automates the translation workflow by eliminating manual copy-paste and ensuring consistency between design team's spreadsheets and production code.

## My role

I architected and developed the complete CLI tool as a personal developer productivity project. Responsible for Rust implementation, Google Sheets API integration with OAuth2 flow, code generation templates for multiple frameworks, cross-platform compilation, Homebrew tap creation for macOS distribution, and Next.js landing page for documentation.

## Tech stack

### CLI Tool (Rust)
**Language & Runtime:**
- Rust 2021 edition
- Tokio 1.x async runtime
- Single binary with zero runtime dependencies

**CLI Framework:**
- Clap 4 (with derive macros for subcommands and argument parsing)

**Google Sheets Integration:**
- google-sheets4 crate (official Google API client)
- yup-oauth2 for OAuth2 authentication flow
- Hyper + Rustls for HTTPS (no OpenSSL dependency)

**HTTP & Networking:**
- Reqwest 0.11 (with rustls-tls feature, no system SSL)
- Tiny HTTP 0.12 (local server for OAuth callback)
- hyper-rustls 0.27.7 for TLS

**Serialization:**
- Serde + Serde JSON for data processing

**Additional:**
- Colored (terminal output coloring)
- Open (browser launch for OAuth)
- Dotenvy (environment variable loading)

### Landing Page (Next.js)
**Framework:**
- Next.js 15.3.5 (App Router)
- React 19.1.0
- Static site export

**Styling:**
- Tailwind CSS 4 (via @tailwindcss/postcss)

### Distribution
**Homebrew Tap:**
- Custom formula for macOS installation
- Binary distribution via GitHub Releases

## System architecture

### CLI Architecture (Command Pattern)

```
lsync_cli/src/
├── main.rs                      # Entry point, Clap CLI setup
├── commands/
│   ├── setup.rs                # OAuth flow + credentials storage
│   └── sync.rs                 # Fetch sheets + generate files
├── google_sheet_api_client.rs  # Google Sheets API wrapper
├── models/
│   ├── translation.rs          # Translation data structures
│   └── config.rs               # Project configuration
├── utils/
│   ├── constants.rs            # OAuth URIs, app metadata
│   ├── logging.rs              # Colored terminal output
│   └── codegen/
│       ├── flutter.rs          # .arb + Dart generator
│       └── react.rs            # JSON locale generator
└── env.rs                      # Environment variable loading
```

### Command Flow

**Setup Command:**
```
lsync setup
    ↓
Load CLIENT_ID & CLIENT_SECRET from .env
    ↓
Launch OAuth2 flow (InstalledFlowAuthenticator)
    ↓
Start local HTTP server on port 8080
    ↓
Open browser to Google consent screen
    ↓
User authorizes, redirected to localhost:8080
    ↓
Exchange code for access token
    ↓
Persist token to .lsync/credentials.json
    ↓
✓ Setup complete
```

**Sync Command:**
```
lsync sync
    ↓
Load credentials from .lsync/credentials.json
    ↓
Authenticate with Google Sheets API
    ↓
Fetch spreadsheet data (all languages)
    ↓
Parse rows into Translation structs
    ↓
Detect project type (Flutter or React/Next.js)
    ↓
Generate localization files:
  Flutter:
    ├→ lib/l10n/intl_en.arb
    ├→ lib/l10n/intl_es.arb
    └→ lib/l10n/app_localizations.dart
  React:
    ├→ public/locales/en/translation.json
    └→ public/locales/es/translation.json
    ↓
✓ Sync complete
```

### Google Sheets API Integration

```rust
pub struct GoogleSheetApiClient {}

impl GoogleSheetApiClient {
    pub async fn get_client() -> Sheets<AppConnector> {
        let client_secret = yup_oauth2::ApplicationSecret {
            client_id: CLIENT_ID.to_string(),
            client_secret: CLIENT_SECRET.to_string(),
            auth_uri: AUTH_URI.to_string(),
            token_uri: TOKEN_URI.to_string(),
            redirect_uris: vec![REDIRECT_URI_1.to_string()],
            ..Default::default()
        };

        let auth = yup_oauth2::InstalledFlowAuthenticator::builder(
            client_secret,
            yup_oauth2::InstalledFlowReturnMethod::HTTPRedirect,
        )
        .persist_tokens_to_disk(format!(".{}/credentials.json", APPNAME))
        .build()
        .await
        .unwrap_or_else(|e| {
            log_error(&format!("Google OAuth failed: {}", e));
            std::process::exit(1);
        });

        let http_connector = hyper_rustls::HttpsConnectorBuilder::new()
            .with_native_roots()
            .unwrap()
            .https_or_http()
            .enable_http1()
            .build();

        let client = HyperLegacyClient::builder(TokioExecutor::new())
            .build(http_connector);

        Sheets::new(client, auth)
    }
}
```

## Key technical decisions

**Rust over Node.js/Python:**
Compiled Rust binary has zero runtime dependencies - no Node.js installation, no Python interpreter, no system SSL libraries (rustls). Single executable distributable via GitHub Releases or Homebrew. Startup time: <10ms vs Node.js >500ms. Memory usage: ~5MB vs Node.js ~50MB.

**Tokio async runtime:**
Google Sheets API requires async HTTP requests. Tokio provides production-grade async I/O with hyper compatibility. Enables concurrent processing if syncing multiple sheets simultaneously.

**Rustls over OpenSSL:**
Eliminated system SSL library dependencies (libssl-dev, openssl). Cross-compilation simplified - macOS, Linux, Windows binaries built without target-specific SSL setup. Reduces security surface area.

**Clap derive macros over manual parsing:**
Compile-time command structure validation. Help text auto-generated. Subcommand pattern (`lsync setup`, `lsync sync`) with typed arguments prevents runtime parsing errors.

**Persist tokens to disk:**
OAuth tokens saved in `.lsync/credentials.json` eliminates re-authentication on every sync. Refresh tokens handled automatically by yup-oauth2. Token storage follows XDG directory conventions.

**Template-based code generation:**
Flutter and React have different localization formats. Template pattern allows adding new target frameworks (Vue i18n, Angular i18n) without modifying core sync logic.

**Homebrew tap for distribution:**
macOS developers install via `brew install your-tap/lsync`. Auto-updates via Homebrew. Alternative to npm global install which pollutes Node.js global namespace.

## Notable challenges solved

**OAuth2 redirect flow in CLI:**
Challenge: OAuth requires web redirect, but CLI has no HTTP server. Solution: Spawn ephemeral HTTP server on localhost:8080 using tiny_http. Server listens for one request (OAuth callback), extracts auth code, shuts down. Browser automatically opens to Google consent screen via `open` crate.

**Cross-platform binary compilation:**
Challenge: Building binaries for macOS (Intel + ARM), Linux (x86_64, ARM), Windows (x86_64). Solution: GitHub Actions matrix builds with cross-compilation targets. Rustls eliminates OpenSSL dependency hell. Artifacts uploaded to GitHub Releases.

**Type-safe translation keys:**
Challenge: Developers typing incorrect translation keys at runtime ("hero.titel" instead of "hero.title"). Solution: Flutter code generator creates Dart class with getter methods. TypeScript users import JSON with `as const` for literal type inference. Compile-time errors for invalid keys.

**Handling missing translations:**
Challenge: Sheet has English and Spanish, but Spanish row missing for key "settings.title". Solution: Fallback strategy - use English value if translation missing. Log warning to console. Prevents blank UI text in production.

**Nested translation keys:**
Challenge: Flat spreadsheet structure needs to map to nested JSON (`{ "hero": { "title": "Welcome" } }`). Solution: Key column uses dot notation ("hero.title"). Parser splits on dots and builds nested object tree. Works for arbitrary nesting depth.

**Sheet format detection:**
Challenge: Different teams use different sheet layouts (columns: key/en/es vs en/es/key). Solution: Auto-detect header row position and column order. Flexible parser supports multiple common formats.

## Code highlights

### [src/main.rs](src/main.rs) - CLI Entry Point

```rust
mod commands;
mod env;
mod google_sheet_api_client;
mod models;
mod utils;

use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "lsync", version, about = "Sync translations from Google Sheets")]
#[command(arg_required_else_help = true)]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    #[command(name = "setup", about = "Setup the fetcher")]
    Setup,
    #[command(name = "sync", about = "Sync latest available translations")]
    Sync,
}

#[tokio::main]
async fn main() {
    dotenvy::dotenv().ok();

    let cli = Cli::parse();

    match cli.command {
        Commands::Setup => {
            commands::setup::Setup::run().await;
        }
        Commands::Sync => {
            commands::sync::Sync::run().await;
        }
    }
}
```

**Why this matters:** Demonstrates Clap's derive macro ergonomics. Type-safe command dispatch via pattern matching. Tokio runtime annotation (`#[tokio::main]`) enables async Google Sheets API calls.

### [src/google_sheet_api_client.rs](src/google_sheet_api_client.rs) - OAuth Integration

Shows OAuth2 flow with InstalledFlowAuthenticator, token persistence, and rustls HTTP client configuration. Eliminates need for service account JSON - uses user OAuth for personal spreadsheets.

### [src/commands/sync.rs](src/commands/sync.rs) - Sync Logic

Fetches sheet data, parses rows into translation map, detects project type (Flutter vs React), generates code via templates.

### [src/utils/codegen/](src/utils/codegen/) - Code Generators

**Flutter Generator:**
- Outputs `.arb` files (JSON format with `@key` metadata)
- Generates `app_localizations.dart` with type-safe getters
- Integrates with Flutter's `flutter_localizations` package

**React Generator:**
- Outputs nested JSON files per locale
- Compatible with i18next, react-i18next, next-i18next
- Supports namespace splitting

## Deployment & environment

**Development:**
```bash
cd lsync_cli

# Run in watch mode
cargo watch -x run

# Build release binary
cargo build --release

# Binary location
./target/release/lsync
```

**Environment Variables:**
```bash
# .env file
CLIENT_ID=your-google-client-id
CLIENT_SECRET=your-google-client-secret
SPREADSHEET_ID=1abc...xyz
```

**Installation Methods:**

**Homebrew (macOS):**
```bash
brew tap your-username/lsync
brew install lsync
```

**Binary Download:**
```bash
# Download from GitHub Releases
curl -L https://github.com/your-username/lsync/releases/download/v0.1.3/lsync-macos -o lsync
chmod +x lsync
sudo mv lsync /usr/local/bin/
```

**Cargo (all platforms):**
```bash
cargo install lsync
```

**Usage:**
```bash
# First time setup
lsync setup

# Sync translations
lsync sync
```

**Build Optimizations:**
```toml
[profile.release]
opt-level = "z"      # Optimize for size
lto = true           # Link-time optimization
codegen-units = 1    # Single codegen unit for better optimization
```

Final binary size: ~8MB (includes Google Sheets API client).

**Cross-Platform Builds:**
- GitHub Actions builds for macOS (Intel + ARM), Linux (x86_64), Windows (x86_64)
- Artifacts uploaded to GitHub Releases
- Homebrew formula points to macOS binary URL

## Landing Page

**Next.js Static Site:**
- Documentation and installation instructions
- Code examples for Flutter and React
- Deployed to Vercel/Netlify
- Repository: `lsync/landing/`

## Public links

Open-source personal project. GitHub repository, Homebrew tap, and documentation site URLs available upon request for portfolio review.
