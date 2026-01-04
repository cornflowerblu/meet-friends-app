# Tasks: Event Creation

**Input**: Design documents from `/specs/002-event-creation/`
**Prerequisites**: plan.md (required), spec.md (required for user stories)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **API/Backend**: `api/src/` (NestJS, TypeORM)
- **Mobile**: `mobile/src/` (React Native, Expo)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and module structure for event creation

- [ ] T001 Create events module structure at `api/src/modules/events/`
- [ ] T002 [P] Create vibe-labels module structure at `api/src/modules/vibe-labels/`
- [ ] T003 [P] Create templates module structure at `api/src/modules/templates/`
- [ ] T004 [P] Create events screen folder structure at `mobile/src/screens/events/`
- [ ] T005 [P] Create events components folder at `mobile/src/components/events/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core entities and database migrations that MUST be complete before ANY user story can be implemented

**CRITICAL**: No user story work can begin until this phase is complete

### Database Entities & Migrations

- [ ] T006 Create Event entity with all fields in `api/src/entities/event.entity.ts` (id, host_id, activity_id, title, description, date_time, duration_minutes, location fields, capacity fields, custom_vibe_text, status, series_id, canceled_reason)
- [ ] T007 [P] Create VibeLabel entity in `api/src/entities/vibe-label.entity.ts` (id, name, display_name, category, icon, is_active)
- [ ] T008 [P] Create EventVibeLabel join table entity in `api/src/entities/event-vibe-label.entity.ts`
- [ ] T009 Generate database migration for Event and VibeLabel entities
- [ ] T010 Create seed data for curated vibe labels in `api/src/database/seeds/vibe-labels.seed.ts`

### Shared Services & Guards

- [ ] T011 Create verified host guard in `api/src/common/guards/verified-host.guard.ts` (requires phone + photo per FR-004)
- [ ] T012 [P] Create event validation utilities in `api/src/modules/events/utils/event-validation.ts` (date >= now + 2hrs, capacity ranges)

### Mobile Service Layer

- [ ] T013 Create eventService with API client methods in `mobile/src/services/eventService.ts`

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Create a Basic Event (Priority: P1) MVP

**Goal**: Enable onboarded users to create events with activity, date/time, location and publish for discovery

**Independent Test**: Create an event and verify it appears in the system, visible to other users

### Backend Implementation for US1

- [ ] T014 [US1] Create CreateEventDto in `api/src/modules/events/dto/create-event.dto.ts` with validation decorators
- [ ] T015 [P] [US1] Create EventResponseDto in `api/src/modules/events/dto/event-response.dto.ts`
- [ ] T016 [US1] Implement EventsService.create() method in `api/src/modules/events/events.service.ts`
- [ ] T017 [US1] Implement EventsService.findOne() method in `api/src/modules/events/events.service.ts`
- [ ] T018 [US1] Implement EventsService.findByHost() method in `api/src/modules/events/events.service.ts`
- [ ] T019 [US1] Create EventsController with POST /events endpoint in `api/src/modules/events/events.controller.ts`
- [ ] T020 [US1] Add GET /events/:id endpoint to EventsController
- [ ] T021 [US1] Add GET /events/my-events endpoint to EventsController (host's events)
- [ ] T022 [US1] Register EventsModule in `api/src/modules/events/events.module.ts` with all dependencies

### Mobile Implementation for US1

- [ ] T023 [US1] Create CreateEventScreen entry point in `mobile/src/screens/events/CreateEventScreen.tsx`
- [ ] T024 [P] [US1] Create ActivitySelectScreen in `mobile/src/screens/events/ActivitySelectScreen.tsx` (select from user's interests)
- [ ] T025 [P] [US1] Create DateTimeScreen in `mobile/src/screens/events/DateTimeScreen.tsx` (date, time, duration with 90min default)
- [ ] T026 [P] [US1] Create LocationInput component in `mobile/src/components/events/LocationInput.tsx` (venue name, address, or general area)
- [ ] T027 [US1] Create LocationScreen in `mobile/src/screens/events/LocationScreen.tsx` using LocationInput component
- [ ] T028 [US1] Create ReviewPublishScreen in `mobile/src/screens/events/ReviewPublishScreen.tsx` (summary and publish button)
- [ ] T029 [US1] Wire up event creation flow navigation in CreateEventScreen
- [ ] T030 [US1] Implement createEvent API call in eventService and connect to ReviewPublishScreen

**Checkpoint**: User Story 1 complete - users can create and publish basic events

---

## Phase 4: User Story 2 - Set Event Vibe Labels (Priority: P2)

**Goal**: Allow event creators to add curated vibe labels to communicate expected atmosphere

**Independent Test**: Add vibe labels to an event and verify they display on the event listing

### Backend Implementation for US2

- [ ] T031 [US2] Create ActivityVibeLabel join table entity in `api/src/entities/activity-vibe-label.entity.ts` (activity-specific labels)
- [ ] T032 [US2] Create seed data for activity-vibe-label relationships in `api/src/database/seeds/activity-vibe-labels.seed.ts`
- [ ] T033 [US2] Implement VibeLabelsService in `api/src/modules/vibe-labels/vibe-labels.service.ts` with findByActivity() method
- [ ] T034 [US2] Create VibeLabelsController with GET /activities/:activityId/vibe-labels in `api/src/modules/vibe-labels/vibe-labels.controller.ts`
- [ ] T035 [US2] Register VibeLabelsModule in `api/src/modules/vibe-labels/vibe-labels.module.ts`
- [ ] T036 [US2] Update EventsService.create() to handle vibe_label_ids array and custom_vibe_text

### Mobile Implementation for US2

- [ ] T037 [US2] Create VibeChip component in `mobile/src/components/events/VibeChip.tsx` (selectable chip with icon)
- [ ] T038 [US2] Create VibeSelectScreen in `mobile/src/screens/events/VibeSelectScreen.tsx` (multi-select 1-3 labels, custom text input)
- [ ] T039 [US2] Add getVibeLabels API call to eventService for fetching activity-specific labels
- [ ] T040 [US2] Integrate VibeSelectScreen into event creation flow after LocationScreen
- [ ] T041 [US2] Display vibe labels prominently in ReviewPublishScreen

**Checkpoint**: User Story 2 complete - events can have vibe labels attached and displayed

---

## Phase 5: User Story 3 - Set Capacity Limits (Priority: P3)

**Goal**: Allow event creators to set min/max participant limits with smart defaults (4-8)

**Independent Test**: Set capacity limits and verify RSVPs are limited accordingly

### Backend Implementation for US3

- [ ] T042 [US3] Add capacity validation logic to EventsService (min: 2, max: 20, defaults: 4-8)
- [ ] T043 [US3] Add current_rsvp_count and remaining_spots computed fields to EventResponseDto
- [ ] T044 [US3] Implement capacity display data in event listings (attendance count, remaining spots)

### Mobile Implementation for US3

- [ ] T045 [US3] Create CapacityPicker component in `mobile/src/components/events/CapacityPicker.tsx` (slider or stepper for min/max)
- [ ] T046 [US3] Create CapacityScreen in `mobile/src/screens/events/CapacityScreen.tsx` with defaults and validation
- [ ] T047 [US3] Integrate CapacityScreen into event creation flow after VibeSelectScreen
- [ ] T048 [US3] Display capacity info (current/max) in ReviewPublishScreen

**Checkpoint**: User Story 3 complete - events have controlled capacity with visible attendance info

---

## Phase 6: User Story 4 - Edit and Cancel Events (Priority: P4)

**Goal**: Enable hosts to modify event details or cancel with proper attendee notifications

**Independent Test**: Edit event details and cancel an event, verify attendees are notified

### Backend Implementation for US4

- [ ] T049 [US4] Create UpdateEventDto in `api/src/modules/events/dto/update-event.dto.ts` with partial validation
- [ ] T050 [US4] Implement EventsService.update() method with change detection
- [ ] T051 [US4] Implement EventsService.cancel() method with reason support
- [ ] T052 [US4] Add PATCH /events/:id endpoint to EventsController
- [ ] T053 [US4] Add POST /events/:id/cancel endpoint to EventsController
- [ ] T054 [US4] Create attendee notification trigger service in `api/src/modules/events/services/event-notification.service.ts`
- [ ] T055 [US4] Integrate notification service with update and cancel operations

### Mobile Implementation for US4

- [ ] T056 [US4] Create EditEventScreen in `mobile/src/screens/events/EditEventScreen.tsx` (edit date, time, location, vibes, capacity)
- [ ] T057 [US4] Create MyEventsScreen in `mobile/src/screens/events/MyEventsScreen.tsx` (list hosted events with edit/cancel actions)
- [ ] T058 [US4] Add event cancellation confirmation modal with optional reason input
- [ ] T059 [US4] Add updateEvent and cancelEvent API calls to eventService
- [ ] T060 [US4] Display canceled events with status and reason in MyEventsScreen

**Checkpoint**: User Story 4 complete - hosts can manage their events with proper notifications

---

## Phase 7: User Story 5 - Repeat and Template Events (Priority: P5)

**Goal**: Enable frequent hosts to create recurring events and save/reuse event templates

**Independent Test**: Create a template and use it to spawn a new event

### Backend Implementation for US5

- [ ] T061 [US5] Create EventSeries entity in `api/src/entities/event-series.entity.ts` (recurrence_type, day_of_week, dates, base_template)
- [ ] T062 [P] [US5] Create EventTemplate entity in `api/src/entities/event-template.entity.ts` (name, activity, duration, location, capacities, vibe_label_ids)
- [ ] T063 [US5] Generate database migration for EventSeries and EventTemplate entities
- [ ] T064 [US5] Implement recurring event creation logic in `api/src/modules/events/services/event-series.service.ts` (weekly/biweekly/monthly)
- [ ] T065 [US5] Implement TemplatesService in `api/src/modules/templates/templates.service.ts` with CRUD operations
- [ ] T066 [US5] Create TemplatesController with POST /event-templates endpoint in `api/src/modules/templates/templates.controller.ts`
- [ ] T067 [US5] Add POST /event-templates/:id/create-event endpoint to create event from template
- [ ] T068 [US5] Add GET /event-templates endpoint for user's templates
- [ ] T069 [US5] Register TemplatesModule in `api/src/modules/templates/templates.module.ts`
- [ ] T070 [US5] Add recurrence options to CreateEventDto and EventsService.create()

### Mobile Implementation for US5

- [ ] T071 [US5] Create RecurringOptions component in `mobile/src/components/events/RecurringOptions.tsx` (weekly/biweekly/monthly toggle)
- [ ] T072 [US5] Integrate RecurringOptions into event creation flow (optional step after CapacityScreen)
- [ ] T073 [US5] Add "Create Similar" action to completed events in MyEventsScreen
- [ ] T074 [US5] Create MyTemplatesScreen in `mobile/src/screens/events/MyTemplatesScreen.tsx` (list and launch templates)
- [ ] T075 [US5] Add "Save as Template" option in ReviewPublishScreen
- [ ] T076 [US5] Add template API calls to eventService (create, list, createEventFromTemplate)

**Checkpoint**: User Story 5 complete - power users can efficiently create recurring events and templates

---

## Phase 8: Polish and Cross-Cutting Concerns

**Purpose**: Host reliability tracking, final validation, and quality improvements

- [ ] T077 [P] Implement host reliability tracking fields in User entity (events_created, events_completed, events_canceled_with_rsvps)
- [ ] T078 [P] Add host reliability score calculation (events_completed / total) in EventsService
- [ ] T079 Add "not a dating app" framing guidance in event creation UI (FR-011)
- [ ] T080 [P] Add API response time logging for performance monitoring (<200ms p95 goal)
- [ ] T081 Validate full event creation flow completes in under 3 minutes (SC-001)
- [ ] T082 Code cleanup and ensure consistent error handling across all endpoints

---

## Dependencies and Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-7)**: All depend on Foundational phase completion
  - User stories can proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 -> P2 -> P3 -> P4 -> P5)
- **Polish (Phase 8)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Extends event creation with vibe labels
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Extends event creation with capacity
- **User Story 4 (P4)**: Depends on US1 completion (needs events to exist to edit/cancel)
- **User Story 5 (P5)**: Depends on US1 completion (needs event creation for templates/series)

### External Dependencies

- **Requires**: 001-user-onboarding (Verified user, Activity entity)
- **Provides to**: 003-event-discovery, 005-rsvp-attendance

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, US1, US2, US3 can start in parallel
- Within each user story, tasks marked [P] can run in parallel
- Backend and mobile implementation for each story can proceed in parallel

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- All events are public (no private/invite-only for MVP)
- Default duration is 90 minutes, default capacity is 4-8 people
