# Design Document: Nyx AI Socratic Debugger

## Overview

Nyx AI is a comprehensive Socratic learning platform that teaches programming and debugging through guided discovery, practice, and structured learning plans. The system consists of multiple client interfaces (React.js web app, Kiro extension, VS Code extension) and a Python FastAPI backend deployed on AWS Lambda that orchestrates AI-powered analysis using AWS Bedrock (Claude 3 Sonnet).

The platform offers three core learning modes:

- **Debug Mode**: Analyzes broken code with error logs to help users find and fix bugs through progressive hints
- **Learning Mode**: Analyzes any code to help users understand how it works, with goal-based learning plans and scheduled practice
- **Practice Mode**: Generates similar problems after successful solutions to reinforce learning and build confidence

Key innovations include:

1. Progressive hinting with prerequisite concept identification
2. Adaptive difficulty adjustment based on user performance
3. Goal-based learning plans with curated materials and scheduled tests
4. Practice problem generation at similar difficulty levels
5. Multi-platform support (web, Kiro, VS Code)

The architecture follows a client-server model with stateful session management, where each learning interaction is tracked through its lifecycle from code submission to completion, practice, and long-term goal achievement.

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Interfaces                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Web App    │  │     Kiro     │  │   VS Code    │     │
│  │  (React.js)  │  │  Extension   │  │  Extension   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Code Editor  │  │ Image Upload │  │ Hint Display │     │
│  │  Component   │  │  Component   │  │  Component   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐                       │
│  │   Practice   │  │ Learning Plan│                       │
│  │   Problems   │  │   Dashboard  │                       │
│  └──────────────┘  └──────────────┘                       │
└─────────────────────────────────────────────────────────────┘
                            │
                         HTTPS/REST
                            │
┌─────────────────────────────────────────────────────────────┐
│                    Backend (FastAPI)                         │
│                    AWS Lambda + API Gateway                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Session    │  │   Analysis   │  │     Hint     │     │
│  │   Manager    │  │    Engine    │  │  Generator   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Concept Card │  │ Prerequisite │  │    Image     │     │
│  │  Generator   │  │   Analyzer   │  │    Parser    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Practice   │  │ Learning Plan│  │   Progress   │     │
│  │   Problem    │  │   Generator  │  │   Tracker    │     │
│  │  Generator   │  │              │  │              │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                      AWS Services
                            │
┌─────────────────────────────────────────────────────────────┐
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ AWS Bedrock  │  │ AWS Textract │  │   DynamoDB   │     │
│  │(Claude 3.5)  │  │     (OCR)    │  │  (Sessions)  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

└─────────────────────────────────────────────────────────────┘
│
HTTPS/REST
│
┌─────────────────────────────────────────────────────────────┐
│ Backend (FastAPI) │
│ AWS Lambda + API Gateway │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ │ Session │ │ Analysis │ │ Hint │ │
│ │ Manager │ │ Engine │ │ Generator │ │
│ └──────────────┘ └──────────────┘ └──────────────┘ │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ │ Concept Card │ │ Prerequisite │ │ Image │ │
│ │ Generator │ │ Analyzer │ │ Parser │ │
│ └──────────────┘ └──────────────┘ └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
│
AWS Services
│
┌─────────────────────────────────────────────────────────────┐
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ │ AWS Bedrock │ │ AWS Textract │ │ DynamoDB │ │
│ │(Claude 3.5) │ │ (OCR) │ │ (Sessions) │ │
│ └──────────────┘ └──────────────┘ └──────────────┘ │
└─────────────────────────────────────────────────────────────┘

````

### Technology Stack

- **Frontend**: React.js with Monaco Editor for code editing, Axios for HTTP requests
- **Responsive Design**: Tailwind CSS or Material-UI for responsive layouts, CSS Grid and Flexbox for adaptive layouts
- **Editor Extensions**:
  - Kiro Extension API
  - VS Code Extension API
- **Backend**: Python 3.11+ with FastAPI framework
- **Deployment**: AWS Lambda (backend), AWS API Gateway (REST API), S3 + CloudFront (frontend hosting)
- **AI Service**: AWS Bedrock with Claude 3 Sonnet model
- **OCR Service**: AWS Textract for image-to-text extraction
- **Session Storage**: AWS DynamoDB for session state persistence
- **Monitoring**: AWS CloudWatch for logging and metrics

## Responsive Design Strategy

### Breakpoints

The web application uses the following responsive breakpoints:

- **Mobile**: ≤767px (portrait phones, small landscape phones)
- **Tablet**: 768px-1023px (tablets, large phones in landscape)
- **Desktop**: ≥1024px (laptops, desktops, large screens)

### Layout Adaptations

**Desktop Layout (≥1024px):**
- Three-column layout: Code Editor (left, 50%), Hints Panel (right-top, 50%), Concept Cards (right-bottom, 50%)
- Full Monaco Editor with all features enabled
- Side-by-side comparison views for practice problems

**Tablet Layout (768px-1023px):**
- Two-column layout: Code Editor (top/left, 60%), Hints Panel (bottom/right, 40%)
- Collapsible panels for concept cards
- Simplified Monaco Editor toolbar

**Mobile Layout (≤767px):**
- Single-column stacked layout
- Tabbed interface to switch between Code Editor, Hints, and Concept Cards
- Mobile-optimized code input with syntax highlighting
- Floating action button for primary actions (Submit, Next Hint)
- Bottom sheet for concept cards
- Camera integration for direct code image capture

### Touch Optimization

- Minimum touch target size: 44x44px for all interactive elements
- Swipe gestures for navigating between hint stages
- Pull-to-refresh for reloading session state
- Touch-friendly code editor with zoom support

## Components and Interfaces

### Frontend Components

#### Code Editor Component

**Responsibilities:**

- Render Monaco Editor for code input
- Provide textarea for error log input
- Handle code submission to backend
- Display syntax highlighting for multiple languages

**Interface:**

```typescript
interface CodeEditorProps {
  onSubmit: (code: string, errorLog: string, language: string) => void;
  isLoading: boolean;
}

interface CodeSubmission {
  code: string;
  errorLog: string;
  language: string;
}
````

#### Hint Display Component

**Responsibilities:**

- Display current hint to user
- Show hint stage indicator (1, 2, or 3)
- Provide "Next Hint" and "Give Up" buttons
- Track which hints have been viewed

**Interface:**

```typescript
interface HintDisplayProps {
  sessionId: string;
  currentStage: number;
  hint: string | null;
  onRequestHint: (stage: number) => void;
  onGiveUp: () => void;
}

interface HintResponse {
  stage: number;
  hint: string;
  canRequestNext: boolean;
}
```

#### Concept Card Component

**Responsibilities:**

- Display educational flashcard after bug resolution
- Show concept name, definition, and examples
- Provide save/export functionality

**Interface:**

```typescript
interface ConceptCardProps {
  concept: ConceptCard;
  onSave: (concept: ConceptCard) => void;
  onClose: () => void;
}

interface ConceptCard {
  conceptName: string;
  definition: string;
  prerequisites: string[];
  examples: string[];
  relatedConcepts: string[];
}
```

#### Image Upload Component (Web App Only)

**Responsibilities:**

- Provide interface for image upload
- Preview uploaded images
- Trigger OCR extraction
- Display extraction progress and results

**Interface:**

```typescript
interface ImageUploadProps {
  onImageExtracted: (code: string) => void;
  isProcessing: boolean;
}

interface ImageUploadResponse {
  extractedCode: string;
  confidence: number;
  language: string | null;
}
```

#### Editor Extension Interface

**Responsibilities:**

- Integrate with Kiro and VS Code editor APIs
- Capture selected code from editor
- Display hints and concept cards in editor panels
- Maintain session state during editor session

**Interface (Kiro Extension):**

```typescript
interface KiroExtension {
  activate(context: ExtensionContext): void;
  analyzeSelection(): void;
  showHintPanel(hint: string, stage: number): void;
  showConceptCard(card: ConceptCard): void;
}
```

**Interface (VS Code Extension):**

```typescript
interface VSCodeExtension {
  activate(context: vscode.ExtensionContext): void;
  registerCommands(): void;
  analyzeSelection(): void;
  createHintWebview(hint: string, stage: number): void;
  createConceptCardWebview(card: ConceptCard): void;
}
```

### Backend Components

#### Session Manager

**Responsibilities:**

- Create new debugging sessions
- Store and retrieve session state from DynamoDB
- Track hint stage progression
- Manage session expiration (24-hour TTL)

**Interface:**

```python
class SessionManager:
    def create_session(self, code: str, error_log: str | None, language: str, mode: str) -> str:
        """Creates a new session and returns session_id. Mode is 'debug' or 'learn'"""
        pass

    def get_session(self, session_id: str) -> Session:
        """Retrieves session by ID"""
        pass

    def update_hint_stage(self, session_id: str, stage: int) -> None:
        """Updates the current hint stage for a session"""
        pass

    def mark_complete(self, session_id: str) -> None:
        """Marks a session as resolved"""
        pass

class Session:
    session_id: str
    code: str
    error_log: str | None
    language: str
    mode: str  # 'debug' or 'learn'
    analysis: dict
    current_hint_stage: int
    is_complete: bool
    created_at: datetime
    expires_at: datetime
```

#### Analysis Engine

**Responsibilities:**

- Send code to AWS Bedrock for analysis
- Support both debug mode (with errors) and learning mode (without errors)
- Parse AI response to extract analysis
- Ensure AI doesn't reveal solutions directly
- Handle API failures and retries

**Interface:**

```python
class AnalysisEngine:
    def analyze_code(self, code: str, error_log: str | None, language: str, mode: str) -> Analysis:
        """Analyzes code using AWS Bedrock. Mode is 'debug' or 'learn'"""
        pass

    def _build_analysis_prompt(self, code: str, error_log: str | None, language: str, mode: str) -> str:
        """Constructs the system prompt for Bedrock based on mode"""
        pass

    def _call_bedrock(self, prompt: str, max_retries: int = 3) -> str:
        """Calls AWS Bedrock API with retry logic"""
        pass

class Analysis:
    mode: str  # 'debug' or 'learn'
    # For debug mode:
    bug_type: str | None
    bug_location: dict | None  # {line_number: int, code_section: str}
    root_cause: str | None
    # For learning mode:
    code_patterns: list[str] | None
    improvement_opportunities: list[str] | None
    # Common fields:
    concept_category: str
    prerequisites: list[str]
```

#### Hint Generator

**Responsibilities:**

- Generate stage-appropriate hints from analysis
- Include prerequisite concepts in hints
- Ensure progressive specificity (vague → specific → solution)
- Prevent hint stage skipping
- Adapt hints based on mode (debug vs learn)

**Interface:**

```python
class HintGenerator:
    def generate_hint(self, analysis: Analysis, stage: int, prerequisites: list[str]) -> str:
        """Generates a hint for the specified stage with prerequisites"""
        pass

    def _generate_stage1_hint(self, analysis: Analysis, prerequisites: list[str]) -> str:
        """Vague, conceptual hint with basic prerequisites"""
        pass

    def _generate_stage2_hint(self, analysis: Analysis, prerequisites: list[str]) -> str:
        """Specific location hint with detailed prerequisites"""
        pass

    def _generate_stage3_hint(self, analysis: Analysis, prerequisites: list[str]) -> str:
        """Complete solution/explanation with all prerequisites"""
        pass
```

#### Prerequisite Analyzer

**Responsibilities:**

- Identify foundational concepts needed to understand the code/bug
- Rank prerequisites by importance
- Generate brief explanations for each prerequisite

**Interface:**

```python
class PrerequisiteAnalyzer:
    def identify_prerequisites(self, analysis: Analysis, language: str) -> list[Prerequisite]:
        """Identifies prerequisite concepts from the analysis"""
        pass

    def _call_bedrock_for_prerequisites(self, concept_category: str, language: str) -> list[Prerequisite]:
        """Uses Bedrock to identify prerequisites"""
        pass

class Prerequisite:
    concept_name: str
    importance: int  # 1-5, where 5 is most important
    brief_explanation: str
```

#### Image Parser

**Responsibilities:**

- Accept image uploads (PNG, JPG, JPEG, WebP)
- Extract text from images using AWS Textract
- Detect programming language from extracted text
- Handle extraction failures gracefully

**Interface:**

```python
class ImageParser:
    def parse_image(self, image_bytes: bytes, image_format: str) -> ImageParseResult:
        """Extracts code from an image using AWS Textract"""
        pass

    def _call_textract(self, image_bytes: bytes) -> str:
        """Calls AWS Textract API"""
        pass

    def _detect_language(self, code: str) -> str | None:
        """Attempts to detect programming language from code"""
        pass

class ImageParseResult:
    extracted_code: str
    confidence: float  # 0.0 to 1.0
    detected_language: str | None
    success: bool
    error_message: str | None
```

#### Concept Card Generator

**Responsibilities:**

- Generate educational flashcards using AWS Bedrock
- Include prerequisites in concept cards
- Format concept cards with name, definition, prerequisites, examples
- Cache generated cards to reduce API calls

**Interface:**

```python
class ConceptCardGenerator:
    def generate_card(self, analysis: Analysis, prerequisites: list[Prerequisite]) -> ConceptCard:
        """Generates a concept card based on the analysis and prerequisites"""
        pass

    def _build_card_prompt(self, concept_category: str, prerequisites: list[Prerequisite]) -> str:
        """Constructs prompt for concept card generation"""
        pass

class ConceptCard:
    concept_name: str
    definition: str
    prerequisites: list[str]
    examples: list[str]
    related_concepts: list[str]
```

#### Practice Problem Generator

**Responsibilities:**

- Generate practice problems similar to solved problems
- Adjust difficulty based on user performance
- Ensure problems test the same concepts
- Track problem difficulty levels

**Interface:**

```python
class PracticeProblemGenerator:
    def generate_problem(self, analysis: Analysis, difficulty_level: int, concept_category: str) -> PracticeProblem:
        """Generates a practice problem at the specified difficulty level"""
        pass

    def adjust_difficulty(self, current_level: int, user_succeeded: bool) -> int:
        """Adjusts difficulty based on user performance (+1 if succeeded, -1 if failed)"""
        pass

    def _build_problem_prompt(self, concept_category: str, difficulty_level: int) -> str:
        """Constructs prompt for problem generation"""
        pass

class PracticeProblem:
    problem_id: str
    description: str
    difficulty_level: int  # 1-10
    concept_category: str
    starter_code: str | None
    test_cases: list[dict]
    hints: list[str]
```

#### Learning Plan Generator

**Responsibilities:**

- Create structured learning plans based on user goals
- Recommend learning materials (articles, videos, docs)
- Schedule practice problems and tests
- Track progress toward goals

**Interface:**

```python
class LearningPlanGenerator:
    def generate_plan(self, goal: str, current_skill_level: int, target_date: datetime | None) -> LearningPlan:
        """Generates a learning plan for the specified goal"""
        pass

    def recommend_materials(self, concept_category: str) -> list[LearningMaterial]:
        """Recommends learning materials for a concept"""
        pass

    def schedule_tests(self, plan: LearningPlan, is_dsa_focused: bool) -> list[ScheduledTest]:
        """Schedules practice tests throughout the learning plan"""
        pass

    def _build_plan_prompt(self, goal: str, skill_level: int) -> str:
        """Constructs prompt for plan generation"""
        pass

class LearningPlan:
    plan_id: str
    goal: str
    milestones: list[Milestone]
    estimated_duration_days: int
    created_at: datetime

class Milestone:
    milestone_id: str
    title: str
    description: str
    concepts_to_learn: list[str]
    practice_problems: list[str]  # Problem IDs
    learning_materials: list[LearningMaterial]
    is_complete: bool

class LearningMaterial:
    title: str
    type: str  # 'article', 'video', 'documentation', 'tutorial'
    url: str
    estimated_time_minutes: int

class ScheduledTest:
    test_id: str
    scheduled_date: datetime
    concepts_covered: list[str]
    problem_count: int
    difficulty_range: tuple[int, int]
```

#### Progress Tracker

**Responsibilities:**

- Track user progress across sessions
- Monitor practice problem performance
- Update learning plan completion status
- Calculate skill level improvements

**Interface:**

```python
class ProgressTracker:
    def record_session_completion(self, user_id: str, session_id: str, success: bool) -> None:
        """Records completion of a session"""
        pass

    def record_practice_problem(self, user_id: str, problem_id: str, success: bool, time_taken: int) -> None:
        """Records practice problem attempt"""
        pass

    def update_milestone_progress(self, user_id: str, plan_id: str, milestone_id: str) -> None:
        """Updates progress on a learning plan milestone"""
        pass

    def get_user_stats(self, user_id: str) -> UserStats:
        """Retrieves user statistics"""
        pass

class UserStats:
    total_sessions: int
    successful_sessions: int
    current_difficulty_level: int
    concepts_mastered: list[str]
    active_learning_plans: list[str]
    practice_problems_solved: int
```

#### Review Scheduler

**Responsibilities:**

- Schedule weekly review sessions
- Select concepts for review based on time and performance
- Track retention scores for concepts
- Send review reminders

**Interface:**

```python
class ReviewScheduler:
    def schedule_weekly_review(self, user_id: str) -> WeeklyReview:
        """Creates a weekly review session with selected concepts"""
        pass

    def get_concepts_for_review(self, user_id: str) -> list[str]:
        """Selects concepts that need review based on time and performance"""
        pass

    def update_retention_score(self, user_id: str, concept: str, performance: float) -> None:
        """Updates retention score based on review performance"""
        pass

    def send_review_reminder(self, user_id: str, review_id: str) -> None:
        """Sends a reminder for an upcoming review"""
        pass

class WeeklyReview:
    review_id: str
    user_id: str
    scheduled_date: datetime
    concepts_to_review: list[ConceptReview]
    is_complete: bool

class ConceptReview:
    concept_name: str
    last_reviewed: datetime
    retention_score: int  # 0-100
    problems: list[str]  # Problem IDs
```

    examples: list[str]
    related_concepts: list[str]

````

### API Endpoints

#### POST /api/sessions

**Purpose:** Create a new learning session

**Request:**

```json
{
  "code": "string",
  "errorLog": "string | null",
  "language": "string",
  "mode": "debug | learn"
}
````

**Response:**

```json
{
  "sessionId": "string",
  "mode": "debug | learn",
  "message": "Session created successfully"
}
```

#### POST /api/images/parse

**Purpose:** Extract code from an uploaded image (web app only)

**Request:** Multipart form data with image file

**Response:**

```json
{
  "extractedCode": "string",
  "confidence": 0.95,
  "detectedLanguage": "string | null",
  "success": true
}
```

#### GET /api/sessions/{session_id}/hints

**Purpose:** Retrieve a hint for the current or next stage

**Query Parameters:**

- `stage`: integer (1, 2, or 3)

**Response:**

```json
{
  "stage": 1,
  "hint": "string",
  "prerequisites": ["string"],
  "canRequestNext": true
}
```

#### POST /api/sessions/{session_id}/complete

**Purpose:** Mark a session as complete

**Response:**

```json
{
  "message": "Session marked as complete"
}
```

#### GET /api/sessions/{session_id}/concept-card

**Purpose:** Retrieve the concept card for a session

**Response:**

```json
{
  "conceptName": "string",
  "definition": "string",
  "prerequisites": ["string"],
  "examples": ["string"],
  "relatedConcepts": ["string"]
}
```

#### POST /api/practice-problems/generate

**Purpose:** Generate a practice problem based on a completed session

**Request:**

```json
{
  "sessionId": "string",
  "difficultyLevel": 5,
  "conceptCategory": "string"
}
```

**Response:**

```json
{
  "problemId": "string",
  "description": "string",
  "difficultyLevel": 5,
  "starterCode": "string | null",
  "hints": ["string"]
}
```

#### POST /api/practice-problems/{problem_id}/submit

**Purpose:** Submit a solution to a practice problem

**Request:**

```json
{
  "code": "string",
  "userId": "string"
}
```

**Response:**

```json
{
  "success": true,
  "nextDifficultyLevel": 6,
  "feedback": "string",
  "nextProblemAvailable": true
}
```

#### POST /api/learning-plans/create

**Purpose:** Create a new learning plan based on user goals

**Request:**

```json
{
  "userId": "string",
  "goal": "string",
  "currentSkillLevel": 3,
  "targetDate": "2024-12-31T00:00:00Z | null",
  "isDSAFocused": true
}
```

**Response:**

```json
{
  "planId": "string",
  "goal": "string",
  "milestones": [
    {
      "milestoneId": "string",
      "title": "string",
      "conceptsToLearn": ["string"],
      "learningMaterials": [
        {
          "title": "string",
          "type": "article",
          "url": "string",
          "estimatedTimeMinutes": 30
        }
      ]
    }
  ],
  "scheduledTests": [
    {
      "testId": "string",
      "scheduledDate": "2024-06-15T00:00:00Z",
      "conceptsCovered": ["string"]
    }
  ]
}
```

#### GET /api/learning-plans/{plan_id}

**Purpose:** Retrieve a learning plan and its progress

**Response:**

```json
{
  "planId": "string",
  "goal": "string",
  "progress": 45.5,
  "milestones": [
    {
      "milestoneId": "string",
      "title": "string",
      "isComplete": false,
      "practiceProblems": ["string"]
    }
  ]
}
```

#### POST /api/learning-plans/{plan_id}/milestones/{milestone_id}/complete

**Purpose:** Mark a milestone as complete

**Response:**

```json
{
  "message": "Milestone marked as complete",
  "nextMilestone": "string | null"
}
```

#### GET /api/users/{user_id}/stats

**Purpose:** Retrieve user statistics and progress

**Response:**

```json
{
  "totalSessions": 42,
  "successfulSessions": 35,
  "currentDifficultyLevel": 6,
  "conceptsMastered": ["string"],
  "activeLearningPlans": ["string"],
  "practiceProblemsSolved": 28
}
```

#### POST /api/reviews/weekly/schedule

**Purpose:** Schedule a weekly review session

**Request:**

```json
{
  "userId": "string"
}
```

**Response:**

```json
{
  "reviewId": "string",
  "scheduledDate": "2024-06-22T00:00:00Z",
  "conceptsToReview": [
    {
      "conceptName": "string",
      "retentionScore": 75,
      "problemCount": 3
    }
  ]
}
```

#### GET /api/reviews/{review_id}

**Purpose:** Retrieve a weekly review session

**Response:**

```json
{
  "reviewId": "string",
  "scheduledDate": "2024-06-22T00:00:00Z",
  "conceptsToReview": [
    {
      "conceptName": "string",
      "lastReviewed": "2024-06-15T00:00:00Z",
      "retentionScore": 75,
      "problems": ["problem_id_1", "problem_id_2"]
    }
  ],
  "isComplete": false
}
```

#### POST /api/reviews/{review_id}/complete

**Purpose:** Mark a weekly review as complete and update retention scores

**Request:**

```json
{
  "conceptPerformance": [
    {
      "conceptName": "string",
      "performance": 0.85
    }
  ]
}
```

**Response:**

```json
{
  "message": "Review completed",
  "updatedRetentionScores": [
    {
      "conceptName": "string",
      "newRetentionScore": 82
    }
  ]
}
```

## Data Models

### DynamoDB Schema

**Table: nyx-ai-sessions**

```python
{
  "session_id": "string",  # Partition Key
  "code": "string",
  "error_log": "string",
  "language": "string",
  "bug_analysis": {
    "bug_type": "string",
    "bug_location": {
      "line_number": "number",
      "code_section": "string"
    },
    "root_cause": "string",
    "concept_category": "string"
  },
  "current_hint_stage": "number",
  "hints_generated": {
    "stage1": "string",
    "stage2": "string",
    "stage3": "string"
  },
  "concept_card": {
    "concept_name": "string",
    "definition": "string",
    "examples": ["string"],
    "related_concepts": ["string"]
  },
  "is_complete": "boolean",
  "created_at": "string",  # ISO 8601 timestamp
  "ttl": "number"  # Unix timestamp for DynamoDB TTL
}
```

**Table: nyx-ai-practice-problems**

```python
{
  "problem_id": "string",  # Partition Key
  "description": "string",
  "difficulty_level": "number",
  "concept_category": "string",
  "starter_code": "string | null",
  "test_cases": [
    {
      "input": "string",
      "expected_output": "string"
    }
  ],
  "hints": ["string"],
  "created_at": "string"
}
```

**Table: nyx-ai-learning-plans**

```python
{
  "plan_id": "string",  # Partition Key
  "user_id": "string",  # GSI Partition Key
  "goal": "string",
  "milestones": [
    {
      "milestone_id": "string",
      "title": "string",
      "description": "string",
      "concepts_to_learn": ["string"],
      "practice_problems": ["string"],
      "learning_materials": [
        {
          "title": "string",
          "type": "string",
          "url": "string",
          "estimated_time_minutes": "number"
        }
      ],
      "is_complete": "boolean"
    }
  ],
  "scheduled_tests": [
    {
      "test_id": "string",
      "scheduled_date": "string",
      "concepts_covered": ["string"],
      "problem_count": "number",
      "difficulty_range": ["number", "number"]
    }
  ],
  "estimated_duration_days": "number",
  "created_at": "string",
  "updated_at": "string"
}
```

**Table: nyx-ai-user-progress**

```python
{
  "user_id": "string",  # Partition Key
  "total_sessions": "number",
  "successful_sessions": "number",
  "current_difficulty_level": "number",
  "concepts_mastered": ["string"],
  "active_learning_plans": ["string"],
  "practice_problems_solved": "number",
  "session_history": [
    {
      "session_id": "string",
      "timestamp": "string",
      "success": "boolean",
      "concept_category": "string"
    }
  ],
  "practice_history": [
    {
      "problem_id": "string",
      "timestamp": "string",
      "success": "boolean",
      "time_taken_seconds": "number"
    }
  ],
  "concept_retention": {
    "concept_name": {
      "retention_score": "number",  # 0-100
      "last_reviewed": "string",
      "review_count": "number"
    }
  },
  "updated_at": "string"
}
```

**Table: nyx-ai-weekly-reviews**

```python
{
  "review_id": "string",  # Partition Key
  "user_id": "string",  # GSI Partition Key
  "scheduled_date": "string",
  "concepts_to_review": [
    {
      "concept_name": "string",
      "last_reviewed": "string",
      "retention_score": "number",
      "problems": ["string"]
    }
  ],
  "is_complete": "boolean",
  "completion_date": "string | null",
  "performance_results": [
    {
      "concept_name": "string",
      "performance": "number"  # 0.0 to 1.0
    }
  ],
  "created_at": "string"
}
```

### AWS Bedrock Prompt Templates

**Analysis Prompt Template:**

```
You are a computer science professor helping a student debug their code.
Your role is to analyze the code and error, but DO NOT provide the solution directly.

Code:
{code}

Error Log:
{error_log}

Language: {language}

Analyze this code and identify:
1. The type of bug (e.g., off-by-one error, null pointer, type mismatch)
2. The exact location (line number and code section)
3. The root cause of the bug
4. The general concept category this bug falls under

Respond in JSON format:
{
  "bug_type": "string",
  "bug_location": {"line_number": number, "code_section": "string"},
  "root_cause": "string",
  "concept_category": "string"
}
```

**Concept Card Prompt Template:**

```
Create an educational flashcard for the following programming concept: {concept_category}

The flashcard should include:
1. A clear, concise name for the concept
2. A definition that a junior developer can understand
3. The prerequisite concepts needed to understand this concept
4. 2-3 common examples of this concept in practice
5. Related concepts the student should learn

Respond in JSON format:
{
  "concept_name": "string",
  "definition": "string",
  "prerequisites": ["string"],
  "examples": ["string"],
  "related_concepts": ["string"]
}
```

**Practice Problem Generation Prompt Template:**

```
Generate a practice coding problem for the concept: {concept_category}

Difficulty level: {difficulty_level} (1-10 scale)
Language: {language}

The problem should:
1. Test understanding of {concept_category}
2. Be at difficulty level {difficulty_level}
3. Include a clear description
4. Provide starter code if appropriate
5. Include 3-5 test cases
6. Provide 2-3 progressive hints

Respond in JSON format:
{
  "description": "string",
  "starter_code": "string | null",
  "test_cases": [{"input": "string", "expected_output": "string"}],
  "hints": ["string"]
}
```

**Learning Plan Generation Prompt Template:**

```
Create a structured learning plan for the following goal: {goal}

Current skill level: {skill_level} (1-10 scale)
Target date: {target_date}
DSA focused: {is_dsa_focused}

The plan should include:
1. 5-8 milestones with clear titles and descriptions
2. Concepts to learn for each milestone
3. Recommended learning materials (articles, videos, documentation)
4. Estimated duration in days
5. If DSA focused, include scheduled practice tests

Respond in JSON format:
{
  "milestones": [
    {
      "title": "string",
      "description": "string",
      "concepts_to_learn": ["string"],
      "learning_materials": [
        {
          "title": "string",
          "type": "article|video|documentation",
          "url": "string",
          "estimated_time_minutes": number
        }
      ]
    }
  ],
  "estimated_duration_days": number,
  "scheduled_tests": [
    {
      "scheduled_date": "string",
      "concepts_covered": ["string"],
      "problem_count": number,
      "difficulty_range": [number, number]
    }
  ]
}
```

### Extension Data Storage

_A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees._

### Property 1: Code Formatting Preservation

_For any_ code snippet with formatting and syntax, when pasted into the Code_Editor, the formatting and syntax highlighting should be preserved in the rendered output.

**Validates: Requirements 1.2**

### Property 2: Multi-Language Support

_For any_ supported programming language, when code in that language is submitted, the System should accept and process it without language-specific errors.

**Validates: Requirements 1.3**

### Property 3: Error Log Storage

_For any_ code submission with an error log, the error log should be stored alongside the code in the session data.

**Validates: Requirements 1.4**

### Property 4: Empty Input Validation

_For any_ input that is empty or contains only whitespace, the Code_Editor should prevent submission and maintain the current state.

**Validates: Requirements 1.5**

### Property 5: Bedrock API Invocation

_For any_ code and error log submission, the Analysis_Engine should make an API call to AWS Bedrock Claude 3 Sonnet.

**Validates: Requirements 2.1, 7.2**

### Property 6: Bug Analysis Structure

_For any_ completed analysis, the result should contain bug location, root cause, and bug type fields, and should not contain direct solution code or fix instructions.

**Validates: Requirements 2.2, 2.4**

### Property 7: Progressive Hint Specificity

_For any_ bug analysis, the hints generated for stages 1, 2, and 3 should increase in specificity, where stage 1 contains no location references, stage 2 contains line numbers or code sections, and stage 3 contains the complete solution.

**Validates: Requirements 3.1, 3.2, 3.3, 3.5**

### Property 8: Hint Stage Enforcement

_For any_ session, requesting stage 3 hints should be rejected if stages 1 and 2 have not been viewed.

**Validates: Requirements 3.6**

### Property 9: Session State Management

_For any_ session, the system should track the current hint stage, update it when hints are requested, and mark the session as complete when resolved, with all state changes persisted to storage.

**Validates: Requirements 5.2, 5.3, 5.4**

### Property 10: Unique Session Creation

_For any_ code submission, a new session should be created with a unique session ID that doesn't collide with existing sessions.

**Validates: Requirements 5.1**

### Property 11: Session TTL

_For any_ newly created session, the TTL (time-to-live) should be set to at least 24 hours from creation time.

**Validates: Requirements 5.5**

### Property 12: Concept Card Completeness

_For any_ generated concept card, it should contain a concept name, definition, examples list, and related concepts list, and should be generated using AWS Bedrock.

**Validates: Requirements 4.2, 4.5**

### Property 13: Concept Card Triggering

_For any_ session that is marked as complete or where stage 3 hints are viewed, a concept card should be generated.

**Validates: Requirements 4.1**

### Property 14: Concept Card Persistence

_For any_ concept card, the save/export functionality should successfully store or export the card data.

**Validates: Requirements 4.4**

### Property 15: API Retry Logic

_For any_ AWS Bedrock API call that fails, the system should retry up to 3 times before returning an error to the user.

**Validates: Requirements 6.2**

### Property 16: Error Message Quality

_For any_ error condition (validation failure, network error, etc.), the error message should be specific about the problem and should not contain stack traces or internal technical details.

**Validates: Requirements 6.3, 6.5**

### Property 17: Rate Limiting

_For any_ sequence of rapid API requests exceeding the configured threshold, the system should apply rate limiting and reject excess requests.

**Validates: Requirements 7.3**

### Property 18: API Logging

_For any_ AWS Bedrock API call, the system should create a log entry containing the request details and response status.

**Validates: Requirements 7.5**

### Property 19: UI Component Rendering

_For any_ application state (hint displayed, concept card generated, etc.), the corresponding UI components should be rendered in their designated containers (hint panel, modal, etc.).

**Validates: Requirements 8.2, 8.3, 8.5**

### Property 20: Syntax Highlighting

_For any_ supported programming language, when code in that language is displayed in the Code_Editor, syntax highlighting should be applied.

**Validates: Requirements 8.4**

### Property 21: API Endpoint Functionality

_For any_ valid API request to the code submission, hint retrieval, session completion, or concept card endpoints, the system should process the request and return the appropriate response.

**Validates: Requirements 9.1, 9.2, 9.3, 9.4**

### Property 22: API Error Handling

_For any_ malformed API request, the system should return an appropriate HTTP status code (4xx) and a descriptive error message.

**Validates: Requirements 9.5**

### Property 23: CORS Headers

_For any_ API response, the system should include CORS headers allowing requests from authorized origins.

**Validates: Requirements 9.6**

### Property 24: Session Data Deletion

_For any_ session that has expired (TTL reached), the session data should be deleted from storage.

**Validates: Requirements 10.1**

### Property 25: Input Sanitization

_For any_ user input (code, error logs, etc.), the system should sanitize the input to remove or escape potentially malicious content before processing.

**Validates: Requirements 10.3**

### Property 26: API Authentication

_For any_ API request without valid authentication credentials, the system should reject the request with a 401 or 403 status code.

**Validates: Requirements 10.4**

### Property 27: Data Encryption

_For any_ sensitive session data stored in DynamoDB, the data should be encrypted at rest.

**Validates: Requirements 10.5**

### Property 28: Image Code Extraction

_For any_ valid image file in a supported format (PNG, JPG, JPEG, WebP), the Image_Parser should extract text and return it as code.

**Validates: Requirements 1.4, 12.2, 12.3**

### Property 29: Learning Mode Activation

_For any_ code submission without error logs, the System should operate in learning mode and generate learning-focused analysis.

**Validates: Requirements 1.6, 13.1**

### Property 30: Learning Mode Analysis Content

_For any_ learning mode analysis, the result should contain code patterns and improvement opportunities rather than bug information.

**Validates: Requirements 2.3, 13.2**

### Property 31: Prerequisite Identification

_For any_ code analysis (debug or learning mode), the Prerequisite_Analyzer should identify and include foundational concepts needed to understand the topic.

**Validates: Requirements 3.2**

### Property 32: Prerequisites in Hints

_For any_ generated hint, the hint should include prerequisite concepts with brief explanations.

**Validates: Requirements 3.3**

### Property 33: Prerequisites in Concept Cards

_For any_ generated concept card, the card should include a list of prerequisite concepts needed to understand the main concept.

**Validates: Requirements 4.2**

### Property 34: Extension API Integration

_For any_ code selection in the editor extension, activating the analyze command should send the code to the backend API.

**Validates: Requirements 11.4**

### Property 35: Extension Hint Display

_For any_ hint received by the extension, the hint should be displayed in a dedicated panel within the editor.

**Validates: Requirements 11.5**

### Property 36: Extension Stage Support

_For any_ hint stage (1, 2, or 3), the extension should support requesting and displaying that stage.

**Validates: Requirements 11.6**

### Property 37: Extension Concept Card Display

_For any_ concept card generated during an extension session, the card should be displayed within the editor interface.

**Validates: Requirements 11.7**

### Property 38: Extension Session Persistence

_For any_ extension session, the session state should be maintained throughout the editor session and preserved for 24 hours after the editor closes.

**Validates: Requirements 11.8, 11.9**

### Property 39: Image Format Support

_For any_ common image format (PNG, JPG, JPEG, WebP), the Image_Parser should accept and process the image.

**Validates: Requirements 12.2**

### Property 40: Extracted Text Population

_For any_ successful image text extraction, the extracted code should populate the code editor field.

**Validates: Requirements 12.4**

### Property 41: Empty Extraction Validation

_For any_ image extraction that results in empty or whitespace-only text, the System should prevent submission.

**Validates: Requirements 12.6**

### Property 42: OCR Service Usage

_For any_ image processing request, the Image_Parser should use AWS Textract for OCR.

**Validates: Requirements 12.7**

### Property 43: Learning Mode Hint Content

_For any_ hint generated in learning mode, the hint should focus on understanding how the code works rather than fixing bugs.

**Validates: Requirements 13.3**

### Property 44: Learning Mode Improvement Suggestions

_For any_ hint generated in learning mode, the hint should include potential improvements or alternative approaches.

**Validates: Requirements 13.4**

### Property 45: Learning Mode Concept Cards

_For any_ concept card generated in learning mode, the card should explain concepts used in the code.

**Validates: Requirements 13.5**

### Property 46: Mode Indication

_For any_ active session, the System should clearly display whether it is operating in debug mode or learning mode.

**Validates: Requirements 13.6**

### Property 47: Practice Problem Generation on Completion

_For any_ completed debugging session, the System should generate a practice problem with the same concept category and difficulty level.

**Validates: Requirements 14.1**

### Property 48: Difficulty Progression on Success

_For any_ successfully solved practice problem, the System should offer the next problem at difficulty level +1.

**Validates: Requirements 14.2**

### Property 49: Practice Progress Tracking

_For any_ practice problem attempt, the System should record the attempt in the user's progress history.

**Validates: Requirements 14.3**

### Property 50: Concept Consistency in Practice Problems

_For any_ generated practice problem, the problem should test the same concept category as the original session.

**Validates: Requirements 14.4**

### Property 51: Practice Problem Availability

_For any_ user request for practice problems, the System should generate and return a problem.

**Validates: Requirements 14.5**

### Property 52: Difficulty Reduction on Failure

_For any_ failed practice problem attempt, the System should offer the next problem at difficulty level -1 (minimum 1).

**Validates: Requirements 14.6**

### Property 53: Learning Plan Generation

_For any_ learning goal, the System should generate a structured plan with milestones, concepts, and materials.

**Validates: Requirements 15.2**

### Property 54: Learning Material Recommendations

_For any_ generated learning plan, the plan should include learning materials (articles, videos, docs) relevant to the goal.

**Validates: Requirements 15.3**

### Property 55: Progressive Practice Problems in Plans

_For any_ learning plan, the practice problems should be ordered by increasing difficulty level.

**Validates: Requirements 15.4**

### Property 56: DSA Test Scheduling

_For any_ learning plan where the goal includes "DSA" or "Data Structures and Algorithms", the plan should include scheduled practice tests.

**Validates: Requirements 15.5**

### Property 57: Learning Plan Progress Tracking

_For any_ milestone completion, the System should update the plan's progress percentage and mark the milestone as complete.

**Validates: Requirements 15.6**

### Property 58: Milestone Completion Feedback

_For any_ completed milestone, the System should return feedback and the next milestone ID (if available).

**Validates: Requirements 15.7**

### Property 59: Learning Plan Regeneration

_For any_ learning plan where the goal is updated, the System should regenerate the plan with new milestones and materials.

**Validates: Requirements 15.8**

### Property 60: Weekly Review Scheduling

_For any_ completed concept, the System should schedule a weekly review session 7 days after completion.

**Validates: Requirements 16.1**

### Property 61: Time-Based Review Inclusion

_For any_ concept learned 7 or more days ago, the System should include it in the next weekly review.

**Validates: Requirements 16.2**

### Property 62: Review Problem Coverage

_For any_ weekly review session, the problems should cover all concepts listed in the review.

**Validates: Requirements 16.3**

### Property 63: Struggling Concept Prioritization

_For any_ weekly review, concepts with retention scores below 70 should appear before concepts with higher scores.

**Validates: Requirements 16.4**

### Property 64: Retention Score Updates

_For any_ completed weekly review, the retention score for each concept should be updated based on the user's performance (0.0-1.0).

**Validates: Requirements 16.5**

### Property 65: Review Reminders

_For any_ scheduled weekly review, the System should send a reminder 24 hours before the scheduled date.

**Validates: Requirements 16.6**

### Property 66: Low-Performance Concept Tracking

_For any_ concept with a retention score below 60, the System should flag it for additional review in the next session.

**Validates: Requirements 16.7**

## Error Handling

### Error Categories

**1. User Input Errors**

- Empty code submission → Return 400 with message "Code cannot be empty"
- Invalid language specified → Return 400 with message "Unsupported language: {language}"
- Malformed JSON in API requests → Return 400 with message "Invalid request format"

**2. AWS Bedrock Errors**

- Connection timeout → Retry up to 3 times, then return 503 with message "Analysis service temporarily unavailable"
- Rate limit exceeded → Return 429 with message "Too many requests, please try again later"
- Invalid API credentials → Log error, return 500 with message "Service configuration error"

**3. Session Errors**

- Session not found → Return 404 with message "Session not found or expired"
- Invalid hint stage request → Return 400 with message "Cannot skip to stage {N} without viewing previous stages"
- Session already complete → Return 409 with message "Session already marked as complete"

**4. DynamoDB Errors**

- Write failure → Retry up to 3 times, then return 500 with message "Failed to save session data"
- Read failure → Retry up to 3 times, then return 500 with message "Failed to retrieve session data"

**5. Image Processing Errors**

- Unsupported image format → Return 400 with message "Unsupported image format. Please use PNG, JPG, JPEG, or WebP"
- Image too large → Return 413 with message "Image file too large. Maximum size is 10MB"
- OCR extraction failed → Return 500 with message "Could not extract text from image. Please try a clearer image or paste code directly"
- Empty extraction result → Return 400 with message "No code detected in image. Please try a different image or paste code directly"

**6. Extension Errors**

- No code selected → Display notification "Please select code to analyze"
- API connection failed → Display notification "Could not connect to Nyx AI service. Please check your internet connection"
- Session expired → Display notification "Your session has expired. Please start a new analysis"

**7. Practice Problem Errors**

- Problem generation failed → Return 500 with message "Could not generate practice problem. Please try again"
- Invalid difficulty level → Return 400 with message "Difficulty level must be between 1 and 10"
- Problem not found → Return 404 with message "Practice problem not found"
- Solution validation failed → Return 400 with message "Could not validate solution. Please check your code"

**8. Learning Plan Errors**

- Invalid goal → Return 400 with message "Please provide a valid learning goal"
- Plan not found → Return 404 with message "Learning plan not found"
- Milestone not found → Return 404 with message "Milestone not found in plan"
- Cannot complete milestone → Return 409 with message "Previous milestones must be completed first"
- Plan generation failed → Return 500 with message "Could not generate learning plan. Please try again"

**9. Weekly Review Errors**

- Review not found → Return 404 with message "Weekly review not found"
- Review already completed → Return 409 with message "This review has already been completed"
- No concepts to review → Return 400 with message "No concepts available for review yet"
- Invalid performance data → Return 400 with message "Performance scores must be between 0.0 and 1.0"

### Error Response Format

All API errors follow this structure:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {} // Optional additional context
  }
}
```

### Logging Strategy

- **INFO**: Session creation, hint requests, concept card generation, image uploads, extension activations, mode switches, practice problem generation, learning plan creation, milestone completion
- **WARN**: Retry attempts, rate limiting triggered, low OCR confidence scores, difficulty level adjustments
- **ERROR**: API failures after retries, unexpected exceptions, image processing failures, problem generation failures, plan generation failures
- All logs include session_id or user_id for traceability
- Extension logs include editor type (Kiro/VS Code) and version
- Practice problem logs include problem_id and difficulty_level
- Learning plan logs include plan_id and milestone_id

## Testing Strategy

### Dual Testing Approach

The testing strategy employs both unit tests and property-based tests to ensure comprehensive coverage:

- **Unit tests**: Verify specific examples, edge cases, and error conditions
- **Property tests**: Verify universal properties across all inputs using randomized test data

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property tests verify general correctness across a wide range of inputs.

### Property-Based Testing Configuration

**Library Selection:**

- **Frontend (TypeScript/React)**: fast-check
- **Backend (Python)**: Hypothesis

**Test Configuration:**

- Each property test must run a minimum of 100 iterations
- Each test must include a comment tag referencing the design property
- Tag format: `# Feature: nyx-ai, Property {N}: {property_text}`

**Property Test Implementation:**

- Each correctness property listed above must be implemented as a single property-based test
- Tests should generate random valid inputs (code snippets, session IDs, error logs, etc.)
- Tests should verify the property holds for all generated inputs

### Unit Testing Focus

Unit tests should focus on:

- Specific examples demonstrating correct behavior (e.g., a known off-by-one error being analyzed correctly)
- Edge cases (e.g., empty error logs, very long code submissions, special characters)
- Error conditions (e.g., Bedrock API returning malformed JSON)
- Integration points between components (e.g., Session Manager → Analysis Engine → Hint Generator flow)

### Test Coverage Goals

- **Backend**: Minimum 80% code coverage
- **Frontend**: Minimum 70% code coverage for business logic components
- **API Endpoints**: 100% coverage of all endpoints and error paths

### Example Property Test (Python/Hypothesis)

```python
from hypothesis import given, strategies as st
import pytest

# Feature: nyx-ai, Property 10: Unique Session Creation
@given(
    code=st.text(min_size=1),
    error_log=st.text(),
    language=st.sampled_from(['python', 'javascript', 'java'])
)
def test_unique_session_creation(code, error_log, language):
    """For any code submission, a new session should be created with a unique session ID"""
    session_manager = SessionManager()

    session_id_1 = session_manager.create_session(code, error_log, language)
    session_id_2 = session_manager.create_session(code, error_log, language)

    assert session_id_1 != session_id_2
    assert len(session_id_1) > 0
    assert len(session_id_2) > 0
```

### Example Property Test (TypeScript/fast-check)

```typescript
import fc from "fast-check";

// Feature: nyx-ai, Property 4: Empty Input Validation
test("empty input validation", () => {
  fc.assert(
    fc.property(
      fc.oneof(fc.constant(""), fc.stringOf(fc.constantFrom(" ", "\t", "\n"))),
      (emptyInput) => {
        // For any input that is empty or whitespace, submission should be prevented
        const result = validateCodeInput(emptyInput);
        expect(result.isValid).toBe(false);
        expect(result.error).toContain("empty");
      },
    ),
    { numRuns: 100 },
  );
});
```

### Integration Testing

Integration tests should verify:

- End-to-end flow: Code submission → Analysis → Hint generation → Concept card
- Learning mode flow: Code without errors → Learning analysis → Understanding hints → Concept card
- Practice flow: Session completion → Practice problem generation → Solution submission → Difficulty adjustment
- Learning plan flow: Goal setting → Plan generation → Material recommendations → Milestone completion → Progress tracking
- Image upload flow: Image upload → OCR extraction → Code population → Analysis
- Extension flow: Code selection → API call → Hint display → Concept card display
- AWS Bedrock integration (using mocked responses in CI, real API in staging)
- AWS Textract integration (using mocked responses in CI, real API in staging)
- DynamoDB operations (using local DynamoDB in tests)
- Frontend-backend API integration
- Extension-backend API integration

### Extension Testing

**Kiro Extension:**

- Test extension activation and command registration
- Test code selection and analysis triggering
- Test hint panel rendering
- Test concept card webview rendering
- Test session state persistence

**VS Code Extension:**

- Test extension activation and command registration
- Test code selection and analysis triggering
- Test webview creation for hints
- Test webview creation for concept cards
- Test workspace state management

### Manual Testing Checklist

While most functionality is covered by automated tests, manual testing should verify:

- UI/UX flow feels natural and educational
- Hints are pedagogically appropriate (not too revealing, not too vague)
- Prerequisites are helpful and correctly identified
- Concept cards are accurate and helpful
- Learning mode provides valuable insights for working code
- Practice problems are at appropriate difficulty levels
- Difficulty progression feels smooth and motivating
- Learning plans are comprehensive and achievable
- Recommended learning materials are relevant and high-quality
- Scheduled tests are appropriately spaced
- Progress tracking is accurate and motivating
- Image upload and OCR extraction work with real-world screenshots
- Extensions integrate smoothly with editor workflows
- Visual design and accessibility

## Responsive Design Properties

### Property 67: Responsive Layout Adaptation

**Statement:** For any viewport width, the application SHALL render with an appropriate layout that maintains usability and readability.

**Formal Definition:**

```
∀ viewport_width ∈ [320px, 3840px]:
  layout = render_application(viewport_width)

  IF viewport_width ≤ 767px THEN
    assert layout.columns == 1
    assert layout.navigation_type == "hamburger_menu"
    assert layout.panel_display == "tabbed"

  ELSE IF 768px ≤ viewport_width ≤ 1023px THEN
    assert layout.columns ∈ {1, 2}
    assert layout.navigation_type == "tab_navigation"
    assert layout.panel_display == "collapsible"

  ELSE IF viewport_width ≥ 1024px THEN
    assert layout.columns ∈ {2, 3}
    assert layout.navigation_type == "full_navbar"
    assert layout.panel_display == "side_by_side"

  assert layout.is_usable == True
  assert layout.content_overflow == False
```

**Test Strategy:**

- Use viewport testing tools (Cypress viewport commands, Playwright)
- Test at standard breakpoints: 375px (mobile), 768px (tablet), 1024px (desktop), 1920px (large desktop)
- Verify no horizontal scrolling on any viewport
- Verify all interactive elements are accessible

**Validates:** Requirements 17.1, 17.2, 17.3, 17.4, 17.7

---

### Property 68: Touch Target Sizing

**Statement:** For any interactive element on mobile devices, the touch target SHALL be at least 44x44 pixels to ensure accessibility.

**Formal Definition:**

```
∀ element ∈ interactive_elements:
  IF viewport_width ≤ 767px THEN
    dimensions = get_element_dimensions(element)
    assert dimensions.width ≥ 44px
    assert dimensions.height ≥ 44px
    assert element.has_adequate_spacing == True
```

**Test Strategy:**

- Query all buttons, links, and interactive elements
- Measure computed dimensions on mobile viewport
- Verify spacing between adjacent touch targets (minimum 8px)
- Test with actual touch events on mobile devices

**Validates:** Requirements 17.6

---

### Property 69: Mobile Navigation Functionality

**Statement:** On mobile devices, navigation SHALL be accessible through a hamburger menu that reveals all navigation options without obscuring content.

**Formal Definition:**

```
∀ viewport_width ≤ 767px:
  navigation = get_navigation_component()

  assert navigation.type == "hamburger_menu"
  assert navigation.is_visible == True
  assert navigation.icon_size ≥ 44px

  WHEN user_clicks(navigation.icon):
    menu = get_menu_state()
    assert menu.is_open == True
    assert menu.options.length > 0
    assert menu.obscures_content == False
    assert menu.is_dismissible == True
```

**Test Strategy:**

- Test hamburger menu visibility on mobile viewport
- Verify menu opens/closes on click/tap
- Verify all navigation items are present
- Test menu dismissal (click outside, close button)
- Verify menu doesn't prevent access to underlying content when closed

**Validates:** Requirements 17.4, 17.7

---
