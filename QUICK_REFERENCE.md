# FitClip Quick Reference Card

**For:** Using Claude in Cursor to build FitClip  
**Last Updated:** November 28, 2025

---

## 🚀 QUICK START (5 MINUTES)

```bash
# 1. Open project in Cursor
cd /Users/davidegan/fitclip
cursor .

# 2. Create .cursorrules file (copy from CURSOR_CLAUDE_GUIDE.md)
# This gives Claude context about the project

# 3. Start development
docker compose up -d

# 4. Press Cmd+K in Cursor and say:
"Set up the FitClip backend project following @DEVELOPMENT_SETUP.md"
```

---

## 📋 ESSENTIAL DOCUMENTS (Always Reference)

| Document | Purpose | When to Use |
|----------|---------|-------------|
| `README.md` | Project overview | First read, orientation |
| `CURSOR_CLAUDE_GUIDE.md` | This guide | Building with Claude |
| `TECHNICAL_ARCHITECTURE.md` | System design | Architecture questions |
| `API_REFERENCE.md` | API contracts | Building endpoints |
| `FEATURE_SPECS.md` | Feature details | Implementing features |
| `DEVELOPMENT_SETUP.md` | Local setup | Setup, troubleshooting |

---

## 💬 ESSENTIAL CURSOR SHORTCUTS

| Shortcut | Action |
|----------|--------|
| `Cmd+K` | Open Claude chat (Mac) |
| `Ctrl+K` | Open Claude chat (Windows/Linux) |
| `Cmd+L` | Inline edit with Claude |
| `Cmd+I` | Generate code from comment |
| `@filename` | Add file to Claude's context |
| `Tab` | Accept autocomplete suggestion |

---

## 🎯 COMMON CLAUDE PROMPTS

### Project Setup

```
Create the FitClip backend structure:
Context: @DEVELOPMENT_SETUP.md @TECHNICAL_ARCHITECTURE.md

Initialize Node.js 20 + TypeScript with Express, Prisma, Socket.IO
Set up folder structure: /routes, /services, /middleware, /lib
Configure ESLint, Prettier, Jest
```

### Implement Feature

```
Implement TrackMan Integration:
Context: @FEATURE_SPECS.md (Feature 1, pages 1-15)
Context: @API_REFERENCE.md (TrackMan endpoints)

Build:
1. API client for TrackMan
2. POST /v1/trackman/connect endpoint
3. Polling worker (every 2 seconds)
4. WebSocket for real-time updates
5. Tests (unit + integration)

Follow exact specifications in feature specs.
```

### Create API Endpoint

```
Create this API endpoint following @API_REFERENCE.md:

POST /v1/fittings/:id/shots

Include:
- Zod validation schema
- Authenticate middleware
- Error handling
- TypeScript types
- Unit tests
```

### Fix Bug

```
Debug this error: [paste error]

Context: @DEVELOPMENT_SETUP.md (Troubleshooting)

Provide:
1. Root cause analysis
2. Fix (with code)
3. How to prevent future occurrence
```

### Write Tests

```
Write comprehensive tests for FittingService:

Context: @FEATURE_SPECS.md (Testing Requirements)

Include:
- Unit tests for all methods
- Mock Prisma client
- Test edge cases
- 80%+ coverage

Use Jest + TypeScript
```

### Generate Documentation

```
Generate JSDoc comments for this code:

[paste code]

Include:
- Function description
- @param descriptions with types
- @returns description
- @throws for errors
- @example usage
```

---

## 🏗️ BUILD ORDER (3-Week MVP)

### Week 1: Foundation + TrackMan

**Day 1-2:** Project Setup
```
"Set up backend, frontend, video service, and Docker Compose"
Context: @DEVELOPMENT_SETUP.md
```

**Day 3-5:** Feature 1 - TrackMan Integration
```
"Implement TrackMan integration following Feature 1 spec"
Context: @FEATURE_SPECS.md (pages 1-15)
```

### Week 2: AI + Video

**Day 6-7:** Feature 2 - AI Highlight Selection
```
"Implement highlight selection algorithm"
Context: @FEATURE_SPECS.md (pages 15-25)
```

**Day 8-10:** Feature 3 - Video Generation
```
"Implement FFmpeg video generation pipeline"
Context: @FEATURE_SPECS.md (pages 25+)
```

### Week 3: Dashboard + Testing

**Day 11-12:** Dashboard + Notifications
```
"Build shop dashboard with authentication"
Context: @API_REFERENCE.md (all endpoints)
```

**Day 13-15:** Testing + Refinement
```
"Write integration tests for full fitting flow"
Run manual testing with pilot shops
```

---

## 🔧 COMMON TASKS

### Start Services

```bash
# Terminal 1: Infrastructure
docker compose up -d

# Terminal 2: Backend
cd backend && npm run dev

# Terminal 3: Video Service  
cd video-service
source venv/bin/activate
celery -A app.celery worker --loglevel=info

# Terminal 4: Frontend
cd frontend && npm run dev
```

### Run Tests

```bash
# Backend
cd backend && npm test

# Video Service
cd video-service && pytest

# Frontend
cd frontend && npm test

# E2E
cd frontend && npm run test:e2e

# Coverage
npm run test:coverage
```

### Database Operations

```bash
# Create migration
cd backend
npx prisma migrate dev --name add_trackman

# Generate Prisma Client
npx prisma generate

# Seed database
npx prisma db seed

# Open Prisma Studio (GUI)
npx prisma studio

# Reset database (DEV ONLY!)
npx prisma migrate reset
```

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/trackman-integration

# Commit (use conventional commits)
git add .
git commit -m "feat(trackman): implement polling worker"

# Push
git push origin feature/trackman-integration

# Create PR (GitHub)
```

---

## 🐛 DEBUGGING GUIDE

### API Not Starting

```
Error: Port 3000 already in use

Fix:
lsof -ti:3000 | xargs kill -9
npm run dev
```

### Database Connection Failed

```
Error: connect ECONNREFUSED 127.0.0.1:5432

Fix:
docker compose restart postgres
# Wait 10 seconds for startup
npm run dev
```

### Prisma Client Out of Sync

```
Error: Prisma schema and client out of sync

Fix:
npx prisma generate
```

### Video Generation Fails

```
Error: FFmpeg not found

Fix (Mac):
brew install ffmpeg

Fix (Linux):
sudo apt-get install ffmpeg
```

### WebSocket Not Connecting

```
Error: WebSocket connection failed

Fix:
1. Check Redis is running: redis-cli ping
2. Check Socket.IO initialized in backend
3. Check firewall not blocking WebSocket port
```

---

## 📊 KEY METRICS TO TRACK

| Metric | Target | How to Check |
|--------|--------|--------------|
| Test Coverage | >80% | `npm run test:coverage` |
| API Response Time | <200ms (p95) | Datadog APM |
| Video Generation | <30 min (p99) | Check logs |
| Shot Capture | 99.9% success | Analytics dashboard |
| Conversion Rate | 68% | Shop analytics |

---

## 🎯 MVP ACCEPTANCE CRITERIA

### Feature 1: TrackMan Integration
- [ ] Connect TrackMan via API (98%+ success)
- [ ] Capture shots in real-time (<5s latency)
- [ ] Display shots in fitter app (live updates)
- [ ] Handle offline mode (buffer shots)
- [ ] Tests pass (80%+ coverage)

### Feature 2: AI Highlight Selection
- [ ] Score shots 0-100 (algorithm implemented)
- [ ] Select 7 highlights (variety + quality)
- [ ] Identify before/after pairs
- [ ] Fitter can override selections (30% of time)
- [ ] 85% agreement with human experts

### Feature 3: Video Generation
- [ ] Render 1080p video (<30 min, p99)
- [ ] Include shop branding (logo, colors)
- [ ] Upload to S3 (99.5% success)
- [ ] Send SMS + email notifications
- [ ] 92% video watch rate

### Feature 4: Shop Dashboard
- [ ] Login with JWT authentication
- [ ] View fittings list (with filters)
- [ ] View analytics (conversion rate, revenue)
- [ ] Real-time updates (WebSocket)
- [ ] Mobile responsive (iPad)

---

## 🚨 BEFORE YOU COMMIT

```bash
# Run this checklist:
npm run lint                    # No linting errors
npm test                        # All tests pass
npm run build                   # Build succeeds
git status                      # Only intended files staged

# Then commit:
git commit -m "feat(scope): description"
```

---

## 🎓 LEARNING RESOURCES

### Ask Claude to Explain

```
"Explain how [concept] works in FitClip:
Context: @TECHNICAL_ARCHITECTURE.md

Break it down with:
1. High-level overview
2. Step-by-step flow
3. Code examples
4. Common pitfalls"
```

### Architecture Questions

```
"Why does FitClip use [technology]?
Context: @TECHNICAL_ARCHITECTURE.md

Compare it to alternatives and explain trade-offs."
```

### Code Review

```
"Review this code I wrote:

[paste code]

Check for:
1. Bugs or errors
2. Best practices violations
3. Performance issues
4. Security concerns
5. Missing tests"
```

---

## 🎉 LAUNCH CHECKLIST

**MVP Complete When:**
- [ ] All 5 features implemented
- [ ] All tests passing (80%+ coverage)
- [ ] 3-5 pilot shops onboarded
- [ ] 60%+ conversion rate achieved
- [ ] <30 min video generation (p99)
- [ ] 99.9% shot capture success
- [ ] Security audit passed
- [ ] Documentation complete
- [ ] Monitoring configured (Datadog)
- [ ] Backup/recovery tested

---

## 📞 GET HELP

**From Claude (in Cursor):**
```
Press Cmd+K and ask:
"I'm stuck on [problem]. Context: @[relevant-doc].md
Help me [solve it]."
```

**From Documentation:**
- Setup issues → `@DEVELOPMENT_SETUP.md`
- Architecture questions → `@TECHNICAL_ARCHITECTURE.md`
- API details → `@API_REFERENCE.md`
- Feature requirements → `@FEATURE_SPECS.md`

**Still Stuck?**
- Search codebase: `Cmd+Shift+F`
- Check GitHub issues
- Review similar code in codebase

---

## 💡 PRO TIPS

1. **Use @mentions:** Always add context with `@filename`
2. **Be specific:** "Create X following Y spec" > "Create X"
3. **Iterate:** Claude rarely gets it perfect first try
4. **Test often:** `npm test` after every feature
5. **Read errors:** Claude can debug from terminal output
6. **Save prompts:** Reuse successful prompts
7. **Ask why:** "Explain why you chose this approach"
8. **Review code:** Don't blindly accept generated code

---

## 🏆 DAILY WORKFLOW

```
1. Morning: git pull, docker compose up -d
2. Plan: Pick feature from FEATURE_SPECS.md
3. Build: Use Cmd+K to implement with Claude
4. Test: npm test, manual testing
5. Review: git diff, check quality
6. Commit: git commit -m "feat: ..."
7. Evening: git push, docker compose down
```

---

**REMEMBER:**

📖 All documentation is in `/Users/davidegan/fitclip/`  
🤖 Claude knows the project context (via .cursorrules)  
💬 Press `Cmd+K` to ask Claude anything  
📚 Reference docs with `@filename`  
✅ Follow the build order in `CURSOR_CLAUDE_GUIDE.md`

---

**YOU'VE GOT THIS! START BUILDING! 🚀⛳️**

