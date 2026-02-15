# Government Schemes Voice Assistant - Requirements Document

## Executive Summary

Over ₹500 billion in government welfare benefits remain unclaimed annually in India due to language barriers, low digital literacy, complex portals, and poor awareness. This disproportionately affects farmers, rural laborers, elderly citizens, and marginalized communities who need these schemes most.

The Government Schemes Voice Assistant is a voice-first, multilingual AI system that helps eligible citizens discover, understand, and access government welfare schemes through natural conversation. By asking ≤6 adaptive questions, the system guarantees eligibility matches, explains benefits in regional languages, guides document preparation, and works offline in low-bandwidth environments.

**Expected Impact:**
- Enable 10M+ underserved citizens to discover eligible schemes
- Unlock ₹50B+ in welfare benefits in first year
- Reduce scheme discovery time from hours to <5 minutes
- Bridge the digital divide through voice-first accessibility

---

## Target Users & Personas

### 1. Small Farmer - Ramesh (45, Karnataka)
- Owns 1.5 acres of land
- Speaks only Kannada
- Basic feature phone, limited internet
- Unaware of PM-KISAN, crop insurance schemes
- Needs: Simple voice guidance, offline access

### 2. Rural Daily Wage Laborer - Lakshmi (38, Bihar)
- Works in construction
- Illiterate, speaks Bhojpuri/Hindi
- Borrowed smartphone from neighbor
- Eligible for MGNREGA, housing schemes
- Needs: Voice-only interface, no reading required

### 3. Elderly Citizen - Gopal (72, Tamil Nadu)
- Retired government employee
- Speaks Tamil, limited English
- Struggles with websites and apps
- Eligible for pension, healthcare schemes
- Needs: Patient, clear voice explanations

### 4. Woman Beneficiary - Fatima (29, Uttar Pradesh)
- Homemaker, primary school education
- Speaks Urdu/Hindi
- Limited phone access (shared device)
- Eligible for women empowerment, skill training schemes
- Needs: Quick sessions, privacy-conscious design

### 5. CSC Operator / NGO Worker - Priya (32, Maharashtra)
- Assists 50+ villagers monthly
- Bilingual (Marathi/English)
- Needs efficient tool to serve multiple users
- Requires: Batch eligibility checking, progress tracking

---

## User Problems

### Language Barriers
- Government portals primarily in English/Hindi
- 22+ official languages, hundreds of dialects
- Technical jargon incomprehensible to rural users
- No voice-based alternatives for non-readers

### Low Digital Literacy
- 67% of rural India has limited smartphone experience
- Fear of making mistakes on government websites
- Unable to navigate complex multi-step forms
- Difficulty understanding eligibility criteria

### Lack of Awareness
- Citizens unaware of schemes they qualify for
- Information scattered across multiple websites
- No centralized discovery mechanism
- Schemes announced but not communicated to target users

### Complex Eligibility Criteria
- Multiple overlapping conditions (age, income, caste, location)
- Confusing documentation requirements
- No clear "am I eligible?" answer
- Users waste time on irrelevant schemes

### Poor Internet Connectivity
- 40% of rural areas have unreliable internet
- High data costs for low-income users
- Government portals require constant connectivity
- No offline alternatives available

---

## Functional Requirements

### FR1: Voice-First Interaction
- **FR1.1** Accept voice input in user's preferred language
- **FR1.2** Provide voice output for all responses
- **FR1.3** Support conversational, natural language queries
- **FR1.4** Handle accents, dialects, and speech variations
- **FR1.5** Allow voice interruption and correction

### FR2: Multilingual Support
- **FR2.1** Support minimum 5 languages at launch: Hindi, Tamil, Telugu, Bengali, Marathi
- **FR2.2** Language auto-detection from first utterance
- **FR2.3** Seamless language switching mid-conversation
- **FR2.4** Culturally appropriate responses and examples
- **FR2.5** Roadmap for 15+ languages within 6 months

### FR3: AI-Driven Eligibility Matching
- **FR3.1** Adaptive questioning based on user responses
- **FR3.2** Maximum 6 questions to determine eligibility
- **FR3.3** Context-aware follow-up questions
- **FR3.4** Infer information to minimize user burden
- **FR3.5** Confidence scoring for each scheme match

### FR4: Guaranteed-Eligibility Scheme Listing
- **FR4.1** Display only schemes user is 100% eligible for
- **FR4.2** Rank schemes by benefit value and urgency
- **FR4.3** Explain why user qualifies for each scheme
- **FR4.4** No false positives or irrelevant suggestions
- **FR4.5** Highlight time-sensitive schemes (deadlines)

### FR5: Voice-Based Scheme Explanation
- **FR5.1** Explain scheme benefits in simple language
- **FR5.2** Describe application process step-by-step
- **FR5.3** Provide examples relevant to user's context
- **FR5.4** Answer follow-up questions about schemes
- **FR5.5** Summarize key information (benefit amount, timeline)

### FR6: Document Readiness Check
- **FR6.1** List all required documents for each scheme
- **FR6.2** Check which documents user already has
- **FR6.3** Explain how to obtain missing documents
- **FR6.4** Provide alternative document options where applicable
- **FR6.5** Estimate time needed to gather documents

### FR7: Offline / Low-Bandwidth Support
- **FR7.1** Core eligibility matching works completely offline
- **FR7.2** Cache scheme database locally (updated monthly)
- **FR7.3** Voice processing on-device where possible
- **FR7.4** Graceful degradation when offline
- **FR7.5** Background sync when connectivity available

### FR8: Application Progress Tracking
- **FR8.1** Track status: Not Started / In Progress / Submitted / Approved
- **FR8.2** Voice-based status updates
- **FR8.3** Reminders for pending actions
- **FR8.4** Estimated processing timelines
- **FR8.5** Sync progress across devices (when online)

### FR9: Human Escalation
- **FR9.1** Detect when user is confused or stuck
- **FR9.2** Provide nearest CSC / NGO contact information
- **FR9.3** Offer helpline numbers for specific schemes
- **FR9.4** Generate summary for human helper
- **FR9.5** SMS fallback for critical information

---

## Non-Functional Requirements

### NFR1: Performance
- **NFR1.1** Voice response time < 2 seconds for offline flows
- **NFR1.2** Voice response time < 5 seconds for online flows
- **NFR1.3** App startup time < 3 seconds
- **NFR1.4** Scheme database load time < 1 second
- **NFR1.5** Support 1000+ concurrent users per server

### NFR2: Device Compatibility
- **NFR2.1** Run on Android 8.0+ devices
- **NFR2.2** Function on devices with 2GB RAM
- **NFR2.3** App size < 50MB (excluding voice models)
- **NFR2.4** Offline cache < 100MB
- **NFR2.5** Battery consumption < 5% per 10-minute session

### NFR3: Offline-First Design
- **NFR3.1** 80% of features work without internet
- **NFR3.2** Automatic sync when connectivity restored
- **NFR3.3** Clear indicators for online-only features
- **NFR3.4** No data loss during offline usage
- **NFR3.5** Conflict resolution for offline edits

### NFR4: Data Privacy
- **NFR4.1** No permanent storage of voice recordings
- **NFR4.2** Voice data deleted after processing
- **NFR4.3** Minimal PII collection (only what's required for eligibility)
- **NFR4.4** Local encryption for cached user data
- **NFR4.5** GDPR/Indian data protection compliance

### NFR5: Reliability
- **NFR5.1** 99.5% uptime for online services
- **NFR5.2** Graceful error handling with user-friendly messages
- **NFR5.3** Automatic retry for failed operations
- **NFR5.4** Data integrity checks for scheme database
- **NFR5.5** Rollback capability for failed updates

### NFR6: Scalability
- **NFR6.1** Support 10M users in first year
- **NFR6.2** Horizontal scaling for backend services
- **NFR6.3** CDN for scheme database distribution
- **NFR6.4** Load balancing for voice processing
- **NFR6.5** Database sharding by state/region

---

## Accessibility & Inclusion Requirements

### AR1: Voice-Only Mode
- **AR1.1** Complete user journey possible without reading
- **AR1.2** No dependency on visual UI elements
- **AR1.3** Audio cues for all interactions
- **AR1.4** Voice confirmation for critical actions
- **AR1.5** Adjustable speech rate and volume

### AR2: Simple, Conversational Language
- **AR2.1** Avoid technical jargon and bureaucratic terms
- **AR2.2** Use everyday language and local idioms
- **AR2.3** Short sentences (< 15 words)
- **AR2.4** Active voice, direct instructions
- **AR2.5** Contextual examples from user's life

### AR3: Regional Language Support
- **AR3.1** Native speaker voice quality
- **AR3.2** Regional variations and dialects
- **AR3.3** Culturally appropriate greetings and phrases
- **AR3.4** Local scheme names and terminology
- **AR3.5** Regional festival and calendar awareness

### AR4: Low Digital Skill Support
- **AR4.1** Guided onboarding with voice tutorial
- **AR4.2** Forgiving input (accept variations, errors)
- **AR4.3** Proactive help when user hesitates
- **AR4.4** Repeat and clarify options on request
- **AR4.5** No penalty for mistakes or back-tracking

### AR5: Inclusive Design
- **AR5.1** Gender-neutral language options
- **AR5.2** Support for users with visual impairments
- **AR5.3** Support for users with hearing impairments (text fallback)
- **AR5.4** Elderly-friendly pacing and clarity
- **AR5.5** Child-friendly mode for young beneficiaries

---

## Success Metrics

### User Acquisition & Engagement
- **M1** 100,000 users in first 3 months
- **M2** 60% user retention after first interaction
- **M3** Average 3 sessions per user per month
- **M4** 80% of users complete eligibility flow
- **M5** Net Promoter Score (NPS) > 50

### Eligibility & Accuracy
- **M6** Eligibility match accuracy > 95%
- **M7** Zero false positives (no ineligible schemes shown)
- **M8** Average 3.2 eligible schemes per user
- **M9** 90% user confidence in recommendations
- **M10** < 5% escalation to human support

### Application & Impact
- **M11** 40% of users initiate at least one application
- **M12** ₹50B+ estimated welfare value unlocked in year 1
- **M13** Average ₹15,000 benefit value per user
- **M14** 70% of users gather required documents within 2 weeks
- **M15** 25% application approval rate within 3 months

### Technical Performance
- **M16** 95% of sessions completed offline
- **M17** Average session duration < 8 minutes
- **M18** Voice recognition accuracy > 90%
- **M19** App crash rate < 0.1%
- **M20** Average response time < 2.5 seconds

### Accessibility & Inclusion
- **M21** 70% of users are first-time digital service users
- **M22** 50% of users are women
- **M23** 40% of users are from rural areas
- **M24** Support for 5 languages at launch, 15 within 6 months
- **M25** 80% of users complete journey without human help

---

## Out of Scope (Phase 1)

- Direct application submission to government portals
- Payment processing or financial transactions
- Real-time application status from government systems
- Video-based verification or KYC
- Integration with Aadhaar authentication
- Scheme comparison and recommendation engine
- Community forums or peer support
- Grievance redressal and complaint filing

---

## Assumptions & Dependencies

### Assumptions
- Users have access to Android smartphone (owned or borrowed)
- Basic phone literacy (can answer/end calls)
- Willingness to share basic demographic information
- Government scheme data is publicly available
- CSC/NGO network available for human escalation

### Dependencies
- Government scheme database access and updates
- Speech-to-text and text-to-speech APIs
- LLM API for reasoning (with offline fallback)
- Cloud infrastructure for backend services
- Partnership with CSCs/NGOs for ground support

---

## Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| Inaccurate eligibility matching | High | Rigorous testing, human review, confidence thresholds |
| Poor voice recognition for dialects | High | Dialect-specific training, fallback to text input |
| Scheme data becomes outdated | Medium | Monthly automated updates, version control |
| Low user trust in AI recommendations | Medium | Transparency, explain reasoning, human escalation |
| Internet dependency limits reach | High | Offline-first architecture, 80% features work offline |
| Privacy concerns with voice data | High | No permanent storage, local processing, clear privacy policy |
| Device compatibility issues | Medium | Extensive device testing, minimum spec requirements |
| User abandonment mid-flow | Medium | Save progress, resume capability, SMS reminders |

---

*Document Version: 1.0*  
*Last Updated: February 2026*  
*Prepared for: Amazon AI Hackathon*
