# Implementation Plan: Event Creation

**Branch**: `002-event-creation` | **Date**: 2026-01-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/002-event-creation/spec.md`

## Summary

Implement the Event Creation feature enabling onboarded users to create, edit, and cancel public events with activity selection, date/time scheduling, location, vibe labels, and capacity limits. The technical approach uses React Native (Expo) for mobile screens, NestJS for backend APIs, and PostgreSQL for data persistence. Events support recurring schedules (weekly/biweekly/monthly) and templates for frequent hosts.

## Technical Context

**Language/Version**: TypeScript 5.x (Mobile + Backend)
**Primary Dependencies**: React Native (Expo SDK 50+), NestJS 10.x, TypeORM
**Storage**: PostgreSQL (RDS/Aurora) for events, vibe labels, templates, and series
**Testing**: Jest (unit/integration), Detox for E2E mobile tests
**Target Platform**: iOS 15+, Android API 26+, with backend on ECS Fargate
**Project Type**: Mobile + API (React Native app + REST backend)
**Performance Goals**: Event creation completes in under 3 minutes (per SC-001), API responses under 200ms p95
**Constraints**: Monthly burn under $500-800 (per constitution), no per-message LLM calls
**Scale/Scope**: MVP targeting 10k users, ~1000 events/month initially

## Constitution Check

| Principle | Check | Status |
|-----------|-------|--------|
| I. IRL-First | Events are the core product enabling real meetups. Default 60-90 min duration, 4-8 people capacity, public events only. | PASS |
| II. Trust & Safety | Host must be verified (phone + photo). "Not a dating app" framing enforced. Host reliability tracking implemented. | PASS |
| III. Vibe Compatibility | Vibe labels prominently displayed on events. Labels curated per activity type. | PASS |
| IV. AI as Support | No AI required for event creation MVP. | PASS |
| V. Cost-Conscious MVP | Rule-based implementation. No ML for MVP. Uses approved stack. | PASS |

## Project Structure

### Source Code

```text
api/
├── src/
│   ├── modules/
│   │   ├── events/
│   │   │   ├── events.controller.ts
│   │   │   ├── events.service.ts
│   │   │   ├── events.module.ts
│   │   │   └── dto/
│   │   ├── vibe-labels/
│   │   │   ├── vibe-labels.controller.ts
│   │   │   └── vibe-labels.service.ts
│   │   └── templates/
│   │       ├── templates.controller.ts
│   │       └── templates.service.ts
│   ├── entities/
│   │   ├── event.entity.ts
│   │   ├── event-series.entity.ts
│   │   ├── event-template.entity.ts
│   │   └── vibe-label.entity.ts
│   └── common/
│       └── guards/
└── tests/

mobile/
├── src/
│   ├── screens/
│   │   └── events/
│   │       ├── CreateEventScreen.tsx
│   │       ├── ActivitySelectScreen.tsx
│   │       ├── DateTimeScreen.tsx
│   │       ├── LocationScreen.tsx
│   │       ├── VibeSelectScreen.tsx
│   │       ├── CapacityScreen.tsx
│   │       ├── ReviewPublishScreen.tsx
│   │       ├── EditEventScreen.tsx
│   │       └── MyEventsScreen.tsx
│   ├── components/
│   │   └── events/
│   │       ├── VibeChip.tsx
│   │       ├── CapacityPicker.tsx
│   │       ├── LocationInput.tsx
│   │       └── RecurringOptions.tsx
│   └── services/
│       └── eventService.ts
└── tests/
```

## Data Model Design

### Event Entity

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | PK |
| host_id | UUID | FK -> User, NOT NULL |
| activity_id | UUID | FK -> Activity, NOT NULL |
| title | VARCHAR(100) | NOT NULL |
| description | TEXT | NULLABLE |
| date_time | TIMESTAMP | NOT NULL, >= NOW() + 2hrs |
| duration_minutes | INTEGER | NOT NULL, DEFAULT 90 |
| location_name | VARCHAR(200) | NOT NULL |
| location_address | VARCHAR(500) | NULLABLE |
| location_lat | DECIMAL(10,7) | NULLABLE |
| location_lng | DECIMAL(10,7) | NULLABLE |
| min_capacity | INTEGER | NOT NULL, DEFAULT 4, MIN 2 |
| max_capacity | INTEGER | NOT NULL, DEFAULT 8, MAX 20 |
| custom_vibe_text | VARCHAR(200) | NULLABLE |
| status | ENUM | draft/published/canceled/completed |
| series_id | UUID | FK -> EventSeries, NULLABLE |
| canceled_reason | VARCHAR(500) | NULLABLE |

### VibeLabel Entity

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | PK |
| name | VARCHAR(50) | NOT NULL, UNIQUE |
| display_name | VARCHAR(50) | NOT NULL |
| category | ENUM | skill/energy/social |
| icon | VARCHAR(50) | NULLABLE |
| is_active | BOOLEAN | DEFAULT true |

### EventSeries Entity

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | PK |
| host_id | UUID | FK -> User, NOT NULL |
| recurrence_type | ENUM | weekly/biweekly/monthly |
| day_of_week | INTEGER | 0-6 for weekly/biweekly |
| start_date | DATE | NOT NULL |
| end_date | DATE | NULLABLE |
| base_event_template | JSONB | NOT NULL |

### EventTemplate Entity

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | PK |
| host_id | UUID | FK -> User, NOT NULL |
| name | VARCHAR(100) | NOT NULL |
| activity_id | UUID | FK -> Activity |
| duration_minutes | INTEGER | NOT NULL |
| location_name | VARCHAR(200) | NULLABLE |
| min_capacity | INTEGER | NOT NULL |
| max_capacity | INTEGER | NOT NULL |
| vibe_label_ids | UUID[] | NOT NULL |

## API Contract Design

### Create Event

```yaml
POST /api/v1/events
Request:
  activity_id: uuid
  date_time: ISO8601
  duration_minutes: 90
  location: { name, address?, lat?, lng? }
  vibe_label_ids: [uuid, uuid]
  custom_vibe_text?: string
  min_capacity: 4
  max_capacity: 8
  recurrence?: { type: weekly, end_date?: date }

Response 201:
  id: uuid
  status: published
  host: { id, name, photo_url }
  activity: { id, name, icon }
  ...

Errors:
  400: PAST_DATE, INVALID_CAPACITY
  403: UNVERIFIED_HOST
```

### Get Vibe Labels

```yaml
GET /api/v1/activities/{activity_id}/vibe-labels
Response 200:
  labels: [{ id, name, display_name, category, icon, is_default }]
```

### Update Event

```yaml
PATCH /api/v1/events/{event_id}
Request: { date_time?, location?, notify_attendees: true }
Response 200: { ...updated event }
```

### Cancel Event

```yaml
POST /api/v1/events/{event_id}/cancel
Request: { reason?: string }
Response 200: { id, status: canceled, attendees_notified: 5 }
```

### Templates

```yaml
POST /api/v1/event-templates
Request: { name, activity_id, duration_minutes, location?, vibe_label_ids, capacities }
Response 201: { ...template }

POST /api/v1/event-templates/{template_id}/create-event
Request: { date_time }
Response 201: { ...event }
```

## Implementation Phases

### Phase 1: Core Event CRUD (P1)
1. Create Event entity and database migration
2. Implement VibeLabel entity with seed data
3. Create EventsService with create, read, update operations
4. Implement EventsController with validation
5. Add verification guard (phone + photo required)
6. Build CreateEventScreen flow in mobile app

### Phase 2: Vibe Labels (P2)
1. Seed ActivityVibeLabel relationships
2. Create GET /activities/{id}/vibe-labels endpoint
3. Build VibeSelectScreen with activity-specific labels
4. Implement VibeChip component with multi-select (1-3)

### Phase 3: Capacity Limits (P3)
1. Add capacity validation (min 2, max 20, defaults 4-8)
2. Build CapacityPicker component
3. Display capacity on event listings

### Phase 4: Edit and Cancel (P4)
1. Implement PATCH /events/{id} endpoint
2. Implement POST /events/{id}/cancel endpoint
3. Build EditEventScreen
4. Add attendee notification trigger

### Phase 5: Recurring and Templates (P5)
1. Create EventSeries entity and migration
2. Implement recurring event creation logic
3. Create EventTemplate entity
4. Build "Create Similar" and "My Templates" features

## Host Reliability Tracking (FR-012)

Track per host:
- events_created: Count of all events
- events_completed: Events that reached completion
- events_canceled_with_rsvps: Events canceled after having attendees

Reliability score = events_completed / (events_completed + events_canceled_with_rsvps)

## Dependencies

**Requires**:
- 001-user-onboarding: Verified user, Activity entity

**Provides to**:
- 003-event-discovery: Published events for discovery
- 005-rsvp-attendance: Event entity for RSVPs
