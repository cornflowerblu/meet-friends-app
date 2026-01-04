# Tasks: Event Discovery

**Input**: Design documents from `/specs/003-event-discovery/`
**Prerequisites**: plan.md (required), spec.md (required for user stories)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **API**: `api/src/modules/events/`
- **Mobile**: `mobile/src/screens/`, `mobile/src/components/events/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and database setup for geospatial queries

- [ ] T001 Create events module structure at `api/src/modules/events/events.module.ts`
- [ ] T002 [P] Create DTO directory structure at `api/src/modules/events/dto/`
- [ ] T003 [P] Create mobile events components directory at `mobile/src/components/events/`
- [ ] T004 [P] Create mobile hooks directory structure at `mobile/src/hooks/`
- [ ] T005 [P] Create mobile services directory at `mobile/src/services/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

- [ ] T006 Add PostGIS extension migration at `api/src/database/migrations/`
- [ ] T007 Create geospatial index on events table location column in `api/src/database/migrations/`
- [ ] T008 [P] Create base event response DTO at `api/src/modules/events/dto/event-response.dto.ts`
- [ ] T009 [P] Create event feed query DTO at `api/src/modules/events/dto/event-feed-query.dto.ts`
- [ ] T010 [P] Create location service for geolocation handling at `mobile/src/services/locationService.ts`
- [ ] T011 [P] Create base event service for API client at `mobile/src/services/eventService.ts`
- [ ] T012 Create events controller skeleton at `api/src/modules/events/events.controller.ts`
- [ ] T013 Create events service skeleton at `api/src/modules/events/events.service.ts`
- [ ] T014 [P] Configure Redis caching layer for event feed in events module

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Browse Nearby Events (Priority: P1)

**Goal**: Display a personalized list of upcoming events near the user's location, prioritized by their activity interests

**Independent Test**: View the event feed and confirm relevant events appear sorted by date within relevance tiers

### Backend Implementation for User Story 1

- [ ] T015 [US1] Implement geospatial feed query with ST_DWithin in `api/src/modules/events/events.service.ts`
- [ ] T016 [US1] Implement event ranking algorithm (Tier 1: matching interests, Tier 2: other activities) in `api/src/modules/events/events.service.ts`
- [ ] T017 [US1] Add pagination with cursor-based scrolling to GET /events endpoint in `api/src/modules/events/events.controller.ts`
- [ ] T018 [US1] Implement Redis caching for event feed with 5-minute TTL in `api/src/modules/events/events.service.ts`
- [ ] T019 [US1] Create spots remaining calculation logic in `api/src/modules/events/events.service.ts`

### Mobile Implementation for User Story 1

- [ ] T020 [P] [US1] Create EventCard component at `mobile/src/components/events/EventCard.tsx`
- [ ] T021 [P] [US1] Create EventCardSkeleton loading component at `mobile/src/components/events/EventCardSkeleton.tsx`
- [ ] T022 [P] [US1] Create EmptyState component for no events at `mobile/src/components/events/EmptyState.tsx`
- [ ] T023 [US1] Create EventList component with infinite scroll at `mobile/src/components/events/EventList.tsx`
- [ ] T024 [US1] Create useEvents hook for feed data fetching at `mobile/src/hooks/useEvents.ts`
- [ ] T025 [US1] Create EventFeedScreen with pull-to-refresh at `mobile/src/screens/EventFeedScreen.tsx`
- [ ] T026 [US1] Implement background polling (30s interval) for feed updates in `mobile/src/hooks/useEvents.ts`

**Checkpoint**: User Story 1 complete - users can browse nearby events with personalized ordering

---

## Phase 4: User Story 2 - Filter Events by Activity (Priority: P2)

**Goal**: Allow users to filter the event feed by activity type to find specific events

**Independent Test**: Apply an activity filter and verify only matching events appear

### Backend Implementation for User Story 2

- [ ] T027 [US2] Create event filter DTO at `api/src/modules/events/dto/event-filter.dto.ts`
- [ ] T028 [US2] Implement activity filtering with OR semantics in `api/src/modules/events/events.service.ts`
- [ ] T029 [US2] Create GET /activities endpoint with event counts at `api/src/modules/events/events.controller.ts`
- [ ] T030 [US2] Add Redis caching for activity counts in `api/src/modules/events/events.service.ts`

### Mobile Implementation for User Story 2

- [ ] T031 [P] [US2] Create ActivityFilterChips component at `mobile/src/components/events/ActivityFilterChips.tsx`
- [ ] T032 [US2] Create useFilters hook for filter state management at `mobile/src/hooks/useFilters.ts`
- [ ] T033 [US2] Create FilterSheet modal component at `mobile/src/screens/FilterSheet.tsx`
- [ ] T034 [US2] Integrate activity filters with EventFeedScreen at `mobile/src/screens/EventFeedScreen.tsx`
- [ ] T035 [US2] Add clear filter functionality to FilterSheet at `mobile/src/screens/FilterSheet.tsx`

**Checkpoint**: User Story 2 complete - users can filter events by activity type

---

## Phase 5: User Story 3 - Filter Events by Vibe (Priority: P3)

**Goal**: Allow users to filter events by vibe labels to find events matching their preferred atmosphere

**Independent Test**: Apply vibe filters and verify events with matching labels appear

### Backend Implementation for User Story 3

- [ ] T036 [US3] Implement vibe filtering with OR semantics in `api/src/modules/events/events.service.ts`
- [ ] T037 [US3] Create GET /vibes endpoint for available vibe labels at `api/src/modules/events/events.controller.ts`
- [ ] T038 [US3] Handle combined activity + vibe filter queries in `api/src/modules/events/events.service.ts`

### Mobile Implementation for User Story 3

- [ ] T039 [P] [US3] Create VibeFilterChips component at `mobile/src/components/events/VibeFilterChips.tsx`
- [ ] T040 [US3] Add vibe filter support to useFilters hook at `mobile/src/hooks/useFilters.ts`
- [ ] T041 [US3] Integrate VibeFilterChips into FilterSheet at `mobile/src/screens/FilterSheet.tsx`
- [ ] T042 [US3] Add no-results messaging when filters yield nothing at `mobile/src/components/events/EmptyState.tsx`

**Checkpoint**: User Story 3 complete - users can filter events by vibe labels

---

## Phase 6: User Story 4 - View Event Details (Priority: P4)

**Goal**: Display full event information including host profile, attendee preview, and vibe context

**Independent Test**: Open an event and verify all expected details are displayed

### Backend Implementation for User Story 4

- [ ] T043 [US4] Implement GET /events/:id endpoint with host profile aggregation at `api/src/modules/events/events.controller.ts`
- [ ] T044 [US4] Add attendee preview query (first N RSVPs with photos) in `api/src/modules/events/events.service.ts`
- [ ] T045 [US4] Include host history (events hosted count) in event detail response in `api/src/modules/events/events.service.ts`
- [ ] T046 [US4] Add real-time availability data (spots remaining, cancellation status) in `api/src/modules/events/events.service.ts`

### Mobile Implementation for User Story 4

- [ ] T047 [US4] Create useEventDetail hook for single event fetching at `mobile/src/hooks/useEventDetail.ts`
- [ ] T048 [US4] Create EventDetailScreen with full event info at `mobile/src/screens/EventDetailScreen.tsx`
- [ ] T049 [US4] Add host profile section (photo, name, hosting history) to EventDetailScreen at `mobile/src/screens/EventDetailScreen.tsx`
- [ ] T050 [US4] Add attendee preview section with RSVP photos to EventDetailScreen at `mobile/src/screens/EventDetailScreen.tsx`
- [ ] T051 [US4] Add vibe labels and custom vibe message display to EventDetailScreen at `mobile/src/screens/EventDetailScreen.tsx`
- [ ] T052 [US4] Handle event cancellation display state in EventDetailScreen at `mobile/src/screens/EventDetailScreen.tsx`

**Checkpoint**: User Story 4 complete - users can view full event details with host and attendee info

---

## Phase 7: User Story 5 - Search for Events (Priority: P5)

**Goal**: Enable keyword search across activity, location, and host name

**Independent Test**: Enter search terms and verify matching events appear

### Backend Implementation for User Story 5

- [ ] T053 [US5] Create event search DTO at `api/src/modules/events/dto/event-search.dto.ts`
- [ ] T054 [US5] Implement GET /events/search with PostgreSQL full-text search at `api/src/modules/events/events.controller.ts`
- [ ] T055 [US5] Add search across title, description, location, and host name fields in `api/src/modules/events/events.service.ts`
- [ ] T056 [US5] Combine search results with geospatial filtering in `api/src/modules/events/events.service.ts`

### Mobile Implementation for User Story 5

- [ ] T057 [P] [US5] Create useEventSearch hook with debounced search at `mobile/src/hooks/useEventSearch.ts`
- [ ] T058 [US5] Create EventSearchScreen with search bar at `mobile/src/screens/EventSearchScreen.tsx`
- [ ] T059 [US5] Implement local search history storage in EventSearchScreen at `mobile/src/screens/EventSearchScreen.tsx`
- [ ] T060 [US5] Add search history display with quick re-access in EventSearchScreen at `mobile/src/screens/EventSearchScreen.tsx`
- [ ] T061 [US5] Add no-results state with suggestions in EventSearchScreen at `mobile/src/screens/EventSearchScreen.tsx`

**Checkpoint**: User Story 5 complete - users can search for events using keywords

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T062 [P] Add offline caching for event feed data in `mobile/src/services/eventService.ts`
- [ ] T063 [P] Implement show_full_events toggle for capacity filtering in feed query at `api/src/modules/events/events.service.ts`
- [ ] T064 [P] Add waitlist suggestion messaging for full events in `mobile/src/components/events/EventCard.tsx`
- [ ] T065 Handle location unavailable state with manual location prompt at `mobile/src/screens/EventFeedScreen.tsx`
- [ ] T066 Add navigation integration between EventFeedScreen, EventDetailScreen, and EventSearchScreen

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-7)**: All depend on Foundational phase completion
  - User stories can proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 -> P2 -> P3 -> P4 -> P5)
- **Polish (Phase 8)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Extends US1 filter capabilities
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Extends US2 filter capabilities
- **User Story 4 (P4)**: Can start after Foundational (Phase 2) - Uses EventCard from US1
- **User Story 5 (P5)**: Can start after Foundational (Phase 2) - Independent search flow

### Within Each User Story

- Backend tasks before mobile tasks that consume the API
- DTOs before services
- Services before controllers
- Core components before screen integration
- Commit after each task or logical group

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel
- Mobile component tasks marked [P] can be built in parallel
- Different user stories can be worked on in parallel by different team members

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (Browse Nearby Events)
4. **STOP and VALIDATE**: Test event feed with location-based filtering
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational -> Foundation ready
2. Add User Story 1 -> Test independently -> Deploy/Demo (MVP!)
3. Add User Story 2 -> Activity filtering works -> Deploy/Demo
4. Add User Story 3 -> Vibe filtering works -> Deploy/Demo
5. Add User Story 4 -> Event details viewable -> Deploy/Demo
6. Add User Story 5 -> Search functionality complete -> Deploy/Demo
7. Each story adds value without breaking previous stories

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Redis caching is critical for feed performance (<2 second load time requirement)
- PostGIS ST_DWithin queries require proper geospatial indexes
