# Feature Specification: Event Discovery

**Feature Branch**: `003-event-discovery`
**Created**: 2026-01-04
**Status**: Draft
**Input**: Event Discovery: Browse and search events by activity, location, and vibe filters

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Browse Nearby Events (Priority: P1)

An onboarded user opens the app and sees a list of upcoming events near their location, filtered to activities they're interested in. This is the primary discovery mechanism that connects users to real meetups.

**Why this priority**: Without discovery, events have no attendees. This is the critical path from "app user" to "event attendee" that validates the IRL-First principle.

**Independent Test**: Can be fully tested by viewing the event feed and confirming relevant events appear. Delivers a browsable list of upcoming events.

**Acceptance Scenarios**:

1. **Given** an onboarded user opens the app, **When** they view the main feed, **Then** they see events within their configured distance radius
2. **Given** a user has activity interests set, **When** they view the feed, **Then** events matching their interests are prioritized at the top
3. **Given** events exist in the user's area, **When** they browse the feed, **Then** events are sorted by date (soonest first) within relevance tiers
4. **Given** a user views the feed, **When** they scroll, **Then** they see event cards showing activity, date/time, location, vibe labels, and spots remaining

---

### User Story 2 - Filter Events by Activity (Priority: P2)

A user filters the event feed to show only events for a specific activity (e.g., "show me only golf events"). This enables focused discovery when users know what they want to do.

**Why this priority**: Users often have a specific activity in mind. Activity filtering reduces noise and increases the likelihood of finding a relevant event quickly.

**Independent Test**: Can be tested by applying an activity filter and verifying only matching events appear. Delivers filtered results by activity type.

**Acceptance Scenarios**:

1. **Given** a user is viewing the event feed, **When** they tap the activity filter, **Then** they see a list of available activities with event counts
2. **Given** a user selects "Golf" as a filter, **When** the filter is applied, **Then** only golf events appear in the feed
3. **Given** a user has multiple activity interests, **When** they apply multiple filters, **Then** events matching any selected activity appear
4. **Given** a user has an active filter, **When** they clear the filter, **Then** all relevant events reappear

---

### User Story 3 - Filter Events by Vibe (Priority: P3)

A user filters events by vibe labels to find events matching their preferred atmosphere (e.g., "casual only", "beginners welcome"). This leverages the app's key differentiator.

**Why this priority**: Vibe matching is the core value proposition. Allowing users to filter by vibe ensures they find events where they'll feel comfortable and have a good experience.

**Independent Test**: Can be tested by applying vibe filters and verifying events with matching labels appear. Delivers vibe-filtered discovery.

**Acceptance Scenarios**:

1. **Given** a user is viewing the event feed, **When** they tap the vibe filter, **Then** they see available vibe labels (casual, competitive, beginners welcome, etc.)
2. **Given** a user selects "Casual" vibe filter, **When** the filter is applied, **Then** only events with the "casual" label appear
3. **Given** a user selects multiple vibe filters, **When** filters are applied, **Then** events matching any selected vibe appear
4. **Given** no events match the vibe filter, **When** user views results, **Then** they see a message suggesting they broaden filters or check back later

---

### User Story 4 - View Event Details (Priority: P4)

A user taps on an event card to see full details including description, host information, attendee list, and vibe context. This provides the information needed to decide whether to RSVP.

**Why this priority**: Users need sufficient information to commit to attending. Event details build trust and set expectations for what the meetup will be like.

**Independent Test**: Can be tested by opening an event and verifying all expected details are displayed. Delivers complete event information view.

**Acceptance Scenarios**:

1. **Given** a user taps an event card, **When** the detail view opens, **Then** they see activity, date, time, duration, and location
2. **Given** a user views event details, **When** they scroll, **Then** they see host profile (photo, name, hosting history) and vibe labels
3. **Given** a user views event details, **When** they check attendance, **Then** they see current attendee count, spots remaining, and attendee preview (photos of first few RSVPs)
4. **Given** an event has a description, **When** the user reads it, **Then** the host's custom vibe message and any additional context is visible

---

### User Story 5 - Search for Events (Priority: P5)

A user searches for events using keywords (activity name, location, host name). This supports users who know exactly what they're looking for.

**Why this priority**: Search provides an alternative discovery path for power users and supports specific queries that filters can't handle.

**Independent Test**: Can be tested by entering search terms and verifying matching events appear. Delivers keyword-based event search.

**Acceptance Scenarios**:

1. **Given** a user taps the search bar, **When** they enter "pickleball downtown", **Then** they see events matching both terms
2. **Given** a user searches by venue name, **When** results load, **Then** events at that venue appear
3. **Given** a search returns no results, **When** user views the page, **Then** they see suggestions (broaden search, try different terms, explore nearby)
4. **Given** a user has recent searches, **When** they tap search, **Then** they see their search history for quick re-access

---

### Edge Cases

- What happens when there are no events in the user's area? Show message encouraging them to create an event or expand their search radius
- What happens when all nearby events are at capacity? Show full events with "Join Waitlist" option; suggest expanding filters
- What happens when user's location is unavailable? Prompt to enable location or manually set a search area
- What happens when a user searches for an activity not in the curated list? Show "no results" with suggestion to request the activity be added
- What happens if an event is canceled while user is viewing details? Show cancellation notice and return to feed

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST display events within user's configured distance radius (default: 25 miles)
- **FR-002**: System MUST prioritize events matching user's activity interests in the feed
- **FR-003**: System MUST allow filtering by activity type (single or multiple selection)
- **FR-004**: System MUST allow filtering by vibe labels (single or multiple selection)
- **FR-005**: System MUST sort events by date within relevance tiers
- **FR-006**: System MUST display event cards with: activity, date/time, location, vibe labels, spots remaining
- **FR-007**: System MUST show event details including host profile, full description, and attendee preview
- **FR-008**: System MUST provide keyword search across activity, location, and host name
- **FR-009**: System MUST show "no results" state with actionable suggestions when filters/search yield nothing
- **FR-010**: System MUST hide events that are at capacity unless user specifically requests to see full events
- **FR-011**: System MUST update event availability in real-time (spots remaining, cancellations)

### Key Entities

- **Event Feed**: Represents the personalized list of discoverable events; influenced by location, interests, and filters
- **Event Card**: Represents the summary view of an event; includes key info for quick scanning
- **Event Detail**: Represents the full view of an event; includes all information needed to decide to RSVP
- **Filter State**: Represents active filters (activity, vibe, distance); persists during session
- **Search Query**: Represents user search input; includes keyword and search history

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users find and view an event within 30 seconds of opening the app
- **SC-002**: 70% of users who view an event detail page proceed to RSVP or waitlist
- **SC-003**: Average session shows 5+ events viewed before first RSVP
- **SC-004**: Search returns relevant results for 90% of queries (measured by click-through)
- **SC-005**: Filter usage leads to RSVP within same session 60% of the time
- **SC-006**: Less than 5% of users see "no events" state after applying reasonable filters
- **SC-007**: Event feed loads within 2 seconds on standard mobile connection

## Assumptions

- Users have granted location permission or set a manual location during onboarding
- Default search radius is 25 miles; users can adjust in settings
- Event feed refreshes automatically when user pulls down or returns to app
- Filters persist during a session but reset when app is closed
- Search history is stored locally on device (not synced across devices)
- Events are considered "nearby" based on event location, not venue location

## Scope Boundaries

**In Scope**:
- Event feed with personalized ordering
- Activity and vibe filters
- Keyword search
- Event detail view with host and attendee info
- Real-time availability updates
- "No results" helpful messaging

**Out of Scope**:
- Map-based discovery (future feature)
- Calendar integration
- Event recommendations based on friends' RSVPs
- Push notifications for new matching events
- Saved/bookmarked events
- Event sharing to social media
