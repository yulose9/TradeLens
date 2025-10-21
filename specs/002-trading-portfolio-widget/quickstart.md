# Quickstart: Trading Portfolio Widget Development

**Feature**: Trading Portfolio Widget
**Branch**: `002-trading-portfolio-widget`
**Last Updated**: 2025-10-21

## Prerequisites

### Required Tools
- **JDK**: OpenJDK 17 or higher (Android requires JDK 11+, but JDK 17 recommended)
- **Android SDK**: Android SDK Platform 34 (Android 14) with Build Tools 34.0.0+
- **Android SDK CLI Tools**: For command-line builds and emulator management
- **Git**: Version control
- **git-secrets**: For pre-commit secret detection hook

### Optional Tools
- **Android Studio**: Hedgehog (2023.1.1) or later (recommended for debugging)
- **VS Code**: With Kotlin Language extension (alternative for lightweight editing)
- **ADB**: Android Debug Bridge (included with Android SDK Platform Tools)

### System Requirements
- **OS**: Linux (Ubuntu 22.04+ recommended), macOS 12+, or Windows 10+ with WSL2
- **RAM**: 8GB minimum, 16GB recommended (for emulator)
- **Disk**: 20GB free space (Android SDK + emulator images)

---

## Initial Setup

### 1. Clone Repository

```bash
git clone https://github.com/yulose9/TradeLens.git
cd TradeLens
git checkout 002-trading-portfolio-widget
```

### 2. Install Android SDK (Command-Line Setup)

#### On Ubuntu/Debian Linux:
```bash
# Install JDK
sudo apt update
sudo apt install openjdk-17-jdk

# Download Android SDK Command-Line Tools
mkdir -p ~/Android/Sdk
cd ~/Android/Sdk
wget https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip
unzip commandlinetools-linux-9477386_latest.zip -d cmdline-tools
mv cmdline-tools/cmdline-tools cmdline-tools/latest

# Set environment variables (add to ~/.bashrc or ~/.zshrc)
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator

# Install required SDK components
sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
sdkmanager "system-images;android-34;google_apis;x86_64"
sdkmanager "emulator"
```

#### On macOS:
```bash
# Install JDK via Homebrew
brew install openjdk@17

# Download Android SDK Command-Line Tools
mkdir -p ~/Library/Android/sdk
cd ~/Library/Android/sdk
curl -O https://dl.google.com/android/repository/commandlinetools-mac-9477386_latest.zip
unzip commandlinetools-mac-9477386_latest.zip -d cmdline-tools
mv cmdline-tools/cmdline-tools cmdline-tools/latest

# Set environment variables (add to ~/.zshrc or ~/.bash_profile)
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator

# Install required SDK components
sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
sdkmanager "system-images;android-34;google_apis;arm64-v8a"
sdkmanager "emulator"
```

### 3. Install git-secrets (Pre-commit Hook)

```bash
# On Ubuntu/Debian
git clone https://github.com/awslabs/git-secrets.git
cd git-secrets
sudo make install

# On macOS
brew install git-secrets

# Configure git-secrets in repo
cd /path/to/TradeLens
git secrets --install
git secrets --register-aws  # Adds AWS patterns
# Add custom patterns for Trading 212 tokens
git secrets --add '(api[-_]?key|token|secret|password)\s*[:=]\s*['\''"][^'\''\"]{20,}['\''"]'
```

### 4. Verify Setup

```bash
# Check Java version
java -version  # Should show 17.x.x

# Check Android SDK
sdkmanager --list_installed

# Check Gradle (uses wrapper, no global install needed)
cd /path/to/TradeLens
./gradlew --version
```

---

## Build & Run

### Build Debug APK

```bash
# From repository root
./gradlew assembleDebug

# Output: app/build/outputs/apk/debug/app-debug.apk
```

### Run on Emulator

#### Create Emulator (first time only)
```bash
# Create AVD (Android Virtual Device)
avdmanager create avd \
  --name Pixel_6_API_34 \
  --package "system-images;android-34;google_apis;x86_64" \
  --device "pixel_6"

# List available AVDs
emulator -list-avds
```

#### Start Emulator
```bash
# Start in background
emulator -avd Pixel_6_API_34 -no-snapshot-load &

# Wait for boot (check with adb)
adb wait-for-device
adb shell getprop sys.boot_completed  # Should output: 1
```

#### Install APK
```bash
# Install debug APK
adb install -r app/build/outputs/apk/debug/app-debug.apk

# Launch app
adb shell am start -n com.tradelens/.ui.settings.SettingsActivity
```

### Run on Physical Device

```bash
# Enable USB Debugging on device (Settings > Developer Options)
# Connect device via USB

# Verify device connection
adb devices

# Install and run
./gradlew installDebug
adb shell am start -n com.tradelens/.ui.settings.SettingsActivity
```

---

## Development Workflow

### Run Static Analysis

```bash
# ktlint (code formatting)
./gradlew ktlintCheck
./gradlew ktlintFormat  # Auto-fix issues

# detekt (code quality)
./gradlew detekt
```

### Run Tests

```bash
# Unit tests
./gradlew testDebugUnitTest

# Unit tests with coverage
./gradlew testDebugUnitTest jacocoTestReport
# View report: app/build/reports/jacoco/test/html/index.html

# Instrumentation tests (requires emulator/device)
./gradlew connectedDebugAndroidTest
```

### Clean Build

```bash
./gradlew clean assembleDebug
```

---

## Widget Development

### Add Widget to Home Screen (Emulator)

1. Long-press on home screen
2. Tap "Widgets"
3. Find "TradeLens Portfolio"
4. Drag to home screen
5. Select widget size (1x1, 2x2, 4x1, 4x2)
6. Configure in settings if prompted

### View Widget Logs

```bash
# Filter logs for widget updates
adb logcat | grep "PortfolioWidget"

# View WorkManager logs
adb logcat | grep "WM-"
```

### Force Widget Update (Debug)

```bash
# Trigger manual refresh broadcast
adb shell am broadcast \
  -a com.tradelens.WIDGET_REFRESH \
  -n com.tradelens/.ui.widget.WidgetReceiver
```

---

## Configuration

### API Token Setup (Development)

**SECURITY WARNING**: Never commit API tokens to source control!

#### Option 1: EncryptedSharedPreferences (Recommended)
- Enter token via Settings UI in app
- Token is encrypted at rest on device

#### Option 2: Environment Variable (CI/Debug)
```bash
# Add to ~/.bashrc or ~/.zshrc
export TRADING212_API_TOKEN="your-token-here"

# Access in build.gradle.kts
buildConfigField("String", "API_TOKEN", "\"${System.getenv("TRADING212_API_TOKEN") ?: ""}\"")
```

### Build Variants

```bash
# Debug build (with logging, no ProGuard)
./gradlew assembleDebug

# Release build (ProGuard enabled, signing required)
./gradlew assembleRelease
```

---

## Debugging

### Android Studio Setup (Optional)

1. Open Android Studio
2. File > Open > Select `/path/to/TradeLens`
3. Wait for Gradle sync
4. Run > Edit Configurations > Add Android App configuration
5. Select module: `app`
6. Run/Debug with breakpoints

### VS Code Setup (Lightweight)

1. Install extensions:
   - Kotlin Language (mathiasfrohlich.Kotlin)
   - Gradle for Java (vscjava.vscode-gradle)
2. Open workspace: `code /path/to/TradeLens`
3. Use integrated terminal for Gradle commands
4. Use Android Studio for advanced debugging

### Common Issues

#### Issue: "SDK location not found"
```bash
# Create local.properties
echo "sdk.dir=$ANDROID_HOME" > local.properties
```

#### Issue: "Daemon process timed out"
```bash
# Increase Gradle heap size
echo "org.gradle.jvmargs=-Xmx4096m" >> gradle.properties
```

#### Issue: Widget not updating
```bash
# Check WorkManager constraints
adb shell dumpsys jobscheduler | grep com.tradelens

# Force WorkManager execution
adb shell cmd jobscheduler run -f com.tradelens <job-id>
```

---

## CI/CD (GitHub Actions)

### Local CI Simulation

```bash
# Run full CI pipeline locally
./gradlew clean \
  ktlintCheck \
  detekt \
  testDebugUnitTest \
  assembleDebug
```

### View CI Results

- GitHub Actions: `https://github.com/yulose9/TradeLens/actions`
- PR checks must pass before merge

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                  Presentation Layer                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ Glance Widget│  │Settings Screen│  │ViewModels │ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
└─────────────────────────────────────────────────────┘
                         │
┌─────────────────────────────────────────────────────┐
│                    Domain Layer                      │
│  ┌──────────────────────────────────────────────┐  │
│  │  UseCases (GetPortfolio, RefreshToken, etc) │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
                         │
┌─────────────────────────────────────────────────────┐
│                     Data Layer                       │
│  ┌──────────────┐  ┌─────────────┐  ┌────────────┐ │
│  │ Repositories │  │ Room Cache  │  │ Retrofit   │ │
│  │              │  │ (Offline)   │  │ (API)      │ │
│  └──────────────┘  └─────────────┘  └────────────┘ │
└─────────────────────────────────────────────────────┘
                         │
                 ┌───────┴────────┐
                 │                │
          ┌──────▼────┐    ┌─────▼──────┐
          │  Device   │    │  Trading   │
          │  Storage  │    │ 212 API    │
          └───────────┘    └────────────┘
```

---

## Key Files Reference

| File Path | Description |
|-----------|-------------|
| `app/build.gradle.kts` | App-level Gradle config (dependencies, build variants) |
| `app/src/main/AndroidManifest.xml` | App manifest (permissions, widget provider) |
| `app/src/main/java/com/tradelens/di/AppModule.kt` | Hilt DI configuration |
| `app/src/main/java/com/tradelens/ui/widget/PortfolioWidget.kt` | Main widget Glance composable |
| `app/src/main/java/com/tradelens/data/remote/Trading212Api.kt` | Retrofit API interface |
| `app/src/main/res/xml/widget_info.xml` | Widget metadata (sizes, update frequency) |
| `.github/workflows/ci.yml` | CI pipeline configuration |

---

## Next Steps

1. **Implement Phase 1 Infrastructure**:
   - Set up Hilt modules
   - Configure Retrofit + OkHttp
   - Create Room database schema
   - Implement EncryptedSharedPreferences wrapper

2. **Build MVP Widget (v1)**:
   - Implement `GetPortfolioSnapshotUseCase`
   - Create `PortfolioWidget` with 2x2 layout
   - Show balance + P/L only
   - Add manual refresh action

3. **Add Testing**:
   - Unit tests for repositories and use cases
   - Mock API responses with MockK
   - Robolectric tests for ViewModels
   - Achieve ≥60% coverage on core modules

4. **Set Up CI**:
   - Configure GitHub Actions workflow
   - Add ktlint, detekt, unit tests to CI
   - Set up Dependabot for dependency scanning

---

## Support

- **Documentation**: `/specs/002-trading-portfolio-widget/`
- **Constitution**: `/.specify/memory/constitution.md`
- **Issues**: https://github.com/yulose9/TradeLens/issues
- **Pull Requests**: https://github.com/yulose9/TradeLens/pulls

For questions or issues, open a GitHub issue with the `002-trading-portfolio-widget` label.
