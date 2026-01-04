# Implementation Plan: User Onboarding

**Branch**: `001-user-onboarding` | **Date**: 2026-01-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-user-onboarding/spec.md`

## Summary

This feature implements the complete user onboarding flow for the friend-matching app, including phone verification (SMS-based identity), profile photo upload with quality validation, vibe profile collection across 6 behavioral dimensions, and activity selection. The implementation follows the constitution's "IRL-First" and "Trust & Safety" principles, using React Native (Expo) for mobile, NestJS for backend API, PostgreSQL for storage, and S3/CDN for image hosting.

## Technical Context

**Language/Version**: TypeScript 5.x (both mobile and backend)
**Primary Dependencies**: React Native 0.81 (Expo SDK 54), React 19.1, NestJS 11.x, Twilio Verify for SMS
**Storage**: PostgreSQL (RDS) for user data, S3 + CloudFront for images
**Testing**: Jest (unit/integration), Detox (E2E mobile), Supertest (API)
**Target Platform**: iOS 15.1+, Android API 24+ (targeting API 36), AWS (backend)
**Project Type**: Mobile + API (React Native Expo + NestJS)
**Performance Goals**: SMS delivery < 60s (99%), API response < 200ms p95, onboarding < 5 min
**Constraints**: Monthly burn < $500-800 (constitution), no per-message LLM calls
**Scale/Scope**: MVP targeting 10k users, 4 onboarding screens, 8-12 vibe questions

## Constitution Check

| Principle | Requirement | Compliance | Notes |
|-----------|-------------|------------|-------|
| I. IRL-First | Every feature MUST directly support getting people to show up in person | PASS | Onboarding collects activity preferences and vibe data essential for IRL matching |
| II. Trust & Safety | Identity friction MUST be enforced (phone verification, real photos) | PASS | FR-001 (SMS verification), FR-003 (face photo required) directly implement this |
| II. Trust & Safety | Intent framing MUST be clear ("Not a dating app") | PASS | FR-008 explicitly requires this in onboarding copy |
| III. Vibe Compatibility | No personality labels/diagnoses displayed to users | PASS | FR-007 prohibits showing internal vibe scores |
| III. Vibe Compatibility | Behavioral matching across 6 dimensions | PASS | FR-004 collects all 6 vibe dimensions |
| IV. AI as Support | AI MUST convert free-text to structured traits | DEFER | Vibe collection uses structured prompts initially; AI enhancement is future phase |
| V. Cost-Conscious MVP | Monthly burn < $500-800 | PASS | Uses managed services (Twilio, S3, RDS) with predictable costs |
| V. Cost-Conscious MVP | Features start rule-based before ML/AI | PASS | Photo validation uses basic rules first; AI moderation is manual review fallback |

**Gate Status: PASS**

## Project Structure

### Documentation (this feature)

```text
specs/001-user-onboarding/
├── spec.md              # Feature specification (exists)
├── plan.md              # This file
├── research.md          # Phase 0 output - technology decisions
├── data-model.md        # Phase 1 output - entity definitions
├── quickstart.md        # Phase 1 output - dev setup guide
├── contracts/           # Phase 1 output - API contracts
│   └── openapi.yaml     # REST API specification
└── tasks.md             # Phase 2 output (from /speckit.tasks)
```

### Source Code (repository root)

```text
api/
├── src/
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── sms.service.ts
│   │   │   └── dto/
│   │   ├── users/
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── entities/
│   │   │   └── dto/
│   │   ├── photos/
│   │   │   ├── photos.controller.ts
│   │   │   ├── photos.service.ts
│   │   │   ├── upload.service.ts
│   │   │   └── validation.service.ts
│   │   ├── vibe-profiles/
│   │   │   ├── vibe.controller.ts
│   │   │   ├── vibe.service.ts
│   │   │   └── questions.service.ts
│   │   └── activities/
│   │       ├── activities.controller.ts
│   │       └── activities.service.ts
│   ├── common/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── pipes/
│   └── config/
└── tests/
    ├── unit/
    ├── integration/
    └── e2e/

mobile/
├── app/                  # Expo Router file-based routing
│   ├── (auth)/
│   │   ├── phone.tsx     # Phone number entry
│   │   └── verify.tsx    # Code verification
│   ├── (onboarding)/
│   │   ├── photos.tsx    # Photo upload
│   │   ├── vibes.tsx     # Vibe questions
│   │   └── activities.tsx # Activity selection
│   └── _layout.tsx
├── components/
│   ├── ui/               # Reusable UI components
│   ├── forms/            # Form components
│   └── onboarding/       # Onboarding-specific components
├── services/
│   ├── api.ts            # API client
│   ├── auth.ts           # Auth state management
│   └── storage.ts        # Secure storage
├── hooks/
│   ├── useAuth.ts
│   └── useOnboarding.ts
└── __tests__/
```

## Phase 0: Research Decisions

### Decision 1: SMS Verification Provider

**Decision**: Twilio Verify API
**Rationale**: Industry standard for phone verification with built-in rate limiting, fraud detection, and global SMS delivery. Predictable pricing at $0.05/verification.
**Alternatives Rejected**: AWS SNS (requires building verification logic), Firebase Auth (adds dependency)

### Decision 2: Image Storage & CDN

**Decision**: AWS S3 + CloudFront
**Rationale**: Constitution specifies "S3 + CDN for images". Cost-effective for MVP scale.

### Decision 3: Photo Validation Approach

**Decision**: Rule-based validation with manual review fallback
**Rationale**: Per constitution "Features MUST start with rule-based implementations before ML/AI". Initial validation: file type, minimum dimensions (800x800), file size (< 10MB).

### Decision 4: Vibe Question Format

**Decision**: Multiple-choice scenario questions (not sliders/scales)
**Rationale**: Spec requires "scenario-based questions (not clinical scales)". Each question presents a social scenario with 3-4 response options.

### Decision 5: Onboarding State Persistence

**Decision**: Server-side progress tracking with device-local draft state
**Rationale**: Spec FR-006 requires progress persistence. Server tracks completed steps; client caches in-progress form data.

## Phase 1: Data Model

### Entities

#### User
```
User
├── id: UUID (PK)
├── phone_number: string (unique, E.164 format)
├── phone_verified: boolean
├── phone_verified_at: timestamp?
├── onboarding_step: enum (PHONE, PHOTO, VIBE, ACTIVITIES, COMPLETE)
├── general_location: string? (city/region)
├── notification_consent: boolean
├── created_at: timestamp
└── updated_at: timestamp
```

#### ProfilePhoto
```
ProfilePhoto
├── id: UUID (PK)
├── user_id: UUID (FK -> User)
├── s3_key: string
├── cdn_url: string
├── is_primary: boolean
├── status: enum (PENDING, APPROVED, REJECTED)
├── width: integer
├── height: integer
└── uploaded_at: timestamp
```

#### VibeProfile
```
VibeProfile
├── id: UUID (PK)
├── user_id: UUID (FK -> User, unique)
├── casual_competitive: float (-1.0 to 1.0)
├── chatty_focused: float (-1.0 to 1.0)
├── planner_spontaneous: float (-1.0 to 1.0)
├── solo_group: float (-1.0 to 1.0)
├── high_low_energy: float (-1.0 to 1.0)
├── reliability_flexibility: float (-1.0 to 1.0)
├── completed_at: timestamp?
└── skipped: boolean

Note: Scores are NEVER exposed to users (FR-007, Constitution III)
```

#### Activity
```
Activity
├── id: UUID (PK)
├── name: string
├── slug: string (unique)
├── icon_url: string
├── has_skill_levels: boolean
└── display_order: integer

Seed data (10-15 activities for MVP):
golf, pickleball, bowling, running, tennis, hiking, biking, yoga,
basketball, volleyball, soccer, climbing, kayaking, skiing
```

#### ActivityInterest
```
ActivityInterest
├── id: UUID (PK)
├── user_id: UUID (FK -> User)
├── activity_id: UUID (FK -> Activity)
├── skill_level: enum? (BEGINNER, INTERMEDIATE, ADVANCED)
└── created_at: timestamp
```

## Phase 1: API Contracts

### Authentication Endpoints

```yaml
POST /api/v1/auth/send-code
  Request: { phone_number: string }
  Response 200: { message: "Verification code sent", expires_in_seconds: 300 }
  Response 429: { error: "Too many requests", retry_after_seconds: 300 }

POST /api/v1/auth/verify-code
  Request: { phone_number: string, code: string }
  Response 200: { access_token: string, refresh_token: string, user: {...} }
  Response 400: { error: "Invalid code", attempts_remaining: number }

POST /api/v1/auth/request-voice-call
  Request: { phone_number: string }
  Response 200: { message: "Voice call initiated" }
```

### Photo Endpoints

```yaml
POST /api/v1/photos/upload-url
  Response 200: { upload_url: string, photo_id: UUID, expires_in_seconds: 300 }

POST /api/v1/photos/{id}/confirm
  Request: { width: number, height: number }
  Response 200: { id: UUID, cdn_url: string, status: string }

DELETE /api/v1/photos/{id}
  Response 204: (no content)

PATCH /api/v1/photos/{id}/set-primary
  Response 200: { ...photo with is_primary: true }
```

### Vibe Profile Endpoints

```yaml
GET /api/v1/vibe/questions
  Response 200: { questions: [{ id, scenario, options: [{id, text}], dimension }] }

POST /api/v1/vibe/responses
  Request: { responses: [{ question_id, selected_option_id }] }
  Response 200: { vibe_profile_complete: true }

POST /api/v1/vibe/skip
  Response 200: { vibe_profile_complete: false, skipped: true }
```

### Activity Endpoints

```yaml
GET /api/v1/activities
  Response 200: { activities: [{ id, name, slug, icon_url, has_skill_levels }] }

POST /api/v1/users/me/activities
  Request: { activities: [{ activity_id, skill_level? }] }
  Response 200: { activity_interests: [...] }
```

## Implementation Sequence

### Phase A: Foundation (API)
1. Set up NestJS project with TypeScript strict mode
2. Configure PostgreSQL connection and migrations
3. Implement User entity and basic CRUD
4. Set up JWT authentication module
5. Configure S3 client and CloudFront distribution

### Phase B: Phone Verification
1. Integrate Twilio Verify API
2. Implement send-code endpoint with rate limiting
3. Implement verify-code endpoint with attempt tracking
4. Add voice call fallback after 3 SMS failures

### Phase C: Photo Upload
1. Implement presigned URL generation for S3
2. Build photo confirmation with dimension validation
3. Create photo management endpoints
4. Set up moderation queue for flagged photos

### Phase D: Vibe Profile
1. Design and seed 8-12 scenario questions
2. Implement questions endpoint
3. Build response processor that maps to 6 dimensions
4. Handle skip flow with reduced match quality flag

### Phase E: Activities
1. Seed activities table with 10-15 activities
2. Implement activities list endpoint
3. Build user activity interest management
4. Complete onboarding state machine

### Phase F: Mobile App
1. Set up Expo project with Router
2. Implement auth flow screens (phone, verify)
3. Build photo upload screen with camera/gallery
4. Create vibe question carousel
5. Build activity selection grid
6. Implement onboarding progress persistence

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| SMS delivery failures | Voice call fallback after 3 failures; monitoring alerts |
| Photo abuse | Moderation queue with manual review |
| Onboarding drop-off | Analytics on each step; save partial progress |
| Cost overrun | Rate limiting on SMS; budget alerts |

## Dependencies

**Provides to other features**:
- User entity with verified phone and photos
- VibeProfile for matching (004-vibe-matching)
- ActivityInterest for event discovery (003-event-discovery)
