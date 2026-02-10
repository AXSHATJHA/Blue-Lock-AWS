# Requirements Document: GramSetu AI

## Introduction

GramSetu AI is a voice-first AI platform designed to bridge the "Semantic Divide" and "Digital Literacy" barrier that prevents rural citizens in India from accessing government schemes and services. The system provides a "No-UI" voice-bridge that allows users to call from any phone (no internet required) and speak naturally in Hinglish or local dialects. Unlike traditional IVR systems or smartphone apps, GramSetu AI moves beyond information retrieval to provide Transactional Agency - the AI checks eligibility and files grievances directly through government databases.

## Glossary

- **GramSetu_System**: The complete voice-first AI platform including telephony, orchestration, cognitive intelligence, and action layers
- **Voice_Bridge**: The telephony interface that connects PSTN calls to the cloud-based AI system
- **Orchestration_Engine**: The Vapi.ai-based component managing real-time voice conversations with sub-second latency
- **Cognitive_Intelligence**: The LLM-based reasoning system (Llama 3 on Groq or GPT-4) that understands user intent
- **Acoustic_Model**: The speech recognition system (Deepgram Nova-2 or Sarvam AI) that converts speech to text
- **Knowledge_Base**: The Pinecone Vector DB containing government policy documents for RAG
- **Action_Layer**: The AWS Lambda + API Setu integration that executes transactions with UMANG and CPGRAMS
- **UMANG**: Unified Mobile Application for New-age Governance - government service platform
- **CPGRAMS**: Centralized Public Grievance Redress and Monitoring System
- **Voice_OTP**: Voice-based One-Time Password for user authentication
- **Slot_Filling**: The process of collecting required information (name, location, issue) through natural conversation
- **Human_Handoff**: The process of transferring a call to a human operator when AI confidence is low
- **Crisis_Detection**: The capability to identify urgent or emergency situations requiring immediate escalation
- **Full_Duplex**: The ability to handle simultaneous speaking and listening (natural interruption handling)
- **RAG**: Retrieval-Augmented Generation - using vector search to provide context to the LLM

## Requirements

### Requirement 1: Voice Bridge Connectivity

**User Story:** As a rural citizen, I want to call GramSetu AI from any phone without internet, so that I can access government services regardless of my connectivity or device.

#### Acceptance Criteria

1. WHEN a user dials the GramSetu phone number from any PSTN or mobile network, THEN THE Voice_Bridge SHALL establish a connection within 3 seconds
2. WHEN a call is connected, THEN THE Voice_Bridge SHALL maintain audio quality with less than 150ms latency
3. WHEN network conditions degrade, THEN THE Voice_Bridge SHALL maintain the call connection and notify the user of audio quality issues
4. THE Voice_Bridge SHALL support simultaneous connections for at least 1000 concurrent users
5. WHEN a call is dropped due to network issues, THEN THE GramSetu_System SHALL allow the user to resume their session within 5 minutes by calling back

### Requirement 2: Language Selection and Recognition

**User Story:** As a rural citizen, I want to speak in my preferred language or dialect, so that I can communicate naturally without language barriers.

#### Acceptance Criteria

1. WHEN a call is initiated, THEN THE GramSetu_System SHALL prompt the user to select their preferred language from Hindi, English, and regional dialects
2. WHEN a user speaks in their selected language, THEN THE Acoustic_Model SHALL transcribe speech with at least 85% accuracy for standard Hindi/English and 75% accuracy for regional dialects
3. WHEN a user switches languages mid-conversation, THEN THE GramSetu_System SHALL detect the language change and adapt within 2 seconds
4. THE Acoustic_Model SHALL support Hindi, English, Hinglish, Tamil, Telugu, Bengali, Marathi, Gujarati, Kannada, Malayalam, Punjabi, and Odia
5. WHEN dialect-specific vocabulary is used, THEN THE Acoustic_Model SHALL prioritize Sarvam AI for Indic speech recognition

### Requirement 3: Context-Aware Slot Filling

**User Story:** As a rural citizen, I want to provide information through natural conversation, so that I don't have to navigate robotic menus or remember specific formats.

#### Acceptance Criteria

1. WHEN collecting user information, THEN THE Cognitive_Intelligence SHALL extract required slots (name, location, issue) from natural conversation without explicit prompts
2. WHEN a required slot is missing, THEN THE Cognitive_Intelligence SHALL ask clarifying questions in a conversational manner
3. WHEN a user provides information out of order, THEN THE GramSetu_System SHALL accept and process the information correctly
4. WHEN ambiguous information is provided, THEN THE Cognitive_Intelligence SHALL ask for clarification before proceeding
5. THE GramSetu_System SHALL complete slot filling for a typical grievance in under 2 minutes of conversation time

### Requirement 4: Voice Authentication

**User Story:** As a rural citizen, I want to authenticate my identity using my voice, so that I can access services securely without remembering passwords or PINs.

#### Acceptance Criteria

1. WHEN a user calls for the first time, THEN THE GramSetu_System SHALL create a voice profile by recording a passphrase
2. WHEN a returning user calls, THEN THE GramSetu_System SHALL verify their identity using voice biometrics with at least 95% accuracy
3. WHERE voice biometric authentication fails, THEN THE GramSetu_System SHALL fall back to Voice_OTP sent via SMS or voice call
4. WHEN Voice_OTP is used, THEN THE GramSetu_System SHALL generate a 6-digit code valid for 5 minutes
5. THE GramSetu_System SHALL comply with Aadhaar authentication standards where applicable

### Requirement 5: Full-Duplex Conversation Handling

**User Story:** As a rural citizen, I want to interrupt the AI when needed, so that I can have a natural conversation without waiting for the AI to finish speaking.

#### Acceptance Criteria

1. WHEN a user speaks while the AI is speaking, THEN THE Orchestration_Engine SHALL detect the interruption within 300ms
2. WHEN an interruption is detected, THEN THE Orchestration_Engine SHALL stop speaking and listen to the user
3. WHEN a user pauses mid-sentence, THEN THE Orchestration_Engine SHALL wait at least 1.5 seconds before responding
4. THE Orchestration_Engine SHALL maintain conversation context across interruptions
5. WHEN background noise causes false interruptions, THEN THE Orchestration_Engine SHALL filter noise and only respond to genuine speech

### Requirement 6: Government Scheme Eligibility Checking

**User Story:** As a rural citizen, I want the AI to check if I'm eligible for government schemes, so that I can know which benefits I can access.

#### Acceptance Criteria

1. WHEN a user inquires about a government scheme, THEN THE Cognitive_Intelligence SHALL retrieve eligibility criteria from the Knowledge_Base
2. WHEN eligibility criteria are retrieved, THEN THE Cognitive_Intelligence SHALL ask the user for required information to assess eligibility
3. WHEN all required information is collected, THEN THE Action_Layer SHALL query UMANG APIs to verify eligibility
4. WHEN eligibility is confirmed, THEN THE GramSetu_System SHALL inform the user and offer to help with application
5. WHEN a user is ineligible, THEN THE GramSetu_System SHALL explain why and suggest alternative schemes

### Requirement 7: Grievance Filing

**User Story:** As a rural citizen, I want to file grievances directly through the AI, so that I can report issues without visiting government offices or using complex apps.

#### Acceptance Criteria

1. WHEN a user wants to file a grievance, THEN THE Cognitive_Intelligence SHALL collect all required information through natural conversation
2. WHEN all information is collected, THEN THE Action_Layer SHALL submit the grievance to CPGRAMS via API
3. WHEN a grievance is submitted, THEN THE GramSetu_System SHALL provide the user with a reference number via voice
4. WHEN a grievance submission fails, THEN THE GramSetu_System SHALL retry up to 3 times before escalating to human handoff
5. THE Action_Layer SHALL send a confirmation SMS with the grievance reference number to the user's registered mobile

### Requirement 8: Knowledge Base RAG System

**User Story:** As a rural citizen, I want accurate information about government policies, so that I can make informed decisions about which services to access.

#### Acceptance Criteria

1. WHEN a user asks about a government policy, THEN THE Knowledge_Base SHALL retrieve relevant documents using vector similarity search
2. WHEN relevant documents are retrieved, THEN THE Cognitive_Intelligence SHALL generate a response based on the retrieved context
3. THE Knowledge_Base SHALL contain policy documents for at least 50 major central and state government schemes
4. WHEN policy information is outdated, THEN THE GramSetu_System SHALL indicate the last update date
5. THE Knowledge_Base SHALL be updated within 48 hours when new policies are published

### Requirement 9: Low Confidence Human Handoff

**User Story:** As a rural citizen, I want to speak with a human operator when the AI cannot help me, so that I can still get assistance with complex issues.

#### Acceptance Criteria

1. WHEN THE Cognitive_Intelligence confidence score falls below 60%, THEN THE GramSetu_System SHALL offer to transfer the call to a human operator
2. WHEN a user explicitly requests human assistance, THEN THE GramSetu_System SHALL initiate handoff within 10 seconds
3. WHEN transferring to a human operator, THEN THE GramSetu_System SHALL provide the operator with conversation context and collected information
4. WHEN no human operators are available, THEN THE GramSetu_System SHALL offer to schedule a callback within 24 hours
5. THE GramSetu_System SHALL log all handoff events with reason codes for quality improvement

### Requirement 10: Crisis Detection and Escalation

**User Story:** As a rural citizen in distress, I want the AI to recognize urgent situations, so that I can get immediate help during emergencies.

#### Acceptance Criteria

1. WHEN a user mentions keywords indicating crisis (medical emergency, violence, disaster), THEN THE Cognitive_Intelligence SHALL flag the call as high-priority
2. WHEN a crisis is detected, THEN THE GramSetu_System SHALL immediately transfer to a human operator or emergency services
3. WHEN crisis keywords are detected, THEN THE GramSetu_System SHALL capture the user's location information
4. THE Cognitive_Intelligence SHALL maintain a crisis keyword dictionary covering medical, safety, and disaster scenarios
5. WHEN a crisis call is escalated, THEN THE GramSetu_System SHALL send alerts to designated emergency response contacts

### Requirement 11: Sub-Second Latency Processing

**User Story:** As a rural citizen, I want the AI to respond quickly, so that the conversation feels natural and I don't have to wait long for answers.

#### Acceptance Criteria

1. WHEN a user finishes speaking, THEN THE Orchestration_Engine SHALL begin responding within 800ms
2. THE Acoustic_Model SHALL transcribe speech with less than 300ms latency
3. THE Cognitive_Intelligence SHALL generate responses at a rate of at least 300 tokens per second
4. WHEN using RAG, THEN THE Knowledge_Base SHALL return relevant documents within 200ms
5. THE Action_Layer SHALL respond to API calls within 1 second for 95% of requests

### Requirement 12: Session Management and Resume

**User Story:** As a rural citizen, I want to continue my conversation if the call drops, so that I don't have to start over and repeat information.

#### Acceptance Criteria

1. WHEN a call is active, THEN THE GramSetu_System SHALL maintain session state in memory
2. WHEN a call ends, THEN THE GramSetu_System SHALL persist session state for 5 minutes
3. WHEN a user calls back within 5 minutes, THEN THE GramSetu_System SHALL recognize the user and offer to resume the previous session
4. WHEN resuming a session, THEN THE GramSetu_System SHALL restore all collected slot information and conversation context
5. WHEN a session expires, THEN THE GramSetu_System SHALL archive the session data for audit purposes

### Requirement 13: MCP Server for Government API Integration

**User Story:** As a system administrator, I want a robust integration layer for government APIs, so that the AI can reliably execute transactions on behalf of users.

#### Acceptance Criteria

1. THE Action_Layer SHALL implement a Python/FastAPI MCP server for UMANG and CPGRAMS integration
2. WHEN the MCP server receives a transaction request, THEN it SHALL validate all required parameters before calling external APIs
3. WHEN an API call fails, THEN THE Action_Layer SHALL implement exponential backoff retry logic with a maximum of 3 attempts
4. THE Action_Layer SHALL log all API requests and responses for audit and debugging
5. WHEN API credentials expire, THEN THE Action_Layer SHALL automatically refresh tokens without interrupting service

### Requirement 14: Audio Quality and Noise Handling

**User Story:** As a rural citizen calling from a noisy environment, I want the AI to understand me despite background noise, so that I can use the service from anywhere.

#### Acceptance Criteria

1. WHEN background noise is detected, THEN THE Acoustic_Model SHALL apply noise suppression filters
2. THE Acoustic_Model SHALL maintain at least 70% transcription accuracy in environments with up to 60dB background noise
3. WHEN audio quality is too poor for reliable transcription, THEN THE GramSetu_System SHALL ask the user to move to a quieter location
4. THE Voice_Bridge SHALL support acoustic echo cancellation to prevent feedback
5. WHEN multiple speakers are detected, THEN THE Acoustic_Model SHALL focus on the primary speaker

### Requirement 15: Analytics and Monitoring

**User Story:** As a system administrator, I want to monitor system performance and user interactions, so that I can identify issues and improve the service.

#### Acceptance Criteria

1. THE GramSetu_System SHALL log all calls with metadata including duration, language, outcome, and user satisfaction
2. WHEN a call completes, THEN THE GramSetu_System SHALL record performance metrics including latency, accuracy, and completion rate
3. THE GramSetu_System SHALL provide a dashboard showing real-time metrics for concurrent calls, average handling time, and success rate
4. WHEN error rates exceed 5%, THEN THE GramSetu_System SHALL send alerts to administrators
5. THE GramSetu_System SHALL generate weekly reports on usage patterns, common issues, and improvement opportunities

### Requirement 16: Scalability and Reliability

**User Story:** As a system administrator, I want the platform to handle high call volumes reliably, so that users can access services during peak times.

#### Acceptance Criteria

1. THE GramSetu_System SHALL support at least 1000 concurrent calls without degradation in performance
2. WHEN call volume exceeds capacity, THEN THE GramSetu_System SHALL queue calls and provide estimated wait times
3. THE GramSetu_System SHALL maintain 99.5% uptime during business hours (9 AM - 6 PM IST)
4. WHEN a component fails, THEN THE GramSetu_System SHALL automatically failover to backup instances within 30 seconds
5. THE GramSetu_System SHALL implement auto-scaling to handle traffic spikes up to 200% of baseline capacity

### Requirement 17: Data Privacy and Security

**User Story:** As a rural citizen, I want my personal information to be secure, so that I can trust the system with sensitive data.

#### Acceptance Criteria

1. THE GramSetu_System SHALL encrypt all voice data in transit using TLS 1.3
2. THE GramSetu_System SHALL encrypt all stored user data using AES-256 encryption
3. WHEN collecting personal information, THEN THE GramSetu_System SHALL obtain explicit verbal consent from the user
4. THE GramSetu_System SHALL comply with India's Personal Data Protection Act and IT Act 2000
5. WHEN a user requests data deletion, THEN THE GramSetu_System SHALL remove all personal data within 30 days

### Requirement 18: Feedback and Continuous Improvement

**User Story:** As a rural citizen, I want to provide feedback on my experience, so that the service can improve over time.

#### Acceptance Criteria

1. WHEN a call ends, THEN THE GramSetu_System SHALL ask the user to rate their experience on a scale of 1-5
2. WHEN a user provides a rating below 3, THEN THE GramSetu_System SHALL ask for specific feedback on what went wrong
3. THE GramSetu_System SHALL store all feedback with associated call metadata for analysis
4. WHEN common issues are identified in feedback, THEN THE GramSetu_System SHALL flag them for review by the development team
5. THE GramSetu_System SHALL implement a monthly review process to incorporate user feedback into system improvements
