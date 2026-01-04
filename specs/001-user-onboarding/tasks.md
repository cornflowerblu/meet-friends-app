# Tasks: User Onboarding

**Input**: Design documents from `/specs/001-user-onboarding/`
**Prerequisites**: plan.md (required), spec.md (required for user stories)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **API**: `api/src/modules/...`
- **Mobile**: `mobile/src/...`, `mobile/app/...`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure for both API and mobile

- [ ] T001 [P] Initialize NestJS project with TypeScript strict mode in `api/`
- [ ] T002 [P] Initialize Expo project with TypeScript and Router in `mobile/`
- [ ] T003 [P] Configure ESLint and Prettier for API in `api/.eslintrc.js`
- [ ] T004 [P] Configure ESLint and Prettier for mobile in `mobile/.eslintrc.js`
- [ ] T005 [P] Create environment configuration for API in `api/src/config/env.config.ts`
- [ ] T006 [P] Create environment configuration for mobile in `mobile/src/config/env.ts`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**CRITICAL**: No user story work can begin until this phase is complete

### Database & ORM

- [ ] T007 Configure PostgreSQL connection and TypeORM in `api/src/config/database.config.ts`
- [ ] T008 Create base User entity in `api/src/modules/users/entities/user.entity.ts`
- [ ] T009 Create User migration with onboarding_step enum in `api/src/migrations/`

### Authentication Framework

- [ ] T010 [P] Implement JWT authentication module in `api/src/modules/auth/auth.module.ts`
- [ ] T011 [P] Create JWT strategy and guards in `api/src/common/guards/jwt-auth.guard.ts`
- [ ] T012 [P] Implement token service in `api/src/modules/auth/token.service.ts`

### API Infrastructure

- [ ] T013 [P] Setup global exception filter in `api/src/common/filters/http-exception.filter.ts`
- [ ] T014 [P] Configure request validation pipe in `api/src/common/pipes/validation.pipe.ts`
- [ ] T015 [P] Setup logging interceptor in `api/src/common/interceptors/logging.interceptor.ts`

### Mobile Infrastructure

- [ ] T016 [P] Create API client service in `mobile/src/services/api.ts`
- [ ] T017 [P] Implement secure storage service in `mobile/src/services/storage.ts`
- [ ] T018 [P] Create auth state management in `mobile/src/services/auth.ts`
- [ ] T019 [P] Setup app layout with navigation in `mobile/app/_layout.tsx`

### External Services

- [ ] T020 [P] Configure Twilio Verify client in `api/src/modules/auth/twilio.config.ts`
- [ ] T021 [P] Configure AWS S3 client in `api/src/config/s3.config.ts`
- [ ] T022 [P] Configure CloudFront distribution settings in `api/src/config/cloudfront.config.ts`

**Checkpoint**: Foundation ready - user story implementation can now begin

---

## Phase 3: User Story 1 - Phone Verification & Account Creation (Priority: P1)

**Goal**: Enable users to create verified accounts via SMS phone verification

**Independent Test**: Complete signup flow and receive SMS verification code; verify account is created

### Backend Implementation (US1)

- [ ] T023 [P] [US1] Create SendCodeDto in `api/src/modules/auth/dto/send-code.dto.ts`
- [ ] T024 [P] [US1] Create VerifyCodeDto in `api/src/modules/auth/dto/verify-code.dto.ts`
- [ ] T025 [P] [US1] Create AuthResponseDto in `api/src/modules/auth/dto/auth-response.dto.ts`
- [ ] T026 [US1] Implement SMS service with Twilio Verify in `api/src/modules/auth/sms.service.ts`
- [ ] T027 [US1] Implement rate limiting decorator in `api/src/common/decorators/rate-limit.decorator.ts`
- [ ] T028 [US1] Implement auth service with send/verify code logic in `api/src/modules/auth/auth.service.ts`
- [ ] T029 [US1] Create auth controller with endpoints in `api/src/modules/auth/auth.controller.ts`:
  - POST /api/v1/auth/send-code
  - POST /api/v1/auth/verify-code
  - POST /api/v1/auth/request-voice-call
- [ ] T030 [US1] Implement attempt tracking for failed verifications in `api/src/modules/auth/auth.service.ts`
- [ ] T031 [US1] Add voice call fallback after 3 SMS failures in `api/src/modules/auth/sms.service.ts`
- [ ] T032 [US1] Create users service with basic CRUD in `api/src/modules/users/users.service.ts`
- [ ] T033 [US1] Create users controller in `api/src/modules/users/users.controller.ts`

### Mobile Implementation (US1)

- [ ] T034 [P] [US1] Create phone input component in `mobile/src/components/forms/PhoneInput.tsx`
- [ ] T035 [P] [US1] Create verification code input component in `mobile/src/components/forms/CodeInput.tsx`
- [ ] T036 [US1] Implement useAuth hook in `mobile/src/hooks/useAuth.ts`
- [ ] T037 [US1] Create phone number entry screen in `mobile/app/(auth)/phone.tsx`
- [ ] T038 [US1] Create code verification screen in `mobile/app/(auth)/verify.tsx`
- [ ] T039 [US1] Add countdown timer and resend logic to verify screen
- [ ] T040 [US1] Implement voice call request option after 3 SMS failures
- [ ] T041 [US1] Handle existing phone number detection with account recovery prompt

**Checkpoint**: User Story 1 complete - users can verify phone and create accounts

---

## Phase 4: User Story 2 - Profile Photo Upload (Priority: P2)

**Goal**: Enable users to upload profile photos with quality validation

**Independent Test**: Upload various image types and verify acceptance/rejection criteria

### Backend Implementation (US2)

- [ ] T042 [P] [US2] Create ProfilePhoto entity in `api/src/modules/photos/entities/profile-photo.entity.ts`
- [ ] T043 [P] [US2] Create ProfilePhoto migration in `api/src/migrations/`
- [ ] T044 [P] [US2] Create UploadUrlResponseDto in `api/src/modules/photos/dto/upload-url-response.dto.ts`
- [ ] T045 [P] [US2] Create ConfirmPhotoDto in `api/src/modules/photos/dto/confirm-photo.dto.ts`
- [ ] T046 [US2] Implement S3 upload service with presigned URLs in `api/src/modules/photos/upload.service.ts`
- [ ] T047 [US2] Implement photo validation service (dimensions, file type) in `api/src/modules/photos/validation.service.ts`
- [ ] T048 [US2] Implement photos service in `api/src/modules/photos/photos.service.ts`
- [ ] T049 [US2] Create photos controller with endpoints in `api/src/modules/photos/photos.controller.ts`:
  - POST /api/v1/photos/upload-url
  - POST /api/v1/photos/{id}/confirm
  - DELETE /api/v1/photos/{id}
  - PATCH /api/v1/photos/{id}/set-primary
- [ ] T050 [US2] Implement moderation queue for flagged photos in `api/src/modules/photos/photos.service.ts`

### Mobile Implementation (US2)

- [ ] T051 [P] [US2] Create photo preview component in `mobile/src/components/onboarding/PhotoPreview.tsx`
- [ ] T052 [P] [US2] Create photo upload button component in `mobile/src/components/onboarding/PhotoUploadButton.tsx`
- [ ] T053 [US2] Implement photo upload hook in `mobile/src/hooks/usePhotoUpload.ts`
- [ ] T054 [US2] Create photo upload screen with camera/gallery in `mobile/app/(onboarding)/photos.tsx`
- [ ] T055 [US2] Implement client-side image validation (dimensions, size)
- [ ] T056 [US2] Add upload progress indicator and error handling
- [ ] T057 [US2] Handle photo quality rejection with user guidance

**Checkpoint**: User Story 2 complete - users can upload validated profile photos

---

## Phase 5: User Story 3 - Vibe Profile Collection (Priority: P3)

**Goal**: Collect behavioral preferences across 6 vibe dimensions via conversational prompts

**Independent Test**: Complete vibe questionnaire and verify trait scores are recorded (not visible to user)

### Backend Implementation (US3)

- [ ] T058 [P] [US3] Create VibeProfile entity in `api/src/modules/vibe-profiles/entities/vibe-profile.entity.ts`
- [ ] T059 [P] [US3] Create VibeQuestion entity in `api/src/modules/vibe-profiles/entities/vibe-question.entity.ts`
- [ ] T060 [P] [US3] Create VibeProfile migration in `api/src/migrations/`
- [ ] T061 [P] [US3] Create QuestionResponseDto in `api/src/modules/vibe-profiles/dto/question-response.dto.ts`
- [ ] T062 [P] [US3] Create SubmitResponsesDto in `api/src/modules/vibe-profiles/dto/submit-responses.dto.ts`
- [ ] T063 [US3] Seed 8-12 scenario-based questions in `api/src/modules/vibe-profiles/data/questions.seed.ts`
- [ ] T064 [US3] Implement questions service in `api/src/modules/vibe-profiles/questions.service.ts`
- [ ] T065 [US3] Implement vibe scoring logic (maps responses to 6 dimensions) in `api/src/modules/vibe-profiles/vibe.service.ts`
- [ ] T066 [US3] Create vibe controller with endpoints in `api/src/modules/vibe-profiles/vibe.controller.ts`:
  - GET /api/v1/vibe/questions
  - POST /api/v1/vibe/responses
  - POST /api/v1/vibe/skip
- [ ] T067 [US3] Handle skip flow with reduced match quality flag

### Mobile Implementation (US3)

- [ ] T068 [P] [US3] Create vibe question card component in `mobile/src/components/onboarding/VibeQuestionCard.tsx`
- [ ] T069 [P] [US3] Create vibe option button component in `mobile/src/components/onboarding/VibeOptionButton.tsx`
- [ ] T070 [US3] Implement useVibeProfile hook in `mobile/src/hooks/useVibeProfile.ts`
- [ ] T071 [US3] Create vibe questions carousel screen in `mobile/app/(onboarding)/vibes.tsx`
- [ ] T072 [US3] Implement progress indicator for questions
- [ ] T073 [US3] Add skip option with reduced quality warning
- [ ] T074 [US3] Ensure no vibe scores or labels are displayed to user (FR-007)

**Checkpoint**: User Story 3 complete - users can complete vibe profile via conversational questions

---

## Phase 6: User Story 4 - Activity Selection (Priority: P4)

**Goal**: Enable users to select activities with optional skill levels

**Independent Test**: Select activities and verify they appear in user preferences

### Backend Implementation (US4)

- [ ] T075 [P] [US4] Create Activity entity in `api/src/modules/activities/entities/activity.entity.ts`
- [ ] T076 [P] [US4] Create ActivityInterest entity in `api/src/modules/activities/entities/activity-interest.entity.ts`
- [ ] T077 [P] [US4] Create Activity migration in `api/src/migrations/`
- [ ] T078 [P] [US4] Create ActivityInterestDto in `api/src/modules/activities/dto/activity-interest.dto.ts`
- [ ] T079 [P] [US4] Create SaveActivitiesDto in `api/src/modules/activities/dto/save-activities.dto.ts`
- [ ] T080 [US4] Seed 10-15 activities with icons in `api/src/modules/activities/data/activities.seed.ts`
- [ ] T081 [US4] Implement activities service in `api/src/modules/activities/activities.service.ts`
- [ ] T082 [US4] Create activities controller with endpoints in `api/src/modules/activities/activities.controller.ts`:
  - GET /api/v1/activities
  - POST /api/v1/users/me/activities
- [ ] T083 [US4] Update user onboarding_step to COMPLETE after activity selection

### Mobile Implementation (US4)

- [ ] T084 [P] [US4] Create activity card component in `mobile/src/components/onboarding/ActivityCard.tsx`
- [ ] T085 [P] [US4] Create skill level selector component in `mobile/src/components/onboarding/SkillLevelSelector.tsx`
- [ ] T086 [US4] Implement useActivities hook in `mobile/src/hooks/useActivities.ts`
- [ ] T087 [US4] Create activity selection grid screen in `mobile/app/(onboarding)/activities.tsx`
- [ ] T088 [US4] Implement skill level modal for activities that support it
- [ ] T089 [US4] Add minimum selection validation (at least 1 activity)
- [ ] T090 [US4] Navigate to main app after completing onboarding

**Checkpoint**: User Story 4 complete - users can select activities with skill levels

---

## Phase 7: Cross-Cutting & Polish

**Purpose**: Onboarding flow integration and improvements that affect multiple user stories

### Onboarding State Management

- [ ] T091 [P] Implement onboarding progress hook in `mobile/src/hooks/useOnboarding.ts`
- [ ] T092 [P] Create onboarding progress persistence in `mobile/src/services/onboardingState.ts`
- [ ] T093 Implement onboarding state machine (resume from last step) in API
- [ ] T094 Add onboarding progress API endpoint GET /api/v1/users/me/onboarding-status

### Location Collection (FR-009)

- [ ] T095 [P] Add general_location field handling in user service
- [ ] T096 [P] Create location input component in `mobile/src/components/forms/LocationInput.tsx`
- [ ] T097 Integrate location collection into onboarding flow

### Notification Consent (FR-010)

- [ ] T098 [P] Create notification consent screen in `mobile/src/components/onboarding/NotificationConsent.tsx`
- [ ] T099 [P] Add notification_consent field handling in user service
- [ ] T100 Request system notification permission after user consent

### UX Polish

- [ ] T101 [P] Add "Not a dating app" framing copy throughout onboarding (FR-008)
- [ ] T102 [P] Implement loading states and skeleton screens
- [ ] T103 [P] Add haptic feedback for key interactions
- [ ] T104 Implement smooth transitions between onboarding screens

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phases 3-6)**: All depend on Foundational phase completion
  - User stories can proceed in priority order (P1 -> P2 -> P3 -> P4)
  - Or in parallel if team capacity allows
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational - Independent of US1
- **User Story 3 (P3)**: Can start after Foundational - Independent of US1/US2
- **User Story 4 (P4)**: Can start after Foundational - Independent of US1/US2/US3

### Within Each User Story

- DTOs and entities (marked [P]) can be created in parallel
- Services depend on entities being complete
- Controllers depend on services being complete
- Mobile screens depend on corresponding API endpoints

### Parallel Opportunities

Within Setup (Phase 1):
- T001-T006 can all run in parallel

Within Foundational (Phase 2):
- T010-T015 (Auth & API infrastructure) can run in parallel
- T016-T019 (Mobile infrastructure) can run in parallel
- T020-T022 (External services) can run in parallel

Within each User Story:
- All tasks marked [P] can run in parallel
- Backend and mobile work can proceed in parallel once APIs are defined

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (Phone Verification)
4. **STOP and VALIDATE**: Test phone verification flow end-to-end
5. Deploy/demo if ready - users can create verified accounts

### Incremental Delivery

1. Complete Setup + Foundational -> Foundation ready
2. Add User Story 1 -> Test independently -> Deploy (Verified accounts!)
3. Add User Story 2 -> Test independently -> Deploy (Photos!)
4. Add User Story 3 -> Test independently -> Deploy (Vibe profiles!)
5. Add User Story 4 -> Test independently -> Deploy (Full onboarding!)
6. Add Phase 7 Polish -> Final release

---

## Notes

- [P] tasks = different files, no dependencies within that group
- [US#] label maps task to specific user story for traceability
- Each user story is independently completable and testable
- Commit after each task or logical group
- FR-007: Never expose vibe scores to users
- FR-008: Maintain "not a dating app" framing in all copy
