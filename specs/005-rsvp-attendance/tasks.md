# Tasks: RSVP & Attendance

**Input**: Design documents from `/specs/005-rsvp-attendance/`
**Prerequisites**: plan.md (required), spec.md (required for user stories)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **API**: `api/src/modules/` for backend services
- **Mobile**: `mobile/src/features/` for React Native components
- **Database**: `api/src/database/migrations/` for schema changes

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and module structure

- [ ] T001 Create RSVP module directory structure at `api/src/modules/rsvp/`
- [ ] T002 [P] Create waitlist module directory structure at `api/src/modules/waitlist/`
- [ ] T003 [P] Create attendance module directory structure at `api/src/modules/attendance/`
- [ ] T004 [P] Create my-events module directory structure at `api/src/modules/my-events/`
- [ ] T005 [P] Create trust-signals module directory structure at `api/src/modules/trust-signals/`
- [ ] T006 [P] Create mobile RSVP feature directory at `mobile/src/features/rsvp/`
- [ ] T007 [P] Create mobile my-events feature directory at `mobile/src/features/my-events/`
- [ ] T008 [P] Create mobile confirmation feature directory at `mobile/src/features/confirmation/`
- [ ] T009 [P] Create mobile trust-signals feature directory at `mobile/src/features/trust-signals/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**CRITICAL**: No user story work can begin until this phase is complete

### Database Migrations

- [ ] T010 Create RSVP table migration at `api/src/database/migrations/create-rsvp-table.ts` with columns: id (UUID PK), user_id (FK), event_id (FK), status (ENUM), created_at, confirmed_at, canceled_at, cancellation_reason, UNIQUE(user_id, event_id)
- [ ] T011 [P] Create WaitlistPosition table migration at `api/src/database/migrations/create-waitlist-position-table.ts` with columns: id (UUID PK), user_id (FK), event_id (FK), position (INT), status (ENUM), claim_expires_at, created_at, notified_at, UNIQUE(user_id, event_id)
- [ ] T012 [P] Create TrustSignal table migration at `api/src/database/migrations/create-trust-signal-table.ts` with columns: id (UUID PK), rater_user_id (FK), rated_user_id (FK), event_id (FK), signal (ENUM), created_at, UNIQUE(rater_user_id, rated_user_id, event_id)
- [ ] T013 [P] Create UserReliability table migration at `api/src/database/migrations/create-user-reliability-table.ts` with columns: id (UUID PK), user_id (FK UNIQUE), total_rsvps, confirmed_attendances, no_shows, late_cancellations, reliability_score (DECIMAL), updated_at

### Core Entities

- [ ] T014 Create RSVP entity at `api/src/modules/rsvp/entities/rsvp.entity.ts` with status enum (pending, confirmed, canceled, no_show)
- [ ] T015 [P] Create WaitlistPosition entity at `api/src/modules/waitlist/entities/waitlist-position.entity.ts` with status enum (waiting, notified, claimed, expired, declined)
- [ ] T016 [P] Create AttendanceConfirmation entity at `api/src/modules/attendance/entities/attendance-confirmation.entity.ts`
- [ ] T017 [P] Create TrustSignal entity at `api/src/modules/trust-signals/entities/trust-signal.entity.ts` with signal enum (positive, negative)
- [ ] T018 [P] Create UserReliability entity at `api/src/modules/trust-signals/entities/user-reliability.entity.ts`

### Module Registration

- [ ] T019 Create RSVP module at `api/src/modules/rsvp/rsvp.module.ts` and register in app module
- [ ] T020 [P] Create Waitlist module at `api/src/modules/waitlist/waitlist.module.ts` and register in app module
- [ ] T021 [P] Create Attendance module at `api/src/modules/attendance/attendance.module.ts` and register in app module
- [ ] T022 [P] Create MyEvents module at `api/src/modules/my-events/my-events.module.ts` and register in app module
- [ ] T023 [P] Create TrustSignals module at `api/src/modules/trust-signals/trust-signals.module.ts` and register in app module

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - RSVP to an Event (Priority: P1)

**Goal**: One-tap RSVP with confirmation - the critical action that converts browsing into actual meetups

**Independent Test**: Can be tested by RSVPing to an event and confirming the user appears on the attendee list

### Backend Implementation

- [ ] T024 [US1] Implement RSVP service at `api/src/modules/rsvp/rsvp.service.ts` with createRSVP method that checks event capacity and creates RSVP with pending status
- [ ] T025 [US1] Add real-time attendee count update logic to RSVP service (increment on create, FR-002)
- [ ] T026 [US1] Add duplicate RSVP prevention logic to RSVP service (check existing RSVP before creating)
- [ ] T027 [US1] Implement RSVP controller at `api/src/modules/rsvp/rsvp.controller.ts` with POST /events/:eventId/rsvp endpoint
- [ ] T028 [US1] Add capacity check to RSVP creation - return 202 with waitlist option when full (FR-001)
- [ ] T029 [US1] Add getUserRSVPStatus method to RSVP service for checking if user already RSVPed

### Mobile Implementation

- [ ] T030 [P] [US1] Create useRSVP hook at `mobile/src/features/rsvp/hooks/useRSVP.ts` with createRSVP mutation and RSVP status query
- [ ] T031 [US1] Create RSVPButton component at `mobile/src/features/rsvp/components/RSVPButton.tsx` with states: "I'm In" (available), "You're Going" (RSVPed), loading state
- [ ] T032 [US1] Add haptic feedback and confirmation animation to RSVPButton on successful RSVP
- [ ] T033 [US1] Integrate RSVPButton into Event detail view with real-time status updates
- [ ] T034 [US1] Add optimistic UI update for instant RSVP feedback (< 2 seconds per SC-001)

**Checkpoint**: User Story 1 complete - users can RSVP to events with one tap

---

## Phase 4: User Story 2 - Cancel RSVP (Priority: P2)

**Goal**: Cancel with spot release - frees up spots and maintains accurate attendance counts

**Independent Test**: Can be tested by canceling an RSVP and verifying the spot is freed

### Backend Implementation

- [ ] T035 [US2] Add cancelRSVP method to RSVP service at `api/src/modules/rsvp/rsvp.service.ts` with status update to canceled, timestamp, and optional reason
- [ ] T036 [US2] Add attendee count decrement logic on cancellation (FR-002)
- [ ] T037 [US2] Add late cancellation detection (within 24 hours of event) for reliability tracking
- [ ] T038 [US2] Add DELETE /events/:eventId/rsvp endpoint to RSVP controller with optional reason in request body
- [ ] T039 [US2] Implement waitlist promotion trigger on cancellation - emit event to waitlist service (FR-003)

### Mobile Implementation

- [ ] T040 [P] [US2] Create CancelRSVPModal component at `mobile/src/features/rsvp/components/CancelRSVPModal.tsx` with confirmation prompt explaining spot will open for others
- [ ] T041 [US2] Add late cancellation warning message to CancelRSVPModal (shown when within 24 hours of event)
- [ ] T042 [US2] Add cancel functionality to useRSVP hook with cancelRSVP mutation
- [ ] T043 [US2] Update RSVPButton to show cancel option when user is RSVPed (long-press or secondary button)

**Checkpoint**: User Story 2 complete - users can cancel RSVPs and spots are released

---

## Phase 5: User Story 3 - Join Waitlist (Priority: P3)

**Goal**: Waitlist management with claim windows - captures demand and fills canceled spots quickly

**Independent Test**: Can be tested by joining a waitlist and being promoted when a spot opens

### Backend Implementation

- [ ] T044 [US3] Implement Waitlist service at `api/src/modules/waitlist/waitlist.service.ts` with joinWaitlist method that assigns position (FIFO order per FR-004)
- [ ] T045 [US3] Add getWaitlistPosition method to Waitlist service for retrieving user's position
- [ ] T046 [US3] Implement waitlist promotion worker at `api/src/modules/waitlist/waitlist-promotion.worker.ts` using SQS consumer pattern
- [ ] T047 [US3] Add promoteNextInLine method to Waitlist service - updates status to notified, sets 4-hour claim window (FR-005)
- [ ] T048 [US3] Add claimSpot method to Waitlist service - creates RSVP and removes from waitlist
- [ ] T049 [US3] Add declineSpot method to Waitlist service - triggers next promotion
- [ ] T050 [US3] Implement claim window expiration handler - auto-expire and promote next (FR-006)
- [ ] T051 [US3] Add GET /events/:eventId/waitlist/position endpoint for position checking
- [ ] T052 [US3] Add POST /waitlist/:positionId/claim endpoint for claiming opened spot
- [ ] T053 [US3] Add POST /waitlist/:positionId/decline endpoint for declining opened spot
- [ ] T054 [US3] Configure SNS/SQS queue for waitlist promotion events at `api/src/modules/waitlist/waitlist.queue.ts`

### Mobile Implementation

- [ ] T055 [P] [US3] Create WaitlistBadge component at `mobile/src/features/rsvp/components/WaitlistBadge.tsx` showing position (e.g., "3rd on waitlist")
- [ ] T056 [US3] Update RSVPButton to show "Join Waitlist" when event is at capacity
- [ ] T057 [US3] Add useWaitlist hook at `mobile/src/features/rsvp/hooks/useWaitlist.ts` with join, claim, decline mutations
- [ ] T058 [US3] Create WaitlistClaimModal component for time-limited claim window with countdown timer
- [ ] T059 [US3] Handle push notification for waitlist promotion - deep link to claim modal

**Checkpoint**: User Story 3 complete - users can join waitlists and claim spots when available

---

## Phase 6: User Story 4 - View My Events (Priority: P4)

**Goal**: Personal event calendar - central hub for upcoming and past meetups

**Independent Test**: Can be tested by viewing My Events and confirming all RSVPed events appear

### Backend Implementation

- [ ] T060 [US4] Implement MyEvents service at `api/src/modules/my-events/my-events.service.ts` with getUpcomingEvents method (sorted by date, FR-007)
- [ ] T061 [US4] Add getPastEvents method to MyEvents service for event history
- [ ] T062 [US4] Add event status enrichment (today badge, canceled status) to MyEvents service
- [ ] T063 [US4] Implement MyEvents controller at `api/src/modules/my-events/my-events.controller.ts` with GET /me/events endpoint returning { upcoming: [...], past: [...] }

### Mobile Implementation

- [ ] T064 [P] [US4] Create MyEventsScreen at `mobile/src/features/my-events/screens/MyEventsScreen.tsx` with tab navigation for upcoming/past
- [ ] T065 [P] [US4] Create UpcomingEventsTab at `mobile/src/features/my-events/screens/UpcomingEventsTab.tsx` with sorted event list
- [ ] T066 [P] [US4] Create PastEventsTab at `mobile/src/features/my-events/screens/PastEventsTab.tsx` for event history
- [ ] T067 [US4] Create TodayBadge component at `mobile/src/features/my-events/components/TodayBadge.tsx` for highlighting today's events
- [ ] T068 [US4] Add canceled event status display with visual indicator
- [ ] T069 [US4] Add navigation from My Events to event detail view
- [ ] T070 [US4] Add pull-to-refresh and loading states to My Events screens

**Checkpoint**: User Story 4 complete - users can view all their RSVPed events in one place

---

## Phase 7: User Story 5 - Confirm Attendance (Priority: P5)

**Goal**: Day-of confirmation flow - reduces uncertainty for hosts and increases show rates

**Independent Test**: Can be tested by confirming attendance on event day

### Backend Implementation

- [ ] T071 [US5] Implement Confirmation service at `api/src/modules/attendance/confirmation.service.ts` with confirmAttendance method updating RSVP status to confirmed (FR-009)
- [ ] T072 [US5] Add getConfirmationStatus method to Confirmation service for checking if user has confirmed
- [ ] T073 [US5] Add getPendingConfirmations method to get events requiring confirmation today (FR-008)
- [ ] T074 [US5] Implement Confirmation controller at `api/src/modules/attendance/confirmation.controller.ts` with POST /events/:eventId/rsvp/confirm endpoint
- [ ] T075 [US5] Add host notification trigger when attendee confirms attendance
- [ ] T076 [US5] Add getAttendeeConfirmationStatuses method for host to view confirmation status of all attendees

### Mobile Implementation

- [ ] T077 [P] [US5] Create ConfirmationPrompt component at `mobile/src/features/confirmation/components/ConfirmationPrompt.tsx` with "Still Going" and "Can't Make It" options
- [ ] T078 [P] [US5] Create ConfirmationBanner component at `mobile/src/features/confirmation/components/ConfirmationBanner.tsx` for app-level reminder
- [ ] T079 [US5] Add useConfirmation hook at `mobile/src/features/confirmation/hooks/useConfirmation.ts` with confirm and cancel mutations
- [ ] T080 [US5] Integrate ConfirmationBanner into main app view for day-of events
- [ ] T081 [US5] Add confirmation status indicator to attendee list (visible to host)
- [ ] T082 [US5] Connect "Can't Make It" flow to cancel RSVP logic from US2

**Checkpoint**: User Story 5 complete - users can confirm day-of attendance

---

## Phase 8: User Story 6 - Post-Event Trust Scoring (Priority: P6)

**Goal**: Private trust signal collection - "Would you meet again?" feedback for matching

**Independent Test**: Can be tested by completing post-event survey

### Backend Implementation

- [ ] T083 [US6] Implement TrustSignals service at `api/src/modules/trust-signals/trust-signals.service.ts` with submitFeedback method storing thumbs up/down per attendee (FR-010)
- [ ] T084 [US6] Add getPendingFeedback method to TrustSignals service for events requiring feedback (within 24 hours of completion)
- [ ] T085 [US6] Add getEventAttendeesForFeedback method to get list of attendees user interacted with
- [ ] T086 [US6] Implement UserReliability service at `api/src/modules/trust-signals/user-reliability.service.ts` with updateReliabilityScore method (FR-011)
- [ ] T087 [US6] Add no-show detection logic - mark RSVP as no_show if not confirmed and event passed
- [ ] T088 [US6] Add reliability score calculation: (confirmed_attendances - no_shows - late_cancellations) / total_rsvps
- [ ] T089 [US6] Implement TrustSignals controller at `api/src/modules/trust-signals/trust-signals.controller.ts` with POST /events/:eventId/feedback endpoint
- [ ] T090 [US6] Add GET /events/:eventId/feedback-prompt endpoint for checking if feedback is pending

### Mobile Implementation

- [ ] T091 [P] [US6] Create PostEventFeedbackScreen at `mobile/src/features/trust-signals/screens/PostEventFeedbackScreen.tsx` with attendee list for rating
- [ ] T092 [P] [US6] Create AttendeeRatingCard component at `mobile/src/features/trust-signals/components/AttendeeRatingCard.tsx` with thumbs up/down buttons
- [ ] T093 [US6] Add useTrustSignals hook at `mobile/src/features/trust-signals/hooks/useTrustSignals.ts` with submitFeedback mutation
- [ ] T094 [US6] Add post-event feedback prompt trigger when opening app within 24 hours of event completion
- [ ] T095 [US6] Add skip/dismiss functionality with option to access later from event history
- [ ] T096 [US6] Add feedback completion confirmation with thank you message

**Checkpoint**: User Story 6 complete - private trust signals collected for matching

---

## Phase 9: Integration & Cross-Cutting Concerns

**Purpose**: Connect user stories and handle edge cases

### Notifications Integration

- [ ] T097 Configure notification worker for waitlist promotions at `api/src/modules/notifications/notification.worker.ts`
- [ ] T098 [P] Add push notification for day-of confirmation reminder (FR-008)
- [ ] T099 [P] Add push notification for event cancellation to RSVPed users
- [ ] T100 [P] Add push notification for host when minimum capacity reached (FR-012)

### Edge Cases & Error Handling

- [ ] T101 Add overlapping event warning when RSVPing to events at same time
- [ ] T102 [P] Add event canceled handling - move to canceled status in My Events
- [ ] T103 [P] Add suspended user handling - hide RSVPs from hosts, block new RSVPs
- [ ] T104 Add waitlist fallback when all users decline - reopen spots for new RSVPs

### Reliability Integration

- [ ] T105 Connect reliability score updates to RSVP lifecycle events (create, confirm, cancel, no-show)
- [ ] T106 Export reliability score for matching service integration (provides to 004-vibe-matching)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phases 3-8)**: All depend on Foundational phase completion
  - US1 (P1): Can start after Foundational
  - US2 (P2): Depends on US1 (uses RSVP service)
  - US3 (P3): Depends on US1 (waitlist triggers on RSVP)
  - US4 (P4): Depends on US1 (shows RSVPed events)
  - US5 (P5): Depends on US1 and US2 (confirms or cancels RSVP)
  - US6 (P6): Depends on US1 (tracks RSVPed events for feedback)
- **Integration (Phase 9)**: Depends on all user stories being complete

### Parallel Opportunities

- All Setup tasks (T001-T009) marked [P] can run in parallel
- All database migrations (T010-T013) marked [P] can run in parallel
- All entity creation tasks (T014-T018) marked [P] can run in parallel
- All module registration tasks (T019-T023) marked [P] can run in parallel
- Mobile components within each user story marked [P] can run in parallel
- Notification tasks (T097-T100) marked [P] can run in parallel
- Edge case tasks (T101-T104) marked [P] can run in parallel

### External Dependencies

**Requires**:
- 001-user-onboarding: User entity for user_id foreign keys
- 002-event-creation: Event entity with capacity for capacity checks
- 003-event-discovery: Event detail view for RSVPButton integration

**Provides to**:
- 006-group-chat: Chat access granted on RSVP
- 004-vibe-matching: Trust signals and reliability scores feed into matching

---

## Notes

- [P] tasks = different files, no dependencies within phase
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- RSVP action must complete in < 2 seconds (SC-001)
- Waitlist claim window is 4 hours (configurable)
- Post-event feedback window is 24 hours
- Trust signals are NEVER displayed publicly (privacy requirement)
