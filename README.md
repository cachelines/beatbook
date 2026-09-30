# CACHELINES Field Intelligence & Beat Management System

[![Flutter Version](https://img.shields.io/badge/Flutter-3.47.5-blue.svg)](https://flutter.dev)
[![Dart Version](https://img.shields.io/badge/Dart-3.13.4-teal.svg)](https://dart.dev)
[![Architecture](https://img.shields.io/badge/Architecture-Clean%20Architecture-darkgreen.svg)](#2-architecture--design-principles)
[![Storage](https://img.shields.io/badge/Storage-Offline--First%20SQLite%20v7-orange.svg)](#8-database-architecture--migration-strategy)
[![License Status](https://img.shields.io/badge/License-Multi--Device%20Proprietary-red.svg)](#7-license-system--offline-policy)
[![Milestone](https://img.shields.io/badge/Milestone-Milestone%209%20Complete-brightgreen.svg)](#10-official-approved-milestone-roadmap)

---

## 1. Product Overview

The **CACHELINES Field Intelligence & Beat Management System** is a mission-critical, offline-first mobile operations platform engineered for authorized field staff to digitally manage their assigned beat/area information, territorial boundaries, area directories, verification records, administrative contacts, people & places directory, and protected evidence.

- **Product Name:** CACHELINES Field Intelligence & Beat Management System
- **Company:** CACHELINES ([https://cachelines.github.io/](https://cachelines.github.io/))
- **Founder / Software Architect / Product Engineer:** Atif Syed ([https://iatifsyed.github.io/](https://iatifsyed.github.io/))
- **WhatsApp Support:** [+92 300 4860591](https://wa.me/923004860591)
- **Current Status:** Milestone 9 (Security Hardening + Testing + Production Build) Complete / Frozen. 267/267 Automated Tests Passing. 0 Analyzer Issues. Database Schema v7. Native Windows Release Binary Verified.

---

## 2. Architecture & Design Principles

The application strictly enforces **Clean Architecture** with a feature-first structure:

```
Presentation (UI & Controllers) ➔ Domain (Repositories & Models) ➔ Data (DataSources & Persistence)
```

- **Separation of Concerns:** UI widgets never directly interact with SQLite, hardware secure storage, or HTTP networks.
- **Dependency Injection:** Repositories, database services, and secure storage instances are registered centrally via `DependencyInjection` (`lib/app/dependency_injection.dart`).
- **Territorial Hierarchy:** Structured territorial and administrative relationships without duplicating parent information.
- **Offline First:** All territorial operations (viewing beat book, searching directory, adding/updating areas, auditing verifications) run locally against SQLite with full offline resilience.

---

## 3. Territorial Hierarchy

Milestone 2 establishes the geographic and administrative foundation of the digital Beat Book:

```
District
  └── Tehsil
        └── Police Station
              └── Beat (Urban / Rural / Mixed)
                    ├── Area / Village / Locality / Mohallah / Ward
                    ├── Local Representatives (Numberdars & Community Elders)
                    ├── Government Officers (Administrative & Gazetted Contacts)
                    └── Verification History (Immutable Audit Trail)
```

1. **A Beat** belongs to exactly one Police Station, one Tehsil, and one District.
2. **A Police Station** belongs to one Tehsil and District.
3. **A Tehsil** belongs to one District.
4. **An Area / Village / Locality** belongs to an assigned Beat.
5. **Parent-Child Area Nesting:** Urban beats can model hierarchical locality trees (e.g. Urban Locality ➔ Mohallah 1, Mohallah 2). Circular hierarchies are strictly detected and prevented in the domain and data layer.

---

## 4. Milestone 3 — People & Places Directory + Protected Evidence Foundation

Milestone 3 extends the Beat Book from its territorial foundation into a controlled **People & Places Directory** coupled with an isolated **Protected Evidence Foundation**.

### Core Domain Capabilities:
1. **People Directory (`PublicRolePerson`):** Controlled directory for legitimate administrative, public-role, or community representatives. Recording is strictly limited to public institutional classifications:
   - Elected / Public Officials (e.g. Chairmen, Councilors)
   - Government Officials (Administrative contacts)
   - Institution Administrators (Heads of facilities)
   - Community Representatives (Elders, trade leaders)
   - Religious Institution Contacts (Imam/Khateeb, Sajjadah Nasheen, Pastor, Priest as institutional administrators)
   - Social / Civic Organization Office-Bearers
   - Prominent Public Professionals
2. **Places Directory (`Place`):** Institutional facility and public landmark directory:
   - Places of Worship: Mosques, Imambargahs, Shrines/Darbars, Churches, and other worship facilities recorded as public institutional entities.
   - Educational Institutions: Schools, Colleges, Universities, and Madrassas recorded with public/private classification, approximate aggregate enrollment counts, and hostel availability.
   - Other Important Places: Hospitals, Government Offices, Courts, Banks, Public Markets, Transport Facilities, Industrial Units, and Community Centers.
3. **Institutional Relationships (`PersonPlaceRoleModel`):** A strictly controlled relationship mapping persons to places via formal institutional roles (Administrator, Principal, Manager, Pastor, Priest, Imam/Khateeb, Managing Committee Member, Sajjadah Nasheen, Official Contact). Arbitrary person-to-person relationship mapping is prohibited.
4. **Deliberate Point-Location GPS Capture:** Optional point geographic coordinates recorded exclusively via explicit user initiation. Pre-capture explanation dialog is shown (`LocationPrivacyDialog`).
5. **Protected Evidence Records:** User-initiated capture of official photographs and documents. Saved to isolated private application storage with pre-capture privacy notice (`MediaPrivacyDialog`) and cryptographic SHA-256 integrity hash verification. Controlled retention states (`active`, `archived`, `pendingDeletion`).
6. **Immutable Verification History:** All people and place records maintain permanent historical audit logs of verification status (`unverified`, `verified`, `requiresUpdate`), verified timestamp, verifier officer ID, and field remarks.

---

## 5. Privacy Safeguards & Explicit Negative Guarantees

CACHELINES enforces strict privacy principles and purpose limitations across all operations:

> ### MANDATORY NEGATIVE GUARANTEES:
> 
> 1. **"Milestone 3 does not implement continuous/background GPS tracking."**  
>    Location collection is strictly manual, user-initiated, visible, and tied to a specific institutional record. The application requests no background location permissions and runs no passive location collection services. Denying location permission does not prevent saving or viewing place records.
> 
> 2. **"Milestone 3 does not store plaintext CNIC values."**  
>    No raw 13-digit CNIC numbers are stored in SQLite tables, logs, search indexes, URLs, or UI views. Identity references utilize protected token identifiers (`identity_reference`).
> 
> 3. **"Religious/community records are institutional directory records and are not used to infer or profile individual religious affiliation."**  
>    Places of worship and religious functionaries are cataloged strictly as public landmarks and administrative contacts. The system does not create congregation rosters, list attendees, or perform religious, sectarian, ethnic, or political profiling.

---

## 6. Milestone 3 Entities & Data Models

All models reside in `lib/shared/models/`:

| Entity | Primary Fields | Key Relationships & Behaviors |
| :--- | :--- | :--- |
| **PersonModel** | `id`, `beatId`, `areaId`, `name`, `publicRole`, `organizationName`, `organizationRole`, `category`, `officialContact`, `publicAddress`, `identityReference`, `status`, `verificationStatus`, timestamps | Public-role classifications only; tokenized `identityReference` (no plaintext CNIC) |
| **PlaceModel** | `id`, `beatId`, `areaId`, `name`, `placeType`, `address`, `description`, `responsiblePersonId`, `officialContact`, `publicOrPrivate`, `approxEnrollment`, `hasHostel`, `status`, `verificationStatus`, timestamps | Generic institutional place model across 17 classifications; educational specs; no student/congregation lists |
| **PersonPlaceRoleModel** | `id`, `placeId`, `personId`, `roleType`, `title`, `isPrimaryContact`, `status`, timestamps | Institutional role links between people and facilities; avoids arbitrary social-graph mapping |
| **PlaceLocationModel** | `id`, `placeId`, `latitude`, `longitude`, `accuracyMeters`, `capturedAt`, `capturedBy`, `captureMethod`, `notes`, `status`, timestamps | Single point-in-time coordinates; manual capture only |
| **MediaEvidenceModel** | `id`, `entityType`, `entityId`, `localPath`, `fileHash`, `mediaType`, `description`, `capturedAt`, `capturedBy`, `verificationStatus`, `status`, timestamps | Private isolated local storage; SHA-256 integrity digest; retention states |
| **VerificationHistoryModel** | `id`, `entityType`, `entityId`, `verifiedBy`, `verifiedAt`, `previousStatus`, `newStatus`, `remarks` | Immutable operational sign-off audit trail across all territorial and directory entities |

---

## 7. Authorization & Role-Based Access Control

Centralized in `PeoplePlacesAuthorizationService` (`lib/features/people_places/domain/services/people_places_authorization_service.dart`):

| Action | Field User | Supervisor | Administrator |
| :--- | :---: | :---: | :---: |
| **View Beat People & Places** | Assigned Beat Only (`user.beatId == beatId`) | Allowed (Jurisdiction-wide) | Allowed (All) |
| **Create / Edit People & Places** | Assigned Beat Only | Allowed | Allowed |
| **Capture Point GPS Coordinates** | Assigned Beat Only (Explicit User Action) | Allowed | Allowed |
| **Attach Protected Media** | Assigned Beat Only (Explicit User Action) | Allowed | Allowed |
| **Submit Verification** | Allowed (Preliminary Field Sign-off) | Allowed (Audit Verification) | Allowed (Audit Verification) |
| **Archive Directory Records** | Prohibited | Allowed | Allowed |

---

## 8. Database Architecture & Migration Strategy

The SQLite database version has progressed from **Version 2** to **Version 3**.

### Schema Version 3 Tables (19 Tables Total):
1. `clients` (Milestone 1)
2. `users` (Milestone 1)
3. `devices` (Milestone 1)
4. `licenses` (Milestone 1)
5. `audit_logs` (Milestone 1 — Operational append-only audit logging with sanitized metadata)
6. `districts` (Milestone 2)
7. `tehsils` (Milestone 2)
8. `police_stations` (Milestone 2)
9. `beats` (Milestone 2)
10. `areas` (Milestone 2)
11. `villages` (Milestone 2)
12. `local_representatives` (Milestone 2)
13. `government_officers` (Milestone 2)
14. `verification_history` (Milestone 2)
15. `public_role_persons` (Milestone 3 — Controlled public-role person directory)
16. `places` (Milestone 3 — Institutional places and public landmarks)
17. `person_place_roles` (Milestone 3 — Institutional contact role mapping)
18. `place_locations` (Milestone 3 — Explicit point GPS coordinates)
19. `media_evidence` (Milestone 3 — Protected evidence metadata with SHA-256)

### Migration Safety:
- Migrations are managed inside `DatabaseService._onUpgrade(db, oldVersion, newVersion)`.
- When upgrading from Version 2 to Version 3, all Milestone 1 and Milestone 2 records remain 100% intact.
- Verified by automated regression tests.

---

## 9. Screens Implemented

### Milestone 1 & 2 Screens:
1. **Splash Screen (`/splash`)**
2. **License Activation Screen (`/license`)**
3. **License Status Screen (`/license-status`)**
4. **Login Screen (`/login`)**
5. **Dashboard Shell (`/dashboard`)**
6. **Settings Screen (`/settings`)**
7. **About & License Screen (`/about`)**
8. **My Beat (`/my-beat`)**
9. **Beat Book Screen (`/beat-book`)**
10. **Beat Edit Screen (`/beat-edit`)**
11. **Area Directory Screen (`/area-directory`)**
12. **Area Edit Screen (`/area-edit`)**
13. **Representative Edit Screen (`/representative-edit`)**
14. **Officer Edit Screen (`/officer-edit`)**
15. **Territorial Search Screen (`/territorial-search`)**

### Milestone 3 Screens:
16. **People Directory Screen (`/people-directory`):** Public-role directory with filters (Beat, Role Category, Verification status), search, and privacy warning banner.
17. **Person Edit Screen (`/person-edit`):** Structured form for public-role figures with administrative beat/area jurisdiction, official contact, tokenized identity reference, and audit sign-off.
18. **Places Directory Screen (`/places-directory`):** Institutional facilities and public landmarks with filters (Beat, Place Type, Verification status), instant offline search, and quick classification badges.
19. **Place Details Screen (`/place-details`):** Deep inspection interface displaying:
    - Basic institutional details and educational specifications
    - Explicit GPS Point Location capture with `LocationPrivacyDialog`
    - Institutional Contact Role assignment and management (`PersonPlaceRoleModel`)
    - Protected Evidence Gallery with `MediaPrivacyDialog` and SHA-256 digest
    - Permanent Verification Audit History
20. **Place Edit Screen (`/place-edit`):** Create and update institutional facilities across 17 classifications with specialized fields for educational institutions.

---

## 10. Official Approved Milestone Roadmap

* [x] **Milestone 1:** Foundation + Login + License *(Complete)*
* [x] **Milestone 2:** Beat Book + Area Directory *(Complete)*
* [x] **Milestone 3:** People + Places + Photos + GPS *(Complete)*
* [x] **Milestone 4:** Configurable Categories + Field Records *(Complete)*
* [x] **Milestone 5:** Daily Report + WhatsApp Sharing *(Complete / Frozen)*
* [x] **Milestone 6:** Offline Sync + Backup *(Complete / Frozen)*
* [x] **Milestone 7:** Windows License Manager *(Complete / Frozen)*
* [x] **Milestone 8:** Payments + Invoices + Dashboard *(Complete / Frozen)*
* [x] **Milestone 9:** Security Hardening + Testing + Production Build *(Complete / Frozen)*

---

## 11. Milestone 4 — Configurable Categories & Field Records

### Architecture Overview
Milestone 4 introduces dynamic operational categorization and structured field reporting while maintaining absolute territorial hierarchy, normalized SQLite storage, strict role-based authorization, and privacy boundaries.

* **Database Version:** `4`
* **New Normalized Tables:**
  1. `field_categories`: Configurable administrative/operational classifications (`id`, `name`, `code`, `description`, `status`, `sort_order`, `is_system`, `icon_name`, `created_at`, `updated_at`, `created_by`, `updated_by`).
  2. `field_definitions`: Typed attributes for each category (`id`, `category_id`, `field_key`, `label`, `description`, `data_type`, `is_required`, `sort_order`, `options`, `min_value`, `max_value`, `status`, `created_at`, `updated_at`).
  3. `field_records`: Operational logs bound strictly to a territorial Beat (`id`, `category_id`, `beat_id`, `area_id`, `person_id`, `place_id`, `title`, `description`, `record_date`, `follow_up_status`, `verification_status`, `status`, `created_by`, `created_at`, `updated_at`, `last_verified_at`, `last_verified_by`).
  4. `field_record_values`: Normalized EAV storage for dynamic fields (`id`, `record_id`, `field_definition_id`, `value_text`, `created_at`, `updated_at`).

### Strict Territorial Integrity
Field records mandate an assigned Beat. If an Area, Person, or Place is associated with a record, the repository layer strictly verifies that the entity belongs to that same Beat. Inconsistent cross-boundary references (e.g. record in Beat A with Place from Beat B) are strictly rejected with a `DatabaseFailure`.

### Supported Safe Data Types
* `text` (single-line text)
* `multilineText` (multi-line observations/narratives)
* `integer` (validated numeric integer)
* `decimal` (validated floating-point number)
* `boolean` (true/false toggle)
* `date` (ISO-8601 calendar date)
* `dateTime` (ISO-8601 calendar date and timestamp)
* `singleChoice` (dropdown selection restricted to configured options)
* `multiChoice` (multiple selection from configured options)
* `personReference` (controlled foreign reference to Person directory)
* `placeReference` (controlled foreign reference to Places directory)

*No executable code or arbitrary SQL is permitted.*

### Role-Based Authorization
* **Field User:** Create, view, and edit field records within assigned Beat; submit records for verification; use active categories.
* **Supervisor:** View and edit records within territorial jurisdiction; conduct formal operational verification and reject invalid entries; archive records.
* **Administrator:** Full system-level category configuration; define and modify dynamic fields; reorder categories; manage status.

### Explicit Privacy Boundaries (Phase 21)
Milestone 4 strictly forbids and contains no mechanism for:
* Religious profiling, sectarian profiling, ethnic profiling, or political profiling
* Background/continuous GPS surveillance
* Automated predictive risk scoring of citizens
* Arbitrary person-to-person social network mapping
* Call/SMS/contact harvesting or social media scraping

All records reflect legitimate operational, public-service, and infrastructure observations.

---

---

## 12. Milestone 5 — Daily Report & WhatsApp Sharing (Phase 22)

Milestone 5 implements a practical, operational **Daily Report** module and explicit user-initiated **WhatsApp Sharing** built purely as a reporting/presentation layer over existing data models (Field Records, Categories, Beat, Area, People, Places, Follow-ups, and Verifications).

### Key Architectural Decisions
* **Zero Database Bloat / No Redundant Tables:** The report is dynamically generated on-demand from existing records (`field_records`, `field_record_values`, `field_categories`, `field_definitions`, etc.). Schema remains frozen at **v4**.
* **Session-Based Report Lifecycle (No False Archival Claims):** The `draft`/`finalized` status represents an active in-memory/session preview display state. Finalizing does not write to a dedicated report table or mutate source records, avoiding false claims of permanent report archival when reports are synthesized dynamically.
* **Operational Date Filtering:** The report rigorously filters on the operational `recordDate` (`YYYY-MM-DD`), rather than internal database insertion timestamps (`createdAt`).
* **Territorial & Role Authorization:** 
  - **Field User:** Restricted strictly to their assigned Beat and Areas within that Beat. Attempting to generate, view, or share cross-beat reports is rejected at both UI and repository levels.
  - **Supervisor:** Can select and generate reports for any Beat within their administrative jurisdiction.
  - **Administrator:** Can generate system-wide reports across all Beats and Areas.
* **Preservation of Historical Data:** Records referencing categories or field definitions that were later deactivated remain fully readable with their historical names and dynamic values.
* **Sanitized Human-Readable Output:** Report text formats operational activity clearly while strictly excluding:
  - Database primary keys / UUIDs
  - Passwords, session tokens, and license keys
  - Raw GPS coordinates and private local storage file paths
* **Explicit User-Initiated Sharing & Accurate Semantics:**
  - Sharing is NEVER automated, scheduled, or run in the background.
  - Requires deliberate user tap on "Share via WhatsApp" or "System Share Sheet", followed by an explicit confirmation dialog.
  - The application initiates opening the target share mechanism only; it **does NOT track or claim message delivery or recipient receipt**.
  - Fallback chain: `whatsapp://send?text=...` deep-link → WhatsApp Web browser URL → native OS share sheet (`share_plus`) → device clipboard copy (with user instruction to paste manually).
  - Operates 100% offline; local report generation requires no network or cloud connectivity.
* **Operational Audit Logging:**
  - `REPORT_GENERATED` is emitted strictly on deliberate report generation operations (and not during harmless widget rebuilds or text formatting).
  - `REPORT_SHARED` logs the specific initiation mechanism (`whatsapp_deep_link`, `whatsapp_web_fallback`, `native_share`, or `clipboard`), never falsely logging clipboard copies as sent messages or claiming delivery confirmation.

---

## 13. Milestone 6: Offline Sync & Backup Architecture

Milestone 6 introduces a fully robust offline synchronization engine and cryptographic backup system:

### 1. Offline-First Autonomous Operation
* The application is fully operable with zero network connectivity. Users can create, update, and archive operational records, generate Daily Reports, and enqueue local mutations offline.
* Changes survive application restart, process termination, and prolonged network outages.

### 2. Durable Local Outbox (`sync_outbox`)
* Every synchronizable mutation generates a durable outbox operation with a stable UUID operation ID (`id`).
* Tracks `entity_type`, `entity_id`, `operation_type` (`create`, `update`, `archive`, `delete`), `payload_json`, `client_mutation_timestamp`, `base_version`, `status` (`pending`, `syncing`, `succeeded`, `failed`, `conflict`), `retry_count`, and `last_error`.
* Bounded retries: transient network failures increment retry count without infinite loops.

### 3. Entity Versioning & Optimistic Concurrency (`entity_sync_metadata`)
* Maintains universal entity versioning in `entity_sync_metadata` tracking: `entity_type`, `entity_id`, `version`, `last_synced_at`, `last_synced_version`, `sync_status`.
* Mutations record their `base_version` (the version the edit was based upon).
* Prevents silent overwrites when remote and local states diverge.

### 4. Conflict Detection & Explicit Resolution (`sync_conflicts`)
* Conflicts are detected automatically during push (stale `base_version`) or pull (divergent remote version when local edit is pending).
* When a conflict occurs:
  - Both local and remote states are preserved in `sync_conflicts`.
  - The local database is **never silently overwritten**.
  - Exposes conflict to the user in the Conflict Resolution UI.
* Supported Resolution Strategies:
  - **Keep Local:** Re-bases outbox mutation onto remote version and re-queues it as authoritative.
  - **Keep Remote:** Applies remote payload to local SQLite tables, updates entity sync metadata to remote version, and marks conflict resolved.
  - **Merge:** Allows editing/combining fields, updates local SQLite tables, increments revision, and re-queues mutation for sync.
* Conflict resolution clears the conflict state and emits an operational audit event (`SYNC_CONFLICT`).

### 5. Territorial Authorization Boundaries
* Enforced during synchronization and conflict resolution by `SyncAuthorizationService`:
  - **Field User:** Restricted strictly to their assigned Beat (`user.beatId`). Unauthorized records or conflicts outside assigned Beat are rejected.
  - **Supervisor:** Authorized across their jurisdiction (District / Police Station).
  - **Administrator:** Full system-wide synchronization authority.

### 6. Idempotency
* Synchronization requests transmit stable operation IDs (`id`).
* Retrying the exact same mutation after lost network responses acknowledges idempotently without creating duplicate database rows.

### 7. Versioned Backup Export & Cryptographic Integrity (Format v1, Schema v5)
* Exports operational database tables wrapped in a documented, versioned JSON backup package (`BackupPackage`):
  - `format_version: 1`
  - `schema_version: 5`
  - `created_at`: ISO-8601 timestamp
  - `sha256_checksum`: Cryptographic SHA-256 hash computed across the canonical serialized tables dataset.
  - `tables`: Map of all operational tables and records.
* **Strict Security Exclusion:** Sensitive secrets (user password hashes, session tokens, license private keys, API credentials) are **never included** in backups.
* Tamper Detection: Modified, truncated, corrupted, or incompatible backups are detected and rejected via SHA-256 verification.

### 8. Safe Restore & Atomic Rollback
* Restore requires explicit user confirmation via dialog.
* **Integrity Validation First:** Re-verifies SHA-256 checksum and schema version (<= 5). If validation fails, restore is aborted.
* **Safety Snapshot:** Creates an automatic safety snapshot of current data before modifying live tables.
* **Atomic Transaction:** Executes within a single SQLite transaction. If any error occurs, transaction rolls back and **the active database remains completely unchanged**.

### 9. Media Synchronization Boundaries
* Synchronizes `media_evidence` metadata and content SHA-256 hashes.
* Private device filesystem paths are never used as remote identifiers.
* Media synchronization failures do not corrupt associated entity metadata.

### 10. Operational Audit Logging
* Emits sanitized operational events:
  - `SYNC_STARTED`, `SYNC_COMPLETED`, `SYNC_FAILED`, `SYNC_CONFLICT`
  - `BACKUP_CREATED`, `BACKUP_RESTORED`, `BACKUP_RESTORE_FAILED`
* Audit logs record operation IDs, entity types, record counts, and status while excluding passwords, tokens, license keys, and full record payloads.

### 11. Remote Backend Integration Status
* Per Sections 2 and 32 of Milestone 6 specification:
  - **Remote synchronization backend integration remains pending.**
  - Clean `ISyncRemoteAdapter` interface and default `SyncRemoteAdapterImpl` are provided.
  - Tested deterministically using `DeterministicFakeRemoteAdapter`.
  - The application does not pretend remote cloud synchronization has occurred.

---

## 14. Milestone 6 Architecture Addendum: Future Windows + Android Multi-Client Readiness

This architectural correction addendum ensures that the completed Milestone 6 offline synchronization engine is safely extensible to multiple clients and a central backend without coupling sync logic to Windows desktop assumptions.

### 1. Current State vs. Future Intended Architecture

```text
Current State:
Windows client + local offline SQLite v5 + sync abstraction (ISyncRemoteAdapter)

Future Intended Architecture:
                ┌─────────────────────────┐
                │   Central Backend/API   │
                │                         │
                │ Users                   │
                │ Devices                 │
                │ Licenses                │
                │ Sync API                │
                │ Authorization           │
                │ Audit                   │
                └────────────┬────────────┘
                             │
               ┌─────────────┴─────────────┐
               │                           │
      ┌────────▼────────┐         ┌────────▼────────┐
      │ Windows Client  │         │ Android Client  │
      │ CACHELINES      │         │ Future Admin    │
      │                  │         │ App             │
      │ Offline SQLite  │         │ Future local DB │
      │ Sync Engine     │         │ Sync/API client │
      └──────────────────┘         └─────────────────┘
```

### 2. Client Identity Separation

Identity boundaries are strictly differentiated across five distinct orthogonal concepts:

| Identity Concept | Description | Implementation | Derivation Source |
| :--- | :--- | :--- | :--- |
| **User Identity** | Authenticated human operator (`UserModel.id`) | Role, beat assignment, and district | Auth credentials |
| **Client Type** | Application platform tier (`clientType`) | `ClientType.windows`, `ClientType.android`, etc. | Platform runtime / deployment |
| **Installation Identity** | Local installation lifecycle identifier (`installationId`) | UUID stored in hardware-backed secure storage | Random UUID (no hardware fingerprinting) |
| **Session Identity** | Ephemeral token or session key (`sessionId`) | Optional sync session identifier | Remote auth token |
| **Operation Identity** | Idempotent mutation UUID (`operationId`) | Stable unique ID per outbox mutation | Crypto UUID v4 per mutation |

**Strict Isolation Guarantees:**
- Installation ID is **NEVER** derived from username, password, license key, or GPS coordinates.
- Installation ID is **NEVER** used as a secret cryptographic key; it is safe to transmit with authenticated sync requests.
- No hardware fingerprinting or invasive device tracking is employed.
- Installation ID is resettable when the app is explicitly reset or uninstalled.

### 3. Remote Adapter Contract & Multi-Client Metadata

The remote sync boundary (`ISyncRemoteAdapter`) is platform-neutral and supports:
- `pushMutations(operations, {SyncClientContext? clientContext})`
- `pullChanges({sinceTimestamp, authorizedBeatIds, SyncClientContext? clientContext})`
- `checkRemoteRevision({entityType, entityId})`
- Structured acknowledgements via `RemoteOperationAck` (`accepted`, `rejected`, `conflict`, `idempotent_duplicate`).

**Conceptual Future Request Metadata:**
```json
{
  "clientContext": {
    "userId": "usr-field-01",
    "clientType": "windows",
    "installationId": "550e8400-e29b-41d4-a716-446655440000",
    "sessionId": "sess-live-token"
  },
  "operations": [
    {
      "operationId": "op-uuid-1",
      "entityType": "place",
      "entityId": "place-uuid-2",
      "operationType": "update",
      "baseVersion": 3,
      "payload": { ... },
      "clientMutationTimestamp": "2026-09-30T03:00:00Z"
    }
  ]
}
```

### 4. Synchronization Authorization & Client-Neutral Security

- **Client type does NOT determine authorization.** An Android client or Windows client is subject to identical role-based rules:
  - **Field User:** Restricted strictly to assigned Beat (`user.beatId`).
  - **Supervisor:** Restricted to assigned District / jurisdiction (`user.district`).
  - **Administrator:** Full system-wide authorization.
- There are **no special privilege bypasses** for Android or Windows.

### 5. Media & Backup Portability

- Media evidence records (`media_evidence`) utilize stable content hashes (SHA-256) and record IDs.
- Windows-specific absolute filesystem paths (`C:\...`, `D:\...`) are sanitized during backup export, producing portable relative paths (`media/filename.ext`) that restore cleanly across Windows, Android, and Linux environments.

### 6. Implementation Status & Explicit Scope Boundaries

| Capability | Status | Notes |
| :--- | :--- | :--- |
| **Multi-Client Sync Contract** | **Implemented** | `SyncClientContext`, `RemoteOperationAck`, `ClientIdentityProvider` |
| **Portable Backup Sanitization** | **Implemented** | Eliminates Windows-specific paths from `.clbak` packages |
| **Windows License Manager** | **Implemented** | Administrative client interface, multi-device capacity, user management |
| **Production Remote Backend** | **PENDING** | Interface abstractions ready (`ILicenseRemoteAdapter`); honest pending status |
| **Android Application / UI** | **NOT IMPLEMENTED** | Platform-neutral domain contracts; Android UI reserved for future milestone |
| **Payments & Invoicing** | **Implemented** | Milestone 8 billing domain, invoices, payment ledger, admin dashboard (gateway pending) |
| **Security Hardening Pass** | **Implemented** | Hardened domain authorization, path traversal rejection, log sanitization, release build verification |

---

## 15. Milestone 7: Windows License Manager + Users + Devices + License Activation

Milestone 7 establishes a dedicated, Windows-oriented license, user, and device management capability architecturally compatible with a future centralized backend and a future Android administrative client.

### 1. Target Topology & Platform Neutrality

```text
                    CENTRAL BACKEND / API (Pending)
                                 │
             ┌───────────────────┼───────────────────┐
             │                   │                   │
           Users              Licenses            Devices
             │                   │                   │
             └───────────────────┼───────────────────┘
                                 │
                     ┌───────────┴───────────┐
                     │                       │
              Windows Client           Future Android
              License Manager          Admin Client
```

- **Clean Administrative Separation:** Field Users cannot create, suspend, or revoke licenses, nor activate arbitrary devices.
- **Client Neutrality:** Windows does not gain authorization privileges solely from the platform (`clientType == 'windows'`). The future Android admin client will use identical domain authorization rules.

### 2. Consolidated License Domain Model & Explicit Lifecycle

- **Consolidated Model:** Extended existing Milestone 1 `LicenseModel` without creating competing entities.
- **Authoritative Fields:** `id`, `licenseKey`, `status`, `plan`, `startsAt`, `expiryDate`, `graceUntil`, `gracePeriodDays` (30 days), `maxDevices`, `currentDeviceCount`, `assignedUserId`, `createdAt`, `updatedAt`.
- **Explicit Lifecycle States:**
  - `pending`: Awaiting authoritative activation.
  - `active`: Fully operational.
  - `expiring`: Within configured 30-day grace period; operational with warning.
  - `expired`: Grace period ended; operational access blocked.
  - `suspended`: Administratively paused.
  - `revoked`: Permanently terminated.

### 3. Centralized License Validation Service

- `LicenseValidationService` evaluates effective entitlement authority:
  - Determines operational status (`isValid`, `isOperational`, `isWithinGrace`).
  - Enforces active device capacity (`currentActiveDevices < license.maxDevices`).
  - Validates user account active status.
  - Rejects blocked or deactivated devices.

### 4. Device Registration & Multi-Device Capacity Enforcement

- **Identity Standard:** Powered by `ClientIdentityProvider` and stable installation UUIDs. **Strictly prohibits hardware fingerprinting (no CPU, BIOS, MAC, or biometric identifiers).**
- **Explicit Capacity Limits:** If a license has `maxDevices: 2` and 2 active devices exist, a 3rd activation is rejected with `DeviceLimitExceededFailure`.
- **No Silent Deactivations:** Administrators must explicitly deactivate an old device before activating a replacement.
- **Device Blocking:** Compromised or stolen devices can be blocked by administrators. Blocked devices cannot activate any license.

### 5. Role-Based Administrative Authorization

- **Administrator:** Full authority to create, suspend, reactivate, or revoke licenses, assign users, deactivate/block devices, and inspect audit logs.
- **Supervisor:** Strictly prohibited from license administration unless explicitly granted system administrator role.
- **Field User:** Strictly prohibited from all license, device, and user management functions.

### 6. Windows License Manager Desktop UI

A dedicated 5-tab administrative console:
1. **Overview Tab:** System KPIs (Active Licenses, Device Capacity, Registered Users), Offline Policy banner, and **honest "Pending Server" notice**.
2. **Licenses Tab:** Lists licenses with masked keys (`CL-ENT-****-1234`), plan badges, capacity ratios (`1 / 3 devices`), and lifecycle actions (Suspend, Revoke, Reactivate).
3. **Devices Tab:** Lists installations with sanitized IDs (`INST-****-001`), client types (`WINDOWS`), statuses, and controls (Deactivate, Block, Unblock).
4. **Users Tab:** Lists users, roles, statuses (Active/Inactive), and license assignments.
5. **Audit Tab:** Displays operational lifecycle audit events (`LICENSE_CREATED`, `DEVICE_ACTIVATED`, `DEVICE_BLOCKED`).

### 7. Database Schema v6 & Migration Strategy

- **Schema Version:** Bumped to `6` in `AppConfig`.
- **New Association Table:** `license_devices` tracking M:N device activations with foreign keys, compound indexes, and unique constraints `(license_id, device_id)`.
- **Column Extensions:**
  - `licenses`: `max_devices`, `starts_at`, `grace_until`, `plan`, `server_signature`, `created_by`, `updated_by`.
  - `devices`: `client_type`, `display_name`, `deactivated_at`, `created_at`, `updated_at`.
- **Preservation & Seeding:** On upgrade from v5, existing Milestone 1–6 records are preserved 100%, and existing device-license links are automatically seeded into `license_devices`.

### 8. Sanitized Operational Audit Logging & Secrets Security

- **Strict Masking:** Full license keys and installation secrets are never written to plaintext audit logs or backup archives.
- **Masked Formats:** Keys appear as `CL-ENT-****-1234`; devices appear as `INST-****-001`.
- **Zero Secret Leaks:** Password hashes, session tokens, and private signatures are strictly excluded from logs and backups.

### 9. Remote API Boundary, Authority Model & Server Signature Review

- **Authority Model:**
  - **Locally Persisted State:** When running without a production backend, local SQLite stores administrative records and cached entitlements. It is **not** represented as an independent authoritative licensing server.
  - **Server Authorization:** Only genuine remote authority responses can establish server-verified state.
  - **Case D Remote Revocation Override:** When the remote adapter reports a revoked or suspended license, remote authoritative state immediately overrides local active state in both SQLite and secure storage.
- **Server Signature Review:**
  - `server_signature` is strictly **reserved/future metadata** in Schema v6.
  - Because no centralized public-key infrastructure or signature verification service currently exists, `server_signature` is never treated as cryptographic proof of server authorization.
- **Boundary Interface:** `ILicenseRemoteAdapter` defines future network contracts (`validateLicenseOnline`, `activateDeviceOnline`, `deactivateDeviceOnline`, `fetchLicenseOnline`).
- **Deterministic Fake:** `DeterministicFakeLicenseRemoteAdapter` provided for automated testing only.
- **Honest Status:** `LicenseRemoteAdapterImpl` clearly reports:
  ```text
  Production backend integration: PENDING
  ```
  The application never falsely claims cloud validation or online server confirmation.

---

## 16. Implemented vs. Non-Implemented Scope

### Implemented (Milestones 1–9)
* Foundation, hardware identity, licensing, and session auth
* Territorial hierarchy: District → Tehsil → Police Station → Beat → Area
* Directory of local community representatives and government officers
* People & Places directories with institutional roles
* Point-in-time GPS capture & user-initiated photo/evidence foundation
* Configurable field categories & dynamic typed field definitions
* Structured field records with normalized values
* Territorial consistency enforcement
* Daily Field Report module (date, beat, area filtering; summary, follow-up, and verification breakdowns)
* Explicit user-initiated WhatsApp and native OS sharing with operational audit logging
* Offline-first synchronization engine & durable outbox queue (`sync_outbox`)
* Entity versioning & optimistic concurrency (`entity_sync_metadata`)
* Conflict detection & explicit resolution (`Keep Local`, `Keep Remote`, `Merge`)
* Idempotency & bounded retry behavior
* Territorial authorization boundaries for sync and conflict resolution
* Versioned backup export (Format v1, Schema v7) with SHA-256 integrity verification
* Safe atomic restore with pre-restore safety backup and automatic rollback
* Operational audit events for sync, backup, license, and billing operations
* Client identity provider abstraction (`ClientIdentityProvider`, `DefaultClientIdentityProvider`)
* Multi-client request metadata (`SyncClientContext`) and structured acknowledgements (`RemoteOperationAck`)
* Platform-neutral remote revision verification (`checkRemoteRevision`)
* Portable backup media path normalization (safe across Windows, Android, Linux)
* Windows License Manager UI with 5 administrative tabs (`LicenseManagerScreen`)
* Consolidated license lifecycle & 30-day grace period calculation (`LicenseValidationService`)
* Multi-device registration & capacity enforcement (`maxDevices`, `currentDeviceCount`, `license_devices`)
* Device deactivation, blocking, and unblocking controls
* Administrative user-license assignment and account status management
* Role-based license authorization guard (`LicenseAuthorizationService`)
* Milestone 8 Billing & Payments Domain (`PaymentModel`, `InvoiceModel`, `PlanPricing`)
* Integer Minor Currency Units (Paisa / `amountMinor`, avoiding floating-point currency errors)
* Deterministic collision-resistant offline invoice numbering (`INV-YYYY-XXXX`)
* Deterministic payment reconciliation (partial payments, remaining balance, overpayment prevention)
* Role-based financial authorization guard (`BillingAuthorizationService` — Admin only; platform neutrality enforced)
* Payment provider abstraction (`IPaymentProvider`) with honest production boundary (`ProductionPaymentProviderImpl: PENDING`)
* Financial Admin Dashboard UI (`BillingDashboardScreen` with Overview, Invoices, Payments, Pricing tabs)
* Detailed Invoice Presentation Screen (`InvoiceDetailsScreen`) with manual payment recording and itemized receipts
* Sanitized financial audit logging (`PAYMENT_CREATED`, `PAYMENT_CONFIRMED`, `INVOICE_CREATED`, `INVOICE_ISSUED`, etc.)
* **Milestone 9 Security Hardening & Defenses**
* Automated secret, credential, token, GPS, card, and filesystem path redaction in operational logs (`EncryptionService.sanitizeForLogging`)
* Defensive path traversal prevention in media attachments and backup packages (`EncryptionService.sanitizeFilePath`)
* Robust input boundary verification (rejection of NaN/Infinity, malformed dates, cross-beat foreign entities)
* Android application release hardening (`allowBackup="false"`, `usesCleartextTraffic="false"`, explicit permission scope)
* Native Windows release binary compilation and verification (`build\windows\x64\runner\Release\beatbook.exe`)
* Comprehensive automated test suite expansion (267/267 automated tests passing, 0 analyzer issues)
* Complete offline SQLite autonomy (Schema v7)

### Explicitly Not Implemented (Strictly Reserved for Future Roadmap / Server Deployments)
* Production remote payment gateway / server backend (REST/GraphQL/Stripe/Easypaisa): **PENDING (Honest Status Displayed)**
* Android application / Android UI / Android auth screens: **NOT IMPLEMENTED**
* Local Database File-Level Encryption (SQLCipher): **NOT IMPLEMENTED (Standard SQLite v7 at rest; client architecture avoids home-made encryption)**
* Continuous/background GPS tracking, surveillance, social scraping, or biometric identification: **NEVER PLANNED**

---

## 17. Milestone 8 Billing & Financial Architecture

### Database Schema Version 7:
- **`invoices`**: Primary billing ledger storing invoice metadata, plan identifier, minor-unit subtotal, tax, discounts, total, and reconciled paid amount.
- **`payments`**: Transaction records storing integer `amount_minor`, currency, payment status (`pending`, `initiated`, `successful`, `failed`, `cancelled`, `refunded`), payment method, and masked references.

### Payment Lifecycle & Financial Truth:
1. **Local Offline Creation**: When a manual payment or billing draft is created offline, it enters `pending` status (`Pending Verification`).
2. **Manual Admin Verification vs. Gateway Confirmation**:
   - When an administrator manually confirms an offline cash or bank receipt, it is recorded strictly as `Manually Verified` with `is_gateway_confirmed = false`.
   - The verifier's identity (`adminUserId`) is auditable, and the operational event `PAYMENT_MANUALLY_VERIFIED` is logged.
   - `is_gateway_confirmed == true` is reserved strictly for real remote gateway/server verification.
3. **No Fake Gateway Success**: In the absence of a live centralized gateway, `ProductionPaymentProviderImpl` honestly reports `Production Payment Gateway Integration: PENDING`.
4. **Deterministic Source-of-Truth Reconciliation**:
   - `paid_amount_minor` on invoices is transactionally reconciled from the confirmed payment ledger (`SUM(amount_minor) WHERE status = 'successful'`).
   - Overpayments are strictly blocked at record time (exact remaining balance boundary enforced).
5. **Deterministic Full-Refund Semantics**:
   - Only confirmed successful payments can be refunded; double-refunds and refunding unconfirmed payments are rejected.
   - Refunds execute within an isolated SQLite transaction, reducing `paid_amount_minor` and reverting invoice status deterministically.
6. **Ledger-Based Dashboard Definitions**:
   - **Total Billed**: Sum of active billable invoices (`issued`, `partiallyPaid`, `overdue`, `paid`), excluding drafts and cancellations.
   - **Confirmed Received**: Sum of all confirmed successful payments minus confirmed refunds.
   - **Outstanding Balance**: Sum of `remainingBalanceMinor` across active billable invoices (drafts, cancelled, and refunded invoices are excluded).
7. **Offline Invoice Numbering**:
   - Human-readable numbers (`INV-YYYYMMDD-XXXX`) are local draft identifiers. The stable UUID v4 (`id`) remains the authoritative global entity identity across clients.
8. **Backup / Restore Inclusion**:
   - `invoices` and `payments` are fully integrated into `BackupRepositoryImpl.exportableTables` in strict referential dependency order.
9. **Strict Authorization**:
   - Enforced at repository and domain level below the UI (`BillingAuthorizationService`).
   - Only active `administrator` users may perform billing mutations. Supervisors, field users, and inactive administrators are blocked. Platform identity (`clientType == 'windows'`) cannot elevate privileges.

---

## 18. Milestone 9 Security Hardening & Release Architecture

### Core Hardening Principles
1. **Threat Model & Credential Protection:**
   - Production passwords, tokens, and hardware secrets are never persisted in SQLite or unencrypted storage.
   - All logging is filtered through `EncryptionService.sanitizeForLogging` which automatically redacts CNICs, passwords, PINs, tokens, GPS coordinates, license keys, card/CVV secrets, and filesystem paths.
2. **Defensive Path Traversal Mitigation:**
   - Media attachments and restored backup packages enforce `EncryptionService.sanitizeFilePath`.
   - Directory escaping sequences (`../`, `..\`) and dangerous executable extensions (`.exe`, `.bat`, `.sh`, `.php`) are blocked.
3. **Multi-Tier Authorization Boundaries:**
   - Authorization is enforced below the UI at the domain and repository layers (`TerritorialAuthorizationService`, `PeoplePlacesAuthorizationService`, `DailyReportAuthorizationService`, `SyncAuthorizationService`, `LicenseAuthorizationService`, `BillingAuthorizationService`).
   - Client platform (`clientType`) can never elevate privileges.
4. **Database Integrity & Input Fuzzing Protections:**
   - Dynamic field records strictly reject `NaN`, `Infinity`, malformed decimals, non-integer inputs, and invalid ISO dates.
   - Cross-beat references are blocked at validation time.
5. **Database Encryption Status:**
   - Standard unencrypted SQLite v7 is currently employed. Database file encryption via SQLCipher remains an honest production limitation (no home-made encryption used).
6. **Platform Release Hardening:**
   - **Android**: `android:allowBackup="false"` prevents local extraction; `android:usesCleartextTraffic="false"` prevents cleartext HTTP; background location tracking is strictly forbidden.
   - **Windows**: Native release build compiled and verified: `build\windows\x64\runner\Release\beatbook.exe`.

---

## 19. Running the Application & Tests

### Execute Full Test Suite (267 Tests):
```powershell
& "C:\Users\IamAt\.puro\envs\stable\flutter\bin\flutter.bat" test
```

### Execute Static Analysis (0 Issues):
```powershell
& "C:\Users\IamAt\.puro\envs\stable\flutter\bin\flutter.bat" analyze
```

---

## 20. Support & Contact

**CACHELINES**  
Software Architect & Lead Engineer: **Atif Syed**  
WhatsApp Hotline: **+92 300 4860591**  
Company: [https://cachelines.github.io/](https://cachelines.github.io/)  
Portfolio: [https://iatifsyed.github.io/](https://iatifsyed.github.io/)



