# FitClip Technical Architecture

**Version:** 1.0  
**Last Updated:** November 28, 2025  
**Status:** Production Ready  
**Architecture Owner:** Technical Lead

---

## TABLE OF CONTENTS

1. [System Overview](#system-overview)
2. [Technology Stack](#technology-stack)
3. [Architecture Diagrams](#architecture-diagrams)
4. [Infrastructure Components](#infrastructure-components)
5. [Data Architecture](#data-architecture)
6. [Integration Architecture](#integration-architecture)
7. [Security Architecture](#security-architecture)
8. [Scalability & Performance](#scalability--performance)
9. [Monitoring & Observability](#monitoring--observability)
10. [Disaster Recovery](#disaster-recovery)

---

## 1. SYSTEM OVERVIEW

### High-Level Architecture

FitClip is a cloud-native SaaS platform built on AWS with a microservices-oriented architecture. The system consists of:

- **Client Layer:** React SPAs for shop dashboard and customer portal
- **API Gateway:** Node.js Express API with JWT authentication
- **Business Logic Layer:** Node.js services + Python ML services
- **Data Layer:** PostgreSQL (primary), Redis (cache), S3 (storage)
- **Integration Layer:** Launch monitor APIs, video processing, notifications
- **Infrastructure:** AWS ECS (containers), CloudFront (CDN), Route 53 (DNS)

### Architecture Principles

1. **Cloud-First:** AWS managed services preferred over self-hosted
2. **Stateless Services:** All services can scale horizontally
3. **Event-Driven:** Async processing for video generation, notifications
4. **API-First:** All features exposed via REST APIs
5. **Security by Design:** Encryption at rest and in transit, least privilege access
6. **Fail Fast:** Circuit breakers, retries, graceful degradation
7. **Observable:** Comprehensive logging, metrics, distributed tracing

---

## 2. TECHNOLOGY STACK

### Frontend Stack

**Shop Dashboard & Customer Portal:**
- **Framework:** React 18.2+ (TypeScript)
- **State Management:** Redux Toolkit + RTK Query
- **UI Library:** Material-UI (MUI) v5
- **Routing:** React Router v6
- **Forms:** React Hook Form + Zod validation
- **Video Player:** Video.js (HLS adaptive streaming)
- **Real-Time:** Socket.IO client (WebSocket)
- **Build Tool:** Vite (fast HMR, modern bundling)
- **Testing:** Jest + React Testing Library + Playwright

**FitClip Agent (Shop-Side Desktop App):**
- **Framework:** Electron 28+ (Node.js + Chromium)
- **Purpose:** Connect to launch monitors on shop's local network
- **Auto-Update:** electron-updater (silent background updates)

### Backend Stack

**API Server:**
- **Runtime:** Node.js 20 LTS
- **Framework:** Express.js 4.18+
- **Language:** TypeScript 5.3+
- **Authentication:** JWT (jsonwebtoken), bcrypt for passwords
- **Validation:** Zod schemas
- **ORM:** Prisma 5.7+ (type-safe database access)
- **WebSocket:** Socket.IO (real-time shot updates)
- **API Docs:** OpenAPI 3.0 spec + Swagger UI

**Video Processing Service:**
- **Runtime:** Python 3.11+
- **Framework:** FastAPI (async Python web framework)
- **Video Engine:** FFmpeg 6.0+ (via ffmpeg-python wrapper)
- **ML Library:** XGBoost 2.0+ (highlight selection)
- **Task Queue:** Celery + Redis (async job processing)
- **Storage:** AWS S3 SDK (boto3)

**Background Workers:**
- **Node.js Workers:** BullMQ (Redis-backed job queue)
  - TrackMan polling (every 2 seconds per active fitting)
  - Token refresh (every 23 hours)
  - Email/SMS sending (async)
- **Python Workers:** Celery
  - Video rendering (CPU-intensive)
  - ML model inference (highlight selection)

### Database & Storage

**Primary Database:**
- **Type:** PostgreSQL 15.5 (AWS RDS)
- **Instance:** db.t4g.medium (2 vCPU, 4 GB RAM) for MVP
- **Storage:** 100 GB SSD (gp3) with auto-scaling to 500 GB
- **Backups:** Automated daily snapshots, 30-day retention
- **Read Replicas:** 1 replica for analytics queries (post-MVP)

**Cache Layer:**
- **Type:** Redis 7.2 (AWS ElastiCache)
- **Instance:** cache.t4g.micro (0.5 GB RAM) for MVP
- **Use Cases:**
  - Session storage (JWT refresh tokens)
  - Rate limiting (API throttling)
  - Job queues (BullMQ, Celery)
  - Frequently accessed data (shop settings, user profiles)

**Object Storage:**
- **Type:** AWS S3
- **Buckets:**
  - `fitclip-videos-prod` - Rendered videos (Standard tier)
  - `fitclip-videos-archive` - Videos >2 years old (Glacier)
  - `fitclip-assets-prod` - Shop logos, branding assets
  - `fitclip-backups-prod` - Database backups
- **CDN:** CloudFront distribution (200+ edge locations)
- **Lifecycle:** Auto-archive to Glacier after 2 years

### Infrastructure & DevOps

**Container Orchestration:**
- **Service:** AWS ECS (Elastic Container Service) with Fargate
- **Why ECS over EKS:** Simpler for small team, less operational overhead
- **Container Registry:** AWS ECR (private Docker images)
- **Scaling:** Auto-scaling based on CPU/memory thresholds

**CI/CD Pipeline:**
- **Source Control:** GitHub (private repos)
- **CI/CD:** GitHub Actions
- **Deployment:** Blue-green deployments to ECS
- **Infrastructure as Code:** Terraform 1.6+

**Monitoring & Logging:**
- **APM:** Datadog (application performance monitoring)
- **Error Tracking:** Sentry (exception monitoring)
- **Uptime:** Pingdom (synthetic monitoring)
- **Alerts:** PagerDuty (on-call rotation)
- **Logs:** CloudWatch Logs → Datadog (centralized)

### External Services

**Third-Party Integrations:**
- **Launch Monitors:** TrackMan, Foresight, FlightScope APIs
- **Email:** SendGrid (transactional email, 100K/month free tier)
- **SMS:** Twilio (programmable SMS, $0.0075/message)
- **Video Transcoding:** AWS MediaConvert (pay-per-minute)
- **Analytics:** Segment (customer data platform) → Mixpanel
- **Payments:** Stripe (subscription billing, PCI-compliant)
- **Auth (Future):** Auth0 or AWS Cognito (SSO, 2FA)

---

## 3. ARCHITECTURE DIAGRAMS

### 3.1 System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         EXTERNAL USERS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐        │
│  │ Golf Shop    │     │ Fitter       │     │ Golfer       │        │
│  │ Owner        │     │ (iPad)       │     │ (Mobile)     │        │
│  │ (Desktop)    │     │              │     │              │        │
│  └───────┬──────┘     └──────┬───────┘     └──────┬───────┘        │
│          │                   │                    │                 │
└──────────┼───────────────────┼────────────────────┼─────────────────┘
           │                   │                    │
           │                   │                    │
┌──────────▼───────────────────▼────────────────────▼─────────────────┐
│                    CLOUDFRONT CDN (HTTPS)                            │
│  - SSL/TLS Termination                                               │
│  - Static Asset Caching (videos, images, JS/CSS)                    │
│  - DDoS Protection (AWS Shield)                                      │
└──────────┬───────────────────┬────────────────────┬─────────────────┘
           │                   │                    │
    ┌──────▼──────┐     ┌──────▼──────┐     ┌──────▼──────┐
    │  React SPA  │     │  React SPA  │     │  React SPA  │
    │  (Shop      │     │  (Fitter    │     │  (Customer  │
    │  Dashboard) │     │  App)       │     │  Portal)    │
    └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
           │                   │                    │
           └───────────────────┴────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────────┐
│                      APPLICATION LOAD BALANCER                       │
│  - Health Checks                                                     │
│  - SSL Offloading                                                    │
│  - Rate Limiting (AWS WAF)                                           │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────────┐
│                   ECS CLUSTER (API SERVICES)                         │
│                                                                       │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │  API Service (Node.js + Express)                            │    │
│  │  - 3 tasks (containers) for high availability               │    │
│  │  - Auto-scaling: 3-10 tasks based on CPU                    │    │
│  │  - Health endpoint: GET /health                              │    │
│  │                                                              │    │
│  │  Responsibilities:                                           │    │
│  │  - REST API endpoints (shops, fittings, shots, videos)      │    │
│  │  - JWT authentication & authorization                        │    │
│  │  - WebSocket connections (Socket.IO)                        │    │
│  │  - Business logic (non-ML, non-video)                       │    │
│  └───────┬────────────────────────────────────────────────────┘    │
│          │                                                           │
│  ┌───────▼────────────────────────────────────────────────────┐    │
│  │  Video Service (Python + FastAPI)                           │    │
│  │  - 2 tasks (containers) for video processing                │    │
│  │  - Auto-scaling: 2-5 tasks based on queue depth             │    │
│  │                                                              │    │
│  │  Responsibilities:                                           │    │
│  │  - Video generation (FFmpeg)                                 │    │
│  │  - ML highlight selection (XGBoost)                         │    │
│  │  - S3 upload & CloudFront invalidation                      │    │
│  └───────┬────────────────────────────────────────────────────┘    │
│          │                                                           │
└──────────┼───────────────────────────────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────────────────────────────┐
│                       DATA & CACHE LAYER                              │
│                                                                        │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │  PostgreSQL      │  │  Redis           │  │  S3 Buckets      │  │
│  │  (RDS)           │  │  (ElastiCache)   │  │                  │  │
│  │                  │  │                  │  │  - Videos        │  │
│  │  - Shops         │  │  - Sessions      │  │  - Assets        │  │
│  │  - Customers     │  │  - Job Queues    │  │  - Backups       │  │
│  │  - Fittings      │  │  - Rate Limits   │  │                  │  │
│  │  - Shots         │  │  - Cache         │  │  Lifecycle:      │  │
│  │  - Videos        │  │                  │  │  Standard→Glacier│  │
│  │  - Analytics     │  │  TTL: 1-24 hrs   │  │  after 2 years   │  │
│  │                  │  │                  │  │                  │  │
│  │  Replicas: 1     │  │  Cluster: No     │  │  Versioning: On  │  │
│  │  (read-only)     │  │  (single node)   │  │                  │  │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘  │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                     BACKGROUND WORKERS                                  │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────┐     │
│  │  Node.js Workers (BullMQ)                                     │     │
│  │  - TrackMan Polling Worker (every 2s per fitting)            │     │
│  │  - Token Refresh Worker (every 23h)                          │     │
│  │  - Email Worker (async via SendGrid)                         │     │
│  │  - SMS Worker (async via Twilio)                             │     │
│  │  - Analytics Worker (batch events to Segment)                │     │
│  └──────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────┐     │
│  │  Python Workers (Celery)                                      │     │
│  │  - Video Rendering Worker (CPU-intensive)                    │     │
│  │  - ML Inference Worker (highlight selection)                 │     │
│  │  - Batch Processing Worker (nightly jobs)                    │     │
│  └──────────────────────────────────────────────────────────────┘     │
│                                                                          │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                     EXTERNAL INTEGRATIONS                               │
│                                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │ TrackMan    │  │ Foresight   │  │ SendGrid    │  │ Twilio      │  │
│  │ API         │  │ API         │  │ (Email)     │  │ (SMS)       │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │
│                                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │ Stripe      │  │ Segment     │  │ Datadog     │  │ Sentry      │  │
│  │ (Payments)  │  │ (Analytics) │  │ (Monitoring)│  │ (Errors)    │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │
│                                                                          │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                      MONITORING & OBSERVABILITY                         │
│                                                                          │
│  - Datadog APM (distributed tracing)                                   │
│  - CloudWatch Logs (centralized logging)                               │
│  - Pingdom (uptime monitoring)                                         │
│  - PagerDuty (incident management)                                     │
│  - Sentry (error tracking)                                             │
│                                                                          │
└────────────────────────────────────────────────────────────────────────┘
```

---

### 3.2 Fitting Session Data Flow

```
SHOP NETWORK                    FITCLIP CLOUD                    CUSTOMER
─────────────────────────────────────────────────────────────────────────

┌──────────────┐
│ TrackMan     │
│ Launch       │
│ Monitor      │
└──────┬───────┘
       │ (1) Shot data via API
       ├─────────────────┐
       │                 │
┌──────▼───────┐         │
│ FitClip      │         │
│ Agent        │         │
│ (Electron)   │         │
│              │         │
│ - Polls TM   │         │
│   every 2s   │         │
│ - Buffers    │         │
│   offline    │         │
└──────┬───────┘         │
       │                 │
       │ (2) WebSocket   │
       │     connection  │
       │                 │
       ▼                 ▼
   ┌────────────────────────┐
   │  API Gateway (ALB)     │
   │  - Authenticate        │
   │  - Rate limit          │
   └──────────┬─────────────┘
              │
              │ (3) Store shots
              ▼
   ┌────────────────────────┐
   │  API Service           │
   │  (Node.js)             │
   │                        │
   │  POST /fittings/:id/   │
   │       shots            │
   └──────────┬─────────────┘
              │
              │ (4) INSERT INTO shots
              ▼
   ┌────────────────────────┐
   │  PostgreSQL (RDS)      │
   │                        │
   │  shots table           │
   │  - fitting_id          │
   │  - ball_speed          │
   │  - carry_distance      │
   │  - timestamp           │
   └────────────────────────┘
              │
              │ (5) Real-time update
              ▼
   ┌────────────────────────┐
   │  Socket.IO             │
   │  (WebSocket)           │
   │                        │
   │  Emit: "new_shot"      │
   └──────────┬─────────────┘
              │
              │ (6) Display shot
              ▼
   ┌────────────────────────┐                ┌────────────────┐
   │  Fitter iPad App       │                │  Fitter        │
   │  (React)               │───────────────▶│  sees shot     │
   │                        │  Live update    │  instantly     │
   │  Shot list updates     │                └────────────────┘
   └────────────────────────┘

   [Fitting continues... 30-60 shots captured]

   ┌────────────────────────┐
   │  Fitter taps           │
   │  "Complete Fitting"    │
   └──────────┬─────────────┘
              │
              │ (7) PATCH /fittings/:id/complete
              ▼
   ┌────────────────────────┐
   │  API Service           │
   │  - Update status       │
   │  - Trigger video job   │
   └──────────┬─────────────┘
              │
              │ (8) Enqueue job
              ▼
   ┌────────────────────────┐
   │  Redis (Job Queue)     │
   │                        │
   │  Job: generate_video   │
   │  Payload: fitting_id   │
   └──────────┬─────────────┘
              │
              │ (9) Pick up job
              ▼
   ┌────────────────────────┐
   │  Python Worker         │
   │  (Celery)              │
   │                        │
   │  1. Fetch shots        │
   │  2. ML highlight       │
   │     selection          │
   │  3. Render video       │
   │  4. Upload to S3       │
   └──────────┬─────────────┘
              │
              │ (10) Video file
              ▼
   ┌────────────────────────┐
   │  S3 + CloudFront       │
   │                        │
   │  Video URL:            │
   │  cdn.fitclip.golf/     │
   │  videos/abc123.mp4     │
   └──────────┬─────────────┘
              │
              │ (11) Send notifications
              ▼
   ┌────────────────────────┐
   │  Notification Workers  │
   │                        │
   │  - SMS (Twilio)        │
   │  - Email (SendGrid)    │
   └──────────┬─────────────┘
              │
              │ (12) Video link
              ▼
                                                 ┌────────────────┐
                                                 │  Golfer        │
                                                 │  receives SMS  │
                                                 │  & email       │
                                                 │                │
                                                 │  "Your video   │
                                                 │  is ready!"    │
                                                 └───────┬────────┘
                                                         │
                                                         │ (13) Click link
                                                         ▼
                                                 ┌────────────────┐
                                                 │  Customer      │
                                                 │  Portal        │
                                                 │  (React)       │
                                                 │                │
                                                 │  Video player  │
                                                 │  + Buy Now CTA │
                                                 └────────────────┘
```

**Timeline:**
- Steps 1-6: Real-time (sub-5 seconds per shot)
- Steps 7-10: Video generation (15-30 minutes)
- Steps 11-13: Notification delivery (<1 minute)

**Total:** Customer receives video within 30 minutes of fitting completion

---

### 3.3 Video Generation Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                    VIDEO GENERATION WORKFLOW                         │
└─────────────────────────────────────────────────────────────────────┘

Trigger: PATCH /fittings/:id/complete
│
├─ (1) API Service validates fitting is complete
│      - Check status = "in_progress"
│      - Check shot_count > 0
│      - Update status = "completed"
│
├─ (2) Enqueue video job in Redis
│      Job ID: video_job_abc123
│      Payload: { fitting_id, shop_id, customer_id }
│      Priority: normal (high for enterprise customers)
│
└─▶ Celery Worker picks up job

┌─────────────────────────────────────────────────────────────────────┐
│               STEP 1: FETCH DATA (5 seconds)                         │
└─────────────────────────────────────────────────────────────────────┘

Python Worker:
├─ Query PostgreSQL for fitting data
│  SELECT * FROM fittings WHERE id = :fitting_id
│  SELECT * FROM shots WHERE fitting_id = :fitting_id AND is_valid = TRUE
│  SELECT * FROM shops WHERE id = :shop_id (for branding)
│
├─ Fetch shop branding assets from S3
│  - Logo (PNG, 512x512)
│  - Brand colors (hex codes)
│  - Intro/outro templates
│
└─ Result: fitting_data = { customer, shots[], shop_branding }

┌─────────────────────────────────────────────────────────────────────┐
│         STEP 2: ML HIGHLIGHT SELECTION (10 seconds)                  │
└─────────────────────────────────────────────────────────────────────┘

ML Service (XGBoost):
├─ Load model from disk
│  model = xgb.Booster('highlight_selector_v1.model')
│
├─ Score each shot (0-100)
│  for shot in shots:
│      shot['score'] = calculate_shot_quality_score(shot)
│
├─ Select 7 highlights
│  highlights = select_highlights(shots, num=7)
│  - Longest shot
│  - Straightest shot
│  - Best overall
│  - Biggest improvement
│  - 3 high-quality diverse shots
│
└─ Result: highlight_shots = [shot1, shot2, ..., shot7]

┌─────────────────────────────────────────────────────────────────────┐
│            STEP 3: VIDEO RENDERING (8-20 minutes)                    │
└─────────────────────────────────────────────────────────────────────┘

FFmpeg Pipeline:
│
├─ (3a) Generate intro (3 seconds)
│       - Shop logo fade-in
│       - Text: "Your Personalized Fitting Recap"
│       - Brand colors overlay
│
│       ffmpeg -loop 1 -i logo.png -i intro_audio.mp3 \
│              -vf "fade=in:0:30" -t 3 intro.mp4
│
├─ (3b) Generate shot clips (7 clips × 15-20 seconds each)
│       For each highlight:
│       - Fetch swing footage (if available from TrackMan cameras)
│       - OR generate static frame with club image
│       - Overlay data graphics (ball speed, carry distance, spin)
│       - Add text: "Shot 1: Driver - 245 yards"
│
│       ffmpeg -i swing_footage.mp4 -i overlay.png \
│              -filter_complex "[0:v][1:v]overlay=10:10" \
│              -t 15 shot_1.mp4
│
├─ (3c) Generate before/after comparison (30 seconds)
│       - Split screen: baseline shot | best test shot
│       - Metrics comparison table
│       - Green arrows showing improvement
│
│       ffmpeg -i baseline.mp4 -i best_shot.mp4 \
│              -filter_complex "[0:v][1:v]hstack" \
│              before_after.mp4
│
├─ (3d) Generate recommendations (20 seconds)
│       - Text: "Based on your fitting..."
│       - Club recommendations with images
│       - "Buy Now" CTA button overlay
│
├─ (3e) Generate outro (5 seconds)
│       - Shop contact info
│       - Social media icons
│       - "Share Your Results" prompt
│
└─ (3f) Concatenate all clips
        ffmpeg -f concat -i filelist.txt -c copy final_video.mp4

        Encoding settings:
        - Resolution: 1080p (1920×1080)
        - Frame rate: 60fps (smooth swing footage)
        - Codec: H.264 (widely compatible)
        - Bitrate: 5 Mbps (good quality, reasonable file size)
        - Audio: AAC 128 kbps (background music + narration)

        Output: final_video.mp4 (size: 30-50 MB)

┌─────────────────────────────────────────────────────────────────────┐
│              STEP 4: UPLOAD TO S3 (1-3 minutes)                      │
└─────────────────────────────────────────────────────────────────────┘

Python Worker:
├─ Upload to S3 bucket
│  s3.upload_file(
│      'final_video.mp4',
│      bucket='fitclip-videos-prod',
│      key=f'videos/{fitting_id}/recap.mp4',
│      ExtraArgs={'ContentType': 'video/mp4'}
│  )
│
├─ Generate thumbnail (first frame)
│  ffmpeg -i final_video.mp4 -ss 00:00:01 -vframes 1 thumbnail.jpg
│  s3.upload_file('thumbnail.jpg', ...)
│
├─ Update database
│  UPDATE videos SET
│      status = 'ready',
│      video_url = 'https://cdn.fitclip.golf/videos/.../recap.mp4',
│      thumbnail_url = '...thumbnail.jpg',
│      duration_seconds = 180,
│      file_size_mb = 42.3,
│      generated_at = NOW()
│  WHERE fitting_id = :fitting_id
│
└─ Invalidate CloudFront cache
   cloudfront.create_invalidation(paths=['/videos/.../*'])

┌─────────────────────────────────────────────────────────────────────┐
│         STEP 5: SEND NOTIFICATIONS (<1 minute)                       │
└─────────────────────────────────────────────────────────────────────┘

Notification Workers:
│
├─ SMS (Twilio)
│  twilio.messages.create(
│      to=customer.phone,
│      from_='+14155550100',
│      body='Tom, your FitClip video is ready! Watch now: https://...'
│  )
│
├─ Email (SendGrid)
│  sendgrid.send(
│      to=customer.email,
│      subject='Your Golf Fitting Video is Ready!',
│      template_id='video_ready_v2',
│      data={
│          customer_name: 'Tom',
│          video_url: 'https://...',
│          thumbnail_url: 'https://...',
│          shop_name: 'Precision Golf Lab'
│      }
│  )
│
└─ Analytics event
   segment.track('video_sent', {
       fitting_id, customer_id, shop_id,
       video_duration: 180, generation_time: 1247
   })

┌─────────────────────────────────────────────────────────────────────┐
│                 TOTAL TIME: 15-30 MINUTES                            │
└─────────────────────────────────────────────────────────────────────┘

Performance targets:
- p50 (median): 18 minutes
- p95: 28 minutes
- p99: 30 minutes
- SLA: 99% of videos delivered <30 minutes
```

---

## 4. INFRASTRUCTURE COMPONENTS

### 4.1 AWS Account Structure

**Environment Strategy:**
- **Production (prod):** Customer-facing, high availability
- **Staging (staging):** Pre-production testing, mirrors prod
- **Development (dev):** Developer sandbox, cost-optimized
- **CI/CD (ci):** GitHub Actions runners, ephemeral

**AWS Regions:**
- **Primary:** us-east-1 (N. Virginia) - lowest latency for East Coast US
- **DR Region:** us-west-2 (Oregon) - disaster recovery failover
- **Future:** eu-west-1 (Ireland) for European expansion

### 4.2 Compute (ECS Fargate)

**ECS Cluster: fitclip-prod**

**Service 1: API Service**
```yaml
Service Name: fitclip-api
Task Definition: fitclip-api:v42
Desired Count: 3 (min: 3, max: 10)
Launch Type: FARGATE
CPU: 1 vCPU (1024 units)
Memory: 2 GB
Network: awsvpc mode (ENI per task)

Container:
  Image: 123456789.dkr.ecr.us-east-1.amazonaws.com/fitclip-api:latest
  Port: 3000
  Environment:
    NODE_ENV: production
    DATABASE_URL: postgresql://...
    REDIS_URL: redis://...
    JWT_SECRET: <from Secrets Manager>
  
Health Check:
  Command: curl -f http://localhost:3000/health || exit 1
  Interval: 30s
  Timeout: 5s
  Retries: 3

Auto Scaling:
  Metric: CPU Utilization
  Target: 70%
  Scale Out: +2 tasks when >70% for 2 minutes
  Scale In: -1 task when <30% for 5 minutes
```

**Service 2: Video Service**
```yaml
Service Name: fitclip-video
Task Definition: fitclip-video:v28
Desired Count: 2 (min: 2, max: 5)
Launch Type: FARGATE
CPU: 2 vCPU (2048 units) - video rendering is CPU-intensive
Memory: 4 GB

Container:
  Image: 123456789.dkr.ecr.us-east-1.amazonaws.com/fitclip-video:latest
  Port: 8000
  Environment:
    CELERY_BROKER_URL: redis://...
    S3_BUCKET: fitclip-videos-prod
    FFMPEG_THREADS: 2
  
Auto Scaling:
  Metric: Custom (Redis queue depth)
  Target: <10 jobs in queue
  Scale Out: +1 task when >10 jobs
  Scale In: -1 task when <3 jobs for 10 minutes
```

### 4.3 Database (RDS PostgreSQL)

**Instance: fitclip-prod-db**
```yaml
Engine: postgres
Version: 15.5
Instance Class: db.t4g.medium (2 vCPU, 4 GB RAM)
Storage: 100 GB SSD (gp3, 3000 IOPS)
Multi-AZ: Yes (automatic failover)
Backup: Automated daily snapshots, 30-day retention
Maintenance Window: Sunday 3-4 AM EST
Encryption: Yes (AES-256, AWS KMS)

Read Replica: fitclip-prod-db-read (same specs, us-east-1b)
  Purpose: Analytics queries, dashboard reporting
  Replication Lag: <1 second

Connection Pooling: PgBouncer (max 100 connections)
```

**Scaling Plan:**
- **Month 1-6 (MVP):** db.t4g.medium (sufficient for 5,000 fittings/month)
- **Month 7-12:** Upgrade to db.t4g.large (8 GB RAM) at 20,000 fittings/month
- **Year 2:** db.r6g.xlarge (32 GB RAM) for 100,000 fittings/month

### 4.4 Cache (ElastiCache Redis)

**Cluster: fitclip-prod-redis**
```yaml
Engine: redis
Version: 7.2
Node Type: cache.t4g.micro (0.5 GB RAM) for MVP
Nodes: 1 (single node, no cluster mode)
Encryption: In-transit (TLS), at-rest (AES-256)

Use Cases:
  - Session storage (JWT refresh tokens): TTL 30 days
  - Rate limiting: TTL 1 hour (per-shop API limits)
  - Job queues: BullMQ (Node.js), Celery (Python)
  - Cache: Frequently accessed data (shop settings): TTL 1 hour

Scaling Plan:
  - Year 1: cache.t4g.small (1.5 GB RAM) at 100 shops
  - Year 2: cache.r6g.large (13 GB RAM) + cluster mode at 500+ shops
```

### 4.5 Storage (S3 + CloudFront)

**S3 Buckets:**

**Bucket 1: fitclip-videos-prod**
```yaml
Purpose: Rendered videos (primary content)
Region: us-east-1
Versioning: Disabled (videos are immutable)
Lifecycle Policy:
  - After 90 days: Transition to Intelligent-Tiering
  - After 2 years: Transition to Glacier Flexible Retrieval
  - After 7 years: Delete (GDPR compliance)

Encryption: SSE-S3 (server-side encryption)
Access: Private (CloudFront only, via OAI)
Cost Estimate: 25 TB/year × $0.023/GB = $575/month
```

**Bucket 2: fitclip-assets-prod**
```yaml
Purpose: Shop logos, branding assets, static images
Lifecycle Policy: None (assets rarely change)
Access: Public-read (served via CloudFront)
```

**CloudFront Distribution:**
```yaml
Distribution ID: E1234567890ABC
Origin: fitclip-videos-prod.s3.amazonaws.com
Price Class: Use All Edge Locations (200+ POPs)
SSL/TLS: AWS Certificate Manager (ACM) cert for cdn.fitclip.golf
Cache Behavior:
  - Videos: Cache 7 days (most views happen in first 72 hours)
  - Assets: Cache 30 days (rarely change)
  - Invalidation: On-demand after video upload

Signed URLs: Yes (time-limited access, 7-day expiration)
  - Prevents hotlinking
  - Allows video access revocation (customer requests deletion)
```

### 4.6 Networking (VPC)

**VPC: fitclip-prod-vpc**
```yaml
CIDR: 10.0.0.0/16 (65,536 IP addresses)
Availability Zones: 3 (us-east-1a, us-east-1b, us-east-1c)

Subnets:
  Public Subnets (Internet-facing):
    - 10.0.1.0/24 (us-east-1a) - ALB, NAT Gateway
    - 10.0.2.0/24 (us-east-1b) - ALB, NAT Gateway
    - 10.0.3.0/24 (us-east-1c) - ALB, NAT Gateway
  
  Private Subnets (Internal services):
    - 10.0.11.0/24 (us-east-1a) - ECS tasks, RDS
    - 10.0.12.0/24 (us-east-1b) - ECS tasks, RDS replica
    - 10.0.13.0/24 (us-east-1c) - ECS tasks, ElastiCache

Internet Gateway: Attached to VPC (public subnet egress)
NAT Gateways: 3 (one per AZ for high availability)
Route Tables:
  - Public: 0.0.0.0/0 → Internet Gateway
  - Private: 0.0.0.0/0 → NAT Gateway (per AZ)

Security Groups:
  - alb-sg: Allow 443 from 0.0.0.0/0 (public internet)
  - api-sg: Allow 3000 from alb-sg
  - video-sg: Allow 8000 from alb-sg
  - rds-sg: Allow 5432 from api-sg, video-sg
  - redis-sg: Allow 6379 from api-sg, video-sg
```

---

## 5. DATA ARCHITECTURE

### 5.1 Database Schema (Complete)

See `DATABASE_MIGRATIONS.md` for full SQL definitions. Summary:

**Core Tables:**
- `shops` (300 rows at 500 shops)
- `users` (600 rows: shop owners + fitters)
- `customers` (300,000 rows at scale)
- `fittings` (600,000 rows/year)
- `shots` (36M rows/year: 60 shots × 600K fittings)
- `videos` (600,000 rows/year: 1 per fitting)
- `trackman_connections` (500 rows: 1 per shop)
- `analytics_events` (10M+ rows/year: clicks, views, purchases)

**Indexes:**
```sql
-- Critical for query performance
CREATE INDEX idx_shots_fitting_id ON shots(fitting_id);
CREATE INDEX idx_shots_highlight ON shots(fitting_id, is_highlight);
CREATE INDEX idx_videos_fitting_id ON videos(fitting_id);
CREATE INDEX idx_fittings_shop_date ON fittings(shop_id, completed_at);
CREATE INDEX idx_analytics_shop_event ON analytics_events(shop_id, event_type, timestamp);
```

**Partitioning Strategy (Year 2+):**
```sql
-- Partition shots table by month (reduce query scan size)
CREATE TABLE shots_2026_01 PARTITION OF shots
  FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
-- ... repeat for each month

-- Old partitions can be archived to S3 (pg_dump + DROP)
```

### 5.2 Data Retention Policies

| Data Type | Hot Storage | Warm Storage | Cold Storage | Deletion |
|-----------|-------------|--------------|--------------|----------|
| Fittings metadata | Forever | - | - | On customer request (GDPR) |
| Shots data | 2 years | - | S3 archive (Glacier) | After 7 years |
| Videos | 90 days (S3 Standard) | 2 years (Intelligent-Tiering) | 7 years (Glacier) | After 7 years or on request |
| Analytics events | 1 year (PostgreSQL) | 3 years (S3 Parquet) | - | After 3 years |
| Logs | 7 days (CloudWatch) | 90 days (S3) | - | After 90 days |

**GDPR "Right to Erasure":**
- Customer requests deletion via portal
- Soft delete: `customers.deleted_at = NOW()`
- Video deleted from S3 immediately
- Database records anonymized: name → "Deleted User", email → "deleted_X@privacy.local"
- Backups purged after 30 days (cannot restore deleted data)

---

## 6. INTEGRATION ARCHITECTURE

### 6.1 TrackMan Integration

**Integration Pattern:** Polling (TrackMan doesn't support webhooks)

**Architecture:**
```
FitClip Agent (Shop PC)  ←→  TrackMan Launch Monitor (LAN)
         │                           │
         │  (1) Poll every 2s        │
         │      GET /api/shots/recent│
         │ ◀─────────────────────────┘
         │
         │  (2) Buffer shots locally (SQLite)
         │      if network fails
         │
         │  (3) WebSocket (secure)
         ▼
FitClip API (Cloud)
         │
         │  (4) INSERT INTO shots
         ▼
PostgreSQL
```

**FitClip Agent (Electron App):**
- Installed on shop's Windows/Mac PC
- Lightweight (<50 MB disk, <100 MB RAM)
- Auto-updates silently in background
- Retry logic: Exponential backoff if API fails
- Offline queue: Stores up to 1,000 shots in SQLite

**TrackMan API Authentication:**
- OAuth 2.0 client credentials flow
- Token refresh every 24 hours
- Encrypted storage of refresh token (Windows: DPAPI, Mac: Keychain)

**Data Mapping:**
```javascript
// TrackMan API response → FitClip schema
{
  "TotalSpin": 2650,          → spin_rate
  "BallSpeed": 145.2,         → ball_speed_mph
  "LaunchDirection": -8.5,    → offline_distance_yards (negative = left)
  "CarryDistance": 245.3,     → carry_distance_yards
  // ... 30+ fields mapped
}
```

### 6.2 Foresight & FlightScope Integrations

**Foresight FSX API:**
- Similar architecture to TrackMan
- Uses HTTP REST API (not WebSocket)
- Poll interval: 3 seconds (slightly slower)

**FlightScope API:**
- Supports both polling and webhooks (ideal)
- Webhook endpoint: `POST /api/v1/webhooks/flightscope`
- Signature verification: HMAC-SHA256

**Multi-Launch-Monitor Support:**
- Shop can have multiple launch monitor types (e.g., TrackMan + Foresight)
- FitClip Agent detects which is active per bay
- Single unified shot data format internally

---

## 7. SECURITY ARCHITECTURE

### 7.1 Authentication & Authorization

**Authentication Flow:**
```
User (Shop Owner)
    │
    │ (1) POST /auth/login { email, password }
    ▼
API Server
    │
    │ (2) bcrypt.compare(password, hashed_password_from_db)
    │     Valid? ✓
    │
    │ (3) Generate JWT tokens
    │     - Access token (15 min expiration)
    │     - Refresh token (30 day expiration, stored in Redis)
    │
    ▼
User receives tokens
    │
    │ (4) Store access token in memory (React state)
    │     Store refresh token in httpOnly cookie
    │
    │ (5) All API requests: Authorization: Bearer <access_token>
    │
    ▼
API Server validates JWT
    │
    │ (6) If access token expired, use refresh token
    │     POST /auth/refresh
    │     Returns new access token
    │
    └─▶ Access granted
```

**Authorization (RBAC - Role-Based Access Control):**
```javascript
Roles:
  - shop_owner (full access to shop data, billing, settings)
  - shop_manager (view shop data, manage fitters)
  - fitter (create fittings, view own fittings only)
  - customer (view own videos only)

Permissions:
  shops.read      → All roles except customer
  shops.write     → shop_owner only
  fittings.create → shop_owner, shop_manager, fitter
  fittings.read   → fitter can only read own fittings
  videos.read     → customer can only read own videos

Enforcement:
  // Middleware on every API route
  async function checkPermission(req, res, next) {
    const user = req.user; // from JWT
    const resource = req.params.id;
    
    // Example: Fitter trying to access fitting
    if (user.role === 'fitter') {
      const fitting = await db.fittings.findById(resource);
      if (fitting.fitter_id !== user.id) {
        return res.status(403).json({ error: 'Forbidden' });
      }
    }
    
    next();
  }
```

### 7.2 Data Encryption

**In Transit:**
- TLS 1.3 for all API connections (HTTPS)
- CloudFront serves HTTPS only (HTTP → HTTPS redirect)
- Database connections: SSL/TLS (RDS force_ssl = true)
- Redis connections: TLS enabled

**At Rest:**
- S3: Server-side encryption (SSE-S3, AES-256)
- RDS: Encryption enabled (AWS KMS, AES-256)
- EBS volumes (for ECS tasks): Encrypted
- Secrets: AWS Secrets Manager (automatic rotation)

**Sensitive Data Encryption:**
```javascript
// TrackMan API keys, Stripe keys stored encrypted in DB
const crypto = require('crypto');

function encrypt(plaintext, key) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  const encrypted = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();
  return Buffer.concat([iv, tag, encrypted]).toString('base64');
}

function decrypt(ciphertext, key) {
  const buffer = Buffer.from(ciphertext, 'base64');
  const iv = buffer.slice(0, 16);
  const tag = buffer.slice(16, 32);
  const encrypted = buffer.slice(32);
  const decipher = crypto.createDecipheriv('aes-256-gcm', key, iv);
  decipher.setAuthTag(tag);
  return decipher.update(encrypted) + decipher.final('utf8');
}
```

### 7.3 API Security

**Rate Limiting:**
```javascript
// Per-shop rate limits (prevent abuse)
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 1000, // 1000 requests per 15 min per shop
  keyGenerator: (req) => req.user.shop_id,
  store: new RedisStore({ client: redisClient }),
  message: 'Too many requests, please try again later'
});

app.use('/api/v1/', apiLimiter);
```

**Input Validation:**
```typescript
// Zod schemas for all API inputs
import { z } from 'zod';

const createFittingSchema = z.object({
  shop_id: z.string().uuid(),
  customer: z.object({
    first_name: z.string().min(1).max(100),
    last_name: z.string().min(1).max(100),
    email: z.string().email(),
    phone: z.string().regex(/^\+?[1-9]\d{1,14}$/) // E.164 format
  }),
  fitting_type: z.enum(['driver', 'irons', 'full_bag', 'wedges'])
});

app.post('/api/v1/fittings', async (req, res) => {
  const result = createFittingSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error.issues });
  }
  // Proceed with validated data
});
```

**SQL Injection Prevention:**
```javascript
// Prisma ORM prevents SQL injection (parameterized queries)
const fitting = await prisma.fitting.create({
  data: {
    shop_id: shopId,
    customer_id: customerId,
    fitting_type: fittingType
  }
});

// NEVER do this:
// const query = `INSERT INTO fittings (shop_id) VALUES ('${shopId}')`;
```

**CORS Policy:**
```javascript
const cors = require('cors');

app.use(cors({
  origin: [
    'https://dashboard.fitclip.golf',
    'https://app.fitclip.golf',
    'https://customer.fitclip.golf'
  ],
  credentials: true, // Allow cookies (refresh token)
  methods: ['GET', 'POST', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

---

## 8. SCALABILITY & PERFORMANCE

### 8.1 Horizontal Scaling

**Auto-Scaling Triggers:**

**API Service (Node.js):**
- Scale out: CPU >70% for 2 minutes OR memory >80%
- Scale in: CPU <30% for 5 minutes
- Min: 3 tasks, Max: 10 tasks
- Cool down: 60 seconds (prevent flapping)

**Video Service (Python):**
- Scale out: Redis queue depth >10 jobs
- Scale in: Redis queue depth <3 jobs for 10 minutes
- Min: 2 tasks, Max: 5 tasks (video rendering is expensive)

**Database Read Replicas:**
- Add replicas when primary CPU >60% (read-heavy queries)
- Route analytics queries to read replicas (Prisma read replica support)

### 8.2 Caching Strategy

**Layer 1: CloudFront (Edge Cache)**
- Videos: 7-day TTL (most views in first 72 hours)
- Static assets: 30-day TTL
- Hit rate target: >80%

**Layer 2: Redis (Application Cache)**
```javascript
// Cache shop settings (change infrequently)
async function getShopSettings(shopId) {
  const cacheKey = `shop:${shopId}:settings`;
  let settings = await redis.get(cacheKey);
  
  if (!settings) {
    settings = await db.shops.findById(shopId);
    await redis.setex(cacheKey, 3600, JSON.stringify(settings)); // 1 hour TTL
  }
  
  return JSON.parse(settings);
}
```

**Layer 3: PostgreSQL Query Results**
```sql
-- Materialized views for analytics (refresh nightly)
CREATE MATERIALIZED VIEW shop_performance_daily AS
  SELECT
    shop_id,
    DATE(completed_at) as date,
    COUNT(*) as fitting_count,
    COUNT(*) FILTER (WHERE purchased = TRUE) as conversion_count,
    ROUND(COUNT(*) FILTER (WHERE purchased = TRUE)::NUMERIC / COUNT(*) * 100, 2) as conversion_rate
  FROM fittings
  WHERE completed_at > NOW() - INTERVAL '90 days'
  GROUP BY shop_id, DATE(completed_at);

-- Refresh nightly at 2 AM
REFRESH MATERIALIZED VIEW CONCURRENTLY shop_performance_daily;
```

### 8.3 Performance Benchmarks

**API Response Times (Target):**
| Endpoint | p50 | p95 | p99 |
|----------|-----|-----|-----|
| GET /fittings | 50ms | 150ms | 300ms |
| POST /fittings | 80ms | 200ms | 400ms |
| GET /analytics | 200ms | 500ms | 1000ms |
| POST /fittings/:id/shots | 30ms | 100ms | 200ms |

**Video Generation (Target):**
| Metric | Target | Actual (MVP) |
|--------|--------|--------------|
| p50 (median) | 18 min | 22 min |
| p95 | 28 min | 32 min |
| p99 | 30 min | 35 min |
| Success rate | 99.5% | 98.2% |

**Database Performance:**
- Read queries: <50ms (p95)
- Write queries: <100ms (p95)
- Connection pool saturation: <80%

---

## 9. MONITORING & OBSERVABILITY

### 9.1 Logging Strategy

**Log Levels:**
- **DEBUG:** Detailed info for troubleshooting (development only)
- **INFO:** General informational messages (request logs, job starts)
- **WARN:** Warning conditions (rate limit approaching, slow query)
- **ERROR:** Error events (API failures, exceptions)
- **FATAL:** Severe errors requiring immediate action (database down)

**Structured Logging (JSON):**
```javascript
const logger = require('pino')({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => { return { level: label }; }
  }
});

logger.info({
  msg: 'Fitting created',
  shop_id: 'abc123',
  fitting_id: 'def456',
  customer_email: 'tom@example.com',
  duration_ms: 145
});

// Output:
// {"level":"info","msg":"Fitting created","shop_id":"abc123",...,"time":1701234567890}
```

**Log Aggregation:**
- CloudWatch Logs (primary)
- Datadog (secondary, with advanced search/dashboards)
- Retention: 7 days (CloudWatch), 90 days (S3 archive)

### 9.2 Metrics & Dashboards

**Datadog Dashboards:**

**Dashboard 1: System Health**
- ECS task count (API, video)
- CPU utilization (per service)
- Memory utilization
- Network throughput
- RDS connection count
- Redis hit rate

**Dashboard 2: Business Metrics**
- Fittings created (per hour)
- Videos generated (per hour)
- Video generation time (p50, p95, p99)
- API error rate (%)
- TrackMan polling errors

**Dashboard 3: Customer Experience**
- Video watch rate (%)
- Video watch duration (seconds)
- "Buy Now" click rate
- Page load times (shop dashboard, customer portal)

**Custom Metrics:**
```javascript
// Datadog StatsD client
const StatsD = require('hot-shots');
const dogstatsd = new StatsD({ host: 'localhost', port: 8125 });

// Track video generation time
dogstatsd.histogram('video.generation_time', durationMs, {
  shop_id: shopId,
  video_quality: '1080p'
});

// Track business events
dogstatsd.increment('fitting.created', 1, { shop_id: shopId });
dogstatsd.increment('video.watched', 1, { customer_id: customerId });
```

### 9.3 Alerting

**PagerDuty Alert Rules:**

**P1 (Critical - Page On-Call Engineer):**
- API service down (all tasks unhealthy for >2 minutes)
- Database unreachable (connection failures >10 in 1 minute)
- Video generation failure rate >5% (10+ failures in 5 minutes)
- S3 upload errors >10 in 5 minutes

**P2 (High - Slack Alert):**
- API error rate >1% (100+ errors in 5 minutes)
- Video generation time >45 minutes (p95)
- RDS CPU >85% for 5 minutes
- Redis memory >90%

**P3 (Medium - Email Alert):**
- TrackMan API errors >5% (integration degradation)
- Email/SMS delivery failures >2%
- Slow database queries (>1 second) detected

**Alert Escalation:**
1. P1: Immediate PagerDuty page → On-call engineer has 15 min to acknowledge
2. P2: Slack #alerts channel → Team responds within 1 hour
3. P3: Email to engineering@ → Addressed in next business day

---

## 10. DISASTER RECOVERY

### 10.1 Backup Strategy

**Database Backups:**
- Automated daily snapshots (AWS RDS)
- Retention: 30 days
- Point-in-time recovery: Any moment in last 7 days
- Cross-region backup: Weekly snapshot copied to us-west-2

**S3 Backups:**
- Versioning enabled (recover deleted files within 30 days)
- Cross-region replication: fitclip-videos-prod → fitclip-videos-dr (us-west-2)
- Glacier backups: After 2 years (long-term retention)

**Code & Configuration:**
- GitHub (source code)
- Terraform state in S3 with versioning (infrastructure)
- Secrets in AWS Secrets Manager (automatic backup)

### 10.2 Recovery Objectives

**RTO (Recovery Time Objective):**
- Database: 1 hour (restore from snapshot)
- API service: 15 minutes (redeploy from Docker image)
- Videos: 4 hours (restore from S3 cross-region replica)

**RPO (Recovery Point Objective):**
- Database: <15 minutes (point-in-time recovery)
- Videos: 0 minutes (S3 replicated in real-time)

### 10.3 Disaster Recovery Runbook

**Scenario 1: Complete AWS us-east-1 Region Failure**

1. **Immediate (T+0 minutes):**
   - PagerDuty pages on-call engineer
   - Switch DNS (Route 53) to point to us-west-2
   - TTL: 60 seconds (customers cut over within 1 minute)

2. **Database Failover (T+15 minutes):**
   - Promote RDS read replica in us-west-2 to primary
   - Update connection strings in ECS task definitions

3. **Deploy Services (T+30 minutes):**
   - Pull Docker images from ECR (replicated to us-west-2)
   - Deploy ECS services in us-west-2 cluster
   - Scale to production capacity (3 API, 2 video tasks)

4. **Verify (T+60 minutes):**
   - Health checks passing
   - Smoke tests (create fitting, generate video)
   - Customer-facing services operational

**Total RTO: 1 hour**

---

## APPENDIX: TECHNOLOGY DECISION LOG

### Why Node.js for API?
- ✅ Excellent async performance (non-blocking I/O for API requests)
- ✅ JavaScript/TypeScript expertise common among devs
- ✅ Rich ecosystem (Express, Prisma, Socket.IO)
- ❌ Less ideal for CPU-intensive tasks (video rendering) → Python for that

### Why Python for Video Service?
- ✅ FFmpeg wrapper libraries mature (ffmpeg-python)
- ✅ ML ecosystem (XGBoost, scikit-learn) best-in-class
- ✅ Celery for distributed task queues
- ❌ Slower than Go/Rust but adequate for our use case

### Why PostgreSQL over MongoDB?
- ✅ ACID transactions (critical for billing, payments)
- ✅ Strong relational data (shops → fittings → shots)
- ✅ JSON support (JSONB) when needed
- ❌ MongoDB better for flexible schemas (not our use case)

### Why ECS over EKS?
- ✅ Simpler for small team (less operational overhead)
- ✅ Fargate = serverless containers (no EC2 management)
- ❌ EKS better for multi-cloud, complex orchestration (overkill for us)

### Why Stripe over Custom Payment Processing?
- ✅ PCI compliance handled by Stripe
- ✅ Subscription billing built-in
- ✅ 2.9% + $0.30 fee acceptable for our margins
- ❌ Custom processing = regulatory nightmare

---

**Document End**

For questions or clarifications, contact: architecture@fitclip.golf

