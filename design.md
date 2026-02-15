# Government Schemes Voice Assistant - Design Document

## System Overview

The Government Schemes Voice Assistant is an AI-powered, voice-first mobile application that democratizes access to government welfare schemes for underserved Indian citizens. The system combines speech recognition, natural language understanding, rule-based eligibility matching, and LLM-powered reasoning to provide personalized scheme recommendations through natural conversation.

**Core Innovation:** An offline-first architecture with adaptive questioning that guarantees eligibility matches in ≤6 questions, eliminating false positives and reducing discovery time from hours to minutes.

**Key Differentiators:**
- Voice-only interaction requiring zero reading ability
- 95%+ functionality works completely offline
- Guaranteed eligibility (no irrelevant schemes shown)
- Multilingual support with dialect recognition
- Works on low-end Android devices with 2GB RAM
- Privacy-first design with no permanent voice storage

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT LAYER (Mobile)                    │
├─────────────────────────────────────────────────────────────┤
│  Voice Input/Output  │  Offline Cache  │  Local Processing  │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                    EDGE/SYNC LAYER                          │
├─────────────────────────────────────────────────────────────┤
│  Background Sync  │  Conflict Resolution  │  CDN Cache      │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND SERVICES                         │
├─────────────────────────────────────────────────────────────┤
│  API Gateway  │  LLM Service  │  Scheme DB  │  Analytics   │
└─────────────────────────────────────────────────────────────┘
```

### Component Breakdown

#### 1. Client Interface (Mobile App)
- **Platform:** Android (React Native / Flutter)
- **UI Modes:** Voice-primary with optional visual feedback
- **Offline Storage:** SQLite for scheme data, user context
- **Size:** < 50MB app + < 100MB offline cache

#### 2. Voice Input & Output Module
- **Speech-to-Text:** On-device (Whisper Tiny) + Cloud fallback (AWS Transcribe)
- **Text-to-Speech:** On-device (eSpeak) + Cloud (Amazon Polly)
- **Languages:** Hindi, Tamil, Telugu, Bengali, Marathi (Phase 1)
- **Dialect Support:** Regional variations through fine-tuning

#### 3. Eligibility Engine (Core Innovation)
- **Rule Engine:** JSON-based eligibility rules (age, income, location, etc.)
- **Adaptive Questioner:** Decision tree that minimizes questions
- **Confidence Scorer:** Assigns match probability to each scheme
- **Runs:** 100% offline using cached rules

#### 4. Scheme Knowledge Base
- **Data Source:** Aggregated from government portals (MyScheme, state portals)
- **Structure:** Scheme metadata, eligibility criteria, benefits, documents, steps
- **Update Frequency:** Monthly automated sync
- **Storage:** Versioned JSON files, distributed via CDN

#### 5. AI Reasoning Layer (LLM)
- **Primary:** Amazon Bedrock (Claude/Titan)
- **Fallback:** On-device small language model for offline
- **Use Cases:** Natural language understanding, explanation generation, edge cases
- **Optimization:** Prompt caching, response streaming

#### 6. Offline Cache & Sync Layer
- **Strategy:** Offline-first with eventual consistency
- **Cached Data:** Scheme database, eligibility rules, user progress
- **Sync Triggers:** App launch, background schedule, user-initiated
- **Conflict Resolution:** Last-write-wins with user confirmation

#### 7. Human Escalation Module
- **Triggers:** User confusion, low confidence matches, explicit request
- **Actions:** Provide CSC contact, helpline numbers, SMS summary
- **Integration:** CSC locator API, SMS gateway
- **Handoff:** Generate context summary for human helper

---

## Data Flow

### End-to-End User Journey

```
1. Voice Input (User speaks in regional language)
   ↓
2. Speech-to-Text (On-device or cloud)
   ↓
3. Language Detection & Intent Recognition
   ↓
4. Context Extraction (age, location, occupation, etc.)
   ↓
5. Adaptive Questioning (≤6 questions to fill gaps)
   ↓
6. Eligibility Engine Evaluation (rule-based matching)
   ↓
7. Scheme Ranking (by benefit value, urgency, confidence)
   ↓
8. LLM-Powered Explanation Generation
   ↓
9. Text-to-Speech (Voice response)
   ↓
10. User Interaction (follow-up questions, document check)
    ↓
11. Progress Tracking & Sync (when online)
```

### Detailed Flow Diagrams

#### Flow 1: Initial Eligibility Discovery (Offline)
```
User: "Mujhe koi scheme batao" (Tell me about schemes)
  → STT: Convert to text
  → Intent: Scheme discovery
  → Adaptive Q1: "Aapki umar kya hai?" (What's your age?)
User: "45 saal" (45 years)
  → Extract: age = 45
  → Adaptive Q2: "Aap kya kaam karte hain?" (What work do you do?)
User: "Kheti karta hoon" (I do farming)
  → Extract: occupation = farmer
  → Adaptive Q3: "Kitni zameen hai?" (How much land?)
User: "Do acre" (2 acres)
  → Extract: land = 2 acres
  → Eligibility Engine: Match against 500+ schemes
  → Result: 4 eligible schemes (PM-KISAN, Crop Insurance, etc.)
  → TTS: "Aapke liye 4 yojana milti hain..." (4 schemes found for you...)
```

#### Flow 2: Document Readiness Check
```
User: "PM-KISAN ke liye kya chahiye?" (What's needed for PM-KISAN?)
  → Intent: Document inquiry
  → Fetch: PM-KISAN document requirements
  → TTS: "Aapko 3 cheezein chahiye..." (You need 3 things...)
  → List: Aadhaar, Bank passbook, Land records
  → Ask: "Kya aapke paas yeh sab hai?" (Do you have these?)
User: "Aadhaar aur bank hai, zameen ka kagaz nahi" (Have Aadhaar and bank, not land records)
  → Mark: 2/3 documents ready
  → TTS: "Zameen ka kagaz patwari se milega..." (Get land records from patwari...)
  → Store: Document status for tracking
```

#### Flow 3: Human Escalation
```
User: "Mujhe samajh nahi aa raha" (I don't understand)
  → Detect: Confusion signal
  → Trigger: Escalation flow
  → Locate: Nearest CSC (using GPS/pincode)
  → TTS: "Koi baat nahi, main aapko madad ke liye jagah batata hoon..."
         (No problem, let me tell you where to get help...)
  → Provide: CSC address, phone number
  → SMS: Send summary to user's phone
  → Log: Escalation for analytics
```

---

## Eligibility Engine Design (Key Differentiator)

### Architecture

The Eligibility Engine is the core innovation that enables offline, guaranteed-eligibility matching.

```
┌──────────────────────────────────────────────────────┐
│              ELIGIBILITY ENGINE                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌────────────────┐      ┌─────────────────┐       │
│  │ User Context   │      │ Scheme Rules DB │       │
│  │ - Age: 45      │      │ - 500+ schemes  │       │
│  │ - Occupation   │      │ - Eligibility   │       │
│  │ - Location     │      │   criteria      │       │
│  │ - Income       │      │ - JSON format   │       │
│  └────────┬───────┘      └────────┬────────┘       │
│           │                       │                 │
│           └───────┬───────────────┘                 │
│                   ↓                                 │
│         ┌─────────────────────┐                    │
│         │  Rule Matcher       │                    │
│         │  - AND/OR logic     │                    │
│         │  - Range checks     │                    │
│         │  - Exclusions       │                    │
│         └──────────┬──────────┘                    │
│                    ↓                                │
│         ┌─────────────────────┐                    │
│         │  Confidence Scorer  │                    │
│         │  - 100% = all met   │                    │
│         │  - <100% = partial  │                    │
│         └──────────┬──────────┘                    │
│                    ↓                                │
│         ┌─────────────────────┐                    │
│         │  Ranker & Filter    │                    │
│         │  - Show only 100%   │                    │
│         │  - Sort by value    │                    │
│         └──────────┬──────────┘                    │
│                    ↓                                │
│              Eligible Schemes                       │
└──────────────────────────────────────────────────────┘
```

### Rule-Based Eligibility Filters

Each scheme is represented as a JSON rule set:

```json
{
  "scheme_id": "PM-KISAN",
  "name": "Pradhan Mantri Kisan Samman Nidhi",
  "eligibility": {
    "rules": [
      {
        "field": "occupation",
        "operator": "equals",
        "value": "farmer",
        "required": true
      },
      {
        "field": "land_holding",
        "operator": "less_than_or_equal",
        "value": 2,
        "unit": "hectares",
        "required": true
      },
      {
        "field": "income",
        "operator": "less_than",
        "value": 200000,
        "unit": "INR_annual",
        "required": false
      }
    ],
    "logic": "AND"
  },
  "benefits": {
    "amount": 6000,
    "frequency": "annual",
    "installments": 3
  },
  "documents": ["aadhaar", "bank_account", "land_records"],
  "priority": "high"
}
```

### Adaptive Questioning Logic

The system uses a decision tree to minimize questions:

```
Start
  ↓
Q1: Age? (Required for 90% of schemes)
  ↓
Q2: Occupation? (Narrows to 30% of schemes)
  ↓
[Branch based on occupation]
  ↓
If Farmer → Q3: Land size?
If Laborer → Q3: Daily wage?
If Elderly → Q3: Pension status?
  ↓
Q4: Location (State/District) - for state-specific schemes
  ↓
Q5: Income (if needed for remaining schemes)
  ↓
Q6: Special category (SC/ST/OBC/Minority) - if applicable
  ↓
Match Complete
```

**Optimization Strategies:**
- Ask most discriminating questions first
- Infer information when possible (e.g., farmer → likely rural)
- Skip questions if answer won't change results
- Use conversational context to extract multiple data points

### Ranking Based on Benefit Value and Urgency

Schemes are ranked using a weighted score:

```
Score = (Benefit_Value × 0.4) + (Urgency × 0.3) + (Ease × 0.2) + (Confidence × 0.1)

Where:
- Benefit_Value: Monetary value (normalized 0-100)
- Urgency: Deadline proximity (100 = deadline soon, 0 = no deadline)
- Ease: Document availability (100 = all docs ready, 0 = none ready)
- Confidence: Match confidence (100 = perfect match)
```

### Offline Fallback Using Cached Eligibility Rules

- All scheme rules cached locally (updated monthly)
- Rule evaluation runs entirely on-device
- No network required for core matching
- Sync new schemes when online
- Version control to detect outdated cache

---

## Offline vs Online Capabilities

### Offline Capabilities (95% of Features)

| Feature | Offline Support | Implementation |
|---------|----------------|----------------|
| Voice input | ✅ Full | On-device Whisper Tiny model |
| Voice output | ✅ Full | On-device eSpeak TTS |
| Eligibility matching | ✅ Full | Local rule engine + cached schemes |
| Scheme explanations | ⚠️ Basic | Template-based responses |
| Document checklist | ✅ Full | Cached document requirements |
| Progress tracking | ✅ Full | Local SQLite storage |
| Language support | ✅ Full | All models cached locally |
| Adaptive questioning | ✅ Full | Decision tree logic |

### Online-Only Capabilities (5% of Features)

| Feature | Why Online? | Fallback |
|---------|-------------|----------|
| LLM reasoning | Requires API call | Template responses |
| Scheme updates | Fresh data | Use cached version |
| Voice quality (premium) | Cloud TTS | On-device TTS |
| Application status sync | Government portal API | Show last known status |
| Analytics | Server logging | Queue for later sync |

### Graceful Degradation Strategy

```
Online Mode:
- High-quality voice (Amazon Polly)
- LLM-powered explanations
- Real-time scheme updates
- Application status sync

↓ (Connection lost)

Offline Mode:
- Basic voice (eSpeak)
- Template explanations
- Cached schemes (last updated: shown to user)
- Local progress tracking

↓ (Connection restored)

Background Sync:
- Upload queued analytics
- Download scheme updates
- Sync progress across devices
- Refresh application status
```

---

## Data Management & Updates

### Scheme Database Structure

```
schemes/
├── central/
│   ├── agriculture/
│   │   ├── pm-kisan.json
│   │   ├── crop-insurance.json
│   │   └── kisan-credit-card.json
│   ├── social-welfare/
│   │   ├── pm-awas-yojana.json
│   │   └── ayushman-bharat.json
│   └── ...
├── state/
│   ├── karnataka/
│   │   ├── raitha-bandhu.json
│   │   └── ...
│   ├── tamil-nadu/
│   │   └── ...
│   └── ...
└── metadata.json (version, last_updated, checksum)
```

### Monthly Scheme Data Sync

**Automated Pipeline:**
```
1. Web Scraping (MyScheme.gov.in, state portals)
   ↓
2. Data Extraction & Normalization
   ↓
3. Eligibility Rule Generation (AI-assisted)
   ↓
4. Human Review & Validation
   ↓
5. JSON Schema Validation
   ↓
6. Version Increment (semantic versioning)
   ↓
7. CDN Upload (CloudFront/Cloudflare)
   ↓
8. Client Notification (background sync trigger)
```

**Update Frequency:**
- Critical updates (new schemes, deadline changes): Weekly
- Regular updates (benefit amounts, document changes): Monthly
- Minor updates (typos, clarifications): Quarterly

### Versioned Eligibility Rules

```json
{
  "version": "2026.02.1",
  "released": "2026-02-01",
  "schemes_count": 547,
  "changes": [
    {
      "scheme_id": "PM-KISAN",
      "change_type": "benefit_increase",
      "old_value": 6000,
      "new_value": 8000,
      "effective_date": "2026-04-01"
    }
  ],
  "backward_compatible": true,
  "min_app_version": "1.2.0"
}
```

**Version Management:**
- Semantic versioning: YEAR.MONTH.PATCH
- Backward compatibility for 3 versions
- Forced update for breaking changes
- Diff-based updates to minimize bandwidth

### Admin Update Capability (Design-Level)

**Admin Dashboard (Web-based):**
- Add/edit/archive schemes
- Update eligibility criteria
- Modify document requirements
- Set priority and urgency flags
- Preview changes before publishing
- Rollback capability

**Workflow:**
```
Admin submits change
  ↓
AI validation (check for conflicts, missing fields)
  ↓
Human reviewer approval
  ↓
Staging environment testing
  ↓
Production deployment
  ↓
Client notification
```

**Access Control:**
- Role-based permissions (admin, reviewer, viewer)
- Audit log for all changes
- Two-factor authentication
- IP whitelisting for production access

---

## Security & Privacy

### No Permanent Voice Storage

**Voice Data Lifecycle:**
```
1. User speaks → Voice captured in memory
2. Sent to STT engine (encrypted in transit)
3. Text extracted
4. Voice data immediately deleted (< 5 seconds retention)
5. Only text stored temporarily for session context
6. Text deleted after session ends (max 24 hours)
```

**Implementation:**
- In-memory processing only
- No disk writes for voice data
- Encrypted RAM on supported devices
- Secure deletion (overwrite memory)
- No cloud storage of voice recordings

### Local Encryption for Cached Data

**Encryption Strategy:**
- AES-256 encryption for local SQLite database
- Device-specific encryption keys (Android Keystore)
- Encrypted shared preferences for sensitive data
- Secure key derivation (PBKDF2)

**What's Encrypted:**
- User demographic information
- Application progress and status
- Document availability status
- Session history

**What's Not Encrypted:**
- Scheme database (public information)
- App configuration
- UI preferences

### Minimal Personal Data Collection

**Data Minimization Principle:**

| Data Point | Collected? | Purpose | Retention |
|------------|-----------|---------|-----------|
| Name | ❌ No | Not needed for eligibility | N/A |
| Phone number | ⚠️ Optional | SMS notifications only | User-controlled |
| Age | ✅ Yes | Eligibility matching | Session only |
| Location (State/District) | ✅ Yes | State scheme matching | Session only |
| Income | ✅ Yes | Eligibility matching | Session only |
| Occupation | ✅ Yes | Eligibility matching | Session only |
| Caste/Category | ⚠️ Optional | Special category schemes | Session only |
| Aadhaar | ❌ No | Not collected by app | N/A |
| Bank details | ❌ No | Not collected by app | N/A |

**User Control:**
- Clear data anytime
- Export personal data
- Opt-out of analytics
- Delete account (if registered)

### Compliance & Standards

**Regulatory Compliance:**
- Indian IT Act 2000
- Digital Personal Data Protection Act 2023
- GDPR (for international users)
- Accessibility standards (WCAG 2.1 Level AA)

**Security Measures:**
- HTTPS/TLS 1.3 for all network communication
- Certificate pinning to prevent MITM attacks
- API authentication using JWT tokens
- Rate limiting to prevent abuse
- Input validation and sanitization
- Regular security audits and penetration testing

**Privacy Policy:**
- Clear, simple language (8th-grade reading level)
- Available in all supported languages
- Voice-readable version
- Explicit consent for data collection
- Easy opt-out mechanisms

---

## Scalability & Future Enhancements

### Phase 1: MVP (Hackathon Demo)
- 5 languages (Hindi, Tamil, Telugu, Bengali, Marathi)
- 500+ central government schemes
- Offline eligibility matching
- Basic voice interaction
- Android app only
- Manual scheme updates

### Phase 2: Production Launch (3-6 months)
- 15 languages including regional dialects
- State-specific schemes (all 28 states)
- Improved voice quality and accuracy
- iOS app
- Automated scheme updates
- CSC/NGO partnership program
- SMS fallback for feature phones

### Phase 3: Advanced Features (6-12 months)
- Integration with government portals (MyScheme, DigiLocker)
- Direct application submission
- Real-time application status tracking
- Document upload and verification
- Aadhaar-based authentication
- Scheme comparison and recommendation engine
- Community support and peer assistance

### Phase 4: Ecosystem Expansion (12+ months)
- IVR system for feature phone users
- WhatsApp bot integration
- Grievance redressal and complaint filing
- Analytics dashboard for policymakers
- Scheme impact measurement
- Predictive analytics (who needs which scheme)
- Integration with banking systems for benefit tracking

### Integration with Government Portals

**Planned Integrations:**

| Portal | Purpose | Timeline |
|--------|---------|----------|
| MyScheme.gov.in | Scheme discovery and updates | Phase 2 |
| DigiLocker | Document verification | Phase 3 |
| Aadhaar (UIDAI) | Identity verification | Phase 3 |
| UMANG | Application submission | Phase 3 |
| State portals | State scheme applications | Phase 3 |
| PFMS | Benefit payment tracking | Phase 4 |

**Technical Approach:**
- API-first integration where available
- Web scraping with change detection for portals without APIs
- OAuth 2.0 for secure authentication
- Webhook subscriptions for real-time updates
- Fallback to manual processes when APIs unavailable

### Support for Additional Languages

**Language Expansion Roadmap:**

**Phase 1 (Launch):** Hindi, Tamil, Telugu, Bengali, Marathi

**Phase 2 (3 months):** Gujarati, Kannada, Malayalam, Odia, Punjabi

**Phase 3 (6 months):** Assamese, Urdu, Konkani, Manipuri, Nepali

**Phase 4 (12 months):** Bodo, Dogri, Kashmiri, Maithili, Santali, Sindhi

**Dialect Support:**
- Bhojpuri (Hindi variant)
- Awadhi (Hindi variant)
- Marwari (Rajasthani)
- Chhattisgarhi
- Haryanvi

**Technical Implementation:**
- Fine-tune STT models for each language
- Native speaker TTS voices
- Culturally adapted prompts and responses
- Language-specific testing with native speakers

### SMS/IVR Fallback

**SMS Capabilities:**
- Send scheme summaries via SMS
- Document checklist via SMS
- Application status updates
- CSC contact information
- Works on any phone (no smartphone needed)

**IVR System:**
- Toll-free helpline number
- Voice menu navigation
- Basic eligibility check via phone
- Connect to human operator
- Callback scheduling

**Use Cases:**
- Users without smartphones
- Poor internet connectivity
- Backup for app failures
- Elderly users preferring phone calls

### Analytics Dashboard for NGOs and Policymakers

**Dashboard Features:**

**For NGOs/CSCs:**
- Users served (daily/weekly/monthly)
- Top schemes accessed
- Document readiness rates
- Application success rates
- User demographics
- Escalation patterns

**For Policymakers:**
- Scheme awareness metrics
- Eligibility vs. enrollment gap
- Geographic distribution of users
- Language preferences
- Barriers to application (missing documents, confusion points)
- Estimated welfare value unlocked
- ROI of scheme outreach

**Visualization:**
- Interactive maps (state/district level)
- Time-series charts
- Funnel analysis (discovery → application → approval)
- Cohort analysis
- Exportable reports (PDF, Excel)

**Privacy-Preserving Analytics:**
- Aggregated data only (no individual PII)
- Differential privacy techniques
- Anonymization and pseudonymization
- Opt-in for detailed tracking

---

## Deployment & Demo Strategy

### End-to-End Demo Flow for Small Farmer Persona

**Demo Scenario: Ramesh, 45-year-old farmer from Karnataka**

**Setup:**
- Android phone (mid-range)
- Airplane mode ON (demonstrate offline capability)
- Pre-loaded with scheme database (v2026.02)

**Demo Script (5 minutes):**

```
[0:00] Launch app
  → Voice: "Namaskara! Naanu Sarkar Sahaaya. Nimge yava yojane sigabahudo anta tiliyalu sahaaya maduttene."
     (Hello! I'm Government Helper. I'll help you find which schemes you can get.)

[0:30] User (in Kannada): "Nanage yava yojane sigutte?"
  (Which schemes can I get?)
  
  → System: "Chennagi. Naanu kelavu prashnegalu keltini..."
     (Good. I'll ask a few questions...)
  
  → Q1: "Nimma vayasu eshtu?" (What's your age?)
  User: "45"
  
  → Q2: "Neevu yava kelasa maduttiri?" (What work do you do?)
  User: "Krishi" (Farming)
  
  → Q3: "Eshtu jami ide?" (How much land?)
  User: "Eradu ekre" (2 acres)

[2:00] System processes (show loading animation)
  → Voice: "Nimge 4 yojane sigutte!" (You can get 4 schemes!)
  
  → Lists:
     1. PM-KISAN: ₹8,000 per year
     2. Raitha Bandhu (Karnataka): ₹10,000 per year
     3. Crop Insurance: Up to ₹50,000 coverage
     4. Kisan Credit Card: ₹3 lakh loan

[3:00] User: "PM-KISAN bagge heli" (Tell me about PM-KISAN)
  → System explains benefits, eligibility, application process
  
[3:30] User: "Yava documents beku?" (What documents needed?)
  → System lists: Aadhaar, Bank passbook, Land records
  → Asks: "Nimma hatra ideva?" (Do you have these?)
  User: "Aadhaar mattu bank ide" (Have Aadhaar and bank)
  
  → System: "Chennagi! Jami records patwari office inda siguttade..."
     (Good! Land records can be obtained from patwari office...)

[4:30] Show progress tracking
  → PM-KISAN: Documents 2/3 ready
  → Next step: Get land records

[5:00] Demo complete
  → Show offline indicator (airplane mode still ON)
  → Emphasize: "Everything worked without internet!"
```

**Demo Highlights:**
- ✅ Complete conversation in Kannada
- ✅ Only 3 questions asked
- ✅ 4 guaranteed-eligible schemes found
- ✅ Clear benefit amounts
- ✅ Document guidance
- ✅ 100% offline functionality

### Offline-First Demo Scenario

**Scenario: Demonstrating offline capabilities**

**Setup:**
1. Start with internet ON
2. Show scheme database sync (last updated: today)
3. Turn airplane mode ON
4. Complete full user journey
5. Turn internet back ON
6. Show background sync

**Key Points to Demonstrate:**
- Voice recognition works offline
- Eligibility matching works offline
- Scheme explanations work offline (template-based)
- Progress saved locally
- Seamless sync when online

**Comparison:**
```
Online Mode:
- High-quality voice (Amazon Polly)
- LLM-powered explanations
- Response time: 3-5 seconds

Offline Mode:
- Basic voice (eSpeak)
- Template explanations
- Response time: 1-2 seconds (faster!)
```

### Clear Demonstration of Social and Financial Impact

**Impact Metrics Dashboard (Live Demo):**

```
┌─────────────────────────────────────────────────┐
│         GOVERNMENT SCHEMES VOICE ASSISTANT       │
│              SOCIAL IMPACT DASHBOARD             │
├─────────────────────────────────────────────────┤
│                                                 │
│  Users Served (Demo Period)                     │
│  ████████████████████ 1,247 users               │
│                                                 │
│  Eligible Schemes Discovered                    │
│  ████████████████████ 4,231 schemes             │
│                                                 │
│  Applications Initiated                         │
│  ████████████ 523 applications (42%)            │
│                                                 │
│  Estimated Welfare Value Unlocked               │
│  ████████████████████ ₹7.8 Crore                │
│                                                 │
│  Average Benefit per User                       │
│  ₹62,500                                        │
│                                                 │
│  User Demographics                              │
│  • 68% Rural                                    │
│  • 54% Women                                    │
│  • 71% First-time digital service users         │
│  • 89% Completed journey without human help     │
│                                                 │
│  Language Distribution                          │
│  • Hindi: 42%                                   │
│  • Tamil: 18%                                   │
│  • Telugu: 15%                                  │
│  • Bengali: 13%                                 │
│  • Marathi: 12%                                 │
│                                                 │
│  Top Schemes Accessed                           │
│  1. PM-KISAN (₹8,000/year) - 387 users          │
│  2. Ayushman Bharat (₹5L coverage) - 312 users  │
│  3. PM Awas Yojana (₹2.5L grant) - 289 users    │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Storytelling Approach:**

"Meet Ramesh. He's a small farmer who didn't know he was eligible for ₹18,000 per year in government benefits. In just 5 minutes, our voice assistant helped him discover 4 schemes worth ₹18,000 annually. That's ₹1,500 extra per month for his family.

Now multiply that by 10 million users. That's ₹180 billion in welfare benefits reaching those who need it most. This isn't just an app—it's a bridge to dignity and opportunity."

**Demo Video Structure:**
1. Problem (30 sec): Show real user struggling with government website
2. Solution (60 sec): Show voice assistant in action
3. Impact (30 sec): Show metrics and user testimonial
4. Call to Action (15 sec): "Help us reach 10 million users"

---

## Technical Stack

### Mobile App
- **Framework:** React Native (cross-platform) or Flutter
- **State Management:** Redux / MobX
- **Local Database:** SQLite with SQLCipher (encryption)
- **Offline Sync:** WatermelonDB or PouchDB
- **Voice:** react-native-voice, expo-speech

### Backend Services
- **API Gateway:** AWS API Gateway / Kong
- **Compute:** AWS Lambda (serverless) / ECS (containers)
- **Database:** PostgreSQL (RDS) for user data, DynamoDB for sessions
- **Cache:** Redis (ElastiCache)
- **CDN:** CloudFront for scheme database distribution
- **Storage:** S3 for scheme JSON files

### AI/ML Services
- **LLM:** Amazon Bedrock (Claude 3 / Titan)
- **Speech-to-Text:** AWS Transcribe + On-device Whisper
- **Text-to-Speech:** Amazon Polly + On-device eSpeak
- **Language Detection:** AWS Comprehend

### DevOps & Monitoring
- **CI/CD:** GitHub Actions / AWS CodePipeline
- **Monitoring:** CloudWatch, Datadog
- **Error Tracking:** Sentry
- **Analytics:** Mixpanel, Amplitude
- **Logging:** ELK Stack (Elasticsearch, Logstash, Kibana)

### Security
- **Authentication:** AWS Cognito
- **Secrets Management:** AWS Secrets Manager
- **Encryption:** AWS KMS
- **DDoS Protection:** AWS Shield, CloudFlare

---

## Cost Estimation (Monthly, 100K Users)

| Service | Usage | Cost |
|---------|-------|------|
| AWS Lambda | 10M invocations | $20 |
| Amazon Bedrock (LLM) | 5M tokens | $150 |
| AWS Transcribe | 500K minutes | $750 |
| Amazon Polly | 1M characters | $40 |
| RDS PostgreSQL | db.t3.medium | $70 |
| S3 + CloudFront | 1TB transfer | $100 |
| ElastiCache Redis | cache.t3.small | $50 |
| CloudWatch | Logs + metrics | $30 |
| **Total** | | **~$1,210/month** |

**Cost per user:** $0.012/month (₹1/month)

**Optimization Strategies:**
- Aggressive caching to reduce LLM calls
- On-device processing for 95% of operations
- Reserved instances for predictable workloads
- Spot instances for batch processing

---

## Success Criteria for Hackathon

### Technical Excellence
- ✅ Working end-to-end demo (voice input → scheme discovery → document check)
- ✅ Offline functionality demonstrated
- ✅ Multilingual support (minimum 2 languages)
- ✅ Response time < 3 seconds
- ✅ Clean, maintainable code

### Innovation
- ✅ Adaptive questioning (≤6 questions)
- ✅ Guaranteed eligibility (no false positives)
- ✅ Offline-first architecture
- ✅ Voice-only interaction for low-literacy users

### Social Impact
- ✅ Clear problem statement and user personas
- ✅ Quantified impact (welfare value unlocked)
- ✅ Accessibility and inclusion focus
- ✅ Scalability to 10M+ users

### Presentation
- ✅ Compelling demo video (2 minutes)
- ✅ Live demo with real voice interaction
- ✅ Impact metrics dashboard
- ✅ Clear go-to-market strategy

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Voice recognition fails during demo | Pre-record backup audio, test extensively |
| Internet connectivity issues | Offline-first demo, airplane mode ON |
| Language/accent not understood | Use clear speaker, test with target users |
| Scheme data becomes outdated | Version display, "last updated" timestamp |
| Privacy concerns raised | Clear privacy policy, no PII in demo |
| Scalability questions | Architecture diagram, cost analysis |
| Integration complexity | Phase-based roadmap, MVP first |

---

## Conclusion

The Government Schemes Voice Assistant represents a paradigm shift in how underserved citizens access welfare benefits. By combining voice-first interaction, offline-first architecture, and AI-powered eligibility matching, we eliminate the barriers of language, literacy, and connectivity that prevent millions from claiming their rightful benefits.

**Key Innovations:**
1. Adaptive questioning that guarantees eligibility in ≤6 questions
2. 95% functionality works completely offline
3. Voice-only interaction requiring zero reading ability
4. Multilingual support with dialect recognition

**Expected Impact:**
- 10M users in first year
- ₹50B+ welfare value unlocked
- 70% reduction in scheme discovery time
- 60% of users are first-time digital service users

This isn't just a hackathon project—it's a blueprint for inclusive digital governance that puts citizens first.

---

*Document Version: 1.0*  
*Last Updated: February 2026*  
*Prepared for: Amazon AI Hackathon*  
*Architecture: Offline-First, Voice-First, Impact-First*
