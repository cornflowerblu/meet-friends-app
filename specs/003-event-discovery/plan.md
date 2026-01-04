# Implementation Plan: Event Discovery

**Branch**: `003-event-discovery` | **Date**: 2026-01-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/003-event-discovery/spec.md`

## Summary

The Event Discovery feature enables onboarded users to browse, filter, and search for events based on location, activity interests, and vibe preferences. The implementation requires a personalized event feed API with geospatial queries, a filter/search system, and mobile UI components for event cards and details. This feature is the critical bridge from "app user" to "event attendee" and directly supports the IRL-First constitution principle.

## Technical Context

**Language/Version**: TypeScript 5.x (React Native + Node.js)
**Primary Dependencies**: React Native 0.81 (Expo SDK 54), React 19.1, NestJS 11.x, PostgreSQL with PostGIS
**Storage**: PostgreSQL (RDS) with geospatial extensions, Redis for caching
**Testing**: Jest (API unit/integration), React Native Testing Library (mobile)
**Target Platform**: iOS 15.1+, Android API 24+ (targeting API 36), Linux server (ECS Fargate)
**Project Type**: Mobile + API
**Performance Goals**: Event feed loads in <2 seconds (SC-007), support 10k concurrent users MVP
**Constraints**: <$500-800/mo infrastructure, no per-message LLM calls, offline-capable feed caching
**Scale/Scope**: Initial 10k users, ~50 events/region, 6 screens

## Constitution Check

| Principle | Status | Rationale |
|-----------|--------|-----------|
| I. IRL-First | PASS | Feature directly drives event attendance - the core product |
| II. Trust & Safety | PASS | Shows only public events, enforces group-first meetups |
| III. Vibe Compatibility | PASS | Prioritizes events by activity interests, supports vibe filtering |
| IV. AI as Support | PASS | No AI required for MVP discovery; rule-based ranking sufficient |
| V. Cost-Conscious MVP | PASS | Uses PostGIS (free), Redis caching, no LLM calls |

## Project Structure

### Source Code

```text
api/
├── src/
│   ├── modules/
│   │   └── events/
│   │       ├── events.module.ts
│   │       ├── events.controller.ts
│   │       ├── events.service.ts
│   │       └── dto/
│   │           ├── event-feed-query.dto.ts
│   │           ├── event-filter.dto.ts
│   │           ├── event-search.dto.ts
│   │           └── event-response.dto.ts
│   └── database/
│       └── migrations/
└── tests/

mobile/
├── src/
│   ├── screens/
│   │   ├── EventFeedScreen.tsx
│   │   ├── EventDetailScreen.tsx
│   │   ├── EventSearchScreen.tsx
│   │   └── FilterSheet.tsx
│   ├── components/
│   │   └── events/
│   │       ├── EventCard.tsx
│   │       ├── EventCardSkeleton.tsx
│   │       ├── EventList.tsx
│   │       ├── EventFilters.tsx
│   │       ├── VibeFilterChips.tsx
│   │       ├── ActivityFilterChips.tsx
│   │       └── EmptyState.tsx
│   ├── services/
│   │   ├── eventService.ts
│   │   └── locationService.ts
│   └── hooks/
│       ├── useEvents.ts
│       ├── useEventDetail.ts
│       ├── useEventSearch.ts
│       └── useFilters.ts
└── tests/
```

## Implementation Approach

### Phase 0: Research & Validation

1. **Geospatial Query Strategy**: PostGIS with `ST_DWithin` for radius queries
2. **Feed Ranking Algorithm**:
   - Tier 1: Events matching user's activity interests
   - Tier 2: Events in user's area but different activities
   - Within tiers: Sort by date (soonest first)
3. **Real-time Updates**: Pull-to-refresh + background polling (30s interval) for MVP

### Phase 1: Data Model & API Design

**API Endpoints**:

| Endpoint | Method | Purpose | Query Params |
|----------|--------|---------|--------------|
| `/events` | GET | Personalized event feed | `lat`, `lng`, `radius`, `activities[]`, `vibes[]`, `show_full`, `cursor`, `limit` |
| `/events/:id` | GET | Event detail with host and attendees | - |
| `/events/search` | GET | Keyword search | `q`, `lat`, `lng`, `radius`, `cursor`, `limit` |
| `/activities` | GET | List activities with event counts | `lat`, `lng`, `radius` |
| `/vibes` | GET | List vibe labels | - |

### Phase 2: Implementation Sequence

**Backend Tasks**:
1. Database migration: Add PostGIS extension, create geospatial index
2. Event Feed Service: Personalized feed with geo-filtering and interest ranking
3. Filter Logic: Activity and vibe filtering with OR semantics
4. Search Service: Full-text search on title, description, location, host name
5. Event Detail Aggregation: Join host profile, attendee preview, RSVP counts
6. Caching Layer: Redis cache for activity counts, event cards (5-minute TTL)

**Mobile Tasks**:
1. Event Service Layer: API client with offline caching
2. Event Feed Screen: Infinite scroll list with pull-to-refresh
3. Event Card Component: Activity, date/time, location, vibes, spots remaining
4. Filter Sheet: Activity chips, vibe chips, apply/clear actions
5. Event Detail Screen: Full info, host profile, attendee preview
6. Search Screen: Search bar, history, results list
7. Empty States: No events, no results, no location

### Data Model Extensions

```
FilterState (session memory)
├── activities: string[]  -- selected activity IDs
├── vibes: string[]       -- selected vibe label IDs
├── distance_radius: number (miles, default 25)
└── show_full_events: boolean (default false)

SearchHistory (local storage)
├── query: string
├── timestamp: Date
└── user_id: string
```

## Testing Strategy

| Layer | Type | Coverage |
|-------|------|----------|
| API | Unit | Event ranking algorithm, filter logic, distance calculation |
| API | Integration | Feed endpoint with filters, search results accuracy |
| Mobile | Component | Event card rendering, filter state management |
| Mobile | Screen | Feed loading states, filter application |
| E2E | Flow | User views feed -> applies filter -> opens detail |

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| PostGIS unavailable | Use application-level Haversine calculation |
| Slow feed under load | Redis caching for hot regions; pagination with cursor |
| Search accuracy | Start with PostgreSQL full-text; add Elasticsearch later if needed |
| Real-time sync delays | Accept 30s staleness for MVP; add WebSocket for v2 |

## Dependencies

**Requires**:
- 001-user-onboarding: User entity, location, activity interests
- 002-event-creation: Event entity, vibe labels

**Provides to**:
- 004-vibe-matching: Event feed to integrate compatibility badges
- 005-rsvp-attendance: Event detail view for RSVP actions
