# SparkForge Implementation Prompts for Claude Code

## Phase 1: Project Setup & Foundation

### Prompt 1: Initialize Project Structure
```
Create a new Next.js 14 project called "sparkforge" with TypeScript, Tailwind CSS, and shadcn/ui. 

Set up the following directory structure:
- /app (Next.js app router)
- /components (reusable UI components)
- /lib (utilities, database, API helpers)
- /types (TypeScript interfaces)
- /hooks (custom React hooks)

Install these dependencies:
- @tanstack/react-query for data fetching
- zustand for state management
- lucide-react for icons
- recharts for data visualization
- framer-motion for animations

Create a dark purple gradient theme using Tailwind config that matches the SparkForge aesthetic.
```

### Prompt 2: Database Schema Design
```
Using Prisma ORM, create database schemas for:

1. Idea model:
   - id, title, description, rawContent (for voice/text input)
   - clusterId, clusterName, priorityScore, status (enum: new, prototype, validated, spec)
   - embeddings (for semantic search)
   - createdAt, updatedAt, userId

2. Prototype model:
   - id, ideaId (relation to Idea)
   - designVersion, artifactUrl, generatedCode
   - status (draft, testing, validated)
   - createdAt, updatedAt

3. ValidationSession model:
   - id, prototypeId
   - completionRate, avgTimeOnTask, satisfactionScore, totalSessions
   - createdAt, updatedAt

4. Feedback model:
   - id, validationSessionId, prototypeId
   - userId, userName, rating (positive/negative/neutral)
   - comment, timestamp

5. Specification model:
   - id, prototypeId, ideaId
   - userStories (JSON), acceptanceCriteria (JSON)
   - technicalRequirements (JSON), apiEndpoints (JSON)
   - status (draft, reviewed, exported)
   - exportedToKiroAt

Set up relationships between models. Include migration scripts.
```

---

## Phase 2: Backend API & Business Logic

### Prompt 3: Idea Capture API
```
Create Next.js API routes for idea management:

POST /api/ideas/create
- Accept: { title, description, source: 'text' | 'voice' | 'sketch' }
- Generate embeddings using OpenAI embeddings API
- Auto-cluster ideas using cosine similarity (group similar ideas)
- Calculate priority score based on: recency, cluster size, keyword frequency
- Return created idea with cluster assignment

GET /api/ideas/list
- Support filtering by: status, cluster, date range
- Include pagination
- Return with aggregated cluster stats

PATCH /api/ideas/:id/update
- Update idea properties
- Re-calculate clusters if content changed

DELETE /api/ideas/:id

GET /api/ideas/clusters
- Return all clusters with idea counts and priority distributions
```

### Prompt 4: AI-Powered Clustering System
```
Create a clustering service in /lib/clustering.ts that:

1. Generates embeddings for new ideas using OpenAI's text-embedding-3-small model
2. Calculates cosine similarity between all idea embeddings
3. Uses a threshold (0.7) to group similar ideas into clusters
4. Auto-generates cluster names using GPT-4 by analyzing idea titles in each cluster
5. Updates cluster assignments when new ideas are added
6. Provides a re-cluster function that can be manually triggered

Include functions:
- generateEmbedding(text: string): Promise<number[]>
- calculateCosineSimilarity(vecA: number[], vecB: number[]): number
- clusterIdeas(ideas: Idea[]): Promise<Cluster[]>
- generateClusterName(ideaTitles: string[]): Promise<string>
```

### Prompt 5: Prototype Generation API
```
Create prototype generation routes:

POST /api/prototypes/generate
- Accept: { ideaId, templateType: 'web-app' | 'mobile' | 'dashboard' | 'api' }
- Use Claude Sonnet API to generate React component code based on idea description
- Create a prompt that instructs Claude to generate:
  * Modern, interactive UI using Tailwind
  * Working state management
  * Sample data
  * Responsive design
- Save generated code and return prototype ID

GET /api/prototypes/:id
- Return prototype with generated code and metadata

POST /api/prototypes/:id/iterate
- Accept feedback and regenerate with improvements

The generation prompt should be:
"Generate a working React component for: [IDEA_TITLE]
Description: [IDEA_DESCRIPTION]
Create an interactive, production-ready prototype with:
- Modern UI using Tailwind CSS utility classes only
- Working state management with useState
- Sample realistic data
- Responsive design (mobile-first)
- Accessibility features
- Loading states and error handling
Return only the component code, no explanations."
```

---

## Phase 3: Frontend Components & UI

### Prompt 6: IdeaSpark Module Components
```
Create React components for the IdeaSpark module:

1. IdeaCaptureForm (/components/ideas/IdeaCaptureForm.tsx)
   - Text input for title and description
   - Voice input button (using Web Speech API)
   - Image upload for sketches
   - Real-time character count
   - Submit button with loading state

2. IdeaCard (/components/ideas/IdeaCard.tsx)
   - Display idea title, description, cluster tag, priority score
   - Status badge (new, prototype, validated, spec)
   - Action buttons: Generate Prototype, View, Delete
   - Expandable details section
   - Drag handle for reordering

3. ClusterGrid (/components/ideas/ClusterGrid.tsx)
   - Display cluster cards with names and idea counts
   - Visual priority distribution chart
   - Click to filter ideas by cluster
   - Animated transitions

4. IdeaList (/components/ideas/IdeaList.tsx)
   - Sortable, filterable list of all ideas
   - Search functionality
   - Bulk actions (delete, move to cluster)
   - Infinite scroll pagination

Use shadcn/ui components (Card, Button, Input, Badge) and Framer Motion for animations.
```

### Prompt 7: ProtoForge Module Components
```
Create components for prototype viewing and iteration:

1. PrototypeViewer (/components/prototypes/PrototypeViewer.tsx)
   - Tabs for: Design View, Interactive View, Code View
   - Design View: iframe rendering of generated component
   - Interactive View: live component with interaction tracking
   - Code View: syntax-highlighted code with copy button
   - Version history timeline

2. PrototypeEditor (/components/prototypes/PrototypeEditor.tsx)
   - Split pane: code editor on left, live preview on right
   - Monaco Editor for code editing
   - Real-time preview updates
   - Save and version buttons

3. IterationPanel (/components/prototypes/IterationPanel.tsx)
   - Text area for feedback/iteration instructions
   - "Regenerate" button that calls AI to improve prototype
   - History of iterations with diff view

4. SpecGeneratorButton (/components/prototypes/SpecGeneratorButton.tsx)
   - Prominent CTA button
   - Shows validation readiness (metrics threshold met)
   - Triggers spec generation flow
```

### Prompt 8: ValidationLoop Module Components
```
Create validation tracking and analytics components:

1. MetricsDashboard (/components/validation/MetricsDashboard.tsx)
   - Four metric cards: Completion Rate, Avg Time on Task, Satisfaction, Sessions
   - Use Recharts for trend visualization
   - Color coding (green for good, yellow for acceptable, red for poor)
   - Export metrics as CSV

2. FeedbackCollector (/components/validation/FeedbackCollector.tsx)
   - Embedded widget that can be injected into prototypes
   - Thumbs up/down buttons
   - Text comment field
   - Quick reactions (emoji-based)
   - Submit feedback via API

3. FeedbackList (/components/validation/FeedbackList.tsx)
   - Display all user feedback with sentiment badges
   - Filter by: positive/negative/neutral, date range
   - Sentiment analysis visualization
   - Export feedback as JSON

4. InsightsSummary (/components/validation/InsightsSummary.tsx)
   - AI-generated insights from feedback and metrics
   - Bullet points with checkmarks/warnings
   - Recommendations for improvements
   - Link insights to specific prototype elements
```

### Prompt 9: SpecWeaver Module Components
```
Create specification generation and export components:

1. SpecificationViewer (/components/specs/SpecificationViewer.tsx)
   - Sections: User Stories, Acceptance Criteria, Technical Requirements
   - Editable text fields for manual refinement
   - Preview mode and edit mode toggle
   - Print/PDF export functionality

2. UserStoryList (/components/specs/UserStoryList.tsx)
   - Display user stories in standard format
   - Add/edit/delete individual stories
   - Link stories to prototype features
   - Assign story points

3. TechnicalRequirements (/components/specs/TechnicalRequirements.tsx)
   - API endpoints table with methods and descriptions
   - Data model visualization
   - Component hierarchy tree
   - Dependencies list

4. KiroExporter (/components/specs/KiroExporter.tsx)
   - "Export to Kiro-ai" button
   - Format spec as JSON/Markdown for Kiro ingestion
   - Show export status and history
   - Webhook integration for automated handoff
```

---

## Phase 4: AI Integration & Advanced Features

### Prompt 10: Spec Generation Engine
```
Create an AI service that generates specifications from prototypes:

Location: /lib/ai/spec-generator.ts

Function: generateSpecification(prototypeId: string): Promise<Specification>

Process:
1. Fetch prototype code, idea description, validation metrics, and feedback
2. Create a detailed prompt for Claude that includes:
   - Original idea context
   - Generated prototype code
   - Validation metrics (completion rate, satisfaction, etc.)
   - User feedback summary
   - Request for: user stories, acceptance criteria, technical requirements, API endpoints, data models

3. Parse Claude's response into structured spec object
4. Save to database and return

Example prompt structure:
"You are a technical product manager. Generate a complete specification for this validated prototype.

IDEA: [title and description]
PROTOTYPE: [code]
VALIDATION RESULTS:
- Completion Rate: X%
- User Satisfaction: Y/5
- Key Feedback: [summary]

Generate:
1. User Stories (3-5 stories in format: As a [role], I want [feature] so that [benefit])
2. Acceptance Criteria (specific, testable requirements)
3. Technical Requirements (components, API endpoints, data models)
4. Success Metrics (based on validation data)

Return as JSON."
```

### Prompt 11: Real-Time Collaboration Features
```
Implement real-time collaboration using WebSockets or Pusher:

1. Create a collaboration service (/lib/collaboration.ts) that:
   - Tracks who's viewing which prototype
   - Shows live cursors when multiple users edit specs
   - Broadcasts prototype updates in real-time
   - Handles conflict resolution for concurrent edits

2. Add presence indicators to UI:
   - Show avatars of active collaborators
   - Display "X is viewing this prototype" notifications
   - Lock editing when someone else is actively editing

3. Implement activity feed:
   - "Sarah generated a prototype for Idea #15"
   - "James added feedback to Dark Mode Toggle"
   - "Priya exported spec to Kiro-ai"
```

### Prompt 12: Analytics & Insights Dashboard
```
Create an analytics overview page (/app/analytics/page.tsx) that shows:

1. Innovation Funnel Visualization:
   - Ideas captured → Prototypes generated → Validated → Specs created
   - Conversion rates at each stage
   - Time spent at each stage

2. Team Performance Metrics:
   - Ideas per user per week
   - Average time from idea to prototype
   - Validation success rate
   - Most active clusters

3. Trend Analysis:
   - Idea velocity over time (line chart)
   - Popular clusters (bar chart)
   - User satisfaction trends (area chart)

4. AI Usage Statistics:
   - Number of prototypes generated
   - Number of specs auto-generated
   - AI iteration requests
   - Token usage tracking

Use Recharts for all visualizations. Make it responsive and exportable.
```

---

## Phase 5: Integration & Polish

### Prompt 13: Kiro-ai Integration API
```
Create integration endpoints for seamless handoff to Kiro-ai:

POST /api/integrations/kiro/export
- Accept specificationId
- Format spec as Kiro-compatible JSON:
  {
    projectName: string,
    description: string,
    userStories: Story[],
    technicalSpec: {
      components: Component[],
      apis: Endpoint[],
      dataModels: Model[]
    },
    validationData: {
      metrics: Metrics,
      feedback: Feedback[]
    }
  }
- Send to Kiro-ai webhook URL
- Track export status and create audit log

POST /api/integrations/kiro/webhook
- Receive status updates from Kiro-ai
- Update spec status in SparkForge
- Notify relevant team members

GET /api/integrations/kiro/status/:specId
- Check current status of exported spec in Kiro-ai
- Return build progress, code generation status, etc.
```

### Prompt 14: Search & Discovery Features
```
Implement advanced search and discovery:

1. Global Search (/components/search/GlobalSearch.tsx):
   - Search across ideas, prototypes, specs, and feedback
   - Semantic search using embeddings
   - Filters: date range, status, creator, cluster
   - Keyboard shortcuts (Cmd+K to open)

2. Smart Recommendations (/lib/recommendations.ts):
   - "Ideas similar to this one" using embedding similarity
   - "Prototypes that used similar patterns" 
   - "Users who worked on this also worked on..."

3. Saved Views & Filters:
   - Allow users to save custom filters/views
   - Quick access to "My Ideas", "High Priority", "Needs Validation"
   - Share views with team members
```

### Prompt 15: Authentication & Permissions
```
Implement authentication and role-based access control:

1. Use NextAuth.js with these providers:
   - Email/password
   - Google OAuth
   - GitHub OAuth

2. User roles:
   - Viewer: Can view ideas and prototypes
   - Contributor: Can create ideas and provide feedback
   - Creator: Can generate prototypes and specs
   - Admin: Full access including analytics and settings

3. Permissions middleware (/lib/auth/permissions.ts):
   - Check permissions before API operations
   - Row-level security in database
   - Team/workspace isolation

4. User profile page with:
   - Activity history
   - Ideas created
   - Prototypes generated
   - Specs exported
   - Settings and preferences
```

### Prompt 16: Onboarding & Help System
```
Create a comprehensive onboarding experience:

1. Welcome Tour (/components/onboarding/WelcomeTour.tsx):
   - Interactive walkthrough using react-joyride
   - 5-step tour: Capture Idea → Generate Prototype → Collect Feedback → View Metrics → Export Spec
   - Skip button and progress indicator
   - Save completion status

2. Empty States:
   - Design informative empty states for each module
   - Include "Get Started" buttons with helpful tips
   - Show example ideas/prototypes on first visit

3. Contextual Help:
   - Help icons with tooltips throughout UI
   - Link to documentation for complex features
   - Video tutorials embedded in relevant sections

4. Template Library (/app/templates/page.tsx):
   - Pre-built idea templates for common use cases
   - Sample prototypes users can clone
   - Best practices guide
```

---

## Phase 6: Testing & Deployment

### Prompt 17: Comprehensive Testing Setup
```
Set up testing infrastructure:

1. Unit tests using Jest and React Testing Library:
   - Test all utility functions in /lib
   - Test React hooks in /hooks
   - Test API route handlers

2. Integration tests:
   - Test complete flows: idea → prototype → spec
   - Test AI integration with mocked responses
   - Test database operations

3. E2E tests using Playwright:
   - User journey: Create account → Add idea → Generate prototype → Export spec
   - Test all critical user paths
   - Visual regression testing

4. Create test data seeds:
   - Sample ideas across different clusters
   - Mock validation data
   - Test user accounts with different roles

Write tests for the IdeaCaptureForm component, clustering algorithm, and spec generation service.
```

### Prompt 18: Performance Optimization
```
Optimize SparkForge for production performance:

1. Implement code splitting:
   - Lazy load heavy components (Monaco Editor, Recharts)
   - Route-based code splitting
   - Component-level dynamic imports

2. Database optimization:
   - Add indexes on frequently queried fields (userId, status, clusterId)
   - Implement database connection pooling
   - Use database query caching for analytics

3. Caching strategy:
   - Cache embeddings to avoid re-generation
   - Cache cluster calculations (invalidate on new ideas)
   - Use React Query for client-side caching
   - Implement Redis for server-side caching

4. Asset optimization:
   - Image optimization with next/image
   - Font optimization
   - Bundle size analysis and reduction

5. Add loading states and skeleton screens everywhere to improve perceived performance.
```

### Prompt 19: Deployment Configuration
```
Set up production deployment on Vercel:

1. Create environment configuration:
   - Development, staging, and production environments
   - Environment variables for:
     * Database URLs
     * API keys (OpenAI, Claude)
     * Authentication secrets
     * Kiro-ai integration endpoints

2. CI/CD pipeline:
   - GitHub Actions workflow for:
     * Running tests on PR
     * Linting and type checking
     * Automated deployments to staging
     * Production deployment on main branch merge

3. Database migrations:
   - Prisma migration strategy for production
   - Backup procedures before migrations
   - Rollback plan

4. Monitoring and observability:
   - Set up error tracking with Sentry
   - Analytics with Vercel Analytics
   - Log aggregation with LogDrain
   - Uptime monitoring with Better Uptime

5. Create deployment checklist document.
```

### Prompt 20: Documentation & Launch Prep
```
Create comprehensive documentation:

1. Developer Documentation (/docs/dev/):
   - Architecture overview with diagrams
   - API documentation (all endpoints with examples)
   - Database schema documentation
   - Deployment guide
   - Contributing guide

2. User Documentation (/docs/user/):
   - Getting started guide
   - Feature guides for each module
   - Best practices for idea capture
   - Tips for writing effective prototypes
   - FAQ section

3. Video tutorials:
   - Script for 3-minute product demo video
   - Individual feature tutorials
   - Advanced workflows tutorial

4. Launch materials:
   - Product Hunt launch copy
   - Blog post announcing SparkForge
   - Social media announcement templates
   - Email to existing SpecConnex users

5. Create a changelog system for tracking updates and improvements.
```

---

## Bonus: Advanced Features for Future Iterations

### Prompt 21: Voice-to-Idea Feature
```
Implement advanced voice capture:

1. Use Web Speech API for voice recording
2. Transcribe with OpenAI Whisper API
3. Auto-extract key entities (features, users, benefits) from transcription
4. Generate structured idea with AI-suggested title and description
5. Support multiple languages
6. Add speaker diarization for multi-person brainstorms
```

### Prompt 22: AI Prototype Critic
```
Create an AI feature that analyzes prototypes and suggests improvements:

1. Analyze generated code for:
   - Accessibility issues
   - Performance bottlenecks
   - Security vulnerabilities
   - UX best practices violations

2. Compare against similar validated prototypes
3. Suggest specific code improvements with examples
4. Calculate "prototype quality score"
5. Auto-apply safe improvements with user approval
```

### Prompt 23: Prototype A/B Testing Framework
```
Build A/B testing capability:

1. Generate multiple prototype variants from one idea
2. Split traffic between variants
3. Track metrics separately for each variant
4. Statistical significance calculator
5. Declare winner automatically when significance reached
6. Auto-generate spec from winning variant
```

---

## Usage Instructions for Claude Code

To implement SparkForge using these prompts with Claude Code:

1. **Sequential Execution**: Start with Prompt 1 and work through in order
2. **Context Building**: Reference previous prompts when needed (e.g., "using the database schema from Prompt 2...")
3. **Iterative Refinement**: After each implementation, ask Claude Code to review and suggest improvements
4. **Testing as You Go**: After completing phases 1-3, use Prompt 17 to add tests
5. **Integration Points**: When you reach integration prompts, ensure all dependencies are properly connected

**Example Usage:**
```
Me: [Paste Prompt 1]
Claude Code: [Implements project structure]
Me: Now implement Prompt 2, ensuring the Prisma schema integrates with the Next.js setup from Prompt 1
Claude Code: [Implements database schema]
Me: Review the schema and suggest any optimizations for the SparkForge use case
Claude Code: [Provides review and suggestions]
```

Each prompt is designed to be self-contained but builds on previous work. Adjust based on your specific tech stack preferences.
