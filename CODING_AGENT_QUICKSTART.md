# FitClip - Coding Agent Quickstart Guide

**Purpose:** Get AI coding agents productive immediately  
**Target:** Claude, GPT-4, Cursor AI, Copilot, etc.  
**Last Updated:** November 28, 2025

---

## 🎯 MISSION

Build FitClip: An AI-powered SaaS platform that turns golf club fittings into personalized video recaps, increasing shop conversion rates from 40% to 68%.

---

## 📚 DOCUMENTATION HIERARCHY

### **MUST READ FIRST** (in this order):

1. **[README.md](./README.md)** - Master index, project overview
2. **[TECHNICAL_ARCHITECTURE.md](./TECHNICAL_ARCHITECTURE.md)** - How the system works
3. **[API_REFERENCE.md](./API_REFERENCE.md)** - What to build (endpoints)
4. **[DEVELOPMENT_SETUP.md](./DEVELOPMENT_SETUP.md)** - How to get started

### **READ WHEN BUILDING FEATURES:**

5. **[FEATURE_SPECS.md](./FEATURE_SPECS.md)** - Detailed feature requirements
6. **[PRD.md](./PRD.md)** - Product requirements and user stories

### **READ FOR CONTEXT:**

7. **[LANDING_PAGE_BLUEPRINT.md](./LANDING_PAGE_BLUEPRINT.md)** - Marketing copy
8. **[index.html](./index.html)** - Landing page demo

---

## ⚡ QUICKSTART CHECKLIST

### Step 1: Environment Setup (30 minutes)

```bash
# Clone repo
git clone <repo-url>
cd fitclip

# Start infrastructure
docker compose up -d

# Install backend dependencies
cd backend && npm install

# Install video service dependencies
cd ../video-service
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Install frontend dependencies
cd ../frontend && npm install

# Run database migrations
cd ../backend && npx prisma migrate dev

# Seed database
npx prisma db seed
```

### Step 2: Run Services (3 terminals)

**Terminal 1: Backend API**
```bash
cd backend
npm run dev
# Should start on http://localhost:3000
```

**Terminal 2: Video Service**
```bash
cd video-service
source venv/bin/activate
celery -A app.celery worker --loglevel=info
```

**Terminal 3: Frontend**
```bash
cd frontend
npm run dev
# Should start on http://localhost:5173
```

### Step 3: Verify Setup

```bash
# Test API
curl http://localhost:3000/health
# Expected: {"status":"ok"}

# Test database
psql -U fitclip -d fitclip_dev -c "SELECT COUNT(*) FROM shops;"
# Expected: 1 (from seed data)

# Open frontend
open http://localhost:5173
# Should see login page
```

---

## 🏗️ ARCHITECTURE SUMMARY

### Technology Stack

**Frontend:**
- React 18 + TypeScript
- Redux Toolkit (state)
- Material-UI (components)
- Vite (build tool)

**Backend:**
- Node.js 20 + Express
- TypeScript
- Prisma ORM
- PostgreSQL database
- Redis cache
- Socket.IO (WebSocket)

**Video Service:**
- Python 3.11 + FastAPI
- FFmpeg (video rendering)
- XGBoost (ML highlight selection)
- Celery + Redis (job queue)

**Infrastructure:**
- AWS ECS (containers)
- AWS RDS (PostgreSQL)
- AWS S3 + CloudFront (video storage/CDN)
- AWS ElastiCache (Redis)

### Key Services

| Service | Port | Purpose |
|---------|------|---------|
| Frontend | 5173 | Shop dashboard, customer portal |
| API | 3000 | REST API + WebSocket |
| Video Service | 8000 | Video generation worker |
| PostgreSQL | 5432 | Primary database |
| Redis | 6379 | Cache + job queues |
| LocalStack | 4566 | AWS emulator (dev only) |

---

## 📋 CORE FEATURES TO BUILD (Priority Order)

### MVP Features (Month 1-3)

#### Feature 1: TrackMan Integration (PRIORITY 1)
**File:** `FEATURE_SPECS.md` (Page 1-15)

**What to build:**
- FitClip Agent (Electron app) that polls TrackMan API
- Backend API endpoints to receive shot data
- WebSocket for real-time shot updates to fitter's iPad
- Invalid shot detection (bad swings filtered out)

**Key endpoints:**
```
POST /v1/trackman/connect
POST /v1/fittings/:id/shots
GET  /v1/fittings/:id/shots
PATCH /v1/shots/:id
```

**Acceptance criteria:**
- 99.9% shot capture success rate
- <5 second latency from shot to database
- Works offline (buffers shots locally)

---

#### Feature 2: AI Highlight Selection (PRIORITY 1)
**File:** `FEATURE_SPECS.md` (Page 15-25)

**What to build:**
- Scoring algorithm for shot quality (0-100 scale)
- Selection algorithm (picks 7 best shots)
- Before/after comparison logic
- Fitter review/override UI (optional workflow)

**Key algorithm:**
```python
def calculate_shot_quality_score(shot):
    # Distance: 40% weight
    # Accuracy: 30% weight
    # Contact: 20% weight
    # Spin: 10% weight
    return score (0-100)

def select_highlights(shots, num=7):
    # 1. Longest shot
    # 2. Straightest shot
    # 3. Best overall
    # 4. Biggest improvement from baseline
    # 5-7. High-quality variety
    return [shot_ids]
```

**Acceptance criteria:**
- 85% agreement with human experts
- <10 seconds processing time for 60 shots
- Fitter can override 30% of time

---

#### Feature 3: Video Generation (PRIORITY 2)
**File:** `FEATURE_SPECS.md` (Page 25+)

**What to build:**
- FFmpeg pipeline for video rendering
- Data overlay graphics (ball speed, carry distance, etc.)
- Shop branding (logo, colors)
- S3 upload + CloudFront distribution
- Video ready notification (SMS + email)

**Key workflow:**
```
1. Fitting completed → Enqueue job in Redis
2. Celery worker picks up job
3. Fetch shots + shop branding from database
4. ML selects highlights
5. FFmpeg renders video (2-3 minutes)
   - Intro (3s)
   - 7 highlight clips (15-20s each)
   - Before/after comparison (30s)
   - Recommendations (20s)
   - Outro (5s)
6. Upload to S3
7. Send SMS + email with link
```

**Acceptance criteria:**
- <30 minutes total (p99)
- 1080p, 60fps, H.264
- 30-50 MB file size
- 99.5% success rate

---

#### Feature 4: Multi-Channel Delivery (PRIORITY 2)
**File:** `API_REFERENCE.md` (Video endpoints)

**What to build:**
- SMS delivery via Twilio
- Email delivery via SendGrid
- Customer portal (video player)
- Analytics tracking (video views, clicks)

**Key endpoints:**
```
GET /v1/videos/:id
POST /v1/analytics/track
```

**Acceptance criteria:**
- Video link sent within 1 minute of completion
- 92% watch rate
- Works on mobile (80% of views)

---

#### Feature 5: Shop Dashboard (PRIORITY 2)
**File:** `API_REFERENCE.md` (Shop/Analytics endpoints)

**What to build:**
- Login/authentication (JWT)
- Fittings list (with filters, search)
- Fitting detail view (shots + video)
- Analytics dashboard (conversion rate, revenue)
- Settings (branding, TrackMan connection)

**Key pages:**
```
/login
/dashboard (metrics overview)
/fittings (list)
/fittings/:id (detail)
/analytics (charts)
/settings (shop config)
```

**Acceptance criteria:**
- <2 second page load
- Real-time updates (WebSocket)
- Mobile responsive (iPad)

---

## 🗄️ DATABASE SCHEMA (Key Tables)

**See:** `TECHNICAL_ARCHITECTURE.md` Section 5.1

### Core Tables

```sql
-- Shops (300 rows at 500 shops)
shops {
  id UUID PK
  name VARCHAR
  email VARCHAR
  subscription_tier VARCHAR
  subscription_status VARCHAR
  created_at TIMESTAMP
}

-- Fittings (600K rows/year)
fittings {
  id UUID PK
  shop_id UUID FK
  customer_id UUID FK
  fitting_type VARCHAR
  status VARCHAR  -- in_progress, completed, video_generated
  started_at TIMESTAMP
  completed_at TIMESTAMP
}

-- Shots (36M rows/year: 60 shots × 600K fittings)
shots {
  id UUID PK
  fitting_id UUID FK
  club_type VARCHAR
  ball_speed_mph DECIMAL
  carry_distance_yards DECIMAL
  is_valid BOOLEAN
  is_highlight BOOLEAN
  highlight_score DECIMAL  -- 0-100
  timestamp TIMESTAMP
}

-- Videos (600K rows/year)
videos {
  id UUID PK
  fitting_id UUID FK
  status VARCHAR  -- pending, rendering, ready, failed
  url VARCHAR
  duration_seconds INTEGER
  generated_at TIMESTAMP
  watch_count INTEGER
}
```

### Indexes (Critical)

```sql
CREATE INDEX idx_shots_fitting ON shots(fitting_id);
CREATE INDEX idx_shots_highlight ON shots(fitting_id, is_highlight);
CREATE INDEX idx_fittings_shop_date ON fittings(shop_id, completed_at);
```

---

## 🔑 KEY API ENDPOINTS

**See:** `API_REFERENCE.md` for full documentation

### Authentication

```http
POST /v1/auth/login
POST /v1/auth/refresh
POST /v1/auth/logout
```

### Fittings

```http
POST   /v1/fittings                    # Create fitting
GET    /v1/fittings                    # List fittings
GET    /v1/fittings/:id                # Get fitting details
PATCH  /v1/fittings/:id/complete       # Complete + trigger video
DELETE /v1/fittings/:id                # Delete fitting
```

### Shots

```http
POST  /v1/fittings/:id/shots           # Add shot (from TrackMan)
GET   /v1/fittings/:id/shots           # Get all shots
PATCH /v1/shots/:id                    # Update shot (mark valid/invalid)
```

### Videos

```http
GET    /v1/videos/:id                  # Get video details + URL
POST   /v1/videos/:id/regenerate       # Regenerate (if failed)
DELETE /v1/videos/:id                  # Delete (GDPR)
```

### TrackMan

```http
POST   /v1/trackman/connect            # Connect launch monitor
GET    /v1/trackman/:id/status         # Check connection
DELETE /v1/trackman/:id                # Disconnect
```

---

## 🧪 TESTING REQUIREMENTS

### Unit Tests (80% coverage)

**Backend:**
```bash
cd backend
npm test

# Specific test
npm test -- src/services/fitting.service.test.ts
```

**Video Service:**
```bash
cd video-service
pytest

# Specific test
pytest tests/test_highlight_selection.py
```

### Integration Tests

**Example: End-to-end fitting flow**
```typescript
// backend/tests/integration/fitting-flow.test.ts
it('should complete full fitting flow', async () => {
  // 1. Create fitting
  const fitting = await api.post('/v1/fittings', {...});
  
  // 2. Add 10 shots
  for (let i = 0; i < 10; i++) {
    await api.post(`/v1/fittings/${fitting.id}/shots`, {...});
  }
  
  // 3. Complete fitting
  await api.patch(`/v1/fittings/${fitting.id}/complete`);
  
  // 4. Wait for video (up to 60 seconds)
  let videoReady = false;
  for (let i = 0; i < 60; i++) {
    await sleep(1000);
    const status = await api.get(`/v1/fittings/${fitting.id}`);
    if (status.video?.status === 'ready') {
      videoReady = true;
      break;
    }
  }
  
  expect(videoReady).toBe(true);
});
```

---

## 🚨 COMMON PITFALLS & SOLUTIONS

### Issue 1: Shot Data Not Appearing in Frontend

**Cause:** WebSocket not connected or Redis not running

**Solution:**
```bash
# Check Redis
redis-cli ping  # Should return PONG

# Check WebSocket connection in browser console
# Should see: "WebSocket connected"

# Restart backend
cd backend && npm run dev
```

---

### Issue 2: Video Generation Fails

**Cause:** FFmpeg not installed or S3 not configured

**Solution:**
```bash
# Install FFmpeg
brew install ffmpeg  # macOS
sudo apt-get install ffmpeg  # Linux

# Verify
ffmpeg -version

# Check S3 (LocalStack for dev)
aws --endpoint-url=http://localhost:4566 s3 ls
# Should see: fitclip-videos-dev
```

---

### Issue 3: TrackMan API Not Connecting

**Cause:** Using production API keys in development

**Solution:**
```bash
# Use sandbox/test keys in .env
TRACKMAN_API_KEY=sandbox_tm_xxxxxxxx
TRACKMAN_API_URL=https://sandbox-api.trackman.com
```

---

## 📊 SUCCESS METRICS TO TRACK

While building, instrument these metrics:

```javascript
// Track video generation time
datadog.histogram('video.generation_time_ms', durationMs);

// Track shot capture success
datadog.increment('shot.captured', 1, { shop_id, fitting_id });

// Track API errors
datadog.increment('api.error', 1, { endpoint, error_code });

// Track business metrics
datadog.increment('fitting.completed', 1);
datadog.increment('video.watched', 1);
datadog.increment('purchase.clicked', 1);
```

**Target Metrics:**
- Video generation: <30 min (p99)
- Shot capture: 99.9% success
- API uptime: 99.9%
- Conversion rate: 68%
- Video watch rate: 92%

---

## 🎯 DEFINITION OF DONE

For each feature to be considered complete:

- [ ] **Code:** Implemented per feature spec
- [ ] **Tests:** Unit tests written (80%+ coverage)
- [ ] **Integration:** Integration test for happy path
- [ ] **Documentation:** API Reference updated (if endpoints changed)
- [ ] **Linting:** No ESLint/Pylint errors
- [ ] **PR Review:** Approved by 1+ team members
- [ ] **CI/CD:** All automated tests passing
- [ ] **Manual Test:** Tested locally by developer
- [ ] **Demo:** Demonstrated to product team

---

## 🚀 DEPLOYMENT CHECKLIST

Before deploying to production:

- [ ] All tests passing (unit + integration)
- [ ] Database migrations tested
- [ ] Environment variables set (production secrets)
- [ ] Load testing completed (500 concurrent users)
- [ ] Security audit passed (no critical vulnerabilities)
- [ ] Monitoring configured (Datadog alerts)
- [ ] Rollback plan documented
- [ ] Stakeholders notified (maintenance window if needed)

---

## 📞 GETTING HELP

**Questions about:**
- **Architecture:** Read `TECHNICAL_ARCHITECTURE.md`
- **API contracts:** Read `API_REFERENCE.md`
- **Feature requirements:** Read `FEATURE_SPECS.md`
- **Setup issues:** Read `DEVELOPMENT_SETUP.md`
- **Product vision:** Read `PRD.md`

**Still stuck?**
- Check `DEVELOPMENT_SETUP.md` Troubleshooting section
- Search codebase for similar patterns
- Review existing test files for examples

---

## 🎓 LEARNING RESOURCES

### Recommended Reading Order:

1. **Day 1:** `README.md` → `TECHNICAL_ARCHITECTURE.md`
2. **Day 2:** `API_REFERENCE.md` → `DEVELOPMENT_SETUP.md`
3. **Day 3:** `FEATURE_SPECS.md` (Feature 1: TrackMan)
4. **Day 4:** Start coding Feature 1
5. **Day 5+:** Continue with Features 2-5

### Example Code Patterns:

**Backend API Endpoint:**
```typescript
// backend/src/routes/fittings.ts
import { Router } from 'express';
import { z } from 'zod';
import { authenticate } from '../middleware/auth';
import { FittingService } from '../services/fitting.service';

const router = Router();
const fittingService = new FittingService();

const createFittingSchema = z.object({
  shop_id: z.string().uuid(),
  customer: z.object({
    first_name: z.string(),
    last_name: z.string(),
    email: z.string().email(),
  }),
  fitting_type: z.enum(['driver', 'irons', 'full_bag']),
});

router.post('/', authenticate, async (req, res) => {
  try {
    const data = createFittingSchema.parse(req.body);
    const fitting = await fittingService.create(data);
    res.status(201).json(fitting);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

export default router;
```

**Video Generation Worker:**
```python
# video-service/app/tasks.py
from celery import Celery
from .video_generator import VideoGenerator

celery = Celery('fitclip', broker='redis://localhost:6379/0')

@celery.task
def generate_video(fitting_id: str):
    """Generate video for completed fitting."""
    generator = VideoGenerator()
    
    # 1. Fetch data
    fitting_data = generator.fetch_fitting_data(fitting_id)
    
    # 2. Select highlights
    highlights = generator.select_highlights(fitting_data['shots'])
    
    # 3. Render video
    video_path = generator.render_video(fitting_data, highlights)
    
    # 4. Upload to S3
    video_url = generator.upload_to_s3(video_path)
    
    # 5. Update database
    generator.update_video_status(fitting_id, 'ready', video_url)
    
    # 6. Send notifications
    generator.send_notifications(fitting_id)
    
    return {'status': 'success', 'video_url': video_url}
```

---

## ✅ FINAL CHECKLIST

Before declaring "FitClip MVP is complete":

- [ ] All 5 MVP features implemented
- [ ] 3-5 pilot shops onboarded and actively using
- [ ] 60%+ conversion rate achieved (up from 40%)
- [ ] 90%+ video watch rate
- [ ] <30 minute video generation time (p99)
- [ ] 99.9% shot capture success rate
- [ ] <5% monthly churn
- [ ] NPS >50 from pilot shops
- [ ] All critical bugs fixed
- [ ] Production monitoring configured
- [ ] Disaster recovery plan tested
- [ ] Security audit passed
- [ ] Documentation complete
- [ ] Team trained on support processes

---

**YOU ARE NOW READY TO BUILD FITCLIP! 🎉**

Start with Feature 1 (TrackMan Integration) in `FEATURE_SPECS.md` and work through the priority list. Good luck! ⛳️

---

**Document Version:** 1.0  
**Last Updated:** November 28, 2025  
**Questions?** Refer to `README.md` for support resources

