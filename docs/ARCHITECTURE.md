# GramSeva Health - Architecture Documentation

> **⚠️ Core Requirements**: This architecture is designed around 5 critical requirements. See [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md) for the complete reference.

## Table of Contents
1. [High-Level Design (HLD)](#high-level-design-hld)
2. [Requirements-Driven Architecture](#requirements-driven-architecture)
3. [Low-Level Design (LLD)](#low-level-design-lld)
4. [Design Patterns](#design-patterns)
5. [System Components](#system-components)
6. [Data Flow](#data-flow)
7. [Security Considerations](#security-considerations)
8. [Scalability & Performance](#scalability--performance)
9. [Monitoring & Observability](#monitoring--observability)

---

## High-Level Design (HLD)

### System Overview

GramSeva Health is a **rural telemedicine platform** designed to bridge the healthcare gap in Tier-2/3 cities and villages through an assisted kiosk model with offline-first capabilities.

### Architecture Principles

1. **Offline-First Architecture**: Store-and-forward pattern for unreliable connectivity
2. **Low-Bandwidth Optimization**: Video compression, adaptive streaming, data prioritization
3. **Assisted Model**: Sahayak (health worker) as intermediary between patient and doctor
4. **Privacy & Compliance**: DPDP Act compliance with role-based access control
5. **Scalability**: Microservices architecture for horizontal scaling
6. **Resilience**: Circuit breakers, retry mechanisms, graceful degradation

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Kiosk App    │  │ Sahayak App  │  │ Doctor App   │         │
│  │ (Patient UI) │  │ (Assisted)   │  │ (Desktop)    │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
│         │                  │                  │                  │
└─────────┼──────────────────┼──────────────────┼─────────────────┘
          │                  │                  │
          │                  │                  │
┌─────────┼──────────────────┼──────────────────┼─────────────────┐
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │           API Gateway / Load Balancer              │         │
│  │         (Authentication, Rate Limiting)            │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │              Microservices Layer                   │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Tele-      │ │ AI Triage  │ │ Prescription│   │         │
│  │ │ Consultation│ │ & Translation│ │ & Fulfillment│ │         │
│  │ │ Service    │ │ Service    │ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Patient    │ │ Consent    │ │ Offline    │    │         │
│  │ │ Management │ │ & Privacy  │ │ Sync       │    │         │
│  │ │ Service    │ │ Service    │ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Visual     │ │ Govt       │ │ WhatsApp   │    │         │
│  │ │ Diagnosis  │ │ Scheme     │ │ Bot        │    │         │
│  │ │ Service    │ │ Integration│ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │         Message Queue (Event Bus)                  │         │
│  │    (Kafka/RabbitMQ for async communication)       │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │              Data Layer                            │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Primary    │ │ Offline    │ │ Cache      │    │         │
│  │ │ Database   │ │ Local DB   │ │ (Redis)    │    │         │
│  │ │ (PostgreSQL│ │ (SQLite)   │ │            │    │         │
│  │ │ /MongoDB)  │ │            │ │            │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  └───────────────────────────────────────────────────┘         │
│                                                                  │
│  ┌───────────────────────────────────────────────────┐         │
│  │         External Services Integration              │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Ayushman   │ │ Pharmacy   │ │ IoT Device │    │         │
│  │ │ Bharat API │ │ Supply API │ │ Gateway    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  └───────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Key Components

#### 1. **Kiosk Application (Patient Interface)**
- **Purpose**: Patient-facing interface at kiosk locations
- **Technology**: React Native / Flutter (offline-capable)
- **Features**: 
  - Patient registration/login
  - Symptom input (voice/text in local dialect)
  - Video consultation interface
  - Prescription viewing
  - Appointment scheduling

#### 2. **Sahayak Application (Health Worker Interface)**
- **Purpose**: Assisted interface for health workers
- **Technology**: React Native / Flutter
- **Features**:
  - Patient onboarding
  - IoT device integration (BP monitor, Oximeter)
  - Vitals upload
  - Consultation assistance
  - Offline data capture

#### 3. **Doctor Application (Desktop/Web)**
- **Purpose**: Doctor-facing consultation interface
- **Technology**: React / Angular (WebRTC for video)
- **Features**:
  - Patient queue management
  - Video consultation
  - AI-translated symptom summaries
  - Prescription generation
  - Patient history viewing

#### 4. **Tele-Consultation Service**
- **Purpose**: Video consultation orchestration
- **Technology**: WebRTC, Janus/Kurento media server
- **Features**:
  - Low-bandwidth video optimization
  - Multi-party video (Patient, Sahayak, Doctor)
  - Screen sharing
  - Recording (with consent)

#### 5. **AI Triage & Translation Service**
- **Purpose**: Symptom analysis and translation
- **Technology**: LLM (GPT-4/Claude), Speech-to-Text, Translation APIs
- **Features**:
  - Dialect-to-English translation
  - Medical symptom summarization
  - Severity scoring
  - Triage recommendations

#### 6. **Prescription & Fulfillment Service**
- **Purpose**: Prescription management and pharmacy integration
- **Technology**: REST APIs, Pharmacy partner APIs
- **Features**:
  - Digital prescription generation
  - Medicine availability check
  - Order placement
  - Delivery tracking

#### 7. **Offline Sync Service**
- **Purpose**: Store-and-forward data synchronization
- **Technology**: Background sync, Conflict resolution
- **Features**:
  - Local data caching
  - Conflict detection and resolution
  - Incremental sync
  - Sync status tracking

#### 8. **Consent & Privacy Service**
- **Purpose**: DPDP Act compliance and access control
- **Technology**: OAuth 2.0, RBAC, Encryption
- **Features**:
  - Role-based access control
  - Consent management (OTP/Biometric)
  - Data encryption at rest and in transit
  - Audit logging

#### 9. **Visual Diagnosis Service**
- **Purpose**: Computer vision for skin conditions/wounds
- **Technology**: TensorFlow/PyTorch, Image processing
- **Features**:
  - Image upload and analysis
  - Condition classification
  - Severity assessment
  - Report generation

#### 10. **Government Scheme Integration Service**
- **Purpose**: Ayushman Bharat integration
- **Technology**: REST APIs, ABHA ID verification
- **Features**:
  - ABHA ID validation
  - Insurance eligibility check
  - Health record fetching
  - Scheme benefit verification

#### 11. **WhatsApp Bot Service**
- **Purpose**: Follow-up reminders and communication
- **Technology**: WhatsApp Business API, NLP
- **Features**:
  - Medication reminders (vernacular)
  - Appointment scheduling
  - Follow-up queries
  - Health tips

---

## Requirements-Driven Architecture

> This section presents the architecture **specifically designed to fulfill the 5 core requirements**. Each requirement is mapped to its architectural components, data flows, and technology stack.

### Architecture Overview: 5 Core Requirements

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    REQUIREMENT-DRIVEN ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  R1: Assisted Tele-Consultation    R2: AI Triage & Translation           │
│  ┌──────────────────────────┐    ┌──────────────────────────┐          │
│  │ • WebRTC Media Server    │    │ • LLM Service            │          │
│  │ • IoT Device Gateway      │    │ • Speech-to-Text         │          │
│  │ • Vitals Manager         │    │ • Translation Engine     │          │
│  │ • Bandwidth Optimizer    │    │ • Severity Scorer        │          │
│  └──────────────────────────┘    └──────────────────────────┘          │
│           │                                    │                          │
│           └────────────┬───────────────────────┘                          │
│                        │                                                    │
│  R3: Prescription & Fulfillment                                            │
│  ┌──────────────────────────┐                                            │
│  │ • Prescription Service   │                                            │
│  │ • Pharmacy Integration   │                                            │
│  │ • Order Manager          │                                            │
│  └──────────────────────────┘                                            │
│           │                                                                │
│           │                                                                │
│  ┌────────▼──────────────────────────────────────────┐                   │
│  │         FOUNDATION LAYER                          │                   │
│  ├──────────────────────────────────────────────────┤                   │
│  │  R4: Offline Sync & Storage                       │                   │
│  │  ┌──────────────────────────────────────────┐    │                   │
│  │  │ • Local SQLite DB                        │    │                   │
│  │  │ • Sync Queue Manager                     │    │                   │
│  │  │ • Conflict Resolver                      │    │                   │
│  │  │ • Background Sync Service               │    │                   │
│  │  └──────────────────────────────────────────┘    │                   │
│  │                                                   │                   │
│  │  R5: Consent & Privacy (DPDP Act)                │                   │
│  │  ┌──────────────────────────────────────────┐    │                   │
│  │  │ • RBAC Service                           │    │                   │
│  │  │ • Consent Manager                       │    │                   │
│  │  │ • Access Control Middleware              │    │                   │
│  │  │ • Audit Logger                          │    │                   │
│  │  │ • Encryption Service                    │    │                   │
│  │  └──────────────────────────────────────────┘    │                   │
│  └──────────────────────────────────────────────────┘                   │
│                                                                           │
└─────────────────────────────────────────────────────────────────────────┘
```

### R1: Assisted Tele-Consultation Platform

#### Architecture Components

```
┌─────────────────────────────────────────────────────────────────┐
│              R1: TELE-CONSULTATION ARCHITECTURE                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐         ┌──────────────┐         ┌──────────┐ │
│  │  Patient App  │         │ Sahayak App  │         │Doctor App│ │
│  │  (Kiosk)      │         │  (Mobile)    │         │ (Desktop)│ │
│  └──────┬───────┘         └──────┬───────┘         └────┬─────┘ │
│         │                         │                       │       │
│         │ Video Stream            │ Vitals Upload         │       │
│         │ (Low Bandwidth)         │ (IoT Devices)          │       │
│         │                         │                       │       │
│  ┌──────▼─────────────────────────▼───────────────────────▼─────┐ │
│  │              Tele-Consultation Service                        │ │
│  ├──────────────────────────────────────────────────────────────┤ │
│  │  ┌──────────────────┐  ┌──────────────────┐                │ │
│  │  │  Media Manager    │  │  Vitals Manager  │                │ │
│  │  │  • WebRTC Setup  │  │  • IoT Gateway   │                │ │
│  │  │  • Bandwidth Opt │  │  • Real-time Sync│                │ │
│  │  │  • Quality Adapt │  │  • Data Format   │                │ │
│  │  └──────────────────┘  └──────────────────┘                │ │
│  │                                                              │ │
│  │  ┌──────────────────┐  ┌──────────────────┐                │ │
│  │  │  Room Manager    │  │  Queue Manager   │                │ │
│  │  │  • Create Room   │  │  • Prioritize    │                │ │
│  │  │  • Join Room    │  │  • Assign Doctor  │                │ │
│  │  │  • Manage Users  │  │  • Notify        │                │ │
│  │  └──────────────────┘  └──────────────────┘                │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │           Media Server (Janus/Kurento)                        │ │
│  │  • Multi-party video routing                                   │ │
│  │  • Adaptive bitrate streaming                                  │ │
│  │  • Low-bandwidth optimization (256 Kbps min)                   │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │           IoT Device Gateway                                  │ │
│  │  • BLE Connection (BP Monitor, Oximeter)                       │ │
│  │  • Data Validation                                            │ │
│  │  • Real-time Transmission to Doctor Screen                    │ │
│  └──────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### Technology Stack

**Frontend**:
- **Patient App**: React Native (offline-capable, video support)
- **Sahayak App**: React Native (BLE for IoT, video, vitals upload)
- **Doctor App**: React/Web (WebRTC, real-time vitals display)

**Backend**:
- **Tele-Consultation Service**: Node.js/Python (WebRTC signaling)
- **Media Server**: Janus or Kurento (multi-party video)
- **IoT Gateway**: Node.js (BLE communication, data processing)

**Infrastructure**:
- **CDN**: CloudFlare (video content delivery)
- **WebRTC TURN/STUN**: Coturn server (NAT traversal)

#### Data Flow

```
1. Patient arrives at kiosk
   ↓
2. Sahayak initiates consultation
   ↓
3. System creates video room (Media Server)
   ↓
4. Sahayak connects IoT devices (BLE)
   ↓
5. Patient, Sahayak, Doctor join video call
   ↓
6. Sahayak uploads vitals → IoT Gateway → Vitals Manager
   ↓
7. Vitals appear on Doctor's screen in real-time
   ↓
8. Consultation proceeds with low-bandwidth optimization
   ↓
9. Doctor can see vitals, patient, and Sahayak simultaneously
```

#### Key Architectural Decisions

1. **Media Server Choice**: Janus/Kurento for multi-party support
2. **Bandwidth Strategy**: Adaptive bitrate (240p-480p based on connection)
3. **IoT Protocol**: BLE (Bluetooth Low Energy) for device communication
4. **Real-time Sync**: WebSocket for vitals transmission
5. **Offline Support**: Vitals cached locally, synced when online (R4)

---

### R2: AI-Powered Triage & Translation

#### Architecture Components

```
┌─────────────────────────────────────────────────────────────────┐
│          R2: AI TRIAGE & TRANSLATION ARCHITECTURE               │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐                                               │
│  │  Patient     │                                               │
│  │  (Voice/Text)│                                               │
│  │  Local Dialect                                               │
│  └──────┬───────┘                                               │
│         │                                                        │
│  ┌──────▼──────────────────────────────────────────────────────┐ │
│  │         AI Triage & Translation Service                      │ │
│  ├──────────────────────────────────────────────────────────────┤ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Translation Engine                                │     │ │
│  │  │  • Language Detection (Auto)                        │     │ │
│  │  │  • Speech-to-Text (if voice)                      │     │ │
│  │  │  • Dialect → English Translation                  │     │ │
│  │  │  • Text Normalization                             │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Symptom Analyzer                                  │     │ │
│  │  │  • Extract Medical Symptoms (LLM)                  │     │ │
│  │  │  • Classify Conditions                             │     │ │
│  │  │  • Generate Medical Summary                        │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Triage Engine                                      │     │ │
│  │  │  • Calculate Severity Score (1-10)                 │     │ │
│  │  │  • Recommend Priority (LOW/MEDIUM/HIGH/CRITICAL)   │     │ │
│  │  │  • Suggest Triage Action                           │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  LLM Service                                       │     │ │
│  │  │  • GPT-4/Claude for Medical Summarization         │     │ │
│  │  │  • Prompt Engineering for Accuracy                 │     │ │
│  │  │  • Response Parsing                               │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  External Services                                            │ │
│  │  • Speech-to-Text API (Google/AWS)                           │ │
│  │  • Translation API (Custom Model/Google)                     │ │
│  │  • LLM API (OpenAI/Anthropic)                               │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Output:                                                       │ │
│  │  • Translated English Medical Summary                         │ │
│  │  • Extracted Symptoms                                         │ │
│  │  • Severity Score (1-10)                                      │ │
│  │  • Triage Priority                                             │ │
│  │  → Sent to Doctor's Screen                                    │ │
│  └──────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### Technology Stack

**AI/ML Services**:
- **Speech-to-Text**: Google Cloud Speech-to-Text / AWS Transcribe
- **Translation**: Custom model (fine-tuned for medical terms) or Google Translate API
- **LLM**: OpenAI GPT-4 / Anthropic Claude (for medical summarization)
- **Severity Scoring**: Custom ML model (TensorFlow/PyTorch)

**Backend**:
- **AI Triage Service**: Python (FastAPI) - optimal for ML/AI workloads
- **Translation Service**: Python (handles multiple language models)
- **Caching**: Redis (cache translations for common symptoms)

#### Processing Pipeline

```
Input: Patient Symptoms (Voice/Text in Local Dialect)
    ↓
[Step 1: Language Detection]
    ↓
[Step 2: Speech-to-Text] (if voice input)
    ↓
[Step 3: Translation to English]
    ↓
[Step 4: Medical Symptom Extraction] (LLM)
    ↓
[Step 5: Severity Classification] (ML Model)
    ↓
[Step 6: Medical Summary Generation] (LLM)
    ↓
[Step 7: Triage Priority Assignment]
    ↓
Output: Structured Medical Data
    • Translated English Summary
    • Extracted Symptoms
    • Severity Score (1-10)
    • Priority (LOW/MEDIUM/HIGH/CRITICAL)
    → Sent to Doctor + Consultation Queue
```

#### Key Architectural Decisions

1. **Hybrid Approach**: Custom models for common symptoms, LLM for complex cases
2. **Caching Strategy**: Cache translations for common symptoms (reduce LLM calls)
3. **Offline Support**: Basic translation works offline (R4), LLM requires online
4. **Language Support**: 10+ Indian languages (Hindi, Telugu, Tamil, Bengali, etc.)
5. **Accuracy Target**: >85% translation accuracy, >80% severity score accuracy

---

### R3: Prescription & Fulfillment Engine

#### Architecture Components

```
┌─────────────────────────────────────────────────────────────────┐
│          R3: PRESCRIPTION & FULFILLMENT ARCHITECTURE             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐                                               │
│  │  Doctor      │                                               │
│  │  (Generates  │                                               │
│  │  Prescription)                                               │
│  └──────┬───────┘                                               │
│         │                                                        │
│  ┌──────▼──────────────────────────────────────────────────────┐ │
│  │         Prescription & Fulfillment Service                   │ │
│  ├──────────────────────────────────────────────────────────────┤ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Prescription Generator                            │     │ │
│  │  │  • Create Digital Prescription                    │     │ │
│  │  │  • Medicine Details (name, dosage, frequency)    │     │ │
│  │  │  • Doctor Instructions                            │     │ │
│  │  │  • Follow-up Date                                 │     │ │
│  │  │  • Digital Signature (tamper-proof)               │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Pharmacy Integration Layer                        │     │ │
│  │  │  • Check Medicine Availability                     │     │ │
│  │  │  • Get Pharmacy List (by location)               │     │ │
│  │  │  • Place Order                                    │     │ │
│  │  │  • Track Delivery                                 │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Order Manager                                      │     │ │
│  │  │  • Create Order                                     │     │ │
│  │  │  • Update Status                                   │     │ │
│  │  │  • Handle Cancellations                            │     │ │
│  │  │  • Delivery Tracking                               │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Pharmacy Partner APIs                                       │ │
│  │  • Local Pharmacy APIs                                       │ │
│  │  • Online Pharmacy APIs (1mg, Netmeds, etc.)                 │ │
│  │  • Medicine Availability API                                │ │
│  │  • Order Placement API                                       │ │
│  │  • Delivery Tracking API                                     │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Output:                                                     │ │
│  │  • Digital Prescription (PDF)                                │ │
│  │  • Medicine Order (if patient chooses)                      │ │
│  │  • Order Tracking                                           │ │
│  │  → Sent to Patient (SMS/WhatsApp/App)                       │ │
│  └──────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### Technology Stack

**Backend**:
- **Prescription Service**: Node.js/Python (REST APIs)
- **PDF Generation**: PDFKit / ReportLab (tamper-proof prescriptions)
- **Digital Signature**: Cryptographic signing (RSA/ECDSA)

**Integration**:
- **Pharmacy APIs**: REST APIs (Adapter pattern for different pharmacy partners)
- **Payment Gateway**: Razorpay/Stripe (for medicine payments)
- **Notification Service**: SMS, WhatsApp, Push notifications

**Database**:
- **Prescription Storage**: PostgreSQL (structured data)
- **Order Tracking**: MongoDB (flexible schema for order status)

#### Data Flow

```
1. Doctor completes consultation
   ↓
2. Doctor generates prescription (Prescription Service)
   ↓
3. Prescription saved (with digital signature)
   ↓
4. Patient receives prescription (SMS/WhatsApp/App)
   ↓
5. Patient chooses to order medicines
   ↓
6. System checks availability (Pharmacy Integration)
   ↓
7. Patient selects pharmacy
   ↓
8. Order placed (Order Manager)
   ↓
9. Order tracked (Delivery Tracking)
   ↓
10. Medicine delivered to patient
```

#### Key Architectural Decisions

1. **Adapter Pattern**: Different pharmacy APIs unified through adapters
2. **Digital Signature**: Cryptographic signing ensures tamper-proof prescriptions
3. **Offline Support**: Prescriptions generated offline, synced later (R4)
4. **Consent Required**: Patient consent needed before sharing with pharmacy (R5)
5. **Multiple Pharmacy Partners**: Support 3-5 pharmacy partners initially

---

### R4: Offline Sync & Storage (Foundation)

#### Architecture Components

```
┌─────────────────────────────────────────────────────────────────┐
│          R4: OFFLINE SYNC & STORAGE ARCHITECTURE                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Mobile Apps (Patient, Sahayak)                               │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Local SQLite Database                             │     │ │
│  │  │  • Patient Data                                    │     │ │
│  │  │  • Consultations                                  │     │ │
│  │  │  • Vitals                                         │     │ │
│  │  │  • Prescriptions                                  │     │ │
│  │  │  • Sync Queue                                    │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  └──────────────────────────────────────────────────────────────┘ │
│         │                                                          │
│         │ (All writes go to local DB first)                        │
│         │                                                          │
│  ┌──────▼──────────────────────────────────────────────────────┐ │
│  │         Offline Sync Service                                 │ │
│  ├──────────────────────────────────────────────────────────────┤ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Sync Queue Manager                                │     │ │
│  │  │  • Enqueue Operations (CREATE/UPDATE/DELETE)      │     │ │
│  │  │  • Priority-based Sync                             │     │ │
│  │  │  • Retry Failed Operations                        │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Conflict Resolver                                  │     │ │
│  │  │  • Detect Conflicts (timestamp/version)            │     │ │
│  │  │  • Auto-resolve (last-write-wins)                  │     │ │
│  │  │  • Flag for Manual Resolution                      │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Background Sync Service                           │     │ │
│  │  │  • Detect Connectivity                             │     │ │
│  │  │  • Trigger Sync                                   │     │ │
│  │  │  • Incremental Sync                               │     │ │
│  │  │  • Status Tracking                                │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  └──────────────────────────────────────────────────────────────┘ │
│         │                                                          │
│         │ (Sync when online)                                       │
│         │                                                          │
│  ┌──────▼──────────────────────────────────────────────────────┐ │
│  │  Remote Database (PostgreSQL/MongoDB)                        │ │
│  │  • Centralized Data Storage                                  │ │
│  │  • Conflict Resolution                                       │ │
│  │  • Data Integrity                                           │ │
│  └──────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### Technology Stack

**Mobile**:
- **Local Database**: SQLite (React Native SQLite / Flutter sqflite)
- **Sync Library**: Custom implementation or use libraries like WatermelonDB

**Backend**:
- **Sync Service**: Node.js/Python (handles sync requests)
- **Remote Database**: PostgreSQL (structured data) / MongoDB (flexible)

**Infrastructure**:
- **Message Queue**: RabbitMQ/Kafka (for async sync operations)
- **Cache**: Redis (sync status, conflict resolution)

#### Sync Strategy

**Store-and-Forward Pattern**:
1. **Write Path**: All writes → Local SQLite → Queue for sync
2. **Read Path**: Read from local cache → Fallback to server if online
3. **Sync Trigger**: 
   - On connectivity restore
   - Periodic (every 5 minutes when online)
   - Manual trigger
4. **Conflict Resolution**:
   - **Auto-resolve**: Last-write-wins (timestamp comparison)
   - **Manual**: Flag critical conflicts for user review

#### Key Architectural Decisions

1. **Local-First**: All operations work offline, sync is secondary
2. **Queue-Based**: Operations queued, synced in background
3. **Conflict Strategy**: Automatic for 90% of cases, manual for critical
4. **Incremental Sync**: Only sync changed data (efficient)
5. **Priority-Based**: Critical data (prescriptions, vitals) synced first

---

### R5: Consent & Privacy (DPDP Act Compliance)

#### Architecture Components

```
┌─────────────────────────────────────────────────────────────────┐
│      R5: CONSENT & PRIVACY (DPDP ACT) ARCHITECTURE              │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  All Data Access Requests                                    │ │
│  │  (Consultations, Vitals, Prescriptions, Records)              │ │
│  └──────────────────┬───────────────────────────────────────────┘ │
│                     │                                               │
│  ┌──────────────────▼───────────────────────────────────────────┐ │
│  │         Access Control Middleware                             │ │
│  │  • Intercepts all requests                                    │ │
│  │  • Checks RBAC permissions                                    │ │
│  │  • Verifies consent                                           │ │
│  │  • Enforces restrictions                                      │ │
│  └──────────────────┬───────────────────────────────────────────┘ │
│                     │                                               │
│  ┌──────────────────▼───────────────────────────────────────────┐ │
│  │         Consent & Privacy Service                             │ │
│  ├──────────────────────────────────────────────────────────────┤ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  RBAC Service                                       │     │ │
│  │  │  • Role Definitions (PATIENT, SAHAYAK, DOCTOR, etc.)│     │ │
│  │  │  • Permission Matrix                                 │     │ │
│  │  │  • Access Control Checks                            │     │ │
│  │  │  • Sahayak: Only Active Patient Data                │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Consent Manager                                    │     │ │
│  │  │  • Request Consent (OTP/Biometric)                  │     │ │
│  │  │  • Verify Consent                                  │     │ │
│  │  │  • Store Consent Records                           │     │ │
│  │  │  • Revoke Consent                                  │     │ │
│  │  │  • Consent Expiration                               │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Encryption Service                                 │     │ │
│  │  │  • Encrypt at Rest (AES-256)                       │     │ │
│  │  │  • Encrypt in Transit (TLS 1.3)                     │     │ │
│  │  │  • Key Management                                  │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  │                                                              │ │
│  │  ┌────────────────────────────────────────────────────┐     │ │
│  │  │  Audit Logger                                       │     │ │
│  │  │  • Log All Access                                   │     │ │
│  │  │  • Log Consent Actions                              │     │ │
│  │  │  • Immutable Audit Trail                            │     │ │
│  │  │  • Compliance Reports                               │     │ │
│  │  └────────────────────────────────────────────────────┘     │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Verification Services                                        │ │
│  │  • OTP Service (SMS)                                         │ │
│  │  • Biometric Service (Fingerprint/Face)                      │ │
│  │  • Signature Service (Digital Signature)                     │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Output:                                                     │ │
│  │  • Access Granted/Denied                                     │ │
│  │  • Audit Log Entry                                           │ │
│  │  • Encrypted Data (if access granted)                        │ │
│  └──────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### Technology Stack

**Backend**:
- **Consent Service**: Node.js/Python (RBAC, consent management)
- **Encryption**: AES-256 (at rest), TLS 1.3 (in transit)
- **Key Management**: AWS KMS / HashiCorp Vault

**Authentication**:
- **OTP Service**: Twilio / AWS SNS (SMS OTP)
- **Biometric**: Device-native APIs (TouchID, FaceID, Android Biometric)

**Database**:
- **Consent Records**: PostgreSQL (immutable consent logs)
- **Audit Logs**: Time-series database (InfluxDB) or PostgreSQL

#### Access Control Flow

```
Data Access Request
    ↓
[Access Control Middleware]
    ↓
[Check User Role] (RBAC)
    ↓
[Check Resource Permissions]
    ↓
[Check Consent] (if required)
    ↓
[Check Conditions]
    • Sahayak: Only active patient data?
    • Doctor: Assigned to this patient?
    • Consent: Valid and not expired?
    ↓
[Log Access] (Audit Logger)
    ↓
[Encrypt Data] (if sensitive)
    ↓
Grant/Deny Access
```

#### Key Architectural Decisions

1. **Middleware Pattern**: Access control at API gateway level
2. **RBAC Model**: Role-based with fine-grained permissions
3. **Consent Storage**: Immutable consent records (cannot be deleted)
4. **Encryption**: End-to-end encryption for sensitive data
5. **Audit Trail**: All access logged, cannot be modified
6. **DPDP Compliance**: Right to be forgotten, data portability, consent management

---

### Integrated Architecture: All 5 Requirements

#### Complete System Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPLETE SYSTEM FLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Patient arrives at kiosk                                     │
│     ↓                                                             │
│  2. Sahayak registers patient (R5: Consent requested)           │
│     ↓                                                             │
│  3. Patient describes symptoms (local dialect)                   │
│     ↓                                                             │
│  4. R2: AI Triage & Translation                                  │
│     • Detects language                                            │
│     • Translates to English                                       │
│     • Generates medical summary                                   │
│     • Calculates severity score                                   │
│     ↓                                                             │
│  5. Consultation created (R4: Stored locally)                    │
│     ↓                                                             │
│  6. R1: Tele-Consultation starts                                 │
│     • Video room created                                         │
│     • Patient, Sahayak, Doctor join                              │
│     • Sahayak uploads vitals (IoT devices)                       │
│     • Vitals appear on doctor's screen                           │
│     ↓                                                             │
│  7. Doctor sees:                                                  │
│     • Translated symptoms (R2)                                    │
│     • Real-time vitals (R1)                                       │
│     • Patient video                                              │
│     ↓                                                             │
│  8. Doctor generates prescription                                │
│     ↓                                                             │
│  9. R3: Prescription & Fulfillment                               │
│     • Digital prescription created                               │
│     • Pharmacy availability checked                              │
│     • Patient can order medicines                                │
│     ↓                                                             │
│  10. All data synced (R4: Offline Sync)                          │
│      • Local data → Remote database                              │
│      • Conflicts resolved                                         │
│     ↓                                                             │
│  11. R5: Access logged (Audit Trail)                             │
│      • All access logged for compliance                          │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

#### Technology Stack Summary

**Frontend**:
- React Native (Patient, Sahayak apps - offline-capable)
- React/Web (Doctor app - WebRTC support)

**Backend Services**:
- Node.js/Python microservices
- API Gateway (Kong/AWS API Gateway)
- Message Queue (Kafka/RabbitMQ)

**Databases**:
- PostgreSQL (primary data)
- MongoDB (flexible schemas)
- SQLite (local mobile storage)
- Redis (cache, sync status)

**External Services**:
- WebRTC Media Server (Janus/Kurento)
- LLM APIs (OpenAI/Anthropic)
- Speech-to-Text (Google/AWS)
- Translation APIs
- Pharmacy APIs
- OTP Service (SMS)

**Infrastructure**:
- Docker containers
- Kubernetes orchestration
- CDN (CloudFlare)
- Monitoring (Prometheus + Grafana)

---

### Architecture Validation

#### Requirement Coverage Checklist

- [x] **R1: Tele-Consultation**
  - [x] Low-bandwidth video (256 Kbps)
  - [x] Multi-party calls (Patient, Sahayak, Doctor)
  - [x] IoT device integration (BP Monitor, Oximeter)
  - [x] Real-time vitals on doctor's screen

- [x] **R2: AI Triage & Translation**
  - [x] Voice/text input in local dialect
  - [x] Language detection
  - [x] Translation to English
  - [x] Medical summary generation
  - [x] Severity scoring (1-10)
  - [x] Triage priority

- [x] **R3: Prescription & Fulfillment**
  - [x] Digital prescription generation
  - [x] Pharmacy integration
  - [x] Medicine availability check
  - [x] Order management
  - [x] Delivery tracking

- [x] **R4: Offline Sync & Storage**
  - [x] Local SQLite storage
  - [x] Store-and-forward pattern
  - [x] Automatic sync when online
  - [x] Conflict resolution

- [x] **R5: Consent & Privacy**
  - [x] RBAC implementation
  - [x] Sahayak restrictions (active patients only)
  - [x] Consent management (OTP/Biometric)
  - [x] Audit logging
  - [x] DPDP Act compliance

---

## Low-Level Design (LLD)

### 1. Tele-Consultation Service - Detailed Design

#### Component Structure
```
TeleConsultationService
├── ConsultationController
│   ├── createConsultation()
│   ├── joinConsultation()
│   ├── endConsultation()
│   └── getConsultationStatus()
├── MediaManager
│   ├── initializeWebRTC()
│   ├── optimizeBandwidth()
│   ├── handleLowBandwidth()
│   └── recordSession()
├── QueueManager
│   ├── addToQueue()
│   ├── assignDoctor()
│   ├── prioritizeQueue()
│   └── notifyDoctor()
└── NotificationService
    ├── sendNotification()
    └── updateStatus()
```

#### Data Models

**Consultation**
```typescript
interface Consultation {
  id: string;
  patientId: string;
  sahayakId: string;
  doctorId: string | null;
  status: 'QUEUED' | 'IN_PROGRESS' | 'COMPLETED' | 'CANCELLED';
  priority: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  scheduledAt: Date;
  startedAt: Date | null;
  endedAt: Date | null;
  symptoms: string;
  translatedSymptoms: string;
  severityScore: number;
  videoRoomId: string;
  recordingUrl: string | null;
  consentGiven: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

**Vitals**
```typescript
interface Vitals {
  id: string;
  consultationId: string;
  patientId: string;
  bloodPressure: { systolic: number; diastolic: number };
  oxygenSaturation: number;
  heartRate: number;
  temperature: number;
  recordedBy: string; // Sahayak ID
  recordedAt: Date;
  deviceType: 'BP_MONITOR' | 'OXIMETER' | 'THERMOMETER';
}
```

#### API Endpoints

```
POST   /api/v1/consultations
GET    /api/v1/consultations/:id
POST   /api/v1/consultations/:id/join
POST   /api/v1/consultations/:id/end
POST   /api/v1/consultations/:id/vitals
GET    /api/v1/consultations/queue
POST   /api/v1/consultations/:id/consent
```

#### Sequence Diagram: Consultation Flow

```
Patient → Sahayak App → TeleConsultation Service → AI Service → Doctor App
   │            │                │                    │              │
   │──Register──>│                │                    │              │
   │            │──Create Consult─>                   │              │
   │            │                │──Translate Symptoms─>              │
   │            │                │<──Translated Summary              │
   │            │                │──Add to Queue───────>             │
   │            │                │                    │              │
   │            │                │                    │──Notify Doctor
   │            │                │                    │              │
   │            │                │<──Doctor Accepts───              │
   │            │                │                    │              │
   │            │──Upload Vitals─>│                    │              │
   │            │                │                    │              │
   │            │<──Video Room───│                    │              │
   │            │                │                    │              │
   │<──Join Room─│                │                    │              │
   │            │                │                    │<──Join Room──
   │            │                │                    │              │
   │──Consultation in Progress───│                    │              │
   │            │                │                    │              │
   │            │                │<──End Consultation─              │
   │            │                │                    │              │
   │<──Prescription──────────────│                    │              │
```

### 2. AI Triage & Translation Service - Detailed Design

#### Component Structure
```
AITriageService
├── TranslationEngine
│   ├── translateDialect()
│   ├── detectLanguage()
│   └── normalizeText()
├── SymptomAnalyzer
│   ├── extractSymptoms()
│   ├── classifySeverity()
│   └── generateSummary()
├── TriageEngine
│   ├── calculateSeverityScore()
│   ├── recommendPriority()
│   └── suggestTriage()
└── LLMService
    ├── callLLM()
    ├── formatPrompt()
    └── parseResponse()
```

#### Data Models

**SymptomInput**
```typescript
interface SymptomInput {
  id: string;
  consultationId: string;
  patientId: string;
  rawText: string;
  audioUrl?: string;
  detectedLanguage: string;
  dialect: string;
  timestamp: Date;
}

interface SymptomOutput {
  id: string;
  symptomInputId: string;
  translatedText: string;
  extractedSymptoms: string[];
  severityScore: number; // 1-10
  priority: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  medicalSummary: string;
  suggestedTriage: string;
  confidence: number; // 0-1
  processedAt: Date;
}
```

#### Processing Flow

```
1. Receive symptom input (text/audio)
2. Detect language and dialect
3. Translate to English (if needed)
4. Extract medical symptoms using LLM
5. Classify severity using ML model
6. Generate medical summary
7. Calculate triage priority
8. Return structured output
```

### 3. Offline Sync Service - Detailed Design

#### Component Structure
```
OfflineSyncService
├── LocalStorageManager
│   ├── storeLocally()
│   ├── retrieveLocal()
│   └── clearLocal()
├── SyncManager
│   ├── detectConnectivity()
│   ├── queueForSync()
│   ├── syncData()
│   └── handleConflict()
├── ConflictResolver
│   ├── detectConflict()
│   ├── resolveConflict()
│   └── mergeData()
└── SyncStatusTracker
    ├── trackSyncStatus()
    ├── getPendingItems()
    └── retryFailedSync()
```

#### Data Models

**SyncQueue**
```typescript
interface SyncQueueItem {
  id: string;
  entityType: 'CONSULTATION' | 'VITALS' | 'PRESCRIPTION' | 'PATIENT';
  entityId: string;
  operation: 'CREATE' | 'UPDATE' | 'DELETE';
  payload: any;
  deviceId: string;
  timestamp: Date;
  status: 'PENDING' | 'SYNCING' | 'SYNCED' | 'FAILED' | 'CONFLICT';
  retryCount: number;
  lastError?: string;
  syncedAt?: Date;
}
```

#### Sync Strategy

**Store-and-Forward Pattern:**
1. **Write Path**: All writes go to local SQLite first, then queued for sync
2. **Read Path**: Read from local cache, fallback to server if online
3. **Sync Trigger**: 
   - On connectivity restore
   - Periodic background sync (every 5 minutes when online)
   - Manual sync trigger
4. **Conflict Resolution**: Last-write-wins with timestamp comparison, manual resolution for critical conflicts

### 4. Consent & Privacy Service - Detailed Design

#### Component Structure
```
ConsentPrivacyService
├── ConsentManager
│   ├── requestConsent()
│   ├── verifyConsent()
│   ├── revokeConsent()
│   └── getConsentStatus()
├── AccessControl
│   ├── checkPermission()
│   ├── enforceRBAC()
│   └── auditAccess()
├── EncryptionService
│   ├── encryptData()
│   ├── decryptData()
│   └── rotateKeys()
└── AuditLogger
    ├── logAccess()
    ├── logConsent()
    └── generateAuditReport()
```

#### Data Models

**Consent**
```typescript
interface Consent {
  id: string;
  patientId: string;
  consentType: 'DATA_SHARING' | 'RECORDING' | 'PHARMACY_SHARING' | 'GOVT_SCHEME';
  grantedTo: string; // Role or specific user ID
  grantedBy: string; // Patient ID
  verificationMethod: 'OTP' | 'BIOMETRIC' | 'SIGNATURE';
  verificationToken: string;
  status: 'PENDING' | 'GRANTED' | 'REVOKED' | 'EXPIRED';
  grantedAt: Date;
  expiresAt: Date;
  revokedAt: Date | null;
}

interface AccessLog {
  id: string;
  userId: string;
  role: 'SAHAYAK' | 'DOCTOR' | 'ADMIN';
  resourceType: string;
  resourceId: string;
  action: 'READ' | 'WRITE' | 'DELETE';
  ipAddress: string;
  timestamp: Date;
  consentId: string | null;
  success: boolean;
}
```

#### RBAC Model

```
Roles:
- PATIENT: Can view own data, grant consent
- SAHAYAK: Can view active patient data (only during consultation)
- DOCTOR: Can view assigned patient data, generate prescriptions
- ADMIN: Full access (with audit trail)
- PHARMACY: Can view prescription details (with consent)
- GOVT_SCHEME: Can verify eligibility (with consent)
```

### 5. Database Schema Design

#### Core Tables

**patients**
```sql
CREATE TABLE patients (
  id UUID PRIMARY KEY,
  abha_id VARCHAR(50) UNIQUE,
  name VARCHAR(255) NOT NULL,
  phone VARCHAR(20) NOT NULL,
  date_of_birth DATE,
  gender VARCHAR(10),
  address TEXT,
  village VARCHAR(100),
  district VARCHAR(100),
  state VARCHAR(100),
  emergency_contact VARCHAR(20),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  encrypted_data JSONB -- For sensitive fields
);
```

**consultations**
```sql
CREATE TABLE consultations (
  id UUID PRIMARY KEY,
  patient_id UUID REFERENCES patients(id),
  sahayak_id UUID REFERENCES users(id),
  doctor_id UUID REFERENCES users(id),
  status VARCHAR(20) NOT NULL,
  priority VARCHAR(20),
  severity_score INTEGER,
  symptoms TEXT,
  translated_symptoms TEXT,
  video_room_id VARCHAR(100),
  recording_url TEXT,
  consent_given BOOLEAN DEFAULT FALSE,
  scheduled_at TIMESTAMP,
  started_at TIMESTAMP,
  ended_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_patient_id (patient_id),
  INDEX idx_doctor_id (doctor_id),
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);
```

**vitals**
```sql
CREATE TABLE vitals (
  id UUID PRIMARY KEY,
  consultation_id UUID REFERENCES consultations(id),
  patient_id UUID REFERENCES patients(id),
  blood_pressure_systolic INTEGER,
  blood_pressure_diastolic INTEGER,
  oxygen_saturation DECIMAL(5,2),
  heart_rate INTEGER,
  temperature DECIMAL(5,2),
  recorded_by UUID REFERENCES users(id),
  device_type VARCHAR(50),
  recorded_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_consultation_id (consultation_id),
  INDEX idx_patient_id (patient_id)
);
```

**prescriptions**
```sql
CREATE TABLE prescriptions (
  id UUID PRIMARY KEY,
  consultation_id UUID REFERENCES consultations(id),
  patient_id UUID REFERENCES patients(id),
  doctor_id UUID REFERENCES users(id),
  medicines JSONB NOT NULL, -- Array of medicine objects
  instructions TEXT,
  follow_up_date DATE,
  status VARCHAR(20) DEFAULT 'PENDING',
  pharmacy_order_id VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_consultation_id (consultation_id),
  INDEX idx_patient_id (patient_id)
);
```

**consents**
```sql
CREATE TABLE consents (
  id UUID PRIMARY KEY,
  patient_id UUID REFERENCES patients(id),
  consent_type VARCHAR(50) NOT NULL,
  granted_to VARCHAR(100) NOT NULL,
  verification_method VARCHAR(20),
  verification_token VARCHAR(255),
  status VARCHAR(20) NOT NULL,
  granted_at TIMESTAMP,
  expires_at TIMESTAMP,
  revoked_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_patient_id (patient_id),
  INDEX idx_status (status)
);
```

**sync_queue**
```sql
CREATE TABLE sync_queue (
  id UUID PRIMARY KEY,
  entity_type VARCHAR(50) NOT NULL,
  entity_id UUID NOT NULL,
  operation VARCHAR(20) NOT NULL,
  payload JSONB NOT NULL,
  device_id VARCHAR(100),
  status VARCHAR(20) DEFAULT 'PENDING',
  retry_count INTEGER DEFAULT 0,
  last_error TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  synced_at TIMESTAMP,
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);
```

---

## Design Patterns

### 1. **Store-and-Forward Pattern**
**Problem**: Unreliable network connectivity in rural areas
**Solution**: 
- All data writes go to local database first
- Background service queues data for sync when connectivity is available
- Conflict resolution strategy for concurrent updates

**Implementation**:
```typescript
// Pseudo-code structure
class OfflineSyncManager {
  async writeData(entity: Entity) {
    // Write to local DB
    await localDB.save(entity);
    // Queue for sync
    await syncQueue.add({
      entityType: entity.type,
      entityId: entity.id,
      operation: 'CREATE',
      payload: entity
    });
    // Trigger sync if online
    if (this.isOnline()) {
      this.sync();
    }
  }
}
```

### 2. **Circuit Breaker Pattern**
**Problem**: External service failures (Govt APIs, Pharmacy APIs) can cascade
**Solution**: 
- Monitor failure rates
- Open circuit after threshold failures
- Fail fast during open state
- Attempt recovery periodically

**Implementation**:
```typescript
class CircuitBreaker {
  private failures = 0;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  
  async execute(fn: Function) {
    if (this.state === 'OPEN') {
      throw new Error('Circuit breaker is OPEN');
    }
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
}
```

### 3. **Adapter Pattern**
**Problem**: Multiple IoT device protocols and formats
**Solution**: 
- Create device adapters for each IoT device type
- Normalize data format internally
- Easy to add new devices

**Implementation**:
```typescript
interface DeviceAdapter {
  connect(): Promise<void>;
  readVitals(): Promise<Vitals>;
  disconnect(): Promise<void>;
}

class BPMonitorAdapter implements DeviceAdapter {
  // BP Monitor specific implementation
}

class OximeterAdapter implements DeviceAdapter {
  // Oximeter specific implementation
}
```

### 4. **Strategy Pattern**
**Problem**: Different bandwidth optimization strategies based on network conditions
**Solution**: 
- Define bandwidth optimization strategies
- Switch strategies dynamically based on network quality

**Implementation**:
```typescript
interface BandwidthStrategy {
  optimizeVideo(stream: VideoStream): VideoStream;
}

class LowBandwidthStrategy implements BandwidthStrategy {
  optimizeVideo(stream) {
    // Reduce resolution, frame rate
  }
}

class HighBandwidthStrategy implements BandwidthStrategy {
  optimizeVideo(stream) {
    // Maintain quality
  }
}
```

### 5. **Observer Pattern**
**Problem**: Multiple components need to react to consultation status changes
**Solution**: 
- Event-driven architecture
- Publish consultation events
- Subscribers react accordingly

**Implementation**:
```typescript
class ConsultationEventBus {
  private subscribers: Map<string, Function[]> = new Map();
  
  subscribe(event: string, handler: Function) {
    // Add subscriber
  }
  
  publish(event: string, data: any) {
    // Notify all subscribers
  }
}

// Usage
eventBus.subscribe('CONSULTATION_STARTED', (data) => {
  notificationService.notifyDoctor(data);
  queueService.removeFromQueue(data.id);
});
```

### 6. **Repository Pattern**
**Problem**: Decouple data access logic from business logic
**Solution**: 
- Abstract data access behind repository interfaces
- Easy to swap implementations (local DB, remote DB, cache)

**Implementation**:
```typescript
interface ConsultationRepository {
  findById(id: string): Promise<Consultation>;
  save(consultation: Consultation): Promise<void>;
  findByPatientId(patientId: string): Promise<Consultation[]>;
}

class LocalConsultationRepository implements ConsultationRepository {
  // SQLite implementation
}

class RemoteConsultationRepository implements ConsultationRepository {
  // API implementation
}
```

### 7. **Factory Pattern**
**Problem**: Creating different types of notifications (SMS, WhatsApp, Push)
**Solution**: 
- Notification factory creates appropriate notification handler
- Easy to add new notification channels

**Implementation**:
```typescript
interface NotificationHandler {
  send(message: string, recipient: string): Promise<void>;
}

class NotificationFactory {
  create(type: 'SMS' | 'WHATSAPP' | 'PUSH'): NotificationHandler {
    switch(type) {
      case 'WHATSAPP': return new WhatsAppHandler();
      case 'SMS': return new SMSHandler();
      case 'PUSH': return new PushHandler();
    }
  }
}
```

### 8. **Facade Pattern**
**Problem**: Complex interactions between multiple services for consultation flow
**Solution**: 
- ConsultationFacade provides simple interface
- Hides complexity of orchestrating multiple services

**Implementation**:
```typescript
class ConsultationFacade {
  async startConsultation(patientId: string, symptoms: string) {
    // 1. Create consultation
    const consultation = await consultationService.create(patientId);
    // 2. Translate symptoms
    const translated = await aiService.translate(symptoms);
    // 3. Calculate severity
    const severity = await aiService.analyze(translated);
    // 4. Add to queue
    await queueService.add(consultation.id, severity);
    // 5. Notify doctor
    await notificationService.notify(consultation.id);
    return consultation;
  }
}
```

### 9. **Proxy Pattern**
**Problem**: Need to add caching, logging, access control to service calls
**Solution**: 
- Service proxy intercepts calls
- Adds cross-cutting concerns

**Implementation**:
```typescript
class ConsultationServiceProxy {
  constructor(private service: ConsultationService) {}
  
  async getConsultation(id: string) {
    // Check cache
    const cached = await cache.get(id);
    if (cached) return cached;
    
    // Check access
    if (!this.hasAccess(id)) throw new Error('Unauthorized');
    
    // Log access
    await auditLogger.log('READ', id);
    
    // Call actual service
    const result = await this.service.getConsultation(id);
    
    // Cache result
    await cache.set(id, result);
    
    return result;
  }
}
```

### 10. **Command Pattern**
**Problem**: Need to queue, undo, and log operations (especially for offline sync)
**Solution**: 
- Encapsulate operations as command objects
- Queue commands for execution
- Support undo/redo

**Implementation**:
```typescript
interface Command {
  execute(): Promise<void>;
  undo(): Promise<void>;
}

class CreateConsultationCommand implements Command {
  constructor(private consultation: Consultation) {}
  
  async execute() {
    await consultationService.create(this.consultation);
  }
  
  async undo() {
    await consultationService.delete(this.consultation.id);
  }
}

class CommandQueue {
  private queue: Command[] = [];
  
  async execute(command: Command) {
    this.queue.push(command);
    await command.execute();
  }
}
```

### 11. **Template Method Pattern**
**Problem**: Similar flows for different consultation types with slight variations
**Solution**: 
- Define template method with common steps
- Subclasses override specific steps

**Implementation**:
```typescript
abstract class ConsultationFlow {
  async process() {
    await this.validatePatient();
    await this.collectSymptoms();
    await this.analyzeSymptoms();
    await this.assignDoctor();
    await this.startConsultation();
    await this.completeConsultation();
  }
  
  protected abstract collectSymptoms(): Promise<void>;
  protected abstract assignDoctor(): Promise<void>;
}

class EmergencyConsultationFlow extends ConsultationFlow {
  protected async assignDoctor() {
    // Assign available doctor immediately
  }
}

class ScheduledConsultationFlow extends ConsultationFlow {
  protected async assignDoctor() {
    // Assign based on schedule
  }
}
```

### 12. **Decorator Pattern**
**Problem**: Need to add features like encryption, compression, logging to data without modifying core classes
**Solution**: 
- Wrap data handlers with decorators
- Compose features dynamically

**Implementation**:
```typescript
interface DataHandler {
  handle(data: any): Promise<any>;
}

class EncryptionDecorator implements DataHandler {
  constructor(private handler: DataHandler) {}
  
  async handle(data: any) {
    const encrypted = await this.encrypt(data);
    return this.handler.handle(encrypted);
  }
}

class CompressionDecorator implements DataHandler {
  constructor(private handler: DataHandler) {}
  
  async handle(data: any) {
    const compressed = await this.compress(data);
    return this.handler.handle(compressed);
  }
}
```

---

## System Components

### Technology Stack Recommendations

#### Frontend
- **Kiosk/Sahayak Apps**: React Native (offline-first, cross-platform)
- **Doctor App**: React (WebRTC support, desktop-friendly)
- **State Management**: Redux Toolkit / Zustand
- **Offline Storage**: SQLite (via react-native-sqlite-storage)

#### Backend
- **API Framework**: Node.js (Express) / Python (FastAPI) / Java (Spring Boot)
- **Microservices Communication**: gRPC (internal), REST (external)
- **Message Queue**: Apache Kafka / RabbitMQ
- **Service Discovery**: Consul / Eureka

#### Database
- **Primary DB**: PostgreSQL (structured data) / MongoDB (flexible schemas)
- **Offline DB**: SQLite (mobile apps)
- **Cache**: Redis (session, queue, rate limiting)
- **Search**: Elasticsearch (patient search, symptom search)

#### Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **API Gateway**: Kong / AWS API Gateway
- **CDN**: CloudFront / Cloudflare (for static assets, video)
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)

#### AI/ML Services
- **LLM**: OpenAI GPT-4 / Anthropic Claude / Local LLM (for privacy)
- **Speech-to-Text**: Google Speech-to-Text / AWS Transcribe
- **Translation**: Google Translate API / Custom model
- **Computer Vision**: TensorFlow / PyTorch models
- **ML Framework**: TensorFlow Serving / MLflow

#### External Integrations
- **Video**: WebRTC (Janus/Kurento media server)
- **WhatsApp**: WhatsApp Business API
- **Govt APIs**: Ayushman Bharat APIs (REST)
- **Pharmacy**: Partner pharmacy APIs

---

## Data Flow

### Consultation Flow (End-to-End)

```
1. Patient arrives at kiosk
   ↓
2. Sahayak registers/authenticates patient
   ↓
3. Patient describes symptoms (voice/text in dialect)
   ↓
4. Sahayak uploads vitals from IoT devices
   ↓
5. System translates symptoms to English
   ↓
6. AI analyzes and calculates severity score
   ↓
7. Consultation created and added to queue
   ↓
8. Doctor notified (if available)
   ↓
9. Doctor accepts consultation
   ↓
10. Video room created (low-bandwidth optimized)
    ↓
11. Patient, Sahayak, Doctor join video call
    ↓
12. Consultation proceeds (with AI assistance)
    ↓
13. Doctor generates prescription
    ↓
14. Prescription saved and shared with patient
    ↓
15. Pharmacy integration (if consent given)
    ↓
16. Medicine order placed
    ↓
17. Follow-up reminders scheduled (WhatsApp)
    ↓
18. Consultation marked complete
```

### Offline Sync Flow

```
1. App detects offline mode
   ↓
2. All writes go to local SQLite
   ↓
3. Operations queued in sync_queue table
   ↓
4. App detects connectivity restored
   ↓
5. Background sync service starts
   ↓
6. For each queued item:
   a. Send to server
   b. If conflict detected → resolve
   c. Mark as synced
   ↓
7. Update local cache with server data
   ↓
8. Clear synced items from queue
```

### Consent & Access Flow

```
1. Patient needs to share data
   ↓
2. System requests consent (OTP/Biometric)
   ↓
3. Patient verifies identity
   ↓
4. Consent granted and stored
   ↓
5. Access request comes in
   ↓
6. System checks:
   - Role has permission?
   - Consent exists and valid?
   - Access within scope?
   ↓
7. If all checks pass → grant access + log
   ↓
8. If fails → deny + log + notify
```

---

## Security Considerations

1. **Data Encryption**: 
   - At rest: AES-256
   - In transit: TLS 1.3
   - End-to-end encryption for video calls

2. **Authentication**:
   - OAuth 2.0 / JWT tokens
   - Multi-factor authentication for doctors/admins
   - Biometric authentication for patients

3. **Authorization**:
   - Role-Based Access Control (RBAC)
   - Attribute-Based Access Control (ABAC) for fine-grained control
   - Consent-based data sharing

4. **Audit Logging**:
   - All data access logged
   - Immutable audit trail
   - Regular compliance audits

5. **Privacy**:
   - Data minimization
   - Right to be forgotten
   - Consent management
   - DPDP Act compliance

---

## Scalability & Performance

1. **Horizontal Scaling**: Microservices can scale independently
2. **Caching Strategy**: Redis for frequently accessed data
3. **CDN**: Static assets and video content delivery
4. **Database Sharding**: By region/state for patient data
5. **Load Balancing**: Round-robin / Least connections
6. **Rate Limiting**: Per user, per API endpoint
7. **Connection Pooling**: Database connection management
8. **Async Processing**: Background jobs for heavy operations

---

## Monitoring & Observability

1. **Metrics**: 
   - Response times
   - Error rates
   - Queue lengths
   - Active consultations
   - Sync success rates

2. **Logging**:
   - Structured logging (JSON)
   - Log levels (DEBUG, INFO, WARN, ERROR)
   - Centralized log aggregation

3. **Tracing**:
   - Distributed tracing (Jaeger/Zipkin)
   - Request ID propagation
   - Service dependency mapping

4. **Alerting**:
   - High error rates
   - Service downtime
   - Queue backup
   - Sync failures

---

This architecture document provides a comprehensive foundation for building the GramSeva Health platform. Each component can be further detailed based on specific implementation requirements.
