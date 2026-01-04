# Cross-Feature Dependencies & Execution Strategy

**Generated**: 2026-01-04
**Purpose**: Organize implementation sequencing and identify parallel execution opportunities across all 6 features

## Feature Dependency Graph

```
                    ┌─────────────────────────────────────────────────┐
                    │         001-user-onboarding (FOUNDATION)        │
                    │   User, VibeProfile, Activity, ActivityInterest │
                    └───────────────┬─────────────────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│  002-event-creation │  │ 004-vibe-matching   │  │                     │
│  Event, VibeLabel   │  │ MatchingPreference  │  │ (Can be started     │
│  EventSeries        │  │ CompatibilityCalc   │  │  after 001 complete)│
│  EventTemplate      │  │                     │  │                     │
└──────────┬──────────┘  └──────────┬──────────┘  └─────────────────────┘
           │                        │
           │                        │
           ▼                        │
┌─────────────────────┐             │
│  003-event-discovery│◄────────────┘
│  EventFeed, Filters │   (Integrates vibe compatibility)
│  Search, PostGIS    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  005-rsvp-attendance│
│  RSVP, Waitlist     │
│  TrustSignal        │──────────┬──────────────────────┐
│  UserReliability    │          │                      │
└──────────┬──────────┘          │                      │
           │                     │                      │
           ▼                     ▼                      ▼
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│   006-group-chat    │  │ 004-vibe-matching   │  │ Trust scores feed   │
│   EventChat         │  │ (receives trust     │  │ into matching       │
│   ChatParticipant   │  │  signals for        │  │ algorithm           │
│   MessageReport     │  │  enhanced matching) │  │                     │
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
```

## Sequential Dependencies (MUST be in order)

### Critical Path
```
001-user-onboarding → 002-event-creation → 003-event-discovery → 005-rsvp-attendance → 006-group-chat
```

Each feature depends on entities/services from the previous:
1. **001 → 002**: Event creation requires verified User with activities
2. **002 → 003**: Discovery requires Event entity to exist
3. **003 → 005**: RSVP requires event detail view to integrate
4. **005 → 006**: Chat access is granted on RSVP

### Feature-by-Feature Dependencies

| Feature | Requires | Provides To |
|---------|----------|-------------|
| 001-user-onboarding | None | 002, 004 (User, VibeProfile, Activity) |
| 002-event-creation | 001 (Verified User, Activity) | 003, 005 (Event, VibeLabel) |
| 003-event-discovery | 001, 002 | 005 (Event detail view) |
| 004-vibe-matching | 001 (VibeProfile) | 003 (compatibility badges in feed) |
| 005-rsvp-attendance | 001, 002, 003 | 006 (RSVP triggers chat), 004 (trust signals) |
| 006-group-chat | 001, 002, 005 | None (end of chain) |

## Parallel Execution Opportunities

### After 001-user-onboarding completes:

These can be developed IN PARALLEL:
- **002-event-creation** - Needs User entity only
- **004-vibe-matching** - Needs VibeProfile only (backend logic can be built independently)

### After 002-event-creation completes:

These can be developed IN PARALLEL:
- **003-event-discovery** - Needs Event entity
- **004-vibe-matching** (mobile integration) - Badge display can now be added to event cards

### After 003-event-discovery completes:

- **005-rsvp-attendance** - Needs event detail view

### After 005-rsvp-attendance completes:

- **006-group-chat** - Needs RSVP for access control

## Recommended Implementation Strategy

### Strategy 1: Sequential (1 developer or small team)

```
Week 1-2:  001-user-onboarding (104 tasks)
Week 3-4:  002-event-creation (82 tasks)
Week 4-5:  003-event-discovery (66 tasks)
Week 5-6:  004-vibe-matching (58 tasks)
Week 6-8:  005-rsvp-attendance (106 tasks)
Week 8-9:  006-group-chat (70 tasks)
```

### Strategy 2: Parallel Development (2+ developers)

```
Phase A (Foundation):
  - Developer 1: 001-user-onboarding

Phase B (Core Features - can run in parallel after Phase A):
  - Developer 1: 002-event-creation
  - Developer 2: 004-vibe-matching (backend only)

Phase C (Discovery & Integration - can run in parallel):
  - Developer 1: 003-event-discovery + 004 integration
  - Developer 2: 005-rsvp-attendance (can start US1-US3 with event stubs)

Phase D (Final Features):
  - Developer 1: 005-rsvp-attendance completion
  - Developer 2: 006-group-chat
```

## Task Count Summary

| Feature | Total Tasks | Parallel Tasks | User Stories |
|---------|-------------|----------------|--------------|
| 001-user-onboarding | 104 | 38 | 4 (US1-US4) |
| 002-event-creation | 82 | 24 | 5 (US1-US5) |
| 003-event-discovery | 66 | 22 | 5 (US1-US5) |
| 004-vibe-matching | 58 | 18 | 4 (US1-US4) |
| 005-rsvp-attendance | 106 | 36 | 6 (US1-US6) |
| 006-group-chat | 70 | 24 | 6 (US1-US6) |
| **TOTAL** | **486** | **162** | **30** |

## MVP Definition

### Minimal Viable Product (Core Loop)
To have a functional app where users can:
1. Create accounts and verify identity
2. Create events
3. Discover events
4. RSVP to events

**MVP Features:**
- 001-user-onboarding: US1 (Phone), US2 (Photo), US4 (Activities) = Verified users
- 002-event-creation: US1 (Basic Event), US2 (Vibe Labels), US3 (Capacity) = Events exist
- 003-event-discovery: US1 (Browse), US4 (Details) = Users find events
- 005-rsvp-attendance: US1 (RSVP), US4 (My Events) = Users can commit

**MVP Task Count**: ~180 tasks (37% of total)

### Enhanced Product (Full Value)
Add vibe matching, waitlists, chat, and trust scoring:
- 004-vibe-matching: All user stories
- 005-rsvp-attendance: US2 (Cancel), US3 (Waitlist), US5 (Confirm), US6 (Trust)
- 006-group-chat: All user stories

## Shared Infrastructure

Tasks that appear across multiple features (build once, reuse):

| Component | Features Using It | Build During |
|-----------|-------------------|--------------|
| JWT Auth | All | 001 |
| PostgreSQL/TypeORM | All | 001 |
| S3/CloudFront | 001, potentially others | 001 |
| Push Notifications (SNS) | 005, 006 | 005 |
| Redis Caching | 003, 004, 005 | 003 |
| Chat Provider SDK | 006 | 006 |

## Notes

- Each feature has its own `tasks.md` with internal dependencies documented
- Tasks marked `[P]` within each feature can run in parallel
- User story tasks marked `[US#]` enable traceability
- Stop at checkpoints to validate before continuing
- Commit after each task or logical group
