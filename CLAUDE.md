# CLAUDE.md - Flutter Location Plugin

## Project Overview

This is a **Flutter location services plugin** providing cross-platform location tracking with permission management, background updates, and configurable accuracy. It's a **monorepo** managed by Melos containing three packages.

- **Author**: Guillaume Bernos (Lyokone)
- **License**: MIT
- **Current Version**: 4.3.0
- **Dart SDK**: >=2.12.0 <3.0.0 (null-safe)

## Repository Structure

```
flutterlocation/
├── packages/
│   ├── location/                    # Main plugin package (public API)
│   ├── location_platform_interface/ # Platform interface definitions
│   └── location_web/                # Web implementation
├── docs/                            # Web demo deployment
├── .github/                         # GitHub workflows and templates
├── .ci/                             # CI scripts
├── melos.yaml                       # Monorepo configuration
└── analysis_options.yaml            # Dart linting rules
```

## Build & Test Commands

```bash
# Bootstrap the monorepo (install dependencies for all packages)
melos bootstrap

# Run static analysis on all packages
melos run analyze

# Format all Dart code
melos run format

# Run all unit tests
melos run test

# Run complete lint (analyze + format)
melos run lint:all

# Run E2E tests on mobile
melos run test:mobile_e2e
```

### Package-Specific Commands

```bash
# Run tests for a specific package
cd packages/location && flutter test

# Generate mocks for tests
cd packages/location && flutter pub run build_runner build

# Run example app
cd packages/location/example && flutter run
```

## Architecture

### Three-Layer Platform Channel Pattern

1. **Public API** (`packages/location/lib/location.dart`) - User-facing facade
2. **Platform Interface** (`packages/location_platform_interface/`) - Abstract definitions
3. **Platform Implementations**:
   - Android: Java/Kotlin in `packages/location/android/`
   - iOS: Objective-C in `packages/location/ios/`
   - macOS: Objective-C in `packages/location/macos/`
   - Web: Dart+JS in `packages/location_web/`

### Channel Names
- **Method Channel**: `lyokone/location`
- **Event Channel**: `lyokone/locationstream`

## Key Files

| File | Purpose |
|------|---------|
| `packages/location/lib/location.dart` | Main public API (singleton pattern) |
| `packages/location_platform_interface/lib/src/types.dart` | Data models (LocationData, PermissionStatus, LocationAccuracy) |
| `packages/location_platform_interface/lib/src/method_channel_location.dart` | MethodChannel bridge |
| `packages/location/android/src/main/java/com/lyokone/location/FlutterLocation.java` | Core Android logic |
| `packages/location/android/src/main/java/com/lyokone/location/FlutterLocationService.kt` | Android background service |
| `packages/location/ios/Classes/LocationPlugin.m` | iOS CoreLocation implementation |
| `packages/location_web/lib/location_web.dart` | Web Geolocation API implementation |

## Platform Support

| Platform | Status | Min Version | Notes |
|----------|--------|-------------|-------|
| Android | Full | API 21 (5.0) | Uses Fused Location Provider |
| iOS | Full | iOS 8.0+ | Uses CoreLocation |
| macOS | Full | macOS 10.11+ | Uses CoreLocation |
| Web | Limited | All browsers | No background mode, limited accuracy |
| Linux/Windows | Not supported | - | - |

## Code Conventions

### Dart Style
- **Null safety**: Full null-safety required
- **Naming**: camelCase for variables/functions, PascalCase for types
- **Types**: Always specify return types explicitly
- **Const**: Prefer const constructors and literals
- **Imports**: No relative imports to lib, use package imports

### Key Patterns
- **Singleton**: `Location.instance` / `Location()` factory
- **Async/Await**: All location operations are async
- **Streams**: Location updates via `onLocationChanged` stream
- **Platform Exceptions**: Native errors wrapped in `PlatformException`

## Android Configuration

- **compileSdkVersion**: 31
- **minSdkVersion**: 21
- **Kotlin**: 1.4.20
- **Play Services Location**: 18.0.0

Required permissions in AndroidManifest.xml:
- `ACCESS_FINE_LOCATION`
- `ACCESS_COARSE_LOCATION`
- `FOREGROUND_SERVICE` (for background mode)

## iOS Configuration

Required Info.plist keys:
- `NSLocationWhenInUseUsageDescription`
- `NSLocationAlwaysAndWhenInUseUsageDescription`

For background mode, add "location" to `UIBackgroundModes`.

## Data Models

### LocationData
Contains: `latitude`, `longitude`, `accuracy`, `altitude`, `speed`, `speedAccuracy`, `heading`, `time`, `isMock`

### PermissionStatus
- `granted` - Full permission
- `grantedLimited` - iOS 14+ reduced accuracy
- `denied` - Temporarily denied
- `deniedForever` - Permanently denied

### LocationAccuracy
Levels: `powerSave`, `low`, `balanced`, `high`, `navigation`, `reduced`

## Testing

- **Framework**: Flutter Test + Mockito
- **Mock Generation**: Uses `@GenerateMocks` annotation
- **CI**: Cirrus CI with Flutter stable/master channels
- **Coverage**: Integrated with CodeCov

Test files location:
- `packages/location/test/`
- `packages/location_platform_interface/test/`

## Common Development Tasks

### Adding a new platform method
1. Add abstract method to `LocationPlatform` in `location_platform_interface`
2. Implement in `MethodChannelLocation`
3. Add to `Location` class public API
4. Implement native code in each platform
5. Add tests

### Modifying LocationData
1. Update `types.dart` in `location_platform_interface`
2. Update `fromMap` factory constructor
3. Update native platform serialization
4. Update tests in `types_test.dart`

## Important Notes

- Always run `melos bootstrap` after cloning or updating dependencies
- The example app at `packages/location/example/` is the best way to test changes
- Android background service requires notification customization for user experience
- iOS 14+ introduced "reduced accuracy" which should be handled gracefully
- Web implementation has significant limitations compared to native platforms
