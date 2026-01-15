# GramSeva Health - Key Requirements Reference

## Core Requirements (Must-Have Features)

This document serves as the **single source of truth** for the 5 core requirements that form the foundation of GramSeva Health. All architecture, features, and business decisions must align with these requirements.

---

## 1. Assisted Tele-Consultation Platform

### Requirement Statement
**Video consultation module optimized for low bandwidth, allowing the Sahayak (intermediary) to upload vitals from IoT devices (BP monitor, Oximeter) directly to the doctor's screen.**

### Key Components
- ✅ **Low-Bandwidth Video**: Optimized for 256 Kbps minimum
- ✅ **Multi-Party Calls**: Patient, Sahayak, Doctor (3 participants)
- ✅ **IoT Device Integration**: BP Monitor, Oximeter, Thermometer
- ✅ **Real-Time Vitals Upload**: Sahayak uploads vitals during consultation
- ✅ **Doctor Screen Display**: Vitals appear in real-time on doctor's interface

### Architecture Mapping
- **Service**: `TeleConsultationService` (ARCHITECTURE.md)
- **Component**: `MediaManager`, `VitalsManager` (ARCHITECTURE.md)
- **Pattern**: Store-and-Forward, Strategy Pattern (for bandwidth optimization)
- **Technology**: WebRTC, Media Server (Janus/Kurento), BLE for IoT devices

### Feature Mapping
- **Feature ID**: F1 (FEATURES.md)
- **Priority**: P0 (Critical)
- **User Stories**: US1.1, US1.2, US1.3, US1.4 (FEATURES.md)
- **Acceptance Criteria**: FR1.1-FR1.8, NFR1.1-NFR1.5 (FEATURES.md)

### Business Impact
- **Value Proposition**: Core consultation capability (BUSINESS_PERSPECTIVE.md)
- **Revenue Driver**: Primary revenue stream (consultation fees)
- **Competitive Advantage**: Low-bandwidth optimization for rural areas

### Implementation Status
- ✅ Architecture designed
- ✅ Features specified
- ⏳ Implementation pending

---

## 2. AI-Powered Triage & Translation

### Requirement Statement
**A GenAI bot that listens to the patient's symptoms in local dialect, translates them to English medical summaries for the doctor, and suggests a preliminary severity score.**

### Key Components
- ✅ **Voice/Text Input**: Accepts symptoms in local dialect
- ✅ **Language Detection**: Automatically identifies dialect/language
- ✅ **Translation to English**: Converts local dialect to English
- ✅ **Medical Summary Generation**: Creates structured medical summary
- ✅ **Severity Scoring**: Calculates preliminary severity score (1-10)
- ✅ **Triage Priority**: Suggests LOW, MEDIUM, HIGH, CRITICAL

### Architecture Mapping
- **Service**: `AITriageService` (ARCHITECTURE.md)
- **Components**: `TranslationEngine`, `SymptomAnalyzer`, `TriageEngine`, `LLMService`
- **Pattern**: Strategy Pattern (for different translation strategies)
- **Technology**: LLM (GPT-4/Claude), Speech-to-Text, Translation APIs

### Feature Mapping
- **Feature ID**: F2 (FEATURES.md)
- **Priority**: P0 (Critical)
- **User Stories**: US2.1, US2.2, US2.3, US2.4 (FEATURES.md)
- **Acceptance Criteria**: FR2.1-FR2.9, NFR2.1-NFR2.5 (FEATURES.md)

### Business Impact
- **Value Proposition**: Overcomes language barriers (BUSINESS_PERSPECTIVE.md)
- **Revenue Driver**: Enables consultation prioritization (better efficiency)
- **Competitive Advantage**: Multi-language AI (10+ Indian languages)

### Implementation Status
- ✅ Architecture designed
- ✅ Features specified
- ⏳ Implementation pending

---

## 3. Prescription & Fulfillment Engine

### Requirement Statement
**Digital prescription generation that integrates with local pharmacy supply chains for medicine delivery.**

### Key Components
- ✅ **Digital Prescription Generation**: Create prescriptions with medicine details
- ✅ **Pharmacy Integration**: Connect with local pharmacy supply chains
- ✅ **Medicine Availability Check**: Real-time availability checking
- ✅ **Order Management**: Place orders, track delivery
- ✅ **Delivery Integration**: Coordinate with pharmacy delivery systems

### Architecture Mapping
- **Service**: `PrescriptionService` (ARCHITECTURE.md)
- **Components**: `PrescriptionController`, `PharmacyIntegration`, `OrderManager`
- **Pattern**: Adapter Pattern (for pharmacy API integration)
- **Technology**: REST APIs, Pharmacy partner APIs, Order tracking systems

### Feature Mapping
- **Feature ID**: F3 (FEATURES.md)
- **Priority**: P1 (High - completes consultation flow)
- **User Stories**: US3.1, US3.2, US3.3, US3.4 (FEATURES.md)
- **Acceptance Criteria**: FR3.1-FR3.10, NFR3.1-NFR3.5 (FEATURES.md)

### Business Impact
- **Value Proposition**: Complete care cycle (BUSINESS_PERSPECTIVE.md)
- **Revenue Driver**: Medicine sales commission (10-15% of order value)
- **Competitive Advantage**: Seamless pharmacy integration

### Implementation Status
- ✅ Architecture designed
- ✅ Features specified
- ⏳ Implementation pending

---

## 4. Offline Sync & Storage

### Requirement Statement
**"Store and Forward" architecture to cache patient data locally when the internet is down and sync when connectivity is restored.**

### Key Components
- ✅ **Local Data Storage**: SQLite database on mobile devices
- ✅ **Store-and-Forward**: Queue operations when offline
- ✅ **Automatic Sync**: Sync when connectivity restored
- ✅ **Conflict Resolution**: Handle data conflicts intelligently
- ✅ **Incremental Sync**: Only sync changed data

### Architecture Mapping
- **Service**: `OfflineSyncService` (ARCHITECTURE.md)
- **Components**: `LocalStorageManager`, `SyncManager`, `ConflictResolver`
- **Pattern**: Store-and-Forward Pattern (ARCHITECTURE.md)
- **Technology**: SQLite (local), PostgreSQL/MongoDB (remote), Background sync

### Feature Mapping
- **Feature ID**: F4 (FEATURES.md)
- **Priority**: P0 (Critical - foundation for all features)
- **User Stories**: US4.1, US4.2, US4.3, US4.4 (FEATURES.md)
- **Acceptance Criteria**: FR4.1-FR4.10, NFR4.1-NFR4.5 (FEATURES.md)

### Business Impact
- **Value Proposition**: Works in unreliable connectivity (BUSINESS_PERSPECTIVE.md)
- **Revenue Driver**: Enables operations in rural areas (market expansion)
- **Competitive Advantage**: Offline-first architecture (unique in market)

### Implementation Status
- ✅ Architecture designed
- ✅ Features specified
- ⏳ Implementation pending

---

## 5. Consent & Privacy (DPDP Act Compliance)

### Requirement Statement
**Strict role-based access control ensuring Sahayaks can only see data for active patients, with explicit consent management (OTP/Biometric) for sharing records.**

### Key Components
- ✅ **Role-Based Access Control (RBAC)**: Strict permissions by role
- ✅ **Sahayak Restrictions**: Only active patient data visible
- ✅ **Explicit Consent Management**: OTP/Biometric verification
- ✅ **Consent Types**: Data sharing, recording, pharmacy sharing, govt scheme
- ✅ **Audit Logging**: All access logged for compliance
- ✅ **DPDP Act Compliance**: Data privacy and protection

### Architecture Mapping
- **Service**: `ConsentPrivacyService` (ARCHITECTURE.md)
- **Components**: `ConsentManager`, `AccessControl`, `EncryptionService`, `AuditLogger`
- **Pattern**: Proxy Pattern (for access control), Decorator Pattern (for encryption)
- **Technology**: OAuth 2.0, RBAC, Encryption (AES-256), Audit logging

### Feature Mapping
- **Feature ID**: F5 (FEATURES.md)
- **Priority**: P0 (Critical - legal compliance)
- **User Stories**: US5.1, US5.2, US5.3, US5.4 (FEATURES.md)
- **Acceptance Criteria**: FR5.1-FR5.10, NFR5.1-NFR5.5 (FEATURES.md)

### Business Impact
- **Value Proposition**: Trust and compliance (BUSINESS_PERSPECTIVE.md)
- **Revenue Driver**: Enables data sharing (pharmacy, govt schemes)
- **Competitive Advantage**: DPDP Act compliance (regulatory requirement)

### Implementation Status
- ✅ Architecture designed
- ✅ Features specified
- ⏳ Implementation pending

---

## Requirements Dependency Matrix

| Requirement | Depends On | Enables | Critical Path |
|-------------|------------|---------|---------------|
| **R1: Tele-Consultation** | R4 (Offline Sync), R5 (Consent) | R3 (Prescription) | Phase 1 |
| **R2: AI Triage** | R4 (Offline Sync) | R1 (Tele-Consultation) | Phase 1 |
| **R3: Prescription** | R1 (Tele-Consultation), R4, R5 | Additional revenue | Phase 2 |
| **R4: Offline Sync** | None (Foundation) | All features | Phase 0 |
| **R5: Consent & Privacy** | None (Foundation) | All data access | Phase 0 |

### Implementation Priority

**Phase 0 (Foundation - Weeks 1-4)**:
1. ✅ R4: Offline Sync & Storage
2. ✅ R5: Consent & Privacy

**Phase 1 (Core - Weeks 5-10)**:
3. ✅ R2: AI Triage & Translation
4. ✅ R1: Assisted Tele-Consultation

**Phase 2 (Completion - Weeks 11-14)**:
5. ✅ R3: Prescription & Fulfillment

---

## Cross-Reference to Documentation

### Architecture Documentation
- **File**: `ARCHITECTURE.md`
- **Sections**: 
  - HLD: System Architecture (lines 12-93)
  - LLD: Tele-Consultation Service (lines 200-300)
  - LLD: AI Triage Service (lines 350-450)
  - LLD: Prescription Service (lines 500-600)
  - LLD: Offline Sync Service (lines 650-750)
  - LLD: Consent & Privacy Service (lines 800-900)
  - Design Patterns: Store-and-Forward, RBAC (lines 950-1100)

### Features Documentation
- **File**: `FEATURES.md`
- **Sections**:
  - F1: Assisted Tele-Consultation (lines 15-49)
  - F2: AI Triage & Translation (lines 52-88)
  - F3: Prescription & Fulfillment (lines 91-130)
  - F4: Offline Sync & Storage (lines 133-180)
  - F5: Consent & Privacy (lines 183-230)

### Business Perspective
- **File**: `BUSINESS_PERSPECTIVE.md`
- **Sections**:
  - Value Proposition (lines 75-136)
  - Revenue Streams (lines 223-323)
  - International Models (lines 645-950)

### Feature Specifications
- **File**: `FEATURE_SPECIFICATIONS.md`
- **Sections**:
  - F1 Technical Specs (lines 50-200)
  - F2 Technical Specs (lines 250-400)
  - F3 Technical Specs (lines 450-600)
  - F4 Technical Specs (lines 650-850)
  - F5 Technical Specs (lines 900-1100)

---

## Validation Checklist

Use this checklist to ensure all requirements are met in implementation:

### R1: Assisted Tele-Consultation
- [ ] Video works with 256 Kbps bandwidth
- [ ] Sahayak can upload vitals from IoT devices
- [ ] Vitals appear on doctor's screen in real-time
- [ ] Multi-party calls (Patient, Sahayak, Doctor) work
- [ ] Handles connection drops gracefully

### R2: AI Triage & Translation
- [ ] Accepts voice/text in local dialect
- [ ] Detects language automatically
- [ ] Translates to English medical summary
- [ ] Calculates severity score (1-10)
- [ ] Suggests triage priority
- [ ] Supports 10+ Indian languages

### R3: Prescription & Fulfillment
- [ ] Generates digital prescriptions
- [ ] Integrates with pharmacy APIs
- [ ] Checks medicine availability
- [ ] Places orders with pharmacy
- [ ] Tracks delivery status

### R4: Offline Sync & Storage
- [ ] Stores data locally when offline
- [ ] Queues operations for sync
- [ ] Syncs automatically when online
- [ ] Resolves conflicts intelligently
- [ ] Maintains data integrity

### R5: Consent & Privacy
- [ ] RBAC enforced for all access
- [ ] Sahayaks only see active patient data
- [ ] Consent verified via OTP/Biometric
- [ ] All access logged for audit
- [ ] DPDP Act compliant

---

## Success Metrics

### R1: Tele-Consultation
- ✅ 95% video call success rate
- ✅ <10 seconds connection time
- ✅ Vitals upload <2 seconds
- ✅ 90% consultation completion rate

### R2: AI Triage
- ✅ >85% translation accuracy
- ✅ <5 seconds translation time
- ✅ Severity score accuracy >80%
- ✅ 10+ languages supported

### R3: Prescription
- ✅ 100% prescription generation success
- ✅ <3 seconds availability check
- ✅ 95% order delivery success
- ✅ 50% consultation-to-order conversion

### R4: Offline Sync
- ✅ 100% data saved locally when offline
- ✅ 95% sync success rate
- ✅ <5 minutes sync time for typical data
- ✅ 90% automatic conflict resolution

### R5: Consent & Privacy
- ✅ 100% access control enforcement
- ✅ <30 seconds consent verification
- ✅ 100% access logged
- ✅ 0 unauthorized access incidents

---

## Notes

- **These 5 requirements are NON-NEGOTIABLE** - they form the core value proposition
- All other features (Visual Diagnosis, Govt Integration, WhatsApp Bot) are enhancements
- Implementation must prioritize these 5 requirements in the specified order
- Any architecture or design decision must support these requirements
- Business model and revenue streams depend on successful implementation of these requirements

---

**Last Updated**: Based on core requirements from project specification
**Status**: All requirements documented and mapped to architecture/features
**Next Steps**: Begin Phase 0 implementation (R4, R5)
