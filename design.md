# Design Document: GramSetu AI

## Overview

GramSetu AI is a voice-first AI platform that bridges the digital divide for rural citizens in India by providing a "No-UI" voice interface to government services. The system architecture is designed for sub-second latency, high reliability, and natural language interaction in multiple Indian languages and dialects.

The platform consists of five major layers:
1. **Telephony Layer**: PSTN/Mobile network connectivity via SIP trunking
2. **Orchestration Layer**: Real-time voice conversation management with full-duplex support
3. **Cognitive Layer**: LLM-based intent understanding and response generation
4. **Knowledge Layer**: Vector database for government policy retrieval
5. **Action Layer**: Government API integration for transactional capabilities

## Technology Stack

### Core Technologies
- **Telephony**: Twilio or Telnyx SIP Trunking
- **Orchestration**: Vapi.ai for real-time voice conversation management
- **Speech-to-Text**: Deepgram Nova-2 (English/Hindi), Sarvam AI (Indic languages)
- **LLM**: Llama 3 on Groq (primary), GPT-4 (fallback)
- **Text-to-Speech**: Vapi.ai integrated TTS
- **Vector Database**: Pinecone for RAG knowledge base
- **Action Layer**: Python FastAPI MCP Server
- **Government APIs**: UMANG, CPGRAMS
- **Authentication**: Voice biometrics + Voice OTP
- **Deployment**: AWS Lambda (Action Layer), Cloud hosting for other components

### Testing Framework
- **Property-Based Testing**: Hypothesis (Python)
- **Unit Testing**: pytest
- **Integration Testing**: pytest with mocking
- **Load Testing**: Locust for concurrent call simulation

## Architecture

### High-Level Architecture

```
[Rural Citizen] --PSTN/Mobile--> [Telephony Layer: Twilio/Telnyx]
                                         |
                                         v
                              [Orchestration: Vapi.ai]
                                    /    |    \
                                   /     |     \
                                  v      v      v
                              [STT]  [TTS]  [Session Mgr]
                                  \     |     /
                                   \    |    /
                                    v   v   v
                            [Cognitive Intelligence: LLM]
                                    /    |    \
                                   /     |     \
                                  v      v      v
                            [Intent]  [RAG]  [Confidence]
                                  |      |      |
                                  v      v      v
                            [Slot Fill] [Pinecone] [Handoff Logic]
                                         |
                                         v
                                  [Action Layer: MCP Server]
                                         |
                                    /         \
                                   v           v
                              [UMANG API]  [CPGRAMS API]
```

## Components and Interfaces

### 1. Telephony Layer

**Purpose**: Connect PSTN/mobile calls to the cloud-based AI system

**Components**:
- **SIP Trunk Provider**: Twilio or Telnyx for PSTN connectivity
- **Call Router**: Routes incoming calls to available Orchestration Engine instances
- **Session Manager**: Maintains call state and handles reconnection
- **Audio Quality Monitor**: Tracks latency, jitter, packet loss

**Key Interfaces**:
```python
class TelephonyInterface:
    def establish_call(phone_number: str) -> CallSession:
        """Establish connection when user dials in
        Returns: CallSession with unique session_id
        Latency requirement: < 3 seconds
        """
        pass
    
    def maintain_audio_stream(session: CallSession) -> AudioStream:
        """Maintain bidirectional audio stream
        Audio codec: Opus at 16kHz
        Target latency: < 150ms
        """
        pass
    
    def handle_disconnect(session: CallSession, reason: str) -> None:
        """Handle call disconnection and persist session state"""
        pass
    
    def get_call_quality_metrics(session: CallSession) -> QualityMetrics:
        """Monitor latency, jitter, packet loss"""
        pass
    
    def resume_session(phone_number: str, session_id: str) -> CallSession:
        """Resume a dropped call within 5 minutes"""
        pass
```

**Configuration**:
- Maximum concurrent calls: 1000
- Audio codec: Opus at 16kHz
- Target latency: < 150ms
- Jitter buffer: 20-60ms adaptive
- Session persistence: 5 minutes after disconnect


### 2. Orchestration Engine (Vapi.ai)

**Purpose**: Manage real-time voice conversations with full-duplex support and sub-second latency

**Components**:
- **Conversation Manager**: Orchestrates the flow between STT, LLM, and TTS
- **Interrupt Handler**: Detects and handles user interruptions within 300ms
- **Audio Processor**: Noise suppression, echo cancellation, speaker isolation
- **Latency Optimizer**: Ensures end-to-end response time < 800ms

**Key Interfaces**:
```python
class OrchestrationInterface:
    def start_conversation(session: CallSession, language: str) -> Conversation:
        """Initialize conversation with language selection"""
        pass
    
    def process_audio_chunk(audio: bytes, conversation: Conversation) -> None:
        """Process incoming audio in real-time (streaming)"""
        pass
    
    def handle_interruption(conversation: Conversation) -> None:
        """Stop TTS playback and switch to listening mode
        Detection latency: < 300ms
        """
        pass
    
    def detect_end_of_speech(audio_stream: AudioStream) -> bool:
        """Detect when user has finished speaking
        Pause threshold: 1.5 seconds
        """
        pass
    
    def apply_noise_suppression(audio: bytes) -> bytes:
        """Filter background noise up to 60dB"""
        pass
```

**Configuration**:
- Interrupt detection threshold: 300ms
- End-of-speech pause: 1.5 seconds
- Noise suppression: Up to 60dB
- Echo cancellation: Enabled
- Full-duplex mode: Enabled

### 3. Speech-to-Text (STT) Layer

**Purpose**: Convert user speech to text with high accuracy for Indian languages

**Components**:
- **Primary STT**: Deepgram Nova-2 for Hindi, English, Hinglish
- **Indic STT**: Sarvam AI for regional dialects (Tamil, Telugu, Bengali, etc.)
- **Language Detector**: Automatic language/dialect detection
- **Confidence Scorer**: Assess transcription quality

**Key Interfaces**:
```python
class STTInterface:
    def transcribe_audio(audio: bytes, language: str) -> Transcription:
        """Convert speech to text
        Latency: < 300ms
        Accuracy: 85% (standard), 75% (dialects)
        """
        pass
    
    def detect_language(audio: bytes) -> str:
        """Auto-detect spoken language
        Supported: Hindi, English, Hinglish, Tamil, Telugu, Bengali, 
                   Marathi, Gujarati, Kannada, Malayalam, Punjabi, Odia
        """
        pass
    
    def get_transcription_confidence(transcription: Transcription) -> float:
        """Return confidence score 0.0-1.0"""
        pass
```


### 4. Cognitive Intelligence Layer (LLM)

**Purpose**: Understand user intent, manage conversation flow, and generate contextual responses

**Components**:
- **LLM Engine**: Llama 3 on Groq (primary), GPT-4 (fallback)
- **Intent Parser**: Extract user intent from transcribed text
- **Slot Filler**: Collect required information through natural conversation
- **Confidence Scorer**: Assess understanding confidence for handoff decisions
- **Crisis Detector**: Identify emergency keywords and situations
- **Context Manager**: Maintain conversation history and state

**Key Interfaces**:
```python
class CognitiveInterface:
    def parse_intent(text: str, context: ConversationContext) -> Intent:
        """Extract user intent from transcribed speech
        Intents: eligibility_check, file_grievance, policy_inquiry, 
                 general_question, crisis, request_human
        """
        pass
    
    def fill_slots(intent: Intent, text: str, context: ConversationContext) -> SlotFillingResult:
        """Extract required information from natural conversation
        Slots: name, location, issue_description, scheme_name, contact_info
        """
        pass
    
    def generate_response(intent: Intent, rag_context: str, slots: dict) -> str:
        """Generate natural language response
        Token generation rate: > 300 tokens/sec
        """
        pass
    
    def calculate_confidence(intent: Intent, slots: dict) -> float:
        """Calculate confidence score for handoff decision
        Threshold for handoff: < 0.6
        """
        pass
    
    def detect_crisis(text: str) -> CrisisType:
        """Detect emergency situations
        Types: medical_emergency, violence, disaster, none
        """
        pass
    
    def should_handoff(confidence: float, context: ConversationContext) -> bool:
        """Determine if human handoff is needed"""
        pass
```

**Configuration**:
- LLM: Llama 3 70B on Groq (primary)
- Fallback LLM: GPT-4
- Token generation rate: > 300 tokens/sec
- Confidence threshold for handoff: 0.6
- Max conversation turns before forced handoff: 20
- Crisis keywords: Maintained in separate configuration file

### 5. Knowledge Base (RAG) Layer

**Purpose**: Provide accurate government policy information through vector search

**Components**:
- **Vector Database**: Pinecone for document storage and retrieval
- **Embedding Generator**: Convert text to embeddings for similarity search
- **Document Processor**: Chunk and index policy documents
- **Retrieval Engine**: Fetch relevant context for LLM

**Key Interfaces**:
```python
class KnowledgeBaseInterface:
    def query_policies(query: str, top_k: int = 5) -> List[PolicyDocument]:
        """Retrieve relevant policy documents
        Latency: < 200ms
        """
        pass
    
    def add_policy_document(document: PolicyDocument) -> str:
        """Add new policy document to knowledge base
        Returns: document_id
        """
        pass
    
    def update_policy_document(document_id: str, document: PolicyDocument) -> None:
        """Update existing policy document"""
        pass
    
    def get_document_metadata(document_id: str) -> DocumentMetadata:
        """Get document metadata including last_updated date"""
        pass
```


**Configuration**:
- Vector database: Pinecone
- Embedding model: text-embedding-ada-002 or similar
- Chunk size: 512 tokens with 50 token overlap
- Top-k retrieval: 5 documents
- Minimum policy documents: 50 major schemes
- Update SLA: 48 hours for new policies

### 6. Action Layer (MCP Server)

**Purpose**: Execute transactions with government APIs (UMANG, CPGRAMS)

**Components**:
- **MCP Server**: Python FastAPI server implementing Model Context Protocol
- **UMANG Client**: Integration with UMANG API for scheme eligibility
- **CPGRAMS Client**: Integration with CPGRAMS API for grievance filing
- **Retry Logic**: Exponential backoff for failed API calls
- **Authentication Manager**: Handle API credentials and token refresh
- **Audit Logger**: Log all API requests and responses

**Key Interfaces**:
```python
class ActionLayerInterface:
    def check_eligibility(scheme_name: str, user_info: dict) -> EligibilityResult:
        """Check user eligibility for government scheme via UMANG
        Latency: < 1 second (95th percentile)
        Retry: Up to 3 attempts with exponential backoff
        """
        pass
    
    def file_grievance(grievance: GrievanceData) -> GrievanceResult:
        """Submit grievance to CPGRAMS
        Returns: reference_number
        Retry: Up to 3 attempts
        """
        pass
    
    def send_sms_confirmation(phone: str, reference_number: str) -> bool:
        """Send SMS confirmation with reference number"""
        pass
    
    def validate_request(request: dict, schema: dict) -> ValidationResult:
        """Validate request parameters before API call"""
        pass
    
    def refresh_api_credentials() -> None:
        """Automatically refresh expired API tokens"""
        pass
```

**Configuration**:
- Framework: Python FastAPI
- Deployment: AWS Lambda
- Max retries: 3
- Retry backoff: Exponential (1s, 2s, 4s)
- API timeout: 5 seconds
- Credential refresh: Automatic on expiry

### 7. Voice Authentication System

**Purpose**: Secure user authentication using voice biometrics and Voice OTP

**Components**:
- **Voice Biometric Engine**: Create and verify voice profiles
- **OTP Generator**: Generate and validate 6-digit voice OTPs
- **Aadhaar Integration**: Optional Aadhaar-based authentication
- **Fallback Handler**: Switch to OTP when biometrics fail

**Key Interfaces**:
```python
class AuthenticationInterface:
    def create_voice_profile(audio: bytes, passphrase: str) -> VoiceProfile:
        """Create voice profile for first-time user"""
        pass
    
    def verify_voice(audio: bytes, user_id: str) -> AuthResult:
        """Verify user identity using voice biometrics
        Accuracy: > 95%
        """
        pass
    
    def generate_voice_otp(phone: str) -> str:
        """Generate 6-digit OTP valid for 5 minutes"""
        pass
    
    def verify_otp(phone: str, otp: str) -> bool:
        """Verify OTP provided by user"""
        pass
    
    def authenticate_with_aadhaar(aadhaar: str, otp: str) -> AuthResult:
        """Authenticate using Aadhaar OTP"""
        pass
```


### 8. Session Management System

**Purpose**: Maintain conversation state and enable session resumption

**Components**:
- **Session Store**: In-memory cache with persistent backup
- **State Manager**: Track conversation progress and collected information
- **Resume Handler**: Restore session when user calls back
- **Archive Manager**: Archive expired sessions for audit

**Key Interfaces**:
```python
class SessionManagementInterface:
    def create_session(phone: str, language: str) -> Session:
        """Create new conversation session"""
        pass
    
    def update_session(session_id: str, state: dict) -> None:
        """Update session state with new information"""
        pass
    
    def persist_session(session_id: str) -> None:
        """Persist session to storage (5 minute TTL)"""
        pass
    
    def resume_session(phone: str) -> Optional[Session]:
        """Restore session if called back within 5 minutes"""
        pass
    
    def archive_session(session_id: str) -> None:
        """Archive expired session for audit"""
        pass
```

### 9. Human Handoff System

**Purpose**: Transfer calls to human operators when AI confidence is low

**Components**:
- **Handoff Queue**: Manage queue of calls waiting for human operators
- **Operator Dashboard**: Provide operators with conversation context
- **Callback Scheduler**: Schedule callbacks when operators unavailable
- **Handoff Logger**: Track handoff events and reasons

**Key Interfaces**:
```python
class HandoffInterface:
    def initiate_handoff(session: Session, reason: str) -> HandoffResult:
        """Transfer call to human operator
        Latency: < 10 seconds
        """
        pass
    
    def get_operator_availability() -> int:
        """Return number of available operators"""
        pass
    
    def schedule_callback(phone: str, preferred_time: str) -> CallbackTicket:
        """Schedule callback within 24 hours"""
        pass
    
    def provide_context_to_operator(session: Session) -> OperatorContext:
        """Package conversation history and collected info for operator"""
        pass
    
    def log_handoff_event(session_id: str, reason: str, outcome: str) -> None:
        """Log handoff for quality improvement"""
        pass
```

### 10. Crisis Detection and Escalation

**Purpose**: Identify and escalate emergency situations

**Components**:
- **Crisis Keyword Matcher**: Detect emergency keywords in real-time
- **Priority Escalator**: Immediately route crisis calls
- **Location Tracker**: Capture user location during crisis
- **Emergency Notifier**: Alert emergency response contacts

**Key Interfaces**:
```python
class CrisisInterface:
    def detect_crisis_keywords(text: str) -> Optional[CrisisType]:
        """Detect crisis keywords in user speech
        Types: medical, violence, disaster
        """
        pass
    
    def escalate_to_emergency(session: Session, crisis_type: CrisisType) -> None:
        """Immediately transfer to emergency services"""
        pass
    
    def capture_location(session: Session) -> Location:
        """Capture user location information"""
        pass
    
    def send_emergency_alerts(crisis: CrisisEvent) -> None:
        """Alert designated emergency contacts"""
        pass
```


### 11. Analytics and Monitoring System

**Purpose**: Track system performance and user interactions

**Components**:
- **Metrics Collector**: Gather performance and usage metrics
- **Dashboard**: Real-time visualization of system health
- **Alert Manager**: Send alerts when thresholds exceeded
- **Report Generator**: Generate weekly usage reports

**Key Interfaces**:
```python
class AnalyticsInterface:
    def log_call_metadata(session: Session, outcome: str) -> None:
        """Log call with duration, language, outcome, satisfaction"""
        pass
    
    def record_performance_metrics(metrics: PerformanceMetrics) -> None:
        """Record latency, accuracy, completion rate"""
        pass
    
    def get_realtime_metrics() -> DashboardMetrics:
        """Get current concurrent calls, avg handling time, success rate"""
        pass
    
    def check_error_threshold() -> bool:
        """Return True if error rate > 5%"""
        pass
    
    def generate_weekly_report() -> Report:
        """Generate usage patterns and improvement opportunities"""
        pass
```

## Data Models

### Core Data Structures

```python
@dataclass
class CallSession:
    session_id: str
    phone_number: str
    start_time: datetime
    language: str
    audio_stream: AudioStream
    quality_metrics: QualityMetrics

@dataclass
class Conversation:
    session_id: str
    turns: List[ConversationTurn]
    context: ConversationContext
    collected_slots: dict
    confidence_scores: List[float]

@dataclass
class ConversationTurn:
    speaker: str  # "user" or "ai"
    text: str
    audio: Optional[bytes]
    timestamp: datetime
    confidence: float

@dataclass
class Intent:
    type: str  # eligibility_check, file_grievance, policy_inquiry, etc.
    confidence: float
    entities: dict

@dataclass
class SlotFillingResult:
    slots: dict  # {slot_name: slot_value}
    missing_slots: List[str]
    is_complete: bool

@dataclass
class EligibilityResult:
    is_eligible: bool
    scheme_name: str
    reason: str
    required_documents: List[str]

@dataclass
class GrievanceData:
    user_name: str
    phone: str
    location: str
    issue_description: str
    category: str

@dataclass
class GrievanceResult:
    success: bool
    reference_number: Optional[str]
    error_message: Optional[str]

@dataclass
class CrisisEvent:
    session_id: str
    crisis_type: str  # medical, violence, disaster
    location: Location
    timestamp: datetime
    user_phone: str
```


## Correctness Properties

The following properties define the correctness criteria for GramSetu AI. These properties will be validated using property-based testing with the Hypothesis framework.

### Property 1: Call Connection Latency
**Validates: Requirement 1.1**

For all valid phone numbers, when a user dials the GramSetu number, the system SHALL establish a connection within 3 seconds.

```python
@given(phone_number=valid_phone_numbers())
def test_call_connection_latency(phone_number):
    start_time = time.time()
    session = telephony.establish_call(phone_number)
    connection_time = time.time() - start_time
    
    assert session is not None
    assert session.session_id is not None
    assert connection_time <= 3.0
```

### Property 2: Audio Stream Latency
**Validates: Requirement 1.2**

For all active call sessions, the audio stream latency SHALL be less than 150ms.

```python
@given(session=active_call_sessions())
def test_audio_stream_latency(session):
    metrics = telephony.get_call_quality_metrics(session)
    assert metrics.latency_ms < 150
```

### Property 3: Concurrent Call Capacity
**Validates: Requirement 1.4**

The system SHALL support at least 1000 concurrent calls without degradation.

```python
@given(num_calls=st.integers(min_value=1, max_value=1000))
def test_concurrent_call_capacity(num_calls):
    sessions = []
    for i in range(num_calls):
        session = telephony.establish_call(f"+91{9000000000 + i}")
        sessions.append(session)
    
    # All sessions should be established
    assert len(sessions) == num_calls
    assert all(s.session_id is not None for s in sessions)
    
    # Check that quality metrics are maintained
    for session in sessions:
        metrics = telephony.get_call_quality_metrics(session)
        assert metrics.latency_ms < 150
```

### Property 4: Session Resumption
**Validates: Requirement 1.5**

When a call is dropped, the user SHALL be able to resume their session within 5 minutes.

```python
@given(session=active_call_sessions())
def test_session_resumption(session):
    phone = session.phone_number
    session_id = session.session_id
    
    # Simulate call drop
    telephony.handle_disconnect(session, "network_error")
    
    # Wait random time < 5 minutes
    wait_time = random.uniform(0, 299)  # 0 to 299 seconds
    time.sleep(wait_time)
    
    # Resume session
    resumed_session = telephony.resume_session(phone, session_id)
    assert resumed_session is not None
    assert resumed_session.session_id == session_id
```

### Property 5: Speech Recognition Accuracy
**Validates: Requirement 2.2**

For standard Hindi/English speech, transcription accuracy SHALL be at least 85%. For regional dialects, accuracy SHALL be at least 75%.

```python
@given(audio=hindi_english_audio_samples(), expected_text=st.text())
def test_standard_language_accuracy(audio, expected_text):
    transcription = stt.transcribe_audio(audio, language="hi-IN")
    accuracy = calculate_word_error_rate(transcription.text, expected_text)
    assert accuracy >= 0.85

@given(audio=dialect_audio_samples(), expected_text=st.text())
def test_dialect_accuracy(audio, expected_text):
    transcription = stt.transcribe_audio(audio, language="ta-IN")
    accuracy = calculate_word_error_rate(transcription.text, expected_text)
    assert accuracy >= 0.75
```


### Property 6: Language Detection
**Validates: Requirement 2.3**

When a user switches languages mid-conversation, the system SHALL detect and adapt within 2 seconds.

```python
@given(
    initial_language=supported_languages(),
    switch_language=supported_languages()
)
def test_language_switch_detection(initial_language, switch_language):
    assume(initial_language != switch_language)
    
    conversation = orchestration.start_conversation(session, initial_language)
    
    # User speaks in different language
    audio = generate_audio_in_language(switch_language)
    start_time = time.time()
    
    detected_language = stt.detect_language(audio)
    detection_time = time.time() - start_time
    
    assert detected_language == switch_language
    assert detection_time <= 2.0
```

### Property 7: Slot Filling Completeness
**Validates: Requirement 3.1, 3.2**

For all grievance filing intents, the system SHALL extract all required slots (name, location, issue) from natural conversation.

```python
@given(conversation_text=natural_grievance_conversations())
def test_slot_filling_completeness(conversation_text):
    intent = cognitive.parse_intent(conversation_text, context)
    assume(intent.type == "file_grievance")
    
    result = cognitive.fill_slots(intent, conversation_text, context)
    
    required_slots = ["name", "location", "issue_description"]
    for slot in required_slots:
        assert slot in result.slots or slot in result.missing_slots
    
    # If slot is missing, system should ask for it
    if result.missing_slots:
        response = cognitive.generate_response(intent, "", result.slots)
        assert any(slot in response.lower() for slot in result.missing_slots)
```

### Property 8: Out-of-Order Information Handling
**Validates: Requirement 3.3**

The system SHALL correctly process information provided in any order.

```python
@given(
    name=st.text(min_size=1),
    location=st.text(min_size=1),
    issue=st.text(min_size=1),
    order=st.permutations([0, 1, 2])
)
def test_out_of_order_slots(name, location, issue, order):
    info = [name, location, issue]
    slot_names = ["name", "location", "issue_description"]
    
    # Provide information in random order
    conversation_text = " ".join([info[i] for i in order])
    
    intent = Intent(type="file_grievance", confidence=0.9, entities={})
    result = cognitive.fill_slots(intent, conversation_text, context)
    
    # All slots should be filled regardless of order
    assert "name" in result.slots
    assert "location" in result.slots
    assert "issue_description" in result.slots
```

### Property 9: Voice Authentication Accuracy
**Validates: Requirement 4.2**

Voice biometric authentication SHALL achieve at least 95% accuracy.

```python
@given(user_audio=voice_samples(), user_id=st.text())
def test_voice_authentication_accuracy(user_audio, user_id):
    # Create voice profile
    profile = auth.create_voice_profile(user_audio, "my secure passphrase")
    
    # Verify with same user's voice
    result = auth.verify_voice(user_audio, user_id)
    assert result.authenticated == True
    assert result.confidence >= 0.95
```

### Property 10: Voice OTP Validity
**Validates: Requirement 4.4**

Voice OTP SHALL be valid for exactly 5 minutes.

```python
@given(phone=valid_phone_numbers())
def test_voice_otp_validity(phone):
    otp = auth.generate_voice_otp(phone)
    
    # OTP should work within 5 minutes
    time.sleep(random.uniform(0, 299))  # 0 to 299 seconds
    assert auth.verify_otp(phone, otp) == True
    
    # OTP should expire after 5 minutes
    time.sleep(301)  # Wait until after 5 minutes
    assert auth.verify_otp(phone, otp) == False
```


### Property 11: Interruption Detection Latency
**Validates: Requirement 5.1**

The system SHALL detect user interruptions within 300ms.

```python
@given(conversation=active_conversations())
def test_interruption_detection_latency(conversation):
    # Start AI speaking
    orchestration.start_tts_playback(conversation, "This is a long response...")
    
    # User interrupts
    start_time = time.time()
    user_audio = generate_interruption_audio()
    orchestration.process_audio_chunk(user_audio, conversation)
    
    detection_time = time.time() - start_time
    assert detection_time <= 0.3  # 300ms
    assert conversation.tts_stopped == True
```

### Property 12: Pause Detection
**Validates: Requirement 5.3**

The system SHALL wait at least 1.5 seconds after user pauses before responding.

```python
@given(audio_with_pause=audio_samples_with_pauses())
def test_pause_detection(audio_with_pause):
    conversation = orchestration.start_conversation(session, "hi-IN")
    
    # Process audio with mid-sentence pause
    orchestration.process_audio_chunk(audio_with_pause, conversation)
    
    # Measure time before system responds
    pause_duration = measure_pause_duration(conversation)
    assert pause_duration >= 1.5
```

### Property 13: Eligibility Check Accuracy
**Validates: Requirement 6.3, 6.4**

When checking eligibility, the system SHALL correctly determine eligibility based on user information and scheme criteria.

```python
@given(
    scheme=government_schemes(),
    user_info=user_information()
)
def test_eligibility_check_accuracy(scheme, user_info):
    result = action_layer.check_eligibility(scheme.name, user_info)
    
    # Manually calculate expected eligibility
    expected_eligible = calculate_eligibility(scheme.criteria, user_info)
    
    assert result.is_eligible == expected_eligible
    if not result.is_eligible:
        assert result.reason is not None
        assert len(result.reason) > 0
```

### Property 14: Grievance Filing Success
**Validates: Requirement 7.2, 7.3**

When all required information is collected, grievance submission SHALL succeed and return a reference number.

```python
@given(grievance=complete_grievance_data())
def test_grievance_filing_success(grievance):
    result = action_layer.file_grievance(grievance)
    
    assert result.success == True
    assert result.reference_number is not None
    assert len(result.reference_number) > 0
    assert result.error_message is None
```

### Property 15: Grievance Retry Logic
**Validates: Requirement 7.4**

When grievance submission fails, the system SHALL retry up to 3 times before escalating.

```python
@given(grievance=complete_grievance_data())
def test_grievance_retry_logic(grievance):
    # Mock API to fail first 2 times, succeed on 3rd
    with mock_api_failures(fail_count=2):
        result = action_layer.file_grievance(grievance)
    
    assert result.success == True
    assert get_api_call_count() == 3
```

### Property 16: RAG Retrieval Latency
**Validates: Requirement 11.4**

Knowledge base queries SHALL return results within 200ms.

```python
@given(query=policy_queries())
def test_rag_retrieval_latency(query):
    start_time = time.time()
    documents = knowledge_base.query_policies(query, top_k=5)
    retrieval_time = time.time() - start_time
    
    assert retrieval_time <= 0.2  # 200ms
    assert len(documents) <= 5
```


### Property 17: End-to-End Response Latency
**Validates: Requirement 11.1**

The system SHALL respond within 800ms after user finishes speaking.

```python
@given(user_speech=speech_samples())
def test_end_to_end_latency(user_speech):
    conversation = orchestration.start_conversation(session, "hi-IN")
    
    start_time = time.time()
    
    # Process user speech through full pipeline
    transcription = stt.transcribe_audio(user_speech, "hi-IN")
    intent = cognitive.parse_intent(transcription.text, context)
    rag_context = knowledge_base.query_policies(transcription.text)
    response = cognitive.generate_response(intent, rag_context, {})
    
    total_time = time.time() - start_time
    assert total_time <= 0.8  # 800ms
```

### Property 18: Session State Persistence
**Validates: Requirement 12.2, 12.3**

Session state SHALL persist for 5 minutes after call ends and be restorable.

```python
@given(session=active_sessions_with_data())
def test_session_persistence(session):
    original_slots = session.conversation.collected_slots.copy()
    session_id = session.session_id
    phone = session.phone_number
    
    # End call
    telephony.handle_disconnect(session, "user_hangup")
    
    # Wait random time < 5 minutes
    wait_time = random.uniform(0, 299)
    time.sleep(wait_time)
    
    # Resume session
    resumed = session_manager.resume_session(phone)
    
    assert resumed is not None
    assert resumed.session_id == session_id
    assert resumed.conversation.collected_slots == original_slots
```

### Property 19: Low Confidence Handoff
**Validates: Requirement 9.1**

When confidence score falls below 60%, the system SHALL offer human handoff.

```python
@given(
    intent=intents_with_confidence(),
    context=conversation_contexts()
)
def test_low_confidence_handoff(intent, context):
    assume(intent.confidence < 0.6)
    
    should_handoff = cognitive.should_handoff(intent.confidence, context)
    assert should_handoff == True
```

### Property 20: Crisis Detection
**Validates: Requirement 10.1, 10.2**

When crisis keywords are detected, the system SHALL immediately flag and escalate.

```python
@given(text=text_with_crisis_keywords())
def test_crisis_detection(text):
    crisis_type = cognitive.detect_crisis(text)
    
    assert crisis_type is not None
    assert crisis_type in ["medical_emergency", "violence", "disaster"]
    
    # System should escalate immediately
    session = create_test_session()
    crisis_event = CrisisEvent(
        session_id=session.session_id,
        crisis_type=crisis_type,
        location=Location("test", "test"),
        timestamp=datetime.now(),
        user_phone=session.phone_number
    )
    
    # Verify escalation happens
    crisis_handler.escalate_to_emergency(session, crisis_type)
    assert session.escalated == True
```

### Property 21: Noise Handling
**Validates: Requirement 14.2**

The system SHALL maintain at least 70% transcription accuracy in environments with up to 60dB background noise.

```python
@given(
    clean_audio=clean_speech_samples(),
    noise_level=st.floats(min_value=0, max_value=60)
)
def test_noise_handling(clean_audio, noise_level):
    # Add noise to clean audio
    noisy_audio = add_background_noise(clean_audio, noise_level)
    
    # Transcribe
    transcription = stt.transcribe_audio(noisy_audio, "hi-IN")
    expected_text = get_ground_truth_text(clean_audio)
    
    accuracy = calculate_word_error_rate(transcription.text, expected_text)
    assert accuracy >= 0.70
```


### Property 22: API Retry with Exponential Backoff
**Validates: Requirement 13.3**

Failed API calls SHALL retry with exponential backoff (1s, 2s, 4s) up to 3 attempts.

```python
@given(request=api_requests())
def test_api_retry_backoff(request):
    retry_times = []
    
    with mock_api_failures(fail_count=3):
        start_time = time.time()
        try:
            action_layer.check_eligibility(request.scheme, request.user_info)
        except Exception:
            pass
        retry_times = get_retry_timestamps()
    
    # Should have 3 retry attempts
    assert len(retry_times) == 3
    
    # Check exponential backoff timing
    assert retry_times[1] - retry_times[0] >= 1.0  # First retry after 1s
    assert retry_times[2] - retry_times[1] >= 2.0  # Second retry after 2s
```

### Property 23: Concurrent Call Performance
**Validates: Requirement 16.1**

The system SHALL maintain performance with 1000 concurrent calls.

```python
@given(num_concurrent=st.integers(min_value=100, max_value=1000))
def test_concurrent_call_performance(num_concurrent):
    sessions = []
    latencies = []
    
    # Establish concurrent calls
    for i in range(num_concurrent):
        start = time.time()
        session = telephony.establish_call(f"+91{9000000000 + i}")
        latency = time.time() - start
        
        sessions.append(session)
        latencies.append(latency)
    
    # All calls should be established
    assert len(sessions) == num_concurrent
    
    # Performance should not degrade
    avg_latency = sum(latencies) / len(latencies)
    assert avg_latency <= 3.0  # Connection time requirement
    
    # Audio quality should be maintained
    for session in sessions:
        metrics = telephony.get_call_quality_metrics(session)
        assert metrics.latency_ms < 150
```

### Property 24: Data Encryption
**Validates: Requirement 17.1, 17.2**

All voice data SHALL be encrypted in transit (TLS 1.3) and at rest (AES-256).

```python
@given(audio_data=audio_samples())
def test_data_encryption(audio_data):
    # Encrypt in transit
    encrypted_transit = encrypt_for_transit(audio_data)
    assert is_tls_1_3_encrypted(encrypted_transit)
    
    # Encrypt at rest
    encrypted_storage = encrypt_for_storage(audio_data)
    assert is_aes_256_encrypted(encrypted_storage)
    
    # Verify decryption works
    decrypted = decrypt_from_storage(encrypted_storage)
    assert decrypted == audio_data
```

### Property 25: Feedback Collection
**Validates: Requirement 18.1, 18.2**

After call ends, the system SHALL collect user feedback rating (1-5).

```python
@given(session=completed_sessions())
def test_feedback_collection(session):
    # System should ask for feedback
    feedback_prompt = get_feedback_prompt(session)
    assert feedback_prompt is not None
    assert "rate" in feedback_prompt.lower() or "feedback" in feedback_prompt.lower()
    
    # Accept rating 1-5
    rating = random.randint(1, 5)
    feedback = collect_feedback(session, rating)
    
    assert feedback.rating == rating
    assert 1 <= feedback.rating <= 5
    
    # If rating < 3, should ask for details
    if rating < 3:
        assert feedback.detail_requested == True
```


## Implementation Strategy

### Phase 1: Core Infrastructure (Weeks 1-2)
1. Set up Twilio/Telnyx SIP trunking for telephony
2. Integrate Vapi.ai orchestration engine
3. Configure Deepgram Nova-2 and Sarvam AI for STT
4. Set up Llama 3 on Groq with GPT-4 fallback
5. Implement basic call flow: dial-in → language selection → simple Q&A

### Phase 2: Cognitive Intelligence (Weeks 3-4)
1. Implement intent parsing and slot filling
2. Set up Pinecone vector database
3. Ingest 50+ government policy documents
4. Implement RAG-based policy retrieval
5. Add confidence scoring and handoff logic
6. Implement crisis detection keywords

### Phase 3: Action Layer (Weeks 5-6)
1. Build Python FastAPI MCP server
2. Integrate UMANG API for eligibility checks
3. Integrate CPGRAMS API for grievance filing
4. Implement retry logic with exponential backoff
5. Add SMS confirmation system
6. Implement audit logging

### Phase 4: Authentication & Session Management (Week 7)
1. Implement voice biometric authentication
2. Add Voice OTP fallback system
3. Build session state management
4. Implement session resumption logic
5. Add session archival system

### Phase 5: Advanced Features (Week 8)
1. Implement full-duplex interruption handling
2. Add noise suppression and echo cancellation
3. Implement human handoff queue system
4. Build operator dashboard
5. Add callback scheduling

### Phase 6: Monitoring & Analytics (Week 9)
1. Implement metrics collection
2. Build real-time dashboard
3. Add alert system for error thresholds
4. Implement feedback collection
5. Build weekly report generator

### Phase 7: Testing & Optimization (Weeks 10-11)
1. Write property-based tests for all 25 properties
2. Conduct load testing with 1000 concurrent calls
3. Optimize latency bottlenecks
4. Test with real users in rural areas
5. Iterate based on feedback

### Phase 8: Security & Compliance (Week 12)
1. Implement TLS 1.3 encryption
2. Add AES-256 encryption for stored data
3. Ensure PDPA and IT Act 2000 compliance
4. Conduct security audit
5. Implement data deletion workflows

## Testing Strategy

### Unit Testing
- Test individual components in isolation
- Mock external dependencies (APIs, databases)
- Use pytest for Python components
- Target: 80% code coverage

### Property-Based Testing
- Implement all 25 correctness properties using Hypothesis
- Generate random test cases for edge case discovery
- Run PBT suite on every commit
- Target: All properties pass consistently

### Integration Testing
- Test component interactions
- Use real STT/TTS services in staging
- Mock government APIs with realistic responses
- Test full call flows end-to-end

### Load Testing
- Use Locust to simulate concurrent calls
- Test with 100, 500, 1000 concurrent users
- Measure latency, throughput, error rates
- Identify bottlenecks and optimize

### User Acceptance Testing
- Conduct pilot with 50 rural users
- Test in real-world conditions (noise, network issues)
- Collect qualitative feedback
- Iterate on UX and accuracy


## Deployment Architecture

### Infrastructure Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Production Environment                   │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐         ┌──────────────┐                  │
│  │   Twilio/    │         │   Vapi.ai    │                  │
│  │   Telnyx     │◄────────┤  Orchestrator│                  │
│  │  SIP Trunk   │         │   (Hosted)   │                  │
│  └──────────────┘         └──────┬───────┘                  │
│                                   │                          │
│                          ┌────────▼────────┐                 │
│                          │   Deepgram /    │                 │
│                          │   Sarvam AI     │                 │
│                          │   (API)         │                 │
│                          └────────┬────────┘                 │
│                                   │                          │
│  ┌─────────────────────────────────────────────────┐        │
│  │         Cloud Infrastructure (AWS/GCP)          │        │
│  │                                                  │        │
│  │  ┌──────────────┐      ┌──────────────┐        │        │
│  │  │  LLM Engine  │      │   Pinecone   │        │        │
│  │  │ Groq/OpenAI  │◄─────┤  Vector DB   │        │        │
│  │  │   (API)      │      │   (Hosted)   │        │        │
│  │  └──────┬───────┘      └──────────────┘        │        │
│  │         │                                       │        │
│  │  ┌──────▼───────────────────────────┐          │        │
│  │  │      MCP Server (FastAPI)        │          │        │
│  │  │      AWS Lambda / ECS            │          │        │
│  │  │                                  │          │        │
│  │  │  ┌────────────┐  ┌────────────┐ │          │        │
│  │  │  │   UMANG    │  │  CPGRAMS   │ │          │        │
│  │  │  │   Client   │  │   Client   │ │          │        │
│  │  │  └────────────┘  └────────────┘ │          │        │
│  │  └──────────────────────────────────┘          │        │
│  │                                                  │        │
│  │  ┌──────────────────────────────────┐          │        │
│  │  │     Session Store (Redis)        │          │        │
│  │  └──────────────────────────────────┘          │        │
│  │                                                  │        │
│  │  ┌──────────────────────────────────┐          │        │
│  │  │   Analytics DB (PostgreSQL)      │          │        │
│  │  └──────────────────────────────────┘          │        │
│  │                                                  │        │
│  │  ┌──────────────────────────────────┐          │        │
│  │  │   Monitoring (CloudWatch/        │          │        │
│  │  │   Datadog)                       │          │        │
│  │  └──────────────────────────────────┘          │        │
│  └─────────────────────────────────────────────────┘        │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Scaling Strategy

**Horizontal Scaling**:
- MCP Server: Auto-scale Lambda functions or ECS tasks based on call volume
- Session Store: Redis cluster with replication
- Database: PostgreSQL with read replicas

**Vertical Scaling**:
- Increase Lambda memory/CPU for MCP server during peak hours
- Use larger Redis instances for session store

**Geographic Distribution**:
- Deploy in multiple AWS regions (Mumbai, Delhi) for low latency
- Use CloudFront for static assets
- Route calls to nearest region

### High Availability

**Redundancy**:
- Multi-AZ deployment for all services
- Backup SIP trunk provider (Twilio + Telnyx)
- LLM fallback (Groq → GPT-4)
- STT fallback (Deepgram → Sarvam AI)

**Failover**:
- Automatic failover within 30 seconds
- Health checks every 10 seconds
- Circuit breaker pattern for external APIs

**Backup & Recovery**:
- Daily backups of PostgreSQL database
- Redis persistence with AOF
- Session data replicated across AZs
- Recovery Time Objective (RTO): 1 hour
- Recovery Point Objective (RPO): 15 minutes


## Security Considerations

### Data Protection
1. **Encryption in Transit**: TLS 1.3 for all network communication
2. **Encryption at Rest**: AES-256 for stored voice recordings and user data
3. **Key Management**: AWS KMS for encryption key rotation
4. **Data Retention**: Voice recordings deleted after 90 days (configurable)
5. **PII Handling**: Minimize collection, encrypt all PII fields

### Authentication & Authorization
1. **Voice Biometrics**: Secure voice profile storage with encryption
2. **API Authentication**: OAuth 2.0 for government API access
3. **Token Management**: Automatic token refresh, secure storage
4. **Access Control**: Role-based access for operator dashboard
5. **Audit Logging**: Log all authentication attempts and API calls

### Compliance
1. **Personal Data Protection Act (PDPA)**: Obtain explicit consent, provide data deletion
2. **IT Act 2000**: Secure electronic records, digital signatures where required
3. **Aadhaar Act**: Comply with Aadhaar authentication guidelines
4. **GDPR (if applicable)**: Right to access, right to be forgotten
5. **Industry Standards**: Follow OWASP Top 10, CIS benchmarks

### Threat Mitigation
1. **DDoS Protection**: CloudFlare or AWS Shield
2. **Rate Limiting**: Prevent abuse of APIs and phone system
3. **Input Validation**: Sanitize all user inputs to prevent injection attacks
4. **Fraud Detection**: Monitor for suspicious calling patterns
5. **Incident Response**: 24/7 security monitoring, incident response plan

## Performance Optimization

### Latency Reduction
1. **Edge Computing**: Deploy STT/TTS close to users
2. **Caching**: Cache frequently accessed policy documents
3. **Connection Pooling**: Reuse database and API connections
4. **Async Processing**: Use async I/O for non-blocking operations
5. **CDN**: Serve static assets from CDN

### Cost Optimization
1. **Reserved Instances**: Use reserved capacity for baseline load
2. **Spot Instances**: Use spot instances for non-critical workloads
3. **API Cost Management**: Monitor and optimize LLM token usage
4. **Storage Tiering**: Move old data to cheaper storage tiers
5. **Compression**: Compress audio and text data

### Quality Optimization
1. **A/B Testing**: Test different LLM prompts and models
2. **Continuous Training**: Retrain models with real user data
3. **Feedback Loop**: Use user feedback to improve responses
4. **Error Analysis**: Analyze failed calls to identify patterns
5. **Performance Monitoring**: Track and optimize key metrics

## Monitoring & Alerting

### Key Metrics
1. **Call Metrics**: Total calls, concurrent calls, call duration, completion rate
2. **Performance Metrics**: Latency (STT, LLM, TTS, API), response time
3. **Quality Metrics**: Transcription accuracy, intent recognition accuracy, user satisfaction
4. **System Metrics**: CPU, memory, disk usage, network throughput
5. **Business Metrics**: Grievances filed, eligibility checks, handoff rate

### Alerts
1. **Critical**: System down, error rate > 10%, latency > 2x normal
2. **Warning**: Error rate > 5%, latency > 1.5x normal, disk > 80%
3. **Info**: New deployment, configuration change, scheduled maintenance

### Dashboards
1. **Operations Dashboard**: Real-time system health, active calls, error rates
2. **Business Dashboard**: Daily/weekly/monthly usage, success rates, user satisfaction
3. **Performance Dashboard**: Latency trends, throughput, resource utilization
4. **Quality Dashboard**: Transcription accuracy, intent accuracy, handoff reasons

## Maintenance & Operations

### Regular Maintenance
1. **Daily**: Monitor dashboards, review error logs, check alert status
2. **Weekly**: Review performance trends, analyze user feedback, update knowledge base
3. **Monthly**: Security patches, dependency updates, capacity planning
4. **Quarterly**: Disaster recovery drill, security audit, cost optimization review

### Incident Response
1. **Detection**: Automated alerts, user reports, monitoring dashboards
2. **Triage**: Assess severity, identify affected users, determine root cause
3. **Mitigation**: Apply hotfix, failover to backup, scale resources
4. **Resolution**: Deploy permanent fix, verify resolution, update documentation
5. **Post-Mortem**: Analyze incident, identify improvements, update runbooks

### Knowledge Base Management
1. **Content Updates**: Monitor government websites for policy changes
2. **Quality Assurance**: Verify accuracy of policy documents
3. **Version Control**: Track document versions and update dates
4. **Deprecation**: Remove outdated policies, redirect to current versions
5. **Expansion**: Add new schemes, languages, and regional content

## Future Enhancements

### Short-term (3-6 months)
1. **Multi-modal Support**: Add SMS/WhatsApp for follow-ups
2. **Proactive Notifications**: Alert users about new schemes they're eligible for
3. **Voice Cloning**: Personalized AI voice for better user experience
4. **Offline Mode**: Basic functionality without internet (USSD)
5. **Regional Language Expansion**: Add more dialects and languages

### Long-term (6-12 months)
1. **Predictive Analytics**: Predict user needs based on demographics
2. **Blockchain Integration**: Immutable audit trail for grievances
3. **AI Agent Collaboration**: Multiple AI agents for complex queries
4. **Video Support**: Add video calling for document verification
5. **Integration Expansion**: Connect with more government services

## Conclusion

GramSetu AI represents a comprehensive voice-first platform designed to bridge the digital divide for rural citizens in India. The architecture prioritizes sub-second latency, high reliability, and natural language interaction across multiple Indian languages. By combining modern AI technologies (LLMs, RAG, voice biometrics) with robust engineering practices (property-based testing, monitoring, security), the system aims to provide seamless access to government services for millions of underserved citizens.

The design emphasizes correctness through 25 testable properties, scalability through cloud-native architecture, and user-centricity through natural conversation flows. With careful attention to security, compliance, and performance optimization, GramSetu AI is positioned to make a significant impact on digital inclusion in rural India.
