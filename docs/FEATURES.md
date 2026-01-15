# GramSeva Health - Features Documentation

> **⚠️ Core Requirements**: Features F1-F5 are the 5 critical requirements that form the foundation. See [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md) for the complete reference.

## Table of Contents
1. [Core Features](#core-features)
2. [Additional Features](#additional-features)
3. [Feature Dependencies](#feature-dependencies)
4. [Feature Prioritization](#feature-prioritization)
5. [User Stories](#user-stories)
6. [Acceptance Criteria](#acceptance-criteria)
7. [Feature Interaction Matrix](#feature-interaction-matrix)
8. [Detailed Technical Specifications](#detailed-technical-specifications)
9. [API Specifications](#api-specifications)
10. [Data Flow Diagrams](#data-flow-diagrams)
11. [Error Handling](#error-handling)
12. [Feature Implementation Roadmap](#feature-implementation-roadmap)
13. [Resource Allocation](#resource-allocation)
14. [Risk Mitigation](#risk-mitigation)
15. [Testing Strategy](#testing-strategy)
16. [Success Metrics by Phase](#success-metrics-by-phase)

---

## Core Features

> **Note**: Features F1-F5 correspond to the 5 core requirements:
> - **F1** = Assisted Tele-Consultation Platform
> - **F2** = AI-Powered Triage & Translation
> - **F3** = Prescription & Fulfillment Engine
> - **F4** = Offline Sync & Storage
> - **F5** = Consent & Privacy (DPDP Act Compliance)

### F1: Assisted Tele-Consultation Platform

#### Description
A video consultation module optimized for low bandwidth, allowing the Sahayak (intermediary) to upload vitals from IoT devices (BP monitor, Oximeter) directly to the doctor's screen.

#### Functional Requirements
- **FR1.1**: Support multi-party video calls (Patient, Sahayak, Doctor)
- **FR1.2**: Optimize video quality based on available bandwidth (adaptive bitrate)
- **FR1.3**: Allow Sahayak to upload vitals from IoT devices during consultation
- **FR1.4**: Display vitals in real-time on doctor's screen
- **FR1.5**: Support screen sharing for medical records/images
- **FR1.6**: Record consultation sessions (with patient consent)
- **FR1.7**: Handle connection drops gracefully with reconnection
- **FR1.8**: Support audio-only mode for very low bandwidth scenarios

#### Non-Functional Requirements
- **NFR1.1**: Video call should work with minimum 256 Kbps bandwidth
- **NFR1.2**: Latency should be < 500ms for real-time communication
- **NFR1.3**: Support up to 3 concurrent participants
- **NFR1.4**: Video quality should auto-adjust based on network conditions
- **NFR1.5**: Consultation recordings should be encrypted and stored securely

#### Technical Components
- WebRTC for peer-to-peer video communication
- Media server (Janus/Kurento) for multi-party calls
- Adaptive bitrate streaming
- IoT device integration layer
- Real-time data synchronization

#### User Stories
- **US1.1**: As a Doctor, I want to see patient vitals in real-time during consultation so I can make informed decisions
- **US1.2**: As a Sahayak, I want to upload vitals from IoT devices so doctors have accurate patient data
- **US1.3**: As a Patient, I want to have video consultation even with poor internet so I can access healthcare
- **US1.4**: As a Doctor, I want to see consultation recordings so I can review complex cases

---

### F2: AI-Powered Triage & Translation

#### Description
A GenAI bot that listens to the patient's symptoms in local dialect, translates them to English medical summaries for the doctor, and suggests a preliminary severity score.

#### Functional Requirements
- **FR2.1**: Accept symptom input in multiple formats (voice, text, structured form)
- **FR2.2**: Detect and identify local dialect/language automatically
- **FR2.3**: Translate patient symptoms from local dialect to English
- **FR2.4**: Extract medical symptoms and conditions from translated text
- **FR2.5**: Generate structured medical summary for doctor
- **FR2.6**: Calculate severity score (1-10 scale)
- **FR2.7**: Suggest triage priority (LOW, MEDIUM, HIGH, CRITICAL)
- **FR2.8**: Provide confidence score for translation and analysis
- **FR2.9**: Support multiple Indian languages/dialects (Hindi, Telugu, Tamil, Bengali, etc.)

#### Non-Functional Requirements
- **NFR2.1**: Translation should complete within 5 seconds
- **NFR2.2**: Accuracy should be > 85% for common symptoms
- **NFR2.3**: Support at least 10 major Indian languages
- **NFR2.4**: Severity score should be consistent across similar symptoms
- **NFR2.5**: System should handle ambiguous symptoms gracefully

#### Technical Components
- Speech-to-Text service (Google/AWS)
- Language detection model
- Translation service (custom model or API)
- LLM service (GPT-4/Claude) for medical summarization
- Severity scoring ML model
- Triage recommendation engine

#### User Stories
- **US2.1**: As a Patient, I want to describe symptoms in my local language so I can communicate effectively
- **US2.2**: As a Doctor, I want to see translated and summarized symptoms so I can understand patient conditions quickly
- **US2.3**: As a System, I want to prioritize consultations based on severity so critical cases are handled first
- **US2.4**: As a Sahayak, I want the system to translate patient speech so I can help bridge communication gaps

---

### F3: Prescription & Fulfillment Engine

#### Description
Digital prescription generation that integrates with local pharmacy supply chains for medicine delivery.

#### Functional Requirements
- **FR3.1**: Generate digital prescriptions with medicine details, dosage, frequency
- **FR3.2**: Support multiple medicines in a single prescription
- **FR3.3**: Include doctor's instructions and follow-up date
- **FR3.4**: Check medicine availability with pharmacy partners
- **FR3.5**: Allow patient to place order for medicines
- **FR3.6**: Track order status (placed, confirmed, dispatched, delivered)
- **FR3.7**: Support multiple pharmacy partners
- **FR3.8**: Generate prescription PDF for patient records
- **FR3.9**: Send prescription to patient via SMS/WhatsApp
- **FR3.10**: Support prescription refills for chronic conditions

#### Non-Functional Requirements
- **NFR3.1**: Prescription generation should be < 2 seconds
- **NFR3.2**: Medicine availability check should be < 3 seconds
- **NFR3.3**: Support integration with at least 5 pharmacy partners
- **NFR3.4**: Prescription should be tamper-proof (digital signature)
- **NFR3.5**: Order tracking should update in real-time

#### Technical Components
- Prescription generation service
- Pharmacy API integration layer
- Order management system
- PDF generation service
- Notification service (SMS/WhatsApp)
- Digital signature service

#### User Stories
- **US3.1**: As a Doctor, I want to generate digital prescriptions so patients can easily access medicines
- **US3.2**: As a Patient, I want to order medicines online so I don't have to visit pharmacy
- **US3.3**: As a Patient, I want to track my medicine order so I know when to expect delivery
- **US3.4**: As a Pharmacy, I want to receive prescription orders so I can fulfill them

---

### F4: Offline Sync & Storage

#### Description
"Store and Forward" architecture to cache patient data locally when the internet is down and sync when connectivity is restored.

#### Functional Requirements
- **FR4.1**: Store all patient data locally on device (SQLite)
- **FR4.2**: Queue all write operations when offline
- **FR4.3**: Automatically sync when connectivity is restored
- **FR4.4**: Detect and resolve data conflicts
- **FR4.5**: Show sync status to users
- **FR4.6**: Allow manual sync trigger
- **FR4.7**: Support incremental sync (only changed data)
- **FR4.8**: Handle sync failures with retry mechanism
- **FR4.9**: Maintain data integrity during sync
- **FR4.10**: Support partial sync (priority-based)

#### Non-Functional Requirements
- **NFR4.1**: Local database should support at least 1000 patient records
- **NFR4.2**: Sync should complete within 5 minutes for typical data volume
- **NFR4.3**: Conflict resolution should be automatic for 90% of cases
- **NFR4.4**: Data should be encrypted in local storage
- **NFR4.5**: Sync should work in background without blocking UI

#### Technical Components
- Local database (SQLite)
- Sync queue manager
- Conflict resolution engine
- Background sync service
- Data encryption service
- Sync status tracker

#### User Stories
- **US4.1**: As a Sahayak, I want to capture patient data offline so I can work even without internet
- **US4.2**: As a System, I want to sync data automatically so all information is up-to-date
- **US4.3**: As a User, I want to see sync status so I know when data is synchronized
- **US4.4**: As a System, I want to resolve conflicts automatically so data integrity is maintained

---

### F5: Consent & Privacy (DPDP Act Compliance)

#### Description
Strict role-based access control ensuring Sahayaks can only see data for active patients, with explicit consent management (OTP/Biometric) for sharing records.

#### Functional Requirements
- **FR5.1**: Implement role-based access control (RBAC)
- **FR5.2**: Request explicit consent for data sharing
- **FR5.3**: Support multiple consent types (data sharing, recording, pharmacy sharing, govt scheme)
- **FR5.4**: Verify consent using OTP or Biometric
- **FR5.5**: Restrict Sahayak access to active patients only
- **FR5.6**: Allow patients to revoke consent
- **FR5.7**: Log all data access for audit
- **FR5.8**: Encrypt sensitive patient data
- **FR5.9**: Support consent expiration and renewal
- **FR5.10**: Provide consent history to patients

#### Non-Functional Requirements
- **NFR5.1**: Consent verification should complete within 30 seconds
- **NFR5.2**: All data access should be logged within 1 second
- **NFR5.3**: Encryption should use AES-256 standard
- **NFR5.4**: Consent should be stored immutably
- **NFR5.5**: Access control checks should add < 100ms latency

#### Technical Components
- RBAC service
- Consent management service
- OTP service
- Biometric authentication service
- Audit logging service
- Encryption service
- Access control middleware

#### User Stories
- **US5.1**: As a Patient, I want to control who can access my data so my privacy is protected
- **US5.2**: As a Sahayak, I want to access only active patient data so I can assist effectively
- **US5.3**: As a System, I want to log all data access so compliance is maintained
- **US5.4**: As a Patient, I want to revoke consent so I have control over my data

---

## Additional Features

### F6: Visual Diagnosis Aid

#### Description
Computer Vision tools for skin conditions/wounds analysis to assist general practitioners in preliminary assessment.

#### Functional Requirements
- **FR6.1**: Accept image uploads (skin conditions, wounds, rashes)
- **FR6.2**: Analyze images using computer vision models
- **FR6.3**: Identify potential conditions/diseases
- **FR6.4**: Provide confidence scores for identifications
- **FR6.5**: Generate preliminary assessment report
- **FR6.6**: Suggest follow-up actions or tests
- **FR6.7**: Support multiple image formats (JPG, PNG)
- **FR6.8**: Allow image annotation by doctor
- **FR6.9**: Store images securely with patient records
- **FR6.10**: Support image comparison (before/after treatment)

#### Non-Functional Requirements
- **NFR6.1**: Image analysis should complete within 10 seconds
- **NFR6.2**: Accuracy should be > 80% for common skin conditions
- **NFR6.3**: Support images up to 10MB
- **NFR6.4**: Images should be compressed for storage
- **NFR6.5**: Analysis should work offline (on-device model)

#### Technical Components
- Image upload service
- Computer vision model (TensorFlow/PyTorch)
- Image processing service
- Condition classification engine
- Report generation service
- Image storage service

#### User Stories
- **US6.1**: As a Doctor, I want to analyze skin condition images so I can make preliminary assessments
- **US6.2**: As a Patient, I want to upload images of my condition so doctor can see it clearly
- **US6.3**: As a Doctor, I want AI suggestions for conditions so I can consider all possibilities
- **US6.4**: As a System, I want to store images securely so patient privacy is maintained

---

### F7: Government Scheme Integration

#### Description
API integration with Ayushman Bharat (ABHA IDs) to verify insurance eligibility and fetch past health records.

#### Functional Requirements
- **FR7.1**: Validate ABHA ID with government systems
- **FR7.2**: Check insurance eligibility under Ayushman Bharat
- **FR7.3**: Fetch patient's past health records
- **FR7.4**: Display insurance coverage details
- **FR7.5**: Calculate co-payment amount (if any)
- **FR7.6**: Submit claims to insurance (if applicable)
- **FR7.7**: Support multiple government schemes (not just Ayushman Bharat)
- **FR7.8**: Cache eligibility data for offline access
- **FR7.9**: Handle API failures gracefully
- **FR7.10**: Maintain audit trail of all API calls

#### Non-Functional Requirements
- **NFR7.1**: ABHA validation should complete within 5 seconds
- **NFR7.2**: System should handle API timeouts gracefully
- **NFR7.3**: Support retry mechanism for failed API calls
- **NFR7.4**: Cache eligibility data for 24 hours
- **NFR7.5**: All API calls should be logged for audit

#### Technical Components
- Government API integration service
- ABHA validation service
- Insurance eligibility checker
- Health record fetcher
- Claim submission service
- API gateway with circuit breaker
- Caching service

#### User Stories
- **US7.1**: As a Patient, I want to use my ABHA ID so I can access government benefits
- **US7.2**: As a Doctor, I want to see patient's past health records so I can provide better care
- **US7.3**: As a System, I want to verify insurance eligibility so patients know their coverage
- **US7.4**: As a Patient, I want to see my insurance coverage so I can plan my expenses

---

### F8: WhatsApp Bot for Follow-ups

#### Description
Automated vernacular reminders for medication dosage and follow-up appointment scheduling.

#### Functional Requirements
- **FR8.1**: Send medication reminders in patient's preferred language
- **FR8.2**: Support multiple reminder types (medication, appointment, follow-up)
- **FR8.3**: Allow patients to confirm/acknowledge reminders
- **FR8.4**: Support two-way communication (patient can ask questions)
- **FR8.5**: Schedule follow-up appointments via WhatsApp
- **FR8.6**: Send health tips and preventive care information
- **FR8.7**: Handle patient queries using NLP
- **FR8.8**: Escalate complex queries to human support
- **FR8.9**: Support multiple Indian languages
- **FR8.10**: Track medication adherence

#### Non-Functional Requirements
- **NFR8.1**: Reminders should be sent at scheduled times (±5 minutes)
- **NFR8.2**: Bot should respond within 2 seconds for simple queries
- **NFR8.3**: Support at least 1000 concurrent conversations
- **NFR8.4**: Messages should be delivered reliably (99% success rate)
- **NFR8.5**: Bot should handle typos and informal language

#### Technical Components
- WhatsApp Business API integration
- NLP service for query understanding
- Reminder scheduler
- Appointment booking service
- Multi-language support service
- Conversation state manager
- Escalation service

#### User Stories
- **US8.1**: As a Patient, I want to receive medication reminders so I don't miss doses
- **US8.2**: As a Patient, I want to schedule appointments via WhatsApp so it's convenient
- **US8.3**: As a Patient, I want to ask health questions via WhatsApp so I can get quick answers
- **US8.4**: As a System, I want to track medication adherence so I can improve patient outcomes

---

## Feature Dependencies

### Dependency Graph

```
F1 (Tele-Consultation)
  ├── Depends on: F4 (Offline Sync), F5 (Consent)
  └── Enables: F3 (Prescription)

F2 (AI Triage & Translation)
  ├── Depends on: F4 (Offline Sync)
  └── Enables: F1 (Tele-Consultation), Queue prioritization

F3 (Prescription & Fulfillment)
  ├── Depends on: F1 (Tele-Consultation), F5 (Consent)
  └── Enables: F8 (WhatsApp Bot)

F4 (Offline Sync)
  └── Enables: F1, F2, F3, F6, F7 (all features work offline)

F5 (Consent & Privacy)
  └── Required by: F1, F3, F6, F7 (all data access features)

F6 (Visual Diagnosis)
  ├── Depends on: F1 (Tele-Consultation), F5 (Consent)
  └── Can work standalone

F7 (Govt Scheme Integration)
  ├── Depends on: F5 (Consent)
  └── Can work standalone

F8 (WhatsApp Bot)
  ├── Depends on: F3 (Prescription)
  └── Can work standalone
```

### Critical Path

**Phase 1 (Foundation)**:
1. F4 (Offline Sync) - Foundation for all features
2. F5 (Consent & Privacy) - Required for data access

**Phase 2 (Core Consultation)**:
3. F2 (AI Triage & Translation) - Enables smart queuing
4. F1 (Tele-Consultation) - Core consultation feature

**Phase 3 (Completion & Enhancement)**:
5. F3 (Prescription & Fulfillment) - Completes consultation flow
6. F6 (Visual Diagnosis) - Enhances diagnosis
7. F7 (Govt Scheme Integration) - Adds value
8. F8 (WhatsApp Bot) - Improves follow-up

---

## Feature Prioritization

### Priority Levels

#### P0 - Critical (Must Have)
- **F4**: Offline Sync & Storage
- **F5**: Consent & Privacy
- **F1**: Assisted Tele-Consultation Platform
- **F2**: AI-Powered Triage & Translation

**Rationale**: These form the core functionality. Without these, the platform cannot function in rural settings.

#### P1 - High (Should Have)
- **F3**: Prescription & Fulfillment Engine

**Rationale**: Completes the consultation flow and provides end-to-end value.

#### P2 - Medium (Nice to Have)
- **F6**: Visual Diagnosis Aid
- **F7**: Government Scheme Integration

**Rationale**: Enhances the platform but not essential for MVP.

#### P3 - Low (Future Enhancement)
- **F8**: WhatsApp Bot for Follow-ups

**Rationale**: Improves user experience but can be added post-MVP.

---

## User Stories

### Epic 1: Patient Consultation Journey

**As a Patient**, I want to:
- Register at a kiosk with my basic information
- Describe my symptoms in my local language
- Have a video consultation with a doctor
- Receive a digital prescription
- Order medicines online
- Track my medicine delivery
- Receive follow-up reminders

**Acceptance Criteria**:
- Patient can complete full journey from registration to medicine delivery
- All steps work offline (except video call)
- Patient can access consultation history
- Patient receives notifications at each step

### Epic 2: Sahayak Workflow

**As a Sahayak**, I want to:
- Register new patients
- Capture patient vitals using IoT devices
- Upload vitals to doctor's screen
- Assist patients during consultation
- View only active patient data
- Work offline when internet is unavailable

**Acceptance Criteria**:
- Sahayak can complete all tasks offline
- Vitals are uploaded in real-time during consultation
- Sahayak can only access authorized patient data
- Data syncs automatically when online

### Epic 3: Doctor Consultation Experience

**As a Doctor**, I want to:
- See prioritized patient queue
- View translated and summarized symptoms
- See real-time patient vitals
- Conduct video consultations
- Generate digital prescriptions
- Review consultation history
- Analyze patient images for diagnosis

**Acceptance Criteria**:
- Doctor can see all patient information in one place
- Consultations are prioritized by severity
- Prescriptions are generated quickly
- Video quality adapts to bandwidth

---

## Acceptance Criteria

### General Acceptance Criteria

1. **Performance**:
   - All API calls should respond within 3 seconds (95th percentile)
   - Video calls should work with minimum 256 Kbps
   - Offline operations should queue successfully
   - Sync should complete within 5 minutes for typical data

2. **Security**:
   - All data must be encrypted at rest and in transit
   - Access control must be enforced at all endpoints
   - All data access must be logged
   - Consent must be verified before data sharing

3. **Reliability**:
   - System should handle 99.5% uptime
   - Graceful degradation when services are down
   - Automatic retry for failed operations
   - Data should not be lost during sync

4. **Usability**:
   - Interface should support multiple languages
   - Should work on low-end devices
   - Should work with poor internet connectivity
   - Error messages should be clear and actionable

5. **Compliance**:
   - Must comply with DPDP Act
   - Must maintain audit trails
   - Must support consent management
   - Must support data deletion requests

### Feature-Specific Acceptance Criteria

#### F1: Tele-Consultation
- ✅ Video call connects within 10 seconds
- ✅ Vitals appear on doctor's screen within 2 seconds of upload
- ✅ Call quality adapts to bandwidth automatically
- ✅ Reconnection works within 5 seconds after drop

#### F2: AI Triage & Translation
- ✅ Translation completes within 5 seconds
- ✅ Severity score is calculated accurately (>85% accuracy)
- ✅ Supports at least 10 Indian languages
- ✅ Medical summary is clear and actionable

#### F3: Prescription & Fulfillment
- ✅ Prescription generates within 2 seconds
- ✅ Medicine availability check completes within 3 seconds
- ✅ Order tracking updates in real-time
- ✅ Prescription PDF is tamper-proof

#### F4: Offline Sync
- ✅ All data saves locally when offline
- ✅ Sync starts automatically when online
- ✅ Conflicts resolve automatically (90% of cases)
- ✅ Sync status is visible to users

#### F5: Consent & Privacy
- ✅ Consent verification completes within 30 seconds
- ✅ Access is denied without valid consent
- ✅ All access is logged within 1 second
- ✅ Patients can revoke consent anytime

#### F6: Visual Diagnosis
- ✅ Image analysis completes within 10 seconds
- ✅ Condition identification is >80% accurate
- ✅ Images are stored securely
- ✅ Reports are generated automatically

#### F7: Govt Scheme Integration
- ✅ ABHA validation completes within 5 seconds
- ✅ Eligibility check works offline (cached)
- ✅ API failures are handled gracefully
- ✅ All API calls are logged

#### F8: WhatsApp Bot
- ✅ Reminders sent at scheduled times (±5 minutes)
- ✅ Bot responds within 2 seconds
- ✅ Supports multiple languages
- ✅ Handles typos and informal language

---

## Feature Metrics & KPIs

### Success Metrics

1. **Adoption Metrics**:
   - Number of active patients per month
   - Number of consultations per day
   - Number of Sahayaks using the platform
   - Number of doctors on the platform

2. **Performance Metrics**:
   - Average consultation time
   - Video call success rate
   - Sync success rate
   - API response times

3. **Quality Metrics**:
   - Translation accuracy
   - Severity score accuracy
   - Visual diagnosis accuracy
   - Prescription accuracy

4. **User Satisfaction Metrics**:
   - Patient satisfaction score
   - Doctor satisfaction score
   - Sahayak satisfaction score
   - Net Promoter Score (NPS)

5. **Business Metrics**:
   - Medicine order conversion rate
   - Average order value
   - Patient retention rate
   - Cost per consultation

---

## Feature Roadmap

### MVP (Minimum Viable Product) - 3 Months
- F4: Offline Sync & Storage
- F5: Consent & Privacy
- F1: Assisted Tele-Consultation (basic)
- F2: AI Triage & Translation (basic)
- F3: Prescription & Fulfillment (basic)

### Phase 2 - 6 Months
- F1: Enhanced Tele-Consultation (recording, screen sharing)
- F2: Enhanced AI (more languages, better accuracy)
- F6: Visual Diagnosis Aid
- F7: Government Scheme Integration

### Phase 3 - 9 Months
- F8: WhatsApp Bot for Follow-ups
- Advanced analytics and reporting
- Mobile apps (iOS/Android)
- Admin dashboard

### Phase 4 - 12 Months
- AI-powered health recommendations
- Chronic disease management
- Telemedicine for specialized care
- Integration with more government schemes

---

## Feature Interaction Matrix

| Feature | F1 | F2 | F3 | F4 | F5 | F6 | F7 | F8 |
|---------|----|----|----|----|----|----|----|----|
| **F1: Tele-Consultation** | - | Uses | Enables | Uses | Requires | Can use | - | - |
| **F2: AI Triage** | Enables | - | - | Uses | - | - | - | - |
| **F3: Prescription** | Uses | - | - | Uses | Requires | - | - | Enables |
| **F4: Offline Sync** | Required | Required | Required | - | - | Required | Required | - |
| **F5: Consent** | Required | - | Required | - | - | Required | Required | - |
| **F6: Visual Diagnosis** | Can use | - | - | Uses | Requires | - | - | - |
| **F7: Govt Integration** | - | - | - | Uses | Requires | - | - | - |
| **F8: WhatsApp Bot** | - | - | Uses | - | - | - | - | - |

**Legend**:
- **Uses**: Feature depends on another feature
- **Enables**: Feature makes another feature possible
- **Required**: Feature is mandatory for another feature
- **Can use**: Feature can optionally use another feature

---

## Detailed Technical Specifications

### F1: Assisted Tele-Consultation Platform

#### Component Architecture

```
TeleConsultationService
├── ConsultationController
│   ├── POST /api/v1/consultations
│   ├── GET /api/v1/consultations/:id
│   ├── POST /api/v1/consultations/:id/join
│   ├── POST /api/v1/consultations/:id/end
│   └── GET /api/v1/consultations/queue
│
├── MediaManager
│   ├── createVideoRoom(consultationId)
│   ├── joinRoom(userId, roomId)
│   ├── optimizeBandwidth(connectionQuality)
│   ├── handleLowBandwidth()
│   └── recordSession(roomId, consent)
│
├── VitalsManager
│   ├── uploadVitals(consultationId, vitals)
│   ├── getVitals(consultationId)
│   └── streamVitals(consultationId) // Real-time
│
└── QueueManager
    ├── addToQueue(consultationId, priority)
    ├── assignDoctor(consultationId)
    ├── prioritizeQueue()
    └── notifyDoctor(consultationId)
```

#### Data Models

**VideoRoom**
```typescript
interface VideoRoom {
  id: string;
  consultationId: string;
  participants: Participant[];
  status: 'CREATED' | 'ACTIVE' | 'ENDED';
  bandwidthMode: 'LOW' | 'MEDIUM' | 'HIGH';
  recordingEnabled: boolean;
  createdAt: Date;
}

interface Participant {
  userId: string;
  role: 'PATIENT' | 'SAHAYAK' | 'DOCTOR';
  connectionStatus: 'CONNECTING' | 'CONNECTED' | 'DISCONNECTED';
  videoEnabled: boolean;
  audioEnabled: boolean;
  joinedAt: Date;
}
```

#### Bandwidth Optimization Strategy

**Low Bandwidth (< 512 Kbps)**:
- Video: 240p, 15fps
- Audio: Opus codec, 32kbps
- Fallback to audio-only if < 256 Kbps

**Medium Bandwidth (512 Kbps - 1 Mbps)**:
- Video: 360p, 20fps
- Audio: Opus codec, 64kbps

**High Bandwidth (> 1 Mbps)**:
- Video: 480p, 30fps
- Audio: Opus codec, 128kbps

**Adaptive Algorithm**:
1. Monitor connection quality every 5 seconds
2. Adjust quality based on packet loss and latency
3. Notify participants of quality changes
4. Auto-downgrade if quality drops below threshold

#### IoT Device Integration

**Supported Devices**:
- BP Monitor (Bluetooth/BLE)
- Pulse Oximeter (Bluetooth/BLE)
- Digital Thermometer (Bluetooth/BLE)

**Integration Flow**:
1. Sahayak pairs device via app
2. Device sends data via BLE
3. App validates and formats data
4. Upload to server via WebSocket
5. Server broadcasts to doctor's screen

**Vitals Data Format**:
```typescript
interface VitalsData {
  deviceType: 'BP_MONITOR' | 'OXIMETER' | 'THERMOMETER';
  deviceId: string;
  readings: {
    timestamp: Date;
    value: number;
    unit: string;
  }[];
  quality: 'GOOD' | 'FAIR' | 'POOR';
}
```

---

### F2: AI-Powered Triage & Translation

#### Component Architecture

```
AITriageService
├── TranslationEngine
│   ├── detectLanguage(text/audio)
│   ├── translateToEnglish(text, sourceLanguage)
│   └── normalizeText(text)
│
├── SymptomAnalyzer
│   ├── extractSymptoms(translatedText)
│   ├── classifySeverity(symptoms)
│   └── generateSummary(symptoms, severity)
│
├── TriageEngine
│   ├── calculateSeverityScore(symptoms, vitals)
│   ├── recommendPriority(severityScore)
│   └── suggestTriage(priority, symptoms)
│
└── LLMService
    ├── callLLM(prompt, context)
    ├── formatMedicalPrompt(symptoms)
    └── parseMedicalResponse(response)
```

#### Processing Pipeline

```
Input (Voice/Text in Dialect)
    ↓
[Language Detection]
    ↓
[Speech-to-Text] (if voice)
    ↓
[Translation to English]
    ↓
[Medical Symptom Extraction] (LLM)
    ↓
[Severity Classification] (ML Model)
    ↓
[Medical Summary Generation] (LLM)
    ↓
[Priority Assignment]
    ↓
Output (Structured Medical Data)
```

#### LLM Prompts

**Symptom Extraction Prompt**:
```
You are a medical assistant. Extract and structure the following patient symptoms:

Patient Description: {translatedText}

Provide:
1. Primary symptoms (list)
2. Secondary symptoms (list)
3. Duration of symptoms
4. Severity (1-10 scale)
5. Possible conditions (list)
6. Urgency level (LOW/MEDIUM/HIGH/CRITICAL)
```

**Medical Summary Prompt**:
```
Create a concise medical summary for a doctor:

Symptoms: {extractedSymptoms}
Vitals: {vitalsData}
Duration: {duration}
Patient Age: {age}
Gender: {gender}

Format as:
- Chief Complaint
- History of Present Illness
- Review of Systems
- Assessment
- Plan
```

#### Severity Scoring Model

**Factors**:
- Symptom intensity (1-10)
- Symptom duration
- Vital signs deviation
- Pain level (if applicable)
- Functional impact

**Scoring Algorithm**:
```
severityScore = (
  symptomIntensity * 0.3 +
  durationFactor * 0.2 +
  vitalsDeviation * 0.3 +
  painLevel * 0.1 +
  functionalImpact * 0.1
) * 10
```

**Priority Mapping**:
- 0-3: LOW
- 4-6: MEDIUM
- 7-8: HIGH
- 9-10: CRITICAL

#### Supported Languages

**Phase 1** (MVP):
- Hindi
- English
- Telugu
- Tamil

**Phase 2**:
- Bengali
- Marathi
- Gujarati
- Kannada
- Malayalam
- Punjabi

---

### F3: Prescription & Fulfillment Engine

#### Component Architecture

```
PrescriptionService
├── PrescriptionController
│   ├── POST /api/v1/prescriptions
│   ├── GET /api/v1/prescriptions/:id
│   ├── GET /api/v1/prescriptions/patient/:patientId
│   └── POST /api/v1/prescriptions/:id/refill
│
├── PharmacyIntegration
│   ├── checkAvailability(medicineId, quantity)
│   ├── getPharmacyList(location, medicineIds)
│   ├── placeOrder(orderDetails)
│   └── trackOrder(orderId)
│
├── OrderManager
│   ├── createOrder(prescriptionId, pharmacyId)
│   ├── updateOrderStatus(orderId, status)
│   └── getOrderHistory(patientId)
│
└── NotificationService
    ├── sendPrescription(prescriptionId, channel)
    ├── sendOrderUpdate(orderId, status)
    └── sendReminder(prescriptionId, type)
```

#### Prescription Data Model

```typescript
interface Prescription {
  id: string;
  consultationId: string;
  patientId: string;
  doctorId: string;
  medicines: Medicine[];
  instructions: string;
  followUpDate: Date | null;
  status: 'ACTIVE' | 'COMPLETED' | 'CANCELLED';
  createdAt: Date;
  digitalSignature: string; // For tamper-proofing
}

interface Medicine {
  name: string;
  genericName: string;
  dosage: string; // e.g., "500mg"
  frequency: string; // e.g., "Twice daily"
  duration: string; // e.g., "7 days"
  quantity: number;
  instructions: string; // e.g., "After meals"
  available: boolean;
  pharmacyPrice: number;
}
```

#### Pharmacy Integration

**Pharmacy Partner API Contract**:
```typescript
interface PharmacyAPI {
  // Check medicine availability
  checkAvailability(request: {
    medicineIds: string[];
    location: { lat: number; lng: number; };
    radius: number; // km
  }): Promise<PharmacyAvailability[]>;
  
  // Place order
  placeOrder(order: {
    prescriptionId: string;
    medicines: MedicineOrder[];
    deliveryAddress: Address;
    patientId: string;
  }): Promise<OrderResponse>;
  
  // Track order
  trackOrder(orderId: string): Promise<OrderStatus>;
}

interface PharmacyAvailability {
  pharmacyId: string;
  pharmacyName: string;
  distance: number; // km
  medicines: {
    medicineId: string;
    available: boolean;
    price: number;
    estimatedDelivery: Date;
  }[];
}
```

#### Order Status Flow

```
PENDING
  ↓
CONFIRMED (Pharmacy confirms)
  ↓
PREPARING (Pharmacy preparing order)
  ↓
DISPATCHED (Out for delivery)
  ↓
DELIVERED
  ↓
COMPLETED
```

**Alternative Flow**:
```
PENDING
  ↓
CANCELLED (by patient/pharmacy)
```

---

### F4: Offline Sync & Storage

#### Component Architecture

```
OfflineSyncService
├── LocalStorageManager
│   ├── save(entityType, entity)
│   ├── find(entityType, id)
│   ├── findAll(entityType, filters)
│   └── delete(entityType, id)
│
├── SyncQueueManager
│   ├── enqueue(operation)
│   ├── dequeue()
│   ├── getPendingCount()
│   └── clearSynced()
│
├── SyncEngine
│   ├── sync()
│   ├── syncEntity(entityType, entityId)
│   ├── detectConflict(local, remote)
│   └── resolveConflict(conflict)
│
└── ConflictResolver
    ├── resolveByTimestamp(local, remote)
    ├── resolveByVersion(local, remote)
    └── flagForManualResolution(conflict)
```

#### Sync Queue Structure

```typescript
interface SyncOperation {
  id: string;
  entityType: 'CONSULTATION' | 'VITALS' | 'PRESCRIPTION' | 'PATIENT' | 'CONSENT';
  entityId: string;
  operation: 'CREATE' | 'UPDATE' | 'DELETE';
  payload: any;
  localVersion: number;
  remoteVersion: number | null;
  deviceId: string;
  timestamp: Date;
  status: 'PENDING' | 'SYNCING' | 'SYNCED' | 'FAILED' | 'CONFLICT';
  retryCount: number;
  lastError?: string;
  syncedAt?: Date;
}
```

#### Conflict Resolution Strategies

**Strategy 1: Last Write Wins (Timestamp)**
- Compare `updatedAt` timestamps
- Keep the most recent version
- Use for: Non-critical data (preferences, settings)

**Strategy 2: Version-Based**
- Compare version numbers
- Merge if versions are compatible
- Use for: Structured data (patient info)

**Strategy 3: Manual Resolution**
- Flag conflict for user review
- Store both versions
- Use for: Critical data (prescriptions, vitals)

**Strategy 4: Field-Level Merge**
- Merge non-conflicting fields
- Flag conflicting fields
- Use for: Complex entities (consultations)

#### Sync Triggers

1. **Automatic**:
   - On connectivity restore
   - Periodic (every 5 minutes when online)
   - After local write operation (if online)

2. **Manual**:
   - User-triggered sync button
   - Pull-to-refresh gesture

3. **Priority-Based**:
   - Critical operations first (prescriptions, vitals)
   - Then regular operations
   - Finally, background sync (logs, analytics)

#### Local Database Schema (SQLite)

```sql
-- Entities table (generic)
CREATE TABLE entities (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL,
  data TEXT NOT NULL, -- JSON
  version INTEGER DEFAULT 1,
  updated_at INTEGER NOT NULL,
  synced_at INTEGER,
  INDEX idx_type (type),
  INDEX idx_synced (synced_at)
);

-- Sync queue
CREATE TABLE sync_queue (
  id TEXT PRIMARY KEY,
  entity_type TEXT NOT NULL,
  entity_id TEXT NOT NULL,
  operation TEXT NOT NULL,
  payload TEXT NOT NULL, -- JSON
  status TEXT DEFAULT 'PENDING',
  retry_count INTEGER DEFAULT 0,
  created_at INTEGER NOT NULL,
  INDEX idx_status (status)
);

-- Conflict log
CREATE TABLE conflicts (
  id TEXT PRIMARY KEY,
  entity_type TEXT NOT NULL,
  entity_id TEXT NOT NULL,
  local_data TEXT NOT NULL, -- JSON
  remote_data TEXT NOT NULL, -- JSON
  resolution_strategy TEXT,
  resolved_at INTEGER,
  created_at INTEGER NOT NULL
);
```

---

### F5: Consent & Privacy (DPDP Act Compliance)

#### Component Architecture

```
ConsentPrivacyService
├── ConsentManager
│   ├── requestConsent(patientId, consentType, grantedTo)
│   ├── verifyConsent(consentId, method, token)
│   ├── revokeConsent(consentId)
│   └── getConsentStatus(patientId, consentType)
│
├── AccessControl
│   ├── checkPermission(userId, resource, action)
│   ├── enforceRBAC(userId, resource)
│   └── auditAccess(userId, resource, action)
│
├── EncryptionService
│   ├── encryptData(data, keyId)
│   ├── decryptData(encryptedData, keyId)
│   └── rotateKeys()
│
└── AuditLogger
    ├── logAccess(userId, resource, action, result)
    ├── logConsent(consentId, action)
    └── generateAuditReport(filters)
```

#### Consent Types

```typescript
enum ConsentType {
  DATA_SHARING = 'DATA_SHARING', // Share data with doctor
  RECORDING = 'RECORDING', // Record consultation
  PHARMACY_SHARING = 'PHARMACY_SHARING', // Share prescription with pharmacy
  GOVT_SCHEME = 'GOVT_SCHEME', // Share with government schemes
  RESEARCH = 'RESEARCH', // Use data for research (anonymized)
  ANALYTICS = 'ANALYTICS' // Use for analytics (anonymized)
}
```

#### Verification Methods

**OTP Verification**:
1. Generate 6-digit OTP
2. Send via SMS
3. Patient enters OTP
4. Verify and grant consent
5. Expires in 10 minutes

**Biometric Verification**:
1. Request biometric (fingerprint/face)
2. Capture biometric data
3. Verify against stored template
4. Grant consent if match
5. Store verification record

**Signature Verification**:
1. Display consent terms
2. Patient signs on screen
3. Store signature image
4. Grant consent
5. Signature is legally binding

#### RBAC Model

```typescript
enum Role {
  PATIENT = 'PATIENT',
  SAHAYAK = 'SAHAYAK',
  DOCTOR = 'DOCTOR',
  ADMIN = 'ADMIN',
  PHARMACY = 'PHARMACY',
  GOVT_SCHEME = 'GOVT_SCHEME'
}

interface Permission {
  resource: string; // e.g., 'patient.data', 'prescription'
  actions: string[]; // e.g., ['read', 'write']
  conditions?: { // Optional conditions
    patientId?: string; // Only for specific patient
    activeOnly?: boolean; // Only active consultations
  };
}

const RolePermissions: Record<Role, Permission[]> = {
  PATIENT: [
    { resource: 'own.data', actions: ['read', 'write'] },
    { resource: 'own.prescription', actions: ['read'] }
  ],
  SAHAYAK: [
    { resource: 'patient.data', actions: ['read'], conditions: { activeOnly: true } },
    { resource: 'vitals', actions: ['create'] }
  ],
  DOCTOR: [
    { resource: 'patient.data', actions: ['read'], conditions: { patientId: 'assigned' } },
    { resource: 'prescription', actions: ['create', 'read'] }
  ],
  // ... other roles
};
```

#### Access Control Flow

```
Access Request
    ↓
[Check User Role]
    ↓
[Check Resource Permissions]
    ↓
[Check Consent] (if required)
    ↓
[Check Conditions] (activeOnly, patientId, etc.)
    ↓
[Log Access] (audit trail)
    ↓
Grant/Deny Access
```

---

### F6: Visual Diagnosis Aid

#### Component Architecture

```
VisualDiagnosisService
├── ImageProcessor
│   ├── uploadImage(file, patientId)
│   ├── validateImage(file)
│   ├── compressImage(image)
│   └── storeImage(imageId, url)
│
├── CVModel
│   ├── analyzeImage(imageUrl)
│   ├── classifyCondition(image)
│   ├── calculateConfidence(predictions)
│   └── generateReport(analysis)
│
└── ReportGenerator
    ├── createReport(analysis, patientInfo)
    ├── suggestFollowUp(condition)
    └── compareImages(before, after)
```

#### Supported Conditions

**Phase 1**:
- Skin rashes
- Wounds (healing assessment)
- Burns
- Infections
- Allergic reactions

**Phase 2**:
- Dermatitis
- Psoriasis
- Eczema
- Fungal infections
- Vitiligo

#### CV Model Output

```typescript
interface ImageAnalysis {
  imageId: string;
  conditions: ConditionPrediction[];
  confidence: number; // Overall confidence (0-1)
  recommendations: string[];
  severity: 'MILD' | 'MODERATE' | 'SEVERE';
  requiresUrgentCare: boolean;
}

interface ConditionPrediction {
  condition: string;
  confidence: number; // 0-1
  description: string;
  treatmentSuggestions: string[];
}
```

#### Image Requirements

- **Format**: JPG, PNG
- **Size**: Max 10MB
- **Resolution**: Min 640x480
- **Compression**: Automatic compression to < 2MB
- **Privacy**: Images encrypted at rest

---

### F7: Government Scheme Integration

#### Component Architecture

```
GovtSchemeService
├── ABHAService
│   ├── validateABHA(abhaId)
│   ├── verifyIdentity(abhaId, otp)
│   └── fetchHealthRecords(abhaId)
│
├── EligibilityService
│   ├── checkEligibility(abhaId, scheme)
│   ├── getCoverageDetails(abhaId)
│   └── calculateCoPayment(consultation, coverage)
│
├── ClaimService
│   ├── submitClaim(consultationId, abhaId)
│   ├── trackClaim(claimId)
│   └── getClaimHistory(abhaId)
│
└── CacheManager
    ├── cacheEligibility(abhaId, data, ttl)
    ├── getCachedEligibility(abhaId)
    └── invalidateCache(abhaId)
```

#### ABHA Integration

**API Endpoints** (Government):
```
POST /abha/validate
  Request: { abhaId: string }
  Response: { valid: boolean, name: string }

POST /abha/verify
  Request: { abhaId: string, otp: string }
  Response: { verified: boolean, token: string }

GET /abha/health-records
  Headers: { Authorization: Bearer <token> }
  Response: { records: HealthRecord[] }
```

#### Eligibility Check Flow

```
Patient provides ABHA ID
    ↓
[Validate ABHA ID]
    ↓
[Check Cache] (if cached and valid)
    ↓
[Call Govt API] (if not cached)
    ↓
[Check Eligibility]
    ↓
[Calculate Coverage]
    ↓
[Cache Result] (24 hours)
    ↓
[Display to Patient/Doctor]
```

#### Supported Schemes

**Phase 1**:
- Ayushman Bharat Pradhan Mantri Jan Arogya Yojana (AB-PMJAY)

**Phase 2**:
- State-specific health schemes
- Employee State Insurance (ESI)
- Central Government Health Scheme (CGHS)

---

### F8: WhatsApp Bot for Follow-ups

#### Component Architecture

```
WhatsAppBotService
├── MessageHandler
│   ├── receiveMessage(phoneNumber, message)
│   ├── sendMessage(phoneNumber, message)
│   ├── sendReminder(phoneNumber, type, data)
│   └── handleQuery(phoneNumber, query)
│
├── ReminderScheduler
│   ├── scheduleReminder(patientId, type, time)
│   ├── cancelReminder(reminderId)
│   └── getUpcomingReminders(patientId)
│
├── NLPEngine
│   ├── understandIntent(message)
│   ├── extractEntities(message)
│   └── generateResponse(intent, context)
│
└── AppointmentManager
    ├── bookAppointment(patientId, preferredTime)
    ├── confirmAppointment(appointmentId)
    └── cancelAppointment(appointmentId)
```

#### Reminder Types

```typescript
enum ReminderType {
  MEDICATION = 'MEDICATION', // Take medicine
  APPOINTMENT = 'APPOINTMENT', // Upcoming appointment
  FOLLOW_UP = 'FOLLOW_UP', // Follow-up consultation
  TEST = 'TEST', // Lab test reminder
  HEALTH_TIP = 'HEALTH_TIP' // Preventive care tips
}
```

#### Message Templates

**Medication Reminder** (Hindi):
```
नमस्ते {patientName},

यह आपकी दवा का समय है:
- दवा: {medicineName}
- मात्रा: {dosage}
- समय: {time}

कृपया दवा लें और "हाँ" लिखकर पुष्टि करें।

धन्यवाद,
GramSeva Health
```

**Appointment Reminder** (English):
```
Hello {patientName},

Your appointment is scheduled for:
- Date: {date}
- Time: {time}
- Doctor: {doctorName}

Reply "CONFIRM" to confirm or "RESCHEDULE" to change.

Thank you,
GramSeva Health
```

#### NLP Intent Classification

**Intents**:
- `MEDICATION_QUERY`: Questions about medicines
- `APPOINTMENT_BOOKING`: Book/reschedule appointment
- `SYMPTOM_QUERY`: Ask about symptoms
- `PRESCRIPTION_QUERY`: Questions about prescription
- `GENERAL_QUERY`: General health questions
- `GREETING`: Greetings
- `THANK_YOU`: Thank you messages

**Entity Extraction**:
- Medicine names
- Dates/times
- Symptoms
- Appointment IDs

---

## API Specifications

### Common API Patterns

#### Authentication
All APIs require JWT token in header:
```
Authorization: Bearer <token>
```

#### Response Format
```typescript
interface APIResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  timestamp: Date;
}
```

#### Pagination
```typescript
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    total: number;
    totalPages: number;
  };
}
```

#### Error Codes
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden (no consent/permission)
- `404`: Not Found
- `409`: Conflict (sync conflict)
- `429`: Rate Limited
- `500`: Internal Server Error
- `503`: Service Unavailable

---

## Data Flow Diagrams

### Consultation Flow (Detailed)

```
[Patient] → [Sahayak App]
    ↓
[Sahayak registers/authenticates patient]
    ↓
[Patient describes symptoms] → [AI Service]
    ↓
[AI translates & analyzes] → [Severity Score]
    ↓
[Consultation created] → [Queue Service]
    ↓
[Doctor notified] → [Doctor App]
    ↓
[Doctor accepts] → [Video Room created]
    ↓
[All parties join] → [Media Server]
    ↓
[Sahayak uploads vitals] → [Real-time sync]
    ↓
[Doctor sees vitals] → [Consultation proceeds]
    ↓
[Doctor generates prescription] → [Prescription Service]
    ↓
[Prescription sent to patient] → [Notification Service]
    ↓
[Patient orders medicines] → [Pharmacy Service]
    ↓
[Order tracked] → [WhatsApp reminders scheduled]
```

---

## Error Handling

### Error Categories

1. **Network Errors**:
   - Connection timeout → Retry with exponential backoff
   - No connectivity → Queue for offline sync
   - Slow connection → Degrade quality

2. **Authentication Errors**:
   - Token expired → Refresh token
   - Invalid token → Redirect to login
   - No permission → Show error, log access attempt

3. **Data Errors**:
   - Validation failure → Show field-specific errors
   - Conflict → Trigger conflict resolution
   - Not found → Show user-friendly message

4. **Service Errors**:
   - External API failure → Use circuit breaker
   - Service unavailable → Show maintenance message
   - Rate limited → Queue request, retry later

### Error Recovery Strategies

- **Automatic Retry**: Network errors, transient failures
- **Manual Retry**: User-triggered for failed operations
- **Graceful Degradation**: Fallback to basic features
- **Offline Mode**: Queue operations, sync later
- **User Notification**: Clear error messages with actions

---

## Feature Implementation Roadmap

### Phase 0: Foundation (Weeks 1-4)

**Goal**: Establish core infrastructure and foundational features

#### Features
- ✅ **F4: Offline Sync & Storage** (Weeks 1-2)
- ✅ **F5: Consent & Privacy** (Weeks 2-3)
- ✅ **Infrastructure Setup** (Weeks 1-4)
  - Database setup
  - API Gateway
  - Authentication service
  - Logging & monitoring
  - CI/CD pipeline

#### Deliverables
- Local database (SQLite) with sync capability
- RBAC system
- Consent management system
- Basic authentication & authorization
- Infrastructure monitoring

#### Success Criteria
- ✅ Data can be stored offline and synced when online
- ✅ Access control enforced for all endpoints
- ✅ Consent can be requested, verified, and revoked
- ✅ System is monitored and logged

---

### Phase 1: Core Consultation (Weeks 5-10)

**Goal**: Enable basic tele-consultation functionality

#### Features
- ✅ **F2: AI Triage & Translation** (Weeks 5-7)
  - Language detection
  - Translation service
  - Symptom extraction
  - Severity scoring
- ✅ **F1: Assisted Tele-Consultation** (Weeks 7-10)
  - Video call setup
  - Multi-party video
  - Vitals upload
  - Basic bandwidth optimization

#### Dependencies
- Requires: F4 (Offline Sync), F5 (Consent)
- Enables: F3 (Prescription)

#### Deliverables
- AI translation service (10 languages)
- Video consultation platform
- Real-time vitals display
- Consultation queue system

#### Success Criteria
- ✅ Symptoms can be translated from local language to English
- ✅ Video consultation works with 256 Kbps bandwidth
- ✅ Vitals appear on doctor's screen in real-time
- ✅ Consultations are prioritized by severity

---

### Phase 2: Prescription & Fulfillment (Weeks 11-14)

**Goal**: Complete the consultation-to-medicine delivery flow

#### Features
- ✅ **F3: Prescription & Fulfillment** (Weeks 11-14)
  - Digital prescription generation
  - Pharmacy integration
  - Order management
  - Delivery tracking

#### Dependencies
- Requires: F1 (Tele-Consultation), F4 (Offline Sync), F5 (Consent)
- Enables: F8 (WhatsApp Bot)

#### Deliverables
- Prescription generation service
- Pharmacy partner integrations (3-5 partners)
- Order tracking system
- Notification service

#### Success Criteria
- ✅ Prescriptions generated within 2 seconds
- ✅ Medicine availability checked in real-time
- ✅ Orders can be tracked end-to-end
- ✅ At least 3 pharmacy partners integrated

---

### Phase 3: Enhanced Features (Weeks 15-20)

**Goal**: Add value-added features to improve diagnosis and access

#### Features
- ✅ **F6: Visual Diagnosis Aid** (Weeks 15-17)
  - Image upload & processing
  - CV model integration
  - Condition classification
  - Report generation
- ✅ **F7: Government Scheme Integration** (Weeks 17-20)
  - ABHA validation
  - Eligibility checking
  - Health record fetching
  - Claim submission

#### Dependencies
- F6 requires: F1 (Tele-Consultation), F4 (Offline Sync), F5 (Consent)
- F7 requires: F4 (Offline Sync), F5 (Consent)

#### Deliverables
- Image analysis service
- CV model for skin conditions
- ABHA integration
- Government API integration layer

#### Success Criteria
- ✅ Images analyzed within 10 seconds
- ✅ Condition identification >80% accurate
- ✅ ABHA validation works offline (cached)
- ✅ Eligibility checked in real-time

---

### Phase 4: Follow-up & Engagement (Weeks 21-24)

**Goal**: Improve patient engagement and adherence

#### Features
- ✅ **F8: WhatsApp Bot for Follow-ups** (Weeks 21-24)
  - Reminder scheduling
  - Two-way communication
  - Appointment booking
  - Medication adherence tracking

#### Dependencies
- Requires: F3 (Prescription)

#### Deliverables
- WhatsApp Business API integration
- NLP engine for queries
- Reminder scheduler
- Appointment booking via WhatsApp

#### Success Criteria
- ✅ Reminders sent at scheduled times
- ✅ Bot responds within 2 seconds
- ✅ Supports 5+ Indian languages
- ✅ Appointments can be booked via WhatsApp

---

## Feature Dependency Timeline

```
Week 1-4:  Foundation
  F4 ──────┐
  F5 ──────┼───┐
            │   │
Week 5-10:  Core Consultation
  F2 ──────┘   │
  F1 ──────────┼───┐
                │   │
Week 11-14: Prescription
  F3 ──────────┘   │
                    │
Week 15-20: Enhanced Features
  F6 ───────────────┼───┐
  F7 ───────────────┘   │
                        │
Week 21-24: Follow-up
  F8 ───────────────────┘
```

---

## Resource Allocation

### Team Structure

**Phase 0-1 (Foundation & Core)**:
- 2 Backend Engineers
- 1 Frontend Engineer (Mobile)
- 1 Frontend Engineer (Web)
- 1 DevOps Engineer
- 1 AI/ML Engineer
- 1 QA Engineer

**Phase 2 (Prescription)**:
- 2 Backend Engineers
- 1 Integration Engineer (Pharmacy APIs)
- 1 Frontend Engineer
- 1 QA Engineer

**Phase 3 (Enhanced Features)**:
- 1 Backend Engineer
- 1 AI/ML Engineer (CV)
- 1 Integration Engineer (Govt APIs)
- 1 QA Engineer

**Phase 4 (Follow-up)**:
- 1 Backend Engineer
- 1 NLP Engineer
- 1 Integration Engineer (WhatsApp)
- 1 QA Engineer

---

## Risk Mitigation

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Video quality issues in low bandwidth | High | Medium | Implement aggressive bandwidth optimization, audio-only fallback |
| Translation accuracy | High | Medium | Use multiple translation services, human review for critical cases |
| Offline sync conflicts | Medium | High | Implement robust conflict resolution, manual resolution UI |
| Government API reliability | Medium | High | Implement caching, circuit breakers, fallback mechanisms |
| IoT device compatibility | Medium | Medium | Standardize on BLE, create device adapter layer |

### Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Low doctor adoption | High | Medium | Provide training, simplify UI, offer incentives |
| Patient privacy concerns | High | Low | Transparent consent, strong encryption, audit trails |
| Pharmacy partner integration delays | Medium | Medium | Start with 1-2 partners, expand gradually |
| Regulatory compliance issues | High | Low | Legal review, compliance audits, DPDP Act expert consultation |

---

## Testing Strategy

### Phase 0: Foundation Testing
- Unit tests for all core services
- Integration tests for sync mechanism
- Security testing for access control
- Performance testing for local database

### Phase 1: Core Consultation Testing
- End-to-end consultation flow
- Video call quality under various bandwidth conditions
- Translation accuracy testing (multiple languages)
- Load testing for consultation queue

### Phase 2: Prescription Testing
- Prescription generation accuracy
- Pharmacy API integration testing
- Order tracking reliability
- Notification delivery testing

### Phase 3: Enhanced Features Testing
- CV model accuracy testing
- Image processing performance
- Government API integration testing
- Offline caching for eligibility

### Phase 4: Follow-up Testing
- Reminder delivery reliability
- NLP intent classification accuracy
- WhatsApp API integration
- Multi-language message delivery

---

## Success Metrics by Phase

### Phase 0 (Foundation)
- ✅ 100% of data operations work offline
- ✅ 0 unauthorized access incidents
- ✅ 100% consent verification success rate
- ✅ <100ms access control check latency

### Phase 1 (Core Consultation)
- ✅ 95% video call success rate
- ✅ >85% translation accuracy
- ✅ <5 seconds symptom analysis time
- ✅ 90% consultation completion rate

### Phase 2 (Prescription)
- ✅ 100% prescription generation success
- ✅ <3 seconds medicine availability check
- ✅ 95% order delivery success rate
- ✅ 80% prescription-to-order conversion

### Phase 3 (Enhanced Features)
- ✅ >80% CV model accuracy
- ✅ <10 seconds image analysis time
- ✅ 95% ABHA validation success rate
- ✅ 90% eligibility check accuracy

### Phase 4 (Follow-up)
- ✅ 95% reminder delivery success
- ✅ <2 seconds bot response time
- ✅ 70% medication adherence improvement
- ✅ 60% appointment booking via WhatsApp

---

## Go-Live Readiness Checklist

### Technical Readiness
- [ ] All critical features implemented and tested
- [ ] Performance benchmarks met
- [ ] Security audit completed
- [ ] Load testing passed
- [ ] Disaster recovery plan in place
- [ ] Monitoring and alerting configured
- [ ] Documentation complete

### Business Readiness
- [ ] Doctor onboarding process defined
- [ ] Sahayak training materials ready
- [ ] Pharmacy partnerships established
- [ ] Government scheme approvals obtained
- [ ] Pricing model finalized
- [ ] Support process defined

### Compliance Readiness
- [ ] DPDP Act compliance verified
- [ ] Data privacy policy published
- [ ] Consent management process documented
- [ ] Audit logging verified
- [ ] Data retention policy defined

---

## Post-Launch Enhancements

### Quarter 2 (Months 4-6)
- Advanced analytics dashboard
- Chronic disease management module
- Multi-language expansion (5 more languages)
- Enhanced CV models (more conditions)

### Quarter 3 (Months 7-9)
- Specialized care consultations (cardiology, dermatology)
- Lab test integration
- Health records management
- Family health tracking

### Quarter 4 (Months 10-12)
- AI-powered health recommendations
- Preventive care programs
- Community health initiatives
- Integration with more government schemes

---

This features document provides a comprehensive breakdown of all features, their requirements, dependencies, technical specifications, API details, implementation roadmap, and success metrics. Each feature can be further detailed during the design phase.
