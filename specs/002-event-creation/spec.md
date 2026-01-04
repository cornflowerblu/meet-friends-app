# Feature Specification: Event Creation

**Feature Branch**: `002-event-creation`
**Created**: 2026-01-04
**Status**: Draft
**Input**: Event Creation: Create events with activity, time, location, vibe labels, and capacity limits for the friend-matching app

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create a Basic Event (Priority: P1)

An onboarded user creates an event by selecting an activity, setting a date/time, choosing a location, and publishing it for others to discover. This is the foundational flow that enables the core "real meetups" promise.

**Why this priority**: Without event creation, there are no meetups. This is the single most critical feature for proving the IRL-First principle. Events are the product.

**Independent Test**: Can be fully tested by creating an event and verifying it appears in the system. Delivers a published event visible to other users.

**Acceptance Scenarios**:

1. **Given** an onboarded user with completed profile, **When** they tap "Create Event", **Then** they see a guided event creation flow
2. **Given** a user is creating an event, **When** they select an activity from their interests, **Then** the activity is set for the event
3. **Given** a user is creating an event, **When** they set date, time, and duration, **Then** the event schedule is saved (duration defaults to 90 minutes)
4. **Given** a user is creating an event, **When** they specify a location (venue name, address, or general area), **Then** the location is attached to the event
5. **Given** a user has filled required fields, **When** they publish the event, **Then** the event becomes discoverable by other users

---

### User Story 2 - Set Event Vibe Labels (Priority: P2)

An event creator adds vibe labels to clearly communicate the expected atmosphere. Labels like "casual", "no try-hards", "beginners welcome", or "competitive" help attendees self-select and reduce mismatched expectations.

**Why this priority**: Vibe labeling is the key differentiator from generic event apps. Clear vibes reduce bad first meetups and increase repeat attendance by setting accurate expectations.

**Independent Test**: Can be tested by adding vibe labels to an event and verifying they display on the event listing. Delivers events with clear, visible vibe expectations.

**Acceptance Scenarios**:

1. **Given** a user is creating an event, **When** they reach the vibe selection step, **Then** they see curated vibe labels relevant to the activity
2. **Given** a user is setting vibes, **When** they select 1-3 vibe labels, **Then** the labels are attached to the event
3. **Given** a user wants custom vibe messaging, **When** they add a short description, **Then** it appears alongside the standard vibe labels
4. **Given** an event has vibe labels, **When** other users view the event, **Then** the vibe labels are prominently displayed

---

### User Story 3 - Set Capacity Limits (Priority: P3)

An event creator sets minimum and maximum participant limits. Events are optimized for 4-8 people to maintain intimacy and ensure everyone can interact meaningfully.

**Why this priority**: Group size directly affects meetup quality. Too many people dilutes connection; too few risks cancellation. Capacity controls are essential for the "certainty of a real meetup" promise.

**Independent Test**: Can be tested by setting capacity limits and verifying RSVPs are limited accordingly. Delivers events with controlled group sizes.

**Acceptance Scenarios**:

1. **Given** a user is creating an event, **When** they set capacity, **Then** they can specify minimum (default: 4) and maximum (default: 8) participants
2. **Given** an event has reached maximum capacity, **When** another user tries to join, **Then** they are added to a waitlist or shown the event is full
3. **Given** an event is below minimum capacity close to the event time, **When** the threshold is not met, **Then** the host is notified to decide whether to proceed or reschedule
4. **Given** capacity settings, **When** users view the event, **Then** they see current attendance count and remaining spots

---

### User Story 4 - Edit and Cancel Events (Priority: P4)

An event creator can modify event details or cancel the event entirely. Changes notify affected attendees, and cancellations are handled gracefully to maintain trust.

**Why this priority**: Plans change. The ability to edit or cancel prevents ghost events and maintains platform credibility. Proper cancellation handling protects the community's trust.

**Independent Test**: Can be tested by editing event details and canceling an event, verifying attendees are notified. Delivers editable events with proper notification flow.

**Acceptance Scenarios**:

1. **Given** a user has created an event, **When** they access event management, **Then** they can edit date, time, location, vibe labels, and capacity
2. **Given** a user edits an event with RSVPs, **When** they save changes, **Then** all attendees receive notification of the changes
3. **Given** a user cancels an event, **When** they confirm cancellation, **Then** all attendees are notified and the event is removed from discovery
4. **Given** an event was canceled, **When** attendees view their upcoming events, **Then** the canceled event is marked as canceled with reason (if provided)

---

### User Story 5 - Repeat and Template Events (Priority: P5)

A frequent host can create recurring events or save event templates to quickly spin up similar events. This reduces friction for active community builders.

**Why this priority**: Power users (organizers) drive liquidity. Making it easy to create recurring events encourages consistent hosting and builds community rhythm.

**Independent Test**: Can be tested by creating a template and using it to spawn a new event. Delivers reusable event patterns for frequent hosts.

**Acceptance Scenarios**:

1. **Given** a user is creating an event, **When** they enable "repeat weekly", **Then** the system creates a series of events with the same details
2. **Given** a user has hosted an event, **When** they view past events, **Then** they can "create similar" to pre-fill a new event
3. **Given** a user creates a template, **When** they access "My Templates", **Then** they can quickly launch a new event from saved settings

---

### Edge Cases

- What happens when a host sets a date in the past? System prevents past dates; minimum lead time is 2 hours from now
- What happens when the host's account is suspended after creating an event? Event is hidden from discovery; attendees are notified of potential changes
- What happens if location services are denied? User can manually enter address or venue name
- What happens if an event has 0 RSVPs when the event time arrives? Event is auto-archived; host receives suggestions for improving visibility
- What happens if a host creates many events that never reach minimum capacity? Host receives guidance; pattern may affect host's visibility in recommendations

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow onboarded users to create events with activity, date/time, and location
- **FR-002**: System MUST provide curated vibe labels relevant to each activity type
- **FR-003**: System MUST enforce capacity limits (min: 2, max: 20, defaults: 4-8)
- **FR-004**: System MUST notify all RSVPed attendees when event details change or event is canceled
- **FR-005**: System MUST prevent events scheduled less than 2 hours in the future
- **FR-006**: System MUST display current RSVP count and remaining capacity on event listings
- **FR-007**: System MUST support event series (recurring events) with weekly/biweekly/monthly options
- **FR-008**: System MUST allow hosts to save event templates for future use
- **FR-009**: System MUST default event duration to 90 minutes (editable by host)
- **FR-010**: System MUST require events to be public (no private/invite-only events for MVP)
- **FR-011**: System MUST enforce "not a dating app" framing in event descriptions and prompts
- **FR-012**: System MUST track host reliability (events created vs. events that happened)

### Key Entities

- **Event**: Represents a scheduled meetup; has activity, date/time, duration, location, vibe labels, capacity limits, host, and status (draft/published/canceled/completed)
- **Vibe Label**: Represents an atmosphere descriptor; has name, category, and associated activities; system-defined set with optional custom text
- **Event Series**: Represents recurring events; links multiple Event instances with shared base settings
- **Event Template**: Represents saved event settings; owned by a user; can be used to quickly create new events
- **RSVP**: Represents a user's commitment to attend; links User to Event with status (confirmed/waitlisted/canceled)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can create and publish an event in under 3 minutes
- **SC-002**: 90% of published events include at least 1 vibe label
- **SC-003**: 70% of events reach minimum capacity before event time
- **SC-004**: Less than 10% of events are canceled after having RSVPs
- **SC-005**: Hosts who create one event create a second event within 30 days at rate of 40%+
- **SC-006**: 95% of event edits/cancellations result in attendee notifications within 5 minutes
- **SC-007**: Average event capacity is between 4-8 attendees (matching ideal first meetup size)

## Assumptions

- Events are public by default; private/invite-only events are out of scope for MVP
- Location can be a venue name, address, or general area (precise GPS coordinates not required)
- Vibe labels are system-curated; user-generated labels are out of scope for MVP
- Event creation is only available to users who completed onboarding
- All events must have at least one activity associated
- Hosts must have a verified profile (phone + photo) to create events
- Time zones are handled based on event location, not user location

## Scope Boundaries

**In Scope**:
- Basic event creation (activity, time, location)
- Vibe label selection from curated list
- Capacity limits (min/max attendees)
- Event editing and cancellation with notifications
- Recurring events (weekly/biweekly/monthly)
- Event templates for frequent hosts
- RSVP count and capacity display

**Out of Scope**:
- Private/invite-only events
- Paid events or ticketing
- Event chat (separate feature)
- Venue partnerships or integrations
- Weather-based cancellation automation
- Co-host functionality
- Event photos/media attachments
- Post-event feedback collection (separate feature)
