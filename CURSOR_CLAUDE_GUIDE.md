# Building FitClip with Claude in Cursor

**IDE:** Cursor  
**AI Model:** Claude (Sonnet 4.5 recommended)  
**Project:** FitClip - Golf Fitting Video Recap Service  
**Last Updated:** November 28, 2025

---

## 🎯 OVERVIEW

This guide shows you how to use Claude inside Cursor IDE to build FitClip from scratch. You'll leverage Claude's ability to:
- Generate entire features from specifications
- Write tests automatically
- Debug issues in real-time
- Refactor code for best practices
- Implement complex algorithms

---

## 📋 SETUP CHECKLIST

### 1. Install Cursor (if not already installed)

```bash
# Download from https://cursor.sh
# Or via Homebrew (macOS)
brew install --cask cursor

# Launch Cursor
open -a Cursor
```

### 2. Configure Cursor for FitClip

**Open Cursor Settings:**
- Press `Cmd+,` (Mac) or `Ctrl+,` (Windows/Linux)
- Go to "Cursor Settings"

**Set Claude as your model:**
```
AI Model: Claude (Sonnet 4.5)
Context Window: Long (preferred for this project)
Auto-suggestions: On
```

### 3. Open FitClip Project in Cursor

```bash
# Navigate to project
cd /Users/davidegan/fitclip

# Open in Cursor
cursor .
```

### 4. Add Documentation to Cursor's Context

**Method 1: .cursorrules file (Recommended)**

Create `.cursorrules` in project root:

```bash
cat > /Users/davidegan/fitclip/.cursorrules << 'EOF'
# FitClip Project Rules for Claude

## Project Context
You are building FitClip, a SaaS platform that turns golf club fittings into personalized video recaps.

## Key Documentation (always reference):
- README.md - Project overview and index
- TECHNICAL_ARCHITECTURE.md - System architecture
- API_REFERENCE.md - API contracts
- FEATURE_SPECS.md - Feature requirements
- PRD.md - Product requirements
- DEVELOPMENT_SETUP.md - Local setup guide

## Technology Stack:
- Backend: Node.js 20 + Express + TypeScript
- Frontend: React 18 + TypeScript + Material-UI + Vite
- Video Service: Python 3.11 + FastAPI + FFmpeg
- Database: PostgreSQL 15 (via Prisma ORM)
- Cache: Redis 7.2
- Infrastructure: AWS (ECS, RDS, S3, CloudFront)

## Code Standards:
- TypeScript: Use strict mode, ESLint + Prettier
- Python: Use Black formatter, Pylint, type hints
- Tests: Jest (backend), pytest (video service), >80% coverage
- Commits: Conventional Commits format (feat:, fix:, docs:, etc.)
- Error Handling: Always use try-catch, return structured errors
- Logging: Use structured JSON logging (pino for Node.js)

## Architecture Principles:
- API-first: All features exposed via REST APIs
- Stateless services: Horizontal scaling
- Event-driven: Async processing for video generation
- Security: JWT auth, encrypted data, input validation (Zod)

## Database:
- ORM: Prisma (generates type-safe client)
- Migrations: Always create migration before schema changes
- Indexes: Create for all foreign keys and query patterns

## When implementing features:
1. Read the feature spec in FEATURE_SPECS.md first
2. Check API_REFERENCE.md for endpoint contracts
3. Follow acceptance criteria exactly
4. Write tests before or alongside implementation
5. Update API docs if endpoints change

## Common Patterns:

### API Endpoint (Express + TypeScript):
```typescript
import { Router } from 'express';
import { z } from 'zod';
import { authenticate } from '../middleware/auth';

const router = Router();

const schema = z.object({
  field: z.string().min(1)
});

router.post('/', authenticate, async (req, res) => {
  try {
    const data = schema.parse(req.body);
    const result = await service.create(data);
    res.status(201).json(result);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

### Service Layer (Business Logic):
```typescript
export class FittingService {
  constructor(private prisma: PrismaClient) {}
  
  async create(data: CreateFittingDto) {
    return await this.prisma.fitting.create({
      data: {
        ...data,
        status: 'in_progress',
        created_at: new Date()
      }
    });
  }
}
```

### Celery Task (Python):
```python
from celery import Celery

celery = Celery('app', broker='redis://localhost:6379/0')

@celery.task
def generate_video(fitting_id: str):
    """Generate video for completed fitting."""
    # Implementation here
    return {'status': 'success'}
```

## Priority Features (build in this order):
1. TrackMan Integration (FEATURE_SPECS.md page 1-15)
2. AI Highlight Selection (FEATURE_SPECS.md page 15-25)
3. Video Generation (FEATURE_SPECS.md page 25+)
4. Multi-Channel Delivery
5. Shop Dashboard

## Testing:
- Always write tests for new features
- Run `npm test` (backend) or `pytest` (video service) before committing
- Integration tests for critical flows (fitting creation → video generation)

## Questions?
- Refer to documentation first
- Check DEVELOPMENT_SETUP.md for troubleshooting
- API contracts are in API_REFERENCE.md
EOF
```

**Method 2: Cursor's "Add to Context" feature**

In Cursor, press `Cmd+K` and type:
```
@README.md @TECHNICAL_ARCHITECTURE.md @API_REFERENCE.md @FEATURE_SPECS.md
```

This adds these files to Claude's context for the conversation.

---

## 🚀 WORKFLOW: BUILDING WITH CLAUDE IN CURSOR

### Phase 1: Project Setup (Day 1)

**1. Initialize Backend Project**

Press `Cmd+K` in Cursor and prompt Claude:

```
Create the FitClip backend project structure:

Context: @TECHNICAL_ARCHITECTURE.md @DEVELOPMENT_SETUP.md

1. Initialize Node.js 20 + TypeScript project in /backend
2. Install dependencies: express, prisma, zod, bcrypt, jsonwebtoken, socket.io
3. Set up folder structure:
   /src
     /routes      - API endpoints
     /services    - Business logic
     /middleware  - Auth, validation, error handling
     /lib         - Utilities (prisma client, logger)
     /types       - TypeScript types
   /tests
     /unit
     /integration
   /prisma
     schema.prisma

4. Configure TypeScript (tsconfig.json) with strict mode
5. Configure ESLint + Prettier
6. Create package.json scripts: dev, build, test, lint

Please generate all configuration files and basic project structure.
```

Claude will generate all the files. Review and accept.

**2. Set Up Database Schema**

Press `Cmd+K` and prompt:

```
Create the Prisma schema for FitClip:

Context: @TECHNICAL_ARCHITECTURE.md (Section 5.1 - Database Schema)

Generate prisma/schema.prisma with these tables:
- shops
- users  
- customers
- fittings
- shots
- videos
- trackman_connections
- analytics_events

Include all fields, relationships, and indexes as specified in the architecture doc.
Use UUID for IDs, timestamps for dates.
```

**3. Set Up Docker Compose**

Press `Cmd+K`:

```
Create docker-compose.yml for local development:

Context: @DEVELOPMENT_SETUP.md (Docker Compose section)

Services needed:
- PostgreSQL 15.5
- Redis 7.2
- LocalStack (S3 emulator)
- Mailhog (email testing)

Include health checks, volumes, and environment variables.
```

**4. Initialize Frontend**

Press `Cmd+K`:

```
Create the FitClip frontend project:

1. Initialize Vite + React 18 + TypeScript in /frontend
2. Install dependencies: @mui/material, @reduxjs/toolkit, react-router-dom, socket.io-client
3. Set up folder structure:
   /src
     /components  - Reusable UI components
     /pages       - Page components
     /features    - Redux slices (fittings, shops, auth)
     /api         - API client with RTK Query
     /types       - TypeScript interfaces
     /theme       - MUI theme config

4. Configure Vite (vite.config.ts)
5. Set up MUI theme with FitClip colors (green, navy, orange)
```

**5. Initialize Video Service**

Press `Cmd+K`:

```
Create the FitClip video service:

1. Initialize Python 3.11 project in /video-service
2. Create requirements.txt with: fastapi, celery, redis, ffmpeg-python, xgboost, boto3
3. Set up folder structure:
   /app
     tasks.py          - Celery tasks
     video_generator.py - Video rendering
     highlight_selector.py - ML highlight selection
   /models             - ML models
   /tests

4. Create Celery configuration
5. Create FastAPI app for health checks
```

**Verify Setup:**

```bash
# Start services
docker compose up -d

# Terminal 1: Backend
cd backend && npm install && npm run dev

# Terminal 2: Video service
cd video-service && pip install -r requirements.txt
celery -A app.celery worker --loglevel=info

# Terminal 3: Frontend
cd frontend && npm install && npm run dev
```

---

### Phase 2: Build Feature 1 - TrackMan Integration (Days 2-4)

**Step 1: Create Database Models**

Press `Cmd+K`:

```
Implement TrackMan integration database models:

Context: @FEATURE_SPECS.md (Feature 1, Story 1.1-1.4)

1. Update prisma/schema.prisma if not already done:
   - trackman_connections table
   - Add bay_id to fittings table
   - Add trackman_shot_id to shots table

2. Generate Prisma Client: npx prisma generate
3. Create migration: npx prisma migrate dev --name add_trackman
```

**Step 2: Build TrackMan API Client**

Press `Cmd+K`:

```
Create TrackMan API client:

Context: @FEATURE_SPECS.md (Feature 1, Function 1-3)
Context: @TECHNICAL_ARCHITECTURE.md (Section 6.1)

Create src/lib/trackman-client.ts with:

1. TrackManClient class with methods:
   - authenticate(username, apiKey)
   - getShotsSince(timestamp)
   - refreshToken()

2. Implement OAuth 2.0 flow
3. Handle rate limiting (poll every 2 seconds)
4. Retry logic with exponential backoff
5. Error handling for all failure modes

Use axios for HTTP requests.
Include TypeScript types for all responses.
```

**Step 3: Create API Endpoints**

Press `Cmd+K`:

```
Implement TrackMan API endpoints:

Context: @API_REFERENCE.md (TrackMan endpoints)
Context: @FEATURE_SPECS.md (Feature 1, Backend Requirements)

Create src/routes/trackman.ts with these endpoints:

POST   /v1/trackman/connect
GET    /v1/trackman/:id/status  
DELETE /v1/trackman/:id

Also create:
POST   /v1/fittings/:id/shots

Follow the exact API contracts in API_REFERENCE.md.
Include input validation (Zod schemas).
Add authentication middleware.
Implement error handling.
```

**Step 4: Build Polling Worker**

Press `Cmd+K`:

```
Create TrackMan polling background worker:

Context: @FEATURE_SPECS.md (Feature 1, Background Jobs)

Create src/workers/trackman-poller.ts:

1. Poll TrackMan API every 2 seconds for active fittings
2. Store shots in database via Prisma
3. Emit WebSocket event to fitter's device
4. Handle network failures (queue shots locally)
5. Use BullMQ for job queue management

Include proper error handling and logging.
```

**Step 5: Build WebSocket for Real-Time Updates**

Press `Cmd+K`:

```
Implement WebSocket for real-time shot updates:

Context: @FEATURE_SPECS.md (Feature 1, Real-time updates)

1. Set up Socket.IO in src/server.ts
2. Create rooms per fitting (fitting:{fitting_id})
3. Emit "new_shot" event when shot is captured
4. Handle authentication for WebSocket connections
5. Implement heartbeat/reconnection logic
```

**Step 6: Build Frontend (Fitter iPad App)**

Press `Cmd+K`:

```
Create fitter iPad app for live fitting session:

Context: @FEATURE_SPECS.md (Feature 1, UI Specifications)

Create in /frontend/src/pages:

1. FittingSession.tsx component with:
   - Start/stop fitting controls
   - Live shot feed (WebSocket updates)
   - Shot list with details
   - Complete fitting button

2. Redux slice for fitting state (src/features/fittings/fittingSlice.ts)
3. WebSocket hook (src/hooks/useWebSocket.ts)
4. API calls with RTK Query

Use Material-UI components.
Make it responsive for iPad (1024×768).
```

**Step 7: Write Tests**

Press `Cmd+K`:

```
Write tests for TrackMan integration:

Context: @FEATURE_SPECS.md (Feature 1, Testing Requirements)

Create tests in /backend/tests:

1. Unit tests:
   - trackman-client.test.ts (API client)
   - trackman.service.test.ts (business logic)

2. Integration test:
   - trackman-integration.test.ts (full flow: connect → poll → store → emit)

Use Jest + Supertest.
Mock TrackMan API responses.
Target 80%+ coverage.
```

**Step 8: Test Manually**

In Cursor terminal:

```bash
# Start all services
docker compose up -d
npm run dev  # backend (terminal 1)
npm run dev  # frontend (terminal 2)

# Test API
curl -X POST http://localhost:3000/v1/trackman/connect \
  -H "Content-Type: application/json" \
  -d '{"shop_id":"...","username":"test@trackman.com","api_key":"sandbox_..."}'

# Create fitting and add shots
curl -X POST http://localhost:3000/v1/fittings \
  -H "Authorization: Bearer <token>" \
  -d '{"shop_id":"...","customer":{...}}'

# Watch WebSocket updates in browser console
```

---

### Phase 3: Build Feature 2 - AI Highlight Selection (Days 5-6)

**Step 1: Implement Scoring Algorithm**

Press `Cmd+K`:

```
Create shot quality scoring algorithm:

Context: @FEATURE_SPECS.md (Feature 2, Function 1)

Create src/services/highlight-selector.service.ts:

Implement calculateShotQualityScore(shot):
- Distance score: 40% weight
- Accuracy score: 30% weight (based on offline distance)
- Contact score: 20% weight (smash factor)
- Spin score: 10% weight (optimal spin range)

Return score 0-100.

Include TypeScript types for all parameters.
Add detailed comments explaining the math.
```

**Step 2: Implement Selection Algorithm**

Press `Cmd+K`:

```
Create highlight selection algorithm:

Context: @FEATURE_SPECS.md (Feature 2, Function 2)

In highlight-selector.service.ts, implement selectHighlights(shots, num=7):

1. Select longest shot
2. Select straightest shot
3. Select best overall (highest score)
4. Select biggest improvement from baseline
5. Fill remaining slots with high-scoring shots (ensure variety)

Ensure no more than 3 shots per club type.
Return array of 7 shot IDs sorted chronologically.
```

**Step 3: Implement Before/After Logic**

Press `Cmd+K`:

```
Add before/after comparison logic:

Context: @FEATURE_SPECS.md (Feature 2, Function 3)

In highlight-selector.service.ts, implement identifyBeforeAfterPair(shots):

1. Find best baseline shot (customer's old clubs)
2. Find best test shot (new clubs)
3. Ensure meaningful difference (>10 yards or >5%)
4. Return pair with improvement stats

Handle edge case: no baseline shots available.
```

**Step 4: Create API Endpoint**

Press `Cmd+K`:

```
Create API endpoint for highlight selection:

POST /v1/fittings/:id/highlights

This runs the ML selection algorithm on demand (for testing).
In production, it's called automatically when fitting completes.

Add to src/routes/fittings.ts.
Follow API_REFERENCE.md format.
```

**Step 5: Write Tests**

Press `Cmd+K`:

```
Write tests for highlight selection:

Context: @FEATURE_SPECS.md (Feature 2, Testing)

Create tests/unit/highlight-selector.test.ts:

1. Test scoring algorithm with various shot data
2. Test selection algorithm (ensures 7 shots, variety, quality)
3. Test before/after logic (with and without baseline)
4. Test edge cases (all bad shots, <7 valid shots)

Include test fixtures with sample shot data.
```

---

### Phase 4: Build Feature 3 - Video Generation (Days 7-10)

**Step 1: Set Up Python Video Service**

Press `Cmd+K`:

```
Implement video generation service:

Context: @FEATURE_SPECS.md (Feature 3, Video Generation Pipeline)
Context: @TECHNICAL_ARCHITECTURE.md (Section 3.3)

Create video-service/app/video_generator.py:

Class: VideoGenerator with methods:
1. fetch_fitting_data(fitting_id) - Get data from PostgreSQL
2. select_highlights(shots) - Call highlight selector
3. render_video(fitting_data, highlights) - FFmpeg pipeline
4. upload_to_s3(video_path) - Upload to S3
5. send_notifications(fitting_id) - SMS + Email

Use FFmpeg for video rendering:
- Intro (3s): Shop logo + text
- Highlight clips (7 × 15s): Swing + data overlay
- Before/after (30s): Split screen comparison
- Outro (5s): Shop contact info

Output: 1080p, 60fps, H.264, ~40 MB file.
```

**Step 2: Create FFmpeg Video Templates**

Press `Cmd+K`:

```
Create FFmpeg rendering functions:

In video-service/app/ffmpeg_renderer.py:

1. render_intro(shop_logo, text) → intro.mp4
2. render_shot_clip(shot_data, overlay_data) → shot_1.mp4
3. render_before_after(baseline, test_shot) → comparison.mp4
4. render_outro(shop_info) → outro.mp4
5. concatenate_clips(clips[]) → final_video.mp4

Use ffmpeg-python library.
Include progress tracking for each step.
```

**Step 3: Create Celery Task**

Press `Cmd+K`:

```
Create Celery task for video generation:

Context: @FEATURE_SPECS.md (Feature 3, Step 5)

In video-service/app/tasks.py:

@celery.task
def generate_video(fitting_id: str):
    """
    Complete video generation workflow:
    1. Fetch data
    2. Select highlights
    3. Render video (15-30 min)
    4. Upload to S3
    5. Update database
    6. Send notifications
    """

Include error handling (retry on failure).
Log progress at each step.
Update video status in database (pending → rendering → ready → failed).
```

**Step 4: Integrate with Backend**

Press `Cmd+K`:

```
Trigger video generation when fitting completes:

In backend/src/routes/fittings.ts:

PATCH /v1/fittings/:id/complete handler:
1. Update fitting status to "completed"
2. Enqueue video job in Redis:
   videoQueue.add('generate', { fitting_id })
3. Return job_id to client

The Celery worker will pick up the job and start video generation.
```

**Step 5: Implement Notifications**

Press `Cmd+K`:

```
Implement SMS and email notifications:

Context: @FEATURE_SPECS.md (Feature 3, Step 5)

Create backend/src/services/notification.service.ts:

1. sendSMS(phone, message) - Twilio integration
2. sendEmail(email, template, data) - SendGrid integration

Call after video is ready:
- SMS: "Tom, your FitClip video is ready! Watch now: [link]"
- Email: Use HTML template with video thumbnail, "Watch Now" CTA

Include unsubscribe links (CAN-SPAM compliance).
```

**Step 6: Write Tests**

Press `Cmd+K`:

```
Write integration test for video generation:

Context: @FEATURE_SPECS.md (Feature 3, Testing)

Create tests/integration/video-generation.test.ts:

Test full flow:
1. Create fitting with 10 shots
2. Complete fitting
3. Wait for video generation (up to 60 seconds)
4. Verify video exists in S3
5. Verify customer received SMS + email
6. Verify video plays correctly

Mock S3, Twilio, SendGrid for testing.
```

---

### Phase 5: Build Shop Dashboard (Days 11-12)

**Step 1: Authentication System**

Press `Cmd+K`:

```
Implement JWT authentication:

Context: @API_REFERENCE.md (Auth endpoints)

1. Create backend/src/routes/auth.ts with:
   - POST /v1/auth/login
   - POST /v1/auth/refresh
   - POST /v1/auth/logout

2. Create backend/src/middleware/auth.ts:
   - authenticate() middleware (verify JWT)
   - authorize(roles) middleware (check permissions)

3. Use bcrypt for password hashing
4. Store refresh tokens in Redis (30-day TTL)
5. Access tokens expire in 15 minutes
```

**Step 2: Create Login Page**

Press `Cmd+K`:

```
Create login page for shop owners:

In frontend/src/pages/Login.tsx:

1. Material-UI form with email + password
2. Call POST /v1/auth/login on submit
3. Store access token in Redux state
4. Store refresh token in httpOnly cookie
5. Redirect to /dashboard on success
6. Show error messages (invalid credentials, etc.)

Use Formik + Yup for form validation.
```

**Step 3: Create Dashboard Pages**

Press `Cmd+K`:

```
Create shop dashboard pages:

Context: @FEATURE_SPECS.md (Feature 5, Shop Dashboard)

In frontend/src/pages:

1. Dashboard.tsx - Metrics overview (conversion rate, videos sent)
2. Fittings.tsx - List of all fittings (table with filters)
3. FittingDetail.tsx - Individual fitting (shots + video player)
4. Analytics.tsx - Charts and graphs
5. Settings.tsx - Shop config (branding, TrackMan connection)

Use Material-UI components (DataGrid, Card, Chart).
Fetch data via RTK Query.
```

**Step 4: Build Analytics Dashboard**

Press `Cmd+K`:

```
Create analytics dashboard with charts:

Context: @API_REFERENCE.md (Analytics endpoints)

In frontend/src/pages/Analytics.tsx:

1. Conversion rate chart (line chart, last 30 days)
2. Videos sent/watched (bar chart)
3. Fitter performance table (conversion rate per fitter)
4. Revenue impact (total sales attributed to FitClip)

Use Chart.js or Recharts for visualizations.
Fetch data from GET /v1/shops/:id/analytics.
```

**Step 5: Implement Real-Time Updates**

Press `Cmd+K`:

```
Add real-time updates to dashboard:

In frontend/src/hooks/useWebSocket.ts:

1. Connect to WebSocket on dashboard load
2. Listen for events:
   - "new_fitting" → Update fittings list
   - "new_shot" → Update live shot count
   - "video_ready" → Show notification

3. Reconnect on disconnect
4. Show connection status indicator

Use Socket.IO client.
```

---

### Phase 6: Testing & Refinement (Days 13-15)

**Run Full Test Suite**

Press `Cmd+K`:

```
Create comprehensive test suite:

1. Unit tests for all services (80%+ coverage)
2. Integration tests for critical flows:
   - Fitting creation → shots → completion → video → delivery
   - TrackMan integration (mocked)
   - Authentication flow
3. E2E tests with Playwright:
   - Shop owner login
   - Create fitting
   - View video

Run: npm test (backend), pytest (video service), npm run test:e2e (frontend)
```

**Performance Testing**

In Cursor terminal:

```bash
# Load test API with 100 concurrent requests
npm install -g autocannon
autocannon -c 100 -d 30 http://localhost:3000/v1/fittings

# Monitor video generation time
# Target: <30 minutes (p99)
```

**Fix Issues**

Press `Cmd+K` for any bugs:

```
Debug this error: [paste error message]

Context: @DEVELOPMENT_SETUP.md (Troubleshooting section)

Analyze the issue and provide a fix.
```

---

## 💡 CURSOR + CLAUDE BEST PRACTICES

### 1. Use Precise Prompts

**❌ Bad Prompt:**
```
Create the API
```

**✅ Good Prompt:**
```
Create the Fitting API endpoints following the specification in @API_REFERENCE.md:

- POST /v1/fittings (create fitting)
- GET /v1/fittings (list fittings)  
- GET /v1/fittings/:id (get fitting details)
- PATCH /v1/fittings/:id/complete (complete fitting)

Include input validation with Zod, authentication middleware, error handling, and TypeScript types.

Follow the exact request/response formats in API_REFERENCE.md.
```

### 2. Reference Documentation

Always use `@filename` to add context:

```
@FEATURE_SPECS.md - For detailed requirements
@API_REFERENCE.md - For API contracts
@TECHNICAL_ARCHITECTURE.md - For system design
@DEVELOPMENT_SETUP.md - For setup/troubleshooting
```

### 3. Iterate on Generated Code

If Claude's first attempt isn't perfect:

```
The code you generated works, but I need these improvements:

1. Add error handling for network failures
2. Add retry logic with exponential backoff
3. Add TypeScript types for all parameters
4. Add unit tests

Please update the code.
```

### 4. Ask Claude to Explain

If you don't understand generated code:

```
Explain how the highlight selection algorithm works:

1. What is the scoring formula?
2. Why these specific weights (40%, 30%, 20%, 10%)?
3. How does it ensure variety (not all driver shots)?
4. What are the edge cases handled?
```

### 5. Use Cursor's Multi-File Editing

Select multiple files in sidebar, then `Cmd+K`:

```
Update these files to implement TrackMan integration:

Selected:
- src/routes/trackman.ts
- src/services/trackman.service.ts
- src/lib/trackman-client.ts

Make sure they work together following @FEATURE_SPECS.md.
```

### 6. Use Cursor's Terminal

Run commands directly in Cursor terminal:

```bash
# Claude can see terminal output and debug errors
npm test
# If tests fail, press Cmd+K and say:
# "These tests are failing: [paste output]. Fix the issues."
```

### 7. Leverage Cursor's Autocomplete

As you type, Claude suggests completions:
- Function implementations
- Test cases
- API endpoints
- Type definitions

Press `Tab` to accept suggestions.

---

## 🎯 DEVELOPMENT WORKFLOW

### Daily Routine

**Morning (Start of Day):**
1. Pull latest changes: `git pull origin development`
2. Start services: `docker compose up -d`
3. Review today's goals (which feature to build)

**During Development:**
1. Press `Cmd+K` to ask Claude to implement feature
2. Review generated code
3. Run tests: `npm test`
4. Test manually in browser/Postman
5. Iterate if needed

**End of Day:**
1. Run full test suite: `npm run test:all`
2. Commit: `git commit -m "feat: implement TrackMan integration"`
3. Push: `git push origin feature/trackman-integration`
4. Stop services: `docker compose down`

### Weekly Milestones

**Week 1:**
- Day 1-2: Project setup
- Day 3-5: Feature 1 (TrackMan Integration)

**Week 2:**
- Day 6-7: Feature 2 (AI Highlight Selection)
- Day 8-10: Feature 3 (Video Generation)

**Week 3:**
- Day 11-12: Feature 4-5 (Dashboard, Notifications)
- Day 13-15: Testing, refinement, pilot launch

---

## 🐛 DEBUGGING WITH CLAUDE

### When You Hit an Error

**Step 1:** Copy error message

**Step 2:** Press `Cmd+K` and prompt:

```
I'm getting this error:

[paste full error message]

Context: @DEVELOPMENT_SETUP.md (Troubleshooting)

Analyze the error and provide:
1. Root cause
2. Step-by-step fix
3. How to prevent this in future
```

### Common Debugging Prompts

**Database Issues:**
```
Prisma query is failing: [error]
Show me how to fix the query and test it.
```

**API Issues:**
```
API endpoint returns 500 error: [error]
Debug the issue in src/routes/fittings.ts
```

**Frontend Issues:**
```
React component not rendering: [error]
Check src/pages/Dashboard.tsx and fix the issue.
```

**Video Generation Issues:**
```
FFmpeg command failing: [error]
Fix the FFmpeg pipeline in video-service/app/ffmpeg_renderer.py
```

---

## 📊 TRACKING PROGRESS

### Use Cursor's TODO Tree

Add TODO comments that Cursor will track:

```typescript
// TODO: Add retry logic for TrackMan API failures
// FIXME: Video generation timeout after 60 minutes
// NOTE: This needs optimization for large datasets
```

Cursor shows all TODOs in sidebar.

### Git Commits

Use Conventional Commits:

```bash
feat(trackman): implement shot polling worker
fix(api): handle 429 rate limit errors
docs(readme): update setup instructions
test(fittings): add integration tests
chore(deps): update dependencies
```

### Update Project README

Press `Cmd+K`:

```
Update README.md to reflect current progress:

- [x] TrackMan Integration
- [x] AI Highlight Selection
- [ ] Video Generation (in progress)
- [ ] Dashboard
- [ ] Notifications
```

---

## 🎓 LEARNING RESOURCES

### Ask Claude to Teach You

**Learn by doing:**

```
Teach me how Prisma ORM works by showing me:
1. How to define a schema
2. How to create a migration
3. How to query data with type safety
4. Common patterns (transactions, relations)

Then help me implement the Fitting model.
```

**Understand architecture:**

```
Explain the FitClip architecture in simple terms:
1. How does data flow from TrackMan to video?
2. Why use Redis for job queues?
3. Why separate video service from API?
4. How does real-time WebSocket work?

Use diagrams if helpful.
```

---

## 🚀 GOING TO PRODUCTION

### Pre-Launch Checklist

Press `Cmd+K`:

```
Generate a production readiness checklist for FitClip:

Review:
- @TECHNICAL_ARCHITECTURE.md (deployment section)
- @FEATURE_SPECS.md (all acceptance criteria)
- @API_REFERENCE.md (all endpoints)

Create checklist covering:
1. All features implemented and tested
2. Security audit passed
3. Performance benchmarks met
4. Monitoring configured
5. Backup/recovery tested
6. Documentation complete
```

### Deploy to Production

Follow deployment guide in TECHNICAL_ARCHITECTURE.md

```bash
# Build Docker images
docker build -t fitclip-api ./backend
docker build -t fitclip-video ./video-service

# Push to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin <ecr-url>
docker push <ecr-url>/fitclip-api:latest

# Deploy to ECS (via Terraform)
cd infrastructure
terraform apply
```

---

## 🎉 SUCCESS METRICS

Track these as you build:

- [ ] All 5 MVP features complete
- [ ] 80%+ test coverage
- [ ] <30 min video generation (p99)
- [ ] 99.9% shot capture success
- [ ] 3-5 pilot shops onboarded
- [ ] 60%+ conversion rate achieved
- [ ] NPS >50 from pilot shops

---

## 📞 SUPPORT

**Questions?**
- Ask Claude: Press `Cmd+K` and reference documentation
- Check troubleshooting: `@DEVELOPMENT_SETUP.md`
- Review specs: `@FEATURE_SPECS.md`

**Claude can't answer?**
- Search codebase: `Cmd+Shift+F`
- Check GitHub issues
- Review API docs: `@API_REFERENCE.md`

---

## 🎯 FINAL TIPS

1. **Start Small:** Build one feature at a time, test thoroughly
2. **Reference Docs:** Always add context with `@filename`
3. **Iterate:** Don't expect perfect code first try
4. **Test Often:** Run `npm test` after every feature
5. **Commit Frequently:** Small commits are easier to debug
6. **Ask Questions:** Claude is here to help, ask anything!

---

**You're ready to build FitClip with Claude in Cursor! 🚀**

**Start with:** Phase 1 (Project Setup), then follow the workflow day by day.

**Remember:** Claude has access to all documentation. Just reference it with `@filename` and ask specific questions.

**Good luck! You've got this! ⛳️🎥**

