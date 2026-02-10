# GramSetu AI

> Bridging the Digital Divide: Voice-First Government Services for Rural India

## Overview

GramSetu AI is a voice-first AI platform that provides rural citizens in India with seamless access to government schemes and services through natural language conversation. No internet, no app, no literacy barriers—just call and speak.

## The Problem

Millions of rural citizens in India are unable to access government benefits they're entitled to due to:
- **Digital Literacy Barriers**: Complex apps and websites require technical knowledge
- **Connectivity Issues**: Limited or no internet access in rural areas
- **Language Barriers**: Most services available only in English or Hindi
- **Semantic Divide**: Understanding bureaucratic language and processes
- **Access Limitations**: Need to travel to government offices for simple tasks

## The Solution

GramSetu AI provides a "No-UI" voice bridge that allows citizens to:
- **Call from any phone** (PSTN or mobile, no internet required)
- **Speak naturally** in Hinglish or 12+ regional dialects
- **Get real answers** about government schemes and eligibility
- **Take action** by filing grievances and completing applications
- **Receive confirmation** with reference numbers via voice and SMS

### Key Features

🎙️ **Natural Voice Interaction**
- Full-duplex conversation with interruption handling
- Sub-second response latency (< 800ms)
- Support for 12+ Indian languages and dialects

🔐 **Secure Authentication**
- Voice biometric authentication (95%+ accuracy)
- Voice OTP fallback system
- Aadhaar integration compliance

🤖 **Intelligent Understanding**
- Context-aware slot filling through natural conversation
- Intent recognition and confidence scoring
- Crisis detection and emergency escalation

📚 **Government Knowledge Base**
- RAG-based policy retrieval from 50+ schemes
- Real-time eligibility checking via UMANG APIs
- Grievance filing through CPGRAMS integration

👥 **Human-in-the-Loop**
- Automatic handoff when AI confidence is low
- Operator dashboard with conversation context
- Callback scheduling when operators unavailable

📊 **Enterprise-Grade Reliability**
- 1000+ concurrent call support
- 99.5% uptime during business hours
- Session resumption after dropped calls
- Auto-scaling for traffic spikes


## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      GramSetu AI Platform                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  [Rural Citizen] ──PSTN/Mobile──> [Telephony Layer]         │
│                                          │                    │
│                                          ▼                    │
│                              [Orchestration: Vapi.ai]        │
│                                    /    |    \                │
│                                   /     |     \               │
│                                  ▼      ▼      ▼              │
│                              [STT]  [TTS]  [Session]         │
│                                   \     |     /               │
│                                    ▼    ▼    ▼                │
│                          [Cognitive Intelligence]            │
│                              /      |      \                  │
│                             /       |       \                 │
│                            ▼        ▼        ▼                │
│                      [Intent]  [RAG]  [Confidence]           │
│                            |        |        |                │
│                            ▼        ▼        ▼                │
│                      [Slots] [Pinecone] [Handoff]            │
│                                     |                         │
│                                     ▼                         │
│                          [Action Layer: MCP Server]          │
│                                /         \                    │
│                               ▼           ▼                   │
│                         [UMANG API]  [CPGRAMS API]           │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

## Technology Stack

### Core Components
- **Telephony**: Twilio / Telnyx SIP Trunking
- **Orchestration**: Vapi.ai
- **Speech-to-Text**: Deepgram Nova-2, Sarvam AI
- **LLM**: Llama 3 on Groq (primary), GPT-4 (fallback)
- **Text-to-Speech**: Vapi.ai integrated TTS
- **Vector Database**: Pinecone
- **Action Layer**: Python FastAPI MCP Server
- **Government APIs**: UMANG, CPGRAMS
- **Deployment**: AWS Lambda, Cloud Infrastructure

### Testing & Quality
- **Property-Based Testing**: Hypothesis
- **Unit Testing**: pytest
- **Load Testing**: Locust
- **Monitoring**: CloudWatch / Datadog

## Getting Started

### Prerequisites
- Python 3.9+
- AWS Account (for Lambda deployment)
- API Keys for:
  - Twilio or Telnyx
  - Vapi.ai
  - Deepgram
  - Groq / OpenAI
  - Pinecone
  - UMANG and CPGRAMS (government APIs)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/gramsetu-ai.git
cd gramsetu-ai

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your API keys
```

### Configuration

Create a `.env` file with the following:

```env
# Telephony
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
TWILIO_PHONE_NUMBER=your_phone_number

# Vapi.ai
VAPI_API_KEY=your_vapi_key

# Speech Services
DEEPGRAM_API_KEY=your_deepgram_key
SARVAM_API_KEY=your_sarvam_key

# LLM
GROQ_API_KEY=your_groq_key
OPENAI_API_KEY=your_openai_key

# Vector Database
PINECONE_API_KEY=your_pinecone_key
PINECONE_ENVIRONMENT=your_environment

# Government APIs
UMANG_API_KEY=your_umang_key
CPGRAMS_API_KEY=your_cpgrams_key

# AWS
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
AWS_REGION=ap-south-1
```


### Running Locally

```bash
# Start the MCP server
python -m gramsetu.action_layer.server

# Run tests
pytest tests/

# Run property-based tests
pytest tests/properties/

# Load testing
locust -f tests/load/locustfile.py
```

### Deployment

```bash
# Deploy MCP server to AWS Lambda
serverless deploy

# Update Pinecone knowledge base
python scripts/update_knowledge_base.py

# Monitor system health
python scripts/monitor.py
```

## Usage Example

### User Flow

1. **User calls**: `1800-XXX-XXXX`
2. **System greets**: "Namaste! GramSetu AI mein aapka swagat hai. Kripya apni bhasha chunein..."
3. **User selects**: "Hindi"
4. **System asks**: "Main aapki kaise madad kar sakta hoon?"
5. **User speaks**: "Mujhe PM Kisan Yojana ke baare mein jaanna hai"
6. **System retrieves**: Policy information from knowledge base
7. **System responds**: "PM Kisan Yojana ek scheme hai jismein..."
8. **System offers**: "Kya aap apni eligibility check karna chahenge?"
9. **User confirms**: "Haan"
10. **System collects**: Name, location, land details through conversation
11. **System checks**: Eligibility via UMANG API
12. **System informs**: "Aap eligible hain! Kya main aapka application file kar doon?"
13. **User confirms**: "Haan"
14. **System files**: Application and provides reference number
15. **System confirms**: "Aapka reference number hai ABC123. SMS bhi bheja gaya hai."

## Project Structure

```
gramsetu-ai/
├── .kiro/
│   └── specs/
│       └── gramsetu-ai/
│           ├── requirements.md    # Detailed requirements
│           ├── design.md          # Architecture & design
│           └── tasks.md           # Implementation tasks
├── gramsetu/
│   ├── telephony/                 # Telephony layer
│   ├── orchestration/             # Vapi.ai integration
│   ├── stt/                       # Speech-to-text
│   ├── cognitive/                 # LLM & intent parsing
│   ├── knowledge/                 # RAG & Pinecone
│   ├── action_layer/              # MCP server & APIs
│   ├── auth/                      # Voice biometrics & OTP
│   ├── session/                   # Session management
│   ├── handoff/                   # Human handoff
│   ├── crisis/                    # Crisis detection
│   └── analytics/                 # Monitoring & metrics
├── tests/
│   ├── unit/                      # Unit tests
│   ├── properties/                # Property-based tests
│   ├── integration/               # Integration tests
│   └── load/                      # Load tests
├── scripts/
│   ├── update_knowledge_base.py   # Update policies
│   ├── monitor.py                 # System monitoring
│   └── deploy.py                  # Deployment scripts
├── docs/                          # Additional documentation
├── requirements.txt               # Python dependencies
├── serverless.yml                 # Serverless config
├── .env.example                   # Environment template
└── README.md                      # This file
```

## Key Capabilities

### Supported Languages
- Hindi
- English
- Hinglish
- Tamil
- Telugu
- Bengali
- Marathi
- Gujarati
- Kannada
- Malayalam
- Punjabi
- Odia

### Government Services
- **Eligibility Checking**: PM Kisan, Ayushman Bharat, MGNREGA, etc.
- **Grievance Filing**: CPGRAMS integration for all departments
- **Application Assistance**: Step-by-step guidance for scheme applications
- **Status Tracking**: Check application and grievance status
- **Information Retrieval**: Policy details, required documents, deadlines

### Performance Metrics
- **Connection Time**: < 3 seconds
- **Audio Latency**: < 150ms
- **Response Time**: < 800ms
- **Transcription Accuracy**: 85% (standard), 75% (dialects)
- **Voice Auth Accuracy**: 95%+
- **Concurrent Calls**: 1000+
- **Uptime**: 99.5% (business hours)


## Development

### Running Tests

```bash
# Run all tests
pytest

# Run specific test suite
pytest tests/unit/
pytest tests/properties/
pytest tests/integration/

# Run with coverage
pytest --cov=gramsetu tests/

# Run property-based tests with verbose output
pytest tests/properties/ -v --hypothesis-show-statistics
```

### Code Quality

```bash
# Format code
black gramsetu/

# Lint
flake8 gramsetu/
pylint gramsetu/

# Type checking
mypy gramsetu/
```

### Adding New Government Schemes

1. Add policy document to `data/policies/`
2. Run knowledge base update:
   ```bash
   python scripts/update_knowledge_base.py --add data/policies/new_scheme.pdf
   ```
3. Update eligibility logic in `gramsetu/action_layer/eligibility.py`
4. Add tests in `tests/unit/test_eligibility.py`

### Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-feature`
3. Make your changes
4. Run tests: `pytest`
5. Commit: `git commit -am 'Add new feature'`
6. Push: `git push origin feature/new-feature`
7. Create a Pull Request

## Security & Compliance

### Data Protection
- **Encryption in Transit**: TLS 1.3
- **Encryption at Rest**: AES-256
- **Voice Data Retention**: 90 days (configurable)
- **PII Handling**: Minimal collection, encrypted storage

### Compliance
- ✅ Personal Data Protection Act (PDPA)
- ✅ IT Act 2000
- ✅ Aadhaar Authentication Standards
- ✅ OWASP Top 10 Security Practices

### Authentication
- Voice biometric profiles encrypted at rest
- OTP valid for 5 minutes only
- API tokens auto-refreshed
- Role-based access control for operators

## Monitoring & Operations

### Dashboards
- **Operations**: Real-time call metrics, error rates, system health
- **Business**: Daily usage, success rates, user satisfaction
- **Performance**: Latency trends, throughput, resource utilization
- **Quality**: Transcription accuracy, intent recognition, handoff reasons

### Alerts
- **Critical**: System down, error rate > 10%, latency > 2x normal
- **Warning**: Error rate > 5%, latency > 1.5x normal, disk > 80%
- **Info**: Deployments, configuration changes, maintenance

### Maintenance
- **Daily**: Monitor dashboards, review error logs
- **Weekly**: Performance trends, user feedback analysis, knowledge base updates
- **Monthly**: Security patches, dependency updates, capacity planning
- **Quarterly**: Disaster recovery drills, security audits, cost optimization

## Roadmap

### Phase 1: MVP (Completed)
- ✅ Basic call flow with Hindi/English
- ✅ Policy information retrieval
- ✅ Eligibility checking
- ✅ Grievance filing

### Phase 2: Enhancement (In Progress)
- 🔄 Regional language support (12+ languages)
- 🔄 Voice authentication
- 🔄 Session resumption
- 🔄 Human handoff system

### Phase 3: Scale (Q2 2026)
- 📅 1000+ concurrent call support
- 📅 Multi-region deployment
- 📅 Advanced analytics dashboard
- 📅 Proactive notifications

### Phase 4: Innovation (Q3-Q4 2026)
- 📅 Multi-modal support (SMS/WhatsApp)
- 📅 Predictive analytics for user needs
- 📅 Video calling for document verification
- 📅 Blockchain-based audit trails
- 📅 Integration with more government services

## Impact

### Target Metrics
- **Users Served**: 1M+ rural citizens in first year
- **Grievances Filed**: 100K+ through the platform
- **Eligibility Checks**: 500K+ completed
- **Time Saved**: 10M+ hours (vs. office visits)
- **User Satisfaction**: 4+ stars average rating

### Social Impact
- Bridge the digital divide for rural India
- Enable access to government benefits without literacy barriers
- Reduce corruption through transparent, automated processes
- Empower citizens with information and agency
- Save time and money for rural families

## Support

### Documentation
- [Requirements Document](.kiro/specs/gramsetu-ai/requirements.md)
- [Design Document](.kiro/specs/gramsetu-ai/design.md)
- [API Documentation](docs/api.md)
- [Deployment Guide](docs/deployment.md)

### Community
- **Issues**: [GitHub Issues](https://github.com/your-org/gramsetu-ai/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-org/gramsetu-ai/discussions)
- **Email**: support@gramsetu.ai

### Commercial Support
For enterprise deployments, custom integrations, or dedicated support:
- **Email**: enterprise@gramsetu.ai
- **Website**: https://gramsetu.ai

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Government of India for UMANG and CPGRAMS APIs
- Vapi.ai for real-time voice orchestration
- Deepgram and Sarvam AI for speech recognition
- Groq for fast LLM inference
- Open source community for tools and libraries

## Citation

If you use GramSetu AI in your research or project, please cite:

```bibtex
@software{gramsetu_ai_2026,
  title = {GramSetu AI: Voice-First Government Services for Rural India},
  author = {GramSetu Team},
  year = {2026},
  url = {https://github.com/your-org/gramsetu-ai}
}
```

---

**Built with ❤️ for Rural India**

*Empowering citizens through voice, one call at a time.*
