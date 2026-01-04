# Feature Specification: Vibe Matching

**Feature Branch**: `004-vibe-matching`
**Created**: 2026-01-04
**Status**: Draft
**Input**: Vibe Matching: Match users based on behavioral compatibility with why-we-matched explanations

## User Scenarios & Testing *(mandatory)*

### User Story 1 - See Vibe Compatibility on Events (Priority: P1)

When viewing an event, a user sees a vibe compatibility indicator showing how well their behavioral preferences align with the event's vibe and existing attendees. This helps them choose events where they'll feel comfortable.

**Why this priority**: The core value proposition is "same vibe." Without visible compatibility signals, users can't make informed decisions about which events to attend.

**Independent Test**: Can be tested by viewing events and confirming compatibility indicators appear. Delivers visible vibe compatibility on event cards and details.

**Acceptance Scenarios**:

1. **Given** a user views an event card, **When** they look at the vibe section, **Then** they see a compatibility indicator (e.g., "Great match", "Good match", "Different vibe")
2. **Given** an event has attendees, **When** a user views compatibility, **Then** it factors in both event vibe labels and existing attendee vibes
3. **Given** a user with "casual" vibe preference, **When** they view a "casual" labeled event, **Then** compatibility shows higher than for "competitive" events
4. **Given** a user has incomplete vibe profile, **When** they view compatibility, **Then** they see a prompt to complete their profile for better matching

---

### User Story 2 - See Why We Matched Explanations (Priority: P2)

A user can see human-readable explanations of why they're compatible with an event or other attendees. This builds trust by making the matching logic transparent and understandable.

**Why this priority**: Per constitution principle "AI as Support," match explanations must be shown to build trust. Users need to understand why they're being matched, not just trust a black box.

**Independent Test**: Can be tested by opening match explanation and verifying clear, understandable reasoning appears. Delivers transparent matching logic.

**Acceptance Scenarios**:

1. **Given** a user views an event with high compatibility, **When** they tap "Why this match?", **Then** they see specific reasons (e.g., "You both prefer casual meetups", "Similar energy levels")
2. **Given** a user views match explanation, **When** they read it, **Then** no internal scores or personality labels are shown (only behavioral descriptions)
3. **Given** compatibility is based on multiple factors, **When** user views explanation, **Then** they see 2-3 key alignment points in plain language
4. **Given** low compatibility, **When** user views explanation, **Then** they see honest difference summary without judgment (e.g., "This event is more competitive than your usual preference")

---

### User Story 3 - Adjust Matching Preferences (Priority: P3)

A user can adjust how much weight is given to vibe vs. distance vs. activity when calculating compatibility. This gives power users control over their discovery experience.

**Why this priority**: Per constitution, users must be able to adjust weighting. Different users prioritize different factors; flexibility increases satisfaction.

**Independent Test**: Can be tested by adjusting sliders and verifying event rankings change accordingly. Delivers customizable matching weights.

**Acceptance Scenarios**:

1. **Given** a user opens matching preferences, **When** they view settings, **Then** they see sliders for vibe, distance, and activity importance
2. **Given** a user increases "vibe" importance, **When** they return to event feed, **Then** high-vibe-match events rank higher regardless of distance
3. **Given** a user prioritizes "distance", **When** they view feed, **Then** nearby events with lower vibe match may rank higher than distant perfect matches
4. **Given** a user resets preferences, **When** they confirm reset, **Then** defaults are restored (balanced weighting)

---

### User Story 4 - View Attendee Vibe Compatibility (Priority: P4)

Before joining an event, a user can see vibe compatibility with other confirmed attendees (without seeing their actual vibe scores). This helps predict group dynamics.

**Why this priority**: Knowing you'll get along with existing attendees increases confidence to RSVP. Group compatibility matters as much as event compatibility.

**Independent Test**: Can be tested by viewing attendee list with compatibility indicators. Delivers group compatibility preview.

**Acceptance Scenarios**:

1. **Given** an event has confirmed attendees, **When** user views attendee preview, **Then** they see compatibility indicator for each visible attendee
2. **Given** multiple attendees, **When** user views overall compatibility, **Then** they see group average or "mostly compatible" summary
3. **Given** user taps an attendee, **When** they view profile, **Then** they see shared activities and general vibe alignment (not scores)
4. **Given** an attendee has low compatibility, **When** user views explanation, **Then** they understand it's different preferences, not "bad person"

---

### Edge Cases

- What happens when a user has no vibe data? Show events without compatibility scoring; prompt to complete profile
- What happens when an event has no attendees yet? Base compatibility only on event vibe labels and host vibe
- What happens when all nearby events have low compatibility? Show events anyway with honest compatibility info; don't hide opportunities
- What happens when matching factors conflict (high vibe match but far away)? Apply user's preference weighting; show explanation of tradeoffs
- What happens when user's vibe profile changes significantly? Recalculate compatibility for upcoming events; notify if any RSVPs now have low compatibility

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST calculate and display vibe compatibility on all event cards
- **FR-002**: System MUST show compatibility using friendly terms (Great/Good/Fair/Different), never numerical scores
- **FR-003**: System MUST provide "Why this match?" explanations in plain language
- **FR-004**: System MUST NOT expose internal vibe scores, labels, or personality classifications
- **FR-005**: System MUST allow users to adjust vibe/distance/activity weighting with sliders
- **FR-006**: System MUST factor attendee vibes into event compatibility when attendees exist
- **FR-007**: System MUST show attendee-level compatibility on event detail pages
- **FR-008**: System MUST explain low compatibility honestly without negative framing
- **FR-009**: System MUST prompt incomplete vibe profiles to add data for better matching
- **FR-010**: System MUST recalculate compatibility when user updates their vibe profile

### Key Entities

- **Vibe Compatibility Score**: Represents calculated match between user and event/attendees; internal use only, never displayed
- **Match Explanation**: Represents human-readable reasoning for compatibility; shown to users; generated from vibe dimensions
- **Matching Preference**: Represents user's weighting for vibe/distance/activity; affects feed ranking
- **Vibe Dimension**: Represents one of 6 behavioral tendencies (casual/competitive, etc.); collected during onboarding
- **Group Compatibility**: Represents aggregate compatibility with event attendees; summarized for user

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 80% of users who tap "Why this match?" report the explanation as helpful (in-app feedback)
- **SC-002**: Users with high compatibility events have 2x higher RSVP rate than low compatibility
- **SC-003**: Less than 5% of users report confusion about matching in support channels
- **SC-004**: 40% of power users adjust matching preferences at least once
- **SC-005**: Post-event satisfaction is 30% higher when pre-event compatibility was "Great match"
- **SC-006**: Repeat attendance rate is 25% higher for high-compatibility first meetups
- **SC-007**: Users complete vibe profile at 90%+ rate when prompted due to incomplete data

## Assumptions

- Vibe compatibility is calculated using the 6 behavioral dimensions from onboarding
- Initial matching uses rule-based scoring; ML optimization comes later per constitution
- Compatibility factors in both user preferences and observed behavior over time
- Match explanations are generated from templates, not per-request LLM calls (cost control)
- Group compatibility is average of individual attendee compatibilities
- Compatibility updates in near-real-time as attendees join/leave events

## Scope Boundaries

**In Scope**:
- Vibe compatibility indicators on events
- "Why this match?" explanations
- Preference sliders for matching weights
- Attendee compatibility preview
- Profile completion prompts

**Out of Scope**:
- User-to-user matching outside of events (friend suggestions)
- AI-powered natural language match explanations (use templates)
- Compatibility-based push notifications
- Historical compatibility analytics for users
- A/B testing different matching algorithms
