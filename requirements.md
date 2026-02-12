# Requirements Document

## Introduction

Kod-Guru is a Socratic learning tool designed to help computer science students and junior developers learn debugging and programming concepts through guided discovery rather than immediate solutions. The system analyzes code (broken or working) and optionally error logs, then guides users through a progressive hinting system with prerequisite concepts to help them understand and improve their code, reinforcing learning through active problem-solving. The tool is available as a web application and as editor extensions for Kiro and VS Code.

## Glossary

- **System**: The Kod-Guru application (web app, Kiro extension, or VS Code extension)
- **Code_Editor**: The frontend interface component for code input
- **Analysis_Engine**: The backend component that uses AWS Bedrock to analyze code
- **Hint_Generator**: The component responsible for generating progressive hints with prerequisites
- **Concept_Card_Generator**: The component that creates educational flashcards
- **Prerequisite_Analyzer**: The component that identifies foundational concepts needed to understand the code
- **Image_Parser**: The component that extracts code from uploaded images
- **Practice_Problem_Generator**: The component that creates similar problems for practice
- **Learning_Plan_Generator**: The component that creates structured learning plans based on user goals
- **Extension**: The editor plugin version of Kod-Guru (Kiro or VS Code)
- **User**: A student or junior developer using the tool
- **Session**: A single learning interaction from code submission to completion
- **Hint_Stage**: The current level of hint specificity (1=vague, 2=specific, 3=solution)
- **Learning_Mode**: The type of analysis (debug=fix broken code, learn=understand working code)
- **Practice_Problem**: A generated coding problem similar to one the user has solved
- **Learning_Goal**: A user-defined objective for skill improvement
- **Learning_Plan**: A structured sequence of materials, problems, and tests to achieve a learning goal
- **Difficulty_Level**: A numeric rating (1-10) indicating problem complexity
- **Weekly_Review**: A scheduled session to revisit previously learned concepts
- **Retention_Score**: A metric (0-100) indicating how well a user has retained a concept

## Requirements

### Requirement 1: Code Ingestion

**User Story:** As a user, I want to submit code (broken or working) through multiple input methods, so that I can receive guided help in understanding or debugging it.

#### Acceptance Criteria

1. WHEN a user accesses the application, THE Code_Editor SHALL display an interface for pasting code
2. WHEN a user pastes code into the editor, THE Code_Editor SHALL preserve formatting and syntax highlighting
3. WHEN a user submits code, THE System SHALL accept code in multiple programming languages
4. WHERE the web application is used, WHEN a user uploads an image containing code, THE Image_Parser SHALL extract the code text from the image
5. WHEN a user provides error logs, THE System SHALL accept and store the error information alongside the code
6. WHEN a user submits code without error logs, THE System SHALL accept the submission and operate in learning mode
7. THE Code_Editor SHALL validate that submitted content is not empty before allowing submission

### Requirement 2: Socratic Analysis

**User Story:** As a user, I want the system to analyze my code without immediately revealing solutions, so that I can learn through guided discovery.

#### Acceptance Criteria

1. WHEN code is submitted, THE Analysis_Engine SHALL send it to AWS Bedrock Claude 3 Sonnet for analysis
2. WHEN analyzing code with error logs, THE Analysis_Engine SHALL identify the bug location and root cause
3. WHEN analyzing code without error logs, THE Analysis_Engine SHALL identify learning opportunities and code patterns
4. THE Analysis_Engine SHALL be configured with system prompts that strictly prohibit revealing solutions directly
5. WHEN analysis is complete, THE Analysis_Engine SHALL generate a structured analysis without exposing fixes or complete explanations
6. IF the Analysis_Engine fails to connect to AWS Bedrock, THEN THE System SHALL return an error message to the user

### Requirement 3: Progressive Hinting with Prerequisites

**User Story:** As a user, I want to receive increasingly specific hints with prerequisite concepts, so that I can understand the foundational knowledge needed and find solutions at my own pace.

#### Acceptance Criteria

1. WHEN a user requests the first hint, THE Hint_Generator SHALL provide a vague, conceptual hint about the topic or bug category
2. WHEN generating hints, THE Prerequisite_Analyzer SHALL identify foundational concepts required to understand the problem
3. WHEN prerequisite concepts are identified, THE Hint_Generator SHALL include them in the hint with brief explanations
4. WHEN a user requests a second hint, THE Hint_Generator SHALL provide a specific location reference (line number or code section) along with relevant prerequisites
5. WHEN a user requests a third hint or clicks "Give Up", THE Hint_Generator SHALL reveal the complete explanation or fix with all prerequisite concepts explained
6. THE Hint_Generator SHALL maintain hint stage state throughout a session
7. WHEN generating hints, THE Hint_Generator SHALL ensure each hint is more specific than the previous hint
8. THE System SHALL prevent users from skipping directly to Stage 3 without viewing Stage 1 and Stage 2 hints

### Requirement 4: Concept Card Generation

**User Story:** As a user, I want to receive an educational flashcard with prerequisites after completing a session, so that I can understand and remember the underlying concepts and their foundations.

#### Acceptance Criteria

1. WHEN a user completes a session or views the solution, THE Concept_Card_Generator SHALL create a flashcard explaining the main concept
2. THE Concept_Card_Generator SHALL include prerequisite concepts needed to understand the main concept
3. THE Concept_Card_Generator SHALL include the concept name, definition, prerequisites, and common examples in the flashcard
4. WHEN displaying a concept card, THE System SHALL present it in a visually distinct format
5. THE System SHALL allow users to save or export concept cards for future reference
6. WHEN generating concept cards, THE Concept_Card_Generator SHALL use AWS Bedrock to create educational content

### Requirement 5: Session Management

**User Story:** As a user, I want my debugging session to be tracked, so that the system can provide contextually appropriate hints.

#### Acceptance Criteria

1. WHEN a user submits code, THE System SHALL create a new session with a unique identifier
2. WHEN a session is active, THE System SHALL track the current hint stage
3. WHEN a user requests a hint, THE System SHALL retrieve the session state before generating the hint
4. WHEN a session is completed, THE System SHALL mark it as resolved
5. THE System SHALL maintain session state for at least 24 hours

### Requirement 6: Error Handling and Validation

**User Story:** As a user, I want clear feedback when something goes wrong, so that I can correct my input or understand system limitations.

#### Acceptance Criteria

1. WHEN the Analysis_Engine cannot identify a bug, THE System SHALL inform the user that no obvious issues were detected
2. IF AWS Bedrock API calls fail, THEN THE System SHALL retry up to 3 times before returning an error
3. WHEN code submission fails validation, THE System SHALL display specific error messages indicating what is missing
4. IF a session expires, THEN THE System SHALL prompt the user to resubmit their code
5. WHEN network errors occur, THE System SHALL display user-friendly error messages without exposing technical details

### Requirement 7: API Integration

**User Story:** As a developer, I want the backend to integrate with AWS services, so that the system can leverage cloud AI capabilities.

#### Acceptance Criteria

1. THE System SHALL authenticate with AWS Bedrock using secure credentials
2. WHEN making API calls to AWS Bedrock, THE System SHALL use Claude 3 Sonnet model
3. THE System SHALL implement rate limiting to prevent excessive API usage
4. WHEN deploying to AWS Lambda, THE System SHALL handle cold starts gracefully
5. THE System SHALL log all AWS API interactions for monitoring and debugging purposes

### Requirement 8: Frontend User Experience

**User Story:** As a user, I want an intuitive interface, so that I can focus on learning rather than navigating the tool.

#### Acceptance Criteria

1. WHEN the application loads, THE Code_Editor SHALL be immediately visible and ready for input
2. THE System SHALL provide clear visual indicators for hint stages (Stage 1, Stage 2, Stage 3)
3. WHEN a hint is displayed, THE System SHALL show it in a dedicated hint panel
4. THE Code_Editor SHALL support syntax highlighting for common programming languages
5. WHEN a concept card is generated, THE System SHALL display it in a modal or dedicated section
6. THE System SHALL provide a "New Session" button to start debugging a different problem

### Requirement 9: Backend API Design

**User Story:** As a frontend developer, I want well-defined API endpoints, so that I can integrate the UI with backend services.

#### Acceptance Criteria

1. THE System SHALL expose a POST endpoint for code submission that accepts code and error logs
2. THE System SHALL expose a GET endpoint for retrieving hints based on session ID and hint stage
3. THE System SHALL expose a POST endpoint for marking a session as complete
4. THE System SHALL expose a GET endpoint for retrieving concept cards by session ID
5. WHEN API requests are malformed, THE System SHALL return appropriate HTTP status codes and error messages
6. THE System SHALL implement CORS headers to allow frontend requests from authorized origins

### Requirement 10: Security and Privacy

**User Story:** As a user, I want my code to be handled securely, so that my work remains private.

#### Acceptance Criteria

1. THE System SHALL not store user code permanently after session expiration
2. THE System SHALL transmit all data over HTTPS
3. THE System SHALL sanitize user input to prevent injection attacks
4. THE System SHALL implement API authentication to prevent unauthorized access
5. WHEN storing session data, THE System SHALL use encryption for sensitive information

### Requirement 11: Editor Extensions

**User Story:** As a developer, I want to use Kod-Guru directly in my editor, so that I can learn without leaving my development environment.

#### Acceptance Criteria

1. THE System SHALL provide an extension for Kiro editor
2. THE System SHALL provide an extension for VS Code editor
3. WHEN a user selects code in the editor, THE Extension SHALL provide a command to analyze the selected code
4. WHEN the extension is activated, THE Extension SHALL send the selected code to the backend API
5. WHEN analysis results are received, THE Extension SHALL display hints in a dedicated panel within the editor
6. THE Extension SHALL support all hint stages (1, 2, 3) within the editor interface
7. THE Extension SHALL display concept cards within the editor when generated
8. THE Extension SHALL maintain session state during the editor session
9. WHEN the editor is closed, THE Extension SHALL preserve the session for 24 hours

### Requirement 12: Image Input Support

**User Story:** As a user of the web application, I want to upload images of code, so that I can analyze code from screenshots or photos.

#### Acceptance Criteria

1. WHERE the web application is used, THE System SHALL provide an image upload interface
2. WHEN a user uploads an image, THE Image_Parser SHALL accept common image formats (PNG, JPG, JPEG, WebP)
3. WHEN an image is uploaded, THE Image_Parser SHALL use OCR to extract text from the image
4. WHEN text extraction is complete, THE System SHALL populate the code editor with the extracted text
5. IF text extraction fails, THEN THE System SHALL inform the user that the image could not be processed
6. THE System SHALL validate that extracted text is not empty before allowing submission
7. WHEN processing images, THE Image_Parser SHALL use AWS Textract or similar OCR service

### Requirement 13: Learning Mode

**User Story:** As a user, I want to learn from any code (not just broken code), so that I can improve my understanding of programming concepts.

#### Acceptance Criteria

1. WHEN code is submitted without error logs, THE System SHALL operate in learning mode
2. WHEN in learning mode, THE Analysis_Engine SHALL identify interesting patterns, best practices, or improvement opportunities
3. WHEN in learning mode, THE Hint_Generator SHALL guide users to understand how the code works
4. WHEN in learning mode, THE Hint_Generator SHALL suggest potential improvements or alternative approaches
5. WHEN in learning mode, THE Concept_Card_Generator SHALL create cards explaining the concepts used in the code
6. THE System SHALL clearly indicate whether it is in debug mode or learning mode

### Requirement 14: Practice Problem Generation

**User Story:** As a user, I want to receive practice problems after solving a bug, so that I can reinforce my learning and build confidence.

#### Acceptance Criteria

1. WHEN a user completes a debugging session, THE System SHALL generate a similar practice problem at the same difficulty level
2. WHEN a user successfully solves a practice problem, THE System SHALL offer a slightly more difficult problem
3. THE System SHALL track the user's progress across practice problems
4. WHEN generating practice problems, THE System SHALL ensure they test the same concepts as the original problem
5. THE System SHALL allow users to request additional practice problems at any time
6. WHEN a user struggles with a problem, THE System SHALL offer problems at a lower difficulty level

### Requirement 15: Goal-Based Learning Plans

**User Story:** As a student, I want to set learning goals and receive a structured plan, so that I can systematically improve my programming skills.

#### Acceptance Criteria

1. WHEN a user enters learning mode, THE System SHALL prompt them to set a learning goal
2. WHEN a learning goal is set, THE System SHALL generate a structured learning plan with milestones
3. THE System SHALL recommend learning materials (articles, videos, documentation) relevant to the goal
4. WHEN generating learning plans, THE System SHALL include practice problems with progressive difficulty
5. WHERE the goal is DSA (Data Structures and Algorithms) learning, THE System SHALL include scheduled practice tests
6. THE System SHALL track progress toward the learning goal and update the plan accordingly
7. WHEN a user completes a milestone, THE System SHALL provide feedback and suggest the next steps
8. THE System SHALL allow users to adjust their learning goals and regenerate plans

### Requirement 16: Spaced Repetition and Weekly Reviews

**User Story:** As a user, I want to review previously learned concepts weekly, so that I retain knowledge over time.

#### Acceptance Criteria

1. THE System SHALL schedule weekly review sessions for completed concepts
2. WHEN a week has passed since learning a concept, THE System SHALL include it in the weekly review
3. WHEN a user starts a weekly review, THE System SHALL present problems covering previously learned concepts
4. THE System SHALL prioritize concepts that the user struggled with in previous reviews
5. WHEN a user completes a weekly review, THE System SHALL update the retention score for each concept
6. THE System SHALL send reminders for upcoming weekly reviews
7. THE System SHALL track which concepts need more review based on user performance

### Requirement 17: Responsive Design

**User Story:** As a user, I want to access Kod-Guru on any device (PC, tablet, mobile), so that I can learn programming wherever I am.

#### Acceptance Criteria

1. WHERE the web application is used, THE System SHALL provide a responsive layout that adapts to different screen sizes
2. WHEN accessed on desktop (≥1024px width), THE System SHALL display the code editor, hints, and concept cards in a multi-column layout
3. WHEN accessed on tablet (768px-1023px width), THE System SHALL adjust the layout to a two-column or stacked layout as appropriate
4. WHEN accessed on mobile (≤767px width), THE System SHALL display components in a single-column stacked layout
5. WHEN accessed on mobile, THE Code_Editor SHALL provide a mobile-optimized code input experience with appropriate keyboard support
6. WHEN accessed on mobile, THE System SHALL provide touch-friendly buttons and controls with minimum 44x44px touch targets
7. WHEN accessed on tablet or mobile, THE System SHALL allow users to toggle between code editor and hint panels to maximize screen space
8. THE System SHALL use CSS media queries and responsive design frameworks to ensure consistent experience across devices
9. WHEN accessed on any device, THE System SHALL maintain readability with appropriate font sizes and line spacing
10. WHEN accessed on mobile, THE Image_Parser SHALL allow users to capture code images directly using the device camera
