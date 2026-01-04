# Feature Specification: Group Chat

**Feature Branch**: `006-group-chat`
**Created**: 2026-01-04
**Status**: Draft
**Input**: Group Chat: Event-based group messaging with persistent history and moderation

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Access Event Chat (Priority: P1)

When a user RSVPs to an event, they gain access to the event's group chat where attendees can coordinate and connect before the meetup.

**Why this priority**: Chat enables pre-event coordination (directions, what to bring, etc.) and builds anticipation. It's essential for events where logistics need discussion.

**Independent Test**: Can be tested by RSVPing to an event and accessing the group chat. Delivers event-linked group messaging.

**Acceptance Scenarios**:

1. **Given** a user has RSVPed to an event, **When** they view the event, **Then** they see a "Chat" button/tab
2. **Given** a user opens event chat, **When** the chat loads, **Then** they see messages from other attendees and the host
3. **Given** a user is not RSVPed, **When** they view the event, **Then** they cannot access the chat
4. **Given** a user cancels their RSVP, **When** they try to access chat, **Then** they lose access (but can see history if they rejoin)

---

### User Story 2 - Send and Receive Messages (Priority: P2)

Attendees can send text messages to the group chat. Messages appear in real-time for all participants, enabling natural conversation.

**Why this priority**: Messaging is the core chat functionality. Without real-time messaging, the feature doesn't work.

**Independent Test**: Can be tested by sending a message and seeing it appear for other attendees. Delivers real-time group messaging.

**Acceptance Scenarios**:

1. **Given** a user is in event chat, **When** they type and send a message, **Then** it appears in the chat for all attendees
2. **Given** another attendee sends a message, **When** user is viewing chat, **Then** the new message appears immediately
3. **Given** a user sends a message, **When** others view it, **Then** they see sender's name and profile photo
4. **Given** a user is offline, **When** they reconnect, **Then** they see all messages sent while away

---

### User Story 3 - View Persistent Chat History (Priority: P3)

All messages are permanently stored and accessible. Per constitution, "no disappearing messages" is a safety requirement.

**Why this priority**: Persistent chats provide evidence trail for safety. Users can also catch up on missed messages, and late-joiners see context.

**Independent Test**: Can be tested by leaving and re-entering chat, confirming history is preserved. Delivers permanent message storage.

**Acceptance Scenarios**:

1. **Given** a chat has existing messages, **When** a new attendee joins, **Then** they can scroll back to see full history
2. **Given** a user closes the app, **When** they reopen event chat, **Then** all messages are still visible
3. **Given** an event is completed, **When** user views past event, **Then** they can still read the chat history
4. **Given** messages exist, **When** any user views them, **Then** messages cannot be deleted or edited

---

### User Story 4 - Receive Chat Notifications (Priority: P4)

Users receive push notifications when new messages arrive in their event chats. This keeps them engaged and informed about coordination.

**Why this priority**: Users need to know when others message. Without notifications, chat becomes useless for time-sensitive coordination.

**Independent Test**: Can be tested by receiving a message and confirming notification appears. Delivers chat notification alerts.

**Acceptance Scenarios**:

1. **Given** a user is not in the app, **When** a message is sent in their event chat, **Then** they receive a push notification
2. **Given** a user has multiple event chats, **When** notifications arrive, **Then** each shows which event the message is from
3. **Given** a user wants quiet, **When** they mute an event chat, **Then** they stop receiving notifications for that chat
4. **Given** a user taps a notification, **When** app opens, **Then** they go directly to that event's chat

---

### User Story 5 - Report Inappropriate Messages (Priority: P5)

If a message is inappropriate, attendees can report it. Reports trigger review and possible action per the tiered consequences system.

**Why this priority**: Per constitution Trust & Safety principle, AI-assisted moderation must flag patterns. User reports are the input to this system.

**Independent Test**: Can be tested by reporting a message and confirming it reaches moderation queue. Delivers user reporting capability.

**Acceptance Scenarios**:

1. **Given** a user sees an inappropriate message, **When** they long-press/tap-hold, **Then** they see "Report" option
2. **Given** a user reports a message, **When** they confirm, **Then** the report is submitted with message context
3. **Given** a message is reported, **When** moderators review, **Then** they see the message, sender, and reporter
4. **Given** multiple reports on same user, **When** pattern detected, **Then** tiered consequences may apply (per constitution)

---

### User Story 6 - Host Moderation Tools (Priority: P6)

Event hosts have additional powers to moderate their event's chat, including removing disruptive attendees from the event.

**Why this priority**: Hosts are responsible for event quality. They need tools to maintain safe, productive conversations.

**Independent Test**: Can be tested by host removing an attendee and confirming chat access is revoked. Delivers host moderation.

**Acceptance Scenarios**:

1. **Given** a host views their event chat, **When** they see a problematic message, **Then** they can remove the attendee from the event
2. **Given** a host removes an attendee, **When** action completes, **Then** attendee's RSVP is canceled and chat access revoked
3. **Given** a host removes someone, **When** the attendee views their events, **Then** they see "Removed by host" status
4. **Given** removal patterns for a user, **When** system analyzes, **Then** user's trust score is affected

---

### Edge Cases

- What happens when a user is removed mid-conversation? Their access ends immediately; existing messages remain visible
- What happens when an event is canceled? Chat remains accessible for coordination around cancellation; eventually archived
- What happens when chat volume is very high? Implement load-more pagination; newest messages shown first
- What happens if user tries to send inappropriate content? Real-time content filtering may block or flag; pattern analysis triggers review
- What happens when an attendee blocks another? They still see each other in group chat (it's group-based, not 1:1)
- What happens when event reaches 20+ attendees? Chat may become noisy; consider future feature for sub-groups

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST create a group chat automatically when an event is created
- **FR-002**: System MUST grant chat access to all RSVPed attendees and the host
- **FR-003**: System MUST revoke chat access when RSVP is canceled or attendee is removed
- **FR-004**: System MUST deliver messages in real-time to all active participants
- **FR-005**: System MUST persist all messages permanently (no deletion, no editing)
- **FR-006**: System MUST show chat history to new attendees when they RSVP
- **FR-007**: System MUST send push notifications for new messages (unless muted)
- **FR-008**: System MUST allow users to mute/unmute specific event chats
- **FR-009**: System MUST provide message reporting capability for all users
- **FR-010**: System MUST give hosts ability to remove attendees from event (and chat)
- **FR-011**: System MUST track removal patterns for trust scoring
- **FR-012**: System MUST archive chat history after event completion (accessible but inactive)

### Key Entities

- **Event Chat**: Represents the group conversation for an event; has linked event, participants, and message history
- **Chat Message**: Represents a single message; has sender, timestamp, content; immutable once sent
- **Chat Participant**: Represents a user's access to an event chat; has status (active/revoked/muted)
- **Message Report**: Represents a user's report of a message; has reporter, message, reason, and resolution status
- **Chat Notification**: Represents push notification for new message; has target user, message preview

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Messages appear for all participants within 2 seconds of sending
- **SC-002**: 80% of event chats have at least 3 messages before the event
- **SC-003**: Average event chat has 10+ messages for events with 6+ attendees
- **SC-004**: Less than 1% of messages are reported as inappropriate
- **SC-005**: 90% of users read chat at least once before attending event
- **SC-006**: Host removal rate is under 2% of total attendees
- **SC-007**: Push notification click-through rate is 30%+

## Assumptions

- Chat uses managed chat provider per constitution (not built from scratch)
- Messages are text-only for MVP (no images, no reactions, no replies)
- Chat is group-only (no 1:1 DMs between attendees for MVP)
- All messages are visible to all participants (no private sub-chats)
- Chat remains active until 24 hours after event ends, then archived
- Archived chats are read-only accessible from event history

## Scope Boundaries

**In Scope**:
- Event-linked group chats
- Real-time messaging
- Persistent message history
- Push notifications with mute option
- Message reporting
- Host moderation (removal)

**Out of Scope**:
- 1:1 direct messaging between users
- Image/media sharing
- Message reactions (emoji)
- Threaded replies
- Read receipts
- Typing indicators
- Voice/video chat
- AI-powered auto-moderation (reports go to queue for MVP)
