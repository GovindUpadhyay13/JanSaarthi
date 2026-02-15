# JanSaarthi 🎙️
### Government Schemes Voice Assistant - Bridging the Digital Divide

[![Hackathon](https://img.shields.io/badge/Amazon%20AI-Hackathon-orange)](https://github.com/GovindUpadhyay13/JanSaarthi)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Android](https://img.shields.io/badge/platform-Android-green.svg)](https://www.android.com/)

> **Making government welfare schemes accessible to every Indian through voice-first AI**

JanSaarthi is an AI-powered, voice-first mobile application that democratizes access to government welfare schemes for underserved Indian citizens. By combining speech recognition, natural language understanding, and offline-first architecture, it helps users discover eligible schemes in **≤6 questions** without requiring reading ability or internet connectivity.

---

## 🎯 The Problem

Over **₹500 billion** in government welfare benefits remain unclaimed annually in India due to:
- 🌐 **Language Barriers** - Portals primarily in English/Hindi, not accessible to 22+ languages
- 📱 **Low Digital Literacy** - 67% of rural India struggles with complex websites
- 🔍 **Poor Awareness** - Citizens unaware of schemes they qualify for
- 📶 **Connectivity Issues** - 40% of rural areas have unreliable internet
- 📋 **Complex Eligibility** - No clear "am I eligible?" answer

---

## 💡 Our Solution

JanSaarthi transforms government scheme discovery through:

### 🎙️ **Voice-First Interaction**
- Speak in your preferred language (Hindi, Tamil, Telugu, Bengali, Marathi)
- Zero reading ability required
- Natural conversational interface

### 🔒 **Guaranteed Eligibility**
- Adaptive questioning (max 6 questions)
- **100% accurate** eligibility matching
- No false positives or irrelevant schemes

### 📴 **95% Offline Functionality**
- Works completely without internet
- On-device voice processing
- Monthly scheme database sync

### 🌍 **Multilingual Support**
- 5 languages at launch, 15+ planned
- Dialect recognition and regional variations
- Culturally appropriate responses

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| **Adaptive Questioning** | Intelligent decision tree asks only relevant questions |
| **Voice-Only Mode** | Complete journey possible without looking at screen |
| **Document Readiness Check** | Identifies missing documents and how to obtain them |
| **Offline-First** | Core matching works without internet |
| **Privacy-First** | No permanent voice storage, encrypted local data |
| **Human Escalation** | Seamless handoff to CSC/NGO when needed |
| **Progress Tracking** | Resume applications anytime |

---

## 🏗️ Architecture

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
└───────────────────────────────────────��─────────────────────┘
```

### Core Components

- **Eligibility Engine** - Rule-based JSON matching (100% offline)
- **Voice Module** - On-device Whisper Tiny + AWS Transcribe fallback
- **LLM Layer** - Amazon Bedrock (Claude/Titan) with template fallback
- **Scheme Database** - 500+ schemes, versioned JSON, CDN-distributed
- **Human Escalation** - CSC locator, SMS gateway integration

---

## 🚀 Getting Started

### Prerequisites

- Android 8.0+ device
- 2GB RAM minimum
- 150MB free storage (50MB app + 100MB cache)

### Installation

```bash
# Clone the repository
git clone https://github.com/GovindUpadhyay13/JanSaarthi.git

# Navigate to project directory
cd JanSaarthi

# Install dependencies (example for React Native)
npm install
# or for Flutter
flutter pub get

# Run on Android
npm run android
# or
flutter run
```

### Configuration

Create a `.env` file with your API keys:

```env
AWS_BEDROCK_API_KEY=your_bedrock_key
AWS_TRANSCRIBE_REGION=ap-south-1
SCHEME_DB_CDN_URL=https://your-cdn.cloudfront.net
```

---

## 📖 Usage Example

**User Journey: Ramesh, 45-year-old farmer from Karnataka**

```
[User] "Namaskara! Nanage yava yojane sigutte?"
       (Hello! Which schemes can I get?)

[JanSaarthi] "Chennagi. Naanu kelavu prashnegalu keltini..."
             (Good. I'll ask a few questions...)

[Q1] "Nimma vayasu eshtu?" (What's your age?)
[User] "45"

[Q2] "Neevu yava kelasa maduttiri?" (What work do you do?)
[User] "Krishi" (Farming)

[Q3] "Eshtu jami ide?" (How much land?)
[User] "Eradu ekre" (2 acres)

[Result] "Nimge 4 yojane sigutte!" (You can get 4 schemes!)
         1. PM-KISAN: ₹8,000/year
         2. Raitha Bandhu: ₹10,000/year
         3. Crop Insurance: ₹50,000 coverage
         4. Kisan Credit Card: ₹3 lakh loan
```

---

## 🎯 Target Users

| Persona | Needs | How JanSaarthi Helps |
|---------|-------|---------------------|
| **Small Farmer** | Simple voice guidance, offline access | Voice-only Kannada interface, works offline |
| **Rural Laborer** | No reading required | Voice-first, illiterate-friendly design |
| **Elderly Citizen** | Patient explanations | Clear, slow voice responses |
| **Women Beneficiaries** | Quick sessions, privacy | Fast sessions, local data encryption |
| **CSC Operators** | Serve many users efficiently | Batch processing, progress tracking |

---

## 📊 Expected Impact

### Year 1 Goals

- 🎯 **10M+ users** reached
- 💰 **₹50B+** welfare value unlocked
- ⚡ **70% reduction** in scheme discovery time
- 📱 **60%** first-time digital service users

### Success Metrics

- ✅ 95%+ eligibility accuracy
- ✅ Zero false positives
- ✅ <5 minute average session
- ✅ 80% complete journey without human help
- ✅ 90%+ voice recognition accuracy

---

## 🛠️ Technology Stack

### Mobile App
- **Framework:** React Native / Flutter
- **Voice:** Whisper Tiny (on-device) + AWS Transcribe
- **TTS:** eSpeak (offline) + Amazon Polly (online)
- **Database:** SQLite with SQLCipher encryption
- **Offline Sync:** WatermelonDB / PouchDB

### Backend
- **API Gateway:** AWS API Gateway
- **Compute:** AWS Lambda (serverless)
- **LLM:** Amazon Bedrock (Claude 3 / Titan)
- **Database:** PostgreSQL (RDS) + DynamoDB
- **CDN:** CloudFront for scheme distribution
- **Storage:** S3 for scheme JSON files

### AI/ML
- **Speech-to-Text:** AWS Transcribe + Whisper
- **Text-to-Speech:** Amazon Polly + eSpeak
- **Language Detection:** AWS Comprehend
- **LLM Reasoning:** Amazon Bedrock

---

## 🗺️ Roadmap

### Phase 1: MVP (Current - Hackathon Demo)
- ✅ 5 languages (Hindi, Tamil, Telugu, Bengali, Marathi)
- ✅ 500+ central government schemes
- ✅ Offline eligibility matching
- ✅ Android app

### Phase 2: Production Launch (3-6 months)
- 📱 iOS app
- 🌍 15 languages + dialects
- 🏛️ State-specific schemes (28 states)
- 🤝 CSC/NGO partnerships
- 💬 SMS fallback for feature phones

### Phase 3: Advanced Features (6-12 months)
- 🔗 Government portal integration (MyScheme, DigiLocker)
- 📤 Direct application submission
- 📊 Real-time status tracking
- 🆔 Aadhaar-based authentication

### Phase 4: Ecosystem Expansion (12+ months)
- ☎️ IVR system for feature phones
- 💬 WhatsApp bot integration
- 📈 Analytics dashboard for policymakers
- 🏦 Banking system integration

---

## 🔒 Privacy & Security

- ❌ **No permanent voice storage** - Deleted within 5 seconds
- 🔐 **AES-256 encryption** for local data
- 🔑 **Device-specific keys** via Android Keystore
- 📊 **Minimal PII collection** - Only eligibility data
- ✅ **GDPR compliant** + Indian Data Protection Act

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

```bash
# Fork the repo
# Create a feature branch
git checkout -b feature/amazing-feature

# Commit your changes
git commit -m 'Add amazing feature'

# Push to the branch
git push origin feature/amazing-feature

# Open a Pull Request
```

---

## 📄 Documentation

- 📋 [Requirements Document](requirements.md) - Detailed functional & non-functional requirements
- 🏗️ [Design Document](design.md) - Complete architecture and technical design
- 🎨 [UI/UX Guidelines](docs/ui-guidelines.md) - Voice interaction patterns
- 🔌 [API Documentation](docs/api.md) - Backend API reference

---

## 📞 Support & Contact

- **Email:** govind.upadhyay13@example.com
- **Issues:** [GitHub Issues](https://github.com/GovindUpadhyay13/JanSaarthi/issues)
- **Discussions:** [GitHub Discussions](https://github.com/GovindUpadhyay13/JanSaarthi/discussions)

---

## 🙏 Acknowledgments

- Amazon AI Hackathon for the opportunity
- Government of India for open scheme data
- CSCs and NGOs for ground support
- Open-source community for amazing tools

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Made with ❤️ for Bharat**

*"This isn't just an app—it's a bridge to dignity and opportunity."*

[⭐ Star us on GitHub](https://github.com/GovindUpadhyay13/JanSaarthi) | [🐛 Report Bug](https://github.com/GovindUpadhyay13/JanSaarthi/issues) | [💡 Request Feature](https://github.com/GovindUpadhyay13/JanSaarthi/issues)

</div>
