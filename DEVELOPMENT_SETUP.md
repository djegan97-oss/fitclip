# FitClip Development Setup Guide

**Version:** 1.0  
**Last Updated:** November 28, 2025  
**Target Audience:** Developers joining FitClip team

---

## TABLE OF CONTENTS

1. [Prerequisites](#prerequisites)
2. [Repository Setup](#repository-setup)
3. [Local Development Environment](#local-development-environment)
4. [Database Setup](#database-setup)
5. [Running Services](#running-services)
6. [Testing](#testing)
7. [Troubleshooting](#troubleshooting)
8. [IDE Configuration](#ide-configuration)

---

## PREREQUISITES

### Required Software

Install the following on your development machine:

**1. Node.js & npm**
```bash
# Install Node.js 20 LTS via nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
nvm use 20
node --version  # Should be v20.x.x
npm --version   # Should be 10.x.x
```

**2. Python**
```bash
# Install Python 3.11+ via pyenv (recommended)
curl https://pyenv.run | bash
pyenv install 3.11.6
pyenv global 3.11.6
python --version  # Should be 3.11.6
```

**3. Docker & Docker Compose**
```bash
# macOS (via Homebrew)
brew install --cask docker
# Start Docker Desktop app

# Linux
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Verify
docker --version
docker compose version
```

**4. PostgreSQL Client Tools**
```bash
# macOS
brew install postgresql@15

# Linux
sudo apt-get install postgresql-client-15
```

**5. Redis Client Tools**
```bash
# macOS
brew install redis

# Linux
sudo apt-get install redis-tools
```

**6. Git**
```bash
# macOS (pre-installed, verify)
git --version

# Linux
sudo apt-get install git
```

---

## REPOSITORY SETUP

### 1. Clone Repository

```bash
# SSH (recommended)
git clone git@github.com:fitclip/fitclip-backend.git
cd fitclip-backend

# HTTPS (alternative)
git clone https://github.com/fitclip/fitclip-backend.git
cd fitclip-backend
```

### 2. Install Dependencies

**Backend (Node.js API):**
```bash
cd backend
npm install
```

**Video Service (Python):**
```bash
cd video-service
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

**Frontend (React):**
```bash
cd frontend
npm install
```

### 3. Environment Variables

Copy example environment files:

```bash
# Backend
cp backend/.env.example backend/.env

# Video Service
cp video-service/.env.example video-service/.env

# Frontend
cp frontend/.env.example frontend/.env
```

**Edit `.env` files with your local configuration:**

**`backend/.env`:**
```bash
# Environment
NODE_ENV=development

# Server
PORT=3000
API_BASE_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://fitclip:password@localhost:5432/fitclip_dev

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=dev_secret_change_in_production_abc123
JWT_ACCESS_EXPIRATION=15m
JWT_REFRESH_EXPIRATION=30d

# AWS (LocalStack for local dev)
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test
AWS_ENDPOINT=http://localhost:4566  # LocalStack
S3_BUCKET=fitclip-videos-dev

# External APIs (use sandbox/test keys)
TRACKMAN_API_KEY=sandbox_tm_xxxxxxxx
TRACKMAN_API_URL=https://sandbox-api.trackman.com

SENDGRID_API_KEY=SG.test_xxxxxxxx
TWILIO_ACCOUNT_SID=ACtest_xxxxxxxx
TWILIO_AUTH_TOKEN=test_xxxxxxxx

# Logging
LOG_LEVEL=debug
```

**`video-service/.env`:**
```bash
# Environment
ENVIRONMENT=development

# Redis (Celery broker)
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0

# Database
DATABASE_URL=postgresql://fitclip:password@localhost:5432/fitclip_dev

# AWS (LocalStack)
AWS_ENDPOINT=http://localhost:4566
S3_BUCKET=fitclip-videos-dev

# FFmpeg
FFMPEG_PATH=/usr/local/bin/ffmpeg
FFMPEG_THREADS=2

# ML Model
MODEL_PATH=./models/highlight_selector_v1.model
```

**`frontend/.env`:**
```bash
VITE_API_URL=http://localhost:3000
VITE_WS_URL=ws://localhost:3000
VITE_ENV=development
```

---

## LOCAL DEVELOPMENT ENVIRONMENT

### Option 1: Docker Compose (Recommended)

**Single command to start all services:**

```bash
docker compose up -d
```

**`docker-compose.yml`:**
```yaml
version: '3.9'

services:
  # PostgreSQL
  postgres:
    image: postgres:15.5-alpine
    container_name: fitclip-postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: fitclip
      POSTGRES_PASSWORD: password
      POSTGRES_DB: fitclip_dev
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U fitclip"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis
  redis:
    image: redis:7.2-alpine
    container_name: fitclip-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # LocalStack (AWS services emulator)
  localstack:
    image: localstack/localstack:3.0
    container_name: fitclip-localstack
    ports:
      - "4566:4566"  # LocalStack gateway
    environment:
      SERVICES: s3,secretsmanager
      DEBUG: 1
      DATA_DIR: /tmp/localstack/data
    volumes:
      - localstack_data:/tmp/localstack
      - /var/run/docker.sock:/var/run/docker.sock
      - ./scripts/init-aws.sh:/docker-entrypoint-initaws.d/init-aws.sh

  # Mailhog (Email testing)
  mailhog:
    image: mailhog/mailhog:v1.0.1
    container_name: fitclip-mailhog
    ports:
      - "1025:1025"  # SMTP server
      - "8025:8025"  # Web UI
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:8025"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  postgres_data:
  redis_data:
  localstack_data:
```

**Start services:**
```bash
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f postgres
docker compose logs -f redis

# Stop services
docker compose down

# Reset (delete all data)
docker compose down -v
```

### Option 2: Manual Setup

**Start PostgreSQL:**
```bash
# macOS (if installed via Homebrew)
brew services start postgresql@15

# Linux (systemd)
sudo systemctl start postgresql
```

**Start Redis:**
```bash
# macOS
brew services start redis

# Linux
sudo systemctl start redis
```

**Install LocalStack (AWS emulator):**
```bash
pip install localstack
localstack start -d
```

---

## DATABASE SETUP

### 1. Create Database

```bash
# If using Docker Compose, database is auto-created
# If using local PostgreSQL:

createdb fitclip_dev -U fitclip
```

### 2. Run Migrations

**Using Prisma:**
```bash
cd backend
npx prisma migrate dev --name init

# Output:
# ✔ Generated Prisma Client
# ✔ Applied migrations:
#   └─ 20251128000000_init
```

**Alternative: Using SQL scripts directly:**
```bash
psql -U fitclip -d fitclip_dev -f scripts/schema.sql
```

### 3. Seed Database

**Load sample data for development:**

```bash
cd backend
npx prisma db seed

# Or run custom seed script:
npm run seed
```

**`backend/prisma/seed.ts`:**
```typescript
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  console.log('Seeding database...');

  // Create test shop
  const shop = await prisma.shop.create({
    data: {
      name: 'Test Golf Shop',
      email: 'shop@test.com',
      phone: '+14155550001',
      subscription_tier: 'unlimited',
      subscription_status: 'active',
    },
  });

  // Create shop owner user
  const hashedPassword = await bcrypt.hash('password123', 10);
  const user = await prisma.user.create({
    data: {
      email: 'owner@test.com',
      password_hash: hashedPassword,
      first_name: 'Test',
      last_name: 'Owner',
      role: 'shop_owner',
      shop_id: shop.id,
    },
  });

  // Create test customers
  const customers = await prisma.customer.createMany({
    data: [
      {
        shop_id: shop.id,
        first_name: 'John',
        last_name: 'Doe',
        email: 'john@example.com',
        phone: '+14155550101',
        handicap: 15.0,
      },
      {
        shop_id: shop.id,
        first_name: 'Jane',
        last_name: 'Smith',
        email: 'jane@example.com',
        phone: '+14155550102',
        handicap: 8.5,
      },
    ],
  });

  console.log('Database seeded successfully!');
  console.log({
    shop,
    user,
    customers_count: customers.count,
  });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

**Run seed:**
```bash
npx ts-node prisma/seed.ts
```

### 4. Verify Database

```bash
# Connect to database
psql -U fitclip -d fitclip_dev

# List tables
\dt

# Check sample data
SELECT * FROM shops;
SELECT * FROM users;
SELECT * FROM customers;

# Exit
\q
```

---

## RUNNING SERVICES

### Start All Services (Development Mode)

**Terminal 1: Backend API**
```bash
cd backend
npm run dev

# Output:
# [nodemon] starting `ts-node src/server.ts`
# Server running on http://localhost:3000
# Database connected
# Redis connected
```

**Terminal 2: Video Service**
```bash
cd video-service
source venv/bin/activate
celery -A app.celery worker --loglevel=info

# Output:
# [2025-11-28 14:30:00] celery@MacBook-Pro.local ready.
# [tasks]
#   . app.tasks.generate_video
#   . app.tasks.select_highlights
```

**Terminal 3: Frontend**
```bash
cd frontend
npm run dev

# Output:
# VITE v5.0.0  ready in 234 ms
# ➜  Local:   http://localhost:5173/
# ➜  Network: http://192.168.1.100:5173/
```

**Terminal 4: Worker Logs (Optional)**
```bash
# Watch Redis queue
redis-cli MONITOR
```

### Access Services

- **Frontend (Shop Dashboard):** http://localhost:5173
- **Backend API:** http://localhost:3000
- **API Docs (Swagger):** http://localhost:3000/docs
- **Mailhog (Email testing):** http://localhost:8025
- **LocalStack Dashboard:** http://localhost:4566/_localstack/health

### Test API

```bash
# Health check
curl http://localhost:3000/health

# Output: {"status":"ok","timestamp":"2025-11-28T14:30:00.000Z"}

# Login (get access token)
curl -X POST http://localhost:3000/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"owner@test.com","password":"password123"}'

# Output: {"access_token":"eyJhbG...","refresh_token":"..."}

# Create fitting (use token from login)
curl -X POST http://localhost:3000/v1/fittings \
  -H "Authorization: Bearer eyJhbG..." \
  -H "Content-Type: application/json" \
  -d '{
    "shop_id": "...",
    "customer": {"first_name":"Tom","last_name":"Test","email":"tom@test.com"},
    "fitting_type": "driver"
  }'
```

---

## TESTING

### Run Unit Tests

**Backend:**
```bash
cd backend
npm test

# With coverage
npm run test:coverage

# Watch mode
npm run test:watch

# Specific file
npm test -- src/services/fitting.service.test.ts
```

**Video Service:**
```bash
cd video-service
pytest

# With coverage
pytest --cov=app --cov-report=html

# Specific test
pytest tests/test_video_generation.py::test_generate_video_success
```

**Frontend:**
```bash
cd frontend
npm test

# Watch mode
npm test -- --watch

# E2E tests (Playwright)
npm run test:e2e
```

### Run Integration Tests

**End-to-end fitting flow:**
```bash
cd backend
npm run test:integration

# Run specific suite
npm run test:integration -- --grep "Fitting Creation"
```

**Example integration test (`backend/tests/integration/fitting.test.ts`):**
```typescript
import request from 'supertest';
import { app } from '../../src/app';
import { prisma } from '../../src/lib/prisma';

describe('Fitting Creation Flow', () => {
  let authToken: string;
  let shopId: string;

  beforeAll(async () => {
    // Login to get auth token
    const loginResponse = await request(app)
      .post('/v1/auth/login')
      .send({ email: 'owner@test.com', password: 'password123' });
    
    authToken = loginResponse.body.access_token;
    shopId = loginResponse.body.user.shop_id;
  });

  it('should create fitting and generate video', async () => {
    // 1. Create fitting
    const fittingResponse = await request(app)
      .post('/v1/fittings')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        shop_id: shopId,
        customer: {
          first_name: 'Integration',
          last_name: 'Test',
          email: 'integration@test.com',
        },
        fitting_type: 'driver',
      });

    expect(fittingResponse.status).toBe(201);
    const fittingId = fittingResponse.body.id;

    // 2. Add shots
    for (let i = 0; i < 10; i++) {
      await request(app)
        .post(`/v1/fittings/${fittingId}/shots`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          club_type: 'driver',
          ball_speed_mph: 140 + i,
          carry_distance_yards: 230 + i * 3,
          timestamp: new Date().toISOString(),
        });
    }

    // 3. Complete fitting
    const completeResponse = await request(app)
      .patch(`/v1/fittings/${fittingId}/complete`)
      .set('Authorization', `Bearer ${authToken}`)
      .send({});

    expect(completeResponse.status).toBe(200);
    expect(completeResponse.body.status).toBe('completed');

    // 4. Wait for video generation (poll for up to 60 seconds)
    let videoReady = false;
    for (let i = 0; i < 60; i++) {
      await new Promise((resolve) => setTimeout(resolve, 1000));
      
      const fittingStatus = await request(app)
        .get(`/v1/fittings/${fittingId}`)
        .set('Authorization', `Bearer ${authToken}`);

      if (fittingStatus.body.video?.status === 'ready') {
        videoReady = true;
        break;
      }
    }

    expect(videoReady).toBe(true);
  }, 90000); // 90 second timeout

  afterAll(async () => {
    await prisma.$disconnect();
  });
});
```

### Mock External Services

**Mock TrackMan API (`backend/tests/mocks/trackman.mock.ts`):**
```typescript
import nock from 'nock';

export function mockTrackManAPI() {
  nock('https://sandbox-api.trackman.com')
    .post('/v1/auth')
    .reply(200, {
      access_token: 'mock_token_abc123',
      expires_in: 86400,
    });

  nock('https://sandbox-api.trackman.com')
    .get('/v1/shots/recent')
    .query(true)
    .reply(200, {
      shots: [
        {
          shot_id: 'TM_20251128_140000_001',
          ball_speed_mph: 145.2,
          carry_distance_yards: 245.3,
          timestamp: new Date().toISOString(),
        },
      ],
    });
}
```

**Use in tests:**
```typescript
import { mockTrackManAPI } from './mocks/trackman.mock';

beforeEach(() => {
  mockTrackManAPI();
});
```

---

## TROUBLESHOOTING

### Common Issues

**Issue 1: Port already in use**

```bash
# Error: EADDRINUSE: address already in use :::3000

# Solution: Find and kill process
lsof -ti:3000 | xargs kill -9

# Or use different port
PORT=3001 npm run dev
```

**Issue 2: Database connection failed**

```bash
# Error: connect ECONNREFUSED 127.0.0.1:5432

# Check if PostgreSQL is running
docker compose ps postgres

# Or locally
brew services list | grep postgresql

# Restart
docker compose restart postgres
# OR
brew services restart postgresql@15
```

**Issue 3: Prisma Client out of sync**

```bash
# Error: Prisma schema and generated client are out of sync

# Solution: Regenerate Prisma Client
cd backend
npx prisma generate
```

**Issue 4: FFmpeg not found (video service)**

```bash
# Error: FFmpeg not found at /usr/local/bin/ffmpeg

# macOS
brew install ffmpeg

# Linux
sudo apt-get install ffmpeg

# Verify
ffmpeg -version
```

**Issue 5: Redis connection refused**

```bash
# Error: connect ECONNREFUSED 127.0.0.1:6379

# Check if Redis is running
redis-cli ping
# Expected: PONG

# If not running
docker compose up -d redis
# OR
brew services start redis
```

**Issue 6: LocalStack not responding**

```bash
# Check LocalStack status
curl http://localhost:4566/_localstack/health

# Restart LocalStack
docker compose restart localstack

# View logs
docker compose logs localstack
```

### Clear Cache & Reset

**Complete reset (nuclear option):**
```bash
# Stop all services
docker compose down -v

# Remove node_modules
rm -rf backend/node_modules frontend/node_modules

# Remove Python venv
rm -rf video-service/venv

# Reinstall
cd backend && npm install
cd ../frontend && npm install
cd ../video-service && python -m venv venv && source venv/bin/activate && pip install -r requirements.txt

# Restart everything
docker compose up -d
cd backend && npm run dev
```

### Enable Debug Logging

**Backend:**
```bash
LOG_LEVEL=debug npm run dev
```

**Video Service:**
```bash
LOG_LEVEL=DEBUG celery -A app.celery worker --loglevel=debug
```

**Frontend:**
```bash
VITE_DEBUG=true npm run dev
```

---

## IDE CONFIGURATION

### VS Code

**Recommended Extensions:**

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "prisma.prisma",
    "ms-python.python",
    "ms-python.vscode-pylance",
    "bradlc.vscode-tailwindcss",
    "ms-azuretools.vscode-docker"
  ]
}
```

**Settings (`.vscode/settings.json`):**
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[python]": {
    "editor.defaultFormatter": "ms-python.python",
    "editor.formatOnSave": true
  },
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": true,
  "python.formatting.provider": "black",
  "typescript.tsdk": "backend/node_modules/typescript/lib"
}
```

**Launch Configuration (`.vscode/launch.json`):**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Backend (Node.js)",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "cwd": "${workspaceFolder}/backend",
      "console": "integratedTerminal",
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "name": "Frontend (Vite)",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:5173",
      "webRoot": "${workspaceFolder}/frontend"
    },
    {
      "name": "Jest Tests",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/backend/node_modules/.bin/jest",
      "args": ["--runInBand", "--watchAll=false"],
      "cwd": "${workspaceFolder}/backend",
      "console": "integratedTerminal"
    }
  ]
}
```

### JetBrains (WebStorm / PyCharm)

**Run Configurations:**

1. **Backend (npm)**
   - Type: npm
   - Package.json: `backend/package.json`
   - Command: run
   - Scripts: dev

2. **Frontend (npm)**
   - Type: npm
   - Package.json: `frontend/package.json`
   - Command: run
   - Scripts: dev

3. **Tests (Jest)**
   - Type: Jest
   - Configuration file: `backend/jest.config.js`
   - Working directory: `backend`

---

## DEVELOPER WORKFLOW

### Daily Development Cycle

**1. Start work:**
```bash
# Pull latest changes
git pull origin development

# Start services
docker compose up -d
cd backend && npm run dev  # Terminal 1
cd video-service && celery worker ...  # Terminal 2
cd frontend && npm run dev  # Terminal 3
```

**2. Make changes:**
- Edit code in your IDE
- Hot reload automatically reflects changes
- Run tests: `npm test` (backend), `pytest` (video-service)

**3. Commit:**
```bash
# Stage changes
git add .

# Commit with conventional commit message
git commit -m "feat(fittings): add shot validation logic"

# Push to feature branch
git push origin feature/shot-validation
```

**4. Create Pull Request:**
- Open PR on GitHub
- CI/CD runs automated tests
- Request code review
- Merge to `development` after approval

**5. End of day:**
```bash
# Stop services
docker compose down

# Keep services running (optional)
# docker compose up -d
```

### Git Workflow

**Branch Strategy:**
- `main` - Production (tagged releases)
- `staging` - Pre-production testing
- `development` - Active development (default)
- `feature/*` - Feature branches
- `bugfix/*` - Bug fixes
- `hotfix/*` - Production hotfixes

**Commit Convention:**
```
feat(scope): description
fix(scope): description
docs(scope): description
test(scope): description
chore(scope): description
```

---

## NEXT STEPS

**After setup, proceed to:**

1. **Read PRD:** Understand product requirements
2. **Read Feature Specs:** Understand specific features
3. **Read API Reference:** Understand API contracts
4. **Review Database Schema:** Understand data model
5. **Run Tests:** Ensure everything works
6. **Build First Feature:** Follow feature specs

**Helpful Commands:**

```bash
# View all available npm scripts
cd backend && npm run

# View database schema
cd backend && npx prisma studio

# Generate API documentation
cd backend && npm run docs:generate

# Lint code
cd backend && npm run lint
cd frontend && npm run lint

# Format code
cd backend && npm run format
cd frontend && npm run format
```

---

## SUPPORT

**Questions?**
- Slack: #dev-support
- Email: dev-team@fitclip.golf
- Wiki: https://wiki.fitclip.golf

**Onboarding Checklist:**
- [ ] Repository cloned
- [ ] Dependencies installed
- [ ] Docker Compose running
- [ ] Database seeded
- [ ] Backend API running
- [ ] Video service running
- [ ] Frontend running
- [ ] Tests passing
- [ ] IDE configured
- [ ] First commit pushed

**Welcome to FitClip! Happy coding! 🏌️‍♂️⛳️**

