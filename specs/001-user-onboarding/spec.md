# Feature Specification: User Onboarding

**Feature Branch**: `001-user-onboarding`
**Created**: 2026-01-04
**Status**: Draft
**Input**: User Onboarding: Account creation, phone verification, photo upload, and vibe profile collection for the friend-matching app

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Phone Verification & Account Creation (Priority: P1)

A new user downloads the app and creates their account using phone verification. This is the first gate that establishes identity friction and filters out bad actors before they can interact with others.

**Why this priority**: Without verified accounts, the app cannot fulfill its Trust & Safety principle. Phone verification is the foundational identity layer that all other features depend on.

**Independent Test**: Can be fully tested by completing signup flow and receiving SMS verification code. Delivers a verified user account ready for profile completion.

**Acceptance Scenarios**:

1. **Given** a new user opens the app for the first time, **When** they enter a valid phone number, **Then** they receive an SMS verification code within 60 seconds
2. **Given** a user has received a verification code, **When** they enter the correct code, **Then** their phone number is verified and they proceed to profile creation
3. **Given** a user enters an incorrect code 3 times, **When** they attempt a 4th entry, **Then** they must wait 5 minutes before requesting a new code
4. **Given** a phone number is already registered, **When** a new user attempts to register with it, **Then** they are informed the number is in use and offered account recovery options

---

### User Story 2 - Profile Photo Upload (Priority: P2)

A verified user uploads their profile photos. Real photos are required to build trust and enable other users to recognize them at meetups. The system enforces photo quality standards without being overly restrictive.

**Why this priority**: Photos are essential for the "real people, real meetups" promise. Users need to see who they're meeting, and photo requirements deter catfishing and fake accounts.

**Independent Test**: Can be tested by uploading various image types and verifying acceptance/rejection criteria. Delivers a profile with visible, appropriate photos.

**Acceptance Scenarios**:

1. **Given** a verified user is on the photo upload screen, **When** they upload a clear photo showing their face, **Then** the photo is accepted and displayed on their profile
2. **Given** a user attempts to upload a blurry or low-resolution image, **When** the system analyzes it, **Then** the user is prompted to upload a clearer photo
3. **Given** a user uploads a photo without a visible face, **When** the system analyzes it, **Then** they receive guidance to upload a photo where their face is visible
4. **Given** a user has uploaded at least 1 acceptable photo, **When** they view their profile, **Then** their primary photo is visible to other users

---

### User Story 3 - Vibe Profile Collection (Priority: P3)

A user completes their vibe profile by answering behavioral questions. The system collects preferences across the 6 vibe dimensions (casual/competitive, chatty/focused, planner/spontaneous, 1:1/group, high-energy/low-key, reliability/flexibility) through natural conversational prompts rather than clinical assessments.

**Why this priority**: Vibe matching is the app's key differentiator. Without behavioral data, matching falls back to activity-only, losing the core value proposition of "same vibe."

**Independent Test**: Can be tested by completing the vibe questionnaire and verifying trait scores are recorded. Delivers a user profile with matching-ready behavioral data.

**Acceptance Scenarios**:

1. **Given** a user has completed photo upload, **When** they start vibe collection, **Then** they see friendly, scenario-based questions (not clinical scales)
2. **Given** a user is answering vibe questions, **When** they complete all required questions, **Then** their behavioral preferences are saved without showing labels or scores
3. **Given** a user wants to skip vibe collection, **When** they attempt to skip, **Then** they are informed matching quality will be reduced but can proceed
4. **Given** a user completes vibe collection, **When** they finish onboarding, **Then** they never see internal labels like "you are 73% competitive"

---

### User Story 4 - Activity Selection (Priority: P4)

A user selects the activities they're interested in (golf, pickleball, bowling, running, etc.). This determines which events and matches they'll see.

**Why this priority**: Activities are the organizing principle for meetups. Without activity preferences, users cannot discover relevant events or be matched with compatible activity partners.

**Independent Test**: Can be tested by selecting activities and verifying they appear in user preferences. Delivers a profile with activity interests for matching and event discovery.

**Acceptance Scenarios**:

1. **Given** a user is on the activity selection screen, **When** they view available activities, **Then** they see a curated list of supported activities with visual icons
2. **Given** a user selects at least 1 activity, **When** they save their selection, **Then** their chosen activities are stored and used for matching
3. **Given** a user selects an activity, **When** they view that activity's options, **Then** they can indicate skill level (beginner/intermediate/advanced) if relevant
4. **Given** a user completes activity selection, **When** they finish onboarding, **Then** they are ready to discover events and receive match suggestions

---

### Edge Cases

- What happens when SMS delivery fails repeatedly? User is offered alternative verification (voice call) after 3 failed SMS attempts
- What happens when a user's photo is flagged as potentially inappropriate? Photo is queued for review; user can continue onboarding but photo won't be visible until approved
- What happens when a user abandons onboarding mid-flow? Progress is saved; user can resume from last completed step on next app open
- What happens when a user wants to change their phone number later? Change requires re-verification of the new number; old number is released after 30 days
- What happens if the user's device has no camera? User can upload existing photos from their device library

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST verify user identity via SMS code sent to their phone number
- **FR-002**: System MUST enforce rate limiting on verification code requests (max 5 per hour per phone number)
- **FR-003**: System MUST require at least 1 profile photo showing the user's face before completing onboarding
- **FR-004**: System MUST collect behavioral preferences across 6 vibe dimensions through conversational prompts
- **FR-005**: System MUST allow users to select 1 or more activities from the supported activity list
- **FR-006**: System MUST save onboarding progress so users can resume if interrupted
- **FR-007**: System MUST NOT display internal vibe scores, labels, or personality classifications to users
- **FR-008**: System MUST enforce "not a dating app" framing in onboarding copy and prompts
- **FR-009**: System MUST collect user's general location (city/region) for event matching
- **FR-010**: System MUST obtain user consent for notifications before requesting system permission

### Key Entities

- **User**: Represents a registered person; has phone number, verification status, profile photos, vibe preferences, activity interests, and location
- **Vibe Profile**: Stores behavioral tendencies across 6 dimensions; linked to one User; never exposed as scores/labels to users
- **Activity Interest**: Represents a user's interest in a specific activity; includes optional skill level; linked to one User and one Activity
- **Activity**: Represents a supported activity type (golf, pickleball, bowling, etc.); has name, icon, and optional skill levels

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 80% of users who start onboarding complete all steps within a single session
- **SC-002**: Users complete the entire onboarding flow in under 5 minutes
- **SC-003**: Less than 5% of verification codes fail to deliver within 60 seconds
- **SC-004**: 95% of uploaded photos pass quality checks on first attempt
- **SC-005**: Less than 10% of users skip vibe collection entirely
- **SC-006**: 90% of users select at least 2 activities during onboarding
- **SC-007**: User drop-off between onboarding steps is less than 15% at any single step

## Assumptions

- SMS delivery is reliable in target geographic regions (US initially)
- Users have smartphones with cameras capable of taking adequate photos
- The initial activity list will be curated to 10-15 activities for MVP
- Vibe questions will be limited to 8-12 questions for a quick experience
- Photo moderation will use a combination of automated checks and manual review for flagged content
- Location collection will use city/region level, not precise GPS coordinates during onboarding

## Scope Boundaries

**In Scope**:
- Phone number verification via SMS
- Profile photo upload with basic quality validation
- Vibe profile collection via conversational questions
- Activity selection with optional skill levels
- Location collection (city/region)
- Onboarding progress persistence

**Out of Scope**:
- Email-based authentication (phone-only for MVP)
- Social login (OAuth with Facebook, Google, etc.)
- Video verification or liveness detection
- Detailed availability/schedule collection (handled in separate feature)
- Friend invitations during onboarding
- Premium/subscription flows during onboarding
