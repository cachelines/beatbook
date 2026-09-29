# CACHELINES Field Intelligence & Beat Management System

[![Flutter Version](https://img.shields.io/badge/Flutter-3.47.5-blue.svg)](https://flutter.dev)
[![Dart Version](https://img.shields.io/badge/Dart-3.13.4-teal.svg)](https://dart.dev)
[![Architecture](https://img.shields.io/badge/Architecture-Clean%20Architecture-darkgreen.svg)](#architecture)
[![Offline First](https://img.shields.io/badge/Storage-Offline--First%20SQLite-orange.svg)](#database-foundation)
[![License Status](https://img.shields.io/badge/License-Device--Bound%20Proprietary-red.svg)](#license-system)

---

## 🏛️ Executive & Architectural Overview

The **CACHELINES Field Intelligence & Beat Management System** is a mission-critical, offline-first mobile operations platform engineered for authorized law enforcement personnel, beat officers, and territorial field intelligence units.

The system empowers field staff to digitally record, verify, and manage territorial beat jurisdictions, area directories, persons of interest, vulnerable/sensitive places of worship, configurable intelligence categories, daily patrol visits, and automated standard police report dispatches—even when devices operate entirely devoid of cellular or Wi-Fi connectivity.

- **Developer & Licensor:** [CACHELINES](https://cachelines.github.io/)
- **Founder / Software Architect / Product Engineer:** [Atif Syed](https://iatifsyed.github.io/)
- **Direct Support & Operational Hotline:** [+92 300 4860591](https://wa.me/923004860591)
- **Official Portal:** [https://cachelines.github.io/](https://cachelines.github.io/)

---

## 🎯 Development Milestone 1 — Deliverables Completed

This repository contains the completed **Milestone 1 Production Foundation**, implementing:

1. **Flutter & Dart Architecture**: Fully modular, null-safe, strongly typed Clean Architecture with feature folders.
2. **Tactical Design System**: Bespoke, high-contrast tactical law enforcement UI theme (dark/light) with custom elevation, glassmorphism, and status chips.
3. **Hardware-Bound Licensing Engine**:
   - Hardware UUID detection and cryptographic binding.
   - Offline grace period validation (30-day tolerance).
   - Dedicated License Activation screen with validation and test evaluation support.
4. **Officer Authentication & Session Management**:
   - Clear separation between Device Licensing ("Is this hardware authorized?") and Officer Authentication ("Is this officer authorized?").
   - Salted SHA-256 credential hashing; zero plaintext password storage.
   - Support for 4-digit rapid field PINs, passwords, and biometric hardware hooks.
5. **Operational Dashboard Shell & Navigation**:
   - Live officer header, territorial jurisdiction badge, and offline sync indicator.
   - Interactive cards for all 11 core operational modules.
6. **Core Database Foundation (SQLite)**:
   - Full schema definitions, foreign key constraints, timestamps, and indexes for all 20+ required core entities: `User`, `Client`, `Device`, `License`, `Beat`, `Area`, `Village`, `Locality`, `Person`, `Organization`, `Place`, `Category`, `FieldRecord`, `FieldVisit`, `Photo`, `Report`, `ReportTemplate`, `AuditLog`, `SyncQueue`.
   - Dynamic category seeding with JSON custom schema support (no hard-coded categories).
7. **Security, Privacy & Audit Logging**:
   - Mandatory CNIC masking (`35201-*******-1`) for visual privacy.
   - Log sanitization engine preventing CNIC exposure in crash logs, analytics, or URLs.
   - Tamper-evident operational audit logger recording access and administrative events.
8. **Automated Test Suite**:
   - Comprehensive unit tests for cryptography, CNIC masking, log sanitization, licensing, and categories.
   - Widget tests with clean test doubles for screen rendering.

---

## 🏗️ Clean Architecture Directory Structure

```
beatbook/
├── android/                             # Android native host project & manifest permissions
├── assets/
│   ├── icons/                           # Tactical icons and insignias
│   └── images/                          # Brand assets and graphics
├── lib/
│   ├── main.dart                        # Application bootstrap & dependency injection initialization
│   ├── app.dart                         # Root MaterialApp with tactical themes and named routes
│   ├── core/
│   │   ├── constants/
│   │   │   ├── app_constants.dart       # Product metadata, developer credentials, constants
│   │   │   ├── app_colors.dart          # Tactical government color palette & gradients
│   │   │   └── app_theme.dart           # Dark & light ThemeData with high-contrast components
│   │   ├── database/
│   │   │   ├── database_tables.dart     # DDL table schemas, foreign keys, timestamps & indexes
│   │   │   └── app_database.dart        # SQLite database manager, migrations & seed engine
│   │   ├── di/
│   │   │   └── service_locator.dart     # Service locator / dependency injection registry
│   │   ├── errors/
│   │   │   └── failures.dart            # Domain failure hierarchy (License, Auth, Database, Security)
│   │   └── security/
│   │       ├── encryption_service.dart  # Salted SHA-256 hashing, CNIC masking, log sanitization
│   │       ├── secure_storage_service.dart # Hardware-backed encrypted shared preferences
│   │       └── audit_logger.dart        # Tamper-evident security audit logger
│   └── features/
│       ├── splash/                      # Splash initialization & routing decision engine
│       ├── license/                     # Device-bound licensing domain, data & activation screen
│       ├── auth/                        # Credential verification, session tokens & login screen
│       ├── dashboard/                   # Tactical dashboard shell, metric cards & navigation drawer
│       ├── beat/                        # Jurisdiction limits, police station, boundaries & GPS
│       ├── area_directory/              # Localities, villages, numberdars, and community reps
│       ├── persons/                     # Person records with role-based CNIC privacy masking
│       ├── places/                      # Structured monitoring of mosques, imambargahs, schools, banks
│       ├── categories/                  # Dynamic configurable field observation categories
│       ├── field_visits/                # Patrol rounds, contacts made, and follow-up tracking
│       ├── reports/                     # Daily report auto-builder (Urdu police format & Android share)
│       ├── search/                      # Global offline multi-attribute query engine
│       ├── sync/                        # Offline sync queue inspector & background transmission
│       ├── settings/                    # Biometric toggle, session timeout & secure logout
│       └── about/                       # CACHELINES attribution, architect Atif Syed & license details
└── test/
    ├── unit/
    │   ├── encryption_service_test.dart # Unit tests for hashing and CNIC masking
    │   ├── license_model_test.dart      # Unit tests for licensing & 30-day offline grace period
    │   └── category_model_test.dart     # Unit tests for category model and JSON schema
    └── widget/
        └── app_screens_test.dart        # Widget tests for core activation, login & about screens
```

---

## 🗄️ Database Foundation Schema

The offline SQLite database (`cachelines_beatbook.db`) models all operational entities:

| Table | Entity | Key Attributes | Description |
| :--- | :--- | :--- | :--- |
| `clients` | Client Organization | `id`, `name`, `code`, `province`, `district` | Police division or licensed district entity |
| `users` | User / Officer | `id`, `client_id`, `username`, `password_hash`, `role` | Authorized field staff and beat officers |
| `devices` | Hardware Device | `id`, `client_id`, `hardware_uuid`, `device_model` | Bound Android mobile hardware fingerprint |
| `licenses` | Device License | `id`, `license_key`, `device_id`, `status`, `expiry_date` | Cryptographically validated license with 30-day grace |
| `beats` | Beat Jurisdiction | `beat_number`, `police_station`, `north/south/east/west` | Territorial boundary sector (rural/urban/mixed) |
| `areas` | Beat Area | `id`, `beat_id`, `name`, `area_type` | Sub-division under beat jurisdiction |
| `villages` | Revenue Village | `id`, `area_id`, `hadbast_number`, `numberdar_name` | Rural village records with community elders |
| `localities` | Urban Locality | `id`, `area_id`, `name`, `community_representative` | Urban mohallahs, sectors, and blocks |
| `organizations` | Organization | `id`, `beat_id`, `name`, `org_type`, `head_person_name` | Public offices, institutions, commercial centers |
| `places` | Monitored Place | `id`, `beat_id`, `place_category`, `address`, `responsible_person` | Mosques, Imambargahs, Shrines, Churches, Schools, Banks |
| `persons` | Person of Interest | `id`, `beat_id`, `full_name`, `cnic_encrypted`, `role` | Residents, reps, focal persons with masked CNIC |
| `categories` | Field Category | `id`, `name`, `icon`, `sort_order`, `custom_schema_json` | Extensible field categories (no hard-coding) |
| `field_records` | Field Observation | `id`, `category_id`, `description`, `status`, `gps` | Neutral field observations (reported, verified, closed) |
| `field_visits` | Field Patrol Round | `id`, `beat_id`, `visit_date`, `observations`, `contacts` | Daily beat patrol logs and official meetings |
| `photos` | Evidence Photo | `id`, `owner_entity`, `file_hash`, `latitude`, `longitude` | Tamper-evident single-store reusable media |
| `report_templates`| Report Template | `id`, `header_template`, `record_template`, `footer` | Configurable police dispatch formatting |
| `reports` | Compiled Report | `id`, `report_date`, `generated_content`, `include_gps` | Compiled daily or periodic intelligence report |
| `audit_logs` | Security Audit | `id`, `event_type`, `action`, `user_id`, `timestamp` | Tamper-evident audit trail with no sensitive data |
| `sync_queue` | Sync Queue | `id`, `entity_type`, `operation`, `payload_json`, `status` | Offline change buffer for background server synchronization |

---

## 🔒 Security & Privacy Architecture

- **Zero Plaintext Credentials:** Passwords and field PINs are hashed using salted SHA-256 before storage or comparison.
- **Strict CNIC Protection:** National Identity Card numbers are considered confidential law enforcement data. They are masked by default (`35201-*******-1`), unmasked strictly with privileged role access, and sanitized via regular expressions to prevent accidental leakage into debug logs or crash reporters.
- **Device-Bound Licensing:** The software executes only on devices holding a valid cryptographic binding. An offline grace period of 30 days is supported to maintain continuity during remote operations.
- **Neutral Observation Terminology:** As required by operational standards, field records distinguish between observations and verified facts using objective statuses: `Reported`, `Verified`, `Unverified`, `Inactive`, `Closed`, and `Referred`.

---

## 🚀 Running the Application

### Prerequisites

- Flutter SDK (v3.10+ / tested with v3.47.5)
- Dart SDK (v3.0+ / tested with v3.13.4)
- Android Studio / Android SDK (API 34 recommended) or connected physical Android handset

### Step-by-Step Instructions

1. **Clone or Navigate to the Workspace:**
   ```bash
   cd d:\beatbook
   ```

2. **Install Dependencies:**
   ```bash
   flutter pub get
   ```

3. **Execute the Automated Test Suite:**
   ```bash
   flutter test
   ```

4. **Verify Static Code Analysis:**
   ```bash
   flutter analyze
   ```

5. **Launch the Application:**
   ```bash
   flutter run
   ```

### Default Tactical Credentials for Testing

- **License Activation Screen:**
  - Organization Code: `PUNJAB-POLICE-01`
  - License Key: `CL-BEAT-7860-2026-ACTIVE` (or tap the **ACTIVATE TEST / EVALUATION KEY** button)
- **Officer Login Screen:**
  - Username: `officer`
  - Field PIN: `1234` (or Password: `tactical123`)
  - Or tap the **QUICK LOGIN AS BEAT OFFICER** button

---

## 🗺️ Remaining Development Milestones

- [x] **Milestone 1:** Production Foundation, Clean Architecture, SQLite Schema, Splash, License Activation, Login, Dashboard Shell, Tests, and Documentation.
- [ ] **Milestone 2:** CRUD Record Management for Persons, Important Places, and Configurable Categories with Camera & Photo Hash Capture.
- [ ] **Milestone 3:** Daily Report Builder dynamic templating, Urdu report formatter, and native Android Share Sheet dispatch.
- [ ] **Milestone 4:** Background Synchronization Queue Engine, Conflict Resolution, and REST API integration with central server.
- [ ] **Milestone 5:** Biometric Hardware Authentication, Advanced Audit Log Analytics, and Supervisor Review Workflows.
- [ ] **Milestone 6:** Windows Desktop Administration Console (Client management, device licensing, license revocation, and reporting).

---

## 📞 Product Inquiries & Licensing

For operational deployments, custom beat schemas, and enterprise licensing for police departments and security agencies, contact:

**CACHELINES**  
Software Architect & Lead Engineer: **Atif Syed**  
WhatsApp: **+92 300 4860591**  
Company: [https://cachelines.github.io/](https://cachelines.github.io/)  
Portfolio: [https://iatifsyed.github.io/](https://iatifsyed.github.io/)
