# CACHELINES Field Intelligence & Beat Management System ("Beat Book")

An offline-first, mission-critical field intelligence and territorial beat management system designed for authorized law enforcement officers, community policing personnel, field intelligence units, and administrative leadership.

---

## 1. Project Overview
**CACHELINES Field Intelligence & Beat Management System** digitizes traditional police and field beat management into a structured, highly secure, and resilient platform. Built to function without reliable internet connectivity, it empowers patrol officers to document local community infrastructure, track vital neighborhood contacts, conduct point-in-time location logging, record structured field intelligence observations, and compile daily operational briefs with full offline resilience.

---

## 2. Current Implementation Status
* **Milestones Status:** Milestones 1 through 9 are **COMPLETE and FROZEN**.
* **Application Version:** `1.0.0+1` (Build `2026.1`)
* **Local Database Engine:** SQLite Schema Version `v7` (`cachelines_beatbook.db`)
* **Quality Assurance Baseline:** 
  * `flutter analyze`: **0 issues found**
  * `flutter test`: **275 / 275 passing tests (100% pass rate)**

---

## 3. Platform Status

| Platform | Target Architecture | Build Status | Runtime Status | Distribution Artifact |
| :--- | :--- | :--- | :--- | :--- |
| **Windows Desktop** | `x64` | **PASS (Release Build)** | **VERIFIED (Clean-Machine Acceptance Passed)** | Standalone Setup Wizard (`.exe`) / Portable |
| **Android Mobile** | `arm64-v8a`, `armeabi-v7a`, `x86_64` | **PASS (Release APK & AAB Built)** | **BUILD VERIFIED (Device Acceptance Pending)** | Standalone APK (`.apk`) / App Bundle (`.aab`) |

---

## 4. Feature Overview

### 4.1 Territorial Hierarchy (Beat Book)
* Multilevel jurisdictional navigation: **District $\rightarrow$ Tehsil $\rightarrow$ Police Station $\rightarrow$ Beat $\rightarrow$ Area / Locality $\rightarrow$ Village**.
* Full spatial demarcation: Explicit North, South, East, and West territorial boundaries for each Beat.
* Community directory integration: Catalogs local village headmen (Numberdars) and inter-agency government officers.

### 4.2 People & Places Directory
* **Community Contacts:** Legitimate public-role contacts, neighborhood representatives, and verified operational persons.
* **Institutional Places:** Documentation of critical infrastructure, financial institutions, educational sites, places of worship, and commercial hubs.
* **Person-Place Roles:** Direct relational linking (e.g. Bank Manager, School Headmaster, Caretaker).

### 4.3 Field Records & Dynamic Schema
* **Configurable Categories:** Tenant verifications, security audits, dispute precursors, and routine observations.
* **Dynamic Form Attributes:** Typed data fields (`text`, `number`, `date`, `dropdown`, `boolean`) with validation.
* **Follow-Ups & Verification:** Supervisory review workflows (`unverified`, `verified`, `rejected`) and scheduled re-inspections.

### 4.4 Daily Reports & Explicit Sharing
* Automated aggregation of daily field observations and pending follow-ups.
* Supervisory note attachment.
* **Explicit, User-Initiated Dissemination:** WhatsApp deep-link (`whatsapp://send?text=...`), WhatsApp Web, native OS share sheet, and clipboard fallback. (Zero automated or background messaging).

### 4.5 Offline Outbox & Conflict Resolution
* Local mutation queue (`sync_outbox`) with idempotency keys and exponential backoff.
* Quarantined conflict tracking (`sync_conflicts`) supporting `keepLocal`, `keepRemote`, and `merge` strategies.
* Platform-neutral multi-client context (`SyncClientContext`).

### 4.6 Cryptographic Backup & Restoration
* Versioned `.clbak` archive packages (Format v1, Schema v7) protected by cryptographic SHA-256 checksums.
* Automatic pre-restore safety snapshot (`system_pre_restore_safety`).
* Atomic transactional restoration within isolated SQLite database transactions.

### 4.7 Multi-Device Licensing & Financial Ledger
* Terminal Installation Identity (`DEV-...`) bound via secure storage.
* Multi-terminal license capacity management with 30-day offline grace periods.
* Invoicing and transaction tracking stored in minor currency units (cents/paisas).

---

## 5. System Architecture
CACHELINES follows **Clean Architecture** with a strict feature-first organization:
* **Presentation Layer:** Flutter Material 3 UI with desktop and mobile responsive layouts, state managed via `ChangeNotifier` and `Provider`.
* **Domain Layer:** Pure Dart entities, repository contracts, and isolated domain services (`LicenseValidationService`, `BillingAuthorizationService`, `SyncAuthorizationService`).
* **Data Layer:** SQLite implementations (`sqflite` on Android, `sqflite_common_ffi` on Windows), hardware-backed secure storage (`flutter_secure_storage`), and abstract adapter boundaries.

```text
lib/
├── app/                  # App initialization, routing, dependency injection
├── core/                 # Config, SQLite migrations, security, utilities
├── features/             # Feature modules (beat, people_places, sync, license, etc.)
└── shared/               # Shared domain models and UI widgets
```

---

## 6. Screenshots & Interface Previews
*(Official UI screenshot assets are preserved in agency design archives)*

| Desktop Operations Dashboard | Mobile Field Records & GPS |
| :---: | :---: |
| *[Desktop Overview - Territory & Beat Management]* | *[Mobile Interface - Point Location & Dynamic Records]* |

---

## 7. Installation

### Windows Desktop
1. Download `CACHELINES_v1.0.0_Windows_x64_Installer.exe`.
2. Run the installer wizard and follow the setup instructions.
3. If Windows SmartScreen prompts, click **More info** $\rightarrow$ **Run anyway** (Authenticode EV signing pending external enterprise certificate).
4. Launch **CACHELINES Beat Management** from the Start Menu or Desktop shortcut.

### Android Mobile
1. Transfer `app-release.apk` to an authorized Android terminal (Android 8.0+ / API 26+).
2. Install via package manager or execute ADB install:
   ```powershell
   adb install -r app-release.apk
   ```
3. Grant camera and point-in-time location permissions when prompted at first operational use.

---

## 8. Development Setup

### Prerequisites
* **Flutter SDK:** Version 3.47.5 stable (or `>= 3.10.0`)
* **Dart SDK:** Version 3.13.4 (or `>= 3.0.0`)
* **Windows Development:** Visual Studio 2022 with *"Desktop development with C++"* workload.
* **Android Development:** JDK 17 (`JAVA_HOME`) and Android SDK Platforms 34/35 (`ANDROID_HOME`).

### Initializing Environment
```powershell
# Clone the repository
git clone https://github.com/cachelines/beatbook.git
cd beatbook

# Install Flutter dependencies
flutter pub get
```

---

## 9. Running Automated Tests

The repository maintains an extensive test suite verifying domain logic, migrations, security sanitization, and UI interactions:

```powershell
# Run full automated test suite (275 tests)
flutter test

# Run static analysis
flutter analyze
```

---

## 10. Building From Source

### Building Windows Release
```powershell
flutter build windows --release
```
* Executable: `build/windows/x64/runner/Release/beatbook.exe`

### Building Android Release
```powershell
# Build standalone release APK
flutter build apk --release

# Build Google Play App Bundle
flutter build appbundle --release
```
* Outputs: `build/app/outputs/flutter-apk/app-release.apk` and `build/app/outputs/bundle/release/app-release.aab`

---

## 11. Documentation Links

Comprehensive documentation is provided in the `docs/` directory:

* **User Guides:**
  * [Getting Started](file:///D:/beatbook/docs/user/getting-started.md)
  * [Installation Guide](file:///D:/beatbook/docs/user/installation.md)
  * [Login & Account](file:///D:/beatbook/docs/user/login-and-account.md)
  * [Beat Book & Territory](file:///D:/beatbook/docs/user/beat-book.md)
  * [People Directory](file:///D:/beatbook/docs/user/people.md)
  * [Places Directory](file:///D:/beatbook/docs/user/places.md)
  * [GPS & Location Policy](file:///D:/beatbook/docs/user/gps.md)
  * [Media & Evidence](file:///D:/beatbook/docs/user/media.md)
  * [Daily Reports](file:///D:/beatbook/docs/user/daily-reports.md)
  * [Offline Operation](file:///D:/beatbook/docs/user/offline-mode.md)
  * [Backup & Restore](file:///D:/beatbook/docs/user/backup-restore.md)
* **Developer Guides:**
  * [Architecture & Design](file:///D:/beatbook/docs/developer/architecture.md)
  * [Database & Data Dictionary](file:///D:/beatbook/docs/developer/database.md)
  * [Database Migrations](file:///D:/beatbook/docs/developer/migrations.md)
  * [Security & Threat Model](file:///D:/beatbook/docs/developer/security.md)
  * [Testing Baseline](file:///D:/beatbook/docs/developer/testing.md)
  * [Android Engineering](file:///D:/beatbook/docs/developer/android-development.md)
  * [Windows Engineering](file:///D:/beatbook/docs/developer/windows-development.md)
  * [Known Limitations](file:///D:/beatbook/docs/developer/known-limitations.md)
* **Release & Roadmaps:**
  * [Release Checklist](file:///D:/beatbook/docs/release/release-checklist.md)
  * [Future Product Requirements](file:///D:/beatbook/docs/product/future-requirements.md)

---

## 12. Security Architecture
* **Android Manifest Hardening:** `allowBackup="false"` prevents extraction of SQLite databases and keys via ADB. `usesCleartextTraffic="false"` blocks unencrypted HTTP.
* **Path Traversal Protection:** All file paths are strictly sanitized to prevent directory traversal (`../`) and block executable extensions.
* **Sensitive Data Redaction:** Passwords, CNICs, GPS coordinate strings, tokens, and license keys are automatically stripped from log outputs.
* **Secure Storage:** Terminal identities and cryptographic tokens are protected via Windows DPAPI and Android KeyStore.

---

## 13. Privacy & Safety Boundaries
CACHELINES strictly enforces constitutional and operational privacy protections:
* **NO Continuous or Background GPS Tracking:** Location sampling is strictly user-initiated for specific places.
* **NO Covert Surveillance:** No background audio recording or silent photo capture.
* **NO Biometric or Facial Recognition:** No face detection or biometric identification algorithms.
* **NO Contact Harvesting:** No reading of phone address books, SMS logs, or call logs.
* **NO Profiling on Protected Characteristics:** Categorization based solely on race, religion, ethnicity, or lawful political beliefs is strictly prohibited.

---

## 14. Current Limitations
To ensure complete technical transparency, the following items are explicitly acknowledged:
1. **Central Sync Backend:** The client-side outbox engine is complete, but deployment of the central cloud API backend remains pending agency infrastructure provisioning.
2. **Production Payment Gateway:** Billing tables, minor-unit money arithmetic, and invoicing are implemented; live credit card processing gateways remain pending merchant API keys.
3. **Database-at-Rest Encryption (SQLCipher):** Standard SQLite v7 is currently active; full database encryption via SQLCipher is prepared but not configured.
4. **Code Signing:** Windows Authenticode and Android production keystores remain pending external agency certificates.

---

## 15. Production Requirements
Before deploying into live agency environments, system administrators must:
1. Provision the central agency synchronization API server.
2. Sign Windows release installers with an organizational Authenticode EV/OV certificate.
3. Sign Android APK/AAB packages with an official agency release keystore.
4. Configure commercial payment gateway API credentials (if in-app payments are utilized).

---

## 16. Future Roadmap
The following approved features are documented in [future-requirements.md](file:///D:/beatbook/docs/product/future-requirements.md) and scheduled for post-v1.0 releases:
* **90-Day Free Evaluation Trial:** With guaranteed operational data preservation upon expiration.
* **Commercial Multi-Year License Durations:** 1-year, 2-year, 3-year, and 5-year options.
* **Google Drive Cloud Backup:** User-authorized encrypted backup vaults with multi-device restore.
* **Beat Handover Protocol:** Encrypted `.clhop` operational package transfer between outgoing and incoming beat officers without transferring identity credentials.

---

## 17. License
**License: Not yet specified**

All rights reserved by CACHELINES. Unauthorized copying, distribution, modification, or reverse engineering of this software, via any medium, is strictly prohibited pending formal license selection by the organization.

---

## 18. Contribution Guidelines
Contributions must adhere to the engineering standards and security boundaries documented in [CONTRIBUTING.md](file:///D:/beatbook/CONTRIBUTING.md). All changes must pass `flutter analyze` with 0 issues and maintain the 275/275 automated test pass baseline.

---

## 19. Security Vulnerability Reporting
If you identify a security vulnerability or privacy boundary flaw, please follow the coordinated disclosure policy in [SECURITY.md](file:///D:/beatbook/SECURITY.md). Do not disclose security vulnerabilities in public issue trackers.

---

## 20. Support & Contact Information
* **Company:** CACHELINES
* **Founder / Product Architect:** Atif Syed
* **Company Portal:** [https://cachelines.github.io/](https://cachelines.github.io/)
* **Founder Website:** [https://iatifsyed.github.io/](https://iatifsyed.github.io/)
* **WhatsApp Support:** `+92 300 4860591`
