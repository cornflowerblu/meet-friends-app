# Feature Specification: RSVP & Attendance

**Feature Branch**: `005-rsvp-attendance`
**Created**: 2026-01-04
**Status**: Draft
**Input**: RSVP and Attendance: Join events, manage RSVPs, waitlists, and attendance confirmation

## User Scenarios & Testing *(mandatory)*

### User Story 1 - RSVP to an Event (Priority: P1)

A user viewing an event can commit to attending with a single tap. This is the critical action that converts browsing into actual meetups.

**Why this priority**: RSVPs are the bridge between discovery and attendance. Without this, events never happen. This directly enables the IRL-First principle.

**Independent Test**: Can be tested by RSVPing to an event and confirming the user appears on the attendee list. Delivers committed attendance.

**Acceptance Scenarios**:

1. **Given** a user views an event with available spots, **When** they tap "I'm In", **Then** they are added to the attendee list
2. **Given** a user RSVPs successfully, **When** confirmation appears, **Then** they see the event added to "My Events"
3. **Given** a user RSVPs, **When** the host and other attendees view the event, **Then** they see the new attendee
4. **Given** a user has already RSVPed, **When** they view the event, **Then** the button shows "You're Going" instead of "I'm In"

---

### User Story 2 - Cancel RSVP (Priority: P2)

A user who can no longer attend can cancel their RSVP. This frees up the spot for others and maintains accurate attendance counts.

**Why this priority**: Flaking destroys trust and event quality. Making cancellation easy reduces no-shows by encouraging honest communication.

**Independent Test**: Can be tested by canceling an RSVP and verifying the spot is freed. Delivers clean cancellation with spot release.

**Acceptance Scenarios**:

1. **Given** a user is RSVPed to an event, **When** they tap "Cancel RSVP", **Then** they are removed from the attendee list
2. **Given** a user cancels, **When** confirming, **Then** they see a prompt explaining the spot will open for others
3. **Given** a user cancels close to event time (within 24 hours), **When** they confirm, **Then** they see a gentle reminder about reliability
4. **Given** the event had a waitlist, **When** a spot opens, **Then** the next waitlisted person is notified and auto-promoted

---

### User Story 3 - Join Waitlist (Priority: P3)

When an event is at capacity, a user can join a waitlist to be notified if a spot opens. This maintains interest and fills canceled spots quickly.

**Why this priority**: Full events are a good problem. Waitlists capture demand and ensure cancellations don't leave empty spots.

**Independent Test**: Can be tested by joining a waitlist and being promoted when a spot opens. Delivers waitlist management.

**Acceptance Scenarios**:

1. **Given** an event is at capacity, **When** user views the event, **Then** they see "Join Waitlist" instead of "I'm In"
2. **Given** a user joins the waitlist, **When** a spot opens, **Then** they receive a notification with time-limited claim window
3. **Given** a waitlisted user doesn't claim within time limit, **When** window expires, **Then** the next person on waitlist is notified
4. **Given** a user is on the waitlist, **When** they view the event, **Then** they see their position (e.g., "3rd on waitlist")

---

### User Story 4 - View My Events (Priority: P4)

A user can see all events they've RSVPed to, organized by date. This is their central hub for upcoming and past meetups.

**Why this priority**: Users need easy access to their commitments. A clear "My Events" view reduces missed events and builds the habit of checking the app.

**Independent Test**: Can be tested by viewing My Events and confirming all RSVPed events appear. Delivers personal event calendar.

**Acceptance Scenarios**:

1. **Given** a user has RSVPed to events, **When** they open "My Events", **Then** they see upcoming events sorted by date
2. **Given** an event is today, **When** viewing My Events, **Then** the event is highlighted with "Today" badge
3. **Given** a user has attended past events, **When** they view history tab, **Then** they see completed events
4. **Given** an event was canceled, **When** viewing My Events, **Then** it shows with "Canceled" status

---

### User Story 5 - Confirm Attendance (Priority: P5)

The day of the event, a user confirms they're still planning to attend. This reduces uncertainty for the host and other attendees.

**Why this priority**: Day-of confirmation increases show rates. Hosts gain confidence; attendees commit more firmly after confirming.

**Independent Test**: Can be tested by confirming attendance on event day. Delivers day-of confirmation flow.

**Acceptance Scenarios**:

1. **Given** an event is today, **When** user opens the app, **Then** they see a reminder to confirm attendance
2. **Given** a user confirms, **When** they tap "Still Going", **Then** their status updates to "Confirmed" visible to host
3. **Given** a user can't make it, **When** they indicate, **Then** cancellation flow starts and spot opens
4. **Given** multiple attendees confirm, **When** host views attendees, **Then** they see confirmation status for each

---

### User Story 6 - Post-Event Trust Scoring (Priority: P6)

After an event, attendees are asked if they'd meet the other attendees again. This private trust signal informs future matching.

**Why this priority**: Per constitution, private trust scoring ("Would you meet again?") must inform matching. This closes the feedback loop.

**Independent Test**: Can be tested by completing post-event survey. Delivers private trust signal collection.

**Acceptance Scenarios**:

1. **Given** an event has completed, **When** user opens app within 24 hours, **Then** they see post-event prompt
2. **Given** a user views the prompt, **When** they respond, **Then** they give thumbs up/down per attendee they interacted with
3. **Given** a user gives feedback, **When** submitted, **Then** feedback is stored privately (not visible to rated users)
4. **Given** a user skips feedback, **When** they dismiss, **Then** they can access it later from event history

---

### Edge Cases

- What happens when user RSVPs to overlapping events? Allow it with a warning; user manages their own schedule
- What happens when event is canceled after RSVP? User receives notification; event moves to "Canceled" in My Events
- What happens when user's account is suspended? Existing RSVPs are hidden from hosts; user cannot RSVP to new events
- What happens when waitlist claim window expires and user didn't see notification? Spot goes to next person; user remains on waitlist at lower priority
- What happens when all waitlisted users decline? Event shows spots available again for new RSVPs
- What happens when a user consistently no-shows? System tracks reliability; low-reliability users may have reduced visibility in matching

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow one-tap RSVP to events with available capacity
- **FR-002**: System MUST update attendee count in real-time when RSVPs change
- **FR-003**: System MUST allow RSVP cancellation at any time before event starts
- **FR-004**: System MUST manage waitlist with fair ordering (first-come-first-served)
- **FR-005**: System MUST notify waitlisted users when spots open with 4-hour claim window
- **FR-006**: System MUST auto-promote next waitlisted user if claim window expires
- **FR-007**: System MUST provide "My Events" view showing all upcoming RSVPs
- **FR-008**: System MUST send day-of confirmation prompts for upcoming events
- **FR-009**: System MUST track confirmed vs. unconfirmed attendees for host visibility
- **FR-010**: System MUST collect post-event trust signals privately within 24 hours
- **FR-011**: System MUST track user reliability (RSVPs vs. confirmed attendance vs. no-shows)
- **FR-012**: System MUST notify hosts when minimum capacity is reached or at risk

### Key Entities

- **RSVP**: Represents commitment to attend; has status (confirmed/pending/canceled/no-show), timestamps, and linked user/event
- **Waitlist Position**: Represents queue position for full events; has order, claim status, and expiration time
- **My Events**: Represents user's personalized view of upcoming and past RSVPs
- **Attendance Confirmation**: Represents day-of confirmation; has confirmed status and timestamp
- **Trust Signal**: Represents private post-event feedback; has rating per attendee, never displayed publicly
- **User Reliability Score**: Represents attendance track record; affects matching priority, never shown to user

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: RSVP action completes in under 2 seconds with confirmation
- **SC-002**: 85% of RSVPed users confirm attendance on event day
- **SC-003**: No-show rate (RSVP but didn't attend) is under 15%
- **SC-004**: Waitlist-to-RSVP conversion rate is 70%+ when spots open
- **SC-005**: 60% of attendees complete post-event trust feedback
- **SC-006**: Average claim window response time is under 2 hours
- **SC-007**: Cancellation rate increases by 20% vs. baseline (indicating honest communication vs. ghosting)

## Assumptions

- Users receive push notifications for waitlist promotions and confirmations
- Waitlist claim window is 4 hours; configurable based on event timing
- Trust signals are collected via simple thumbs up/down per attendee
- User reliability affects matching but is never displayed to the user
- Events are considered "attended" if user confirms day-of; no check-in at venue required
- Post-event feedback is optional but encouraged

## Scope Boundaries

**In Scope**:
- RSVP and cancellation
- Waitlist management
- My Events view
- Day-of confirmation
- Post-event trust collection
- Reliability tracking

**Out of Scope**:
- Check-in at venue (QR codes, geofencing)
- Guest invitations ("+1" functionality)
- RSVP on behalf of others
- Paid event ticketing/payments
- Calendar sync (export to Google Calendar, etc.)
- Automated reminders beyond day-of
