# Requirements Document

## Introduction

Kod-Guru is a web-based educational tool designed to help computer science students and junior developers develop debugging skills through Socratic questioning rather than immediate code fixes. The system analyzes broken code and guides users to discover bugs themselves through progressive hints, fostering deeper understanding of programming concepts.

## Glossary

- **Kod_Guru_System**: The complete web-based educational debugging platform
- **Code_Editor**: The frontend interface component where users input broken code
- **Socratic_Engine**: The AI-powered analysis component that generates guiding questions
- **Hint_Stage**: A level of hint specificity (Stage 1: Vague, Stage 2: Specific, Stage 3: Solution)
- **Concept_Card**: An educational flashcard explaining the underlying programming concept
- **User_Session**: A single debugging interaction from code submission to resolution
- **Error_Log**: The error message or stack trace provided by the user
- **Bug_Analysis**: The internal identification of the code issue by the AI

## Requirements

### Requirement 1: Code Input and Submission

**User Story:** As a student, I want to submit my broken code and error messages, so that I can receive guidance on debugging.

#### Acceptance Criteria

1. WHEN a user accesses the application, THE Code_Editor SHALL display an interface for code input
2. WHEN a user pastes code into the editor, THE Code_Editor SHALL preserve formatting and syntax highlighting
3. WHEN a user submits code, THE Kod_Guru_System SHALL accept code in multiple programming languages (Python, JavaScript, Java, C++)
4. WHEN a user provides an error log, THE Kod_Guru_System SHALL accept and store the error message alongside the code
5. WHEN code submission is initiated, THE Kod_Guru_System SHALL validate that both code and error information are present

### Requirement 2: Socratic Analysis and Question Generation

**User Story:** As a student, I want to receive guiding questions instead of direct answers, so that I can develop my debugging skills.

#### Acceptance Criteria

1. WHEN code is submitted, THE Socratic_Engine SHALL analyze the code and error log using AWS Bedrock Claude 3 Sonnet
2. WHEN the bug is identified, THE Socratic_Engine SHALL NOT reveal the solution directly in its response
3. WHEN generating questions, THE Socratic_Engine SHALL create questions that guide the user toward the bug location
4. WHEN the analysis is complete, THE Kod_Guru_System SHALL return the first hint within 5 seconds
5. WHEN multiple bugs exist, THE Socratic_Engine SHALL focus on the most critical bug first

### Requirement 3: Progressive Hint System

**User Story:** As a student, I want increasingly specific hints if I'm stuck, so that I can get help without losing the learning opportunity.

#### Acceptance Criteria

1. WHEN a User_Session begins, THE Kod_Guru_System SHALL start at Hint_Stage 1 with a vague, conceptual hint
2. WHEN a user requests the next hint, THE Kod_Guru_System SHALL progress to Hint_Stage 2 with a specific line or location reference
3. WHEN a user requests the final hint, THE Kod_Guru_System SHALL progress to Hint_Stage 3 revealing the fix and explanation
4. WHEN at Hint_Stage 1, THE Socratic_Engine SHALL provide hints about general concepts or areas (e.g., "Have you checked your loop variables?")
5. WHEN at Hint_Stage 2, THE Socratic_Engine SHALL provide hints with specific line numbers or code sections (e.g., "Look at line 14, is the condition correct?")
6. WHEN at Hint_Stage 3, THE Socratic_Engine SHALL provide the corrected code and a detailed explanation of the bug
7. WHEN a user is at Hint_Stage 3, THE Kod_Guru_System SHALL NOT allow further hint requests for the current bug

### Requirement 4: Concept Card Generation

**User Story:** As a student, I want to learn the underlying concept after fixing a bug, so that I can avoid similar mistakes in the future.

#### Acceptance Criteria

1. WHEN a bug is resolved (either by user or through Hint_Stage 3), THE Kod_Guru_System SHALL generate a Concept_Card
2. WHEN generating a Concept_Card, THE Socratic_Engine SHALL identify the underlying programming concept (e.g., "Off-By-One Error", "Null Pointer Exception")
3. WHEN displaying a Concept_Card, THE Kod_Guru_System SHALL include a title, definition, and example
4. WHEN a Concept_Card is created, THE Kod_Guru_System SHALL store it for the user's future reference
5. WHEN a user completes a session, THE Kod_Guru_System SHALL display the relevant Concept_Card

### Requirement 5: User Interaction and Session Management

**User Story:** As a student, I want to control my debugging session, so that I can learn at my own pace.

#### Acceptance Criteria

1. WHEN a user views a hint, THE Kod_Guru_System SHALL provide a button to request the next hint level
2. WHEN a user believes they've fixed the bug, THE Kod_Guru_System SHALL allow code resubmission for validation
3. WHEN a user resubmits code, THE Socratic_Engine SHALL analyze whether the bug is fixed
4. WHEN the bug is confirmed fixed, THE Kod_Guru_System SHALL congratulate the user and display the Concept_Card
5. WHEN a user wants to start over, THE Kod_Guru_System SHALL allow session reset with new code submission
6. WHEN a User_Session is active, THE Kod_Guru_System SHALL maintain session state including current Hint_Stage and conversation history

### Requirement 6: Backend API and LLM Integration

**User Story:** As a system, I want to process code analysis requests efficiently, so that users receive timely feedback.

#### Acceptance Criteria

1. THE Kod_Guru_System SHALL implement a REST API using Python and FastAPI
2. WHEN the API receives a code analysis request, THE Kod_Guru_System SHALL invoke AWS Bedrock Claude 3 Sonnet
3. WHEN calling AWS Bedrock, THE Kod_Guru_System SHALL include system prompts that enforce Socratic teaching methodology
4. WHEN AWS Bedrock returns a response, THE Kod_Guru_System SHALL parse and structure the hint appropriately
5. THE Kod_Guru_System SHALL deploy the backend API on AWS Lambda for scalability
6. WHEN an API error occurs, THE Kod_Guru_System SHALL return a meaningful error message to the frontend

### Requirement 7: Frontend User Interface

**User Story:** As a student, I want an intuitive interface, so that I can focus on learning rather than navigating the tool.

#### Acceptance Criteria

1. THE Kod_Guru_System SHALL implement the frontend using React.js
2. WHEN displaying the Code_Editor, THE Kod_Guru_System SHALL provide syntax highlighting for supported languages
3. WHEN hints are received, THE Kod_Guru_System SHALL display them in a clear, readable format
4. WHEN showing Hint_Stage progression, THE Kod_Guru_System SHALL indicate which stage the user is currently on (1, 2, or 3)
5. WHEN displaying Concept_Cards, THE Kod_Guru_System SHALL use a visually distinct card-style component
6. WHEN the user is waiting for analysis, THE Kod_Guru_System SHALL display a loading indicator

### Requirement 8: Error Handling and Edge Cases

**User Story:** As a student, I want the system to handle unexpected situations gracefully, so that I can continue learning without frustration.

#### Acceptance Criteria

1. WHEN code submission contains no detectable errors, THE Kod_Guru_System SHALL inform the user that no bugs were found
2. WHEN the submitted code is syntactically invalid, THE Kod_Guru_System SHALL provide a hint about syntax errors
3. WHEN AWS Bedrock is unavailable, THE Kod_Guru_System SHALL display an error message and suggest trying again later
4. WHEN the code is too large (>10,000 characters), THE Kod_Guru_System SHALL reject the submission with a size limit message
5. WHEN the error log is missing, THE Kod_Guru_System SHALL prompt the user to provide error information
6. WHEN network errors occur, THE Kod_Guru_System SHALL display a user-friendly error message and retry option
