# FitClip Feature Specifications

**Document Version:** 1.0  
**Last Updated:** November 28, 2025  
**Status:** Ready for Development  
**Related Documents:** PRD.md, LANDING_PAGE_BLUEPRINT.md

---

## TABLE OF CONTENTS

1. [MVP Feature Specifications](#mvp-feature-specifications)
   - Feature 1: TrackMan Integration
   - Feature 2: AI Highlight Selection Engine
   - Feature 3: Automated Video Generation
   - Feature 4: Multi-Channel Video Delivery
   - Feature 5: Shop Analytics Dashboard
   - Feature 6: Shop Onboarding Flow

2. [Technical Implementation Guide](#technical-implementation-guide)
3. [API Documentation](#api-documentation)
4. [Database Schema Details](#database-schema-details)
5. [Development Setup Guide](#development-setup-guide)
6. [Testing Specifications](#testing-specifications)

---

## MVP FEATURE SPECIFICATIONS

---

## FEATURE 1: TRACKMAN INTEGRATION

### 1. Feature Overview

**Feature Name:** TrackMan Launch Monitor Integration  
**Feature Category:** Core Integration  
**Priority Level:** Must-Have (P0 - Blocker)  
**Target Release:** MVP v1.0 (Month 3)

**Problem Statement:**  
Golf shops using TrackMan cannot automatically capture fitting data without manual data entry. This creates friction, increases errors, and prevents real-time video generation. Shops need seamless, zero-touch data capture.

**Success Criteria:**
- 99.9% successful data capture rate during fittings
- Zero manual data entry required by fitters
- Sub-5 second latency from shot to FitClip database
- Works with all TrackMan models (TrackMan 4, TrackMan iO, TrackMan Range)

**User Impact:**
- **Primary:** Shop owners get automated data capture with zero setup friction
- **Secondary:** Fitters conduct sessions normally without workflow changes
- **Tertiary:** Golfers benefit from accurate, complete data in their videos

---

### 2. User Personas & Use Cases

**Primary Persona: Mike Chen - Golf Shop Owner**
- **Demographics:** 42 years old, owns 2 golf shops, uses TrackMan 4 at both locations
- **Goals:** Automate data capture, reduce staff workload, improve conversion rates
- **Pain Points:** Current PDF exports are manual, time-consuming, and error-prone
- **Context:** Wants to "set it and forget it" - FitClip should run invisibly in background

**Secondary Persona: Sarah Martinez - Master Fitter**
- **Demographics:** 34 years old, head fitter, conducts 20+ sessions per week
- **Goals:** Focus on customer experience, not data entry
- **Pain Points:** Constantly interrupted to export data or create reports
- **Context:** Needs seamless integration that doesn't slow down her fitting workflow

**Edge Cases:**
- TrackMan loses network connection mid-fitting
- Multiple bays running simultaneously (data attribution)
- Fitter forgets to "end session" on TrackMan
- Invalid shots (mis-hits, practice swings) need to be filtered
- Customer brings their own clubs (baseline data vs. tested clubs)

---

### 3. Detailed User Stories

**Epic:** As a golf shop owner, I want FitClip to automatically capture all TrackMan data during fittings so that my team doesn't waste time on manual data entry.

---

#### Story 1.1: Initial TrackMan Connection Setup

**As a** golf shop owner setting up FitClip for the first time  
**I want** to connect my TrackMan system to FitClip in under 10 minutes  
**So that** I can start generating videos immediately without technical hassle

**Acceptance Criteria:**

**Given** I have a TrackMan system with network connectivity  
**When** I enter my TrackMan API credentials in the FitClip onboarding flow  
**Then** FitClip authenticates successfully and displays "Connection Successful"  
**And** A test shot is captured to verify data flow  
**And** I see my TrackMan model and serial number displayed in FitClip

**Given** my TrackMan credentials are incorrect  
**When** I attempt to connect  
**Then** I see a clear error message: "Unable to connect. Please verify your TrackMan username and API key."  
**And** I am given a link to TrackMan's API documentation  
**And** I can retry without leaving the page

**Given** I have multiple TrackMan bays (multi-location or multi-bay shop)  
**When** I set up FitClip  
**Then** I can add multiple TrackMan connections (one per bay)  
**And** Each bay is labeled (e.g., "Bay 1 - Main Shop", "Bay 2 - Second Location")  
**And** Fitters can select which bay they're using when starting a fitting

**Definition of Done:**
- Shop owner completes setup in <10 minutes without support call
- Connection success rate >95% (some failures expected due to network issues)
- Clear error handling for all failure modes (wrong credentials, network down, TrackMan API unavailable)
- Multi-bay support tested with 3+ simultaneous fittings

---

#### Story 1.2: Real-Time Shot Data Capture During Fitting

**As a** fitter conducting a club fitting session  
**I want** FitClip to automatically capture every shot I take on TrackMan  
**So that** I can focus on my customer, not data entry

**Acceptance Criteria:**

**Given** I have started a fitting session in FitClip (via iPad app or desktop dashboard)  
**When** a customer hits a shot on TrackMan  
**Then** The shot data appears in FitClip within 5 seconds  
**And** The data includes: ball speed, club speed, launch angle, spin rate, carry distance, total distance, smash factor, offline distance  
**And** The shot is timestamped with UTC time  
**And** The shot is attributed to the correct club (driver, 7-iron, etc.)

**Given** a customer hits multiple shots in quick succession (<2 seconds apart)  
**When** TrackMan records both shots  
**Then** Both shots appear in FitClip as separate entries  
**And** No shots are dropped or duplicated  
**And** Shots appear in chronological order

**Given** TrackMan records an invalid shot (topped ball, air swing, etc.)  
**When** FitClip receives the data  
**Then** The shot is flagged as "invalid" based on rules:
  - Ball speed <50 mph = likely mis-hit
  - Smash factor <1.2 = poor contact
  - Carry distance <50 yards for driver = topped shot  
**And** Fitter can manually override flag if shot was actually good  
**And** Invalid shots are excluded from video highlight selection by default

**Given** the TrackMan loses network connection mid-fitting  
**When** shots are hit during the disconnection  
**Then** TrackMan stores shots locally (standard TrackMan behavior)  
**And** When connection resumes, FitClip pulls missing shots via API  
**And** Shop owner receives notification: "Network interruption detected. All data recovered."  
**And** Video generation is delayed until all data is synchronized

**Definition of Done:**
- 99.9% shot capture success rate in pilot testing
- Sub-5 second latency from shot to FitClip (measured via timestamp comparison)
- Invalid shot detection accuracy >90% (validated against human labeling)
- Network interruption recovery tested and verified
- Performance tested with 60+ shots in a single fitting (high-volume scenario)

---

#### Story 1.3: Session Management (Start/Stop Fitting)

**As a** fitter  
**I want** to clearly mark when a fitting session starts and ends  
**So that** shots are attributed to the correct customer and video

**Acceptance Criteria:**

**Given** I'm about to start a new club fitting  
**When** I open the FitClip app and tap "Start New Fitting"  
**Then** I'm prompted to enter:
  - Customer name
  - Customer email
  - Customer phone (SMS delivery)
  - Fitting type (driver, irons, full bag, wedges)  
**And** I can optionally add: handicap, current clubs, fitting notes  
**And** The form auto-saves as I type (no data loss)

**Given** I've started a fitting session  
**When** I begin hitting shots on TrackMan  
**Then** All shots are automatically associated with this fitting session  
**And** I see a live shot count: "15 shots captured so far"  
**And** I can see the last shot's key metrics (carry distance, ball speed) without leaving TrackMan screen

**Given** the fitting is complete  
**When** I tap "Complete Fitting" in FitClip  
**Then** I'm prompted to confirm: "Ready to generate video? 22 shots captured."  
**And** I can review shot list and remove any unwanted shots (practice swings, warm-ups)  
**And** When I confirm, video generation starts immediately  
**And** TrackMan session is marked as "complete" (no more shots attributed to this customer)

**Given** I forget to tap "Complete Fitting" and start a new customer  
**When** I tap "Start New Fitting" while a session is still active  
**Then** I'm prompted: "You have an active fitting for [Customer Name]. Complete it first?"  
**And** I can choose: "Complete Previous" or "Abandon Previous"  
**And** If abandoned, shots are saved but video is not generated (can be retrieved later)

**Definition of Done:**
- Session start/stop tested with 20+ fitting scenarios
- Auto-save prevents data loss in all cases (battery dies, app crashes, etc.)
- Clear UI feedback for active session (persistent banner: "Fitting in progress for Tom R.")
- Abandoned sessions recoverable from dashboard ("Draft Fittings" tab)

---

#### Story 1.4: Multi-Bay Support & Shot Attribution

**As a** golf shop owner with multiple TrackMan bays  
**I want** FitClip to correctly attribute shots to the right bay and customer  
**So that** we can run multiple fittings simultaneously without mixing up data

**Acceptance Criteria:**

**Given** I have 2 TrackMan bays (Bay 1 and Bay 2) connected to FitClip  
**When** Bay 1 fitter starts a fitting for "Customer A"  
**And** Bay 2 fitter starts a fitting for "Customer B" (simultaneously)  
**Then** Each fitter only sees their own customer's shots  
**And** Shots are correctly attributed to the right customer (no cross-contamination)  
**And** Shop dashboard shows 2 active fittings in real-time

**Given** both bays are active  
**When** a shot is hit in Bay 1  
**Then** Bay 1's FitClip app updates instantly  
**And** Bay 2's app is unaffected (no notification or UI change)  
**And** TrackMan serial number is used to route shots to correct bay

**Given** a fitter forgets to select which bay they're using  
**When** they start a fitting  
**Then** They're prompted: "Which bay are you using? [Bay 1] [Bay 2]"  
**And** The selection is remembered for future fittings (default bay per fitter)

**Definition of Done:**
- Multi-bay tested with up to 4 simultaneous fittings
- Zero cross-contamination of shot data (verified via serial number matching)
- Performance remains <5 second latency even with 4 active sessions
- Dashboard correctly displays all active fittings in real-time

---

### 4. Functional Requirements

**Core Functionality:**

**Function 1: TrackMan API Authentication**
- **Input:** TrackMan username, API key, TrackMan model (TrackMan 4, iO, Range)
- **Processing:**
  1. Send HTTPS POST request to TrackMan API: `https://api.trackman.com/v1/auth`
  2. Receive JWT token with 24-hour expiration
  3. Store encrypted token in FitClip database (AES-256)
  4. Set up token refresh job (every 23 hours)
- **Output:** "Connection Successful" message with TrackMan serial number displayed
- **Validation:**
  - Username must be valid email format
  - API key must be 32-character alphanumeric string
  - Test shot successfully retrieved within 30 seconds of connection

**Function 2: Real-Time Shot Data Polling**
- **Input:** Active fitting session ID, TrackMan serial number
- **Processing:**
  1. Poll TrackMan API every 2 seconds: `GET /v1/shots/recent`
  2. Compare timestamps to last retrieved shot (avoid duplicates)
  3. Parse JSON response and extract fields:
     ```json
     {
       "shot_id": "TM_20251128_143022_001",
       "timestamp": "2025-11-28T14:30:22.123Z",
       "club_speed_mph": 98.5,
       "ball_speed_mph": 145.2,
       "smash_factor": 1.47,
       "launch_angle_deg": 14.2,
       "spin_rate_rpm": 2650,
       "carry_distance_yards": 245.3,
       "total_distance_yards": 262.1,
       "offline_distance_yards": -8.5,
       "apex_height_yards": 32.1
     }
     ```
  4. Insert shot into FitClip `shots` table with `fitting_id` foreign key
  5. Emit WebSocket event to fitter's device: "New shot captured"
- **Output:** Shot appears in fitter's app within 5 seconds, displayed in shot list
- **Edge Cases:**
  - If API returns 429 (rate limit), back off to 5-second polling
  - If API returns 500 (server error), retry 3x with exponential backoff, then alert shop owner
  - If shot data is incomplete (missing ball speed), log warning but still save shot

**Function 3: Invalid Shot Detection & Flagging**
- **Input:** Shot data from TrackMan
- **Processing:**
  1. Apply validation rules:
     - `ball_speed < 50 mph` → Flag as "likely mis-hit"
     - `smash_factor < 1.2` → Flag as "poor contact"
     - `carry_distance < 50 yards` for driver → Flag as "topped"
     - `offline_distance > 50 yards` → Flag as "major slice/hook"
  2. Calculate confidence score (0-100%) that shot is valid
  3. Store flag in database: `is_valid = FALSE`, `invalid_reason = "poor contact"`
- **Output:** Shot displayed with orange badge "⚠️ Mis-hit" in fitter's app
- **Validation:** Fitter can tap shot and toggle "Actually Valid" checkbox

**Business Rules:**
- **Rule 1:** One TrackMan connection per bay (1:1 relationship)
- **Rule 2:** All shots must have a `fitting_id` (orphaned shots are logged but not used)
- **Rule 3:** Shots older than 90 days are archived to cold storage (Glacier) to save costs
- **Rule 4:** Shop owner must re-authenticate TrackMan every 90 days for security

**Integration Requirements:**
- **External APIs:**
  - TrackMan Cloud API v1 (primary)
  - Fallback: TrackMan CSV export if API unavailable
- **Internal Systems:**
  - Connects to FitClip video engine (provides shot data for video generation)
  - Connects to analytics service (tracks shots per fitting, average distances, etc.)
- **Data Synchronization:**
  - Real-time sync during fitting (2-second polling)
  - Batch sync every 24 hours to catch any missed shots (reconciliation job)

---

### 5. Non-Functional Requirements

**Performance Requirements:**
- **Response Time:** Shot appears in FitClip within 5 seconds of being hit (p95)
- **Throughput:** Support 100 simultaneous fittings across all shops (peak Saturday load)
- **Scalability:** System scales to 5,000 shops with 10,000+ TrackMan connections by Year 3
- **Resource Usage:**
  - API polling uses <10 MB/month bandwidth per shop
  - Database storage: ~1 KB per shot, 60 shots/fitting, 6,000 fittings/month = 360 MB/month

**Security Requirements:**
- **Authentication:** TrackMan API keys stored encrypted (AES-256) in database
- **Authorization:** Only shop owner and assigned fitters can view shot data
- **Data Protection:** HTTPS for all TrackMan API calls (TLS 1.3)
- **Audit Trail:** Log all TrackMan API calls with timestamp, shop_id, success/failure

**Usability Requirements:**
- **Accessibility:** TrackMan connection UI follows WCAG 2.1 AA (color blind friendly)
- **Browser Support:** Works on Chrome 90+, Safari 14+, Firefox 88+ (desktop)
- **Mobile Responsiveness:** Fitter app works on iPad Pro, iPad Air, iPad Mini (iOS 15+)
- **User Training:** Zero training required (onboarding wizard guides setup)

---

### 6. Technical Specifications

**Frontend Requirements:**

**Components:**
- `TrackManConnectionForm.tsx` - OAuth-style connection flow
- `FittingSessionManager.tsx` - Start/stop fitting UI
- `LiveShotFeed.tsx` - Real-time shot list with WebSocket updates
- `ShotDetailModal.tsx` - View/edit individual shot data

**State Management (Redux):**
```typescript
// Redux state shape
interface TrackManState {
  connections: TrackManConnection[];
  activeFitting: Fitting | null;
  liveShots: Shot[];
  isPolling: boolean;
  lastShotTimestamp: string;
}

// Actions
const trackmanActions = {
  connectTrackMan(credentials: TrackManCredentials),
  startFitting(customer: Customer, bayId: string),
  completeFitting(fittingId: string),
  addShot(shot: Shot),
  flagInvalidShot(shotId: string, reason: string)
};
```

**Routing:**
- `/dashboard/trackman/connect` - Initial setup
- `/dashboard/fittings/new` - Start fitting flow
- `/dashboard/fittings/:id` - View fitting details and shots

---

**Backend Requirements:**

**API Endpoints:**

```typescript
// TrackMan Connection Management
POST /api/v1/trackman/connect
{
  "shop_id": "uuid",
  "username": "shop@example.com",
  "api_key": "TM_xxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "model": "trackman_4",
  "bay_label": "Bay 1 - Main Shop"
}
Response: {
  "connection_id": "uuid",
  "status": "connected",
  "serial_number": "TM-123456",
  "expires_at": "2025-12-28T14:30:00Z"
}

// Start Fitting Session
POST /api/v1/fittings
{
  "shop_id": "uuid",
  "bay_id": "uuid",
  "customer": {
    "first_name": "Tom",
    "last_name": "Rodriguez",
    "email": "tom@example.com",
    "phone": "+14155551234",
    "handicap": 12.0
  },
  "fitting_type": "driver"
}
Response: {
  "fitting_id": "uuid",
  "status": "in_progress",
  "started_at": "2025-11-28T14:30:00Z"
}

// Poll for New Shots (Internal - called by background job)
GET /api/v1/trackman/shots/poll?connection_id=uuid&since=2025-11-28T14:30:00Z
Response: {
  "shots": [
    {
      "shot_id": "TM_20251128_143022_001",
      "timestamp": "2025-11-28T14:30:22.123Z",
      "ball_speed_mph": 145.2,
      "carry_distance_yards": 245.3,
      // ... all shot data
    }
  ],
  "next_poll_at": "2025-11-28T14:30:27.000Z"
}

// Complete Fitting Session
PATCH /api/v1/fittings/:id/complete
{
  "shot_ids_to_exclude": ["uuid1", "uuid2"], // optional: remove warm-up shots
  "fitter_notes": "Customer loved TaylorMade Stealth driver with Ventus shaft"
}
Response: {
  "fitting_id": "uuid",
  "status": "completed",
  "completed_at": "2025-11-28T15:30:00Z",
  "shot_count": 28,
  "video_job_id": "uuid"
}
```

**Background Jobs (Node.js Workers):**

```javascript
// TrackMan Polling Job (runs every 2 seconds per active fitting)
async function pollTrackManForShots(connectionId, fittingId, lastTimestamp) {
  const connection = await db.trackman_connections.findById(connectionId);
  const trackmanClient = new TrackManAPI(connection.api_key);
  
  const newShots = await trackmanClient.getShotsSince(lastTimestamp);
  
  for (const shot of newShots) {
    // Save to database
    await db.shots.insert({
      fitting_id: fittingId,
      shot_data: shot,
      is_valid: validateShot(shot),
      created_at: new Date()
    });
    
    // Emit WebSocket event to fitter's device
    io.to(`fitting:${fittingId}`).emit('new_shot', shot);
  }
  
  return newShots[newShots.length - 1]?.timestamp || lastTimestamp;
}

// Token Refresh Job (runs every 23 hours)
async function refreshTrackManTokens() {
  const expiringConnections = await db.trackman_connections
    .where('expires_at < NOW() + INTERVAL 1 HOUR')
    .all();
    
  for (const connection of expiringConnections) {
    const newToken = await trackmanAPI.refreshToken(connection.refresh_token);
    await db.trackman_connections.update(connection.id, {
      api_key: encrypt(newToken),
      expires_at: new Date(Date.now() + 24 * 60 * 60 * 1000)
    });
  }
}
```

---

**Database Changes:**

```sql
-- New table: trackman_connections
CREATE TABLE trackman_connections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  shop_id UUID REFERENCES shops(id) NOT NULL,
  bay_label VARCHAR(100) NOT NULL, -- "Bay 1 - Main Shop"
  trackman_model VARCHAR(50) NOT NULL, -- "trackman_4", "trackman_io", "trackman_range"
  serial_number VARCHAR(50) NOT NULL, -- "TM-123456"
  api_key_encrypted TEXT NOT NULL, -- AES-256 encrypted
  api_token_expires_at TIMESTAMP,
  status VARCHAR(20) DEFAULT 'active', -- "active", "disconnected", "expired"
  last_poll_timestamp TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(serial_number) -- One connection per TrackMan device
);

-- Index for fast lookups
CREATE INDEX idx_trackman_shop_id ON trackman_connections(shop_id);
CREATE INDEX idx_trackman_status ON trackman_connections(status) WHERE status = 'active';

-- Add bay_id to fittings table
ALTER TABLE fittings 
  ADD COLUMN bay_id UUID REFERENCES trackman_connections(id);

-- Add TrackMan shot ID to shots table
ALTER TABLE shots
  ADD COLUMN trackman_shot_id VARCHAR(100) UNIQUE;
```

---

### 7. User Interface Specifications

**Wireframes:** See `designs/trackman-connection-flow.fig` in Figma

**Layout Requirements:**

**Screen 1: TrackMan Connection Setup**
```
┌─────────────────────────────────────────┐
│  Connect Your TrackMan                   │
├─────────────────────────────────────────┤
│                                          │
│  Step 1 of 3: Enter TrackMan Credentials│
│                                          │
│  TrackMan Username:                      │
│  [shop@example.com              ]       │
│                                          │
│  TrackMan API Key:                       │
│  [TM_xxxxxxxxxxxxxxxxxxxxxx     ]       │
│  → Where do I find this?                │
│                                          │
│  Bay Label (optional):                   │
│  [Bay 1 - Main Shop             ]       │
│                                          │
│  [ Cancel ]          [ Connect → ]      │
└─────────────────────────────────────────┘
```

**Screen 2: Live Fitting Session**
```
┌─────────────────────────────────────────┐
│  Fitting: Tom Rodriguez | Bay 1  [End] │
├─────────────────────────────────────────┤
│  Driver Fitting • 18 shots captured     │
│                                          │
│  Recent Shots:                           │
│  ┌────────────────────────────────────┐│
│  │ Shot 18 • 2:43 PM • Driver         ││
│  │ 🟢 245 yards • 145 mph ball speed  ││
│  │ Launch: 14.2° • Spin: 2650 rpm    ││
│  └────────────────────────────────────┘│
│  ┌────────────────────────────────────┐│
│  │ Shot 17 • 2:42 PM • Driver ⚠️      ││
│  │ 🟡 178 yards • Mis-hit detected    ││
│  │ Smash: 1.18 • Flag as valid?      ││
│  └────────────────────────────────────┘│
│                                          │
│  [ View All Shots (18) ]                │
└─────────────────────────────────────────┘
```

**Interactive Elements:**
- **Connect Button:** Primary action (orange #FF6B35), disabled until form valid
- **Shot Cards:** Tap to expand full details, swipe to delete
- **End Fitting Button:** Secondary action, requires confirmation modal

**Responsive Design:**
- **iPad (1024×768):** Full-width layout, side-by-side form fields
- **iPad Mini (768×1024):** Stacked form fields, larger tap targets (48px min)
- **Desktop:** Same as iPad but max-width 600px (centered)

---

### 8. Testing Requirements

**Unit Tests:**
```javascript
// Backend: TrackMan API Client
describe('TrackManAPI', () => {
  it('should authenticate with valid credentials', async () => {
    const client = new TrackManAPI();
    const result = await client.authenticate('user@example.com', 'validApiKey');
    expect(result.token).toBeDefined();
    expect(result.expiresAt).toBeGreaterThan(Date.now());
  });
  
  it('should throw error with invalid credentials', async () => {
    const client = new TrackManAPI();
    await expect(client.authenticate('bad@example.com', 'badKey'))
      .rejects.toThrow('Invalid credentials');
  });
  
  it('should parse shot data correctly', () => {
    const rawShot = { /* TrackMan JSON */ };
    const parsed = client.parseShot(rawShot);
    expect(parsed.ball_speed_mph).toBe(145.2);
    expect(parsed.carry_distance_yards).toBe(245.3);
  });
});

// Frontend: Shot Validation
describe('validateShot', () => {
  it('should flag shot with low ball speed as invalid', () => {
    const shot = { ball_speed_mph: 45, smash_factor: 1.15 };
    const result = validateShot(shot);
    expect(result.is_valid).toBe(false);
    expect(result.reason).toBe('likely mis-hit');
  });
  
  it('should pass shot with good metrics', () => {
    const shot = { ball_speed_mph: 145, smash_factor: 1.47 };
    const result = validateShot(shot);
    expect(result.is_valid).toBe(true);
  });
});
```

**Integration Tests:**
```javascript
describe('TrackMan Integration Flow', () => {
  it('should complete full fitting flow end-to-end', async () => {
    // 1. Connect TrackMan
    const connection = await api.post('/trackman/connect', {
      shop_id: testShop.id,
      username: 'test@trackman.com',
      api_key: process.env.TRACKMAN_TEST_KEY
    });
    expect(connection.status).toBe('connected');
    
    // 2. Start fitting
    const fitting = await api.post('/fittings', {
      shop_id: testShop.id,
      bay_id: connection.id,
      customer: { name: 'Test Customer', email: 'test@example.com' }
    });
    
    // 3. Simulate shots via TrackMan test API
    await trackmanTestAPI.hitShot({ ball_speed: 145, carry: 245 });
    await trackmanTestAPI.hitShot({ ball_speed: 148, carry: 250 });
    
    // 4. Poll for shots (simulate background job)
    await wait(3000); // 3 seconds for polling
    const shots = await db.shots.where({ fitting_id: fitting.id }).all();
    expect(shots.length).toBe(2);
    
    // 5. Complete fitting
    await api.patch(`/fittings/${fitting.id}/complete`);
    const completedFitting = await db.fittings.findById(fitting.id);
    expect(completedFitting.status).toBe('completed');
  });
});
```

**User Acceptance Tests:**

**Scenario 1: Happy Path - First-Time TrackMan Connection**
1. Shop owner logs into FitClip dashboard
2. Clicks "Connect TrackMan" in onboarding wizard
3. Enters TrackMan username and API key
4. Clicks "Connect"
5. Sees "Connection Successful" message with serial number
6. Test shot is captured and displayed

**Expected Result:** Connection completes in <60 seconds, test shot appears in UI

**Scenario 2: Fitting Session with 25 Shots**
1. Fitter opens FitClip app on iPad
2. Taps "Start New Fitting"
3. Enters customer details (name, email, phone)
4. Begins hitting shots on TrackMan
5. Watches live shot feed update in real-time (all 25 shots appear)
6. Taps "Complete Fitting"
7. Video generation starts

**Expected Result:** Zero manual data entry, all shots captured, video job triggered

**Scenario 3: Network Interruption Recovery**
1. Fitting in progress with 10 shots captured
2. Wi-Fi network disconnects (unplug router)
3. Customer hits 5 more shots (TrackMan stores locally)
4. Wi-Fi reconnects after 2 minutes
5. FitClip detects reconnection and pulls missing shots

**Expected Result:** All 15 shots present in database, no data loss, notification shown

---

### 9. Implementation Plan

**Development Phases:**

**Phase 1: TrackMan API Research & Sandbox (Week 1-2)**
- Obtain TrackMan API access (apply for developer account)
- Test TrackMan API in sandbox environment
- Document all available endpoints and data fields
- Build proof-of-concept: Connect → Poll → Display shot
- **Deliverable:** Technical spike document with API capabilities

**Phase 2: Backend Integration (Week 3-4)**
- Implement TrackMan authentication flow
- Build polling background job (Node.js worker)
- Create database tables (`trackman_connections`, add fields to `shots`)
- Implement shot validation logic
- Add WebSocket support for real-time updates
- **Deliverable:** API endpoints functional, shots flow from TrackMan to FitClip DB

**Phase 3: Frontend UI (Week 5-6)**
- Build connection setup wizard (React)
- Create fitting session manager (start/stop flow)
- Implement live shot feed with WebSocket updates
- Add shot detail modal (view/edit shots)
- Mobile responsive design for iPad
- **Deliverable:** Complete fitter-facing UI

**Phase 4: Testing & Pilot (Week 7-8)**
- Unit tests (80% code coverage target)
- Integration tests with TrackMan sandbox
- UAT with 2 pilot shops (real fittings)
- Bug fixes and performance optimization
- **Deliverable:** Production-ready feature, pilot shop feedback

**Dependencies:**
- **Blocking:** TrackMan API access (Week 1) - blocks all development
- **Parallel:** Database schema can be built before API access (Week 2)
- **External:** TrackMan partnership agreement (legal, Week 1-2)

**Risk Assessment:**
- **Technical Risk:** TrackMan API may have rate limits we don't know about yet
  - **Mitigation:** Test with high-volume fitting (60+ shots) in sandbox
- **Timeline Risk:** API access delayed beyond Week 2
  - **Contingency:** Build CSV upload fallback, add API later

---

### 10. Success Metrics & Monitoring

**Feature Adoption Metrics:**
- **Target:** 90% of shops connect TrackMan within first week of onboarding
- **Measurement:** `trackman_connections.created_at` vs. `shops.created_at`
- **Dashboard:** Shop onboarding funnel (signup → TrackMan connected → first video sent)

**Technical Metrics:**
- **Shot Capture Success Rate:** >99.9% (shots hit vs. shots in database)
- **API Error Rate:** <0.1% (failed TrackMan API calls / total calls)
- **Latency:** <5 seconds from shot to FitClip (p95)
- **Polling Efficiency:** 2-second intervals maintained under load

**Business Impact Metrics:**
- **Fitter Time Saved:** 10 minutes per fitting (no manual data entry)
- **Shot Data Accuracy:** 100% (vs. 85% with manual entry)
- **Shop Satisfaction:** NPS +20 points (compared to shops without TrackMan integration)

**Monitoring Setup:**
```javascript
// Datadog APM
datadog.track('trackman.shot.captured', {
  shop_id: shopId,
  fitting_id: fittingId,
  latency_seconds: Date.now() - shotTimestamp,
  shot_count: shotNumber
});

datadog.track('trackman.api.error', {
  shop_id: shopId,
  error_code: 429, // rate limit
  endpoint: '/v1/shots/recent'
});

// Alert if error rate >1%
if (errorRate > 0.01) {
  pagerduty.alert('TrackMan API errors elevated', { severity: 'high' });
}
```

---

## FEATURE 2: AI HIGHLIGHT SELECTION ENGINE

### 1. Feature Overview

**Feature Name:** AI-Powered Highlight Selection  
**Feature Category:** Core AI/ML  
**Priority Level:** Must-Have (P0 - Differentiator)  
**Target Release:** MVP v1.0 (Month 3)

**Problem Statement:**  
Fitting sessions produce 30-60 shots, but customers only want to see the best 5-7. Manual selection is time-consuming and subjective. Fitters need an automated system that identifies the most impactful moments to create engaging 2-3 minute videos.

**Success Criteria:**
- AI selects 7 highlights in <10 seconds
- 85% agreement with human expert editors (validated on 100 fittings)
- 92% video watch-through rate (customers watch >90% of video)
- Zero manual editing required for 70% of videos

**User Impact:**
- **Golfers:** Watch engaging videos focused on their best shots and improvements
- **Fitters:** Trust AI to select highlights, override only when needed (30% of cases)
- **Shop Owners:** Videos drive 68% conversion rate (vs. 40% without highlights)

---

### 2. User Personas & Use Cases

**Primary Persona: Tom Rodriguez - Golfer**
- **Demographics:** 38, 12 handicap, watches YouTube golf content daily
- **Goals:** See his best shots, understand improvement, feel confident in purchase
- **Pain Points:** Boring recap videos with too much filler lose his attention
- **Context:** Watches video on phone during commute home from shop (15-minute drive)

**Secondary Persona: Sarah Martinez - Fitter**
- **Goals:** Video showcases her fitting expertise, customer sees clear improvement
- **Pain Points:** AI sometimes selects shots she wouldn't choose (e.g., ignores feel preference)
- **Context:** Reviews AI selections in 30 seconds, overrides 2-3 shots per fitting if needed

**Edge Cases:**
- Customer had no good shots (struggling golfer, all mis-hits)
- Customer improved significantly (dramatic before/after)
- Customer's best shot was with their old clubs (contradicts recommendation)
- Multiple clubs tested, need variety (not 7 driver shots)

---

### 3. Detailed User Stories

**Epic:** As a golfer, I want FitClip to automatically create a highlight reel of my best shots so that I stay engaged and clearly see my improvement.

---

#### Story 2.1: Automatic Highlight Selection After Fitting

**As a** golfer who just completed a club fitting  
**I want** FitClip to automatically select my 7 best shots  
**So that** I see a concise, engaging video (not 60 boring shots)

**Acceptance Criteria:**

**Given** a fitting session with 42 shots captured  
**When** the fitter completes the fitting  
**Then** FitClip's AI analyzes all shots and selects 7 highlights within 10 seconds  
**And** Selected shots include:
  - Longest carry distance (highlight #1)
  - Straightest shot (lowest offline distance) (highlight #2)
  - Best overall (highest composite score) (highlight #3)
  - Biggest improvement from baseline (highlight #4)
  - 3 additional high-quality shots showing variety

**Given** the customer had multiple club types tested (driver, 7-iron, wedge)  
**When** AI selects highlights  
**Then** Highlights include at least one shot per club type (variety)  
**And** Not all 7 highlights are driver shots (balanced representation)

**Given** the customer had several mis-hits  
**When** AI filters shots  
**Then** Invalid shots (flagged earlier) are excluded from consideration  
**And** Only shots with `is_valid = TRUE` are eligible for highlights

**Given** the customer's best shot was with their old clubs (baseline)  
**When** AI selects highlights  
**Then** Baseline shot is excluded (video focuses on new clubs)  
**And** If baseline shot is significantly better, it's included for honest before/after comparison

**Definition of Done:**
- AI runs in <10 seconds for 99% of fittings (60 shots max)
- Selection algorithm validated against 100 human-labeled fittings (85% agreement)
- Video watch-through rate >90% (measured via analytics)

---

#### Story 2.2: Fitter Review & Override (Optional Workflow)

**As a** fitter who knows my customer's preferences  
**I want** to review AI-selected highlights and override if needed  
**So that** the video reflects nuances AI might miss (feel, swing thought, etc.)

**Acceptance Criteria:**

**Given** AI has selected 7 highlights  
**When** the fitter taps "Review Highlights" before completing fitting  
**Then** Fitter sees all 42 shots with 7 marked as "Selected"  
**And** Selected shots are highlighted in green with ⭐️ icon  
**And** Fitter can tap any shot to toggle selection on/off

**Given** fitter wants to override a selection  
**When** fitter deselects Shot #12 (AI's choice) and selects Shot #18 instead  
**Then** AI recalculates order to maintain compelling narrative  
**And** Fitter sees updated highlight list: "7 shots selected (2 manual overrides)"  
**And** Override reason is logged for ML training: "Fitter preferred shot #18"

**Given** fitter is satisfied with AI selections  
**When** fitter taps "Looks Good, Generate Video"  
**Then** Video generation starts immediately with AI-selected highlights  
**And** No manual override required (70% of fittings)

**Definition of Done:**
- Override UI tested with 5 fitters (all successfully changed selections)
- Override rate tracked (target: <30% of fittings)
- Override data logged to retrain ML model

---

#### Story 2.3: Before/After Comparison Shot Selection

**As a** golfer watching my video  
**I want** to see clear before/after comparisons between my old clubs and new clubs  
**So that** I understand exactly why the new clubs are better

**Acceptance Criteria:**

**Given** a fitting session where customer hit 8 baseline shots (their old driver) and 34 test shots (new drivers)  
**When** AI selects highlights  
**Then** At least one before/after pair is included:
  - Baseline shot (old driver): 235 yards, 15° launch, 3200 rpm spin
  - Best test shot (new driver): 252 yards, 13.5° launch, 2650 rpm spin  
**And** These two shots are shown side-by-side in video (split screen)  
**And** Metrics overlay clearly shows improvement: "+17 yards, -550 rpm spin"

**Given** customer's baseline clubs performed terribly (all topped shots)  
**When** AI evaluates baseline shots  
**Then** Best baseline shot is still selected (even if bad) to show honest comparison  
**And** Video narrator explains: "Your old driver averaged 180 yards. The new driver averaged 245 yards."

**Definition of Done:**
- Before/after pairs included in 95% of videos (if baseline shots exist)
- Customer feedback: 90% say comparison was clear and convincing

---

### 4. Functional Requirements

**Core Functionality:**

**Function 1: Scoring Algorithm for Shot Quality**
- **Input:** Shot data (ball speed, carry distance, spin rate, offline distance, smash factor)
- **Processing:**
  ```python
  def calculate_shot_quality_score(shot):
      """
      Score shot on 0-100 scale based on multiple factors.
      Higher score = better shot for video highlight.
      """
      score = 0
      
      # Distance score (40% weight)
      # Normalize to expected distance for club type
      expected_carry = get_expected_carry(shot.club_type, shot.handicap)
      distance_ratio = shot.carry_distance / expected_carry
      distance_score = min(distance_ratio * 40, 40)  # cap at 40
      
      # Accuracy score (30% weight)
      # Penalize offline distance
      offline_penalty = abs(shot.offline_distance) / 50  # 50 yards = full penalty
      accuracy_score = max(30 - (offline_penalty * 30), 0)
      
      # Quality of contact score (20% weight)
      # Smash factor indicates pure strike
      ideal_smash = 1.48 if shot.club_type == 'driver' else 1.35
      smash_diff = abs(shot.smash_factor - ideal_smash)
      contact_score = max(20 - (smash_diff * 40), 0)
      
      # Spin score (10% weight)
      # Optimal spin for distance (2200-2800 rpm for driver)
      if 2200 <= shot.spin_rate <= 2800:
          spin_score = 10
      else:
          spin_diff = min(abs(shot.spin_rate - 2500), 1000)
          spin_score = max(10 - (spin_diff / 100), 0)
      
      total_score = distance_score + accuracy_score + contact_score + spin_score
      return round(total_score, 2)
  ```
- **Output:** Score 0-100 for each shot, stored in `shots.highlight_score`
- **Validation:** Scores calibrated against human expert ratings (Pearson correlation >0.85)

**Function 2: Highlight Selection Algorithm**
- **Input:** All valid shots for a fitting (array of shots with scores)
- **Processing:**
  ```python
  def select_highlights(shots, num_highlights=7):
      """
      Select diverse, high-quality highlights.
      """
      highlights = []
      
      # 1. Longest shot (crowd pleaser)
      longest = max(shots, key=lambda s: s.carry_distance)
      highlights.append(longest)
      
      # 2. Straightest shot (accuracy showcase)
      straightest = min(shots, key=lambda s: abs(s.offline_distance))
      highlights.append(straightest)
      
      # 3. Best overall score
      best_overall = max(shots, key=lambda s: s.highlight_score)
      if best_overall not in highlights:
          highlights.append(best_overall)
      
      # 4. Biggest improvement from baseline (if baseline exists)
      baseline_avg = calculate_avg(shots, is_baseline=True)
      if baseline_avg:
          improvements = [s for s in shots if not s.is_baseline]
          best_improvement = max(improvements, 
                                 key=lambda s: s.carry_distance - baseline_avg.carry_distance)
          highlights.append(best_improvement)
      
      # 5-7. Fill remaining slots with high-scoring shots, ensuring variety
      remaining = [s for s in shots if s not in highlights]
      remaining_sorted = sorted(remaining, key=lambda s: s.highlight_score, reverse=True)
      
      # Ensure club variety (don't show 7 driver shots)
      club_counts = {h.club_type: 1 for h in highlights}
      for shot in remaining_sorted:
          if len(highlights) >= num_highlights:
              break
          # Prefer shots from underrepresented clubs
          if club_counts.get(shot.club_type, 0) < 3:  # max 3 per club type
              highlights.append(shot)
              club_counts[shot.club_type] = club_counts.get(shot.club_type, 0) + 1
      
      # Sort highlights chronologically for natural video flow
      highlights_sorted = sorted(highlights, key=lambda s: s.shot_timestamp)
      
      return highlights_sorted
  ```
- **Output:** Array of 7 shot IDs for video generation
- **Edge Cases:**
  - If <7 valid shots exist, include all valid shots (don't pad with bad shots)
  - If all shots are similar quality, prioritize variety over score

**Function 3: Before/After Pair Identification**
- **Input:** Baseline shots (customer's old clubs), test shots (new clubs)
- **Processing:**
  1. Calculate average carry distance for baseline shots
  2. Find best baseline shot (highest score among baseline shots)
  3. Find best test shot (highest score among test shots)
  4. Ensure meaningful difference (>10 yards or >5% improvement)
  5. If no meaningful difference, skip before/after comparison
- **Output:** Pair of shot IDs: `{baseline_shot_id: uuid, test_shot_id: uuid, improvement: "+17 yards"}`

**Business Rules:**
- **Rule 1:** Never show more than 3 shots per club type (variety requirement)
- **Rule 2:** Always include longest shot (customer expectation)
- **Rule 3:** Exclude invalid shots from highlight consideration (quality baseline)
- **Rule 4:** If fitter manually overrides, log for ML retraining (continuous improvement)

---

### 5. Non-Functional Requirements

**Performance Requirements:**
- **Response Time:** Highlight selection completes in <10 seconds for 60-shot fitting
- **ML Model Inference:** <100ms per shot scoring (parallelizable)
- **Scalability:** Can score 500,000 fittings/year (1.37M shots/day at peak)

**Accuracy Requirements:**
- **Agreement with Human Experts:** 85% (measured on 100 labeled fittings)
- **Customer Satisfaction:** 90% of golfers say highlights were "accurate and engaging"
- **Watch-Through Rate:** >90% of customers watch full video (proxy for quality)

---

### 6. Technical Specifications

**ML Model Stack:**

**Option 1: Rule-Based System (MVP)**
- Pros: Fast to build, interpretable, no training data required
- Cons: Less adaptive, may miss nuanced patterns
- **Decision:** Use for MVP (Month 1-3), upgrade to ML in Month 6+

**Option 2: Gradient Boosting (XGBoost) [Post-MVP]**
- Training data: 1,000 fittings manually labeled by golf pros
- Features: 15 shot metrics (distance, accuracy, spin, smash, launch angle, etc.)
- Target: Binary classification (is_highlight: TRUE/FALSE)
- Model retraining: Monthly as new data arrives

**Backend API:**

```python
# Python microservice for highlight selection
from flask import Flask, request, jsonify
import xgboost as xgb

app = Flask(__name__)
model = xgb.Booster(model_file='models/highlight_selector_v1.model')

@app.route('/api/v1/highlights/select', methods=['POST'])
def select_highlights():
    """
    Input: { "fitting_id": "uuid", "shots": [...] }
    Output: { "highlight_shot_ids": ["uuid1", "uuid2", ...], "reasoning": {...} }
    """
    data = request.json
    shots = data['shots']
    
    # Score each shot
    for shot in shots:
        shot['highlight_score'] = calculate_shot_quality_score(shot)
    
    # Select 7 highlights
    highlights = select_highlights(shots, num_highlights=7)
    
    # Identify before/after pair
    before_after = identify_before_after_pair(shots)
    
    return jsonify({
        'highlight_shot_ids': [h['id'] for h in highlights],
        'before_after_pair': before_after,
        'reasoning': {
            'longest_shot': highlights[0]['id'],
            'straightest_shot': highlights[1]['id'],
            'best_overall': highlights[2]['id']
        },
        'processing_time_ms': 87
    })
```

---

**Database Schema:**

```sql
-- Add highlight metadata to shots table
ALTER TABLE shots
  ADD COLUMN highlight_score DECIMAL(5,2), -- 0-100 quality score
  ADD COLUMN is_highlight BOOLEAN DEFAULT FALSE, -- selected by AI
  ADD COLUMN highlight_position INTEGER, -- order in video (1-7)
  ADD COLUMN manual_override BOOLEAN DEFAULT FALSE, -- fitter changed selection
  ADD COLUMN override_reason TEXT; -- for ML training

-- Index for fast highlight queries
CREATE INDEX idx_shots_highlights ON shots(fitting_id, is_highlight) 
  WHERE is_highlight = TRUE;
```

---

### 7. Testing Requirements

**ML Model Validation:**

```python
# Test on holdout dataset
def validate_model():
    """
    Load 100 human-labeled fittings, compare AI vs. expert selections.
    """
    test_fittings = load_test_data('data/labeled_fittings.json')
    
    agreement_scores = []
    for fitting in test_fittings:
        expert_highlights = fitting['expert_selected_shots']
        ai_highlights = select_highlights(fitting['shots'])
        
        # Calculate Jaccard similarity (overlap)
        overlap = set(ai_highlights) & set(expert_highlights)
        agreement = len(overlap) / 7  # 7 highlights total
        agreement_scores.append(agreement)
    
    avg_agreement = sum(agreement_scores) / len(agreement_scores)
    print(f"Average agreement: {avg_agreement:.2%}")
    assert avg_agreement >= 0.85, "Model accuracy below threshold"
```

**User Acceptance Test:**

**Scenario: Golfer Watches Video with AI-Selected Highlights**
1. Complete fitting with 38 shots
2. AI selects 7 highlights in 8 seconds
3. Video is generated and sent to golfer
4. Golfer watches video on phone
5. Golfer watches >90% of video (tracked via analytics)
6. Golfer clicks "Buy Now" after watching

**Expected Result:** High watch-through rate, golfer reports video was "engaging and accurate"

---

### 8. Success Metrics

**Feature Adoption:**
- 100% of videos use AI highlights (no manual selection option initially)

**Technical Metrics:**
- Highlight selection time: <10 seconds (p95)
- Agreement with experts: 85%+
- Override rate: <30% (fitters trust AI 70%+ of time)

**Business Impact:**
- Video watch-through rate: >90%
- Conversion rate: 68% (up from 40% without video)
- Customer NPS: +15 points for "video quality" attribute

---

## FEATURE 3: AUTOMATED VIDEO GENERATION

[Continue with remaining features following same detailed structure...]

---

# ADDITIONAL HANDOFF DOCUMENTS NEEDED

Based on the PRD and feature specs, here's what else a coding agent will need:

## 1. **API Documentation (OpenAPI/Swagger)**
- Complete REST API spec with all endpoints
- Request/response schemas
- Authentication flows
- Error codes and handling

## 2. **Data Flow Diagrams**
- Fitting session lifecycle (start → shots → complete → video → delivery)
- TrackMan integration data flow
- Video generation pipeline
- Analytics event tracking

## 3. **Development Environment Setup**
- Local development guide (Docker Compose setup)
- Environment variables (.env.example)
- Database seeding scripts (test data)
- API mocking for TrackMan (development without real hardware)

## 4. **Design System & Component Library**
- Figma file with all screens
- React component specifications
- Style guide (colors, typography, spacing)
- Responsive breakpoints

## 5. **Deployment & Infrastructure**
- AWS architecture diagram
- Terraform/IaC scripts
- CI/CD pipeline (GitHub Actions)
- Monitoring & alerting setup

## 6. **Third-Party Integration Guides**
- TrackMan API documentation & SDK
- Twilio SMS setup guide
- SendGrid email templates
- Stripe payment integration

---

Would you like me to create any of these additional documents? I recommend:

1. **TECHNICAL_ARCHITECTURE.md** - Detailed system architecture with diagrams
2. **API_REFERENCE.md** - Complete OpenAPI spec for all endpoints
3. **DEVELOPMENT_SETUP.md** - Step-by-step setup guide for developers
4. **DEPLOYMENT_GUIDE.md** - AWS infrastructure and CI/CD pipeline

Let me know which ones you want next!

