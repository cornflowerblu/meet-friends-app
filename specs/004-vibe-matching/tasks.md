# Tasks: Vibe Matching

**Input**: Design documents from `/specs/004-vibe-matching/`
**Prerequisites**: plan.md (required), spec.md (required for user stories)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **Backend**: `api/src/modules/matching/`
- **Mobile**: `mobile/src/components/`, `mobile/src/screens/`, `mobile/src/hooks/`, `mobile/src/services/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and module structure

- [ ] T001 Create matching module directory structure at `api/src/modules/matching/`
- [ ] T002 [P] Create matching module file at `api/src/modules/matching/matching.module.ts`
- [ ] T003 [P] Create mobile services directory entry at `mobile/src/services/matching.service.ts`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**CRITICAL**: No user story work can begin until this phase is complete

### Database & Entities

- [ ] T004 Create MatchingPreference entity at `api/src/modules/matching/entities/matching-preference.entity.ts` with userId, vibeWeight, distanceWeight, activityWeight fields
- [ ] T005 Create database migration for matching_preferences table with default weights (0.33, 0.33, 0.34)

### DTOs

- [ ] T006 [P] Create CompatibilityResponseDTO at `api/src/modules/matching/dto/compatibility-response.dto.ts` with label, explanationPoints, isProfileComplete
- [ ] T007 [P] Create MatchExplanationDTO at `api/src/modules/matching/dto/match-explanation.dto.ts` with eventId, label, primaryReasons, vibeFactors
- [ ] T008 [P] Create MatchingPreferencesDTO at `api/src/modules/matching/dto/matching-preferences.dto.ts` with vibeWeight, distanceWeight, activityWeight

### Core Services

- [ ] T009 Implement compatibility-calculator.service.ts at `api/src/modules/matching/compatibility-calculator.service.ts` with calculateVibeScore using weighted Euclidean distance
- [ ] T010 Implement score-to-label mapping in compatibility-calculator.service.ts (80-100=great_match, 60-79=good_match, 40-59=fair_match, 0-39=different_vibe)
- [ ] T011 Implement explanation-generator.service.ts at `api/src/modules/matching/explanation-generator.service.ts` with template-based explanation generation
- [ ] T012 Add explanation templates for high_alignment, low_alignment, and dimension_friendly_names in explanation-generator.service.ts

### Module Wiring

- [ ] T013 Create matching.service.ts at `api/src/modules/matching/matching.service.ts` as facade for calculator and explanation services
- [ ] T014 Create matching.controller.ts at `api/src/modules/matching/matching.controller.ts` with route stubs
- [ ] T015 Register all services in matching.module.ts and add to app module

### Redis Caching

- [ ] T016 Configure Redis caching for compatibility scores in matching.service.ts with TTL strategy

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - See Vibe Compatibility on Events (Priority: P1)

**Goal**: Display vibe compatibility indicators on event cards showing how well user preferences align with event/attendees

**Independent Test**: View events and confirm compatibility indicators appear with correct labels

### Backend Implementation for US1

- [ ] T017 [US1] Implement GET /events/{eventId}/compatibility endpoint in matching.controller.ts returning eventId, compatibility object, attendeeCompatibilities, groupSummary
- [ ] T018 [US1] Add calculateEventCompatibility method in compatibility-calculator.service.ts factoring event vibe, host, and attendee vibes
- [ ] T019 [US1] Implement group compatibility averaging logic in compatibility-calculator.service.ts for attendeeCompatibilities
- [ ] T020 [US1] Add isProfileComplete check in matching.service.ts to flag incomplete vibe profiles

### Mobile Implementation for US1

- [ ] T021 [P] [US1] Create VibeCompatibilityBadge component at `mobile/src/components/VibeCompatibilityBadge.tsx` with label prop and color/icon config
- [ ] T022 [P] [US1] Create useEventCompatibility hook at `mobile/src/hooks/useEventCompatibility.ts` to fetch compatibility data
- [ ] T023 [US1] Integrate VibeCompatibilityBadge into event card component with onWhyThisMatch callback prop
- [ ] T024 [US1] Add incomplete profile prompt UI in VibeCompatibilityBadge when isProfileComplete is false
- [ ] T025 [US1] Add matching API client methods in `mobile/src/services/matching.service.ts` for getEventCompatibility

**Checkpoint**: User Story 1 complete - compatibility indicators visible on event cards

---

## Phase 4: User Story 2 - See Why We Matched Explanations (Priority: P2)

**Goal**: Provide human-readable explanations of compatibility when user taps "Why this match?"

**Independent Test**: Open match explanation and verify clear, understandable reasoning appears with 2-3 alignment points

### Backend Implementation for US2

- [ ] T026 [US2] Implement GET /events/{eventId}/match-explanation endpoint in matching.controller.ts
- [ ] T027 [US2] Add generateExplanation method in explanation-generator.service.ts returning primaryReasons and vibeFactors
- [ ] T028 [US2] Implement dimension comparison logic to identify top 2-3 alignment/difference points
- [ ] T029 [US2] Add honest difference summaries for low compatibility without negative framing per FR-008

### Mobile Implementation for US2

- [ ] T030 [P] [US2] Create MatchExplanationModal component at `mobile/src/components/MatchExplanationModal.tsx` displaying primaryReasons and vibeFactors
- [ ] T031 [US2] Wire "Why this match?" button in VibeCompatibilityBadge to open MatchExplanationModal
- [ ] T032 [US2] Add getMatchExplanation method in `mobile/src/services/matching.service.ts`
- [ ] T033 [US2] Style explanation modal with friendly behavioral descriptions (no scores or labels shown per FR-004)

**Checkpoint**: User Story 2 complete - match explanations accessible and understandable

---

## Phase 5: User Story 3 - Adjust Matching Preferences (Priority: P3)

**Goal**: Allow users to adjust vibe/distance/activity weight sliders to customize discovery experience

**Independent Test**: Adjust sliders and verify event rankings change accordingly

### Backend Implementation for US3

- [ ] T034 [US3] Implement GET /users/me/matching-preferences endpoint in matching.controller.ts
- [ ] T035 [US3] Implement PUT /users/me/matching-preferences endpoint with validation ensuring weights sum to 1.0
- [ ] T036 [US3] Add getPreferences and updatePreferences methods in matching.service.ts with default weight initialization
- [ ] T037 [US3] Implement reset to defaults functionality returning vibeWeight=0.33, distanceWeight=0.33, activityWeight=0.34
- [ ] T038 [US3] Invalidate cached compatibility scores when preferences are updated

### Mobile Implementation for US3

- [ ] T039 [P] [US3] Create MatchingPreferencesSlider component at `mobile/src/components/MatchingPreferencesSlider.tsx` with three sliders
- [ ] T040 [P] [US3] Create useMatchingPreferences hook at `mobile/src/hooks/useMatchingPreferences.ts` for preference state management
- [ ] T041 [US3] Create MatchingPreferencesScreen at `mobile/src/screens/MatchingPreferencesScreen.tsx` with sliders and reset button
- [ ] T042 [US3] Add weight normalization logic in MatchingPreferencesSlider ensuring weights always sum to 1.0
- [ ] T043 [US3] Add getMatchingPreferences and updateMatchingPreferences methods in `mobile/src/services/matching.service.ts`
- [ ] T044 [US3] Add navigation to MatchingPreferencesScreen from user settings

**Checkpoint**: User Story 3 complete - users can customize matching weights

---

## Phase 6: User Story 4 - View Attendee Vibe Compatibility (Priority: P4)

**Goal**: Show vibe compatibility with individual attendees and group summary before joining event

**Independent Test**: View attendee list with compatibility indicators and group summary

### Backend Implementation for US4

- [ ] T045 [US4] Add attendeeCompatibilities array to GET /events/{eventId}/compatibility response with per-attendee compatibility
- [ ] T046 [US4] Implement calculateAttendeeCompatibility method in compatibility-calculator.service.ts for user-to-user vibe scoring
- [ ] T047 [US4] Add groupSummary generation logic ("mostly compatible", "mixed group", etc.) based on attendee average
- [ ] T048 [US4] Ensure attendee compatibility shows shared activities and general alignment, not raw scores per acceptance criteria

### Mobile Implementation for US4

- [ ] T049 [P] [US4] Create AttendeeCompatibilityList component at `mobile/src/components/AttendeeCompatibilityList.tsx` displaying per-attendee compatibility badges
- [ ] T050 [US4] Add group compatibility summary header to AttendeeCompatibilityList showing aggregate compatibility
- [ ] T051 [US4] Integrate AttendeeCompatibilityList into event detail screen attendee section
- [ ] T052 [US4] Add attendee tap handler to show profile with shared activities and vibe alignment (not scores)
- [ ] T053 [US4] Handle low compatibility display with neutral framing ("different preferences" not "bad person" per acceptance criteria)

**Checkpoint**: User Story 4 complete - group and attendee compatibility visible before RSVP

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T054 [P] Add error handling for missing vibe profiles across all compatibility endpoints
- [ ] T055 [P] Add loading states to all mobile components (VibeCompatibilityBadge, MatchExplanationModal, etc.)
- [ ] T056 Implement compatibility recalculation trigger when user updates their vibe profile per FR-010
- [ ] T057 [P] Add empty state handling for events with no attendees (base on host vibe only)
- [ ] T058 Performance optimization: ensure compatibility calculation < 50ms per event per plan.md goals

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-6)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 -> P2 -> P3 -> P4)
- **Polish (Phase 7)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Uses VibeCompatibilityBadge from US1 but independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Independent of other stories
- **User Story 4 (P4)**: Can start after Foundational (Phase 2) - Uses compatibility infrastructure, independently testable

### Within Each User Story

- Backend before mobile (API must exist for mobile to consume)
- Core logic before endpoints
- Endpoints before mobile integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational DTO tasks (T006, T007, T008) can run in parallel
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- Within each user story, tasks marked [P] can run in parallel
- Mobile component creation tasks marked [P] within a story can run in parallel

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test compatibility badges on event cards
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational -> Foundation ready
2. Add User Story 1 -> Test independently -> Deploy/Demo (MVP!)
3. Add User Story 2 -> Test independently -> Deploy/Demo
4. Add User Story 3 -> Test independently -> Deploy/Demo
5. Add User Story 4 -> Test independently -> Deploy/Demo
6. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (backend + mobile)
   - Developer B: User Story 2 (can start backend while A finishes US1 mobile)
   - Developer C: User Story 3 (fully independent)
   - Developer D: User Story 4 (can share some US1 components)
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
- Score-to-label mapping: 80-100=great_match, 60-79=good_match, 40-59=fair_match, 0-39=different_vibe
- Explanations use templates, NOT per-request LLM calls per constitution
