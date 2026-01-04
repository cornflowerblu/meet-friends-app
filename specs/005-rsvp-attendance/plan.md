# Implementation Plan: RSVP & Attendance

**Branch**: `005-rsvp-attendance` | **Date**: 2026-01-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/005-rsvp-attendance/spec.md`

## Summary

The RSVP & Attendance feature enables users to commit to events, manage their participation, and build trust through attendance patterns. The implementation creates an RSVP lifecycle management system with real-time waitlist automation, day-of confirmation flows, and private post-event trust signal collection. This is the critical bridge between event discovery and IRL meetups - the core product value.

Technical approach: Event-driven architecture with background workers for waitlist promotion and notification delivery. PostgreSQL for RSVP state management with optimistic locking. SNS/SQS for async notification pipeline. Rule-based reliability scoring.

## Technical Context

**Language/Version**: TypeScript/Node.js 22 LTS (backend), TypeScript/React Native Expo (mobile)
**Primary Dependencies**: NestJS 11.x (backend), React Native 0.81 (Expo SDK 54), React 19.1, PostgreSQL
**Storage**: PostgreSQL (RDS) for RSVP, waitlist, trust signals; Redis for real-time counts
**Testing**: Jest (backend), Detox or React Native Testing Library (mobile)
**Target Platform**: iOS 15.1+, Android API 24+ (targeting API 36), API on ECS Fargate
**Project Type**: Mobile + API
**Performance Goals**: RSVP action < 2 seconds (SC-001), real-time attendee count updates
**Constraints**: 4-hour waitlist claim window, 24-hour post-event feedback window
**Scale/Scope**: MVP targets hundreds of users, thousands of events

## Constitution Check

| Principle | Requirement | Compliance |
|-----------|-------------|------------|
| I. IRL-First | Features MUST support getting people to show up | PASS - RSVP is direct path to attendance |
| II. Trust & Safety | Private trust scoring MUST inform matching | PASS - "Would you meet again?" collected |
| II. Trust & Safety | Tiered consequences for bad behavior | PASS - Reliability score affects priority |
| III. Vibe Compatibility | Soft ranking, not hard gating | PASS - Low reliability affects priority, not access |
| IV. AI as Support | No per-message LLM calls | PASS - Trust signals are simple thumbs up/down |
| V. Cost-Conscious MVP | Uses SQS workers (cheap), rule-based scoring | PASS |

## Project Structure

### Source Code

```text
api/
├── src/
│   ├── modules/
│   │   ├── rsvp/
│   │   │   ├── rsvp.module.ts
│   │   │   ├── rsvp.controller.ts
│   │   │   ├── rsvp.service.ts
│   │   │   └── entities/
│   │   │       └── rsvp.entity.ts
│   │   ├── waitlist/
│   │   │   ├── waitlist.module.ts
│   │   │   ├── waitlist.service.ts
│   │   │   ├── waitlist-promotion.worker.ts
│   │   │   └── entities/
│   │   │       └── waitlist-position.entity.ts
│   │   ├── attendance/
│   │   │   ├── attendance.module.ts
│   │   │   ├── confirmation.controller.ts
│   │   │   ├── confirmation.service.ts
│   │   │   └── entities/
│   │   │       └── attendance-confirmation.entity.ts
│   │   ├── my-events/
│   │   │   ├── my-events.module.ts
│   │   │   ├── my-events.controller.ts
│   │   │   └── my-events.service.ts
│   │   ├── trust-signals/
│   │   │   ├── trust-signals.module.ts
│   │   │   ├── trust-signals.controller.ts
│   │   │   ├── trust-signals.service.ts
│   │   │   └── entities/
│   │   │       ├── trust-signal.entity.ts
│   │   │       └── user-reliability.entity.ts
│   │   └── notifications/
│   │       ├── notifications.module.ts
│   │       └── notification.worker.ts
│   └── database/
│       └── migrations/
└── tests/

mobile/
├── src/
│   ├── features/
│   │   ├── rsvp/
│   │   │   ├── components/
│   │   │   │   ├── RSVPButton.tsx
│   │   │   │   ├── CancelRSVPModal.tsx
│   │   │   │   └── WaitlistBadge.tsx
│   │   │   └── hooks/
│   │   │       └── useRSVP.ts
│   │   ├── my-events/
│   │   │   ├── screens/
│   │   │   │   ├── MyEventsScreen.tsx
│   │   │   │   ├── UpcomingEventsTab.tsx
│   │   │   │   └── PastEventsTab.tsx
│   │   │   └── components/
│   │   │       └── TodayBadge.tsx
│   │   ├── confirmation/
│   │   │   └── components/
│   │   │       ├── ConfirmationPrompt.tsx
│   │   │       └── ConfirmationBanner.tsx
│   │   └── trust-signals/
│   │       ├── screens/
│   │       │   └── PostEventFeedbackScreen.tsx
│   │       └── components/
│   │           └── AttendeeRatingCard.tsx
└── tests/
```

## Data Model

### RSVP Entity

```
RSVP
├── id: UUID (PK)
├── user_id: UUID (FK → users)
├── event_id: UUID (FK → events)
├── status: ENUM (pending, confirmed, canceled, no_show)
├── created_at: TIMESTAMP
├── confirmed_at: TIMESTAMP (nullable)
├── canceled_at: TIMESTAMP (nullable)
├── cancellation_reason: TEXT (nullable)
└── UNIQUE(user_id, event_id)
```

### WaitlistPosition Entity

```
WaitlistPosition
├── id: UUID (PK)
├── user_id: UUID (FK → users)
├── event_id: UUID (FK → events)
├── position: INTEGER
├── status: ENUM (waiting, notified, claimed, expired, declined)
├── claim_expires_at: TIMESTAMP (nullable)
├── created_at: TIMESTAMP
├── notified_at: TIMESTAMP (nullable)
└── UNIQUE(user_id, event_id)
```

### TrustSignal Entity

```
TrustSignal
├── id: UUID (PK)
├── rater_user_id: UUID (FK → users)
├── rated_user_id: UUID (FK → users)
├── event_id: UUID (FK → events)
├── signal: ENUM (positive, negative)
├── created_at: TIMESTAMP
└── UNIQUE(rater_user_id, rated_user_id, event_id)
```

### UserReliability Entity

```
UserReliability
├── id: UUID (PK)
├── user_id: UUID (FK → users, UNIQUE)
├── total_rsvps: INTEGER (default 0)
├── confirmed_attendances: INTEGER (default 0)
├── no_shows: INTEGER (default 0)
├── late_cancellations: INTEGER (default 0)
├── reliability_score: DECIMAL (0.0-1.0)
└── updated_at: TIMESTAMP
```

## State Machines

### RSVP Lifecycle

```
[User Action]        [State]           [Side Effects]
─────────────────────────────────────────────────────────
  Create RSVP    →   pending       →   Increment attendee count
                                   →   Grant chat access (006)
                                   →   Add to My Events

  Confirm        →   confirmed     →   Update confirmation status
  (day-of)                         →   Notify host

  Cancel         →   canceled      →   Decrement attendee count
                                   →   Revoke chat access
                                   →   Trigger waitlist promotion

  No-show        →   no_show       →   Increment no_show count
  (post-event)                     →   Update reliability score
```

### Waitlist Lifecycle

```
[Trigger]            [State]           [Side Effects]
─────────────────────────────────────────────────────────
  Join waitlist  →   waiting       →   Record position
  Spot opens     →   notified      →   Send push, set 4hr window
  User claims    →   claimed       →   Create RSVP, remove from list
  Timer expires  →   expired       →   Notify next in line
  User declines  →   declined      →   Notify next, remove from list
```

## API Contracts

### RSVP Endpoints

```yaml
POST /events/{eventId}/rsvp
Response 201: { rsvp: { id, status: "pending", event } }
Response 202: { waitlist: { id, position } }
Response 409: { error: "Event at capacity", waitlistAvailable: true }

DELETE /events/{eventId}/rsvp
Request: { reason?: string }
Response 200: { message: "RSVP canceled", spotOpened: true }

POST /events/{eventId}/rsvp/confirm
Response 200: { confirmed: true }

GET /me/events
Response 200: { upcoming: [...], past: [...] }
```

### Waitlist Endpoints

```yaml
GET /events/{eventId}/waitlist/position
Response 200: { position: 3, status: "waiting" }

POST /waitlist/{positionId}/claim
Response 200: { rsvp: { id, status: "pending" } }

POST /waitlist/{positionId}/decline
Response 200: { message: "Removed from waitlist" }
```

### Trust Signal Endpoints

```yaml
POST /events/{eventId}/feedback
Request: { signals: [{ userId, signal: "positive"|"negative" }] }
Response 201: { message: "Feedback submitted", count: 3 }

GET /events/{eventId}/feedback-prompt
Response 200: { pending: true, attendees: [...] }
```

## Implementation Phases

### Sprint 1: Core RSVP (P1-P2)
1. Database migrations for RSVP and WaitlistPosition
2. RSVP service with create/cancel logic
3. RSVP API endpoints
4. Mobile RSVPButton component
5. Integration with Event detail view
6. Cancel flow with confirmation modal

### Sprint 2: Waitlist & My Events (P3-P4)
1. Waitlist service with position tracking
2. Waitlist promotion worker (SQS consumer)
3. Claim/decline endpoints and timer
4. My Events service and API
5. Mobile My Events screens
6. Push notifications for waitlist

### Sprint 3: Confirmation & Trust (P5-P6)
1. AttendanceConfirmation entity and service
2. Day-of confirmation prompts
3. Trust signals entity and API
4. Post-event feedback screen
5. UserReliability calculation service
6. Integration with matching

## Dependencies

**Requires**:
- 001-user-onboarding: User entity
- 002-event-creation: Event entity with capacity
- 003-event-discovery: Event detail view

**Provides to**:
- 006-group-chat: Chat access on RSVP
- 004-vibe-matching: Trust signals feed into matching
